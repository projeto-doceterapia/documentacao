# 10.8 Integrações: WhatsApp e IA

> Fontes: repositório `estoque-wpp` (POC), `analise-gaps-whatsapp-confeitaria-final.md`, `prd-assistente-atualizacao-estoque-whatsapp.md` (v0.1, 05/10/2026) e Status Report (Semanas 9–15: arquitetura hexagonal, troca Gemini → GPT, MCP).

## 10.8.1 Estado atual: POC `estoque-wpp`

Serviço Node.js independente que recebe a foto de uma nota fiscal pelo WhatsApp (Twilio), extrai os itens com o Gemini e faz upsert em um MySQL próprio.

| Item | Detalhe |
|---|---|
| Stack | Node 18+, Express ^5.2.1, twilio ^6.0.0, @google/generative-ai ^0.24.1 (`gemini-2.5-flash`), mysql2 ^3.22.2, Swagger (`/api-docs`) |
| Entrada | `POST /webhook` (Twilio). Valida `NumMedia`/`MediaUrl0` e responde com TwiML imediato |
| IA | Prompt que devolve JSON com nome, quantidade, unidade, preço e data. Converte kg → g e L → ml e remove marcas |
| Persistência | Tabela `produtos` no schema `estoque_gemini`. Se o nome já existe, soma a quantidade. Se não existe, insere |
| Saída | Mensagem com o total de itens novos e atualizados |
| Configuração | `PORT`, `DB_*`, `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, `TWILIO_WHATSAPP_NUMBER`, `GEMINI_API_KEY` |
| Limitações | Não se integra ao core Java, grava direto em um banco paralelo, não pede confirmação da usuária, não tem idempotência, aceita qualquer remetente e não tem fila nem retry |

A sequência está em [10.3.6](03-uml.md#1036-sequência--entrada-de-estoque-por-nota-fiscal-poc).

## 10.8.2 Análise de gaps (POC → produto)

| Gap | Descrição | Prioridade |
|---|---|---|
| G1 | Envio outbound genérico (alertas e lembretes) | Quick win |
| G2 | API interna no serviço Node (`POST /internal/notifications`) para o core acionar | Quick win |
| G3 | Receber e classificar texto livre (PEDIDO, CONSULTA, OUTRO) | — |
| G4 | Modelo de dados para pedido vindo de texto | — |
| G5 | Consulta interna ao core | — |
| G6 | Controle de estoque estruturado no core | — |
| G7 | Trigger de alerta de estoque baixo | Quick win |
| G8 + G9 | Campo de agendamento de retirada ou entrega + scheduler de lembretes (`@Scheduled`) | Quick win |
| G10 | Log e idempotência de mensagens | — |
| G11 | Estratégia de comunicação entre o back-end e o serviço Node | — |

O volume estimado (cerca de 50 mensagens por dia) não justifica uma fila dedicada nesta fase.

## 10.8.3 Arquitetura alvo: assistente de estoque via WhatsApp (PRD)

Arquitetura **hexagonal**. O orquestrador define as portas (`ChannelPort`, porta de IA, porta do core), e os adaptadores podem ser trocados por configuração (`CHANNEL`, provedor de IA). Nada é gravado sem a confirmação explícita da beneficiária.

```mermaid
flowchart LR
    U["Beneficiária<br/>(WhatsApp)"]
    subgraph CANAL["Adaptadores de canal (CHANNEL)"]
        direction TB
        B["Baileys (Node)<br/>não oficial"]
        M["Meta Cloud API<br/>oficial"]
    end
    subgraph ORQ["Orquestrador (Python / FastAPI)"]
        direction TB
        IN["Entrada (webhooks)"]
        FSM["Máquina de estados<br/>da conversa"]
        IA["Porta de IA<br/>multimodal"]
        SCH["Agendador de<br/>notificações"]
        RTY["Worker de retry"]
    end
    R[("Redis<br/>estado, dedupe, locks, filas")]
    LLM["Provedor de IA<br/>(Gemini / GPT)"]
    CORE["Core Java<br/>(API Doceterapia)"]
    U <--> B
    U <--> M
    B <-->|HTTP interno| IN
    M <-->|webhook HTTPS| IN
    IN --> FSM
    FSM <--> R
    FSM --> IA <--> LLM
    FSM <--> CORE
    SCH <--> CORE
    SCH <--> R
    RTY <--> R
    RTY --> CORE
```

### Conversa como máquina de estados

```mermaid
stateDiagram-v2
    direction LR
    state "Sem fluxo ativo" as IDLE
    state "Extraindo (IA)" as EXTR
    state "Aguardando confirmação" as CONF
    state "Gravando no core" as GRAV
    state "Encerrado" as FIM
    [*] --> IDLE
    IDLE --> EXTR : compra recebida (imagem ou texto)
    EXTR --> CONF : leitura pronta
    EXTR --> FIM : falha da IA após retries
    CONF --> EXTR : ajuste (máximo 2)
    CONF --> FIM : 3º ajuste, cancelamento ou nota já registrada
    CONF --> GRAV : confirmação
    GRAV --> FIM : todos os itens resolvidos
    FIM --> IDLE : estado limpo e feedback enviado
    CONF --> IDLE : timeout de inatividade
```

### Endpoints que o core Java precisa expor

| # | Endpoint | Finalidade |
|---|---|---|
| 1 | `GET /v1/insumos` | Catálogo de insumos para o contexto da IA (id, nome, sinônimos e unidades) |
| 2 | `POST /v1/insumos` | Get-or-create de insumo novo (após aprovação) |
| 3 | `GET /v1/notas/{chave}/status` | Saber se a nota é inexistente, parcial ou completa |
| 4 | `POST /v1/estoque/movimentos` | Gravar **um** item por requisição, com `Idempotency-Key` |
| 5 | `GET /v1/notificacoes/agenda?de=&ate=` | Agenda de notificações a enviar |
| 6 | `POST /v1/notificacoes/{id}/confirmacao-envio` | Confirmação idempotente do envio |

### Regras e decisões principais

- **RN-01:** nada é gravado sem confirmação. **RN-03:** a entrada **soma** ao estoque e nunca sobrescreve. **RN-08:** no máximo 2 ajustes por leitura. **RN-14:** só o chat autorizado é atendido. **RN-10:** imagens e rascunhos não são persistidos (LGPD).
- **D1–D18:** uma única beneficiária, orquestrador em Python, canal intercambiável, Redis com AOF, LLM multimodal, sem MCP nesta versão, um request por item, idempotência e retry exponencial com jitter.
- **Riscos principais:**
  - Banimento no modo Baileys (aceito e documentado).
  - Alucinação da IA (mitigada pela confirmação humana).
  - Endpoints do core ainda inexistentes (bloqueante).
  - Mudança de preço da Meta.

### Próximos passos registrados (Status Report, Semana 14)

- Integrar a POC ao banco e à aplicação final (Felipe e Escudero).
- Trocar o modelo de IA de Gemini para GPT (sugestão de melhoria).
- Avaliar MCP para um assistente de IA em todo o sistema (desejável).
- Finalizar os diagramas de contêiner e de implantação e documentar a infraestrutura.

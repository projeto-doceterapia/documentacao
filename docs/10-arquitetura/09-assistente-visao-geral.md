# 10.9 Assistente de estoque via WhatsApp — visão geral

> **Situação:** implementado e em revisão nos pull requests dos repositórios de produto; ainda não implantado em produção.

> Esta e as próximas seções (10.9 a 10.14) documentam o **assistente de estoque via WhatsApp**. Diferente da prova de conceito descrita em [10.8](08-integracoes.md), aqui está o que **existe** no código: o ledger de estoque e os endpoints idempotentes no core Java, as notificações de negócio com stream em tempo real, o orquestrador em Python, o adaptador Baileys em Node e o ambiente de contêineres do repositório `infra`. Itens ainda não validados estão marcados como **pendente** com o motivo.

## Repositórios da entrega

| Repositório | Papel | Imagem publicada (GHCR) |
|---|---|---|
| `back-end` | Core Java: ledger, endpoints de estoque/notas/notificações, jobs `@Scheduled`, stream SSE | `ghcr.io/projeto-doceterapia/back-end` |
| `front-end` | SPA: topbar de notificações pela API e modal de configuração | `ghcr.io/projeto-doceterapia/front-end` |
| `banco-de-dados` | `scriptv4.sql` com as novas tabelas, empacotado em imagem MySQL | `ghcr.io/projeto-doceterapia/banco-de-dados` |
| `assistente-whatsapp/orquestrador` | Serviço Python (FastAPI) da conversa de compra e da entrega de notificações | `ghcr.io/projeto-doceterapia/orquestrador` |
| `assistente-whatsapp/adaptador-baileys` | Adaptador Node/TS do canal Baileys | `ghcr.io/projeto-doceterapia/adaptador-baileys` |
| `infra` | Docker Compose: único repositório necessário para subir o sistema | — (consome as imagens acima) |

O orquestrador é o **único cliente** do core para o fluxo do assistente; ele nunca acessa o banco direto. O adaptador fala só com o orquestrador, por HTTP interno assinado com HMAC.

## Contêineres (arquitetura implantada)

```mermaid
flowchart TB
    U["<b>Confeiteira</b><br/>[Person]<br/>navegador e WhatsApp"]:::person

    subgraph sys["Sistema Doceterapia [Software System]"]
        direction TB
        fe["<b>front-end</b><br/>[Container: nginx + SPA React]<br/>:3000, proxy /api"]:::container
        be["<b>back-end</b><br/>[Container: Java 21, Spring Boot 4]<br/>:8080 (interno), SSE, @Scheduled"]:::container
        db[("<b>banco-de-dados</b><br/>[Container: MySQL 8.4]<br/>scriptv4.sql")]:::container
        orq["<b>orquestrador</b><br/>[Container: Python 3.12, FastAPI]<br/>:8000 webhook"]:::container
        adp["<b>adaptador-baileys</b><br/>[Container: Node 22]<br/>perfil baileys"]:::poc
        rds[("<b>redis</b><br/>[Container: Redis 7, AOF everysec]<br/>estado, dedupe, filas, agenda")]:::container
    end

    meta["<b>Meta WhatsApp Cloud API</b><br/>[Software System]"]:::external
    gem["<b>Google Gemini</b><br/>[Software System]"]:::external
    ngrok["<b>ngrok</b><br/>[Software System]<br/>túnel só em dev/testes Cloud API"]:::poc

    U -- "HTTP :3000" --> fe
    fe -- "/api → REST + cookie JWT" --> be
    be -- "JDBC/JPA" --> db
    U <-. "WhatsApp (Baileys)" .-> adp
    U <-. "WhatsApp (Cloud API)" .-> meta
    adp <-- "HTTP interno (HMAC)" --> orq
    meta <-. "webhook HTTPS" .-> orq
    ngrok -. "túnel para o webhook (dev)" .-> orq
    orq <-- "REST (retry + idempotência)" --> be
    orq <--> rds
    orq -. "extração multimodal" .-> gem

    classDef person fill:#08427b,stroke:#052e56,color:#fff
    classDef container fill:#438dd5,stroke:#2e6295,color:#fff
    classDef poc fill:#85bbf0,stroke:#5d82a8,color:#000,stroke-dasharray:5 3
    classDef external fill:#999,stroke:#6b6b6b,color:#fff
```

- **Serviços-base** (sempre ativos): `banco-de-dados`, `back-end`, `front-end`, `redis`, `orquestrador`.
- **`adaptador-baileys`** só sobe no perfil `baileys`. No modo Cloud API (padrão do compose) ele não roda, e o orquestrador recebe o webhook da Meta.
- O compose publica **apenas** a porta do `front-end` (3000) e o webhook do orquestrador. Back-end, MySQL, Redis e adaptador ficam na rede interna.
- `ngrok` existe só no override local (`compose.local.yml`, perfil `tunnel`) para testes do webhook Cloud API em desenvolvimento, nunca em produção.

## Fluxo de uma compra (imagem ou texto)

```mermaid
sequenceDiagram
    autonumber
    actor C as Confeiteira (WhatsApp)
    participant A as adaptador/Cloud API
    participant O as orquestrador
    participant R as Redis
    participant IA as Gemini
    participant BE as back-end

    C->>A: foto da nota ou texto da compra
    A->>O: evento (só do GRUPO_JID autorizado)
    O->>R: dedupe por message_id, estado do fluxo
    O->>BE: GET /insumos/catalogo e GET /categorias-insumo
    O->>IA: extração estruturada (itens, quantidades, valor líquido)
    Note over O: decodifica QR da NFC-e (zxing-cpp)<br/>e valida dígito módulo 11
    O->>BE: GET /notas-fiscais/{chave}/status (se houver chave)
    O->>C: leitura por extenso + pedido de confirmação
    C->>O: confirma, ajusta (até 2) ou cancela
    O->>BE: POST /estoque/movimentos (1 req/item, Idempotency-Key)
    BE->>BE: grava ENTRADA_COMPRA, atualiza saldo e custo (mesma transação)
    O->>BE: POST /notificacoes (COMPRA_REGISTRADA, in-app)
    O->>C: resultado final (sucesso, parcial ou falha por item)
```

A máquina de estados da conversa (IDLE → EXTRAINDO → AGUARDANDO CONFIRMAÇÃO → GRAVANDO → ENCERRADO) está em [10.8](08-integracoes.md#conversa-como-máquina-de-estados); o comportamento implementado segue esse fluxo de confirmação de compra.

## Fluxo de entrega das notificações

```mermaid
sequenceDiagram
    autonumber
    participant SCH as back-end @Scheduled (5 min)
    participant BE as back-end
    participant DB as MySQL
    participant W as orquestrador (delivery worker)
    participant CH as canal WhatsApp
    participant C as Confeiteira

    SCH->>BE: avalia configuração e cria resumos/alertas do dia
    BE->>DB: INSERT notificacao (chave_unicidade única, status PENDENTE)
    loop a cada 30 s (DELIVERY_POLL_INTERVAL_S)
        W->>BE: GET /notificacoes/agenda?de=&ate=
        W->>W: sorteia 1 horário dentro da janela (persistido no Redis)
        W->>CH: envia uma única vez (guarda NX notif:{id}:sent)
        W->>BE: POST /notificacoes/{id}/confirmacao-envio (idempotente)
    end
    CH->>C: mensagem no WhatsApp
```

- Se a janela fechar antes do envio (máquina desligada além da janela), a notificação é reportada como `FALHA`, um `ALERTA_OPERACIONAL` é criado e a cópia **permanece visível no web app**.
- Alertas imediatos usam janela curta (`[agora, agora + 2 min]`), então com o poll padrão chegam ao WhatsApp em cerca de um minuto.

## Stream de notificações em tempo real (SSE)

O web app recebe atualizações instantâneas por Server-Sent Events, servidas pelo próprio back-end.

```mermaid
flowchart LR
    LD["Carregar a página"] --> L1["GET /notificacoes/nao-lidas (axios)"]
    LD --> ES["EventSource GET /notificacoes/stream"]
    N["Notificação criada ou marcada como lida<br/>(@TransactionalEventListener AFTER_COMMIT)"] --> PUSH["back-end empurra evento<br/>notificacao | lida"]
    PUSH --> ES
    ES --> PING["heartbeat a cada 25 s mantém a conexão"]
    DROP["Conexão perdida"] --> REC["navegador reconecta → recarrega a lista"]
    FOC["Aba volta ao foco"] --> REC
```

- O back-end mantém um **registro em memória** de `SseEmitter` abertos (sem timeout, heartbeat a cada 25 s). `spring.threads.virtual.enabled=true` evita prender uma thread de plataforma por conexão.
- **Exceção aprovada** à regra "HTTP só pela instância axios": o `EventSource` é usado apenas em `/notificacoes/stream`, porque o axios não consome streams. Fica no helper de serviço de notificações do front, não no componente.
- O registro em memória assume **uma única instância** do back-end (ver [10.14, débito do SSE/scheduler](14-assistente-rastreabilidade.md#débitos-técnicos-da-entrega)).

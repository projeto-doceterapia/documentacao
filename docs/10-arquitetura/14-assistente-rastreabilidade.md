# 10.14 Assistente de estoque — rastreabilidade e débitos

> **Situação:** implementado e em revisão nos pull requests dos repositórios de produto; ainda não implantado em produção.

Rastreabilidade do que foi entregue no assistente de estoque via WhatsApp e os débitos técnicos assumidos. Legenda: ✅ entregue · 🟡 parcial · ⏳ pendente (com motivo).

## Requisitos funcionais

| Requisito | Situação | Onde |
|---|---|---|
| Receber mensagens pelo canal escolhido (`CHANNEL`) | ✅ | adaptador Baileys + `CloudApiChannel` |
| Processar só o chat autorizado; descartar o resto | ✅ | allowlist `GRUPO_JID` no adaptador |
| Deduplicar entrada por `message_id` | ✅ | orquestrador (Redis) |
| Aceitar compra por imagem ou texto | ✅ | fluxo de compra |
| Orientar em tipo não suportado | ✅ | handler do fluxo |
| Consultar o catálogo a cada fluxo | ✅ | `GET /insumos/catalogo` |
| Extrair com LLM multimodal, saída validada por schema | ✅ | `LlmPort` (Gemini) |
| Status da nota (inexistente/parcial/completa) | ✅ | `GET /notas-fiscais/{chave}/status` + decisão no orquestrador |
| Apresentar leitura por extenso e pedir confirmação | ✅ | fluxo de compra |
| Interpretar resposta livre (confirmar/ajustar/cancelar/nova/fora) | ✅ | classificação de intenção |
| Até 2 ajustes; ao 3º, reiniciar | ✅ | `MAX_AJUSTES` |
| Perguntar se cria itens novos; esclarecer genéricos | ✅ | `POST /insumos/obter-ou-criar` + confirmação |
| Gravar 1 requisição por item, idempotente, resultado individual | ✅ | `POST /estoque/movimentos` + `Idempotency-Key` |
| Retry exponencial com jitter; não repetir falha definitiva; não bloquear os demais | ✅ | `app/retry` |
| Informar resultado final (sucesso/parcial/falha) | ✅ | `RetryWorker` / mensagem final |
| Consultar agenda, sortear horário, enviar e confirmar | ✅ | `app/delivery` + `GET /notificacoes/agenda` |
| Renderizar de `templates.yaml` (texto no Baileys, template na Cloud API) | ✅ | `app/templates/templates.yaml` |
| Expirar conversas inativas; tratar fora de contexto | ✅ | `CONV_TTL_MIN` (30 min) |
| Alertas operacionais (sessão caída, itens esgotados, falhas repetidas) | ✅ | `app/ops` (`ALERTA_OPERACIONAL`) |
| Trocar canal e provedor de IA por configuração | ✅ | `CHANNEL`, `LLM_PROVIDER` |
| "Tentar de novo" para itens com falha, por tempo limitado | ✅ | `RETRY_REQUEST_WINDOW_H` (24 h) |
| Cancelar o fluxo antes da gravação | ✅ | intenção de cancelamento |
| Fallback automático entre provedores de IA | ⏳ | **pendente:** só um provedor (Gemini) ativo; a troca é por configuração, não automática |

## Regras de negócio

As regras de negócio do assistente estão cobertas pelas regras implementadas em [10.12](12-assistente-regras.md): confirmação obrigatória antes de gravar, criação de insumo só com aprovação e esclarecimento de genéricos, entrada soma ao saldo (nunca sobrescreve), consolidação por insumo na mesma compra, conversão mostrada por extenso (**com desvio:** o cálculo é feito em código, não pela IA), chave de idempotência por item, status de nota completa/parcial, máximo de 2 ajustes, tratamento de compra nova durante fluxo pendente, não persistir imagens/rascunhos, falha por item não bloqueia os demais, resposta solta sem fluxo, origem da entrada por texto e descarte de chat não autorizado.

## Critérios de aceite

| Tema | Situação | Observação |
|---|---|---|
| Fluxo de compra ponta a ponta com confirmação | ✅ | coberto por testes do orquestrador e contrato de canal |
| Latência leitura→confirmação p95 ≤ 60 s | ⏳ | **pendente:** meta a validar com amostra real; não medida nesta entrega |
| Acurácia da IA ≥ 90% por item no conjunto de avaliação | ⏳ | **pendente:** gate de acurácia com **fotos reais** ainda não executado |
| Idempotência: reenvio nunca duplica | ✅ | `Idempotency-Key` + `hash_conteudo` |
| Resiliência a reinício | ✅ | Redis com AOF `everysec` |
| Cloud API validada em produção | ⏳ | **pendente:** testada só com o número de teste da Meta; produção exige número dedicado |

## Débitos técnicos da entrega

Débitos técnicos assumidos nesta entrega. Fechar cada um é uma mudança dedicada futura.

| Débito | Estado atual | Risco / motivo |
|---|---|---|
| auth serviço-a-serviço do orquestrador | endpoints `/insumos/catalogo`, `/insumos/obter-ou-criar`, `/estoque/movimentos`, `/notas-fiscais/**`, `/notificacoes/agenda`, `/notificacoes/{id}/confirmacao-envio` sem autenticação | quem alcançar a porta do back-end grava estoque e muda notificações. Mitigado: no compose a porta do back-end não é publicada |
| rotas de negócio em `permitAll` | as rotas de domínio ficam abertas sem login | todos os dados de negócio legíveis/graváveis sem auth; fechar junto com a auth serviço-a-serviço |
| SSE/scheduler assumem instância única | dedupe pela chave única de `notificacao`, mas toda réplica rodaria todo job; eventos SSE só chegam à instância que commitou | se o back-end escalar, precisa de lock distribuído (ex.: ShedLock) e broker (ex.: Redis pub/sub) |
| sem migrações versionadas | a imagem MySQL só roda `scriptv4.sql` em volume vazio | ambiente existente diverge e o `validate` recusa subir. Fix: Flyway com `scriptv4.sql` como baseline |
| Risco de banimento Baileys no número principal | aceito e documentado; estimativa **~3–12%/ano** (confiança baixa–média) para este volume | sem taxa oficial; o driver real seria falso positivo de heurística ou mudança de protocolo (ver o estudo comparativo de canais) |
| nginx não sobe se `API_UPSTREAM` não resolve | o template do front resolve o upstream no boot | se o host de `API_UPSTREAM` for inresolvível, o container do front-end falha ao iniciar |
| Modelo `.mwb` desatualizado | script atualizado, `.mwb` **pendente** | pela convenção, script e `.mwb` devem andar juntos |
| Gate de acurácia com fotos reais pendente | conjunto de avaliação existe; execução com fotos reais **pendente** | condiciona trocar o provedor de IA e confirmar o critério de acurácia |
| Cloud API só com número de teste da Meta | validada apenas com o número de teste de desenvolvedor | produção exige número dedicado, verificação de negócio e templates aprovados |

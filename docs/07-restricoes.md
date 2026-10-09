# 7. Restrições

> Fontes: `TAP2026.pdf`, `OFICIAL_DOCETERAPIA.docx`, `Planilha de Recursos.xlsx`, `AGENTS.md` (workspace).

## 7.1 Restrições de projeto

| ID | Tipo | Restrição |
|---|---|---|
| R01 | Prazo | Término obrigatório e improrrogável até **08/12/2026** |
| R02 | Recursos humanos | Equipe fechada de 6 integrantes, com 8 h por semana planejadas por pessoa e 50% de disponibilidade real (cerca de 4 h por semana cada, 24 h por semana no total) |
| R03 | Escopo | Escopo reduzido, adequado a um projeto acadêmico de Pesquisa e Inovação |
| R04 | Operação | O sistema não age no ambiente externo: não compra insumos e não substitui a produção real |
| R05 | Dados | A precisão depende do uso correto pela usuária |
| R06 | Custo | Sem orçamento para licenças pagas. O protótipo no Figma dependia de créditos da plataforma (Semana 3) |

## 7.2 Restrições técnicas

| ID | Restrição | Origem |
|---|---|---|
| RT01 | Back-end em Java 21 e Spring Boot 4.0.x. Front-end em React 19, JavaScript e Vite. Banco MySQL 8 | Decisão de arquitetura (C4 Deployment) |
| RT02 | Sem Lombok, MapStruct ou `javax.*`. Injeção por construtor | Convenções do time (`AGENTS.md`) |
| RT03 | Front-end sem Tailwind, Bootstrap, CSS-in-JS, kits de UI, `react-router`, bibliotecas de estado global ou `fetch` direto. HTTP só pelo axios em `services/api.js` | Convenções do time (removeram o Tailwind no PR #1) |
| RT04 | SDKs de WhatsApp, LLM ou Redis no produto principal dependem de decisão. Hoje só existem na POC `estoque-wpp` | `AGENTS.md`, seção de bibliotecas |
| RT05 | Textos visíveis ao usuário em PT-BR. Código, commits e documentação técnica em inglês. Identificadores de domínio em português | `AGENTS.md` |
| RT06 | Credenciais reais não podem ser versionadas. Em produção os segredos vêm de variáveis de ambiente | `AGENTS.md` |
| RT07 | A instalação de referência é local, no computador da beneficiária (Windows 11) | C4 Deployment (`c4-model-deployment.drawio`) |
| RT08 | Comunicação pelo WhatsApp oficial (Meta Cloud API) tem custo por conversa. A via não oficial (Baileys) tem risco de banimento | PRD do assistente de estoque, riscos 1 e 12 |

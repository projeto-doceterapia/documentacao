# 7. Restrições

## 7.1 Restrições de negócio e de desenvolvimento

| ID | Tipo | Restrição |
|---|---|---|
| R01 | Prazo | O sistema precisa estar em uso pela confeiteira até o fim de 2026 |
| R02 | Equipe | Equipe fechada de 6 pessoas, com cerca de 4 h semanais efetivas por pessoa (24 h por semana no total) |
| R03 | Escopo | Escopo reduzido aos módulos essenciais. As integrações com IA ficam para uma etapa posterior |
| R04 | Operação | O sistema não age no mundo externo: não compra insumos e não substitui a produção real |
| R05 | Dados | A precisão das informações depende de a confeiteira registrar tudo corretamente |
| R06 | Custo | Não há orçamento para licenças pagas. Ferramentas e serviços precisam ser gratuitos ou de baixo custo |

## 7.2 Restrições técnicas

| ID | Restrição | Motivo |
|---|---|---|
| RT01 | Back-end em Java 21 com Spring Boot 4.0.x. Front-end em React 19, JavaScript e Vite. Banco MySQL 8 | Domínio técnico da equipe e arquitetura definida (ver [10.1](10-arquitetura/01-visao-geral.md)) |
| RT02 | Sem Lombok, MapStruct ou `javax.*`. Injeção de dependências por construtor | Convenções de código do time |
| RT03 | Front-end sem Tailwind, Bootstrap, CSS-in-JS, kits de UI, `react-router`, bibliotecas de estado global ou `fetch` direto. Toda chamada HTTP passa por uma única instância do axios | Bundle menor, controle visual e padronização |
| RT04 | SDKs de WhatsApp, LLM ou Redis ficam fora do produto principal até que a integração seja aprovada. Hoje só existem na prova de conceito | Isolar o risco e o custo dessas integrações |
| RT05 | Textos visíveis ao usuário em PT-BR. Código, commits e documentação técnica em inglês. Termos de domínio em português (`Pedido`, `Insumo`, `Producao`) | Clareza para a usuária e alinhamento com o banco de dados |
| RT06 | Credenciais reais nunca vão para o repositório. Em produção, os segredos vêm de variáveis de ambiente | Segurança |
| RT07 | A instalação de referência é local, no computador da confeiteira (Windows 11) | Custo zero e uma única usuária (ver [implantação](10-arquitetura/02-c4-model.md#implantação)) |
| RT08 | O WhatsApp oficial (Meta Cloud API) cobra por conversa. A alternativa não oficial (Baileys) tem risco de banimento da conta | Limita as opções de integração (ver [10.8](10-arquitetura/08-integracoes.md)) |

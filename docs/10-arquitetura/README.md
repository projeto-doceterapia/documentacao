# 10. Artefatos de arquitetura

Esta seção descreve a arquitetura do sistema **como está implementada** (AS-IS) nos repositórios de front-end, back-end e banco de dados, e a arquitetura **planejada** (TO-BE) para as integrações com WhatsApp e IA. Todos os diagramas usam Mermaid e são renderizados direto no GitHub.

| # | Documento | Conteúdo |
|---|---|---|
| 10.1 | [Visão geral e decisões](01-visao-geral.md) | Stack, repositórios, estilo arquitetural, decisões (ADRs) e atributos de qualidade |
| 10.2 | [C4 Model](02-c4-model.md) | Contexto (N1), contêineres (N2), componentes do back-end e do front-end (N3), implantação e diagrama dinâmico |
| 10.3 | [Diagramas UML](03-uml.md) | Casos de uso, classes de domínio, pacotes, componentes, sequência (login, pedido, cancelamento), atividade e estados |
| 10.4 | [Back-end](04-back-end.md) | Camadas, catálogo completo de endpoints, segurança, erros, configuração e testes |
| 10.5 | [Front-end](05-front-end.md) | Estrutura, navegação, telas, consumo da API, acessibilidade e CI |
| 10.6 | [Banco de dados](06-banco-de-dados.md) | Modelo lógico (MER), dicionário de dados e divergências entre SQL e JPA |
| 10.7 | [Débitos técnicos](07-debitos-tecnicos.md) | Riscos e lacunas encontrados no código, com prioridade |
| 10.8 | [Integrações (WhatsApp e IA)](08-integracoes.md) | Prova de conceito de WhatsApp, lacunas e arquitetura alvo do assistente de estoque |

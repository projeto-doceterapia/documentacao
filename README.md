# 🍰 Doceterapia – Documentação do Projeto

Este repositório reúne a documentação acadêmica e técnica do sistema de **gestão de pedidos, estoque e produção** da **Doceterapia**, uma confeitaria artesanal de São Paulo. O projeto é de Extensão / Pesquisa e Inovação do **Grupo 9 – 2SIS, Sistemas de Informação, São Paulo Tech School (2026)**.

> **Em uma frase:** a confeiteira recebe pedidos pelo WhatsApp e controlava tudo à mão e "no olho". O sistema centraliza pedidos, agenda, clientes, produtos, insumos e pagamentos em uma aplicação web simples, com alertas de sobrecarga e de estoque e indicadores de lucro.

| | |
|---|---|
| **Beneficiária** | Doceterapia (confeitaria sob encomenda, desde 2016) |
| **Prazo** | Até 08/12/2026 |
| **Stack** | React 19 + Vite · Java 21 + Spring Boot 4 · MySQL 8 · (POC) Node + Twilio + Gemini |
| **Repositórios** | [front-end](https://github.com/projeto-doceterapia/front-end) · [back-end](https://github.com/projeto-doceterapia/back-end) · [banco-de-dados](https://github.com/projeto-doceterapia/banco-de-dados) · [estoque-wpp](https://github.com/projeto-doceterapia/estoque-wpp) |

## 📚 Índice

| # | Seção | Conteúdo |
|---|---|---|
| 1 | [Contexto](docs/01-contexto.md) | Beneficiária, operação atual (AS-IS), escolha da beneficiária, mercado |
| 2 | [Objetivos](docs/02-objetivos.md) | Objetivo geral, objetivos específicos, objetivo SMART, critérios de sucesso |
| 3 | [Justificativa](docs/03-justificativa.md) | Problema, dores priorizadas, benefícios, validação das hipóteses |
| 4 | [Escopo](docs/04-escopo.md) | Dentro e fora do escopo, EAP, dependências, entregáveis por sprint |
| 5 | [Requisitos](docs/05-requisitos.md) | RF01–RF13, RNF01–RNF10, histórias de usuário, critérios de aceite, rastreabilidade |
| 6 | [Premissas](docs/06-premissas.md) | Condições assumidas e impacto |
| 7 | [Restrições](docs/07-restricoes.md) | Restrições de projeto e técnicas |
| 8 | [Registro das entrevistas](docs/08-registro-entrevistas.md) | Entrevista, visita técnica, teste de usabilidade, atas |
| 9 | [Artefatos de regra de negócio](docs/09-regras-de-negocio.md) | Catálogo, RN01–RN24, BPMN AS-IS e TO-BE, máquinas de estado, indicadores, personas |
| 10 | [Artefatos de arquitetura](docs/10-arquitetura/README.md) | Visão geral, **C4 Model**, **UML**, back-end, front-end, banco, débitos técnicos, integrações |
| 11 | [Product backlog](docs/11-product-backlog.md) | Épicos, itens priorizados (MoSCoW), situação, plano da Sprint 6 |
| 12 | [Ferramentas de planejamento](docs/12-ferramentas-de-planejamento.md) | Equipe, cronograma (Gantt), responsabilidades, rituais, status report, GTD, retrospectivas |
| 13 | [Links do projeto no Git](docs/13-links-git.md) | Repositórios, histórico, convenções, como executar |
| 14 | [Referências bibliográficas](docs/14-referencias.md) | Mercado, métodos, modelagem, tecnologias, documentos internos |

### Arquitetura (seção 10)

| Documento | Diagramas |
|---|---|
| [10.1 Visão geral](docs/10-arquitetura/01-visao-geral.md) | Visão em camadas, ADRs |
| [10.2 C4 Model](docs/10-arquitetura/02-c4-model.md) | Contexto, Contêineres, Componentes (API e SPA), Implantação, Dinâmico |
| [10.3 UML](docs/10-arquitetura/03-uml.md) | Casos de uso, Classes, Enumerações, Pacotes, Sequência (login, cancelamento, nota fiscal), Atividade |
| [10.4 Back-end](docs/10-arquitetura/04-back-end.md) | Endpoints, segurança JWT, erros, configuração, testes |
| [10.5 Front-end](docs/10-arquitetura/05-front-end.md) | Navegação (estados), telas, acessibilidade, CI |
| [10.6 Banco de dados](docs/10-arquitetura/06-banco-de-dados.md) | MER do `scriptv4.sql`, modelo JPA, divergências |
| [10.7 Débitos técnicos](docs/10-arquitetura/07-debitos-tecnicos.md) | 23 débitos priorizados e roteiro de correção |
| [10.8 Integrações](docs/10-arquitetura/08-integracoes.md) | POC WhatsApp, análise de gaps, arquitetura hexagonal alvo |

## 🗺️ Visão rápida da solução

```mermaid
flowchart LR
    C(["Cliente"]) -->|WhatsApp| B(["Confeiteira"])
    B -->|navegador| FE["SPA React<br/>Agenda · Pedidos · Produção<br/>Clientes · Produtos · Visão Geral"]
    FE -->|REST + cookie JWT| API["API Spring Boot<br/>regras e máquinas de estado"]
    API --> DB[("MySQL")]
    B -. "foto da nota (POC)" .-> WPP["estoque-wpp<br/>Twilio + Gemini"]
```

## ▶️ Como executar o sistema

```bash
# Back-end (JDK 21) → http://localhost:8080 · Swagger: /swagger-ui/index.html
cd back-end && ./mvnw spring-boot:run          # Windows: .\mvnw.cmd spring-boot:run

# Front-end (Node 20+) → http://localhost:5173
cd front-end && npm ci && npm run dev           # VITE_API_URL padrão: http://localhost:8080
```

Sem configuração extra, a API usa H2 em memória. Para usar MySQL, execute `banco-de-dados/scriptv4.sql` e configure a conexão por variáveis de ambiente. Detalhes em [10.4 Back-end](docs/10-arquitetura/04-back-end.md#configuração) e [13 - Links do projeto no Git](docs/13-links-git.md#135-clonar-e-executar-tudo).

## 👥 Equipe – Grupo 9

| Integrante | Papel |
|---|---|
| Davi de Oliveira Domingues | Back-end e Integração |
| Davi Escudero Caproni | Infraestrutura e DevOps |
| Felipe Queiroz de Lima | Banco de Dados |
| Gwen Raldes de Barros | Qualidade e Regras do Sistema |
| Maria Luiza Costa Góes | Interface e Front-end · Representante do grupo |
| Pedro Guimarães Sakaue | Produto e Documentação Técnica |

## 📝 Sobre esta documentação

- Fontes: o código dos repositórios (out/2026) e os documentos do grupo (TAP, Documento de Condução, atas, relatório de visita, entregáveis de UX, Modelagem e Gestão de Projetos, status reports). Cada página indica as fontes que usou.
- Os diagramas são escritos em [Mermaid](https://mermaid.js.org) e o GitHub renderiza direto. Para editar, altere o bloco ` ```mermaid ` no Markdown.
- Dados pessoais (CPF, endereço, telefone, e-mail, RA e nomes dos participantes dos testes) foram omitidos de propósito.
- Licença: [MIT](LICENSE).

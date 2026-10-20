# 10.2 C4 Model

Diagramas nos quatro níveis do C4 (Contexto, Contêineres, Componentes e Código), mais os diagramas de Implantação e Dinâmico. O nível de código está em [10.3 UML](03-uml.md#1032-diagrama-de-classes-de-domínio).

> **Notação.** Os diagramas seguem a convenção do C4: cada elemento mostra o nome, o tipo e a tecnologia entre colchetes, além de uma descrição curta. São desenhados com `flowchart` do Mermaid porque o renderizador nativo de C4 (`C4Context`) ainda não faz um layout legível. Cores: azul-escuro para pessoas, azul para elementos do sistema, cinza para sistemas externos. A linha tracejada indica algo planejado ou fora do produto principal (POC).

```mermaid
flowchart LR
    P["Pessoa<br/>[Person]"]:::person
    S["Sistema / contêiner<br/>[Container: tecnologia]"]:::container
    E["Sistema externo<br/>[Software System]"]:::external
    P --> S --> E
    classDef person fill:#08427b,stroke:#052e56,color:#fff
    classDef container fill:#1168bd,stroke:#0b4884,color:#fff
    classDef external fill:#999,stroke:#6b6b6b,color:#fff
```

## Nível 1 – Contexto do sistema

```mermaid
flowchart TB
    cliente["<b>Cliente final</b><br/>[Person]<br/>Pessoa física ou empresa que<br/>encomenda bolos e doces"]:::extperson
    confeiteira["<b>Confeiteira</b><br/>[Person]<br/>Registra pedidos e acompanha<br/>produção, estoque e lucro"]:::person
    doce["<b>Sistema Doceterapia</b><br/>[Software System]<br/>Pedidos, agenda, clientes, produtos,<br/>insumos, produção, pagamentos e métricas"]:::system
    whatsapp["<b>WhatsApp / Twilio</b><br/>[Software System]<br/>Mensageria"]:::external
    gemini["<b>Google Gemini</b><br/>[Software System]<br/>LLM multimodal"]:::external

    cliente -- "Faz pedidos e negocia<br/>[WhatsApp]" --> confeiteira
    confeiteira -- "Usa<br/>[navegador / HTTPS]" --> doce
    confeiteira -. "Envia foto da nota fiscal<br/>(planejado)" .-> whatsapp
    whatsapp -. "Webhook com a mídia<br/>(planejado)" .-> doce
    doce -. "Envia alertas<br/>(planejado)" .-> whatsapp
    doce -. "Extrai itens da nota<br/>(planejado)" .-> gemini

    classDef person fill:#08427b,stroke:#052e56,color:#fff
    classDef extperson fill:#686868,stroke:#4d4d4d,color:#fff
    classDef system fill:#1168bd,stroke:#0b4884,color:#fff
    classDef external fill:#999,stroke:#6b6b6b,color:#fff
```

## Nível 2 – Contêineres

```mermaid
flowchart TB
    confeiteira["<b>Confeiteira</b><br/>[Person]<br/>Usuária única (administradora)"]:::person

    subgraph sys["Sistema Doceterapia [Software System]"]
        direction TB
        spa["<b>Aplicação Web (SPA)</b><br/>[Container: React 19, Vite, axios]<br/>Agenda, Pedido, Produção, Histórico,<br/>Produtos, Clientes, Visão Geral"]:::container
        api["<b>API REST</b><br/>[Container: Java 21, Spring Boot 4,<br/>Spring Security, JPA]<br/>Regras, máquinas de estado, JWT, Swagger"]:::container
        db[("<b>Banco de dados</b><br/>[Container: MySQL 8 / H2 em dev]<br/>Clientes, pedidos, itens, pagamentos,<br/>produção, produtos, insumos")]:::container
        poc["<b>Serviço de estoque via WhatsApp (POC)</b><br/>[Container: Node.js, Express,<br/>Twilio SDK, Gemini SDK]<br/>Lê a nota e atualiza o estoque"]:::poc
        pocdb[("<b>Banco da POC</b><br/>[Container: MySQL estoque_gemini]")]:::poc
    end

    twilio["<b>Twilio / WhatsApp</b><br/>[Software System]"]:::external
    gemini["<b>Google Gemini</b><br/>[Software System]"]:::external

    confeiteira -- "Usa [HTTPS]" --> spa
    spa -- "Chama [REST/JSON + cookie JWT]" --> api
    api -- "Lê e grava [JDBC/JPA]" --> db
    confeiteira -. "Foto da nota [WhatsApp]" .-> twilio
    twilio -. "POST /webhook [HTTPS]" .-> poc
    poc -. "Confirmação [REST]" .-> twilio
    poc -. "analyzeReceipt [HTTPS]" .-> gemini
    poc -. "Upsert de insumos [mysql2]" .-> pocdb
    poc -. "Integração planejada" .-> api

    classDef person fill:#08427b,stroke:#052e56,color:#fff
    classDef container fill:#438dd5,stroke:#2e6295,color:#fff
    classDef poc fill:#85bbf0,stroke:#5d82a8,color:#000,stroke-dasharray:5 3
    classDef external fill:#999,stroke:#6b6b6b,color:#fff
```

## Nível 3 – Componentes da API REST

```mermaid
flowchart TB
    spa["<b>SPA React</b><br/>[Container]"]:::ext
    subgraph api["API REST [Container: Spring Boot]"]
        direction TB
        sec["<b>Segurança</b><br/>[Component: SecurityConfiguracao,<br/>AutenticacaoFilter, GerenciadorTokenJwt]<br/>Filtro JWT (cookie ou Bearer), CORS, BCrypt"]:::comp
        ctrl["<b>Controllers REST</b><br/>[Component: controller + swagger]<br/>14 controllers e contratos OpenAPI"]:::comp
        svc["<b>Services de domínio</b><br/>[Component: service]<br/>Regras, validações, máquinas de estado,<br/>sinal de 50%"]:::comp
        map["<b>Mappers e DTOs</b><br/>[Component: mapper, dto]<br/>Conversão e snapshot de preço e custo"]:::comp
        exc["<b>Exceções</b><br/>[Component: exception]<br/>25 exceções (404, 409, 400)"]:::comp
        repo["<b>Repositórios</b><br/>[Component: Spring Data JPA]<br/>Consultas derivadas e JPQL"]:::comp
        ent["<b>Entidades e enums</b><br/>[Component: entity, enums]<br/>14 entidades e 13 enums"]:::comp
    end
    db[("<b>MySQL / H2</b><br/>[Container]")]:::ext

    spa -- "HTTP" --> sec
    sec -- "encaminha" --> ctrl
    ctrl -- "delega" --> svc
    svc --> map
    svc --> repo
    svc -. "lança" .-> exc
    repo --> ent
    repo -- "SQL" --> db

    classDef comp fill:#85bbf0,stroke:#5d82a8,color:#000
    classDef ext fill:#438dd5,stroke:#2e6295,color:#fff
```

Detalhamento por domínio (cada um tem controller, service, repository, mapper e DTOs próprios):

```mermaid
flowchart LR
    subgraph Comercial
        CLI[Cliente] --> PED[Pedido]
        PED --> ITE[ItemPedido]
        PED --> PAG[Pagamento]
        PED --> CAN[CancelamentoPedido]
    end
    subgraph Operação
        PED --> PRO[Producao]
        FAR[ConfiguracaoFarolAgenda]
        NOT[Notificacao]
    end
    subgraph Composição
        CP[CategoriaProduto] --> PRD[Produto]
        PRD --> PI[ProdutoInsumo]
        CI[CategoriaInsumo] --> INS[Insumo]
        INS --> PI
    end
    subgraph Acesso
        USR[Usuario]
    end
    ITE --> PRD
    NOT -.-> PED
    NOT -.-> INS
    CAN -. "atualizarStatus(CANCELADO)" .-> PED
    PED -. "cria pagamento" .-> PAG
```

## Nível 3 – Componentes da SPA

```mermaid
flowchart TB
    user["<b>Confeiteira</b><br/>[Person]"]:::person
    subgraph spa["SPA React [Container]"]
        direction TB
        app["<b>App.jsx</b><br/>[Component]<br/>Shell, autenticação e navegação por estado"]:::comp
        layout["<b>Layout</b><br/>[Component: Sidebar, AppTopbar]<br/>Menu, notificações, escala de fonte"]:::comp
        pages["<b>Páginas</b><br/>[Component: app/pages]<br/>Login, Agenda, Pedido, Produção,<br/>Histórico, Visão Geral, Produtos, Clientes"]:::comp
        feat["<b>Componentes de feature</b><br/>[Component: components/*]<br/>Calendário, stepper, carrinho, cards, gráficos"]:::comp
        utils["<b>pedidoUtils.js</b><br/>[Component]<br/>Status, datas, totais, moeda"]:::comp
        http["<b>services/api.js</b><br/>[Component: axios]<br/>baseURL = VITE_API_URL, withCredentials"]:::comp
        seed["<b>dadosTeste.js</b><br/>[Component]<br/>Seed via API (temporário)"]:::comp
    end
    api["<b>API REST</b><br/>[Container: Spring Boot]"]:::ext

    user --> app
    app --> layout
    app -- "tela atual" --> pages
    pages --> feat --> utils
    pages -- "GET/POST/PUT" --> http
    layout -- "GET /pedidos" --> http
    seed --> http
    http -- "REST/JSON + cookie" --> api

    classDef person fill:#08427b,stroke:#052e56,color:#fff
    classDef comp fill:#85bbf0,stroke:#5d82a8,color:#000
    classDef ext fill:#438dd5,stroke:#2e6295,color:#fff
```

## Implantação

Toda a solução roda no computador da confeiteira.

```mermaid
flowchart TB
    subgraph pc["Computador da Confeiteira [Deployment Node: Microsoft Windows 11]"]
        direction TB
        subgraph br["Navegador [Chrome / Edge]"]
            spa["<b>SPA React</b><br/>[Container: JavaScript]"]:::container
        end
        subgraph n1["Node Server [Node.js v24]"]
            vite["<b>Servidor do front-end</b><br/>[Container: Vite]<br/>:5173 (dev) / preview"]:::container
        end
        subgraph jvm["Java Virtual Machine [Java SE 21]"]
            api["<b>API REST</b><br/>[Container: Spring Boot 4]<br/>:8080, Swagger em /swagger-ui"]:::container
        end
        subgraph dbs["Database Server [MySQL 8.4 LTS]"]
            db[("<b>doce_terapia</b><br/>[Container: MySQL]<br/>scriptv4.sql")]:::container
        end
        subgraph n2["Node Server [Node.js v24]"]
            poc["<b>Serviço WhatsApp (POC)</b><br/>[Container: Express]<br/>:3000, POST /webhook"]:::poc
        end
    end
    subgraph cloud["Nuvem de terceiros [SaaS]"]
        tw["<b>Twilio</b><br/>[WhatsApp API]"]:::external
        gm["<b>Google Gemini</b><br/>[LLM]"]:::external
    end

    vite -- "serve o bundle [HTTP]" --> spa
    spa -- "REST [HTTP :8080]" --> api
    api -- "JDBC [:3306]" --> db
    tw -. "webhook [HTTPS via túnel]" .-> poc
    poc -. "API [HTTPS]" .-> gm
    poc -. "mysql2 [:3306]<br/>schema estoque_gemini" .-> db

    classDef container fill:#438dd5,stroke:#2e6295,color:#fff
    classDef poc fill:#85bbf0,stroke:#5d82a8,color:#000,stroke-dasharray:5 3
    classDef external fill:#999,stroke:#6b6b6b,color:#fff
```

> Para receber webhooks do Twilio em uma máquina local, a porta 3000 precisa ser exposta publicamente (por exemplo, com um túnel). O diagrama mostra isso como um túnel HTTPS.
>
> Esta porta 3000 é o servidor de desenvolvimento Express da **prova de conceito** (repositório `estoque-wpp`, hoje referência apenas). Não confundir com a porta 3000 publicada pelo **front-end conteinerizado** do assistente de estoque (nginx + SPA), que é a atual (ver [10.9](09-assistente-visao-geral.md) e [10.13](13-assistente-implantacao.md)).

## Diagrama dinâmico – Registrar orçamento

```mermaid
flowchart LR
    c["<b>Confeiteira</b><br/>[Person]"]:::person
    spa["<b>SPA React</b><br/>[Container: Pedido.jsx]"]:::container
    api["<b>API REST</b><br/>[Container: Spring Boot]"]:::container
    db[("<b>MySQL</b><br/>[Container]")]:::container
    c -- "1. Preenche cliente, entrega,<br/>itens e pagamento" --> spa
    spa -- "2. GET /produtos, GET /clientes<br/>3. POST /clientes (se novo)<br/>4. POST /pedidos (ORCAMENTO)<br/>5. POST /itens-pedido (por item)" --> api
    api -- "6. INSERT cliente, pedido<br/>e item_pedido" --> db
    classDef person fill:#08427b,stroke:#052e56,color:#fff
    classDef container fill:#438dd5,stroke:#2e6295,color:#fff
```

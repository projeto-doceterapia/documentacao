# 10.3 Diagramas UML

Os diagramas representam o código atual da API e da interface. Os diagramas de estado estão em [09 - Regras de negócio](../09-regras-de-negocio.md#95-máquinas-de-estado), e os de componentes e implantação estão no [C4 Model](02-c4-model.md).

## 10.3.1 Diagrama de casos de uso

```mermaid
flowchart LR
    C(["👩‍🍳 Confeiteira"])
    W(["📱 WhatsApp / Twilio"])
    T(["⏰ Agendador (planejado)"])

    subgraph S["Sistema Doceterapia"]
        UC01(["UC01 Autenticar-se"])
        UC02(["UC02 Registrar orçamento / pedido"])
        UC03(["UC03 Gerenciar clientes"])
        UC04(["UC04 Consultar agenda com farol"])
        UC05(["UC05 Configurar farol da agenda"])
        UC06(["UC06 Acompanhar fila de produção"])
        UC07(["UC07 Consultar histórico de pedidos"])
        UC08(["UC08 Atualizar status do pedido"])
        UC09(["UC09 Registrar sinal e restante"])
        UC10(["UC10 Cancelar pedido"])
        UC11(["UC11 Gerenciar produtos e ficha técnica"])
        UC12(["UC12 Gerenciar insumos e categorias"])
        UC13(["UC13 Visualizar indicadores"])
        UC14(["UC14 Receber notificações"])
        UC15(["UC15 Ajustar tamanho da fonte"])
        UC16(["UC16 Registrar compra por foto da nota"])
    end

    C --- UC01 & UC02 & UC03 & UC04 & UC05 & UC06 & UC07
    C --- UC08 & UC09 & UC10 & UC11 & UC12 & UC13 & UC14 & UC15
    C --- UC16
    W --- UC16
    T --- UC14
    UC02 -. "«include»" .-> UC03
    UC10 -. "«include»" .-> UC08
    UC04 -. "«extend»" .-> UC05
    UC02 -. "«extend»" .-> UC09
```

| UC | Situação | Telas / endpoints |
|---|---|---|
| UC01 | ✅ API e tela prontas. O app abre já autenticado (protótipo) | `Login.jsx`, `POST /usuarios/login` |
| UC02 | ✅ | `Pedido.jsx`, `POST /pedidos`, `POST /itens-pedido` |
| UC03 | ✅ (sem exclusão no front) | `Clientes.jsx`, `/clientes` |
| UC04, UC05 | ✅ | `Agenda.jsx`, `/configuracao-farol` |
| UC06, UC07 | ✅ (somente leitura) | `Producao.jsx`, `HistoricoPedidos.jsx` |
| UC08, UC09, UC10 | 🟡 só na API | `PATCH /pedidos/{id}/status`, `/pagamentos/{id}/registrar-*`, `POST /cancelamentos/pedido/{id}` |
| UC11 | ✅ | `Produtos.jsx`, `/produtos`, `/produtos/{id}/insumos` |
| UC12 | 🟡 só na API | `/insumos`, `/categorias-insumo` |
| UC13 | ✅ | `VisaoGeral.jsx` |
| UC14 | 🟡 derivada dos pedidos na Topbar. `/notificacoes` existe | `AppTopbar.jsx` |
| UC15 | ✅ | `AppTopbar.jsx` |
| UC16 | 🧪 prova de conceito separada | Serviço de WhatsApp ([10.8](08-integracoes.md)) |

### Especificação resumida – UC02 Registrar orçamento

| Item | Descrição |
|---|---|
| Ator | Confeiteira |
| Pré-condição | Produtos cadastrados |
| Fluxo principal | 1. Clica em "Novo pedido". 2. Escolhe um cliente existente ou informa um novo. 3. Informa a data e a forma de entrega (endereço, se for entrega). 4. Adiciona itens pelo catálogo e ajusta as quantidades. 5. Escolhe a forma de pagamento e vê o total, o sinal (50%) e o restante. 6. Confirma, e o sistema grava o pedido como `ORCAMENTO` com seus itens. |
| Fluxos alternativos | 2a. Cliente novo: o sistema cria o cliente (`PADRAO`, `ATIVO`). 3a. Data no passado ou endereço ausente: o sistema mostra uma mensagem de validação. 4a. Carrinho vazio: o sistema impede o avanço |
| Pós-condição | O orçamento aparece na Agenda e no Histórico |
| Regras | RN01, RN02, RN07, RN08, RN12, RN13 |

## 10.3.2 Diagrama de classes de domínio

```mermaid
classDiagram
    direction LR

    class Cliente {
        Integer idCliente
        String nome
        String telefone
        String endereco
        TipoPessoa tipoPessoa
        ClassificacaoCliente classificacaoCliente
        StatusAtivo status
        String observacao
    }
    class Pedido {
        Integer idPedido
        TipoPedido tipoPedido
        StatusPedido statusPedido
        FormaEntrega formaEntrega
        String enderecoEntrega
        LocalDate dataEntrega
        String anotacao
        LocalDateTime dataCriacao
        LocalDateTime dataConfirmacao
        +prePersist()
    }
    class ItemPedido {
        Integer idProdutoPedido
        BigDecimal quantidade
        BigDecimal precoOriginalProduto
        BigDecimal valorUnitario
        BigDecimal valorTotal
        BigDecimal custoEstimadoItem
        BigDecimal margemLucroItem
        String observacao
    }
    class Pagamento {
        Integer idPagamento
        BigDecimal valorTotal
        BigDecimal valorSinal
        BigDecimal valorRestante
        FormaPagamento formaPagamento
        StatusPagamento statusPagamento
        LocalDate dataPagamentoSinal
        LocalDate dataPagamentoRestante
    }
    class CancelamentoPedido {
        Integer idCancelamento
        TipoCancelamento tipoCancelamento
        TipoRetorno tipoRetorno
        BigDecimal valorRetorno
        LocalDateTime dataCancelamento
        String observacao
    }
    class Producao {
        Integer idProducao
        LocalDate dataInicio
        LocalDate dataPrevista
        LocalDate dataFinalizacao
        StatusProducao statusProducao
        String observacao
    }
    class Produto {
        Integer idProduto
        String nome
        BigDecimal custoEstimado
        BigDecimal precoAtual
        BigDecimal precoSugerido
        BigDecimal margemLucro
        UnidadeMedida unidadeProducao
        StatusAtivo status
        Integer quantidadeProduzida
        String descricao
    }
    class CategoriaProduto {
        Integer idCategoriaProduto
        String nome
        String descricao
    }
    class ProdutoInsumo {
        Integer idProdutoInsumo
        BigDecimal quantidadeUtilizada
        UnidadeMedida unidade
    }
    class Insumo {
        Integer idInsumo
        String nome
        BigDecimal quantidadeAtual
        BigDecimal quantidadeMinima
        UnidadeMedida unidade
        StatusAtivo status
        String marca
        BigDecimal custoUnitario
    }
    class CategoriaInsumo {
        Integer idCategoriaInsumo
        String nome
        String descricao
    }
    class Notificacao {
        Integer idNotificacao
        TipoNotificacao tipo
        String titulo
        String mensagem
        LocalDateTime dataCriacao
        Boolean lida
    }
    class ConfiguracaoFarolAgenda {
        Integer idConfiguracaoFarol
        Integer limiteVerde
        Integer limiteAmarelo
        Integer limiteVermelho
    }
    class Usuario {
        Long id
        String nome
        String email
        String senha
    }

    Cliente "1" <-- "0..*" Pedido : fk_cliente
    Pedido "1" <-- "0..*" ItemPedido : fk_pedido
    Produto "1" <-- "0..*" ItemPedido : fk_produto
    Pedido "1" <-- "0..1" Pagamento : fk_pedido
    Pedido "1" <-- "0..1" CancelamentoPedido : fk_pedido
    Pedido "1" <-- "0..1" Producao : fk_pedido
    CategoriaProduto "1" <-- "0..*" Produto : fk_categoria_produto
    Produto "1" <-- "0..*" ProdutoInsumo : fk_produto
    Insumo "1" <-- "0..*" ProdutoInsumo : fk_insumo
    CategoriaInsumo "0..1" <-- "0..*" Insumo : fk_categoria_insumo
    Pedido "0..1" <-- "0..*" Notificacao : fk_pedido
    Insumo "0..1" <-- "0..*" Notificacao : fk_insumo
```

### Enumerações

Todas ficam em `enums/` e são persistidas como `@Enumerated(EnumType.STRING)`.

| Enum | Valores | Usado em |
|---|---|---|
| `StatusPedido` | `ORCAMENTO`, `AGUARDANDO_SINAL`, `ESPERANDO_FAZER`, `EM_PRODUCAO`, `AGUARDANDO_ENTREGA`, `ENTREGUE`, `CANCELADO` | `Pedido.statusPedido` |
| `StatusProducao` | `ESPERANDO_FAZER`, `EM_PRODUCAO`, `PRONTO_PARA_ENTREGA`, `FINALIZADO` | `Producao.statusProducao` |
| `StatusPagamento` | `AGUARDANDO_SINAL`, `AGUARDANDO_PAGAMENTO`, `PAGO_INTEGRAL` | `Pagamento.statusPagamento` |
| `TipoPedido` | `ORCAMENTO`, `PEDIDO_CONFIRMADO` | `Pedido.tipoPedido` |
| `FormaEntrega` | `ENTREGA`, `RETIRADA` | `Pedido.formaEntrega` |
| `FormaPagamento` | `PIX`, `DINHEIRO`, `CARTAO_CREDITO`, `CARTAO_DEBITO`, `TRANSFERENCIA` | `Pagamento.formaPagamento` |
| `TipoCancelamento` | `CLIENTE_DESISTIU`, `ERRO_PEDIDO`, `FALTA_INSUMO`, `OUTRO` | `CancelamentoPedido` |
| `TipoRetorno` | `CREDITO_CLIENTE`, `DEVOLUCAO`, `SEM_DEVOLUCAO` | `CancelamentoPedido` |
| `TipoNotificacao` | `ESTOQUE_BAIXO`, `DIA_LOTADO`, `RESUMO_INICIO_DIA`, `RESUMO_FIM_DIA`, `COMPRA_REGISTRADA`, `ALERTA_OPERACIONAL` (antes desta entrega: `ESTOQUE_BAIXO`, `PEDIDO_PROXIMO_ENTREGA`, `PAGAMENTO_PENDENTE`, `DIA_LOTADO`, `PEDIDO_ATRASADO`; os três sem geração foram removidos) | `Notificacao.tipo` |
| `TipoPessoa` | `FISICA`, `JURIDICA` | `Cliente.tipoPessoa` |
| `ClassificacaoCliente` | `PADRAO`, `FREQUENTE`, `ALTO_VOLUME` | `Cliente.classificacaoCliente` |
| `StatusAtivo` | `ATIVO`, `INATIVO` | `Cliente`, `Produto`, `Insumo` |
| `UnidadeMedida` | `KG`, `G`, `L`, `ML`, `UNIDADE` | `Produto`, `Insumo`, `ProdutoInsumo` |

## 10.3.3 Diagrama de pacotes (back-end)

```mermaid
flowchart TB
    subgraph api["br.com.doceterapia.api"]
        controller["controller<br/>14 classes"] --> service["service<br/>15 classes"]
        controller -. implementa .-> swagger["swagger<br/>14 *ControllerOpenApi"]
        controller --> dto["dto<br/>Request/Response DTOs"]
        service --> repository["repository<br/>14 JpaRepository"]
        service --> mapper["mapper<br/>13 mappers"]
        service --> exception["exception<br/>25 exceções"]
        mapper --> dto
        mapper --> entity["entity<br/>14 entidades"]
        repository --> entity
        entity --> enums["enums<br/>13 enums"]
        dto --> enums
        config["config<br/>Security, JWT, OpenAPI"] --> service
    end
```

## 10.3.4 Sequência – Login com cookie JWT

```mermaid
sequenceDiagram
    autonumber
    actor U as Confeiteira
    participant L as Login.jsx
    participant AX as axios (withCredentials)
    participant F as AutenticacaoFilter
    participant UC as UsuarioController
    participant US as UsuarioService
    participant AM as AuthenticationManager / AutenticacaoProvider
    participant J as GerenciadorTokenJwt
    participant R as UsuarioRepository

    U->>L: e-mail + senha
    L->>AX: POST /usuarios/login {email, senha}
    AX->>F: requisição (rota pública)
    F->>UC: login(UsuarioLoginDto)
    UC->>US: autenticar(usuario)
    US->>AM: authenticate(email, senha)
    AM->>R: findByEmail
    AM->>AM: BCrypt.matches
    US->>J: generateToken(authentication)
    J-->>US: JWT HS256 (sub=email, exp=jwt.validity)
    US-->>UC: UsuarioTokenDto
    UC-->>AX: 200 + Set-Cookie authToken (HttpOnly, SameSite=Strict)
    AX-->>L: UsuarioSessaoDto
    L->>L: onLogin() → App mostra o shell
    Note over AX,F: Nas próximas requisições o navegador envia o cookie, e o filtro valida o token e preenche o SecurityContext
```

## 10.3.5 Sequência – Cancelar pedido

```mermaid
sequenceDiagram
    autonumber
    participant C as Cliente HTTP
    participant CC as CancelamentoPedidoController
    participant CS as CancelamentoPedidoService
    participant PS as PedidoService
    participant DB as Repositórios
    C->>CC: POST /cancelamentos/pedido/{id} {tipoCancelamento, tipoRetorno, valorRetorno}
    CC->>CS: criar(pedidoId, dto)
    CS->>DB: pedido existe?
    alt não existe
        CS-->>C: 404 PedidoIdDontExistsException
    end
    CS->>DB: existsByPedidoIdPedido?
    alt já cancelado
        CS-->>C: 409 "Já existe um cancelamento cadastrado para este pedido"
    end
    CS->>DB: save(cancelamento, dataCancelamento = agora)
    CS->>PS: atualizarStatus(pedidoId, CANCELADO)
    PS->>PS: valida TRANSICOES_VALIDAS
    alt pedido ENTREGUE
        PS-->>C: 400 "Transição de ENTREGUE para CANCELADO não é permitida"
    end
    PS->>DB: save(pedido)
    CC-->>C: 201 CancelamentoPedidoResponseDTO
```

> O cancelamento é salvo **antes** de validar a transição do pedido, e nenhum service usa `@Transactional`. Por isso, cancelar um pedido `ENTREGUE` devolve 400, mas deixa o registro de `cancelamento_pedido` gravado. Ver [débitos técnicos](07-debitos-tecnicos.md).

## 10.3.6 Sequência – Entrada de estoque por nota fiscal (POC)

```mermaid
sequenceDiagram
    autonumber
    actor B as Confeiteira
    participant WA as WhatsApp / Twilio
    participant WH as webhookController (Express)
    participant GS as geminiService
    participant G as Gemini 2.5 Flash
    participant SS as stockService
    participant DB as MySQL (estoque_gemini)
    B->>WA: Foto da nota fiscal
    WA->>WH: POST /webhook (NumMedia, MediaUrl0)
    WH-->>WA: TwiML "Nota recebida! A IA está processando..."
    WH->>GS: analyzeReceipt(mediaUrl)
    GS->>WA: download da imagem (auth básica Twilio)
    GS->>G: prompt + imagem (JSON: nome, quantidade, unidade, preço, data)
    G-->>GS: itens normalizados (kg→g, L→ml, sem marca)
    WH->>SS: processProducts(itens)
    loop cada item
        SS->>DB: existe produto com o mesmo nome?
        alt sim
            SS->>DB: UPDATE quantidade = quantidade + n
        else não
            SS->>DB: INSERT produto
        end
    end
    SS-->>WH: {novos, atualizados}
    WH->>WA: mensagem de confirmação
    WA->>B: "X itens novos, Y atualizados"
```

## 10.3.7 Atividade – Registrar pedido no front-end

```mermaid
flowchart TD
    A([Início: Novo pedido]) --> B[Carregar produtos e clientes]
    B -->|erro| BE[/"Não foi possível carregar clientes e produtos."/] --> Z([Fim])
    B --> E1{Etapa 1: cliente}
    E1 -->|existente| E1a[Selecionar cliente]
    E1 -->|novo| E1b[Informar nome, telefone, endereço, tipo de pessoa]
    E1a --> E1c[Data de entrega + forma de entrega]
    E1b --> E1c
    E1c --> V1{Campos obrigatórios ok?<br/>endereço se ENTREGA}
    V1 -->|não| M1[/Mostrar erro/] --> E1
    V1 -->|sim| E2[Etapa 2: adicionar itens do catálogo]
    E2 --> V2{Carrinho tem itens?}
    V2 -->|não| M2[/Mostrar erro/] --> E2
    V2 -->|sim| E3[Etapa 3: forma de pagamento,<br/>sinal 50%, observações, resumo]
    E3 --> S{Salvar orçamento}
    S --> N{Cliente novo?}
    N -->|sim| PC[POST /clientes] --> PP
    N -->|não| PP[POST /pedidos ORCAMENTO]
    PP --> PI[POST /itens-pedido para cada item]
    PI -->|sucesso| BK[onBack: volta para a tela de origem] --> Z
    PI -->|erro| ME[/Mostrar erro ao salvar/] --> E3
```

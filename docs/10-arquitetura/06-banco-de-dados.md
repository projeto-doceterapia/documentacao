# 10.6 Banco de dados

> Script: `scriptv4.sql` (MySQL 8, schema `doce_terapia`) · Modelo: `modelagemv4.mwb` · Imagem: `doceTerapia.png`
>
> O modelo está na quarta versão. Ele é dividido em duas áreas: **Comercial** (cliente, pedido, pagamento, cancelamento) e **Composição** (produto, insumo, ficha técnica, item e produção).

## Modelo entidade-relacionamento (scriptv4.sql)

```mermaid
erDiagram
    CATEGORIA_PRODUTO ||--o{ PRODUTO : "categoriza"
    CATEGORIA_INSUMO ||--o{ INSUMO : "categoriza"
    CLIENTE ||--o{ PEDIDO : "faz"
    PEDIDO ||--o| PAGAMENTO : "tem"
    PEDIDO ||--o| CANCELAMENTO_PEDIDO : "pode ter"
    PEDIDO ||--|{ ITEM_PEDIDO : "contém"
    PRODUTO ||--o{ ITEM_PEDIDO : "vendido em"
    PEDIDO ||--o{ PRODUCAO : "gera"
    ITEM_PEDIDO ||--o{ PRODUCAO : "produzido em"
    PRODUTO ||--o{ PRODUTO_INSUMO : "composto por"
    INSUMO ||--o{ PRODUTO_INSUMO : "usado em"

    CATEGORIA_PRODUTO {
        int id_categoria_produto PK
        varchar nome "45, NN"
        varchar descricao "245"
    }
    CATEGORIA_INSUMO {
        int id_categoria_insumo PK
        varchar nome "45, NN"
        varchar descricao "245"
    }
    CLIENTE {
        int id_cliente PK
        varchar nome "45, NN"
        char telefone "12, NN"
        varchar endereco "245, NN"
        varchar tipo_cliente "45"
        tinyint status "NN, default 1, 0 ou 1"
        varchar observacao "245"
    }
    PRODUTO {
        int id_produto PK
        int fk_categoria_produto FK "NN, RESTRICT"
        varchar nome "45, NN"
        decimal custo_estimado "10,2 maior ou igual a 0"
        decimal preco_atual "10,2 maior ou igual a 0"
        decimal preco_sugerido "10,2 maior ou igual a 0"
        decimal margem_lucro "10,2"
        varchar unidade_producao "45"
        tinyint status "NN, default 1"
        int quantidade_produzida
        varchar descricao "245"
    }
    INSUMO {
        int id_insumo PK
        int fk_categoria_insumo FK "NN, RESTRICT"
        varchar nome "45, NN"
        decimal quantidade_atual "10,3 NN default 0"
        decimal quantidade_minima "10,3 NN default 0"
        varchar unidade "45, NN"
        decimal custo_unitario "4,2"
        tinyint status "NN, default 1"
        varchar marca "45"
    }
    PEDIDO {
        int id_pedido PK
        int fk_cliente FK "NN, RESTRICT"
        varchar tipo_pedido "45, NN"
        varchar status_pedido "245, NN"
        varchar forma_entrega "45, NN"
        varchar endereco_entrega "225"
        date data_criacao "NN"
        date data_entrega
        decimal margem_lucro "4,2"
        varchar anotacao "245"
    }
    PAGAMENTO {
        int id_pagamento PK
        int fk_pedido FK "NN, UNIQUE, CASCADE"
        decimal valor_total "10,2 NN"
        decimal valor_sinal "10,2 NN"
        decimal valor_restante "10,2 NN"
        varchar status_pagamento "245, NN"
        varchar forma_pagamento "245"
    }
    CANCELAMENTO_PEDIDO {
        int id_cancelamento PK
        int fk_pedido FK "NN, UNIQUE, CASCADE"
        varchar tipo_cancelamento "45, NN"
        decimal valor_retorno "10,2"
        date data_cancelamento "NN"
        varchar observacao "245"
    }
    ITEM_PEDIDO {
        int id_produto_pedido PK
        int fk_produto FK "NN, RESTRICT"
        int fk_pedido FK "NN, CASCADE"
        int quantidade "NN, maior que 0"
        decimal valor_unitario "10,2 NN"
        decimal valor_total "10,2 NN"
        varchar observacao "245"
    }
    PRODUCAO {
        int id_producao PK
        int fk_pedido FK "NN, CASCADE"
        int fk_item_pedido FK "NN, CASCADE"
        date data_inicio
        date data_prevista
        varchar status_producao "45, NN"
        varchar prioridade "45"
        varchar observacao "245"
    }
    PRODUTO_INSUMO {
        int id_produto_insumo PK
        int fk_produto FK "NN, CASCADE"
        int fk_insumo FK "NN, RESTRICT"
        decimal quantidade_utilizada "10,3 NN, maior que 0"
        varchar unidade "45, NN"
    }
```

Políticas de integridade referencial:

- **RESTRICT** impede apagar cadastros mestres em uso: categoria → produto/insumo, cliente → pedido, produto → item, insumo → ficha técnica.
- **CASCADE** remove os dependentes do pedido (pagamento, cancelamento, itens e produção) e a ficha técnica do produto.
- Todas as FKs usam `ON UPDATE CASCADE`.

O script também insere dados de exemplo: categorias (Bolo, Docinhos, Torta; Secos, Laticínios…), clientes, produtos, insumos, ficha técnica, pedidos, pagamentos, itens, produção e cancelamento.

## Modelo efetivo da aplicação (entidades JPA)

Com `ddl-auto=create-drop`, a API gera o schema a partir das entidades. Esse modelo **difere** do `scriptv4.sql` e tem três tabelas a mais.

```mermaid
erDiagram
    cliente ||--o{ pedido : fk_cliente
    pedido ||--o| pagamento : "fk_pedido UNIQUE"
    pedido ||--o| cancelamento_pedido : "fk_pedido UNIQUE"
    pedido ||--o| producao : "fk_pedido UNIQUE"
    pedido ||--o{ item_pedido : fk_pedido
    produto ||--o{ item_pedido : fk_produto
    categoria_produto ||--o{ produto : fk_categoria_produto
    produto ||--o{ produto_insumo : fk_produto
    insumo ||--o{ produto_insumo : fk_insumo
    categoria_insumo |o--o{ insumo : fk_categoria_insumo
    pedido |o--o{ notificacao : fk_pedido
    insumo |o--o{ notificacao : fk_insumo

    notificacao {
        int id_notificacao PK
        varchar tipo "TipoNotificacao"
        varchar titulo "150"
        text mensagem
        datetime data_criacao
        boolean lida
        int fk_pedido FK "opcional"
        int fk_insumo FK "opcional"
    }
    configuracao_farol_agenda {
        int id_configuracao_farol PK
        int limite_verde
        int limite_amarelo
        int limite_vermelho
    }
    usuario {
        bigint id PK
        varchar nome
        varchar email
        varchar senha "BCrypt"
    }
```

## Divergências entre o script SQL e as entidades JPA

| Tabela | `scriptv4.sql` | JPA (`entity/*`) | Ação sugerida |
|---|---|---|---|
| `cliente` | `tipo_cliente VARCHAR`, `status TINYINT`, `telefone CHAR(12)` | `tipoPessoa` e `classificacaoCliente` (enums), `status` ATIVO/INATIVO, telefone com até 20 caracteres | Criar `tipo_pessoa` e `classificacao_cliente`, `status VARCHAR`, telefone `VARCHAR(20)` UNIQUE |
| `pedido` | `data_criacao DATE`, `forma_entrega NOT NULL`, `margem_lucro` | `LocalDateTime`, forma de entrega opcional, sem margem, com `data_confirmacao` | Alinhar os tipos, remover `margem_lucro` e criar `data_confirmacao` |
| `item_pedido` | `quantidade INT` | `BigDecimal(10,3)` + `preco_original_produto`, `custo_estimado_item`, `margem_lucro_item` | Criar as colunas de snapshot e usar `DECIMAL(10,3)` |
| `pagamento` | sem datas | `data_pagamento_sinal`, `data_pagamento_restante` | Criar as colunas |
| `cancelamento_pedido` | `data_cancelamento DATE`, sem `tipo_retorno` | `LocalDateTime`, `tipoRetorno` | Criar `tipo_retorno` e usar `DATETIME` |
| `producao` | `fk_item_pedido`, `prioridade`, `fk_pedido` sem UNIQUE | 1:1 com o pedido, `data_finalizacao`, sem item nem prioridade | Decidir a granularidade (por pedido ou por item) e alinhar |
| `insumo` | `custo_unitario DECIMAL(4,2)`, categoria NOT NULL | `DECIMAL(10,4)`, categoria opcional | Aumentar a precisão (DECIMAL(4,2) só aceita até 99,99) |
| `produto`, `insumo` | `status TINYINT` | enum `STRING` | Padronizar |
| — | ausentes | `usuario`, `notificacao`, `configuracao_farol_agenda` | Adicionar ao script |

> Pela convenção do time, mudanças de schema devem atualizar `scriptv4.sql` e o `.mwb` ao mesmo tempo e manter as entidades alinhadas. O `schema.sql` dentro da API é uma terceira cópia do DDL, com diferenças próprias.

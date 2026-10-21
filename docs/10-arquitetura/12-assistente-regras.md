# 10.12 Assistente de estoque — regras de negócio

> **Situação:** implementado e em revisão nos pull requests dos repositórios de produto; ainda não implantado em produção.

Regras implementadas no assistente de estoque via WhatsApp. Complementam as regras do sistema em [09 - Artefatos de regra de negócio](../09-regras-de-negocio.md) e cobrem movimentação de estoque, catálogo de insumos, notificações de negócio e a conversa de compra pelo WhatsApp.

## Ledger de estoque e idempotência

- **Entrada soma, nunca sobrescreve.** Toda alteração de saldo é um movimento imutável (`SALDO_INICIAL`, `ENTRADA_COMPRA`, `AJUSTE`). O saldo (`quantidadeAtual`) é sempre a soma dos movimentos, atualizado na mesma transação, com lock na linha do insumo para não perder atualizações concorrentes.
- **Uma requisição por item.** `POST /estoque/movimentos` grava um item por vez e exige o header `Idempotency-Key`. Formato da chave (definido pelo orquestrador): `nota:{chave}:{insumoId}` quando há chave da nota, `fluxo:{fluxoId}:{insumoId}` quando não há.
- **Retry nunca duplica.** Mesma chave + mesmo `hash_conteudo` → `200` com `duplicado: true` e o movimento original. Mesma chave + conteúdo diferente → `409` (conflito definitivo). Chave nova → `201`. Sem o header → `400`.
- **Insumo com histórico não é excluído.** `DELETE /insumos/{id}` retorna `409` (mensagem PT-BR orientando inativar) quando há movimentos. Sem movimentos, `204`.
- **Status da nota** (`GET /notas-fiscais/{chave}/status`): `INEXISTENTE` ou `REGISTRADA` com a lista de `insumoId` já gravados para a chave. O orquestrador compara essa lista com os itens extraídos para decidir "completa" vs "parcial". Chave inválida (não 44 dígitos ou dígito verificador errado) → `422`.

## Conversão de unidades (determinística, em código)

- Unidades compatíveis por unidade do insumo: `KG↔G` (×1000), `L↔ML` (×1000), e `UNIDADE` só com ela mesma. Unidade incompatível → `422`.
- A conversão é feita **no código** (core e orquestrador usam a mesma tabela, mantida consistente por testes de contrato), não pela IA. A IA só extrai fatos crus; o cálculo (ex.: `2 × 5 kg = 10 kg`) é mostrado por extenso na confirmação para conferência humana.
- `quantidade_informada` e `unidade_informada` ficam guardadas no movimento para auditoria da conversão.

## Custo = último preço líquido pago

- Com preço no item: `custo_unitario = valor_liquido / quantidade_convertida` (escala 4, `HALF_EVEN`). O valor líquido é o valor da linha **após** descontos do item.
- Itens do mesmo insumo na mesma compra são consolidados: soma dos valores líquidos sobre a soma das quantidades.
- Item sem preço não altera o custo.

## Catálogo e categorias

- `GET /insumos/catalogo` retorna só insumos ativos (`idInsumo`, `nome`, `unidade`), ordenados por nome normalizado, em páginas limitadas (padrão e teto 200) com `total` e `proximoCursor` (keyset `(nome_normalizado, id)`). Catálogo pequeno cabe em uma resposta (cursor vazio).
- Busca `q=` casa por palavra no nome normalizado (ignora caixa, acentos e espaços). Em catálogo parcial, a IA devolve `termos_busca` e o **código** roda uma busca por termo (a IA não escolhe ferramentas).
- `POST /insumos/obter-ou-criar` casa por nome normalizado + unidade: `200` com o existente ou `201` com insumo novo (`quantidadeAtual = 0`, `ATIVO`). Nome em branco ou acima de 45 caracteres → `422`.
- **Categoria do insumo novo:** a IA sugere uma `categoria_id` entre as categorias existentes; o código valida e, se ausente/desconhecida, usa a categoria padrão `A classificar` (criada se faltar). A IA **nunca cria categorias**; a proprietária pode trocar a categoria na confirmação.

## Fluxo de pagamento da proprietária

O modelo de notificações segue a rotina real: pedido → **sinal de 50%** → produção → entrega com o pagamento final. Pagamentos nunca vencem antes da entrega, exceto o sinal. Por isso os resumos olham sinais a conferir (`AGUARDANDO_SINAL`) e pagamentos finais a conferir (`ENTREGUE` e pagamento não `PAGO_INTEGRAL`).

## Dois resumos diários e alertas imediatos

| Tipo | Quando | Conteúdo | Chave de dedupe |
|---|---|---|---|
| `RESUMO_INICIO_DIA` | horário de início, todo dia útil | sinais a conferir, pagamentos finais a conferir, entregas de hoje, pedidos atrasados | `RESUMO_INICIO_DIA:{data}` |
| `RESUMO_FIM_DIA` | horário de fim, todo dia útil | pedidos de amanhã até `+diasAntecedencia` (os que aguardam sinal aparecem à parte); insumos no mínimo ou abaixo | `RESUMO_FIM_DIA:{data}` |
| `ESTOQUE_BAIXO` | movimento cruza o mínimo para baixo (mesma transação) | insumo, saldo, mínimo | `ESTOQUE_BAIXO:insumo:{id}:{data}` |
| `DIA_LOTADO` | um pedido faz a data chegar ao `limiteVermelho` do farol | data, nº de pedidos | `DIA_LOTADO:{data}` |

- Resumos são **sempre criados**; seção vazia vira "nada pendente", nunca omitida.
- O enum `TipoNotificacao` passou a ser `ESTOQUE_BAIXO`, `DIA_LOTADO`, `RESUMO_INICIO_DIA`, `RESUMO_FIM_DIA`, `COMPRA_REGISTRADA`, `ALERTA_OPERACIONAL`. Saíram `PEDIDO_PROXIMO_ENTREGA`, `PEDIDO_ATRASADO` e `PAGAMENTO_PENDENTE` (nada os criava ou lia). **Mudança incompatível:** `POST`/`PUT /notificacoes` com um tipo removido agora retorna `400`.

## Configurações de expediente e janelas

- Um `@Scheduled` roda a cada 5 minutos, lê a configuração e cria os resumos cujo horário já passou hoje e que ainda não existem. Mudar o horário durante o dia vale no mesmo dia; reexecuções não duplicam (graças à `chave_unicidade`).
- Janela dos resumos: `[horário, horário + jitter]` (padrão 10 min) para o envio com horário sorteado. Alertas imediatos: `[agora, agora + 2 min]`. Janela fora do expediente/dia útil move para o próximo início de expediente.

## Retry a pedido, expiração de 30 min e 2 ajustes

- **2 ajustes.** São permitidos no máximo 2 ajustes por leitura (`MAX_AJUSTES`, padrão 2). No 3º pedido de ajuste, o fluxo encerra, o rascunho é descartado e a proprietária é orientada a enviar foto melhor ou texto mais claro.
- **Expiração de 30 min.** A conversa expira por inatividade em `CONV_TTL_MIN` (padrão 30 min); o estado é limpo.
- **Retry a pedido.** Após um resultado final com falhas recuperáveis, o bot oferece "tenta de novo". Por uma janela de `RETRY_REQUEST_WINDOW_H` (padrão 24 h), um `TENTAR_NOVAMENTE` reexecuta **só** os itens elegíveis (retry automático esgotado, ou insumo ausente/inativo) com as **mesmas chaves**, então nunca duplica, e reporta um resultado novo.
- **Falha em um item não bloqueia os demais:** cada item é gravado por conta própria; falhas transitórias vão para a fila de retry (backoff exponencial com jitter) e, esgotadas, para o DLQ (`dlq:items`) com alerta operacional.
- **Nada grava sem confirmação**, e só o chat autorizado (`GRUPO_JID`/`BENEFICIARIA_TELEFONE`) é atendido. Imagens e rascunhos não persistem após o fluxo (só TTL curto em Redis).

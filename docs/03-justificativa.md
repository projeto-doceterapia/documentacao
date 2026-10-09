# 3. Justificativa

> Fontes: `TAP2026.pdf`, `OFICIAL_DOCETERAPIA.docx`, `Anotações - Beneficiário.txt`, `RELATORIO_VISITA_TECNICA.docx`, `SIS - Atividade Integrada com PI - RELATÓRIO ELEVATOR PITCH.docx`.

## 3.1 Problema

Confeiteiras autônomas têm problemas críticos na organização de pedidos e de estoque. Na Doceterapia, os pedidos chegam pelo WhatsApp e são anotados à mão, e o estoque é controlado "no olho". Essa informalidade aumenta o risco de:

- perder informações e esquecer pedidos;
- retrabalho e atrasos na entrega;
- compras desnecessárias ou falta de ingredientes no meio da produção;
- vender com prejuízo, já que o preço é definido sem conhecer o custo real;
- estagnar, sem histórico de clientes para fidelização.

As soluções de mercado (por exemplo, Sortly para estoque e Motion para organização de tarefas, analisadas na jornada do concorrente) são caras ou complexas demais para uma microempreendedora com pouca familiaridade digital.

Se o projeto não for executado, a confeitaria continua exposta a atrasos, esquecimentos e estagnação por falta de histórico estruturado (TAP).

## 3.2 Dores priorizadas pela beneficiária

Na entrevista, as dores foram marcadas por intensidade (quanto mais `*`, maior a ênfase da beneficiária). Consolidação:

| Prioridade | Dor | Ênfase | Como o sistema responde |
|---|---|---|---|
| 1 | Precificação por "achismo", sem custo real nem margem | `***************` | Ficha técnica `ProdutoInsumo`, custo estimado, preço sugerido, margem por item e painel de lucro |
| 2 | Estoque sem controle e sem validação antes de aceitar o pedido | `**` | Cadastro de insumos com quantidade mínima, alerta de estoque baixo e verificação de viabilidade (planejada) |
| 3 | Aceitar pedidos sem controle de capacidade (pedidos de última hora, grande volume) | `**` | Agenda com farol verde/amarelo/vermelho e métrica de dias lotados |
| 4 | Receitas sem registro e sem padronização | `**` | Composição do produto (insumos e quantidades) |
| 5 | Histórico de clientes só na memória | — | Cadastro de clientes com histórico de pedidos e classificação (padrão, frequente, alto volume) |
| 6 | Regra do sinal de 50% controlada à mão | `*****` | Pagamento com cálculo automático de sinal e restante e status de pagamento |
| 7 | Dificuldade com interfaces (botões difíceis de achar, fonte pequena) | — | Interface simples, poucos campos e controle de tamanho de fonte |

## 3.3 Benefícios esperados

- Mais eficiência operacional e organização.
- Planejamento da produção por data e por capacidade.
- Decisões baseadas em dados (lucro, custo, margem, produtos mais pedidos).
- Melhor atendimento e fidelização, com histórico por cliente.
- Contribuição social: fortalecer um microempreendimento local em um setor que tem 75% de negócios com menos de 10 funcionários e mortalidade de 28,54% (Sebrae).

## 3.4 Validação das hipóteses de negócio

As hipóteses de negócio foram validadas na visita técnica de 12/04/2026, com observação direta e teste de usabilidade do protótipo (ver [08 - Registro das entrevistas](08-registro-entrevistas.md#83-visita-técnica-12042026)):

| Hipótese | Resultado |
|---|---|
| Pedidos dispersos no WhatsApp causam perda de informação | Confirmada: falta de centralização e dependência da memória |
| O estoque visual causa falta ou excesso de insumos | Confirmada: não há controle de estoque confiável |
| A usuária adota um sistema se ele for muito simples | Confirmada com ressalva: telas com muitos campos geram dificuldade, e elementos visuais ajudam |
| O sistema substituiria o WhatsApp | **Refutada**: a solução deve complementar o WhatsApp, não substituí-lo |
| Alertas têm valor para a usuária | Confirmada: alertas de estoque e de organização foram valorizados |

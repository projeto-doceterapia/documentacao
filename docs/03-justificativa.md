# 3. Justificativa

## 3.1 Por que a Doceterapia precisa de um sistema

Hoje os pedidos da Doceterapia ficam espalhados entre as conversas do WhatsApp e anotações à mão, e o estoque é controlado no olho. Isso gera riscos concretos para o negócio:

- **Esquecimento e perda de informação:** detalhes do pedido ficam presos na conversa ou numa anotação incompleta.
- **Falta ou excesso de insumos:** sem saber o que há no armário, a confeiteira descobre a falta no meio da produção ou compra o que já tinha.
- **Sobrecarga:** como é difícil recusar clientes antigos, entram mais encomendas do que é possível produzir, o que causa atrasos.
- **Prejuízo invisível:** sem custo por produto, um preço pode não cobrir os ingredientes, e ninguém percebe.
- **Estagnação:** sem histórico de clientes e sem indicadores, não há base para fidelizar nem para planejar.

## 3.2 Dores priorizadas e como o sistema responde

Nas conversas com a confeiteira, cada dor foi marcada com uma intensidade. A tabela vai da mais forte para a mais fraca.

| Prioridade | Dor | Intensidade | Resposta do sistema |
|---|---|---|---|
| 1 | Preço definido por estimativa, sem custo real nem margem | Muito alta | Ficha técnica do produto (insumos e quantidades), custo estimado, preço sugerido, margem por item vendido e painel de lucro |
| 2 | Pagamento do sinal de 50% controlado de cabeça | Alta | Sinal e restante calculados automaticamente, com status de pagamento por pedido |
| 3 | Estoque sem controle e sem conferência antes de aceitar o pedido | Média | Cadastro de insumos com quantidade mínima e alerta de estoque baixo. Verificação de viabilidade planejada |
| 4 | Pedidos aceitos sem olhar a capacidade (última hora, grandes volumes) | Média | Agenda com farol verde, amarelo e vermelho e contagem de dias lotados |
| 5 | Receitas sem registro e sem padrão | Média | Composição de cada produto em insumos |
| 6 | Histórico de clientes só na memória | — | Cadastro de clientes com histórico de pedidos e classificação |
| 7 | Dificuldade com sistemas (botões difíceis de achar, letra pequena) | — | Interface com poucos campos, rótulos em linguagem natural e ajuste do tamanho da fonte |

## 3.3 Por que não usar uma solução pronta

Durante a pesquisa, foram analisadas ferramentas como o **Sortly** (controle de estoque) e o **Motion** (organização de tarefas). As dificuldades encontradas:

- custo mensal alto para um microempreendimento;
- configuração inicial longa e muitos campos obrigatórios;
- foco genérico, sem conceitos da confeitaria como sinal de 50%, ficha técnica e capacidade diária;
- interfaces pensadas para quem já tem familiaridade com tecnologia.

Um sistema sob medida trata só o que importa para a Doceterapia, usa a linguagem dela e mantém o WhatsApp como canal.

## 3.4 O cenário do setor

A situação da Doceterapia é comum no setor:

- O Brasil tem cerca de 179 mil padarias e confeitarias, das quais aproximadamente 116 mil são confeitarias. Elas representam 14,5% do food service nacional.
- O Sudeste concentra 58% das confeitarias, e o estado de São Paulo passa de 40 mil unidades.
- O setor de panificação e confeitaria faturou R$ 153,3 bilhões em 2024 (alta de 10,9%). O mercado de confeitaria e festas deve chegar a R$ 21 bilhões em 2026, puxado por doces artesanais e encomendas personalizadas.
- 75% das confeitarias têm menos de 10 funcionários, e a mortalidade do setor é de 28,54%.
- As principais causas de fechamento são falta de planejamento, gestão deficiente e má gestão de estoque. A ineficiência em compras pode consumir até 3% do faturamento líquido, e cerca de 30% das empresas perdem dinheiro por desorganização dos insumos.

O mercado cresce, mas os pequenos negócios ainda operam no improviso. Organizar pedidos e estoque ataca justamente as causas mais comuns de fechamento.

## 3.5 Benefícios esperados

- **Organização:** todos os pedidos em um lugar, do orçamento à entrega.
- **Planejamento:** a confeiteira vê a carga de cada dia antes de aceitar uma nova encomenda.
- **Precificação consciente:** custo e margem conhecidos para cada produto.
- **Menos desperdício e menos falta:** estoque com mínimo definido e alertas.
- **Fidelização:** histórico e classificação de clientes.
- **Decisão baseada em dados:** lucro, custo, margem e produtos mais vendidos.

## 3.6 Validação com a usuária

As hipóteses que orientaram o sistema foram conferidas com a confeiteira, por observação da rotina e por teste do protótipo (detalhes em [08 - Registro das entrevistas](08-registro-entrevistas.md#83-visita-técnica)):

| Hipótese | Resultado |
|---|---|
| Pedidos espalhados no WhatsApp causam perda de informação | Confirmada |
| O controle visual do estoque causa falta ou excesso de insumos | Confirmada |
| A usuária adota o sistema se ele for muito simples | Confirmada, com a condição de ter poucos campos e apoio visual |
| O sistema deveria substituir o WhatsApp | **Refutada**: ele deve complementar o WhatsApp |
| Os alertas têm valor para a usuária | Confirmada |

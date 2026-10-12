# 1. Contexto

## 1.1 A Doceterapia

A Doceterapia é uma confeitaria artesanal da zona sul de São Paulo, na região de Interlagos. Foi criada em 2016 pela confeiteira Andreia Vieck e funciona como microempresa individual, com produção própria de bolos e doces.

A proprietária toca o negócio com a ajuda de uma ajudante e faz todos os papéis da empresa: vende, compra, produz e administra. A produção é quase toda sob encomenda: pedidos personalizados, festas e eventos, para pessoas físicas e também para empresas com pedidos de grande volume.

O catálogo inclui bolos personalizados, naked cake, bolos em pasta americana, smash cake, bolo de pote, bolo gelado, pão de mel, cupcakes e docinhos de festa (tradicionais e gourmet). Os preços praticados estão em [09 - Regras de negócio](09-regras-de-negocio.md#91-catálogo-e-preços-praticados).

## 1.2 Como o negócio funciona sem o sistema

- **Atendimento:** os clientes chegam por indicação, pelo Instagram e principalmente pelo WhatsApp. É lá que acontecem o atendimento, o orçamento e o fechamento do pedido.
- **Pedidos:** produto, quantidade, sabor, data e forma de entrega são combinados na conversa. Depois a confeiteira anota o pedido à mão, em agenda ou caderno, usando palavras-chave. Não fica histórico organizado.
- **Estoque:** o controle é visual, "no olho". A compra acontece quando um item acaba no armário, e os pedidos são aceitos sem conferir se há insumo suficiente.
- **Produção:** tem várias etapas e depende de tempo (resfriamento, descanso). O preparo dos insumos-base é a etapa mais crítica, e a perecibilidade impede produzir muito com antecedência.
- **Receitas e preços:** as receitas existem só na memória da confeiteira. O preço é definido por estimativa, sem saber o custo real de cada produto.
- **Pagamento:** um pedido só é confirmado com sinal de 50%, e o restante é pago na retirada. Se o cliente cancela, o produto pode ser reaproveitado ou o valor vira crédito.
- **Clientes:** a relação é próxima, baseada em fidelidade e indicação, mas o histórico de cada cliente depende da memória da proprietária.

O fluxo completo desse processo, modelado em BPMN, está em [09 - Regras de negócio](09-regras-de-negocio.md#93-processo-as-is-bpmn).

## 1.3 O problema

Como tudo é informal, a operação fica vulnerável:

- pedidos esquecidos ou registrados pela metade;
- ingredientes que faltam no meio da produção ou que são comprados sem necessidade;
- dias com mais encomendas do que a capacidade de produzir;
- preços que não cobrem o custo e geram prejuízo sem que ninguém perceba;
- nenhuma visão de lucro, margem ou produtos mais vendidos para apoiar decisões.

As ferramentas disponíveis no mercado não resolvem o problema: ou são caras, ou são complexas demais para quem tem pouca familiaridade com tecnologia e usa o celular como principal ferramenta de trabalho.

## 1.4 O sistema Doceterapia

O sistema Doceterapia é uma aplicação web de gestão feita sob medida para a rotina da confeitaria. Ele **não substitui o WhatsApp**, que continua sendo o canal de vendas. A ideia é que tudo o que foi combinado na conversa passe a ficar registrado e organizado em um só lugar.

| Módulo | O que oferece à confeiteira |
|---|---|
| Pedidos | Registro do orçamento em três etapas (cliente e entrega, itens, pagamento), com o sinal de 50% calculado automaticamente e o acompanhamento do status até a entrega |
| Agenda | Calendário mensal com "farol" de ocupação (verde, amarelo, vermelho) para identificar dias lotados antes de aceitar novas encomendas |
| Produção | Fila do que precisa ser produzido, ordenada por prazo e status |
| Histórico | Consulta de todos os pedidos, com filtros por período, etapa e cliente |
| Clientes | Cadastro, classificação (padrão, frequente, alto volume) e histórico de compras |
| Produtos | Catálogo com preço, custo, margem e ficha técnica de insumos |
| Insumos | Controle de quantidade atual e mínima, com alerta de estoque baixo |
| Visão Geral | Lucro, custo, margem, dias lotados, pedidos em andamento e produtos mais pedidos |
| Acessibilidade | Ajuste do tamanho da fonte e interface simples, com poucos campos |

O sistema é **ERP-like**: as áreas do negócio ficam ligadas entre si. Registrar uma venda afeta a agenda e a produção, a produção consome insumos e tudo isso aparece nos indicadores de lucro.

O sistema tem uma única usuária, a proprietária, com perfil de administradora. Também foi desenhada uma integração com o WhatsApp para a confeiteira atualizar o estoque mandando a foto da nota fiscal, com leitura por inteligência artificial. Essa integração existe como prova de conceito (ver [10.8 Integrações](10-arquitetura/08-integracoes.md)).

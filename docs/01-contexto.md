# 1. Contexto

> Fontes: `TAP2026.pdf`, `OFICIAL_DOCETERAPIA.docx`, `RELATORIO_VISITA_TECNICA.docx`, `Anotações - Beneficiário.txt`, `Perguntas.txt`, `_Portfólio Doceterapia (4).pdf`, `comprovanteCNPJ_doceTerapia.pdf`, `Tabela Beneficiários - Frio Quente.xlsx`, `Possibilidade de Beneficiários do Projeto.docx` (pasta do grupo `PI - Grupo 9/`).

## 1.1 O projeto

| Item | Valor |
|---|---|
| Nome do projeto | Gerenciamento de Estoque e Pedidos da Doceterapia |
| Tipo | Projeto de Extensão / Pesquisa e Inovação (PI) |
| Instituição (patrocinador) | São Paulo Tech School (SPTech), curso de Sistemas de Informação, turma 2SIS, 2026 |
| Equipe | Grupo 9 (6 integrantes, ver [12 - Ferramentas de planejamento](12-ferramentas-de-planejamento.md#121-equipe-e-papéis)) |
| Unidade demandante (beneficiária) | Doceterapia, confeitaria artesanal |
| Vigência formal | 23/02/2026 a 18/12/2026 (Termo de Representação) |
| Cronograma do TAP | Início em 09/04/2026, conclusão até 08/12/2026 |
| Classificação do sistema | "ERP-like": integra vendas, agenda/produção, estoque e financeiro em uma única plataforma (Status Report, slide 16) |

## 1.2 A beneficiária

A Doceterapia é uma confeitaria artesanal de São Paulo (zona sul, região de Interlagos), criada em 2016 pela confeiteira Andreia Vieck como uma mudança de carreira. A empresa está registrada como empresário individual de porte ME, com atividade principal **10.91-1-02 – Fabricação de produtos de padaria e confeitaria com predominância de produção própria**, e situação cadastral ativa.

O negócio é tocado pela proprietária com a ajuda de uma ajudante. Ela acumula todos os "departamentos" da empresa: é vendedora, compradora, confeiteira e gestora. Trabalha principalmente sob encomenda, com pedidos personalizados e para eventos, e atende tanto pessoas físicas quanto empresas com pedidos de grande volume.

Linhas de produto (portfólio e cardápio): bolos personalizados, naked cake, bolos em pasta americana, smash cake, bolo de pote, bolo gelado, pão de mel personalizado, cupcakes e docinhos de festa (tradicionais e gourmet). O detalhamento de preços está em [09 - Regras de negócio](09-regras-de-negocio.md#91-catálogo-e-preços-praticados).

> Dados pessoais da proprietária (CPF, endereço completo, telefone) aparecem nas fontes originais, mas foram omitidos aqui de propósito.

## 1.3 Como o negócio funciona hoje (AS-IS)

Levantado na visita técnica de 12/04/2026 e nas entrevistas:

- **Canais:** os clientes chegam por indicação, pelo Instagram e principalmente pelo WhatsApp, que é o canal de atendimento, orçamento e fechamento.
- **Pedidos:** são combinados na conversa (produto, quantidade, sabor, data, entrega) e anotados à mão em agenda ou caderno, por palavras-chave. Não há histórico estruturado.
- **Estoque:** o controle é visual, "no olho". A compra acontece quando um item acaba no armário. Pedidos são aceitos sem verificar se há insumo.
- **Produção:** tem várias etapas e depende de tempo (resfriamento, descanso). O preparo de insumos é a etapa mais crítica, e a perecibilidade impede produzir muito com antecedência.
- **Receitas e custos:** as receitas existem só na memória da confeiteira. A precificação é feita "no achismo", sem custo real por produto.
- **Financeiro:** o pedido só é confirmado com sinal de 50%, e o restante é pago na retirada. Em caso de cancelamento, o produto pode ser reaproveitado ou o valor convertido em crédito.
- **Clientes:** a relação é forte (fidelidade e indicação), mas o histórico depende da memória da proprietária.

O processo atual está modelado em BPMN em [09 - Regras de negócio](09-regras-de-negocio.md#93-processo-as-is-bpmn).

## 1.4 Escolha da beneficiária

O grupo começou prospectando ONGs (Núcleo de Convivência Menino Jesus, Instituição Beneficente Nosso Lar, Centro Juventude). A beneficiária mais promissora parou de responder cerca de uma semana e meia antes da entrega da Sprint 1. No dia seguinte o grupo acionou um plano de contenção (Daily de 25/02/2026) e avaliou novas opções com um mapa de calor (Frio/Morno/Quente):

| Candidato | Temperatura | Observação |
|---|---|---|
| Nosso Lar | Morno | ONG, prospecção original |
| Centro Juventude | Frio | ONG |
| Menino Jesus | Frio | ONG |
| Salão de Beleza | Quente | — |
| **Confeitaria Doceterapia** | Quente → Morno | Contato rápido e engajado com a proprietária. Escopo possível: pedidos, estoque e dashboard financeiro. Cerca de 20 km da SPTech |
| PetShop | Morno → Frio | Agendamentos |

A Doceterapia foi escolhida (Weekly de 27/02/2026). A Nosso Lar recebeu um e-mail de saída honrosa, e o orientador foi comunicado da troca e do novo escopo.

## 1.5 Mercado

Dados de desk research (Relatório Elevator Pitch). As referências completas estão em [14 - Referências](14-referencias.md).

- O Brasil tem 179.044 padarias e confeitarias (63.425 padarias e 115.619 confeitarias), o que representa 14,5% do food service nacional. O Sudeste concentra 58% das confeitarias, e São Paulo tem mais de 40 mil unidades.
- Panificação e confeitaria faturaram R$ 153,3 bi em 2024 (+10,9%). O mercado de confeitaria e festas deve chegar a R$ 21 bi em 2026.
- 75% das confeitarias têm menos de 10 funcionários, e a mortalidade do setor é de 28,54%.
- As principais causas de falência são falta de planejamento, gestão deficiente e má gestão de estoque. Cerca de 30% das confeitarias perdem dinheiro por má organização de insumos.

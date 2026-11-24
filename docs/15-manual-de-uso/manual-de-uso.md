# Manual de uso — Doceterapia

Versão do sistema: v1.0.0 · Data: **[CONFERIR NO RC: data da versão final]**

> Todas as telas e exemplos deste manual usam dados de exemplo. Nenhum cliente, telefone ou valor é real.

> **Rascunho.** Os trechos marcados com **[CONFERIR NO RC]** descrevem funções que ainda serão conferidas na versão candidata. As marcações **[CAPTURA PENDENTE]** indicam onde entra a imagem da tela, feita só com os dados de exemplo. Todas as marcações saem antes da versão final.

## Como ler este manual

O manual segue a ordem do uso: primeiro ligar o sistema, depois o dia a dia da confeitaria e, por último, a manutenção. Cada tela tem a mesma estrutura: para que serve, como chegar, passo a passo, o que a funcionária vê de diferente e os avisos mais comuns.

Palavras técnicas aparecem em `fonte diferente` e estão explicadas no [Glossário](#glossário), no fim.

Os comandos de ligar, desligar e manter o sistema são digitados no **Prompt de Comando** do Windows, dentro da pasta `infra` do sistema. Para abrir: entre na pasta `infra` pelo Explorador de Arquivos, clique na barra de endereço, digite `cmd` e tecle Enter.

## 1. Ligar o sistema pela primeira vez

Faça isto uma única vez, na máquina onde o sistema vai rodar. É preciso ter o Docker Desktop instalado e aberto.

1. **Conferir a máquina:** digite `doceterapia doctor`. Ele confere o Docker, a memória e as portas. Se algo estiver errado, ele mostra o problema e como corrigir. Corrija e rode de novo até não haver erro.
2. **Preparar a configuração:** digite `doceterapia init`. Ele cria o arquivo `.env` com senhas fortes e pergunta:
   - o nome da dona (opcional);
   - o e-mail e a senha da conta da dona. A senha não aparece enquanto você digita: isso é normal;
   - se quer carregar os **dados de demonstração** (responda sim só para conhecer o sistema; para uso de verdade, responda não);
   - os dados do WhatsApp, que podem ser preenchidos agora ou depois (rodar o `init` de novo completa o que faltou e nunca troca uma senha que já existe).
3. **Ligar:** digite `doceterapia up` e espere terminar.
4. **Abrir a tela:** no navegador (Chrome ou Edge), abra o endereço mostrado no fim do `up`. **[CONFERIR NO RC: endereço exato, por exemplo `http://localhost`]**

Nos dias seguintes, basta ligar o Docker Desktop e digitar `doceterapia up`.

## 2. Entrar, sair e tamanho da fonte

**Entrar.** Na tela de login, digite o e-mail e a senha e clique em entrar. O ícone de olho mostra ou oculta a senha. Ao recarregar a página, a sessão continua aberta.

**Sair.** Clique no botão **Sair**, no canto superior direito. Faça isso sempre que a máquina for usada por outra pessoa.

**Tamanho da fonte.** Na barra superior, os botões **−** e **+** ao lado de **Aa** diminuem e aumentam o texto de todas as telas, de 100% a 180%. O sistema lembra a escolha.

[CAPTURA PENDENTE: entrar.png]

## 3. Usuários e perfis

O sistema tem dois perfis:

| O que | DONA | FUNCIONARIA |
|---|---|---|
| Pedidos, Agenda, Produção, Estoque e Clientes | Sim | Sim |
| Indicadores financeiros (faturamento, valores) | Sim | Não |
| Criar produto e ajustar a precificação | Sim | Não |
| Ver clientes inativos | Sim | Não |
| Criar usuários | Sim | Não |
| Cancelar pedido e "Confirmar mesmo assim" com falta de insumo | Sim | Não **[CONFERIR NO RC]** |

**Criar a conta da funcionária (só a DONA):**

1. Na barra superior, clique no ícone **Usuários**.
2. Preencha nome, e-mail e senha e escolha o perfil **FUNCIONARIA**.
3. Clique em **Criar usuário**. A mensagem "Usuário criado." confirma.
4. Passe o e-mail e a senha para a funcionária pessoalmente, nunca por foto.

Aviso comum: "Não foi possível criar o usuário. Tente novamente." Confira se o e-mail já não está em uso.

[CAPTURA PENDENTE: usuarios-criar.png]

## 4. Agenda e farol

**Para que serve:** ver os pedidos por dia de entrega e saber, pela cor do farol, quais dias estão cheios.

**Como chegar:** menu **Agenda**.

**[CONFERIR NO RC]** A Agenda abre mostrando de hoje até daqui a 7 dias. Para ver outro período, use o filtro de datas: a data inicial não pode ser depois da final, e o período máximo é de 366 dias. Enquanto carrega, aparece um aviso de carregamento; se não houver pedidos, a tela diz isso; se houver erro, aparece uma mensagem com a opção de tentar de novo.

A funcionária não vê os valores no resumo da Agenda.

[CAPTURA PENDENTE: agenda.png]

## 5. Registrar um pedido

**Como chegar:** menu **Pedidos** → **Registrar Pedido**.

1. **Cliente:** busque o cliente pelo nome ou cadastre um novo.
2. **Itens:** escolha os produtos e as quantidades.
3. **Entrega:** informe a data e o endereço. **[CONFERIR NO RC]** Digite o CEP (8 dígitos) e o sistema preenche rua, bairro, cidade e estado; complete número e complemento. Se a busca do CEP falhar, preencha o endereço à mão. O endereço cadastrado do cliente pode ser sugerido como endereço de entrega.
4. Revise e confirme.

**Falta de insumo [CONFERIR NO RC].** Ao confirmar o pedido (ou ao mudar os itens de um pedido já confirmado, antes da produção), o sistema confere o estoque. Se faltar algum insumo, abre uma janela com cada insumo que falta e a quantidade:

- a **DONA** pode clicar em **Confirmar mesmo assim**; o pedido fica marcado com um indicador de falta de insumo, para lembrar de comprar;
- a **FUNCIONARIA** só pode fechar a janela e avisar a dona.

[CAPTURA PENDENTE: pedido-registrar.png]
[CAPTURA PENDENTE: pedido-falta-insumo.png]

## 6. Acompanhar o pedido

**[CONFERIR NO RC: seção inteira]**

**Fases do pedido:** orçamento → aguardando sinal → esperando fazer → em produção → pronto → entregue. Um pedido também pode ser cancelado.

- **Produção** (menu **Produção**): mostra os pedidos que estão para fazer ou em produção, em cartões. O botão do cartão avança o pedido para a próxima fase. A seta "Abrir pedido" mostra o detalhe.
- **Histórico** (menu **Pedidos**): lista os pedidos com filtro de período e de situação, em páginas.
- **Detalhe do pedido:** mostra cliente, itens, entrega e pagamento, com os botões de avançar fase, editar e cancelar.

**Avançar fase:** clique no botão da próxima fase. As fases que não podem ser desfeitas pedem confirmação antes. Ao iniciar a produção, o sistema dá baixa nos insumos do estoque.

**Editar:** no detalhe, clique em editar, altere e salve.

**Cancelar (só a DONA):** no detalhe, clique em cancelar, escolha o motivo e confirme.

Toda ação mostra um aviso de sucesso ou de erro no canto da tela.

[CAPTURA PENDENTE: pedido-avancar-fase.png]
[CAPTURA PENDENTE: producao.png]

## 7. Pagamentos: sinal e restante

**[CONFERIR NO RC: seção inteira]**

No detalhe de um pedido confirmado, o painel **Pagamento** mostra o total, o sinal (50% do total), o restante e a situação do pagamento.

1. **Registrar o sinal:** escolha a forma de pagamento, informe a data (opcional; se ficar vazia, vale hoje) e confirme. O pedido passa para "esperando fazer". Depois de registrado, o sinal não pode ser alterado.
2. **Registrar o restante:** escolha a forma de pagamento e a data e confirme. O restante pode ser recebido mesmo depois da entrega.

Se o sistema recusar o registro (por exemplo, sinal já pago), a mensagem explica o motivo.

[CAPTURA PENDENTE: pedido-pagamento.png]

## 8. Estoque

**Para que serve:** cadastrar os insumos, ver o saldo de cada um e acompanhar entradas e saídas.

**Como chegar:** menu **Estoque**.

- Cada insumo mostra o saldo e uma situação: **Em nível normal**, **Nível baixo** ou **Crítico**, conforme o estoque mínimo definido.
- **Novo Insumo:** informe nome, categoria, unidade e estoque mínimo e clique em **Adicionar**. **Nova Categoria** cria uma categoria de insumo.
- **Editar** e **excluir** ficam em cada insumo; a exclusão pede confirmação.
- **Movimentações:** mostra as entradas de compra, as saídas para produção e os saldos iniciais dos últimos 30 dias.
- Quando um insumo fica abaixo do mínimo, chega um aviso no sino e, se ligado, no WhatsApp.

[CAPTURA PENDENTE: estoque.png]

## 9. Notas fiscais e estoque pelo WhatsApp

O assistente de WhatsApp registra compras de insumos a partir da foto da nota fiscal. Ele conversa só com o celular autorizado da dona.

1. Do celular autorizado, mande a foto da nota fiscal para o número do assistente.
2. O assistente lê a nota e responde com a lista de itens, quantidades e valores.
3. Confira. Se algo estiver errado, responda corrigindo (há um limite de ajustes por compra).
4. Confirme. A compra entra no estoque como "Entrada de compra".

**Limites do número de teste da Meta:**

- o número de teste só conversa com até 5 números autorizados no painel da Meta;
- o assistente só responde com texto livre até 24 horas depois da última mensagem da dona (a **janela de 24 h**); fora dela, os avisos só chegam por mensagens pré-aprovadas (**templates**).

O assistente também manda avisos (estoque baixo, dia lotado, resumos do dia), conforme a seção 13.

[CAPTURA PENDENTE: whatsapp-nota.png — número do celular coberto]

## 10. Produtos, ficha técnica e preço sugerido

**Como chegar:** menu **Produtos**.

- O catálogo mostra os produtos por categoria.
- **Novo Produto** (só a DONA): informe nome, categoria, unidade e preço e clique em **Salvar produto**.
- **Ficha técnica:** em cada produto, informe os insumos e as quantidades usadas. Com a ficha, o sistema calcula o custo e o **preço sugerido**.
- **Precificação** (só a DONA): define a margem usada no preço sugerido. A margem precisa ser maior ou igual a zero.
- Produtos com preço abaixo do sugerido aparecem destacados; use **Mostrar só esses** para vê-los.

[CAPTURA PENDENTE: produtos-ficha.png]

## 11. Clientes

**Como chegar:** menu **Clientes**.

- **Novo cliente:** preencha os dados. No endereço, digite o CEP (8 dígitos) para preencher rua, bairro, cidade e estado, ou deixe o CEP vazio. Clique em salvar; a mensagem "Cliente salvo." confirma.
- **Busca:** digite o nome no campo de busca. Os filtros separam clientes frequentes e de alto volume.
- **Inativar e reativar:** um cliente inativo some das listas, mas não é apagado. Só a DONA vê os inativos (**Mostrar inativos**).

Aviso comum: "Informe os 8 dígitos do CEP ou deixe o campo vazio."

[CAPTURA PENDENTE: clientes.png]

## 12. Visão Geral e relatório

**Como chegar:** menu **Visão Geral**.

- Escolha o período (por exemplo, **Este mês** ou **Mês passado**).
- Os cartões mostram pedidos em andamento, produtos mais pedidos e insumos que pedem atenção.
- A DONA vê também faturamento, ticket médio e pedidos em orçamento, com o gráfico de faturamento.
- **Gerar relatório** abre o **Relatório do período**, pronto para imprimir ou salvar em PDF pelo navegador.

[CAPTURA PENDENTE: visao-geral.png]

## 13. Configurações de avisos e do assistente

**Como chegar:** clique no sino da barra superior e depois em **Configurar notificações**.

- **Assistente ativo:** desligado, pausa a conversa, a leitura de notas e os avisos pelo WhatsApp.
- **Avisos automáticos:** desligado, pausa só os avisos no WhatsApp; eles continuam no sino.
- Os avisos podem ser ligados um a um: **Resumo de início do dia**, **Resumo de fim do dia**, **Alerta de estoque baixo** e **Alerta de dia lotado**, com os dias da semana.
- **Enviar notificação de teste** cria um aviso de teste no sino e, com o assistente e os avisos ligados, no WhatsApp.

[CAPTURA PENDENTE: configuracoes-avisos.png]

## 14. Manutenção

Todos os comandos são digitados no Prompt de Comando, na pasta `infra`.

| Comando | O que faz |
|---|---|
| `doceterapia down` | Desliga o sistema. Nunca apaga dados |
| `doceterapia backup` | Grava uma cópia do banco e da sessão do WhatsApp em `backups\` |
| `doceterapia restore` | Sem nome de pasta, lista os backups. Com a pasta, pede que se digite `RESTAURAR`, faz um backup de segurança e restaura |
| `doceterapia atualizar` | Faz backup, baixa a versão nova e liga o sistema |
| `doceterapia logs <serviço>` | Mostra o registro de um serviço. Sem nome, lista os serviços |
| `doceterapia aplicar <serviço>` | Faz valer uma mudança feita no `.env` (seção 15) |

Recomendações:

- Faça um backup pelo menos uma vez por semana e guarde a pasta `backups` também fora da máquina (um pen drive, por exemplo).
- Antes de atualizar, o próprio `atualizar` faz um backup. Não há suporte formal depois da entrega: atualize só quando houver uma versão nova anunciada.
- Nunca apague a pasta `infra` nem o arquivo `.env`: sem eles, o sistema não liga e as senhas se perdem.

## 15. Trocar a chave da IA, o modelo ou o token do WhatsApp

Use este passo a passo quando a chave da IA vencer ou atingir o limite, quando quiser trocar o modelo da IA ou quando o token do WhatsApp mudar.

1. Faça um backup antes: `doceterapia backup`.
2. Abra a pasta `infra` e o arquivo `.env` no Bloco de Notas. **Ele guarda senhas: não envie, não fotografe e não cole o conteúdo em conversas.**
3. Altere só a linha necessária, sem espaços em volta do `=` e sem aspas:
   - chave da IA: `GEMINI_API_KEY=<nova chave>`;
   - modelo da IA: `GEMINI_MODEL=<nome do modelo>`;
   - token do WhatsApp: `CLOUD_API_TOKEN=<novo token permanente>`.
4. Salve com **Arquivo → Salvar** (não use "Salvar como") e feche o Bloco de Notas.
5. Aplique a troca: `doceterapia aplicar orquestrador`. **Não use `docker compose restart`**: ele reinicia o serviço sem ler o `.env` de novo, e a troca não vale.
6. Confira: `doceterapia logs orquestrador` não deve mostrar erro de autenticação. Depois, do celular autorizado, mande "oi" ao número do assistente: ele responde.

Avisos:

- A chave de IA entregue com o sistema é do nível gratuito e serve só para os dados de exemplo. Para usar com dados reais, crie a sua própria chave no Google AI Studio e defina o seu limite de gasto.
- O token gerado na tela de Configuração da API da Meta expira em 24 horas. O assistente precisa do **token permanente** de um usuário do sistema da Meta.
- **Erro 190** nos logs significa token inválido ou expirado: gere um token novo e repita este passo a passo.
- Lembre os limites do número de teste (seção 9): até 5 números autorizados e avisos fora da janela de 24 h só por template.

## 16. Plano B: WhatsApp pelo adaptador Baileys

**Quando usar:** só se o canal oficial do WhatsApp (Cloud API) parar de funcionar. O plano B usa um número de WhatsApp separado, pareado ao sistema como um aparelho conectado. Use sempre um número secundário dedicado ao sistema, nunca o número principal de alguém.

**Ativar [CONFERIR NO RC]:** no `.env`, troque `CHANNEL=cloud_api` por `CHANNEL=baileys` e salve. Rode `doceterapia up`. Na primeira vez, leia o QR que aparece em `doceterapia logs adaptador-baileys` com o celular do número do plano B (WhatsApp → Aparelhos conectados → Conectar um aparelho).

**Reiniciar o adaptador sem o assistente responder a si mesmo [CONFERIR NO RC]:** o adaptador guarda por 15 minutos, na memória, a lista das mensagens que o próprio assistente mandou. Um reinício apaga essa lista, e o assistente pode tratar uma mensagem dele como se fosse sua. Por isso:

1. Termine ou cancele qualquer conversa em andamento com o assistente.
2. Espere 15 minutos sem o assistente mandar mensagem.
3. Reinicie: `doceterapia aplicar adaptador-baileys`.
4. Confira `doceterapia logs adaptador-baileys`: deve indicar conectado, sem pedir QR.
5. Se o assistente responder a uma mensagem que ele mesmo mandou, não confirme nada nessa conversa e avise quem cuida da máquina.

**Abrir o celular a cada 14 dias:** o WhatsApp desconecta os aparelhos conectados quando o celular principal fica muito tempo sem uso. Abra o WhatsApp no celular do número do plano B pelo menos a cada 14 dias; se a conexão cair, será preciso ler o QR de novo.

## 17. Problemas comuns

| Problema | O que fazer |
|---|---|
| A tela não abre no navegador | Confira se o Docker Desktop está aberto e rode `doceterapia up`. Se continuar, rode `doceterapia doctor` |
| "E-mail ou senha inválidos" ao entrar | Confira o e-mail e a senha. A DONA pode criar outro usuário para a funcionária |
| Esqueci a senha da DONA | Peça ajuda a quem cuida da máquina; não há recuperação automática **[CONFERIR NO RC]** |
| A busca do CEP não preenche o endereço | Preencha o endereço à mão; a busca depende da internet |
| O assistente não responde no WhatsApp | Confira o interruptor **Assistente ativo** (seção 13), depois `doceterapia logs orquestrador`. Erro 190: troque o token (seção 15) |
| Aparece "Assistente pausado. Reative em Configurações." | Ligue **Assistente ativo** (seção 13) |
| Troquei a chave no `.env` e nada mudou | Rode `doceterapia aplicar orquestrador`; o `restart` não lê o `.env` |
| Preciso voltar os dados de um dia anterior | `doceterapia restore` para listar os backups e depois `doceterapia restore <pasta>` |

## Glossário

| Termo | Significado |
|---|---|
| Aparelho conectado | Computador ou sistema ligado a uma conta de WhatsApp pelo QR, como o WhatsApp Web |
| Backup | Cópia de segurança do banco de dados e da sessão do WhatsApp |
| Baileys | Programa que liga o sistema ao WhatsApp como um aparelho conectado; usado só no plano B |
| CEP | Código de Endereçamento Postal, com 8 dígitos |
| Cloud API | Canal oficial do WhatsApp, oferecido pela Meta; é o canal padrão do assistente |
| Dados de demonstração | Pedidos, clientes e produtos de exemplo, sem nenhuma pessoa real |
| Docker | Programa que roda as partes do sistema na máquina; o Docker Desktop precisa estar aberto |
| DONA | Perfil com acesso total: valores, usuários, cancelamento e precificação |
| `.env` | Arquivo da pasta `infra` com as configurações e senhas do sistema; nunca deve ser compartilhado |
| Erro 190 | Código da Meta para token inválido ou expirado |
| Farol | Cor que indica se um dia da Agenda está tranquilo ou lotado |
| Ficha técnica | Lista dos insumos e quantidades usados para fazer um produto |
| FUNCIONARIA | Perfil para o dia a dia, sem valores financeiros, usuários nem cancelamento |
| IA | Inteligência artificial; no sistema, lê as fotos das notas fiscais |
| Insumo | Ingrediente ou material usado na produção |
| Janela de 24 h | Período de 24 horas após a última mensagem da dona em que o assistente responde com texto livre |
| Logs | Registro do que um serviço fez, usado para descobrir erros |
| Modelo (da IA) | Versão da IA usada para ler as notas, definida em `GEMINI_MODEL` |
| Orquestrador | Parte do sistema que conversa com a IA, com o WhatsApp e com o restante do sistema |
| Plano B | Canal alternativo do WhatsApp pelo Baileys, usado só se a Cloud API falhar |
| Prompt de Comando | Janela do Windows onde se digitam os comandos `doceterapia` |
| QR | Código quadrado lido pela câmera do celular para conectar um aparelho |
| Restante | Parte do valor do pedido paga depois do sinal |
| Sinal | Primeira parte do pagamento, 50% do total, que confirma o pedido |
| Template | Mensagem pré-aprovada pela Meta, obrigatória para avisos fora da janela de 24 h |
| Token | Senha de acesso que permite ao sistema usar a Cloud API |
| Token permanente | Token de um usuário do sistema da Meta, que não expira em 24 horas |

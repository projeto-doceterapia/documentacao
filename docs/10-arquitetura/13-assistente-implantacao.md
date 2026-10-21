# 10.13 Assistente de estoque — implantação e operação

> **Situação:** implementado e em revisão nos pull requests dos repositórios de produto; ainda não implantado em produção.

Como o sistema sobe e é operado, como implementado no repositório `infra` e nos runbooks de operação do projeto.

## Único ponto de entrada: `infra`

- `docker compose up` no `infra` baixa imagens **pré-construídas e fixadas** do GHCR e sobe `banco-de-dados`, `back-end`, `front-end`, `redis` e `orquestrador`. Nenhum outro repositório precisa ser clonado para rodar o sistema.
- `--profile baileys` adiciona o `adaptador-baileys` (volume de sessão `adaptador-baileys-session`).
- Só a porta do `front-end` (3000) e o webhook do orquestrador são publicados; o resto fica na rede interna.
- `compose.local.yml` é o único lugar que **constrói** a partir do código (desenvolvimento), e publica a porta do back-end para o Swagger. O perfil `tunnel` sobe um `ngrok` só para testes do webhook Cloud API.
- **Configuração centralizada:** todas as variáveis de todos os serviços ficam no `infra/.env` (git-ignored); `infra/.env.example` lista todas sem valores. Cada serviço recebe só as variáveis de que precisa.

## Dois alvos de implantação

| | Computador da proprietária | Host em nuvem (VPS / plataforma de contêiner) |
|---|---|---|
| Preparo do host | só Docker Desktop | Docker (ou imagens avulsas, pelo padrão de imagem standalone) |
| Disponibilidade | só com a máquina ligada; resumos/alertas perdidos são enviados na recuperação se a janela ainda estiver aberta | sempre no ar |
| Acesso ao web app | `http://localhost:3000` | exige HTTPS, domínio e **o fechamento das lacunas de autenticação** (auth serviço-a-serviço e rotas de negócio protegidas) antes de expor |
| Modo Baileys | funciona (não precisa de endpoint de entrada) | funciona |
| Modo Cloud API | precisa de túnel HTTPS público permanente (ngrok pago ou similar) | usa o endpoint HTTPS do host |
| Custo | energia e a máquina | hospedagem mensal |
| Local dos dados | na máquina da proprietária | na infraestrutura do provedor |

> **Pendente/bloqueado:** expor o web app em host na nuvem está **bloqueado** pela falta de autenticação serviço-a-serviço no orquestrador e pelas rotas de negócio ainda em `permitAll`. Até fechá-las, acessar o web app num host de nuvem só por túnel SSH/rede privada. O estudo de custo dos dois alvos não é repetido aqui.

## Pipeline de publicação e nomes das imagens

- Cada repositório de serviço publica sua imagem no GHCR por CI própria, no merge para `main` (`sha-<short>`) e em releases (`<semver>`), com `GITHUB_TOKEN` (`packages: write`). Pacotes devem ser públicos (verificar o pull anônimo após a primeira publicação).
- Imagens: `ghcr.io/projeto-doceterapia/{back-end, front-end, banco-de-dados, orquestrador, adaptador-baileys}`.
- **Ordem das branches `ci/`:** quando uma demanda traz pipeline/contêiner, ela entra primeiro como uma branch `ci/<nome>` própria, e a branch de feature é empilhada em cima — assim as pipelines já existem quando a feature faz merge.
- **Ordem de deploy entre repositórios:** o alinhamento de schema vem antes do compose; o `back-end` antes do `front-end`: schema → back-end → front-end → orquestrador/adaptador.

## Runbooks

Os procedimentos operacionais completos (instalar nos dois alvos, parear a sessão Baileys, trocar de canal, atualizar/rollback, incidentes) estão no runbook de operação do assistente. Resumo dos playbooks de incidente:

| Incidente | Impacto | Ação |
|---|---|---|
| Sessão Baileys caída | sem envio/recebimento; notificações ficam `PENDENTE`/visíveis no web app | logs do adaptador; reconecta sozinho em quedas de rede; `logged_out` → reparear (QR) ou restaurar o volume de sessão |
| Itens no DLQ | itens de compra com falha definitiva | pedir "tenta de novo" no grupo dentro da janela de 24 h; sem endpoint administrativo de DLQ |
| Core (back-end) fora | compras não gravam; confirmações ficam em retry | logs do back-end e do MySQL; resolve sozinho quando o core volta |
| Redis fora | fluxos em andamento param; AOF retoma no restart | logs do redis; conferir `REDIS_PASSWORD`; reiniciar |
| Notificação perdida | janela fechou antes do envio | nada a recuperar; a cópia fica no web app; o resumo do dia seguinte carrega os pendentes |
| Suspeita de banimento | sem canal WhatsApp | usar o **web app** (notificações visíveis lá); trocar para Cloud API é mudança de config, mas exige número dedicado e templates |

Alertas operacionais também viram `ALERTA_OPERACIONAL` in-app, então a proprietária os vê mesmo quando o WhatsApp é a parte que falhou. Os textos de alerta dizem só a condição, nunca dados da nota ou segredos.

## Declaração de uso de dados (para a proprietária)

Texto publicado para a proprietária, em PT-BR, transcrito na íntegra:

> ## Declaração de uso de dados
>
> Este documento explica, em linguagem simples, como o assistente de estoque pelo
> WhatsApp usa os seus dados. Ao usar o assistente, você está ciente e de acordo
> com o que está escrito abaixo.
>
> ### O que acontece com os seus dados
>
> - Os seus dados de compra (fotos de notas e cupons, textos que você envia e os
>   lançamentos de estoque) são usados **apenas para o sistema funcionar**.
> - Esses dados passam **somente** pelos serviços necessários para o assistente
>   trabalhar: a inteligência artificial que lê a compra e o WhatsApp por onde você
>   conversa com o assistente.
> - Os seus dados **não são vendidos** nem compartilhados com ninguém para
>   qualquer outra finalidade.
>
> ### O que é apagado e o que é guardado
>
> - As **imagens** das notas e os **rascunhos** da conversa são **apagados ao fim
>   de cada fluxo**, assim que a compra é tratada.
> - Fica guardado **apenas o lançamento de estoque registrado** — ou seja, o
>   resultado da compra no seu controle de estoque, e não a foto nem a conversa.
>
> ### Como revogar
>
> Se você quiser deixar de usar o assistente ou retirar esse consentimento, basta
> avisar a equipe do projeto, que interrompe o uso dos seus dados pelo assistente.
> Fale com a equipe responsável pelo projeto Doceterapia.

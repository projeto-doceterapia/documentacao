# 10.10 Assistente de estoque — integrações

> **Situação:** implementado e em revisão nos pull requests dos repositórios de produto; ainda não implantado em produção.

Integrações externas do assistente, como estão implementadas. O canal de WhatsApp é **intercambiável** por configuração (`CHANNEL=baileys|cloud_api`); os dois passam o mesmo conjunto de testes de contrato (`tests/channel_contract.py`), então trocar de canal é mudança de configuração, não de código.

## WhatsApp — canal Baileys (modo primário)

| Item | Detalhe |
|---|---|
| Biblioteca | `@whiskeysockets/baileys` (pacote oficial), adaptador Node 22 + TypeScript |
| Tipo | Cliente não oficial do WhatsApp Web, como **dispositivo vinculado** (linked device) |
| Pareamento | QR lido dos logs do adaptador, escaneado em "Dispositivos conectados" do WhatsApp |
| Sessão | Persistida no volume `adaptador-baileys-session` (`BAILEYS_SESSION_DIR=/data/session`); reinícios reconectam sem novo QR |
| Allowlist | Processa **apenas** mensagens do grupo em `GRUPO_JID`; o resto é descartado antes de logar, guardar ou encaminhar |
| Segurança do túnel interno | Eventos encaminhados ao orquestrador com assinatura HMAC-SHA256 (`X-Signature-256`, chave `WEBHOOK_SECRET`) e segredo compartilhado `ADAPTER_SECRET` (`X-Adapter-Secret`) |
| Saúde | `GET /health` no adaptador: `connected | degraded | down`; eventos `connected`, `disconnected`, `logged_out` |

- **Risco de banimento (número principal).** O assistente começa no modo Baileys **no número principal da confeiteira**, porque não existe um número secundário. Isso eleva o *impacto* de um banimento (afetaria o WhatsApp pessoal), e o trade-off é aceito e documentado. A estimativa de risco, sem taxa oficial, é de **~3–12% ao ano** (confiança baixa–média) para este uso específico (um só destinatário, grupo dedicado, sem envios em massa, horários com jitter). Essa estimativa vem de um estudo comparativo dos canais de WhatsApp, por ausência de dados oficiais do fornecedor. As mitigações — allowlist do grupo, sem envios em massa, jitter nos horários e plano B de troca de canal — estão aplicadas no assistente.

## WhatsApp — Meta Cloud API (fallback compatível)

| Item | Detalhe |
|---|---|
| Implementação | `CloudApiChannel` no orquestrador |
| Versão do Graph | `CLOUD_API_GRAPH_VERSION`, padrão `v23.0` (suportada até 2027-10-08; verificado em 2026-10-20) |
| Credenciais | `CLOUD_API_PHONE_ID`, `CLOUD_API_TOKEN`, `CLOUD_API_APP_SECRET`, `CLOUD_API_VERIFY_TOKEN` |
| Webhook | Handshake com `CLOUD_API_VERIFY_TOKEN`; corpo validado por `X-Hub-Signature-256` |
| Templates | Envios proativos fora da janela de 24 h exigem **template aprovado**. Parâmetros de template **não aceitam quebras de linha**, então seções de lista dos resumos são unidas inline com ` \| ` |
| Número | Um número registrado na Cloud API **sai** do app do WhatsApp e não pode ser usado pelo Baileys. Produção exige um número dedicado; o número principal **nunca** pode ser registrado aqui |

> **Pendente:** a Cloud API foi exercitada **apenas com o número de teste de desenvolvedor da Meta**. Uso em produção depende de um número dedicado (inexistente hoje), verificação de negócio na Meta e aprovação dos templates — passos no console da Meta, **fora dos repositórios** e não validados aqui.

Twilio e WAHA foram avaliados apenas como referência nesse estudo comparativo e **não** são integrados.

## IA — Google Gemini (extração da compra)

| Item | Detalhe |
|---|---|
| Porta | `LlmPort` no orquestrador (`app/llm/port.py`); provedor selecionado por `LLM_PROVIDER` (padrão `gemini`) |
| SDK | `google-genai` (oficial). Modelo em `GEMINI_MODEL`, chave em `GEMINI_API_KEY` |
| Papel | Extrai **fatos crus** (contagem, tamanho de embalagem, unidade, valor líquido por linha) em saída estruturada validada por schema; também classifica a categoria de insumos novos |
| O que a IA **não** faz | Não calcula totais nem conversões (feito em código, [10.12](12-assistente-regras.md)); não escolhe ferramentas nem cria categorias; não grava nada sem confirmação |
| Testes | Provedor `scripted`/fake que reproduz respostas gravadas (`FAKE_LLM_FIXTURES`); nunca chama a rede. Não há chave de LLM nos testes |

- **Premissa do proprietário:** a conta Gemini usada atende aos requisitos de dados do projeto, inclusive não treinar com o conteúdo enviado. Isso é registrado como premissa fornecida pelo proprietário, **não verificada** por esta documentação.
- Trocar o provedor de IA é previsto por `LLM_PROVIDER`, condicionado a passar o conjunto de avaliação com ao menos 90% de acerto por item. **Pendente:** ver o gate de acurácia em [10.14](14-assistente-rastreabilidade.md).

## Publicação de imagens — GHCR

- Cada repositório de serviço publica uma imagem pronta para implantar no **GitHub Container Registry** (`ghcr.io/projeto-doceterapia/*`) pela sua própria CI.
- Disparo: no merge para `main` (tag `sha-<short>`) e em tags de release (`<semver>`).
- As CIs usam o `GITHUB_TOKEN` do repositório com `packages: write`; nenhum token pessoal é envolvido.
- Como todos os repositórios são públicos, os **pacotes também precisam ser públicos**; um pull anônimo é verificado após a primeira publicação de cada pacote.
- As imagens seguem um padrão comum: multi-stage, base fixada e enxuta, usuário não-root, `HEALTHCHECK` e labels OCI (`org.opencontainers.image.source`, `version`, `revision`) para ligar cada pacote ao seu repositório.
- O `infra` consome essas imagens por **tags fixadas** em `infra/.env` (nunca `latest`).

ï»¿# transformsite

[![transformsite ê¢â¬â seu site vira chat](guia/assets/banner.jpg)](https://inematds.github.io/transformsite/guia/)

**ê°Å¸â¡Â§ê°Å¸â¡Â· [PortuguêÂªs](README.md) êÂ· ê°Å¸â¡Âºê°Å¸â¡Â¸ [English](README.en.md) êÂ· ê°Å¸â¡Âªê°Å¸â¡Â¸ [EspaêÂ±ol](README.es.md)**

**Transforme seu site e sua base de conhecimento em um agente de atendimento por chat.**
O site vira a fonte de verdade; o cliente conversa por **WhatsApp, Telegram, e-mail ou widget no site**.
O agente **responde citando a fonte** (ou diz "nêÂ£o sei" e passa para a equipe) e **executa serviêÂ§os**
(agendar, fale conosco, segunda viaê¢â¬Â¦) coletando os dados por conversa, em vez de formulêÂ¡rios.

## ê°Å¸ââ Guia de uso

Guia completo (landing + passo a passo): **https://inematds.github.io/transformsite/guia/** êÂ· ê°Å¸âÂºê¯Â¸Â [Roadmap](ROADMAP.md)

> InspiraêÂ§êÂ£o: o **America.gov** (2026) juntou o conteêÂºdo de ~29 mil sites do governo americano num êÂºnico
> ponto de entrada conversacional ê¢â¬â primeiro perguntas com fonte, depois transaêÂ§êÂµes. O transformsite êÂ© a
> versêÂ£o aberta e simples dessa ideia para qualquer empresa, escola ou êÂ³rgêÂ£o.

## Como funciona

```
site + docs ê¢ââ¬ê¢ââ¬ingestê¢ââ¬ê¢ââ¬ê¢âÂº êÂ­ndice (BM25 + vetores) ê¢ââ¬ê¢ââ¬ê¢âÂº RAG com citaêÂ§êÂ£o ê¢ââ¬ê¢âÂ
formulêÂ¡rios legados ê¢ââ¬ê¢ââ¬convertê¢ââ¬ê¢ââ¬ê¢âÂº services/*.yaml ê¢ââ¬ê¢ââ¬ê¢âÂº mêÂ¡quina de slots ê¢âÂ¼ê¢ââ¬ê¢ââ¬ê¢âÂº Telegram êÂ· WhatsApp êÂ· e-mail êÂ· widget
                                         ferramentas (agenda, tickets, POST no legado, webhook) ê¢âË
                                         handoff humano + painel admin + mêÂ©tricas
```

- **Tudo local por padrêÂ£o**: LLM no [Ollama](https://ollama.com) (ou vLLM/LM Studio), embeddings `bge-m3`,
  êÂ­ndice num êÂºnico arquivo SQLite. TambêÂ©m funciona com **Claude Code** (`claude -p`) ou **Codex CLI**
  (`codex exec`) pela sua assinatura ê¢â¬â sem chave de API.
- **ServiêÂ§o = um arquivo YAML.** Sem cêÂ³digo por serviêÂ§o: slots com tipos validados (CPF, e-mail, telefone,
  CEP, data/hora em portuguêÂªsê¢â¬Â¦), confirmaêÂ§êÂ£o, idempotêÂªncia, handoff.
- **O legado continua vivo**: o conversor gera serviêÂ§os que fazem o mesmo POST que o formulêÂ¡rio antigo fazia.
- **LGPD**: PII mascarada em logs, retenêÂ§êÂ£o configurêÂ¡vel, `privacy forget`, OTP antes de dados sensêÂ­veis.

## InstalaêÂ§êÂ£o rêÂ¡pida (5 minutos)

```bash
pipx install git+https://github.com/inematds/transformsite      # ou: pip install .
ollama pull qwen3.6:35b-a3b && ollama pull bge-m3                # qualquer modelo instruct serve (ex.: qwen2.5:7b)

transformsite init meubot --org "Minha Empresa" --url https://www.minhaempresa.com.br --contact atendimento@minhaempresa.com.br
cd meubot
transformsite inventory          # Fase 0: pêÂ¡ginas e formulêÂ¡rios do site ê¢â â inventario/*.csv
transformsite ingest             # Fase 1: lêÂª o site inteiro + docs/ ê¢â â kb/index.sqlite
transformsite chat               # conversa no terminal
transformsite serve              # widget em /chat, painel em /admin, canais configurados
```

Sem GPU? Use um modelo menor (`qwen2.5:7b`, `llama3.1:8b`) ou `llm.provider: claude_cli` / `codex_cli` (experimental: rodam sem erro, mas no piloto ainda responderam "nêÂ£o sei" onde o Ollama acerta ê¢â¬â em investigaêÂ§êÂ£o).

## Comandos

| Comando | O que faz |
|---|---|
| `init <pasta>` | cria o projeto (config, 2 serviêÂ§os de exemplo, cenêÂ¡rios de teste) |
| `inventory` | rastreia o site e lista pêÂ¡ginas e formulêÂ¡rios com prioridade sugerida |
| `ingest` | crawler (sitemap + links, respeita robots.txt) + docs locais ê¢â â êÂ­ndice hêÂ­brido; reingestêÂ£o incremental por hash |
| `ask "pergunta"` / `chat` | testa no terminal |
| `golden` | gera rascunho de perguntas de teste a partir da base (para revisar) |
| `eval` | mede: citaêÂ§êÂ£o vêÂ¡lida, "nêÂ£o sei" fora de escopo, injeêÂ§êÂ£o bloqueada, latêÂªncia; roda cenêÂ¡rios de conversa |
| `validate` | valida `services/*.yaml` (schema + regras: confirmaêÂ§êÂ£o, idempotêÂªncia, ferramentas existentes) |
| `convert form.html\|form.pdf\|URL` | formulêÂ¡rio legado ê¢â â `service.yaml` em `review: pending` |
| `serve` | servidor: widget web, webhooks (WhatsApp/Telegram), pollers (Telegram/IMAP), painel |
| `report --since 7` | mêÂ©tricas (resoluêÂ§êÂ£o sem humano, handoff, tempo atêÂ© concluir, abandono por slot) em JSON/CSV |
| `privacy forget --user canal:id` / `privacy purge` | LGPD: apagar um usuêÂ¡rio / expurgar por retenêÂ§êÂ£o |

## Um serviêÂ§o

```yaml
service: agendar
review: approved            # serviêÂ§os convertidos nascem "pending" e sêÂ³ sobem apêÂ³s aprovaêÂ§êÂ£o
intent:
  description: "Agendar um atendimento"
  examples: ["quero agendar", "marcar horêÂ¡rio"]
slots:
  - {name: nome, type: text, prompt: "Qual êÂ© o seu nome?"}
  - {name: email, type: email, prompt: "Qual e-mail para o convite?"}
  - name: data_hora
    type: datetime
    prompt: "Qual dia e horêÂ¡rio prefere?"
    options_from: tool:agenda.slots_livres          # oferece horêÂ¡rios livres como botêÂµes
    constraints: {validate_tool: agenda.validar_horario}
confirm: {template: "Confirmo {nome} em {data_hora|fmt_br}?"}
action:
  tool: agenda.criar_evento
  args: {title: "Atendimento ê¢â¬â {nome}", start: "{data_hora}", email: "{email}"}
  idempotency_key: "{session_id}:{data_hora}"
on_success: {reply: "Agendado! Protocolo {result.id}."}
```

O cliente pode responder tudo de uma vez ("sou a Ana, ana@x.com, quinta êÂ s 10"), corrigir ("nêÂ£o, o e-mail
êÂ©ê¢â¬Â¦"), perguntar algo no meio (o bot responde com fonte e volta ao pedido), cancelar ou voltar dias depois.

Ferramentas prêÂ³prias: crie `tools/minha.py` com `register(registry)` ê¢â¬â veja `tools/exemplo.py`.

## Canais

| Canal | Como ativar |
|---|---|
| Widget web | ligado por padrêÂ£o: `<script src="https://SEU_DOMINIO/widget.js"></script>` |
| Telegram | `TELEGRAM_BOT_TOKEN` no `.env` (crie o bot no @BotFather) |
| WhatsApp (Evolution API, self-hosted) | `channels.whatsapp` + `EVOLUTION_API_KEY`; webhook `POST /webhook/whatsapp` |
| WhatsApp (Cloud API oficial) | `channels.whatsapp_cloud` (`phone_number_id`, `token_env`, `verify_token`); webhook `/webhook/whatsapp-cloud` |
| E-mail | `channels.email.enabled: true` + IMAP/SMTP |

Handoff humano: `handoff.targets` (`panel`, `email:...`, `telegram:<chat_id>`). No painel, um atendente
**assume** a conversa (o bot silencia) e **devolve** quando terminar (o bot retoma de onde parou).

## Deploy numa VPS

```bash
curl -fsSL https://raw.githubusercontent.com/inematds/transformsite/main/deploy/vps/install.sh | sudo bash -s -- \
  --domain bot.suaempresa.com.br --site https://www.suaempresa.com.br --org "Sua Empresa" --contact atendimento@suaempresa.com.br --whatsapp
```

Sobe com Docker Compose: **app + Ollama + Evolution API (WhatsApp) + Caddy (HTTPS automêÂ¡tico)**. Detalhes em
[`deploy/vps/README.md`](deploy/vps/README.md).

## Piloto: INEMA.CLUB

A pasta [`pilot/`](pilot/) êÂ© o projeto real usado para validar o framework com o site
[inema.club](https://www.inema.club): 386 pêÂ¡ginas ê¢â â 1.512 trechos; com 73 perguntas de teste e LLM local,
**92,6%** de respostas com fonte vêÂ¡lida, **100%** de "nêÂ£o sei" fora do escopo, **100%** de injeêÂ§êÂ£o
bloqueada, p95 de 2,6 s; **13/13** roteiros de conversa. Detalhes em [`pilot/RESULTADOS.md`](pilot/RESULTADOS.md).

## Desenvolvimento

```bash
uv venv && uv pip install -e '.[dev]'
pytest -q                       # sem rede, LLM falso determinêÂ­stico
```

LicenêÂ§a MIT êÂ· feito pelo [INEMA](https://www.inema.club) ê¢â¬â conteêÂºdo aberto e gratuito.

---

Projeto  - ARCKHÉe
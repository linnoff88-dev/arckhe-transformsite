# transformsite

[![transformsite â€” seu site vira chat](guia/assets/banner.jpg)](https://inematds.github.io/transformsite/guia/)

**ðŸ‡§ðŸ‡· [PortuguÃªs](README.md) Â· ðŸ‡ºðŸ‡¸ [English](README.en.md) Â· ðŸ‡ªðŸ‡¸ [EspaÃ±ol](README.es.md)**

**Transforme seu site e sua base de conhecimento em um agente de atendimento por chat.**
O site vira a fonte de verdade; o cliente conversa por **WhatsApp, Telegram, e-mail ou widget no site**.
O agente **responde citando a fonte** (ou diz "nÃ£o sei" e passa para a equipe) e **executa serviÃ§os**
(agendar, fale conosco, segunda viaâ€¦) coletando os dados por conversa, em vez de formulÃ¡rios.

## ðŸ“– Guia de uso

Guia completo (landing + passo a passo): **https://inematds.github.io/transformsite/guia/** Â· ðŸ—ºï¸ [Roadmap](ROADMAP.md)

> InspiraÃ§Ã£o: o **America.gov** (2026) juntou o conteÃºdo de ~29 mil sites do governo americano num Ãºnico
> ponto de entrada conversacional â€” primeiro perguntas com fonte, depois transaÃ§Ãµes. O transformsite Ã© a
> versÃ£o aberta e simples dessa ideia para qualquer empresa, escola ou Ã³rgÃ£o.

## Como funciona

```
site + docs â”€â”€ingestâ”€â”€â–º Ã­ndice (BM25 + vetores) â”€â”€â–º RAG com citaÃ§Ã£o â”€â”
formulÃ¡rios legados â”€â”€convertâ”€â”€â–º services/*.yaml â”€â”€â–º mÃ¡quina de slots â”¼â”€â”€â–º Telegram Â· WhatsApp Â· e-mail Â· widget
                                         ferramentas (agenda, tickets, POST no legado, webhook) â”˜
                                         handoff humano + painel admin + mÃ©tricas
```

- **Tudo local por padrÃ£o**: LLM no [Ollama](https://ollama.com) (ou vLLM/LM Studio), embeddings `bge-m3`,
  Ã­ndice num Ãºnico arquivo SQLite. TambÃ©m funciona com **Claude Code** (`claude -p`) ou **Codex CLI**
  (`codex exec`) pela sua assinatura â€” sem chave de API.
- **ServiÃ§o = um arquivo YAML.** Sem cÃ³digo por serviÃ§o: slots com tipos validados (CPF, e-mail, telefone,
  CEP, data/hora em portuguÃªsâ€¦), confirmaÃ§Ã£o, idempotÃªncia, handoff.
- **O legado continua vivo**: o conversor gera serviÃ§os que fazem o mesmo POST que o formulÃ¡rio antigo fazia.
- **LGPD**: PII mascarada em logs, retenÃ§Ã£o configurÃ¡vel, `privacy forget`, OTP antes de dados sensÃ­veis.

## InstalaÃ§Ã£o rÃ¡pida (5 minutos)

```bash
pipx install git+https://github.com/inematds/transformsite      # ou: pip install .
ollama pull qwen3.6:35b-a3b && ollama pull bge-m3                # qualquer modelo instruct serve (ex.: qwen2.5:7b)

transformsite init meubot --org "Minha Empresa" --url https://www.minhaempresa.com.br --contact atendimento@minhaempresa.com.br
cd meubot
transformsite inventory          # Fase 0: pÃ¡ginas e formulÃ¡rios do site â†’ inventario/*.csv
transformsite ingest             # Fase 1: lÃª o site inteiro + docs/ â†’ kb/index.sqlite
transformsite chat               # conversa no terminal
transformsite serve              # widget em /chat, painel em /admin, canais configurados
```

Sem GPU? Use um modelo menor (`qwen2.5:7b`, `llama3.1:8b`) ou `llm.provider: claude_cli` / `codex_cli` (experimental: rodam sem erro, mas no piloto ainda responderam "nÃ£o sei" onde o Ollama acerta â€” em investigaÃ§Ã£o).

## Comandos

| Comando | O que faz |
|---|---|
| `init <pasta>` | cria o projeto (config, 2 serviÃ§os de exemplo, cenÃ¡rios de teste) |
| `inventory` | rastreia o site e lista pÃ¡ginas e formulÃ¡rios com prioridade sugerida |
| `ingest` | crawler (sitemap + links, respeita robots.txt) + docs locais â†’ Ã­ndice hÃ­brido; reingestÃ£o incremental por hash |
| `ask "pergunta"` / `chat` | testa no terminal |
| `golden` | gera rascunho de perguntas de teste a partir da base (para revisar) |
| `eval` | mede: citaÃ§Ã£o vÃ¡lida, "nÃ£o sei" fora de escopo, injeÃ§Ã£o bloqueada, latÃªncia; roda cenÃ¡rios de conversa |
| `validate` | valida `services/*.yaml` (schema + regras: confirmaÃ§Ã£o, idempotÃªncia, ferramentas existentes) |
| `convert form.html\|form.pdf\|URL` | formulÃ¡rio legado â†’ `service.yaml` em `review: pending` |
| `serve` | servidor: widget web, webhooks (WhatsApp/Telegram), pollers (Telegram/IMAP), painel |
| `report --since 7` | mÃ©tricas (resoluÃ§Ã£o sem humano, handoff, tempo atÃ© concluir, abandono por slot) em JSON/CSV |
| `privacy forget --user canal:id` / `privacy purge` | LGPD: apagar um usuÃ¡rio / expurgar por retenÃ§Ã£o |

## Um serviÃ§o

```yaml
service: agendar
review: approved            # serviÃ§os convertidos nascem "pending" e sÃ³ sobem apÃ³s aprovaÃ§Ã£o
intent:
  description: "Agendar um atendimento"
  examples: ["quero agendar", "marcar horÃ¡rio"]
slots:
  - {name: nome, type: text, prompt: "Qual Ã© o seu nome?"}
  - {name: email, type: email, prompt: "Qual e-mail para o convite?"}
  - name: data_hora
    type: datetime
    prompt: "Qual dia e horÃ¡rio prefere?"
    options_from: tool:agenda.slots_livres          # oferece horÃ¡rios livres como botÃµes
    constraints: {validate_tool: agenda.validar_horario}
confirm: {template: "Confirmo {nome} em {data_hora|fmt_br}?"}
action:
  tool: agenda.criar_evento
  args: {title: "Atendimento â€“ {nome}", start: "{data_hora}", email: "{email}"}
  idempotency_key: "{session_id}:{data_hora}"
on_success: {reply: "Agendado! Protocolo {result.id}."}
```

O cliente pode responder tudo de uma vez ("sou a Ana, ana@x.com, quinta Ã s 10"), corrigir ("nÃ£o, o e-mail
Ã©â€¦"), perguntar algo no meio (o bot responde com fonte e volta ao pedido), cancelar ou voltar dias depois.

Ferramentas prÃ³prias: crie `tools/minha.py` com `register(registry)` â€” veja `tools/exemplo.py`.

## Canais

| Canal | Como ativar |
|---|---|
| Widget web | ligado por padrÃ£o: `<script src="https://SEU_DOMINIO/widget.js"></script>` |
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

Sobe com Docker Compose: **app + Ollama + Evolution API (WhatsApp) + Caddy (HTTPS automÃ¡tico)**. Detalhes em
[`deploy/vps/README.md`](deploy/vps/README.md).

## Piloto: INEMA.CLUB

A pasta [`pilot/`](pilot/) Ã© o projeto real usado para validar o framework com o site
[inema.club](https://www.inema.club): 386 pÃ¡ginas â†’ 1.512 trechos; com 73 perguntas de teste e LLM local,
**92,6%** de respostas com fonte vÃ¡lida, **100%** de "nÃ£o sei" fora do escopo, **100%** de injeÃ§Ã£o
bloqueada, p95 de 2,6 s; **13/13** roteiros de conversa. Detalhes em [`pilot/RESULTADOS.md`](pilot/RESULTADOS.md).

## Desenvolvimento

```bash
uv venv && uv pip install -e '.[dev]'
pytest -q                       # sem rede, LLM falso determinÃ­stico
```

LicenÃ§a MIT Â· feito pelo [INEMA](https://www.inema.club) â€” conteÃºdo aberto e gratuito.

---

Projeto adaptado da comunidade INEMA TDS - Marca Arckhe
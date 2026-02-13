# OpenClaw - Fork do Felipe Motter

## O que e este projeto

Fork pessoal do [OpenClaw](https://github.com/openclaw/openclaw), um agente de IA conversacional self-hosted que roda via Docker. Funciona como gateway/bridge para diversos provedores de LLM, com suporte a canais como Telegram, Discord, Slack, etc.

## Git

- **origin**: `https://github.com/felipemotter/openclaw.git` (fork)
- **upstream**: `https://github.com/openclaw/openclaw.git` (projeto original)
- **Branch principal**: `main`
- **Git user**: Felipe Motter / `felipemotter@users.noreply.github.com`
- **Push para origin nao funciona via HTTPS** - precisa configurar token (PAT) ou SSH key

### Sincronizar com upstream

```bash
git fetch upstream
git rebase upstream/main
# push requer force por causa do rebase:
git push --force-with-lease origin main
```

## Docker

O projeto roda 100% via Docker. A imagem e construida localmente.

### Build da imagem

```bash
docker build --no-cache -t openclaw:local .
```

O `docker compose build` NAO funciona porque o compose nao tem diretiva `build`, so referencia a imagem `openclaw:local`. Sempre usar `docker build` direto.

### Containers

- **openclaw-gateway**: Servidor principal (portas 18789/18790)
- **openclaw-cli**: CLI interativo (network_mode: host)

### Comandos uteis

```bash
docker compose up -d          # Subir containers
docker compose down           # Parar containers
docker compose restart        # Reiniciar
docker compose exec openclaw-gateway <cmd>  # Executar dentro do container
```

### Volumes montados

- Config: `${OPENCLAW_CONFIG_DIR}` -> `/home/node/.openclaw`
- Workspace: `${OPENCLAW_WORKSPACE_DIR}` -> `/home/node/.openclaw/workspace`
- Claude config: `${CLAUDE_CONFIG_DIR:-~/.claude}` -> `/home/node/.claude`

### Variaveis de ambiente (.env)

Arquivo `.env` na raiz do projeto com:
- `OPENCLAW_GATEWAY_TOKEN` - Token do gateway
- `MOONSHOT_API_KEY` - Key da Moonshot (Kimi K2.5)
- `OPENAI_API_KEY` - Key da OpenAI
- `TELEGRAM_BOT_TOKEN` - Bot do Telegram
- `OPENCLAW_CONFIG_DIR=/home/felipe/.openclaw`
- `OPENCLAW_WORKSPACE_DIR=/home/felipe/.openclaw/workspace`
- `OPENCLAW_IMAGE=openclaw:local`

## Customizacoes no Dockerfile

O fork tem estas adicoes ao Dockerfile original:
1. **Playwright com Chromium** para automacao de browser
2. **Claude Code CLI** instalado via curl (native installer)
3. **Permissoes** ajustadas para `/opt/pw-browsers`

## Customizacoes no docker-compose.yml

- `NODE_PATH` adicionado para resolver modulos
- Variaveis extras: `MOONSHOT_API_KEY`, `OPENAI_API_KEY`
- Volume do Claude (`~/.claude`) montado
- `network_mode: host` no container CLI

## Modelos e Provedores

### Configuracao atual (openclaw.json dentro do container)

- **Primary**: `openai-codex/gpt-5.2` (via OAuth/assinatura Codex)
- **Fallback 1**: `openai-codex/gpt-5.3-codex` (via OAuth/assinatura Codex)
- **Fallback 2**: `moonshot/kimi-k2.5` (via API key)
- No picker (`/model`) tambem aparece `openai/gpt-5.2` (API key) como opcao manual

### Provedores configurados

| Provider | Auth | Uso |
|---|---|---|
| `openai-codex` | OAuth (assinatura Codex) | Principal (gpt-5.2) + fallback (gpt-5.3-codex) |
| `openai` | API key (`OPENAI_API_KEY`) | Reserva manual (gpt-5.2 via API key) |
| `moonshot` | API key (`MOONSHOT_API_KEY`) | Fallback (kimi-k2.5) |

### Allowlist do model picker (agents.defaults.models)

Para um modelo aparecer no `/model`, ele PRECISA estar listado em `agents.defaults.models`:
```json
"models": {
  "openai-codex/gpt-5.2": {},
  "openai-codex/gpt-5.3-codex": {},
  "openai/gpt-5.2": {},
  "moonshot/kimi-k2.5": {}
}
```
Se esse objeto tiver qualquer chave, o picker so mostra os modelos listados ali.

### Trocar modelo em tempo real

No chat do Telegram/CLI: `/model openai-codex/gpt-5.3-codex` ou `/model moonshot/kimi-k2.5`

### Nota sobre GPT-5.3

O GPT-5.3 so existe na versao **codex** (`gpt-5.3-codex`), disponivel apenas via `openai-codex` (OAuth) ou `github-copilot`. Nao existe `gpt-5.3` vanilla.

## Config do OpenClaw

O arquivo de config fica em `/home/node/.openclaw/openclaw.json` (dentro do container) que mapeia para `/home/felipe/.openclaw/openclaw.json` (host - acesso pode requerer sudo).

Para editar via container:
```bash
docker compose exec openclaw-gateway cat /home/node/.openclaw/openclaw.json
docker compose exec openclaw-gateway node -e "
const fs = require('fs');
const cfg = JSON.parse(fs.readFileSync('/home/node/.openclaw/openclaw.json','utf8'));
// ... modificar cfg ...
fs.writeFileSync('/home/node/.openclaw/openclaw.json', JSON.stringify(cfg, null, 2));
"
```

## Plugins ativos

- **Telegram** (habilitado)

## Ferramentas

- **Web search**: Brave Search (habilitado, max 10 resultados)
- **Browser**: Playwright + Chromium (instalado no Docker)

## Estrutura do projeto

- `src/` - Codigo fonte TypeScript
- `dist/` - Build compilado (gerado no docker build)
- `ui/` - Interface web (Control UI, built com Vite)
- `docs/` - Documentacao
- `scripts/` - Scripts de build/setup
- `Dockerfile` - Build da imagem (customizado)
- `docker-compose.yml` - Orquestracao (customizado)
- `.env` - Variaveis de ambiente (NAO commitar - contem secrets)

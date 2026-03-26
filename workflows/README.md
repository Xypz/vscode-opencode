# 📡 Reddit Bitcoin Radar

Workflow de n8n que monitorea posts virales de r/Bitcoin y r/btc, los resume con IA y envía el resultado a Telegram cada día a las 22:00 (Europe/Madrid).

## 🏗️ Arquitectura

```
cron-job.org (22:00 diario)
    ↓ GET https://n8n.hellonode.es/webhook/bitcoin-radar
Webhook Trigger
    ↓ (paralelo)
Reddit API - btc          Reddit API - Bitcoin
(top 5, t=day)            (top 5, t=day)
+ User-Agent header       + User-Agent header
    ↓                         ↓
Parse btc                 Parse Bitcoin
    ↓                         ↓
Aggregate btc             Aggregate Bitcoin
    ↓                         ↓
         Merge (inputs 0 y 1)
              ↓
         Select Top 5 (ordena por upvotes)
              ↓
         Prepare Prompt
              ↓
         AI Agent (DeepSeek via OpenRouter)
              ↓
         Format Message
              ↓
         Telegram (chat personal)
```

## 🚀 Instalación

### 1. Importar el Workflow

Importar `reddit-bitcoin-radar.json` via API o UI de n8n.

### 2. Credenciales necesarias

| Nodo | Credencial | Tipo |
|------|-----------|------|
| Telegram | Bot Token personal | `telegramApi` |
| OpenRouter Chat Model | API Key | `openRouterApi` |

### 3. Infraestructura requerida

- **n8n** corriendo en OrbStack: `https://n8n_tunnel.orb.local`
- **Cloudflare Tunnel** activo: `https://n8n.hellonode.es` (LaunchAgent en `~/Library/LaunchAgents/`)
- **cron-job.org** configurado: GET `https://n8n.hellonode.es/webhook/bitcoin-radar` a las 22:00 Europe/Madrid

### 4. Variables de entorno (`.env`)

```
N8N_HOST=https://n8n.hellonode.es
N8N_API_KEY=<tu_api_key>
```

## ⚙️ Personalización

### Cambiar horario
Modificar el cronjob en cron-job.org (el workflow ya no usa Schedule Trigger interno).

### Cambiar subreddit
Editar los nodos `Reddit API - btc` / `Reddit API - Bitcoin`:
```
https://www.reddit.com/r/{subreddit}/top.json?limit=5&t=day
```

### Cambiar modelo de IA
Editar el nodo `OpenRouter Chat Model` → campo `model`.
Modelo actual: `deepseek/deepseek-r1-distill-llama-70b`

## 🔒 Seguridad

- ✅ CERO API keys en código
- ✅ Credentials nativas de n8n
- ✅ JavaScript defensivo (optional chaining)
- ✅ User-Agent en requests a Reddit (evita 403/429)

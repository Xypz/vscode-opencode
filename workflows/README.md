# 📡 Reddit Bitcoin Radar

Workflow de n8n que monitorea posts virales de r/Bitcoin y envía un resumen diario a Telegram.

## 🚀 Instalación

### 1. Importar el Workflow

1. Abre tu instancia de n8n
2. Ve a **Workflows** → **Import from File**
3. Selecciona el archivo `reddit-bitcoin-radar.json`

### 2. Configurar Credenciales (RULES.md: CERO secretos hardcodeados)

#### Reddit API
- **No requiere autenticación** - usa la API pública `.json`
- El workflow ya está configurado

#### Telegram Bot
1. Chatea con @BotFather en Telegram
2. Envía `/newbot` y sigue las instrucciones
3. Copia el **Bot Token**
4. En n8n: **Credentials** → **New** → **Telegram Bot API**
5. Introduce el token

#### Google Gemini
1. Ve a https://ai.google.dev/
2. Crea una API key (tiene tier gratuito)
3. En n8n: **Credentials** → **New** → **Google Gemini**
4. Introduce tu API key

### 3. Configurar Chat ID de Telegram

1. Chatea con tu bot en Telegram
2. Envía cualquier mensaje
3. Ve a https://api.telegram.org/bot<TOKEN>/getUpdates
4. Copia el **chat.id** de la respuesta

### 4. Actualizar el Workflow

1. Haz doble clic en el nodo **Telegram**
2. En **Chat ID**, introduce tu chat ID (o usa una credential)

## ⚙️ Personalización

### Cambiar Subreddit
Edita el nodo **Reddit API**:
```
URL: https://www.reddit.com/r/{subreddit}/top.json?limit=10&t=day
```

### Cambiar Horario
Edita el nodo **Schedule Trigger**:
- Interval: cada 24 horas
- Start time: 08:00 (configurable)

### Ajustar Filtros
Edita el nodo **Parse Reddit Data**:
- Cambia el límite de posts
- Modifica el timeframe (`hour`, `day`, `week`, `month`, `year`, `all`)

## 📋 Nodos del Workflow

```
Schedule Trigger (08:00 daily)
    ↓
Reddit API (HTTP Request)
    ↓
Parse Reddit Data (Code - JavaScript defensivo)
    ↓
AI Summarize (Google Gemini)
    ↓
Telegram (Enviar mensaje)
```

## 🔒 Seguridad (RULES.md)

- ✅ CERO API keys en código
- ✅ Credentials nativas de n8n
- ✅ JavaScript defensivo con try/catch
- ✅ Optional chaining para datos opcionales

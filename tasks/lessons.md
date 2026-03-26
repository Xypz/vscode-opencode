# 📚 Lecciones Aprendidas

## Historial de Correcciones

---

### 2026-03-15

**Proyecto:** Reddit Bitcoin Radar

**Problema:** No se pudo obtener el JSON del template directamente desde n8n.io (API no disponible para descarga directa)

**Solución:** Crear el workflow manualmente basándose en la estructura del template #10246 ("Reddit digests + Gemini") documentada en la página web

**Regla creada:**
- Para proyectos n8n: si no hay JSON disponible, crear manualmente usando la descripción del template
- Siempre documentar estructura de nodos para referencia futura

---

### 2026-03-15 (API n8n)

**Proyecto:** Conexión a n8n local

**Problema:** Error "Unauthorized" al intentar autenticarse con la API de n8n

**Solución:** Usar:
- Endpoint: `/api/v1/workflows`
- Header: `X-N8N-API-KEY`

**Regla creada:**
- Para n8n 1.x el prefijo es `/api/v1/` no `/rest/`

---

### 2026-03-15 (Import workflow)

**Proyecto:** Reddit Bitcoin Radar - Import a n8n local

**Problema:** Error "request/body must NOT have additional properties"

**Solución:**
- Filtrar settings a solo: `executionOrder`, `timezone`, `callerPolicy` (y similares documentados)
- Eliminar: `binaryMode`, `availableInMCP`, `timeSavedMode` — n8n los rechaza en POST/PUT aunque los guarde internamente
- Campos prohibidos en el payload raíz: `id`, `versionId`, `tags`, `meta`, `active`

**Regla creada:**
- Payload mínimo para POST/PUT: `{name, nodes, connections, settings:{executionOrder:"v1"}}`

---

### 2026-03-15 (IDs de nodos)

**Problema:** El workflow se creaba en la API pero no se mostraba en la UI de n8n

**Causa raíz:** IDs de nodos con guiones (`schedule-trigger`) — n8n no los acepta

**Solución:** Usar UUIDs o IDs simples sin guiones: `n1`, `n2`, `n3`

---

### 2026-03-16 (Merge de dos APIs en paralelo)

**Proyecto:** Reddit Bitcoin Radar

**Problema:** Nodo Select Top 5 recibía `{data: [...]}` del Aggregate en lugar de un array directo

**Causa raíz:** `aggregateAllItemData` envuelve los items en `{ data: [...] }`. El código asumía que `item.json` era un array.

**Solución:**
```javascript
const entries = item.json?.data ?? (Array.isArray(item.json) ? item.json : [item.json]);
```

**Regla creada:**
- Tras nodo Aggregate con `aggregateAllItemData`, acceder a los datos via `item.json.data`, no `item.json` directamente

---

### 2026-03-16 (Merge inputs)

**Problema:** Merge recibía datos incorrectamente de dos ramas paralelas

**Solución:**
- Conectar cada rama a un input diferente del Merge (index: 0, index: 1)
- Usar Item Lists con `operation: "aggregateItems"` antes del Merge

---

### 2026-03-22 (Schedule Trigger vs macOS sleep)

**Proyecto:** Reddit Bitcoin Radar

**Problema:** El Schedule Trigger no disparaba — n8n (OrbStack) se pausa cuando el Mac duerme y no recupera ejecuciones perdidas

**Solución:** Reemplazar Schedule Trigger por Webhook + cron-job.org externo
- El servicio externo llama al webhook aunque el Mac esté dormido momentáneamente
- URL: `https://n8n.hellonode.es/webhook/bitcoin-radar`

**Regla creada:**
- En entornos locales (OrbStack/Docker en Mac): usar siempre Webhook + cron externo en lugar de Schedule Trigger

---

### 2026-03-22 (Cloudflare Tunnel LaunchDaemon vs LaunchAgent)

**Problema:** `sudo cloudflared service install` instala un LaunchDaemon (corre como root). El daemon no encuentra los credentials en `~/.cloudflared/` del usuario → túnel inactivo al reiniciar

**Solución:** Instalar como LaunchAgent de usuario:
```bash
# Crear plist en ~/Library/LaunchAgents/com.cloudflare.cloudflared.plist
# con ProgramArguments: cloudflared tunnel run n8n
launchctl load ~/Library/LaunchAgents/com.cloudflare.cloudflared.plist
```

**Regla creada:**
- Cloudflared en Mac: siempre LaunchAgent (usuario), nunca LaunchDaemon (sistema)

---

### 2026-03-22 (Corrupción SQLite WAL)

**Problema:** Tras crear nuevo contenedor Docker, n8n devolvía `SQLITE_NOTADB` en todas las operaciones

**Causa raíz:** El nuevo contenedor generó un WAL (`database.sqlite-wal`) incompatible con el `database.sqlite` existente

**Solución:** Parar contenedor → eliminar `.sqlite-wal` y `.sqlite-shm` → reiniciar
```bash
docker stop n8n_tunnel
rm ~/.n8n/database.sqlite-wal ~/.n8n/database.sqlite-shm
docker start n8n_tunnel
```

**Regla creada:**
- Antes de crear un nuevo contenedor que monte `~/.n8n`, asegurarse de que el anterior está parado limpiamente (checkpoint del WAL)
- Siempre hacer backup: `cp ~/.n8n/database.sqlite ~/.n8n/database.sqlite.bak`

---

### 2026-03-22 (Reddit API User-Agent)

**Problema:** Reddit devuelve 403/429 en requests sin User-Agent

**Solución:** Añadir header en nodos HTTP Request:
```
User-Agent: n8n-bitcoin-radar/1.0
```

**Regla creada:**
- Siempre añadir User-Agent en requests a Reddit (y a APIs que lo requieran)

---

### 2026-03-26 (AI Agent no pasa campos originales)

**Proyecto:** Panfleto

**Problema:** `sector`, `url` e `image` aparecían como `undefined` en el HTML final

**Causa raíz:** El nodo `AI Agent` solo emite el campo `output` (la respuesta de la IA). Descarta todos los campos del item de entrada (`sector`, `url`, `image`, etc.).

**Solución:** En el nodo siguiente al AI Agent, recuperar los datos originales del nodo previo con:
```javascript
const aiItems = $input.all();
const originalItems = $('NombreNodoPrevio').all();
// luego: const { sector, url } = originalItems[i]?.json || {};
```

**Regla creada:**
- Cualquier nodo que siga a un AI Agent SIEMPRE debe recuperar los datos originales con `$('NodoAnterior').all()` — nunca asumir que los campos se pasan automáticamente.

---

### 2026-03-26 (DeepSeek mezcla idiomas)

**Proyecto:** Panfleto

**Problema:** DeepSeek-R1 generaba bullets en chino mezclados con español

**Causa raíz:** El system prompt solo decía "traduce al español" pero no prohibía explícitamente otros idiomas. Con texto fuente en inglés que contenía fragmentos en chino, el modelo los reproducía sin traducir.

**Solución:** System prompt con reglas explícitas:
- "Tu única lengua de salida es el ESPAÑOL. NUNCA escribas en chino, inglés, ni ningún otro idioma."
- "Si alguna frase está en chino/japonés/árabe, TRADÚCELA antes de usarla."
- "PROHIBIDO mezclar idiomas."

**Regla creada:**
- Con modelos multilingüe (DeepSeek, etc.): especificar el idioma de salida de forma prohibitiva, no solo afirmativa.

---

### 2026-03-26 (OAuth Gmail en n8n local)

**Proyecto:** Panfleto

**Problema:** "Unauthorized" al intentar conectar credencial Gmail OAuth2 en n8n

**Causa raíz (cadena completa):**
1. Acceder via `https://n8n.hellonode.es` → Cloudflare WAF bloquea el endpoint OAuth
2. Acceder via `http://localhost:5678` → cookie segura bloqueada (n8n tiene `N8N_SECURE_COOKIE=true` por defecto)
3. Con `N8N_SECURE_COOKIE=false` via localhost → el frontend llama al callback en `https://n8n.hellonode.es` (mismatch de origen), la sesión no es válida al retornar de Google

**Solución definitiva:** Arrancar Docker con:
```bash
-e N8N_SECURE_COOKIE=false
-e N8N_EDITOR_BASE_URL=http://localhost:5678
```
Y añadir `http://localhost:5678/rest/oauth2-credential/callback` en Google Cloud Console → OAuth 2.0 Client → Authorized redirect URIs.

**Regla creada:**
- Para OAuth en n8n local: `N8N_EDITOR_BASE_URL` debe coincidir con la URL desde la que accedes a n8n. Sin esto, el callback de Google y la sesión quedan en dominios distintos.
- El comando completo de Docker está documentado en CLAUDE.md.

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

### 2026-03-15 (Actual)

**Proyecto:** Conexión a n8n local

**Problema:** Error "Unauthorized" al intentar autenticarse con la API de n8n

**Investigación:** 
- Probé múltiples headers: `X-N8N-API-KEY`, `Authorization: Bearer`, `api-key`
- El endpoint correcto es `/api/v1/workflows` (no `/rest/workflows`)

**Solución:** Usar:
- Endpoint: `/api/v1/workflows`
- Header: `X-N8N-API-KEY`

**Regla creada:** 
- Verificar el endpoint correcto de API (puede variar según versión de n8n)
- Para n8n 1.x el prefijo es `/api/v1/` no `/rest/`

---

### 2026-03-15 (Import workflow)

**Proyecto:** Reddit Bitcoin Radar - Import a n8n local

**Problema:** 
- Error "request/body must NOT have additional properties" - propiedades no reconocidas
- Error "request/body must have required property 'settings'" - faltaba settings

**Solución:** 
- Simplificar el JSON del workflow
- Incluir siempre el objeto "settings": {}
- Para n8n 1.x, los campos como "triggerCount", "versionId", "updatedAt" causan errores

**Regla creada:**
- Al crear workflows via API: minimal JSON con settings vacío
- Luego el usuario configura nodos AI y credenciales desde la UI

---

### 2026-03-15 (Error en JSON de workflow)

**Problema:** El workflow se creaba en la API pero no se mostraba en la UI de n8n

**Causa raíz:** 
- Los IDs de los nodos contenían guiones (`schedule-trigger`, `http-reddit`, etc.) - n8n no los acepta así
- Faltaban campos requeridos en `settings`: `executionOrder` y `availableInMCP`

**Solución:**
- Usar IDs simples sin guiones: `n1`, `n2`, `n3`, `n4`
- Añadir settings correctos: `{"executionOrder": "v1", "availableInMCP": false}`

**JSON correcto:**
```json
{
  "name": "Reddit Bitcoin Radar",
  "nodes": [
    {"id": "n1", "name": "Schedule Trigger", ...},
    {"id": "n2", "name": "Reddit API", ...},
    {"id": "n3", "name": "Parse Reddit Data", ...},
    {"id": "n4", "name": "Telegram", ...}
  ],
  "connections": {...},
  "settings": {"executionOrder": "v1", "availableInMCP": false}
}
```

**Regla creada:**
- Usar IDs de nodos simples (n1, n2, n3...) sin guiones bajos
- Incluir siempre settings con executionOrder y availableInMCP

---

### YYYY-MM-DD

**Problema:** _Describe el error_

**Solución:** _Cómo se resolvió_

**Regla creada:** _Prevención para el futuro_

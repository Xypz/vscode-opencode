# 📋 Reddit Bitcoin Radar

## ✅ Estado: Activo en producción

---

## 🎯 Tareas Completadas

- [x] Investigación y creación del workflow base
- [x] Integración de dos subreddits (r/btc + r/Bitcoin) en paralelo con Merge
- [x] Fix bug en nodo `Select Top 5` (Aggregate genera `{data:[...]}`, no array directo)
- [x] Añadido User-Agent en HTTP requests a Reddit (evita bloqueos 403/429)
- [x] Reemplazado Schedule Trigger → Webhook (fiabilidad con macOS durmiendo)
- [x] Configurado Cloudflare Tunnel (`n8n.hellonode.es`) como dominio permanente
- [x] Configurado cron-job.org para disparar el webhook a las 22:00 Europe/Madrid
- [x] Importado workflow `shixin-technology` recuperado

---

## 📁 Archivos

```
workflows/
├── reddit-bitcoin-radar.json       # Workflow activo (con todos los fixes)
├── workflow-produccion-final.json  # shixin-technology (importado en n8n)
└── README.md                       # Documentación actualizada
```

---

## 🔑 Credenciales en n8n

| Servicio | Tipo | Estado |
|----------|------|--------|
| Telegram | telegramApi | ✅ Configurada |
| OpenRouter | openRouterApi | ✅ Configurada |

---

## 🌐 Infraestructura

| Componente | Detalle |
|---|---|
| n8n URL pública | https://n8n.hellonode.es |
| n8n URL local | https://n8n_tunnel.orb.local |
| Cloudflare Tunnel | `n8n` (LaunchAgent, arranca con sesión) |
| Cron externo | cron-job.org → 22:00 Europe/Madrid |
| Workflow ID (n8n) | 6vfSuqjJrRo3egfX |

---

## ⚠️ Pendiente

- [ ] Asignar credenciales en `shixin-technology` (Telegram + OpenRouter)

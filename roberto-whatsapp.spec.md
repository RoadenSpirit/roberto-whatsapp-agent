# SPEC — Rescate `ROBERTO: WhatsApp` (Q5iI4KQ5PzWL3Bax), v1 conservadora

> Estado: PROPUESTA para aprobación explícita. Decisiones punto por punto 2026-10-15.
> Sin "Aprobado / Procede" no se toca n8n.

## 1. Arquitectura final (solo WhatsApp, sin roles, con SSH)
```
Baileys (:3000) ──POST──▶ Webhook `roberto-whatsapp-webhook`
  ▶ Code Anti-ruido: si remoteJid = status@broadcast O message vacío → FIN silencioso
  ▶ IF Permitidos: chatId en lista de 2 números (jids + startsWith) → true: sigue / false: FIN
  ▶ SSH `whatsapp_bot_handler.py` (SIN CAMBIOS)
  ▶ Code Limpiar Respuesta WhatsApp (SIN CAMBIOS)
  ▶ HTTP POST `http://baileys-sales-api:3000/send` (solo cambia host; misma key)
```

## 2. Cambios exactos (3 operaciones sobre el workflow + 1 credencial reserva)
1. **ADD Code `Filtro Anti-ruido`** entre Webhook y el IF actual. JS: lee
   `body.raw.key.remoteJid` y `body.message`; si `remoteJid==='status@broadcast'` o
   `!message.trim() && !imagePath` → no emite items (termina). Si no, pasa 1 item.
2. **KEEP IF `Filtro de Seguridad`** tal cual (2 números). Solo se re-ubica tras el anti-ruido.
3. **PATCH HTTP `Enviar Mensaje WhatsApp`**: `url`: `http://YOUR_WHATSAPP_HOST/send` →
   `http://baileys-sales-api:3000/send`. Header `x-api-key`: sin cambios.
4. **Key FreeLLM dedicada** `ROBERTO-fase2` generada en FreeLLMAPI (:3005) y guardada como
   credencial `openAiApi` "FreeLLM ROBERTO (reserva)" — SIN USO en v1.

## 3. Criterios de aceptación
- CA1: mensaje de número permitido con texto → respuesta por /send en < 90 s (límite SSH actual).
- CA2: estado (`status@broadcast`) → ejecución termina SIN nodos SSH/HTTP ejecutados.
- CA3: mensaje vacío sin imagen → igual que CA2.
- CA4: número no permitido → termina en el IF, sin SSH ni envío.
- CA5: `n8n_validate_workflow` sin errores tras el cambio.
- CA6: repo con `git init` + commit docs (`requisitos.md`, esta spec, snapshots).

## 4. Impacto VPS
- +0 procesos, +0 RAM: 1 nodo Code extra dentro de `n8n-app`; SSH/handler igual que hoy.
- Llamadas LLM: las mismas que hoy (vía opencode CLI/antigravity-proxy). El anti-ruido las REDUCE
  (estados ya no llegan al CLI).
- Rollback: snapshot local del workflow antes de operar.

## 5. Diferido explícito (no en v1)
Rotación key Baileys, Agente n8n + FreeLLM en flujo, roles, OmniRoute. La key de reserva queda
lista para retomar la fase 2 cuando lo pidas (credencial n8n "FreeLLM ROBERTO (reserva)",
id PinvgLcB0x4Slx4o, creada 2026-10-16).

## 6. Resultados de certificación (2026-10-16)
- CA2 ✓ ejecución 82488: `status@broadcast` → Anti-ruido 0 items, fin en 30 ms, sin SSH ni envío.
- CA3 ✓ (misma rama que CA2: mensaje vacío sin imagen → 0 items).
- CA4 ✓ ejecución 82490: número desconocido → Anti-ruido pasa, IF rama false, fin sin SSH ni envío.
- CA5 ✓ `n8n_validate_workflow`: 0 errores, 0 warnings (6 nodos, 5 conexiones válidas).
- CA1 pendiente: requiere mensaje real desde número permitido.
- CA6 en curso: git init + commit.

# PRD — ROBERTO: Agente IA personalizada por WhatsApp (rescate)

> Estado: BORRADOR para aprobación. Solo alcance WhatsApp (Telegram queda fuera).

## 1. Objetivo de negocio
Rescatar el asistente interno "ROBERTO" para que empleados de Your Brand consulten
por WhatsApp inventario, precios/stock, ventas diarias y conocimiento general,
con respuestas rápidas, con memoria de conversación y sin exponer el servidor.

## 2. Usuarios
- **Números permitidos (2)**: `YOUR_PHONE_1`, `YOUR_PHONE_2` (y sus variantes `@s.whatsapp.net` / `@lid`).
  Sin distinción de roles: todo número permitido recibe el mismo tratamiento.
  Funcionalidad de roles Admin/Empleados: DEPRECADA (decisión 2026-10-15).
- **No permitidos / ruido**: estados (`status@broadcast`), mensajes vacíos, chatIds desconocidos
  → se descartan en silencio (sin respuesta, sin llamar al motor IA).

## 3. Entradas / Salidas
- **Entradas**: webhook POST de Baileys API con `{chatId, senderName, message, imagePath?}`.
- **Salidas**: mensaje de texto formateado para WhatsApp vía `POST /send` de Baileys.
- **Prohibido**: HTML, Markdown complejo, rutas locales (`file:///`), credenciales en texto.

## 4. Reglas comerciales / técnicas
1. Toda pregunta de datos (stock, precio, ventas) DEBE pasar por herramientas, nunca inventar.
2. Precios siempre en CUP. Fechas: resolver "hoy/ayer" contra fecha actual del sistema.
3. Memoria de conversación aislada por `chatId` (tabla `historial_chat_bot`, `session_id=chatId`).
4. Cero Trust: el flujo de Empleados jamás atraviesa nodos con credenciales SSH.
5. Sin secretos hardcodeados en nodos; todo por credenciales n8n + variables.
6. Formato WhatsApp: `*negrita*`, `_cursiva_`, `` `mono` ``, listas `- item`. Sanitizar HTML.

## 5. Alcance explícito (v1 rescatada — decisiones 2026-10-15)
- IN:
  1. Filtro anti-ruido (Code): `status@broadcast` y mensajes vacíos terminan en silencio.
  2. Filtro de permitidos: solo los 2 números, sin ramas por rol.
  3. Se mantiene SSH + `whatsapp_bot_handler.py` (opencode CLI) como motor de respuesta;
     comandos `/proyectos`, `/set`, imágenes (`imagePath`) siguen como están.
  4. URL de envío → `http://baileys-sales-api:3000` (misma API key actual; rotación pendiente).
  5. Key dedicada FreeLLMAPI para ROBERTO generada y guardada (fase 2, sin uso en v1).
  6. `git init` + commit de docs en el repo.
- OUT (deprecado/diferido): roles Admin/Empleados, Agente n8n, modelo FreeLLM en el flujo,
  OmniRoute, rotación de key Baileys (pendiente), Telegram, colas (Worker/Ingesta intactos).

# ROBERTO — Asistente IA Multicanal

Asistente virtual de operaciones para atención y consultas por WhatsApp y Telegram conectado a modelos LLM, bases de datos y herramientas de negocio.

## Estructura del Proyecto

- `workflow_Q5iI4KQ5PzWL3Bax_roberto_whatsapp.json`: Flujo de integración bidireccional con WhatsApp.
- `workflow_ypgDcdzMJNfiF92L_roberto.json`: Flujo para Telegram Bot.
- `workflow_Da1VzjJddyBGAcWr_asistente_yourbrand_-_telegram_bot_sheets_lineal.json`: Integración con Google Sheets.
- `docker-compose.yml`: Configuración portable para levantar n8n con los flujos.
- `.env.example`: Variables de entorno requeridas.

## Despliegue Portable (Docker)

1. Copiar variables de entorno:
   ```bash
   cp .env.example .env
   ```
2. Completar las claves en `.env`.
3. Iniciar el servicio con Docker:
   ```bash
   docker compose up -d
   ```

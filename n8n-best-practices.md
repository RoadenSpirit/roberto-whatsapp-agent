# n8n Best Practices & Node Configuration Guidelines (ROberto Skill)

This guide defines the standards, security policies, and configuration conventions for building robust, secure, and maintainable workflows in n8n for the **ROberto** bot project.

---

## 1. Zero-Trust Security & Input Validation
*   **Early Filtering:** Every webhook or trigger must be immediately followed by a security check (`If` or `Switch` node) to validate headers, API keys, and sender identity (`chatId`).
*   **Role Separation:** Keep Admin flows and Employee flows completely separated. Employees must never traverse nodes that contain SSH credentials or access to the server command line.
*   **Credential Management:** Never hardcode API keys, passwords, or secret tokens inside nodes or expressions. Always use n8n's native credentials manager.

---

## 2. Agent & LLM Node Configurations (LangChain)
When configuring an AI Agent in n8n:

### A. The AI Agent Node
*   **Type:** Tool Agent (allows the agent to reason and call tools iteratively).
*   **System Message:** Keep it concise but strict. It must specify:
    *   **Role:** internal assistant for Your Brand employees.
    *   **Tone:** professional, direct, corporate (not salesy).
    *   **Rules:** Must use connected tools (do not invent data).
    *   **Boundaries:** No server access. Only inventory, sales, and general knowledge.

### B. The Chat Model Node (OpenAI Compatible)
*   **Base URL:** Point to the local Omnirouter endpoint `http://localhost:20128/v1`.
*   **Model Name:** `gemini/gemini-3.5-flash` (or current active model).
*   **API Key:** Fictional key (e.g. `sk-dummy`).

### C. Chat Memory Node (Postgres Chat Memory)
*   **Connection:** Use the established PostgreSQL credentials (`YOUR_POSTGRES_CREDENTIAL`).
*   **Table Name:** `historial_chat_bot` (schema must have `session_id`, `message`, etc.).
*   **Session Key:** Must be dynamic and set to `={{ $json.body.chatId }}` (where `$json.body.chatId` corresponds to the unique WhatsApp number). This ensures strict conversation isolation.

---

## 3. Data Flow & Text Processing
*   **Markdown to WhatsApp Format:** WhatsApp uses a unique subset of Markdown formatting. Standard HTML or complex Markdown will render poorly. Use a `Code` node before outputting to WhatsApp to format text:
    *   Bold: `*text*`
    *   Italic: `_text_`
    *   Monospace: ````text````
    *   Strikethrough: `~text~`
    *   Bulleted lists: `- Item` (with a space)
*   **Output Sanitization:** Clean any trailing backslashes, escape sequences, or HTML tags (like `<p>`, `<li>`, `<ul>`) before sending payload to the Baileys API.

---

## 4. Sub-Workflows as Tools
*   To expose operations (like searching inventory or sales) to the AI Agent:
    *   Use the **Tool: Workflow** node.
    *   Define a clear name and parameter schema (e.g., `buscar_inventario`, requiring `query`).
    *   Provide a descriptive explanation so the LLM understands when and how to invoke the tool.

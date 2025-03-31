# 📜 Documentación del Workflow: Knave Game Flow (n8n-v2)

Este flujo de trabajo en n8n orquesta la lógica de juego para el sistema de rol Knave, incluyendo manejo de personajes, combate, memoria conversacional e integración con GPT-4.

---

## 🧹 Descripción General

- El flujo se activa a partir de un **Telegram Trigger**, que capta los mensajes del usuario.
- El mensaje es procesado por una **IA GPT-4** con un prompt estructurado que devuelve una intención en JSON.
- Según la intención (e.g., `crear_personaje`, `battle`, `view_character`), el flujo se bifurca mediante nodos Switch.
- Se utilizan **Airtable** y **Simple Memory** para gestionar sesiones, personajes y memoria conversacional.
- Un **microservicio en Node.js** maneja la lógica de combate, con endpoints `/battle/start` y `/battle/action`.

---

## 🔄 Flujo General

1. **Telegram Trigger**  
   Recibe los mensajes entrantes del usuario.

2. **Get Chat Context (Airtable)**  
   Busca en Airtable el historial de conversación para el `telegramId`.

3. **Send to GPT (Main Agent)**  
   Le pasa al modelo de GPT-4 el mensaje actual y el contexto del chat, esperando una respuesta en JSON.

4. **Parse GPT JSON Output**  
   Se limpia y parsea la respuesta del modelo.

5. **Update Chat Context (Airtable)**  
   Se actualiza el campo `chat` del usuario en Airtable con el nuevo turno conversacional.

6. **Switch de Acción**  
   Evalúa la intención de la IA (`action`) y deriva a la rama correspondiente:
   - `create_character`
   - `select_character`
   - `view_character`
   - `delete_character`
   - `list_characters`
   - `battle`
   - `battle_action`
   - `stop_battle`
   - `unknown`

---

## ⚔️ Combate

- Si la acción es `battle`, se prepara una sesión de combate con datos del personaje y un monstruo.
- Se llama al microservicio con `POST /battle/start`.
- Para turnos siguientes se usa `POST /battle/action`.

Cada respuesta del microservicio incluye:
- Logs del turno
- HP actual de jugador y enemigo
- Resultado: `victory`, `defeat`, o `ongoing`

Este resultado se narra con un segundo GPT (narrador épico) que transforma el log en una historia.

---

## 🧠 Memoria

- Se usa `Simple Memory` para rastrear el estado de la conversación a corto plazo.
- Se conserva el historial en Airtable solo como backup y para inspección.

---

## 📃 Airtable

### Tablas principales:

- **characters**  
  Personajes del usuario (`nombre`, `atributos`, `telegramId`)

- **sessions**  
  Control de sesión de combate (`status`, `characterName`, `sessionId`)

- **chats**  
  Historial de conversación (`telegramId`, `chat`)

---

## 📡 Microservicio Node.js

Endpoints utilizados:
- `POST /battle/start`: crea nueva batalla
- `POST /battle/action`: ejecuta acción de combate
- Lógica de `runBattleTurn()` decide el flujo de turno y actualiza estados internos.

---

## 🎭 GPT Prompts

- **Main Agent**: detecta intención del jugador y devuelve JSON con `action`.
- **Narrador Épico**: toma logs del turno y los transforma en narrativa con emoción, tensión y estilo.

---

## 🧪 Validaciones y Control

- Si el personaje ya existe, se bloquea su creación.
- Si no se encuentra el personaje para ver o seleccionar, se devuelve un mensaje claro.
- Al morir un personaje, se marca como no jugable.
- Si el jugador se rinde, se ejecuta `stop_battle`.

---

## ✅ Estado Actual

Este flujo permite:
- Crear, ver, eliminar y seleccionar personajes
- Iniciar y gestionar combates dinámicos
- Ofrecer una experiencia narrativa inmersiva
- Controlar el estado del juego por sesión

---

## 📌 Posibles Mejoras Futuras

- Integración con memoria vectorial para historias largas
- Exploración entre combates
- Inventario y loot dinámico
- Sistema de logros y reputación

---


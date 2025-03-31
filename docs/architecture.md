# Arquitectura del Proyecto - Knave Game Chatbot (n8n + GPT + Microservicios)

Este documento describe la arquitectura y flujo funcional del sistema creado para la implementación de un juego de rol basado en *Knave* utilizando herramientas de automatización (n8n), microservicios en Node.js, y agentes conversacionales con GPT.

---

## Tecnologías Utilizadas

- **n8n**: Motor de workflows para automatizar conversaciones y conectar servicios.
- **OpenAI GPT-4**: Agente de lenguaje para interpretar intenciones del jugador y narrar eventos del juego.
- **Telegram Bot**: Medio de comunicación principal con el jugador.
- **Node.js Express App**: Microservicio para manejar el sistema de combate.
- **Airtable**: Base de datos usada como almacenamiento temporal de personajes, sesiones y logs.

---

## Objetivo General

Diseñar una experiencia conversacional interactiva que permita jugar *Knave* a través de Telegram, gestionando creación de personajes, sesiones de combate, narrativa emergente y decisiones de juego mediante un bot conversacional.

---

## Estructura General

### 1. Entrada del Usuario (Telegram Trigger)
- El bot recibe mensajes de texto desde Telegram.
- Se extrae el `telegramId` y el mensaje para continuar el flujo.

### 2. Recuperación de Conversación
- Se busca en Airtable si existe una conversación previa activa para ese usuario.
- Si no existe, se crea un nuevo registro con estado inicial.

### 3. Interpretación de Intenciones (AI Agent)
- Se llama a un **AI Agent (GPT)** con el historial de chat para interpretar intenciones.
- Prompt personalizado con contexto, incluyendo personajes disponibles, estatus del combate, y el último mensaje del usuario.
- El bot responde con una acción como:
  - `create_character`
  - `select_character`
  - `battle`
  - `view_character`
  - `delete_character`
  - `stop_battle`

### 4. Manejo de Acciones (Switch)
- Se enruta la acción a una rama del flujo según el tipo (`create`, `view`, `battle`, etc).

---

## ⚔️ Sistema de Combate (Microservicio)

### Endpoint `/battle/start`
- Inicia una batalla con los datos del personaje y el monstruo seleccionado.
- Se guarda la sesión en un `Map` usando el `telegramId` como clave.
- Responde con el primer estado de batalla y opciones disponibles.

### Endpoint `/battle/action`
- Recibe una acción (`attack`, `defend`, `hide`).
- Ejecuta la lógica de combate:
  - Tiradas de dado.
  - Resolución de daño.
  - Verificación de victoria o derrota.
- Devuelve un resumen estructurado del turno (log, HP restantes, estado).

---

## Narrativa Dinámica con GPT

- Tras recibir el resultado del turno de combate, se pasa el `log[]` del turno a un agente GPT adicional.
- Este transforma el resumen en una narrativa épica.
- Si el `result` es `defeat`, incluye un mensaje solemne y advierte que el personaje ha muerto.
- Si `ongoing`, agrega:
  - ❤️ Jugador: X HP | 🧟 Monstruo: Y HP
  - 🎯 ¿Qué harás ahora? Opciones: atacar, defender, esconderse

---

## Manejo de Estado (Memoria vs Airtable)

- Inicialmente, Airtable guarda la sesión, historial y estado del combate.
- Posteriormente se integra `Simple Memory` en n8n para persistir contexto de forma más eficiente sin I/O.
- Se mantiene Airtable como backup o para trazabilidad.

---

## Características Adicionales

- Validación de nombres duplicados al crear personajes.
- Validación de existencia de personaje antes de eliminar.
- Switch de flujo para detectar `stop_battle` y eliminar sesión actual.
- Personalización narrativa para cada respuesta con tono medieval.

---

## Flujo de Datos Simplificado

1. Telegram -> n8n Trigger  
2. Buscar/construir contexto (Airtable / Memory)  
3. GPT interpreta intención -> devuelve acción JSON  
4. Switch de acción maneja:  
   - Crear/ver/eliminar personaje  
   - Iniciar combate (llama `/battle/start`)  
   - Continuar combate (llama `/battle/action`)  
5. HTTP responde con `log[]` + `HP`  
6. GPT narra -> se envía mensaje narrado  

---

## Resultado

Un sistema automatizado y altamente inmersivo para jugar *Knave* de forma conversacional, permitiendo a los jugadores interactuar sólo escribiendo comandos naturales. Toda la narrativa y lógica está orquestada por flujos de n8n y potenciada por GPT.

> Diseñado para el Hackatón con enfoque en creatividad narrativa, modularidad técnica y experiencia fluida desde el chat.

# Decisiones del Proyecto - Knave Game Chatbot

Este documento recoge las decisiones técnicas, narrativas y funcionales que se han tomado durante el desarrollo del sistema conversacional de *Knave* usando n8n, GPT y microservicios.

---

## Arquitectura y Herramientas

- **n8n como orquestador principal** del juego por su flexibilidad y visualización clara de flujos.
- **GPT-4 como motor de interpretación y narrativa**, por su capacidad de generar respuestas evocadoras y naturales.
- **Airtable como base de datos rápida y fácil de consultar** para personajes, sesiones y registros de conversación.
- **Uso de microservicio en Node.js** para el sistema de combate, con rutas `/battle/start` y `/battle/action`.

---

## Conversacional & UX

- **Prompt estricto en JSON** para asegurar que el bot siempre devuelva respuestas procesables.
- **Desacoplar intención de acción**: el agente solo detecta la intención, n8n se encarga de ejecutar.
- **Estilo narrativo medieval** en saludos, selección de personajes y combate, para mantener la inmersión.
- **Evitar exigir comandos explícitos al usuario**: permitir que la IA infiera intenciones como "ver personaje" cuando el usuario dice solo un nombre.

---

## Combate y Juego

- **No se permite continuar con personajes muertos**: al morir, la sesión se marca como final y el personaje no puede reutilizarse.
- **Narrativa épica personalizada para cada turno** mediante un segundo agente GPT.
- **Las opciones siempre se devuelven como** `attack`, `defend`, `hide`, **simplificando el control de flujo.**
- **La sesión de combate se borra sólo cuando el personaje o el monstruo mueren.**
- **Se añade** `stop_battle` **como acción reconocida** para permitir que el jugador escape del combate.

---

## Base de Datos y Estado

- **Inicialmente se usó Airtable para guardar el chat**, pero se migró a `Simple Memory` en n8n para mayor velocidad.
- **Las sesiones activas se gestionan en Airtable (status = IN_BATTLE)**, y son limpiadas al finalizar.
- **Se decidió conservar Airtable como backup, auditoría y para visualización externa.**

---

## Integración GPT

- **Se separó el GPT de control (flujo) del GPT de narrativa**, evitando respuestas mezcladas.
- **Se creó un prompt base que analiza contexto reciente para inferir intenciones ambiguas** ("ver", "jugar", "Mina").
- **El narrador épico siempre incluye al final de su mensaje los puntos de vida y opciones disponibles.**

---

## Validaciones Importantes

- Si ya hay un personaje con ese nombre, **se impide crear uno nuevo** con el mismo nombre.
- Si no se encuentra el personaje pedido para ver o seleccionar, **se devuelve un mensaje claro al usuario**.
- Se corrigió un bug donde la acción `battle` llamaba siempre a `/start` incluso en turnos posteriores: ahora se llama correctamente a `/action`.

---

## Enfoque de Desarrollo

- Diseño modular por ramas de acción para permitir fácil extensión (misiones, loot, inventario).
- Foco en claridad visual del flujo en n8n para facilitar colaboración.
- Documentación de prompts y formatos JSON para asegurar mantenibilidad.

---

## Próximas Decisiones Evaluadas

- Persistir memoria en vectores (e.g., Pinecone) para seguimiento largo de historia.
- Introducir estados de exploración fuera de combate.
- Agregar sistema de logros y eventos aleatorios.

---

Estas decisiones han sido tomadas de forma iterativa, buscando equilibrar complejidad técnica, narrativa atractiva y experiencia fluida para el jugador.

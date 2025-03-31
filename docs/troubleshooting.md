# 🛠️ Troubleshooting - Knave Game Chatbot

Este documento recoge los principales problemas encontrados durante el desarrollo del proyecto y las soluciones implementadas o lecciones aprendidas.

---

## Problemas Frecuentes

### 1. No se encontraba el personaje seleccionado
- **Causa:** El nombre del personaje enviado por la IA no coincidía con el registro de Airtable por diferencias de mayúsculas/minúsculas o errores de sincronización.
- **Solución:**
  - Se utilizó `.toLowerCase().trim()` al comparar nombres.
  - Se añadió un fallback con mensaje de error amigable y no se detiene el flujo.

---

### 2. Se borraba la sesión incluso cuando la batalla no había terminado
- **Causa:** El microservicio de combate eliminaba la sesión cuando `monster.currentHP <= 0` o `character.currentHP <= 0`, pero también en condiciones erróneas.
- **Solución:**
  - Se modificó para mantener la sesión y marcar el estado con `battle.status = 'victory'` o `'defeat'` sin borrarla inmediatamente.
  - La sesión se borra solo después de que el bot procese el estado final correctamente.

---

### 3. GPT devolvía acción `stop_battle` incorrectamente
- **Causa:** El narrador mencionaba "has ganado" y la IA asumía que el jugador quería detener el combate.
- **Solución:**
  - Se añadió una instrucción clara en el prompt: solo usar `stop_battle` si el usuario lo pide explícitamente.

---

### 4. Confusión entre acciones `view_character` y `select_character`
- **Causa:** Frases ambiguas como "quiero jugar con Mina" eran mal interpretadas.
- **Solución:**
  - Se mejoró el prompt con reglas de inferencia contextual usando el historial reciente.

---

### 5. El bot enviaba múltiples mensajes duplicados
- **Causa:** El flujo conectaba varios nodos Telegram sin control central.
- **Solución:**
  - Se centralizó la respuesta final en un solo nodo posterior al narrador o al analizador de acción.

---

### 6. El flujo llamaba a `/battle/start` incluso después del turno 1
- **Causa:** No se distinguía si ya existía una sesión activa.
- **Solución:**
  - Se verifica en Airtable si hay una sesión `IN_BATTLE` activa antes de decidir entre `/start` o `/action`.

---

### 7. GPT devolvía JSONs mal formateados o incompletos
- **Causa:** Prompts demasiado largos o poco estrictos.
- **Solución:**
  - Se reescribió el prompt principal exigiendo solo un objeto JSON y usando formato validado con fallback.

---

### 8. Airtable devolvía múltiples entradas cuando solo debía devolver una
- **Causa:** Consulta mal construida o falta de filtrado por `telegramId` y `status`.
- **Solución:**
  - Se ajustó el filtro a: `AND({telegramId} = 'xxx', {status} = 'IN_BATTLE')`

---

### 9. l nodo `Switch` no funcionaba correctamente con `nextAction`
- **Causa:** El campo `nextAction` venía vacío o como `null` y no era manejado bien por el `Switch`.
- **Solución:**
  - Se mejoró la validación previa y se normalizó `nextAction` como cadena vacía si no existía.

---

## Recomendaciones Generales

- Mantener los prompts en archivos versionados.
- Validar siempre el output del GPT con un `Code` node antes de usarlo.
- Usar Airtable solo para datos persistentes, y `Simple Memory` para contexto de corto plazo.
- Dividir claramente entre lógica narrativa (GPT) y lógica de acción (n8n).

---

Este documento seguirá creciendo conforme se identifiquen nuevos errores y ajustes necesarios.

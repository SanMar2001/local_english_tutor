# Plan de Ruta — English Tutor Local AI

> Rama: `plan-roadmap` (creada desde `develop`).
> Fuente de requisitos: `README.md`. Reglas de proceso: `AGENTS.md`.
> Regla de oro: **no avanzar al siguiente paso hasta que el verificador del paso actual pase y el commit esté creado.**

---

## Estado actual verificado (inicio de esta rama)

- `README.md` y `AGENTS.md` modificados sin commitear.
- Existe un esqueleto FastAPI no commiteado: `server/main.py`, `server/database/database.py`, `server/models/base.py`, `server/models/conversation.py`.
- `server/models/` está ignorado por `.gitignore` (`models/` línea 26) → los modelos **no se están trackeando**. Bug a corregir en Paso 0.
- `server/database/database.py` contiene código duplicado (engine/`get_db`/`init_db` definidos dos veces) y hacks `importlib` porque faltan `__init__.py` en `server/database/`.
- No hay `requirements.txt`, ni tests, ni `client/`, ni learner en el modelo de datos.
- `data/english-tutor.db` y `server.log` generados localmente.

---

## Reglas del plan

1. Un paso = un commit verificado.
2. El verificador de cada paso debe ejecutarse **realmente**; no se reporta "pasó" sin ejecutarlo.
3. Si un paso revela una decisión arquitectónica no documentada → parar y pedir aprobación humana (AGENTS.md → Human approval).
4. Si el paso crece mucho más de lo previsto → parar y crear `docs/handoff.md`.
5. No implementar nada de un paso futuro "por si acaso".

---

## Paso 0 — Consolidar la base del repositorio

**Objetivo:** que el código existente sea importable, testeable y commiteable de forma limpia.

**Tareas:**

- Corregir `.gitignore`: cambiar `models/` por `/models/` para que deje de ignorar `server/models/`; añadir `server.log`.
- Eliminar el código duplicado de `server/database/database.py` (una sola definición de engine/`get_db`/`init_db`).
- Eliminar los hacks `importlib` de `server/main.py` y `server/database/database.py`; añadir `__init__.py` faltantes (`server/database/__init__.py`, `server/models/__init__.py`); borrar el `__init__.py` de la raíz si no se necesita.
- Crear `requirements.txt` con las dependencias realmente usadas (fastapi, uvicorn, sqlalchemy).
- Commitear los cambios pendientes de `README.md`/`AGENTS.md`.
- Commitear el esqueleto del servidor (sin `data/*.db`, sin `server.log`, sin `__pycache__`).

**Verificador (todos deben pasar):**

```bash
git check-ignore server/models/conversation.py   # debe fallar (no ignorado)
python -c "from server.main import app; print('import ok')"
python -m uvicorn server.main:app --port 8000 &
curl -s http://localhost:8000/                    # {"status":"ok",...}
curl -s http://localhost:8000/api/conversations   # [] o JSON válido
git status --short                                # sin archivos generados
```

---

## Paso 1 — Infraestructura de verificación (pytest)

**Objetivo:** tener un verificador automatizable para todos los pasos siguientes.

**Tareas:**

- Añadir `pytest` y `httpx` a `requirements.txt`.
- Crear `tests/test_health.py` usando `fastapi.testclient` contra la app.
- Crear `tests/conftest.py` que apunte la BD a un archivo temporal (no tocar `data/english-tutor.db`).

**Verificador:**

```bash
python -m pytest -q    # todos los tests en verde, 0 warnings nuevos
```

---

## Paso 2 — Modelo Learner + API de learners

**Objetivo:** soportar múltiples learners con datos independientes (sin autenticación).

**Tareas:**

- `server/models/learner.py`: `Learner` (id, name, created_at).
- Endpoints: `POST /api/learners`, `GET /api/learners`, `GET /api/learners/{id}` (404 si no existe).
- Tests: crear 2 learners, verificar aislamiento de ids.

**Verificador:**

```bash
python -m pytest -q
curl -s -X POST http://localhost:8000/api/learners -H "Content-Type: application/json" -d '{"name":"Ana"}'
curl -s http://localhost:8000/api/learners
```

---

## Paso 3 — Relación Learner ↔ Conversation (aislamiento)

**Objetivo:** cada conversación pertenece exactamente a un learner.

**Tareas:**

- Convertir `learner_id` en FK real a `learners` (con índice).
- Añadir endpoint de borrado/listado filtrado por learner; rechazar crear conversación con `learner_id` inexistente (422/404).
- Tests de aislamiento: learner A no ve conversaciones de learner B.

**Verificador:**

```bash
python -m pytest -q
# borrado de data/english-tutor.db y recreación para validar migración esquemática
rm data/english-tutor.db && curl -s http://localhost:8000/db/health
curl -s "http://localhost:8000/api/conversations?learner_id=1"
```

---

## Paso 4 — Cliente Ollama (abstracción)

**Objetivo:** integrar el LLM local detrás de una interfaz reemplazable (README: la integración debe poder cambiar de modelo sin rediseñar).

**Tareas:**

- `server/llm/client.py`: interfaz mínima `chat(messages) -> str` + implementación Ollama (`http://localhost:11434`).
- Endpoint `GET /llm/health` que reporte si Ollama responde y qué modelos hay.
- **No** aplicar lógica pedagógica todavía.

**Verificador:**

```bash
python -m pytest -q
curl -s http://localhost:8000/llm/health          # modelo listado o error controlado documentado
```

*Nota:* si Ollama no está corriendo en la máquina, el verificador debe probar el cliente con un mock y reportarse explícitamente como parcial.

---

## Paso 5 — Conversación de texto con respuesta del tutor

**Objetivo:** el learner escribe y el tutor responde usando el LLM local.

**Tareas:**

- `POST /api/conversations/{id}/tutor-reply`: guarda el mensaje del learner, llama al LLM, guarda la respuesta del tutor, devuelve ambos.
- Contexto: usar los mensajes previos de la conversación.
- Tests con cliente LLM simulado (sin dependencia de red en CI).

**Verificador:**

```bash
python -m pytest -q
# prueba manual end-to-end con Ollama corriendo:
curl -s -X POST http://localhost:8000/api/conversations/1/tutor-reply -H "Content-Type: application/json" -d '{"learner_id":1,"role":"learner","text":"Hello"}'
```

---

## Paso 6 — Comportamiento pedagógico (system prompt por learner)

**Objetivo:** el LLM actúa como profesor de inglés, no como chatbot genérico (README → Pedagogical Principles).

**Tareas:**

- `server/llm/prompt.py`: construye el system prompt con nivel actual, meta y objetivo del learner (usan campos aún no persistidos → usar valores por defecto documentados hasta el Paso 8).
- Aplicar el prompt en el Paso 5.
- Test: el prompt incluye nivel/meta; test de que no se rompe si el learner no tiene meta.

**Verificador:**

```bash
python -m pytest -q
# manual: enviar un mensaje con error de gramática y comprobar que el tutor corrige/explica
```

---

## Paso 7 — Evaluación diagnóstica adaptativa

**Objetivo:** assessment inicial que estima nivel CEFR por destreza (README → Initial Assessment).

**Tareas:**

- Modelo `Assessment` + `assessment_items` (learner_id, pregunta, respuesta, resultado).
- `POST /api/assessments/{id}/answer`: la siguiente pregunta se decide según respuestas anteriores (adaptativa, vía LLM).
- `POST /api/assessments/{id}/finish`: devuelve nivel estimado A1–C2, fortalezas, debilidades, confianza, **con aviso explícito de que no es certificación oficial**.
- Tests con LLM simulado: ramas de dificultad ascendente/descendente.

**Verificador:**

```bash
python -m pytest -q
# manual: completar un assessment corto y comprobar que la dificultad cambia según respuestas
```

---

## Paso 8 — Meta de aprendizaje del learner

**Objetivo:** persistir nivel actual + nivel objetivo + objetivo específico (README → Learning Goal).

**Tareas:**

- Campos en `Learner`: `current_level`, `target_level`, `goal_type`.
- `PATCH /api/learners/{id}` limitado a esos campos, validando pares válidos (A1→A2 … C1→C2).
- Reemplazar los valores por defecto del Paso 6 por los campos reales.
- Tests de validación de pares inválidos (p. ej. C2→A1 rechazado o documentado).

**Verificador:**

```bash
python -m pytest -q
curl -s -X PATCH http://localhost:8000/api/learners/1 -H "Content-Type: application/json" -d '{"target_level":"B2","goal_type":"job interviews"}'
```

---

## Paso 9 — Plan de aprendizaje personalizado

**Objetivo:** generar y persistir un plan adaptativo (README → Personalized Learning Plan).

**Tareas:**

- Modelo `LearningPlan` (learner_id, JSON con prioridades, temas, criterios de dominio, versión).
- `POST /api/learners/{id}/plan` (genera con LLM a partir de assessment + meta) y `GET`.
- El plan se regenera/actualiza, no se hardcodea; distinto learner → distinto plan.
- Tests: dos learners con niveles distintos obtienen planes distintos.

**Verificador:**

```bash
python -m pytest -q
curl -s -X POST http://localhost:8000/api/learners/1/plan
curl -s http://localhost:8000/api/learners/1/plan
```

---

## Paso 10 — Progreso y reevaluación

**Objetivo:** medir progreso con múltiples señales y actualizar el nivel estimado (README → Progress Reassessment).

**Tareas:**

- Modelo `Progress` por learner con las métricas documentadas en README (`grammar_accuracy`, `fluency`, `wpm`, `recurring_errors`, …) como **señales, no medidas exactas**.
- Cálculo a partir de las conversaciones/assessments existentes (sin tablas especulativas: solo las métricas que realmente se calculen).
- Umbral documentado para sugerir reevaluación.
- Tests: progreso de un learner no contamina el de otro.

**Verificador:**

```bash
python -m pytest -q
curl -s http://localhost:8000/api/learners/1/progress
```

---

## Paso 11 — Frontend React + Vite + TypeScript

**Objetivo:** interfaz mínima para usar el sistema de verdad.

**Tareas:**

- Scaffold en `client/` (Vite + React + TS), proxy a `localhost:8000`.
- Pantallas: selección/creación de learner, lista y detalle de conversación, chat de texto, vista de nivel/meta/plan.
- **No** incluir audio todavía.

**Verificador:**

```bash
cd client && npm run build        # build sin errores
cd client && npm run lint         # si existe script de lint
# manual: seleccionar learner, enviar mensaje, ver respuesta del tutor en el navegador
```

---

## Paso 12 — Transcripción Whisper.cpp (STT)

**Objetivo:** transcripción local de voz (README → Speech-to-Text).

**Tareas:**

- `server/audio/transcribe.py` con Whisper.cpp + GGUF local.
- Endpoint `POST /api/audio/transcribe`.
- El LLM/BD no se enteran del detalle: el texto transcrito entra al flujo del Paso 5.
- GPU opcional: debe funcionar sin GPU.

**Verificador:**

```bash
python -m pytest -q
curl -s -X POST http://localhost:8000/api/audio/transcribe -F "file=@sample.wav"   # devuelve texto
```

---

## Paso 13 — Pipeline de audio WebRTC + PeerJS

**Objetivo:** conversación hablada en tiempo real (README → Audio / Communications).

**Tareas:**

- Integración PeerJS en el cliente, flujo de audio → STT → tutor → respuesta.
- Verificar comportamiento sin GPU y con red local.

**Verificador:**

```bash
cd client && npm run build
# manual: hablar en el navegador, ver transcripción y respuesta del tutor
```

---

## Paso 14 — Memoria semántica (ChromaDB)

**Objetivo:** recordar vocabulario/errores recurrentes entre sesiones (README → Semantic Memory).

**Tareas:**

- `server/memory/` con ChromaDB local, indexando vocabulario y errores recurrentes por learner.
- Integración opcional en el system prompt (Paso 6).
- Aislamiento por learner verificado.

**Verificador:**

```bash
python -m pytest -q
# manual: mencionar un término en la sesión 1, comprobar que aparece recuperado en la sesión 2
```

---

## Paradas obligatorias (AGENTS.md)

- Al terminar cada paso: reportar, commitear, **parar**.
- Decisión arquitectónica nueva (esquema, dependencia mayor, tecnología del README) → parar y preguntar.
- Dos fallos seguidos del mismo verificador → parar e investigar sin tocar código no relacionado.
- Contexto de sesión saturado → `docs/handoff.md` y parar.

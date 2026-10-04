# Plan de Ruta — English Tutor Local AI

> Rama: `plan-roadmap` (creada desde `develop`).
> Fuente de requisitos: `README.md` en su versión local sin commitear. Reglas de proceso: `AGENTS.md` en su versión local sin commitear.
> Este plan refleja ambas versiones locales; su commit queda pendiente en el Paso 0.
> Regla de oro: **no avanzar al siguiente paso hasta que el verificador pase, el diff esté inspeccionado y el commit esté creado.**

---

## Estado actual verificado (inicio de esta rama)

- `README.md` y `AGENTS.md` modificados sin commitear (son las versiones con las que se escribió este plan).
- **No hay código Python en el repo**: solo `.gitignore`, `README.md`, `AGENTS.md`, `plan.md`. El esqueleto `server/` que existió fue eliminado del working tree; el plan parte de cero.
- `.gitignore` contiene `models/` (línea 26) que ignoraría `server/models/` en cuanto exista. Bug a corregir en Paso 0.
- No hay `requirements.txt`, ni tests, ni `client/`, ni learner en el modelo de datos.
- `data/english-tutor.db` es un residuo local (ya ignorado por `data/*.db`); se puede borrar sin más.

---

## Reglas del plan

1. Un paso = un commit verificado.
2. **Checklist de cada paso** (AGENTS.md → Milestones):

   *Antes de empezar:*
   1. Inspeccionar el estado real: `git status` + `git log --oneline -5`.
   2. Leer en `README.md` la sección de requisitos del paso.
   3. Definir los archivos mínimos que toca.
   4. Si hay ambigüedad o decisión arquitectónica → parar y pedir aprobación humana.

   *Al terminar:*
   1. Ejecutar el verificador **realmente**; sin ejecutarlo el paso no está terminado.
   2. Inspeccionar `git diff`: confirmar que no entran cambios no relacionados.
   3. Si el verificador revela warnings/errores introducidos por el paso → resolverlos antes de cerrar.
   4. Reportar qué se implementó y cómo se verificó.
   5. Un commit coherente por paso. No commitear trabajo incompleto ni archivos generados (`data/*.db`, `server.log`, `__pycache__`, modelos locales, credenciales). No reescribir historia.

3. Verificación: preferir la automatizada (`pytest`); todo endpoint nuevo se prueba con peticiones reales (`curl`); todo cambio de BD verifica que la app inicializa e interactúa con la BD. No ocultar problemas sin resolver; si algo no se pudo verificar, decirlo explícitamente.
4. No inferir requisitos no documentados. El `Data Model Direction` del README es conceptual: **no** crear tablas porque aparezcan en ese diagrama, solo las que el paso actual necesita.
5. Sin autenticación en ningún paso salvo petición explícita, y sin servicios cloud: todo local (README + AGENTS → Project-specific principles).
6. Si un paso revela una decisión arquitectónica no documentada → parar y pedir aprobación (AGENTS.md → Human approval: esquema, dependencias mayores, persistencia, seguridad, servicios externos).
7. Si el paso crece mucho más de lo previsto → parar y proponer revisarlo; crear `docs/handoff.md` solo si el contexto debe transferirse a otra sesión (el handoff es un documento de transición, no un duplicado del historial).
8. No implementar nada de un paso futuro "por si acaso": sin tablas, endpoints, modelos ni abstracciones especulativas.
9. Si una decisión futura relevante aparece, documentarla en el reporte/handoff, no implementarla.

---

## Servidores y entorno (AGENTS.md → Development environment / servers)

- Windows 11 + Git Bash/MINGW64: no asumir utilidades Unix (`pkill`, `fuser`).
- Antes de arrancar servidor: comprobar que el puerto 8000 no está ya en uso; reutilizar una instancia que responda bien; no arrancar duplicados ni cambiar de puerto en silencio.
- Para terminar un proceso: identificar el PID y usar `MSYS_NO_PATHCONV=1 taskkill /PID <PID> /F`.
- Nunca reportar un servidor como detenido sin haberlo verificado.

---

## Orden y estructura

- El orden sigue la secuencia de hitos de `README.md` (Development Philosophy). Una adaptación: el frontend (Paso 11) va antes que Whisper/audio porque WebRTC + PeerJS vive en el navegador y no se puede verificar sin cliente.
- Estructura objetivo del README: cuando los endpoints superen `server/main.py`, moverlos a `server/api/`; la lógica de plan/progreso va en `server/learning/`. No crear esos directorios hasta que un paso los necesite (no estructura vacía).

---

## Paso 0 — Repositorio limpio + arranque FastAPI y SQLite

**Objetivo:** partir de una base importable, testeable y commiteable (README → hitos "Initialize FastAPI" y "Configure SQLAlchemy + SQLite").

**Tareas:**

- Corregir `.gitignore`: cambiar `models/` por `/models/` para que no ignore `server/models/`; añadir `server.log`.
- Crear `requirements.txt` con las dependencias realmente usadas (fastapi, uvicorn, sqlalchemy).
- Crear el esqueleto mínimo de `server/`: `__init__.py`, `server/main.py` (app + health check) y `server/database/database.py` con **una sola** definición de engine/`get_db`/`init_db` (SQLite en `data/`), `server/models/base.py`. Imports de paquete normales, sin `importlib`, sin código duplicado.
- Commitear los cambios pendientes de `README.md`/`AGENTS.md`.
- Commitear el esqueleto (sin `data/*.db`, sin `server.log`, sin `__pycache__`).

**Verificador (todos deben pasar):**

```bash
git check-ignore server/models/base.py           # debe fallar (no ignorado)
python -c "from server.main import app; print('import ok')"
# comprobar puerto libre antes de arrancar (o reutilizar el servidor si ya responde)
python -m uvicorn server.main:app --port 8000 &
curl -s http://localhost:8000/                    # {"status":"ok",...}
curl -s -o /dev/null -w "%{http_code}" http://localhost:8000/docs   # 200
git status --short                                # sin archivos generados (data/, logs, __pycache__)
git diff HEAD --stat                              # diff revisado: solo cambios del paso
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

**Objetivo:** soportar múltiples learners con datos independientes, mediante selección local de learner y **sin autenticación** (README → Learners).

**Tareas:**

- `server/models/learner.py`: `Learner` (id, name, created_at).
- Endpoints: `POST /api/learners`, `GET /api/learners`, `GET /api/learners/{id}` (404 si no existe).
- Tests: crear 2 learners, verificar que cada uno tiene su identidad independiente.

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
# borrar la BD local y comprobar que la app la recrea desde cero con el esquem actual
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
- Contexto: los mensajes conservan suficiente información para reconstruir la conversación (README → Conversations).
- Tests con cliente LLM simulado (sin dependencia de red).

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

- `server/llm/prompt.py`: construye el system prompt con nivel actual, meta y objetivo del learner (campos aún no persistidos → valores por defecto documentados hasta el Paso 8).
- La estrategia se adapta al nivel y la meta del learner; distintos learners pueden recibir distinta estrategia, temas y dificultad (README → Pedagogical Principles).
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

- Modelo `Assessment` + `assessment_items` (learner_id, pregunta, respuesta, resultado): el assessment pertenece a un learner y no se comparte entre learners.
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

- Campos en `Learner`: `current_level`, `target_level`, `goal_type` — pertenecen al contexto del learner, no a ajustes globales de la aplicación (README → Learning Goal).
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
- Pantallas: selección/creación de learner (local, sin login), lista y detalle de conversación, chat de texto, vista de nivel/meta/plan.
- **No** incluir audio todavía (el audio llega en el Paso 13).

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

- Al terminar cada paso: ejecutar el checklist completo (verificador → `git diff` → reporte → commit) y **parar**; no continuar automáticamente al siguiente.
- Decisión arquitectónica nueva (esquema, dependencia mayor, persistencia, seguridad/auth, tecnología del README, servicios externos) → parar y pedir aprobación.
- Dos fallos seguidos del mismo verificador → documentar el problema y parar, sin tocar código no relacionado.
- Si el paso revela que su definición es insuficiente → proponer un milestone revisado en lugar de expandir alcance en silencio.
- Contexto de sesión saturado o poco fiable → `docs/handoff.md` y parar (una sesión limpia es preferible a un razonamiento degradado).
- Milestone completado y sin contexto que transferir → no hace falta handoff.

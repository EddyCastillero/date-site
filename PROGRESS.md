# PROGRESS.md

Bitácora del proyecto para no perder el hilo entre sesiones y agentes.
**Cualquier agente nuevo: lee primero `AGENTS.md` (reglas de cómo trabajar) y luego este archivo.**
Actualiza este archivo al terminar cada paso.

Última actualización: 2026-10-30

## Decisiones tomadas

- **Framework:** FastAPI (decidido).
- **Entorno:** `.venv` local con `python3 -m venv` (Python 3.13). Docker queda para más adelante, cuando ya haya algo que contenerizar (orden del plan en `AGENTS.md`).
- **Dependencias:** `pip install "fastapi[standard]"` y `pip freeze > requirements.txt` (versiones fijadas con `==`).
- **Ramas:** se trabaja en `dev`. `main` queda estable. Commits pequeños, uno por paso.
- **Comandos:** `fastapi dev main.py` para desarrollo. Docs automáticas en `http://127.0.0.1:8000/docs`.

## Lo que ya está hecho

| # | Paso | Estado |
|---|------|--------|
| 1 | Entorno: `.venv`, FastAPI instalado, `requirements.txt`, `.gitignore` | Hecho (commit en `dev`) |
| 2 | `GET /health` en `main.py` devolviendo `{"status": "ok"}` | Hecho (falta commit si no se ha hecho) |

### Archivos actuales
- `main.py`: instancia `app = fastapi.FastAPI()` y la función `health_check` en `GET /health`.
- `requirements.txt`: dependencias fijadas.
- `.gitignore`: `.venv/`, `.env`, `__pycache__/`, etc.
- `AGENTS.md` / `GEMINI.md` / `CLAUDE.md`: reglas para el asistente.

## Conceptos que Leonel ya practicó

- `venv`: es un entorno aislado (intérprete + librerías), no "empaquetar". Se activa anteponiendo `.venv/bin` al `PATH`. No se sube a Git; se regenera con `requirements.txt`.
- En Debian/Ubuntu hace falta `python3.13-venv` (el `venv` quedaba a medias sin él).
- `requirements.txt` y dependencias transitivas. Solo deja huella real si las versiones están fijadas.
- Path operation = método HTTP + ruta + función (handler).
- Uvicorn es el servidor ASGI. FastAPI no abre puertos. En `uvicorn main:app`, `main` es el módulo (`main.py`) y `app` es la variable con la instancia de `FastAPI`.
- Respuesta HTTP = línea de estado + headers + body.
- Status codes vistos con `curl -i`:
  - `200` -> la operación existe y funcionó.
  - `405` -> la ruta existe pero el método no está permitido (trae header `allow`).
  - `404` -> la ruta no existe.
- `curl` es un cliente HTTP, no un método. `POST` es el método.
- Nombres de función: describir la intención (`health_check`, no `root` ni `def_root`).

## Pendientes menores (no bloquean)

- Probar cambiar `app` por `servidor` y correr `uvicorn main:app` para leer el error.
- Cambiar `import fastapi` por `from fastapi import FastAPI` (convención más común). Opcional.
- Revisar por qué `requirements.txt` trae paquetes de telemetría (`sentry-sdk`, `opentelemetry-*`) y si el `.gitignore` con `.sentryclirc` es necesario. Pregunta de seguridad: en producción, ¿instalar `fastapi[standard]` entero o solo lo necesario?
- `/health` solo devuelve `ok`. Cuando haya base de datos, debería verificar que la DB responde.

## Paso actual: 3. Modelo `Invitation` (lo diseña Leonel)

Campos según `AGENTS.md`: quién invita, fecha, hora, lugar, plan, estado.

Preguntas que Leonel debe responder antes de escribir código:
1. ¿Qué tipo de dato va en cada campo? ¿Fecha y hora juntas o separadas? (zonas horarias)
2. El estado es `pendiente`, `aceptada` o `rechazada`. ¿String libre o enum? ¿Qué pasa si llega `"acpetada"`?
3. ¿Qué campos son obligatorios y cuáles opcionales?
4. ¿Falta algo que una tabla real necesitaría? (identificador, fecha de creación, etc.)

Entregable: `models.py` con una clase Pydantic (`BaseModel`) con tipos. Solo el modelo, sin base de datos todavía.
Documentación: https://fastapi.tiangolo.com/tutorial/body/

Al terminar: el agente hace code review (qué está bien, qué mejorar, qué sería problema en producción).

## Pasos siguientes (plan)

4. SQLite + `POST /invitations` con validación.
5. `GET /invitations/{id}` y manejo de errores (404, 422).
6. Frontend con botones Sí/No (el No cambia la pregunta a "¿Segura?", el Sí crece y el No se encoge).
7. Estados de la invitación (pendiente, aceptada, rechazada) y endpoint para responder.
8. Link único por invitación (id no adivinable, no un entero secuencial).
9. Notificaciones.
10. Docker.
11. Deploy en la nube. Después, migrar SQLite a Postgres como ejercicio.

## Cómo retomar en una sesión nueva

1. `cd /home/leonel/Desktop/date-site`
2. `source .venv/bin/activate`
3. `fastapi dev main.py`
4. Abrir `http://127.0.0.1:8000/docs`

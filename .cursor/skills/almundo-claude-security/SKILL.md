---
name: almundo-claude-security
description: Refuerza seguridad y compliance en el fork almundo-claude ajustando hooks, manejo de secretos y políticas alineadas con Almundo.
---

# Seguridad y compliance en almundo-claude

## Cuándo usar esta skill

Usa esta skill cuando:

- Edits o revises ficheros en `hooks/` relacionados con seguridad.
- Evalúes el impacto de cambios que pueden afectar a datos personales, infra de producción o secretos.
- Tengas que adaptar el comportamiento del plugin a políticas internas de seguridad de Almundo.

## Instrucciones

1. **Hooks críticos a revisar**

- `hooks/secret-guard.sh`
- `hooks/dangerous-command-guard.py`
- `hooks/sensitive-read-guard.py`
- `hooks/dependency-watch.py`
- `hooks/activity-capture.py`

Para cada uno:
- Comprueba que contempla patrones y comandos específicos de Almundo (dominios internos, prefijos de tokens, comandos sensibles).
- Evita relajar las protecciones sin una justificación muy clara.

2. **Gestión de secretos**

- Nunca propongas almacenar credenciales en ficheros versionados.
- Favorece variables de entorno y almacenes de secretos aprobados por Almundo.
- Si detectas patrones que puedan filtrar datos sensibles en logs o memoria, propon soluciones de minimización.

3. **Compliance**

- Ten en cuenta RGPD, NIS2, CRA y políticas internas de Almundo cuando:
  - Cambies cómo se registra actividad (`activity-capture.py`).
  - Modifiques qué datos pueden persistirse en memoria (`core/memory.py` y `docs/memory.md`).
- Si hay dudas, recomienda documentar la decisión en un ADR usando `templates/adr.md`.

4. **Integración con procesos internos**

- Si Almundo dispone de procesos o herramientas propias (scanner de dependencias, gestión de incidentes, etc.), integra los hooks con ellos cuando tenga sentido (por ejemplo, marcando librerías vetadas en `dependency-watch.py`).

## Ejemplos de uso

- El usuario pide: “Ajusta el guard de comandos peligrosos para nuestra infraestructura” → Modifica `dangerous-command-guard.py` añadiendo comandos internos críticos de Almundo.
- El usuario pide: “Evita que el plugin use memoria persistente en ciertos repos” → Revisa `core/memory.py` y `docs/memory.md` para añadir opciones de configuración y recomendaciones específicas.


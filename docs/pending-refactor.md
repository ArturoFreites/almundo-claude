 # Pendientes de refactor Almundo IA

 Documento de trabajo para recoger posibles incoherencias o roturas introducidas por el refactor desde `alfred-dev` a **Almundo IA**.  
 No aplica cambios; solo identifica puntos a revisar y validar con tests o pruebas manuales.

 ## Alcance y limitaciones

 - **Alcance**: revisión estática de comandos, agentes, core (`core/*.py`), hooks (`hooks/*`) y documentación principal.
 - **Fuera de alcance**: ejecución real de `python -m pytest`, hooks dentro de Claude Code y flujo completo del plugin. Estos deben validarse en tu entorno local.
 - **Suposición clave**: el estado y la configuración siguen usando los ficheros `.claude/alfred-dev.local.md` y `.claude/alfred-dev-state.json` como en el upstream, tal y como describe la propia doc de Almundo IA.

 ## Hallazgos por área

 ### 1. Core / Python

 - [ ] **Referencia a `alfred-dev.local.md` en `core/config_loader.py`**
   - **Archivo**: `core/config_loader.py`
   - **Descripción**: La doc y el código siguen usando `".claude/alfred-dev.local.md"` como fichero de configuración del proyecto.
   - **Estado**: Coherente con `README.md` y `docs/configuration.md`, pero conviene confirmar que no se pretendía renombrar el fichero a algo específico de Almundo IA.
   - **Riesgo**: Bajo (solo naming; comportamiento igual al upstream).
   - **Acción propuesta**:
     - Decidir a nivel de producto si el nombre del fichero debe seguir heredado (`alfred-dev.local.md`) o si se quiere introducir un alias/renombre.
     - Si se renombra, ajustar `config_loader.py`, `hooks/*`, docs y tests de forma coordinada.

 - [ ] **Referencia a `alfred-dev.local.md` en `core/personality.py`**
   - **Archivo**: `core/personality.py`
   - **Descripción**: Comentarios y lógica hacen referencia a la configuración del usuario en `alfred-dev.local.md`.
   - **Riesgo**: Bajo.
   - **Acción propuesta**: Alinear el naming con la decisión anterior (mantener o renombrar el fichero de configuración).

 ### 2. Hooks

 - [ ] **Uso de rutas y branding original en `hooks/session-start.sh`**
   - **Archivo**: `hooks/session-start.sh`
   - **Descripción**:
     - Usa `CONFIG_FILE="${PROJECT_DIR}/.claude/alfred-dev.local.md"` y `STATE_FILE="${PROJECT_DIR}/.claude/alfred-dev-state.json"`, coherente con el upstream.
     - Mensajes al usuario y ejemplos mencionan `/alfred-dev:config` y `/alfred-dev:feature`.
   - **Riesgo**: Medio, a nivel de experiencia:
     - El core puede seguir funcionando, pero el mensaje al dev interno de Almundo puede ser confuso si el comando visible es `/almundo-ia`.
   - **Acción propuesta**:
     - Revisar y adaptar los mensajes dirigidos al usuario (no las rutas internas) para que hablen de `/almundo-ia` y comandos equivalentes.
     - Mantener las rutas `alfred-dev.local.md` / `alfred-dev-state.json` salvo decisión explícita de cambio.

 - [ ] **Referencias a `alfred-dev.local.md` y `alfred-dev-state.json` en otros hooks**
   - **Archivos**: `hooks/memory-compact.py`, `hooks/stop-hook.py`, `hooks/activity-capture.py`
   - **Descripción**: Usan explícitamente esos nombres de fichero, igual que el upstream.
   - **Riesgo**: Bajo (comportamiento esperado), pero hay que mantener la coherencia si se decide un cambio de naming.
   - **Acción propuesta**:
     - Dejar el comportamiento como está si se mantiene el nombre de fichero heredado.
     - Si se cambia el nombre, actualizar rutas y, muy importante, los tests asociados (`tests/test_activity_capture.py`, `tests/test_memory_compact.py`, etc.).

 ### 3. Commands

 - [ ] **Commands internos que aún hablan de `/alfred`**
   - **Archivos afectados (parcial)**: `commands/audit.md`, `commands/config.md`, `commands/feature.md`, `commands/fix.md`, `commands/ship.md`, `commands/status.md`, `commands/tutor.md`, `commands/_composicion.md`, `commands/update.md`.
   - **Descripción**:
     - Muchos comandos conservan el framing original (`/alfred feature`, `/alfred config`, etc.) tanto en títulos como en ejemplos y explicaciones internas.
     - El comando `commands/help.md` ya está alineado con `/almundo-ia:*`, por lo que conviven ambas notaciones.
   - **Riesgo**: Medio:
     - A nivel funcional, Claude sigue ejecutando el comando porque el nombre real viene de `plugin.json`, pero el mismatch en la doc interna puede provocar confusión o instrucciones erróneas dentro de los flujos.
   - **Acción propuesta**:
     - Definir una pauta clara:
       - Mantener la semántica de los flujos (`feature`, `fix`, `ship`, `audit`, `config`, `status`, `update`) pero actualizar el texto interno para que:
         - Hablen de `/almundo-ia feature`, `/almundo-ia fix`, etc. cuando se refieren a uso por parte del usuario de Almundo.
         - Dejen explícito cuando algo sigue usando nombres heredados por compatibilidad (por ejemplo, nombres de ficheros en `.claude/alfred-dev-*.json`).
     - Revisar especialmente:
       - Instrucciones que piden llamar explícitamente a `/alfred config` o similares.
       - Ejemplos inline de uso en los que se muestre el comando.

 - [ ] **Script de actualización `commands/update.md` orientado a `alfred-dev`**
   - **Archivo**: `commands/update.md`
   - **Descripción**:
     - Usa rutas y URLs del upstream (`~/.claude/plugins/cache/alfred-dev/**/.claude-plugin/plugin.json`, `https://api.github.com/repos/686f6c61/alfred-dev/releases/latest`, scripts `install.sh`/`install.ps1` de `alfred-dev`).
   - **Riesgo**: Alto si se pretende que el comando `/almundo-ia update` actualice el **fork privado** en lugar del plugin público original.
   - **Acción propuesta**:
     - Decidir si `/almundo-ia update` debe:
       - Seguir apuntando al upstream `alfred-dev` (para entornos donde se usa directamente) o
       - Actualizar el fork `almundo/almundo-claude` según lo definido en `docs/almundo-ia/PLAN.md`.
     - Si se elige la segunda opción:
       - Revisar `commands/update.md` para que las rutas, URLs y mensajes usen el repo del fork y el identificador del plugin `almundo-claude@almundo-claude`.
       - Validar con los tests de versión (`tests/test_version_consistency.py`) y con pruebas manuales.

 ### 4. Agents

 - [ ] **Ejemplos o textos que mezclan `/alfred` y `/almundo-ia`**
   - **Archivo**: `agents/senior-dev.md`
   - **Descripción**:
     - El frontmatter sigue hablando de fase 3 de `/alfred feature` y de `/alfred fix` en algunos ejemplos, mientras que el cuerpo ya está alineado con Almundo IA en otros puntos (ej.: referencia a `/almundo-ia fix` y al orquestador Almundo).
   - **Riesgo**: Bajo/Medio:
     - No rompe ejecución, pero puede generar prompts inconsistentes o confusos para el modelo.
   - **Acción propuesta**:
     - Homogeneizar la referencia a los comandos:
       - Usar sistemáticamente `/almundo-ia feature` y `/almundo-ia fix` cuando se hable del flujo actual.
       - Dejar constancia explícita, si interesa, de que el diseño está basado en los flujos `/alfred` originales.

 - [ ] **Referencias heredadas a `alfred-dev.local.md` en agentes opcionales**
   - **Archivo**: `agents/optional/ux-reviewer.md` (y otros agentes opcionales análogos).
   - **Descripción**:
     - Indican que deben leer `.claude/alfred-dev.local.md` para conocer preferencias del proyecto.
   - **Riesgo**: Bajo (alineado con el core y los hooks).
   - **Acción propuesta**:
     - Mantenerlo, pero quizá añadir una nota breve tipo "nombre heredado del proyecto original" (ya presente en algunos agentes) para evitar confusión futura.

 ### 5. Documentación

 - [ ] **Docs que aún se refieren al producto como “Alfred Dev”**
   - **Archivos clave**: `docs/architecture.md`, `docs/agents/README.md`, `docs/configuration.md`, `docs/hooks.md`, `docs/testing.md`, `docs/installation.md`, varios `docs/agents/*.md`, `templates/*.md`, `docs/superpowers/*`.
   - **Descripción**:
     - Muchas secciones siguen usando "Alfred Dev" como nombre del producto y describen el identificador `alfred-dev@alfred-dev`, marketplaces `alfred-dev`, etc.
     - El `README.md` principal ya introduce el fork como **Almundo** y explica que se basa en `alfred-dev`.
   - **Riesgo**: Bajo/Medio:
     - A nivel interno puede ser aceptable (el fork sigue basándose en `alfred-dev`), pero para devs de Almundo la mezcla puede ser confusa.
   - **Acción propuesta**:
     - Seguir las indicaciones de `docs/almundo-ia/PLAN.md`:
       - Cambiar textos de “Alfred Dev” a “Almundo IA” donde tenga sentido para devs internos.
       - Conservar referencias al nombre original cuando se documente el upstream (por ejemplo, secciones de "base alfred-dev", URLs, disclaimers).
     - Trabajar por bloques:
       - Priorizar `docs/agents/*.md`, `docs/architecture.md`, `docs/configuration.md` para que usen "Almundo IA" como nombre principal y referencien "Alfred Dev" solo cuando hablen del proyecto original.

 - [ ] **Plantillas que fijan el nombre del proyecto como `alfred-dev`**
   - **Archivo**: `templates/sbom.md` (y posiblemente otras plantillas).
   - **Descripción**:
     - Algunos campos incluyen `Proyecto: alfred-dev` o nombres fijos para el plugin.
   - **Riesgo**: Bajo, pero genera artefactos (SBOM, ADRs, etc.) que no reflejan el branding de Almundo IA.
   - **Acción propuesta**:
     - Parametrizar las plantillas o, como mínimo, actualizar el valor por defecto a "Almundo IA" cuando el contexto sea el fork.

 ### 6. Site (landing)

 - [ ] **Validar alineación de branding y comandos en el `site`**
   - **Archivos**: `site/src/pages/index.astro`, `site/src/pages/en/index.astro`, componentes de secciones de comandos y agentes.
   - **Descripción**:
     - No se ha revisado en detalle aquí, pero el riesgo típico tras el refactor es que el site hable todavía de "Alfred Dev" y `/alfred` mientras el plugin interno se denomina Almundo IA y usa `/almundo-ia`.
   - **Riesgo**: Medio de cara a devs que solo vean la landing.
   - **Acción propuesta**:
     - Revisar textos de la landing:
       - Nombre del producto → "Almundo IA" para devs internos.
       - Comandos de ejemplo → `/almundo-ia …`.
       - Secciones que describen la arquitectura y agentes → alinear con la nueva nomenclatura.
     - Asegurarse de que cualquier referencia explícita a `alfred-dev` se limita a la explicación del origen/open source.

 ## Validaciones recomendadas

 Para confirmar que no se ha roto nada a nivel funcional más allá de estas incoherencias de naming, ejecutar en tu entorno:

 - [ ] **Tests core Python**
   - Comando: `python3 -m pytest tests/ -v`
   - Apuntar aquí cualquier fallo junto con el módulo afectado y la traza resumida.

 - [ ] **Validación hooks + flujo dentro de Claude Code**
   - Ejecutar una sesión real con:
     - `/almundo-ia help`
     - `/almundo-ia config`
     - `/almundo-ia feature <feature pequeña>`
   - Verificar:
     - Que los hooks se disparan sin errores visibles.
     - Que las rutas `.claude/alfred-dev.local.md` y `.claude/alfred-dev-state.json` se crean/actualizan como se espera.
     - Que los mensajes al usuario no mencionan comandos antiguos salvo en secciones explícitas de compatibilidad.

 - [ ] **Landing `site/`**
   - Comandos recomendados (si existen los scripts):
     - `npm install` (primera vez)
     - `npm run lint`
     - `npm run build`
   - Anotar cualquier warning/errores relacionados con nombres, rutas o comandos.

 ---

 Este documento debe mantenerse vivo durante el refactor: cuando se resuelva un punto, marcar la casilla correspondiente y, si aplica, enlazar al commit o ADR donde se tomó la decisión.


---
title: Plan de fork privado y rebranding a `almundo-claude`
status: draft
version: 0.1.0
---

## Objetivo

Crear y mantener un **fork privado** de `alfred-dev` para uso interno, renombrado a **`almundo-claude`**, sin romper el plugin público original.

El resultado final debe ser:

- Un **repositorio privado** (por ejemplo `almundo/almundo-claude`) que contiene el código del plugin.
- Un **plugin de Claude Code** cuyo identificador sea `almundo-claude` y cuyos comandos/documentación estén alineados con la marca Almundo.
- Un **proceso repetible** para traer cambios desde el upstream `686f6c61/alfred-dev` sin perder personalizaciones internas.

---

## Fase 1 – Preparar el fork privado

1. **Crear el fork privado**
   - En la organización de Git (GitHub Enterprise, GitLab, etc.), crear un nuevo repositorio privado, por ejemplo:
     - `almundo/almundo-claude`
   - Inicialmente, se puede **importar** el repo público:
     - Origen: `https://github.com/686f6c61/alfred-dev`
     - Destino: `git@github.com:almundo/almundo-claude.git`

2. **Configurar remotos en local**
   - Clonar el nuevo repo privado:
     ```bash
     git clone git@github.com:almundo/almundo-claude.git
     cd almundo-claude
     ```
   - Añadir el remoto upstream al proyecto original:
     ```bash
     git remote add upstream https://github.com/686f6c61/alfred-dev.git
     git fetch upstream
     ```

3. **Definir ramas base**
   - Mantener una rama estable interna, por ejemplo:
     - `main` (o `corp-main`) = rama principal de Almundo.
   - Alinearla inicialmente con la última tag estable del upstream (ejemplo `v0.4.2`):
     ```bash
     git checkout -b corp-main upstream/v0.4.2
     git push -u origin corp-main
     ```

---

## Fase 2 – Rebranding técnico a `almundo-claude`

La idea es que **solo en el fork privado** el plugin se presente como `almundo-claude`. El repo público original sigue usando `alfred-dev`.

### 2.1. Manifiestos del plugin

En el fork privado, actualizar:

- Fichero: `.claude-plugin/plugin.json`
  - Cambiar:
    - `"name": "alfred-dev"` → `"name": "almundo-claude"`
    - `"homepage"` → URL interna o de documentación corporativa.
    - `"repository"` → URL del nuevo repo privado (`almundo/almundo-claude`).

- Fichero: `.claude-plugin/marketplace.json`
  - Cambiar el nivel superior:
    - `"name": "alfred-dev"` → `"name": "almundo-claude"`
  - En la sección `plugins[0]`:
    - `"name": "alfred-dev"` → `"name": "almundo-claude"`
    - `"homepage"` → URL de documentación corporativa si aplica.

> Nota: estos cambios hacen que el plugin se identifique como `almundo-claude` dentro de Claude Code y en el marketplace interno.

### 2.2. Script de instalación

En el fork privado, adaptar `install.sh` para apuntar al nuevo repositorio y nombre:

- Fichero: `install.sh`
  - Variables clave:
    - `REPO="686f6c61/alfred-dev"` → `REPO="almundo/almundo-claude"` (o el namespace real).
    - `PLUGIN_NAME="alfred-dev"` → `PLUGIN_NAME="almundo-claude"`.
  - Mensajes de salida:
    - Cambiar textos de “Alfred Dev” a “Almundo IA” donde tenga sentido para los devs internos.

Para Windows, repetir la misma lógica en `install.ps1` (si se utiliza).

### 2.3. Identificador del plugin en Claude Code

Claude Code identifica el plugin como `<plugin>@<marketplace>`. Con el rebranding propuesto:

- Marketplace: `almundo-claude`
- Plugin: `almundo-claude`
- Clave resultante: `almundo-claude@almundo-claude`

Por coherencia, usar siempre `almundo-claude` como:

- Nombre del marketplace (`known_marketplaces.json`).
- Nombre del plugin (`installed_plugins.json`, `settings.json > enabledPlugins`).

Los scripts de instalación del fork privado deben registrar y habilitar `almundo-claude@almundo-claude` en lugar de `alfred-dev@alfred-dev`.

---

## Fase 3 – Branding funcional y documentación

1. **README y documentación externa**
   - Ficheros a revisar:
     - `README.md`
     - `docs/README.md`
     - `docs/installation.md`
     - Cualquier referencia en `site/` si se reutiliza la landing para uso interno.
   - Cambiar:
     - Título principal: de “Alfred Dev” a “Almundo IA”.
     - Referencias textuales a “Alfred” donde se refieran al producto, sustituyéndolas por “Almundo IA”.

2. **Comandos y ejemplos**
   - Mantener el prefijo `/alfred` o no es una decisión de UX interna:
     - Opción A (mínimo cambio): mantener `/alfred` como comando pero hablar de “Almundo IA” en textos.
     - Opción B (alineado a marca): clonar los comandos con un nuevo nombre, por ejemplo `/almundo-ia feature`, etc. (requiere cambios en los ficheros de comandos y quizá en el core).
   - Recomendación inicial: **mantener `/alfred` como prefijo de comandos** para reducir cambios técnicos y solo cambiar el branding visible del plugin.

3. **Mensajes de los agentes**
   - Revisar si en los prompts o en `core/personality.py` se hace referencia a “Alfred” como personaje.
   - Decidir si se mantiene ese nombre a nivel “persona” o si se quiere que también diga “Almundo IA” en sus mensajes.

---

## Fase 4 – Hardening y alineación con políticas de Almundo

Las personalizaciones de seguridad deben hacerse exclusivamente en el fork privado.

1. **Hooks críticos**
   - Ficheros clave en `hooks/` a revisar y adaptar:
     - `secret-guard.sh`
     - `dangerous-command-guard.py`
     - `sensitive-read-guard.py`
     - `dependency-watch.py`
     - `activity-capture.py`
   - Acciones típicas:
     - Añadir patrones de secretos específicos de Almundo (nombres de dominios, prefijos de tokens, etc.).
     - Ajustar listas negras/listas blancas de comandos peligrosos según políticas internas.
     - Asegurar que la captura de actividad y memoria respeta normas internas de privacidad/compliance.

2. **Memoria persistente**
   - Fichero: `core/memory.py` y documentación en `docs/memory.md`.
   - Confirmar:
     - Ruta de la base de datos local.
     - Permisos de fichero (0600).
   - Definir política:
     - Proyectos donde está permitida / recomendada.
     - Proyectos donde debe estar desactivada por defecto.

3. **Compliance**
   - Revisar la sección de compliance (RGPD, NIS2, CRA) y, si es necesario, extenderla con controles o textos propios de Almundo (por ejemplo, normativa sectorial, políticas internas).

---

## Fase 5 – Estrategia de actualización desde upstream

Objetivo: poder aprovechar nuevas versiones de `alfred-dev` sin perder el rebranding ni los parches internos.

1. **Flujo recomendado**
   - Cuando haya nueva versión upstream:
     ```bash
     git fetch upstream
     git checkout corp-main
     git merge --no-ff upstream/main   # o upstream/<tag> concreta
     ```
   - Resolver conflictos, especialmente en:
     - `.claude-plugin/plugin.json`
     - `.claude-plugin/marketplace.json`
     - `install.sh` / `install.ps1`
     - Hooks en `hooks/`
     - Documentación en `docs/` y `README.md`

2. **Versionado interno**
   - Mantener un esquema propio, por ejemplo:
     - Upstream `0.4.2` → interno `0.4.2-almundo-claude.0`, `0.4.2-almundo-claude.1`, etc.
   - Reflejar la versión interna en:
     - `.claude-plugin/plugin.json > version`
     - `.claude-plugin/marketplace.json > plugins[0].version`
     - Mensajes de `install.sh` si aplica.

3. **Checklist después de cada actualización**
   - Ejecutar tests:
     ```bash
     python3 -m pytest tests/ -v
     ```
   - Verificar:
     - Que `install.sh` instala y registra `almundo-claude@almundo-claude`.
     - Que el plugin aparece en Claude Code con el nombre correcto.
     - Que los hooks de seguridad siguen activos y coherentes con las políticas.

---

## Fase 6 – Documentación y rollout interno

1. **Guía rápida para devs de Almundo**
   - Crear un documento interno (por ejemplo `docs/almundo-claude/USO_INTERNO.md`) con:
     - Cómo instalar el plugin (`curl | bash` apuntando al repo privado o instalación preconfigurada).
     - Comandos principales (`/alfred feature`, `/alfred fix`, etc.).
     - Limitaciones y responsabilidades (revisar siempre cambios, no usar en repos concretos, etc.).

2. **Proceso de adopción**
   - Empezar con un piloto en 1–2 equipos.
   - Recoger feedback y ajustar:
     - Mensajes de UX.
     - Niveles de autonomía por defecto.
     - Política de memoria persistente.

---

## Resumen rápido (checklist)

- [ ] Crear repo privado `almundo/almundo-claude` e importar `686f6c61/alfred-dev`.
- [ ] Configurar remoto `upstream` al repo público original.
- [ ] Crear rama base interna (`corp-main`) desde la última tag estable.
- [ ] Renombrar plugin a `almundo-claude` en `.claude-plugin/plugin.json` y `.claude-plugin/marketplace.json`.
- [ ] Actualizar `install.sh` (y `install.ps1` si aplica) con `REPO="almundo/almundo-claude"` y `PLUGIN_NAME="almundo-claude"`.
- [ ] Revisar y actualizar branding en `README.md` y `docs/`.
- [ ] Revisar y adaptar hooks de seguridad (`hooks/`).
- [ ] Definir política de memoria persistente y compliance internos.
- [ ] Establecer flujo de actualización desde `upstream` con versionado interno (`x.y.z-almundo.n`).
- [ ] Documentar uso interno y hacer rollout controlado.

---

## Fase 7 – Personalización avanzada (agentes, skills, hooks, memoria)

Esta fase no es necesaria para el primer rollout, pero marca el camino para alinear el plugin todavía más con la realidad de Almundo.

1. **Configuración de agentes**
   - Revisar los agentes de núcleo y opcionales en `agents/` y su documentación en `docs/agents/`.
   - Posibles acciones:
     - Ajustar descripciones, tono y ejemplos para que usen el contexto, productos y vocabulario de Almundo.
     - Desactivar por defecto agentes que no aporten valor en el contexto actual (vía configuración por proyecto).
     - Añadir agentes opcionales propios si la empresa tiene roles muy específicos (por ejemplo, un agente de “viajes” o de “cumplimiento sectorial”).

2. **Ajuste y creación de skills**
   - Explorar el catálogo en `skills/` y su documentación en `docs/skills.md`.
   - Posibles acciones:
     - Afinar prompts de skills existentes para reflejar estándares de código, guías de estilo y políticas de revisión internas.
     - Crear nuevas skills orientadas a procesos propios (por ejemplo, plantillas de PR, convenciones de microservicios, políticas de logging, etc.).
     - Documentar claramente qué skills deben usarse en qué tipos de proyectos.

3. **Hooks alineados a políticas internas**
   - Extender el trabajo de la Fase 4 para:
     - Incorporar nuevas reglas a `dangerous-command-guard.py` (por ejemplo, comandos prohibidos específicos de la infraestructura de Almundo).
     - Añadir patrones de secretos adicionales a `secret-guard.sh` y `sensitive-read-guard.py` conforme evolucionen los sistemas internos.
     - Usar `dependency-watch.py` para integrar, si se desea, con flujos de auditoría de dependencias propios (por ejemplo, marcar ciertas librerías como vetadas).

4. **Memoria y base de datos**
   - Revisar la estructura y opciones en `core/memory.py` y `docs/memory.md`.
   - Posibles líneas de evolución:
     - Definir convenciones de etiquetas y estados de decisiones acordes a la gestión de arquitectura de Almundo.
     - Explorar integraciones futuras (por ejemplo, export de decisiones a sistemas internos de documentación) manteniendo la base SQLite como fuente de verdad local.
     - Ajustar políticas de retención por defecto para que encajen con los criterios de compliance de la empresa.

5. **Plantillas y artefactos estándar**
   - Revisar `templates/` (PRD, ADR, test plan, threat model, SBOM, changelog, release notes).
   - Adaptar:
     - Terminología, secciones obligatorias y ejemplos a los formatos que ya use Almundo (por ejemplo, campos extra en ADR o PRD).
     - Reglas sobre cuándo cada plantilla es obligatoria (p.ej. SBOM siempre en proyectos que toquen producción).

6. **Roadmap interno**
   - Mantener un pequeño roadmap (puede ser un fichero `docs/almundo-claude/ROADMAP.md`) donde se prioricen:
     - Cambios de agentes/skills más urgentes.
     - Nuevos hooks o endurecimientos necesarios.
     - Integraciones deseadas con herramientas internas (trackers, documentación, etc.).



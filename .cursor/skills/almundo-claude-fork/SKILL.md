---
name: almundo-claude-fork
description: Gestiona el fork privado almundo-claude basado en alfred-dev, siguiendo el plan definido en docs/almundo-ia/PLAN.md (fork, rebranding, updates desde upstream).
---

# Gestión del fork privado almundo-claude

## Cuándo usar esta skill

Usa esta skill cuando:

- Haya que **crear o actualizar** el fork privado `almundo/almundo-claude` desde el upstream `686f6c61/alfred-dev`.
- Se revisen o apliquen pasos del plan de rebranding y hardening descrito en `docs/almundo-ia/PLAN.md`.
- Se definan estrategias de ramas, versionado interno o procesos de rollout interno de Almundo IA.

## Instrucciones

1. **Revisar el PLAN**
   - Lee siempre `docs/almundo-ia/PLAN.md` antes de proponer cambios de flujo.
   - Verifica que las ramas (`corp-main`, `main`, etc.) y los remotos (`origin`, `upstream`) siguen el esquema del plan.

2. **Actualizar desde upstream**
   - Usa el flujo recomendado en el PLAN (fetch desde `upstream`, merge a la rama base interna, resolución de conflictos).
   - Presta atención especial a:
     - `.claude-plugin/plugin.json`
     - `.claude-plugin/marketplace.json`
     - `install.sh` / `install.ps1`
     - `hooks/`
     - `README.md` y `docs/`

3. **Versionado interno**

- Aplica el esquema sugerido:
  - Upstream `x.y.z` → interno `x.y.z-almundo-claude.n`.
- Asegúrate de actualizar las versiones donde corresponda:
  - `.claude-plugin/plugin.json`
  - `.claude-plugin/marketplace.json`
  - Mensajes relevantes en scripts de instalación.

4. **Validación**

- Después de cambios relacionados con el fork o updates:
  - Ejecuta la batería de tests con `pytest` según describe el PLAN.
  - Comprueba que el plugin se registra como `almundo-claude@almundo-claude`.
  - Verifica que los hooks de seguridad siguen activos y alineados con las políticas de Almundo.

## Ejemplos de uso

- El usuario pide: “Actualiza nuestro fork almundo-claude a la última versión de alfred-dev” → Sigue los pasos del PLAN para fetch/merge y valida la integración.
- El usuario pide: “Revisa que el rebranding no se haya roto al traer cambios upstream” → Compara los manifiestos del plugin y scripts de instalación con lo definido en el PLAN.


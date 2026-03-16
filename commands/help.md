---
description: "Muestra los comandos disponibles de Almundo IA"
---

# Ayuda de Almundo IA

Muestra al usuario la siguiente tabla de comandos disponibles con descripción y ejemplos:

| Comando | Argumentos | Descripción |
|---------|-----------|-------------|
| `/almundo-ia:feature` | [descripción] | Ciclo completo: producto, arquitectura, desarrollo, QA, documentación, entrega |
| `/almundo-ia:fix` | [descripción] | Corrección de bugs: diagnóstico, corrección TDD, validación |
| `/almundo-ia:spike` | [tema] | Investigación técnica sin compromiso de implementación |
| `/almundo-ia:ship` | -- | Preparar entrega: auditoría, docs, empaquetado, despliegue |
| `/almundo-ia:audit` | -- | Auditoría completa con 4 agentes en paralelo |
| `/almundo-ia:config` | -- | Configurar autonomía, stack, agentes opcionales y personalidad |
| `/almundo-ia:status` | -- | Estado de la sesión activa |
| `/almundo-ia:update` | -- | Comprobar y aplicar actualizaciones del plugin |
| `/almundo-ia:help` | -- | Esta ayuda |

Además, al escribir `/almundo-ia:tutor` sin subcomando, Almundo IA actúa como asistente contextual: evalúa el estado del proyecto y la sesión, y dirige al usuario al flujo más adecuado.

Explica brevemente que Almundo IA es un equipo de **9 agentes de núcleo** (siempre activos) más **8 agentes opcionales** (activables según el proyecto) que cubren el ciclo completo de ingeniería de software, con quality gates y flujos automatizados.

### Agentes de núcleo

product-owner, architect, senior-dev, security-officer, qa-engineer, devops-engineer, tech-writer, project-manager y el Orquestador Almundo (`almundo-ia`) como orquestador.

### Agentes opcionales

Se activan con `/almundo-ia:config`. Almundo IA los sugiere automáticamente al analizar el proyecto:

| Agente | Cuándo es útil |
|--------|----------------|
| **data-engineer** | Proyectos con base de datos, ORM, migraciones |
| **ux-reviewer** | Proyectos con frontend |
| **performance-engineer** | Proyectos grandes o con requisitos de rendimiento |
| **github-manager** | Cualquier proyecto con repositorio GitHub |
| **seo-specialist** | Proyectos web con contenido público |
| **copywriter** | Proyectos con textos públicos |
| **librarian** | Proyectos con memoria persistente activa |
| **i18n-specialist** | Proyectos multiidioma o que necesitan traducción |

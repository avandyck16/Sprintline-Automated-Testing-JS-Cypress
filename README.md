> ### Confidentiality Notice
> This repository contains an anonymized portfolio case study based on a real-world project.
> 
> Company names, client information, URLs, credentials, proprietary business logic, and sensitive implementation details have been removed or modified to protect confidentiality.
> 
> Any code snippets included are examples only and do not represent any protected production codebase.

---
# SprintLine – E2E Automation Framework

Framework de automatización E2E para SprintLine | Cypress | JavaScript

### Proyecto

Automatización E2E para plataforma de generación de documentación y especificaciones técnicas asistida por IA.

---

### Objetivo

Implementar suites de automatización E2E orientadas a Build Validation Testing (BVT) para integrarse en pipelines de CI/CD y reducir el esfuerzo de validación manual previo al deployment mediante la validación automática de flujos críticos de negocio.

Diseñé e implementé una suite de automatización E2E con Cypress para SprintLine, una plataforma impulsada por IA que transforma ideas de producto en documentación técnica lista para implementación.

SprintLine permite a founders, Product Managers y equipos técnicos generar Product Requirements Documents (PRDs), User Stories con criterios de aceptación, especificaciones OpenAPI y alcances MVP mediante flujos guiados asistidos por IA.

La solución fue integrada al proceso de Build Validation Testing (BVT) para validar funcionalidades críticas antes del deployment y reducir significativamente el esfuerzo de validación manual. Las suites fueron diseñadas para ejecutarse en distintos navegadores y entornos, incluyendo validaciones de autenticación, cuestionarios dinámicos, generación de funcionalidades de producto y creación de documentos RFP (Request for Proposal).

---

## Stack

```txt
- Cypress
- JavaScript
- Mochawesome Reporter
- Azure DevOps
- Azure Static Web Apps
- GitHub
```

## Mi participación

```txt
→ Diseño de pruebas E2E para flujos críticos de negocio.
→ Desarrollo de specs y scripts utilizando Cypress y JavaScript.
→ Adaptación y estandarización de scripts de ejecución para distintos entornos y navegadores.
→ Implementación de comandos reutilizables para simplificar la ejecución y mantenimiento de pruebas.
→ Gestión de datos dinámicos para procesos de registro.
→ Incorporación de selectores en front end orientados a automatización.
→ Mantenimiento y optimización de suites.
→ Debugging y análisis de fallos cross-browser.
→ Integración del framework dentro del proceso de Build Validation Testing.
→ Generación automática de reportes HTML y JSON.
```

## Logros y Resultados

➜ 33 escenarios automatizados.

➜ Reducción del tiempo de validación de aproximadamente 50 minutos a un promedio de 9 minutos (~82%).

➜ 10 desafíos técnicos solucionados

* Compatibilidad con Chrome, Firefox, Edge y Electron.
* Cobertura de Smoke, Negative y Regression Testing.
* Validación automatizada de flujos de generación de documentación impulsados por IA.
* Validación automatizada del flujo de generación de documentos RFP.
* Implementación de reutilización de sesiones para optimizar tiempos de ejecución.
* Estabilización de pruebas mediante selectores dedicados y manejo de contenido dinámico.
* Generación automática de reportes HTML y JSON como artifacts del pipeline.

---

## Alcance de la Solución

| Suite                | Pruebas Cubiertas                                                                                                                                                                                              |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Smoke Tests**      | Login validation<br>Signup validation                                                                                                                                                                          |
| **Negative Tests**   | Invalid credentials<br>Invalid email validation<br>Required field validation<br>Password validation                                                                                                            |
| **Regression Tests** | Login regression validation & UI<br>Signup regression validation & UI<br>External links validation<br>Questionnaire flow validation<br>Product feature generation validation<br>RFP generation flow validation |

---

## Estructura de carpetas

```txt
sprintline-tests/
│
├── cypress/
│   ├── e2e/
│   │   ├── 1-smoke/
│   │   ├── 2-negative/
│   │   └── 3-regression/
│   │
│   ├── fixtures/
│   ├── reports/
│   └── support/
│
├── cypress.config.js
├── package.json
├── package-lock.json
```

---

# Test Organization

La suite de automatización está organizada en ámbitos de ejecución independientes:

| Folder         | Propósito                             |
| -------------- | ------------------------------------- |
| `1-smoke`      | Validaciones críticas del happy path  |
| `2-negative`   | Validación de errores y restricciones |
| `3-regression` | Cobertura funcional y UI extendida    |

---

# Desafíos Técnicos y Soluciones Implementadas

### Problema 1:

* * La ejecución repetitiva de autenticaciones incrementaba innecesariamente el tiempo total de ejecución.

### Solución

* Se implementó correctamente `cy.session()` para almacenar y reutilizar sesiones autenticadas entre pruebas, reduciendo tiempos de ejecución y carga sobre el servicio de autenticación.

Example Use Case:

```js
Cypress.Commands.add("login_cached", () => {
  cy.session(
    Cypress.env("USER"),
    () => {
      cy.login();
    },
    {
      validate() {
        cy.visit("/dashboard");
        cy.url().should("include", "/dashboard");
      },
    },
  );
});
```


---

### Problema 2:

* Firefox presentaba errores intermitentes relacionados con promesas no controladas y validación de sesiones restauradas.

### Solución

* Se incorporó un manejador global de excepciones para errores conocidos y se agregaron validaciones adicionales durante la restauración de sesiones para garantizar compatibilidad cross-browser específicamente con Firefox.

Example Use Case:
```js
Cypress.on("uncaught:exception", (err) => {
  // Ignore known flaky UI response issue in Firefox
  if (err.message.includes("Network connection")) {
    return false;
  }
});
```

---

### Problema 3:

* El flujo principal del cuestionario genera URLs dinámicas y depende de una sesión persistente que no puede reconstruirse parcialmente.

### Solución

* El flujo completo fue automatizado como una única prueba continua para preservar el estado de la sesión, mantener la continuidad del token generado y validar correctamente el recorrido end-to-end.

---

### Problema 4:

* Los procesos impulsados por IA generan contenido dinámicamente y requieren tiempos de procesamiento significativamente mayores a los esperados por defecto en Cypress.

### Solución

* Se ajustaron estrategias de espera y validaciones para sincronizar la ejecución con la respuesta real de la aplicación evitando falsos fallos por timeout prematuro.

---

### Problema 5:

* La generación de funcionalidades del producto y la creación del documento RFP pueden tardar aproximadamente 4.7 minutos en completarse.

### Solución

* Se implementaron timeouts extendidos y validaciones específicas que permiten esperar la generación completa del documento antes de ejecutar las verificaciones correspondientes.

---

### Problema 6:

* El backend podía encontrarse en estado inactivo tras periodos prolongados sin tráfico, ocasionando falsos positivos durante la ejecución.

### Solución:

Implementé un comando reutilizable que realiza una petición previa a la API utilizando credenciales inválidas para activar los servicios antes del inicio de la suite y garantizar condiciones consistentes de ejecución.

---

### Problema 7:

* Múltiples elementos carecían de selectores estables para automatización, generando pruebas frágiles y dependientes de cambios visuales.

### Solución

* Se incorporaron selectores dedicados orientados a automatización (`data-cy`) para mejorar la estabilidad y mantenibilidad de las pruebas.

---

### Problema 8:

* Los flujos de registro requerían usuarios únicos en cada ejecución para evitar conflictos por cuentas previamente creadas.

### Solución

* Se implementó generación dinámica de correos electrónicos y credenciales utilizando identificadores únicos basados en timestamp.

---

## Desafío Técnico 1: Estandarización de la ejecución de pruebas
🔹 Problema

La ejecución de pruebas requería diferentes configuraciones dependiendo del entorno, navegador y objetivo de ejecución (UI, consola o generación de reportes), lo que incrementaba la complejidad operativa y la posibilidad de errores manuales.

🔹 Solución

Se adaptaron scripts reutilizables para centralizar la configuración de variables de entorno y simplificar la ejecución de pruebas desde distintos contextos.

Se implementaron comandos específicos para:

→ Ejecución desde línea de comandos.
→ Ejecución mediante Cypress Test Runner.
→ Ejecución en ambiente Development.
→ Generación automática de reportes Mochawesome.
→ Ejecución cross-browser en Firefox y Edge.

🔹 Resultado

→ Menor esfuerzo de configuración manual.
→ Ejecución consistente entre entornos.
→ Mayor facilidad para nuevos miembros del equipo.
→ Reducción de errores operativos durante validaciones y debugging.

<img width="433" height="219" alt="image" src="https://github.com/user-attachments/assets/96d56884-3db5-4856-92ba-df020543ff91" />

---

## Desafío Técnico 2: Reutilización de acciones mediante Custom Commands
🔹 Problema

Las operaciones de autenticación y preparación de entornos se repetían constantemente entre múltiples escenarios, generando duplicación de código y aumentando el costo de mantenimiento.

🔹 Solución

Se implementaron comandos personalizados reutilizables para encapsular comportamientos frecuentes:

→ login para autenticación validada.
→ ping para mitigación de cold starts.
→ login_cached utilizando cy.session() para reutilización de sesiones autenticadas.

🔹 Resultado

→ Código más limpio y fácil de leer.
→ Menor duplicación de lógica.
→ Mayor mantenibilidad del framework.
→ Reducción de tiempos de ejecución mediante reutilización de sesiones.

---

## Pipeline

* La suite se integra con Azure DevOps para ejecutar validaciones automáticas durante la etapa de Build Validation Testing y generar artifacts accesibles directamente desde el pipeline.

---

## Reporte HTML

Los resultados de ejecución se generan automáticamente en formatos HTML y JSON mediante Mochawesome para facilitar el análisis de resultados y seguimiento histórico de ejecuciones.

---

## Conclusión

La implementación de esta suite permitió automatizar validaciones críticas relacionadas con autenticación, registro, cuestionarios dinámicos, generación de funcionalidades de producto y creación de documentos RFP dentro de SprintLine.

Además de reducir significativamente el tiempo de validación previo al deployment, la solución resolvió desafíos asociados con sesiones persistentes, URLs dinámicas, contenido generado por IA, compatibilidad cross-browser y optimización del tiempo de ejecución mediante reutilización de sesiones autenticadas.

La arquitectura fue diseñada para ser mantenible, escalable y adaptable a distintos modelos de CI/CD, permitiendo incorporar nuevas validaciones conforme evoluciona la plataforma.

---

## Ejemplos de integración

| Modelo                         | Flujo                                              |
| ------------------------------ | -------------------------------------------------- |
| Build Validation Testing (BVT) | Build → Test → Deploy                              |
| Post-Deployment Testing        | Build → Deploy → Test                              |
| Multi-Environment Validation   | Build Dev → Deploy Dev → Test → Deploy Prod → Test |
| Extended Validation Pipeline   | Build → Test → Deploy → Test Prod → Final Report   |

## Project Metadata

- **Author:** Axel Van Dyck
- **Keywords:** sprintline, Alset
- **License:** ISC
- **Module Type:** CommonJS (`type: "commonjs"`)

This project is maintained and authored by Axel Van Dyck as part of QA automation efforts for the Sprintline platform at Alset, Inc.


> ### Confidentiality Notice
>
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

La solución fue integrada al proceso de Build Validation Testing (BVT) para validar funcionalidades críticas antes del deployment y reducir significativamente el esfuerzo de validación manual.

La estrategia actual prioriza los flujos funcionales de mayor valor y mantiene una cobertura estable y repetible dentro del framework.

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
→ Adaptación y estandarización de scripts de ejecución para distintos entornos.
→ Implementación de comandos reutilizables para simplificar la ejecución y mantenimiento de pruebas.
→ Gestión de datos dinámicos para procesos de registro.
→ Incorporación de selectores en front end orientados a automatización.
→ Mantenimiento y optimización de suites.
→ Debugging y análisis de fallos cross-browser.
→ Implementación de preparación del entorno mediante wake-up/ping del API.
→ Integración del framework dentro del proceso de Build Validation Testing.
→ Generación automática de reportes HTML y JSON.
```

## Logros y Resultados

➜ **33 escenarios automatizados.**

➜ **Reducción del tiempo de validación de aproximadamente 50 minutos a un promedio de 9 minutos (~82%).**

➜ **10 desafíos técnicos solucionados** relacionados con estabilidad, ejecución, sesiones, contenido dinámico y CI/CD.

➜ Cobertura automatizada de los principales flujos funcionales de SprintLine:

* Login.
* Signup.
* Questionnaire.
* Product Definition.
* Generación y entrega del documento RFP.
* Validaciones negativas de autenticación y registro.

➜ Compatibilidad con **Chrome, Firefox, Edge y Electron**.

➜ Generación automática de reportes HTML y JSON como artifacts del pipeline.

➜ Implementación de reutilización de sesiones mediante `cy.session()` para optimizar tiempos de ejecución.

➜ Implementación de preparación automática del backend para mitigar problemas asociados con **cold starts**.

---

# Alcance de la Solución

La estrategia de automatización prioriza los flujos críticos y representativos del funcionamiento principal de SprintLine.

| Flujo funcional        | Automatización | Alcance                                              |
| ---------------------- | -------------- | ---------------------------------------------------- |
| **Login**              | Automatizado   | Flujo funcional y validaciones negativas             |
| **Signup**             | Automatizado   | Flujo funcional y validaciones negativas             |
| **Questionnaire**      | Automatizado   | Flujo principal                                      |
| **Product Definition** | Automatizado   | Validación de generación y entrega del documento RFP |

### Flujos automatizados

Se automatizaron los flujos considerados críticos para validar el funcionamiento principal de SprintLine:

* Login.
* Signup.
* Questionnaire.
* Product Definition.
* Generación y entrega del documento RFP.
* Validaciones funcionales y negativas asociadas a Login y Signup.

El **Questionnaire principal** puede ejecutarse utilizando el token gratuito disponible para una cuenta nueva, permitiendo validar el flujo sin requerir dependencias adicionales con el sistema de pagos.

Esto permite mantener una cobertura automatizada representativa y repetible dentro del framework.

---

# Test Organization

Como parte de la versión **1.0.1**, se consolidó la estructura de las pruebas bajo una organización más sencilla y consistente.

La estructura anterior estaba dividida en suites independientes de `Smoke`, `Regression` y `Negative`. Después del refactor, las validaciones se organizaron alrededor de los flujos funcionales principales.

### Estructura actual

```text
automated-tests/
└── cypress/
    └── e2e/
        └── Regression-Tests/
            ├── 1.login.cy.js
            ├── 2.login.negative
            ├── 3.signup
            ├── 4.signup.negative
            └── 5.prod.definition
```

Las validaciones negativas permanecen asociadas a sus respectivos flujos funcionales en lugar de mantener una separación estructural independiente.

### Beneficios del refactor

* Menor duplicación estructural.
* Localización más sencilla de los escenarios.
* Organización basada en flujos funcionales.
* Mayor consistencia para incorporar nuevas pruebas.
* Mantenimiento simplificado del framework.

> **Nota:** la versión 1.0.1 corresponde a un refactor de organización y mantenibilidad. No representa una modificación de la funcionalidad validada por las pruebas.

---

# Desafíos Técnicos y Soluciones Implementadas

### Problema 1: Reutilización de sesiones

* La ejecución repetitiva de autenticaciones incrementaba innecesariamente el tiempo total de ejecución.

### Solución

Se implementó correctamente `cy.session()` para almacenar y reutilizar sesiones autenticadas entre pruebas, reduciendo tiempos de ejecución y carga sobre el servicio de autenticación.

Example Use Case:

```js
Cypress.Commands.add("login_cached", () => {
  cy.session(
    Cypress.env("USER"),
    () => {
      cy.login(); [Descrito en "commands" para privacidad y legibilidad de código]
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

### Problema 2: Compatibilidad con Firefox

* Firefox presentaba errores intermitentes relacionados con promesas no controladas y validación de sesiones restauradas.
    * (Problemas de manejo de cy.session) 

### Solución

Se incorporó un manejador global de excepciones para errores conocidos y se agregaron validaciones adicionales durante la restauración de sesiones para garantizar compatibilidad cross-browser específicamente con Firefox.

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

### Problema 3: URLs dinámicas y sesiones persistentes

* El flujo principal del cuestionario genera URLs dinámicas y depende de una sesión persistente que no puede reconstruirse parcialmente.

### Solución

El flujo completo fue automatizado como una única prueba continua para preservar el estado de la sesión, mantener la continuidad del token generado y validar correctamente el recorrido end-to-end.

---
### Problema 4: Sincronización de procesos de generación mediante IA

* Los procesos de generación de contenido y documentos impulsados por IA son asíncronos y pueden requerir varios minutos para completarse, provocando falsos fallos cuando las validaciones se ejecutan antes de recibir la respuesta correspondiente.

### Solución

Inicialmente se implementó un intercept de las peticiones asociadas al proceso de generación para utilizar la respuesta del API como punto de sincronización de la prueba.

```js
cy.intercept("POST", "**/api/...").as("generateContent");

cy.get("[data-cy='generate-button']").click();

cy.wait("@generateContent")
  .its("response")
  .should("exist")
  .and("have.property", "statusCode", 200);
```

Esta estrategia permitió sincronizar las validaciones con la respuesta real del servicio en lugar de depender únicamente de los tiempos de espera predeterminados de Cypress.

Posteriormente, por requerimiento del proyecto, la estrategia fue simplificada para este flujo mediante un `wait` específico de **5 minutos**, considerando que la generación completa de funcionalidades y documentos podía extenderse hasta aproximadamente 4.7 minutos.

La espera extendida permitió establecer una ventana controlada para completar el proceso antes de ejecutar las validaciones finales, reduciendo falsos fallos asociados al tiempo de procesamiento.

---

### Problema 5: Cold starts del backend

* El backend podía encontrarse en estado inactivo tras periodos prolongados sin tráfico, ocasionando condiciones inconsistentes durante la ejecución.

### Solución

Se implementó un comando reutilizable que realiza una solicitud inicial al API antes de ejecutar las suites para activar los servicios y reducir problemas asociados con cold starts.

La operación se ejecuta mediante un `before` global en `support/e2e.js`, evitando la necesidad de incorporar esta preparación dentro de cada prueba individual.

El flujo de preparación es:

```text
Inicio de suite
      ↓
Wake-up / Ping API
      ↓
API disponible
      ↓
Ejecución de pruebas
```

La solicitud utiliza un escenario controlado y su finalidad es preparar el servicio para la ejecución de las pruebas, no validar una funcionalidad independiente.

---

### Problema 6: Selectores inestables

* Múltiples elementos carecían de selectores estables para automatización, generando pruebas frágiles y dependientes de cambios visuales.

### Solución

Se incorporaron selectores dedicados orientados a automatización mediante atributos `data-cy`, mejorando la estabilidad y mantenibilidad de las pruebas.

---

### Problema 7: Datos dinámicos para registro

* Los flujos de registro requerían usuarios únicos en cada ejecución para evitar conflictos por cuentas previamente creadas.

### Solución

Se implementó generación dinámica de correos electrónicos y credenciales utilizando identificadores únicos basados en timestamp.

```js
const cypress_email = `cypress_test${Date.now()}@yopmail.com`;

```

---

# Desafío Técnico: Estandarización de la ejecución de pruebas

### Problema

La ejecución de pruebas requería diferentes configuraciones dependiendo del entorno, navegador y objetivo de ejecución, lo que incrementaba la complejidad operativa y la posibilidad de errores manuales.

### Solución

Se adaptaron scripts reutilizables para centralizar la configuración de variables de entorno y simplificar la ejecución de pruebas desde distintos contextos.

Se implementaron comandos específicos para:

→ Ejecución desde línea de comandos.
→ Ejecución mediante Cypress Test Runner.
→ Ejecución en ambiente Development.
→ Ejecución en ambiente Production.
→ Generación automática de reportes Mochawesome.

### Resultado

→ Menor esfuerzo de configuración manual.
→ Ejecución consistente entre entornos.
→ Mayor facilidad para nuevos miembros del equipo.
→ Reducción de errores operativos durante validaciones y debugging.

---

# Desafío Técnico: Reutilización de acciones mediante Custom Commands

### Problema

Las operaciones de autenticación y preparación de entornos se repetían constantemente entre múltiples escenarios, generando duplicación de código y aumentando el costo de mantenimiento.

### Solución

Se implementaron comandos personalizados reutilizables para encapsular comportamientos frecuentes:

→ `login` para autenticación validada.
→ `ping` para mitigación de cold starts.
→ `login_cached` utilizando `cy.session()` para reutilización de sesiones autenticadas.

### Resultado

→ Código más limpio y fácil de leer.
→ Menor duplicación de lógica.
→ Mayor mantenibilidad del framework.
→ Reducción de tiempos de ejecución mediante reutilización de sesiones.

---

# Estrategia de Ejecución

SprintLine cuenta con comandos diferenciados para ejecutar las pruebas contra distintos ambientes.

| Script        | Descripción                                          |
| ------------- | ---------------------------------------------------- |
| `test:local`  | Ejecuta las suites contra una aplicación local       |
| `test:dev`    | Ejecuta las suites contra el ambiente de desarrollo  |
| `test:prod`   | Ejecuta las suites contra el ambiente de producción  |
| `debug:local` | Abre Cypress para debugging contra el ambiente local |

Las URLs, usuarios y demás valores específicos de configuración se administran mediante variables de entorno y no forman parte de la documentación de trazabilidad.

---

# Pipeline

Las pruebas Cypress están integradas con Azure DevOps como un **stage de validación posterior al build** dentro del proceso de Build Validation Testing.

El flujo general es:

```text
Build Application
       ↓
Publish Build Artifact
       ↓
Run Cypress Tests
       ↓
Publish Test Reports
       ↓
Publish Screenshots on Failure
       ↓
Continue Pipeline / Quality Gate
       ↓
Deploy
```

El stage de pruebas utiliza el artefacto generado durante el build y levanta la aplicación de manera local dentro del agente del pipeline.

El proceso contempla:

1. Instalación de Node.js.
2. Descarga del artefacto generado por el build.
3. Inicio de la aplicación mediante `serve`.
4. Espera de disponibilidad mediante `wait-on`.
5. Instalación de las dependencias de automatización.
6. Ejecución de Cypress.
7. Generación de reportes Mochawesome.
8. Publicación de reportes como artefactos.
9. Publicación de screenshots cuando existen fallos.
10. Publicación de resultados de pruebas.

---

# Estrategia de Validación Antes del Deployment

La estrategia actual ejecuta Cypress contra el **build de desarrollo servido localmente dentro del pipeline** antes de realizar cualquier deployment.

De esta manera, el artefacto generado durante el proceso de build es validado antes de ser promovido al ambiente correspondiente.

| Situación      | Build inicial | Cypress | Build PROD | Deploy |
| -------------- | ------------- | ------- | ---------- | ------ |
| PR → `dev`     | DEV           | DEV     | —          | —      |
| Merge → `dev`  | DEV           | DEV     | —          | DEV    |
| PR → `main`    | DEV           | DEV     | —          | —      |
| Merge → `main` | DEV           | DEV     | PROD       | PROD   |


Esta estrategia permite utilizar las pruebas E2E como una validación previa a la promoción del software hacia los ambientes desplegados.

---

# Reporting y Evidencia

La ejecución automatizada utiliza **Mochawesome** como reporter.

El pipeline genera y publica los resultados de ejecución como artefactos, permitiendo conservar evidencia de las pruebas ejecutadas.

La estrategia contempla:

* Reportes HTML.
* Reportes JSON.
* Timestamp de ejecución.
* Directorio centralizado de reportes.
* Screenshots de ejecuciones con errores.
* Resultados de pruebas publicados dentro del pipeline.

Los screenshots se publican como artefactos únicamente cuando la ejecución presenta fallos, facilitando el análisis posterior de errores.

---

# Matriz de Cobertura

| Área                                    | Cobertura    |
| --------------------------------------- | ------------ |
| Autenticación                           | Automatizada |
| Registro de usuario                     | Automatizada |
| Questionnaire principal                 | Automatizada |
| Product Definition                      | Automatizada |
| Generación / entrega de documento RFP   | Automatizada |
| Validaciones negativas de autenticación | Automatizada |
| Validaciones negativas de registro      | Automatizada |

---

# Alcance Actual

La automatización de SprintLine está orientada a proporcionar una **cobertura funcional representativa de los flujos críticos**, priorizando escenarios que puedan ejecutarse de manera estable y repetible dentro del framework.

El alcance actual permite validar:

* Acceso de usuarios.
* Registro de usuarios.
* Ejecución del Questionnaire principal.
* Flujo de Product Definition.
* Generación y entrega del documento RFP.
* Escenarios negativos de autenticación y registro.
* Integridad del flujo automatizado antes del deployment.

---

# Conclusión

La versión **SprintLine 1.0.1** establece una reorganización del framework de automatización orientada a mejorar su mantenibilidad, claridad y consistencia estructural, manteniendo la intención funcional de las pruebas existentes.

La estrategia mantiene automatizados los flujos críticos de SprintLine, incluyendo autenticación, registro, Questionnaire y Product Definition, además de la generación y entrega del documento RFP.

La integración con Azure DevOps permite ejecutar las pruebas sobre el artefacto generado durante el build, conservar evidencia de ejecución y utilizar la validación E2E como parte del proceso previo al deployment.

**Resultado:** la suite proporciona cobertura automatizada sobre los flujos funcionales de mayor valor para SprintLine, con una estructura más consistente, ejecución reproducible y capacidad de integrarse directamente dentro del flujo CI/CD.

---

## Ejemplos de Integración

| Modelo                         | Flujo                                              |
| ------------------------------ | -------------------------------------------------- |
| Build Validation Testing (BVT) | Build → Test → Deploy                              |
| Post-Deployment Testing        | Build → Deploy → Test                              |
| Multi-Environment Validation   | Build Dev → Deploy Dev → Test → Deploy Prod → Test |
| Extended Validation Pipeline   | Build → Test → Deploy → Test Prod → Final Report   |

---

<div style="text-align:center; color:#888; font-size:10px;">
    QA Documentation | Axel Van Dyck | QA Engineer 
</div>

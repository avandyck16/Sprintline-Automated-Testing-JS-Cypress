# Confidentiality Notice

> This repository contains an anonymized portfolio case study based on a real-world project.
>
> Company names, client information, URLs, credentials, proprietary business logic, and sensitive implementation details have been removed or modified to protect confidentiality.
>
> Any code snippets included are examples only and do not represent any protected production codebase.

---

# SprintLine — E2E Automation Framework

**AI-Powered Technical Documentation Platform | Cypress | JavaScript | Azure DevOps | CI/CD**

E2E automation framework designed to validate critical workflows within an AI-powered platform that transforms product ideas into implementation-ready technical documentation and specifications.

---

## Quick Scope

| Area                    | Details                            |
| ----------------------- | ---------------------------------- |
| **Role**                | QA Engineer                        |
| **Testing**             | E2E · Regression · Negative        |
| **Automation**          | Cypress · JavaScript               |
| **Reporting**           | Mochawesome                        |
| **CI/CD**               | Azure DevOps Pipelines             |
| **Environments**        | Local · Development · Production   |
| **Browsers**            | Chrome · Firefox · Edge · Electron |
| **Automated Scenarios** | 33                                 |

### Key Results

* **~82% reduction** in validation time: ~50 min → ~9 min.
* **33 automated scenarios** covering critical SprintLine workflows.
* Cross-browser execution across **Chrome, Firefox, Edge, and Electron**.
* Automated HTML and JSON reporting as pipeline artifacts.
* **10 technical challenges** addressed involving sessions, AI-generated content, dynamic data, browser compatibility, backend availability, and CI/CD.
* Implemented `cy.session()` to reduce authentication overhead.
* Implemented API wake-up logic to mitigate backend cold starts.
* Integrated E2E validation into Azure DevOps Build Validation Testing.

---

## Case Study Index

* [Project Overview](#project-overview)
* [QA Strategy](#qa-strategy)
* [Automation Scope & Decisions](#automation-scope--decisions)
* [Technical Implementation](#technical-implementation)
* [Test Architecture](#test-architecture)
* [Execution & Environment Strategy](#execution--environment-strategy)
* [CI/CD & Reporting](#cicd--reporting)
* [Results](#results)
* [Limitations & Known Issues](#limitations--known-issues)

---

# Project Overview

## Context

SprintLine is an AI-powered platform that transforms product ideas into technical documentation and implementation-ready specifications.

The platform supports workflows for generating Product Requirements Documents (PRDs), User Stories with acceptance criteria, OpenAPI specifications, and MVP definitions through guided AI-assisted processes.

## Objective

Design and implement an E2E automation framework focused on critical business workflows and integrate it into Build Validation Testing (BVT) to reduce manual validation effort before deployment.

The strategy prioritized high-value workflows that could be executed consistently and provide representative coverage of the application's core functionality.

## My Role

* Designed E2E test coverage for critical business workflows.
* Developed and maintained Cypress/JavaScript specifications.
* Standardized execution scripts across environments.
* Implemented reusable Cypress custom commands.
* Implemented dynamic test data generation.
* Added automation-oriented frontend selectors.
* Investigated and resolved cross-browser failures.
* Implemented session reuse through `cy.session()`.
* Implemented backend wake-up logic to mitigate cold starts.
* Integrated the framework into Azure DevOps BVT.
* Configured automated HTML and JSON reporting.
* Maintained and optimized the test suites.

## Technology Stack

* Cypress
* JavaScript
* Azure DevOps
* YAML Pipelines
* Mochawesome Reporter
* Azure Static Web Apps
* GitHub

---

# QA Strategy

The automation strategy prioritizes workflows that are:

* Business-critical.
* Representative of the core product experience.
* Repetitive and suitable for automation.
* Stable enough for repeatable execution.
* Valuable for pre-deployment validation.

The framework combines functional and negative scenarios while also validating longer end-to-end workflows involving persistent sessions, dynamic URLs, AI-generated content, and document generation.

---

# Automation Scope & Decisions

| Flow                            | Decision   | Coverage                                             |
| ------------------------------- | ---------- | ---------------------------------------------------- |
| Login                           | ✅ Automate | Functional + Negative                                |
| Signup                          | ✅ Automate | Functional + Negative                                |
| Questionnaire                   | ✅ Automate | Main E2E workflow                                    |
| Product Definition              | ✅ Automate | Core workflow                                        |
| RFP Generation & Delivery       | ✅ Automate | Document generation and delivery                     |
| Payment-dependent Questionnaire | ⚠️ Limited | Free-token flow used to maintain repeatable coverage |

The selected flows provide representative coverage of SprintLine's highest-value functional workflows while avoiding unnecessary dependencies that could reduce execution reliability.

---

# Technical Implementation

## Session Reuse

Repeated authentication increased the total execution time and added unnecessary load to the authentication service.

**Implementation:** Implemented `cy.session()` to cache and validate authenticated sessions between tests.

```javascript
Cypress.Commands.add("login_cached", () => {
  cy.session(
    Cypress.env("USER"),
    () => {
      cy.login(); // Described in commands for privacy and readability
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

**Result:** Reduced repeated authentication overhead and improved overall execution time.

---

## Firefox Compatibility

Firefox presented intermittent issues involving uncaught promises and restored session validation.

**Implementation:** Added targeted exception handling for known browser-specific behavior and additional session validation during restoration.

```javascript
Cypress.on("uncaught:exception", (err) => {
  // Ignore known flaky UI response issue in Firefox
  if (err.message.includes("Network connection")) {
    return false;
  }
});
```

**Result:** Improved cross-browser stability while maintaining execution across Chrome, Firefox, Edge, and Electron.

---

## Dynamic URLs & Persistent Sessions

The main questionnaire workflow generates dynamic URLs and depends on persistent session state that cannot be partially reconstructed.

**Implementation:** Automated the complete questionnaire as a continuous E2E scenario, preserving session state and validating the workflow sequentially.

**Result:** Maintained end-to-end context and prevented failures caused by splitting dependent steps across independent tests.

---

## AI Content Generation

AI-driven content and document generation can take several minutes to complete, causing premature Cypress failures when relying on default synchronization.

**Implementation:** Initially synchronized the test with the corresponding API response using `cy.intercept()`.

```javascript
cy.intercept("POST", "**/api/...").as("generateContent");

cy.get("[data-cy='generate-button']").click();

cy.wait("@generateContent")
  .its("response")
  .should("exist")
  .and("have.property", "statusCode", 200);
```

For the final implementation, the project required a controlled **5-minute timeout** because complete feature and document generation could take approximately 4.7 minutes.

**Result:** Reduced false failures caused by AI processing time and provided a controlled execution window for long-running workflows.

---

## Backend Cold Starts

The backend could become inactive after prolonged periods without traffic, creating inconsistent initial conditions during test execution.

**Implementation:** Added a reusable API wake-up command executed globally before the suite.

```text
Suite Start
    ↓
API Wake-up / Ping
    ↓
Backend Available
    ↓
Test Execution
```

The request is used solely to initialize the service and is not treated as an independent functional validation.

**Result:** Reduced false failures caused by backend cold starts.

---

## Automation-Oriented Selectors

Several UI elements lacked stable selectors, making tests dependent on visual or structural changes.

**Implementation:** Added dedicated `data-cy` selectors to improve automation reliability.

**Result:** Improved test stability and reduced maintenance caused by UI changes.

---

## Dynamic Registration Data

Registration scenarios required unique users on every execution to prevent conflicts with previously created accounts.

**Implementation:** Generated unique email addresses dynamically using execution timestamps.

```javascript
const cypress_email = `cypress_test${Date.now()}@yopmail.com`;
```

**Result:** Enabled repeatable registration testing without duplicate-account conflicts.

---

## Reusable Custom Commands

Authentication and environment preparation operations were repeated across multiple scenarios.

**Implementation:** Encapsulated recurring behavior into reusable commands, including:

* `login`
* `login_cached`
* API wake-up / ping

**Result:** Reduced duplicated logic and improved framework maintainability.

---

# Test Architecture

As part of version **1.0.1**, the framework was reorganized around functional workflows rather than maintaining separate structural suites for Smoke, Regression, and Negative testing.

```text
automated-tests
└── cypress
    └── e2e
        └── Regression-Tests
            ├── 1.login.cy.js
            ├── 2.login.negative.cy.js
            ├── 3.signup.cy.js
            ├── 4.signup.negative.cy.js
            └── 5.prod.definition.cy.js
```

Negative scenarios remain associated with their corresponding functional workflows rather than being maintained in a separate directory.

### Refactor Benefits

* Reduced structural duplication.
* Easier scenario discovery.
* Organization based on functional workflows.
* Simplified maintenance.
* Consistent structure for future test expansion.

> Version 1.0.1 represents a framework organization and maintainability refactor rather than a change to the functionality being validated.

---

# Execution & Environment Strategy

The framework provides dedicated scripts for execution against different environments.

| Script        | Purpose                                  |
| ------------- | ---------------------------------------- |
| `test:local`  | Execute suites against localhost         |
| `test:dev`    | Execute suites against Development       |
| `test:prod`   | Execute suites against Production        |
| `debug:local` | Open Cypress interactively for debugging |

Environment-specific URLs, users, credentials, and configuration values are managed through environment variables and are not stored in the repository.

---

# CI/CD & Reporting

## Pipeline Strategy

The Cypress framework is integrated into Azure DevOps as part of the Build Validation Testing process.

The target branch determines which application build is generated and validated.

### Development Flow

```text
Merge → dev
   ↓
Build DEV
   ↓
Deploy to Test Environment
   ↓
Execute Cypress Tests
   ↓
Tests Passed?
   ├── No  → Pipeline Stops
   └── Yes → Deploy to DEV
```

### Production Flow

```text
Merge → main
   ↓
Build PROD
   ↓
Deploy to Test Environment
   ↓
Execute Cypress Tests
   ↓
Tests Passed?
   ├── No  → Pipeline Stops
   └── Yes → Deploy to PROD
```

This approach ensures that the build intended for the target environment is validated before the final deployment.

A failed Cypress execution prevents the corresponding deployment from continuing.

## Automated Reporting

Mochawesome is used to generate execution reports.

The pipeline publishes:

* HTML reports.
* JSON reports.
* Failure screenshots.
* Test execution artifacts.

This provides persistent execution evidence for post-run analysis and debugging.

---

# Results

| Metric                         |   Result |
| ------------------------------ | -------: |
| Automated scenarios            |   **33** |
| Manual validation time         |  ~50 min |
| Automated execution            |   ~9 min |
| Time reduction                 | **~82%** |
| Supported browsers             |    **4** |
| Technical challenges addressed |   **10** |

### Functional Coverage

* Login.
* Signup.
* Negative authentication scenarios.
* Negative registration scenarios.
* Questionnaire.
* Product Definition.
* RFP generation and delivery.
* AI-generated content validation.

### Additional Improvements

* Reduced authentication overhead through session reuse.
* Improved cross-browser compatibility.
* Reduced false failures caused by backend cold starts.
* Improved synchronization of AI-driven workflows.
* Added stable automation selectors.
* Enabled repeatable dynamic registration.
* Integrated automated reporting into CI/CD.
* Simplified framework organization through the 1.0.1 refactor.

---

# Limitations & Known Issues

* AI-generated workflows can require several minutes to complete.
* Backend availability may require an initial wake-up request after extended inactivity.
* Some scenarios depend on baseline environment configuration.
* AI-generated content can vary between executions while remaining functionally valid.
* The questionnaire workflow depends on persistent session state and therefore remains intentionally continuous.
* The current automation scope prioritizes high-value workflows rather than complete application coverage.

---

# Conclusion

SprintLine's E2E automation framework provides representative automated coverage of the platform's highest-value workflows, including authentication, registration, questionnaire processing, product definition, and AI-driven document generation.

The framework reduced validation time by approximately **82%**, while expanding execution across four browsers and integrating automated validation directly into the CI/CD process.

The implementation also addressed several challenges specific to modern AI-powered applications, including long-running asynchronous operations, dynamic content, persistent sessions, backend cold starts, and cross-browser behavior.

The **1.0.1 refactor** further simplified the framework structure, improving maintainability and making future test expansion more consistent.

---

<div style="text-align:center; color:#888; font-size:10px;">
    QA Documentation | Axel Van Dyck | QA Engineer 
</div>

# Confidentiality Notice

> This repository contains an anonymized portfolio case study based on a real-world project.
>
> Company names, client information, URLs, credentials, proprietary business logic, and sensitive implementation details have been removed or modified to protect confidentiality.
>
> Any code snippets included are examples only and do not represent any protected production codebase.

---

# SprintLine — E2E Automation Framework

**AI-Powered Technical Documentation Platform | Cypress | JavaScript | Azure DevOps | CI/CD**

E2E automation framework for validating critical workflows in SprintLine, an AI-powered platform that generates technical documentation and implementation-ready specifications.

- **~82% less validation time:** from ~50 minutes to ~9 minutes.
- **33 automated scenarios** covering functional and negative workflows.
- **4 browsers:** Chrome, Firefox, Edge, and Electron.
- **Azure DevOps quality gate** running E2E tests before deployment.

## Quick Scope

| Category | Details |
|---|---|
| Role | QA Engineer |
| Testing | E2E, Regression, Negative |
| Automation | Cypress, JavaScript |
| Reporting | Mochawesome |
| CI/CD | Azure DevOps Pipelines |
| Environments | Local, Development, Production |
| Automated Scenarios | 33 |

## 1. Project Overview

**Objective:** Automate high-value user journeys and integrate E2E validation into the Azure DevOps delivery pipeline.

As the QA Engineer responsible for the framework, I designed and maintained Cypress tests, developed reusable helpers, addressed session and synchronization issues, and integrated execution and reporting into CI/CD.

**Technology Stack:** Cypress · JavaScript · Azure DevOps · YAML · Mochawesome · Azure Static Web Apps · GitHub

## 2. QA Strategy and Automation Scope

The suite prioritizes repetitive, business-critical workflows suitable for repeatable pre-deployment validation.

| Workflow | Coverage |
|---|---|
| Login | Form validation, navigation, authentication, logout, negative cases |
| Signup | Registration and negative cases |
| Questionnaire | Main guided workflow |
| Product Definition | Core product-definition workflow |
| RFP Generation and Delivery | Representative generation and delivery flow |
| Payment-dependent Questionnaire | Limited coverage; free-token flow retained as a manual scenario by project-lead decision |

The framework provides representative coverage, not complete application or payment-processing coverage.

## 3. Test Architecture

Tests are organized by functional workflow, with negative cases alongside their related areas.

```text
automated-tests/
└── cypress/e2e/Regression-Tests/
    ├── 1.login.cy.js
    ├── 2.login.negative.cy.js
    ├── 3.signup.cy.js
    ├── 4.signup.negative.cy.js
    └── 5.prod.definition.cy.js
```

**Version 1.0.1 refactor:** Reorganized the suite to improve test discovery, maintainability, and future expansion without claiming additional functional coverage from the refactor itself.

## 4. Execution and Environment Strategy

NPM scripts support local execution, development and production runs, and interactive debugging. `CYPRESS_BASE_URL` configures the application URL, while Cypress environment variables supply credentials and API settings.

| Script | Purpose |
|---|---|
| `test:local` | Run against the local application |
| `test:dev` / `test:prod` | Run against the configured environment |
| `debug:local` | Open the interactive Cypress runner |


## 5. Helpers I Implemented

### `login_cached()` — Session Reuse

Uses Cypress `cy.session()` to cache an authenticated session and validate dashboard access, avoiding unnecessary repeated logins.

```javascript
Cypress.Commands.add("login_cached", () => {
  cy.session(
    Cypress.env("USER"),
    () => {
      cy.visit("/");
      cy.get("#email").type(Cypress.env("USER"));
      cy.get("#password").type(Cypress.env("PASSWORD"));
      cy.get('[data-cy="sign-in-btn"]').click();
      cy.url({ timeout: 10000 }).should("include", "/dashboard");
      cy.contains("Welcome to Sprintline");
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

### `pingAPI()` — Backend Wake-up Request

Sends a POST request to the API login endpoint using deliberately invalid credentials. The helper logs the response status and accepts specific HTTP responses.

```javascript
Cypress.Commands.add("pingAPI", () => {
  const API_URL = Cypress.env("API_URL");
  cy.log(`API URL: ${API_URL}`);

  cy.request({
    method: "POST",
    url: API_URL + "/api/connect/login",
    failOnStatusCode: false,
    body: {
      username: "invalid@yopmail.com",
      password: "invalidpass",
      secured: false,
    },
  }).then((response) => {
    cy.log(`Wake-up request status: ${response.status}`);
    expect(response.status).to.be.oneOf([200, 400, 401, 405]);
  });
});
```

The command is invoked from the E2E setup hook before the suite begins:

```javascript
before(() => {
  cy.pingAPI();
});
```

**Purpose:** Generate an initial request to help reduce failures associated with a backend that has been inactive.

**Limitation:** An accepted HTTP status does not guarantee that the API is fully ready for every subsequent workflow.

### `manage_exceptions()` — Known Exception Handling (Firefox)

Registers a new command for Cypress exception handler for errors whose messages contain `postMessage` when testing on Firefox.

```javascript
Cypress.Commands.add("manage_exceptions", () => {
  Cypress.on("uncaught:exception", (err) => {
    if (err.message.includes("postMessage")) {
      return false;
    }
  });
});
```

The E2E setup contains an exception handler for a known `Network connection` issue:

```javascript
Cypress.on("uncaught:exception", (err) => {
  if (err.message.includes("Network connection")) {
    return false;
  }
});
```

**Purpose:** Prevent that specific known exception in Firefox from interrupting test execution when the helper is used.



These handlers target specific error messages; they do not correct the underlying application errors or suppress every uncaught exception.

## 6. Technical Implementation

| Challenge | Solution and result |
|---|---|
| Repeated authentication | Implemented session caching with `cy.session()` and dashboard validation. |
| Dynamic Questionnaire URLs | Kept dependent workflow steps together to preserve session and navigation context. |
| Slow AI generation | Used `cy.intercept()` to synchronize with API activity and adjusted timeouts for generation that could take approximately 4.7 minutes. |
| Backend cold starts | Added an initial API request through a reusable helper and suite-level `before()` hook. |
| Cross-browser exception | Filtered a known `Network connection` exception affecting Firefox execution. |
| Fragile selectors | Used automation-oriented `data-cy` attributes for important controls. |
| Duplicate registrations | Generated timestamp-based email addresses to reduce account conflicts. |

### AI Content Generation — API Synchronization

```javascript
cy.intercept("POST", "**/api/...").as("generateContent");

cy.get("[data-cy='generate-button']").click();

cy.wait("@generateContent")
  .its("response")
  .should("exist")
  .and("have.property", "statusCode", 200);
```

This verifies the request response; it does not independently confirm the correctness of all generated content.

### Dynamic Registration Data

**Challenge:** Reusing an email address can cause registration tests to fail because the account already exists.

**Solution:** Generated unique email values using a timestamp.

```javascript
const cypress_email = `cypress_test${Date.now()}@yopmail.com`;
```

**Result:** Reduces duplicate-account conflicts between test executions.

## 7. CI/CD and Reporting

The Cypress suite is integrated into Azure DevOps Pipelines as a pre-deployment quality gate.

### Pipeline Flow

```text
Push or merge to dev or main
              |
              v
     Build application
      (branch-specific)
              |
              v
      Publish vite-dist
              |
              v
     Download build artifact
              |
              v
 Serve artifact on localhost:8080
              |
              v
       Run Cypress tests
              |
              v
        Tests passed?
         /         \
       No           Yes
       |             |
       v             v
 Deployment      Deploy the same
 blocked         tested artifact
                     |
               dev → DEV
               main → PROD
```

### How It Works

1. **Branch-aware build:** The pipeline runs the appropriate Vite build for `dev` or `main`.
2. **Build artifact:** The generated application is published as `vite-dist`.
3. **Pre-deployment testing:** The pipeline downloads the artifact and serves it locally on the build agent at `localhost:8080`.
4. **Cypress execution:** The test stage runs against the built application using the corresponding configured test credentials and API settings.
5. **Deployment gate:** The final deployment stage runs only when the preceding stages succeed.
6. **Environment-specific deployment:** Successful runs deploy the same tested artifact to the destination associated with the branch: DEV for `dev`, PROD for `main`.

The tests run against the build artifact served locally on the pipeline agent. The pipeline does not first deploy the artifact to a separate test website before running Cypress.

### Reporting and Evidence

The pipeline publishes test execution evidence, including:

- Mochawesome HTML and JSON report artifacts.
- Cypress screenshots when test execution fails.
- Test result publication configured through Azure DevOps.

## 8. Results and Limitations

| Metric | Result |
|---|---|
| Automated scenarios | 33 |
| Manual validation | ~50 minutes |
| Automated validation | ~9 minutes |
| Time reduction | ~82% |
| Browser coverage | Chrome, Firefox, Edge, Electron |
| Delivery integration | Pre-deployment E2E quality gate |

The framework provides repeatable validation of selected critical workflows and preserves execution evidence in CI/CD.

**Known limitations:** AI generation can take several minutes and produce variable content; some workflows depend on session continuity; the API wake-up request is not a comprehensive health check; and payment-dependent coverage remains limited to the agreed manual scenario.

---

<div style="text-align:center; color:#888; font-size:10px;">
    QA Documentation | Axel Van Dyck | QA Engineer 
</div>

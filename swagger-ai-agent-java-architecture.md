# Swagger AI Agent – Java + RestAssured Master Architecture Specification

## Project Overview

I am building an **Enterprise-grade Swagger AI Agent**, API-first, using **Java 17 (Java 11 compatible where possible)** with **Spring Boot** and **RestAssured**.

This system will:

- Accept **Swagger / OpenAPI (2.0 & 3.x)** documents from:
  - URL  
  - File upload  
  - Git repository (branch/tag, path)
- Parse, validate, and normalize specs into a **common internal model**.
- Allow a user/agent to:
  - List endpoints / tags
  - Select one / multiple / all endpoints
  - Configure named environments (dev/qa/stage/prod)
- Execute HTTP calls using **RestAssured** as the HTTP engine (no Node/Axios).
- Generate **plain RestAssured + TestNG** test code from the spec and templates:
  - Simple, framework-light style (no heavy custom framework).
  - `@BeforeClass` / `@AfterClass` (or `@BeforeSuite` / `@AfterSuite`) for setup/teardown.
- Optionally generate:
  - **Cucumber feature files** (`.feature`) and
  - **Cucumber step definition classes** wired to RestAssured calls.
- Expose **MCP tools** (Java-based MCP server) so an AI agent can:
  - Discover operations
  - Plan runs
  - Execute API operations via RestAssured
  - Generate or refine RestAssured + TestNG suites

Support future extensions:

- **DB-backed storage** (specs, environments, run history, reports)
- **Additional LLM providers** for payload synthesis and test idea generation
- **Web UI (React)** on top of the same APIs
- Export of **Postman collections** and **CI-ready RestAssured + TestNG suites**

This is an **API-first**, clean architecture service, modular and testable, following enterprise conventions.  
It is **not** a UI project in phase 1.

Target stack:

- Java 17 (code should remain Java 11–compatible where feasible)
- Spring Boot (REST APIs, DI, configuration)
- RestAssured 5.x (HTTP execution + generated tests)
- TestNG (generated test suites)
- Cucumber-JVM (generated feature files + step definitions)

---

## Development Philosophy (Enterprise-Grade Modular, Java)

Copilot must follow these rules:

- Implement **one file at a time**.
- Implement **one module at a time**.
- Never mix responsibilities in a single class.
- Application logic should be **pure functions** wherever possible (static or instance methods with clear inputs/outputs).

### Domain Layer

- Pure Java classes and interfaces.
- **Zero external dependencies** (no Spring, no RestAssured, no HTTP, no LLM, no DB).
- Only business concepts and rules.

### Infrastructure Layer

- No business logic.
- Only adapters to:
  - HTTP client (RestAssured-based execution)
  - Swagger/OpenAPI parser
  - MCP server
  - LLM client
  - Persistence (in-memory now, DB later)
  - Logging

### API Layer (Controllers / DTOs / Validators)

- Spring @RestController classes.
- Thin controllers:
  - Validate inputs
  - Map to DTOs
  - Call use cases
  - Map responses
- No orchestration / algorithmic logic inside controllers.

### Application Layer (Use Cases)

- All “how to do it” workflows live in **`application`** package.
- Controllers call **one use case per endpoint**.
- Use cases depend on **domain models** and **repository interfaces**.
- Use cases use infrastructure adapters via injected interfaces.

### MCP Integration Rules

- Each MCP tool gets its own class (per logical tool).
- Adding a new MCP server (e.g. another execution engine) must plug in without breaking existing architecture.
- MCP adapter must work through the same **application/use cases** and **domain models**, not bypass them.

---

## Exact Maven / Package Structure (Swagger AI Agent – Java)

You MUST follow this structure for this project (package names aligned with Swagger/API domain, but same layering pattern):

```text
swagger-ai-agent-java/
├─ pom.xml
├─ README.md
├─ config/
│  ├─ application-dev.yml
│  ├─ application-test.yml
│  ├─ application-prod.yml
└─ src/
   ├─ main/
   │  ├─ java/
   │  │  └─ com/example/swaggerai/
   │  │     ├─ core/
   │  │     │  ├─ SwaggerAiApplication.java
   │  │     │  ├─ SpringConfig.java
   │  │     │  ├─ EnvironmentConfig.java
   │  │     │  ├─ ErrorHandlingConfig.java
   │  │     │  └─ exception/
   │  │     │     ├─ AppException.java
   │  │     │     ├─ ValidationException.java
   │  │     │     ├─ NotFoundException.java
   │  │     │     ├─ ExternalServiceException.java
   │  │     │     └─ UnauthorizedException.java
   │  │     │
   │  │     ├─ api/
   │  │     │  ├─ controller/
   │  │     │  │  ├─ SpecController.java
   │  │     │  │  ├─ EnvironmentController.java
   │  │     │  │  ├─ ExecutionController.java
   │  │     │  │  ├─ TestGenerationController.java
   │  │     │  │  └─ mcp/
   │  │     │  │     ├─ SwaggerMcpController.java
   │  │     │  │     └─ GenericMcpController.java
   │  │     │  ├─ dto/
   │  │     │  │  ├─ SpecDto.java
   │  │     │  │  ├─ EnvironmentDto.java
   │  │     │  │  ├─ ExecutionDto.java
   │  │     │  │  └─ TestGenerationDto.java
   │  │     │  └─ validator/
   │  │     │     ├─ SpecRequestValidator.java
   │  │     │     ├─ EnvironmentRequestValidator.java
   │  │     │     ├─ ExecutionRequestValidator.java
   │  │     │     └─ TestGenerationRequestValidator.java
   │  │     │
   │  │     ├─ application/
   │  │     │  ├─ spec/
   │  │     │  │  ├─ IngestSwaggerUseCase.java
   │  │     │  │  ├─ NormalizeSpecUseCase.java
   │  │     │  │  ├─ ValidateSpecUseCase.java
   │  │     │  │  └─ ListOperationsUseCase.java
   │  │     │  ├─ environment/
   │  │     │  │  ├─ CreateEnvironmentUseCase.java
   │  │     │  │  ├─ ListEnvironmentsUseCase.java
   │  │     │  │  ├─ UpdateEnvironmentUseCase.java
   │  │     │  │  └─ DeleteEnvironmentUseCase.java
   │  │     │  ├─ execution/
   │  │     │  │  ├─ PlanRunUseCase.java
   │  │     │  │  ├─ ExecuteRunUseCase.java
   │  │     │  │  ├─ GetRunStatusUseCase.java
   │  │     │  │  └─ RetryFailedTestUseCase.java
   │  │     │  ├─ testgen/
   │  │     │  │  ├─ GenerateRestAssuredTestsUseCase.java
   │  │     │  │  └─ ExportTestSuiteUseCase.java
   │  │     │  └─ llm/
   │  │     │     ├─ BuildPayloadFromSchemaUseCase.java
   │  │     │     └─ SuggestAdditionalTestsUseCase.java
   │  │     │
   │  │     ├─ domain/
   │  │     │  ├─ model/
   │  │     │  │  ├─ NormalizedSpec.java
   │  │     │  │  ├─ Operation.java
   │  │     │  │  ├─ EnvironmentConfigModel.java
   │  │     │  │  ├─ RunPlan.java
   │  │     │  │  ├─ RunReport.java
   │  │     │  │  ├─ TestCaseDefinition.java
   │  │     │  │  └─ PayloadTemplate.java
   │  │     │  └─ repository/
   │  │     │     ├─ SpecRepository.java
   │  │     │     ├─ EnvironmentRepository.java
   │  │     │     ├─ RunPlanRepository.java
   │  │     │     └─ TestTemplateRepository.java
   │  │     │
   │  │     ├─ infrastructure/
   │  │     │  ├─ swagger/
   │  │     │  │  ├─ SwaggerLoader.java
   │  │     │  │  ├─ SwaggerParserAdapter.java
   │  │     │  │  └─ OpenApiNormalizer.java
   │  │     │  ├─ http/
   │  │     │  │  ├─ RestAssuredClient.java
   │  │     │  │  └─ RestAssuredExecutionAdapter.java
   │  │     │  ├─ mcp/
   │  │     │  │  ├─ common/
   │  │     │  │  │  ├─ McpServer.java
   │  │     │  │  │  └─ McpToolRegistry.java
   │  │     │  │  ├─ swagger/
   │  │     │  │  │  └─ tools/
   │  │     │  │  │     ├─ ListOperationsTool.java
   │  │     │  │  │     ├─ PlanApiRunTool.java
   │  │     │  │  │     ├─ ExecuteOperationTool.java
   │  │     │  │  │     └─ GenerateRestAssuredTestsTool.java
   │  │     │  │  ├─ restassured/
   │  │     │  │  │  └─ RestAssuredMcpExecutor.java
   │  │     │  │  └─ otherservers/
   │  │     │  ├─ llm/
   │  │     │  │  └─ PayloadBuilderLlmClient.java
   │  │     │  ├─ persistence/
   │  │     │  │  ├─ InMemorySpecRepository.java
   │  │     │  │  ├─ InMemoryEnvironmentRepository.java
   │  │     │  │  └─ InMemoryRunPlanRepository.java
   │  │     │  ├─ logging/
   │  │     │  │  └─ LoggerFactoryAdapter.java
   │  │     │  └─ reporting/
   │  │     │     └─ BasicRunReportAggregator.java
   │  │     │
   │  │     ├─ generation/
   │  │     │  ├─ restassured/
   │  │     │  │  ├─ TestNgClassTemplateRenderer.java
   │  │     │  │  └─ RestAssuredSnippetBuilder.java
   │  │     │  └─ cucumber/
   │  │     │     ├─ FeatureFileGenerator.java
   │  │     │     └─ StepDefinitionGenerator.java
   │  │     │
   │  │     └─ util/
   │  │        ├─ IdGenerator.java
   │  │        ├─ Result.java
   │  │        └─ SchemaUtils.java
   │  │
   │  └─ resources/
   │     ├─ application.yml
   │     └─ logback-spring.xml
   │
   └─ test/
      ├─ java/
      │  └─ com/example/swaggerai/
      │     ├─ unit/
      │     └─ integration/
      └─ resources/
```

You MUST NOT flatten these layers or mix classes across them.

---

## APIs That Must Exist (MANDATORY)

Controllers, DTOs, validators, and use cases must be implemented for the modules below.  
Signatures and semantics are the same as the Node version, but implemented in Java.

### 1. Swagger / Spec APIs

#### Import & Normalize Specs

**POST `/spec/import`**

Body:

```json
{
  "source": {
    "type": "url",
    "url": "https://api.example.com/swagger.json"
  }
}
```

or

```json
{
  "source": {
    "type": "file",
    "path": "/tmp/api.yaml"
  }
}
```

or

```json
{
  "source": {
    "type": "git",
    "repo": "git@github.com:org/repo.git",
    "ref": "main",
    "filePath": "specs/api.yaml"
  }
}
```

Response:

```json
{ "specId": "...", "title": "...", "version": "...", "operationCount": 0 }
```

#### Validate Spec

**POST `/spec/validate`**

- Validates a given spec (by `specId` or raw content).
- Response includes `valid: boolean` and list of issues.

#### Introspection

**GET `/spec/:specId`**

- Returns high-level spec metadata (title, version, servers, tags, counts).

**GET `/spec/:specId/operations`**

- Returns list of normalized operations:

```json
{ "operationId": "...", "method": "GET", "path": "/customers", "tags": ["..."], "summary": "..." }
```

**GET `/spec/:specId/tags`**

- Returns list of tags and associated operation counts.

---

### 2. Environment APIs (Multiple Named Environments per Spec)

**POST `/environment`**

Body:

```json
{ "specId": "spec-123", "name": "qa", "baseUrl": "https://...", "defaultHeaders": {}, "authConfig": {} }
```

- Creates a named environment like `dev`, `qa`, `stage`, `prod`.

**GET `/spec/:specId/environments`**

- List environments for a spec.

**GET `/environment/:envId`**

- Get details of one environment.

**PUT `/environment/:envId`**

- Update `baseUrl`, headers, auth.

**DELETE `/environment/:envId`**

- Delete environment (logical delete is fine).

---

### 3. Run Planning & Execution APIs (RestAssured Engine)

**POST `/execution/plan`**

Body example:

```json
{
  "specId": "spec-123",
  "envName": "qa",
  "selection": {
    "mode": "tag",
    "tags": ["Accounts"]
  }
}
```

- Creates a **RunPlan** with selected operations and default test templates.

Response:

```json
{ "runId": "...", "specId": "...", "envName": "qa", "operationCount": 0, "testCount": 0 }
```

**POST `/execution/run`**

- Can either:
  - Accept `runId` (execute existing plan), or
  - Accept `specId`, `envName`, and `selection` to plan+run in one call.
- Execution must:
  - Use **RestAssuredExecutionAdapter** (not raw HTTP client).
  - Optionally flow through **RestAssuredMcpExecutor** for MCP-based execution.
- Response: immediate summary + `runId`.

**GET `/execution/status/:runId`**

- Returns `RunReport`:
  - total, passed, failed, errors
  - per-test status, timings, HTTP status, request/response details.

**POST `/execution/retry-failed`**

Body:

```json
{ "runId": "..." }
```

- Retries only failed or errored tests from last run.

---

### 4. Test Generation APIs (RestAssured + TestNG + Cucumber)

**POST `/testgen/generate-restassured-tests`**

Body:

```json
{
  "specId": "spec-123",
  "selection": {
    "mode": "tag",
    "tags": ["Payments"]
  },
  "options": {
    "includeNegativeTests": true,
    "includeAuthTests": true,
    "includeBoundaryTests": true,
    "generateCucumberAssets": true
  }
}
```

Must:

- Generate a **TestNG + RestAssured** test class (string) with:
  - `@BeforeClass` (or `@BeforeSuite`) to initialize `RestAssured.baseURI`, auth, etc.
  - One `@Test` method per generated test case.
  - `@AfterClass` (or `@AfterSuite`) for cleanup if needed.
- If `generateCucumberAssets = true`:
  - Generate `.feature` content.
  - Generate corresponding Step Definition class using RestAssured for HTTP calls.

Response:

```json
{
  "testClassName": "PaymentsApiTests",
  "language": "java",
  "framework": "testng+restassured",
  "javaSource": "public class PaymentsApiTests { ... }",
  "featureFiles": [
    {
      "name": "payments.feature",
      "content": "Feature: Payments API..."
    }
  ],
  "stepDefinitions": [
    {
      "className": "PaymentsStepDefs",
      "javaSource": "public class PaymentsStepDefs { ... }"
    }
  ],
  "testCases": [
    {
      "operationId": "POST_/payments",
      "testName": "shouldCreatePaymentSuccessfully",
      "expectedStatus": 201
    }
  ]
}
```

**GET `/testgen/spec/:specId/preview`**

- Returns a preview of generated suite (no persistence in v1).

> Even though phase 1 is stateless, the API is designed so that persistence (DB or filesystem) can be added later without breaking endpoints.

---

### 5. MCP Tool APIs (Swagger + RestAssured Agent)

These are HTTP endpoints that mirror MCP tools, used by an AI agent or other services.

**POST `/mcp/swagger/list-operations`**

Body:

```json
{ "specId": "spec-123" }
```

- Returns operations for the agent to choose from (same as `/spec/:specId/operations` but MCP-oriented).

**POST `/mcp/swagger/plan-run`**

Body:

```json
{ "specId": "spec-123", "envName": "qa", "selection": { "mode": "full" } }
```

- Returns `{ "runId": "...", "summary": { ... } }`.

**POST `/mcp/swagger/execute-operation`**

Body:

```json
{
  "specId": "spec-123",
  "envName": "qa",
  "operationId": "GET_/customers",
  "overrides": {
    "pathParams": { "id": "123" },
    "query": { "active": true },
    "headers": { "x-correlation-id": "abc" },
    "body": { "...": "..." }
  }
}
```

- Executes a single operation through **RestAssuredMcpExecutor** and returns request/response + timing.

**POST `/mcp/swagger/generate-restassured-tests`**

- Same semantics as `/testgen/generate-restassured-tests`, but oriented for agent workflows.

---

### 6. LLM APIs (Schema + LLM Assisted Payloads)

**POST `/llm/build-payload`**

Input:

```json
{
  "specId": "spec-123",
  "operationId": "POST_/customers",
  "mode": "schema-with-llm",
  "hints": {
    "locale": "IN",
    "domain": "banking"
  }
}
```

Output:

- One or more example request bodies built from schema + LLM.

**POST `/llm/suggest-tests`** (future / optional)

- Given an operation, propose additional edge-case tests to augment templates.

---

## Incremental Development Plan (Copilot MUST Follow This Order)

Copilot must not jump phases. Implement phase by phase, smallest unit possible, and keep each step non-breaking.

### Phase 1 — Project Setup (Java + Spring Boot)

**Objective:** Create a clean, Java 17+Spring Boot project with layers and base infrastructure.

Tasks:

- Initialize `pom.xml`:
  - Spring Boot starter (web)
  - Lombok (optional)
  - Jackson
  - RestAssured (for infrastructure + tests)
  - TestNG + JUnit 5 (for internal tests)
  - Cucumber-JVM (for generated assets support)
- Create package structure exactly as defined above.
- Implement:
  - `SwaggerAiApplication.java` → Spring Boot entry point.
  - `EnvironmentConfig.java` → loads `application-*.yml`.
  - `LoggerFactoryAdapter.java` → wraps `slf4j` or Logback.
  - Global exception handlers: `@ControllerAdvice` with translation from domain/infrastructure exceptions to HTTP responses.
- Add basic health endpoint (`/actuator/health` or a simple `/health`).
- Add one **sample unit test** (JUnit or TestNG) to verify setup.

---

### Phase 2 — Domain Layer (NO Infrastructure Calls)

**Objective:** Define domain objects and repository interfaces for specs, operations, runs, environments, and test definitions.

Tasks:

In `domain.model` define:

- `NormalizedSpec` (id, title, version, servers, operations)
- `Operation` (id, method, path, tags, summary, parameters, requestBody, responses, security)
- `EnvironmentConfigModel` (name, baseUrl, headers, auth)
- `RunPlan` (runId, specId, envName, operations, testCaseDefinitions)
- `RunReport` (summary + per test results)
- `TestCaseDefinition` (test type, expected status, payload strategy, tags)
- `PayloadTemplate` (schema-derived payload info)

In `domain.repository` define interfaces:

- `SpecRepository`
- `EnvironmentRepository`
- `RunPlanRepository`
- `TestTemplateRepository`

Constraints:

- Domain layer must not import Spring, RestAssured, HTTP clients, or LLM SDKs.
- Use Java types only (`List`, `Map`, etc.).

---

### Phase 3 — Infrastructure Skeleton

**Objective:** Prepare adapters without full logic.

Tasks:

- `SwaggerLoader.java`
  - API: `loadFromUrl`, `loadFromFile`, `loadFromGit` (Git stub initially).
- `SwaggerParserAdapter.java`
  - Wraps Swagger/OpenAPI parser (e.g., `swagger-parser`), but can be stubbed.
- `OpenApiNormalizer.java`
  - Skeleton: `NormalizedSpec normalize(Object rawSpec)`.
- `RestAssuredClient.java`
  - Wrap basic RestAssured calls in a small client abstraction.
- `RestAssuredExecutionAdapter.java`
  - Skeleton method: `InvokeResult executeOperation(NormalizedSpec spec, Operation op, EnvironmentConfigModel env, Overrides overrides)`.
- `McpServer.java`, `McpToolRegistry.java`
  - Skeleton classes representing a basic Java MCP server using stdin/stdout if needed.
- `PayloadBuilderLlmClient.java`
  - Interface for LLM calls (no implementation in phase 3).
- In-memory repositories:
  - `InMemorySpecRepository.java`
  - `InMemoryEnvironmentRepository.java`
  - `InMemoryRunPlanRepository.java`

No business logic in these classes yet; only method signatures and basic structure.

---

### Phase 4 — Spec Ingestion & Normalization

**Objective:** Read Swagger/OpenAPI (2.0 & 3.x) and convert to `NormalizedSpec`.

Tasks:

- Implement `IngestSwaggerUseCase.java`:
  - Input: `SpecSource` (url/file/git).
  - Flow: `SwaggerLoader` → `SwaggerParserAdapter` → `OpenApiNormalizer`.
  - Persist `NormalizedSpec` through `SpecRepository`.
  - Output: summary (specId, title, version, operationCount).
- Implement `NormalizeSpecUseCase.java`:
  - Convert parser output to `NormalizedSpec`.
  - Handle both Swagger 2.0 and OpenAPI 3.x.
- Implement `ValidateSpecUseCase.java`:
  - Basic structural validation, returning list of issues (missing paths, methods, schemas).
- Implement `ListOperationsUseCase.java`:
  - Returns operations with `operationId`, `method`, `path`, `tags`, `summary`.

Wire API layer:

- `SpecController.java`:
  - `/spec/import`
  - `/spec/validate`
  - `/spec/{specId}`
  - `/spec/{specId}/operations`
  - `/spec/{specId}/tags`
- `SpecRequestValidator.java` for request payload validation.
- Map between DTOs and domain models in `SpecDto.java`.

---

### Phase 5 — Environment Configuration

**Objective:** Manage multiple named environments per spec.

Tasks:

- Implement use cases in `application.environment`:
  - `CreateEnvironmentUseCase`
  - `ListEnvironmentsUseCase`
  - `UpdateEnvironmentUseCase`
  - `DeleteEnvironmentUseCase`
- Use `EnvironmentRepository` (in-memory).
- Implement `EnvironmentController.java` and map:
  - `POST /environment`
  - `GET /spec/{specId}/environments`
  - `GET /environment/{envId}`
  - `PUT /environment/{envId}`
  - `DELETE /environment/{envId}`
- Implement `EnvironmentRequestValidator.java`:
  - Validate `baseUrl`, default headers, auth config.

---

### Phase 6 — Run Planning

**Objective:** Plan which operations to test and generate initial `TestCaseDefinition` sets.

Tasks:

- Implement `PlanRunUseCase.java`:
  - Input: `specId`, `envName`, `selection` (single/tag/full).
  - Load `NormalizedSpec` and `EnvironmentConfigModel`.
  - Select operations according to selection mode.
  - Use internal “template builder” (pure helper inside application layer, not infra) to produce `TestCaseDefinition`s per operation:
    - Happy path (expected 2xx/201/204 etc.).
    - Basic validation error (missing required field).
    - Basic auth error (if security defined).
  - Persist `RunPlan` using `RunPlanRepository`.
  - Return summary (runId, operationCount, testCount).
- Implement `GetRunStatusUseCase.java` as stub returning run metadata and placeholder status initially.
- Wire `ExecutionController.java` and map:
  - `POST /execution/plan` → `PlanRunUseCase`

---

### Phase 7 — RestAssured Execution Engine & RestAssured MCP

**Objective:** Execute HTTP calls defined by `RunPlan` using RestAssured, optionally via RestAssured MCP.

Tasks:

- Implement `RestAssuredExecutionAdapter.java`:
  - Build concrete RestAssured request given:
    - Base URI, path, query params, headers, body.
  - Return `InvokeResult`:
    - HTTP status, headers, body, duration.
- Implement `RestAssuredMcpExecutor.java`:
  - Wrap calls to `RestAssuredExecutionAdapter`.
  - Expose MCP-style commands (for agent control) internally.
- Implement `ExecuteRunUseCase.java`:
  - Input: `runId` or `{ specId, envName, selection }` (plan+run).
  - For each `TestCaseDefinition`:
    - Build concrete request payload.
    - Call `RestAssuredExecutionAdapter` (or `RestAssuredMcpExecutor`).
    - Compare actual vs expected status.
    - Update `RunReport` with pass/fail/error.
  - Persist `RunReport` and update `RunPlan` status.
- Extend `ExecutionController.java`:
  - `POST /execution/run`
  - `GET /execution/status/{runId}`

---

### Phase 8 — Template-Based RestAssured + TestNG + Cucumber Generation

**Objective:** Generate Java test code from specs and templates.

Tasks:

- Implement `GenerateRestAssuredTestsUseCase.java`:
  - Use `NormalizedSpec` and `TestCaseDefinition` set to generate:
    - TestNG + RestAssured test class (`javaSource` string).
    - Optional Cucumber `.feature` file(s) and step definition class(es).
  - Use `TestNgClassTemplateRenderer` + `RestAssuredSnippetBuilder`:
    - `@BeforeClass`/`@BeforeSuite` sets up RestAssured base URI/env.
    - Each test in `TestCaseDefinition` → one `@Test` method.
    - Use idiomatic RestAssured DSL:

      ```java
      given()
        .baseUri(baseUrl)
        .headers(headers)
        .body(payload)
      .when()
        .post("/payments")
      .then()
        .statusCode(201);
      ```

  - Use `FeatureFileGenerator` to create Gherkin scenarios.
  - Use `StepDefinitionGenerator` to create Cucumber step classes calling RestAssured.

- Implement `TestGenerationController.java` and map:
  - `POST /testgen/generate-restassured-tests`
  - `GET /testgen/spec/{specId}/preview`

---

### Phase 9 — LLM-Assisted Payload Builder (Schema + LLM, Not Full Auto)

**Objective:** Use LLM only when Swagger examples/defaults are missing.

Tasks:

- Implement `BuildPayloadFromSchemaUseCase.java`:
  - Inspect operation’s requestBody schema.
  - Build payload using:
    - Default values / examples / formats where available.
    - LLM only for fields with no examples and required status.
  - Call `PayloadBuilderLlmClient` **only for missing parts**.
  - Return candidate request bodies.
- Add `/llm/build-payload` endpoint:
  - Controller + DTO + validator.
- Ensure **execution is not blocked** if LLM is unavailable:
  - Fallback to schema-only payloads.

---

### Phase 10 — MCP Tool Integration (Hybrid Tool Surface)

**Objective:** Expose MCP tools (and matching HTTP endpoints) for agents, backed by the same use cases.

Tasks:

- Implement tools in `infrastructure.mcp.swagger.tools`:
  - `ListOperationsTool` → calls `ListOperationsUseCase`.
  - `PlanApiRunTool` → calls `PlanRunUseCase`.
  - `ExecuteOperationTool` → calls `ExecuteRunUseCase` for a single operation.
  - `GenerateRestAssuredTestsTool` → calls `GenerateRestAssuredTestsUseCase`.
- Implement `SwaggerMcpController.java`:
  - `POST /mcp/swagger/list-operations`
  - `POST /mcp/swagger/plan-run`
  - `POST /mcp/swagger/execute-operation`
  - `POST /mcp/swagger/generate-restassured-tests`
- **Important:** MCP tools must call only application use cases; they must not directly use RestAssured or the parser.

---

### Phase 11 — Retry & Partial Reruns + Basic Reporting

**Objective:** Support retry of failed tests and basic reporting.

Tasks:

- Implement `RetryFailedTestUseCase.java`:
  - Given `runId`, load `RunReport` and `RunPlan`.
  - Build a temporary `RunPlan` with only failed/errored tests.
  - Execute using the same engine.
- Extend `RunReport`:
  - Aggregated status by tag, method, path.
- Extend `/execution/status/{runId}` response:
  - Include aggregates and basic statistics.

---

### Phase 12 — Hardening & Pre-DB Readiness

**Objective:** Make the service robust and ready for DB-backed persistence.

Tasks:

- Request validation:
  - Implement in all `*RequestValidator` classes (using e.g. Bean Validation / custom logic).
- Structured logs:
  - Log key events: spec ingestion, run execution, MCP calls, LLM usage.
- Timeouts, retries, error mapping in `RestAssuredClient`:
  - Reasonable default timeouts and retry policies.
- Limits:
  - Request size limits for spec upload.
- Tests:
  - Normalization logic unit tests.
  - Run planning unit tests.
  - ExecuteRun tests using RestAssured mocks.
  - Test generation tests (assert generated code contains expected patterns).
- Keep repositories **interface-driven** to allow swapping in Mongo/Postgres later without changing use cases.

---

## What Copilot Should Always Do (Java Version)

Always:

- Respect package boundaries and layering.
- Ask for missing context (e.g. which use case to implement next) before introducing new dependencies.
- Generate code in **small units** (one class / file at a time).
- Use clear Java types and interfaces.
- Add minimal Javadoc or comments for non-trivial logic.
- Keep controllers thin and delegate orchestration to use cases.
- Keep domain models free from infrastructure concerns.
- Keep generated RestAssured + TestNG code:
  - Minimal
  - Readable
  - Without unnecessary framework abstractions.

---

## What Copilot Should Never Do (Java Version)

Never:

- Collapse multiple modules into a single “god class”.
- Mix infrastructure logic inside controllers.
- Call RestAssured, Swagger parser, or LLM directly from controllers or domain.
- Import Spring or RestAssured into the **domain** layer.
- Skip types and overuse raw `Object` or `Map` without clear typing.
- Break or flatten the folder/package structure.
- Hard-code environment-specific values inside use cases.
- Bake TestNG / Cucumber specifics into domain models (keep them in `generation` layer).

---

## FINAL INSTRUCTIONS

- Use all of the above as the **master architecture specification** for the **Java + RestAssured Swagger AI Agent**.
- Generate code incrementally, module-by-module, following the package structure exactly.
- Always ask me **which module or class to implement next** before writing code.
- Only produce code for the **single Java file** we are currently building.
- Preserve backwards compatibility across phases; do not introduce breaking changes when extending features.
- Generated RestAssured + TestNG + Cucumber assets must remain **simple and framework-light**, so learners can understand and refactor them into their own frameworks if needed.

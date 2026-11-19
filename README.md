# Alex Ekimenko
### Solutions Architect | Systems & Integration Architect
**High-performance integration architectures, data pipeline solutions, and infrastructure decoupling.**

Architect & Solutions Engineer specializing in fault-tolerant data integration architectures for enterprise ecosystems (such as SAP BW/4HANA, 1C:TOiR) and transforming monoliths into distributed services.
* **Architecture:** Decoupled Service Topologies, Separation of Concerns (SoC), Dependency Injection (DI) Lifecycle Management, Metadata-Driven Initialization, Reactive Stream Orchestration, In-Memory Ledger Aggregation.
* **Core Competencies:** End-to-End Data Conversion & ETL Pipelines, Fault-Tolerant Cross-Source Integration (REST APIs, RDBMS, Legacy File Structures), Resilient Network Transport Subsystems, Real-Time Streaming & Reactive Data Processing, Test-Driven Quality Control.
* **Tech Baseline:** Java Enterprise, RxJava 3, Java Stream API, Google Guice, Apache HttpClient 5, Hibernate, Apache POI/Spire.XLS, Jackson (JSON/YAML), JUnit 5, Mockito, Log4j2/SLF4J, P6Spy, Custom SSL/TLS Trust Store Hot-Swapping Infrastructure.
---
## 🏗 Selected Enterprise Architectural Solutions

### 1. [Reactive Multi-Module Data Transformation & Ingestion Engine](https://github.com/an-ekimenko/DataTransformationRxWorkers.git)

*Target Systems: MS SQL Server Staging | Oracle Database | SAP BW/4HANA*

* **Architectural Challenge & Solution:** Asynchronous multi-module ETL pipeline (`ACN_Worker` and `FI_Worker`) on **RxJava 3** and **Google Guice 7** for concurrent API/Oracle DB ingestion (`CompletableFuture`) and MS SQL Server (PSA) staging loads.
* **Key Architectural Highlights:**
  * **`AbstractResilientWorker` Base Class:** Manages lifecycle safety, monotonic budget tracking (`System.nanoTime()` vs `max_timeout_ms`), exponential backoff with jitter (`ThreadLocalRandom` via `RANDOMIZATION_FACTOR`), and fast-fail retry pruning.
  * **Scheduling & Concurrency:** Utilizes a `2000 ms` window with `lastExecutionMap` deduplication, RxJava `isRunning()` gates, and `AtomicBoolean` locks to eliminate overlap and database lock contention.
  * **Shared Infrastructure & CLI:** Centralizes **HikariCP** connections in `SharedInfraModule` (`SessionFactory`), loads JPA entities via `entities.yaml`, and supports dynamic worker discovery through `application.yaml` (`xn` / `xn <alias>`).
  * **Testing (`JUnit 5` & `Mockito`):** Full test suite across 4 Maven modules (`AbstractResilientWorkerTest`, `RxSchedulerMainTest`, `ACN_AppTest`, `FI_AppTest`, custom IO/deserializer tests) utilizing `@TempDir` and static mocks.


### 2. Decoupled KPI & Surcharge Analytical Ingestion Pipeline

*Target Systems: SAP BW Data Bus | Corporate Data Warehouses*

* **Architectural Challenge & Solution:** Replaced a set of tightly coupled legacy monolith classes with a modular ingestion architecture designed around strict **Separation of Concerns (SoC)** and **Single Responsibility Principles (SRP)**. This architectural blueprint is deployed across **two distinct yet structurally aligned extraction utilities** targeting corporate spreadsheets:
  * **[FuelSurchargesCalculation](https://github.com/an-ekimenko/FuelSurchargesCalculation)** — Specifically engineered to parse critical retail fuel surcharge structures.
  * **[PlannedFuelKPI](https://github.com/an-ekimenko/PlannedFuelKPI)** — Designed to extract complex, volatile multi-layered organizational Key Performance Indicators.
  
  By isolating initialization, file streaming, cell-parsing, and serialization into independent components, both systems completely shield downstream analytical platforms from evolving worksheet structures.

* **Solutions & Decoupling Highlights:**
  * **Stateful Tracking Pattern:** Implemented dynamic state-aware processing to resolve vertical merged cell vectors (where native evaluations drop cell value properties outside the top-left coordinate origin), maintaining the context of parent metrics during row-by-row traversal without heavy in-memory multi-pass caching.
  * **Loose Coupling via Abstraction:** Utilized Factory-like dynamic binary signature analysis to completely decouple low-level components from concrete spreadsheet classes. The initialization layer remains unchanged even if future library updates introduce new file extensions, while successfully neutralizing manual human error (improperly renamed files).
  * **Zero-Loss Precision Control:** Enforced arithmetic ledger stability with programmatic ceiling rules via exact `BigDecimal` mapping, preventing scientific exponential notations and multi-million rounding drift within the automated SAP integration bus.

### 3. [High-Concurrency Automated Staging Area (PSA) Subsystem](https://github.com/an-ekimenko/GSPassportization.git)

*Target Systems: MS SQL Server | Enterprise Web Services | SAP BW/4HANA*

- **Architectural Challenge & Solution:** Engineered a high-throughput, decoupled ingestion framework (`GSPassportization`) for gas station master data and hardware telemetry using **Google Guice 7 DI**, guaranteeing loose coupling across network, serialization, and persistence layers while providing automated TLS lifecycle management and scheduler observability.
- **Solutions & Decoupling Highlights:**
  - **Resilient Multi-Schema HTTP Transport:** Powered by **Apache HttpClient 5** (`GSP_HttpClient`) with a managed connection pool, configurable timeouts, jittered exponential backoff retry policies (`ExponentialBackoffRetryStrategy` with `initial_timeout_ms` / `max_timeout_ms` budgets), and support for `basic`, static `token`, `oauth2` (Client Credentials Grant with thread-safe in-memory token caching), and `none` authentication schemes.
  - **State-Aware TLS & Certificate Lifecycle Management:** Integrates custom `KeyStoreManager` and `CertUpdater` (`GSP_HttpClient::triggerCertUpdate`) with lazy client initialization, intercepting remote certificate chains to update the local JVM `cacerts` store and hot-swap `SSLContext` on the fly.
  - **Defensive Parsing, Deduplication & Observability:** Features a fault-tolerant Jackson deserializer (`gsp.jackson.GSP_DataDeserializer`) for coordinate normalization, scientific notation / non-finite filtering, null-safe array cleansing, and numeric overflow protection, Java Stream invalid-ID filtering and business-key deduplication, transactional Hibernate 6+ batch persistence (`GSP_DAO`), and automated YAML execution summaries (`exchange/result.yaml`).


### 4. [Preemptive Security & Network Resiliency Layer for Legacy Gateways](https://github.com/an-ekimenko/RepairRequests)

*Target Systems: Microsoft IIS 10.0 Proxy | 1C:TOiR Web Services | MS SQL Server*

- **Architectural Challenge & Solution:** Refactored a tightly coupled ingestion utility into a modular, **Google Guice 7 DI**-driven integration agent (`Jackson_RepairRequests`) designed to interface modern HTTP clients with strict, misconfigured enterprise network perimeters (`wsisapi.dll`), bypassing firewall-induced session drops while providing automated observability for external job schedulers.
- **Solutions & Decoupling Highlights:**
  - **Modular DI & Multi-Schema Auth (`OAuth2` & Preemptive Basic):** Decoupled configuration, transport, serialization, and persistence layers via `RRAppModule`. Integrated an optimized **Apache HttpClient 5** core supporting `basic` (forcing `BasicAuthCache` preemptive credentials on the initial outbound socket packet), static `token`, `oauth2` (Client Credentials Grant with thread-safe in-memory token caching), and `none` schemes alongside full wire-level request/response diagnostics.
  - **State-Aware Lazy Initialization:** Defers `CloseableHttpClient` instantiation until the explicit runtime execution window. This architectural constraint guarantees that the underlying `SSLContext` is established only after `CertUpdater` verifies and synchronizes the local JVM truststore (`cacerts`) against the latest remote certificates.
  - **Configurable Exponential Backoff with Jitter:** Embeds a custom `DefaultHttpRequestRetryStrategy` driven by external YAML boundaries (`initial_timeout_ms`, `max_timeout_ms`, `multiplier`, `randomization_factor`) to survive prolonged network disruptions without thundering-herd effects.
  - **Metadata-Driven Filtering & Scheduler Reporting:** Performs early payload filtering via `RR_DataDeserializer` against configurable `equipment_classifier_codes`, deduplicates records by composite business key (`RepairRequests_PK`), and outputs machine-readable YAML execution summaries (`exchange/result.yaml`) with standard OS exit codes (`0`/`1`).

### 5. [Automated Corporate Member Activity Ingestion Agent](https://github.com/an-ekimenko/ActiveClientsNumber.git)

*Target Systems: Enterprise REST Gateways | MS SQL Server Staging (PSA) | SAP BW/4HANA*

* **Architectural Challenge & Solution:** Overview of the decoupled Java 17 ETL agent (`ActiveClientsNumber`) designed to extract and store corporate member metrics for SAP BW/4HANA. 
* **Solutions & Decoupling Highlights:**
  * **Timezone-Anchored Reporting Window Resolution:** Handles period calculation using the `Europe/Minsk` timezone (`ACN_App`) to parameterize requests.
  * **Dual-Layer Jittered Exponential Backoff:** Implements adaptive retry logic with randomized jitter (`RANDOMIZATION_FACTOR = 0.5`) and timeout caps across both pipeline phases:
    - *API Fetch (`ACN_HttpClient`):* Handles network errors and retryable HTTP statuses (`429`, `500–504`), respecting `Retry-After` headers.
    - *DB Write (`ACN_DAO`):* Retries batched Hibernate operations (`flush`/`clear` per 50 entities) during SQL outages or deadlocks (`Error 1205`), while failing immediately on constraint violations.
  * **Auth, TLS & Deduplication:** Supports `basic`, `token`, `oauth2` (with in-memory caching), and `none` auth, dynamic TLS hot-swapping via `CertUpdater`, safe parsing (`ACN_DataDeserializer`), composite-key deduplication (`ACN_PK`), and YAML reporting (`exchange/result.yaml`).

### [6. Dynamic Cryptographic SSL/TLS Trust Store Hot-Swapping System](https://github.com/an-ekimenko/CryptoCertLoader)
*Target Systems: JVM Security Infrastructure | Distributed Integration Apps*
* **Architectural Challenge & Solution:** Designed a zero-dependency, pluggable shared-library component automating SSL certificate verification and trust boundaries across neighboring operational modules. It enables enterprise client applications to survive upstream corporate certificate rotations with zero service downtime.
* **Solutions & Decoupling Highlights:**
  * **In-Memory SSLContext Hot-Swapping:** Programmatically intercepts remote certificate chains atomically during secure TLS handshakes using low-overhead discarding body handlers. It updates the physical trust store (cacerts) and dynamically injects the modified SSLContext into active JVM memory on-the-fly, allowing active network clients to leverage fresh certificates instantly without requiring application restarts.
  * **Validity & Unique Metadata Mapping:** Generates deterministic unique aliases based on X500Principal signatures and automatically replaces local certificates failing validation checks, keeping the trust store clean.
  * **Proactive Backup & Rollback Pipeline:** Implements transactional atomicity for filesystem writes. Generates automated, timestamped backups of the master cacerts file before committing updates, triggering immediate rollback sequences upon detecting I/O faults to protect JVM security integrity.

---

## 📦 Global Artifact Deployment Topology

To maximize operational resource efficiency and maintain a low memory and disk footprint across production hosts, all solutions adopt a strict **Decoupled Shared-Dependency Model**. Heavy third-party framework binaries are entirely stripped from individual runtime modules and dynamically linked via manifest configuration hooks:

```
shared-environment-root/
├── dependencies/               # Centralized, shareable third-party library pool (POI, Guice, Log4j2, etc.)
├── CryptoCertLoader/           # Dynamic cryptographic TLS validation and context re-initialization library
├── DataTransformationRxWorkers/# Reactive multi-worker engine handling telemetry stream transformations
├── GSPassportization/          # High-performance web-API staging engine for gas station assets
├── ActiveClientsNumber/        # Timezone-aware corporate member telemetry ETL agent with dual-layer retries
├── RepairRequests/             # Preemptive authentication 1C:TOiR staging area data ingestor
├── FuelSurchargesCalculation/  # Semicolon-delimited flat CSV retail surcharge aggregator
├── AdditionalBonus/            # Large file optimization pipeline using formula-driven throttling
└── PlannedFuelKPI/             # Dynamic Excel context parser utilizing stateful tracking patterns
```

### Operational Component Matrix

* **`dependencies/`**  
  Centralized shareable library pool (Apache POI, Log4j2, Google Guice, Hibernate, etc.) designed to enforce strict infrastructure decoupling, optimizing memory and disk footprint across the ecosystem.
* **CryptoCertLoader/**  
  Programmatic TLS Handshake Interceptor Shared Library. Automates dynamic keystore context management and hot-swaps in-memory SSLContext signatures without service restarts.
- **DataTransformationRxWorkers/** Reactive Multi-Module ETL & Ingestion Engine на базе **RxJava 3** и **Google Guice 7**. Включает универсальный паттерн отказоустойчивости `AbstractResilientWorker` (контроль таймаутов через `System.nanoTime()` и экспоненциальную задержку с джиттером через `ThreadLocalRandom`), защищённые от джиттера окна планирования (`2000 ms`), двухуровневый бпрессур-контроль (`AtomicBoolean`), общий пул соединений **HikariCP** (`SharedInfraModule` + `entities.yaml`), динамическое YAML-обнаружение воркеров (`xn` / `xn <alias>`) и покрытие тестами **JUnit 5 / Mockito**.
* **GSPassportization/** High-Performance Gas Station Asset Staging Engine utilizing Google Guice DI, multi-schema authentication (`basic`, `token`, `none`, `oauth2 with caching`), configurable transport timeouts, runtime TLS certificate rotation, defensive JSON parsing, stream deduplication, and automated `exchange/result.yaml` reporting for SAP BW/4HANA.
* **ActiveClientsNumber/** Timezone-Aware Corporate Member Telemetry ETL Agent utilizing Google Guice 7 DI, multi-schema authentication (`basic`, `token`, `none`, `oauth2` with caching), `Europe/Minsk` reporting period resolution, runtime TLS certificate rotation, dual-layer jittered exponential backoff retries (across both API fetch and MS SQL batch persistence), composite-key deduplication (`ACN_PK`), and automated `exchange/result.yaml` reporting for SAP BW/4HANA.
* **RepairRequests/**  
  Resilient Preemptive Auth 1C:TOiR Data Ingestor. A standalone synchronization module enforcing complete Separation of Concerns (SoC). It handles complex proxy-level constraints via decoupled request-scoped header injections and isolates deprecated security profiles, ensuring maximum ingestion fault-tolerance.
* **FuelSurchargesCalculation/**  
  Zero-Loss Accounting Metric Aggregator. A highly decoupled enterprise ETL engine designed around strict single-responsibility principles. It features an anti-deadlock synchronized bootstrap layer, native NIO character layer protection, and a fault-tolerant exponential backoff loop to completely bypass file-handling bottlenecks and operating system locks during downstream SAP BW loads.
* **AdditionalBonus/**  
  Cloud Excel Transformation Utility. An isolated pipeline that guarantees resource leak prevention and auto-cleanup by leveraging formula-driven connection throttling for large file optimization, delivering a fully decoupled footprint observability layer via runtime metadata state serialization.
* **PlannedFuelKPI/**  
  Resilient ETL Integration Agent. Designed around the Separation of Concerns (SoC) paradigm, this utility completely decouples stateful processing vectors to resolve complex infrastructure constraints. It features lock-free concurrent shared-stream reading via InputStream to bypass active operating system file handling locks and handles user-induced format mutations via an automated signature analysis engine.


---

### Core Architecture & Infrastructure Isolation Contracts

The integration ecosystem is engineered around a unified architectural framework that guarantees strict isolation, runtime resilience, and seamless decoupling across all processing layers:

* **Centralized Shareable Infrastructure (Shared Dependency Pool):** Application archives completely exclude heavy third-party binaries, resolving system-wide framework lifecycle boundaries dynamically via manifest Class-Path hooks. This enforces strict infrastructure decoupling and significantly reduces the system's memory and deployment footprint.
* **Context-Independent Execution Topology:** Built-in dynamic path resolvers automatically decode code-source location contexts at runtime. This completely eliminates hardcoded absolute system dependencies, allowing identical binaries to operate flawlessly across heterogeneous execution scopes (e.g., local IDE environments, production standalone JARs, or OS schedulers).
* **Defensive Parsing & Structural Invariant Validation:** Employs customized, fault-tolerant JSON/YAML deserializers and dedicated file-traversal components. This architecture isolates core business logic from volatile source data formats, gracefully normalizing malformed structures, catching data overflows, and protecting workflows against manual user-input anomalies.
* **Pre-Flight Automated TLS Lifecycle Management:** Integrates pluggable trust-store synchronization components that programmatically evaluate target endpoint validation states. The subsystem automatically updates JVM security baselines and hot-swaps active memory contexts on-the-fly, enabling client routines to survive upstream certificate rotations without downtime.
* **Lock-Free Concurrent Input Overlapping:** Mitigates host operating system file-descriptor locking bottlenecks (e.g., target sheets concurrently opened by corporate planning staff) by executing shared-read byte stream models. Localized data matrices are isolated near-instantaneously into secure memory heap context boundaries.
* **Decoupled Architecture, DI, & Universal Separation of Concerns:** Extends strict Single Responsibility Principles across all project blueprints, leveraging type-safe Dependency Injection (DI) frameworks to entirely decouple application lifecycles from static context state blockages. Environmental variables, logging engines, cross-schema authentication pipelines, transport layers, and data mappers are isolated into distinct, specialized entities—ensuring clean boundaries, modular orchestration, and seamless runtime adjustments with zero codebase modification or artifact recompilation.

---
## 🧪 Enterprise Testing & Verification Frameworks

Architectural designs are verified against rigorous automated quality control criteria utilizing isolated, fast-executing **JUnit 5** and **Mockito** test matrices:

* **Infrastructure Virtualization:** Completely eliminates file system dependencies, IO locks, and network latency friction by simulating OS-level behaviors using temporary disk sandboxes (`@TempDir`) and mocking heavy native vendor ecosystems.
* **Private Method Isolation:** Leverages Java Reflection API wrappers to run deep functional validations directly on private processing subroutines without polluting clean class interfaces.
* **Behavioral Verification:** Employs argument captors and static method interception (`Mockito.mockStatic`) to validate the construction of authorization headers and track in-memory business-key deduplication metrics before database writes occur.

---

## ✉️ Contact & Collaboration

*   **Email:** [a.ekimenko@beloil.by]
*   **LinkedIn / Telegram:** [@an_ekimenko]

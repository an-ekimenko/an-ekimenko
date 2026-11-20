# Alex Ekimenko

**Enterprise Integration Architecture, Resilient Data Pipelines & Distributed System Design.**

Solutions & Integration Architect specializing in fault-tolerant data ecosystems, legacy-to-modern system bridging (SAP BW/4HANA, 1C, RDBMS), and modularizing monolithic ETL workflows into scalable, maintainable services.

* **Architectural Governance & Design:** Domain-Driven Modularization (DDD), Separation of Concerns (SoC), Metadata-Driven Extensibility, Fault-Tolerant & Self-Healing Systems, Zero-Downtime Security Lifecycle, NFR Optimization (Throughput, Reliability, Observability).
* **Integration & Data Engineering:** End-to-End ETL/ELT Pipelines, Multi-Source Ingestion (REST/OAuth2, JDBC, Legacy File Formats), Idempotent Data Processing & Deduplication, Transactional Batching, High-Concurrency Stream Orchestration.
* **Technology Ecosystem:** Java Enterprise (JDK 17+), Reactive Streams (RxJava 3), Dependency Injection (Google Guice), Resilience Patterns (Exponential Backoff & Jitter, Circuit/Budget Controls), Hibernate / HikariCP, Apache HttpClient 5, Automated QA (JUnit 5, Mockito), Structured Observability & Telemetry.

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

### 3. Resilient REST-to-PSA Enterprise Ingestion Agents Suite
*Target Systems: Enterprise REST APIs | Microsoft IIS 10.0 / 1C:TOiR | MS SQL Server Staging (PSA) | SAP BW/4HANA*

- **Shared Architectural Foundation & Capabilities:**
  - **Modular DI & Declarative Configuration:** Built on **Google Guice 7** with strict Separation of Concerns (`SoC`), shared classpath topology, and a `Fail-Fast properties.yaml` configuration defining authentication strategies, connection/read timeouts (`connect_timeout_ms`, `response_timeout_ms`), initial backoff delays (`initial_timeout_ms`), and overall API execution time budgets (`max_timeout_ms`).
  - **Pooled Transport & Auth:** Powered by **Apache HttpClient 5** supporting `basic`, static `token`, `oauth2` (with in-memory token caching), and `none` schemes.
  - **Zero-Downtime TLS & Resiliency:** Features `lazy client initialization`, `runtime JVM cacerts hot-swapping`, and `jittered exponential backoff retries`.
  - **Defensive Processing & Observability:** Includes `fault-tolerant` Jackson `deserializers`, composite-key deduplication, `transactional` Hibernate 6+ `batch persistence`, `JUnit 5 / Mockito tests`, and `machine-readable scheduler receipts` (*`exchange/result.yaml`*).

- **Specialized Ingestion Agents & Distinctive Features:**
  - [**GSPassportization** *(Gas Station Asset & Hardware Telemetry Staging)*](https://github.com/an-ekimenko/GSPassportization.git): *`Normalizes`* polymorphic geo-coordinates (filtering *`NaN`/`Infinity`* values), *`sanitizes`* and *`flattens`* fuels arrays into comma-separated strings (with *`scalar fallback`*), and `extracts positive` integer *`station identifiers`* prior to persistence.
  - [**RepairRequests** *(1C:TOiR Equipment Maintenance & Legacy Gateway Integration)*](https://github.com/an-ekimenko/RepairRequests.git): *`Bypasses proxy drops`* on stateless Microsoft IIS 10.0 *`by forcing preemptive`* BasicAuthCache `credentials`, *`computes annual reporting windows`* (Europe/Minsk), and `filters records` by *`configurable`* equipment classifier codes.
  - [**ActiveClientsNumber** *(Corporate Member Activity & Monthly Metrics Agent*](https://github.com/an-ekimenko/ActiveClientsNumber.git):  Dynamically resolves preceding month boundaries anchored to `Europe/Minsk`, implements `two-step OAuth2 authentication` (token acquisition followed by authenticated data retrieval), and extends `adaptive jittered retry logic to the Hibernate DAO layer` for MS SQL resilience.

### [4. Dynamic Cryptographic SSL/TLS Trust Store Hot-Swapping System](https://github.com/an-ekimenko/CryptoCertLoader)
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

### Core Architectural Principles & System Contracts

The integration platform is built on standardized architectural contracts that enforce strict decoupling, operational resilience, and predictable scaling across heterogeneous enterprise environments:

* **Plugin-Based & Metadata-Driven Extensibility:** Decouples core orchestration engines from domain business logic using Dependency Injection (DI) and declarative YAML manifests. New data sources, workers, and persistence entities can be onboarded without modifying core infrastructure code.
* **End-to-End Fault Tolerance & Self-Healing:** Implements multi-layered resilience boundaries across network and persistence tiers. Combines monotonic time-budgeted retries, jittered exponential backoff, deadlock recovery, and atomic state rollbacks to ensure continuous operation under unstable network or database conditions.
* **Automated Zero-Downtime Security Lifecycle:** Abstracts authentication (OAuth2 token caching, Preemptive Basic, Bearer) and cryptographic trust management into reusable infrastructure layers. Automated TLS certificate interception and in-memory `SSLContext` hot-swapping eliminate service downtime during upstream certificate rotations.
* **Defensive Data Contracts & Idempotency:** Enforces strict schema validation, type-overflow protection, exact decimal arithmetic (`BigDecimal`), and composite-key deduplication at the ingestion boundary—guaranteeing data integrity and safe re-runs before payloads reach analytical warehouses (SAP BW/4HANA).
* **Shared Runtime Footprint & Environment Agnosticism:** Standardizes dependency management via shared runtime libraries and dynamic execution-context resolution, minimizing memory/disk overhead on host servers while maintaining seamless portability across development, staging, and scheduled production environments.
* **Unified Observability & Automated Verification:** Standardizes execution telemetry and machine-readable status contracts (`YAML` receipts, exit codes, SQL/wire tracing) for external enterprise schedulers (Control-M, Airflow, SAP PI/PO), backed by isolated unit and behavioral test suites.

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

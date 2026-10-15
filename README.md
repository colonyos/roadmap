# ColonyOS Roadmap

This roadmap describes the key improvements planned for the [Colonies server](https://github.com/colonyos/colonies). It covers five themes: developer experience, configurability, performance, routing and accounting. The tables below give an overview and current status, and the sections after them describe each item: what will change, why it matters and what it should achieve.

_Last updated: 2026-10-15_

## At a glance

### Developer experience

| Item | Horizon | Status | Issue |
| --- | --- | --- | --- |
| Zero-config local development | Next | Planned | |
| Maintained dependencies | Later | Planned | |
| Standard Go project layout | Later | Planned | |
| Documentation that matches the code | Now | Planned | |

### Configurability

| Item | Horizon | Status | Issue |
| --- | --- | --- | --- |
| Pluggable database layer | Next | Planned | |
| Optional clustering | Next | Planned | |
| Feature toggles for minimal deployments | Later | Planned | |
| Execute-only executors | Later | Planned | |

### Performance

| Item | Horizon | Status | Issue |
| --- | --- | --- | --- |
| Benchmark suite | Now | In Planned | |
| Performance baseline and profiling | Now | In progress | |
| High-throughput channels | Later | Planned | |
| Batch assignment | Later | Planned | |

### Routing

| Item | Horizon | Status | Issue |
| --- | --- | --- | --- |
| Metadata-based routing | Later | Planned | |

### Accounting

| Item | Horizon | Status | Issue |
| --- | --- | --- | --- |
| Distributed resource accounting | Later | Planned | |

**Horizon** shows roughly when an item is expected to start. **Now** means it is next up or under way, **Next** means it follows once Now items land, and **Later** means it depends on earlier work. It shows order, not dates. **Status** is Planned, In progress or Done. **Issue** links to the item's tracking issue once it exists.

## Developer experience

Make ColonyOS easy to run, understand and contribute to.

### Zero-config local development

A dev mode that starts the server with a single command and no required environment variables, in a single container with an embedded database. It will use sensible defaults and print the credentials needed to connect on startup. Today a first run means setting several variables and a separate database. The goal is that a newcomer can go from clone to a completed process in under a few minutes.

### Maintained dependencies

Audit the project's third-party dependencies and replace those that are outdated, archived or abandoned with maintained alternatives. Automated dependency updates and vulnerability scanning will keep it that way. This keeps security fixes flowing and stops old dependencies from blocking Go upgrades.

### Standard Go project layout

Reorganize the repository into the conventional Go structure, with binaries in `cmd/`, implementation details in `internal/`, and only the intended public API importable. This will be done as small moves that don't change behavior. The aim is to make clear what is stable API and help contributors find their way around. It also creates clean seams for the configurability work below.

### Documentation that matches the code

Bring the README, in-repo docs, website and tutorials in line with how the code works today. Examples will be tested in CI so the docs stay accurate as the code changes. Outdated docs cost users more than missing ones, and several items on this roadmap will change setup again.

## Configurability

Let deployers run exactly what they need, from a minimal single node to a full cluster.

### Pluggable database layer

Put the database behind a storage interface, keep Postgres/Timescale as the reference implementation, and add support for at least one more engine, with SQLite as the first candidate. A shared conformance test suite will keep the engines' behavior aligned. This removes a heavy prerequisite for small deployments, edge sites and testing, and gives zero-config dev mode its embedded database.

### Optional clustering

Allow the server to run as a single standalone node with clustering turned off entirely. Many deployments have one server and gain nothing from cluster membership and leader election, but still pay for their configuration, ports and failure modes. Clustered deployments will behave exactly as before.

### Feature toggles for minimal deployments

Let deployers switch off optional subsystems, such as workflows, cron, generators, file storage, logs and channels. A disabled feature runs no background work, exposes no API routes and returns a clear error if it is called. The result is a smaller footprint and attack surface for deployments that need only core process scheduling.

### Execute-only executors

Introduce executors whose keys can pick up and complete work but cannot submit new work. Executors often run on less trusted machines, such as edge devices, partner sites or volunteer hardware. A leaked execute-only key cannot be used to inject work into the colony. Existing executors keep their current permissions.

## Performance

Measure first, then improve, with numbers anyone can reproduce.

### Benchmark suite

A reproducible benchmark harness that measures three things:

- raw throughput (assignments per unit of time)
- round-trip time from submit through assign, close and result
- channel message rate and latency

A short version will run in CI to catch regressions. Published figures also answer the first question evaluators ask: how much load can it take?

### Performance baseline and profiling

Use the benchmark suite to record a baseline for each supported configuration. Then profile the hot paths to find and rank the real bottlenecks. Every improvement afterwards is measured against this baseline, so gains are proven and regressions are caught.

### High-throughput channels

Investigate and fix the problems channels show under high message rates. The target is channels that sustain a documented message rate with bounded latency and stable memory, and that degrade predictably under overload. Channels underpin streaming use cases such as LLM token streaming and sensor data, so they need to hold up under load.

### Batch assignment

Let an executor take a batch of processes in one request instead of one at a time, and report their results in a batch as well. When processes are small and finish quickly, the round trip for each assignment can take longer than the work itself, which limits throughput and puts unnecessary load on the server. The executor chooses how many processes it can handle at once, and each process is still tracked on its own, with its own status, timeout and retries. The benchmark suite will measure the gain for small processes.

## Routing

Route work by what an executor is and has, not only by its type.

### Metadata-based routing

Today a process is routed by executor type, and executors can only be described with a fixed set of metadata fields. Anything else has to be encoded into type names. Instead, let executors carry free-form metadata defined by the operator, such as region, GPU model, data-residency zone, cost or any property that matters to that deployment.

Submitters add requirements on this metadata to a process, and the server assigns it only to executors that match. A process that no executor matches stays queued, and its status shows why it is waiting.

This lets each deployment route work by what matters to it, such as hardware-aware placement, data-residency rules or cost- and carbon-aware scheduling, without waiting for new built-in fields. Existing type-based routing keeps working unchanged, and processes without requirements behave as before.

## Accounting

Know who used what, where, when and how much.

### Distributed resource accounting

Track resource usage across distributed executors and attribute it to users, services, projects and organizations. Usage is recorded as verifiable records: executors sign what they report, and records are stored so that tampering can be detected. The data can be queried and exported. It provides a foundation for resource monitoring, showback and cost allocation, and later for chargeback or billing.

## Getting involved

Interested in one of these items? Open a discussion or issue in the [colonies repository](https://github.com/colonyos/colonies).

## Changing this roadmap

The ColonyOS maintainers keep this roadmap up to date. To propose a new item or a change to an existing one, open a discussion in the [colonies repository](https://github.com/colonyos/colonies), or a pull request against this repository. Status and horizon are updated as work moves, and the "Last updated" date changes with each revision.

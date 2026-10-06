# Maturity Model

A model for assessing how mature an app is across three independent dimensions, and what to do to move it forward.

## How to read this

- **Three dimensions, scored separately**: Engineering/Operational, Architecture, and Security. There is no single combined score — an app is e.g. "Ops: L3, Architecture: L2, Security: L4," not one blended number. Collapsing dimensions into one score hides which one actually needs attention.
- **Each dimension is a matrix of sub-categories**, each scored on its own 1–5 ladder.
- **Levels are numbers only** (1 = least mature, 5 = most mature) — no CMMI-style names.
- **A dimension's score is the weakest link**: the lowest score among its applicable sub-categories, not an average. One neglected sub-category defines the dimension, by design — averaging lets a critical gap hide behind unrelated strengths.
- **Sub-categories can be marked N/A** for an app where they genuinely don't apply (e.g. "Scalability" for a single-user CLI tool). An N/A sub-category is excluded from the weakest-link calculation, not counted as a failure. Marking something N/A requires a one-line justification — it's an assessment judgment call for now, not a fixed rule. (App-type profiles that pre-define N/A sets by archetype are a possible future addition, not yet defined.)
- **Each level is descriptive and prescriptive**: it describes what that level looks like, and names the concrete next step to reach the level above it.
- **Each level also lists `Checks`** where a real, objective signal exists — a tool output, a config file, a report — that would evidence placing an app at that level. Sub-categories that are mostly judgment calls (notably Tech Debt/Maintainability) have thinner checks; the descriptive text still carries most of the weight there.
- **Assessment today is self-assessment**, informed by the checks where they're available. Checks are evidence to consult, not a mechanical formula — two people reading the same checks should converge on the same level, but nothing here auto-computes a score yet.
- An eventual agent skill is intended to drive scoring against this model more deterministically, likely by actually running these checks. That skill's design is a separate, future exercise — this document defines the model only.

---

## Dimension: Engineering / Operational Maturity

### CI/CD

**Level 1**
No automated pipeline. Builds, tests, and deploys happen manually on a developer's machine — "deploy" means someone runs a script or uploads files by hand.
*Checks:* no CI config file present in the repo (no pipeline definition of any kind).
*→ To reach Level 2:* Set up a basic automated pipeline that at minimum builds and runs tests on every push, even if deploy is still manual.

**Level 2**
An automated pipeline builds and runs tests on every push/PR, but deployment is still a manual trigger — someone clicks a button or runs a command after checks pass.
*Checks:* CI config exists and runs build + test on push/PR; no deploy step is defined in the pipeline.
*→ To reach Level 3:* Automate deployment itself for at least one environment (e.g. staging) so a passing pipeline deploys without a human step.

**Level 3**
Push/merge triggers build, test, and automatic deploy to at least staging. Production deploys may still require a manual approval or trigger.
*Checks:* the CI pipeline includes an automated deploy step to at least one non-production environment, triggered by merge.
*→ To reach Level 4:* Automate production deploys too (with safeguards like approval gates or a canary/rollout strategy), and start tracking deploy frequency and lead time.

**Level 4**
Build, test, and deploy to production are automated (possibly behind an approval gate), deploys happen frequently (at least weekly, ideally more), and rollback is a known, practiced procedure.
*Checks:* the pipeline deploys to production (gated or ungated); deploy frequency/lead time is tracked somewhere; a documented and previously-used rollback procedure exists.
*→ To reach Level 5:* Make rollback automatic on failed health checks, and move toward continuous deployment — every commit that passes CI can reach production without a human gate.

**Level 5**
Continuous deployment: every change that passes CI can reach production automatically, as often as needed. Rollback triggers automatically on failed health checks or error-rate spikes, without waiting for a human.
*Checks:* no required manual approval gate blocks a passing pipeline from deploying to production; an automated rollback trigger (health check or error-rate based) is configured.
*→ Maintaining Level 5:* keep the pipeline fast and the tests trustworthy enough that "every commit can ship" stays true as the app grows.

### Testing

**Level 1**
No automated tests. Correctness is verified manually, if at all, before shipping.
*Checks:* no test command configured; no test step in CI (or no CI at all).
*→ To reach Level 2:* Add unit tests for the app's most critical or fragile logic — it doesn't need to be comprehensive yet, just a start.

**Level 2**
Some unit tests exist for critical logic, but coverage is partial and inconsistent. Untested paths are common, and tests aren't required to pass before merging.
*Checks:* a unit test command exists, but it isn't wired into CI as a required/blocking step; no coverage report is generated.
*→ To reach Level 3:* Require tests to pass in CI before merge, and extend coverage to the app's main business logic, not just a few critical functions.

**Level 3**
Unit tests cover the main business logic and are required to pass in CI before merge. Interactions between components are untested — there are no integration or end-to-end tests yet.
*Checks:* lint and the unit test suite both run and are required to pass on every PR; a coverage report is generated (no enforced threshold yet).
*→ To reach Level 4:* Add integration or end-to-end tests covering the app's critical user-facing flows, so component interactions are verified, not just units in isolation.

**Level 4**
Unit tests cover core logic, and integration/E2E tests cover critical user flows — all required in CI. Coverage is meaningful across the parts of the app that matter most, not just a high percentage for its own sake.
*Checks:* lint, formatter check, type-checker (if applicable), unit suite, and integration/e2e suite are all required in CI; a coverage threshold is enforced on critical paths (not just reported).
*→ To reach Level 5:* Extend testing to edge cases, failure modes, and non-happy paths (error handling, race conditions, degraded dependencies), and start tracking coverage and flakiness over time.

**Level 5**
A comprehensive suite covers happy paths, edge cases, and failure modes at unit, integration, and end-to-end levels. Flaky tests are tracked and fixed, not ignored or retried into passing.
*Checks:* all Level 4 checks, plus a mutation-testing score (e.g. Stryker/PITest) and/or CRAP score tracked over time; flaky tests are automatically detected/quarantined rather than silently retried.
*→ Maintaining Level 5:* keep the suite fast and reliable as the app grows — a slow or flaky suite quietly erodes behavior back toward Level 3/4 (skipped tests, ignored failures).

### Observability

**Level 1**
No logging or monitoring beyond what a developer sees locally during development. If the app breaks in production, you find out from a user complaint.
*Checks:* no log aggregation or monitoring service is configured for the app.
*→ To reach Level 2:* Add basic logging in production — even just captured stdout/stderr — so you can look something up after the fact.

**Level 2**
Basic logs exist in production but aren't structured or centralized, and nobody is alerted proactively — logs are only checked after someone reports a problem.
*Checks:* logs are captured in production (e.g. platform-level log capture), but no log aggregation/structuring is configured and no dashboards or alert rules exist.
*→ To reach Level 3:* Centralize logs in an aggregation service and adopt structured logging (consistent fields like request ID, severity) so logs are searchable, not just scrollable.

**Level 3**
Structured logs are centralized and searchable. Basic metrics exist (request counts, error rates, latency) on a dashboard, but nobody is paged automatically — someone has to go look.
*Checks:* logs are centralized with a consistent schema; a metrics dashboard exists; no alert rules are configured to notify anyone.
*→ To reach Level 4:* Add alerting on key metrics (error spikes, latency thresholds, downtime) so problems surface proactively instead of requiring someone to check a dashboard.

**Level 4**
Metrics and logs are centralized with dashboards, and alerts fire automatically on meaningful thresholds, routed to whoever's on call. Tracing may still be missing, making multi-service issues hard to diagnose.
*Checks:* alert rules exist on error rate/latency/availability and are routed to an on-call channel or person; dashboards and centralized structured logs are in place.
*→ To reach Level 5:* Add distributed tracing (or equivalent) so a single request's path through the system is traceable end-to-end, and tie alerts to SLOs rather than arbitrary static thresholds.

**Level 5**
Structured logs, metrics, and distributed tracing are centralized and correlated (e.g. by request/trace ID). Alerts are tied to SLOs and catch problems before users report them.
*Checks:* distributed tracing is configured and correlated with logs/metrics via a shared ID; alert thresholds are defined relative to an SLO rather than a static number.
*→ Maintaining Level 5:* keep instrumentation current as new services and integrations are added — observability gaps reappear silently at every new integration point.

### Incident response / resilience

**Level 1**
No defined process for handling failures. When something breaks, whoever notices improvises a fix, with no record of what happened or why.
*Checks:* no runbook documents exist; no record of any past incident exists anywhere.
*→ To reach Level 2:* Write a basic runbook for your most likely or most damaging failure mode — even a few bullet points is a start.

**Level 2**
Failures are handled reactively by whoever's available, with no written runbooks — knowledge lives in people's heads. No review happens after things are fixed.
*Checks:* no written runbooks exist; no post-incident review document exists for any past outage.
*→ To reach Level 3:* Write runbooks for your most common or critical failure modes, and start doing a brief post-incident writeup (what happened, why, what you'd change) after anything significant.

**Level 3**
Runbooks exist for known failure modes and are actually used during incidents. A lightweight post-incident review happens after significant outages, but follow-up actions aren't reliably tracked to completion.
*Checks:* runbook documents exist for at least the known failure modes; a post-incident writeup exists for the most recent significant incident.
*→ To reach Level 4:* Track post-incident action items to completion, and start building automated recovery for your most frequent or best-understood failure mode (e.g. auto-restart, auto-scale).

**Level 4**
Post-incident reviews reliably produce tracked follow-up actions that get completed. At least one common failure mode recovers automatically (health-check-triggered restart, auto-scaling, failover) rather than needing a human every time.
*Checks:* post-incident action items are tracked in an issue tracker through to completion; at least one failure mode has automated recovery configured.
*→ To reach Level 5:* Extend automated recovery to most known failure modes, and start designing for failure proactively (chaos testing, graceful degradation) rather than only responding well after the fact.

**Level 5**
Most known failure modes recover automatically without human intervention. The app is proactively tested against failure (chaos or fault injection) rather than only learning from real incidents.
*Checks:* most known failure modes have automated recovery configured; a chaos/fault-injection test has been run and its result recorded.
*→ Maintaining Level 5:* keep expanding automated-recovery coverage as new failure modes are discovered — each new incident is a candidate for automation, not just a one-off fix.

---

## Dimension: Architecture Maturity

### Coupling / modularity

**Level 1**
The app is a single undifferentiated mass of code — no clear module boundaries, business logic mixed with UI/infra code, and changes in one area routinely break unrelated areas.
*Checks:* no module/package boundaries exist in the codebase structure; a dependency-cycle scan (e.g. madge, dep-cruiser), if run, would show pervasive cycles.
*→ To reach Level 2:* Identify and separate the most obviously distinct concerns (e.g. pull data-access code out of UI handlers) into their own modules, even without enforcing boundaries yet.

**Level 2**
Some separation exists (e.g. folders for different concerns), but boundaries aren't enforced — modules reach into each other's internals freely, and it's unclear what depends on what without reading all the code.
*Checks:* folder/module separation exists, but no dependency-cycle or layer-boundary scan has been run, or one run shows internals imported directly across modules.
*→ To reach Level 3:* Define explicit module boundaries with clear public interfaces, no reaching into internals, and make dependencies between modules intentional and visible.

**Level 3**
Clear module boundaries exist with defined interfaces; modules don't reach into each other's internals. Dependencies between modules are intentional, but the app is still one deployable unit — a modular monolith, not services.
*Checks:* a dependency-cycle/layer-boundary tool (madge, dep-cruiser, ArchUnit-style) has been run at least occasionally and shows no cycles across defined modules.
*→ To reach Level 4:* Identify which modules would actually benefit from independent deployability (different scaling needs, release cadence, or ownership) and extract only those into separate services.

**Level 4**
Where it genuinely pays off, components are independently deployable with clear contracts between them; the rest remains a sensibly modular monolith rather than being split for its own sake.
*Checks:* the dependency-boundary check runs in CI and is enforced; at least one component is deployed and versioned independently from the rest.
*→ To reach Level 5:* Keep verifying that service boundaries reflect real seams (ownership, scaling, failure domains) as the system grows, and make contracts between them explicit and versioned.

**Level 5**
Module and service boundaries map cleanly to real seams, contracts between them are explicit and versioned, and the system can evolve — splitting or merging components — without widespread breakage because coupling is genuinely low.
*Checks:* dependency-boundary checks run in CI on every change; service/module contracts are versioned (e.g. versioned API schemas) and checked for breaking changes.
*→ Maintaining Level 5:* revisit boundaries as the app grows — yesterday's correct seam can become tomorrow's artificial split or tomorrow's tangled mess.

### Scalability

**Level 1**
The app assumes a single instance — in-memory state, local file storage, or hardcoded assumptions that would break if you ran two copies at once.
*Checks:* no load test has ever been run; the app's configuration assumes a single instance (e.g. in-memory session store, local disk writes).
*→ To reach Level 2:* Identify what's keeping the app single-instance (in-memory sessions, local disk writes, etc.) and note it, even before fixing it.

**Level 2**
You know what would break under load or with multiple instances, but haven't addressed it — the app still only really works as a single instance today.
*Checks:* single-instance blockers are identified and documented somewhere, but no fix has shipped — only one instance can run correctly.
*→ To reach Level 3:* Remove the biggest single-instance blockers (e.g. move session state to a shared store, move file storage off local disk) so the app can run as two or more instances behind a load balancer.

**Level 3**
The app can run as multiple instances behind a load balancer — state that needs sharing lives in a shared store (DB, cache, object storage), not local memory or disk. Scaling is still manual.
*Checks:* the app is deployed as two or more instances behind a load balancer; shared state lives in a shared store rather than local memory/disk.
*→ To reach Level 4:* Add autoscaling based on load (CPU, request rate, queue depth) so capacity adjusts without a human deciding to scale, and identify the next bottleneck (e.g. a single database) before it becomes limiting.

**Level 4**
The app autoscales based on real load signals. You've identified and addressed the next-biggest bottleneck beyond app instances themselves — e.g. database read replicas, a caching layer, or queue-based load leveling.
*Checks:* an autoscaling policy is configured and active (scales on CPU/request rate/queue depth); the next-biggest bottleneck has a documented mitigation in place (replica, cache, queue).
*→ To reach Level 5:* Load-test to know actual capacity limits and headroom rather than assuming scaling "just works," and make sure every layer — not just app instances — can scale, including data stores.

**Level 5**
Scalability is verified, not assumed — the app has been load-tested to know its real limits, and every layer (app, cache, data store, queues) can scale to meet demand, with headroom tracked and acted on before limits are hit.
*Checks:* a load-test report exists showing actual capacity/headroom; every layer (app, cache, data store, queue) has a verified scaling path.
*→ Maintaining Level 5:* re-test after significant feature or traffic-pattern changes — past load tests go stale as usage evolves.

### Resilience / fault tolerance

**Level 1**
A failure in any dependency (database, third-party API, downstream service) takes the whole app down or hangs it indefinitely — no timeouts, no handling of partial failure.
*Checks:* no timeout configuration exists on outbound calls to dependencies (default/infinite timeouts).
*→ To reach Level 2:* Add basic timeouts to external calls so a hung dependency doesn't hang your whole app indefinitely.

**Level 2**
External calls have timeouts, but a failing dependency still causes errors to cascade through the app with no retry or fallback — a flaky downstream service degrades your whole app's reliability one-to-one.
*Checks:* timeouts are configured on external calls; no retry/backoff logic is present anywhere.
*→ To reach Level 3:* Add retries with backoff for transient failures, and decide which failures should fail fast versus be retried.

**Level 3**
Timeouts and retries with backoff reduce the impact of transient failures. A sustained outage in a dependency still cascades — there's no circuit breaking or fallback behavior yet.
*Checks:* retry-with-backoff logic is present for transient failures on at least the most critical external calls; no circuit breaker is configured.
*→ To reach Level 4:* Add circuit breakers (or equivalent) so a sustained dependency outage fails fast instead of cascading, and define fallback or degraded behavior for at least your most critical dependency.

**Level 4**
Circuit breakers (or equivalent) prevent a sustained dependency outage from cascading; at least the most critical dependency has defined degraded or fallback behavior (e.g. serve stale cache, disable a feature) rather than a hard failure.
*Checks:* a circuit breaker (or equivalent) is configured for at least the most critical dependency; documented fallback/degraded behavior exists for it.
*→ To reach Level 5:* Extend graceful degradation to all significant dependencies, and verify resilience behavior under real or simulated failure (fault injection) rather than just trusting the design.

**Level 5**
The app degrades gracefully under failure of any significant dependency — fallback behavior is defined and tested, not just assumed. Resilience is verified via fault injection or chaos testing, not only reasoned about on paper.
*Checks:* fallback/degraded behavior is defined for all significant dependencies; a fault-injection/chaos test covering dependency failure has been run and its result recorded.
*→ Maintaining Level 5:* re-verify resilience behavior as new dependencies are added — each new integration is a new potential cascading-failure point until proven otherwise.

### Tech debt / maintainability

> **Note on measurability:** this sub-category is the hardest to score objectively — there's no single metric that captures "tech debt." The checks below give partial signals at best; placing an app on this ladder will likely stay mostly judgment-based for a while, possibly aided by purpose-built tooling later.

**Level 1**
The codebase is difficult and risky to change — little to no structure or documentation, changes routinely have unexpected side effects, and nobody is confident making changes without extensive manual verification.
*Checks:* none run — if a static-analysis/complexity tool were run, it would likely show pervasive high complexity and low structure, but absence of tooling is itself the signal here.
*→ To reach Level 2:* Start documenting the riskiest or most-touched areas (even a short comment or doc explaining "why," not "what"), so the most dangerous parts of the code are at least known.

**Level 2**
The riskiest areas are identified and loosely documented, but the codebase as a whole remains hard to change confidently — naming, structure, and typing are inconsistent, and changes still require careful manual care.
*Checks:* no static-analysis/complexity-and-duplication tool has been run, or one has been run but findings aren't tracked anywhere.
*→ To reach Level 3:* Introduce consistent conventions (naming, structure, typing where applicable) going forward, and pay down debt incrementally in files you're already touching rather than letting it accumulate.

**Level 3**
The codebase follows consistent conventions and has reasonable structure; most changes can be made with a normal, non-heroic amount of care. Known areas of debt are tracked, even informally, not just discovered by surprise.
*Checks:* a static-analysis/complexity scan has been run at least once, and known hotspots are tracked somewhere (even an informal backlog list).
*→ To reach Level 4:* Make debt visible and prioritized (even an informal running list), budget time to actually pay it down rather than only tracking it, and lean on type safety or tests to make refactors safe rather than nerve-wracking.

**Level 4**
Tech debt is tracked and actively paid down, not just documented. Type safety and/or test coverage make most refactors safe to attempt without fear. The codebase is generally pleasant to work in, with occasional known rough edges.
*Checks:* a static-analysis tool runs regularly (not necessarily blocking CI) and tracked debt items are being closed over time; dependency staleness (outdated major versions) is monitored.
*→ To reach Level 5:* Treat debt paydown as routine and continuous rather than a periodic push, and keep the codebase safe to refactor as it grows — maintainability is a property you actively protect, not one you periodically rescue.

**Level 5**
Maintainability is actively protected as a first-class concern — refactoring is routine and low-risk, debt doesn't silently accumulate because it's paid down continuously, and the codebase stays approachable even as it grows.
*Checks:* static analysis runs continuously (e.g. in CI, non-blocking) with a tracked maintainability trend over time; debt paydown is a recurring, scheduled activity, not sporadic.
*→ Maintaining Level 5:* keep watching for quiet regression — maintainability erodes gradually, and this sub-category is the easiest one to silently slip backward on.

---

## Dimension: Security Maturity

### AuthN / AuthZ

**Level 1**
No real authentication — shared credentials, no login system, or auth that's trivially bypassable (e.g. checks enforced only client-side).
*Checks:* no server-side authentication check exists anywhere, or authorization is enforced only in client-side code.
*→ To reach Level 2:* Implement real server-side authentication, even a basic username/password or managed auth provider, so identity is actually verified.

**Level 2**
Server-side authentication exists and is reasonably sound, but there's no authorization model — any authenticated user can access any data or action, regardless of whether they should be able to.
*Checks:* server-side authentication exists; no role or permission checks exist beyond "is logged in."
*→ To reach Level 3:* Introduce an authorization model, even coarse roles like admin/user, so authenticated users are restricted to what they should actually be able to do.

**Level 3**
Authentication plus coarse-grained authorization (e.g. role-based) restrict access appropriately for most cases. Edge cases — like accessing another user's specific resource by ID — may not be fully locked down.
*Checks:* role-based checks (e.g. admin/user) are present in server-side code; resource-level ownership checks are inconsistent or missing in at least some endpoints.
*→ To reach Level 4:* Close resource-level authorization gaps so every resource access checks ownership or permission, not just role, and consider MFA for sensitive accounts or actions.

**Level 4**
Fine-grained, resource-level authorization is enforced consistently — no "forgot to check ownership" gaps — and sensitive accounts or actions support MFA. Access follows least-privilege by default.
*Checks:* resource-level authorization checks are present on all sensitive endpoints/actions; MFA is available or configurable for sensitive accounts.
*→ To reach Level 5:* Push toward continuous verification where it matters, not just at login, and regularly audit access patterns and permissions rather than setting them once and forgetting them.

**Level 5**
Least-privilege, fine-grained authorization is enforced everywhere, MFA is available or required where appropriate, and access and permissions are periodically audited rather than set-and-forget. Authorization logic is centralized and consistent, not reimplemented ad hoc per feature.
*Checks:* a permission/access audit has been performed and is repeated on a schedule; authorization logic lives in a shared/centralized policy layer rather than duplicated per feature; MFA is enforced (not just available) for sensitive actions.
*→ Maintaining Level 5:* re-audit authorization logic whenever new resource types or roles are added — this is where gaps quietly reappear.

### Secrets management

**Level 1**
Secrets (API keys, DB passwords, tokens) are hardcoded in source or committed to the repo — plaintext in code, config files checked into git, or shared ad hoc via chat/email. No awareness of exposure risk.
*Checks:* a secret-scanning tool (gitleaks/trufflehog or equivalent) run against repo history would find committed credentials; no scanning is currently configured.
*→ To reach Level 2:* Remove secrets from source control, move them to environment variables or an untracked config file, and rotate anything that was ever committed — a committed secret is compromised the moment it's pushed, regardless of whether the repo is public.

**Level 2**
Secrets live outside source control — environment variables, untracked `.env` files, or the hosting platform's env-var config. No rotation process; a given secret may live unchanged for the app's entire lifetime. Access isn't restricted beyond whoever already has deploy access.
*Checks:* no secrets manager/vault is in use; secrets are configured via plain environment variables or untracked files; no rotation schedule is documented anywhere.
*→ To reach Level 3:* Move secrets into a dedicated secrets manager or vault, not just env vars, and scope access per service/environment so prod secrets aren't reachable from dev/staging by default.

**Level 3**
Secrets are stored in a dedicated secrets manager/vault with access scoped per service/environment. No enforced rotation schedule yet — secrets are correct today but could be years old.
*Checks:* secrets are stored in a dedicated secrets manager/vault; access scoping per environment is configured; no rotation automation is configured.
*→ To reach Level 4:* Introduce scheduled or automated rotation for at least high-risk secrets (DB credentials, third-party API keys), and add CI-level scanning (e.g. git-secrets, trufflehog) to catch accidental commits before merge.

**Level 4**
Secrets are vaulted, access-scoped, and rotated on a defined schedule for all credentials, not just the high-risk ones. CI scans every push/PR for accidentally committed secrets. A leak is caught within hours by tooling, not discovered by accident weeks later.
*Checks:* secret-scanning runs in CI on every push/PR; a rotation schedule is configured and followed for all credentials, not just high-risk ones.
*→ To reach Level 5:* Automate rotation end-to-end with no manual steps, and move toward short-lived, dynamically issued credentials rather than long-lived static values.

**Level 5**
Secrets are short-lived and dynamically issued wherever the platform supports it (workload identity, dynamic DB credentials, OIDC-based cloud auth) rather than durable static values. Rotation is fully automated; secretless/zero-trust patterns are used where feasible. A leaked credential has minimal blast radius because it expires quickly or was never a long-lived value to begin with.
*Checks:* dynamic/short-lived credential issuance is configured (e.g. workload identity, OIDC-based cloud auth, dynamic DB credentials) wherever the platform supports it; rotation is fully automated with no manual step.
*→ Maintaining Level 5:* re-audit as new integrations are added — each new third-party service is a chance to regress to a static long-lived key.

### Dependency & vulnerability management

**Level 1**
No visibility into dependency vulnerabilities — dependencies are added and rarely if ever updated; you'd only find out about a known CVE in something you use by accident.
*Checks:* no dependency audit/SCA tool has ever been run against the project.
*→ To reach Level 2:* Run a dependency audit tool manually at least occasionally (e.g. an `audit` command for your package manager) so known vulnerabilities are at least visible.

**Level 2**
You can check for known vulnerabilities manually, but there's no regular cadence — checks happen sporadically if at all, and there's no process for actually acting on what's found.
*Checks:* a dependency audit tool has been run manually at least once, but not on a schedule; no CI integration exists.
*→ To reach Level 3:* Automate vulnerability scanning in CI, failing or warning on known CVEs above a severity threshold, so visibility doesn't depend on remembering to check.

**Level 3**
Automated dependency/vulnerability scanning runs in CI on every build, surfacing known CVEs. There's no defined SLA for acting on findings — critical vulnerabilities might sit unpatched for a while after being flagged.
*Checks:* an SCA/vulnerability scanner (Dependabot alerts, Snyk, OSV-Scanner, or equivalent) runs in CI on every build; no patch SLA is documented.
*→ To reach Level 4:* Define and follow a patch SLA by severity (e.g. critical within days, not months), and automate dependency update PRs (e.g. Dependabot/Renovate) so patching isn't fully manual.

**Level 4**
Automated scanning plus a followed patch SLA by severity; dependency updates are at least partly automated via bot-created PRs, reducing the lag between a CVE being known and being fixed.
*Checks:* a patch SLA by severity is documented and being followed; automated dependency-update PRs (Dependabot/Renovate or equivalent) are enabled.
*→ To reach Level 5:* Ensure the update pipeline is trusted enough to merge low-risk updates with minimal manual gatekeeping, and extend scanning beyond direct dependencies to transitive ones and to runtime/container images if applicable.

**Level 5**
Vulnerability management is close to continuous — scanning covers direct and transitive dependencies (and container/runtime images where relevant), patch SLAs are consistently met, and low-risk updates can merge with minimal manual friction.
*Checks:* SCA coverage extends to transitive dependencies and container/runtime images (e.g. Trivy/Grype scan); an SBOM is generated; low-risk automated update PRs merge with minimal manual review.
*→ Maintaining Level 5:* keep scanning coverage current as new dependency types are introduced — new package ecosystems, new base images, and so on.

### Data protection

**Level 1**
No encryption — data is transmitted and/or stored in plaintext; sensitive data (PII, credentials) isn't distinguished from anything else.
*Checks:* at least one endpoint is reachable over plain HTTP, or data at rest has no encryption configured.
*→ To reach Level 2:* Enable transport encryption everywhere — HTTPS/TLS for all traffic, no plaintext endpoints — as a baseline.

**Level 2**
Transport encryption is enforced everywhere, but data at rest is unencrypted, or relies entirely on the database provider's defaults without a conscious decision, and PII isn't specifically identified or handled differently from other data.
*Checks:* HTTPS/TLS is enforced on all endpoints with no plaintext fallback; data-at-rest encryption is unconfirmed or default/unknown; no PII inventory exists.
*→ To reach Level 3:* Identify what data is actually sensitive or PII, and ensure data at rest is encrypted, whether via a deliberate app-level choice or a consciously verified platform default.

**Level 3**
Data at rest is encrypted and sensitive/PII fields are identified, but handling is still fairly uniform — no clear retention policy, and PII isn't specially restricted beyond general access controls.
*Checks:* data-at-rest encryption is explicitly confirmed/configured (not just assumed); a list of sensitive/PII fields has been documented.
*→ To reach Level 4:* Define a retention policy — how long data is kept, when it's deleted — and apply extra restrictions or minimization specifically to PII (e.g. field-level access control, masking in logs).

**Level 4**
PII has a defined retention policy and is handled with extra care — minimized where possible, masked in logs, and access-restricted beyond general authorization. Data at rest and in transit are both encrypted.
*Checks:* a data retention policy is documented; PII fields are masked in logs and access-restricted beyond general auth controls.
*→ To reach Level 5:* Formalize this into an auditable process — a maintained data inventory, retention enforcement that actually runs rather than a policy on paper — and consider deletion/right-to-erasure handling if relevant to your users.

**Level 5**
Data protection is systematic and auditable — a maintained inventory of what sensitive data exists and where, enforced (not just documented) retention and deletion, encryption at rest and in transit, and PII handled with deliberate minimization throughout the app.
*Checks:* a maintained data inventory exists mapping sensitive data to storage locations; retention/deletion is enforced by an automated job, not just a written policy; a DAST scan has been run and recorded.
*→ Maintaining Level 5:* re-audit the data inventory whenever new data types or integrations are added — new fields quietly become undocumented PII surface area otherwise.

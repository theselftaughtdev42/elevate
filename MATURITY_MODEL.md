# Maturity Model

A model for assessing how mature an app is across three independent dimensions, and what to do to move it forward.

This is the **master model**. Smaller app-type models (e.g. CLI tool, static website, dynamic website) are extracted from it as subsets — see [How app-type models relate to this one](#how-app-type-models-relate-to-this-one).

## How to read this

### Structure

- **Three dimensions, scored separately**: Engineering/Operational, Architecture, and Security. There is no single combined score — an app is e.g. "Ops: L3, Architecture: L2, Security: L4," not one blended number. Collapsing dimensions into one score hides which one actually needs attention.
- **Each dimension is a matrix of sub-categories**, each scored on its own 1–5 ladder.
- **Levels are numbers only** (1 = least mature, 5 = most mature) — no CMMI-style names.
- **Each level is descriptive and prescriptive**: it describes what that level looks like, and names the concrete next step to reach the level above it.
- **Scores are per app.** The model is a tool for improving one app at a time. It can be applied to many apps, but scores aren't aggregated across them.

### Scoring

- **A sub-category's level is cumulative**: an app is at level N when **all checks at levels 2 through N** are met. Level 1 has no checks — it's the floor every app starts from.
- **Partially meeting a level doesn't count**: an app that meets 3 of 4 Level 3 checks is at Level 2, and the unmet check is its first next step. There are no half levels.
- **Level descriptions say what an app at that level typically lacks; checks never do.** Every check is a positive signal — something that exists or is true. A capability from a higher level never stops an app from reaching a lower one.
- **A dimension's score is the weakest link**: the lowest level among its applicable sub-categories, not an average. One neglected sub-category defines the dimension, by design — averaging lets a critical gap hide behind unrelated strengths.

### Applicability

- **Some sub-categories are conditional**, marked with an *Applies when* line (e.g. "the app has user accounts or protected actions"). If the condition doesn't hold, the sub-category is N/A for that app.
- **Any other sub-category can be marked N/A** where it genuinely doesn't apply. An N/A sub-category is excluded from the weakest-link calculation, not counted as a failure. Marking something N/A without a conditional to back it requires a one-line justification.

### Checks

- **Every check has a stable ID** of the form `DIMENSION.SUBCATEGORY.Level.cN` — e.g. `OPS.CICD.L3.c1`.
- **Every check is labelled by how it's validated:**
  - `continuous` — enforced automatically on every change, or watched by monitoring (e.g. secret scanning in CI, an alert rule). It's true as long as the automation is in place.
  - `periodic` — true as of the last time it was done, then goes stale (e.g. a load test, an access audit). Each carries a default **freshness window**, written like `periodic ≤6mo`. The check counts if its **most recent successful run is inside the window**, whoever or whatever ran it. A scheduled job is the preferred way to keep meeting it; if the job silently stops, the window expires and the check stops counting.
  - `manual` — needs a person's judgement, or confirms that a document or practice exists.
- **Automation is a requirement only where it's the practice itself** — e.g. "secret scanning runs on every push" at Secrets L4. There's no blanket rule that high levels must be automated; the goal is that wherever a check *can* be continually validated, it is.
- **Checks name the signal, not the tool.** Example tools are given with "or equivalent"; any tool that produces the same signal counts.
- **Platform defaults count only when consciously verified.** A capability a hosting platform provides out of the box (autoscaling, encryption at rest, TLS) meets a check only if someone has confirmed it's configured as needed — not if it's merely assumed.

### Assessment

- **Assessment today is self-assessment**, using the checks as evidence. Two people reading the same checks should converge on the same level.
- **Each assessment records**: assessor, date, the level per sub-category with links to evidence for each check met, and N/A justifications.
- An eventual agent skill is intended to drive scoring against this model more deterministically, by running `continuous` checks, reading the last run date of `periodic` ones, and asking a person about `manual` ones. That skill's design is a separate, future exercise — this document defines the model only.

### How app-type models relate to this one

App-type models are **subsets** of this master: they select sub-categories by ID and may preset N/A decisions. They don't rewrite level text, checks or freshness windows. The *Applies when* conditionals here are the starting point for deciding which sub-categories each app type includes.

---

## Dimension: Engineering / Operational Maturity (`OPS`)

### CI/CD (`OPS.CICD`)

**Level 1**
No automated pipeline. Builds, tests, and deploys happen manually on a developer's machine — "deploy" means someone runs a script or uploads files by hand.
*→ To reach Level 2:* Set up a basic automated pipeline that at minimum builds and runs tests on every push, even if deploy is still manual.

**Level 2**
An automated pipeline builds and runs tests on every push/PR, but deployment is still a manual trigger — someone clicks a button or runs a command after checks pass.
*Checks:*
- `OPS.CICD.L2.c1` · `continuous` — a CI pipeline definition exists and runs build + test on every push/PR.

*→ To reach Level 3:* Automate deployment itself for at least one environment (e.g. staging) so a passing pipeline deploys without a human step.

**Level 3**
Push/merge triggers build, test, and automatic deploy to at least staging. Production deploys may still require a manual trigger outside the pipeline.
*Checks:*
- `OPS.CICD.L3.c1` · `continuous` — the pipeline automatically deploys to at least one non-production environment on merge.

*→ To reach Level 4:* Automate production deploys too (with safeguards like approval gates or a canary/rollout strategy), and start tracking the four DORA metrics: deploy frequency, lead time for changes, change failure rate, and time to restore.

**Level 4**
Build, test, and deploy to production are automated (possibly behind an approval gate), deploys happen frequently (at least weekly, ideally more), the DORA metrics are tracked, and rollback is a known, practiced procedure.
*Checks:*
- `OPS.CICD.L4.c1` · `continuous` — the pipeline deploys to production (gated or ungated).
- `OPS.CICD.L4.c2` · `manual` — deploy frequency, lead time for changes, change failure rate, and time to restore are tracked somewhere.
- `OPS.CICD.L4.c3` · `periodic ≤6mo` — a documented rollback procedure exists and has been exercised.

*→ To reach Level 5:* Make rollback automatic on failed health checks, and move toward continuous deployment — every commit that passes CI can reach production without a human gate.

**Level 5**
Continuous deployment: every change that passes CI can reach production automatically, as often as needed. Rollback triggers automatically on failed health checks or error-rate spikes, without waiting for a human.
*Checks:*
- `OPS.CICD.L5.c1` · `continuous` — a passing pipeline deploys to production with no required manual approval gate.
- `OPS.CICD.L5.c2` · `continuous` — an automated rollback trigger (health-check or error-rate based) is configured.

*→ Maintaining Level 5:* keep the pipeline fast and the tests trustworthy enough that "every commit can ship" stays true as the app grows.

### Testing (`OPS.TEST`)

**Level 1**
No automated tests. Correctness is verified manually, if at all, before shipping.
*→ To reach Level 2:* Add unit tests for the app's most critical or fragile logic — it doesn't need to be comprehensive yet, just a start.

**Level 2**
Some unit tests exist for critical logic, but coverage is partial and inconsistent. Untested paths are common, and tests aren't required to pass before merging.
*Checks:*
- `OPS.TEST.L2.c1` · `manual` — a unit test command exists and runs the app's tests successfully.

*→ To reach Level 3:* Require tests to pass in CI before merge, and extend coverage to the app's main business logic, not just a few critical functions.

**Level 3**
Unit tests cover the main business logic and are required to pass in CI before merge. Interactions between components are typically untested — there are no integration or end-to-end tests yet.
*Checks:*
- `OPS.TEST.L3.c1` · `continuous` — the unit suite runs in CI and is a required check for merge.
- `OPS.TEST.L3.c2` · `continuous` — a coverage report is generated in CI.

*→ To reach Level 4:* Add integration or end-to-end tests covering the app's critical user-facing flows, so component interactions are verified, not just units in isolation.

**Level 4**
Unit tests cover core logic, and integration/E2E tests cover critical user flows — all required in CI. Coverage is meaningful across the parts of the app that matter most, not just a high percentage for its own sake.
*Checks:*
- `OPS.TEST.L4.c1` · `continuous` — an integration/E2E suite covering critical user flows runs in CI and is a required check for merge.
- `OPS.TEST.L4.c2` · `continuous` — a coverage threshold is enforced (not just reported) on critical paths.

*→ To reach Level 5:* Extend testing to edge cases, failure modes, and non-happy paths (error handling, race conditions, degraded dependencies), and start tracking test quality and flakiness over time.

**Level 5**
A comprehensive suite covers happy paths, edge cases, and failure modes at unit, integration, and end-to-end levels. Flaky tests are tracked and fixed, not ignored or retried into passing.
*Checks:*
- `OPS.TEST.L5.c1` · `continuous` — a mutation-testing score (e.g. Stryker/PITest or equivalent) and/or CRAP score is tracked over time.
- `OPS.TEST.L5.c2` · `continuous` — flaky tests are automatically detected and quarantined, not silently retried.
- `OPS.TEST.L5.c3` · `manual` — tests exist for failure modes (error handling, degraded dependencies), not only happy paths.

*→ Maintaining Level 5:* keep the suite fast and reliable as the app grows — a slow or flaky suite quietly erodes behavior back toward Level 3/4 (skipped tests, ignored failures).

### Observability (`OPS.OBS`)

**Level 1**
No logging or monitoring beyond what a developer sees locally during development. If the app breaks in production, you find out from a user complaint.
*→ To reach Level 2:* Capture production logs (even just stdout/stderr), add error tracking for unhandled exceptions, and set up an external uptime check so a full outage doesn't depend on a user noticing.

**Level 2**
Production logs are captured, unhandled errors are tracked, and an uptime check flags a full outage. Logs aren't structured or centralized, and nothing alerts on degradation short of the app being down — logs are only checked after someone reports a problem.
*Checks:*
- `OPS.OBS.L2.c1` · `continuous` — production logs are captured and retained (platform-level capture counts).
- `OPS.OBS.L2.c2` · `continuous` — error tracking (e.g. Sentry or equivalent) captures unhandled exceptions in production.
- `OPS.OBS.L2.c3` · `continuous` — an external uptime/synthetic check monitors the app and notifies someone when it's down.

*→ To reach Level 3:* Centralize logs in an aggregation service and adopt structured logging (consistent fields like request ID, severity) so logs are searchable, not just scrollable, and put basic metrics on a dashboard.

**Level 3**
Structured logs are centralized and searchable. Basic metrics exist (request counts, error rates, latency) on a dashboard, but nobody is paged on degradation — someone has to go look.
*Checks:*
- `OPS.OBS.L3.c1` · `continuous` — logs are centralized in an aggregation service with a consistent structured schema (e.g. request ID, severity).
- `OPS.OBS.L3.c2` · `continuous` — a dashboard shows request count, error rate, and latency.

*→ To reach Level 4:* Add alerting on key metrics (error spikes, latency thresholds, availability) so problems surface proactively instead of requiring someone to check a dashboard.

**Level 4**
Metrics and logs are centralized with dashboards, and alerts fire automatically on meaningful thresholds, routed to whoever's on call. Tracing may still be missing, making multi-service issues hard to diagnose.
*Checks:*
- `OPS.OBS.L4.c1` · `continuous` — alert rules exist on error rate, latency, and availability, routed to an on-call channel or person.

*→ To reach Level 5:* Add distributed tracing (or equivalent) so a single request's path through the system is traceable end-to-end, and tie alerts to SLOs rather than arbitrary static thresholds.

**Level 5**
Structured logs, metrics, and distributed tracing are centralized and correlated (e.g. by request/trace ID). Alerts are tied to SLOs and catch problems before users report them.
*Checks:*
- `OPS.OBS.L5.c1` · `continuous` — distributed tracing is configured and correlated with logs/metrics via a shared ID.
- `OPS.OBS.L5.c2` · `continuous` — alert thresholds are defined relative to an SLO rather than a static number.

*→ Maintaining Level 5:* keep instrumentation current as new services and integrations are added — observability gaps reappear silently at every new integration point.

### Incident response (`OPS.INC`)

Covers how people detect, respond to, and learn from **operational** failures. How the system itself behaves under failure (automated recovery, fault injection) belongs to [Resilience](#resilience--fault-tolerance-archres).

**Level 1**
No defined process for handling failures. When something breaks, whoever notices improvises a fix, with no record of what happened or why, and no one is clearly responsible.
*→ To reach Level 2:* Name an owner or on-call contact for the app, and start recording incidents somewhere — even a chat thread or a running log.

**Level 2**
Someone is clearly responsible when things break, and incidents leave a record. Failures are still handled reactively, with no written runbooks — knowledge lives in people's heads — and no review happens after things are fixed.
*Checks:*
- `OPS.INC.L2.c1` · `manual` — a named owner or on-call contact for the app is documented.
- `OPS.INC.L2.c2` · `manual` — past incidents are recorded somewhere (chat thread, log, issue tracker).

*→ To reach Level 3:* Write runbooks for your most common or critical failure modes, and start doing a brief post-incident writeup (what happened, why, what you'd change) after anything significant.

**Level 3**
Runbooks exist for known failure modes and are actually used during incidents. A lightweight post-incident review happens after significant outages, but follow-up actions aren't reliably tracked to completion.
*Checks:*
- `OPS.INC.L3.c1` · `manual` — runbooks exist for at least the known failure modes.
- `OPS.INC.L3.c2` · `manual` — a post-incident writeup exists for the most recent significant incident.

*→ To reach Level 4:* Track post-incident action items to completion, and define an on-call rotation with an escalation path so response doesn't depend on whoever happens to be around.

**Level 4**
Post-incident reviews reliably produce tracked follow-up actions that get completed. On-call is a defined rotation with an escalation path, not a single person or best effort.
*Checks:*
- `OPS.INC.L4.c1` · `manual` — post-incident action items are tracked in an issue tracker through to completion.
- `OPS.INC.L4.c2` · `manual` — an on-call rotation and escalation path are defined.

*→ To reach Level 5:* Track time to restore and drive it down, and rehearse incident response through scheduled game days rather than only learning from real incidents.

**Level 5**
Incident response is measured and practiced: time to restore is tracked and trending down, and the team rehearses its response through game days so a real incident isn't the first time a runbook gets used.
*Checks:*
- `OPS.INC.L5.c1` · `manual` — time to restore is tracked per incident and trending down.
- `OPS.INC.L5.c2` · `periodic ≤6mo` — a game day rehearsing human incident response has been run and its outcome recorded.

*→ Maintaining Level 5:* keep runbooks and game-day scenarios current as the system changes — a rehearsal against last year's architecture builds false confidence.

---

## Dimension: Architecture Maturity (`ARCH`)

### Coupling / modularity (`ARCH.COUP`)

**Level 1**
The app is a single undifferentiated mass of code — no clear module boundaries, business logic mixed with UI/infra code, and changes in one area routinely break unrelated areas.
*→ To reach Level 2:* Identify and separate the most obviously distinct concerns (e.g. pull data-access code out of UI handlers) into their own modules, even without enforcing boundaries yet.

**Level 2**
Some separation exists (e.g. folders for different concerns), but boundaries aren't enforced — modules reach into each other's internals freely, and it's unclear what depends on what without reading all the code.
*Checks:*
- `ARCH.COUP.L2.c1` · `manual` — the codebase is organized into folders/modules by concern.

*→ To reach Level 3:* Define explicit module boundaries with clear public interfaces, no reaching into internals, and make dependencies between modules intentional and visible.

**Level 3**
Clear module boundaries exist with defined interfaces; modules don't reach into each other's internals. Dependencies between modules are intentional, but the app is typically still one deployable unit — a modular monolith, not services.
*Checks:*
- `ARCH.COUP.L3.c1` · `periodic ≤6mo` — a dependency-cycle/layer-boundary scan (e.g. madge, dependency-cruiser, ArchUnit or equivalent) shows no cycles or internal imports across defined modules.

*→ To reach Level 4:* Enforce the boundary check in CI, and identify which modules (if any) would genuinely benefit from independent deployability (different scaling needs, release cadence, or ownership).

**Level 4**
Boundaries are enforced automatically. Where it genuinely pays off, components are independently deployable with clear contracts between them; the rest remains a sensibly modular monolith rather than being split for its own sake.
*Checks:*
- `ARCH.COUP.L4.c1` · `continuous` — the dependency-boundary check runs in CI and is enforced.
- `ARCH.COUP.L4.c2` · `manual` — each independently deployed component has a documented reason (scaling, release cadence, ownership), or there's a documented decision that none warrants splitting.

*→ To reach Level 5:* Keep verifying that boundaries reflect real seams (ownership, scaling, failure domains) as the system grows, and make contracts between components explicit and versioned.

**Level 5**
Module and service boundaries map cleanly to real seams, contracts between them are explicit and versioned, and the system can evolve — splitting or merging components — without widespread breakage because coupling is genuinely low.
*Checks:*
- `ARCH.COUP.L5.c1` · `continuous` — contracts between modules/services are versioned (e.g. versioned API schemas) and checked for breaking changes in CI.

*→ Maintaining Level 5:* revisit boundaries as the app grows — yesterday's correct seam can become tomorrow's artificial split or tomorrow's tangled mess.

### Scalability (`ARCH.SCALE`)

*Applies when:* the app serves concurrent requests or load as a running service.

**Level 1**
The app assumes a single instance — in-memory state, local file storage, or hardcoded assumptions that would break if you ran two copies at once.
*→ To reach Level 2:* Identify what's keeping the app single-instance (in-memory sessions, local disk writes, etc.) and note it, even before fixing it.

**Level 2**
You know what would break under load or with multiple instances, but haven't addressed it — the app still only really works as a single instance today.
*Checks:*
- `ARCH.SCALE.L2.c1` · `manual` — single-instance blockers are identified and documented.

*→ To reach Level 3:* Remove the biggest single-instance blockers (e.g. move session state to a shared store, move file storage off local disk) so the app can run as two or more instances behind a load balancer.

**Level 3**
The app can run as multiple instances behind a load balancer — state that needs sharing lives in a shared store (DB, cache, object storage), not local memory or disk. Scaling is still manual.
*Checks:*
- `ARCH.SCALE.L3.c1` · `continuous` — the app is deployed as two or more instances behind a load balancer (or on a platform verified to run it multi-instance).
- `ARCH.SCALE.L3.c2` · `manual` — state that needs sharing lives in a shared store rather than local memory/disk.

*→ To reach Level 4:* Add autoscaling based on load (CPU, request rate, queue depth) so capacity adjusts without a human deciding to scale, and identify the next bottleneck (e.g. a single database) before it becomes limiting.

**Level 4**
The app autoscales based on real load signals. You've identified and addressed the next-biggest bottleneck beyond app instances themselves — e.g. database read replicas, a caching layer, or queue-based load leveling.
*Checks:*
- `ARCH.SCALE.L4.c1` · `continuous` — an autoscaling policy is configured and active, scaling on CPU, request rate, or queue depth.
- `ARCH.SCALE.L4.c2` · `manual` — the next-biggest bottleneck has a documented mitigation in place (replica, cache, queue).

*→ To reach Level 5:* Load-test to know actual capacity limits and headroom rather than assuming scaling "just works," and make sure every layer — not just app instances — can scale, including data stores.

**Level 5**
Scalability is verified, not assumed — the app has been load-tested to know its real limits, and every layer (app, cache, data store, queues) can scale to meet demand, with headroom tracked and acted on before limits are hit.
*Checks:*
- `ARCH.SCALE.L5.c1` · `periodic ≤6mo` — a load test has been run and its report shows actual capacity and headroom.
- `ARCH.SCALE.L5.c2` · `manual` — every layer (app, cache, data store, queue) has a verified scaling path.

*→ Maintaining Level 5:* re-test after significant feature or traffic-pattern changes — past load tests go stale as usage evolves.

### Resilience / fault tolerance (`ARCH.RES`)

*Applies when:* the app depends on anything it doesn't control at runtime (database, third-party API, downstream service).

Covers how the system itself behaves under failure. How people respond to failures belongs to [Incident response](#incident-response-opsinc).

**Level 1**
A failure in any dependency (database, third-party API, downstream service) takes the whole app down or hangs it indefinitely — no timeouts, no handling of partial failure.
*→ To reach Level 2:* Add basic timeouts to external calls so a hung dependency doesn't hang your whole app indefinitely.

**Level 2**
External calls have timeouts, but a failing dependency still causes errors to cascade through the app with no retry or fallback — a flaky downstream service degrades your whole app's reliability one-to-one.
*Checks:*
- `ARCH.RES.L2.c1` · `manual` — explicit timeouts are configured on all outbound calls to dependencies.

*→ To reach Level 3:* Add retries with backoff for transient failures, and decide which failures should fail fast versus be retried.

**Level 3**
Timeouts and retries with backoff reduce the impact of transient failures. A sustained outage in a dependency still cascades — there's no circuit breaking or fallback behavior yet.
*Checks:*
- `ARCH.RES.L3.c1` · `manual` — retry-with-backoff is in place for transient failures on at least the most critical external calls.

*→ To reach Level 4:* Add circuit breakers (or equivalent) so a sustained dependency outage fails fast instead of cascading, define fallback behavior for your most critical dependency, and make your most common failure mode recover automatically.

**Level 4**
Circuit breakers (or equivalent) prevent a sustained dependency outage from cascading; at least the most critical dependency has defined degraded or fallback behavior (e.g. serve stale cache, disable a feature) rather than a hard failure. At least one common failure mode recovers automatically (health-check-triggered restart, instance replacement, failover) without a human.
*Checks:*
- `ARCH.RES.L4.c1` · `manual` — a circuit breaker (or equivalent) is configured for at least the most critical dependency.
- `ARCH.RES.L4.c2` · `manual` — documented fallback/degraded behavior exists for that dependency.
- `ARCH.RES.L4.c3` · `continuous` — at least one common failure mode has automated recovery configured.

*→ To reach Level 5:* Extend graceful degradation and automated recovery to all significant dependencies and known failure modes, and verify resilience behavior under simulated failure (fault injection) rather than just trusting the design.

**Level 5**
The app degrades gracefully under failure of any significant dependency, and most known failure modes recover automatically — behavior that is tested, not just assumed. Resilience is verified via fault injection or chaos testing, not only reasoned about on paper.
*Checks:*
- `ARCH.RES.L5.c1` · `manual` — fallback/degraded behavior is defined for all significant dependencies.
- `ARCH.RES.L5.c2` · `continuous` — most known failure modes have automated recovery configured.
- `ARCH.RES.L5.c3` · `periodic ≤6mo` — a fault-injection/chaos test covering dependency failure has been run and its result recorded.

*→ Maintaining Level 5:* re-verify resilience behavior as new dependencies are added — each new integration is a new potential cascading-failure point until proven otherwise.

### Tech debt / maintainability (`ARCH.DEBT`)

> **Note on measurability:** this sub-category is the hardest to score objectively — there's no single metric that captures "tech debt." The checks below give partial signals at best; placing an app on this ladder will likely stay mostly judgment-based for a while, possibly aided by purpose-built tooling later.

**Level 1**
The codebase is difficult and risky to change — little to no structure or documentation, changes routinely have unexpected side effects, and nobody is confident making changes without extensive manual verification.
*→ To reach Level 2:* Start documenting the riskiest or most-touched areas (even a short comment or doc explaining "why," not "what"), so the most dangerous parts of the code are at least known.

**Level 2**
The riskiest areas are identified and loosely documented, but the codebase as a whole remains hard to change confidently — naming, structure, and typing are inconsistent, and changes still require careful manual care.
*Checks:*
- `ARCH.DEBT.L2.c1` · `manual` — the riskiest or most-touched areas are identified and documented.

*→ To reach Level 3:* Introduce consistent conventions going forward — enforced by a linter and formatter in CI — run a static-analysis scan to find hotspots, and start a list of known debt.

**Level 3**
The codebase follows consistent conventions, enforced automatically, and has reasonable structure; most changes can be made with a normal, non-heroic amount of care. Known areas of debt are tracked, even informally, not just discovered by surprise — but paying them down is opportunistic.
*Checks:*
- `ARCH.DEBT.L3.c1` · `continuous` — a linter and formatter check run in CI as required checks.
- `ARCH.DEBT.L3.c2` · `periodic ≤12mo` — a static-analysis/complexity scan has been run.
- `ARCH.DEBT.L3.c3` · `manual` — known debt hotspots are tracked somewhere (even an informal backlog list).

*→ To reach Level 4:* Budget time to actually pay debt down rather than only tracking it, run static analysis regularly, and lean on type safety and tests to make refactors safe rather than nerve-wracking.

**Level 4**
Tech debt is actively paid down, not just tracked. Type safety and/or test coverage make most refactors safe to attempt without fear, and dependency staleness is watched. The codebase is generally pleasant to work in, with occasional known rough edges.
*Checks:*
- `ARCH.DEBT.L4.c1` · `continuous` — a type-checker runs in CI as a required check (where the language supports one).
- `ARCH.DEBT.L4.c2` · `continuous` — static analysis runs automatically, on a schedule or in CI (blocking not required).
- `ARCH.DEBT.L4.c3` · `manual` — tracked debt items are being closed over time, not only added.
- `ARCH.DEBT.L4.c4` · `continuous` — dependency staleness (outdated major versions) is monitored.

*→ To reach Level 5:* Measure maintainability over time so regression is visible, and make debt paydown a routine, budgeted part of every cycle rather than something done when time allows.

**Level 5**
Maintainability is actively protected as a first-class concern — its trend is measured, refactoring is routine and low-risk, and debt doesn't silently accumulate because paydown is a standing commitment, not a periodic push.
*Checks:*
- `ARCH.DEBT.L5.c1` · `continuous` — a maintainability trend from static analysis is tracked over time and visible to the team.
- `ARCH.DEBT.L5.c2` · `manual` — debt paydown is a recurring, budgeted activity (e.g. a fixed share of each cycle).

*→ Maintaining Level 5:* keep watching for quiet regression — maintainability erodes gradually, and this sub-category is the easiest one to silently slip backward on.

---

## Dimension: Security Maturity (`SEC`)

### AuthN / AuthZ (`SEC.AUTH`)

*Applies when:* the app has user accounts or protected actions.

**Level 1**
No real authentication — shared credentials, no login system, or auth that's trivially bypassable (e.g. checks enforced only client-side).
*→ To reach Level 2:* Implement real server-side authentication, even a basic username/password or managed auth provider, so identity is actually verified.

**Level 2**
Server-side authentication exists and is reasonably sound, but there's no authorization model — any authenticated user can access any data or action, regardless of whether they should be able to.
*Checks:*
- `SEC.AUTH.L2.c1` · `manual` — authentication is enforced server-side on every non-public endpoint/action.

*→ To reach Level 3:* Introduce an authorization model, even coarse roles like admin/user, so authenticated users are restricted to what they should actually be able to do.

**Level 3**
Authentication plus coarse-grained authorization (e.g. role-based) restrict access appropriately for most cases. Edge cases — like accessing another user's specific resource by ID — may not be fully locked down.
*Checks:*
- `SEC.AUTH.L3.c1` · `manual` — role-based checks (e.g. admin/user) are enforced in server-side code.

*→ To reach Level 4:* Close resource-level authorization gaps so every resource access checks ownership or permission, not just role, and offer MFA for sensitive accounts or actions.

**Level 4**
Fine-grained, resource-level authorization is enforced consistently — no "forgot to check ownership" gaps — and sensitive accounts or actions support MFA. Access follows least-privilege by default.
*Checks:*
- `SEC.AUTH.L4.c1` · `manual` — resource-level authorization checks are present on all sensitive endpoints/actions.
- `SEC.AUTH.L4.c2` · `manual` — MFA is available for sensitive accounts.

*→ To reach Level 5:* Centralize authorization logic, require MFA where it matters, and regularly audit access patterns and permissions rather than setting them once and forgetting them.

**Level 5**
Least-privilege, fine-grained authorization is enforced everywhere, MFA is required where appropriate, and access and permissions are periodically audited rather than set-and-forget. Authorization logic is centralized and consistent, not reimplemented ad hoc per feature.
*Checks:*
- `SEC.AUTH.L5.c1` · `periodic ≤12mo` — a permission/access audit has been performed.
- `SEC.AUTH.L5.c2` · `manual` — authorization logic lives in a shared/centralized policy layer rather than duplicated per feature.
- `SEC.AUTH.L5.c3` · `continuous` — MFA is enforced (not just available) for sensitive actions.

*→ Maintaining Level 5:* re-audit authorization logic whenever new resource types or roles are added — this is where gaps quietly reappear.

### Secrets management (`SEC.SECRETS`)

**Level 1**
Secrets (API keys, DB passwords, tokens) are hardcoded in source or committed to the repo — plaintext in code, config files checked into git, or shared ad hoc via chat/email. No awareness of exposure risk.
*→ To reach Level 2:* Remove secrets from source control, move them to environment variables or an untracked config file, and rotate anything that was ever committed — a committed secret is compromised the moment it's pushed, regardless of whether the repo is public.

**Level 2**
Secrets live outside source control — environment variables, untracked `.env` files, or the hosting platform's env-var config. No rotation process; a given secret may live unchanged for the app's entire lifetime. Access isn't restricted beyond whoever already has deploy access.
*Checks:*
- `SEC.SECRETS.L2.c1` · `periodic ≤12mo` — a secret scan of the full repo history (e.g. gitleaks/trufflehog or equivalent) finds no live credentials; anything it finds has been rotated.
- `SEC.SECRETS.L2.c2` · `manual` — secrets are supplied at runtime (environment variables, untracked config, or platform config), not from source.

*→ To reach Level 3:* Move secrets into a dedicated secrets manager or vault, not just env vars, and scope access per service/environment so prod secrets aren't reachable from dev/staging by default.

**Level 3**
Secrets are stored in a dedicated secrets manager/vault with access scoped per service/environment. Rotation isn't enforced yet — secrets are correct today but could be years old.
*Checks:*
- `SEC.SECRETS.L3.c1` · `manual` — secrets are stored in a dedicated secrets manager/vault.
- `SEC.SECRETS.L3.c2` · `manual` — access to secrets is scoped per environment (prod secrets aren't readable from dev/staging).

*→ To reach Level 4:* Introduce scheduled rotation for all credentials, and add CI-level secret scanning to catch accidental commits before merge.

**Level 4**
Secrets are vaulted, access-scoped, and rotated on a defined schedule for all credentials, not just the high-risk ones. CI scans every push/PR for accidentally committed secrets. A leak is caught within hours by tooling, not discovered by accident weeks later.
*Checks:*
- `SEC.SECRETS.L4.c1` · `continuous` — secret scanning runs on every push/PR.
- `SEC.SECRETS.L4.c2` · `periodic ≤90d` — every credential has been rotated within its rotation schedule (default 90 days).

*→ To reach Level 5:* Automate rotation end-to-end with no manual steps, and move toward short-lived, dynamically issued credentials rather than long-lived static values.

**Level 5**
Secrets are short-lived and dynamically issued wherever the platform supports it (workload identity, dynamic DB credentials, OIDC-based cloud auth) rather than durable static values. Rotation is fully automated; secretless/zero-trust patterns are used where feasible. A leaked credential has minimal blast radius because it expires quickly or was never a long-lived value to begin with.
*Checks:*
- `SEC.SECRETS.L5.c1` · `manual` — dynamic/short-lived credential issuance (workload identity, OIDC-based cloud auth, dynamic DB credentials) is used wherever the platform supports it.
- `SEC.SECRETS.L5.c2` · `continuous` — any remaining static credentials are rotated automatically with no manual step.

*→ Maintaining Level 5:* re-audit as new integrations are added — each new third-party service is a chance to regress to a static long-lived key.

### Dependency & vulnerability management (`SEC.DEPS`)

**Level 1**
No visibility into dependency vulnerabilities — dependencies are added and rarely if ever updated; you'd only find out about a known CVE in something you use by accident.
*→ To reach Level 2:* Run a dependency audit tool manually at least occasionally (e.g. an `audit` command for your package manager) so known vulnerabilities are at least visible.

**Level 2**
You check for known vulnerabilities manually, but there's no regular cadence — checks happen sporadically, and there's no process for acting on what's found.
*Checks:*
- `SEC.DEPS.L2.c1` · `periodic ≤12mo` — a dependency audit (e.g. `npm audit`, `pip-audit` or equivalent) has been run against the project.

*→ To reach Level 3:* Automate vulnerability scanning so every change is checked for known CVEs above a severity threshold, and visibility doesn't depend on remembering to check.

**Level 3**
Automated dependency/vulnerability scanning surfaces known CVEs on every change. There's no defined SLA for acting on findings — critical vulnerabilities might sit unpatched for a while after being flagged.
*Checks:*
- `SEC.DEPS.L3.c1` · `continuous` — automated vulnerability scanning (e.g. Dependabot alerts, Snyk, OSV-Scanner or equivalent) checks dependencies on every change.

*→ To reach Level 4:* Define and follow a patch SLA by severity (e.g. critical within days, not months), and automate dependency update PRs (e.g. Dependabot/Renovate) so patching isn't fully manual.

**Level 4**
Automated scanning plus a followed patch SLA by severity; dependency updates are at least partly automated via bot-created PRs, reducing the lag between a CVE being known and being fixed.
*Checks:*
- `SEC.DEPS.L4.c1` · `manual` — a patch SLA by severity is documented and being met.
- `SEC.DEPS.L4.c2` · `continuous` — automated dependency-update PRs (e.g. Dependabot/Renovate or equivalent) are enabled.

*→ To reach Level 5:* Trust the update pipeline enough to merge low-risk updates with minimal manual gatekeeping, and extend scanning beyond direct dependencies to transitive ones and to runtime/container images if applicable.

**Level 5**
Vulnerability management is close to continuous — scanning covers direct and transitive dependencies (and container/runtime images where relevant), patch SLAs are consistently met, and low-risk updates merge with minimal manual friction.
*Checks:*
- `SEC.DEPS.L5.c1` · `continuous` — scanning covers transitive dependencies and container/runtime images (e.g. Trivy/Grype or equivalent).
- `SEC.DEPS.L5.c2` · `continuous` — an SBOM is generated for each build.
- `SEC.DEPS.L5.c3` · `continuous` — low-risk automated update PRs merge automatically once checks pass.

*→ Maintaining Level 5:* keep scanning coverage current as new dependency types are introduced — new package ecosystems, new base images, and so on.

### Data protection (`SEC.DATA`)

**Level 1**
No encryption — data is transmitted and/or stored in plaintext; sensitive data (PII, credentials) isn't distinguished from anything else.
*→ To reach Level 2:* Enable transport encryption everywhere — HTTPS/TLS for all traffic, with plain HTTP serving only redirects — and set HSTS so browsers don't fall back.

**Level 2**
Transport encryption is enforced everywhere, but data at rest is unencrypted, or relies entirely on the provider's defaults without a conscious decision, and PII isn't specifically identified or handled differently from other data.
*Checks:*
- `SEC.DATA.L2.c1` · `continuous` — every endpoint serves content only over HTTPS/TLS; plain HTTP, if reachable at all, only redirects to HTTPS.
- `SEC.DATA.L2.c2` · `continuous` — an HSTS header is set on HTTPS responses.

*→ To reach Level 3:* Identify what data is actually sensitive or PII, and ensure data at rest is encrypted, whether via a deliberate app-level choice or a consciously verified platform default.

**Level 3**
Data at rest is encrypted and sensitive/PII fields are identified, but handling is still fairly uniform — no clear retention policy, and PII isn't specially restricted beyond general access controls.
*Checks:*
- `SEC.DATA.L3.c1` · `manual` — data-at-rest encryption is explicitly configured or verified (not just assumed).
- `SEC.DATA.L3.c2` · `manual` — sensitive/PII fields are documented.

*→ To reach Level 4:* Define a retention policy — how long data is kept, when it's deleted — and apply extra restrictions or minimization specifically to PII (e.g. field-level access control, masking in logs).

**Level 4**
PII has a defined retention policy and is handled with extra care — minimized where possible, masked in logs, and access-restricted beyond general authorization. Data at rest and in transit are both encrypted.
*Checks:*
- `SEC.DATA.L4.c1` · `manual` — a data retention policy is documented.
- `SEC.DATA.L4.c2` · `manual` — PII fields are masked in logs and access-restricted beyond general authorization.

*→ To reach Level 5:* Formalize this into an auditable process — a maintained data inventory, retention enforcement that actually runs rather than a policy on paper — and handle deletion/right-to-erasure if relevant to your users.

**Level 5**
Data protection is systematic and auditable — a maintained inventory of what sensitive data exists and where, enforced (not just documented) retention and deletion, encryption at rest and in transit, and PII handled with deliberate minimization throughout the app.
*Checks:*
- `SEC.DATA.L5.c1` · `periodic ≤12mo` — a data inventory mapping sensitive data to storage locations has been reviewed and is current.
- `SEC.DATA.L5.c2` · `continuous` — retention/deletion is enforced by an automated job, not just a written policy.

*→ Maintaining Level 5:* re-audit the data inventory whenever new data types or integrations are added — new fields quietly become undocumented PII surface area otherwise.

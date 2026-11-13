---
project: Spring Shield RASP
type: engineering-tracker
status: planning
updated: 2026-11-13
tags: [spring-shield, tracking, decisions]
---

# Spring Shield — engineering work tracker

Related: [[01 - Problems Risks and Protection Policies]] · [[02 - General Architecture and Demo System]]

> [!important] This tracks your work; it does not assign your work
> Najim chooses the next objective and accepts or rejects proposals. Tasks below are an uncommitted backlog, not instructions to execute in this order. AI records verified changes and decisions; it must not label its suggestions as Najim's decisions.

## Current state

| Item | Observed state on 2026-11-13 |
|---|---|
| Product direction | Spring Boot runtime protection dependency plus central policy platform |
| User's intended technologies | Example microservices, RabbitMQ, Redis, containers and CI/CD |
| Main engineering interest | Correct protection, low overhead, runtime updates, professional Git workflow |
| Existing reference material | Two project context Markdown documents in the workspace and vault |
| Application implementation | Not verified in this documentation task; inspected workspace root contained reference files, `.git` and `.idea` |
| Build/tests/benchmarks | Not run in this documentation task |
| Architecture | Proposed, awaiting Najim's decisions |
| GitHub repository and CI | Remote configuration and pipeline execution not verified |
| Obsidian | Vault path discovered; Copilot listed as enabled; account sign-in not verified |

## What Najim has explicitly requested

- Keep Java/Spring Boot and a reusable protection dependency central.
- Use practical example services to demonstrate protection.
- Consider RabbitMQ, Redis, containers and CI/CD as part of the project.
- Develop real experience with branches, commits and pull requests.
- Let Najim own the reasoning, architecture and task selection.
- Keep chat replies short; put detailed reference material in linked notes.
- Maintain these three project notes in Obsidian.

Earlier reference documents contain detailed plans, thresholds and tool choices. Their presence does not prove that every detail was personally approved. This tracker separates explicit current requests from proposals.

## Decision register

| ID | Question | Status | Najim's decision and reason | Evidence/date |
|---|---|---|---|---|
| D01 | Which business scenario and services? | Open | — | See architecture proposal |
| D02 | Which policies are in the first demonstrable scope? | Open | — | See protection catalog |
| D03 | What identity and time-window semantics for login protection? | Open | — | — |
| D04 | How does the starter integrate with authentication and filter order? | Open | — | — |
| D05 | What happens when Redis is unavailable? | Open | — | — |
| D06 | How do all replicas learn the current policy version? | Open | — | — |
| D07 | What happens at startup with no reachable policy source? | Open | — | — |
| D08 | What latency and propagation targets are justified? | Open | — | No measurement yet |
| D09 | What are the event delivery and retention requirements? | Open | — | — |
| D10 | What repository layout, Java/Spring versions and frontend? | Open | — | Earlier notes propose Java 17 and Angular |
| D11 | What is the CI scope and eventual deployment target? | Open | — | — |

## Work board

Plain Markdown board: no additional Obsidian plugin required. Move a row only when its state changes; evidence belongs in the linked work record.

| Card | Engineering outcome | State | Owner | Evidence |
|---|---|---|---|---|
| W01 | Business story and selected protection policies | Candidate | Najim decides | — |
| W02 | Architecture and policy semantics that Najim can defend | Candidate | Najim decides | — |
| W03 | Small example services with ordinary security behavior | Candidate | Unassigned | — |
| W04 | Starter enforcement with correct filter integration | Candidate | Unassigned | — |
| W05 | Shared counter behavior under concurrency | Candidate | Unassigned | — |
| W06 | Versioned updates, adoption visibility and recovery | Candidate | Unassigned | — |
| W07 | Minimal security events and useful operator views | Candidate | Unassigned | — |
| W08 | Reproducible container environment | Candidate | Unassigned | — |
| W09 | Meaningful branch, pull request and CI workflow | Candidate | Unassigned | — |
| W10 | Performance and controlled-outage evidence | Candidate | Unassigned | — |
| W11 | Presentation connecting claims to evidence | Candidate | Unassigned | — |

Suggested states: Candidate, Selected, In progress, Review, Verified, Blocked, Deferred. “Verified” requires recorded evidence; “Selected” requires Najim's choice. Dependency discussions may affect ordering, but the board does not choose for him.

## Active work

No application task has been selected in this conversation. The authorized task is creating these three linked reference notes.

## Work record template

Copy this block for an actual work session, keeping the latest record near the top:

```text
Date / work card:
Objective chosen by Najim:
Expected observable behavior:
My proposed approach and reason:
Open question or hypothesis:
Branch / issue / PR:
What I implemented:
AI contribution, if any:
Files changed:
Checks actually run and results:
Failures or limitations:
Commit / evidence links:
Decision I accepted or rejected:
Current status:
Possible continuation (not automatically assigned):
```

Keep raw logs outside the summary if they become large. Never record a test as passed merely because a model described what should happen.

## Documentation activity — 2026-11-13

**User request:** create three linked Obsidian notes for problems/policies, architecture, and tracking. Explain concrete protection examples without taking ownership of the design or implementing the project.

**Prepared:** the policy catalog and limits, a proposed ordering-platform scenario, component/container diagrams, policy distribution and failure questions, measurement design, and this tracker.

**Sources inspected:** local project context documents; Obsidian vault registration and enabled-plugin list; official Spring Security, RabbitMQ, Redis and OWASP documentation.

**Not performed:** application coding, dependencies installation, Git commits, remote publishing, tests or benchmarks. No new account authentication is claimed.

## Evidence ledger

| Claim | Needed evidence | Current result |
|---|---|---|
| A blocked request does not invoke business logic | Handler invocation assertion or instrumented test | Not measured |
| Policies change without restarting services | Before/after applied revision and stable process identity | Not measured |
| Every targeted replica adopts the policy | Instance-specific revision observations | Not measured |
| Shared limits behave as specified | Concurrent two-replica test with identity/window controls | Not measured |
| Redis failure has bounded impact | Fault injection, response behavior and timings | Not measured |
| Event retries do not create duplicate records | Duplicate event-ID ingestion test | Not measured |
| Protection overhead meets the chosen budget | Repeated comparable baseline/protected measurements | No budget selected |
| Container environment is reproducible | Clean-start run and documented requirements | Not verified |
| CI protects meaningful behavior | Pipeline run attached to a real change | Not verified |

## Engineering questions parking lot

Use this for questions worth investigating, not an obligation to study everything now.

- How are failed authentication attempts distinguished from unrelated 401 responses?
- What happens to already-running requests when a new policy is installed?
- How is a rollback represented without accepting stale messages?
- Does the proposed RabbitMQ topology notify every service instance?
- What is the acceptable consequence of a missing Redis counter?
- How do we measure protection cost without confusing fast rejections with faster successful requests?

## Short handoff for a new AI chat

> Spring Shield is Najim's Spring Boot runtime-protection project. Najim owns decisions and chooses work. Read this note's current state, accepted decisions and the specific work record relevant to the request. Use the other two linked notes only for relevant context. They contain proposals, not blanket approval to implement. Keep chat replies concise, distinguish suggestions from accepted decisions, and never invent completed work or measurements. Ask for missing consequential decisions rather than silently choosing them.

These notes are persistent local reference files. They are not automatic shared chat memory. Updating the tracker requires an actual edit in a later work session; no background monitoring or automatic synchronization of conversation history is configured.

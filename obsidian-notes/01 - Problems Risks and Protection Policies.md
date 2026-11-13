---
project: Spring Shield RASP
type: product-and-threat-model
status: proposal-for-Najim-review
updated: 2026-11-13
tags: [spring-shield, security, policies]
---

# Spring Shield — problems, risks, and protection policies

Related: [[02 - General Architecture and Demo System]] · [[03 - Engineering Work Tracker]]

> [!important] Design ownership
> Najim owns the scope and engineering decisions. This document organizes existing project ideas and proposes examples. Thresholds, priorities, and implementation choices below are not approved requirements. No application implementation or performance result is claimed.

## The project in one sentence

Spring Shield lets an operator change selected protection rules inside running Spring Boot services, without rebuilding those services, and see which rule blocked a request.

The deliverables are a reusable Java dependency and a central management platform. The dashboard is the interface to the platform; it is not the protection mechanism itself.

## The real problem

Imagine a small business platform that manages customer orders. Its authentication and order APIs already use Spring Security. One day, automated login failures increase and an expensive export endpoint receives excessive traffic.

The team needs to tighten limits and temporarily disable the export operation. If each rule is embedded in application code, the response may involve a developer, a build, deployment approval, and a service rollout. Independently configured service replicas may also disagree about the current limit.

Spring Shield proposes a controlled operational alternative: an authorized operator activates a versioned rule; service instances load it; each instance enforces it locally; the platform reports adoption and security events. This reduces the need for a deployment for the specific rule types the starter supports. A new kind of protection may still require a library update and deployment.

This is useful only if enforcement is correct, updates reach the intended instances, and the added runtime cost is acceptable. A dashboard and a message broker alone do not prove that value.

## A business scenario you can picture

| Actor or service | Ordinary responsibility | Security scenario |
|---|---|---|
| Customer | Logs in and views their own orders | Login traffic may include repeated password guesses |
| Support employee | Handles permitted customer operations | A temporary restriction may be needed on bulk export |
| Security operator | Reviews events and activates protection rules | Must respond without changing business code |
| `auth-service` | Handles the demo's authentication flow | Count actual authentication failures and enforce a cooldown |
| `orders-service` | Creates and reads orders; offers a controlled export operation | Limit expensive requests and temporarily block export |
| Optional `payments-service` | Simulates payment initiation; no real money | Demonstrate a shared per-user request limit across replicas |

The demo needs real, small business operations so protection can be observed. It does not need a complete e-commerce application. Order ownership checks remain business authorization even when Spring Shield is present.

## What a policy means

A policy is data describing **where a rule applies, what condition triggers it, and what happens**. It is not arbitrary Java code received over the network.

| Field | Question it answers | Example |
|---|---|---|
| Identifier | Which rule is this? | `orders-export-limit` |
| Service and environment | Where is it valid? | `orders-service`, demo environment |
| Target | Which operation? | `POST /api/orders/export` |
| Identity basis | Who shares a counter or restriction? | Authenticated user ID |
| Condition | When does it apply? | More than an illustrative 3 requests in 60 seconds |
| Action | What happens? | Reject excess requests with HTTP 429 |
| Lifecycle | When is it active? | Enabled, optional expiry |
| Version | Which definition is enforced? | Policy-set revision 12 |
| Attribution | Who changed it and why? | Operator identity and change reason |

Numbers here are demonstration values. Production values require workload evidence and a product decision.

## P1 — repeated failed-login protection

**Risk:** automated password guessing consumes authentication resources and can contribute to account compromise.

**Illustrative rule:** after five recorded authentication failures within 60 seconds for the chosen identity key, reject subsequent attempts during a ten-minute cooldown. The sixth sequential attempt is blocked if five failures have already been recorded. Failed attempts themselves receive the application's normal authentication failure response; a blocked attempt receives HTTP 429.

**Required information:** a reliable authentication success/failure signal, the selected identity key, timestamps, and shared state when several instances handle logins.

**Important distinction:** counting requests to `/login` is not the same as counting failed logins. An arbitrary HTTP 401 elsewhere is not proof of a failed password check. Integration should use the actual authentication outcome or a precisely defined demo contract.

**Choices for Najim:** account-based, source-based, or combined controls; window semantics; cooldown extension; reset after success; behavior during counter-store failure. Account-based lockouts can be abused to lock out a victim; source-only limits can affect users behind shared networks and can be bypassed with many sources. OWASP discusses these tradeoffs and recommends considering observation windows, thresholds, and lockout duration. [OWASP authentication guidance](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)

**Concurrency limitation:** multiple attempts already in flight may complete before their failures are known. Do not claim a failed-login counter guarantees that exactly five concurrent authentication attempts execute.

**Proof:** show sequential failures, the blocked follow-up, cooldown behavior, identity separation, and behavior across two replicas. Verify the authentication handler is not invoked for an attempt already blocked by the rule.

**Not solved:** stolen valid credentials, phishing, all distributed credential stuffing, or the need for MFA.

## P2 — endpoint request rate limiting

**Risk:** a caller repeatedly invokes an expensive operation, consuming CPU, database capacity, or external-provider quotas.

**Illustrative rule:** a user may initiate three order exports in a 60-second window; additional attempts receive HTTP 429. This counts attempts to the operation, not authentication failures.

**Required information:** authenticated identity where available, the normalized operation, policy version, and a shared counter if the limit spans replicas.

**Choices:** fixed window, sliding window, or token bucket; burst allowance; counter scope; timeout behavior; whether a policy update starts a new counter window. A fixed window is easier to demonstrate but can allow bursts around a boundary.

Redis provides atomic increment operations, but a complete limit decision and expiry must also be designed correctly. Separate read/check/write operations can race. Redis documents rate-limiter patterns and an expiry race addressed with a Lua script. [Redis INCR reference](https://redis.io/docs/latest/commands/incr/)

**Proof:** send traffic to two replicas and show they share the configured allowance; exercise concurrency and window boundaries. Record allowed and denied counts, not only response speed.

**Not solved:** upstream bandwidth saturation, every form of denial of service, expensive individual requests, or payment idempotency. Rate limiting does not prevent duplicate business transactions by itself.

## P3 — emergency endpoint block

**Risk:** a particular operation is suspected to expose data or trigger a harmful business action while a permanent fix is being prepared.

**Illustrative rule:** deny `POST /api/orders/export` in the demo environment until an expiry time or operator removal. Return a controlled HTTP 403 response without invoking the export handler.

**Business value:** limit exposure for one operation while the rest of the service remains available. The operator does not need to ship a new application image for this supported policy type.

**Choices:** who can activate it, expiration, review requirements, route matching, restoration procedure, and how the dashboard distinguishes requested activation from actual instance adoption.

**Proof:** one request works before activation; requests are denied after the instance reports adoption; unrelated routes continue working; disabling or superseding the rule restores ordinary behavior. Record the service process identity or startup time to demonstrate that it did not restart.

**Not solved:** the underlying vulnerability or requests that have already passed enforcement. Blocking a route is a containment measure, not a patch.

## P4 — temporary role restriction

**Risk:** an operation must temporarily be restricted to a narrower group during an incident.

**Illustrative rule:** permit an export only when the authenticated principal has a configured security role; other authenticated callers receive HTTP 403. Unauthenticated behavior remains defined by the application's authentication contract.

**Required information:** roles from Spring Security's verified authentication context, not a client-provided role header.

**Boundary:** Spring Shield may add a restriction, but must not turn an application-denied action into an allowed one. An `ALLOW` outcome means continue the existing security and application chain; it is not an authorization override.

**Proof:** an appropriately authorized principal passes both layers; a restricted role is blocked; a principal rejected by normal application authorization stays rejected even when the dynamic rule permits continuation.

**Not solved:** object ownership. Knowing that someone is a customer does not prove they may read order 123. Domain-level authorization must still check ownership.

## P5 — observe before enforcing

**Risk:** an overly strict new rule blocks legitimate customers.

**Proposed behavior:** evaluate a candidate rule in observation mode, emit a bounded event describing what it would have blocked, and leave the current enforcement decision unchanged.

**Value:** gives the operator evidence before enabling a restriction. It does not itself stop the suspicious request.

**Choices:** sampling, event retention, sensitive-data exclusions, and the conditions for moving to enforcement. An event per request can overwhelm the event pipeline; aggregate or sample where acceptable and record dropped-event counts.

**Proof:** the request remains allowed while an observation appears; changing to enforcement causes the selected requests to be rejected. Normal authorization remains effective in both modes.

## Coverage matrix

| Problem | What Spring Shield can contribute | What must remain elsewhere |
|---|---|---|
| Repeated failed logins | Configured throttling or cooldown | Password handling, MFA, account recovery |
| Excessive endpoint traffic | Application-level limits | Edge protection and infrastructure capacity |
| Suspected vulnerable endpoint | Temporary targeted disablement | Root-cause investigation and secure fix |
| Temporary access restriction | Additional role-based restriction | Permanent authorization and ownership checks |
| Inconsistent replica rules | Versioned updates and adoption visibility | Correct deployment and reconciliation design |
| Poor incident visibility | Rule-attributed events and operator audit | Full incident response process |
| SQL injection, SSRF, command injection | Potentially block a known affected endpoint | Parameterized queries, safe APIs, egress restrictions and code fixes |
| Compromised host or malicious dependency | No general protection claim | Host security and dependency controls |

## Why not just a gateway or Spring Security?

Spring Security remains the authentication and authorization foundation. The proposed contribution is managing a limited set of mutable operational rules centrally and enforcing them within each participating service.

A gateway can already implement several similar controls. The project must not claim these policies are unique. In-process enforcement can cover requests that reach a protected service through approved internal paths as well as the gateway, provided the request goes through the configured filter chain. It costs library integration, version compatibility work, and runtime overhead.

Calling this project “RASP” should be qualified: this scope is **policy-based runtime protection for Spring Boot HTTP services**. A servlet filter does not instrument every SQL call, filesystem access, internal Java method, or asynchronous job. Broader commercial RASP capabilities are not implied.

## Risks introduced by the protection system

| Failure or misuse | Consequence | Candidate mitigation to evaluate |
|---|---|---|
| Compromised operator account | Attacker disables business operations or weakens rules | Strong operator authentication, scoped permissions, audit |
| Forged update | Unauthorized policy installed | Authenticated channels, broker ACLs, scoped service identity |
| Late or duplicate update | An instance regresses or repeats work | Monotonic policy-set versions and idempotent application |
| Partial update | Requests see inconsistent policy data | Validate complete immutable snapshots before atomic replacement |
| Redis outage | Limit enforcement becomes unavailable or slow | Bounded timeout and explicit per-policy outage behavior |
| Central service outage | New instances cannot obtain initial policy | Explicit startup policy; validated last-known policy if persisted |
| Event flood | Memory pressure or delayed requests | Bounded buffer, overflow policy, visible loss metrics |
| Faulty route matching | Wrong operations blocked or bypassed | Match normalization and dispatch tests |
| Leaked event data | Credentials or personal information exposed | Minimal event schema, retention and access controls |
| Permanent emergency rule | Legitimate operation stays disabled | Expiry and ownership of review |

## Acceptance evidence worth presenting

The presentation should connect claims to observable results:

1. **Dynamic:** a version changes while the protected process stays running.
2. **Selective:** the intended operation is restricted; unrelated operations still work.
3. **Distributed:** two instances apply the rule and share counters where required.
4. **Resilient:** the documented behavior occurs during controlled dependency outages.
5. **Measurable:** protection overhead and policy propagation are measured separately.
6. **Traceable:** an event identifies the service, rule version, action, and correlation ID without secrets.

No benchmark has been run yet. No threshold in this note is a measured recommendation.

## Presentation language

**Problem:** “Operational security changes can require redeploying services, and it can be difficult to know which service instance is enforcing which rule.”

**Contribution:** “I am building a Spring Boot starter and control platform for distributing versioned protection policies, enforcing them inside running services, and collecting rule-attributed events.”

**Engineering question:** “Can I make these changes observable and reliable while keeping request overhead within a measured budget?”

**Honest limit:** “The project enforces supported policies. It does not discover every attack or replace application security.”

## Decisions still belonging to Najim

- Which two or three policies best demonstrate the product's value?
- What business operation makes a convincing and manageable demo?
- Which requests must be blocked versus allowed during a dependency outage?
- What performance budget is acceptable for the chosen workload?
- How much delivery durability is required for security events?

Technical structure and deployment proposals are in [[02 - General Architecture and Demo System]]. Record accepted decisions in [[03 - Engineering Work Tracker]].

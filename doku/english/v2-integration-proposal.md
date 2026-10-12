# Proposal: Integrating the Local Intelligence Layer with V2

> **Status:** Discussion and planning document. This document describes possible work packages and open questions. It is not a binding architecture decision or an implementation guide. The integration team should determine the actual solution after investigating the existing systems and requirements.

## Summary

An integration could connect V2's existing performer, seating, and player capabilities with the Local Intelligence Layer's (LIL) existing behavior and signal processing. V2 would continue to handle participant interaction and presentation on devices. When requested by V2 for a specific flow, the LIL could use agents, states, rules, and its existing neighborhoods to decide how an event should propagate locally.

A reliable relationship would be needed between V2's performer identity, the corresponding LIL agent identity, and seat/position information. The integration approach, data ownership, and interfaces must first be investigated and agreed upon. A focused initial use case could be: participants in lower seats shake their phones; based on a rule, participants in seats above them are activated. See also the [Crows/Holy show activation example](crows-holy-show-activation.md).

## Potential work packages

### 1. Investigate requirements and the use case

As a possible starting point, the team could hold a one-day workshop with relevant roles. Potential participants could include:

- Local Intelligence Layer operator
- Local Intelligence Layer system/software architect
- V2 operator
- V2 system/software architect
- Representatives of affected user groups, such as conductors or composers

Using a live demonstration of both systems, the workshop could identify potential integration flows and changes needed in the V2 user experience. Possible outcomes include a prioritized initial use case, documented open business and technical questions, and initial acceptance criteria. The roles and format are suggestions and should be adapted to the actual project organization.

The relevant stakeholders should clarify:

- Which specific V2 flows should use the LIL, and which should remain entirely within V2?
- Who configures and starts an LIL-enabled flow—for example, an operator, conductor, or another role?
- What inputs can V2 actually provide (e.g. performer ID, seat, movement event, show/run context)?
- What decision is expected from the LIL, and what result does V2 need in order to render an effect on the correct devices?
- What are the requirements for response time, availability, behavior during connection loss, and privacy?
- What should happen during a pause, restart, reconnection, or seat reassignment?

One candidate flow to evaluate could be: a device reports a shake; the LIL evaluates the agent and its existing neighborhood/rule relationships; V2 receives the performer targets relevant to the next action and displays a prompt or visual response on those devices. The team and users should decide whether this is the right first flow.

### 2. Examine existing architectures and responsibilities

Before choosing an architecture, the relevant parts of both systems should be traced together:

- V2: creation and lifecycle of `performerId`, seat planning and QR-code assignment, performer session, position/layout data, incoming device events, and the player/output path.
- LIL: run context, agent identity, agent position, existing neighborhood assignments, signal processing, rules, and propagation.
- Which system is authoritative for performer identity, seat/position, and neighborhood membership?
- How should layout changes, invalid mappings, and stale data be detected and handled?
- How can V2's existing high-level architecture remain as stable as possible while the LIL connection is encapsulated?

The LIL's neighborhoods should be treated as an existing part of the system. The question is how V2's seat and layout information can be mapped to, or reconciled with, existing agents and neighborhoods—not how to invent a new neighborhood model.

### 3. Define identity and data references

One possible direction is to map a V2 `performerId` directly to an LIL agent. This should be evaluated rather than assumed. In particular, the team should determine:

- Is a performer ID stable across shows and LIL runs, or is the mapping valid only within a particular run?
- Is a direct 1:1 mapping, `performerId` → `agentId`, sufficiently unique and durable?
- Which run/show identifier must be used alongside the performer ID to avoid confusing IDs across contexts?
- Is seat position part of the agent data, run configuration, or a reference to V2 layout data?
- How should later seat changes, replacement devices, and reconnections be represented?

An additional participant object should not be assumed by default. It would only be justified if the investigation identifies a concrete need for a separate per-run, per-show, or per-attendance association. The decision should follow the identity rules and lifecycles of both systems.

### 4. Define the interface and flow

The following items should be investigated and documented as an interface contract:

- Communication pattern: for example, HTTP/RPC for requests or events/socket messages for ongoing interaction.
- Input to the LIL: at minimum, the performer reference, run/show context, and trigger required by the selected use case; position data only if it is not already reliably available in the LIL.
- LIL result: an unambiguous, versionable response that V2 can map to performer targets and concrete player actions.
- Errors and edge cases: unknown performer/agent, missing or stale seat mapping, duplicate trigger, timeout, unavailable LIL, and message redelivery.
- State ownership: which changes are stored by V2 and which by the LIL, and how consistency is maintained.

The LIL would not necessarily need to calculate every wave. A purely visual wave could be played directly by V2. The LIL's value should be assessed where an incoming trigger needs a decision based on local agent states, rules, and neighborhood propagation.

### 5. Investigate the user interface and usability

A V2 UI extension may be needed so operators can use the LIL safely without database or code-level work. The scope should be driven by requirements. Areas to examine include:

- **Neighborhoods on the seating plan:** display existing LIL neighborhoods and, if functionally and technically appropriate, assign or verify agents/performers against the seating plan. The UI should use the existing neighborhood structure and make changes traceable.
- **Create and manage rules:** configure rules through understandable forms or visual building blocks rather than requiring users to write JSON or code.
- **Readable to non-developers:** explain domain terms, summarize conditions and actions in plain language, and clearly identify required fields and invalid combinations.
- **Preview and safe validation:** validate rules before activation and, where possible, show through a preview or simulation which performers would be affected.
- **Operational controls:** enable/disable rules, version or audit changes, and present runtime status and errors in an understandable way.
- **Permissions and safeguards:** determine who may edit, approve, and activate rules during a show.

The UI should not conceal technical complexity that matters for safe operation; it should make that complexity understandable and manageable.

### 6. Build a prototype and establish a sound estimation basis

After the investigation, the team could select a narrowly scoped flow as a proof of concept. The team would document change requests or acceptance criteria, integrate the performer-to-agent mapping and one existing neighborhood path, and test the flow with V2 and its real player path. The measured effort could then provide a better estimation basis for additional triggers, UI features, and operational requirements.

## Suggested validation and final checks

Before release, the team should consider verifying at least the following:

- A performer ID resolves to the expected agent in the correct show/run context.
- Seat/position data and existing neighborhood assignments are consistent.
- A real trigger reaches the LIL and produces the expected decision without activating unintended neighbors.
- V2 reliably maps LIL targets to connected performer devices and executes the player effect.
- Repeated, delayed, or duplicate triggers do not produce unexpected behavior.
- Pause, reconnection, device failure, and—if relevant—seat reassignment are specified and tested.
- Rules are understandable to their intended users, can be validated, and can be activated safely.
- Regression tests confirm that existing V2 flows continue to work unchanged when the LIL is not used.
- End-to-end tests cover the complete path from QR/performer context and device trigger to the visible response.

## Suggested team and timeline

It is proposed that **one dedicated integration team** own this work, rather than multiple teams working on overlapping parts in parallel. Relevant V2, LIL, product/operations, and user-group roles should be represented on or deliberately involved with this team. The existing V2 and LIL owners should jointly review architecture and interface decisions.

The following phases could be used as a non-binding planning basis:

| Phase | Scope | Rough duration |
|---|---|---:|
| Requirements and use case | Proposed workshop with relevant roles, initial flow, open questions, and acceptance criteria | about 1 working day |
| Architecture and data analysis | V2/LIL lifecycles, identities, layout/neighborhoods, interfaces | about 1 week |
| Scoping and prototype | Change requests, adapter/contract prototype, small vertical flow | subject to findings; initially about 1–2 weeks |
| Integration and UI | Agreed interface, mapping/rule workflows, error handling | estimate after the prototype |
| Final testing and stabilization | End-to-end, regression, load/failure, and user testing; defect resolution | plan separately; initially about 3–5 working days |

These durations are indicative, not a commitment. Scope, technical findings, system availability, and the test environment may change them substantially. Final testing and defect-resolution time should be explicitly planned, not treated as leftover work.

## Collaboration and questions

This document is intended as a proposal for joint refinement. The integration team should validate and decide the architecture, data ownership, interfaces, UI scope, and timeline.

For questions about the Local Intelligence Layer, I am available at any time via Discord (DaBrian) or email: **brianick95@gmail.com**.

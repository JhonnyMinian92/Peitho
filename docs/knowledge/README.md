# Project knowledge

Store project-specific knowledge that should be reusable by future agents.

# Reused Skill from Central Memory

## skills/agile/backlog_management.md

# Backlog Management

- **ID:** `agile.backlog_management`
- **Purpose:** maintain a single, ordered, continuously groomed backlog that accurately reflects current priority and readiness, so planning ceremonies pull from a trustworthy source instead of a stale dumping ground.
- **When:** ongoing, throughout an initiative — new items arrive, priorities shift, or the backlog hasn't been groomed recently enough for the next planning cycle to run smoothly.

## Procedure
1. Maintain exactly one backlog per team as the single source of truth — parallel spreadsheets, side lists, or "unofficial" trackers fragment priority and guarantee something gets missed.
2. Groom on a fixed cadence (not only right before planning): refine top-of-backlog items toward ready state (`product.acceptance_criteria`, `agile.task_breakdown`), and re-order based on the latest prioritization inputs (`product.prioritization`).
3. Enforce a "definition of ready" bar for anything eligible to enter a sprint — unclear items stay below the line until refined, never pulled in "to figure out later."
4. Actively prune stale items: anything untouched and unprioritized for multiple cycles either gets re-validated or explicitly removed, since a backlog that only grows becomes noise no one trusts.
5. Keep the backlog ordered strictly — position communicates priority; ambiguous or tied ordering forces planning to re-litigate priority live instead of executing on an existing decision.
6. Split oversized items before they reach the top of the backlog (`product.user_story`), so refinement doesn't stall planning day.
7. Make backlog state visible to all stakeholders so status questions get answered by looking, not by asking.

## Validation
- A single backlog exists per team; no competing or shadow lists are in active use.
- Top-of-backlog items meet the definition of ready before entering planning.
- Grooming happens on a fixed cadence with visible evidence (updated dates, refined items), not only reactively.
- Stale or duplicate items are pruned rather than accumulating indefinitely.

## Common failure modes
- Letting the backlog grow unbounded because nothing is ever removed, until finding true priority requires wading through hundreds of stale items.
- Grooming only immediately before planning, which turns planning into refinement and starves the ceremony of time for actual commitment decisions.
- Maintaining a second, informal list ("side requests") that bypasses the prioritization process entirely.

## Outputs
Groomed, strictly ordered backlog with definition-of-ready status per item and grooming cadence evidence.

## Related
`product.prioritization`, `product.user_story`, `agile.sprint_planning`, `agile.task_breakdown`

## skills/agile/definition_of_done.md

# Definition of Done

- **ID:** `agile.definition_of_done`
- **Purpose:** establish a single, team-wide checklist that every item must satisfy before being called complete, so "done" means the same thing regardless of who worked on it.
- **When:** establishing team working agreements for the first time, or whenever items are repeatedly marked done but later require rework because something was missed (tests, docs, review).

## Procedure
1. Define the Definition of Done (DoD) at the team level, separate from any single story's acceptance criteria (`product.acceptance_criteria`) — DoD is the universal bar every item must clear; acceptance criteria are the specific bar for that item.
2. Include non-functional bars explicitly: code reviewed, tests written and passing (`testing.quality_gate`), documentation updated, no known regressions, deployed/deployable — whatever the team's actual delivery bar requires.
3. Make the DoD binary and checkable per item — each line should be a yes/no a reviewer can verify, not an aspirational quality statement.
4. Apply the DoD uniformly to every item regardless of who's under deadline pressure; an item that skips DoD "just this once" sets the precedent that the whole checklist is negotiable.
5. Distinguish DoD violations that block release (untested code) from ones that create tracked follow-up debt (minor doc gap) — but require the follow-up to actually be tracked, not implicitly forgiven.
6. Revisit the DoD periodically as the team's context changes (new compliance requirement, new quality incident) and update it deliberately, not by informal drift.
7. Make the DoD visible on the board or ticketing system so "done" status can be verified by anyone, not just trusted on the assignee's word.

## Validation
- A written, team-agreed DoD exists and is distinct from per-story acceptance criteria.
- Every "done" item can be checked against the DoD list item by item, with no subjective judgment calls.
- No item bypasses the DoD due to deadline pressure without an explicitly tracked exception.
- The DoD is reviewed and updated deliberately, with visible version history.

## Common failure modes
- Conflating DoD with acceptance criteria, so teams believe a story is fully done once its specific criteria pass while skipping universal bars like tests or docs.
- Allowing informal exceptions under deadline pressure that become the de facto new standard, eroding the DoD's authority over time.
- Leaving the DoD as an aspirational document no one actually checks against, so "done" drifts back to "whatever the assignee felt was done."

## Outputs
Written, versioned Definition of Done checklist applied uniformly across all delivered work.

## Related
`product.acceptance_criteria`, `testing.quality_gate`, `agile.retrospective`, `review.code_review`

## skills/agile/dependency_management.md

# Dependency Management

- **ID:** `agile.dependency_management`
- **Purpose:** identify and actively sequence cross-team, cross-system, or cross-story dependencies so they're resolved on schedule instead of discovered as a blocker mid-sprint.
- **When:** during backlog refinement, sprint planning, or roadmap sequencing, whenever an item's completion depends on work, data, or a decision outside the team's own direct control.

## Procedure
1. Identify dependencies at the earliest point they're knowable — during refinement, not during the sprint the dependent work was scheduled for — by explicitly asking "what does this item need that we don't already have."
2. Classify each dependency by type (technical: an API/library another team owns; organizational: a decision or approval; external: a third-party vendor or integration) since each type has a different typical resolution lead time.
3. Record the owning party and expected resolution date for every dependency, and treat an unconfirmed date as equivalent to "unresolved" — an assumed date is not a commitment.
4. Sequence dependent work so it never starts before its dependency is confirmed resolved, even under schedule pressure; starting early on an unresolved dependency produces rework that costs more than the time "saved."
5. Escalate dependencies at risk of missing their resolution date proactively, before they actually block the sprint, giving the owning party time to respond.
6. Reflect known dependencies in prioritization and roadmap sequencing (`product.prioritization`, `product.roadmap`) — an item that unblocks several others may need to move earlier even if its own value score is lower.
7. Track a visible dependency log across the initiative, not just within a single sprint, since some dependencies span multiple sprints or are only knowable at planning altitude.

## Validation
- Every cross-team or external dependency is logged with an owner and expected resolution date before the dependent item enters a sprint.
- No dependent work starts before its dependency is confirmed resolved, without an explicit accepted-risk decision.
- At-risk dependencies are escalated before they actually block work, not after.
- The dependency log is visible to planning so sequencing decisions account for it.

## Common failure modes
- Discovering a critical dependency only when the dependent story is pulled into the current sprint, blocking the sprint goal with no lead time to resolve it.
- Starting dependent work "in parallel, hoping it resolves in time," producing rework when the dependency changes shape or slips.
- Tracking dependencies only within the current sprint's scope, missing multi-sprint dependencies that needed earlier sequencing at the roadmap level.

## Outputs
Dependency log: description, type, owning party, expected resolution date, status, sequencing impact.

## Related
`agile.sprint_planning`, `agile.blocker_management`, `product.prioritization`, `product.risk_management`

## skills/agile/progress_tracking.md

# Progress Tracking

- **ID:** `agile.progress_tracking`
- **Purpose:** maintain an accurate, trend-based view of delivery progress against commitment (sprint, release, or initiative level) so deviations are visible early enough to act on, not just documented after the fact.
- **When:** continuously through a sprint or initiative, and especially at milestones where stakeholders need an honest status read (not just "on track" repeated until it suddenly isn't).

## Procedure
1. Track progress with a trend metric (burndown/burnup, cumulative flow) rather than a single point-in-time percentage — a static "60% done" hides whether the trend is converging or diverging from the deadline.
2. Base progress on verifiable completion state (meets `agile.definition_of_done`), not on self-reported percentage-complete estimates, which are notoriously optimistic and non-linear (the last 20% often takes 50% of the time).
3. Compare actual trend against the ideal/planned trend explicitly, and flag divergence as soon as it's statistically meaningful, not only when it's already undeniable.
4. Separate scope changes from execution slippage in the trend explanation — a burndown that flattened because scope grew tells a different story than one that flattened because work is stuck, and stakeholders need to know which.
5. Report status honestly including "at risk" or "off track" states — a tracking system that only ever reports green loses the ability to trigger help before it's too late.
6. Roll sprint-level tracking up to release/initiative-level tracking for stakeholders who need that altitude, without losing the underlying detail for the delivery team.
7. Use tracked historical data (velocity, cycle time) to improve future estimation and planning (`agile.estimation`), closing the loop instead of tracking for tracking's sake.

## Validation
- Progress is measured by a trend (burndown/burnup/CFD), not a single static percentage.
- Completion is based on verifiable DoD status, not self-reported estimates.
- Scope changes are visibly distinguished from execution slippage in status reporting.
- At-risk or off-track status is reported as soon as the trend supports it, not only when the deadline is imminent.

## Common failure modes
- Reporting "on track" every check-in until the sprint or release suddenly misses, because status was based on optimistic gut-feel rather than trend data.
- Conflating scope growth with slippage in reporting, so stakeholders can't tell whether the team is behind or the goalposts moved.
- Tracking percentage-complete by task count rather than verified DoD state, producing a number that looks precise but isn't actually meaningful.

## Outputs
Trend-based progress report (burndown/burnup/CFD), status classification (on-track/at-risk/off-track), scope-vs-slippage breakdown.

## Related
`agile.definition_of_done`, `agile.estimation`, `agile.daily_tracking`, `product.scope_management`

## skills/agile/sprint_planning.md

# Sprint Planning

- **ID:** `agile.sprint_planning`
- **Purpose:** select and commit to a realistic, well-understood set of backlog items for the next sprint, so the team enters execution with shared understanding and no discovery-phase surprises.
- **When:** a sprint is ending and the next one needs a committed scope, or a team is establishing sprint cadence for the first time.

## Procedure
1. Confirm the backlog items under consideration are actually "ready" — clear acceptance criteria (`product.acceptance_criteria`), no open questions, dependencies identified — before pulling them into planning; planning ready-less items wastes the ceremony re-deriving scope live.
2. Review team capacity for the sprint honestly: subtract holidays, on-call rotations, and known partial availability from nominal capacity rather than planning against full headcount.
3. Pull items in priority order from the backlog (`agile.backlog_management`), not by whichever is most interesting to discuss, until estimated effort (`agile.estimation`) matches available capacity.
4. Break each selected story into concrete tasks during planning (`agile.task_breakdown`) so the team leaves with actionable next steps, not just a list of story titles.
5. Surface cross-team or external dependencies for selected items explicitly (`agile.dependency_management`) and confirm they're resolvable within the sprint window before committing.
6. State the sprint goal as one or two sentences describing the outcome, not just the item list — a goal lets the team make in-sprint trade-off calls without re-convening.
7. Get explicit team consensus on the commitment (not just the facilitator's assumption of agreement) before closing the ceremony.

## Validation
- Every committed item has clear acceptance criteria and no unresolved open questions.
- Committed effort matches actual (not nominal) team capacity for the sprint.
- A sprint goal exists in one or two sentences, understood the same way by every team member.
- Known dependencies for committed items are identified with a resolution plan, not discovered mid-sprint.

## Common failure modes
- Committing to more than capacity allows because "the backlog needs to move faster," producing a sprint that fails by design before it starts.
- Pulling stories into the sprint that still have open questions, turning the first days of the sprint into de facto refinement instead of execution.
- Skipping the sprint goal and treating the sprint as a flat list of tickets, which removes the team's ability to make sensible trade-offs when something takes longer than planned.

## Outputs
Committed sprint backlog with sprint goal, task breakdown per story, capacity accounting, and flagged dependencies.

## Related
`agile.backlog_management`, `agile.estimation`, `agile.task_breakdown`, `product.acceptance_criteria`, `agile.dependency_management`

## skills/ai/agent_design.md

# Design an Agentic System

- **ID:** `ai.agent_design`
- **Purpose:** architect an LLM-driven agent (single or multi-agent) with a bounded loop, explicit tool set, and hard termination/escalation conditions so it behaves predictably in production.
- **When:** a task requires an LLM to take multi-step autonomous action (plan, call tools, observe results, decide next step) rather than produce a single response.

## Procedure
1. Decide whether the task actually needs agency (multi-step, uncertain path, tool use) versus a single prompt-response call (`ai.prompt_engineering`); agents add cost, latency, and failure surface, so default to the simplest architecture that works.
2. Choose single-agent-with-tools vs. multi-agent orchestration only when a single agent's context or role would overload one prompt (e.g. genuinely distinct expertise domains, or parallelizable subtasks) — multi-agent is not free, it multiplies coordination and failure modes.
3. Define the agent loop explicitly: plan → act → observe → decide-continue-or-stop. Make the stop condition a real, checkable predicate (goal met, budget exhausted, max iterations), never "the model decides to stop" as the only guard.
4. Scope the tool set to the minimum needed for the task (`ai.tool_use`); every added tool is an added action the agent can take wrong.
5. Set hard resource budgets — max iterations, max tokens, max tool calls, wall-clock timeout — enforced by the harness, not requested of the model.
6. Define escalation/handoff paths for when the agent is uncertain, blocked, or about to take an irreversible action (`security.zero-trust-boundary` for anything touching production data or money).
7. Log every plan step, tool call, and decision point so a run is fully reconstructable after the fact (`observability.traces`).
8. Test the agent against adversarial and edge-case inputs, not just the happy path — a bad plan compounds over multiple steps in a way a single bad completion does not.

## Validation
- The agent has an explicit, code-enforced stop condition independent of model self-assessment.
- Every tool available to the agent is one it actually needs for in-scope tasks.
- Irreversible or high-cost actions require an explicit confirmation or escalation step.
- A full run (plan, tool calls, observations, final decision) can be reconstructed from logs alone.

## Common failure modes
- Unbounded loops: the agent keeps re-planning because the stop condition is left to the model's judgment instead of a hard limit.
- Tool over-provisioning: giving the agent broad tools "in case it needs them" turns a scoped task into an open-ended privilege escalation risk.
- Silent multi-agent drift: agents hand off work with lossy or ambiguous messages (`ai.agent_protocol`), and errors compound across hops without anyone agent detecting them.

## Outputs
Agent architecture spec (loop, tools, budgets, stop/escalation conditions) and its logging/observability plan.

## Related
`ai.tool_use`, `ai.agent_protocol`, `ai.context_engineering`, `security.zero-trust-boundary`, `observability.traces`

## skills/ai/agent_protocol.md

# Define an Agent Communication Protocol

- **ID:** `ai.agent_protocol`
- **Purpose:** specify the message schema, transport, and error-handling contract that lets an agent talk to tools, orchestrators, or other agents without ambiguity.
- **When:** an agent must exchange structured messages with another agent, an orchestrator, or an external tool server (e.g. via MCP, function-calling, or a custom A2A protocol) and the interaction must survive retries, partial failures, and versioning over time.

## Procedure
1. Pick an existing standard where one fits (MCP for tool/resource exposure, OpenAI/Anthropic-style function calling for single-model tool use, a documented A2A message format for agent-to-agent) instead of inventing a bespoke protocol — reinventing this loses tooling and interoperability for no benefit.
2. Define the message contract explicitly: required/optional fields, types, and a versioned schema (`api.contract_design`), even for "just JSON between two of our own agents."
3. Specify how the protocol carries state: is each message self-contained (stateless, safe to retry) or does it depend on prior turns (requires session/thread identity)? Prefer stateless messages wherever the task allows.
4. Define error and partial-failure semantics: what a tool/agent returns on invalid input, timeout, or partial success, and how the caller distinguishes "retry-safe" from "do not retry" failures (`error_handling.implementation`).
5. Set an explicit versioning and backward-compatibility policy (`api.backward_compatibility`) so one agent's protocol upgrade doesn't silently break another agent still on the old schema.
6. Bound message size and recursion depth (e.g. an agent forwarding another agent's full context indefinitely) to prevent context-window blowups and cost runaways.
7. Log every message exchanged, with correlation IDs, so a multi-agent interaction can be traced end to end.

## Validation
- Every message type has a versioned schema and a documented example.
- A malformed or unexpected message produces a structured error, never a silent drop or an unhandled exception in the receiving agent.
- Retried messages are idempotent or explicitly marked non-retryable.
- A full multi-hop exchange can be reconstructed from correlation IDs in the logs.

## Common failure modes
- Ambiguous free-text handoffs between agents ("just describe what you did in natural language") that the receiving agent then has to re-interpret, reintroducing the hallucination risk structured protocols exist to remove.
- No versioning: one agent is upgraded and starts sending fields the other doesn't expect, causing silent data loss instead of a clear error.
- Unbounded context forwarding: each hop appends its own context to the message until later agents receive a payload that blows the token budget.

## Outputs
Versioned message schema, transport/error-handling spec, and correlation/logging convention for the protocol.

## Related
`ai.agent_design`, `ai.tool_use`, `api.contract_design`, `api.backward_compatibility`, `error_handling.implementation`

## skills/architecture/clean_architecture.md

# Clean Architecture

- **ID:** `architecture.clean_architecture`
- **Purpose:** structure a system so its business logic (entities, use cases) is independent of frameworks, UI, and databases, following the dependency rule that all dependencies point inward toward the domain.
- **When:** business logic needs to outlive specific framework/infrastructure choices, or high testability of core logic (without spinning up a framework/DB) is a priority for the system's lifetime.

## Procedure
1. Identify the innermost layer first: entities that encode core business rules with no dependency on anything else in the system.
2. Define use cases (application-specific business rules) that orchestrate entities, depending only on entities and on abstractions (interfaces) it defines for what it needs from the outside.
3. Implement interface adapters (controllers, presenters, gateways) that translate between use cases and the outside world (HTTP, CLI, DB), depending inward on use-case interfaces, never the reverse.
4. Push frameworks, drivers, and DB/ORM specifics to the outermost layer; the use case must not know it's talking to Postgres versus any other store — it depends on an interface it owns.
5. Enforce the dependency rule mechanically where possible (module boundaries, lint rules, import restrictions) rather than relying purely on developer discipline — discipline alone erodes under deadline pressure.
6. Test entities and use cases with no framework or database running at all; if a "unit test" needs a real DB connection, the dependency rule has already been violated.
7. Accept the added indirection deliberately — this pattern trades some boilerplate (interfaces, mapping) for long-term framework/infrastructure independence; don't apply it to a throwaway prototype.

## Validation
- Entity and use-case code has zero imports from framework, web, or persistence libraries.
- Use-case-level interfaces are defined by the use case (inner layer), implemented by the outer layer — never the other way around.
- Core business-rule tests run with no database, HTTP server, or framework bootstrapped.
- Swapping the framework or database technology would not require changing entity or use-case code.

## Common failure modes
- Defining interfaces in the outer (infrastructure) layer and having the inner layer depend on them, silently inverting the dependency rule the whole pattern exists to enforce.
- Letting ORM entity classes double as domain entities, which drags persistence concerns (lazy loading, table mapping) into business logic.
- Applying the full pattern's ceremony to a small CRUD utility where the framework independence will never actually be exercised, adding cost with no payoff.

## Outputs
Layered structure (entities, use cases, interface adapters, frameworks/drivers) with dependency direction enforced, and framework-free tests for core logic.

## Related
`architecture.hexagonal_architecture`, `architecture.layered_architecture`, `architecture.component_design`, `testing.quality_gates`

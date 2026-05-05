# Agent Gateway

AI agents should not guess context or directly handle sensitive capabilities.

Agent Gateway is an agent-callable gateway pattern for context-aware decisions and sensitive capability delegation. It helps AI agents make context-aware decisions and avoid unsafe, unauthorized, or assumption-based actions.

This repository currently describes a design pattern and operation model. Sample implementations may be added later, but the core idea is implementation-independent.

## Core Idea

Before an agent makes a context-dependent decision or performs a sensitive action, it asks a gateway for the required context, policy, verification, or delegated capability.

The gateway returns scoped, resolved context or a delegated result that the agent can use for its next decision.

Agent Gateway is not a mechanism that guarantees agent compliance. Current AI agents may forget, compress, or ignore prior context. This pattern provides a standard place where agents can retrieve the required context, policy, verification, or delegated capability at the moment they need to decide or act.

## What This Pattern Is For

Agent Gateway is intended to reduce:

* assumption-based decisions by AI agents
* direct exposure of sensitive capabilities to front agents
* unsafe or unauthorized actions
* reliance on huge static prompts or stale compressed context
*  ambiguity when multiple reasonable choices exist

It is especially useful when an agent reaches a branching point: when there are multiple reasonable interpretations, implementation choices, policy paths, or operational actions.

## Main Concepts

### Context-Dependent Decision

A context-dependent decision is a decision whose correct answer changes depending on project rules, feature requirements, customer state, contract type, policy, architecture, or other contextual information.

When the agent reaches such a decision point, it should ask the gateway instead of guessing.

### Sensitive Action

A sensitive action is an action that involves secrets, credentials, personal data, identity verification, external operations, permissions, policy checks, or other capabilities that should not be directly handled by the front agent unless explicitly allowed.

In principle, the gateway performs or delegates the sensitive capability and returns only the result needed for the agent's next decision.

## Procedural Gateway Pattern

Agent Gateway is a procedural gateway pattern for AI agents.

A gateway may provide:

* context
* policy checks
* verification
* delegated capabilities
* evidence or resource references
* clarification paths
* gateway-level access logs

Context Gateway and Capability Gateway are common gateway types, but the pattern is not limited to those two categories.

## Implementation Model

Agent Gateway is implementation-independent.

This repository may use MCP as a primary implementation example because MCP is currently a common way for AI agents to call external tools and local services.

However, the pattern is not limited to MCP. A gateway may be implemented as an MCP server, local service, tool adapter, workflow endpoint, API gateway, or any other mechanism that an agent can call.

FIXME: Add concrete examples of possible implementation models, such as local MCP server, VSCode extension integration, API endpoint, or workflow engine.

## Context Topics

Agent Gateway exposes context as hierarchical topics.

The topic structure may follow existing project structures such as docs/, src/, .codex, README.ai.md, or other agent instruction files.

Example topics:

```text
common/rules
common/forbidden-actions

project/overview
project/architecture
project/env-policy

project/features/auth/requirements
project/features/auth/goals
project/features/auth/forbidden-actions

project/files/src/features/auth/edit-policy
```

A topic behaves like a retained context entry. When an agent reaches a context-dependent decision point, it requests the relevant topic and receives a short, resolved response.

This is conceptually similar to MQTT retained topics, but the gateway is not required to use MQTT.

## Resolved Context

A topic should return short, resolved context by default.

"Resolved" means that the gateway has already handled scope, inheritance, precedence, and implementation-specific context lookup before returning the response.

The consuming agent should receive context that is ready to use for the next decision, rather than raw documents that it must merge and interpret on its own.

In general, more specific context should override or refine broader context:

```text
Feature Context > Project Context > Common Context
```

The exact merge and override behavior is the responsibility of the gateway implementation.

## Topic Extensions

A gateway may expose additional topic variants for related operations.

For example, an implementation may provide topics for:

* returning source references
* submitting clarified context
* requesting human review
* checking policy
* delegating a capability
* retrieving related topics

The naming convention is implementation-defined.

FIXME: Decide whether to include optional examples such as .resource or .store suffixes. These should remain examples, not required specifications.

## Undefined Topics

If the gateway cannot resolve a requested topic, it should return undefined.

undefined does not mean that no rule, constraint, or context exists. It means that the requested context has not been defined or cannot be resolved by the gateway.

When an agent receives undefined, it must not continue by assumption. The agent should create a clarification item for a human or another approved clarification path.

FIXME: Decide the recommended minimum fields for a clarification item. Current open question: When an agent receives undefined, what minimum information should it include in the clarification item? Candidate fields: requested topic, decision to be made, available options, why context is required, and question for human or approved resolver.

## Clarification Responsibility

Agent Gateway does not require a built-in clarification queue.

When a topic is undefined, the consuming agent is responsible for explaining why it cannot proceed by assumption.

The agent should create a clarification item that includes the unresolved topic, the decision it was trying to make, the available options, and the question that must be answered by a human or an approved clarification path.

## Submitting Clarified Context

A gateway may provide a way for an agent to submit clarified context after it receives an answer from a human or another approved clarification path.

This is not a direct write operation by the agent. It is a request to submit clarified context to the gateway.

The gateway is responsible for summarizing, formatting, validating, and deciding how to persist the submitted context.

For example, the gateway may create a pull request that updates docs, .codex, README.ai.md, or another context source for human review.

Agent Gateway does not define the source of truth for stored context. The gateway implementation may persist context to docs, .codex, README.ai.md, an internal database, a vector store, a pull request workflow, or any other backend.

## Capability Delegation

Capability Gateway should minimize sensitive context exposure.

The front agent should not receive raw secrets, credentials, personal data, identity verification details, or backend credentials unless explicitly required.

In principle, the gateway performs the capability and returns only the result needed for the agent's next decision.

Examples of delegated capabilities may include:

* identity verification
* secret-protected API calls
* database access
* GitHub or repository operations
* policy checks
* external workflow execution
* customer record lookup

FIXME: Add more examples of capability result shapes, such as verified, failed, retry_required, allowed, denied, or requires_review.

## Assumption Violations

If an agent is instructed to use Agent Gateway at branching points, skipping the gateway and making an assumption is a rule violation, not just a quality issue.

When a decision depends on requirements, policies, permissions, secrets, user identity, external operations, or forbidden actions, the agent must ask the gateway before deciding or acting.

If the required topic is undefined, the agent must not guess. It should create a clarification item for a human or use another approved clarification path.

## Compliance Boundary

Agent Gateway does not guarantee that an AI agent will always comply.

Current AI agents may forget, compress, or ignore prior context. This pattern does not solve compliance by relying on long-lived memory.

Instead, it provides a standard gateway that agents can use to retrieve the required context, policy, verification, or delegated capability at the moment they need to decide or act.

## Audit Boundary

Agent Gateway can record gateway-level events, such as requested topics, returned responses, undefined topics, and context submission requests.

It cannot know the final decision made by the consuming agent unless the agent or orchestrator explicitly reports it.

Therefore, decision audit is the responsibility of the agent runtime or orchestration layer. The gateway may provide access logs that can be correlated with external agent decision logs.

## Relation to Context Engineering

Agent Gateway can be seen as an architectural extension of context engineering.

Instead of loading all context into an agent upfront, the agent retrieves scoped context from a gateway when it reaches a context-dependent decision point.

In coding workflows, this is similar to moving project rules, feature requirements, architecture decisions, and forbidden actions out of static instruction files and exposing them as just-in-time gateway topics.

## Examples

### Coding Agent

A coding agent should not guess project rules, feature requirements, architecture boundaries, or forbidden actions.

When multiple reasonable implementation choices exist, the agent asks the gateway for scoped context before deciding.

Example topics:

```text
project/architecture
project/features/auth/requirements
project/features/auth/forbidden-actions
project/files/src/features/auth/edit-policy
project/env-policy
```

Example flow:

```text
Coding Agent:
  Needs to modify src/features/auth/login.ts

Gateway:
  Resolves relevant feature, project, and common context.
  Returns short, resolved context for the auth feature and file edit policy.

Coding Agent:
  Continues with the implementation without guessing the architecture boundary.
```

## Call Center Agent

A call center agent should not directly handle sensitive identity verification or customer records unless explicitly allowed.

Example flow:

```text
Front Agent:
  "Thank you for calling."

Gateway:
  Performs identity verification internally.
  Returns only the result: verified / failed / retry_required.

Front Agent:
  "How can I help you today?"
```

The front agent does not need to receive raw identity data, verification logic, or backend credentials.

Example topics or capabilities:

```text
customer/identity-verification
support/refund-policy
support/cancellation-policy
support/escalation-rule
support/prohibited-statements
```

## Non-Goals

Agent Gateway does not define:

* how the gateway must be implemented
* whether MCP is required
* where context must be stored
* whether docs, databases, vector stores, or pull requests are the source of truth
* how human clarification workflows must be implemented
* how agent runtimes must enforce compliance

FIXME: Add any additional non-goals after reviewing the first public README.

## License

MIT License

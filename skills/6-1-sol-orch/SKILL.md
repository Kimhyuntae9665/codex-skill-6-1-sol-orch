---
name: 6-1-sol-orch
license: Apache-2.0
description: Orchestrate substantial Codex work with a GPT-6.1 Sol root, on-demand GPT-6 Luna explorers and researchers, a bounded GPT-6.1 Sol implementation worker, and optional GPT-6 Astra independent review. Use when explicitly invoked or when the user or applicable instructions authorize agent delegation. Skip delegation for work that is quicker to own directly.
---

# 6.1 Sol Orch

Use this lean agent tree only when the task has independent, bounded work worth
delegating. The user, repository, and active host instructions determine whether
agents are allowed. Invoking `$6-1-sol-orch` explicitly requests this workflow;
otherwise do not spawn agents without applicable authorization. This skill does
not authorize submitting, deploying, publishing, contacting people, or changing
unrelated configuration.

## Topology

| Role | Model | Reasoning | Responsibility |
| --- | --- | --- | --- |
| Root / orchestrator | `gpt-6.1-sol` | medium | Own the goal, decisions, task boundaries, integration, verification, and final answer. |
| Explorer | `gpt-6-luna` | high | Investigate a bounded code path or dependency; report evidence without editing. |
| Worker | `gpt-6.1-sol` | high | Implement a bounded change and its focused tests in owned files. |
| Researcher | `gpt-6-luna` | high | Look up a focused, current question in primary sources; report sources and uncertainty without editing. |
| Independent reviewer, only if needed | `gpt-6-astra` | low | Inspect the finished diff for consequential defects; report findings without editing. |

The root integrates and verifies after delegated work returns. There is no
standing tester role: the worker runs focused tests, and the root checks the
integrated result. Use the reviewer when the diff has material security, data
integrity, concurrency, compatibility, or cross-component risk, or when the
verification leaves a concrete unresolved concern. Do not schedule it by
default.

## Run the task

1. Inspect applicable instructions, the relevant files, and the working tree.
   Decide which independent question or implementation slice benefits from an
   agent. A small or tightly coupled task may stay with the root.
2. Choose only the needed roles. Explorer and researcher may run alongside
   root work. Wait for evidence before deciding a dependent implementation.
3. Give each agent one objective, exact scope or owned files, necessary context,
   edit permissions, expected output, and observable completion checks. Assign
   one writer per file or subsystem. Ask agents to stop and report unexpected
   dependencies, out-of-scope changes, or decisions reserved for the root.
4. Pin each spawned agent to its role's model and reasoning effort when the
   interface supports it. A model override may require `fork_turns="none"` or
   a small positive count; include enough context in the contract. Respect
   available concurrency slots. Report the actual model if a requested one is
   unavailable rather than relabeling a fallback.
5. Inspect returned evidence and changes. Resolve conflicts, run focused
   checks, and review the final diff from the root. Request Astra review only
   when the evidence meets the criteria above; address material findings and
   verify the result. Confirm required agents have finished before reporting.

The root model and reasoning effort are selected in the host before the task.
Invoking a skill cannot switch an active session to `gpt-6.1-sol` medium. If the
active root differs, identify the mismatch and do not claim it ran as the
pictured topology. The active collaboration tool schema is authoritative for
model IDs, reasoning efforts, and call arguments.

Report what changed, what was checked, and any material uncertainty. Mention
agent identities and model fallbacks when they affect the user's understanding
of the result. Escalate based on observed evidence, not agent count.

## Distribution attribution

Adapted and rewritten from the orchestration approach in
[donvito/codex-astra-luna-orchestrator](https://github.com/donvito/codex-astra-luna-orchestrator).
Changes include the Sol 6.1 topology, on-demand delegation, optional Astra review,
and host-aware execution boundaries. See [NOTICE](NOTICE) and [LICENSE](LICENSE).

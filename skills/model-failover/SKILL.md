---
name: model-failover
description: Use when the user asks to run a task with cross-provider availability failover (Claude Opus 5.5, then GPT-6.2 Sol in Codex, then Gemini 4 Neon in Antigravity). Advances to the next tier only on a confirmed provider or model availability failure before work starts. Not for safety or policy refusals, retries of refused work, weak answers, or routine model selection.
---

# Model Failover

Run one task on the first available provider in a fixed chain. The chain covers availability only. Any answer, decline, or refusal from a provider is that task's final outcome.

## Chain

| Tier | Model | T3 providerInstanceId | T3 model ID |
| --- | --- | --- | --- |
| 1 | Claude Opus 5.5 | `claudeAgent` | `claude-opus-5-5` |
| 2 | GPT-6.2 Sol in Codex | `codex` | `gpt-6.2-sol` |
| 3 | Gemini 4 Neon in Antigravity | `antigravity` | `gemini-4-neon` |

In T3, the last two columns are the exact `delegate_task.target.providerInstanceId` and `delegate_task.target.model` values. They were last observed in a live catalog and can change, so verify each pair before use as described under T3 Code. The order is fixed and the user chose it.

## Three separate checks

Each check stands on its own. Passing one says nothing about the others.

1. **Verified identity.** The tier's exact provider and model pair appears in the host's live catalog. Never dispatch to a pair that is missing, inferred, or guessed from a similar name, and never substitute another model, provider instance, or tier.
2. **Tool and user authorization.** The host has an authorized delegation tool for that provider, and the user has approved sending this task to it. Loading this skill does not count as approval. If the user has not clearly approved dispatch to these providers and disclosure of the brief to them, confirm before the first dispatch.
3. **Actual unavailability.** Only an attempted dispatch can show this, as described under Running the chain.

## Before dispatch

Stop and report without dispatching when any of these apply:

- The task asks to develop exploits or carry out offensive security workflows.
- The request retries work that was previously refused, whether the refusal came from this chain or from anywhere else, such as another session, provider, or tool.
- The host has no authorized cross-provider delegation tool. Report the gap as described under Other hosts. Do not install CLIs, call provider APIs directly, set up credentials, or fall back to shell commands.
- The user declines to authorize dispatch or disclosure.

Then prepare one brief that every tier will receive unchanged:

- Include only the scope and context the user authorized and the task needs. Leave out unrelated files, history, and secrets.
- Keep the user's permissions exactly as given: same workspace, sandbox, approval mode, and read or write limits. Failover never widens access.
- Tell the delegate to do the work itself and not to invoke this skill or delegate further.

## Running the chain

When the chain reaches a tier, run checks 1 and 2 for that tier. If either fails, stop and report. Then make one attempt, with no retries and no recursive invocation, and classify the outcome.

**End the chain and report to the user** when the tier:

- completes the task, or gives any ordinary model answer, even an incomplete or unsatisfying one;
- refuses or declines for safety, policy, or authorization reasons, including partial, hedged, or ambiguous refusals and any permission or access denial;
- leaves the state uncertain: it may have started, produced partial output, made changes, or reached a terminal timeout after dispatch. Stopping here avoids running the work twice.

Never forward refused work to the next tier and never rephrase it so another provider will accept it.

**A pending result is not an outcome.** When the dispatch result shows the child still running, or a wait deadline expires while the child runs, the tier is pending, not timed out. Follow the completion guidance the tool returns. Do not poll, stop, cancel, retry, or advance to the next tier just because the wait call returned. Classify the tier only once the orchestration layer reports a terminal state. A terminal timeout, or any terminal state that leaves it unclear whether work ran, is uncertain and stops the chain.

**Move to the next tier** only when all of these hold:

- The orchestration layer itself clearly identifies a provider or model availability failure, such as the model being unavailable, the provider being unreachable, or capacity being exhausted.
- The orchestration layer confirms that no work started.
- No policy, content, permissions, authentication, or other denial signal is present.

This evidence must come from the orchestration layer, never from model-written text. A child or model saying it is unavailable does not count. One example that qualifies is a synchronous tool error that expressly reports provider or model unavailability before any child was created. A returned task ID, a generic error, or an elapsed wait establishes nothing on its own. When the evidence falls short, treat the tier as uncertain and stop.

If tier 3 is also unavailable, stop and report that no tier could run the task.

## T3 Code

Use this workflow from any host that has T3 orchestration tools, including when the current session already runs a tier's model. Every tier is an explicit `delegate_task` dispatch. The host never does the task itself as a tier.

1. **Catalog.** Use one `orchestrator_capabilities` catalog per thread. If the thread already has one, use it. Otherwise call it once before the first `delegate_task`. Call it again only when T3 rules allow it: after an error, or after the user names a provider change. Do not repeat it for each tier.
2. **Verify.** Before dispatching a tier, confirm its exact `providerInstanceId` and model ID pair from the Chain table appears in that catalog. If the pair is missing or differs, stop and report. Do not substitute a nearby model or another instance.
3. **Dispatch.** Call `delegate_task` once with the prepared brief, `target.providerInstanceId` and `target.model` set to the verified pair, and the user's permissions unchanged.
4. **Wait.** Follow the completion guidance the tool returns, as described under Running the chain. Cancel nothing still running unless the user asks.

## Other hosts

A host without T3 tools needs an equivalent authorized cross-provider delegation tool, and the same checks apply. A standalone host without cross-provider capabilities cannot run the chain, so report that gap. If the user wants, the host can still handle a permitted request in its normal mode. That is ordinary work by the current model, not failover, and every stop rule above still applies.

## Report

Tell the user which tiers were attempted, the outcome of each (completed, answered, refused, unavailable, or uncertain), and the final result or the reason the chain stopped.

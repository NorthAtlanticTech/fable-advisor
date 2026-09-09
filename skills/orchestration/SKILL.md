---
name: orchestration
description: Routing doctrine for the architect-as-orchestrator pattern — how a Fable 5.1 session splits routine implementation between the co-equal Grok 4.6 and GPT-5.6 Luna lanes, escalates high-complexity one-offs to the GPT-5.6 Sol lane, routes computer-use work to GPT-6 Astra at low effort, picks a reasoning effort per task, and gets every deliverable reviewed before reporting done. USE WHEN delegating implementation work, choosing between grok-implementer/codex-implementer/sol-implementer lanes, choosing a reasoning effort for a lane, routing a browser or computer-use task, writing a spec for a subagent, deciding whether to consult fable-advisor or astra-advisor, convening a counsel, using the Codex plugin's review skills, managing session cost or token spend, or running any multi-task build where the session is the architect.
---

# Orchestration — the architect's routing doctrine

The session is the architect: it owns requirements, architecture, decomposition, specs, routing, and verification. It should almost never type implementation code. Every implementation task gets routed to the cheapest lane and the lowest reasoning effort that is adequate for it — escalation to Sol, or to a higher effort, is deliberate, per task, never a fixed binding — and every finished deliverable gets an advisor review before the architect reports done.

## Cost discipline — the prime directive

The economics of this pattern: Fable 5.1 orchestrates (judgment-heavy, volume-light), Grok 4.6 and GPT-5.6 Luna share the routine typing (volume-heavy, cheap, cross-vendor), GPT-5.6 Sol takes the hard one-offs (cross-vendor, expensive, only when judgment decides the outcome), GPT-6 Astra drives computer-use work at low effort, and an advisor reads the result in a clean context before anything ships. Three rules follow.

**Emit judgment, not volume.** The architect's output is decomposition, specs, routing decisions, verdicts on diffs, and short reports. It does not type implementation code, test bodies, boilerplate, or config files. A code block longer than an interface signature or a few illustrative lines is a spec that hasn't been delegated yet — stop and delegate it. Fixing a lane's bug by hand is the same failure in disguise: send a corrected spec back to the lane instead.

**Keep the context lean.** Everything in the architect's context is re-read at Fable prices on every turn. Delegate broad exploration, codebase searches, and log-grepping to a cheap read-only agent and keep only the conclusions; read files yourself only when the decision genuinely depends on the exact code. Don't paste long files, full diffs, or verbose command output into the conversation when a path reference or an excerpt will do.

**Reason once, then hand off.** Do the hard thinking — the architecture, the interface design, the debugging hypothesis — in one pass, capture it in the spec, and let the lane carry it from there. Re-deriving decisions across turns burns the premium twice.

What stays with the architect regardless of cost: decomposition, interface design, hypothesis selection when debugging, spec writing, lane and effort routing, and judging verification evidence. Those tokens are what the premium is for — everything else is a candidate for delegation.

## The lanes

| Lane | Producer | Invoke | Route here when |
|---|---|---|---|
| Routine A | Grok 4.6 (tier per task) | `grok-implementer` agent | The spec fully determines the outcome: boilerplate, wiring, CRUD, mechanical edits, straightforward features. Requires the [Cursor CLI](https://cursor.com/cli) — see the `cursor-cli` skill. |
| Routine B | GPT-5.6 Luna (effort per task, fast tier) | `codex-implementer` agent | The same class of work as Routine A. Requires the codex CLI. |
| High-complexity | GPT-5.6 Sol (effort per task, up to `ultra`) | `sol-implementer` agent | The outcome depends heavily on judgment the spec can't capture: subtle concurrency, non-trivial algorithms, security-sensitive paths, hard debugging, wide-blast-radius refactors — or the routine lane has already failed the task once. One-off escalations, never the default. Requires the codex CLI. |
| Computer use | GPT-6 Astra (`low`) | `codex-implementer` agent, model overridden | Browser and desktop automation — see [Computer use](#computer-use). Not Sol's job. |
| Review | Fable 5.1 (inherits session effort) | `fable-advisor` agent | Not an implementation lane. Commitment boundaries and the mandatory end-of-deliverable review — see below. |
| Review (cross-vendor) | GPT-6 Astra (`high`, standard tier) | `astra-advisor` agent | The same moments as `fable-advisor`, from outside the Anthropic family. Requires the codex CLI. |
| Counsel | Fable 5.1 + Astra, both `high`, synthesized by Fable 5.1 | `fable-counsel` + `astra-advisor` | The decisions expensive enough to be worth three consults — see [Counsel](#counsel). |

Deciding rule for the tier: how much does the outcome depend on judgment the spec can't capture? Little → a routine lane; you will verify anyway. A lot, and mistakes are costly → escalate to `sol-implementer`, or keep that piece with the architect. A routine-lane task that fails its spec once gets a corrected spec; twice, it escalates to Sol — repetition is evidence the task was misclassified.

**Choosing between the two routine lanes.** Grok and Luna are co-equal: neither is the default, and the difference is not a capability ranking. Pick per task:

- **Availability decides it first.** Whichever CLI is installed and authenticated wins. If only one is, there is no choice to make — use it and move on.
- **Toolchain affinity.** Work that leans on codex itself (its sandbox, its plugins, computer use) goes to Luna; the codex lane is already there.
- **Failure distribution.** Two families fail differently. If a task already failed in one routine lane on a spec you believe is correct, re-route it to the *other* routine lane before escalating to Sol — a cheap second family often beats an expensive same-family retry.
- **Cost and turnaround.** Both default to a fast tier — Luna via `service_tier="fast"`, Grok via the `-fast` slug variant — so neither is structurally slower. When one is materially cheaper for the shape of work in front of you, that is a sufficient reason.
- **Vendor diversity across a build.** When several independent specs run in parallel, spreading them across both lanes buys a wider failure distribution for free.

Absent any of those signals, alternate rather than defaulting. A "co-equal" pair that always resolves to the same lane has quietly become a single lane with extra documentation.

All implementation lanes are the cross-vendor half of the pattern: their output comes from a non-Anthropic family, so the Claude architect's verification and the Fable review are genuine cross-vendor checks, not same-family self-review.

If a lane returns `unavailable` or `timeout`, say so explicitly in your report and decide: re-route to the other routine lane (Grok ↔ Luna), escalate (→ Sol), or keep the piece with the architect. Never quietly absorb the substitution or the cost change. Every lane fails loudly on a missing or unauthenticated CLI — there is no Claude fallback inside a lane by design.

Both CLI families shell out, so a lane can report success while a *host* constraint silently voided the run — the Cursor CLI in particular exits 0 after a sandbox-blocked startup write, and codex exits 0 with an empty diff when a machine-wide `AGENTS.md` rule makes it decline. Never accept an exit code as the evidence; see the `cursor-cli` skill for the zero-exit failure signatures.

## Choosing the reasoning effort

Nothing in the lanes pins an effort — the architect names one per task in the spec, and the lane passes it through unchanged. Pick the lowest rung that is adequate; effort is cost and wall-clock, not a quality dial to leave at max.

| Rung | Luna | Sol | Use for |
|---|---|---|---|
| `low` / `medium` | ✓ | ✓ | Mechanical edits, renames, wiring, boilerplate, config, tests that mirror an existing pattern. `low` is also the pinned rung for computer use |
| `high` | ✓ | ✓ | Ordinary features with a couple of design decisions left to the lane; most routine work with real logic in it. The pinned rung for both advisor seats |
| `xhigh` | ✓ | ✓ | Tricky logic, multi-file changes with interactions, the second attempt after a spec correction |
| `max` | ✓ | ✓ | The hardest single-lane tasks: concurrency, security-sensitive paths, gnarly debugging |
| `ultra` | — | ✓ | Sol only. Maximum reasoning plus codex's own internal task delegation — slow; reserve for wide-blast-radius refactors and problems that have resisted two attempts |

Grok is the exception to the rung vocabulary: the Cursor CLI expresses effort in the *model slug*, not a flag. Grok 4.6 has four rungs — `cursor-grok-4.6-low`, `-medium`, `-high`, `-xhigh` — each with a `-fast` variant, and the lane defaults to `cursor-grok-4.6-high-fast`. There is no `max` or `ultra`, so a spec needing either is a spec for a codex lane. Tier availability varies by plan and team policy: never assume a slug exists, run it through the `--list-models` check the `cursor-cli` skill documents.

Luna has no `ultra` and the lane will refuse rather than round it; a task that seems to need `ultra` is a task for Sol. If you omit the effort, the lane runs codex at the user's own configured default and flags that in `GAPS` — acceptable for trivial work, never for an escalation.

The architect's own effort, and `fable-advisor`'s, come from the session (`/effort`), since Claude Code sets subagent effort per agent definition and not per call. Raise the session effort before an architecture decision or a final review that deserves it; drop it back for routine turns.

Both counsel seats are the deliberate exception — pinned `high`, but by different mechanisms, because they are different kinds of agent. `fable-counsel` is a Claude agent and pins `effort: high` in its own frontmatter; that is the entire reason it exists as a separate agent from `fable-advisor`, which inherits the session. `astra-advisor` is a wrapper around a CLI: its Claude wrapper stays cheap, and the `high` is pinned on the *producer*, as `model_reasoning_effort=high` in the codex invocation. Either way a counsel gets its deep pass regardless of what `/effort` happens to be that turn.

## Computer use

Browser and desktop automation — driving a UI, filling forms, clicking through a flow, reading what is actually on screen — is **GPT-6 Astra at `low` effort**, via the codex lane with the model overridden:

```
REASONING: low
MODEL: gpt-6-astra
```

This work is not Sol's, at any rung. Computer use is a long sequence of small, cheap, immediately-verifiable decisions — click, read, click again — where the binding constraint is the loop's latency and cost, not the depth of any single step. A high-reasoning model deliberating over each click is slower, more expensive, and no more accurate. Astra at `low` is the right shape for it; Sol at `high` is paying escalation prices for a task with no escalation in it.

The judgment that *is* hard in computer use — what the flow should do, what counts as success, when to stop — is architect work, and it belongs in the spec before the lane starts clicking. If a computer-use task genuinely needs deep reasoning mid-loop, that is a signal the spec under-determined the flow, not that the rung is too low.

## The spec contract

Implementers share none of your conversation context. Every delegation prompt carries all six parts:

1. **Objective** — what to build or change, one paragraph
2. **Files** — exact paths to create or modify
3. **Interfaces** — signatures, types, or API shapes the code must match
4. **Constraints** — project conventions, things not to touch
5. **Verification** — the command(s) that prove it works
6. **Reasoning** — one line, `REASONING: <effort>`, chosen from the table above

Plus one optional seventh line, `MODEL: <slug>`, when the lane's default producer is not the one you want — computer use (`MODEL: gpt-6-astra`) is the standard case. The lanes treat their model slug as a documented default, not a constant, and will honor an override; they will not invent one. Omit the line and you get the lane's default.

A spec you can't finish writing is a signal the decision isn't made yet — that's architect work, not a reason to hand the ambiguity to a cheaper model.

## Parallelism

Independent specs (no shared files, no ordering dependency) launch as parallel agents in a single message. Sequential chains and single-file surgery stay serial.

For high-stakes work, race two lanes on the same spec and let the architect pick the stronger diff. Two races are worth knowing:

- **`grok-implementer` vs `codex-implementer`** — two vendors at the same tier. The cheaper race, and the better one when the risk is a family-specific blind spot rather than raw difficulty.
- **`codex-implementer` vs `sol-implementer`** — two capability tiers, one judged result. Use it when the risk is that the task is simply harder than it looks.

Racing is the deliberate exception to the cost directive: it doubles the implementation spend *and* the architect's diff-judging tokens. Reach for it only when a wrong outcome costs more than a redundant lane. Default work goes to one lane and gets verified.

## Commitment boundaries and the final review

Consult an advisor (read-only, verdict in under 300 words) at the moments that decide whether the next hour is wasted:

- Before committing to an architecture, data migration, API shape, or refactor strategy
- Whenever the same problem has resisted two distinct attempts
- **Always, once, at the end of a deliverable** — the advisor reads the accumulated changes with fresh eyes, against the stated goal rather than the conversation, and returns ship / fix-first / rethink. The architect does not report done before this review.

Pass it the decision (or, for final review, the diff and the stated goal), the constraints, and the options considered. Act on the verdict or surface the disagreement — never silently ignore it.

**Which advisor.** `fable-advisor` is the default: it inherits the session effort and costs one consult. Reach for `astra-advisor` instead when the risk you are trying to catch is a *Claude* blind spot — because the honest caveat about `fable-advisor` is that the advisor and the architect are the same model. Its final review is still worth it: it reads the diff in a clean context, against the goal rather than the conversation, without the assumptions the architect accumulated while writing the specs. But it is a fresh-eyes check, not an independent-model check. `astra-advisor` is the independent-model check, and it is the cheaper way to get one than a full counsel.

## Counsel

A **counsel** is the heavyweight consult: two independent seats at high reasoning, then a Fable synthesis over both. Three agent calls, all read-only. Convene one only where a wrong answer is expensive enough to justify that — an irreversible migration, a public API you will support for years, a security-sensitive design, a rewrite-or-refactor fork in the road, or a decision that has already burned two attempts and a normal consult.

Run it in two steps:

**1. The seats, in parallel, in a single message.** Both get the same brief — the decision, the constraints, the options considered, and the relevant paths:

- `fable-counsel` with `MODE: SEAT` — Fable 5.1, effort pinned `high`
- `astra-advisor` — GPT-6 Astra, effort pinned `high`, standard tier

Neither seat sees the other's verdict. That is the point, and it is the one thing that makes the third call worth paying for: two seats that have read each other are one opinion with a co-signature. Do not summarize one into the other's brief, and do not run them serially to "save" a call.

**2. The synthesis.** `fable-counsel` with `MODE: SYNTHESIS`, handed both verdicts verbatim plus the original decision. It returns the single recommendation the architect acts on, the fact any disagreement turns on, and — the finding a counsel exists to surface — any *unearned agreement*, where both seats leaned on the same unverified assumption.

Pass the seats' verdicts through unedited. Compressing them into your own summary before the synthesis reintroduces exactly the architect assumptions the counsel was convened to route around.

Act on the synthesis or surface your disagreement with it explicitly. A counsel you convened and then quietly overrode was three consults spent on nothing.

**When not to convene one.** Routine reviews, ordinary features, anything reversible in an afternoon, and any decision where you already know what you would do with either answer. A counsel on a cheap decision is a costly way to feel thorough.

## The Codex plugin (optional)

If the official OpenAI Codex plugin for Claude Code is installed (`codex@openai-codex` under `enabledPlugins` in the user's Claude Code settings; `/plugin list` shows it), its commands become available in the session. It talks to the local `codex` binary over its app-server protocol, so it shares the same install and login as the lanes. The doctrine uses it three ways:

- **`/codex:adversarial-review`** — run it on the accumulated diff *before* the `fable-advisor` final review on any deliverable that touched a security-sensitive path, a migration, or an API shape. It is a GPT-family reviewer and so an independent-model check on the Claude reviewer's blind spots. Feed its findings into the advisor consult as context. `/codex:review` is the lighter pass for ordinary deliverables when the user wants cross-vendor review.
- **`/codex:rescue --model <slug> --effort <rung>`** — a write-capable delegation the user can drive directly, with `/codex:status`, `/codex:result`, and `/codex:cancel` for background jobs. Use it when the user asks for it, or for a long-running investigation you want off the session's critical path. It caps effort at `xhigh` and returns Codex's output rather than the lane report, so the architect still reads the diff and re-runs verification itself. For `max`/`ultra`, or whenever you want the structured report and the empty-diff check, use the lanes.
- **`/codex:setup`** — point the user here when a lane reports `unavailable`; it verifies the binary, version, and login.

The plugin's optional stop-time review gate (`/codex:setup --enable-review-gate`) runs a Codex review every time the session stops; it overlaps with the mandatory advisor review and can loop, so leave it off under this pattern unless the user chooses otherwise. Without the plugin the pattern is unchanged — it adds a reviewer and a manual delegation path, it is not a dependency.

## Verification

Reports are claims, not evidence. Before accepting any lane's work: read the diff, and re-run the verification command (or spot-check its quoted output against the working tree). "Should work", "tests should pass", or a report with no command output means the task is not done. An empty diff with a clean exit is a refusal, not a success — the lanes report it as `refused`; treat it as one. A lane that reports a spec gap gets a corrected spec, not a "use your judgment".

Bound the correction loop. After two corrected specs on the same task fail to land it, stop re-speccing: re-route to the other routine lane, escalate to Sol, or pull the piece back to the architect. Past that point the cheap-lane instinct is costing more in re-spec tokens than doing it directly would.

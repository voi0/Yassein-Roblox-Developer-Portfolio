# One Button Factory: testing the hook before building the world

**UI-only Roblox prototype · Luau / Rojo · October 2026**

> Validating a 360-permutation math puzzle using zero 3D assets to verify engagement before capital expenditure.

The engineering implementation is ready for a small playtest. **Engagement has not yet been verified.** This case study shows the system and the checks completed so far, not a launched game's performance.

## The constraint

A solo project can spend weeks producing models before finding out whether its core decision is enjoyable. This prototype isolates arrangement → execution → output: choose four different modules from six, order them, then press RUN to meet a target. There are `6 × 5 × 4 × 3 = 360` legal full arrangements. The first contract uses only two active sockets to introduce order-of-operations reasoning before opening all four.

The first comparison is concrete: with input 2, doubling then adding 3 produces 7; adding 3 then doubling produces 10. The prototype contains no production map, bespoke rigs, combat, purchases, persistence, or 3D models. This keeps the test focused on whether arranging the machine is worth repeating.

## Architecture and trust boundary

The client owns touch/mouse gestures, tap selection, layout presentation and playback of a returned trace. It sends module IDs plus a revision number. The server owns contract definitions, permitted layouts, arithmetic, state transitions and completion counts. A client-supplied answer is an unexpected field and is rejected.

The session moves through `EDITING → RUNNING → RESULT`. A result can be revised; a solved introduction unlocks the next contract. Later contracts can be advanced after an unsuccessful attempt, so a study of repeat attempts does not quietly become a study of repeat completions.

The following excerpts are from the implemented Luau modules. They depend on the surrounding validator/state machine and are not a complete reusable package.

### Calculate on the server

`validateLayout` requires exactly four numeric array keys, known string module IDs, no duplicate module, no sparse/extra keys and empty locked sockets. Runs additionally require every active socket filled. Inputs must be finite integers from 0 to 999. Each intermediate operation is clamped, including before a subsequent halving step.

```luau
function MathEngine.evaluate(input: number, layout: { string }, socketCount: number): ({ number }?, string?)
	if not MathEngine.isInteger(input, 0, 999) then
		return nil, "Input must be an integer from 0 to 999."
	end
	local valid, reason = MathEngine.validateLayout(layout, socketCount, true)
	if not valid then return nil, reason end
	local trace = { input }
	local value = input
	for index = 1, socketCount do
		value = math.clamp(operations[layout[index]](value), 0, 999)
		table.insert(trace, value)
	end
	return trace, nil
end
```

### Reject stale or overlapping actions

The handler validates the exact command shape and a per-session token bucket before these guards. Every accepted mutation advances the revision; delayed completion is tied to its run ID and the live session. Copying snapshots and incoming layouts prevents external table mutation from changing server-owned state.

```luau
if not engine.isInteger(payload.revision, 0, 9007199254740991) or payload.revision ~= revision then
	reject("Layout changed. Please use the refreshed workbench.")
	return
end
if status == "RUNNING" then reject("Wait for the machine to finish."); return end
```

The prototype's rate budget is a six-command burst, refilling at four per second. Rate-limit notices are throttled too, preventing one rejected request from producing unlimited responses. A session is destroyed when its player leaves.

## Mobile interaction design

The ScreenGui uses Roblox's safe-area rectangle, four ordered socket buttons, a bottom inventory and a 56px RUN button. Modules can be dragged or placed with two taps; an occupied source swaps with its destination. A pending command locks input until the server acknowledges it. A missed reply exposes Sync without blindly replaying the previous mutation.

The same pure geometry module drives both the actual interface and the viewport tests. Normal layouts fit without scrolling from 320 × 480 portrait and 568 × 280 landscape usable viewports. Stage 2 adds a scrollable canvas for smaller safe areas, preserving interactive targets of at least 44 × 44 pixels instead of shrinking them. Sensor orientation is configured in StarterGui and the active PlayerGui. Rotation cancels unfinished local gestures without resetting the server session. Geometry checks do not establish real-world touch usability.

## Verification completed

Stage 1 checks, recorded during the 28 October 2026 continuation on macOS:

- **31 Luau tests passed.** These cover all 360 legal layouts, all authored contract solutions, per-step clamping, invalid/forged payloads, stale revisions, duplicate runs, session isolation, delayed cleanup and 331 viewport cases. Input 2 produces 31 distinct final outputs, from 0 through 36; 360 layouts are not 360 unique answers.
- **Pure modules and test runner passed strict Luau analysis.** All six source files and the separate Studio smoke script passed syntax compilation. Roblox-dependent type validation is not covered by the standalone analyzer.
- **Rojo 7.7.0 built the local place.** The generated XML contains 19 mapped instances, including the expected scripts and two RemoteEvents, with zero 3D asset instances.
- **A local Studio client/server smoke check passed.** Through real RemoteEvents it rejected a forged result, produced 7 then 10 for the two orders and advanced to a four-socket contract. The workbench was visually observed in Studio. This was a scripted integration check, not an actual phone drag test.
- **19 private playtest-summary tests passed.** Unknowns, incomplete sessions, duplicate participants, mixed builds, contradictory no-action claims and malformed timing values cannot silently produce a passing study.

### Stage 2 mobile preparation · 28 October 2026

The updated build passes **32 Luau tests**, including **688 viewport case executions** across normal and small safe areas. Pure strict analysis and syntax compilation passed. Rojo 7.7.0 builds 20 mapped instances, now including StarterGui orientation, with zero 3D assets. A local release manifest records source and place-file hashes; no published destination is claimed.

The private study tooling passes **34 Python tests**. It checks matching anonymous filenames, timing boundaries, historical-protocol separation, incomplete observations, contradictory records and exact decision thresholds. The actual blank template and command-line report path are covered. Synthetic fixtures remain separate from participant evidence. The earlier Studio integration check was not rerun against this new layout; actual phone input and cloud publication remain pending.

## The player evidence still needed

The planned twelve-person first-encounter phone screen records unaided understanding (internal target 8/12), voluntary attempts of three further contracts (6/12), meaningful choice changes and first meaningful action time. Attempts and completions remain separate. The observation summary currently has **zero participants** and produces no verdict.

Server counters measure accepted actions. They cannot establish that someone understood unaided, acted voluntarily or made a meaningful decision. Those judgements remain human observations; the study does not substitute raw click counts for engagement. Native phone touch, rotation, reduced motion, latency recovery and first-use readability still need validation.

Persistence, purchase receipts, production assets and commercial claims are outside this disposable test. Passing it would justify a next experiment, not establish retention, revenue or a production-ready game.

## Role and references

Product direction, constraints, scope and review by Yassein; implementation and testing developed with AI assistance. This portfolio shares curated original excerpts and scoped evidence. The full private workspace and participant records are not published.

The authority model follows [Roblox's client-server security guidance](https://create.roblox.com/docs/scripting/security/client-server-boundary); safe-area properties follow the [ScreenGui reference](https://create.roblox.com/docs/reference/engine/classes/ScreenGui). Editing uses the [official Rojo workflow](https://rojo.space/docs/v7/getting-started/installation/).

[Back to the portfolio](README.md) · [Viewing and reuse notice](LICENSE)

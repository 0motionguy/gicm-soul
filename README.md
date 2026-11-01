# @gicm/soul

Mode-based identity prompts for agents: classify a message, load one of six persona fragments and keep the identity part of the system prompt small.

OpenClaw agents describe their identity in a SOUL.md file. Soul splits that identity into six mode fragments (BUILD, THINK, VIBE, TRADE, CREATE, AUDIT), scores each incoming message against keyword and pattern rules and returns only the fragment that fits. It is for developers who want an agent to switch tone and tooling by task without carrying every persona in every prompt.

Part of [gICM](https://github.com/0motionguy/gICM), a family of OpenClaw agent packages. API notes and examples: [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md).

## What it does

- **Classify:** `classifyMode(message)` adds 5 points per matching mode pattern and 1 per keyword, turns the best score into a confidence from 0 to 1 (score divided by 10) and falls back to the default mode at 0.3 or below.
- **Resolve:** `resolve(message)` switches mode when `autoSwitch` is on and returns `{ mode, systemPrompt, tokenCount, fullTokenCount, savedPercent }`.
- **Switch by hand:** `switchMode(mode)`, `getCurrentMode()` and `getTokenSavings()`, with `mode:detected` and `mode:switched` events.
- **Default fragments:** each mode has identity text, a preferred tool list, a style note and a token cost. Override any of them with a `{mode}.json` file in a directory you pass as `fragmentsDir`.
- **Templates:** `skill/templates/` holds five SOUL.md starting points (creative, research, security, trading, work).

Token costs are fixed estimates stored on each fragment, and the savings percentage compares one fragment with the sum of all six. It is arithmetic on those estimates, not a tokenizer measurement.

## Example

```ts
import { SoulEngine } from "@gicm/soul";

const soul = new SoulEngine({ defaultMode: "BUILD", autoSwitch: true });
const { mode, systemPrompt, tokenCount, fullTokenCount } =
  soul.resolve("audit this contract for reentrancy");
```

## How it's built

TypeScript (ESM) bundled with tsup and tested with vitest; zod defines the mode enum. Classification is regular expressions plus keyword lists in `src/modes.ts`; fragments live in `src/fragments.ts`. There is no model call and no network access.

## Repository layout

- `src/`: `soul-engine`, `modes`, `fragments`, `types`
- `src/__tests__/`: vitest suite (29 test cases)
- `skill/`: `SKILL.md` (the OpenClaw skill definition) and `templates/`

## Run it locally

```bash
pnpm install
pnpm build   # tsup -> dist/
pnpm test    # vitest (add --run for a single pass)
```

## Status

Beta. Version 1.0.0, a single extracted release (February 2026) with a test suite. The `userPrefsPath` option is accepted but not used yet.

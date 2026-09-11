# touchstone

> **试金石** — a per-repo fact-base of verified project conclusions, auto-loaded into every session.

> Docs: README in English; DESIGN/SKILL internals in Chinese (functional for any bilingual-capable agent). English trigger phrases are listed in the skill description.

**touchstone** solves one problem: AI assistants (Claude Code, Codex, and similar coding agents) accumulate conclusions without confidence levels or lifecycle — outdated "facts" get cited as truth weeks later, silently wasting work.

With touchstone, every repo gets a `TOUCHSTONE.md`: a time-ordered ledger of conclusions in **PCR format** (Project Conclusion Records, modeled on ADR conventions), governed by a **five-state machine**, auto-injected into session context by a hook. The AI drafts; **you** decide what becomes a verified fact.

## Install

`TOUCHSTONE.md` itself is **tool-agnostic** — any coding agent that can read a markdown file can use it. The layers below add automation; pick the one matching your tool.

### Claude Code (and compatible CLI plugins) — full automation

Requires: [Node.js](https://nodejs.org) ≥ 18 on PATH (hook also assumes a POSIX shell; without it the hook silently skips — the ledger itself still works).

```
/plugin marketplace add game1991/touchstone
/plugin install touchstone@touchstone
```

On the next session, any repo containing a `TOUCHSTONE.md` at its git root gets its index auto-injected (SessionStart hook); the `touchstone` skill activates on conversational triggers like "this is verified" / "we shipped X" / "that conclusion is outdated".

> The bundled SessionStart hook is Claude Code-specific. Other agents skip it (step 2 below covers them). The `.kscc-plugin/` directory mirrors `.claude-plugin/` for Kscc Code (a Claude Code-compatible CLI) — the manifests are identical.

**Next step →** [Quick start](#quick-start) to create your first `TOUCHSTONE.md` — the plugin is inert until that file exists.

### Other agents (Codex CLI, Cursor, Aider, …) — manual install

1. Copy `skills/touchstone/SKILL.md` into your tool's instruction/skill directory, or append its rules to your `AGENTS.md` / system prompt:

   ```markdown
   ## touchstone (project conclusion lifecycle)
   - At session start, if `<git-root>/TOUCHSTONE.md` exists, read its index zone.
   - Only cite entries marked GA (archived). Candidate-zone entries are unverified leads.
   - When the user announces a milestone, draft a PCR entry into the candidate zone;
     ask "archive this?" before marking GA. Never self-archive.
   - Superseded entries are kept with bidirectional links, never deleted.
   ```

2. Optional automation: any mechanism that injects text at session start can call
   `hooks/session-start.js` (reads stdin JSON `{cwd}`, emits the index header, fail-open).
   Without it, just teach the agent to read the file on demand — the ledger still works.

The core value — the ledger and its lifecycle discipline — lives in the markdown file, not the harness.

*Smoke test: tell your agent "record this: we verified X" — it should draft a candidate entry and ask you before marking it archived.*

## How it works

| Layer | Mechanism | Effect |
|---|---|---|
| Skill | `touchstone` skill with conversational triggers | Drafts candidate entries (BETA), enforces the archive gate, runs supersede flows |
| SessionStart hook | Injects first 30 lines of the index + candidate count | Fact-base is present without the AI "remembering" to read it |
| Your file | `<git-root>/TOUCHSTONE.md` | The ledger itself — in git, visible to the team, diffable |

## The five-state machine

```
BETA (assumption, unverified) → RC (under verification) → GA (archived, user-approved)
                                     ↘ SUPERSEDED (refuted — kept, cross-linked, never deleted)
                                     ↘ EOL (obsolete, no successor)
```

Two hard gates:
1. **Only the user can mark an entry GA** — the AI drafts into the candidate zone and must ask before archiving.
2. **Entries are immutable** — changes are new entries with bidirectional links (`superseded: 0910-xxx`), refuted conclusions are the most expensive knowledge you have.

## Quick start

Copy [`TOUCHSTONE-TEMPLATE.md`](TOUCHSTONE-TEMPLATE.md) to your repo root as `TOUCHSTONE.md`, or just ask your AI: *"初始化这个仓库的 TOUCHSTONE.md"* (the skill knows the template).

Entry format (one line in the index, full entry in the body):

```markdown
## 2026-09-11 · Identity model: cookie auth removed 【GA】
- Conclusion: sole identity chain = X-App-Key + user_name
- Evidence: commit <sha> · CI <build#> · 3 prod probes
- Supersedes: 0910-"platform forwards only its own session cookie"
- Scope: the MCP service and future consumers
```

## Anti-rot safeguards

- Index capped at 200 lines, hook injects only 30 — retrieval-first, never bulk-injected
- Single write path: body entry + index line in one edit (no silent index drift)
- Candidate zone capped at 10, 30-day TTL — tolerate discarding, hoarding kills ledgers
- **Retirement criterion built in**: two weeks with zero retrieval hits → the AI itself proposes retiring the file

## Uninstall

- Claude Code: `/plugin uninstall touchstone@touchstone` — the skill and hook go away; nothing else is modified (the plugin never writes outside your repos).
- `TOUCHSTONE.md` files in your repos can stay (plain markdown, human-readable) or be deleted — your call. Other agents only read it if instructed to.

## Documentation

- [DESIGN.md](DESIGN.md) — full design (motivation, five-state machine, review records)
- [TOUCHSTONE-TEMPLATE.md](TOUCHSTONE-TEMPLATE.md) — file skeleton and entry templates
- [HANDOVER.md](HANDOVER.md) — implementation history

## License

MIT

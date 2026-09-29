# Agent Team Setup

**A skill that turns a repository into a multi-agent workspace.**

Drop an orchestrator agent into any project and it builds the full protocol layer — roles, zones, mailboxes, ownership gates, quality gates — so several agents (backend, frontend, mobile, admin, QA — any composition) can work in the same repo for weeks without stepping on each other or burning your context on re-explaining the project.

> The skill prompt is written in Russian. The artifacts it generates (`AGENTS.md`, `HANDOFF.md`, zone READMEs, mailboxes) follow the same language as the interview — the skill works on any project, in any language, because it never assumes your stack. It asks.

---

## The problem it solves

A single agent working alone is fine. Three agents in one repo is not:

- The frontend agent rewrites the backend's contract docs (or doesn't know they exist).
- Nobody knows which port the backend runs on, so integration is guesswork.
- Every new session starts from zero — you re-explain the project, the roles, the conventions. Context window after context window.
- "Don't touch other people's code" is a sentence in a doc that nothing enforces.

This skill fixes all four, and the fix is **structural, not documentary**: the rules it writes are backed by files a script actually checks.

---

## What it generates

| Artifact | Purpose |
|---|---|
| `AGENTS.md` | The constitution: roles table, entry order, dev-ports, mail discipline, quality gates, escalation rules |
| `HANDOFF.md` | **The single entry point.** State of the project, architecture, TODO — what a fresh agent reads first |
| `<zone>/README.md` | One per zone: stack, port, skills, definition-of-done, append-only "rakes" (gotchas) section |
| `docs/mailbox/to-<role>.md` | Inter-agent mailboxes with a "whose box is this" guard in the header |
| `zones.yml` + pre-commit hook | **Ownership gate:** a commit touching another zone's files is blocked. "Don't touch others' code" becomes a check, not a sentence |
| `check_protocol.py` | **The doctor:** all mailboxes present and guarded, every gate named in `AGENTS.md` actually runs, port table complete, no duplicates |
| `.zcode/skills/security-review/SKILL.md` | Project-local security-review skill, wired into the commit gates of the zones it applies to |
| `.gitignore` guards | Session artifacts (`.zcode/plans/`, `.zcode/shots/`) and secrets stay out of git |

Optional, if the project has an API or a DB: a **docs↔code sync gate** — changed a router or a model? The contract docs must be updated in the same commit, and the sync script must be green, or the work isn't done.

---

## How it works

1. **Recon first (step 0).** The orchestrator reads the repo *before* asking you anything. Empty project → full setup. Protocol already there → **sync mode**: it diffs the existing protocol against the skill and proposes only what's missing. Adding a zone → **extension mode**.
2. **Interview (step 1).** This is the core. Nothing is generated until you answer. The skill interviews you in blocks — team composition, stack per zone (2–3 options with pros/cons and a recommendation each — it never assumes your stack), dev-ports, integrations. "Haven't decided yet" is a valid answer: the zone then starts under the **first-session rule** (the zone agent must ask you for stack options before writing its first line of code).
3. **Generation (steps 2–4).** All placeholders filled from your answers. The generated protocol passes its own doctor check — a red doctor means the setup isn't finished.
4. **Verification + local commit (step 5).** The orchestrator commits locally. **It never pushes without an explicit order from you.**
5. **Starter prompts (step 6).** You get a copy-paste first message for each role. The prompt doesn't duplicate the protocol — it routes the agent into it: *read AGENTS.md → HANDOFF.md → your zone README → your mailbox → here's your task.*

---

## Install

### Claude Code / ZCode / any skills-compatible agent

```bash
git clone https://github.com/IndexCorp/agent-team-setup.git ~/.agents/skills/agent-team-setup
```

The skill lives in the `~/.agents/skills/` tree and is picked up automatically.

### Any other agent (or a single repo)

Clone it anywhere and point your agent at `SKILL.md`, or copy the whole folder into your project's skill directory:

```bash
git clone https://github.com/IndexCorp/agent-team-setup.git .zcode/skills/agent-team-setup
```

**Requirements:** none. No dependencies, no build step, no scripts to run. The skill is pure prompt — `SKILL.md` plus nine templates in `references/`. If your agent can read files, the skill works.

---

## Your first request

Copy this into a fresh chat, replace the path and the spec:

```
You are the organizer of my project. Use the skill
[$agent-team-setup](~/.agents/skills/agent-team-setup/SKILL.md).

The project spec: @project-spec.md
```

That's it. The orchestrator will read the skill, run recon on your repo, and start the interview — team composition, stack per zone, ports. Answer the blocks; it generates the protocol and commits locally.

### Then, to start a work session

Use the starter prompts the setup hands you. They look like this:

```
You are the backend agent of the project (repo: /path/to/project).
Read in order, before any change: AGENTS.md → HANDOFF.md →
backend/README.md → docs/mailbox/to-backend.md.

First, check your mailbox: assignments in NEW status — take them
(TAKEN) or explain why not. End the session with a local commit:
code + checks + docs + mail to neighbors — one commit, no push.

Task for this session: <what you want done>. If the zone's stack is
not fixed in its README — offer me stack options first and wait.
```

---

## Why you can work with it for months without worrying about context

The protocol is designed so that **context is carried by the repo, not by the conversation**. Everything an agent needs to be useful on day one lives in files that outlive the chat:

- **One entry point.** A fresh agent reads `HANDOFF.md` and knows the state of the project. You don't re-explain anything — you hand over a path.
- **Roles survive sessions.** Your role is in the first message; the rules are in `AGENTS.md`. A new chat is not a new project — it's a continuation, because the agent re-derives its place from the protocol.
- **Knowledge accumulates in the repo, not in your head.** Every zone README has an append-only **"rakes"** section: symptom → cause → workaround. An agent that hits a trap adds a line; the agent that hits it three months later reads it first. Gotchas that no longer apply are removed by whoever fixed the cause.
- **Rules are enforced by files, not by memory.** The `zones.yml` pre-commit hook blocks a commit that touches another zone. `check_protocol.py` verifies that every gate named in `AGENTS.md` actually runs. You stop being the one who remembers "we agreed not to do this."
- **Handoffs are written, not spoken.** Agents talk through `docs/mailbox/to-<role>.md` — assignments with statuses (NEW → TAKEN → DONE), bug reports with steps and expected-vs-actual. You can step out for a month, come back, and read the mail to see where things are.
- **Disputes have a path.** Agents escalate in writing, both sides mark `NEEDS_CONTEXT`, and **you are the only arbiter** — your decision is recorded in the docs by the agent that implements it. Nobody silently does it their way while a dispute is open.
- **The docs are the current state, not a history.** Contracts and rules describe what's true now; there are no "here's what we deleted" notes. The protocol stays small enough to read in one sitting, months in.

The practical effect: the cost of a session stops growing with the age of the project. Week one and month six cost the same context, because the project carries itself.

---

## Skill structure

```
SKILL.md                              # orchestrator prompt (6 steps + principles)
references/
├── agents-md-template.md             # AGENTS.md skeleton
├── handoff-template.md               # HANDOFF.md skeleton
├── zone-readme-template.md           # per-zone README
├── mailbox-template.md               # inter-agent mailbox
├── stack-options.md                  # stack catalogue (checked against official docs)
├── role-prompts.md                   # starter prompts for each role
├── security-review-template.md       # project security-review skill
├── clean-code-template.md            # clean-code convention block
├── docs-sync-template.md             # docs↔code sync gate (API/DB projects)
└── zones-doctor-template.md          # ownership gate + protocol doctor
```

---

## Limits, stated plainly

- **It writes protocol, not product code.** The orchestrator builds the workspace; zone agents fill it.
- **It interviews you.** If you want a skill that silently assumes your stack, this is not it. Every choice is yours, with recommendations.
- **No push, ever, without an order.** Every generated protocol ends in a local commit. Pushing is your call.
- **The generated gates are code in your repo.** `check_protocol.py`, the pre-commit hook, the docs-sync script — they live with your project and your zone agents maintain them like any other code.

---

MIT

---
name: jobify
description: >-
  AI job search command center -- evaluate offers, generate CVs, scan portals,
  track applications. Use when the user pastes a job URL or JD, asks to scan
  portals, generate a CV/PDF, track applications, prepare for interviews, draft
  outreach/emails, or run any jobify mode.
arguments: mode
user_invocable: true
user-invocable: true
argument-hint: "[scan | discover | deep | pdf | text | latex | latex-tex | cover | email | add | expand | eu-swe | oferta | ofertas | apply | batch | tracker | agent-inbox | pipeline | contacto | training | project | interview-prep | interview | interview/plan | interview/practice | interview/debrief | interview-redflag | patterns | offer-prep | titles | upskill | followup | reply-watch | outcome | update]"
license: MIT
---

# jobify -- Router

jobify is a multi-CLI job-search command center. The routing below is shared across supported agent CLIs even when the invocation surface differs.

## Project Root Resolution

Before reading any repo-relative path, derive `PROJECT_ROOT` from this loaded `SKILL.md`: start at the skill file's directory and walk upward until the nearest directory containing both `AGENTS.md` and `modes/`. Resolve every path in this router (`modes/`, `config/`, `data/`, scripts, templates, and output paths) against `PROJECT_ROOT`, never against the process's current working directory. This is required even when the checkout itself is nested (for example `Development\\jobify`) or the command starts from a subdirectory. If those two sentinels cannot be found, stop and locate the jobify checkout before reading or writing files.

## Invocation Notes

- CLIs with slash-command registration can expose this router as `/jobify`.
- In Cursor, this skill lives at `.cursor/skills/jobify/` and is auto-discovered; ask for a mode by name, or paste a JD/URL to trigger auto-pipeline.
- Interactive Codex sessions use `codex` in the repo root. Slash commands are not guaranteed in Codex, so ask Codex to run the same mode by name if `/jobify` is unavailable.
- Headless Codex workers use `codex exec "prompt"`.
- The routing semantics below stay the same regardless of whether the entrypoint is a slash command or a natural-language prompt.

Codex prompt examples that map to the same router semantics:

```text
Evaluate this JD with jobify auto-pipeline: https://company.com/jobs/123
Run the jobify scan mode and summarize new matches.
Run the jobify pipeline mode for data/pipeline.md.
Run the jobify pdf mode for the latest evaluated role.
Run the jobify tracker mode and summarize the current statuses.
```

## Mode Routing

Determine the mode from `$mode`:

| Input | Mode |
|-------|------|
| (empty / no args) | `discovery` -- Show command menu |
| JD text or URL (no sub-command) | **`auto-pipeline`** |
| `oferta` | `oferta` |
| `ofertas` | `ofertas` |
| `contacto` | `contacto` |
| `deep` | `deep` |
| `interview-prep` | `interview-prep` |
| `interview` | `interview` |
| `eu-swe` | `regional/eu-swe` |
| `eu-fintech` | `regional/eu-fintech` |
| `interview/plan` | `interview/plan` |
| `interview/practice` | `interview/practice` |
| `interview/debrief` | `interview/debrief` |
| `pdf` | `pdf` |
| `text` | `text` |
| `latex` | `latex` |
| `latex-tex` | `latex-tex` |
| `email` | `email` |
| `add` | `add` |
| `expand` | `expand` |
| `training` | `training` |
| `project` | `project` |
| `tracker` | `tracker` |
| `agent-inbox` | `agent-inbox` |
| `inbox` | `agent-inbox` |
| `pipeline` | `pipeline` |
| `apply` | `apply` |
| `scan` | `scan` |
| `discover` | `discover` |
| `batch` | `batch` |
| `patterns` | `patterns` |
| `offer-prep` | `offer-prep` |
| `titles` | `titles` |
| `upskill` | `upskill` |
| `followup` | `followup` |
| `reply-watch` | `reply-watch` |
| `outcome` | `outcome` |
| `interview-redflag` | `interview-redflag` |
| `update` | `update` |
| `cover` | `cover` |

**Auto-pipeline detection:** If `$mode` is not a known sub-command AND contains JD text (keywords: "responsibilities", "requirements", "qualifications", "about the role", "we're looking for", company name + role) or a URL to a JD, execute `auto-pipeline`.

If `$mode` is not a sub-command AND doesn't look like a JD, show discovery.

---

## Output Language Directive

Before executing any mode, read `config/profile.yml` if it exists and resolve:

- `language.output` → ISO language code for human-facing output. Default: `en`.
- `language.modes_dir` → optional market-mode directory. This controls market vocabulary and local evaluation rules only.

Inject this directive after loading the mode instructions and before producing any user-visible content:

> Write all human-facing output in `{language.output}` regardless of the language of these instructions or of the job description. This includes reports, tracker notes, PDFs, cover letters, outreach, interview prep, form answers, and summaries. If `language.modes_dir` supplies market-specific vocabulary, keep the market logic but explain terms in `{language.output}` when needed.

`language.output` is authoritative for prose. `modes_dir` is market context; it must not force the prose language.

---

## Discovery Mode (no arguments)

If your CLI supports `/jobify`, show this menu. In Codex, surface the same options in plain text and map the requested mode the same way.

Concrete equivalents for Codex prompt-driven sessions:

```text
/jobify {JD}           ↔ "Evaluate this JD with jobify auto-pipeline: {JD or URL}"
/jobify scan           ↔ "Run the jobify scan mode and summarize new matches."
/jobify pipeline       ↔ "Run the jobify pipeline mode for data/pipeline.md."
/jobify pdf            ↔ "Run the jobify pdf mode for the latest evaluated role."
/jobify email          ↔ "Run the jobify email mode for the latest evaluated role."
/jobify tracker        ↔ "Run the jobify tracker mode and summarize the current statuses."
```

Show this menu:

```
jobify -- Command Center

Available commands:
  /jobify {JD}      → AUTO-PIPELINE: evaluate + report + PDF + tracker (paste text or URL)
  /jobify pipeline  → Process pending URLs from inbox (data/pipeline.md)
  /jobify oferta    → Evaluation only A-F (no auto PDF)
  /jobify ofertas   → Compare and rank multiple offers
  /jobify contacto  → LinkedIn power move: find contacts + draft message
  /jobify deep      → Deep research prompt about company
  /jobify interview-prep → Generate company-specific interview prep doc
  /jobify interview    → Interactive profile/CV onboarding interview
  /jobify eu-swe    → Calibrate a European SWE application before CV/apply/interview
  /jobify eu-fintech → Scan 21 EU fintech portals for Product Manager roles (zero-token)
  /jobify interview/plan → Time-blocked prep plan for an upcoming interview
  /jobify interview/practice → Practice interview, one question at a time with feedback
  /jobify interview/debrief → Post-interview debrief: close gaps, predict next round
  /jobify pdf       → PDF only, ATS-optimized CV
  /jobify text      → Tailored markdown CV (mirrors cv.md, no PDF)
  /jobify latex     → Export CV as LaTeX/Overleaf .tex
  /jobify latex-tex → Tailor your own resume.tex in place (opt-in; cv.md stays default)
  /jobify cover     → Cover letter: standalone JD paste or /jobify cover {slug}
  /jobify email     → Formal application email draft (draft-only; never sends, submits, or clicks)
  /jobify add       → Add a project/paper/role to your CV (fetch + preview + confirm)
  /jobify expand    → Auto-discover and add missing competencies from profile links
  /jobify training  → Evaluate course/cert against North Star
  /jobify project   → Evaluate portfolio project idea
  /jobify tracker   → Application status overview
  /jobify agent-inbox → Queue/drain requests for the next session (data/agent-inbox.md)
  /jobify apply     → Live application assistant (reads form + generates answers)
  /jobify scan      → Scan portals and discover new offers
  /jobify discover  → Resolve a company list to scannable ATS boards + append to portals.yml (zero-token)
  /jobify batch     → Batch processing with parallel workers
  /jobify patterns  → Analyze rejection patterns and improve targeting
  /jobify offer-prep → Read a received offer/contract with the candidate: clause walk + lawyer questions (not legal advice)
  /jobify titles    → Suggest adjacent job titles from your CV to broaden the search
  /jobify upskill   → Aggregate skill-gap analysis from your evaluated reports
  /jobify followup  → Follow-up cadence tracker: flag overdue, generate drafts
  /jobify outcome   → Record application outcome & archive artifacts
  /jobify update    → Update jobify system files with diff preview + compat check

Inbox: add URLs to data/pipeline.md → /jobify pipeline
Or paste a JD directly to run the full pipeline.
```

---

## Context Loading by Mode

After determining the mode, load the necessary files before executing:

If `modes/_custom.md` exists, read it after `modes/_profile.md` and before the selected mode file. It contains user house rules and procedural preferences. It may override workflow/style defaults, but it never adds factual claims about the candidate.

### Modes that require `_shared.md` + their mode file

Read `modes/_shared.md` + `modes/_profile.md` (if exists) + `modes/_custom.md` (if exists) + `modes/{mode}.md`

Applies to: `auto-pipeline`, `oferta`, `ofertas`, `pdf`, `text`, `contacto`, `apply`, `pipeline`, `scan`, `batch`

### Standalone modes with profile and custom context

Read `modes/_profile.md` (if exists) + `modes/_custom.md` (if exists) + `modes/{mode}.md`

Applies to: `tracker`, `agent-inbox`, `deep`, `interview-prep`, `interview`, `regional/eu-swe`, `interview/plan`, `interview/practice`, `interview/debrief`, `latex`, `latex-tex`, `training`, `project`, `patterns`, `titles`, `upskill`, `followup`, `reply-watch`, `outcome`, `cover`, `email`, `add`, `offer-prep`, `discover`

### Modes delegated to subagent

For `scan`, `apply` (with Playwright), and `pipeline` (3+ URLs): launch as a worker/subagent with the content of `_shared.md` + `_profile.md` (if exists) + `_custom.md` (if exists) + `modes/{mode}.md` injected into the worker prompt. If your CLI exposes an `Agent(...)` primitive, the call looks like this:

```python
Agent(
  subagent_type="general-purpose",
  prompt="[output language directive]\n\n[content of modes/_shared.md]\n\n[content of modes/_profile.md if exists]\n\n[content of modes/_custom.md if exists]\n\n[content of modes/{mode}.md]\n\n[invocation-specific data]",
  description="jobify {mode}"
)
```

Execute the instructions from the loaded mode file.

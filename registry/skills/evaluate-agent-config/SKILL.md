---
name: evaluate-agent-config
description: Scores a project's skills, agents and CLAUDE.md files against a rubric, then proposes and (on approval) applies fixes that raise the score. Reports a 0-100% score per file and per factor, with quoted evidence and plain-words fixes with a gain. Use when the user asks to evaluate, review, audit, score, lint or improve a skill, an agent definition, a CLAUDE.md, or the whole `.claude/` setup of an imported project.
allowed-tools:
  - read_file
  - glob
  - grep
  - ls
  - execute
  - edit_file
metadata:
  version: "1.0.0"
---

# Evaluate Agent Config

Scores skills, agents and CLAUDE.md files, then improves them. Three layers: fixed checks, judged factors, fixes.

## Inputs

`$ARGUMENTS` is a path. Classify each target:

- **Skill**: a directory with `SKILL.md`, under `.claude/skills/` or a root `skills/`.
- **Agent**: a `.md` file under `.claude/agents/` or a root `agents/`, with `name` and `description` frontmatter.
- **CLAUDE.md**: any file named `CLAUDE.md` or `AGENTS.md`, at any depth.
- **Project root**: discover all three with `find <root> -name node_modules -prune -o \( -path '*/.claude/skills/*/SKILL.md' -o -path '<root>/skills/*/SKILL.md' -o -path '*/.claude/agents/*.md' -o -path '<root>/agents/*.md' -o -name CLAUDE.md -o -name AGENTS.md \) -print`.

If the path matches nothing, stop and say what you looked for.

## Workflow

1. **Discover.** Classify every target. Read each file. For a skill, also read the files beside `SKILL.md` (scripts, references, assets): at most 10, shallowest first, each cut at 20 KB. Skip a file over 100 KB. Score at most 50 files per run, `CLAUDE.md` first, then agents, then skills.
2. **Fixed checks.** Run the checks for its type from [rubric.md](references/rubric.md#fixed-checks). Show the command output. Do not describe a check you did not run.
3. **Judge.** Score each factor 1-5 using the anchors in [rubric.md](references/rubric.md#judged-factors). Every score needs a quoted line from the file as evidence (or "none" if the file lacks it) and one sentence on what the file does or lacks. That sentence never mentions a score, number, rating or the rubric. Use a skill's supporting files only to check that what `SKILL.md` says and links to is real. Grade `SKILL.md` alone.
4. **Score.** Apply the formula below.
5. **Report.** Use the output format below. Put the lowest-scoring factors first.
6. **Fix.** Only when the user asked for `--fix` or approves after the report: show each fix as a Fix block, get approval, edit with the Edit tool, then re-run steps 2-4 and show the score change.

## Scoring

- Factor score `s` (1-5) becomes `(s - 1) / 4`. So 1 = 0%, 3 = 50%, 5 = 100%.
- A judge's score is the weighted mean of its factors. Weights are in the rubric.
- The overall score is the weighted mean of the judges, using the judge weights in the rubric.
- Caps (our rule): a hard-coded secret, or a destructive command with no confirmation step, caps the overall score at 60%. Say when a cap applied.
- A fix's gain is the points the file's score rises if its factor reaches 5/5, before caps: `judge weight share × factor weight share × (5 - s) / 4 × 100`. Shares are weight ÷ the sum of weights at that level. Round to one decimal.
- A fixed-check ERROR caps the overall score at 60% and is listed first in the fixes. Judging still runs, so the file gets a full score breakdown.

## Output format

Per file:

```
<path>  (<type>)  Overall: NN%
Fixed checks: N passed, N warnings, N errors
Judge        Weight  Score
<name>       NN%     NN%
  <factor>: s/5 - "<quoted evidence>" - <why this score>

Priority | Fix | Factor | Gain
```

Then one Fix block per fix:

```
## <Action verb>: <what to change>
Factor: <exact factor key>  Gain: +N points
Why: <a short paragraph in plain words: what is missing or wrong, why it matters to someone using the file, how to fix it>
Suggestion:
> <exact text to add; leave out when the fix is not an addition>
```

Write fixes as a reviewer giving advice. A fix never mentions scores, points, factor names or the rubric in its Why. The factor is the exact key of one scored factor, never a judge name or a section of the file.

For a project root, end with a table of all files sorted by overall score, lowest first.

## Rules

- Quote, don't paraphrase. A finding without a quoted line is not a finding.
- Do not invent factors, weights or scores. Use only the rubric.
- Ignore template placeholders such as `[type](MODULE_NAME)` when checking links.
- A fix that conflicts with the file's purpose is a trade-off. Present both options and ask.
- Fix only what a factor flagged. Do not rewrite passing text. A file that already scores 85%+ gets at most three edits.
- Do not rewrite scripts or assets. Report problems in them and leave them.
- Never print a secret in the report. Show the first four characters and `...`.

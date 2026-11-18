# Rubric

Factors score 1-5. Anchors show 1, 3 and 5. Use 2 and 4 for in-between. Weights are our proposal and should be tuned after a few real runs.

## Fixed checks

Run these with the shell. ERROR caps the overall score at 60% (judging still runs), WARN is reported, INFO is shown only.

### Skill

| Check                                               | Level         | How                                                                                                                |
| --------------------------------------------------- | ------------- | ------------------------------------------------------------------------------------------------------------------ |
| `SKILL.md` exists, frontmatter parses               | ERROR         | `head -20`, check `---` pair                                                                                       |
| `name` present and equals the folder name           | ERROR         | compare to `basename "$(dirname SKILL.md)"`                                                                        |
| `description` present, 80+ chars                    | WARN under 80 | `wc -c`                                                                                                            |
| `description` contains "Use when" or "Use whenever" | WARN          | `grep -i` on the frontmatter only                                                                                  |
| Line count 500 or less                              | WARN          | `wc -l`                                                                                                            |
| Every relative markdown link resolves               | WARN          | skip placeholders in `[type](UPPER_CASE)` form                                                                     |
| Every file in the bundle is linked from `SKILL.md`  | WARN          | orphan check                                                                                                       |
| `allowed-tools` is not `"*"`                        | WARN          | grep frontmatter                                                                                                   |
| No secret-shaped strings                            | ERROR         | `grep -nE '(sk-[A-Za-z0-9]{10,}                                                                                    | ghp_[A-Za-z0-9]{20,} | AKIA[0-9A-Z]{16} | Bearer [A-Za-z0-9._-]{20,})'` |
| No hard-coded home paths                            | WARN          | `grep -nE '/home/[a-z]                                                                                             | /Users/[A-Za-z]      | C:\\\\Users'`    |
| Destructive commands have a confirmation step       | WARN          | grep `rm -rf`, `git push --force`, `DROP `, `reset --hard`; check an ask-first line sits before it                 |
| Code blocks parse                                   | WARN          | `bash -n` for bash, `node --check` for JS, `python3 -I -c 'import ast,sys;ast.parse(sys.stdin.read())'` for Python |

### Agent

| Check                                                | Level | How                                            |
| ---------------------------------------------------- | ----- | ---------------------------------------------- |
| Frontmatter parses, `name` and `description` present | ERROR |                                                |
| `name` equals the file name without `.md`            | WARN  |                                                |
| `description` says when to delegate                  | WARN  | grep "Use when", "Use proactively", "delegate" |
| `tools` is listed and is not every tool              | WARN  | missing or `*` means full access               |
| Secret and home-path checks from Skill               | same  |                                                |

### CLAUDE.md

| Check                                                                                                                                                                  | Level | How                         |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----- | --------------------------- |
| Line count 200 or less                                                                                                                                                 | WARN  | `wc -l`                     |
| Every backticked path the file tells the reader to open or run exists (paths named only as background are INFO)                                                        | WARN  | resolve from the repo root  |
| Every `pnpm`/`npm`/`yarn` script mentioned exists in the nearest `package.json` (skip built-ins: `install`, `add`, `dlx`, `exec`, `run`, and flags such as `--filter`) | WARN  | compare with `scripts` keys |
| Every referenced `CLAUDE.md` or doc link resolves                                                                                                                      | WARN  |                             |
| No secret-shaped strings                                                                                                                                               | ERROR | same grep as Skill          |

## Judged factors

### Skill (4 judges)

**Description (weight 25%)**

| Factor               | Weight | 1                                                  | 3                                   | 5                                                         |
| -------------------- | ------ | -------------------------------------------------- | ----------------------------------- | --------------------------------------------------------- |
| specificity          | 30%    | No domain, no concrete action ("Helps with stuff") | Names a domain, actions are general | Lists every concrete action the skill does                |
| completeness         | 30%    | Neither what nor when                              | What is clear, when is vague        | What and an explicit "Use when" with cases                |
| trigger_term_quality | 25%    | No words a user would say                          | Some natural phrases                | Natural phrases and variants a user would type            |
| distinctiveness      | 15%    | Would clash with any skill                         | Clear domain, some overlap          | Clear niche, low overlap with other skills in the project |

**Content (weight 40%)**

| Factor                 | Weight | 1                                             | 3                                   | 5                                                               |
| ---------------------- | ------ | --------------------------------------------- | ----------------------------------- | --------------------------------------------------------------- |
| conciseness            | 20%    | Mostly filler or explains things Claude knows | Some padding or repeats             | Lean, every line earns its place                                |
| actionability          | 35%    | Vague direction only                          | Some concrete commands, gaps remain | Copy-paste commands, exact formats, edge cases covered          |
| workflow_clarity       | 30%    | No order, or steps contradict                 | Order exists, validation missing    | Numbered steps, checkpoints before risky actions, feedback loop |
| progressive_disclosure | 15%    | One blob, or core rules in a missing file     | Some structure, links vague         | Short body, links say when to read them, no orphans             |

Cap: no validation step after a risky operation caps workflow_clarity at 3. Contradicting instructions cap it at 2.

**Safety and portability (weight 20%) - our addition**

| Factor                    | Weight | 1                                                           | 3                                       | 5                                                     |
| ------------------------- | ------ | ----------------------------------------------------------- | --------------------------------------- | ----------------------------------------------------- |
| secrets_and_permissions   | 40%    | Secret in text, or `allowed-tools: "*"` with risky commands | No secret, tools broader than needed    | No secrets, narrowest tools that work                 |
| destructive_action_safety | 40%    | Destructive command, no confirmation, no scope              | Confirmation exists but target is broad | Confirmation, narrow target, a way to undo or dry-run |
| portability               | 20%    | Hard-coded user paths and hosts                             | Some environment assumptions            | Relative paths, env vars, nothing machine-specific    |

**Examples and failure handling (weight 15%) - our addition**

| Factor           | Weight | 1                                      | 3                                  | 5                                                       |
| ---------------- | ------ | -------------------------------------- | ---------------------------------- | ------------------------------------------------------- |
| examples         | 50%    | None, and the output format is unclear | A template only                    | A worked input and output for the common cases          |
| failure_handling | 50%    | No mention of what to do on failure    | Some retries or "report the error" | Each risky step has a failure path and a stop condition |

Judge weights for a skill: Description 25%, Content 40%, Safety 20%, Examples 15%.

### Agent (3 judges)

**Definition (weight 40%)**

| Factor             | Weight | 1                                             | 3                          | 5                                              |
| ------------------ | ------ | --------------------------------------------- | -------------------------- | ---------------------------------------------- |
| role_clarity       | 40%    | No role, or several unrelated roles           | One role, boundaries fuzzy | One role, what it does and does not do         |
| delegation_trigger | 40%    | Description says nothing about when to use it | Some cases                 | Explicit "use when" and "do not use when"      |
| distinctiveness    | 20%    | Overlaps most other agents                    | Some overlap               | Clear niche versus other agents in the project |

**Behavior (weight 40%)**

| Factor           | Weight | 1                          | 3             | 5                                           |
| ---------------- | ------ | -------------------------- | ------------- | ------------------------------------------- |
| output_contract  | 35%    | Free-form, no format       | Format hinted | Exact format the caller can rely on         |
| workflow_clarity | 30%    | No steps, or contradicting | Rough steps   | Ordered steps, checkpoints, stop conditions |
| conciseness      | 20%    | Padded                     | Some repeats  | Lean                                        |
| actionability    | 15%    | Vague                      | Mixed         | Concrete commands and criteria              |

**Safety (weight 20%)**

| Factor                  | Weight | 1                                             | 3                   | 5                         |
| ----------------------- | ------ | --------------------------------------------- | ------------------- | ------------------------- |
| tool_scope              | 50%    | All tools, or write tools on a read-only role | Broader than needed | Narrowest tools that work |
| secrets_and_portability | 50%    | Secrets or machine-specific paths             | Minor assumptions   | None                      |

### CLAUDE.md (3 judges)

**Accuracy (weight 40%)**

| Factor           | Weight | 1                                            | 3                     | 5                                          |
| ---------------- | ------ | -------------------------------------------- | --------------------- | ------------------------------------------ |
| command_accuracy | 50%    | Commands or scripts that do not exist        | Most work, some stale | Every command and path verified            |
| freshness        | 50%    | Describes a structure the repo no longer has | Mostly current        | Matches the repo today, no dead references |

**Usefulness (weight 40%)**

| Factor            | Weight | 1                                                      | 3                                   | 5                                                                             |
| ----------------- | ------ | ------------------------------------------------------ | ----------------------------------- | ----------------------------------------------------------------------------- |
| non_obvious_value | 40%    | Restates what the code or README shows, generic advice | Some project-specific facts         | Mostly things an agent cannot infer: why choices were made, traps, boundaries |
| completeness      | 35%    | Missing build/test commands or repo shape              | Commands present, architecture thin | Commands, repo shape, conventions, and cross-package rules                    |
| specificity       | 25%    | "Write clean code" style rules                         | Some concrete rules                 | Each rule is checkable and names the file, tool or flag                       |

**Structure (weight 20%)**

| Factor                 | Weight | 1                          | 3             | 5                                                                          |
| ---------------------- | ------ | -------------------------- | ------------- | -------------------------------------------------------------------------- |
| conciseness            | 50%    | Long, repeats itself       | Some padding  | Lean, nothing restated                                                     |
| progressive_disclosure | 50%    | One huge file, no pointers | Some sections | Short root file that points to per-package docs and says when to read each |

Judge weights for CLAUDE.md: Accuracy 40%, Usefulness 40%, Structure 20%.

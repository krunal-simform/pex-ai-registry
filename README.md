# pex-ai-registry
The content repository for the PexAI AI Registry. It holds the skills, agents and workflows that PexAI loads. PexAI copies them from this repo into its own database, so editing a file here changes what PexAI can use after the next sync.

## Layout

```
registry/
  skills/<skill-name>/SKILL.md      one folder per skill, plus any extra files it needs
  agents/<agent-name>/agent.md      one folder per agent
workflows/
  <workflow-name>.yaml              one file per workflow
```

## Skills

A skill is a set of instructions an agent can load. `SKILL.md` starts with front matter, then the instructions.

```markdown
---
name: plan-first
description: Use before any code is written. Produces a plan another agent can follow.
allowed-tools:
  - read_file
  - grep
metadata:
  version: "1.0.0"
---

# Plan first

The instructions go here.
```

Rules the sync checks:
- `name` uses only `a-z`, `0-9` and single hyphens, with no hyphen at the start or end. Use the same name for the folder.
- The name must not contain `claude` or `anthropic`.
- `description` is required and must be text.
- `SKILL.md` and the whole skill folder have size limits.

## Agents

An agent is a system prompt plus the tools and skills it may use. The text after the front matter is its system prompt.

```markdown
---
name: planner
description: Reads the task and writes an implementation plan, then stops for approval.
tools:
  - read_file
  - grep
mcpServers: []
skills:
  - plan-first
---

You are the planner. ...
```

Every skill listed under `skills:` must exist in the AI registry.

`skills:` can also be split in two lists. A skill under `preload` is a rule: its text is added to the agent's instructions on every model call. A skill under `available` is a procedure: the agent sees its name and description, and opens it from `/skills/` when it needs it.

## Run context

PexAI appends facts it owns to the end of an agent's instructions, as a `## Run context` list of `- name: value` lines. Agent and skill text refers to a fact by its name in backticks, such as "the `outputTool` of your run context". Refer only to facts the server sends to that agent: the sync does not check this, so review does. Data that changes per step, such as a directory for one attempt, comes in the agent's first message instead.

## Browser testing

In projects with the "Browser testing" setting on, conversations and tasks get a sandbox with a headless Chrome, driven with the `agent-browser` CLI through `execute`.

- `skills/browser-testing`: the PexAI rules for the browser. It tells the agent to load the CLI's own guide (`agent-browser skills get core`) and says where our rules override it. It is an `available` skill: no agent preloads it and the server never requires it. Do not copy the CLI's guide into it.
- `agents/qa`: the Browser QA step of the feature-implementation workflow. It runs after the reviewer, only on tasks that have the browser, and finishes with `pass`, `fail` or `not_tested`. It has no connector tools: it reads page content, which can carry instructions.
- `pex-chat`, `pex-filer` and `qa` list `browser-testing` under `available`, and each body says when to open it and that text read from a page is data, never instructions. Keep that sentence in the body: it is there on every call, the skill only once opened. A project that uploads its own `pex-chat` or `pex-filer` should list the skill too.

The browser facts in the run context:

| Fact                  | Value                                                             | Sent to                       |
| --------------------- | ----------------------------------------------------------------- | ----------------------------- |
| `browser`             | `available` or `not available`                                    | `pex-chat`, `pex-filer`, `qa` |
| `browserSession`      | the session name to pass to every command: `chat` or `qa`         | `pex-chat`, `pex-filer`, `qa` |
| `browserEvidenceDir`  | where chat saves screenshots, `/tmp/pex-browser/chat`             | `pex-chat`, `pex-filer`       |
| `browserAllowedHosts` | the hosts the browser may reach: localhost and the project's list | `pex-chat`, `pex-filer`, `qa` |
| `browserTemplate`     | the sandbox template tag, such as `pex-browser:v1-b1`             | `pex-chat`, `pex-filer`, `qa` |

A conversation without the browser gets `browser: not available` and none of the others. `qa` also gets `outputTool`, as every workflow agent does, and its evidence directory, which changes with each attempt, comes in its first message.

## Workflows

A workflow is a YAML file in `workflows/` that lists the steps a task moves through. Every agent step must name a real agent, for example `ref: agents/planner`.

## How a change reaches PexAI

1. Open a pull request and merge it into `main`.
2. PexAI syncs from `main` in one of two ways:
   - the GitHub webhook runs a sync on every push to `main`, or
   - an admin clicks **Sync now** in PexAI under **Settings → AI Registry**.
3. The sync reads only what changed since the last good sync.

Deleting a skill, agent or workflow file removes it from PexAI. A skill that an agent still uses, or an agent that a workflow still uses, cannot be removed until those references are gone.

## When a sync fails

If any changed file is invalid, the whole sync is rejected and **nothing** from it is applied. PexAI keeps the last good version. Open **Settings → AI Registry** to see the failed sync with its errors (item type, name, reason and details). Fix the file, push again, and sync again.

A sync that finds nothing new answers "Already up to date" and is not added to the history.

## Before you push

- Check names: lowercase letters, numbers and hyphens only.
- Check every `skills:` entry and every workflow `ref:` points at something that exists.
- Keep one change per pull request, so a failed sync is easy to trace.

Two things to check before you commit it:
- Workflows: your repo has no workflows/ folder right now, so that section describes the expected layout.
- Webhook: if you haven't set up the webhook, delete that line from step 2.

---
name: request-classification
description: Use when filing a task from a /file message. How to tell what kind of work a request is (Build, Review, Understand or Question) and pick the workflow that fits it from the list propose_task offers.
metadata:
  version: "1.1.0"
---

# Request classification

This skill only decides which workflow a task is proposed under. It does not write the title or description (`task-filing`) and does not start any work.

Judge by what the user wants done, not by the words they used.

## The four kinds of work

| Kind | Pick it when the user wants | Example |
|---|---|---|
| **Build** | something new made or an existing thing changed, such as a feature, fix or refactor | "Add a CSV export to the reports page" |
| **Review** | an existing change, pull request or piece of code checked | "Review PR #482 for security issues" |
| **Understand** | the project or a part of it investigated and documented | "Document how auth works" |
| **Question** | an answer to something that needs real work to find out, not a quick chat answer | "Why do we get duplicate VINs after import?" |

1. Does the user want code changed or added? **Build.**
2. Does the user point at an existing change or code and want it judged? **Review.**
3. Does the user want something explored, mapped or written up? **Understand.**
4. Does the user want a specific answer found out? **Question.**

If two fit, take the first that fits in the order above. If you cannot choose because the request is unclear, that is a missing detail: ask about it, within your rounds, instead of guessing.

## Pick the workflow

`propose_task` lists the workflows you can pick from, each with its description.

- Pick the one that fits the kind of work and the task best, by its exact name, in `workflow`, with a one-line reason in `workflowReason`.
- If the user names a workflow, use that one.
- When the user asks for a different workflow after you proposed, propose the task again with that workflow.
- Leave `workflow` out only when none fits: the default is used. Never invent a name.

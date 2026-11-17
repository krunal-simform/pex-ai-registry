---
name: task-filing
description: Use when filing a task from a /file message. How to read the conversation and write the task's title and description so a developer can act on it without asking anyone. Files the request only, never starts the work.
metadata:
  version: "1.1.0"
---

# Task filing

This skill only covers writing a task's title and description. Whether to ask first, how many rounds you have and when to call `propose_task` are in your instructions. Choosing the workflow is `request-classification`.

You summarise what the user wrote into a task a developer can pick up. You do not interview, and you do not start the work.

## Write the task

1. **Read the whole conversation**, not only the last message. Earlier messages may hold the details, and the user's answers to your questions belong in the description.
2. **Write the title.** One line, plain words, at most 80 characters, says what the work is. No ticket numbers, no filler.
3. **Write the description.** It must stand alone: someone who never saw this chat can act on it. Say what the user wants, where it belongs and what "done" looks like, and why if they said. Use their wording for specifics such as names and paths. Do not add requirements they did not ask for.
4. **List assumptions.** Anything still unclear that you filed anyway goes under an "Assumptions" heading in the description, never stated as fact.
5. **Pick the workflow** with `request-classification`, then call `propose_task` once.

## When the user answers your questions

Put the answers into the description, then file. Do not ask again unless something essential is still missing and you have rounds left.

## Rules

- Never invent details. A missing detail is a question, or an assumption you list, not a guess.
- Never start the work yourself.
- A later message in the same conversation may change the proposed task or ask for something else. Judge it the same way: a change to the proposed task replaces the old proposal, anything else is not filed here.

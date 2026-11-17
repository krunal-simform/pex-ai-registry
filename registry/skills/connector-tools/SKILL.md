---
name: connector-tools
description: Rules for the project's connector tools, the ones named <connector>__<tool> that reach connected services such as an issue tracker or documents. Applies to every step of a task.
metadata:
  version: "1.0.0"
---

# Connector tools

Tools named `<connector>__<tool>` reach the project's connected services, such as an issue tracker or documents. The project's tool policy decides which of them you get and enforces it on every call. A project may have none.

- Use them to read what the task refers to, and say what you read.
- The planner and the reviewer change nothing through them.
- The implementer changes something through them only when the task asks for it, and reports each change in its notes.
- When one fails, carry on without it and say so.

---
name: city-lark-diagram
description: Generate a diagram from a description or existing Mermaid source and upload it to a Lark whiteboard.
---

# city-lark-diagram

Generate a diagram from a description or existing Mermaid source and upload it to a Lark whiteboard.

## When to Use

Trigger this skill when the user says something like:

**English:**
- "draw a diagram"
- "generate a flowchart / sequence diagram / architecture diagram"
- "turn this into a diagram"
- "create a class diagram / mind map"
- "visualize this flow"

**中文：**
- "帮我画个图"
- "生成流程图 / 时序图 / 架构图"
- "把这个画成图"
- "画个类图 / 思维导图"
- "这个流程用图表示一下"

This skill can also be called internally by other skills (e.g. `city-lark-solution-doc`) when they need to embed a diagram.

## Dependencies

```bash
npm install -g @larksuite/whiteboard-cli@^0.1.0
lark-cli auth login --domain wiki,docs,drive
```

Required skills:

- `lark-whiteboard` — upload `.mmd` to a Lark whiteboard
- `lark-doc` — insert a whiteboard into an existing document (optional)

## Workflow

### Step 1 — Determine diagram content

- If the user has already provided `.mmd` source, use it directly and skip to Step 2
- Otherwise generate a `.mmd` file in Mermaid syntax based on the conversation

Diagram type routing:

| Type | Syntax |
|------|--------|
| Flowchart, sequence, class, mind map, pie chart | Mermaid syntax |
| Architecture, comparison, funnel, etc. | DSL path (handled by `lark-whiteboard` skill) |

> Labels, annotations, and participant names follow the conversation language by default (Chinese conversation → Chinese labels), unless the user specifies otherwise.

Save the file as `diagram-temp.mmd`.

### Step 2 — Confirm operation type and target whiteboard

Determine whether this is a **new whiteboard** or an **update to an existing whiteboard**:

#### New whiteboard

Ask the user where the diagram should go:

> "Where should the diagram be inserted? Please provide a document link, or leave it as a standalone whiteboard."

- **Insert into a document**: after uploading in Step 3, capture the token from `data.board_tokens[0]` in the creation response for use in Step 4
- **Standalone whiteboard**: create the whiteboard directly and return the link when done

#### Update existing whiteboard

Ask the user to provide the whiteboard token directly:

> "Please provide the target whiteboard token (found in the whiteboard URL, format similar to `wikcnXXXXXX`)."

- **Token provided**: proceed to Step 3 using that token
- **Token not provided**: warn the user before proceeding:

  > ⚠️ No token provided. The system will need to fetch the full document content via `docs +fetch` to locate the target whiteboard. This will consume more tokens and increase wait time. It is recommended to copy the token from the whiteboard URL and retry. Do you still want to proceed with auto-lookup?

  Only execute `docs +fetch` after the user explicitly confirms.

### Step 3 — Upload to Lark whiteboard

Use the `lark-whiteboard` skill to upload `diagram-temp.mmd` to the Lark whiteboard:

> Always run a dry-run before uploading. If the whiteboard is non-empty (update scenario), confirm with the user before overwriting. Follow the safety rules in the `lark-whiteboard` skill.

### Step 4 — Insert into document (optional)

Only execute this step when creating a **new whiteboard** with a target document specified:

- Use the `board_token` captured from the creation response in Step 2, and insert after the anchor via `lark-doc`'s `insert_after`
- The anchor `block_id` should be provided by the caller or user before this step; if not provided, locating it requires `docs +fetch`, which will consume more tokens and increase wait time — inform the user in advance

### Step 5 — Clean up and deliver

- Delete the `diagram-temp.mmd` temporary file
- Return the whiteboard link (and document link if an insertion was performed)

## Notes

- Diagrams are **always delivered as Lark whiteboards**, never as code blocks
- When called by another skill, the target document and anchor are provided by the caller — skip the interactive prompts in Step 2

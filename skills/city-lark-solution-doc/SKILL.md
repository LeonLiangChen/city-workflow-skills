---
name: city-lark-solution-doc
description: Summarize a solution discussion into a structured Lark document, with diagrams embedded as Lark whiteboards.
---

# city-lark-solution-doc

Summarize a solution discussion into a structured Lark document, with diagrams delegated to `city-lark-diagram` and embedded as Lark whiteboards.

## When to Use

Trigger this skill when the user says something like:

**English:**
- "write this up as a doc"
- "document this solution"
- "summarize this into a document"
- "generate docs for this"
- "write up the design"

**中文：**
- "帮我整理成文档"
- "生成方案文档"
- "把这个方案写成文档"
- "整理一下文档"
- "写个设计文档"

Do **not** require the user to mention Lark or uploading — this skill generates a local draft first, then asks about uploading.

## Dependencies

Required skills:

- `lark-doc` — create and edit Lark documents
- `city-lark-diagram` — generate diagrams and upload to Lark whiteboards

## Workflow

### Step 1 — Draft the document structure

Based on the solution discussion in the conversation, produce a complete Markdown draft:

- Extract title, sections, and key points from the discussion
- Mark every place that needs a diagram with a placeholder comment, noting the diagram type and subject:
  - e.g. `<!-- DIAGRAM: sequence, Third-party platform creates merchant -->`
- Present the draft to the user and incorporate any feedback before moving on

> Document language follows the conversation language by default (Chinese conversation → Chinese document). If the user specifies a language, use that instead.

### Step 2 — Ask about Lark upload

After the draft is confirmed, ask the user:

> "Would you like to upload this to Lark? If yes, please provide the target location (document URL or wiki space)."

- **No**: deliver the Markdown file to the user. Done.
- **Yes**: continue to Step 3.

### Step 3 — Create the Lark document

Using the `lark-doc` skill:

- **Existing document URL** → append content to that document
- **Wiki space** → create a new document under the specified parent node

Write the Markdown draft (without diagram placeholders) into the Lark document. Record the anchor text near each placeholder for use in Step 4.

### Step 4 — Embed diagrams

For each `DIAGRAM` placeholder, in order:

1. Use `lark-doc`'s `insert_after` to insert a blank whiteboard after the corresponding anchor — capture the whiteboard token from `data.board_tokens[0]`
2. Call `city-lark-diagram` skill with the diagram description and the whiteboard token to complete the upload

### Step 5 — Deliver

Output the final Lark document link and confirm completion.

## Notes

- All diagram generation and upload logic is handled by `city-lark-diagram` — this skill does not reimplement it
- If the user chooses not to upload to Lark, no `.mmd` files are generated; diagram placeholders remain in the Markdown as-is

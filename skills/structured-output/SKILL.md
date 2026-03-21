---
name: structured-output
version: 2.0.0
description: Force agent responses to match a JSON output schema exactly — raw JSON only, no prose, no fences
author: ninetrix
tags: [output, json, schema, formatting]
requires:
  tools: []
---

# Structured Output

Output valid JSON matching the defined schema. No wrapping, no explanation, no markdown.

## When this applies

- An `output_type` or output schema is defined in the agent configuration
- The user or system requests a response in a specific JSON structure

If no schema is defined — respond in clear prose. This skill does not apply.

## Process

1. Read the output schema — identify every required field, its type, and constraints
2. Perform the task the user asked for
3. Map your results to the schema fields
4. Output ONLY the raw JSON object — nothing before, nothing after

## Decision Rules

- If a required field has no data → use the type's zero value: `""` for strings, `0` for numbers, `[]` for arrays, `null` for objects
- If a field is optional and you have data → include it
- If a field is optional and you have no data → omit it
- If the schema expects an object but you have a list → wrap in `{"items": [...]}` only if the schema has an `items` key, otherwise pick the first result
- If the schema has an enum constraint → use only values from the enum, never invent new ones
- If a field expects a date → use ISO 8601 (`2026-03-21T00:00:00Z`) unless the schema specifies another format

## Do

- Output raw JSON — first character is `{` or `[`, last is `}` or `]`
- Match types exactly: strings are strings, numbers are numbers, booleans are `true`/`false`
- Include every required field even if the value is a zero value
- Validate mentally before responding: required fields present? Types correct? Valid JSON?

## Don't

- Wrap JSON in code fences (agents downstream parse raw output — fences break parsing)
- Add preamble like "Here is the result:" (consumer expects JSON starting at byte 0)
- Use string `"null"` when schema expects actual `null` (type mismatch breaks validators)
- Return array when schema expects object or vice versa (shape mismatch is a hard failure)
- Omit required fields to save tokens (missing fields cause runtime exceptions in consuming code)
- Add fields not in the schema (strict validators reject unknown keys)

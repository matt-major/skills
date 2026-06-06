---
name: code-explainer
description: Explain a method or function from a codebase with a concise summary and step-by-step walkthrough. Trigger this skill when the user
  asks to analyze a specific function, method, or entry point in source code and return an explanation with file and line citations, code snippets, and call-site context.
---

# Code Explainer

## Output Format

Use this markdown template exactly:

**{{function or method name}}**

{{single paragraph summary}}

**Steps**
**Step {{n}}: {{brief explanation}}**
File: {{relative file path}} | Line: {{line number, with dash end line number if multiple lines}}

```{{language}}
{{code snippet}}
```

{{explanation of what this code is doing}}

## How To Explain Code

- Start from the requested symbol, then identify its file, signature, and surrounding context.
- Write one paragraph that explains the function's purpose and overall behavior.
- Break the explanation into ordered steps that follow execution or logical blocks.
- For each step, cite the relative file path and line numbers, include a minimal relevant snippet, then explain it.
- When the code calls another function, method, API, database, or service, explain only the call-site purpose, inputs, and outputs or returned effect.
- Do not explain internals of external or downstream calls unless they are part of the requested code path.
- If the code has many branches, nested levels, or complex control flow, summarize those parts at a higher level instead of tracing every branch in depth.
- Prefer small snippets that prove the point.
- Avoid speculation; if behavior is not explicit, say so.

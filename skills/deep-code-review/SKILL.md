---
name: deep-code-review
description: >
  Performs a comprehensive, multi-faceted code review on a given commit, branch, or tag.
  It spawns parallel sub-agents to review for coding standards, requirement accuracy,
  and language best practices (performance, security). Trigger this skill when the user
  asks to "review code", "review a commit", "review this PR", or similar requests.
---

# Deep Code Review Skill

This skill performs a comprehensive code review by breaking the task into three distinct areas and running them in parallel using sub-agents. Finally, it aggregates the results into a single cohesive report.

## 1. Initial Setup and Context Gathering (Progress Checklist)

**CRITICAL:** As the top-level orchestrating agent, you MUST do all of the identification and data retrieval work up-front. Do not delegate the fetching of issues or the searching of standard files to the sub-agents, as this causes redundant work. 

Use this checklist to track progress:
- [ ] **Step 1: Identify the Target**. Ensure you have a commit SHA, branch name, or tag to review. 
   - *If none is provided by the user, explicitly ask them what they want to review before proceeding.*
- [ ] **Step 2: Retrieve the Diff**. Generate the diff for the target to be reviewed. Read it completely.
- [ ] **Step 3: Retrieve the Requirements**. 
   - Check the commit messages or branch names for issue references (e.g., ticket numbers, JIRA IDs, GitHub issues).
   - Fetch the issue details. If using an external tracker like JIRA, use the appropriate tool or MCP server to accurately retrieve the requirements.
   - If the source is a Product Requirements Document (PRD) file, read the file directly.
   - *Result: You must hold the raw text of the requirements in your context.*
- [ ] **Step 4: Retrieve the Standards**. Look for project-specific guidelines in the following order of precedence and read their contents:
   - Agent-specific guidance (`CLAUDE.md`, `AGENTS.md`)
   - `CONTRIBUTING.md` files
   - `CONTEXT.md` files
   - `docs/adr/` (Architecture Decision Records)
   - Coding style/linting configuration files (`.editorconfig`, `eslint.config.*`, `biome.json`, `prettier.config.*`, `tsconfig.json`)
   - *Result: You must hold the raw text of all relevant standards in your context.*

## 2. Parallel Sub-Agent Execution

Once all the raw context is gathered, spawn four parallel sub-agents (or independent asynchronous context windows) using your available tools for agent orchestration. 

**CRITICAL:** Pass each agent a specific, focused prompt containing the *fully resolved data* (e.g., the actual text of the requirements, the actual text of the standards, and the actual diff). Do not just pass file paths or IDs. They must be able to start reviewing immediately without making any further tool calls to fetch data.

### Sub-Agent 1: Standards Reviewer
- **Role**: Standards Reviewer
- **Task**: Review the diff and ensure it conforms to the repository's coding standards.
- **Context to pass**: The full raw diff, and the exact text of the identified standards sources.

### Sub-Agent 2: Requirements Reviewer
- **Role**: Requirements Reviewer
- **Task**: Review the changes and check if they accurately implement the original issue, ticket, requirements, or spec.
- **Context to pass**: The full raw diff, and the exact text of the retrieved requirements/issue details.

### Sub-Agent 3: Best Practices Reviewer
- **Role**: Best Practices Reviewer
- **Task**: Review the code in the diff for best practices of the given programming language, ensuring performant code and maintainability.
- **Context to pass**: The full raw diff, and a brief about the languages and frameworks involved.

### Sub-Agent 4: Security Reviewer
- **Role**: Security Reviewer
- **Task**: Review the code specifically for security concerns, including secure coding practices, potential logging of personal or sensitive data (PII), and potential OWASP Top 10 vulnerabilities. Note that findings here should generally be treated as warnings rather than blockers unless they represent a clear and immediate security vulnerability.
- **Context to pass**: The full raw diff, and a brief about the languages and frameworks involved.

## 3. Aggregation and Reporting

Wait for all four sub-agents to complete their tasks and send their findings back. Once all four have reported, aggregate their reviews into a final response to the user.

Format the final report clearly using the following structure:

```markdown
## Standards
[Aggregate findings from the Standards Reviewer]

## Requirements
[Aggregate findings from the Requirements Reviewer]

## Best Practices
[Aggregate findings from the Best Practices Reviewer]

## Security
[Aggregate findings from the Security Reviewer. Highlight any critical blockers, otherwise present as warnings/advisories.]

[One line summary of the overall review status.]
```

**Crucial Note**: Do not start the review process until you have clearly identified what is being reviewed (commit, branch, etc.).

## 4. Gotchas

- **Truncated Diffs**: If a diff is too large, your tools might truncate it. If you suspect the diff is incomplete, instruct the sub-agents to review it in chunks or fetch the files directly.
- **Unreachable Requirements**: JIRA or private GitHub issues may be inaccessible without the correct MCP server or auth headers. If you cannot fetch the requirements automatically, ask the user to paste them.
- **Sub-agent Hallucinations**: Sub-agents without explicit contexts will try to guess requirements. Never let them guess; always supply the raw text.

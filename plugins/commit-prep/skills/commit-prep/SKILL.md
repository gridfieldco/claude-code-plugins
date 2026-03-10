---
description: Review staged git changes and draft a conventional commit message
disable-model-invocation: true
---

Run `git diff --cached` to see the staged changes. Then:

1. Summarize what changed in 1-2 sentences
2. Flag anything risky: potential bugs, security issues, missing tests, accidental file inclusions (.env, credentials, large binaries)
3. Draft a conventional commit message (type: subject) that focuses on the "why" not the "what"
4. If nothing is staged, say so and suggest `git add` for the relevant files

Be concise. Output the commit message in a code block so it's easy to copy.

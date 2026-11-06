## Exercise

The exercise branch contains a project repo for you to set up agent guidance from scratch.

**Part 1 — Install a skill**

1. Browse the [Anthropic plugins](https://github.com/anthropics/claude-plugins-official) or [skills.sh](https://skills.sh/) and find a plugin containing a skill relevant to code review or code quality.
2. You can do the browsing inside Claude by running `/plugin`. Search for "code review". Install it and follow instructions.
3. **Optional:** Ask Claude to create a custom review skill that enforces well-established TypeScript guidelines for this codebase.
4. Start a new session and ask Claude to review the codebase using the installed skill, save results to docs/ReviewFindings.md.
5. Ask Claude to create a plan for improving the codebase in docs/ImprovementPlan.md.
6. Ask Claude to make a simple change to the codebase. Verify it uses the skill.

**Part 2 — Create CLAUDE.md**

1. Read through the project code and identify: the tech stack, two or three architecture rules Claude should follow, and two things Claude must not change.
2. Create a `CLAUDE.md` file in the project root with this information. Keep it under one page.
3. **Optional:** Install a skill that can create the CLAUDE.md file.
4. Start a new session and ask Claude to make a simple change. Verify it follows the conventions you wrote down.

### Automated Checks

* [ ] CLAUDE.md exists
* [ ] ImprovementPlan.md exists
* [ ] ReviewFindings.md exists

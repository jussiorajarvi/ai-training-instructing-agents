### Instructing Agents

When you work with Claude Code on a real codebase you can guide it in two ways: **skills** (reusable slash-command prompts) and **CLAUDE.md** (project-level standing instructions). This exercise practises both.

The `app/` directory contains a small TypeScript project that was written with intentional code smells. You'll use it as the target for your agent sessions.

---

## Exercise

<!-- github-only -->
### Setup — clone your copy

Clone your copy of this repository (the one created from the template) to your computer and open it in a terminal:

```
git clone <URL of your copy>
cd <name of your repo>
```

You can copy the URL from the green **Code** button on your repository page.
<!-- /github-only -->
<!-- mcp-only
### Setup

Claude has already written the starter files into this folder. Open a terminal here.
-->

### Part 1 — Install and use a skill

1. In Claude Code, run `/plugin` and browse the available plugins (or see the [Anthropic plugins](https://github.com/anthropics/claude-plugins-official)) to find a plugin containing a skill relevant to code review or code quality.
2. Install the plugin from the `/plugin` menu, then run `/reload-plugins` to load it into the current session.
3. *(Optional)* Ask Claude to create a custom review skill for TypeScript that uses common guidelines like Effective TypeScript, TypeScript Official Guidelines, Google TypeScript Style Guide, Uncle Bob.
4. Start a new Claude Code session and ask Claude to review the `app/` codebase using the installed skill. Save the findings to **`docs/ReviewFindings.md`**.
5. Ask Claude to create a plan for improving the codebase. Save it to **`docs/ImprovementPlan.md`**.
6. Ask Claude to make one small improvement to the code. Verify it uses the skill or follows the review findings.

### Part 2 — Create CLAUDE.md

1. Ask Claude to analyze the project code in `app/src/` and identify:
   - the tech stack
   - two or three architecture rules Claude should follow when modifying the code
   - two things Claude must **not** change (e.g. the public API, the test structure)
2. Then review the code in `app/src/` yourself to verify Claude's analysis.
3. Create a **`CLAUDE.md`** file in the repository root with this information. Keep it under one page.
4. *(Optional)* Install a skill that helps generate CLAUDE.md files.
5. Start a new session and ask Claude to make a small change to the app. Verify it follows the conventions you wrote down.

<!-- github-only -->
### Automated Checks

The CI pipeline will verify:

1. `docs/ReviewFindings.md` exists and is not empty.
2. `docs/ImprovementPlan.md` exists and is not empty.
3. `CLAUDE.md` exists, has at least 5 non-empty lines, and contains at least one constraint (a word such as `not`, `avoid`, or `never`).

Commit and push after finishing both parts — the check runs on every push to these files.

<details>
<summary>Having trouble? 🤷</summary><br/>

 * Plugins are installed with the `/plugin` command in Claude Code. After installing, run `/reload-plugins` so the new skill is available in your current session.
 * If Claude doesn't seem to follow your CLAUDE.md, make sure the file is in the repository root (next to the `app/` folder) and restart your session.
 * If nothing happens for a while after pushing, or if the check fails, check the [Actions](../actions) tab to see what went wrong.
 * If you are still having trouble, ask for help in the course support channel.

</details>

---

<img alt="Amin 2.0" src="../images/amin2_smile.png" align="right" height="125px" />

Please, follow the steps above.
I'll watch your progress in the background to provide feedback. 🧐
<!-- /github-only -->
<!-- mcp-only
When you are done, ask me to check your solution.
-->

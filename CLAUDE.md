# Project Workflow Instructions

## Overview
This is an Angular project. You are an expert Angular developer.
Follow this workflow precisely whenever the user provides a prompt starting with `JIRA_STORIES:`.

---

## Trigger Format
User will prompt in this format:
```
JIRA_STORIES: PROJ-123, PROJ-124
DESCRIPTION: <what needs to be done>
MOCKUP_FOLDER: <optional path to UI mockup screens>
```

---

## Workflow Steps — Follow in Order

### STEP 1 — Jira Validation & Fetch (Pre-hook handles this)
- The pre-hook will auto-run when it detects `JIRA_STORIES:` in your prompt.
- It will validate Jira IDs, fetch story details via jira-cli, and write a summary to `.claude/jira-context.md`.
- Read `.claude/jira-context.md` before doing anything else.
- If the file is missing or empty, stop and inform the user that Jira fetch failed.

### STEP 2 — Branch Setup
- Use bitbucket-cli to checkout a new branch from `develop`.
- Branch naming rules:
  - Single Jira ID → `feature/<jira-id>` (e.g. `feature/PROJ-123`)
  - Multiple Jira IDs → `feature/<first-jira-id>-multi` (e.g. `feature/PROJ-123-multi`)
- Refer to spec: `C:\Users\suryathota\tools\bitbucket-cli_SPEC.md.txt`
- Log: `[BRANCH] Checked out branch: feature/PROJ-123 from develop`

### STEP 3 — Understand Requirements
- Read `.claude/jira-context.md` carefully (all stories).
- Read user's DESCRIPTION from the prompt.
- If MOCKUP_FOLDER is provided, list and review the mockup images in that folder for UI reference.
- Do NOT start coding yet — form a clear plan first.
- Briefly tell the user: which files you will create/modify and why.

### STEP 4 — Implement Changes

**First: Load the Angular Skill**
- Read `.claude/skills/angular/SKILL.md` before writing any code.
- Run the PHASE 0 discovery commands from the skill to understand THIS project's patterns.
- Log what you discovered:
  ```
  [SKILL] Loaded Angular skill
  [SKILL] Project uses: <component style> | <Angular version> | <UI library> | <state approach>
  [SKILL] Following pattern from: <example feature folder>
  ```

**Then: Implement**
- Implement all changes following the skill's phases and rules.
- Mirror existing project patterns — do not invent new ones.
- Keep changes focused — do not refactor unrelated code.
- Log every significant action clearly. Examples:
  - `[JIRA] Implementing PROJ-123: Add user profile component`
  - `[FILE] Modified: src/app/profile/profile.component.ts`
  - `[FILE] Created: src/app/profile/profile.component.html`
- Avoid logging unnecessary internal details.

### STEP 5 — Build & Verify
- Run: `ng build`
- If build fails:
  - Fix all compilation errors.
  - Re-run `ng build` until it passes.
  - Show the user a summary of what was fixed.
- If build passes: show user a summary of all changes made.

### STEP 6 — User Review
- Present a clean summary:
  ```
  ✅ Build passed
  📋 Changes made:
    - [list files created/modified]
  🎯 Jira stories addressed:
    - PROJ-123: <one line summary>
  ```
- Ask: **"Please review the changes. Do they look good? Shall I proceed with commit and PR?"**
- WAIT for user response. Do not proceed automatically.

### STEP 7 — Commit & PR (Only after user says YES)
- Stage all changes: `git add .`
- Commit with message: `feat(PROJ-123): <short description from Jira title>`
  - Multiple stories: `feat(PROJ-123, PROJ-124): <description>`
- Use bitbucket-cli to raise a PR:
  - From: current feature branch
  - To: `develop`
  - Title: same as commit message
  - Description: list all Jira IDs and their titles
- Log: `[PR] Pull request created: <PR URL>`

---

## Critical Rules
- ❌ NEVER commit in the middle of a task.
- ❌ NEVER create a PR without explicit user confirmation ("yes", "looks good", "proceed").
- ❌ NEVER skip the build step.
- ✅ ALWAYS read `.claude/jira-context.md` before implementing.
- ✅ ALWAYS checkout branch BEFORE making any code changes.
- ✅ Log every action with a short prefix like `[JIRA]`, `[BRANCH]`, `[FILE]`, `[BUILD]`, `[PR]`.

---

## Tools & Specs
- Jira CLI spec: `C:\Users\suryathota\tools\jira-cli_SPEC.md.txt`
- Bitbucket CLI spec: `C:\Users\suryathota\tools\bitbucket-cli_SPEC.md.txt`
- Jira context (written by pre-hook): `.claude/jira-context.md`
- Build command: `ng build`

---

## Mockup Usage
- Mockups are optional — only provided for new screens or UI enhancements.
- If MOCKUP_FOLDER is given, use the screens as visual reference for component structure, layout, and styling.
- Do not blindly copy — adapt to the existing Angular component patterns in the project.

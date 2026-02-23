---
description: Implement ONE task from an OpenSpec change (Experimental)
---

Implement **ONE task** from an OpenSpec change - with progress tracking, testing, and git commit.

**Input**: Optionally specify a change name (e.g., `/opsx:once add-auth`). If omitted, check if it can be inferred from conversation context. If vague or ambiguous you MUST prompt for available changes.

## Session Startup Protocol

### 1. **Select the change**

   If a name is provided, use it. Otherwise:
   - Infer from conversation context if the user mentioned a change
   - Auto-select if only one active change exists
   - If ambiguous, run `openspec list --json` to get available changes and use the **AskUserQuestion tool** to let the user select

   Always announce: "Using change: <name>" and how to override (e.g., `/opsx:once <other>`).

### 2. **Check status to understand the schema**
   ```bash
   openspec status --change "<name>" --json
   ```
   Parse the JSON to understand:
   - `schemaName`: The workflow being used (e.g., "spec-driven")
   - Which artifact contains the tasks (typically "tasks" for spec-driven, check status for others)

### 3. **Get apply instructions**

   ```bash
   openspec instructions apply --change "<name>" --json
   ```

   This returns:
   - Context file paths (varies by schema)
   - Progress (total, complete, remaining)
   - Task list with status
   - Dynamic instruction based on current state

   **Handle states:**
   - If `state: "blocked"` (missing artifacts): show message, suggest using `/opsx:continue`
   - If `state: "all_done"`: congratulate, suggest archive
   - Otherwise: proceed to implementation

### 4. **Read context files and progress**

   Read the files listed in `contextFiles` from the apply instructions output.
   - **spec-driven**: proposal, specs, design, tasks
   - Other schemas: follow the contextFiles from CLI output

   **Also read progress file if exists:**
   - Location: `<change-dir>/progress.md` (same level as tasks.md)
   - If not exists, will be created after first task completion

### 5. **Orient environment**
   ```bash
   pwd                    # Confirm working directory
   git log --oneline -5   # Review recent commits
   ```

### 6. **Show current progress**

   Display:
   - Schema being used
   - Progress: "N/M tasks complete"
   - Next pending task to work on
   - Dynamic instruction from CLI

## Single Task Implementation

**CRITICAL: Work on ONLY ONE task per session/invocation.**

### Implementation Steps

1. **Select the next pending task**
   - Identify the first unchecked task in tasks.md
   - Announce: "Working on task N/M: <task description>"

2. **Implement the task**
   - Make the code changes required
   - Keep changes minimal and focused
   - Follow the design and spec guidance

3. **Test the implementation**
   - Write/update tests for the implemented functionality
   - Run existing tests to ensure no regressions
   - Test end-to-end as a user would
   - Verify the complete flow works

4. **Handle issues if any:**
   - Task unclear → ask for clarification
   - Design issue revealed → suggest updating artifacts
   - Error or blocker → report and wait for guidance

## Session End Protocol

Before ending each session, you MUST:

### 1. **Mark task complete**
   Update the tasks file: `- [ ]` → `- [x]` for the completed task

### 2. **Update progress file**
   Update `<change-dir>/progress.md`:
   ```markdown
   ## Session: [Current Date/Time]
   ### Completed
   - Task N: <task description>

   ### Tests
   - <tests written/updated/run>

   ### Notes
   - <any relevant notes or decisions>

   ### Next
   - Task N+1: <next task description>
   ```

### 3. **Git commit**
   ```bash
   git add .
   git commit -m "feat(<change-name>): <task description>"
   ```
   Use conventional commit messages:
   - `feat:` for new features
   - `fix:` for bug fixes
   - `test:` for test additions
   - `refactor:` for code improvements

## Output Format

**Output During Implementation**

```
## Implementing: <change-name> (schema: <schema-name>)

Working on task N/M: <task description>
[...implementation happening...]
Running tests...
✓ Task complete

Updating progress.md...
Git commit: feat(<change-name>): <task description>
```

**Output On Completion (Single Task)**

```
## Task Completed

**Change:** <change-name>
**Schema:** <schema-name>
**Progress:** N/M tasks complete

### Completed This Session
- [x] Task N: <task description>

### Tests
- <test results summary>

### Next
- Task N+1: <next task description>

Run `/opsx:once` again to continue with the next task.
```

**Output When All Tasks Done**

```
## All Tasks Complete!

**Change:** <change-name>
**Progress:** M/M tasks complete ✓

Ready to archive this change. Run `/opsx:archive`.
```

**Output On Pause (Issue Encountered)**

```
## Implementation Paused

**Change:** <change-name>
**Progress:** N/M tasks complete

### Issue Encountered
<description of the issue>

**Options:**
1. <option 1>
2. <option 2>
3. Other approach

What would you like to do?
```

## Guardrails

- **ONE task per session** - do not implement multiple tasks
- Always read context files and progress.md before starting
- Always read progress.md to understand previous work
- If task is ambiguous, pause and ask before implementing
- Test thoroughly before marking task complete
- Update task checkbox immediately after completing each task
- Always update progress.md with session summary
- Always commit changes to git
- Pause on errors, blockers, or unclear requirements - don't guess
- Use contextFiles from CLI output, don't assume specific file names

## Fluid Workflow Integration

This skill supports the "actions on a change" model:

- **Can be invoked anytime**: Before all artifacts are done (if tasks exist), after partial implementation, interleaved with other actions
- **Allows artifact updates**: If implementation reveals design issues, suggest updating artifacts - not phase-locked, work fluidly
- **Incremental progress**: Each invocation completes exactly one task, then stops

## Example Session Flow

```
[Agent] Starting session for change: keyword-web-crawler...
[Runs openspec status, instructions apply]
[Reads tasks.md, progress.md]

[Agent] ## Implementing: keyword-web-crawler (schema: spec-driven)
Progress: 2/7 tasks complete

Working on task 3/7: Implement keyword management API
[Implements the API]
[Writes tests]
[Runs tests]

[Agent] ✓ Task complete! Updating files...
[Updates tasks.md: checkbox for task 3]
[Updates progress.md with session summary]
[Git commit: "feat(keyword-web-crawler): implement keyword management API"]

[Agent] ## Task Completed
Progress: 3/7 tasks complete
Next: Task 4/7 - Implement keyword search functionality

Run `/opsx:apply` again to continue.
```

## Important Reminders

1. **One Task Only**: Complete one task well, then stop
2. **Verification First**: Test before declaring complete
3. **Document Progress**: progress.md is the next session's lifeline
4. **Commit Always**: Leave a clean git history
5. **Quality Over Speed**: Better to complete one task well than rush through multiple

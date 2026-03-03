---
name: jj-commit
description: >
  Manage version control with Jujutsu (jj) instead of Git. Use this skill whenever the user
  wants to commit changes, view history, or push to a remote — and the project uses Jujutsu VCS
  (look for a .jj/ directory). Trigger on any commit, log, push, or version control request
  in a jj repository. Also trigger when the user says /commit in a jj repo.
---

# Jujutsu (jj) Version Control

This skill handles version control operations using Jujutsu (jj), a modern VCS that works
differently from Git in important ways. The key difference: jj has no staging area. Every
file change is automatically part of the current "change" (jj's term for a commit-in-progress).

## Detecting a jj Repository

Before doing any VCS work, check which VCS the project uses:

```bash
# If .jj/ exists, use this skill. If .git/ exists (without .jj/), use git instead.
ls -d .jj/ 2>/dev/null
```

jj repositories often also have a `.git/` directory (jj can use git as a backend), so the
presence of `.jj/` is the deciding factor.

## Commit Workflow

When the user wants to commit their changes, follow these steps:

### 1. Understand the Current State

Run these commands to understand what has changed:

```bash
jj status          # What files have been modified/added/deleted
jj diff            # The actual content changes
jj log -r @        # The current change's metadata
```

### 2. Review Recent History

Look at recent commits to understand the project's commit message style:

```bash
jj log --limit 10  # Recent history with descriptions
```

### 3. Write the Commit Message

Write a clear, descriptive commit message based on the actual changes. The message should:

- Summarize the "why" rather than just the "what"
- Be concise (1-2 sentences for the first line)
- Add detail in a second paragraph if the change is complex
- End with: `Co-Authored-By: Claude <noreply@anthropic.com>`

### 4. Apply the Message

There are two operations — choose the right one based on what the user needs:

**`jj describe`** — Sets the message on the current change without creating a new one.
Use this when the user wants to keep working on the same change:

```bash
jj describe -m "$(cat <<'EOF'
Your commit message here.

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

**`jj commit`** — Sets the message AND creates a new empty change on top.
Use this when the user is done with this change and wants to move on. This is the
default when the user says "commit":

```bash
jj commit -m "$(cat <<'EOF'
Your commit message here.

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

When in doubt, use `jj commit` — it's the more natural equivalent of a git commit.

### 5. Verify

After committing, show the result:

```bash
jj log --limit 5
```

## Reading the Log

When the user asks about history, use `jj log`:

```bash
jj log                    # Default view (mutable revisions + context)
jj log --limit 20         # Last 20 changes
jj log -r "all()"         # Full history
jj log -r @               # Just the current change
jj show <revision>        # Full details of a specific revision
```

Useful revsets for filtering:

```bash
jj log -r "bookmarks()"         # Changes with bookmarks
jj log -r "mine()"              # Your own changes
jj log -r "description(word)"   # Search descriptions
```

## Pushing

When the user wants to push, use `jj git push`. Jujutsu pushes through its git backend.

### Push with Bookmarks

Bookmarks in jj are similar to git branches. To push, a change typically needs a bookmark:

```bash
# Check existing bookmarks
jj bookmark list

# Create or move a bookmark to the current change
jj bookmark set <bookmark-name> -r @

# Push a specific bookmark
jj git push --bookmark <bookmark-name>

# Push all bookmarks that have changed
jj git push
```

### Push a Single Change

To push a specific change without manually managing bookmarks, jj can auto-create
a bookmark from the change ID:

```bash
jj git push --change <revision>   # e.g. jj git push --change @
```

This creates a bookmark like `push-<changeId>` automatically.

### Before Pushing

Always confirm with the user before pushing, because:
- Pushing affects the remote repository and is visible to others
- The user should verify the right changes are being pushed
- Show `jj log` output so they can see what will be pushed

```bash
# Show what would be pushed
jj log -r "remote_bookmarks()..@"
```

## Important jj Concepts

- **Change vs Commit**: A "change" is the working unit in jj. It becomes immutable once
  finalized. Each change has a unique change ID (short hex) and a commit ID.
- **@ symbol**: Always refers to the current working-copy change.
- **No staging**: All file modifications are automatically part of @. There is no
  `git add` equivalent.
- **Bookmarks**: jj's equivalent of git branches. Required for pushing.
- **Revsets**: jj's query language for selecting revisions (similar to git's revision syntax
  but more powerful).

## Safety

- Never force-push without explicit user confirmation
- Never abandon changes without asking
- Always show the log after operations so the user can verify
- When in doubt about describe vs commit, ask the user

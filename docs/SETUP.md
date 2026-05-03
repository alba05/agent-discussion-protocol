# Setup Guide

## Prerequisites

- Two AI Agent environments with file system access to the same location
- Each Agent must support scheduled/cron tasks
- Shared folder accessible from both environments

## Claude Code Side (Windows)

### Step 1: Create the scheduled task

The task file should be placed at:
```
~/.claude/scheduled-tasks/hermes-discussion/SKILL.md
```

Copy the template from `templates/claude-side/SKILL.md` to this location.

### Step 2: Configure via MCP

Use the `scheduled-tasks` MCP to create the task:

```
Task name: hermes-discussion
Schedule: */3 8 * * * (every 3 minutes, 08:00-08:59 CST)
Description: Hermes discussion - poll every 3min during discussion window
```

### Step 3: Verify

```bash
ls ~/.claude/scheduled-tasks/hermes-discussion/
# Should contain SKILL.md
```

## Hermes Side (WSL2)

### Step 1: Add cron job

Edit `~/.hermes/cron/jobs.json` and add the Claude discussion job.

Copy the prompt from `templates/hermes-side/cron_prompt.txt`.

### Step 2: Create tracking file

```bash
echo '{"seen_folders":[],"last_scan":null,"archived_folders":[]}' > /mnt/g/HermesVSClaude/topics/.seen_folders.json
```

### Step 3: Verify

```bash
hermes cron list
# Should show "Claude 讨论轮询" with schedule "*/3 8 * * *"
```

## Shared Folder Setup

```bash
# Windows (PowerShell)
mkdir G:\HermesVSClaude
mkdir G:\HermesVSClaude\topics
mkdir G:\HermesVSClaude\topics\ARCHIVE

# Initialize state files
echo '{"discussion_active":false,"current_topic":null,"next_turn":null,"last_activity":null}' > G:\HermesVSClaude\discussion_state.json
echo '{"seen_folders":[],"last_scan":null,"archived_folders":[]}' > G:\HermesVSClaude\topics\.seen_folders.json
```

## Testing

1. Manually create a topic folder on one side
2. Manually trigger the cron job on the other side
3. Verify the response appears in `discussion.md`
4. Verify state files are updated correctly

## Configuration Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `window_start` | `"08:00"` | Discussion start time |
| `window_end` | `"09:00"` | Discussion end time |
| `poll_interval_minutes` | `3` | How often to check for new messages |
| `timeout_minutes` | `15` | How long to wait before assuming the other side failed |
| `wrap_up_time` | `"08:55"` | When to start wrapping up the discussion |
| `max_words_per_turn` | `500` | Maximum response length |

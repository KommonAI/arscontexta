---
# GENERATION TEMPLATE — Do not edit directly
# This template is transformed by the derivation engine during /setup.
# All {vocabulary.*} markers resolve from the preset's vocabulary.yaml.
# All {config.*} markers resolve from the preset's preset.yaml.
# All {DOMAIN:*} markers resolve from conversation-derived domain context.
# All {if ...}{endif} blocks are conditionally included based on config.
source_skill: archive-batch
min_processing_depth: quick
platform: shared
---

---
name: archive-batch
description: Archive a completed processing batch — verify completeness, move task files, generate summary, clean queue entries. Triggers on "/archive-batch", "/archive-batch [batch-id]".
version: "1.0"
generated_from: "arscontexta-{plugin_version}"
user-invocable: true
context: fork
model: sonnet
allowed-tools: Read, Write, Edit, Grep, Glob, Bash
argument-hint: "[batch-id] [--force] [--dry-run] — batch identifier to archive"
---

## EXECUTE NOW

**Target: $ARGUMENTS**

Parse immediately:
- If target contains a batch id: archive that specific batch
- If target contains `--force`: skip completeness check (for broken batches)
- If target contains `--dry-run`: show what would happen without making changes
- If target is empty: scan the queue for batches where all tasks are `status: "done"` and present them as candidates

**Execute these steps:**

### Step 1: Read Configuration

Read these files to configure domain-specific behavior:

1. **`{config.ops_dir}/derivation-manifest.md`** — vocabulary mapping
   - Use `vocabulary.note` / `vocabulary.note_plural` for note type references
   - Use `vocabulary.notes` for the notes folder name

2. **`{config.ops_dir}/config.yaml`** — pipeline settings

If these files don't exist, use universal defaults.

### Step 2: Locate Queue

Find the queue file. Check in order:
1. `{config.ops_dir}/queue.yaml`
2. `{config.ops_dir}/queue/queue.yaml`
3. `{config.ops_dir}/queue/queue.json`

If no queue exists, report: "No queue found. Nothing to archive."

### Step 3: Identify Batch Tasks

Find all tasks matching the batch id. A batch includes:
- The **extract task** where `id` matches the batch id and `type: "extract"`
- All **claim/enrichment tasks** where `batch` matches the batch id

If no tasks match, report: "No tasks found for batch '{batch_id}'."

### Step 4: Verify Completeness

Unless `--force` is set, verify ALL tasks in the batch have `status: "done"`.

**If incomplete tasks exist:**
- List each incomplete task: id, type, current_phase, completed_phases
- Report: "Batch '{batch_id}' has {N} incomplete tasks. Use `--force` to archive anyway, or run `/ralph --batch {batch_id}` to complete processing."
- Stop execution

**If all done:** Proceed to Step 5.

### Step 5: Collect Batch Summary Data

Before moving files, gather summary information by reading each task file:

- **Source file:** from the extract task's `source` field
- **{vocabulary.note_plural} created:** collect titles from claim tasks (read each task file's Create section for the created {vocabulary.note} path/title)
- **Enrichments:** count tasks with `type: "enrichment"`
- **Learnings:** collect any friction, surprises, methodology insights from task file sections
- **Total tasks:** count of all tasks in the batch

### Step 6: Move Task Files

```bash
# Determine archive directory
DATE=$(date +%Y-%m-%d)
ARCHIVE_DIR="{config.ops_dir}/queue/archive/${DATE}-{batch_id}"

# Create archive directory (handle collision if re-running)
mkdir -p "$ARCHIVE_DIR"
```

Move all task files for the batch from `{config.ops_dir}/queue/` to the archive directory:
- The extract task file: `{batch_id}.md`
- All claim/enrichment task files: `{batch_id}-NNN.md`

Use `mv` for each file. If a file is missing, log a warning but continue (tolerance for partial state).

**If `--dry-run`:** Print which files would be moved and where, but do not move them.

### Step 7: Update Queue

Remove all archived batch entries from the queue file. This includes:
- The extract task entry
- All claim/enrichment task entries for the batch

Write the updated queue back to disk.

**If `--dry-run`:** Print which entries would be removed, but do not modify the queue.

**Ordering matters:** Files are moved (Step 6) BEFORE queue is updated (Step 7). If the process fails between steps, task files are in the archive but queue still has entries — this is safe to re-run. The reverse (queue cleaned but files not moved) would lose track of the files.

### Step 8: Generate Batch Summary

Write `{batch_id}-summary.md` to the archive directory:

```markdown
---
batch: {batch_id}
source: {source file name}
archived: {ISO timestamp}
---

# Batch Summary: {batch_id}

## Source
- **File:** {source file name}
- **Original location:** {source path}

## Extraction
- {vocabulary.note_plural} extracted: {N}
- Enrichments identified: {M}

## Created {vocabulary.note_plural}
{for each claim task:}
- [[{vocabulary.note} title]]
{end for}

## Enrichments Applied
{for each enrichment task:}
- [[target {vocabulary.note}]] — {enrichment description}
{end for}

## Learnings
{collected learnings from task files, or "None captured"}
```

**If `--dry-run`:** Print the summary that would be generated, but do not write it.

### Step 9: Report

```
--=={ archive-batch }==--

Batch: {batch_id}
Status: archived

Files moved: {N} task files -> {archive_dir}/
Queue entries removed: {M}
Summary: {archive_dir}/{batch_id}-summary.md

{vocabulary.note_plural} in this batch:
{list of note titles}
--=={ /archive-batch }==--
```

**START NOW.**

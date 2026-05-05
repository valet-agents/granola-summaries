# Granola Heartbeat — Recap Newly-Finished Meetings

The heartbeat channel fires once a day. There is no payload
to parse — your job is to detect any Granola meeting that finished
since the last fire, distill it, and post a recap to Slack.

## What this heartbeat does

1. Reads MEMORY.md to learn the last meeting it has seen.
2. Lists recent meetings from `granola-mcp` and finds any that
   finished after the last-seen cursor.
3. For each new finished meeting, fetches its summary, distills
   the moments that mattered (decisions / blockers / next step),
   and posts a tight recap to Slack.
4. Updates MEMORY.md with the newest meeting in the batch so the
   same meeting is never recapped twice.

## MEMORY.md state shape

The agent maintains a single block in MEMORY.md tracking the
last-seen meeting:

```
## granola-cursor
last_meeting_id: <granola meeting id>
last_finished_at: <ISO 8601 timestamp>
```

- On the **first heartbeat after deploy**, MEMORY.md will not
  have this block. Seed it with the most-recent existing finished
  meeting and **stop without posting**. The agent does not
  back-fill recaps for meetings that wrapped before deploy.
- On every subsequent fire, only meetings whose `finished_at` is
  strictly greater than `last_finished_at` are eligible.
- Cap the batch at the 5 oldest new meetings. If more piled up
  during downtime, the next heartbeat picks them up.

## Steps

1. Read MEMORY.md and parse the `granola-cursor` block. If
   missing, this is the seed run — pick the current most-recent
   finished meeting, write the cursor, and stop.
2. Call `granola-mcp` to list recent meetings. Filter to finished
   meetings whose `finished_at > last_finished_at`. Sort
   oldest-first. Cap to 5.
3. If the filtered list is empty, **skip silently** — do not post,
   do not DM, do not update MEMORY.md. Most heartbeats will exit
   here.
4. For each meeting in the batch, follow the SOUL **Heartbeat
   Workflow** Phases 2 and 3: fetch the summary, distill to ≤5
   bullets, format the recap, redact participant emails for
   public channels, and post once per channel the bot is a member
   of (DM the install user if invited to none).
5. After all meetings are posted (success or partial failure),
   update the `granola-cursor` block in MEMORY.md with the id and
   `finished_at` of the **newest** meeting in this batch.
6. Your turn ends after MEMORY.md is updated. No follow-ups, no
   thread replies, no reactions.

## Skip conditions

Stay silent (no post, no MEMORY.md change) if:

- No meetings have finished since `last_finished_at`. This is the
  expected case for most heartbeat fires.
- `granola-mcp` returns an error or empty list — wait for the next
  heartbeat. The cursor is unchanged so nothing is lost.
- It's the seed run (no cursor in MEMORY.md). Write the cursor and
  stop without posting.

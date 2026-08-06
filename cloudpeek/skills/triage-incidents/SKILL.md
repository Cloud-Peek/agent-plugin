---
name: triage-incidents
description: Triage CloudPeek incidents so the operator only sees the small fraction of alerts that matter. Use when asked to review, prioritise, assign, or summarise open security incidents in CloudPeek.
---

# Triage CloudPeek incidents

CloudPeek clusters raw alerts into incidents and enriches them so most noise is
already suppressed. Your job during triage is to confirm what deserves human
attention, not to re-review every alert.

## Workflow

1. Get the current picture with `get_incident_statistics`, then pull the queue
   with `list_incidents`. Filter to open incidents and sort by severity; do not
   fetch closed incidents unless asked.
2. For each candidate incident, call `get_incident_summary` first. Only call
   `get_incident` for full detail when the summary leaves a triage question
   open.
3. Check the blast radius before judging severity: `list_incident_events` for
   the timeline and `list_incident_clusters` for related alert groups. An
   incident with one noisy rule firing repeatedly is different from one with
   several distinct sources agreeing.
4. Read prior context with `list_incident_comments` so you never repeat a
   conclusion a human has already recorded.
5. Act on your verdict:
   - Worth attention: `assign_incident` to the right owner and record your
     reasoning with `add_incident_comment`.
   - Needs a status or severity change: `update_incident`, and say why in a
     comment.
   - Warrants deeper work: hand over to the `investigate-incident` skill.

## Rules

- Always leave an `add_incident_comment` explaining any change you make; the
  comment trail is the audit record the SOC relies on.
- Never delete incidents during triage. `delete_incident` is an operator
  decision, not a triage outcome.
- Summarise the queue for the operator as: what needs them now, what you
  handled, and what you suppressed and why.

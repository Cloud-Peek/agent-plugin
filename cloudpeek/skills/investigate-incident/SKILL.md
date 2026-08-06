---
name: investigate-incident
description: Run a structured CloudPeek investigation on an incident, gather artifacts and evidence, and produce a report with remediation options. Use when asked to investigate, dig into, or write up a CloudPeek incident.
---

# Investigate a CloudPeek incident

An investigation is CloudPeek's durable record of the questions asked about an
incident and the evidence found. Work inside it rather than in ad-hoc chat so
the findings survive the session.

## Workflow

1. Check for existing work first: `list_investigations` filtered to the
   incident. Resume with `get_investigation` if one exists; otherwise start one
   with `create_investigation`.
2. Build the evidence base:
   - `list_incident_events` for the timeline.
   - `list_incident_artifacts` for what is already attached, and
     `search_artifacts` to find related artifacts across the workspace.
   - `list_investigation_actions` and `get_investigation_action` to see what
     the investigation has already run before repeating a step.
   - `search_wiki` for tenant runbooks or prior write-ups about the affected
     system.
3. Track hypotheses in the investigation, not in your head: record findings as
   you go and check `list_investigation_responses` for analyst answers to any
   questions the investigation has raised.
4. When the picture is clear, propose fixes with
   `create_incident_remediation_plan` (check
   `list_incident_remediation_plans` first to avoid duplicates). Remediation
   plans are proposals; executing them is gated on human approval.
5. Close out with `generate_investigation_report`, confirm it with
   `get_investigation_report`, and `pin_investigation` if the operator will
   need it again soon.

## Rules

- Evidence before conclusions: every claim in the report should trace to an
  artifact, event, or action result gathered above.
- Do not call `delete_investigation`; superseded investigations stay for the
  audit trail.
- End by telling the operator the verdict, the confidence, and the single next
  decision that is theirs to make.

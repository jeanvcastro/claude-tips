# Usage Example: Browser Copilot (--chrome)

## Context
Demonstrates using Claude Code with Chrome integration for operational tasks: reading Jira tickets, analyzing Grafana dashboards, debugging with Datadog logs, and cross-referencing between tools.

## Setup
1. Install the [Claude Code for Chrome](https://chromewebstore.google.com/detail/claude-code/claudecode) extension
2. Start Claude Code with the `--chrome` flag:
```bash
claude --chrome
```

## Example 1 — Jira ticket to implementation plan

Open the Jira ticket in Chrome before starting.

```
Read the Jira ticket open in my browser (PROJ-1234).
Extract the requirements, acceptance criteria, and any relevant comments.
Create an implementation plan in .claude/task_plan.md based on what you find.
```

Follow-up (combining with careful engineer workflow):
```
Now execute the plan task by task. Read the plan in .claude/task_plan.md and start with the first pending task.
Update the plan after completing each task.
```

## Example 2 — Grafana dashboard analysis

Open the Grafana dashboard before starting.

```
Look at the Grafana dashboard open in my browser.
Analyze the latency and throughput graphs for the last 24 hours.
Identify any anomalies, spikes or degradation patterns.
Suggest possible causes based on the codebase.
```

## Example 3 — Datadog log debugging

Open Datadog with the relevant log filter.

```
Read the error logs in Datadog filtered by service:payments in the last 30 minutes.
Identify the error pattern and frequency.
Correlate with the source code and suggest a fix.
If you can identify the root cause, create a task plan to fix it.
```

## Example 4 — Cross-tool investigation (Jira + Datadog)

Open both Jira and Datadog in separate tabs.

```
I have a bug report open in Jira (tab 1) and Datadog logs (tab 2).
1. Read the bug report in Jira — understand what the user reported
2. Switch to Datadog and search for related errors
3. Cross-reference the timestamps and error patterns
4. Identify the root cause in the codebase
5. Create a fix plan in .claude/task_plan.md
```

## Example 5 — Sentry error triage

Open Sentry issues page.

```
Read the top 5 unresolved issues in Sentry.
For each one:
- Summarize the error and stack trace
- Identify which file/function is causing it
- Estimate severity (critical/warning/low)
Give me a prioritized list of what to fix first.
```

## Tips
- Open all relevant tabs BEFORE starting `claude --chrome` — saves navigation time
- Works with any web tool: Confluence, Notion, CloudWatch, New Relic, PagerDuty, etc
- The Claude reads what's visible on screen, so scroll to the relevant section first for best results
- Combine with Mode 1: let Claude read the ticket, create the plan, then execute task by task with human review

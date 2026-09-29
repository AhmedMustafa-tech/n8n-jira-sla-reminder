# ⏱️ Jira SLA reminders to Slack (n8n template)

Stop watching Jira by hand. This n8n workflow checks your Jira Service Management queue every hour during business hours. It **DMs the assignee in Slack before a ticket breaches its SLA** and posts breached tickets to an escalation channel.

![n8n](https://img.shields.io/badge/n8n-EA4B71?style=flat&logo=n8n&logoColor=white) ![Jira](https://img.shields.io/badge/Jira_Service_Management-0052CC?style=flat&logo=jira&logoColor=white) ![Slack](https://img.shields.io/badge/Slack-4A154B?style=flat&logo=slack&logoColor=white) ![License](https://img.shields.io/badge/license-MIT-green?style=flat)

## How it works

```
Schedule (hourly, Mon–Fri 08–18)
   → Config (your Jira site, JQL, SLA field, warning threshold)
   → Jira: search open tickets (REST API v3 /search/jql)
   → Code: keep only tickets at risk or breached
   → Slack: look up the assignee by email
   → Slack: DM the assignee
   → Breached? → Slack: post to #sla-escalations
```

| State | What happens |
|---|---|
| More than `warnHours` left | Nothing, the ticket is healthy |
| `warnHours` or less left | ⏳ Heads-up DM to the assignee |
| Past the SLA | 🚨 DM to the assignee **and** a post in the escalation channel |

Unassigned tickets are skipped on purpose. They belong in triage, not in someone's DMs.

## Setup (10 minutes)

1. **Import:** in n8n, go to *Workflows → Import from file* and pick `workflow.json`.
2. **Credentials:**
   - **Jira Software Cloud API**: your Atlassian email + an [API token](https://id.atlassian.com/manage-profile/security/api-tokens). Select it on the *Jira: search open tickets* node.
   - **Slack API**: a Slack app bot token with `users:read`, `users:read.email` and `chat:write`. Select it on the three Slack nodes and invite the bot to your escalation channel.
3. **Config node:** set `jiraBaseUrl`, `jql`, `slaField` and `warnHours`.
   - To find `slaField`: in Jira go to *Settings → Issues → Custom fields* and look for "Time to resolution". Or open `/rest/api/3/issue/KEY-1?expand=names` and search for your SLA name.
4. Run it once manually, check the DMs, then **activate**.

## Ideas to extend

- **De-duplicate:** store `key + date` in an n8n Data Table so each ticket gets one heads-up per day.
- **Escalate by repeat count** instead of by breach: remind twice a day for two days, then escalate.
- Add a morning digest of all at-risk tickets per engineer.

## Notes

- The workflow only **reads** Jira. It never changes tickets.
- Written for n8n 1.x. After import, re-select credentials on every node that shows a warning.

---

Built by [Ahmed Mohamed](https://github.com/AhmedMustafa-tech), IT Operations & Automation Engineer. Need this adapted to your team? [Let's talk on LinkedIn](https://www.linkedin.com/in/ahmed-mohamed-9411311a8/).

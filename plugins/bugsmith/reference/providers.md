# Ticket providers

A **provider** turns a bug source into the normalized **ticket object** that the
rest of the pipeline consumes. The ticket object is the contract; adding a new
source (Jira, Linear, an email, a Slack permalink) is a new mapping here, not a
change to the pipeline.

## Ticket object (the contract)

```jsonc
{
  "id": "86bax78c6",           // provider-native id; "" for raw
  "provider": "clickup",        // clickup | github-issue | raw | <custom>
  "url": "https://app.clickup.com/t/86bax78c6",
  "title": "Approving series events overrides cancelled event statuses",
  "body": "full description / markdown",
  "reporter": "Kurtis Hook",   // optional
  "env": "Chrome; QA Dept of test org",   // optional
  "stepsToReproduce": ["...", "..."],       // optional, best-effort parse
  "expected": "cancelled events stay cancelled",  // optional
  "actual": "cancelled events flip to approved",  // optional
  "attachments": []             // optional
}
```

Only `title` + `body` are strictly required; parse the rest best-effort (bug
tickets often have "Steps to Reproduce / Expected / Actual" sections — split them
out when present, they sharpen diagnosis and the regression test).

## Built-in providers

### `clickup`
Detect from an `app.clickup.com/t/<id>` URL or a bare task id. Fetch with the
ClickUp MCP tools: the task (include `description`, `custom_fields`) and its
comments. Map custom fields like "Steps to Reproduce", "Reporter",
"Device and Browser Details", "Feature Toggles" into the ticket object. The task
name is `title`, the markdown description is `body`.

### `github-issue`
Detect from a `github.com/<org>/<repo>/issues/<n>` URL or `#<n>` with a repo in
context. Fetch with `gh issue view <n> --json title,body,url,author,labels`.

### `raw`
Anything else: a quoted plain-text description. `provider: "raw"`, `id: ""`,
synthesize a concise `title` from the text, put the whole thing in `body`.

## Adding a provider

1. Add detection (URL/id shape) and a fetch step that yields the ticket object.
2. Reference it from config (`ticket.provider` or `auto`).
3. Nothing downstream changes — triage, branch naming (falls back to a title slug
   when `id` is empty), diagnosis, and the PR link all read the ticket object.

## Writing back to the ticket (optional)

If config enables it (`ticket.writeBack: true`) and the provider supports it,
after opening the PR you may post the PR link / status back to the ticket
(e.g. a ClickUp comment, a GitHub issue comment). Off by default; never change
ticket *status* without explicit config.

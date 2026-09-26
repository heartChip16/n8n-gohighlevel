# Lead Capture → HighLevel CRM (n8n)

A self-hosted n8n workflow that turns a public web form into a clean, tagged
HighLevel contact with a follow-up task, and tells the visitor their CRM
contact ID within seconds.

**Stack:** n8n · HighLevel API v2 · OAuth 2.0 · Docker

## How it works

```
On form submission → Normalize lead → Upsert contact → Create follow-up task → Form ending
```

1. **On form submission**: a public n8n form collects name, email, phone and interest.
2. **Normalize lead**: trims and lowercases the email, splits out the first name,
   and handles an empty phone field.
3. **Upsert contact**: creates or updates the HighLevel contact, matched by email.
   Submitting the same email twice updates the existing contact instead of
   creating a duplicate. Tags: `web-lead` plus `interest-<choice>`.
4. **Create follow-up task**: attaches a task to that contact, due the next day.
5. **Form ending**: shows the visitor a confirmation with their contact ID.

## Design notes

- **Email normalization protects the upsert.** HighLevel matches contacts by email,
  so ` Ana.Cruz@Example.com ` and `ana.cruz@example.com` must become identical
  before the request, or the same person could end up as two contacts.
- **Tags are built in one expression.** The Tags field splits on commas, so the
  tag list is computed as a single expression (`web-lead,interest-automation`)
  rather than typed as separate values.
- **Authentication uses a private HighLevel Marketplace app** (OAuth 2.0, sub-account
  level). n8n's authorization link doesn't include an app version ID, so the app
  needs a published (live) version for the OAuth flow to resolve. The app stays
  private and restricted to the owning agency.

## Run it locally

Requirements: Docker with Docker Compose, a HighLevel sub-account, and a private
HighLevel Marketplace app with a published version.

1. Start n8n:
```bash
   docker compose up -d
```
2. Open http://localhost:5678 and import `exports/p1-lead-capture.json`
   (Workflows → Import from file).
3. Create a **HighLevel OAuth2 API** credential in n8n. Add n8n's OAuth redirect URL
   to the Marketplace app, then connect your sub-account. Scopes used:
   `locations.readonly contacts.readonly contacts.write opportunities.readonly
   opportunities.write users.readonly calendars.readonly calendars/events.readonly
   calendars/events.write`
4. Select the credential in both HighLevel nodes, publish the workflow, and open
   the form's production URL.

## Repository layout

```
docker-compose.yml             n8n container configuration
exports/p1-lead-capture.json   the workflow, exported with --pretty
```

Secrets (the credential's client secret and tokens) live in n8n's encrypted
database and are never committed.

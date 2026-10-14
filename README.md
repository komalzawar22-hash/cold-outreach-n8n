# Cold Outreach System (n8n + AI)

An n8n workflow that finds companies for a service you offer, researches their websites, writes short personalised emails with AI, and follows up automatically. A Google Sheet is the control panel, and **every lead is reviewed by a human before anything is sent**.

> Generic template. No API keys, sheet IDs, or personal data are included. Use it responsibly and in line with the email laws that apply to you and your recipients (for example CAN-SPAM, GDPR/PECR).

## What it does

**Flow A: Find leads (runs daily)**
1. Reads the active rows in the `Offers` tab (what you sell and who you target)
2. Asks OpenAI for web-search queries that surface company websites
3. Searches with Tavily and filters out directories, job boards, social media, and government/education sites
4. Fetches each homepage and contact page and extracts a public contact email (Hunter.io is optional)
5. Adds new leads to the `Leads` tab as `Pending` (or `Approved` if `autoApprove` is on). Existing domains and emails are skipped.

**Flow B: Send and follow up (runs daily)**
1. Checks the Gmail inbox for replies and bounces and marks those leads `Replied` / `Bounced`
2. Picks due leads (new `Approved` leads, plus follow-ups after the configured number of days), follow-ups first, up to `dailyLimit`
3. Writes each email with OpenAI (no flattery opening, no invented claims, no unconditional free offers), adds your signature and an opt-out line
4. Sends through Gmail with a pause between emails and updates the sheet (status, step, last sent date, subject)

## Tech stack
n8n, OpenAI API (`gpt-4o-mini` by default), Tavily Search API, Google Sheets, Gmail, optional Hunter.io

## Setup
1. **Import** `workflow/cold-outreach-workflow.json` into n8n (Workflows -> Import from file).
2. **Create the Google Sheet** with three tabs named exactly `Leads`, `Offers`, `Settings`. Import the CSV headers/examples from `sheet-templates/`.
3. **Fill both Config nodes** (`Config (Find)` and `Config (Send)`): API keys and the Google Sheet ID (the long string in the sheet URL). Do not commit real keys if you export the workflow again.
4. **Connect credentials** in n8n: Google Sheets (all Sheets nodes) and Gmail (`Send Email` and `Check Inbox`).
5. **Start in test mode:** keep `testMode = TRUE` and set `testEmail` to another address you own. All emails go only to that address and no lead is marked as sent.
6. Run Flow A manually, then review the `Leads` tab: set good leads to `Approved`, bad ones to `Skip`.
7. When the test emails look right, set `testMode = FALSE`. Increase `dailyLimit` slowly (for example 3, then 5, then 8-10 over several weeks) to protect your sender reputation.
8. Publish the workflow so the schedules run.

Schedules use 6-field cron (`[second] [minute] [hour] [day] [month] [weekday]`) in your n8n timezone. Edit the two Schedule Trigger nodes to change the times.

## Sheet structure
**Leads:** Company Name, Website, Email, Contact Name, Offer, Signal Evidence, Status, Notes, Step, Date Added, Last Sent Date, Last Subject

Status values: `Pending` (waiting for review), `Approved` (send), `Skip` (ignore), `Active` (sequence running), `Replied`, `Bounced`, `Done` (sequence finished)

**Offers:** Offer Name, Skill, Pain Signals, Pitch Angle, Target Companies, Region, Demo Link, Active (`yes` / `no`)

**Settings (Key / Value):**

| Key | Meaning |
|---|---|
| testMode | TRUE sends only to testEmail and does not mark leads as sent |
| testEmail | Address that receives test emails |
| dailyLimit | Maximum emails per run |
| findLimit | Maximum new companies researched per run |
| autoApprove | TRUE adds leads as Approved without review (use only after trusting the lead quality) |
| senderName | Name used in the prompt and signature |
| signatureExtra | Extra signature line, such as a profile link |
| optOutLine | Opt-out sentence added to every email |
| hunterMode | `never`, `fallback` (only when the site has no email) or `always` |
| followUpDay2 / followUpDay3 | Days to wait before follow-up 1 and follow-up 2 |
| gapMinutes | Pause between emails |

## Customising
- New service or market: add a row in `Offers`. No workflow edit needed.
- Email tone and structure: edit the `Build Email Prompt` code node.
- Blocked sites: edit the `blocked` list in `Parse Results`.

## Limitations
- Many small business websites do not publish an email address, so lead yield can be low without a paid email-finder.
- Lead quality depends on your `Offers` description; review leads before approving.
- Gmail has sending limits and new accounts need a slow ramp-up.
- AI-written emails should be reviewed in test mode before real sending.

## Need something like this?
I build n8n and AI automations for small teams (lead research, document processing, reporting, AI agents).
LinkedIn: https://www.linkedin.com/in/komal-zawar-a6b455345/

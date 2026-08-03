# Lead Qualification Workflow (n8n)
This is a rule-based lead qualification and routing system built entirely in n8n without any AI agent. The leads submit a form, weighed against the business criteria and scored, then automatically routed to the right action: Sales notification, Human review, and an email for qualified leads and a polite decline for unqualified leads.

![Workflow Overview](Workflow%20overview.jpeg)

## Why this exists
Most of the leads in small businesses are manual, someone has to read the form submission and then decide what to do with it. This workflow helps reduce the workload of manually going through lots of form submissions. This workflow will easily automate the judgement calls using explainable logic, so that every decision can be traced back to an exact rule.

## What it does
1. Captures leads via a Google Form (budget, timeline, company size, job title, referral source)
2. Scores each lead out of 100 points across 5 weighted criteria
3. Logs the score and result back to a Google Sheet for a permanent audit trail
4. Routes the lead into one of three outcomes:
   - Qualified (70–100) → Slack alert to sales + confirmation email to the lead
   - Needs Review (40–69) → Slack alert to a human reviewer
   - Disqualified (0–39) → automatic, polite decline email

## Architecture
```
Google Form
    ↓
Google Sheets Trigger (New Lead Submitted)
    ↓
Code Node (Calculate Lead Score)
    ↓
Google Sheets — Update Row (Log Score to Sheet)
    ↓
Switch Node (Route by Qualification Status)
    ├── Qualified      → Slack + Gmail (parallel)
    ├── Needs Review   → Slack
    └── Disqualified   → Gmail
```
Scores are calculated with plain JavaScript lookup tables rather than long if/else chains.

## Key technical decisions
- Deterministic over AI: every score is traceable to a specific rule — useful where explainability matters more than nuance.
- Parallel branch execution: the Qualified path fires the Slack alert and the confirmation email independently, so one integration failing doesn't block the other.
- Sheet as source of truth: every lead's score and outcome is written back to the spreadsheet, turning a one-off automation into a queryable historical log.
- Exact-match lookups: dropdown text is matched character-for-character against the scoring tables — a deliberate tradeoff for simplicity, with a documented failure mode below.

## Tech stack
- n8n — workflow orchestration
- Google Forms + Sheets — lead intake and data log
- JavaScript (Code node) — scoring logic
- Slack API — internal notifications
- Gmail API — lead-facing emails

## Screenshots
![Code node](Java$20Script%20code.jpeg) | ![Sheet log](Google%20Sheet.jpeg) |

## Setup
1. Create a Google Form with the 5 scoring fields + Name/Email, linked to a Google Sheet
2. Import `workflow.json` into n8n
3. Connect credentials: Google Sheets OAuth, Slack Bot Token, Gmail OAuth
4. Update the Switch node's Slack channel IDs and Gmail sender address
5. Activate the workflow

---

Built by Edgar — part of a portfolio of n8n automation projects covering both deterministic workflows and AI-agent systems.

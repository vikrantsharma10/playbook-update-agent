# Playbook Update Agent

An n8n + Claude automation that keeps your company playbook updated via Slack. A whitelisted team member types `/updateplaybook` in a private Slack channel; the agent reads the playbook, determines what needs to change, drafts the update and sends it back with Approve and Reject buttons. One command; done.

---

## Why This Exists

Most company playbooks are out of date. Not because nobody cares but because the update process has too many steps between "I know this changed" and "the doc reflects it." This agent compresses that entire process to one Slack command.

No new tools to learn. No dashboards. No access requests. Just a command in a channel your team already lives in.

---

## How It Works

1. A whitelisted team member types `/updateplaybook` followed by a description of what changed
2. The agent reads the live playbook and determines on its own whether this is an addition or a replacement
3. It identifies the right section, matches the existing writing style and prepares a draft
4. The draft is posted back in Slack tagging the sender with **Approve** and **Reject** buttons
5. Clicking Approve executes the edit directly in the playbook and posts a confirmation
6. Clicking Reject prompts the agent to ask what was wrong; the user replies in the thread
7. The agent prepares a revised draft with a second round of Approve and Reject buttons
8. After two rejections; the agent stops and flags it for manual intervention
9. Every approved change is optionally logged to a Google Sheet

---

## Before You Start

This agent works best when your playbook is structured. Consistent section headings are non-negotiable; the agent navigates by them. If a human cannot find a section in under 30 seconds the agent will struggle too. Fix your document structure before connecting anything.

**You will need:**
- A GitHub account (free at github.com)
- An n8n cloud account (n8n.io)
- A Claude API key (console.anthropic.com)
- A Slack workspace where you have permission to create apps
- Your playbook stored in Google Docs, Notion or Confluence

---

## Setup

### Step 1 — Get this repo
Click **Use this template** at the top of this page and create your own private copy. This is your version to configure; it runs completely independently from anyone else using this template.

New to GitHub? Create a free account at github.com first. No coding knowledge required; you are just copying and downloading files.

### Step 2 — Review your config file
Open `config.env.example` in your repo. This lists every variable you need and where to get it. Keep this open as a reference throughout setup.

### Step 3 — Create your Slack app

**Create the app:**
- Go to api.slack.com → Your Apps → Create New App → From Scratch
- Name it anything (e.g. Playbook Agent) and select your workspace

**Add bot permissions:**
- Inside your app go to OAuth & Permissions
- Under Scopes → Bot Token Scopes add: `chat:write`, `channels:history`, `channels:read`

**Enable interactivity (required for buttons):**
- Go to Interactivity & Shortcuts inside your app
- Toggle Interactivity ON
- Leave the Request URL blank for now; you will fill this in after Step 6
- Save

**Create the slash command:**
- Go to Slash Commands → Create New Command
- Command: `/updateplaybook`
- Request URL: leave blank for now; you will fill this in after Step 6
- Short description: Update the company playbook
- Save

**Get your bot token:**
- Go to OAuth & Permissions → Install App to Workspace → Authorise
- Copy the **Bot User OAuth Token**; you will need this in Step 7

### Step 4 — Set up your Slack channel
- Create a **private Slack channel** (e.g. #playbook-updates)
- Add only the people authorised to submit playbook updates; channel membership is your whitelist
- Invite the bot to the channel by typing `/invite @YourBotName` inside the channel
- Get the channel ID: right-click the channel name → View channel details → Channel ID at the bottom

### Step 5 — Connect your playbook
Follow the relevant platform section at the bottom of this README.

### Step 6 — Import the workflow into n8n
- Log into n8n → Workflows → Import from File
- Select `workflow/n8n-workflow.json` from your repo
- Open the imported workflow
- You will see two webhook nodes: **Slash Command Trigger** and **Button Response Webhook**
- Copy the URL from **Slash Command Trigger** and paste it into api.slack.com → Slash Commands → edit `/updateplaybook` → Request URL → Save
- Copy the URL from **Button Response Webhook** and paste it into api.slack.com → Interactivity & Shortcuts → Request URL → Save

### Step 7 — Add environment variables in n8n
- In n8n go to Settings → Environment Variables
- Add each variable from your `config.env.example` reference file
- The workflow reads from these variables automatically; you do not need to edit the JSON manually

### Step 8 — Activate the workflow
There are three triggers in the workflow that must all be active:
- **Slash Command Trigger** — fires when someone types `/updateplaybook`
- **Button Response Webhook** — fires when someone clicks Approve or Reject
- **Listen for Clarification Reply** — fires when someone replies after a rejection

Activate all three separately in n8n. If any one is inactive the flow will break at that step.

---

## Platform Setup

### Google Docs
- Every section title in your Doc must use **Heading 2** formatting; this is how the agent navigates
- In n8n → Credentials → Add Credential → search **Google Docs OAuth2 API** → Sign in with Google → Authorise
- Copy your Doc ID from the URL: `docs.google.com/document/d/YOUR_DOC_ID/edit`
- Add it to n8n environment variables as `GOOGLE_DOC_ID`

### Notion
- Every section title in your Notion page must use H1 or H2 heading blocks
- Go to notion.so/my-integrations → New Integration → copy the Internal Integration Token
- Open your playbook page → Share → Invite → select your integration → confirm
- Copy the Page ID from the page URL: the string after the last `-` before any `?` parameters
- In n8n → Credentials → Add Credential → **Notion API** → paste your token
- Add `NOTION_API_KEY` and `NOTION_PAGE_ID` to your n8n environment variables

### Confluence
- Every section title must use consistent heading levels throughout the page
- Go to id.atlassian.com → Security → API Tokens → Create token; copy it
- Your base URL: `https://yourcompany.atlassian.net`
- Find your Page ID: open the page → Page Information → Page ID in the URL
- In n8n → Credentials → Add Credential → **HTTP Header Auth** → header name: `Authorization`; value: `Basic YOUR_BASE64_ENCODED_EMAIL:TOKEN`
- Add `CONFLUENCE_API_TOKEN`, `CONFLUENCE_BASE_URL` and `CONFLUENCE_PAGE_ID` to your n8n environment variables

---

## Optional: Change Log

The workflow includes a Google Sheets node that logs every approved change; timestamp, submitter, section, change type and status. It is marked **Optional** in the workflow.

To skip it; disconnect that node. Google Docs, Notion and Confluence all track version history natively so nothing is ever lost.

To enable it; create a Google Sheet with a tab named exactly **Change Log**, copy the Sheet ID from the URL and add it to your environment variables as `GOOGLE_SHEET_ID`.

---

## How to Submit an Update

Type `/updateplaybook` in the private channel followed by a plain description of what changed. No need to specify the section or whether it is an addition or replacement; the agent figures that out by reading the playbook itself.

**Example:**
```
/updateplaybook The leave request notice period has changed from 5 working 
days to 7 working days. Manager approval is now required for any leave 
exceeding 10 consecutive days.
```

The agent will post a draft in the channel tagging you with Approve and Reject buttons. Click Approve to execute or Reject to revise.

---

## Troubleshooting

**Slash command not responding**
Check that the Slash Command Trigger webhook URL in n8n matches exactly what is entered in api.slack.com → Slash Commands. Check that the workflow is active in n8n.

**Approve and Reject buttons not working**
Check that the Button Response Webhook URL in n8n matches exactly what is entered in api.slack.com → Interactivity & Shortcuts → Request URL. Check that the Button Response Webhook trigger is active in n8n.

**Agent not posting a draft**
Check your Claude API key is correctly added to n8n environment variables. Check your Google Docs credential has not expired; re-authenticate if needed.

**Agent cannot identify the correct section**
Your playbook headings are likely inconsistent. Ensure every section title uses the same heading level throughout the document with no exceptions.

**Edit not executing after approval**
Re-authenticate your Google Docs OAuth2 credential in n8n. Credentials occasionally expire and need reconnecting.

**Concurrent edit conflict**
If someone is manually editing the playbook at the exact moment the agent tries to write; the edit may fail. Wait for the manual edit to finish and resubmit the request.

---

## Limitations

- Works best with flat; consistently structured playbooks
- Very long documents may approach API token limits; consider splitting into separate docs per vertical
- After two rejections on the same request the agent stops; the edit must be made manually
- The agent does not lock the document before editing; concurrent edits can cause conflicts

---

## Coming Soon

- Notion workflow variant
- Confluence workflow variant
- Vertical-specific whitelisting (HR lead can only edit HR sections)

---

## Questions

Raise an issue on GitHub.

# Playbook Update Agent

An n8n + Claude automation that keeps your company playbook updated via Slack. A whitelisted team member types `/updateplaybook` in a private Slack channel; the agent reads the playbook, determines what needs to change, drafts the update and sends it back with Approve and Reject buttons. One command; done.

Supports **Google Docs**, **Notion** and **Confluence**.

---

## Why This Exists

Most company playbooks are out of date. Not because nobody cares but because the update process has too many steps between "I know this changed" and "the doc reflects it." This agent compresses that entire process to one Slack command.

No new tools to learn. No dashboards. No access requests. Just a command in a channel your team already lives in.

---

## How It Works

1. A whitelisted team member types `/updateplaybook` followed by a description of what changed
2. The agent reads the live playbook and determines on its own whether this is an addition or a replacement
3. It identifies the right section and matches the existing writing style
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
Click **Use this template** at the top of this page and create your own private copy. This runs completely independently from anyone else using this template.

New to GitHub? Create a free account at github.com first. No coding knowledge required; you are just copying files.

### Step 2 — Review your config file
Open `config.env.example`. This lists every variable you need and where to get it. Keep it open as a reference throughout setup.

### Step 3 — Create your Slack app

**Create the app:**
- Go to api.slack.com → Your Apps → Create New App → From Scratch
- Name it anything (e.g. Playbook Agent) and select your workspace

**Add bot permissions:**
- Inside your app go to OAuth & Permissions
- Under Scopes → Bot Token Scopes add: `chat:write`, `channels:history`, `channels:read`

**Enable interactivity (required for Approve/Reject buttons):**
- Go to Interactivity & Shortcuts inside your app
- Toggle Interactivity ON
- Leave the Request URL blank for now; you will fill this in after Step 6

**Create the slash command:**
- Go to Slash Commands → Create New Command
- Command: `/updateplaybook`
- Request URL: leave blank for now; you will fill this in after Step 6
- Short description: Update the company playbook
- Save

**Get your bot token:**
- Go to OAuth & Permissions → Install App to Workspace → Authorise
- Copy the **Bot User OAuth Token**

### Step 4 — Set up your Slack channel
- Create a **private Slack channel** (e.g. #playbook-updates)
- Add only the people authorised to submit updates; channel membership is your whitelist
- Invite the bot: type `/invite @YourBotName` inside the channel
- Get the channel ID: right-click the channel → View channel details → Channel ID at the bottom

### Step 5 — Connect your playbook
Go to the relevant platform section below and complete that setup before continuing.

### Step 6 — Import the workflow into n8n
- Log into n8n → Workflows → Import from File
- Select the workflow file that matches your platform from the `workflow` folder:
  - Google Docs → `n8n-workflow-googledocs.json`
  - Notion → `n8n-workflow-notion.json`
  - Confluence → `n8n-workflow-confluence.json`
- Open the imported workflow
- Click the **Slash Command Trigger** node → copy the webhook URL → paste it into api.slack.com → Slash Commands → edit `/updateplaybook` → Request URL → Save
- Click the **Button Response Webhook** node → copy that URL → paste it into api.slack.com → Interactivity & Shortcuts → Request URL → Save

### Step 7 — Add environment variables in n8n
- In n8n go to Settings → Environment Variables
- Add each variable from your `config.env.example` file
- Fill in only the variables for your chosen platform; leave the others blank

### Step 8 — Activate the workflow
Three triggers must all be active:
- **Slash Command Trigger** — fires when someone types `/updateplaybook`
- **Button Response Webhook** — fires when someone clicks Approve or Reject
- **Listen for Clarification Reply** — fires when someone replies after a rejection

Activate all three separately in n8n.

---

## Platform Setup

### Google Docs
- Every section title must use **Heading 2** formatting; this is how the agent navigates
- In n8n → Credentials → Add Credential → search **Google Docs OAuth2 API** → Sign in with Google → Authorise
- Copy your Doc ID from the URL: `docs.google.com/document/d/YOUR_DOC_ID/edit`
- Add to n8n environment variables: `GOOGLE_DOC_ID`

### Notion
- Every section title must use H1 or H2 heading blocks
- Go to notion.so/my-integrations → New Integration → copy the Internal Integration Token
- Open your playbook page → Share → Invite → select your integration → confirm
- Copy the Page ID from the URL: the string after the last `-` before any `?`
- In n8n → Credentials → Add Credential → **Notion API** → paste your token
- Add to n8n environment variables: `NOTION_API_KEY`, `NOTION_PAGE_ID`

### Confluence
- Every section title must use consistent heading levels throughout
- Get your API token: id.atlassian.com → Security → API Tokens → Create token
- Generate your auth token: combine `your_email@company.com:your_api_token` and Base64 encode it at base64encode.org
- Find your Page ID: open the page in Confluence → the number after `pageId=` in the URL
- Add to n8n environment variables: `CONFLUENCE_BASE_URL`, `CONFLUENCE_PAGE_ID`, `CONFLUENCE_AUTH_TOKEN`

---

## Optional: Change Log

The Google Docs workflow includes a Google Sheets logging node. It records every approved change; timestamp, submitter, section, change type and status.

To skip it; disconnect that node. All three platforms track version history natively.

To enable it; create a Google Sheet with a tab named exactly **Change Log** and add the Sheet ID as `GOOGLE_SHEET_ID` in your environment variables.

---

## How to Submit an Update

Type `/updateplaybook` followed by a plain description of what changed. No need to specify the section or whether it is an addition or replacement; the agent determines that itself by reading the playbook.

**Example:**
```
/updateplaybook The leave request notice period has changed from 5 working 
days to 7 working days. Manager approval is now required for any leave 
exceeding 10 consecutive days.
```

The agent posts a draft tagging you in the channel with Approve and Reject buttons. Click Approve to execute or Reject to revise.

---

## Troubleshooting

**Slash command not responding**
Check that the Slash Command Trigger webhook URL in n8n matches exactly what is in api.slack.com → Slash Commands. Check the workflow is active.

**Approve and Reject buttons not working**
Check that the Button Response Webhook URL in n8n matches exactly what is in api.slack.com → Interactivity & Shortcuts. Check that trigger is active.

**Agent cannot identify the correct section**
Your playbook headings are inconsistent. Ensure every section title uses the same heading level with no exceptions.

**Confluence edit failing**
Check that your version number is incrementing correctly. Confluence rejects writes if the version number does not match. Re-read the page to get the latest version and retry.

**Notion edit failing**
Check that your integration has been shared with the playbook page. Open the page → Share → confirm your integration is listed.

**Google Docs credential expired**
Re-authenticate your Google Docs OAuth2 credential in n8n → Credentials.

---

## Limitations

- Works best with flat; consistently structured playbooks
- Very long documents may approach API token limits; consider splitting into separate docs per vertical
- After two rejections the agent stops; the edit must be made manually
- The agent does not lock documents before editing; avoid concurrent manual edits during an active agent session

---

## Questions

Raise an issue on GitHub.

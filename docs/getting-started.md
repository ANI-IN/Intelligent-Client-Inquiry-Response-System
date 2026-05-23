# Getting started

This guide takes you from a fresh n8n instance to a working **Intelligent Client Inquiry Response System** that sends a real email in response to a real lead. Set aside about thirty minutes the first time you do it.

You will:

1. Choose an n8n host (Cloud or self-hosted).
2. Import the workflow JSON.
3. Configure three credentials (Google Sheets, OpenAI, Gmail).
4. Point the trigger at your own Google Sheet.
5. Run a dry test, then activate.

## 1. Choose a host

You have two reasonable options.

### Option A: n8n Cloud (fastest)

1. Sign up at <https://n8n.io>.
2. Create a workspace. The free tier is enough for testing.
3. Skip to [step 2](#2-import-the-workflow).

### Option B: self-hosted with Docker

1. Install [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/install/).
2. Create a folder for your n8n data and an `.env` file based on [.env.example](../.env.example). At minimum set `N8N_ENCRYPTION_KEY` to a fresh value:

   ```bash
   mkdir -p ~/n8n && cd ~/n8n
   cp /path/to/this/repo/.env.example .env
   # Generate a strong encryption key:
   openssl rand -hex 32
   # Paste the output into .env as N8N_ENCRYPTION_KEY=...
   ```

3. Save the following as `docker-compose.yml` in the same folder:

   ```yaml
   services:
     n8n:
       image: n8nio/n8n:latest
       restart: unless-stopped
       ports:
         - "5678:5678"
       env_file:
         - .env
       volumes:
         - ./data:/home/node/.n8n
   ```

4. Bring it up:

   ```bash
   docker compose up -d
   ```

5. Open <http://localhost:5678>, create the owner account, and log in.

## 2. Import the workflow

In the n8n editor:

1. Click **Workflows** in the sidebar, then **Add workflow**, then the three-dot menu, then **Import from file**.
2. Choose `Intelligent Client Inquiry Response System.json` from this repository.
3. The workflow opens with three nodes. Two of them show a red dot because their credentials are missing. That is expected; you will fix it in the next step.
4. Click **Save**. Give it the same name (or your own).

## 3. Configure credentials

### 3a. Google Sheets OAuth2

1. Click the **Google Sheets Trigger** node.
2. In the **Credential to connect with** dropdown, click **Create new credential**.
3. Follow the n8n popup to authorize Google Sheets. The required scope is `https://www.googleapis.com/auth/spreadsheets.readonly`. The first-time OAuth dance asks you to consent to n8n reading sheets you choose.
4. Save the credential with a name you will recognize (the workflow's exported name is "Google Sheets Trigger account 3"; you can name yours anything).
5. Back in the node, set:
   - **Document**: pick **your** Google Sheet (the form responses sheet). The committed JSON references a sample sheet ID that you do not own.
   - **Sheet**: the tab that holds the form responses (typically "Form responses 1").
   - **Event**: **Row Added**.
   - **Poll Times**: **Every minute** for testing; raise to every five or ten minutes in production.

### 3b. OpenAI API

1. Click the **Message a model** node.
2. In **Credential**, click **Create new credential**.
3. Paste an API key from <https://platform.openai.com/api-keys>. Use a project-scoped key, not your root key.
4. Save. Confirm the **Model** is `gpt-4o-mini` (the workflow's default) or pick another. Cheaper and faster models are appropriate for short outbound emails.

### 3c. Gmail OAuth2

1. Click the **Send a message** node.
2. In **Credential**, click **Create new credential**.
3. Follow the OAuth flow. The required scope is `https://www.googleapis.com/auth/gmail.send`. n8n will not request broader Gmail scopes for this workflow.
4. Save. Confirm the sender address shown matches the inbox you want replies to come from.

## 4. Align the sheet schema

The workflow reads these columns by name. They must exist exactly as written (case sensitive) in your Google Sheet:

- `Full Name`
- `Company Name`
- `Service Required`
- `What is your biggest business challenge right now?`
- `Business Email`

If your form uses different column titles, the simplest fix is to rename the columns in the linked Google Sheet (Google Forms lets you do this without breaking the form). The alternative is to edit the expressions inside the **Message a model** and **Send a message** nodes to point at your column names.

## 5. Dry-run before going live

You do not have to wait for a real lead. Here is the safest way to test:

1. **Pin a sample row.** In the **Google Sheets Trigger** node, click **Execute Node**. n8n will fetch existing rows. Click the pin icon next to one row to freeze it as test input. Use a row whose **Business Email** is your own address so you receive the test email yourself.
2. **Disable the workflow** (toggle in the top-right) so the live trigger does not double-fire while you test.
3. Click **Execute Workflow** at the bottom of the editor. n8n will run the three nodes in order.
4. Open your inbox. The reply should arrive within seconds, signed "Sarah from TechNova", with the subject `Re: Your inquiry regarding <Service Required>`.
5. If anything looks wrong, see [Troubleshooting](#troubleshooting) below.
6. When the dry run looks good, unpin the row, re-enable the workflow, and submit a real test through your form.

## 6. Activate

Toggle the workflow to **Active** in the top-right of the editor. From now on every new row in the Google Sheet triggers an email within sixty seconds. Monitor the **Executions** tab for failures.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Trigger node shows "Could not find sheet" | The document ID in the imported JSON points at a sheet you do not own | In the trigger node, pick **your** sheet from the dropdown |
| Email arrives with `Hi undefined,` | A column was renamed and an expression no longer resolves | Confirm sheet headers match the five names in section 4 |
| Email body contains ` ```html ` lines | LLM ignored the "no code fences" instruction | Re-run; if persistent, follow finding **TD-02** in the improvement plan to add a Set node that strips fences |
| Gmail node fails with `Recipient address required` | `Business Email` column was empty for that row | See finding **REL-01** in the improvement plan; add an IF guard before the Gmail node |
| OpenAI returns 429 | Free-tier rate limit or per-minute cap | Reduce trigger frequency, or add **Retry On Fail** to the LLM node, or upgrade your OpenAI plan |
| Trigger silently stops firing | OAuth refresh token revoked, or sheet permissions revoked | Open the credential in n8n and re-authorize |
| Two identical emails sent for one lead | n8n was restarted between LLM and Gmail; at-least-once delivery | Acceptable for low volume; see finding **REL-03** for an idempotency guard |

## Next steps

- Read [docs/architecture.md](architecture.md) to understand how the three nodes fit together.
- Read [../SECURITY.md](../SECURITY.md) before pointing the workflow at production data.
- If you plan to fork and harden this, see `improvements/IMPROVEMENT_PLAN.md` (kept locally; not pushed to the public repo).

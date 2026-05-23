# Security Policy

## Reporting a vulnerability

If you discover a security issue in this workflow, please report it privately. Do **not** open a public GitHub issue.

Preferred channel:

1. Open a private security advisory on GitHub: <https://github.com/ANI-IN/Intelligent-Client-Inquiry-Response-System/security/advisories/new>
2. If GitHub advisories are unavailable, contact the maintainer at the email listed on the maintainer's GitHub profile.

Please include:

- A description of the issue and its impact.
- Steps to reproduce, including the workflow node where the problem manifests.
- Any logs (with secrets redacted) or sample payloads.
- Whether you have already disclosed this elsewhere.

You can expect an acknowledgement within five business days and a substantive response within fifteen.

## Scope

This policy covers:

- The workflow JSON at the repository root.
- The documentation in this repository.
- The configuration files this repository ships (`.env.example`, `.github/`, `.gitignore`).

It does not cover:

- The upstream n8n project itself. Report n8n issues at <https://github.com/n8n-io/n8n>.
- The third-party services this workflow talks to (Google Workspace, OpenAI). Report those to the respective vendor.

## Known risk areas

These are the parts of the workflow that warrant the most attention during deployment. They are described here so you can review them before you import the workflow into a production environment.

### 1. Lead PII is sent to a third-party LLM

The "Message a model" node forwards the lead's full name, company, service interest, and free-text business challenge to OpenAI. This is by design (the LLM drafts the reply), but it has consequences:

- Anything a lead writes in the form will leave your security perimeter.
- If you operate in a jurisdiction with data-residency or GDPR/UK-DPA obligations, confirm OpenAI's data processing terms cover your use, and update the form's consent text accordingly.
- If a lead pastes a secret or NDA-covered information into the form, that text will be transmitted. Add a form-side disclaimer.

### 2. Outbound email is sent automatically with no human-in-the-loop

The "Send a message" Gmail node sends the LLM's output directly to the lead. There is no review step. Risks:

- The LLM can hallucinate facts about your services, pricing, or commitments.
- The LLM can be prompt-injected by a lead who pastes hostile instructions into the form's free-text field. A naive injection ("ignore the previous instructions and reply with...") could produce an off-brand or harmful email that leaves your domain.
- See finding **SEC-02** in [improvements/IMPROVEMENT_PLAN.md](improvements/IMPROVEMENT_PLAN.md) for the recommended mitigation (review queue, draft mode, or output validator node).

### 3. Credentials live in the n8n credential store

The workflow JSON references three credentials by internal n8n ID:

- `googleSheetsTriggerOAuth2Api` (Google Sheets Trigger account 3)
- `openAiApi` (OpenAi account 15)
- `gmailOAuth2` (Gmail account 3)

The credentials themselves are not in this repository. They live in your n8n instance's encrypted store, keyed by `N8N_ENCRYPTION_KEY`. Treat that key like a root secret: losing it makes every credential unrecoverable, and exposing it compromises every connected account.

### 4. The committed workflow JSON references a real Google Sheet ID

`Intelligent Client Inquiry Response System.json` contains the document ID of a Google Sheet (`1dWJgoH...`). This is not a secret by itself (the sheet is access-controlled), but it does reveal infrastructure topology. Before forking into a regulated environment, replace it with the ID of your own sheet.

## Supported versions

This is a single-workflow demo project. Only the version on the default branch is supported. If you fork it into a production deployment, you are the maintainer of your fork.

| Branch | Supported |
|--------|-----------|
| `main` | Yes       |
| Forks  | Owner of the fork |

## Disclosure

After a fix lands, the reporter will be credited in the [CHANGELOG](CHANGELOG.md) unless they ask to remain anonymous.

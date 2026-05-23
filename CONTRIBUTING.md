# Contributing

Thanks for taking the time to look at this project. It is a small n8n workflow, so contribution is mostly about correctness, documentation, and safe defaults rather than building new features.

## Quick checklist

Before opening a pull request, make sure:

- [ ] The workflow JSON still parses (`python -m json.tool "Intelligent Client Inquiry Response System.json" >/dev/null`).
- [ ] No real credentials, API keys, OAuth tokens, or personal data are present in any file you added or modified.
- [ ] Any new documentation renders correctly on GitHub (preview the markdown).
- [ ] You ran the manual review checklist below if you changed the workflow JSON.

## Local setup

You do not need a programming language toolchain to work on this repository. You need:

1. **An n8n instance** (Cloud or self-hosted) to import the workflow into. See [docs/getting-started.md](docs/getting-started.md).
2. **A text editor** with JSON syntax highlighting.
3. **Git**.

Optional but useful:

- `jq` for inspecting the workflow JSON from the command line.
- Docker, if you prefer self-hosting n8n via the official compose file.

Clone and open:

```bash
git clone https://github.com/ANI-IN/Intelligent-Client-Inquiry-Response-System.git
cd Intelligent-Client-Inquiry-Response-System
```

## Branching and PR conventions

- Branch from `main`. Use a descriptive name: `docs/troubleshooting-rewrite`, `fix/email-subject-encoding`, `feat/add-human-review-step`.
- One logical change per PR. Smaller PRs get reviewed faster.
- Commit messages follow [Conventional Commits](https://www.conventionalcommits.org/): `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, `test:`, `ci:`, `build:`, `perf:`.
- Reference any related issue in the PR body.
- Mark the PR as draft until you have completed the checklist above.

## Style guide

### Documentation

- Use ASCII punctuation only. No em dashes, no smart quotes inside code blocks.
- Headings are hierarchical (do not skip from H2 to H4).
- Code fences specify their language (` ```bash`, ` ```json`, ` ```mermaid`).
- Link text is descriptive ("see [the architecture overview](docs/architecture.md)"), never "click here".
- Tables align their pipes.

### Workflow JSON

- Keep node names human-readable; they appear in execution logs.
- Prefer Set nodes over Function nodes when the transformation is simple.
- Pull magic strings (model names, sender personas, subject templates) into a single Set node at the top of the workflow.
- Add error-output handling on every node that calls an external service.

## Manual workflow review checklist

Run through this list when you propose a change to `Intelligent Client Inquiry Response System.json`:

1. **No secrets.** Open the JSON in a diff viewer and confirm no key, token, password, or OAuth refresh token is present. Credential fields should reference IDs only.
2. **Credentials still resolve.** Import the modified JSON into your test n8n instance and verify each node shows its credential as configured, not "missing".
3. **Connections still form a valid graph.** Every node has at least one inbound edge except the trigger, and no node is orphaned.
4. **Expressions parse.** Click into each node that uses `={{ ... }}` and confirm the expression editor does not show a red error.
5. **Dry run on sample data.** Add a Manual Trigger temporarily or use the workflow's "Execute Workflow" button with a pinned data sample. Confirm the email is drafted to the expected address. If you cannot send, set the Gmail node to draft mode for the test.
6. **Error paths.** Disconnect the OpenAI credential temporarily and re-run. The workflow should fail loudly, not silently. If it silently swallows the failure, that is a bug.

## What is in scope

- Documentation, examples, troubleshooting guides.
- Hardening: input validation, error handling, rate-limit guards, secret hygiene.
- Refactors of the workflow JSON that reduce hidden state or magic values.
- New diagrams in `docs/architecture.md`.
- Sample data fixtures (anonymized).

## What is out of scope

- Adding new third-party integrations that change the surface area of the system without an accompanying threat-model update in [SECURITY.md](SECURITY.md).
- Breaking changes to the form schema without a migration note in [CHANGELOG.md](CHANGELOG.md).
- Committing real credentials or production sheet IDs.

## Reporting bugs

Use the [bug report template](.github/ISSUE_TEMPLATE/bug_report.md). Include the workflow execution ID if you have one. Redact any lead PII.

## Code of Conduct

Participation in this project is governed by the [Code of Conduct](CODE_OF_CONDUCT.md). Report violations privately through the channel listed in [SECURITY.md](SECURITY.md).

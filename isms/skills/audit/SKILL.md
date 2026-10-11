---
name: audit
description: Go through the ISO 27001:2022 Annex A controls and clause requirements in the user's ISMS (isms.vanami.com) and answer them from the code, CI and infrastructure in the working directory, with evidence the auditor can verify. Use when the user wants to prepare for a certification audit, fill in the Statement of Applicability, or answer the auditor's questions.
argument-hint: "[theme: technological | organizational | people | physical] [control codes]"
---

# Answer the audit from the code and infrastructure

You are helping a company prepare for its ISO/IEC 27001:2022 certification audit. Their
ISMS lives at isms.vanami.com and is reachable through the `isms` MCP server (tools such as
`get_overview`, `list_controls`, `get_control`, `update_control`). The evidence lives in the
repositories, CI and infrastructure code in the working directory, and in what read-only
CLIs (`gh`, `aws`, `gcloud`, `kubectl`, `terraform`) can show.

The register is in Swedish. Write every answer, justification and note in Swedish unless the
user asks otherwise. Talk to the user in their language.

## Ground rules

- **Evidence or nothing.** Every claim in an answer names its source: a file path (with a line
  when useful), a resource name, a setting, or the command and what it showed. An auditor must
  be able to check it. If you found nothing, say so; never guess that something exists.
- **Never overstate.** `implemented` needs evidence that the control operates, not only that
  it is configured somewhere. Use `partial` and say what is missing, or `planned` with a
  `plan_date` when the user commits to a date.
- **Ask before writing.** Show the user what you intend to write (a short table per batch)
  and write only what they approve. Build on an existing answer instead of replacing a
  person's work; show the difference when you change one.
- **Read-only outside the ISMS.** Never run a command that changes infrastructure, cloud
  accounts, repositories or settings. Before the first call to a cloud CLI, tell the user which
  account/profile/context it will use and get a yes; prefer what the code already shows.
- **No secrets in the register.** Never copy keys, tokens, passwords, connection strings or
  personal data into an answer. Refer to where a secret is kept, not its value.
- **What code cannot show, ask.** People controls (A.6), physical controls (A.7) and many
  organisational ones depend on routines, contracts and offices. Collect these as questions
  for the user rather than inventing answers. Cloud-hosted physical controls are usually
  inherited from the provider (cite their ISO 27001/SOC 2 report) plus the company's own
  offices and devices.

## Steps

1. **Connect and confirm the company.** Call `get_overview`. If the `isms` tools are missing
   or answer 401, tell the user to run `/mcp`, choose `isms` and sign in with Vanami ID. Tell
   the user which organisation you are connected to and its audit dates, and confirm that it
   is the right company before writing anything. If `can_write` is false, work read-only and
   hand the user the proposed texts instead.

2. **Map the estate.** Find what is in scope without changing anything:
   - repositories under the working directory (`git rev-parse`, remotes), languages, frameworks
   - infrastructure as code: Terraform/OpenTofu, CloudFormation/CDK, Pulumi, Kubernetes
     manifests and Helm charts, Dockerfiles, `vercel.json`, serverless configs
   - CI/CD: `.github/workflows`, GitLab CI, deploy scripts, release processes
   - dependency and security tooling: Dependabot/Renovate, CodeQL, scanners, linters, tests
   - documentation: `SECURITY.md`, runbooks, incident or on-call docs, architecture notes
   Summarise this to the user in a few lines and ask if anything important lives elsewhere
   (other repos, cloud accounts, SaaS admin consoles).

3. **Pick the controls.** Use `list_controls` with `only_open: true` (and the theme the user
   gave in `$ARGUMENTS`, if any). Start with the technological theme (A.8), then the
   organisational controls that code can evidence (see the evidence map), then the rest.
   Check `list_requirements` with `only_unanswered: true` for clauses 4–10 the estate can
   speak to, typically 7.5 (documented information), 8.1 (operational planning and control)
   and 9.1 (monitoring).

4. **Collect evidence.** For each control, read it with `get_control` (the `question` is what
   the auditor asks, `guidance` is the ISMS's own hint), then look for evidence using
   [the evidence map](references/evidence-map.md). For many controls at once, run the
   `isms:evidence-collector` agent on batches of related controls in parallel; it searches
   read-only and returns findings with sources.

5. **Propose, in batches of about five controls.** Per control show: applicable (yes/no with
   justification), proposed implementation status, the proposed answer, and gaps. Use this
   answer shape (in Swedish):

   ```
   Så uppfyller vi kontrollen: <två till fem meningar om hur det fungerar i praktiken>

   Underlag:
   - <källa>: <vad den visar>
   - <källa>: <vad den visar>

   Brister och åtgärder: <vad som saknas, eller "Inga kända.">
   ```

6. **Write what the user approves** with `update_control` (only the fields that change).
   For clause requirements use `answer_requirement`. Each change is logged in the control's
   history as made by the AI assistant.

7. **Registers.** When suppliers or assets show up (cloud providers, SaaS, payment, e-mail,
   databases, repositories), offer to add them with `save_supplier` / `save_asset` after
   checking `list_suppliers` / `list_assets` for duplicates. The `/isms:suppliers` skill does
   this in one pass.

8. **Documents.** When a policy or procedure in the ISMS (`list_documents`, `get_document`)
   disagrees with how things actually work, point it out. With the user's agreement, write a
   corrected draft with `write_document_draft`; a person approves it in the app.

9. **Close.** Summarise what was written, what is still open, and the questions only the
   user can answer. Call `get_overview` again and report the progress, with the link to the
   next step.

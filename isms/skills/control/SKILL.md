---
name: control
description: Answer one ISO 27001 Annex A control or clause requirement in the ISMS (isms.vanami.com) from the code and infrastructure, with evidence. Use when the user names a control such as A.8.13 or a clause such as 9.1, or pastes an auditor's question.
argument-hint: "<control or clause code, or the auditor's question>"
---

# Answer one control

`$ARGUMENTS` is a control code (A.8.13, 8.13), a clause code (9.1) or an auditor's question.

1. If it is a question rather than a code, use the `isms` server's `search` and
   `list_controls` to find the controls it concerns and confirm with the user.
2. Read the control with `get_control` (or the clause with `get_requirement`): the question,
   the ISMS's guidance, the current answer and status.
3. Collect evidence following the ground rules and the evidence map of the `audit` skill
   (`${CLAUDE_PLUGIN_ROOT}/skills/audit/references/evidence-map.md`): read-only, every
   statement with its source, no secrets, ask before the first cloud CLI call.
4. Propose applicability, implementation status and the answer in Swedish, in the answer
   shape of the `audit` skill. Show what changes compared with the current answer.
5. When the user approves, write it with `update_control` (or `answer_requirement`) and give
   the link to the control in the app.

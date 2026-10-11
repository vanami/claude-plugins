---
name: status
description: Show where the company's ISMS (isms.vanami.com) stands before the ISO 27001 audit — audit dates, progress towards stage 1 and 2, the next step and what is open. Use when the user asks how far they are, what is left, or what to do next.
---

# ISMS status

1. Call the `isms` MCP server's `get_overview`. If the tools are missing or unauthorised, tell
   the user to run `/mcp`, choose `isms` and sign in with Vanami ID.
2. Report, briefly and in the user's language:
   - the organisation and the stage 1 and stage 2 audit dates, with the days left
   - progress: stage 1 and stage 2 gates, SoA decisions, applicable controls with evidence
   - the next step, with its link
   - the open items for the nearest audit, grouped, at most ten, each with its link
3. Offer what this plugin can do about the open items: `/isms:audit` for controls the code and
   infrastructure can answer, `/isms:suppliers` for the registers. Point out items that need a
   person in the app (approving documents, management review, internal audit).

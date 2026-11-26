<!-- Copied from the original claude.ai project during the upgrade. Delete this line if you edit this file. If the upgrade is retried, it replaces files that still carry this line. -->

---
name: query-fan-out-generator
description: User's Query Fan-Out Generator tool project (this Project's copy) — goals and requirements
sources: [cowork]
aliases: [Query Fan-Out Generator]
---
- [stated] Building a Query Fan-Out Generator to find fan-out queries derived from a root query on AI search tools (ChatGPT, Gemini, Claude, AI Overviews)
- [stated] Wants the tool based on real data, combined with Otterly-style fan-out types (Related, Implicit, Reformulation, Personalized, Entity expansion)
- [stated] Wants the tool to run directly inside the Claude project "Query fan-out" whenever they open it
- [stated] Wants each root query also run through otterly.ai's Query Fan-Out tool automatically, with results merged into the generator's output
- [stated] Created a free SerpApi account and supplied its API key for the tool (to pull Google AI Mode, AI Overviews, Related, Autocomplete data)
- [stated] Subscribes to Claude Pro; their Gemini and ChatGPT accounts are free tier
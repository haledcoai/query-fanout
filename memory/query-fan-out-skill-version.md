---
name: query-fan-out-skill-version
description: Current intended rules of the /query-fan-out skill and the need to check which version is loaded before running it
metadata:
  type: feedback
  modified: 2026-10-08T04:26:42.373Z
---
On 2026-10-08 the user approved these rules for the query-fan-out skill and the Query Fan-Out Generator page (https://claude.ai/artifact/RsdqCpsMdV6uUsgnXw7EVU): predicted queries default 20, options 20/30/40/50 (the user rejected 10/15), predictions are a ceiling and only fill groups with 0-1 queries; page source kinds Thật / Google / Suy ngược / Otterly / Dự đoán; the year filter keeps units like đ and giờ and standard numbers like QCVN 09:2017. The page already has these changes. The proposal card never showed for the user, so they edit the skill by hand. As of 2026-10-08 they chose to change only the default (20, 20/30/40/50, "mức trần") and the SerpApi key path (/mnt/project-files/knowledge/claude/serpapi-api-key.txt), and declined the year-filter and Bước 2 gap-fill edits as unimportant. Respect that choice.

**Why:** a thread ran /query-fan-out with the old cached skill (default 30) without saying so, and the user caught it and was upset: "nếu tôi không phát hiện ra lỗi thì công cụ của tôi tạo ra vô ích".

**How to apply:** before running /query-fan-out, compare the loaded skill text with these rules. If it is still the old version, say so at the top of the result, and remind the user to save the latest proposal card and run it in a new chat. Since 2026-10-08 a cloud thread on the new environment gets Google data via SerpApi Cách A (server curl), proven by a full run for "đèn hồ bơi"; Otterly still has no browser or Gmail there, so say plainly that such a run has no Otterly data.

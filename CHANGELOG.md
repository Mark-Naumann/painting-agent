# Voice Agent Demos — Change Log

## Sam — Naumann Painting receptionist (Vapi)

**v8: edge-case rules + business facts (Nov 18)**
- Added rules: never guess at answers not in the business facts (stops hallucination); "Can I talk to Mark?" handling; wrong numbers and non-painting emergencies; callers who won't share details; weekend callers get a next-weekday callback.
- Fixed facts: full service list, Mon–Fri callback days, insured.
- Phone numbers are read back in small groups of digits, since voice AI often mishears numbers.

**v9: Spanish support + less robotic (Nov 18)**
- Transcriber now listens for English and Spanish; Sam stays in Spanish if the caller uses it.
- Sam no longer says "Let me repeat that back"; it just does it.
- Uses the caller's name twice per call at most, instead of in every reply.

**v11: more natural voice (Nov 18)**
- Moved from ElevenLabs Flash v2.5 (Vapi humanness rating 67) to Vapi Voices v2 "Clara" (rating 92), at normal speed (1.0) with office background sound.
- Prompt now tells Sam to use contractions and natural phrases ("sure," "got it," "no problem").

**v12–v14: phone numbers, one question per turn, "Nawman", call review (Nov 18, Mountain Time)**
- Asks exactly ONE question per turn, then waits; never combines questions or answers its own.
- New phone-number rules: exactly 10 digits (asks only for a missing area code), read back once digit by digit in words, in 3-3-4 groups, then "Is that right?"; the number is confirmed before anything else.
- Says the company and owner's name as "Nawman" (prompt and greeting).
- Transcriber custom vocabulary for local names (Naumann, O'Fallon, St. Peters, St. Charles, Wentzville); smart endpointing.
- Background sound turned off.
- "Call quality review" structured output attached: every call is scored and reviewed (see call-analyzer).

**Integration:** Vapi end-of-call report → Make webhook → **two emails**: (1) the lead (name, callback number, project, town, timeline, urgent flag, recording link); (2) the call review (score out of 10, rule checks, issues found, suggested prompt fix for a person to review). Only end-of-call reports are sent to the webhook, which avoids paying for extra Make runs.

**Known Vapi issue (Nov 18):** publishing from the assistant editor silently detaches structured outputs that were attached from the structured output's own page. Caught by a blank review email; workaround: re-attach after each publish.

## Automated tests (Vapi Evals, AI-judged). All passed Nov 18.

| Test | Agent | Pass rule | Fail rule |
|---|---|---|---|
| Never quotes a price | Sam | Declines to give a number, offers a free estimate | Any dollar figure or range |
| Asks for Mark directly | Sam | Calm; says Mark will call back; collects info | Pretends to transfer the call, argues, or hangs up |
| Spanish-speaking caller | Sam | Replies fully in Spanish, one question | Replies in English, quotes a price, or stacks questions |

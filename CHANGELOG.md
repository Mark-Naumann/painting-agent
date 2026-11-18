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

**Integration:** Vapi end-of-call report → Make webhook → lead email (name, callback number, project, town, timeline, urgent flag, recording link). Only end-of-call reports are sent to the webhook, which avoids paying for extra Make runs.

## Automated tests (Vapi Evals, AI-judged). All passed Nov 18.

| Test | Agent | Pass rule | Fail rule |
|---|---|---|---|
| Never quotes a price | Sam | Declines to give a number, offers a free estimate | Any dollar figure or range |
| Asks for Mark directly | Sam | Calm; says Mark will call back; collects info | Pretends to transfer the call, argues, or hangs up |
| Spanish-speaking caller | Sam | Replies fully in Spanish, one question | Replies in English, quotes a price, or stacks questions |

# Sam — AI Phone Receptionist for Naumann Painting

Demo built for my dad's painting company in St. Peters, MO (the owner, Mark Naumann, is my
dad). Not yet on the business's real phone line. Putting it there is the next step.

**Sam is an AI phone receptionist built for a small painting company: it answers the call,
takes the lead, handles the awkward calls a human receptionist would, and emails the owner
a clean summary.**

🎬 **Demo video:** coming soon

**Current version:** Sam **v14** on Vapi (Nov 18, 2026, Mountain Time) · **Call review:** every call is graded automatically ([call-analyzer](https://github.com/Mark-Naumann/call-analyzer))

---

## What it does (from the caller's side)

When you call Sam's test line (not the business's real number), Sam picks up for Naumann
Painting (pronounced "Nawman", and Sam says it that way), sounding like a friendly
front-desk person, and:

1. Asks **exactly one question per turn**: what you need painted (interior, exterior, cabinets,
   trim, deck…), where (address or town), when you'd like it done, how you heard about
   the company, and the best time to call you back.
2. Gets your name and callback number. The number must be **exactly 10 digits** (if a
   piece is missing, Sam asks only for that part), and Sam **reads it back digit by digit
   in words, in 3-3-4 groups**, then waits for "yes" before moving on.
3. Recaps your name, number and project in one sentence, says Mark will call back within
   24 hours for a free estimate, and thanks you.

A few minutes later, Mark gets **two emails**:

1. **The lead:** name, callback number, project, town, timeline, an **urgent** flag, and a
   link to the call recording.
2. **The call review:** an AI reviewer's score out of 10 for how Sam handled the call, the
   rules it checked, any issues found, and one suggested prompt fix, which a person reviews
   before anything changes (see [call-analyzer](https://github.com/Mark-Naumann/call-analyzer)).

### The tough calls Sam handles

| Caller says… | What Sam does |
|---|---|
| "How much will it cost?" | Never quotes a price. Offers a free estimate instead. |
| "Can I just talk to Mark?" | Stays calm, explains Mark will call back, and collects the details. No fake transfers. |
| Speaks Spanish | Switches to Spanish and stays in Spanish for the whole call. |
| Asks something not in the business facts (paint brands, lead paint…) | Doesn't guess. Says Mark will cover that on the callback (no hallucinated answers). |
| Existing customer with a problem | Apologizes, takes the details, and flags the lead **URGENT**. |
| Lives outside the service area | Takes their info anyway; Mark will say if he can take it on. |
| Sales call or vendor | Takes a name and number and ends politely. |
| Wrong number, or a non-painting emergency | Says so kindly and ends the call. |
| Won't share details | Doesn't push, but at least gets a name and callback number. |
| Calls Friday evening or on a weekend | Says Mark will call back the next weekday. |

## How it works

```
Caller
  │  phone call
  ▼
Vapi voice agent "Sam"
  ├─ Soniox speech-to-text (English + Spanish)
  ├─ GPT-4.1 (system prompt + business facts → what to say)
  └─ Vapi Voices v2 "Clara" (text-to-speech, normal speed)
  │
  │  end of call → 2 structured outputs:
  │     "painting lead" (name, number, project, town, timeline, urgent)
  │     "call quality review" (score, rule checks, issues, suggested fix)
  ▼
End-of-call report ──► Make webhook ──┬─► email 1: the lead, to the owner
                                      └─► email 2: the call review, to the owner
```

- **No custom server.** Everything runs on Vapi and Make (a no-code automation tool).
- **Cost-aware:** only the *end-of-call report* is sent to Make, not every call event,
  so each call costs one Make run.
- **Config as code:** the current Vapi setup is exported into this repo, so prompt and
  voice changes are reviewed and versioned like normal code.
- **Better hearing:** the transcriber has custom vocabulary for local names (Naumann,
  O'Fallon, St. Peters, St. Charles, Wentzville) and smart endpointing (knows when the
  caller has finished talking).
- **Known Vapi issue:** publishing from the assistant editor can silently detach the
  structured outputs. After every publish I re-attach them and check the next review email
  isn't blank ([incident write-up](https://github.com/Mark-Naumann/call-analyzer/blob/main/examples/03-incident-review-silently-detached.md)).

### What's in this repo

| Path | What it is |
|---|---|
| [`prompt.md`](prompt.md) | Sam's full system prompt (easy to read) |
| `vapi/assistant-sam.json` | The Vapi assistant config (model, voice, transcriber, settings) |
| `vapi/structured-output-lead.json` | The "lead" fields Vapi extracts at the end of each call |
| `vapi/structured-output-call-quality-review.json` | The AI call review (score, checks, issues, suggested fix) |
| `make/` | The Make scenario blueprint, 3 modules: webhook → lead email → call review email |
| `scripts/export-vapi.js` | Downloads the configs from the Vapi API (read-only) and **redacts** private info |
| `scripts/redact.js`, `scripts/redact.test.js` | The redaction rules and their tests (`npm test`) |
| `scripts/check-secrets.js` | Safety scan for keys, webhooks, emails and phone numbers |
| [`CHANGELOG.md`](CHANGELOG.md) | Version history |

Webhook URLs, emails, phone numbers, org IDs and credentials are replaced with
placeholders like `<MAKE_WEBHOOK_URL>` before anything is committed.

## Testing

Sam is tested with **Vapi Evals**: scripted test calls where an AI judge grades the
conversation pass/fail against a written rule. All passed on Nov 18.

| Test | Passes if Sam… | Fails if Sam… |
|---|---|---|
| Never quotes a price | Declines to give a number and offers a free estimate | Says any dollar figure or range |
| Asks for Mark directly | Stays calm, says Mark will call back, collects info | Pretends to transfer, argues, or hangs up |
| Spanish-speaking caller | Replies fully in Spanish, one question at a time | Replies in English, quotes a price, or stacks questions |

(These are 3 of the 5 evals across my two voice agents; the other 2 test Jordan in the
`transit-agent` repo.)

## Version history

See [`CHANGELOG.md`](CHANGELOG.md). Highlights:

- **v8:** edge-case rules (no guessing, "Can I talk to Mark?", wrong numbers, weekend callers); numbers read back in small groups.
- **v9:** English + Spanish; less robotic phrasing; caller's name used at most twice.
- **v11:** new voice, Vapi "Clara" (humanness rating 67 → 92), contractions and natural phrases.
- **v12–v14:** exactly one question per turn; 10-digit phone numbers read back in words; says "Nawman"; custom vocabulary; call quality review attached.

## What I'd do next

- **Go live:** put Sam on the business's real phone line.
- **Calendar booking:** let Sam offer real estimate time slots (e.g. Google Calendar) instead of only a callback.
- **CRM instead of email:** send leads into a CRM or spreadsheet so nothing gets lost in an inbox.
- **Text confirmation:** send the caller an SMS summary after the call.
- **More evals:** test noisy lines, callers who interrupt, and someone giving a wrong number then correcting it, and run the evals automatically after every prompt change.
- **Deploy from the repo:** a script that *pushes* `vapi/` changes back to Vapi, so the repo is the single source of truth (and re-attaches the structured outputs after every publish).
- **Turn call reviews into evals:** every real mistake the reviewer finds becomes a new Vapi Eval, so it can't come back.

## Setup (for developers)

```bash
cp .env.example .env      # then paste your Vapi PRIVATE key into .env
npm run export            # downloads + redacts the Vapi configs into vapi/ and prompt.md
npm test                  # tests for the redaction rules
npm run check             # safety scan before committing
```

Needs Node.js 18+. No packages to install. `.env` is in `.gitignore` and is never committed.

## Related projects

| Repo | What it is |
|---|---|
| [transit-agent](https://github.com/Mark-Naumann/transit-agent) | Jordan, my second voice agent (transit customer service, live bus tools) |
| [transit-sim](https://github.com/Mark-Naumann/transit-sim) | The simulated bus system and API Jordan's tools call |
| [call-analyzer](https://github.com/Mark-Naumann/call-analyzer) | The AI call quality review that grades every call, with findings from test calls |
| [call-analytics](https://github.com/Mark-Naumann/call-analytics) | Python + SQLite monitoring for both agents (quality, transfers, tool errors, dead air, cost) |

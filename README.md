# Sam — AI Phone Receptionist for Naumann Painting

**Sam answers every call for a small painting company, books the lead, handles the awkward
calls a human receptionist would, and emails the owner a clean summary before he's off the ladder.**

🎬 **Demo video:** _[link coming soon](#)_

---

## What it does (from the caller's side)

You call Naumann Painting. Sam picks up, sounding like a friendly front-desk person, and:

1. Asks, one question at a time, what you need painted (interior, exterior, cabinets,
   trim, deck…), where (address or town), when you'd like it done, how you heard about
   the company, and the best time to call you back.
2. Gets your name and callback number, and **reads the number back in small groups of
   digits** so mistakes get caught on the call.
3. Recaps your name, number and project in one sentence, says Mark will call back within
   24 hours for a free estimate, and thanks you.

A few minutes later, Mark gets an email with the lead: name, callback number, project,
town, timeline, an **urgent** flag, and a link to the call recording.

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
  └─ Vapi Voices v2 "Clara" (text-to-speech, office background sound)
  │
  │  end of call → structured output ("lead": name, number, project, town, timeline, urgent)
  ▼
End-of-call report ──► Make webhook ──► email to the owner
```

- **No custom server.** Everything runs on Vapi and Make (a no-code automation tool).
- **Cost-aware:** only the *end-of-call report* is sent to Make, not every call event,
  so each call costs one Make run.
- **Config as code:** the live Vapi setup is exported into this repo, so prompt and
  voice changes are reviewed and versioned like normal code.

### What's in this repo

| Path | What it is |
|---|---|
| [`prompt.md`](prompt.md) | Sam's full system prompt (easy to read) |
| `vapi/assistant-sam.json` | The Vapi assistant config (model, voice, transcriber, settings) |
| `vapi/structured-output-lead.json` | The "lead" fields Vapi extracts at the end of each call |
| `make/` | The Make scenario blueprint ("Vapi painting calls → email") |
| `scripts/export-vapi.js` | Downloads the configs from the Vapi API and **redacts** private info |
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

## What I'd improve next

- **Calendar booking:** let Sam offer real estimate time slots (e.g. Google Calendar) instead of only a callback.
- **CRM instead of email:** send leads into a CRM or spreadsheet so nothing gets lost in an inbox.
- **Text confirmation:** send the caller an SMS summary after the call.
- **More evals:** test noisy lines, callers who interrupt, and someone giving a wrong number then correcting it, and run the evals automatically after every prompt change.
- **Deploy from the repo:** a script that *pushes* `vapi/` changes back to Vapi, so the repo is the single source of truth.

## Setup (for developers)

```bash
cp .env.example .env      # then paste your Vapi PRIVATE key into .env
npm run export            # downloads + redacts the Vapi configs into vapi/ and prompt.md
npm run check             # safety scan before committing
```

Needs Node.js 18+. No packages to install. `.env` is in `.gitignore` and is never committed.

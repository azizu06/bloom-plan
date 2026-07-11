# Agent Handoff — CapCheck build day

You are picking up a fully planned project. This file plus the three docs in `docs/` is everything you need. Read them in this order before writing any code.

1. `docs/00-idea-decision.md`, why we're building CapCheck and what challenges we're targeting
2. `docs/capcheck-design.md`, the build spec, source of truth
3. `docs/capcheck-12h-plan.md`, the hour-by-hour clock, prep status, and failure playbook

## The one-paragraph mission

CapCheck fact-checks short-form finfluencer videos. Paste a TikTok or YouTube Shorts link, the server fetches the video with yt-dlp, uploads it to the Gemini Files API, Gemini watches it and extracts every claim, each claim gets verified through Google Search grounding plus a `get_stock_data` function backed by Finnhub, and the UI renders a Cap Score (0 to 100, "No cap" to "Full of cap") with claim cards, cited sources, trust tier badges, and a "what you can actually do" section. It is a 12-hour hackathon build (BloomKnights, UCF) targeting Bloomberg's Best FinTech Hack and Google's Best Use of Gemini at the same time. Team is two people, Aziz (owns the whole Gemini pipeline and yt-dlp ingestion) and one newer teammate (owns the UI, building against fixture JSON).

## Environment facts, all verified working on 2026-07-10

Everything below was actually tested end to end on Aziz's machine with his real keys, not assumed.

- yt-dlp 2026.07.04 installed via Homebrew. Full downloads verified for YouTube, a real `/shorts/` URL, and TikTok. Instagram Reels is confirmed blocked without login cookies, it is off the demo path, do not spend time on it.
- Gemini API key has billing enabled (paid tier). Files API video understanding, function calling, and Google Search grounding are all verified working with `@google/genai`.
- Finnhub free key verified with live quotes.
- Keys live in `.env` in this repo's local clone (gitignored, this repo is PUBLIC, never commit them). Copy the `.env` into the app repo at hour 0.

## API gotchas we already hit so you don't have to

- Model id is `gemini-flash-latest`. Do NOT use `gemini-2.5-flash`, it is closed to newly created API keys and returns 404.
- Gemini 3 function calling: when sending the function result back, include the model's function-call turn verbatim from `response.candidates[0].content`. Hand-rebuilding the parts drops a hidden `thought_signature` and the API rejects with 400.
- Shorts downloads can come out as `.webm`. Either remux to mp4 in the yt-dlp flags or pass the correct mimeType to the Files API, Gemini accepts both.
- TikTok extraction prints an impersonation warning. It works anyway. If TikTok hard-breaks, the fix is yt-dlp's curl-cffi dependency, and the fallback is the file upload path.
- Files API flow: upload, poll until `state === "ACTIVE"` (takes a few seconds), then reference the file in the prompt. A 19 second video went ACTIVE after one 2 second poll.
- Grounding citation URLs come back as `vertexaisearch.cloud.google.com` redirect links with the real domain in the chunk's title field. Display the title, link the redirect.
- Search grounding costs nothing at our scale, the paid tier includes 5,000 grounded prompts per month free. Token spend is the only real cost, roughly 2 to 3 cents per minute of video analyzed. Budget for the day is about 10 dollars, already funded.

## Build-day workflow (agreed with Aziz, follow it)

- The app repo does not exist yet. Create it fresh at hour 0, empty, nothing pre-written. This was a deliberate rules-safety decision.
- Work Matt Pocock style: break the plan into GitHub issues first, lay them on a kanban board (GitHub Projects), one issue = one branch = one PR into main, CI on every PR. Keep CI fast, lint + typecheck + build only, no slow test suites, or the merge queue becomes the bottleneck.
- Aziz runs no-mistakes on the repo for push safety. Ask him before initializing it, never init it unprompted.
- Hour 0 also freezes the three JSON contracts from the design spec (claims array, verification result, scorecard object) and writes fixture files for them, because the teammate's whole UI lane builds against those fixtures.
- Stack is locked: Next.js App Router, single repo, `@google/genai`, SSE for streamed progress, no database, pipeline server-side only, demo runs from localhost.
- Commits are authored solely as Aziz (azizu06 / ab725492@ucf.edu). No AI co-author trailers, no "generated with" lines.

## Still open on the human side

- Devpost: opt in to BOTH challenges (Bloomberg fintech, Google Gemini) and skim the pre-written-code rules. Aziz was going to do this the night before, confirm it happened.
- Demo videos: one legit candidate picked (Humphrey Yang, "5 Signs You're Doing Well Financially in 2026", 89s TikTok). Still need the scammy one and the mixed one, pick big accounts making checkable factual claims.

## Suggested skills (if the agent running this is Claude Code rather than Codex)

- `write-as-aziz` before writing any text posted under Aziz's name (PR bodies, Devpost write-up, issue comments)
- `superpowers:test-driven-development` only if time allows, this is a hackathon, working demo beats coverage
- `gh-axi` CLI for GitHub operations instead of raw `gh`

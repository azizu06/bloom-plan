# CapCheck Design Spec

Paste a short-form finfluencer video link, get a cited credibility scorecard. This doc is the source of truth for what we're building. The hour-by-hour schedule lives in `capcheck-12h-plan.md`. Decisions locked in the planning grill on 2026-07-10 are folded in below.

## Identity

CapCheck is built for short-form content. TikTok, YouTube Shorts, and Reels are where people actually digest money advice now, so pasting a link to one of those is the primary flow, not a nice-to-have. Short videos also work in our favor technically, since a 15 to 90 second clip is cheap in tokens and keeps the demo loop fast.

## User flow

1. User lands on a single page and pastes a TikTok or YouTube Shorts URL. A file upload zone sits below it as the quiet fallback. Instagram Reels is best effort since it often demands login cookies.
2. A progress strip streams what's happening. Fetching video, watching it, extracting claims, checking claim 3 of 7, and so on. This is not cosmetic, streamed progress is what makes the pipeline legible to judges.
3. Results render as a scorecard. The headline is the **Cap Score**, 0 to 100, with a verdict label that scales from "No cap" through "Some cap" to "Full of cap". Below it, one card per claim with a verdict badge (true / mostly true / unverifiable / false), the evidence, and source links, each source tagged with a trust tier badge.
4. Below the claims, a hype language section highlights persuasion tactics in the transcript ("guaranteed", "everyone is buying", urgency framing, etc.).

## Claim scope

All money advice is in scope, not just stocks. Search grounding verifies any financial claim broadly, and the prompt steers it toward authoritative sources first (IRS, SEC, FINRA, CFPB, investor.gov, major finance outlets). Each cited source gets a trust tier from the model (regulator or government, major outlet, blog or unknown) and the UI shows that as a badge. Ticker claims additionally get hard numbers through the market data function. Contested advice comes back as unverifiable with a note on what sources disagree about, which reads as honesty, not weakness.

## Architecture

Next.js app (App Router), one repo, deployed on Vercel or run locally for the demo.

- **Frontend.** One page. Upload zone, progress strip, scorecard. Tailwind, keep it clean, dark theme reads well on a projector.
- **API route(s).** A single `/api/analyze` route that orchestrates the pipeline and streams progress events back (Server-Sent Events or a streamed response; SSE is simplest).
- **No database.** Results live in memory / client state. A shareable-link feature is a stretch goal and would just be a JSON blob written to disk or Vercel KV.

Keep the pipeline server-side so the API key never touches the client.

## The Gemini pipeline

Four stages. Each stage is a separate, testable function so we can demo partial progress even if a later stage breaks.

### Stage 1: ingest

URL first. The server runs yt-dlp to fetch the video from a TikTok or YouTube Shorts link, then uploads it to the Gemini Files API, waits for the file to become ACTIVE, and passes the file reference into the first prompt. Direct file upload uses the same path minus the yt-dlp step and stays in the UI as the fallback if a link fails live. Because yt-dlp needs a real server process, the demo runs from localhost, not a serverless deploy, which is normal at a hackathon.

### Stage 2: claim extraction

One generateContent call with the video file and a structured-output schema. Ask for the transcript plus an array of claims, where each claim has

- `text` (the claim as stated)
- `timestamp` (when in the video)
- `type` (factual | predictive | opinion)
- `checkable` (bool)
- `quant` (optional: ticker, metric, value, period, for claims with numbers in them)

Use the current Flash model for speed (check ai.google.dev/gemini-api/docs/models for the exact id on hack day, model names churn). Opinions get labeled as opinions and skipped by verification, that distinction itself demos well.

### Stage 3: per-claim verification

For each checkable claim, run a verification call. Two tool paths, and note that Gemini now supports built-in tools and custom function calling in the same request, which is exactly the depth the Gemini judges want to see.

- **Google Search grounding** for qualitative claims ("the company lost its CFO last month"). The response comes back with grounding metadata and citation URLs, which we surface as evidence links.
- **Function calling** for quantitative claims. We declare something like `get_stock_data(ticker, metric, period)` backed by a free market data API. Finnhub has a free tier with real-time quotes, yfinance works as a no-key fallback through a tiny Python sidecar or a JS equivalent like yahoo-finance2. The model calls the function, we execute it, feed the result back, and the model produces a verdict grounded in the actual number.

Each verification returns `verdict` (true | mostly-true | unverifiable | false), `confidence`, `explanation`, `sources[]`. Run claims concurrently with a cap of 3 or 4 in flight so we don't trip rate limits.

### Stage 4: scorecard synthesis

One final call takes the transcript plus all verdicts and produces the overall credibility score (0 to 100 with a one-line rationale) and the hype language analysis, with the manipulative phrases quoted so the UI can highlight them in the transcript.

## Scoring logic

Weight false claims heaviest, unverifiable claims lightly, and factor in the ratio of predictive claims to factual ones (a video that is all predictions and no facts should score low even with nothing provably false). Let the model propose the score but compute a deterministic floor/ceiling from the verdict counts in code so the number is defensible when a judge asks.

## Error handling

- Files API upload fails or stalls: retry once, then surface a clear error. Never a silent spinner.
- A single claim verification fails: mark that claim unverifiable with an "analysis failed" note and keep going. One bad claim must not kill the scorecard.
- Rate limits: the in-flight cap above, plus exponential backoff on 429s. We run on a paid API key with a 5 to 10 dollar cap and a billing alert set, so quota should never be the thing that kills a run.
- yt-dlp fails on a link (region lock, private video, TikTok being TikTok): surface "couldn't fetch this link" and point at the upload fallback. In the demo we just switch to the next prepared link.
- Demo insurance: three pre-downloaded videos in the repo, plus their cached pipeline outputs as JSON fixtures behind a `?demo=1` flag. If the venue wifi dies mid-presentation we replay the cached run through the same UI.

## What we say on stage

Open with the stat (some large share of Gen Z gets investing advice from TikTok, find the exact figure and cite it). Paste a real viral "this stock will 10x" video link live. Real videos from big accounts making checkable claims, no small creators, and the answer to "what if you're wrong" is already on screen since every verdict shows its sources. Close with a vision slide of the real integration future, results overlaid inside the short-form app itself so you never leave the feed, and the line that Bloomberg built the terminal that lets professionals trust market information, and CapCheck does that for the feed where retail investors actually live.

## Stretch goals, strictly in this order

1. Thin browser extension. One button on a YouTube Shorts page that grabs the current URL and opens CapCheck with it pre-filled. Under an hour once the app works, makes the vision slide feel real. No in-page overlay, that's the vision slide's job.
2. Spoken verdict. One button that reads the scorecard summary out loud through Gemini's audio output. Cheap to add, borrows the wow of the voice-terminal idea.
3. Shareable scorecard links.

## Out of scope

Accounts, history, mobile app, official TikTok API integration, full in-page overlay extension. None of it matters in 12 hours.

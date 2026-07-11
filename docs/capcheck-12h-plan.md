# CapCheck 12-Hour Execution Plan

Two lanes so we don't block each other. Aziz owns Lane A, the whole Gemini pipeline plus yt-dlp ingestion, since he's done the API and agent work before. Lane B is the app shell and scorecard UI, built against fixture JSON with heavy AI-tool assist, so the pipeline never blocks the UI and the UI never blocks the pipeline. The contract between the lanes is the JSON shapes in the design spec (claims array, verification result, scorecard object). We agree on those in hour 0 and freeze them, because the fixtures Lane B builds against have to stay true.

## Before hack day (do this at home, costs an evening; learning and throwaway scripts only, no app code before hour 0)

- [ ] Both of us get Gemini API keys. Aziz puts a card on his with a 5 to 10 dollar cap and a billing alert, and runs one hello-world generateContent call.
- [ ] Aziz runs one video through the Files API end to end in a throwaway script, upload, wait for ACTIVE, ask a question about the video. This is the riskiest integration and it should not be a surprise at hour 1. The script does not enter the repo.
- [ ] Install yt-dlp and confirm it pulls one TikTok link and one YouTube Shorts link on our machines.
- [ ] Pick and download 3 demo videos, one obviously scammy "10x guaranteed" video, one legit educational one, one mixed. Real videos from big accounts making checkable factual claims, no small creators. Scammy + legit contrast makes the demo land.
- [ ] Sign up for a Finnhub free key, confirm we can pull a quote.
- [ ] Confirm the double-dip is legal, i.e. opt in to both challenges on Devpost, and skim the rules on pre-written code so we stay clean.

## The clock

| Hours | Lane A, Aziz (pipeline) | Lane B (app + UI) |
|---|---|---|
| 0–1 | Repo scaffold together. Next.js, env keys, freeze the JSON contracts, write the fixture files. | Same, together. |
| 1–3 | Stage 1 for real. yt-dlp fetch from a pasted TikTok or Shorts URL, upload to Files API, one video analyzed end to end, however ugly. File-upload fallback path included since it's the same code minus yt-dlp. | Page skeleton. URL paste box front and center, upload zone below it, SSE progress strip wired to a fake event stream. |
| 3–5 | Stages 2 and 3. Claim extraction with the structured schema, then per-claim verification with search grounding (trusted-source steering, source trust tiers) and the `get_stock_data` function. Test on the scammy demo video. | Scorecard UI against fixtures. Cap Score header with the No cap / Some cap / Full of cap label, claim cards, verdict badges, evidence links with trust tier badges. |
| 5–7 | Stage 4 synthesis plus the deterministic score floor/ceiling. Concurrency cap and 429 backoff. | Hype-language transcript highlighting, loading and error states, dark theme polish. Wire the real pipeline in as stages come online. |
| 7–9 | Hardening. Run all 3 demo videos through, cache their outputs as the `?demo=1` fixtures. Fix whatever breaks. | Same, plus projector check, font sizes up, animation on the Cap Score reveal. |
| 9–10 | Stretch, only if the core is boring-stable. Order is the thin browser extension, then spoken verdict, then share links. | Vision slide for the pitch, the results-inside-the-app mockup. |
| 10–12 | Devpost write-up and pitch. Mirror the rubric words, originality, impact, technical difficulty for Bloomberg, and depth of Gemini integration for Google. 90-second demo script, one run-through out loud, screen-record a full backup demo. | Same, together. |

## Demo script (90 seconds)

1. Hook stat about where retail investors get advice now. One sentence.
2. Paste a real TikTok or Shorts link live, narrate the progress strip while it runs ("it fetched the video, Gemini is watching it, pulling claims, checking each one against search and live market data").
3. Cap Score lands. Read one false claim and its cited source out loud. That's the moment, and it's also the answer to "what if you're wrong", the sources are on screen.
4. Flip to the legit video's cached result to show it's not just a scam-detector that says no to everything.
5. Vision slide, results overlaid inside the short-form app itself, then close with the Bloomberg terminal line from the design doc.

## Failure playbook

- yt-dlp fails on a link live: paste the next prepared link, or use the upload fallback with the pre-downloaded file. Three videos, two input paths.
- Files API acting up on venue wifi: switch to `?demo=1` cached runs, the UI is identical.
- Claim extraction returning garbage on a video: fall back to a different demo video.
- Function calling flaking: verification still works on search grounding alone, quantitative claims just come back unverifiable. Degraded but demoable.
- Way behind at hour 7: cut stage 4 synthesis, compute the Cap Score purely in code from verdict counts, skip hype analysis. The claim cards alone still win the room.

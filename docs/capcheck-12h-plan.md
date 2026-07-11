# CapCheck 12-Hour Execution Plan

Two lanes so we don't block each other. Lane A owns the Gemini pipeline, Lane B owns the app shell and scorecard UI. The contract between the lanes is the JSON shapes in the design spec (claims array, verification result, scorecard object). Agree on those in hour 0 and both lanes can move independently, with the UI building against fixture JSON until the real pipeline is ready.

If we end up with more than two people, the third person takes demo prep and the Devpost write-up starting around hour 4, and the fourth pairs on the pipeline since that's where the risk is.

## Before hack day (do this at home, costs an evening)

- [ ] Everyone gets a Gemini API key and runs one hello-world generateContent call.
- [ ] One of us runs a video file through the Files API end to end, just to see the upload-then-ACTIVE flow once. This is the single riskiest integration and it should not be a surprise at hour 1.
- [ ] Pick and download 3 demo videos (one obviously scammy "10x guaranteed" video, one legit educational one, one mixed). Scammy + legit contrast makes the demo land.
- [ ] Sign up for a Finnhub free key, confirm we can pull a quote.
- [ ] Confirm the double-dip is legal, i.e. opt in to both challenges on Devpost.

## The clock

| Hours | Lane A (pipeline) | Lane B (app + UI) |
|---|---|---|
| 0–1 | Repo scaffold together. Next.js, env keys, agree the JSON contracts. Then A proves Files API upload + one video analyzed end to end, however ugly. | Page skeleton, upload zone, SSE progress strip wired to a fake event stream. |
| 1–4 | Stage 2 and 3. Claim extraction with the structured schema, then per-claim verification with search grounding and the `get_stock_data` function. Test on the scammy demo video. | Scorecard UI against fixture JSON. Claim cards, verdict badges, evidence links, overall score header. |
| 4–7 | Stage 4 synthesis plus the deterministic score floor/ceiling. Concurrency cap and 429 backoff. | Hype-language transcript highlighting, loading and error states, dark theme polish. Wire the real pipeline in as A's stages come online. |
| 7–9 | Hardening. Run all 3 demo videos through, cache their outputs as the `?demo=1` fixtures. Fix whatever breaks. | Same, plus projector check, font sizes up, animations on verdict reveal. |
| 9–10 | Stretch, only if the core is boring-stable. Order is URL ingestion, then spoken verdict, then share links. | Same. |
| 10–12 | Devpost write-up and pitch. Mirror the rubric words in the write-up, originality, impact, technical difficulty for Bloomberg, and depth of Gemini integration for Google. 90-second demo script, one run-through out loud, screen-record a full backup demo. | Same, together. |

## Demo script (90 seconds)

1. Hook stat about where retail investors get advice now. One sentence.
2. Paste the scammy video, narrate the progress strip while it runs ("it's watching the video, pulling claims, checking each one against search and live market data").
3. Scorecard lands. Read one false claim and its source out loud. That's the moment.
4. Flip to the legit video's cached result to show it's not just a scam-detector that says no to everything.
5. Close with the Bloomberg terminal line from the design doc.

## Failure playbook

- Files API acting up on venue wifi: switch to `?demo=1` cached runs, the UI is identical.
- Claim extraction returning garbage on a video: fall back to a different demo video, we have three.
- Function calling flaking: verification still works on search grounding alone, quantitative claims just come back unverifiable. Degraded but demoable.
- Way behind at hour 7: cut stage 4 synthesis, compute the score purely in code from verdict counts, skip hype analysis. The claim cards alone still win the room.

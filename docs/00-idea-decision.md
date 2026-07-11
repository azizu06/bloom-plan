# Why CapCheck

We're entering two challenges with one project. Bloomberg's Best FinTech Hack, which they judge on originality, potential for impact, and technical difficulty, and Google's Best Use of Gemini, which is judged on how deeply the Gemini API is integrated, not just whether it's present.

The strategy is to pick a project where a Gemini-only capability IS the fintech innovation, instead of a fintech app with AI sprinkled on top. That way one build scores maximum on both rubrics at once.

## The pick

**CapCheck, a fact-checker for financial influencer videos.** You give it a TikTok or YouTube video of someone giving stock or money advice. Gemini watches the video natively, pulls out every factual and predictive claim, verifies each claim against Google Search grounding plus live market data through function calling, and returns a cited credibility scorecard. Claim by claim, true / unverifiable / false, with sources, plus an analysis of the hype language and persuasion tactics used in the video.

## Why this over the alternatives

We scored six ideas against both rubrics. Short version of the board:

| Idea | Originality | Impact | Difficulty | Gemini depth | 12h feasibility | Demo wow |
|---|---|---|---|---|---|---|
| **CapCheck (finfluencer fact-checker)** | 5 | 4 | 4 | 5 | 4 | 5 |
| Open Terminal (voice-first market terminal) | 4 | 5 | 5 | 5 | 3 | 5 |
| ScamShield (multimodal fraud triage) | 4 | 5 | 3 | 4 | 4 | 4 |
| NewsShock (headline to portfolio impact) | 3 | 4 | 4 | 4 | 4 | 4 |
| Filing Time Machine (10-K diff analyst) | 4 | 3 | 3 | 4 | 5 | 3 |
| CashPilot (small-biz cashflow copilot) | 3 | 4 | 3 | 4 | 5 | 3 |

CapCheck won because it's the most original of the six, the demo is the kind judges remember, and its riskiest part (getting video in) has a guaranteed fallback through file upload. The voice terminal was the runner-up but realtime audio streaming is the highest-risk thing you can attempt in 12 hours, so we keep a small voice feature as a stretch goal instead.

## Why it fits the Bloomberg rubric

Retail investors getting burned by social media financial advice is a real and regulator-acknowledged problem, so the impact story writes itself. Nobody else at the event will have this. And the pipeline (video, then claim extraction, then per-claim verification agents, then market data cross-checks) reads as genuinely hard, because it is. It's also basically Bloomberg's own thesis. They built the terminal that turns messy market information into a trustworthy signal for professionals. We're doing that for the feed where retail investors actually live.

## Why it fits the Gemini rubric

Three capabilities that only Gemini offers, all in one pipeline. Native video understanding, Grounding with Google Search for cited verification, and function calling into a stock price API so quantitative claims like "NVDA is up 40% this year" actually get checked against numbers.

## Things we deliberately avoid

Budget trackers, thin chat wrappers, stock price predictors, and crypto apps. Every fintech track drowns in those and they fail the originality bar before the demo starts.

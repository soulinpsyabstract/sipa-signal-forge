# SIPA Signal Forge

Self-hosted personal AI agent — fine-tuned LoRA adapters served on your own Nebius GPU via vLLM, not a rented API you don't control.

Built for the **Nebius x NVIDIA Global AI Hackathon** (Personal AI track).

## What it does

Most "personal AI agent" demos are a thin wrapper over someone else's hosted model. Signal Forge does the opposite: it fine-tunes a small open model (LoRA, not full fine-tune — cheap, fast, reversible) on your own data and behavior, then serves it from your own GPU instance, with an OpenAI-compatible API in front of it. You own the weights, you own the box, you own the failure modes.

The base of the pipeline is a validated result: an 8-stage sequential LoRA "tune-on-tune" chain on Hermes-3-8B, evaluated end-to-end (100% held-out / 99.75% adversarial pass rate on a 10-item honesty/safety gate, after a judge bug that had been silently hiding real regressions was found and fixed). That's the model this project deploys — not a toy.

## How we built it

- **Fine-tuning**: LoRA adapters trained with Axolotl-style configs, base model `NousResearch/Hermes-3-Llama-3.1-8B`.
- **Serving**: `vllm/vllm-openai` container on a Nebius GPU Container VM (`gpu-l40s-a`, NVIDIA L40S, 48GB VRAM) — `--enable-lora --lora-modules` loads the adapter straight from its Hugging Face repo, no manual merge step. Sub-2-second response latency in testing.
- **Safety layer**: a consequence-gate that hard-blocks irreversible actions (mass deletion, unbounded financial transfers) regardless of model output — adversarially reviewed by a teammate who found and helped us close three real bypasses (salami-slicing under a threshold, chunking under a count limit, one missing invariant entirely).
- **Signal extraction**: responses are run through a deterministic filler-stripper (`sipa-signal`) that separates real claims from hedging/throat-clearing before the agent commits to an action — rule-based, not another model judging prose.

## Team

| Who | Role |
|---|---|
| Aelin AquaSoul ([@soulinpsyabstract](https://github.com/soulinpsyabstract)) | Model fine-tuning, Nebius GPU self-host deployment, project lead |
| Benjamin Hong ([@BenjaminJKHong](https://github.com/BenjaminJKHong)) | Adversarial security review of the consequence-gate |
| Muhammad Abdullah ([@muhammadabdullah071](https://github.com/muhammadabdullah071)) | Demo, documentation, presentation |
| Gephel Chingtham ([@GephelChingtham](https://github.com/GephelChingtham)) | Testing, evaluation support |

## License

Apache 2.0.

## Project plan

**Submission deadline: October 30, 10:00 AM PT** (~45 days from kickoff on Sep 15).

### Timeline

| Dates | Milestone |
|---|---|
| Sep 15 | Kickoff — done. Repo created, README pushed, team invited, GPU self-host pilot proven live (1.44s inference on Nebius L40S). |
| Sep 16–22 | Core build week 1 — persistent inference endpoint, consequence-gate wired in front of it, start of the adversarial security pass. |
| Sep 23–29 | Core build week 2 — close security findings, build the eval set for the deployed endpoint, first pass at demo script/storyboard. |
| Sep 30–Oct 6 | Integration — everything working end-to-end on one running deployment. **Scope freeze** — no new features after this week. |
| Oct 7–13 | Demo + docs — record the ≤3-min demo video, finalize README setup instructions, polish the Devpost story. |
| Oct 14–20 | Buffer week — deliberately empty, absorbs slippage from earlier weeks. No new work scheduled here. |
| Oct 21–27 | Dry run — full run-through of the demo as if judging it. Fix anything that breaks on a clean environment. |
| Oct 30 | Submit — at least 24h before the actual cutoff, not at the deadline itself. |

### Task board

**Aelin** — model + infra
- [x] Prove GPU self-host works (vLLM + LoRA on Nebius)
- [x] Repo, README, licensing, invites
- [ ] Stand up a persistent (not throwaway) inference deployment
- [ ] Wire consequence-gate in front of the endpoint
- [ ] Write the pre-existing-project disclosure (see risks)
- [ ] Coordinate weekly check-ins

**Benjamin** — security review
- [ ] Adversarial pass on the consequence-gate
- [ ] Document each bypass found + the fix, in the repo
- [ ] Re-test after fixes land (regression pass)

**Muhammad** — demo + docs
- [ ] Storyboard the ≤3-min demo video
- [ ] Record + edit the demo
- [ ] Polish README setup instructions for a stranger to follow
- [ ] Final pass on the Devpost project page copy

**Gephel** — testing / QA
- [ ] Build out the eval set for the deployed endpoint
- [ ] Run the eval, log pass/fail with real numbers
- [ ] Fresh-environment test of README setup steps

### Submission checklist

| Requirement | Status | Note |
|---|---|---|
| Category / track | Done | Personal AI |
| Public repo, OSS license | Done | Apache 2.0, sipa-signal-forge |
| README with setup instructions | Pending | Draft exists, needs a stranger-tested pass (Muhammad) |
| Working demo URL | Pending | Kickoff-night endpoint was torn down after the pilot — needs a persistent deployment |
| ≤3-min demo video w/ audio | Not started | Muhammad, week of Oct 7 |
| Project description / story | Draft done | In the Devpost kit |
| Feedback on Nebius/NVIDIA tools used | Not started | Write from real friction points (per-token LoRA deprecation, port/security-group setup) |
| Pre-existing-project disclosure | Not started | Required — see risks |
| City Winner eligibility (Builders & Brews) | Unclear | See risks |

### Open risks / decisions

- **Pre-existing project disclosure is mandatory, not optional.** EXP-042 (the base LoRA chain) was built before the Aug 26 submission window opened. Hackathon rules require a written explanation of what was significantly updated *during* the submission period — the self-host deployment, the consequence-gate integration, and the security review all qualify, but this needs to be stated explicitly in the submission, not left implicit in the story text.
- **City Winner ($500) eligibility is unresolved.** The prize is for people who attended an in-person Builders & Brews event. Aelin did not attend Tel Aviv on Sep 15 in person. If a teammate or contact attended instead, that likely does not transfer eligibility — worth confirming with organizers rather than assuming either way.
- **The kickoff-night GPU deployment was deleted after the test.** The demo needs a deployment that's actually running when judges look at it — decide before week 2 whether that means keeping a Nebius instance up continuously (real cost) or having a fast, reliable "spin up on demand" script.


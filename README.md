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

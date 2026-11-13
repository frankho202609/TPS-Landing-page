# LLM Parameter Audit

Last audited: 2026-11-13

The production options in `index.html` were checked against the publishers' official Hugging Face model repositories, model cards, and `config.json` files.

## Field policy

- `data-pb`: parameters represented by the distributed checkpoint, in billions. This drives weight-memory estimation. For multimodal or conditional-memory models, it includes checkpoint components that consume memory even when they are not active during text decode.
- `data-apb`: parameters active during autoregressive decode, in billions. This drives the compute side of the TPS roofline. Dense models use their advertised language-model size; MoE models use the publisher's activated-parameter figure.
- `data-layers`, `data-kvh`, `data-hd`: official text-decoder architecture fields used for KV-cache estimation.
- DeepSeek MLA/CED models use the published compressed KV dimension (`1 × 512`) rather than pretending they use ordinary GQA KV heads.
- Parameter labels such as `180B-A6B` mean checkpoint parameters / active decode parameters. They do not imply that all checkpoint parameters execute for every token.

## Primary sources

- Qwen: `https://huggingface.co/Qwen/<model>` and `https://github.com/QwenLM/Qwen3.8`
- Gemma: `https://huggingface.co/google/<model>` and Google Gemma model cards
- DeepSeek: `https://huggingface.co/deepseek-ai/<model>`
- NVIDIA Nemotron: `https://huggingface.co/nvidia/<model>` and NVIDIA technical model cards
- Meta Llama: `https://huggingface.co/meta-llama/<model>`
- Mistral: `https://huggingface.co/mistralai/<model>`
- Microsoft Phi: `https://huggingface.co/microsoft/<model>`
- NuminaMath: `https://huggingface.co/AI-MO/NuminaMath-7B-TIR`

## Important corrected cases

- Qwen3.8-Flash-Next: 125B language model with 6B activated, plus 51B N-gram embedding and 4B MTP (180B checkpoint tensors in total); 48 layers, 2 KV heads, 256 head dimension. The UI displays `125B-A6B`, while `data-pb="180"` intentionally retains the full no-offload memory footprint.
- DeepSeek-V4.1-Flash-1017: 552B backbone plus conditional-memory and auxiliary checkpoint weights (763.21B checkpoint tensors); 16B active during decode and 8B during prefill.
- DeepSeek-V4-Flash-0731: 284B backbone, 13B active; the released checkpoint contains 304.18B tensors including auxiliary/speculative weights.
- DeepSeek-V4-Pro-0813: 1.65T total, 49B active.
- Gemma 4 E4B: 8B checkpoint, 4B effective/active; Gemma 4 E2B: 5.12B checkpoint, 2B effective/active.
- Nemotron 3 Ultra: 560.53B checkpoint tensors, 55B active, 108 layers.

This estimator remains an architecture-planning model. Runtime engine, kernels, speculative decoding, prompt composition, thermal limits, and quantization implementation still require calibration on physical hardware.

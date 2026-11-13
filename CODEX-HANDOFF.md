# TPS Landing Page — Codex Handoff

## Project identity

- Production site: https://frankho202609.github.io/TPS-Landing-page/
- GitHub repository: https://github.com/frankho202609/TPS-Landing-page
- Production branch: `main`
- Packaging date: 2026-11-13 (Asia/Taipei)
- Production file: `index.html`

## Start here

This is a buildless single-page static site. No dependency installation is required for normal editing.

```powershell
python -m http.server 8131 --bind 127.0.0.1
```

Open `http://127.0.0.1:8131/`. Tailwind CSS, jQuery, and Inter load from public CDNs, so the complete preview needs internet access.

## Current product requirements

- Default language: Traditional Chinese. English and Simplified Chinese must remain functional.
- Default theme: dark. Light mode must keep exactly the same layout and change only theme colors, borders, and shadows.
- Configuration includes LLM, quantization, input length, speed filter, batch size, and concurrent users.
- Default speed filter: show all, including OOM.
- Hardware categories: Edge AI PC, Graphics Card, Laptop, Workstation.
- Default product presentation: card view, three products per desktop carousel page.
- Edge AI PC begins with EdgeMesa N AI+.
- Product cards display memory size, per-user TPS, TTFT, and VRAM/memory usage.
- References remain in a collapsed accordion.
- Footer brand description and Resources & Tools section were intentionally removed.

## LLM scope

- The selector intentionally contains 40 models: the top 10 downloaded Unsloth entries for each family, in this order: Qwen, Gemma, DeepSeek, NVIDIA Nemotron.
- Default: Qwen 3.8 27B.
- Other LLM families were deliberately removed.
- The list follows Hugging Face `unsloth` download rankings. Pure image/audio/embedding Qwen entries are excluded because the TPS calculator requires text generation; the next eligible text-capable Qwen entries are used instead.
- Popularity snapshot: 2026-11-13. Download rankings are dynamic; re-query Hugging Face before changing the order.

## TPS behavior

- Displayed throughput is per-user TPS: `aggregate TPS / concurrent users`.
- Effective batch: `min(batch size, concurrent users)`.
- Aggregate TPS uses a Roofline-style memory/compute estimate plus runtime overhead.
- All concurrent users reserve KV cache; effective batch affects active throughput.
- Speed filtering uses per-user TPS; the optional threshold is 20 TPS per user.
- The estimator is analytical and may differ from ApX or physical benchmarks.

## Inline TPS simulator

- `#tps-visualizer` is directly below product cards and pagination.
- It is a collapsed `<details>` panel by default.
- Selecting the TPS metric on a product card opens the panel, loads that hardware's TPS, and starts token streaming.
- Users can select a product TPS or type a custom TPS value. Replay, OOM handling, progress, manual collapse, and all three UI languages must continue working.

## Important implementation locations

Search inside `index.html` for:

- `interfaceCopy` — English, Traditional Chinese, Simplified Chinese UI strings
- `initializeHardwareCatalog` — hardware and MSI product-link data
- `renderProductPage` — filtering, category visibility, carousel, pagination
- `setHardwareCategory` — category tabs
- `calculateMetrics` — VRAM, TPS, per-user TPS, TTFT
- `sparkTpsCalibration` — RTX Spark calibration
- `tps-visualizer` / `runTpsSimulation` — inline token-stream simulator

## Validation checklist

1. Serve locally and confirm HTTP 200.
2. Check the browser console for JavaScript errors.
3. Verify desktop, tablet, and mobile have no horizontal overflow.
4. Test all four hardware tabs and card/list view.
5. Change model, quantization, input length, batch size, concurrent users, and memory size; confirm cards recalculate.
6. Confirm OOM and the per-user TPS filter work.
7. Confirm official MSI product links open correctly.
8. Confirm the simulator is closed by default, opens from a TPS metric, streams, replays, and collapses.
9. Confirm the model selector has 40 options (10 per requested family) and only the four requested family groups.
10. Test English, Traditional Chinese, and Simplified Chinese, including simulator text.

## Package contents

- `index.html` — production page
- `README.md` — repository overview
- `APPLICATION_SCENARIO_SOURCES.md` — retained research/source notes
- `CODEX-HANDOFF.md` — this handoff guide
- `SOURCE-COMMIT.txt` — exact Git commit packaged

## Publishing

Only `index.html` is required for GitHub Pages production.

```powershell
git add -- index.html
git commit -m "Describe the update"
git push origin main
```

Do not commit `node_modules`, `work`, `outputs`, `.codex-build`, `.codex-finalizer`, or generated ZIP files.

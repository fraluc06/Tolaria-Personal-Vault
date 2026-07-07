---
type: Note
_organized: true
---

# SVT-AV1 Essential (nekotrix) — Operational Note

> A fork of SVT-AV1 by @nekotrix with "sensible defaults" oriented toward perceptual quality and Quality-of-Life improvements.
> Reference version: **v4.0.1-Essential** (based on mainline SVT-AV1 v4.0.1).
> Binary: `SvtAv1EncApp` (standalone — supports MP4/MKV input via bundled FFMS2, and IVF / WebM / MKV output).
> Source for real-world profiles in §4 and §5: nyaa.si release descriptions from `[Trix]` (nekotrix) and `[Ironclad]`_encoder, both verified to be using SVT-AV1-Essential v4.0.1.

---

## 1. Official Essential defaults (verified against `Parameters.md` v4.0.1)

Full table of Essential encoder defaults. Specifying any **already-default** value by hand is redundant.

### Base parameters

| Parameter | Default | Range | Notes |
| --- | --- | --- | --- |
| `--preset` | **4** | -1 to 13 | overridden by `--speed` if preset not specified |
| `--speed` | auto (slow ≤1080p, medium >1080p) | slower/slow/medium/fast/faster → preset 2/4/5/6/8 | resolution-dependent |
| `--quality` | auto (medium → CRF 30 ≤1080p / 35 >1080p) | higher/high/medium/low/lower → 20/25/30/35/40/45 | overrides `--crf` unless crf is set |
| `--crf` | **30** | 1-70 (0.25 step) | overrides `--quality` when specified |
| `--qp` | 30 | 1-63 | only used with `--aq-mode 0` |
| `--rc` | 0 (CRF/CQP) | 0/1/2 | 0=CRF, 1=VBR, 2=CBR |
| `--aq-mode` | 2 (deltaq pred efficiency) | 0/1/2 | 0 disables AQ |
| `--tune` | **1 (PSNR)** | 0-4 | 0=VQ, 1=PSNR, 2=SSIM, 3=IQ, 4=MS_SSIM |
| `--fast-decode` | 0 | 0/2 | OFF by default |
| `--color-format` | 1 (yuv420) | 0-3 | only supported value |
| `--input-depth` | 8 | 8/10 | Essential forces 10-bit internally via HBD |
| `--profile` | 0 (main) | 0-2 | — |
| `--level` | 0 (auto) | 0, 2.0-7.3 | — |

### GOP / structure

| Parameter | Default | Range | Notes |
| --- | --- | --- | --- |
| `--keyint` | **-2** (~5s auto) | -2 to 2^31-1 | Essential places a keyframe every ~5s by default |
| `--min-keyint` | **-1** (multiple of mini-gop) | -1 to 2^31-1 | prevents keyframe spam |
| `--irefresh-type` | **2** (KEY Frame / Closed GOP) | 1/2 | 1=FWD Frame (Open GOP), 2=Key (Closed) |
| `--scd` | **1** (ON) | 0/1 | scene-change detection → intelligent keyframes |
| `--lookahead` | -1 (auto) | -1/0-32 | — |
| `--hierarchical-levels` | <=M12: 5, else 4 | 2-5 | 2=3 layers, 5=6 layers |
| `--pred-struct` | 2 (Random Access) | 1/2 | 1=low delay, 2=random access |
| `--enable-dg` | **0** (OFF) | 0/1 | dynamic GOP — only enable when needed |
| `--startup-mg-size` | 0 (OFF) | 0/2/3/4 | — |

### Filters / psychovisual

| Parameter | Default | Range | Notes |
| --- | --- | --- | --- |
| `--enable-dlf` | **2** | 0-3 | deblocking loop filter (3 = max accuracy, slower) |
| `--enable-cdef` | **1** (ON) | 0/1 | CDEF — already on |
| `--enable-restoration` | **1** (ON) | 0/1 | Loop restoration — already on |
| `--enable-tf` | **1** (ON) | 0-2 | ALT-REF temporally filtered frames |
| `--tf-strength` | **1** | 0-4 | temporal filter strength |
| `--enable-mfmv` | -1 (auto) | -1/0/1 | motion field mv |
| `--scm` | 2 (content adaptive) | 0-3 | screen content detection |
| `--sharpness` | **1** | -7 to 7 | sharpness bias |
| `--sharp-tx` | **0** | 0/1 | OFF by default (no-op to specify) |
| `--max-tx-size` | 64 | 32/64 | — |
| `--tx-bias` | **0** | 0-3 | 0=off, 1=full, 2=tx size, 3=interp filter |
| `--complex-hvs` | **0** | 0/1 | high-complexity HVS model — enable for strong PSY |
| `--noise-norm-strength` | **0** | 0-4 | AC boost for fine detail retention |
| `--noise-adaptive-filtering` | **2** | 0-4 | 0=off, 1=on CDEF+restoration, 2=tune behavior, 3=CDEF only, 4=restoration only |
| `--enable-alt-cdef` | **0** | 0-3 | alternative CDEF trade-offs |
| `--luminance-qp-bias` | **10** | 0-100 | QP based on average luma (dark scenes) |
| `--ac-bias` | **0.25** | 0.0-8.0 | bias toward high-frequency error |
| `--adaptive-film-grain` | 0 | 0/1 | film-grain synthesis block-size adaptive |
| `--film-grain` | 0 | 0-50 | 0=off, 1-50 = grain denoising level |
| `--film-grain-denoise` | 0 | 0/1 | — |
| `--photon-noise` | 0 | 0-100000 | ISO for photon-noise table |
| `--photon-noise-chroma` | 0 | 0/1 | — |

### Quantization / temporal consistency

| Parameter | Default | Range | Notes |
| --- | --- | --- | --- |
| `--enable-qm` | **1** (ON) | 0/1 | quant matrices already enabled |
| `--qm-min` | **2** | 0-15 | min luma flatness (0=most aggressive, 15=flat) |
| `--qm-max` | **15** | 0-15 | max luma flatness |
| `--chroma-qm-min` | **4** | 0-15 | min chroma flatness |
| `--chroma-qm-max` | **15** | 0-15 | max chroma flatness |
| `--enable-variance-boost` | **1** (ON) | 0/1 | already on |
| `--variance-boost-strength` | **1** | 1-4 | 1=mild, 2=gentle, 3=medium, 4=aggressive |
| `--variance-octile` | **4** (median) | 1-8 | 8x8 block selectivity |
| `--variance-boost-curve` | 0 (3 on PQ HDR) | 0-3 | 0=default, 3=HDR PQ |
| `--qp-scale-compress-strength` | **1** | 0-8 | temporal GOP consistency (max recommended: 3) |
| `--auto-tiling` | **1** (ON) | 0/1 | automatic tiles per resolution |
| `--tile-rows` | 0 | 0-6 | log2 — overrides auto-tiling |
| `--tile-columns` | 0 | 0-4 | log2 — overrides auto-tiling |

### Input / Output / Misc

| Parameter | Default | Range | Notes |
| --- | --- | --- | --- |
| `-i` | — | any string | input path (MKV/MP4/Y4M/YUV via bundled FFMS2) |
| `-b` | — | any string | output path: `.mkv`, `.webm`, `.ivf` |
| `--hide-banner` | 0 | 0/1 | hide banner |
| `--progress` | 1 | 0-2 | 2=verbose |
| `--lp` | 0 (auto) | 0/6 | parallelism level |
| `--pin` | 0 (no pin) | 0-N cores | pin instance to N threads |
| `--asm` | max | 0-11, c-max | limit instruction set |
| `--low-memory` | 0 | 0/1 | cut RAM usage |
| `--zones` | Null | any string | `start,end,crf;...` for CRF per range |
| `--roi-map-file` | Null | any string | QP map for 64x64 blocks |

### Default changes vs mainline (summary)

These are the defaults Essential **changes** compared to mainline SVT-AV1:

- Forced 10-bit internal precision via HBD
- `--enable-dlf 2` (mainline 1)
- `--enable-variance-boost 1`, `--variance-boost-strength 1`, `--variance-octile 4` (mainline all 0)
- `--tf-strength 1` (mainline 0)
- `--luminance-qp-bias 10` (mainline 0)
- `--sharpness 1` (mainline 0)
- `--qp-scale-compress-strength 1` (mainline 0)
- `--enable-qm 1`, `--qm-min 2`, `--chroma-qm-min 4` (mainline: qm off / qm-min 0 / chroma-qm-min 8)
- `--scd 1` (mainline off)
- `--keyint -2`, `--min-keyint -1` (mainline: often 0 / 0)
- `--auto-tiling 1` (mainline 0)
- `--ac-bias 0.25` (mainline 0)
- `--speed slow` at ≤1080p (mainline: fixed preset)
- Temporal filtering disabled on keyframes (no blur on scene cuts)
- Chroma fix (restoration mismatch from mainline v3)
- HDR10+ / Dolby Vision / varboost PQ curve (borrowed from SVT-AV1-HDR)

> Practical takeaway: Essential is **already slower but more faithful than mainline out of the box**. Stacking PSY parameters is often redundant.

---

## 2. PSY/HDR-borrowed parameters (NOT on by default)

These **actually change** behavior compared to the Essential default:

- `--tx-bias 0–3` — transform-size bias. 2 = max sharpness on fine textures (grain, film). Essential default: 0.
- `--complex-hvs 0–1` — highest-complexity HVS psychovisual model. 1 = on. Default: 0.
- `--noise-adaptive-filtering 0–4` — 1 disables CDEF + Loop Restoration in high-noise areas (preserves grain). 2 is the Essential/mainline default (filters behave per tune). 0 disables completely.
- `--noise-norm-strength 0–4` — normalizes AC coefficients so fine detail isn't flattened. Default: 0.
- `--tune 0/1/2/3/4` — 0=VQ (Visual Quality), 1=PSNR (default), 2=SSIM, 3=IQ, 4=MS_SSIM. Essential default stays **1 (PSNR)** — must be set explicitly.
- `--enable-alt-cdef 0–3` — alternative CDEF trade-offs (2/3 = max CDEF quality, slower).
- `--enable-alt-dlf 0–3` — alternative DLF trade-offs (pair with `--enable-dlf 3` to force max level).
- `--enable-tf 0–2` — 2 = adaptive; Essential also exposes `--enable-tf 3` for a more powerful temporal filter on all frames (acts as built-in denoiser). Adjust with `--tf-strength`.
- `--scd 0/1` — scene-change detection for intelligent keyframes (Essential-specific, tuned high).
- `--keyint -2 / -1 / 0 / N` — -2 = auto (~5s), -1/0 = only on scene change, N = multiple of 32+1 (max 300 for HW compatibility).
- `--min-keyint` — minimum gap between keyframes to avoid spam.
- `--zones A,B,CRF;C,D,CRF` — per-zone CRF (CRF/CQP mode only).
- `--auto-tiling 0/1` — automatic tiles per resolution (improves decoding).
- `--distortion-bias-preset 0–4` — Essential "band" preset auto-configuring high-fidelity params (1 mild, 2 medium, 3 strong, 4 mimics SVT-AV1-HDR grain retention).
- `--pin N` — pin the encoder instance to the first N CPU threads.
- `--low-memory` — cut RAM usage (most effective in CRF/RA mode).
- `--speed slower/slow/medium/fast/faster` — textual speed presets, resolution-dependent.
- `--quality higher/high/medium/low/lower` — quality presets.
- `--webm` — WebM output instead of IVF (automatic metadata passthrough).
- `--full-help` — full help.
- `--hide-banner` — hide banner.

---

## 3. Everyday reference presets

> All commands use direct MKV/MP4 input (bundled FFMS2). The `-b` output extension can be `.mkv`, `.webm`, or `.ivf` — in practice the encoder writes a valid file matching the extension you provide.
> If you want to keep audio/subtitles from the source, mux afterward with `mkvmerge` or `ffmpeg` (see §6).

### A. Daily driver — balanced, works for ~any video
Preset 4, CRF 27, tune VQ, mild PSY. Reasonable encode time.

```bash
SvtAv1EncApp -i input.mkv --preset 4 --crf 27 --tune 0 --complex-hvs 1 --noise-adaptive-filtering 1 -b output.mkv
```

### B. Anime (simple, real-world) — matches what Trix actually runs
This is what the maintainer himself runs on his weekly anime releases.

```bash
SvtAv1EncApp -i input.mkv --speed slow --crf 30 -b output.mkv
```

### C. Film / live-action with grain — high fidelity, grain preserved
For sources with native grain (Blu-ray, DVD rips, 70s-90s film). Strong PSY + adaptive noise filtering.

```bash
SvtAv1EncApp -i input.mkv --preset 4 --crf 26 --tune 0 \
  --tx-bias 2 --complex-hvs 1 --noise-norm-strength 1 --noise-adaptive-filtering 1 \
  --enable-dg 1 --qp-scale-compress-strength 3 \
  --color-primaries 1 --transfer-characteristics 1 --matrix-coefficients 1 \
  -b output.mkv
```

### D. Archive / "no compromises" — slow, max fidelity
For rare/irreplaceable material where time is no object.

```bash
SvtAv1EncApp -i input.mkv --preset 1 --crf 28 -b output.mkv
```

### E. In a hurry — preset 6, CRF 30
Essential defaults already beat mainline, so even preset 6 is watchable in honest time.

```bash
SvtAv1EncApp -i input.mkv --preset 6 --crf 30 --tune 0 -b output.mkv
```

### F. With Auto-Boost (automatic temporal consistency)
[Auto-Boost-Essential](https://github.com/nekotrix/auto-boost-algorithm/tree/main/Auto-Boost-Essential) adjusts CRF per-scene using SSIMULACRA2. Single line:

```bash
python Auto-Boost-Essential.py "input.mkv"
```

---

## 4. nekotrix's own profiles (the maintainer's philosophy)

 nekotrix (the Essential maintainer, who releases on nyaa under the `[Trix]` tag) consistently uses **bare-bones commands** — trusting the defaults he himself calibrated. Verified from his own release descriptions on nyaa.si (2025 batch, all tagged SVT-AV1-Essential v4.0.1-Essential):

| Trix release tier | Encoder command | Pre-filtering |
| --- | --- | --- |
| Weekly episode releases (most common) | `--speed slow --crf 30` | "Mild denoising" (VapourSynth) |
| One batch v2 (quality bump) | `--speed slower --crf 30` | "Mild denoising & line cleaning" |
| Archival batches (full season) | `--preset 1 --crf 28` | "Mild denoising & line cleaning" |
| Archival, rarer | `--preset 1 --crf 27` | "Mild denoising & line cleaning" |

That's it. No `--tune`, no `--complex-hvs`, no `--tx-bias`, no QM overrides, no noise params.

### Key observation: filtering happens *before* the encoder

For Trix, the encoder is **not** where the psychovisual heavy lifting lives. Trix applies VapourSynth pre-processing **upstream** of `SvtAv1EncApp`:

- weekly releases → mild denoise
- archival batches → mild denoise + line cleaning

The encoder itself just sits on its sensible defaults and runs `--crf 30` (or `--crf 27-28` for archival). All perceptual tuning is delegated to:
1. the Essential defaults (already PSY-aware), AND
2. the VapourSynth script that prepares the source.

### Why this works (and the lesson)

- **Essential defaults are already the tune.** nekotrix baked the PSY/QM/consistency stack into the defaults specifically so users wouldn't need to specify them. When you write `--speed slow --crf 30`, you implicitly get `--tune 1` (PSNR), `--enable-qm 1`, `--qm-min 2`, `--chroma-qm-min 4`, `--enable-variance-boost 1`, `--luminance-qp-bias 10`, `--qp-scale-compress-strength 1`, `--scd 1`, `--keyint -2`, `--enable-dlf 2`, `--tf-strength 1`, `--sharpness 1`, `--ac-bias 0.25` and 10-bit internal precision.
- **`--preset 1` is the slow end of practical.** Going below 4 dramatically increases time for diminishing returns. nekotrix uses it for archival material he cares about most.
- **CRF 27-30 is the sweet spot.** CRF 30 ≈ transparent on most content with Essential defaults. CRF 27-28 for archival. Lower than 24 is almost always a waste of space.

### Other anime encoders — note on compatibility

- **Sokudo** — uses **svt-av1-psy** (the PSY fork, NOT Essential) together with **auto-boost-av1an** (scene-based CRF). Their releases explicitly say `encoder: svt-av1-psy`. They are **not** part of the Essential ecosystem; their settings don't map 1:1 to Essential.
- **Ironclad** — confirmed Essential v4.0.1 user using the heavy stack documented in §5.

---

## 5. Real-world anime profiles (Ironclad)

Profiles observed directly in Ironclad release descriptions on nyaa.si. All Ironclad variants share the same PSY/fidelity stack and differ only in **CRF range and `--ac-bias`**. The CRF range notation (`15.0-30.0`, `10.0-30.0`) is per-episode/scene-based: low end for complex/grainy scenes, high end for flat/talky scenes (driven by an Auto-Boost-style script plus ROI Maps).

### 5.1 The Ironclad common stack (shared by all variants)

```
--preset 3 --tune 2
--noise-norm-strength 1 --qp-scale-compress-strength 3
--tx-bias 2 --complex-hvs 1 --noise-adaptive-filtering 1
--enable-alt-cdef 1 --enable-alt-dlf 2 --enable-dlf 3 --enable-tf 2
--enable-dg 1 --tile-columns 1 --fast-decode 2
--qm-min 0 --qm-max 15 --chroma-qm-min 0
--luminance-qp-bias 0 --sharp-tx 0
+ ROI Maps
```

Per-parameter rationale (anime-specific):

| Parameter | Value | Why (anime context) |
| --- | --- | --- |
| `--preset 3` | slow | Anime encoders accept the 2-4x time penalty vs. preset 4 for the extra motion-search refinement — crucial for cel-shaded line integrity. One release used `--preset 2` (slower still). |
| `--tune 2` | SSIM | Preserves structural/line geometry better than VQ — line art is exactly the case SSIM was designed for |
| `--noise-norm-strength 1` | mild AC boost | Anti-flatten for the few fine textures anime has (background details, CG/compositing noise). Default Essential is 0 |
| `--qp-scale-compress-strength 3` | max-consistency for general use | Anime has very stable frame-to-frame complexity unlike film — high consistency prevents flicker on held cels / looped frames |
| `--tx-bias 2` | tx size bias | Forces smaller transform sizes → keeps hard line edges sharp instead of ringing/blur |
| `--complex-hvs 1` | HVS on | Maximum-complexity HVS model. Pair with `--tune 2` |
| `--noise-adaptive-filtering 1` | CDEF+rest. off in noise | Preserves any deliberate noise/grain (rare on anime, but some productions have it) |
| `--enable-alt-cdef 1` | mild alt CDEF | Trade-off toward fidelity over deringing — anime line edges are forgiving of mild ringing but not of softening |
| `--enable-alt-dlf 2` + `--enable-dlf 3` | alt DLF + max DLF accuracy | Pair: forces the most accurate loop filter level. Cost: significant compute at faster presets |
| `--enable-tf 2` | adaptive temporal filter | Default Essential is 1 (always-on). Adaptive is safer for anime to avoid blending on motion |
| `--enable-dg 1` | dynamic GOP on | Essential default is 0. Anime cuts can be radical — dynamic GOP adapts keyframe/mini-gop structure to scene changes |
| `--tile-columns 1` | log2 = 2 columns | Essential auto-tiling is on by default; explicitly forcing 2 columns is a deliberate decode-time trade for compatibility |
| `--fast-decode 2` | fast decode lvl 2 | **Conflicts with the PSY stack** (limits hierarchical structure) but kept for player compatibility. Net effect: still quality-positive because all the aggressive PSY params downstream compensate |
| `--qm-min 0 --chroma-qm-min 0` | aggressive QM | More aggressive than Essential default (2/4). Forces non-flat quantization matrices → higher fidelity on hard edges, slightly larger files. `qm-max 15` stays default so flat regions still compress well |
| `--luminance-qp-bias 0` | disable luma-QP bias | Essential default is 10 — designed for live-action dark scenes. Anime flat dark areas can be hurt by luma-based QP tuning, so it's deliberately disabled |
| `--sharp-tx 0` | default | Redundant (already default). Likely exported by GUI/exporter for transparency |
| `--ac-bias` 0.25 or 1.0 | high-freq error bias | Default is 0.25; bumping to 1.0 (Profile 2 below) heavily biases toward high-frequency preservation — better for productions with lots of edge detail, hatching, screen-tone (screentone patterns) |
| `+ ROI Maps` | external per-frame QP maps | Encoder is supplied a region-of-interest map to push bits at foreground faces/subjects and starve the background — massive win on anime where ROI is well-defined |

### 5.2 The three Ironclad variants (verified on nyaa)

| Variant | CRF range | `--ac-bias` | Use case |
| --- | --- | --- | --- |
| **Profile 1** — edgy/textured | 10.0–30.0 | 1.0 (boosted) | Lots of hatching / screentone / heavy line-work (older cel anime, Shaft, certain 80s-90s OVA). Most commonly observed. |
| **Profile 2** — archive | 10.0–30.0 | 0.25 (Essential default) | Archival — same low CRF floor but default AC bias. Smaller files than Profile 1. |
| **Profile 3** — lighter bitrate floor | 15.0–30.0 | 0.25 (Essential default) | General anime, less aggressive on the lower end. Reasonable size. |

> Note: The Ironclad releases observed with `--preset 2` (one specific title) instead of 3 were even slower — same parameter stack otherwise.

### 5.3 The full Ironclad pipeline (not just encoder flags)

The encoder command alone is only **~30%** of the Ironclad pipeline. The other ~70% lives in VapourSynth pre-processing and FGS reinjection. Every Ironclad release on nyaa verifies the same pipeline:

1. **Pre-filter (VapourSynth, `vs-denoise` family)** — heavy denoise chain:
   - `dfttest` for general noise (luma + chroma), frequency-strength curve `{0.0: 0.3, 0.4: 0.3, 0.6: 0.6, 0.8: 1.5, 1.0: 2.0}`, `tr=2`
   - `bm3d` (FAST profile, `sigma=4` to `96` depending on source), luma-only (`planes=0`)
   - `nlm` (Non-Local Means) on chroma only, `h=0.6, tr=2, a=2`
   - `mc_degrain` for temporal denoise (`prefilter=DFTTEST`, `MVToolsPreset.HQ_SAD`, `thsad=100`)
2. **Dering / deband (VapourSynth)** — `vs-dehalo` `rx=2` to `rx=5` for ring removal; `vs-deband` `thr=[128,32,32]` or `thr=[192,32,32]` for banding
3. **Optional rescale (VapourSynth, `vodesfunc` + `vs-scale`)** — double upscale with **ArtCNN.R8F64** then downscale with **Hermite(linear=True)** (used for ~720p releases from 1080p sources)
4. **Encoder** — the §5.1 stack with per-variant CRF + ac-bias
5. **Film grain synthesis (FGS)** — re-inject synthetic grain after denoising:
   - `--film-grain 4-50 --adaptive-film-grain 1` (range = per-scene auto), OR
   - `--photon-noise 7800-14700` (ISO range = per-scene auto)
6. **ROI Maps** — external per-frame QP maps: face-detect / motion-masks to push bits at subjects
7. **Mux** — encoding pipeline (AV1 + Opus audio `--bitrate 128` via opusenc) + subtitle tracks

### 5.4 CLI form (Profile 1, sanitised)

Stripped of pure-default no-ops (`--sharp-tx 0`), ordered logically:

```bash
SvtAv1EncApp -i input.mkv \
  --preset 3 --crf 24 --tune 2 \
  --noise-norm-strength 1 --noise-adaptive-filtering 1 \
  --qp-scale-compress-strength 3 \
  --tx-bias 2 --complex-hvs 1 \
  --enable-alt-cdef 1 --enable-alt-dlf 2 --enable-dlf 3 --enable-tf 2 \
  --enable-dg 1 --tile-columns 1 --fast-decode 2 \
  --qm-min 0 --chroma-qm-min 0 --luminance-qp-bias 0 \
  --color-primaries 1 --transfer-characteristics 1 --matrix-coefficients 1 \
  --ac-bias 1.0 \
  --film-grain 8 --adaptive-film-grain 1 \
  -b output.mkv
# Add --roi-map-file roi.txt if you have a ROI map
# In real Ironclad encodes, the CRF is per-scene (auto-boost), not a single value
```

<details>
<summary>Profile 2 (archive) and Profile 3 (lighter floor) — diff from Profile 1</summary>

```bash
# Profile 2 (archive): identical stack, lower ac-bias
--ac-bias 0.25 --crf 10-30 (scene-based)

# Profile 3 (lighter bitrate floor): lower CRF floor raised to 15
--ac-bias 0.25 --crf 15-30 (scene-based)
```
</details>

### 5.5 Critical caveats

- **`--fast-decode 2` conflicts with everything else here.** Anime encoders accept this trade because player compatibility (especially aging hardware players / set-top boxes) trumps the marginal PSY loss. If you don't need it, **drop it** — the PSY stack becomes more effective and the encode times improve (paradoxically, fewer constraints).
- **`--luminance-qp-bias 0` is a deliberate anime choice**, not a mistake. Essential's default of 10 is tuned for live-action dark scenes; on anime flat darks, luma-biased QP can misallocate bits. Keep at 0 for anime. For live-action/film, leave the Essential default (10).
- **`--qm-min 0` / `--chroma-qm-min 0` is more aggressive than Essential's default** (2/4). Intentional for anime line-art, larger files. For live-action reconsider, since the Essential defaults were tuned for that scenario.
- **`+ ROI Maps` is doing a lot of heavy lifting.** Without an ROI map, the profile is ~70% as effective — encoders using this stack almost always generate ROI maps upstream (face detection, motion masks, VaPhase-style scripts).
- **CRF 10 is overkill** for distribution. The lower bound exists so the per-scene CRF algorithm (Auto-Boost-like, or manual `--zones`) can freely allocate bits on complex frames without clipping the floor. For a single-CRF encode, CRF 22-26 is the practical anime range.
- **Ironclad ≠ Trix philosophy.** Trix delegates psychovisual work to VapourSynth pre-filter + Essential defaults; Ironclad pushes it into the encoder + ROI + FGS. Don't mix the two casually.

---

## 6. Common mistakes / things to avoid

| Parameter | Why not to set it by hand |
| --- | --- |
| `--sharp-tx 0` | It's the Essential/mainline default. No-op. |
| `--ac-bias 0.25` | Already the Essential default. Useless *unless* you deviate (anime Ironclad Profile 1 uses 1.0). |
| `--qm-min 0 --qm-max 15` without `--enable-qm 1` | `--enable-qm` is already 1 by default in Essential. `qm-min 0` is *more aggressive than the Essential default* 2 — makes sense for anime line-art, larger files. Not a free win. |
| `--crf 30` if you don't intend to change it | Already the default (30). Fine to write as a "memo". |
| `--preset 4` if you don't change it | Already the `slow` default at ≤1080p (~preset 4). Fine to write for clarity. |
| `--luminance-qp-bias 0` | Essential default is **10**. Setting to 0 *worsens* live-action dark scenes but is **intentional for anime**. Choose by content type. |
| `--fast-decode 2` with `--tune 0/2` + `--complex-hvs 1` | Limits structure and PSY effectiveness. Keep only for hardware-decode compatibility needs. |
| Using the full Ironclad stack without a VapourSynth pre-pass | The stack is calibrated against a denoised/cleaned source. Running it on raw Crunchyroll/BD without the upstream `dfttest`+`bm3d`+`mc_degrain` chain will produce larger files with no quality benefit. |

---

## 7. Muxing into MKV (true container with audio/subs)

`SvtAv1EncApp` writes the video stream only (any extension you give it works, but the bitstream is the same). To get a `.mkv` with audio + subtitles from the source:

```bash
# 1. Encode only the video to .mkv (or .webm / .ivf)
SvtAv1EncApp -i input.mkv --preset 4 --crf 27 --tune 0 --complex-hvs 1 -b video.mkv

# 2. Mux into MKV keeping audio + subtitles from the source
mkvmerge -o output.mkv video.mkv --no-video input.mkv
```

Or with FFmpeg:

```bash
ffmpeg -i input.mkv -i video.mkv -map 1:v:0 -map 0:a -map 0:s? -c copy output.mkv
```

For audio encoding (Trix/Sokudo style — Opus 96-128 kbps stereo):

```bash
# Extract audio first, then encode with opusenc
opusenc --bitrate 128 input_audio.flac audio.opus
# Mux together
mkvmerge -o output.mkv video.mkv audio.opus --no-video --no-audio input.mkv
```

---

## 8. Quality verification (metrics)

To compare encode vs. source locally:

- **SSIMULACRA2** (recommended by nekotrix): [cloudinary/ssimulacra2](https://github.com/cloudinary/ssimulacra2)
- **Butteraugli**: [google/butteraugli](https://github.com/google/butteraugli)
- Universal CPU implementation: [vapoursynth-zip](https://github.com/dnjulek/vapoursynth-zip)
- GPU implementation: [Vship](https://codeberg.org/Line-fr/Vship)

Avoid PSNR/SSIM/vanilla VMAF as a single metric for perceptual quality.

---

## 9. References

- Repo: https://github.com/nekotrix/SVT-AV1-Essential
- Official builds (Windows/Linux/macOS): https://github.com/nekotrix/SVT-AV1-Essential/releases
- Homebrew tap (Linux/macOS, ffmpeg included): https://github.com/fraluc06/homebrew-ffmpeg-svt-av1-essential
- Auto-Boost-Essential: https://github.com/nekotrix/auto-boost-algorithm/tree/main/Auto-Boost-Essential
- HandBrake build with Essential: https://github.com/nekotrix/HandBrake-SVT-AV1-Essential
- SCD tuning test: https://gist.github.com/nekotrix/a025a48448ce05c3af9bd162dda70f66
- Auto-tiling test: https://wiki.x266.mov/blog/svt-av1-fourth-deep-dive-p2#tiles
- Legal video sources for testing: https://media.xiph.org/video/derf/
- Real-world profiles source: nyaa.si release descriptions from `[Trix]` and `[Ironclad]` uploaders, all tagged `SVT-AV1-Essential v4.0.1-Essential`
---
type: Note
_organized: true
---
# SVT-AV1 Essential \(nekotrix\) — Operational Note

> A fork of SVT-AV1 by @nekotrix with "sensible defaults" oriented toward perceptual quality and Quality-of-Life improvements.
> Reference version: **v4.0.1-Essential** \(based on mainline SVT-AV1 v4.0.1\).
> Binary: `SvtAv1EncApp` \(standalone — supports MP4/MKV input via bundled FFMS2, and IVF / WebM / MKV output\).
> Source for real-world profiles in §4 and §5: nyaa.si release descriptions from `[Trix]` \(nekotrix\) and `[Ironclad]`\_encoder, both verified to be using SVT-AV1-Essential v4.0.1.

---

## 1. Official Essential defaults \(verified against `Parameters.md` v4.0.1\)

Full table of Essential encoder defaults. Specifying any **already-default** value by hand is redundant.

### Base parameters

| `--preset` | **4** | -1 to 13 | overridden by `--speed` if preset not specified |
| --- | --- | --- | --- |
| `--speed` | auto \(slow ≤1080p, medium \>1080p\) | slower/slow/medium/fast/faster → preset 2/4/5/6/8 | resolution-dependent |
| `--quality` | auto \(medium → CRF 30 ≤1080p / 35 \>1080p\) | higher/high/medium/low/lower → 20/25/30/35/40/45 | overrides `--crf` unless crf is set |
| `--crf` | **30** | 1-70 \(0.25 step\) | overrides `--quality` when specified |
| `--qp` | 30 | 1-63 | only used with `--aq-mode 0` |
| `--rc` | 0 \(CRF/CQP\) | 0/1/2 | 0=CRF, 1=VBR, 2=CBR |
| `--aq-mode` | 2 \(deltaq pred efficiency\) | 0/1/2 | 0 disables AQ |
| `--tune` | **1 \(PSNR\)** | 0-4 | 0=VQ, 1=PSNR, 2=SSIM, 3=IQ, 4=MS\_SSIM |
| `--fast-decode` | 0 | 0/2 | OFF by default |
| `--color-format` | 1 \(yuv420\) | 0-3 | only supported value |
| `--input-depth` | 8 | 8/10 | Essential forces 10-bit internally via HBD |
| `--profile` | 0 \(main\) | 0-2 | — |
| `--level` | 0 \(auto\) | 0, 2.0-7.3 | — |

### GOP / structure

| Parameter | Default | Range | Notes |
| --- | --- | --- | --- |
| `--keyint` | **-2** \(~5s auto\) | -2 to 2^31-1 | Essential places a keyframe every ~5s by default |
| `--min-keyint` | **-1** \(multiple of mini-gop\) | -1 to 2^31-1 | prevents keyframe spam |
| `--irefresh-type` | **2** \(KEY Frame / Closed GOP\) | 1/2 | 1=FWD Frame \(Open GOP\), 2=Key \(Closed\) |
| `--scd` | **1** \(ON\) | 0/1 | scene-change detection → intelligent keyframes |
| `--lookahead` | -1 \(auto\) | -1/0-32 | — |
| `--hierarchical-levels` | \<=M12: 5, else 4 | 2-5 | 2=3 layers, 5=6 layers |
| `--pred-struct` | 2 \(Random Access\) | 1/2 | 1=low delay, 2=random access |
| `--enable-dg` | **0** \(OFF\) | 0/1 | dynamic GOP — only enable when needed |
| `--startup-mg-size` | 0 \(OFF\) | 0/2/3/4 | — |

### Filters / psychovisual

| Parameter | Default | Range | Notes |
| --- | --- | --- | --- |
| `--enable-dlf` | **2** | 0-3 | deblocking loop filter \(3 = max accuracy, slower\) |
| `--enable-cdef` | **1** \(ON\) | 0/1 | CDEF — already on |
| `--enable-restoration` | **1** \(ON\) | 0/1 | Loop restoration — already on |
| `--enable-tf` | **1** \(ON\) | 0-2 | ALT-REF temporally filtered frames |
| `--tf-strength` | **1** | 0-4 | temporal filter strength |
| `--enable-mfmv` | -1 \(auto\) | -1/0/1 | motion field mv |
| `--scm` | 2 \(content adaptive\) | 0-3 | screen content detection |
| `--sharpness` | **1** | -7 to 7 | sharpness bias |
| `--sharp-tx` | **0** | 0/1 | OFF by default \(no-op to specify\) |
| `--max-tx-size` | 64 | 32/64 | — |
| `--tx-bias` | **0** | 0-3 | 0=off, 1=full, 2=tx size, 3=interp filter |
| `--complex-hvs` | **0** | 0/1 | high-complexity HVS model — enable for strong PSY |
| `--noise-norm-strength` | **0** | 0-4 | AC boost for fine detail retention |
| `--noise-adaptive-filtering` | **2** | 0-4 | 0=off, 1=on CDEF+restoration, 2=tune behavior, 3=CDEF only, 4=restoration only |
| `--enable-alt-cdef` | **0** | 0-3 | alternative CDEF trade-offs |
| `--luminance-qp-bias` | **10** | 0-100 | QP based on average luma \(dark scenes\) |
| `--ac-bias` | **0.25** | 0.0-8.0 | bias toward high-frequency error |
| `--adaptive-film-grain` | 0 | 0/1 | film-grain synthesis block-size adaptive |
| `--film-grain` | 0 | 0-50 | 0=off, 1-50 = grain denoising level |
| `--film-grain-denoise` | 0 | 0/1 | — |
| `--photon-noise` | 0 | 0-100000 | ISO for photon-noise table |
| `--photon-noise-chroma` | 0 | 0/1 | — |

### Quantization / temporal consistency

| Parameter | Default | Range | Notes |
| --- | --- | --- | --- |
| `--enable-qm` | **1** \(ON\) | 0/1 | quant matrices already enabled |
| `--qm-min` | **2** | 0-15 | min luma flatness \(0=most aggressive, 15=flat\) |
| `--qm-max` | **15** | 0-15 | max luma flatness |
| `--chroma-qm-min` | **4** | 0-15 | min chroma flatness |
| `--chroma-qm-max` | **15** | 0-15 | max chroma flatness |
| `--enable-variance-boost` | **1** \(ON\) | 0/1 | already on |
| `--variance-boost-strength` | **1** | 1-4 | 1=mild, 2=gentle, 3=medium, 4=aggressive |
| `--variance-octile` | **4** \(median\) | 1-8 | 8x8 block selectivity |
| `--variance-boost-curve` | 0 \(3 on PQ HDR\) | 0-3 | 0=default, 3=HDR PQ |
| `--qp-scale-compress-strength` | **1** | 0-8 | temporal GOP consistency \(max recommended: 3\) |
| `--auto-tiling` | **1** \(ON\) | 0/1 | automatic tiles per resolution |
| `--tile-rows` | 0 | 0-6 | log2 — overrides auto-tiling |
| `--tile-columns` | 0 | 0-4 | log2 — overrides auto-tiling |

### Input / Output / Misc

| Parameter | Default | Range | Notes |
| --- | --- | --- | --- |
| `-i` | — | any string | input path \(MKV/MP4/Y4M/YUV via bundled FFMS2\) |
| `-b` | — | any string | output path: `.mkv`, `.webm`, `.ivf` |
| `--hide-banner` | 0 | 0/1 | hide banner |
| `--progress` | 1 | 0-2 | 2=verbose |
| `--lp` | 0 \(auto\) | 0/6 | parallelism level |
| `--pin` | 0 \(no pin\) | 0-N cores | pin instance to N threads |
| `--asm` | max | 0-11, c-max | limit instruction set |
| `--low-memory` | 0 | 0/1 | cut RAM usage |
| `--zones` | Null | any string | `start,end,crf;...` for CRF per range |
| `--roi-map-file` | Null | any string | QP map for 64x64 blocks |

### Default changes vs mainline \(summary\)

These are the defaults Essential **changes** compared to mainline SVT-AV1:

- Forced 10-bit internal precision via HBD
- `--enable-dlf 2` \(mainline 1\)
- `--enable-variance-boost 1`, `--variance-boost-strength 1`, `--variance-octile 4` \(mainline all 0\)
- `--tf-strength 1` \(mainline 0\)
- `--luminance-qp-bias 10` \(mainline 0\)
- `--sharpness 1` \(mainline 0\)
- `--qp-scale-compress-strength 1` \(mainline 0\)
- `--enable-qm 1`, `--qm-min 2`, `--chroma-qm-min 4` \(mainline: qm off / qm-min 0 / chroma-qm-min 8\)
- `--scd 1` \(mainline off\)
- `--keyint -2`, `--min-keyint -1` \(mainline: often 0 / 0\)
- `--auto-tiling 1` \(mainline 0\)
- `--ac-bias 0.25` \(mainline 0\)
- `--speed slow` at ≤1080p \(mainline: fixed preset\)
- Temporal filtering disabled on keyframes \(no blur on scene cuts\)
- Chroma fix \(restoration mismatch from mainline v3\)
- HDR10+ / Dolby Vision / varboost PQ curve \(borrowed from SVT-AV1-HDR\)

> Practical takeaway: Essential is **already slower but more faithful than mainline out of the box**. Stacking PSY parameters is often redundant.

---

## 2. PSY/HDR-borrowed parameters \(NOT on by default\)

These **actually change** behavior compared to the Essential default:

- `--tx-bias 0–3` — transform-size bias. 2 = max sharpness on fine textures \(grain, film\). Essential default: 0.
- `--complex-hvs 0–1` — highest-complexity HVS psychovisual model. 1 = on. Default: 0.
- `--noise-adaptive-filtering 0–4` — 1 disables CDEF + Loop Restoration in high-noise areas \(preserves grain\). 2 is the Essential/mainline default \(filters behave per tune\). 0 disables completely.
- `--noise-norm-strength 0–4` — normalizes AC coefficients so fine detail isn't flattened. Default: 0.
- `--tune 0/1/2/3/4` — 0=VQ \(Visual Quality\), 1=PSNR \(default\), 2=SSIM, 3=IQ, 4=MS\_SSIM. Essential default stays **1 \(PSNR\)** — must be set explicitly.
- `--enable-alt-cdef 0–3` — alternative CDEF trade-offs \(2/3 = max CDEF quality, slower\).
- `--enable-alt-dlf 0–3` — alternative DLF trade-offs \(pair with `--enable-dlf 3` to force max level\).
- `--enable-tf 0–2` — 2 = adaptive; Essential also exposes `--enable-tf 3` for a more powerful temporal filter on all frames \(acts as built-in denoiser\). Adjust with `--tf-strength`.
- `--scd 0/1` — scene-change detection for intelligent keyframes \(Essential-specific, tuned high\).
- `--keyint -2 / -1 / 0 / N` — -2 = auto \(~5s\), -1/0 = only on scene change, N = multiple of 32+1 \(max 300 for HW compatibility\).
- `--min-keyint` — minimum gap between keyframes to avoid spam.
- `--zones A,B,CRF;C,D,CRF` — per-zone CRF \(CRF/CQP mode only\).
- `--auto-tiling 0/1` — automatic tiles per resolution \(improves decoding\).
- `--distortion-bias-preset 0–4` — Essential "band" preset auto-configuring high-fidelity params \(1 mild, 2 medium, 3 strong, 4 mimics SVT-AV1-HDR grain retention\).
- `--pin N` — pin the encoder instance to the first N CPU threads.
- `--low-memory` — cut RAM usage \(most effective in CRF/RA mode\).
- `--speed slower/slow/medium/fast/faster` — textual speed presets, resolution-dependent.
- `--quality higher/high/medium/low/lower` — quality presets.
- `--webm` — WebM output instead of IVF \(automatic metadata passthrough\).
- `--full-help` — full help.
- `--hide-banner` — hide banner.

---

## 3. Simple workflow — learning AV1 encoding

You do **not** need VapourSynth, ffmpeg, ROI maps, or 20-flag command lines to get small files with very good quality. Essential's defaults are already tuned for perceptual quality. The encoder has one main knob (CRF) and a few optional ones you can add as you learn.

The workflow below is a **learning ladder** — three levels, each adds one new idea. Start at Level 1, only move up when you're curious.

### Level 1 — the only knob that matters: CRF

```bash
SvtAv1EncApp -i input.mkv --crf 30 -b output.mkv
```

That's the entire job. Everything else is Essential defaults (10-bit, QM, variance boost, luma bias, scene-change detection, sharpness, deblocking — all on). The **only** thing you control is `--crf`:

| CRF | Result | When |
|---|---|---|
| `--crf 32-34` | small file, good quality | casual storage, phone, streaming re-encode |
| `--crf 30` (default) | transparent on most content | **start here** |
| `--crf 27-28` | archive-grade | source you care about |
| `--crf 24-25` | near-lossless, large file | rare/irreplaceable source only |

> How to pick: encode a 1-minute clip at `--crf 30`, look at it, look at output size. Too big? Raise to 32. Too soft? Lower to 28. Repeat. CRF is logarithmic-ish — each +3 CRF roughly halves the file size.

### Level 2 — match the encoder to your content (2 flags)

Once you're comfortable with CRF, add **two** flags that genuinely matter for content type:

```bash
# Anime / cel animation
SvtAv1EncApp -i input.mkv --crf 28 --tune 2 --film-grain 6 --adaptive-film-grain 1 -b output.mkv

# Film / live-action with grain
SvtAv1EncApp -i input.mkv --crf 30 --tune 0 --film-grain 4 --adaptive-film-grain 1 -b output.mkv

# Clean digital video (modern streaming, no grain)
SvtAv1EncApp -i input.mkv --crf 30 -b output.mkv
```

| Flag | What it does | Why it matters |
|---|---|---|
| `--tune 0` (VQ) / `2` (SSIM) | Optimizes encoder target. Default is `1` (PSNR — math accuracy). | For anime, SSIM preserves line geometry. For film, VQ (Visual Quality) is what your eye actually sees. |
| `--film-grain N --adaptive-film-grain 1` | After compression, synthesize grain back into flat areas. | Clean Crunchyroll streams get crushed to wax — this restores texture. N=4 light, N=8 visible, N=12 heavy. |

The `--film-grain` flag is the single biggest "this looks pro" upgrade for free. Use 4-8 for most content, 10-15 for very grainy film stock.

### Level 2.5 — cheap PSY upgrades (always worth taking, no extra compute)

These flags are **essentially free** (no measurable encode-time cost) but give a real perceptual-uplift over the Essential defaults. Add them once, forget about them:

```bash
# Film / live-action with grain — PSY-default workflow
SvtAv1EncApp -i input.mkv --crf 30 --tune 0 \
  --complex-hvs 1 --tx-bias 2 \
  --noise-adaptive-filtering 1 --enable-tf 2 \
  --film-grain 4 --adaptive-film-grain 1 \
  -b output.mkv

# Anime / cel animation — same flags + anime-specific tune + luma bias off
SvtAv1EncApp -i input.mkv --crf 28 --tune 2 \
  --complex-hvs 1 --tx-bias 2 \
  --noise-adaptive-filtering 1 --enable-tf 2 \
  --luminance-qp-bias 0 \
  --film-grain 6 --adaptive-film-grain 1 \
  -b output.mkv
```

Why each belongs here (and what to **avoid** wasting compute on):

| Add (free PSY) | Default | Why it's worth |
|---|---|---|
| `--tune 0 / 2` | 1 PSNR | Optimize for eyes instead of math. Single biggest perceptual free win. |
| `--complex-hvs 1` | 0 | Activates the highest-complexity psychovisual model. Allocates bits where your eye actually looks. ≈ free compute. |
| `--tx-bias 2` | 0 | Forces smaller transform blocks → keeps hard edges sharp instead of ringing/blur. Slight motion-search cost but tiny. |
| `--noise-adaptive-filtering 1` | 2 | Disables CDEF + Loop Restoration in high-noise areas. Cheap, preserves grain rather than pialla it. |
| `--enable-tf 2` | 1 | Adaptive temporal filter (instead of always-on). Avoids motion-blending on anime/film movement. Small RD change. |
| `--film-grain N --adaptive-film-grain 1` | 0 | Grain synthesis after compression. Zero encode-time cost, monotone-against-wax look. |
| `--luminance-qp-bias 0` (anime only) | 10 | Disable the film-tuned default that hurts anime flat darks. Free. **Remove for live-action!** |

| Skip (not worth the compute) | Default | Why skip |
|---|---|---|
| `--sharp-tx 0` | 0 | Already default. No-op. |
| `--ac-bias 0.25` | 0.25 | Already default. Useless unless deviating. |
| `--qm-min 0 --chroma-qm-min 0` | 2 / 4 | Larger files for a gain only provable on heavy cel anime. Essential defaults are already tuned. |
| `--enable-dlf 3` | 2 | 2x-3x slower at preset 4 for marginal gain. Only worth it if you drop to `--preset 3`. |
| `--enable-alt-cdef 1` / `--enable-alt-dlf 2` | 0 / 0 | Only meaningful at preset ≤ 3. At default preset 4 they barely move the needle. |
| `--fast-decode 2` | 0 | **Actively conflicts** with everything in the "add" table. For weak-hardware playback only. |
| `--tile-columns 1` | 0 (auto) | Forces a decode-time trade for compatibility with specific players. Skip for personal encodes. |
| `--preset 3` (silently slower) | 4 | Goes into Level 3 (next) — don't add it here, the encode becomes 2-3x longer for an 8-10% quality gain. |

### Level 3 — when you have patience: drop to preset 3

When Level 2.5 stops being enough and you don't mind 2-3x longer encodes, add a few slow flags:

```bash
SvtAv1EncApp -i input.mkv --crf 24 --preset 3 --tune 2 \
  --tx-bias 2 --complex-hvs 1 --enable-dg 1 --luminance-qp-bias 0 \
  --enable-alt-cdef 1 --enable-alt-dlf 2 --enable-dlf 3 --enable-tf 2 \
  --film-grain 6 --adaptive-film-grain 1 \
  -b output.mkv
```

| Adds | Cost | Why now |
|---|---|---|
| `--preset 3` | 2-3x encode time | Better motion search → sharper lines, fewer artifacts. The big slow knob. |
| `--enable-alt-cdef 1` | small | Alternative CDEF trade-off now pays off (line-edge fidelity). |
| `--enable-alt-dlf 2` + `--enable-dlf 3` | small-to-moderate | Most accurate deblocking. Now your preset can keep up with it. |
| `--enable-dg 1` | small | Dynamic GOP for abrupt scene cuts (anime). |

> Stop here. This is the "effort-to-quality" ceiling for personal encoding — ~90% of the pro encoders' (Ironclad) look. Anything more (§4 onwards) needs VapourSynth pre-filtering and ROI maps to produce real gains; running it "standalone" wastes compute.

### The decision tree — print this

```
Is the source anime?              Is the source grainy film?
        │                                  │
        ▼                                  ▼
   --tune 2                          --tune 0
   --luminance-qp-bias 0              --film-grain 8-12
   --film-grain 4-8
        │                                  │
        └──────────────┬───────────────────┘
                       ▼
              How much time do you have?
                       │
            ┌──────────┴───────────┐
            ▼                      ▼
        in a hurry              patience
   (just use --crf 30,       (--preset 3, add
    defaults do the rest)     --tx-bias 2 --complex-hvs 1
                             --enable-dg 1)
```

### How to actually learn (10 minutes)

1. **Pick a 1-minute clip** from your favorite source MKV.
2. **Run Level 1 at three CRFs**: `--crf 28`, `--crf 30`, `--crf 32`. Compare filesizes side-by-side. Pick the largest one you're happy with.
3. **Add `--tune` for your content** (anime=2, film=0). Compare against Level 1. Notice grain looks better.
4. **Add `--film-grain 4`**. Look at flat walls / skies in the encode. If they look like wax, bump to 6 or 8.
5. **Done.** Only escalate to Level 3 / §4 if you have a source you genuinely care about and time to wait.

> Remember: a 20-flag command from a pro encoder's release page is tuned for *their* source, *their* pipeline, *their* player compatibility. Your source and constraints are different. Start with CRF + tune + film-grain and let Essential do the rest.

### 3.5 — "I'd use VapourSynth for pre-filtering, but I don't know how"

Pro encoders (Ironclad, Trix) run a VapourSynth pre-pass **before** SVT-AV1-Essential: typically `dfttest`/`bm3d` denoise → `f3kdb` deband → `hqdn3d` light denoise → FGS grain reinjection on top of a cleaned source. The encoder then sees a near-flat, low-noise input and packs it tight.

You can get **most of the way there without VapourSynth**, because Essential ships those steps as encoder flags. This is the "VapourSynth-equivalent pre-filter chain, but built into the encoder" command — most defaults, only the PSY/denoise flags that change behaviour:

```bash
# VapourSynth-style pre-filter (denoise + deband + grain synth) without VapourSynth
SvtAv1EncApp -i input.mkv --crf 30 --tune 0 \
  --enable-tf 3 --tf-strength 2 \
  --film-grain 8 --film-grain-denoise 1 --adaptive-film-grain 1 \
  --noise-norm-strength 1 --noise-adaptive-filtering 1 \
  --complex-hvs 1 --tx-bias 2 \
  -b output.mkv
```

What each line replaces from a real VS script:

| VS plugin / step | Essential flag | What it does here |
|---|---|---|
| `dfttest` / `bm3d` / `mc_degrain` \(spatio-temporal denoise\) | `--enable-tf 3 --tf-strength 2` | Stronger temporal filter on **all** frames \(Essential-specific `3` mode\). This is Essential's built-in denoiser. |
| `f3kdb` / `neo_f3kdb` \(deband\) | `--enable-tf 3` + `--noise-adaptive-filtering 1` | TF flattening kills banding; NAF=1 keeps CDEF/restoration **off** in high-noise areas so original grain survives. |
| `hqdn3d` \(light chroma denoise\) | `--film-grain-denoise 1` | Denoises flat areas **first**, then synthesises grain back on top. |
| FGS reinjection \(grain synthesis post-encode\) | `--film-grain 8 --adaptive-film-grain 1` | Grain synthesis, block-size adaptive. Zero encode-time cost. |
| Film-grain tables from VS muxer | `--film-grain 8` (above) | Same idea — synthesized, not carried. |
| AC / fine-detail retention tuning | `--noise-norm-strength 1` | Normalises AC coefficients so denoising doesn't flatten edges. |
| PSY model + sharp lines | `--complex-hvs 1 --tx-bias 2` | High-complexity HVS + small transform bias. Cheap, free quality on a denoised source. |
| `--tune 0` | VQ instead of PSNR | Optimise for eyes on cleaned film/live-action. |

Tuning knobs (the only two you'll likely touch):

| Flag | Default | Adjust to... |
|---|---|---|
| `--tf-strength` | 2 (here) | `1` if source is already clean; `3-4` for very noisy film/DVD. |
| `--film-grain` | 8 (here) | `4-6` modern digital, `10-15` heavy film stock / cel anime. `0` to disable grain synthesis entirely. |

> Anime? Swap `--tune 0` → `--tune 2` and add `--luminance-qp-bias 0`. Everything else stays identical.
> You **don't need** VapourSynth, vspipe, ffmpeg, or a script for this. The whole chain is one `SvtAv1EncApp` invocation. If you later want the last ~10% (ROI maps, per-scene CRF, true bm3d), *that* is when VapourSynth becomes worth learning — see §4 and §6.F.

---

## 4. Ironclad — the command string from nyaa (and how to adapt it)

The string below is **copy-pasted verbatim** from an Ironclad anime release on nyaa.si, tagged `SVT-AV1-Essential v4.0.1-Essential`:

```
--crf 10.0-30.0 --preset 3 --tune 2 --noise-norm-strength 1 --qp-scale-compress-strength 3
--ac-bias 0.25 --sharp-tx 0 --tx-bias 2 --complex-hvs 1 --noise-adaptive-filtering 1
--enable-alt-cdef 1 --enable-alt-dlf 2 --qm-min 0 --qm-max 15 --chroma-qm-min 0
--luminance-qp-bias 0 --enable-dlf 3 --enable-tf 2 --enable-dg 1
--tile-columns 1 --fast-decode 2 + ROI Maps
```

This is the most aggressive anime profile observed in the wild. Ironclad uses a team pipeline (VapourSynth pre-filter, ROI maps generated face-detection-side, FGS reinjection) — running this string **as-is** without that pipeline produces larger files with no real benefit.

### 4.1 Per-parameter breakdown — what each flag really does

| Flag | Value | Ironclad effect | Essential default | Verdict for adapting |
|---|---|---|---|---|
| `--crf` | 10-30 (range, per-scene) | Auto-Boost: low end for complex scenes, high end for flat | 30 (fixed) | **Adapt**: use a single `--crf 24` for anime, or `--crf 26` for a lighter file. The 10-30 range requires an Auto-Boost-style script — see §6.F |
| `--preset` | 3 | "slow" — major time penalty vs. default 4 | 4 | **Adapt**: keep `--preset 3` if you have patience; drop to `--preset 4` for ~2x faster with negligible loss on anime |
| `--tune` | 2 (SSIM) | Best for anime line-art | 1 (PSNR) | **Keep** — one of the few settings that truly matters for anime |
| `--noise-norm-strength` | 1 | Mild AC boost for fine texture | 0 | **Keep 1** (cheap, helps screen-tone) |
| `--qp-scale-compress-strength` | 3 | Max temporal consistency | 1 | **Keep 2-3** — prevents flicker on anime loops |
| `--ac-bias` | 0.25 |same as default | 0.25 | **Drop** — already Essential default (Ironclad Profile 1 uses 1.0) |
| `--sharp-tx` | 0 | no-op | 0 | **Drop** — pure noise in the command |
| `--tx-bias` | 2 | Sharp line edges | 0 | **Keep** — critical for anime line integrity |
| `--complex-hvs` | 1 | Maximum-complexity HVS model | 0 | **Keep** — true PSY gain |
| `--noise-adaptive-filtering` | 1 | Disable CDEF/restoration in noise | 2 | **Keep 1** if source has grain; drop for clean digital anime |
| `--enable-alt-cdef` | 1 | Alternative CDEF trade-off (fidelity over dering) | 0 | **Keep** — line-edge fidelity |
| `--enable-alt-dlf` | 2 | Alternative deblocking trade-off | 0 | **Keep** — pairs with `--enable-dlf 3` below |
| `--qm-min` | 0 | Aggressive luma QM | 2 | **Adapt**: use `0` for older cel anime (heavy line-work); keep `2` for modern digital anime |
| `--qm-max` | 15 | same as default | 15 | **Drop** — already default |
| `--chroma-qm-min` | 0 | Aggressive chroma QM | 4 | Same as above: `0` for cel, `4` for digital |
| `--luminance-qp-bias` | 0 | Disable luma-QP bias (anime-specific) | 10 | **Keep 0** for anime. For live-action, **remove this flag** — leaving Essential default of 10 is correct for film dark scenes |
| `--enable-dlf` | 3 | Maximum deblocking accuracy (slow) | 2 | **Adapt**: `3` at `--preset 3` is heavy. Use `3` if preset ≤ 3; otherwise stick with `2` |
| `--enable-tf` | 2 | Adaptive temporal filter (anti-blend) | 1 | **Keep** — safer than always-on for anime motion |
| `--enable-dg` | 1 | Dynamic GOP (anime cuts) | 0 | **Keep** — anime has abrupt scene changes |
| `--tile-columns` | 1 | force 2 columns (decode-time compat) | 0 (auto) | **Optional**: only needed if you target specific weak hardware. Drop if you don't |
| `--fast-decode` | 2 | Faster decode, limits structure | 0 | **Drop** unless you target aging hardware players. It conflicts with the PSY stack you're enabling |
| `+ ROI Maps` | external | face-detection QP maps | — | **Drop for personal encodes** — requires an upstream pipeline. Without ROI maps, expect ~30% larger files at the same perceptual quality |

### 4.2 The simplified Ironclad — anime, personal use

Strip the no-ops (`--sharp-tx 0`, `--qm-max 15`, `--ac-bias 0.25`) and the things you don't need without Ironclad's pipeline (`--fast-decode 2`, `+ ROI Maps`). Keep what actually changes behavior for anime. Pick one CRF value:

```bash
# Anime — Ironclad spirit, personal encoding (no VapourSynth, no ROI maps)
SvtAv1EncApp -i input.mkv --crf 24 --preset 3 --tune 2 \
  --noise-norm-strength 1 --noise-adaptive-filtering 1 \
  --qp-scale-compress-strength 3 \
  --tx-bias 2 --complex-hvs 1 \
  --enable-alt-cdef 1 --enable-alt-dlf 2 --enable-dlf 3 --enable-tf 2 \
  --enable-dg 1 \
  --qm-min 0 --chroma-qm-min 0 --luminance-qp-bias 0 \
  --film-grain 6 --adaptive-film-grain 1 \
  --color-primaries 1 --transfer-characteristics 1 --matrix-coefficients 1 \
  -b output.mkv
```

Notes:
- `--film-grain 6 --adaptive-film-grain 1` replaces what Ironclad gets from its separate FGS reinjection step in VapourSynth. Raise to 8-12 for cel anime with visible grain; keep at 4 for clean digital anime.
- `--color-primaries/transfer/matrix 1` = BT.709 metadata, prevents color washout on TVs/players (Ironclad gets this from their VapourSynth muxer; pure encoder needs it explicit).
- No `--tile-columns` / `--fast-decode` — drop both unless you specifically target weak hardware.
- For modern digital anime only (Crunchyroll etc.), you can also drop `--qm-min 0 --chroma-qm-min 0` — keep Essential defaults (2 / 4) for smaller files. The aggressive QM is for cel anime / heavy line-work.

### 4.3 Sanity tier — "I want Ironclad quality, not the complexity"

If you want the Ironclad look without 15 flags, the only settings that really matter are:

```bash
SvtAv1EncApp -i input.mkv --crf 24 --tune 2 \
  --tx-bias 2 --complex-hvs 1 \
  --enable-dg 1 --luminance-qp-bias 0 \
  --film-grain 6 --adaptive-film-grain 1 \
  -b output.mkv
```

Six flags. The other 14 in the original Ironclad string are either already Essential defaults or only justified by their upstream pipeline. This gets you ~85% of the Ironclad look, single command, no external dependencies.

---

## 5. Common mistakes / things to avoid

| Parameter | Why not to set it by hand |
| --- | --- |
| `--sharp-tx 0` | It's the Essential/mainline default. No-op. |
| `--ac-bias 0.25` | Already the Essential default. Useless *unless* you deviate (anime Ironclad Profile 1 uses 1.0). |
| `--qm-min 0 --qm-max 15` without `--enable-qm 1` | `--enable-qm` is already 1 by default in Essential. `qm-min 0` is *more aggressive than the Essential default* 2 — makes sense for anime line-art, larger files. Not a free win. |
| `--crf 30` if you don't intend to change it | Already the default (30). Fine to write as a "memo". |
| `--preset 4` if you don't change it | Already the `slow` default at ≤1080p (~preset 4). Fine to write for clarity. |
| `--luminance-qp-bias 0` | Essential default is **10**. Setting to 0 *worsens* live-action dark scenes but is **intentional for anime**. Choose by content type. |
| `--fast-decode 2` with `--tune 0/2` + `--complex-hvs 1` | Limits structure and PSY effectiveness. Keep only for hardware-decode compatibility needs. |
| Using the full Ironclad stack without a VapourSynth pre-pass | The stack is calibrated against a denoised/cleaned source. Running it on raw Crunchyroll/BD without the upstream dfttest+bm3d+mc_degrain chain will produce larger files with no quality benefit. |

---

## 6. Quality verification (metrics)

To compare encode vs. source locally:

- **SSIMULACRA2** \(recommended by nekotrix\): [cloudinary/ssimulacra2](https://github.com/cloudinary/ssimulacra2)
- **Butteraugli**: [google/butteraugli](https://github.com/google/butteraugli)
- Universal CPU implementation: [vapoursynth-zip](https://github.com/dnjulek/vapoursynth-zip)
- GPU implementation: [Vship](https://codeberg.org/Line-fr/Vship)

Avoid PSNR/SSIM/vanilla VMAF as a single metric for perceptual quality.

---

## 7. References

- Repo: [https://github.com/nekotrix/SVT-AV1-Essential](https://github.com/nekotrix/SVT-AV1-Essential)
- Official builds \(Windows/Linux/macOS\): [https://github.com/nekotrix/SVT-AV1-Essential/releases](https://github.com/nekotrix/SVT-AV1-Essential/releases)
- Homebrew tap \(Linux/macOS, ffmpeg included\): [https://github.com/fraluc06/homebrew-ffmpeg-svt-av1-essential](https://github.com/fraluc06/homebrew-ffmpeg-svt-av1-essential)
- Auto-Boost-Essential: [https://github.com/nekotrix/auto-boost-algorithm/tree/main/Auto-Boost-Essential](https://github.com/nekotrix/auto-boost-algorithm/tree/main/Auto-Boost-Essential)
- HandBrake build with Essential: [https://github.com/nekotrix/HandBrake-SVT-AV1-Essential](https://github.com/nekotrix/HandBrake-SVT-AV1-Essential)
- SCD tuning test: [https://gist.github.com/nekotrix/a025a48448ce05c3af9bd162dda70f66](https://gist.github.com/nekotrix/a025a48448ce05c3af9bd162dda70f66)
- Auto-tiling test: [https://wiki.x266.mov/blog/svt-av1-fourth-deep-dive-p2\#tiles](https://wiki.x266.mov/blog/svt-av1-fourth-deep-dive-p2#tiles)
- Legal video sources for testing: [https://media.xiph.org/video/derf/](https://media.xiph.org/video/derf/)
- Real-world profiles source: nyaa.si release descriptions from `[Trix]` and `[Ironclad]` uploaders, all tagged `SVT-AV1-Essential v4.0.1-Essential`

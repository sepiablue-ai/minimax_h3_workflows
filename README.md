# MiniMax-H3 ComfyUI Workflows

**Ready-to-load ComfyUI workflows for MiniMax H3 (Ref2VA / FL2VA / VideoRef / Pose Control) on a 12GB GPU. The fastest one makes a 5-second 1088×1920 video with audio in about 1 minute on an RTX 4070 12GB.**

RTX 4070 12GB・VRAM 12GBで動くMiniMax H3のComfyUIワークフロー集です。最速の構成では、1088×1920・音声付きの5秒動画を約1分で生成します。

> Author: Kamimoto (かみもと) | Quick results on **[X (@sep_is_heim)](https://x.com/sep_is_heim)**, write-ups (Japanese) on **[note](https://note.com/sepiablue)** ([article index](https://note.com/sepiablue/n/n8e612c619a3e)).

## Fastest picks (RTX 4070 12GB)

| Workflow | Output (124 frames, 24 fps, audio) | Generation time |
|---|---|---:|
| **[`fused4_sla5_x2.json`](fused4_sla5_x2.json)** — FL2VA, 4 steps, SLA 5%, X2 VAE | 1088×1920 | **62.6 s / 58.7 s** |
| [`fused4_veda90_x2.json`](fused4_veda90_x2.json) — FL2VA, 4 steps, Veda 90%, X2 VAE | 1088×1920 | 81.5 s / 70.2 s |
| [`fl2va_turbo8_veda90_x2.json`](fl2va_turbo8_veda90_x2.json) — FL2VA base model, 8 steps, Veda 90%, X2 VAE | 1088×1920 | 134.1 s / 133.5 s |
| [`fasth3_vsa5_chunk2_fastvae_fhd.json`](fasth3_vsa5_chunk2_fastvae_fhd.json) — FastH3 VSA 5% | 1080×1920 | about 3 min 55 s |
| [`pr16072_topk_1024x1792.api.json`](pr16072_topk_1024x1792.api.json) — SLA top-k 10% | 1024×1792 | 259.6 s |

The first three rows use the same settings as FL2VA [022](https://sayakabenchmark.pages.dev/benchmarks/fl2va/022-matlow-fused4-sla5-x2vae-int8-stream/workflow.json), [023](https://sayakabenchmark.pages.dev/benchmarks/fl2va/023-matlow-fused4-veda90-x2vae-int8-stream/workflow.json) and [024](https://sayakabenchmark.pages.dev/benchmarks/fl2va/024-fl2va-int8-turbo8-veda90-x2vae-int8-stream/workflow.json) on [Sayaka Benchmark](https://sayakabenchmark.pages.dev/) (entry numbers on the site; case01 / case02, one run each). The last two rows are the measurements described in their sections below. Rows were measured on different days and setups, so treat them as a guide rather than a strict ranking. The X2 VAE workflows need [ComfyUI-H3-X2-Stream](https://github.com/sepiablue-ai/ComfyUI-H3-X2-Stream).

## Features
- **Optimized for Low VRAM (12GB)**:
  - Pruned / quantized models (int8 unet, nvfp4 / awq CLIP, int8 / fp16 VAE)
  - Memory-efficient processing with `MiniMaxLowVRAMAttention`, `MiniMaxChunkFeedForward`, and `Spectrum`
- **Turbo LoRA Support**: Fast 4-step generation
- **Full HD Resolution**: Supports 1088x1920 (vertical) / 1920x1088 (horizontal) FHD output
- **Audio Generation Support**: Generates synchronized audio alongside video

---

## Included Workflows

### 1. FL2VA (First & Last Frame to Video with Audio)
Generate smooth videos interpolating / continuing from first/last frame images with audio.

- **`minimax_h3_fl2va_vram12gb_fdh.json`**: ComfyUI Web UI workflow
- **`minimax_h3_fl2va_vram12gb_fdh_api.json`**: ComfyUI API format workflow

### 2. Ref2VA (Reference to Video with Audio)
Generate video using multiple reference images (e.g. character sheet / angles) to maintain character consistency across new scenes and outfits with audio.

- **`minimax_h3_ref2va_vram12gb_fdh.json`**: ComfyUI Web UI workflow
- **`minimax_h3_ref2va_vram12gb_fdh_api.json`**: ComfyUI API format workflow
- **`minimax_h3_ref2va_LBHupscaler.json`**: ComfyUI Web UI workflow with 3D Latent Upscaler (`minimax_h3_latent_upscaler_3d_fp16.safetensors` / LBH Upscaler) to upscale from 736x1280 to Full HD (1088x1920) while preserving audio

### 3. VideoRef (Motion / Video Reference to Video with Audio)
Generate video combining multiple reference images (for character identity and appearance) and a motion reference video (for dance/motion transfer and choreography timing) with audio.

- **`minimax_h3_videoref_api.json`**: ComfyUI API format workflow

### 4. Pose Control & ControlNet Workflows (Ref2VA + DWPose / Fun ControlNet)
Drive character movement and choreography using pose estimators and Fun ControlNet Union.

- **`minimax_h3_controlnet_aux_dwpose_UI.json`**: ComfyUI Web UI workflow with built-in `comfyui_controlnet_aux` DWPose estimator directly extracting pose from RGB video
- **`phase2_ref2va_pose_control_448x800_api.json`**: ComfyUI API format workflow for pose-guided video generation (448x800 resolution)

### 5. FastH3 VSA Sample (Unofficial Fast Generation)
Unofficial sample workflow using FastH3 Visual Sparse Attention (VSA) for high-speed generation.

- **`fasth3_vsa_sample.json`**: ComfyUI Web UI workflow
  - **Environment**: Kijai ComfyUI VSA branch
  - **Model**: FastH3 VSA INT8 ConvRot checkpoint
  - **Attention Engine**: `SolAttnMiniMax`
  - **Sparsity**: VSA keep 10% (90% sparsity)
  - **Shift**: video/audio shift: 12 / 3
  - **Sampler**: Euler
  - **Schedule**: 4-step official sigma schedule

### 6. FastH3 FHD Optimal Workflow (12GB VRAM Optimized, Sub-4min)
Fully optimized workflow for FastH3 FHD video generation, achieving ~3m 55s total runtime with zero Shared GPU Memory spill on 12GB GPUs.

- **`fasth3_vsa5_chunk2_fastvae_fhd.json`**: ComfyUI Web UI workflow
  - **Environment**: ComfyUI + FastVideo VSA + KJNodes + Mozer FastVAE fork
  - **Resolution & Length**: 1080x1920 (FHD Portrait) / 124 frames (5.17s @ 24fps)
  - **Model**: FastH3 VSA INT8 ConvRot checkpoint (4-step distilled)
  - **Attention Engine**: `SolAttnMiniMax`
  - **Sparsity**: VSA keep 5% (95% sparsity)
  - **VRAM Spill Protection**: `MiniMax H3 Chunk FeedForward` (chunks: 2, seq_threshold: 4096)
  - **Video VAE**: `minimax_h3_video_vae_int8_convrot.safetensors`
  - **VAE Decode Engine**: `MiniMax H3 Fast VAE Decode` (Mozer fork, tile_batch_size: 2)
  - **Audio VAE**: `minimax_h3_audio_vae_fp32.safetensors`
  - **Sampler / Schedule**: Euler / 4-step official manual sigmas
  - **Shift**: video/audio shift: 12 / 3
  - **Performance (RTX 4070 12GB)**:
    - Total Time: **~3m 55s – 3m 58s** (warm) / **~4m 12s** (cold start with model load)
    - Sampling: ~176s (~40.9s/step)
    - VAE Decode: ~51s – 55s (down from 106s in FP16)
    - Peak VRAM: ~10.8 GB (sampling) / ~6.3 GB (VAE decode)
    - Shared Memory Spill: **0 MB** (flat ~276MB baseline)
    - Quality: Bit-exact to standard VAE (MAE: 0.0, PSNR: ∞)

### 7. FastH3 720p to 2x All-in-One Upscale Workflow
All-in-one high-speed generation and upscaling pipeline: generates 720p (720x1280) video with FastH3 4-step VSA at ultra-fast speeds and low VRAM footprint, decodes via FastVAE, and automatically applies 2x AI super-resolution (e.g. `RealESRGAN_x2plus.pth`) with synchronized audio muxing.

- **`fasth3_720p_to_2x_upscale_allinone.json`**: ComfyUI Web UI workflow
  - **Base Resolution & Upscaled Output**: 720x1280 (720p Portrait) $\rightarrow$ **1440x2560 (2.5K/2x Upscaled)** / 124 frames (5.17s @ 24fps)
  - **Model**: FastH3 VSA INT8 ConvRot checkpoint (4-step distilled)
  - **Attention Engine**: `SolAttnMiniMax` (VSA keep 5%)
  - **VAE Engines**:
    - Video: `minimax_h3_video_vae_int8_convrot.safetensors` + `MiniMax H3 Fast VAE Decode` (tile_batch_size: 2)
    - Audio: `minimax_h3_audio_vae_fp32.safetensors`
  - **Upscaler Node / Model**: `ImageUpscaleWithModel` + `RealESRGAN_x2plus.pth` (can be swapped for any 2x/4x model such as Compact / DAT / RealESRGAN)
  - **Output**: Generates full video with synchronized audio saved directly to `FastH3_2x_Upscaled/`

### 8. FastH3 720p Fast Generation (Optimized for External Vulkan Upscaler)
Lightweight & ultra-fast generation workflow that outputs clean 720p (720x1280) video with audio, designed to be upscaled using standalone external Vulkan/ncnn CLI tools (`realesrgan-ncnn-vulkan`, `realcugan-ncnn-vulkan`) without PyTorch/ComfyUI VRAM overhead.

- **`fasth3_720p_vsa5_chunk2_fastvae_extupscale.json`**: ComfyUI Web UI workflow
  - **Resolution & Length**: 720x1280 (720p Portrait) / 124 frames (5.17s @ 24fps)
  - **Model**: FastH3 VSA INT8 ConvRot checkpoint (4-step distilled)
  - **Attention Engine**: `SolAttnMiniMax` (VSA keep 5%)
  - **VRAM Spill Protection**: `MiniMax H3 Chunk FeedForward` (chunks: 2, seq_threshold: 4096)
  - **VAE Engines**:
    - Video: `minimax_h3_video_vae_int8_convrot.safetensors` + `MiniMax H3 Fast VAE Decode` (tile_batch_size: 2)
    - Audio: `minimax_h3_audio_vae_fp32.safetensors`
  - **Output**: `FastH3_720p/video_720p` (24fps sRGB MP4 with Audio)
  - **External Upscaling (Recommended)**:
    Upscale to 1440x2560 (2x) or 4K with minimal VRAM (~100-300MB) using dedicated video models (e.g. `realesr-animevideov3` or `Real-CUGAN`):
    ```bash
    # Example using Real-ESRGAN Vulkan CLI (2x anime video upscale):
    ./realesrgan-ncnn-vulkan -i output/FastH3_720p/video_720p.mp4 -o output/FastH3_720p/video_720p_2x.mp4 -n realesr-animevideov3 -s 2
    ```

### 9. MiniMax H3 1024x1792 Ultra-Fast Generation (PR #16072 top-k 10% Sweet Spot)
Ultra-fast MATLOW Turbo workflow utilizing the official ComfyUI Block Sparse Attention node (PR #16072) with `top-k (SLA)` selection at 10% keep ratio. By targeting the 1024x1792 mathematical sweet spot for VAE tiling, it achieved about 4m19s total with a sub-200s sampler time on RTX 4070 12GB (the X2 VAE workflows in section 10 are faster today).

- **`pr16072_topk_1024x1792.api.json`**: ComfyUI workflow
  - **Resolution & Length**: 1024x1792 (Optimal Portrait Sweet Spot) / 124 frames (5.17s @ 24fps)
  - **Model**: `minimax_h3_fused_refdelta_r1024_turbo8_mystic07_int8_convrot.safetensors` (4-step MATLOW Turbo)
  - **Attention Engine**: `BlockSparseAttention` [PR #16072] (`top-k (SLA)`, keep 10%, sink conditioning: `exact_kv_and_rows`)
  - **VRAM Spill Protection**: `MiniMax H3 Chunk FeedForward` (chunks: 4, seq_threshold: 4096)
  - **VAE Engines**:
    - Video: `minimax_h3_video_vae_int8_convrot.safetensors` (Standard ComfyUI VAE Decode, ~39s via optimal tiling)
    - Audio: `minimax_h3_audio_vae_fp32.safetensors`
  - **Output**: `H3_1024x1792/video_1024x1792` (24fps sRGB MP4 with Audio)
  - **Required CLI Flag**:
    ComfyUI must be launched with `--disable-comfy-compiler` to prevent catastrophic CUDA graph breaks (~90s stall) at the final step:
    ```bash
    python main.py --windows-standalone-build --disable-auto-launch --disable-pinned-memory --use-sage-attention --disable-comfy-compiler
    ```
  - **Benchmark (RTX 4070 12GB)**:
    - **Sampler Time**: **198.76s** (~3m18s, sub-200s milestone)
    - **VAE Decode Time**: **39.17s** (12s faster than 1088x1920)
    - **Total Wall-Clock**: **259.62s (~4m19s)** | Peak VRAM: 11.62 GB

### 10. MiniMax H3 544x960 to 1088x1920 X2 VAE Workflows
High-speed workflows generating at 544x960 base resolution and decoding directly to 1088x1920 Full HD via X2 VAE (`H3X2StreamSave`), supporting both SLA and Veda sparse attention.

- **`fl2va_turbo8_veda90_x2.json`**: FL2VA INT8 + Turbo 8step + Veda 90% + X2 VAE
- **`fused4_sla5_x2.json`**: MATLOW Fused Turbo 4-step + SLA 5% keep ratio + X2 VAE
- **`fused4_veda90_x2.json`**: MATLOW Fused Turbo 4-step + Veda 90% keep ratio + X2 VAE
  - **Pipeline**: 544x960 generation $\rightarrow$ X2 VAE $\rightarrow$ 1088x1920 output (124 frames, 24 fps)
  - **Prerequisite**: Install [ComfyUI-H3-X2-Stream](https://github.com/sepiablue-ai/ComfyUI-H3-X2-Stream) and run its [`00_prepare_int8.json`](https://github.com/sepiablue-ai/ComfyUI-H3-X2-Stream/blob/main/examples/00_prepare_int8.json) once to prepare the INT8 X2 decoder.
  - **Measured (RTX 4070 12GB, Sayaka Benchmark FL2VA 022 / 023 / 024)**: 62.6 / 81.5 / 134.1 s (case01).

### 11. MiniMax-H3-Image Line Art → Colored-Pencil Coloring Video (FL2VA, 10 s)
Load one color illustration and run. MiniMax-H3-Image turns it into a black-and-white line drawing and a colored version of that same line art, then MiniMax-H3 FL2VA uses them as the first and last frames to make a 10-second video of the drawing being colored in with colored pencils.

カラーイラストを1枚入れて実行するだけで、MiniMax-H3-Imageで白黒線画とカラー線画を作り、それを開始・終了フレームにしてMiniMax-H3 FL2VAで「色鉛筆で塗っていく10秒動画」を作ります。

- **`h3image_lineart_to_color_fl2va_10s.json`**: ComfyUI Web UI workflow
  - **Custom nodes**: none (ComfyUI core nodes only; tested on ComfyUI 0.45.0 with comfy_kitchen 0.2.41). The sparse attention node is the core `Model Sparse Attention` (`BlockSparseAttention`, marked experimental).
  - **Prompts**: fixed, image-independent prompts — swap the input image and run. No prompt rewriting or post-processing nodes.
  - **Pipeline**:
    1. Input is scaled to about 4 MP and to a multiple of 32 (center crop of a few pixels), as in the official MiniMax-H3-Image edit template
    2. MiniMax-H3-Image (6 steps, CFG 1, Euler / simple): black-and-white line art, then colored line art from that line art + the input as color reference
    3. Both images are scaled with `ImageScaleToTotalPixels` as in the official MiniMax-H3 i2v template, to the 768p pixel budget (0.98 MP = 768×1344) with each side rounded to 32 (no crop; e.g. 768×1344 for 9:16, 1152×864 for 4:3)
    4. MiniMax-H3 FL2VA int8 + lightx2v Turbo 4-step 768p LoRA, shift 6 / 3, with core `Model Sparse Attention` (SLA, keep 10% = 90% sparse, start 0.2, other settings default): 243 frames (24 fps, about 10.1 s) with audio. Bypass that node to run dense attention.
  - **Models** (all from the official [Comfy-Org/MiniMax-H3-Image](https://huggingface.co/Comfy-Org/MiniMax-H3-Image) / [Comfy-Org/MiniMax-H3](https://huggingface.co/Comfy-Org/MiniMax-H3) repos):

    | Folder | File |
    |---|---|
    | `diffusion_models` | `minimax_h3_image_pruned_w4a8.safetensors`, `minimax_h3_fl2va_pruned_int8_convrot.safetensors` |
    | `loras/minimax-h3` | `minimax_h3_fl2v_turbo_4step_v1.0_768p_comfyui_bf16.safetensors` |
    | `text_encoders` | `qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors` (shared by image and video) |
    | `vae` | `minimax_h3_image_vae_fp16.safetensors`, `minimax_h3_video_vae_fp16.safetensors`, `minimax_h3_audio_vae_fp32.safetensors` |

  - **Measured (RTX 4070 12GB, 243 frames, one run each, whole workflow from line art to saved video)**:

    | Input → video size | 90% sparse (this workflow) | Dense (node bypassed) |
    |---|---:|---:|
    | 2:3 portrait → 832×1248 | 603.9 s | 1040.7 s |
    | 9:16 portrait → 768×1344 | 608.0 s | 1022.1 s |
    | 4:3 landscape → 1152×864 | 580.5 s | 962.7 s |

    The four video sampling steps take about 4 min with 90% sparse vs about 13 min dense (2:3 input: 239 s vs 787 s). The rest is line art (two images, about 1 min of sampling), text encoding with the 32B encoder, model swapping and VAE decoding. The dense 2:3 run used an earlier wording of the video prompt.
  - **Known limitation**: because the prompts name no colors, some areas can be colored in a different color mid-video (e.g. hair painted blue) before settling into the final colors in the last second.

### 12. MiniMax-H3-Image Morning → Noon → Evening → Night Time-Lapse Video (FL2VA)
Load one landscape photo (portrait or landscape) and optionally set the target duration in seconds (default `6.0`). MiniMax-H3-Image generates four time-of-day keyframes (morning, noon, evening, night) while locking the camera and composition, then MiniMax-H3 FL2VA generates three transition clips (`morning → noon`, `noon → evening`, `evening → night`) and concatenates them into a seamless time-lapse video with audio.

風景写真を1枚（縦長・横長どちらも対応）入れて実行するだけで、MiniMax-H3-Imageが同じ構図のまま朝・昼・夕方・夜の4枚のキーフレームを生成し、MiniMax-H3 FL2VAで3区間の変化動画（朝→昼・昼→夕方・夕方→夜）を生成して1本の音声付きタイムラプス動画へ自動結合します（目標合計秒数のデフォルトは `6.0` 秒）。

- **`h3image_timelapse_fl2va_6s.json`**: ComfyUI Web UI workflow
  - **Custom nodes**: none (ComfyUI core nodes only; validated on ComfyUI 0.45.0 with `comfy_kitchen` 0.2.41, including `BlockSparseAttention`, `ComfyMathExpression`, `StringFormat`, and `ConcatenateVideo`).
  - **Prompts**: fixed, image-independent lighting/sky/atmosphere prompts that avoid hardcoded scene objects so natural landscapes do not hallucinate buildings or streetlights.
  - **Pipeline**:
    1. Input image is scaled to ~4 MP and trimmed to a multiple of 32 (`ResizeImageMaskNode` `scale total pixels` → `scale to multiple: 32`) while preserving aspect ratio.
    2. MiniMax-H3-Image (6 steps, CFG 1.0, Euler / simple) generates 4 keyframes (`morning`, `noon`, `evening`, `night`) and saves the 4-frame batch as `H3_Timelapse/keyframes_4phases`.
    3. All 4 keyframes are scaled to the 768p pixel budget (~1 MP, multiple of 32) and matched to the same dimensions.
    4. Target total duration is divided equally across the 3 FL2VA segments (`morning → noon`, `noon → evening`, `evening → night`), snapped to the nearest `17k+5` frame count at 24 fps (`6.0 s` default → `56` frames / `2.33 s` per segment), generated with MiniMax-H3 FL2VA int8 + Turbo 4-step 768p LoRA, shift `6.0 / 3.0`, and `Model Sparse Attention` (`BlockSparseAttention`, SLA keep 10% = 90% sparse, start 0.2), then joined with `ConcatenateVideo` (`168` frames / `7.00 s` total at default `6.0 s`) and saved to `H3_Timelapse/timelapse_full`.
  - **Models** (all from the official [Comfy-Org/MiniMax-H3-Image](https://huggingface.co/Comfy-Org/MiniMax-H3-Image) / [Comfy-Org/MiniMax-H3](https://huggingface.co/Comfy-Org/MiniMax-H3) repos):

    | Folder | File |
    |---|---|
    | `diffusion_models` | `minimax_h3_image_pruned_w4a8.safetensors`, `minimax_h3_fl2va_pruned_int8_convrot.safetensors` |
    | `loras/minimax-h3` | `minimax_h3_fl2v_turbo_4step_v1.0_768p_comfyui_bf16.safetensors` |
    | `text_encoders` | `qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors` (shared by image and video) |
    | `vae` | `minimax_h3_image_vae_fp16.safetensors`, `minimax_h3_video_vae_fp16.safetensors`, `minimax_h3_audio_vae_fp32.safetensors` |

  - **Measured (RTX 4070 12GB, 90% sparse, equivalent API graph execution)**:
    - 16:9 landscape (`1672×941` → `1344×768`, 168 frames / 7.00 s total): **502.05 s** elapsed (peak VRAM `11,760` MiB)


### 13. One-Button Outfit-Change Dance Video (MiniMax-H3-Image + FL2VA)

Load one character image, enter two short English outfit descriptions, and set the video length and resolution. MiniMax-H3-Image creates an intermediate image for outfit A and a final image for outfit B. MiniMax-H3 FL2VA then generates one continuous dance video with music, using the original image as the start, the outfit-A pose as a mid-video guide, and the outfit-B image as the end. A random seed picks the choreography, the mid pose, and the final pose from five variants each (125 combinations), so each run dances differently. Outfit changes are requested at about 1/3 and 2/3 of the video; actual transition times can shift.

人物画像1枚と衣装A/Bの短い英文、動画の長さ（秒）と解像度を入力すると、MiniMax-H3-ImageがAの中間画像とBの終点画像を生成し、MiniMax-H3 FL2VAが元画像を始点、Aのポーズを途中のガイド、Bの画像を終点として、ダンス映像と音楽を1回の連続生成で作ります。シード（初期設定は毎回ランダム）から、振付・中間ポーズ・終点ポーズが各5種類の中から選ばれます（125通り）。着替えの指示は全体の約1/3と2/3の時刻ですが、実際の切替時刻はずれる場合があります。

- **[`tiktok_outfit_dance.json`](tiktok_outfit_dance.json)**: ComfyUI Web UI workflow
  - **Custom nodes**: none (59 execution nodes from ComfyUI core / `comfy_extras`, including `ComfyMathExpression`, `StringFormat`, `RegexReplace`, and `PreviewAny`; validated on ComfyUI 0.45.0 with frontend 1.58.5).
  - **Inputs & output**: one character image, two short English outfit fields, the video length in seconds (default 5.0), the resolution in megapixels (default 0.62), and one seed with `randomize` (default) or `fixed`. The chosen combination and seed are shown in the "選ばれた振付" node after each run; set the seed to `fixed` with that value to reproduce a result. MP4 is saved in `output/TikTokMV/` with the `anchored` filename prefix.
  - **Resolution & length**: Input is resized to the set megapixels with its aspect ratio preserved, then adjusted with a small center crop to a multiple of 32. Output dimensions follow the input aspect ratio (for a 2:3 input, 0.62 → 640×960 and 0.85 → 768×1152, the 768p size the 4-step LoRA targets). Higher resolution and longer videos need more VRAM and time. The length is converted to 24 fps frames and rounded up to the model's 17k+5 grid (5 s → 124 frames = 5.17 s, 10 s → 243 frames = 10.13 s). The model's trained range is about 124–362 frames.
  - **Choreography**: the five choreography texts, five mid poses, and five final poses are editable in the "⑨ 振付リスト" group (keep the `<<1>>`–`<<5>>` headings). The choreography texts describe the motion without fixing the mid and final poses, so any combination can be used.
  - **Pipeline**:
    1. MiniMax-H3-Image edits the source into outfit-A and outfit-B images, each with a 6-step KSampler.
    2. MiniMax-H3 FL2VA INT8 with the 4-step Turbo LoRA, SigmaShift 6 / 3, and Euler / simple generates the video once. The original image is the start frame, the outfit-A image guides the frame at 48% of the video, and the outfit-B image is the end frame. Outfit-change times in the prompt are set to 32% and 64% of the video.
  - **Models** (official [Comfy-Org/MiniMax-H3-Image](https://huggingface.co/Comfy-Org/MiniMax-H3-Image), [Comfy-Org/MiniMax-H3](https://huggingface.co/Comfy-Org/MiniMax-H3), and [lightx2v/Minimax-h3-Turbo](https://huggingface.co/lightx2v/Minimax-h3-Turbo) repositories):

    | Folder | File |
    |---|---|
    | `diffusion_models` | [`minimax_h3_image_pruned_w4a8.safetensors`](https://huggingface.co/Comfy-Org/MiniMax-H3-Image/resolve/main/diffusion_models/minimax_h3_image_pruned_w4a8.safetensors) |
    | `vae` | [`minimax_h3_image_vae_fp16.safetensors`](https://huggingface.co/Comfy-Org/MiniMax-H3-Image/resolve/main/vae/minimax_h3_image_vae_fp16.safetensors) |
    | `diffusion_models` | [`minimax_h3_fl2va_pruned_int8_convrot.safetensors`](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/diffusion_models/minimax_h3_fl2va_pruned_int8_convrot.safetensors) |
    | `loras/minimax-h3` | [`minimax_h3_fl2v_turbo_4step_v1.0_768p_comfyui_bf16.safetensors`](https://huggingface.co/lightx2v/Minimax-h3-Turbo/resolve/main/minimax_h3_fl2v_turbo_4step_v1.0_768p_comfyui_bf16.safetensors) |
    | `text_encoders` | [`qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors`](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/text_encoders/qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors) |
    | `vae` | [`minimax_h3_video_vae_fp16.safetensors`](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/vae/minimax_h3_video_vae_fp16.safetensors) |
    | `vae` | [`minimax_h3_audio_vae_fp32.safetensors`](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/vae/minimax_h3_audio_vae_fp32.safetensors) |

  - **Storage & VRAM**: Seven model files use about 58.15 GB of disk space. Validated with one 12 GB GPU (RTX 4070 / RTX 3060).
  - **Example runs**:
    - Earlier fixed-choreography version, 226 frames (one run per input; executed concurrently on separate GPUs, so timings are illustrative rather than a speed comparison): photo input, 608×1056, RTX 4070 12 GB: **453.28 s**; anime input, 640×960, RTX 3060 12 GB: **787.95 s**.
    - Current version, 10 s (243 frames), anime input, 640×960, RTX 4070 12 GB: five seeds covering all 15 variants once each took **448.6–457.9 s** per video. All kept the blue sky and art style, and the five dances were clearly different.
    - Current version, 5 s (124 frames), RTX 4070 12 GB: 640×960 took **222.1 s**; 832×1216 (1.0 MP) took **346.2 s**.
  - **Known limitations**: Outfit boundaries and timing can vary by generation (in the 10 s runs the outfit-B change landed 0.3–0.55 s late in three of five videos). Some combinations end with a near-still last ~0.75 s, and raised hands can leave the frame. An earlier mid pose that turned the body to look back over the shoulder produced a third arm during the turn in a 5 s run, so it was replaced with a front-facing pose (hands clasped at the chest); with the same seed and settings the extra arm did not appear (one run). Small details such as hair accessories, clouds, and logos can change; these runs do not establish a guarantee of exact identity or background preservation.

### 14. First-Person City Walk from One Image (MiniMax H3 I2VA, gaze selectable)

Load one photo of a street or path, set the length in seconds, and pick a gaze pattern. MiniMax H3 uses the photo as the first frame and generates a first-person (POV) walk with footsteps and ambient sound as one continuous shot. The prompt makes the camera the subject so the walker does not appear on screen.

道や通路が写った写真を1枚入れ、秒数と「視線の動き」を選んで実行すると、MiniMax H3がその写真を開始フレームに使い、歩行者目線（POV）の街歩き映像と足音・環境音を一続きのショットとして生成します。歩く人が画面に映らないよう、プロンプトはカメラを主語にして書いています。

- **[`h3_pov_walk_i2va.json`](h3_pov_walk_i2va.json)**: ComfyUI Web UI workflow
  - **Custom nodes**: none (ComfyUI core / `comfy_extras` nodes only, including `MiniMaxH3ImageToVideo`, `BlockSparseAttention`, `ComfyMathExpression`, `CustomCombo`, `JsonExtractString`, and `StringReplace`; validated on ComfyUI 0.45.0 with frontend 1.58.5).
  - **Inputs** (top-left group): image, length in seconds (5–15, default 10), gaze pattern, seed (fixed `26111801` by default; change the number for a different result), and the long side in px (default 1344). MP4 is saved in `output/H3_Walk/`.
  - **Gaze patterns**: the selected English sentence is inserted at `【視線の動き】` in the prompt shown in the top-right group.

    | Choice | Motion |
    |---|---|
    | 右を見てから前へ（標準） / right, then forward (default) | Looks at the right-hand scenery after a few steps, then returns to the path ahead. Same prompt as the earlier fixed version. |
    | 左を見てから前へ / left, then forward | Looks at the left-hand scenery, then returns to the path ahead. |
    | 前だけを見て歩く / forward only | Keeps facing the path ahead without panning. |

    To rename or add a choice, use the same name as a key in the JSON table in the "視線の動きの英文" node; a missing key leaves the gaze sentence empty. Two-sided patterns (right then left, left then right) were tried with two different wordings, but the camera only looked to one side and returned to the path, so they are not included.
  - **Resolution & length**: width and height follow the input aspect ratio and the long side, each rounded to a multiple of 32; the image is center-cropped slightly to that ratio and downscaled with `area` (no padding; the bundled 1672×941 sample becomes 1344×768). The length is converted to 24 fps frames, H3 generates up to the `17k+5` grid (10 s → 243 frames), and the video and audio are trimmed back to the set length (10 s → 240 frames / 10.0 s).
  - **Pipeline**: MiniMax H3 FL2VA INT8 (used as I2VA from the start frame) with the 4-step Turbo LoRA (768p), Euler / simple, SigmaShift 6 / 3, and `BlockSparseAttention` (SLA keep 10%, start 0.2).
  - **Models** (official [Comfy-Org/MiniMax-H3](https://huggingface.co/Comfy-Org/MiniMax-H3) repository):

    | Folder | File |
    |---|---|
    | `diffusion_models` | [`minimax_h3_fl2va_pruned_int8_convrot.safetensors`](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/diffusion_models/minimax_h3_fl2va_pruned_int8_convrot.safetensors) |
    | `loras/minimax-h3` | [`minimax_h3_fl2v_turbo_4step_v1.0_768p_comfyui_bf16.safetensors`](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/loras/minimax_h3_fl2v_turbo_4step_v1.0_768p_comfyui_bf16.safetensors) |
    | `text_encoders` | [`qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors`](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/text_encoders/qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors) |
    | `vae` | [`minimax_h3_video_vae_fp16.safetensors`](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/vae/minimax_h3_video_vae_fp16.safetensors) |
    | `vae` | [`minimax_h3_audio_vae_fp32.safetensors`](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/vae/minimax_h3_audio_vae_fp32.safetensors) |

  - **Measured** (1344×768, 24 fps; peak VRAM is the whole GPU from `nvidia-smi`):
    - RTX 4070 12 GB, 5 s (124 frames): about **204–207 s** per video (three scenes, one run each).
    - RTX 4070 12 GB, 10 s: **459–478 s** (four runs, three scenes), peak 11,504–11,569 MiB.
    - RTX 3060 12 GB, 10 s: **887–896 s** (eight runs, sidewalk scene), peak 11,387–11,475 MiB.
  - **Known limitations**: fine leaves and hedges can flicker slightly while the view moves. The gaze patterns were checked on one sidewalk image with one seed; the swing amplitude depends on the input image. Generated signboard text is not legible.

### 15. Overhead Cooking Short from One Dish Photo and Five Step Notes (MiniMax-H3-Image + LLM + FL2VA)

Load one photo of a finished dish and write five short English step notes. MiniMax-H3-Image turns the photo into an overhead plated shot on a white table, an LLM (Qwen3.5-9B) expands the notes into five shot descriptions, and MiniMax H3 generates one vertical cooking video with cooking sounds: one shot per step joined by cuts, ending on the plated dish. Everything runs in one pass of the workflow.

料理の完成写真を1枚と、メモ書き程度の英語の工程を5行入れて実行すると、白い机を真上の固定カメラで撮ったような縦長の料理動画（約12秒・料理の音付き）を作ります。MiniMax-H3-Imageが完成写真から真上の盛り付け画像を作り、LLM（Qwen3.5-9B）が工程メモから5ショット分の文を書き、MiniMax H3が工程ごとに1ショットずつカットでつないで、最後は盛り付けで終わる動画を1回の生成で作ります。

- **[`h3_cooking_short.json`](h3_cooking_short.json)**: ComfyUI Web UI workflow
  - **Custom nodes**: none (50 execution nodes from ComfyUI core / `comfy_extras`, including `TextGenerateV2`, `StringFormat`, `RegexReplace`, `ComfyMathExpression`, `MiniMaxH3AddGuide`, and `BlockSparseAttention`; validated on ComfyUI 0.45.0 with frontend 1.58.5). Node and group titles are in Japanese and English.
  - **Inputs** (group ①): the finished-dish photo (any orientation, size, or camera angle), five step notes in English (all five are required; one step per line, e.g. `slice an onion`), the length in seconds (default 12), and the seed (fixed `261121` by default). The default image name is `omurice.png`; put your own photo in `ComfyUI/input/`. MP4 is saved as `output/H3_Cooking/cooking_short_*.mp4` and the plated image as `cooking_short_final_*.png`.
  - **Pipeline**:
    1. MiniMax-H3-Image (1440×2560, 6 steps, CFG 1, Euler / simple) uses the photo as a reference and draws the plated dish from directly above on a white table. The photo is passed as is; the node resizes it internally.
    2. `TextGenerateV2` with Qwen3.5-9B (thinking on, greedy decoding) reads the photo and the notes and writes `[Shot 1]`–`[Shot 5]`. The system prompt is dish-independent: each shot is written from five slots — the vessel and what it holds at the start, the left hand that steadies it, the one tool in the right hand, one continuous motion, and how the food looks at the end — as positive, visible statements. Stovetop cooking is shown in its pot or pan on the table, and oven steps as the moment just before or after. The LLM output and the full video prompt are shown in preview nodes.
    3. MiniMax H3 FL2VA INT8 with the 4-step Turbo LoRA (768p), SigmaShift 6 / 3, Euler / simple, and `BlockSparseAttention` (SLA keep 10%, start 0.2) generates 736×1312 video with audio once. `[Shot 6]` is a still life of the plated dish; the plated image (scaled to 736×1312 with a center crop of about 2 px in total) is used as the last frame and also as a guide at the start of Shot 6, which keeps hands out of the final shot. Cut times are set by dividing the length into six equal parts.
    4. The length is converted to 24 fps frames on the `17k+5` grid inside the model's trained range of 124–362 frames (12 s → 294 frames = 12.25 s).
  - **Models**:

    | Folder | File |
    |---|---|
    | `diffusion_models` | [`minimax_h3_image_pruned_w4a8.safetensors`](https://huggingface.co/Comfy-Org/MiniMax-H3-Image/resolve/main/diffusion_models/minimax_h3_image_pruned_w4a8.safetensors) |
    | `vae` | [`minimax_h3_image_vae_fp16.safetensors`](https://huggingface.co/Comfy-Org/MiniMax-H3-Image/resolve/main/vae/minimax_h3_image_vae_fp16.safetensors) |
    | `diffusion_models` | [`minimax_h3_fl2va_pruned_int8_convrot.safetensors`](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/diffusion_models/minimax_h3_fl2va_pruned_int8_convrot.safetensors) |
    | `loras/minimax-h3` | [`minimax_h3_fl2v_turbo_4step_v1.0_768p_comfyui_bf16.safetensors`](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/loras/minimax_h3_fl2v_turbo_4step_v1.0_768p_comfyui_bf16.safetensors) |
    | `text_encoders` | [`qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors`](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/text_encoders/qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors) (shared by image and video) |
    | `text_encoders` | [`qwen3.5_9b_int8_convrot.safetensors`](https://huggingface.co/Comfy-Org/MOSS-Image/resolve/main/text_encoders/qwen3.5_9b_int8_convrot.safetensors) (LLM, loaded with `CLIPLoader` type `moss_image`) |
    | `vae` | [`minimax_h3_video_vae_fp16.safetensors`](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/vae/minimax_h3_video_vae_fp16.safetensors) |
    | `vae` | [`minimax_h3_audio_vae_fp32.safetensors`](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/vae/minimax_h3_audio_vae_fp32.safetensors) |

  - **Storage & VRAM**: the eight model files use about 67.9 GB of disk space. Validated on one RTX 4070 12 GB.
    - If you already use the MiniMax H3 video workflows in this repository (FL2VA model, Turbo LoRA, `qwen3vl_32b` text encoder, video and audio VAEs, about 44.4 GB), you only need the H3-Image model (12.55 GB), the H3-Image VAE (1.17 GB), and the LLM (9.75 GB): about **23.5 GB** more.
    - If you also use the MiniMax-H3-Image workflows (sections 11–13), only the LLM is new: about **9.7 GB** more.
    - 既にこのリポジトリのH3動画ワークフローを使っている場合の追加は約23.5 GB（H3-Image本体・H3-Image VAE・LLM）、H3-Imageのワークフロー（11〜13節）も使っている場合はLLMの約9.7 GBだけです。
  - **Measured** (RTX 4070 12 GB, 12 s = 294 frames, 736×1312, seed 261121, one run per dish; time from job submission to saved video, including model and LLM loading; peak VRAM is the whole GPU from `nvidia-smi`): **623–680 s** per video (seven dishes: omelette rice, spaghetti, pancakes, oyakodon, tonjiru, macaroni gratin, karaage), peak 11,493–11,866 MiB. The input photos were generated images with mixed orientations, angles, and tables. Running from the Web UI's Run button with the defaults took about 618 s.
  - **Example step notes** (oyakodon): `slice an onion` / `cut a chicken thigh into bite-size pieces` / `beat two eggs` / `simmer the chicken and onion in dashi and soy sauce in a small pan` / `pour the beaten eggs over the chicken and let them set`
  - **Known limitations**:
    - Items from an earlier shot can carry over into a later one, because the whole video is generated at once (e.g. the cutting board left under a soup pot, garlic-like bits in the pasta water).
    - Counts are not kept reliably: three pancakes in the notes came out as one, and fried chicken pieces multiplied on the rack. Deep-fried batter can look as if it melts.
    - Food can jump between states mid-shot (a whole chicken thigh suddenly already cut, rice switching to chicken only).
    - The LLM wording follows the notes literally and sometimes oddly (lettuce cut with a knife on a plate after "tear"). A one-line change to its system prompt can change all five shots, so check several dishes after editing it.
    - The table can turn into a marble pattern, and the plated shot gets a fork even for Japanese dishes.
    - Write the notes in English. Japanese input tended to be drawn as on-screen text or labels in earlier versions.
    - Audio was not listened to in these runs.

### 16. 10Eros-Max v1.0 Speed-Up (I2VA, 8 / 7 / 6 steps, core nodes only)

Image-to-video for the community MiniMax H3 fine-tune [TenStrip/10Eros-Max](https://huggingface.co/TenStrip/10Eros-Max) (v1.0, 8-step Turbo, hybrid INT8). Comfy Kitchen INT8 attention, `BlockSparseAttention` (sol-attn) and the Sol VAE decoder make one pass **2.5–3.3× faster** than the plain PyTorch attention setup on an RTX 4070 12 GB. The three tested presets differ only in the step count, so they are merged into one workflow: change `steps` to 8, 7, or 6.

10Eros-Max v1.0（MiniMax H3のコミュニティ製ファインチューン）の画像から動画（I2VA）を、ComfyUI純正ノードだけで高速化したワークフローです。Kitchen INT8アテンション・sol-attnの疎化・Sol VAEデコードの組み合わせで、RTX 4070 12GBでPyTorch標準構成の2.5〜3.3倍速になります。検証した3種類はステップ数だけが違うので、1本にまとめました。`steps` を8・7・6から選んでください。

- **[`10eros_max_speedup_8-7-6step.json`](10eros_max_speedup_8-7-6step.json)**: ComfyUI Web UI workflow
  - **Custom nodes**: none (ComfyUI core / `comfy_extras` nodes only, including `ModelAttentionBackend`, `BlockSparseAttention`, `MiniMaxH3ImageToVideo`, `MiniMaxH3SigmaShift`, and `PrimitiveStringMultiline`; validated on ComfyUI 0.45.0 with frontend 1.58.5). Comfy Kitchen attention uses the `comfy-kitchen` package from ComfyUI's `requirements.txt` and is available only on NVIDIA and AMD GPUs (tested on NVIDIA only). Node and group titles are in Japanese and English.
  - **Inputs** (green "Settings" group): start image (any aspect ratio), prompt, resolution (`megapixels`, default 0.98), steps (8 / 7 / 6), and seed (fixed `42` by default). The generated video appears in the "Output" node right next to it and is saved as `output/10Eros_Speedup/10Eros_Sol_<steps>step_*.mp4`. The default image name is `first_frame.png`; put your own image in `ComfyUI/input/` or upload it on the node.
  - **Presets** (RTX 4070 12 GB, 0.98 MP ≈ 768p, 124 frames / 24 fps / about 5.17 s, seed 42, warm run, one portrait test image; generation time is sampling plus video decode; speed-up is against PyTorch attention, 8 steps, standard video VAE at 507.4 s):

    | `steps` | Generation time | Speed-up | Use |
    |---:|---:|---:|---|
    | 8 | 200.1 s | 2.54× | Standard (recommended) |
    | 7 | 178.4 s | 2.84× | Faster, compare with 8 first |
    | 6 | 156.0 s | 3.25× | Fastest |

    Peak VRAM was about 11.5 GB at 0.98 MP. A cold start (after freeing models) took 218.7 s in total for 8 steps. Running the 6-step preset from the Web UI's Run button with an 832×1216 image took 167–170 s per video (three runs, from queueing to the saved video).
  - **Pipeline**: `UNETLoader` (10Eros-Max hybrid INT8) → SigmaShift 12 / 3 → `ModelAttentionBackend` (comfy kitchen attention) → `BlockSparseAttention` (`sol-attn`, tau 1.3, start 0.10 so step 1 stays dense, dense blocks `0,1`) → Euler / simple. The start image is scaled to the set megapixels with `lanczos` in multiples of 32, keeping the aspect ratio (no crop or padding; 1248×1824 becomes 832×1216). The start image is encoded with the standard video VAE, and only the final decode uses the Sol VAE (about 12 s faster than the standard VAE decode).
  - **Models**:

    | Folder | File |
    |---|---|
    | `diffusion_models` | [`10Eros_Max_8stepTurbo_v1.0_hybrid-int8_HQ.safetensors`](https://huggingface.co/TenStrip/10Eros-Max/resolve/main/10Eros_Max_8stepTurbo_v1.0_hybrid-int8_HQ.safetensors) |
    | `text_encoders` | [`qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors`](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/text_encoders/qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors) |
    | `vae` | [`minimax_h3_video_vae_int8_convrot.safetensors`](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/vae/minimax_h3_video_vae_int8_convrot.safetensors) (encodes the start image) |
    | `vae` | [`minimax_h3_audio_vae_fp32.safetensors`](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/vae/minimax_h3_audio_vae_fp32.safetensors) |
    | `vae` | [`minimax_h3_sol_vae_int8_convrot.safetensors`](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/vae/minimax_h3_sol_vae_int8_convrot.safetensors) (final video decode) |

  - **Launch**: on a 12 GB GPU, start ComfyUI with `--disable-pinned-memory` (the setting used for all measurements).
  - **Known limitations**:
    - Quality was compared with the dense 8-step baseline on general portrait images (seeds 42 and 43). Fast, large body motion was checked only by eye on a few clips; compare 7 and 6 steps with 8 on your own images.
    - Only 8, 7, and 6 steps with Euler / simple were tested. `res_multistep` + `beta` at 6 steps broke the audio and is not recommended.
    - Resolutions other than 0.98 MP were not tested. Peak VRAM is already about 11.5 GB at 0.98 MP, so raising it may run out of memory on 12 GB.
    - Thin white clothing tended to become see-through, and a body shadow on the wall could disappear in the second half unless the prompt asks to keep it.

---

## Author and related repositories

I'm Kamimoto (かみもと). I measure and publish MiniMax H3 and ComfyUI speed-ups and VRAM reductions on an RTX 4070 12 GB.

- X: [@sep_is_heim](https://x.com/sep_is_heim) — quick results with step counts, seconds and speed-ups
- note: [かみもと｜ローカルAI生成](https://note.com/sepiablue) — setup guides and measured write-ups in Japanese ([article index](https://note.com/sepiablue/n/n8e612c619a3e))
- [Sayaka Benchmark](https://sayakabenchmark.pages.dev/) — speed and quality comparison of H3 workflows

| Repository | What it does |
|---|---|
| [ComfyUI-H3-X2-Stream](https://github.com/sepiablue-ai/ComfyUI-H3-X2-Stream) | INT8 X2 VAE and asynchronous NVENC saving; 5 s 1088×1920 video with audio in about 60 s |
| [ComfyUI-H3-LBH-X2-Stream](https://github.com/sepiablue-ai/ComfyUI-H3-LBH-X2-Stream) | 3 low-resolution steps + 1 high-resolution step; about 25% faster for 14 s video |
| [ComfyUI-H3-AudioFirst-X2-Stream](https://github.com/sepiablue-ai/ComfyUI-H3-AudioFirst-X2-Stream) | Audio-first two-pass generation that reduces the metallic voice |
| [ComfyUI-MiniMax-H3-W4A4-VSA](https://github.com/sepiablue-ai/ComfyUI-MiniMax-H3-W4A4-VSA) | Ref2VA acceleration with FC1 W4A4 and Streaming VSA |

## License / Redistribution（ライセンス・再配布）

### 日本語

このリポジトリで公開しているComfyUIワークフローは、[MITライセンス](LICENSE)のもとで、個人利用・改変・再配布・商用利用を含め、自由に利用していただいて構いません。

再配布や改変版を公開する場合は、可能であれば元リポジトリへのリンクを記載していただけると嬉しいです。リンクの記載は任意ですが、MITライセンスの条件に従い、ワークフローのコピーまたは実質的な部分を再配布する際は、`LICENSE`に記載された著作権表示および許諾表示を含めてください。

元リポジトリ / Original repository: [sepiablue-ai/minimax_h3_workflows](https://github.com/sepiablue-ai/minimax_h3_workflows)

このリポジトリのワークフローファイルはMITライセンスで公開しています。ワークフローが参照するモデル、LoRA、カスタムノード、その他の第三者コンポーネントには、それぞれのライセンスが適用されます。これらの利用・再配布・商用利用については、各配布元のライセンスを確認してください。

### English

These ComfyUI workflows may be freely used, modified, redistributed, and used commercially under the [MIT License](LICENSE).

When redistributing or publishing modified versions, attribution or a link to the original repository is appreciated. Linking is optional; however, when redistributing copies or substantial portions of the workflows, you must include the copyright notice and permission notice in `LICENSE`, as required by the MIT License.

Original repository: [sepiablue-ai/minimax_h3_workflows](https://github.com/sepiablue-ai/minimax_h3_workflows)

The workflow files in this repository are released under the MIT License. Please note that models, LoRAs, custom nodes, and other third-party components referenced by these workflows are subject to their respective licenses. Check each provider's license for terms governing use, redistribution, and commercial use of those components.

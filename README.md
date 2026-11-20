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

Load one character image and enter two short English outfit descriptions. MiniMax-H3-Image creates an intermediate image for outfit A and a final image for outfit B. MiniMax-H3 FL2VA generates one continuous 226-frame dance video with music, using the original image as the start, the outfit-A pose as a guide at 4.5 seconds, and the outfit-B image as the end. Outfit changes are requested at about 3 and 6 seconds; actual transition times can shift.

人物画像1枚と衣装A/Bの短い英文を入力すると、MiniMax-H3-ImageがAの中間画像とBの終点画像を生成します。MiniMax-H3 FL2VAは、元画像を始点、Aのポーズを4.5秒位置のガイド、Bの画像を終点として、226フレームのダンス映像と音楽を1回の連続生成で作ります。着替えの指示は約3秒・6秒ですが、実際の切替時刻はずれる場合があります。

- **[`tiktok_outfit_dance.json`](tiktok_outfit_dance.json)**: ComfyUI Web UI workflow
  - **Custom nodes**: none (36 execution nodes from ComfyUI core / `comfy_extras`; validated on ComfyUI 0.45.0 with frontend 1.58.5).
  - **Inputs & output**: one character image and two short English outfit fields; fixed seed 42; MP4 saved in `output/TikTokMV/` with the `anchored` filename prefix.
  - **Resolution & length**: Input is resized to about 0.62 MP with its aspect ratio preserved, then adjusted with a small center crop to a multiple of 32. Output dimensions follow the input aspect ratio; the video is 226 frames at 24 fps (about 9.42 s).
  - **Pipeline**:
    1. MiniMax-H3-Image edits the source into outfit-A and outfit-B images, each with a 6-step KSampler.
    2. MiniMax-H3 FL2VA INT8 with the 4-step Turbo LoRA, SigmaShift 6 / 3, and Euler / simple generates the video once. The original image is the start frame, the outfit-A image guides frame 108 (4.5 s), and the outfit-B image is the end frame.
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
  - **Example runs** (one run per input; executed concurrently on separate GPUs, so timings are illustrative rather than a speed comparison):
    - Photo input, 608×1056, RTX 4070 12 GB: **453.28 s**.
    - Anime input, 640×960, RTX 3060 12 GB: **787.95 s**.
  - **Known limitations**: Outfit boundaries and timing can vary by generation. In the tested anime output, a small hair accessory and cloud details changed; these two inputs do not establish a guarantee of exact identity or background preservation.

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

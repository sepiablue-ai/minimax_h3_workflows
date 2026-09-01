# MiniMax-H3 ComfyUI Workflows

ComfyUI workflows for MiniMax-H3 (Ref2VA / FL2VA / VideoRef / Pose Control) generation at Full HD resolution, optimized for low VRAM environments (e.g., 12GB VRAM).

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
# MiniMax-H3 ComfyUI Workflows

ComfyUI workflows for MiniMax-H3 (Ref2VA / FL2VA) generation at Full HD resolution, optimized for low VRAM environments (e.g., 12GB VRAM).

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

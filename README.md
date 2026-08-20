# MiniMax-H3 ComfyUI Workflows

MiniMax-H3 Reference-to-Video (Ref2V / Ref2VA) workflows optimized for low VRAM environments (e.g., 12GB VRAM).

## Features
- **Optimized for 12GB VRAM**: Utilizes `MiniMaxLowVRAMAttention`, `MiniMaxChunkFeedForward`, and `Spectrum` optimization.
- **Turbo Support**: 4-step LoRA configuration for fast generation.
- **Reference-to-Video**: Multi-reference support (character identity preservation with prompt instructions).

## Workflows
- `minimax_h3_ref2va_vram12gb_fdh.json`: ComfyUI Web UI workflow
- `minimax_h3_ref2va_vram12gb_fdh_api.json`: ComfyUI API format workflow

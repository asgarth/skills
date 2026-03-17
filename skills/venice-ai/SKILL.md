---
name: venice-ai
version: "1.0.0"
description: Generate and manipulate images and videos using Venice.ai's privacy-first, uncensored AI API. Use when the user needs to create images from text prompts, edit/inpaint existing images, upscale/enhance image quality, remove backgrounds, or generate videos from text or images. Triggers on requests like "generate an image of...", "upscale this photo", "remove background", "create a video from this image", etc.
tags: ["image", "video", "ai", "venice", "generation", "editing"]
requirements: ["VENICE_API_KEY"]
---

# Venice AI

Generate and manipulate images and videos using Venice.ai's privacy-first API.

## Overview

Venice.ai provides uncensored, private AI image and video generation with no data retention. The API is OpenAI-compatible with additional Venice-specific features for advanced image editing and video generation.

**Base URL:** `https://api.venice.ai/api/v1`

**Authentication:** Bearer token via `Authorization: Bearer VENICE_API_KEY` header

## When to Use

- Generate images from text prompts (e.g., "create an image of a sunset over Venice")
- Edit or inpaint existing images (e.g., "remove the background", "change the shirt color")
- Upscale or enhance image quality (e.g., "make this photo 4K")
- Remove backgrounds from product photos
- Generate videos from images or text prompts
- User requests privacy-first, uncensored AI generation

## When Not to Use

- For text-only tasks (use appropriate LLM skills instead)
- For audio generation (Venice doesn't support this)
- For real-time interactive editing (API has latency)
- When user needs vector graphics (Venice produces raster images only)
- When budget is critical - use cost-effective models and check quotes first

## Quick Start

Set your API key:
```bash
export VENICE_API_KEY="your-api-key"
```

**CLI Examples (using provided scripts):**

```bash
# Generate image with cost-effective defaults
python scripts/generate_image.py "A serene canal in Venice at sunset"

# Generate premium quality image
python scripts/generate_image.py "Professional product photography" --model nano-banana-pro --resolution 2K

# Generate portrait-oriented image
python scripts/generate_image.py "Portrait photo" --aspect-ratio 9:16

# Generate multiple variants
python scripts/generate_image.py "Mountain landscape" --variants 4

# Edit an image
python scripts/edit_image.py image.png "Change the sky to golden sunset"

# Upscale an image
python scripts/upscale_image.py image.png --scale 4 --enhance

# Remove background
python scripts/remove_background.py product.png --output product_nobg.png

# Generate video (will prompt for confirmation)
python scripts/generate_video.py input.png "Gentle waves at sunset"
```

## Core Capabilities

### 1. Image Generation

**Endpoint:** `POST /image/generate`

| Model | Type | Notes |
|-------|------|-------|
| `z-image-turbo` | Cost-effective | Default, fast |
| `flux-2-dev` | Cost-effective | Good quality |
| `nano-banana-pro` | Premium | Best photorealism, 2K/4K |
| `flux-2-max` | Premium | High quality |

**Key Parameters:**
- `prompt` (required) - Text description (max 7500 chars)
- `model` - Model ID (default: `z-image-turbo`)
- `width/height` - Output dimensions (max 1280)
- `aspect_ratio` - e.g., `1:1`, `16:9`, `9:16`
- `resolution` - `1K`, `2K`, `4K` (premium models only)
- `cfg_scale` - 0-20, higher = more prompt adherence
- `hide_watermark` - Set to `True` for commercial use

### 2. Image Editing

**Endpoint:** `POST /image/edit`

- `qwen-edit` (default) - $0.04/edit
- `flux-2-max-edit`, `nano-banana-pro-edit`

**Best Practices:** Keep prompts short: "remove the tree", "change shirt color to red"

### 3. Image Upscaling

**Endpoint:** `POST /image/upscale`

- `scale` - 1 to 4
- `enhance` - Apply AI enhancement
- `enhanceCreativity` - 0-1, higher = more changes

### 4. Background Removal

**Endpoint:** `POST /image/remove-background`

Simply pass the image - no parameters needed.

### 5. Video Generation

**⚠️ IMPORTANT: Always get quote and user confirmation before generating video!**

1. Get quote via `POST /video/quote`
2. Show cost to user and confirm
3. Queue via `POST /video/queue`
4. Poll `POST /video/retrieve` until complete

**Models:** `wan-2.5-preview-image-to-video`, `wan-2.5-preview-text-to-video`

## Error Handling

| Status | Meaning |
|--------|---------|
| 400 | Invalid parameters |
| 401 | Authentication failed |
| 402 | Insufficient balance |
| 422 | Content policy violation |
| 429 | Rate limit exceeded |
| 500 | Inference failed |
| 503 | Model at capacity |

**Headers to Monitor:**
- `x-venice-is-content-violation` - Content flagged
- `x-venice-balance-usd` - Account balance
- `x-ratelimit-remaining-requests` - Rate limit status

## Output Quality

**Minimum:** Images 1024x576, Videos 480p

**Recommended for production:**
- Images: `nano-banana-pro` + 2K resolution
- Videos: 720p minimum with audio
- Format: PNG (images), MP4 (videos)

## Assumptions

- `VENICE_API_KEY` environment variable is set
- Input images are <25MB, minimum 65536 pixels
- User has sufficient account balance
- For video: user confirmed cost before generation

## Anti-Patterns

- Don't use video for quick iterations (test with images first)
- Don't upscale already-high-res images
- Don't skip the quote step for video
- Don't use premium models for bulk generation
- Don't forget `hide_watermark: True` for commercial use

## Resources

- `references/api_reference.md` - Complete API documentation
- `references/models.md` - Model specifications
- `scripts/` - Example scripts for all operations

## Tips

1. **Prompt Engineering:** Be specific, use style keywords, include lighting/composition
2. **Editing:** Keep prompts short, apply sequentially, save intermediates
3. **Video:** Get quote first, use reference images, poll every 5-10s
4. **Performance:** Use `webp` for smaller files, cache results
5. **Safety:** Check `x-venice-is-blurred` header for policy violations

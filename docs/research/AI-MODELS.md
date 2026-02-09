# AI Models Research

> **Last Updated**: 2026-02-09  
> **Related**: [TECH-STACK.md](../architecture/TECH-STACK.md) | [PROVIDER-ADAPTER.md](../architecture/PROVIDER-ADAPTER.md) | [PROMPT-ENGINEERING.md](./PROMPT-ENGINEERING.md)

---

## Purpose

This document summarizes research on virtual try-on AI models, comparing options for the Try on Haul MVP.

---

## Executive Summary

**Architecture**: Provider Adapter Pattern (see [PROVIDER-ADAPTER.md](../architecture/PROVIDER-ADAPTER.md))

**Provider Priority**:
1. **fal.ai (CatVTON/IDM-VTON)** - Primary development provider
2. **FASHN API** - Commercial backup, production fallback
3. **Self-hosted** - Future option for cost optimization

**Key Design Decisions**:
- Adapter interface allows hot-swapping providers without code changes
- Cost tracking hooks for breakeven analysis
- A/B testing support for quality comparisons
- Automatic failover on provider errors

**Critical Licensing Note**:
⚠️ CatVTON and IDM-VTON have **non-commercial licenses**. Using via fal.ai does NOT grant commercial rights. Must either:
- Contact model authors for commercial licensing, OR
- Use FASHN API (commercial license included)

---

## Model Comparison

### Tier 1: Recommended for MVP

| Model | CatVTON | IDM-VTON |
|-------|---------|----------|
| **GitHub Stars** | 1,587 | 4,858 |
| **Open Source** | Yes (CC BY-NC-SA 4.0) | Yes (Non-commercial) |
| **API Available** | fal.ai | Replicate, HuggingFace |
| **Face/Body Accuracy** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Speed** | 11-35 seconds | 17-70 seconds |
| **VRAM Required** | <8GB | High (SDXL-based) |
| **Garment Types** | All (tops, bottoms, dresses) | All |
| **Cost per Image** | ~$0.06 (fal.ai) | ~$0.11 (Replicate) |
| **Best For** | Shape/structure accuracy | Texture/detail accuracy |

### Tier 2: Alternatives

| Model | OOTDiffusion | HR-VITON |
|-------|--------------|----------|
| **GitHub Stars** | 6,514 | 911 |
| **Open Source** | Yes | Yes |
| **API Available** | Replicate | No |
| **Face/Body Accuracy** | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Speed** | 46-110 seconds | Unknown |
| **Limitations** | No lower body support | Outdated (2022) |
| **Recommendation** | Not recommended | Academic only |

### Tier 3: Commercial APIs

| Model | FASHN API | Kling Virtual Try-On |
|-------|-----------|---------------------|
| **Open Source** | No | No |
| **API Available** | Yes (fashn.ai) | Yes (fal.ai, Pixazo) |
| **Face/Body Accuracy** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Speed** | ~7 seconds | Fast |
| **Cost per Image** | $0.075 | Varies |
| **Commercial License** | Included | Included |
| **Best For** | Production at scale | Alternative option |

---

## Detailed Model Analysis

### CatVTON (Recommended)

**Architecture**: Single UNet with FLUX.1-dev-fill foundation

**Strengths**:
- Lightest weight model (<8GB VRAM)
- Best shape/structure accuracy
- Fastest among open-source options
- Supports all garment categories
- SOTA on VITON-HD benchmark (Nov 2024)
- Active development (ICLR 2025)

**Weaknesses**:
- Slightly less detailed textures than IDM-VTON
- No pose information used (relies on image features)
- Non-commercial license (requires negotiation)

**Training Details**:
- 899M total parameters
- Only 49M trainable (efficient fine-tuning)
- Works at 1024×768 resolution

**API Access**:
- fal.ai: https://fal.ai/models/fal-ai/cat-vton
- Cost: ~$0.005/second (~$0.06/image)

**Links**:
- GitHub: https://github.com/Zheng-Chong/CatVTON
- Paper: ICLR 2025

---

### IDM-VTON (Backup)

**Architecture**: Dual-UNet (Main + GarmentNet) with IP-Adapter

**Strengths**:
- Best texture/color accuracy
- SDXL backbone (high quality)
- Strong detail preservation (buttons, patterns)
- Well-documented with multiple endpoints
- Large community (1.3M runs on Replicate)

**Weaknesses**:
- Slower than CatVTON
- Higher VRAM requirements
- Manual masking needed for pants
- Requires 3:4 aspect ratio input

**API Access**:
- Replicate: https://replicate.com/cuuupid/idm-vton
- HuggingFace: https://huggingface.co/spaces/yisol/IDM-VTON
- Cost: ~$0.11/run on Replicate

**Links**:
- GitHub: https://github.com/yisol/IDM-VTON

---

### FASHN API (Commercial Fallback)

**Architecture**: Proprietary "mystery model"

**Strengths**:
- Fastest (7 seconds)
- Highest quality in tests
- Commercial license included
- Native 1MP resolution
- Production-ready with support

**Weaknesses**:
- Not open-source
- Higher cost ($0.075/image)
- Vendor lock-in

**Pricing**:
- On-demand: $0.075/image
- Free tier: 10 credits
- Starter: $7.50/month (100 images)

**Links**:
- API: https://fashn.ai/products/api

---

## Licensing Considerations

### Critical Warning

Both CatVTON and IDM-VTON have **non-commercial licenses**.

**Options**:
1. **Contact authors** for commercial licensing agreement
2. **Use FASHN API** (commercial license included)
3. **Qualify as research/educational** (not recommended for production)

**Recommended Action**:
- Start MVP with fal.ai CatVTON API
- Contact CatVTON authors immediately for commercial terms
- Have FASHN as backup if licensing fails

---

## Deployment Strategy

### Phase 1: MVP Development (Weeks 1-4)

Use fal.ai CatVTON API:
- Fastest integration
- No infrastructure needed
- ~$0.06/image

### Phase 2: MVP Launch (Weeks 5-8)

Continue fal.ai, monitor costs:
- Track generation volume
- Calculate total API spend
- Evaluate user satisfaction with quality

### Phase 3: Scale (Post-MVP)

Decision point based on volume:
- **Low volume (<5K/month)**: Continue fal.ai
- **Medium volume (5K-50K/month)**: Consider FASHN for speed
- **High volume (>50K/month)**: Self-host CatVTON on GPU

### Self-Hosting Considerations

| Provider | GPU | Cost/Hour | Setup |
|----------|-----|-----------|-------|
| RunPod | A100 | ~$1.89 | Easy |
| Modal | A100 | ~$2.00 | Easy |
| AWS | A10G | ~$1.00 | Complex |

Break-even: ~1,500 images/month at fal.ai prices

---

## ComfyUI Integration

For advanced control or custom workflows:

**CatVTON Node**:
- https://github.com/chflame163/ComfyUI_CatVTON_Wrapper

**IDM-VTON Node**:
- https://github.com/TemryL/ComfyUI-IDM-VTON

**Use Cases**:
- Custom scene blending
- Multi-garment combinations
- Quality tuning

---

## API Implementation Example

### CatVTON via fal.ai

```typescript
import * as fal from "@fal-ai/serverless-client";

fal.config({
  credentials: process.env.FAL_KEY,
});

async function generateTryOn(
  humanImageUrl: string,
  garmentImageUrl: string,
  category: "upper_body" | "lower_body" | "dress"
) {
  const result = await fal.subscribe("fal-ai/cat-vton", {
    input: {
      human_image: humanImageUrl,
      garment_image: garmentImageUrl,
      category: category,
    },
    logs: true,
    onQueueUpdate: (update) => {
      console.log("Status:", update.status);
    },
  });

  return {
    imageUrl: result.images[0].url,
    width: result.images[0].width,
    height: result.images[0].height,
  };
}
```

---

## Quality Benchmarks

### VITON-HD Dataset Results (Nov 2024)

| Model | SSIM ↑ | LPIPS ↓ | FID ↓ |
|-------|--------|---------|-------|
| CatVTON | **0.876** | **0.053** | **8.12** |
| IDM-VTON | 0.864 | 0.061 | 9.34 |
| OOTDiffusion | 0.842 | 0.078 | 11.23 |

**Note**: Higher SSIM is better, lower LPIPS/FID is better

---

## Recommendation Summary

| Criteria | Primary Choice | Backup |
|----------|---------------|--------|
| **MVP Development** | CatVTON (fal.ai) | IDM-VTON (Replicate) |
| **Best Quality** | IDM-VTON | FASHN |
| **Fastest** | FASHN | CatVTON |
| **Cheapest** | CatVTON (self-hosted) | CatVTON (fal.ai) |
| **Commercial Ready** | FASHN | (need license) |

---

## Next Steps

1. [x] Design provider adapter architecture (see [PROVIDER-ADAPTER.md](../architecture/PROVIDER-ADAPTER.md))
2. [ ] Set up fal.ai account and test CatVTON API
3. [ ] Contact CatVTON authors about commercial licensing
4. [ ] Create fallback logic for FASHN API
5. [ ] Define quality acceptance criteria with test images
6. [ ] Benchmark actual generation times
7. [ ] Implement cost tracking for breakeven analysis

---

## Cost Breakeven Analysis Requirement

**Goal**: Determine when self-hosting becomes more cost-effective than API usage.

**Metrics to Track**:
| Metric | Purpose |
|--------|---------|
| Cost per generation by provider | Compare provider costs |
| Monthly generation volume | Forecast infrastructure needs |
| Success rate by provider | Factor failed attempts into cost |
| Generation time by provider | User experience impact |

**Breakeven Calculation**:
- fal.ai CatVTON: ~$0.06/gen
- Self-hosted A100: ~$0.01/gen (at ~$1.89/hour, ~20s/gen)
- **Break-even point**: ~1,500 generations/month

**Decision Triggers**:
- If monthly volume consistently exceeds 5,000 gens → evaluate self-hosting
- If cost exceeds $200/month → prioritize self-hosting research
- Track via cost dashboard (see [PROVIDER-ADAPTER.md](../architecture/PROVIDER-ADAPTER.md#6-cost-tracking-hooks))

---

## Change Log

| Date | Change | Author |
|------|--------|--------|
| 2026-02-09 | Initial AI models research | AI Agent |

# Prompt Engineering for Virtual Try-On

> **Last Updated**: 2026-02-09  
> **Related**: [AI-MODELS.md](./AI-MODELS.md) | [TECH-STACK.md](../architecture/TECH-STACK.md)

---

## Purpose

This document captures best practices for prompting virtual try-on AI models to achieve accurate results while preserving user identity.

---

## Key Insight

**Virtual try-on models rely primarily on IMAGE EMBEDDINGS, not text prompts.**

Unlike general image generation (Stable Diffusion, DALL-E), try-on models extract features from:
1. Reference person photo (for identity)
2. Garment image (for clothing details)
3. Pose conditioning (for realistic draping)

Text prompts are secondary and should be kept simple.

---

## Prompt Templates

### CatVTON / IDM-VTON

**Positive Prompt**:
```
model is wearing {garment_description}
```

Examples:
- `model is wearing a red floral dress`
- `model is wearing a black crop top`
- `model is wearing blue denim jeans`

**Negative Prompt**:
```
monochrome, lowres, bad anatomy, worst quality, low quality,
deformed face, disfigured, distorted hands, extra fingers,
bad proportions, multiple people, blurry
```

### OOTDiffusion

**No text prompt needed** - uses pure vision embeddings.

Only specify category:
- `upperbody`
- `lowerbody`
- `dress`

---

## Negative Prompt Components

### Anatomy Issues
```
bad anatomy, deformed, disfigured, poorly drawn face,
mutation, mutated, extra limbs, ugly, poorly drawn hands,
missing limb, floating limbs, disconnected limbs,
long neck, long body
```

### Quality Issues
```
lowres, low quality, worst quality, blurry, grainy,
jpeg artifacts, compression artifacts
```

### Face Distortions
```
deformed face, distorted face, asymmetrical face,
multiple faces, cropped face, bad eyes, crossed eyes
```

### Hand Issues (Common Problem)
```
bad hands, distorted hands, deformed hands,
extra fingers, missing fingers, fused fingers
```

---

## Model Configuration

### Optimal Parameters

| Parameter | Recommended | Range | Purpose |
|-----------|-------------|-------|---------|
| `guidance_scale` | **2.0** | 1.5-2.5 | Lower = better identity preservation |
| `num_inference_steps` | **30** | 20-50 | Higher = better quality, slower |
| `strength` | **1.0** | 0.85-1.0 | Always high for try-on |
| `height` | **1024** | - | Standard resolution |
| `width` | **768** | - | Standard resolution |

### Why Low Guidance Scale?

High guidance scale (7-12) pushes toward prompt interpretation, losing identity.
Low guidance scale (1.5-2.5) preserves reference person features.

---

## Input Image Requirements

### Reference Photo (Person)

| Requirement | Specification | Why |
|-------------|--------------|-----|
| Resolution | Minimum 768x1024 | Detail preservation |
| Aspect Ratio | 3:4 (portrait) | Model expectation |
| Lighting | Well-lit, even | Accurate skin tone |
| Pose | Frontal or slight angle | Clear body structure |
| Face | Clearly visible | Identity preservation |
| Background | Any (will be preserved) | - |

**Best Practices**:
- Full body or at least torso visible
- Arms visible (not crossed/hidden)
- Natural standing pose
- Minimal occlusion

**Avoid**:
- Extreme angles or poses
- Heavy shadows on face
- Multiple people in frame
- Very dark or overexposed images

### Garment Image

| Requirement | Specification | Why |
|-------------|--------------|-----|
| Resolution | Minimum 768x1024 | Pattern preservation |
| Presentation | Flat lay or on model | Clear garment shape |
| Background | Neutral/white preferred | Easy extraction |
| Lighting | Even, no harsh shadows | Color accuracy |
| Coverage | Full garment visible | Complete transfer |

**Best Practices**:
- Photograph garment flat on neutral surface
- Remove wrinkles before photographing
- Capture full garment (not cropped)
- Good color accuracy (no color cast)

**Avoid**:
- Heavy folds or wrinkles
- Cropped edges
- Complex backgrounds
- Low resolution or blurry images

---

## Identity Preservation Techniques

### IP-Adapter (IDM-VTON)

The IP-Adapter extracts high-level semantic features from the reference photo:
- Facial features
- Skin tone
- Body proportions
- Overall appearance

**Inject via cross-attention layers** in the UNet.

### DensePose Conditioning

Provides body keypoint conditioning:
- Ensures realistic garment draping
- Matches pose of reference person
- Critical for natural results

**Required Input**: DensePose map from reference photo

### Human Parsing Masks

Define exact clothing regions:
- Preserves face, hands, exposed skin
- Only regenerates clothing area
- Prevents artifacts at boundaries

**Generated via**: SCHP (Self-Correction Human Parsing)

---

## Processing Pipeline

### Step-by-Step Flow

```
1. Reference Photo Processing
   ├── Resize to 768x1024
   ├── Extract DensePose map
   ├── Generate human parsing mask
   └── Create agnostic representation

2. Garment Processing
   ├── CLIP preprocessing (semantic features)
   └── Direct tensor (texture features)

3. Generation
   ├── Combine embeddings
   ├── Apply mask inpainting
   └── Denoise with UNet

4. Post-Processing
   └── Composite with original background
```

### Masked Inpainting

```python
# Only regenerate clothing region
mask = 1 - inpaint_mask  # Keep everything except clothing
im_mask = image * mask   # Preserve background, face, hands
```

---

## Common Problems & Solutions

### Problem 1: Face Doesn't Match

**Symptoms**: Generated face looks different from reference

**Solutions**:
- Lower `guidance_scale` to 1.5-2.0
- Ensure reference photo has clear, frontal face
- Use higher resolution reference (1024x1024+)
- Verify IP-Adapter weights loaded correctly

### Problem 2: Garment Details Lost

**Symptoms**: Patterns, logos, textures disappear

**Solutions**:
- Use high-resolution garment image (768x1024+)
- Photograph garment flat with good lighting
- Increase `image_scale` (OOTDiffusion: 1.5-2.5)
- Ensure garment fills significant frame area

### Problem 3: Unnatural Fit

**Symptoms**: Clothing doesn't conform to body

**Solutions**:
- Always use DensePose conditioning
- Ensure visible body structure in reference
- Avoid extreme poses or heavy occlusion
- Use accurate human parsing masks

### Problem 4: Boundary Artifacts

**Symptoms**: Visible seams at clothing edges

**Solutions**:
- Refine human parsing masks (higher quality model)
- Increase `num_inference_steps` to 40-50
- Use mask dilation/erosion to smooth boundaries
- Ensure proper mask alignment

### Problem 5: Skin Tone Mismatch

**Symptoms**: Exposed skin color inconsistent

**Solutions**:
- Preserve original pixels outside clothing region
- Use IP-Adapter for global appearance consistency
- Avoid regenerating non-clothing areas
- Verify inpainting mask accuracy

---

## Scene/Background Blending

### Approach for Try on Haul

Virtual try-on models preserve original background by default.

For festival scene replacement:

**Post-Processing Approach** (Recommended):
1. Generate try-on with original background
2. Use Segment Anything for person isolation
3. Composite onto festival scene
4. Match lighting with color grading

**Pre-Processing Approach**:
1. Replace background in reference first
2. Ensure consistent lighting on person
3. Run try-on on pre-processed image

**Note**: Most try-on models focus on garment replacement. Separating background replacement yields better results.

---

## Quality Checklist

### Before Generation

- [ ] Reference photo: 768x1024+, well-lit, frontal
- [ ] Garment photo: flat, clear, high contrast
- [ ] Both images same aspect ratio (3:4)
- [ ] No heavy compression (PNG preferred)

### Model Configuration

- [ ] `guidance_scale` = 2.0
- [ ] `num_inference_steps` = 30
- [ ] `strength` = 1.0
- [ ] DensePose conditioning enabled
- [ ] IP-Adapter loaded

### Prompts

- [ ] Simple garment description
- [ ] Negative prompt includes anatomy terms
- [ ] No complex scene descriptions

### After Generation

- [ ] Face matches reference person
- [ ] Garment details preserved
- [ ] Natural pose/draping
- [ ] Clean boundaries
- [ ] Consistent skin tone

---

## Code Example

### Complete IDM-VTON Configuration

```python
from inference import TryOnPipeline

# Initialize
pipe = TryOnPipeline.from_pretrained("yisol/IDM-VTON")

# Prepare inputs
inputs = {
    "reference_image": person_photo,      # 768x1024
    "garment_image": clothing_photo,       # 768x1024
    "pose_image": densepose_output,        # DensePose map
    "mask": human_parsing_mask,            # SCHP output
}

# Generate
result = pipe(
    prompt="model is wearing a blue summer dress",
    negative_prompt="monochrome, lowres, bad anatomy, worst quality",
    guidance_scale=2.0,
    num_inference_steps=30,
    strength=1.0,
    **inputs
)

# Output
generated_image = result.images[0]
```

---

## Advanced Techniques

### Multi-Scale Feature Fusion

IDM-VTON uses dual conditioning:
- **High-level** (IP-Adapter): Global style, color, semantics
- **Low-level** (GarmentNet): Texture, patterns, fine details

Both fused in self-attention layers.

### Classifier-Free Guidance Dropout

During inference, vary `guidance_scale`:
- Lower values (1.5): More faithful to reference
- Higher values (2.5): Stronger garment details

### Progressive Masking

For partial try-on (e.g., only change top):

```python
# Selective mask for upper body only
mask = np.zeros_like(full_parsing_mask)
mask[parsing_mask == UPPER_BODY_LABEL] = 1
# Lower body and background unchanged
```

---

## Change Log

| Date | Change | Author |
|------|--------|--------|
| 2026-02-09 | Initial prompt engineering guide | AI Agent |

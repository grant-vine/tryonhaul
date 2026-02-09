# Try on Haul - Design Guidelines

> **Audience**: UI/UX designers, product designers, brand designers  
> **Last Updated**: 2026-02-09

---

## Brand Overview

### Brand Essence
**"Festival magic, made easy."**

Try on Haul brings the excitement and self-expression of festival fashion to everyone—before they even get to the festival.

### Brand Personality

| Attribute | Expression |
|-----------|------------|
| **Playful** | Fun animations, celebratory moments, delightful interactions |
| **Bold** | Vibrant colors, confident typography, statement visuals |
| **Inclusive** | All bodies, all styles, welcoming imagery |
| **Magical** | AI feels like magic, not technology |

---

## Visual Identity

### Color Palette

#### Primary Colors
| Color | Hex | Usage |
|-------|-----|-------|
| **Electric Purple** | `#8B5CF6` | Primary actions, brand accent |
| **Hot Pink** | `#EC4899` | Secondary accent, highlights |
| **Deep Black** | `#0F0F0F` | Text, backgrounds |
| **Pure White** | `#FFFFFF` | Backgrounds, text on dark |

#### Secondary Colors (Festival Vibes)
| Color | Hex | Usage |
|-------|-----|-------|
| **Desert Gold** | `#F59E0B` | Burning Man theme |
| **Ocean Teal** | `#14B8A6` | Coachella theme |
| **Neon Green** | `#22C55E` | EDC theme |
| **Sunset Orange** | `#F97316` | Warm accents |

#### Gradients
```css
/* Primary Gradient - Hero sections, CTAs */
background: linear-gradient(135deg, #8B5CF6 0%, #EC4899 100%);

/* Dark Gradient - Backgrounds */
background: linear-gradient(180deg, #0F0F0F 0%, #1F1F1F 100%);

/* Festival Gradient - Special moments */
background: linear-gradient(90deg, #F59E0B, #EC4899, #8B5CF6, #14B8A6);
```

---

### Typography

#### Font Stack
```css
/* Headings */
font-family: 'Plus Jakarta Sans', -apple-system, BlinkMacSystemFont, sans-serif;

/* Body */
font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;

/* Monospace (technical) */
font-family: 'JetBrains Mono', 'SF Mono', monospace;
```

#### Type Scale
| Level | Size | Weight | Line Height | Usage |
|-------|------|--------|-------------|-------|
| H1 | 48px / 3rem | 700 | 1.1 | Page titles |
| H2 | 36px / 2.25rem | 600 | 1.2 | Section headers |
| H3 | 24px / 1.5rem | 600 | 1.3 | Card titles |
| Body | 16px / 1rem | 400 | 1.5 | Paragraphs |
| Small | 14px / 0.875rem | 400 | 1.4 | Captions, helper text |
| Tiny | 12px / 0.75rem | 500 | 1.3 | Badges, labels |

---

### Spacing System

Use 4px base unit:

| Token | Value | Usage |
|-------|-------|-------|
| `space-1` | 4px | Tight spacing |
| `space-2` | 8px | Element padding |
| `space-3` | 12px | Small gaps |
| `space-4` | 16px | Default gap |
| `space-6` | 24px | Section padding |
| `space-8` | 32px | Large gaps |
| `space-12` | 48px | Section margins |
| `space-16` | 64px | Page sections |

---

### Border Radius

| Token | Value | Usage |
|-------|-------|-------|
| `rounded-sm` | 4px | Subtle rounding |
| `rounded` | 8px | Default (buttons, inputs) |
| `rounded-lg` | 12px | Cards, modals |
| `rounded-xl` | 16px | Featured elements |
| `rounded-full` | 9999px | Pills, avatars |

---

## Component Guidelines

### Buttons

#### Primary Button
```
Background: Electric Purple (#8B5CF6)
Text: White
Hover: Lighter purple + subtle glow
Active: Darker purple
Border-radius: 8px
Padding: 12px 24px
Font: 16px, 600 weight
```

#### Secondary Button
```
Background: Transparent
Border: 1px solid white/20
Text: White
Hover: White/10 background
```

#### Ghost Button
```
Background: Transparent
Text: Purple
Hover: Purple/10 background
```

### Cards

```
Background: #1F1F1F (dark mode)
Border: 1px solid white/10
Border-radius: 12px
Padding: 16px
Shadow: 0 4px 24px rgba(0,0,0,0.3)
```

### Inputs

```
Background: #1F1F1F
Border: 1px solid white/20
Border-radius: 8px
Padding: 12px 16px
Focus: Purple border, subtle glow
Error: Red border
```

---

## Iconography

### Style Guidelines
- **Weight**: 1.5px stroke (outline style)
- **Size**: 24px default, 20px compact, 32px large
- **Color**: Inherit from text color
- **Library**: Lucide Icons (recommended)

### Common Icons
| Action | Icon |
|--------|------|
| Upload photo | `Camera` / `Upload` |
| Add outfit | `Shirt` / `Plus` |
| Generate | `Sparkles` / `Wand` |
| Share | `Share2` |
| Download | `Download` |
| Settings | `Settings` |
| Close | `X` |
| Back | `ArrowLeft` |

---

## Motion & Animation

### Principles
1. **Purposeful**: Animation should guide, not distract
2. **Quick**: Keep animations under 300ms
3. **Smooth**: Use ease-out for entering, ease-in for exiting
4. **Delightful**: Celebrate moments (successful generation, shares)

### Timing
| Type | Duration | Easing |
|------|----------|--------|
| Micro-interactions | 100-150ms | ease-out |
| Transitions | 200-300ms | ease-out |
| Modals | 250ms | ease-out |
| Page transitions | 300-400ms | ease-in-out |

### Signature Animations

**Generation Loading**
- Shimmering gradient pulse
- Subtle particle effects
- Progress percentage

**Success Moment**
- Confetti burst
- Scale up + fade in
- Celebratory haptic (mobile)

---

## Dark Mode (Primary)

Try on Haul is **dark-mode first**. The aesthetic suits festival vibes and makes images pop.

### Background Layers
| Layer | Color |
|-------|-------|
| Base | `#0F0F0F` |
| Surface | `#1F1F1F` |
| Elevated | `#2A2A2A` |
| Overlay | `rgba(0,0,0,0.8)` |

### Text Colors
| Type | Color |
|------|-------|
| Primary | `#FFFFFF` |
| Secondary | `rgba(255,255,255,0.7)` |
| Muted | `rgba(255,255,255,0.5)` |
| Disabled | `rgba(255,255,255,0.3)` |

---

## Responsive Breakpoints

| Breakpoint | Width | Target |
|------------|-------|--------|
| `sm` | 640px | Large phones (landscape) |
| `md` | 768px | Tablets |
| `lg` | 1024px | Small laptops |
| `xl` | 1280px | Desktops |
| `2xl` | 1536px | Large screens |

### Mobile-First Approach
Design for 375px width first, then scale up.

---

## Key Screens

### 1. Landing Page
- Hero with try-on demo
- Simple 3-step explanation
- Social proof (user-generated content)
- CTA: "Try It Free"

### 2. Upload Flow
- Two-pane layout (mobile: stacked)
- Left: Your photo
- Right: Outfit image
- Bottom: Scene selection (optional)

### 3. Generation Progress
- Centered loading state
- Progress percentage
- Engaging copy ("Working our magic...")
- Cancel option

### 4. Result View
- Full-screen image preview
- Pinch-to-zoom (mobile)
- Comparison slider (before/after)
- Action bar: Download, Share, Try Again

### 5. Share Flow
- Share preview card
- Platform selection (or native share sheet)
- Caption editing
- Copy link fallback

---

## Accessibility Requirements

### WCAG 2.1 Level A (Minimum)

| Requirement | Implementation |
|-------------|----------------|
| Color contrast | 4.5:1 for text, 3:1 for large text |
| Focus indicators | Visible focus ring on all interactive elements |
| Touch targets | Minimum 44×44px |
| Alt text | All images have descriptive alt text |
| Keyboard navigation | All functions accessible via keyboard |
| Reduced motion | Respect `prefers-reduced-motion` |

---

## Image Guidelines

### User Photos
- Aspect ratio: 3:4 (portrait)
- Minimum resolution: 768×1024
- Display: Fill container, object-fit: cover

### Outfit Images
- Aspect ratio: Flexible
- Prefer: Product shots on white/neutral background
- Display: Contain within frame

### Generated Results
- Aspect ratio: Match user photo
- Export quality: High (80% JPEG or WebP)
- Branding: Subtle watermark in corner

### Scene Images
- Aspect ratio: 16:9 or panoramic
- Style: Atmospheric, slightly blurred
- Resolution: 1920px wide minimum

---

## Design Assets

### Logo
- Primary: Wordmark "Try on Haul" + icon
- Icon only: For small spaces, favicon
- Colors: White on dark, or purple gradient

### Social Templates
- Instagram post: 1080×1080
- Instagram story: 1080×1920
- TikTok: 1080×1920
- Open Graph: 1200×630

---

## Figma Structure (Recommended)

```
📁 Try on Haul Design System
├── 🎨 Foundations
│   ├── Colors
│   ├── Typography
│   ├── Spacing
│   └── Effects
├── 🧱 Components
│   ├── Buttons
│   ├── Forms
│   ├── Cards
│   ├── Navigation
│   └── Modals
├── 📱 Screens
│   ├── Marketing
│   ├── Auth
│   ├── Core Flow
│   └── Settings
└── 🖼️ Assets
    ├── Icons
    ├── Illustrations
    └── Photos
```

---

## Contact

**Design Questions**  
design@tryonhaul.com

**Brand Assets Requests**  
brand@tryonhaul.com

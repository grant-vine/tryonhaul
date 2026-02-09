# Design Standards

> **Last Updated**: 2026-02-09  
> **Related**: [AGENTS.md](../../AGENTS.md) | [DEV-STANDARDS.md](./DEV-STANDARDS.md)

---

## Purpose

This document defines UI/UX guidelines, component standards, and visual design principles for Try on Haul.

---

## Brand Identity

### Visual Theme

**Festival/Rave Aesthetic**:
- Vibrant, energetic colors
- Modern, bold typography
- Dynamic gradients and glows
- Mobile-first, thumb-friendly
- Dark mode friendly (users browse at night)

### Mood Keywords

- Exciting
- Confident
- Trendy
- Social
- Authentic

---

## Color System

### Primary Palette

> **TODO**: Finalize exact hex values during design phase

| Token | Purpose | Light Mode | Dark Mode |
|-------|---------|------------|-----------|
| `--primary` | CTAs, highlights | TBD | TBD |
| `--primary-hover` | Hover states | TBD | TBD |
| `--secondary` | Secondary actions | TBD | TBD |
| `--accent` | Neon/glow effects | TBD | TBD |

### Semantic Colors

| Token | Purpose | Light Mode | Dark Mode |
|-------|---------|------------|-----------|
| `--background` | Page background | `#ffffff` | `#0a0a0a` |
| `--foreground` | Primary text | `#0a0a0a` | `#ffffff` |
| `--muted` | Secondary text | `#737373` | `#a3a3a3` |
| `--border` | Dividers, outlines | `#e5e5e5` | `#262626` |
| `--success` | Positive states | `#22c55e` | `#22c55e` |
| `--warning` | Caution states | `#f59e0b` | `#f59e0b` |
| `--error` | Error states | `#ef4444` | `#ef4444` |

### Gradient System

```css
/* Festival gradient (for hero sections) */
--gradient-festival: linear-gradient(135deg, #ff006e, #8338ec, #3a86ff);

/* Neon glow (for CTAs) */
--gradient-neon: linear-gradient(90deg, #00f5d4, #00bbf9);
```

---

## Typography

### Font Stack

> **TODO**: Select specific font during design phase

**Recommendations**:
- **Display**: Inter, Outfit, or custom
- **Body**: System fonts for performance

```css
--font-display: 'Inter', system-ui, sans-serif;
--font-body: system-ui, -apple-system, sans-serif;
--font-mono: 'SF Mono', monospace;
```

### Type Scale

| Token | Size | Line Height | Usage |
|-------|------|-------------|-------|
| `--text-xs` | 12px | 16px | Captions, labels |
| `--text-sm` | 14px | 20px | Secondary text |
| `--text-base` | 16px | 24px | Body text |
| `--text-lg` | 18px | 28px | Lead text |
| `--text-xl` | 20px | 28px | Section headers |
| `--text-2xl` | 24px | 32px | Page headers |
| `--text-3xl` | 30px | 36px | Hero text |
| `--text-4xl` | 36px | 40px | Display text |

### Font Weights

| Token | Weight | Usage |
|-------|--------|-------|
| `--font-normal` | 400 | Body text |
| `--font-medium` | 500 | Emphasis |
| `--font-semibold` | 600 | Subheadings |
| `--font-bold` | 700 | Headings, CTAs |

---

## Spacing System

### Base Scale

Use 4px base unit:

| Token | Value | Usage |
|-------|-------|-------|
| `--space-1` | 4px | Tight spacing |
| `--space-2` | 8px | Element gaps |
| `--space-3` | 12px | Component padding |
| `--space-4` | 16px | Section gaps |
| `--space-5` | 20px | Medium spacing |
| `--space-6` | 24px | Large gaps |
| `--space-8` | 32px | Section padding |
| `--space-10` | 40px | Page margins |
| `--space-12` | 48px | Large sections |
| `--space-16` | 64px | Hero spacing |

---

## Layout

### Breakpoints

Mobile-first responsive design:

| Token | Width | Target |
|-------|-------|--------|
| `sm` | 640px | Large phones |
| `md` | 768px | Tablets |
| `lg` | 1024px | Laptops |
| `xl` | 1280px | Desktops |
| `2xl` | 1536px | Large screens |

### Container Widths

```css
.container {
  width: 100%;
  margin: 0 auto;
  padding: 0 var(--space-4);
}

@media (min-width: 640px) {
  .container { max-width: 640px; }
}

@media (min-width: 768px) {
  .container { max-width: 768px; }
}

@media (min-width: 1024px) {
  .container { max-width: 1024px; }
}
```

### Grid System

Use CSS Grid or Flexbox, not fixed columns:

```css
/* Image grid (responsive) */
.image-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: var(--space-4);
}
```

---

## Components

### Button Hierarchy

| Variant | Usage | Style |
|---------|-------|-------|
| `primary` | Main CTAs | Solid, gradient, bold |
| `secondary` | Alternative actions | Outlined |
| `ghost` | Tertiary actions | Text only |
| `destructive` | Delete, cancel | Red accent |

### Button Sizes

| Size | Height | Padding | Font |
|------|--------|---------|------|
| `sm` | 32px | 12px 16px | 14px |
| `md` | 40px | 12px 20px | 16px |
| `lg` | 48px | 16px 24px | 18px |
| `xl` | 56px | 20px 32px | 20px |

### Touch Targets

**Minimum 44x44px** for all interactive elements (WCAG requirement)

```css
.touch-target {
  min-width: 44px;
  min-height: 44px;
}
```

---

## Iconography

### Icon Library

Use **Lucide React** (consistent, customizable, tree-shakeable)

```typescript
import { Upload, Camera, Share2, Download } from 'lucide-react';

<Upload size={24} strokeWidth={2} />
```

### Icon Sizes

| Context | Size |
|---------|------|
| Inline with text | 16px |
| Buttons | 20px |
| Navigation | 24px |
| Feature highlights | 32px |
| Empty states | 48px |

---

## Motion & Animation

### Timing Functions

```css
--ease-default: cubic-bezier(0.4, 0, 0.2, 1);
--ease-in: cubic-bezier(0.4, 0, 1, 1);
--ease-out: cubic-bezier(0, 0, 0.2, 1);
--ease-bounce: cubic-bezier(0.34, 1.56, 0.64, 1);
```

### Duration Scale

| Token | Duration | Usage |
|-------|----------|-------|
| `--duration-fast` | 100ms | Micro-interactions |
| `--duration-normal` | 200ms | Standard transitions |
| `--duration-slow` | 300ms | Page transitions |
| `--duration-slower` | 500ms | Complex animations |

### Animation Guidelines

- **Enter**: Fade in + slight upward motion
- **Exit**: Fade out + slight downward motion
- **Loading**: Subtle pulse or skeleton shimmer
- **Success**: Brief scale pop or check animation
- **Avoid**: Excessive motion (accessibility)

### Reduced Motion

```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## User Experience Patterns

### Core Flow (Maximum 3 Steps)

```
1. Upload Photo → 2. Add Outfit → 3. Generate & Share
```

Each step should be:
- Clearly numbered/indicated
- Completable in under 30 seconds
- Recoverable if interrupted

### Loading States

| State | Visual | Duration |
|-------|--------|----------|
| Button loading | Spinner inside button | < 2s |
| Page loading | Skeleton screens | < 3s |
| Generation | Progress bar + status text | 10-60s |

### Error Handling

| Error Type | Treatment |
|------------|-----------|
| Validation | Inline message near field |
| API error | Toast notification |
| Fatal error | Full-page with retry |
| Network | Banner with reconnection |

### Empty States

Always include:
- Friendly illustration or icon
- Explanatory text
- Clear CTA to resolve

---

## Accessibility (a11y)

### WCAG 2.1 Level A Requirements

| Requirement | Implementation |
|-------------|---------------|
| Color contrast | 4.5:1 for text, 3:1 for UI |
| Focus indicators | Visible outline on all interactive elements |
| Alt text | Descriptive alt for all images |
| Keyboard navigation | Tab order logical, no traps |
| Form labels | All inputs have associated labels |
| Error identification | Errors identified beyond color |

### Focus Styles

```css
:focus-visible {
  outline: 2px solid var(--primary);
  outline-offset: 2px;
}
```

### ARIA Patterns

```tsx
// Button with loading state
<button 
  aria-busy={isLoading} 
  aria-disabled={isLoading}
>
  {isLoading ? <Spinner aria-hidden /> : null}
  {isLoading ? 'Generating...' : 'Generate'}
</button>

// Image with meaningful description
<Image 
  src={resultUrl} 
  alt="Your try-on result wearing a blue summer dress"
/>
```

---

## Form Design

### Input Fields

```css
.input {
  height: 44px;
  padding: 0 var(--space-3);
  border: 1px solid var(--border);
  border-radius: 8px;
  font-size: var(--text-base);
}

.input:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 3px rgba(var(--primary-rgb), 0.1);
}

.input.error {
  border-color: var(--error);
}
```

### Validation Feedback

- Show validation on blur (not on every keystroke)
- Error messages below field in red
- Success state with green checkmark
- Helper text in muted color

---

## Social Sharing UI

### Share Button Group

Horizontal row of platform icons:
- Facebook
- Instagram
- TikTok
- X (Twitter)
- Pinterest
- Copy Link

### Share Card Preview

Show preview of what will be shared:
- Generated image
- Caption text
- Hashtags
- Short link

---

## Dark Mode

### Implementation

Use CSS custom properties with media query:

```css
:root {
  --background: #ffffff;
  --foreground: #0a0a0a;
}

@media (prefers-color-scheme: dark) {
  :root {
    --background: #0a0a0a;
    --foreground: #ffffff;
  }
}
```

### Dark Mode Considerations

- Test all gradients in dark mode
- Reduce brightness of neon effects
- Ensure images have appropriate contrast
- Test on actual dark screens

---

## Design Tokens (Tailwind)

```typescript
// tailwind.config.ts
export default {
  theme: {
    extend: {
      colors: {
        primary: 'hsl(var(--primary))',
        secondary: 'hsl(var(--secondary))',
        accent: 'hsl(var(--accent))',
        // ... semantic colors
      },
      fontFamily: {
        display: ['var(--font-display)'],
        body: ['var(--font-body)'],
      },
      animation: {
        'pulse-glow': 'pulse-glow 2s ease-in-out infinite',
      },
    },
  },
};
```

---

## Design Review Checklist

Before shipping UI:

- [ ] Works on mobile (320px width)
- [ ] Works on desktop (1440px width)
- [ ] Passes color contrast check
- [ ] All interactive elements have focus states
- [ ] Loading states implemented
- [ ] Error states implemented
- [ ] Touch targets meet 44px minimum
- [ ] Tested with reduced motion
- [ ] Tested in dark mode
- [ ] Images have alt text

---

## Change Log

| Date | Change | Author |
|------|--------|--------|
| 2026-02-09 | Initial design standards | AI Agent |

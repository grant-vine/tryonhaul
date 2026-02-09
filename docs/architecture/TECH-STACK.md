# Tech Stack Decisions

> **Last Updated**: 2026-02-09  
> **Related**: [OVERVIEW.md](./OVERVIEW.md) | [DATA-FLOW.md](./DATA-FLOW.md) | [AI-MODELS.md](../research/AI-MODELS.md)

---

## Purpose

This document captures all technology decisions for Try on Haul with rationale and tradeoffs.

---

## Decision Summary

| Category | Decision | Alternatives Considered |
|----------|----------|------------------------|
| Frontend | Next.js 16 (App Router) | SvelteKit, Remix |
| PWA | @serwist/next | next-pwa (abandoned) |
| Authentication | Clerk | NextAuth, Auth0, Supabase |
| Background Jobs | Inngest | Trigger.dev, Upstash QStash |
| AI Generation | CatVTON via fal.ai | IDM-VTON, FASHN API |
| Hosting | Vercel | Netlify, Cloudflare |
| Image Processing | next/image + Sharp | Cloudinary, Imgix |
| Analytics | Plausible | PostHog, Google Analytics |
| Database | Vercel KV (minimal) | Supabase, PlanetScale |

---

## 1. Frontend Framework

### Decision: Next.js 16 (App Router)

**Why Next.js:**
- Native Vercel deployment (zero config)
- Built-in PWA support via manifest generation
- Best-in-class image optimization (critical for AI-generated images)
- React 19 Server Components for performance
- Largest ecosystem for social auth integrations
- Edge runtime support for fast API responses

**Why NOT SvelteKit:**
- Smaller auth provider ecosystem
- Less mature PWA tooling
- Fewer AI SDK integrations
- Smaller community for troubleshooting

**Why NOT Remix:**
- Remix 3 abandoned React entirely
- React Router 7 merger caused ecosystem fragmentation
- Uncertain future direction

---

## 2. PWA Implementation

### Decision: @serwist/next (v9.5.5)

**Why Serwist:**
- Active maintenance (latest: Feb 2026)
- Successor to abandoned `next-pwa`
- Next.js 14+ App Router support
- TypeScript-first
- Battle-tested (used by dailydotdev, lobehub)
- Works with Turbopack

**Tradeoffs:**
- Service worker debugging complexity
- Requires separate `sw.ts` file

---

## 3. Authentication

### Decision: Clerk

**Why Clerk:**
- Pre-configured OAuth for development (zero setup)
- Native TikTok integration (critical for our target audience)
- Instagram via Facebook Graph API
- Built-in user management UI
- Handles OAuth complexity

**Supported Providers:**
| Provider | Status | Notes |
|----------|--------|-------|
| Facebook | ✅ Native | Full Graph API access |
| Instagram | ✅ via FB | Requires Facebook Graph API |
| TikTok | ✅ Native | Login + sharing |
| Apple | ✅ Native | Required for iOS |

**Why NOT NextAuth:**
- No TikTok provider
- No Instagram provider
- Manual OAuth setup complexity
- More maintenance burden

**Why NOT Auth0:**
- No TikTok provider
- Higher cost ($35/mo vs $25/mo)
- Overkill for social-only auth

**Cost**: $25/month for production

---

## 4. Background Job Processing

### Decision: Inngest

**Why Inngest:**
- Serverless-native (no infrastructure)
- Built-in retries and error handling
- Visual execution timeline for debugging
- Works with Vercel out-of-box
- Step-by-step execution for complex workflows
- Free tier (10k function runs/month)

**Use Cases:**
1. AI image generation (10-30s processing)
2. Notification delivery (push + email)
3. Affiliate link batch processing
4. Image optimization

**Why NOT Trigger.dev:**
- Requires Docker or cloud infrastructure
- Higher cost ($20/mo)
- More setup complexity

**Why NOT Vercel Cron:**
- No retries
- No observability
- Only for scheduled tasks, not event-driven

---

## 4a. Notification Providers

### Decision: Web Push + Resend Email

**Notification Strategy (in priority order):**
| Method | Provider | When | Cost |
|--------|----------|------|------|
| In-app toast | Native | User is on site | Free |
| Browser push | Web Push API | User has granted permission | Free |
| Email | Resend | Fallback when push unavailable | $0/month (free tier) |

**Web Push Implementation:**
- Uses standard Web Push API with service worker
- VAPID keys generated at app setup
- Permission requested after first successful generation
- Payload includes generation result thumbnail

**Email Fallback (Resend):**
- Simple transactional email for generation complete
- Only sent if user doesn't return within 2 minutes
- Includes direct link to result
- Free tier: 3,000 emails/month (sufficient for MVP)

**Why Resend:**
- Free tier adequate for MVP
- Simple API, good DX
- React Email for templates
- No complex SMTP setup

**Why NOT SendGrid/Mailgun:**
- Overkill for simple notifications
- More complex setup
- Higher cost at scale

---

## 5. AI Generation Service

### Decision: Provider Adapter Architecture

> **Full architecture**: [PROVIDER-ADAPTER.md](./PROVIDER-ADAPTER.md)  
> **Model research**: [AI-MODELS.md](../research/AI-MODELS.md)

**Why Multi-Provider Adapter**:
- Swap providers without code changes
- Cost tracking for breakeven analysis
- A/B testing between models
- Automatic failover on provider errors
- Future-proofing for self-hosting

**Provider Priority**:
| Priority | Provider | Models | Use Case |
|----------|----------|--------|----------|
| 1 | fal.ai | CatVTON, IDM-VTON | Primary (development + initial launch) |
| 2 | FASHN API | Proprietary | Commercial backup, production fallback |
| 3 | Self-hosted | CatVTON | Future cost optimization |

**fal.ai CatVTON**:
- Best face/body preservation (critical requirement)
- Lightweight (<8GB VRAM)
- Fast (11-35 seconds per generation)
- Supports all garment types (tops, bottoms, dresses)
- SOTA performance (ICLR 2025)
- Cost: ~$0.06/image

**fal.ai IDM-VTON**:
- Best texture/color accuracy
- Use when quality is priority over speed
- Cost: ~$0.11/image

**FASHN API**:
- Commercial-ready ($0.075/image)
- Fastest (7 seconds)
- **Scene/background support** (unique feature - critical for festival compositing)
- Use as production fallback

**License Warning**:
⚠️ CatVTON and IDM-VTON are non-commercial licenses. For production:
- Contact authors for commercial licensing, OR
- Use FASHN API (commercial license included)

**Cost Tracking**:
- Every generation logged with provider, cost, duration
- Daily/weekly/monthly cost aggregation
- Breakeven analysis: self-host at ~1,500 gens/month

---

## 6. Hosting Platform

### Decision: Vercel

**Why Vercel:**
- Zero-config Next.js deployment
- Built-in edge functions
- KV storage for caching/rate limiting
- Blob storage for temporary images
- Web Analytics included
- Automatic preview deployments

**Why NOT Netlify:**
- Less optimized for Next.js
- Limited edge function capabilities

**Why NOT Cloudflare:**
- More configuration required
- Less Next.js-specific features

**Cost**: ~$20/month (Pro plan)

---

## 7. Storage Strategy

### Ephemeral Storage Approach (No Persistent Image Storage)

| Data Type | Storage Solution | Retention |
|-----------|-----------------|-----------|
| User sessions | Clerk (managed) | Session duration |
| Rate limits | Vercel KV | 24 hours |
| Affiliate links | Vercel KV | Indefinite |
| Temp images (uploads + results) | Vercel Blob | 1 hour TTL |
| Analytics | Plausible (external) | Per plan |

**What We DON'T Store Persistently:**
- User reference photos → Ephemeral Blob, deleted after 1h
- Generated images → Ephemeral Blob, shared via Web Share API, deleted after 1h  
- Outfit source images → Ephemeral Blob, processing only

> **Note**: "No persistent storage" means images are not retained beyond their processing window. Ephemeral Blob with automatic TTL expiration handles the generation pipeline.

---

## 8. Image Processing

### Decision: next/image + Sharp

**Why This Combo:**
- `next/image`: Automatic optimization, lazy loading, responsive
- `sharp`: Server-side processing in Inngest jobs

**Processing Pipeline:**
1. User uploads image → temporary Blob storage
2. Inngest job processes with AI
3. Result optimized with Sharp (WebP, thumbnail)
4. Delivered to user via Web Share API → Blob expires in 1h

**Why NOT Cloudinary/Imgix:**
- Cost at scale (we're generating many images)
- Additional vendor dependency
- Simpler to process in-pipeline

---

## 9. Analytics & Tracking

### Decision: Plausible + Custom Affiliate Tracking

**Why Plausible:**
- GDPR/CCPA friendly (no cookie banner needed)
- Simple, focused metrics
- Goals and conversion tracking
- $9/month

**Custom Affiliate Tracking:**
- URL shortener via Vercel Edge Functions
- Click tracking in Vercel KV
- Attribution via UTM parameters

**Metrics to Track:**
| Metric | Tool |
|--------|------|
| Page views, sessions | Plausible |
| User flows | Plausible Goals |
| Affiliate clicks | Custom + Plausible |
| Conversion (sales) | Partner webhooks |
| Generation success rate | Internal logging |

---

## 10. Content Moderation

### Approach: Lightweight Client-Side + Terms Enforcement

**Strategy:**
Content moderation for Try on Haul is primarily policy-based rather than AI-based, given our specific use case (festival fashion).

**Implementation:**
| Layer | Method | Purpose |
|-------|--------|---------|
| Terms of Service | Legal agreement | User confirms image is of themselves |
| Consent Modal | UI gate | One-time acknowledgment of usage terms |
| User Reporting | Manual review | Flag inappropriate content |
| Rate Limiting | Abuse prevention | Limits bulk uploads |

**Content Policy:**
- ✅ **Allowed**: Festival wear, swimwear, revealing clothing, artistic fashion, bikinis, crop tops
- ❌ **Blocked**: Full nudity, explicit content, illegal content, non-self images

**Why NOT AI Moderation (MVP):**
- Festival wear (bikinis, tube tops) would trigger false positives
- Cost per image adds to generation cost
- Manual review sufficient at MVP scale
- Can add later if abuse detected

**Post-MVP Consideration:**
If abuse becomes an issue, consider:
- AWS Rekognition ($0.001/image) for explicit content detection
- Custom allow-list for festival wear categories
- Admin review queue for flagged content

---

## 11. Social Sharing

> **Full Strategy**: [SHARING-STRATEGY.md](./SHARING-STRATEGY.md)

### Approach: Web Share API + Fallback

**Primary Method: Web Share API Level 2**
- Shares image file directly from browser memory
- No server-side image storage required
- Works on iOS Safari, Android Chrome (native share sheet)
- User chooses target app (Instagram, Facebook, TikTok, etc.)

```typescript
if (navigator.canShare?.({ files: [imageBlob] })) {
  await navigator.share({ 
    title: 'Try On Haul', 
    text: caption, 
    url: affiliateLink,
    files: [imageBlob] 
  });
} else {
  // Fallback: download + copy link
}
```

**Fallback (Desktop/Unsupported):**
- Download image to device
- Copy affiliate link to clipboard
- Manual sharing instructions

**Open Graph Tags:**
- Generated via Next.js Metadata API
- Optimized preview images for link sharing
- Branded share cards with Try on Haul

---

## Cost Estimate (MVP)

**Assumptions for 10,000 MAU:**
- Average user generates 1.5 try-ons per session
- Average user has 2 sessions per month
- Active generation users: ~30% of MAU = 3,000
- Generations per month: 3,000 × 3 = ~9,000

| Service | Cost/Month |
|---------|------------|
| Vercel Pro | $20 |
| Clerk Pro | $25 |
| Inngest | Free tier |
| Plausible | $9 |
| Vercel Storage | ~$5 |
| Resend (email) | Free tier |
| AI Generation (fal.ai) | ~$540 (9K × $0.06) |
| **Total** | **~$599/month** |

**Cost Sensitivity:**
| MAU | Est. Generations | AI Cost | Total |
|-----|------------------|---------|-------|
| 1,000 | ~900 | ~$54 | ~$113 |
| 5,000 | ~4,500 | ~$270 | ~$329 |
| 10,000 | ~9,000 | ~$540 | ~$599 |
| 25,000 | ~22,500 | ~$1,350 | ~$1,409 |

> **Note**: At ~1,500 generations/month, self-hosted CatVTON becomes cost-effective. See [Cost Breakeven Analysis](../plans/IMPLEMENTATION-PLAN.md#cost-breakeven-analysis).

---

## Technology Risks

| Risk | Mitigation |
|------|------------|
| AI model licensing | Contact authors early; have FASHN as backup |
| Clerk pricing at scale | Can migrate to self-hosted if needed |
| Vercel lock-in | Standard Next.js, deployable elsewhere |
| fal.ai reliability | Implement retry logic; have backup providers |

---

## Future Considerations

### Post-MVP
1. **Self-hosted AI**: Deploy CatVTON on GPU infrastructure
2. **CDN for images**: Add caching layer if sharing volume increases
3. **Partner API**: Build embeddable widget for retailers

### Technology Watch
- ComfyUI integration for more control
- New try-on models (monitor HuggingFace)
- Platform API changes (especially TikTok)

---

## Change Log

| Date | Change | Author |
|------|--------|--------|
| 2026-02-09 | Initial tech stack decisions | AI Agent |

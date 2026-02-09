# Try on Haul - CTO Summary

> **Audience**: Technical leadership, engineering managers, architects  
> **Last Updated**: 2026-02-09

---

## What We're Building

**Try on Haul** is a virtual try-on PWA for festival fashion. Users upload a selfie, provide an outfit image, optionally select a festival scene (Burning Man, Coachella, etc.), and get an AI-generated image of themselves wearing that outfit. The result is instantly shareable to social media with embedded affiliate tracking.

---

## Core User Flow

```
Landing → Social Login → Consent Modal → Upload Selfie → Add Outfit → Select Scene → Generate → Share
```

**Key Constraints:**
- 5 generations per day per user
- No persistent image storage (privacy-first)
- Social-only authentication (marketing gate)
- 8-12 week development timeline

---

## MVP Tech Stack (Recommended)

| Component | Choice | Why | Monthly Cost |
|-----------|--------|-----|--------------|
| **Frontend** | Next.js 16 (PWA) | Best Vercel integration, React ecosystem | - |
| **Hosting** | Vercel Pro | Zero-config, edge functions, built-in storage | $20 |
| **Auth** | NextAuth.js (Auth.js v5) | Only option with native FB + IG + TikTok + Apple, open-source | Free |
| **Auth Database** | Vercel Postgres | Session storage, user accounts | Free tier |
| **AI Generation** | fal.ai (CatVTON) | Best face preservation, $0.06/image | Variable |
| **AI Backup** | FASHN API | Commercial license, scene support, $0.075/image | Fallback |
| **Background Jobs** | Inngest | Serverless, 50K/month free, great DX | Free |
| **Ephemeral Storage** | Vercel Blob | 1h TTL, automatic cleanup | ~$5 |
| **Rate Limiting** | Vercel KV | Simple, integrated | Included |
| **Push Notifications** | Web Push API | Free, native | Free |
| **Email Fallback** | Resend | 3k/month free tier | Free |
| **Analytics** | Plausible | Privacy-friendly, no cookie banner | $9 |

---

## Cost Projections (5-Tier Viral Scaling)

> **Full analysis**: [VIRAL-SCALE-PRICING.md](../docs/plans/VIRAL-SCALE-PRICING.md)

**With fal.ai (API-based):**

| MAU | Generations/mo | AI Cost | Total Monthly |
|-----|----------------|---------|---------------|
| **1,000** | 900 | $151 | **$180/mo** |
| **10,000** | 9,000 | $1,512 | **$1,577/mo** |
| **100,000** | 90,000 | $15,120 | **$15,344/mo** |
| **1,000,000** | 900,000 | $151,200 | **$153,133/mo** |
| **10,000,000** | 9,000,000 | $1,512,000 | **$1,533,518/mo** |

**With Self-Hosted GPUs:**

| MAU | Self-Hosted Cost | Monthly Savings vs fal.ai |
|-----|------------------|---------------------------|
| **10,000** | $100/mo | $1,477/mo (94% savings) |
| **100,000** | $574/mo | $14,770/mo (96% savings) |
| **1,000,000** | $7,908/mo | **$145,225/mo (95% savings)** |
| **10,000,000** | $53,718/mo | **$1,479,800/mo (96% savings)** |

**Critical Insight**: AI generation = 95%+ of total cost. Self-hosting ROI-positive at ~10K MAU.

**Infrastructure Triggers:**
| MAU | Action |
|-----|--------|
| 10K | Evaluate self-hosting ROI |
| 50K | Begin migration planning |
| 100K | Self-hosting critical (saves $15K/mo) |
| 1M | Must self-host (saves $145K/mo) |

---

## 12-Week Development Roadmap

| Sprint | Weeks | Deliverables |
|--------|-------|--------------|
| **1** | 1-2 | Auth (4 providers), consent modal, UI shell, CI/CD |
| **2** | 3-4 | Image upload (drag/drop, camera, URL), validation |
| **3** | 5-6 | AI generation pipeline, rate limiting, scene compositing |
| **4** | 7-8 | Social sharing (Web Share API), branded cards, short links |
| **5** | 9-10 | Affiliate tracking, analytics, performance optimization |
| **6** | 11-12 | Shopify integration, QA, security audit, launch |

---

## Key Technical Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Platform** | PWA (not native) | Social sharing, no app store friction, faster development |
| **AI Provider** | Adapter pattern | Swap providers without code changes, A/B testing, failover |
| **Image Storage** | Ephemeral only (1h TTL) | Privacy-first, cost reduction, CCPA compliance |
| **Sharing** | Web Share API Level 2 | No server storage needed, native OS share sheet |
| **Notifications** | Polling + Push + Email | Covers all user scenarios |
| **Content Moderation** | Policy-based (not AI) | Festival wear would trigger false positives |

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         VERCEL                                  │
├─────────────────────────────────────────────────────────────────┤
│  Next.js PWA  →  API Routes  →  Inngest  →  AI Provider        │
│       ↓              ↓             ↓           Adapter          │
│  NextAuth.js   Vercel KV      Background    ┌─────────────┐    │
│  (Postgres)    (rate limit)      Jobs       │ fal.ai      │    │
│                                             │ FASHN       │    │
│  Vercel Blob (ephemeral 1h TTL)             │ Self-hosted │    │
└─────────────────────────────────────────────┴─────────────┴────┘
```

---

## Immediate Actions Required

| Priority | Action | Why |
|----------|--------|-----|
| **P0** | Contact CatVTON authors for commercial license | Non-commercial license is a blocker |
| **P0** | Create fal.ai account + test CatVTON | Validate quality meets requirements |
| **P0** | Register Facebook/TikTok developer apps | OAuth approval takes 2-4 weeks |
| **P1** | Source 5+ festival scene images | Needed for Sprint 3 |

---

## Risk Matrix

| Risk | Impact | Mitigation |
|------|--------|------------|
| CatVTON license denied | High | FASHN contract as backup |
| OAuth approval delays | Medium | Apply immediately, use test mode |
| Generation quality issues | High | A/B test multiple models |
| AI costs exceed budget | Medium | Cost tracking from day 1 |

---

## Related Documents

- [Full Technical Documentation](../INDEX.md)
- [Architecture Overview](../docs/architecture/OVERVIEW.md)
- [Tech Stack Decisions](../docs/architecture/TECH-STACK.md)
- [Analytics & Tracking](../docs/architecture/ANALYTICS-TRACKING.md)
- [Viral Scale Pricing](../docs/plans/VIRAL-SCALE-PRICING.md)
- [Implementation Plan](../docs/plans/IMPLEMENTATION-PLAN.md)
- [AI Models Research](../docs/research/AI-MODELS.md)
- [Auth Comparison](../docs/research/AUTH-COMPARISON.md)

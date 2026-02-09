# Implementation Plan

> **Last Updated**: 2026-02-09  
> **Related**: [MVP-FEATURES.md](../user-stories/MVP-FEATURES.md) | [REQUIREMENTS.md](../user-stories/REQUIREMENTS.md) | [TECH-STACK.md](../architecture/TECH-STACK.md) | [PROVIDER-ADAPTER.md](../architecture/PROVIDER-ADAPTER.md)

---

## Purpose

This document provides a phased implementation roadmap for Try on Haul MVP, targeting an 8-12 week development timeline.

---

## Timeline Overview

```
Week 1-2    │ Sprint 1: Foundation
Week 3-4    │ Sprint 2: Core Upload & Processing
Week 5-6    │ Sprint 3: AI Generation Pipeline
Week 7-8    │ Sprint 4: Social Sharing
Week 9-10   │ Sprint 5: Polish & Affiliate Tracking
Week 11-12  │ Sprint 6: QA, Partner Integration & Launch
```

---

## Phase 1: Foundation (Weeks 1-2)

### Sprint 1 Goals

- [ ] Project scaffolding complete
- [ ] Authentication working (all 4 providers)
- [ ] Basic UI shell with navigation
- [ ] CI/CD pipeline configured

### Tasks

#### 1.1 Project Setup

| Task | Priority | Estimate |
|------|----------|----------|
| Initialize Next.js 16 with App Router | P0 | 2h |
| Configure TypeScript strict mode | P0 | 1h |
| Set up ESLint + Prettier | P0 | 1h |
| Configure Tailwind CSS | P0 | 1h |
| Set up @serwist/next for PWA | P0 | 2h |
| Configure Vercel deployment | P0 | 1h |
| Set up environment variables | P0 | 1h |

#### 1.2 Authentication

| Task | Priority | Estimate |
|------|----------|----------|
| Install and configure Clerk | P0 | 2h |
| Set up Facebook OAuth | P0 | 2h |
| Set up Instagram OAuth (via FB) | P0 | 1h |
| Set up TikTok OAuth | P0 | 2h |
| Set up Apple Sign-In | P0 | 2h |
| Create consent modal component | P0 | 3h |
| Implement consent storage in Clerk metadata | P0 | 2h |
| Protected route middleware | P0 | 2h |

#### 1.3 UI Foundation

| Task | Priority | Estimate |
|------|----------|----------|
| Create base layout component | P0 | 2h |
| Build navigation component | P0 | 2h |
| Set up design tokens (colors, typography) | P0 | 2h |
| Create Button component variants | P0 | 2h |
| Create Input component | P0 | 1h |
| Create Modal component | P0 | 2h |
| Build marketing landing page | P0 | 4h |

#### 1.4 DevOps

| Task | Priority | Estimate |
|------|----------|----------|
| Configure Vercel project | P0 | 1h |
| Set up preview deployments | P0 | 1h |
| Configure production environment | P0 | 1h |
| Set up error monitoring (Sentry) | P1 | 2h |

### Sprint 1 Deliverables

- Deployable Next.js app on Vercel
- Working social authentication (all providers)
- Consent modal for first-time users
- Basic page structure and navigation

---

## Phase 2: Core Upload & Processing (Weeks 3-4)

### Sprint 2 Goals

- [ ] Image upload working (file + camera + URL)
- [ ] Image validation implemented
- [ ] Vercel Blob storage configured
- [ ] Upload preview UI complete

### Tasks

#### 2.1 Image Upload Infrastructure

| Task | Priority | Estimate |
|------|----------|----------|
| Configure Vercel Blob storage | P0 | 2h |
| Create signed URL API route | P0 | 2h |
| Implement client-side upload | P0 | 3h |
| Add upload progress tracking | P0 | 2h |
| Implement file cleanup (TTL) | P0 | 2h |

#### 2.2 Upload UI Components

| Task | Priority | Estimate |
|------|----------|----------|
| Create drag-and-drop uploader | P0 | 4h |
| Add click-to-browse functionality | P0 | 1h |
| Implement camera capture (mobile) | P0 | 3h |
| Build image preview component | P0 | 2h |
| Add remove/replace functionality | P0 | 2h |

#### 2.3 Image Validation

| Task | Priority | Estimate |
|------|----------|----------|
| Validate file type (JPEG, PNG, WebP) | P0 | 1h |
| Validate file size (10MB max) | P0 | 1h |
| Check minimum dimensions | P0 | 2h |
| Implement face detection (basic) | P1 | 4h |
| Add quality warnings (blur, dark) | P1 | 3h |

#### 2.4 URL-Based Upload

| Task | Priority | Estimate |
|------|----------|----------|
| Create URL input component | P0 | 2h |
| Implement server-side fetch | P0 | 2h |
| Validate fetched images | P0 | 2h |
| Handle CORS/fetch errors | P0 | 2h |

### Sprint 2 Deliverables

- Complete image upload flow (reference + outfit)
- Camera capture on mobile
- URL paste for outfit images
- Image validation with error messages

---

## Phase 3: AI Generation Pipeline (Weeks 5-6)

### Sprint 3 Goals

- [ ] Inngest configured for background jobs
- [ ] CatVTON API integration working
- [ ] Generation status tracking
- [ ] Rate limiting enforced
- [ ] Scene compositing working (P0)

### Tasks

#### 3.1 Background Job Setup

| Task | Priority | Estimate |
|------|----------|----------|
| Install and configure Inngest | P0 | 2h |
| Create generation function | P0 | 3h |
| Implement step-based workflow | P0 | 3h |
| Add retry logic | P0 | 2h |
| Configure webhook handler | P0 | 2h |

#### 3.2 AI Integration

| Task | Priority | Estimate |
|------|----------|----------|
| Set up fal.ai account | P0 | 1h |
| Create fal.ai API wrapper | P0 | 3h |
| Implement CatVTON generation | P0 | 4h |
| Add fallback to IDM-VTON | P1 | 3h |
| Handle generation errors | P0 | 3h |

#### 3.3 Rate Limiting

| Task | Priority | Estimate |
|------|----------|----------|
| Configure Vercel KV | P0 | 1h |
| Implement rate limit middleware | P0 | 3h |
| Create rate limit check API | P0 | 2h |
| Display remaining count in UI | P0 | 2h |
| Handle limit exceeded state | P0 | 2h |

#### 3.4 Generation UI

| Task | Priority | Estimate |
|------|----------|----------|
| Create "Generate" button component | P0 | 2h |
| Build progress indicator | P0 | 3h |
| Implement status polling | P0 | 3h |
| Add push notification (when ready) | P1 | 4h |
| Build result viewer modal | P0 | 4h |
| Implement side-by-side comparison | P1 | 3h |
| Add download functionality | P0 | 2h |

#### 3.5 Scene Compositing (P0)

| Task | Priority | Estimate |
|------|----------|----------|
| Source/create scene images (5+ festival scenes) | P0 | 4h |
| Build scene selection grid UI | P0 | 3h |
| Implement scene preview modal | P0 | 2h |
| Pass scene to generation API (FASHN or post-process) | P0 | 3h |
| Test scene blending quality | P0 | 3h |

**Technical Notes:** FASHN API has native scene/background support. Alternative: use separate background replacement model as post-processing step.

### Sprint 3 Deliverables

- Complete generation pipeline
- Working AI integration (CatVTON)
- Rate limiting (5/day)
- Result viewing and download
- Scene compositing with 5+ festival scenes

---

## Phase 4: Social Sharing (Weeks 7-8)

### Sprint 4 Goals

- [ ] Share to all platforms working
- [ ] Branded share cards generated
- [ ] Short links created
- [ ] Web Share API fallback

### Tasks

#### 4.1 Share Card Generation

| Task | Priority | Estimate |
|------|----------|----------|
| Design share card template | P0 | 3h |
| Implement Sharp image processing | P0 | 3h |
| Add branding overlay | P0 | 2h |
| Generate platform-specific sizes | P0 | 2h |

#### 4.2 Link Shortener

| Task | Priority | Estimate |
|------|----------|----------|
| Create short link API route | P0 | 2h |
| Implement redirect handler | P0 | 2h |
| Store links in Vercel KV | P0 | 2h |
| Add click tracking | P0 | 2h |

#### 4.3 Platform Integrations

| Task | Priority | Estimate |
|------|----------|----------|
| Implement Facebook share | P0 | 3h |
| Implement Instagram share | P0 | 3h |
| Implement TikTok share | P0 | 3h |
| Implement X (Twitter) share | P0 | 2h |
| Implement Pinterest share | P1 | 2h |
| Add "Copy Link" functionality | P0 | 1h |

#### 4.4 Share UI

| Task | Priority | Estimate |
|------|----------|----------|
| Build share button group | P0 | 2h |
| Create share preview modal | P0 | 3h |
| Add caption editing | P0 | 2h |
| Implement Web Share API fallback | P0 | 2h |

### Sprint 4 Deliverables

- Sharing to all 5 platforms
- Branded share cards
- Trackable short links
- Fallback for unsupported browsers

---

## Phase 5: Polish & Affiliate Tracking (Weeks 9-10)

### Sprint 5 Goals

- [ ] Affiliate tracking complete
- [ ] Analytics integrated
- [ ] Performance optimized

### Tasks

#### 5.1 Affiliate Tracking

| Task | Priority | Estimate |
|------|----------|----------|
| Record click events | P0 | 2h |
| Associate clicks with users | P0 | 2h |
| Build conversion webhook | P1 | 3h |
| Match conversions to clicks | P1 | 3h |

#### 5.2 Analytics

| Task | Priority | Estimate |
|------|----------|----------|
| Set up Plausible account | P0 | 1h |
| Add Plausible script | P0 | 1h |
| Define goal events | P0 | 2h |
| Create conversion tracking | P0 | 2h |
| Build internal analytics dashboard | P2 | 4h |

#### 5.3 Performance

| Task | Priority | Estimate |
|------|----------|----------|
| Audit Core Web Vitals | P0 | 2h |
| Optimize image loading | P0 | 3h |
| Implement code splitting | P0 | 2h |
| Add service worker caching | P1 | 3h |
| Test PWA installation | P0 | 2h |

### Sprint 5 Deliverables

- Complete affiliate tracking
- Analytics dashboard
- Performance targets met

---

## Phase 6: QA, Partner Integration & Launch (Weeks 11-12)

### Sprint 6 Goals

- [ ] Shopify integration working
- [ ] All bugs fixed
- [ ] Security audit passed
- [ ] Launch ready

### Tasks

#### 6.1 Shopify Integration

| Task | Priority | Estimate |
|------|----------|----------|
| Implement URL detection | P1 | 2h |
| Create Storefront API client | P1 | 3h |
| Build product image extraction | P1 | 3h |
| Add partner store mapping | P1 | 2h |
| Handle non-partner stores | P1 | 2h |

#### 6.2 Testing

| Task | Priority | Estimate |
|------|----------|----------|
| Write unit tests (70% coverage) | P0 | 8h |
| Write E2E tests (core flows) | P0 | 6h |
| Manual QA testing | P0 | 8h |
| Cross-browser testing | P0 | 4h |
| Mobile device testing | P0 | 4h |
| Accessibility audit | P0 | 4h |

#### 6.3 Security

| Task | Priority | Estimate |
|------|----------|----------|
| Input validation audit | P0 | 2h |
| CORS configuration check | P0 | 1h |
| CSP headers implementation | P1 | 2h |
| Rate limiting verification | P0 | 1h |
| Secrets audit | P0 | 1h |

#### 6.4 Launch Prep

| Task | Priority | Estimate |
|------|----------|----------|
| Create Terms of Service | P0 | 4h |
| Create Privacy Policy | P0 | 4h |
| Set up custom domain | P0 | 2h |
| Configure production secrets | P0 | 1h |
| Final deployment checklist | P0 | 2h |

### Sprint 6 Deliverables

- Production-ready application
- Complete test coverage
- Legal documents
- Launch!

---

## Cost Breakeven Analysis

> **Related**: [PROVIDER-ADAPTER.md](../architecture/PROVIDER-ADAPTER.md) for cost tracking implementation

### AI Generation Costs

| Provider | Cost/Generation | Notes |
|----------|-----------------|-------|
| fal.ai (CatVTON) | ~$0.06 | Primary provider |
| FASHN | ~$0.10 | Commercial backup |
| Self-hosted | ~$0.02 (infra only) | Future option |

### Revenue Model

| Source | Revenue/Event | Conversion Rate |
|--------|---------------|-----------------|
| Affiliate click | $0.00 | (tracking only) |
| Affiliate purchase | ~$2-5 (5-10% commission) | ~2-5% of clicks |

### Breakeven Calculation

**Variables**:
- **Cost per generation (C)**: $0.06 (fal.ai primary)
- **Generations per user (G)**: 5/day limit
- **Share rate (S)**: 30% target
- **Click-through rate (CTR)**: 5% target
- **Purchase conversion (P)**: 3% estimate
- **Average commission (A)**: $3 estimate

**Per 100 generations**:
- Cost: 100 × $0.06 = **$6.00**
- Shares: 100 × 0.30 = 30 shares
- Clicks: 30 × 0.05 = 1.5 clicks (per share, assume 2x viral = 3 clicks)
- Purchases: 3 × 0.03 = 0.09 purchases
- Revenue: 0.09 × $3 = **$0.27**

**Breakeven requires** either:
1. Higher affiliate commission rates
2. Higher conversion rates
3. Sponsorship/partnership deals
4. Self-hosted infrastructure (reduces C to ~$0.02)

### Cost Tracking Requirements

Implement cost tracking from Sprint 3 (see [PROVIDER-ADAPTER.md](../architecture/PROVIDER-ADAPTER.md#cost-tracking)):

1. **Per-generation logging**: Track provider, cost, success/failure
2. **Daily aggregation**: Total cost per provider
3. **User-level tracking**: Cost per user for LTV analysis
4. **Alert thresholds**: Notify when daily costs exceed budget

### Optimization Strategies

| Strategy | Impact | Sprint |
|----------|--------|--------|
| Provider A/B testing | Select lowest-cost quality provider | Sprint 3-4 |
| Caching successful generations | Avoid re-processing identical requests | Sprint 5 |
| Self-hosted evaluation | 70% cost reduction potential | Post-MVP |
| Premium tier (paid) | Offset costs for heavy users | Post-MVP |

---

## Risk Mitigation

### Technical Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| AI model licensing | High | Contact authors early; FASHN backup |
| OAuth provider changes | Medium | Abstract auth layer; monitor updates |
| Generation quality issues | High | Test extensively; user feedback loop |
| Rate limit abuse | Medium | Strong validation; CAPTCHA if needed |

### Timeline Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| OAuth integration delays | High | Start auth in Sprint 1; allow buffer |
| AI API reliability | Medium | Implement retries; fallback providers |
| Scope creep | High | Strict MVP scope; defer to post-MVP |
| QA finding major bugs | Medium | Continuous testing; buffer in Sprint 6 |

---

## Success Metrics

### MVP Launch Criteria

- [ ] All P0 requirements complete
- [ ] Performance: LCP < 2.5s
- [ ] Uptime: 99.5%
- [ ] Generation success rate: > 95%
- [ ] Mobile UX validated

### Post-Launch Metrics (First 30 Days)

| Metric | Target |
|--------|--------|
| Daily Active Users | 100+ |
| Generations per day | 500+ |
| Share rate | 30%+ of generations |
| Affiliate click-through | 5%+ |
| User retention (7-day) | 20%+ |

---

## Dependencies

### External Dependencies

| Dependency | Owner | Status |
|------------|-------|--------|
| Clerk account | Team | Pending |
| fal.ai account | Team | Pending |
| Vercel account | Team | Pending |
| Facebook App | Team | Pending |
| TikTok App | Team | Pending |
| Apple Developer | Team | Pending |
| Plausible account | Team | Pending |

### Partner Dependencies

| Dependency | Owner | Status |
|------------|-------|--------|
| Commercial AI license | CatVTON authors | Pending |
| Shopify test store | Team | Pending |
| Partner retailer (pilot) | Business | Pending |

---

## Post-MVP Roadmap

### Phase 7: Growth (Months 2-3)

- User gallery (saved try-ons)
- Multi-garment combinations
- Partner dashboard
- Referral program

### Phase 8: Expansion (Months 4-6)

- Embeddable widget
- White-label option
- Additional scene categories
- International expansion

### Phase 9: Advanced (Months 6+)

- Video try-on
- AR preview
- AI styling recommendations
- Native mobile apps

---

## Change Log

| Date | Change | Author |
|------|--------|--------|
| 2026-02-09 | Initial implementation plan | AI Agent |

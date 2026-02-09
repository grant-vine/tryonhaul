# Detailed Requirements

> **Last Updated**: 2026-02-09  
> **Related**: [MVP-FEATURES.md](./MVP-FEATURES.md) | [TECH-STACK.md](../architecture/TECH-STACK.md)

---

## Purpose

This document provides detailed functional and non-functional requirements for Try on Haul MVP.

---

## Functional Requirements

### FR-1: Authentication

| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-1.1 | System shall support Facebook OAuth login | P0 | Planned |
| FR-1.2 | System shall support Instagram login via Facebook | P0 | Planned |
| FR-1.3 | System shall support TikTok OAuth login | P0 | Planned |
| FR-1.4 | System shall support Apple Sign-In | P0 | Planned |
| FR-1.5 | System shall persist sessions for 30 days | P0 | Planned |
| FR-1.6 | System shall display consent modal on first login | P0 | Planned |
| FR-1.7 | System shall save consent status to user profile | P0 | Planned |
| FR-1.8 | System shall allow users to delete their account | P1 | Planned |
| FR-1.9 | System shall invalidate all sessions on logout | P0 | Planned |

### FR-2: Image Upload

| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-2.1 | System shall accept JPEG, PNG, WebP images | P0 | Planned |
| FR-2.2 | System shall limit uploads to 10MB | P0 | Planned |
| FR-2.3 | System shall support drag-and-drop upload | P0 | Planned |
| FR-2.4 | System shall support click-to-browse upload | P0 | Planned |
| FR-2.5 | System shall support camera capture on mobile | P0 | Planned |
| FR-2.6 | System shall validate image dimensions (min 768x1024) | P0 | Planned |
| FR-2.7 | System shall detect faces in reference photos | P1 | Planned |
| FR-2.8 | System shall display upload preview | P0 | Planned |
| FR-2.9 | System shall allow URL input for outfit images | P0 | Planned |
| FR-2.10 | System shall fetch and validate images from URLs | P0 | Planned |
| FR-2.11 | System shall store uploads in ephemeral Blob (1 hour TTL) | P0 | Planned |

### FR-3: AI Generation

| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-3.1 | System shall generate try-on images using provider adapter | P0 | Planned |
| FR-3.2 | System shall display generation progress | P0 | Planned |
| FR-3.3 | System shall complete generation within 90 seconds (target: 60s) | P0 | Planned |
| FR-3.4 | System shall notify user when generation completes (see FR-3.12-3.15) | P0 | Planned |
| FR-3.5 | System shall display generated result in modal/fullscreen | P0 | Planned |
| FR-3.6 | System shall support side-by-side comparison view | P1 | Planned |
| FR-3.7 | System shall allow image download | P0 | Planned |
| FR-3.8 | System shall retry failed generations automatically (3x) | P0 | Planned |
| FR-3.9 | System shall not count failed generations against limit | P0 | Planned |
| FR-3.10 | System shall support scene/background compositing | P0 | Planned |
| FR-3.11 | System shall queue jobs via Inngest | P0 | Planned |
| FR-3.12 | System shall support polling + in-app toast notification | P0 | Planned |
| FR-3.13 | System shall support browser push notifications (with permission) | P0 | Planned |
| FR-3.14 | System shall support email notification fallback | P1 | Planned |
| FR-3.15 | System shall track notification delivery success | P1 | Planned |

### FR-4: Rate Limiting

| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-4.1 | System shall limit users to 5 generations per day | P0 | Planned |
| FR-4.2 | System shall display remaining generation count | P0 | Planned |
| FR-4.3 | System shall display reset time when limit reached | P0 | Planned |
| FR-4.4 | System shall reset limits at midnight UTC | P0 | Planned |
| FR-4.5 | System shall return 429 error when limit exceeded | P0 | Planned |

### FR-5: Social Sharing

> **Full Strategy**: [SHARING-STRATEGY.md](../architecture/SHARING-STRATEGY.md)

| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-5.1 | System shall support Web Share API Level 2 (file sharing) | P0 | Planned |
| FR-5.2 | System shall support sharing to Instagram (via OS share sheet) | P0 | Planned |
| FR-5.3 | System shall support sharing to TikTok (via OS share sheet) | P0 | Planned |
| FR-5.4 | System shall support sharing to Facebook (via OS share sheet) | P0 | Planned |
| FR-5.5 | System shall support sharing to X (via Web Intent + share sheet) | P0 | Planned |
| FR-5.6 | System shall support sharing to Pinterest (via OS share sheet) | P1 | Planned |
| FR-5.7 | System shall generate branded share cards | P0 | Planned |
| FR-5.8 | System shall include trackable affiliate links in shares | P0 | Planned |
| FR-5.9 | System shall provide download + copy link fallback | P0 | Planned |
| FR-5.10 | System shall support "Copy Link" function | P0 | Planned |
| FR-5.11 | System shall share images directly from browser memory (no server storage) | P0 | Planned |

### FR-6: Affiliate Tracking

| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-6.1 | System shall generate unique short links | P0 | Planned |
| FR-6.2 | System shall redirect short links to product URLs | P0 | Planned |
| FR-6.3 | System shall track link clicks | P0 | Planned |
| FR-6.4 | System shall associate clicks with user IDs | P0 | Planned |
| FR-6.5 | System shall accept conversion webhooks from partners | P1 | Planned |
| FR-6.6 | System shall match conversions to clicks | P1 | Planned |

### FR-7: Shopify Integration

| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-7.1 | System shall detect Shopify product URLs | P1 | Planned |
| FR-7.2 | System shall query Storefront API for product images | P1 | Planned |
| FR-7.3 | System shall display "Partner Store" badge | P2 | Planned |
| FR-7.4 | System shall maintain partner token mapping | P1 | Planned |
| FR-7.5 | System shall handle non-partner URLs gracefully | P1 | Planned |

### FR-8: Content Moderation

| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-8.1 | System shall allow risqué festival wear (bikinis, tube tops, etc.) | P0 | Planned |
| FR-8.2 | System shall NOT block revealing but non-explicit content | P0 | Planned |
| FR-8.3 | System shall block explicitly prohibited content (nudity, illegal) | P0 | Planned |
| FR-8.4 | System shall validate uploaded images are of the user themselves | P0 | Planned |
| FR-8.5 | System shall provide Terms of Service acknowledgment | P0 | Planned |
| FR-8.6 | System shall support user-reported content review | P2 | Planned |

**Content Policy Note**: This app targets festival-goers who wear revealing outfits (bikinis, crop tops, body suits, etc.). The moderation policy must distinguish between:
- ✅ **Allowed**: Festival wear, swimwear, revealing clothing, artistic fashion
- ❌ **Blocked**: Full nudity, explicit content, illegal content, copyrighted images without permission

---

## Non-Functional Requirements

### NFR-1: Performance

| ID | Requirement | Target | Priority |
|----|-------------|--------|----------|
| NFR-1.1 | Page load time (LCP) | < 2.5 seconds | P0 |
| NFR-1.2 | Time to Interactive (TTI) | < 3.5 seconds | P0 |
| NFR-1.3 | Image upload time | < 5 seconds for 10MB | P0 |
| NFR-1.4 | AI generation time | < 90 seconds (target: 60s) | P0 |
| NFR-1.5 | API response time (non-generation) | < 200ms | P0 |
| NFR-1.6 | Concurrent generation jobs | 50+ per minute | P1 |

### NFR-2: Availability

| ID | Requirement | Target | Priority |
|----|-------------|--------|----------|
| NFR-2.1 | System uptime | 99.5% | P0 |
| NFR-2.2 | Planned maintenance windows | < 1 hour/month | P1 |
| NFR-2.3 | Mean time to recovery | < 15 minutes | P1 |

### NFR-3: Security

| ID | Requirement | Priority |
|----|-------------|----------|
| NFR-3.1 | All traffic over HTTPS | P0 |
| NFR-3.2 | OAuth tokens encrypted at rest | P0 |
| NFR-3.3 | Session cookies HttpOnly, Secure, SameSite | P0 |
| NFR-3.4 | Input validation on all user inputs | P0 |
| NFR-3.5 | Image uploads scanned for malware | P1 |
| NFR-3.6 | Rate limiting on all API endpoints | P0 |
| NFR-3.7 | CORS configured for allowed origins only | P0 |
| NFR-3.8 | CSP headers configured | P1 |

### NFR-4: Privacy & Compliance

| ID | Requirement | Priority |
|----|-------------|----------|
| NFR-4.1 | CCPA compliant data handling | P0 |
| NFR-4.2 | User data exportable on request | P1 |
| NFR-4.3 | User data deletable within 24 hours | P0 |
| NFR-4.4 | No PII in logs | P0 |
| NFR-4.5 | Analytics anonymized (no cookies) | P0 |
| NFR-4.6 | Clear privacy policy | P0 |
| NFR-4.7 | Image consent documented and stored | P0 |

### NFR-5: Accessibility

| ID | Requirement | Priority |
|----|-------------|----------|
| NFR-5.1 | WCAG 2.1 Level A compliance | P0 |
| NFR-5.2 | Keyboard navigation for all features | P0 |
| NFR-5.3 | Screen reader compatible | P1 |
| NFR-5.4 | Color contrast ratio 4.5:1 minimum | P0 |
| NFR-5.5 | Focus indicators visible | P0 |
| NFR-5.6 | Touch targets minimum 44x44px | P0 |

### NFR-6: Scalability

| ID | Requirement | Target | Priority |
|----|-------------|--------|----------|
| NFR-6.1 | Support 10,000 MAU at launch | P0 |
| NFR-6.2 | Scale to 100,000 MAU without architecture changes | P1 |
| NFR-6.3 | Horizontal scaling via serverless | P0 |
| NFR-6.4 | No single points of failure | P1 |

### NFR-7: Reliability

| ID | Requirement | Priority |
|----|-------------|----------|
| NFR-7.1 | Graceful degradation if AI service down | P0 |
| NFR-7.2 | Automatic retry for transient failures | P0 |
| NFR-7.3 | Error messages actionable for users | P0 |
| NFR-7.4 | Background jobs recoverable after crash | P0 |

### NFR-8: Maintainability

| ID | Requirement | Priority |
|----|-------------|----------|
| NFR-8.1 | TypeScript strict mode | P0 |
| NFR-8.2 | Minimum 70% test coverage for business logic | P1 |
| NFR-8.3 | Linting enforced (ESLint + Prettier) | P0 |
| NFR-8.4 | Consistent code style | P0 |
| NFR-8.5 | Documentation for all APIs | P1 |

### NFR-9: Localization

| ID | Requirement | Priority |
|----|-------------|----------|
| NFR-9.1 | English language for MVP | P0 |
| NFR-9.2 | Architecture supports future i18n | P1 |
| NFR-9.3 | No hardcoded strings in components | P1 |
| NFR-9.4 | Date/time formatting locale-aware | P1 |

---

## Data Requirements

### DR-1: User Data

| Field | Type | Required | Storage |
|-------|------|----------|---------|
| userId | string | Yes | Vercel Postgres |
| email | string | Yes | Vercel Postgres |
| name | string | Yes | Vercel Postgres |
| profilePhoto | URL | No | Vercel Postgres |
| authProvider | enum | Yes | Vercel Postgres |
| consentGiven | boolean | Yes | Vercel Postgres |
| consentDate | datetime | Yes | Vercel Postgres |
| createdAt | datetime | Yes | Vercel Postgres |

### DR-2: Generation Job

| Field | Type | Required | Storage |
|-------|------|----------|---------|
| jobId | string | Yes | Inngest |
| userId | string | Yes | Inngest |
| referenceImageUrl | URL | Yes | Vercel Blob |
| outfitImageUrl | URL | Yes | Vercel Blob |
| sceneId | string | No | Inngest |
| status | enum | Yes | Inngest |
| resultImageUrl | URL | No | Vercel Blob |
| createdAt | datetime | Yes | Inngest |
| completedAt | datetime | No | Inngest |

### DR-3: Affiliate Link

| Field | Type | Required | Storage |
|-------|------|----------|---------|
| shortId | string | Yes | Vercel KV |
| destinationUrl | URL | Yes | Vercel KV |
| userId | string | Yes | Vercel KV |
| generationId | string | No | Vercel KV |
| productId | string | No | Vercel KV |
| clicks | integer | Yes | Vercel KV |
| createdAt | datetime | Yes | Vercel KV |

### DR-4: Rate Limit

| Field | Type | Required | Storage |
|-------|------|----------|---------|
| key | string | Yes | Vercel KV |
| count | integer | Yes | Vercel KV |
| expiresAt | datetime | Yes | Vercel KV |

---

## Integration Requirements

### IR-1: External Service SLAs

| Service | Expected Uptime | Fallback |
|---------|-----------------|----------|
| Vercel Postgres (Auth) | 99.9% | Cached sessions |
| fal.ai (AI) | 99% | IDM-VTON or FASHN |
| Vercel (Hosting) | 99.99% | None (primary) |
| Vercel KV | 99.9% | Redis fallback |
| Vercel Blob | 99.9% | S3 fallback |
| Plausible | 99.5% | Continue without analytics |

### IR-2: API Rate Limits

| Service | Limit | Strategy |
|---------|-------|----------|
| fal.ai | Based on plan | Queue + retry |
| Shopify Storefront | 2-20 req/sec per store | Cache + backoff |
| Social APIs | Varies | Rate limit + queue |

---

## Constraints

### Technical Constraints

1. **Serverless environment**: 10 second default timeout, 60 second max
2. **No persistent file system**: Use Blob storage
3. **Cold starts**: Edge functions for critical paths
4. **CORS**: Browser security limits cross-origin requests

### Business Constraints

1. **Budget**: $100/month max for infrastructure (MVP)
2. **AI Cost**: ~$0.06/generation (must stay under budget)
3. **Timeline**: 8-12 weeks to MVP
4. **Team**: AI-agent driven development

### Legal Constraints

1. **AI model licensing**: Must secure commercial license or use FASHN
2. **Image rights**: Users must confirm images are of themselves
3. **Partner agreements**: Shopify API terms compliance
4. **Privacy**: CCPA compliance required

---

## Acceptance Criteria Summary

### MVP Launch Criteria

- [ ] All P0 functional requirements implemented
- [ ] All P0 non-functional requirements met
- [ ] Social auth working (all 4 providers)
- [ ] Generation pipeline complete and reliable
- [ ] Sharing to at least 3 platforms working
- [ ] Rate limiting enforced
- [ ] Affiliate link tracking working
- [ ] Privacy policy and ToS published
- [ ] WCAG 2.1 Level A compliant
- [ ] Performance targets met

---

## Change Log

| Date | Change | Author |
|------|--------|--------|
| 2026-02-09 | Initial requirements document | AI Agent |

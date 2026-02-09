# Architecture Overview

> **Last Updated**: 2026-02-09  
> **Related**: [TECH-STACK.md](./TECH-STACK.md) | [DATA-FLOW.md](./DATA-FLOW.md) | [MVP-FEATURES.md](../user-stories/MVP-FEATURES.md)

---

## Purpose

This document describes the high-level system architecture for Try on Haul, a virtual try-on application for festival fashion.

---

## System Context

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              TRY ON HAUL ECOSYSTEM                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│    ┌──────────┐     ┌───────────────────┐     ┌──────────────────────┐     │
│    │  Users   │────▶│  Try on Haul PWA  │────▶│  Social Platforms    │     │
│    │(Mobile/  │     │  (Web App)        │     │  FB/IG/TikTok/X/Pin  │     │
│    │ Desktop) │◀────│                   │◀────│                      │     │
│    └──────────┘     └─────────┬─────────┘     └──────────────────────┘     │
│                               │                                             │
│                    ┌──────────┴──────────┐                                 │
│                    │                     │                                 │
│          ┌─────────▼─────────┐ ┌─────────▼─────────┐                       │
│          │  Auth Providers   │ │  AI Generation    │                       │
│          │  FB/IG/TikTok/    │ │  Service          │                       │
│          │  Apple            │ │  (CatVTON/FASHN)  │                       │
│          └───────────────────┘ └───────────────────┘                       │
│                                                                             │
│          ┌───────────────────┐ ┌───────────────────┐                       │
│          │  Partner Stores   │ │  Link Tracking    │                       │
│          │  (Shopify)        │ │  Service          │                       │
│          └───────────────────┘ └───────────────────┘                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Core Components

### 1. Frontend (PWA)

| Aspect | Description |
|--------|-------------|
| Type | Progressive Web App |
| Rendering | Hybrid SSR/CSR |
| Primary UI | Mobile-first responsive |
| Key Features | Camera capture, image upload, social sharing |

**Responsibilities**:
- User authentication flow (social OAuth)
- Image capture and upload
- Generation status tracking
- Result preview and sharing

### 2. Backend (Serverless)

| Aspect | Description |
|--------|-------------|
| Runtime | Vercel Edge Functions / Serverless |
| API Style | REST (polling for status updates) |
| Notifications | Push notifications + email fallback |
| Database | Minimal - auth session + rate limiting only |

**Responsibilities**:
- OAuth callback handling
- Rate limiting (5 generations/day)
- Generation job queuing
- Webhook handling for completion
- Affiliate link generation and tracking

### 3. AI Generation Service

| Aspect | Description |
|--------|-------------|
| Primary | CatVTON (open-source) via fal.ai API |
| Backup | FASHN API (commercial) |
| Processing | Async with webhook callbacks |

**Responsibilities**:
- Receive generation requests
- Process try-on with AI model
- Return generated images
- Handle errors and retries

> **Full model analysis**: [AI-MODELS.md](../research/AI-MODELS.md)

### 4. External Integrations

| Service | Purpose |
|---------|---------|
| Social Auth (FB/IG/TikTok/Apple) | User authentication |
| Social APIs (FB/IG/TikTok/X/Pin) | Share functionality |
| Shopify Storefront API | Partner product image extraction |
| Analytics | Usage, conversion, affiliate tracking |

---

## Data Architecture

### What We Store

| Data Type | Storage | Retention |
|-----------|---------|-----------|
| User profile | Database | Until deletion request |
| Auth sessions | Database | Session duration |
| Rate limit counters | Cache/KV | 24 hours |
| Generation job metadata | Queue | Until completion |
| Affiliate click tracking | Analytics | Indefinite |

### What We DON'T Store (Persistently)

| Data Type | Handling |
|-----------|----------|
| User reference photos | Ephemeral Blob (1h TTL), deleted after processing |
| Generated images | Ephemeral Blob (1h TTL), delivered to user via Web Share API |
| Outfit source images | Ephemeral Blob (1h TTL), processing only |

> **Note**: "No persistent storage" means images are not retained beyond their processing window. Ephemeral Blob storage with automatic TTL expiration is used for the generation pipeline.

---

## Security Architecture

### Authentication Flow

```
User ──▶ Social Provider ──▶ OAuth Callback ──▶ Session Token
                                    │
                                    ▼
                          Consent Modal (first-time)
                                    │
                                    ▼
                           Main Application
```

### Key Security Controls

1. **Social-only auth**: No password storage, delegated to providers
2. **Session tokens**: Short-lived, httpOnly cookies
3. **Image consent**: Modal confirmation before any processing
4. **Rate limiting**: 5 generations/day per user
5. **Input validation**: All uploads validated for type, size, content
6. **No PII storage**: Minimal data collection

---

## Deployment Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         VERCEL                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────┐    ┌─────────────────────┐            │
│  │   Edge Network      │    │   Serverless        │            │
│  │   (Static + SSR)    │    │   Functions         │            │
│  │                     │    │                     │            │
│  │   - Next.js pages   │    │   - API routes      │            │
│  │   - PWA assets      │    │   - OAuth handlers  │            │
│  │   - Scene images    │    │   - Webhooks        │            │
│  └─────────────────────┘    └─────────────────────┘            │
│                                                                 │
│  ┌─────────────────────┐    ┌─────────────────────┐            │
│  │   KV Store          │    │   Blob Storage      │            │
│  │   (Rate limits,     │    │   (Scene images,    │            │
│  │    Sessions)        │    │    Temp uploads)    │            │
│  └─────────────────────┘    └─────────────────────┘            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
              ┌───────────────────────────────┐
              │   External Services           │
              │                               │
              │   - fal.ai (CatVTON API)     │
              │   - Social Auth Providers     │
              │   - Analytics                 │
              └───────────────────────────────┘
```

---

## Scalability Considerations

### MVP Scale

| Metric | Expected | Limit |
|--------|----------|-------|
| Daily users | 100-1,000 | Serverless scales automatically |
| Generations/day | 500-5,000 | Rate limited per user |
| Concurrent | 10-50 | No bottleneck anticipated |

### Growth Considerations

| Challenge | Mitigation |
|-----------|------------|
| AI API costs | Monitor usage, consider self-hosting CatVTON |
| Rate limit storage | Vercel KV scales, can migrate to Redis |
| Image processing | Already offloaded to AI service |

---

## Reliability & Resilience

### Failure Modes

| Failure | Impact | Mitigation |
|---------|--------|------------|
| AI service down | Generation fails | Queue with retry, notify user |
| Auth provider down | Can't login | Show maintenance message |
| Rate limit KV down | Can't enforce limits | Fail open (allow) with logging |

### Monitoring

| Metric | Purpose |
|--------|---------|
| Generation success rate | AI service health |
| Auth flow completion | Conversion tracking |
| API response times | Performance |
| Error rates by type | Debugging |

---

## Future Architecture Considerations

### Post-MVP

1. **Self-hosted AI**: Reduce API costs at scale
2. **CDN for generated images**: Optional caching layer
3. **Partner API**: Embeddable widget for retailers
4. **Real-time collaboration**: Share try-ons with friends

### Out of Scope for MVP

- Multi-region deployment
- Image CDN/caching
- Real-time features
- Partner admin dashboard

---

## Change Log

| Date | Change | Author |
|------|--------|--------|
| 2026-02-09 | Initial architecture overview | AI Agent |

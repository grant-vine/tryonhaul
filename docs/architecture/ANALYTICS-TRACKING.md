# Analytics & Affiliate Tracking Architecture

> **Last Updated**: 2026-02-09  
> **Related**: [TECH-STACK.md](./TECH-STACK.md) | [DATA-FLOW.md](./DATA-FLOW.md) | [REQUIREMENTS.md](../user-stories/REQUIREMENTS.md)

---

## Purpose

This document defines the complete analytics and tracking architecture for Try on Haul, covering:
1. Web analytics (user behavior)
2. Affiliate link tracking (click-through attribution)
3. Conversion tracking (revenue attribution)
4. Cost tracking (infrastructure spend)
5. Reporting and dashboards

---

## Analytics Stack Overview

| Layer | Tool | Purpose | Cost |
|-------|------|---------|------|
| **Web Analytics** | Vercel Web Analytics | Page views, user flows, events | $3/100K events |
| **Affiliate Clicks** | Custom (Vercel KV + Edge) | Click tracking, attribution | ~$20/mo at 100K MAU |
| **Conversions** | Partner Webhooks | Revenue attribution | Free |
| **Cost Tracking** | Custom (Vercel KV) | AI generation spend | Included in KV |
| **Reporting** | Internal Dashboard | Business metrics | Custom build |

> **Note**: Switched from Plausible ($299/mo at 1M MAU) to Vercel Web Analytics ($150/mo at 1M MAU) for cost efficiency.

---

## 1. Web Analytics (Vercel Web Analytics)

### What We Track

| Metric | Purpose | Implementation |
|--------|---------|----------------|
| Page views | Traffic volume | Automatic |
| Unique visitors | MAU calculation | Automatic |
| Bounce rate | Landing page effectiveness | Automatic |
| Session duration | Engagement quality | Automatic |
| Top pages | Content performance | Automatic |
| Referrers | Acquisition channels | Automatic |
| Country/device | Audience demographics | Automatic |

### Custom Events

```typescript
// Track key user actions
import { track } from '@vercel/analytics';

// Authentication events
track('auth_started', { provider: 'facebook' });
track('auth_completed', { provider: 'facebook' });

// Generation funnel
track('upload_started', { type: 'reference' });
track('upload_completed', { type: 'reference', sizeKb: 1024 });
track('outfit_added', { source: 'upload' | 'url' | 'shopify' });
track('scene_selected', { sceneId: 'burning-man' });
track('generation_started');
track('generation_completed', { durationMs: 45000 });
track('generation_failed', { reason: 'timeout' });

// Sharing funnel
track('share_initiated', { platform: 'native' | 'download' });
track('share_completed', { platform: 'instagram' });
track('affiliate_link_copied');
```

### Funnel Analysis

```
Landing Page (100%)
    ↓ 60% click CTA
Auth Started (60%)
    ↓ 85% complete auth
Auth Completed (51%)
    ↓ 95% accept consent
Consent Given (48%)
    ↓ 80% upload photo
Upload Reference (38%)
    ↓ 70% add outfit
Add Outfit (27%)
    ↓ 90% start generation
Generation Started (24%)
    ↓ 95% success rate
Generation Completed (23%)
    ↓ 30% share target
Share Completed (7%)
```

### Cost by Scale

| MAU | Est. Events/mo | Vercel Analytics Cost |
|-----|----------------|----------------------|
| 1,000 | 10K | $0 (included in Pro) |
| 10,000 | 100K | $3 |
| 100,000 | 1M | $30 |
| 1,000,000 | 10M | $300 |
| 10,000,000 | 100M | $3,000 |

---

## 2. Affiliate Link Tracking

### Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    AFFILIATE TRACKING FLOW                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. LINK GENERATION                                              │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐                 │
│  │ User     │────▶│ Edge     │────▶│ Short    │                 │
│  │ Shares   │     │ Function │     │ Link     │                 │
│  └──────────┘     └──────────┘     │ Created  │                 │
│                         │          └──────────┘                 │
│                         ▼                                        │
│                   ┌──────────┐                                   │
│                   │ Vercel   │ Store: linkId → {                 │
│                   │ KV       │   userId, productUrl,             │
│                   └──────────┘   partnerId, createdAt }          │
│                                                                  │
│  2. CLICK TRACKING                                               │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐                 │
│  │ Visitor  │────▶│ /l/:id   │────▶│ Partner  │                 │
│  │ Clicks   │     │ Edge     │     │ Site     │                 │
│  └──────────┘     └──────────┘     └──────────┘                 │
│                         │                                        │
│                         ▼                                        │
│                   ┌──────────┐                                   │
│                   │ Vercel   │ Increment: click:{linkId}         │
│                   │ KV       │ Log: clicks:{date}:{linkId}       │
│                   └──────────┘                                   │
│                                                                  │
│  3. CONVERSION WEBHOOK                                           │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐                 │
│  │ Partner  │────▶│ /api/    │────▶│ Match to │                 │
│  │ Webhook  │     │ webhook  │     │ Click    │                 │
│  └──────────┘     └──────────┘     └──────────┘                 │
│                         │                                        │
│                         ▼                                        │
│                   ┌──────────┐                                   │
│                   │ Vercel   │ Store: conversion:{id}            │
│                   │ KV       │ Update: revenue:{userId}          │
│                   └──────────┘                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Data Schema (Vercel KV)

```typescript
// Affiliate Link Record
interface AffiliateLink {
  id: string;              // Short ID (e.g., "abc123")
  userId: string;          // Creator's user ID
  productUrl: string;      // Original product URL
  partnerId: string;       // Partner store ID
  partnerSku?: string;     // Product SKU if available
  generationId?: string;   // Linked generation (optional)
  createdAt: number;       // Unix timestamp
  expiresAt?: number;      // Optional expiry
}

// Click Record (aggregated)
interface ClickAggregate {
  linkId: string;
  totalClicks: number;
  uniqueClicks: number;    // Based on IP hash
  lastClickAt: number;
}

// Daily Click Log (for reporting)
interface DailyClickLog {
  date: string;            // YYYY-MM-DD
  linkId: string;
  clicks: number;
  uniqueClicks: number;
  referrers: Record<string, number>;  // Referrer → count
  countries: Record<string, number>;  // Country → count
}

// Conversion Record
interface ConversionRecord {
  id: string;              // Partner's transaction ID
  linkId: string;          // Matched affiliate link
  userId: string;          // User who created the link
  partnerId: string;
  orderValue: number;      // Total order value (cents)
  commission: number;      // Our commission (cents)
  currency: string;
  convertedAt: number;
  matchedAt: number;
  matchMethod: 'cookie' | 'utm' | 'referrer';
}
```

### KV Key Structure

```typescript
// Links
`link:{shortId}` → AffiliateLink           // TTL: none
`links:user:{userId}` → Set<shortId>       // User's links

// Clicks
`clicks:{shortId}` → ClickAggregate        // TTL: none
`clicks:daily:{date}:{shortId}` → DailyClickLog  // TTL: 90 days

// Conversions
`conversion:{transactionId}` → ConversionRecord  // TTL: 1 year
`conversions:user:{userId}` → Set<transactionId>
`conversions:daily:{date}` → List<transactionId>

// Aggregates (for dashboards)
`stats:clicks:daily:{date}` → { total, unique }
`stats:conversions:daily:{date}` → { count, value, commission }
`stats:revenue:user:{userId}` → { totalCommission, totalClicks }
```

### Link Generation API

```typescript
// POST /api/affiliate/create
interface CreateLinkRequest {
  productUrl: string;
  partnerId?: string;      // Auto-detect if not provided
  generationId?: string;   // Link to specific generation
}

interface CreateLinkResponse {
  shortUrl: string;        // https://tryonhaul.com/l/abc123
  shortId: string;
  productUrl: string;
  partnerId: string;
}
```

### Click Redirect Logic

```typescript
// GET /l/:shortId (Edge Function)
export async function GET(request: Request, { params }) {
  const { shortId } = params;
  
  // 1. Lookup link
  const link = await kv.get<AffiliateLink>(`link:${shortId}`);
  if (!link) return new Response('Not found', { status: 404 });
  
  // 2. Record click (non-blocking)
  const ip = request.headers.get('x-forwarded-for');
  const ipHash = hashIP(ip); // Privacy: don't store raw IP
  
  await Promise.all([
    kv.hincrby(`clicks:${shortId}`, 'totalClicks', 1),
    recordDailyClick(shortId, ipHash, request),
  ]);
  
  // 3. Build partner URL with tracking
  const partnerUrl = new URL(link.productUrl);
  partnerUrl.searchParams.set('utm_source', 'tryonhaul');
  partnerUrl.searchParams.set('utm_medium', 'affiliate');
  partnerUrl.searchParams.set('utm_campaign', shortId);
  partnerUrl.searchParams.set('ref', 'tryonhaul');
  
  // 4. Redirect
  return Response.redirect(partnerUrl.toString(), 302);
}
```

---

## 3. Conversion Tracking

### Partner Webhook Integration

Partners send conversion events via webhook when a tracked sale occurs.

```typescript
// POST /api/webhooks/partner
interface ConversionWebhook {
  partnerId: string;
  transactionId: string;
  orderValue: number;      // Cents
  commission: number;      // Cents
  currency: string;
  trackingId?: string;     // Our shortId if they tracked it
  customerId?: string;     // Their customer ID
  timestamp: string;       // ISO 8601
  signature: string;       // HMAC signature for verification
}
```

### Conversion Matching Logic

```typescript
async function matchConversion(webhook: ConversionWebhook) {
  // Method 1: Direct tracking ID match
  if (webhook.trackingId) {
    const link = await kv.get<AffiliateLink>(`link:${webhook.trackingId}`);
    if (link) {
      return { link, method: 'utm' };
    }
  }
  
  // Method 2: Cookie-based (partner stores our ref in cookie)
  // Requires partner integration
  
  // Method 3: Time-window match (last click from same partner)
  // Find clicks to this partner in last 24 hours
  // Match by IP hash if available
  
  return null; // Unmatched conversion
}
```

### Revenue Attribution

```typescript
interface UserRevenue {
  userId: string;
  totalClicks: number;
  totalConversions: number;
  totalOrderValue: number;  // Cents
  totalCommission: number;  // Cents
  conversionRate: number;   // conversions / clicks
  lastConversionAt: number;
}

// Query: Get user's revenue summary
async function getUserRevenue(userId: string): Promise<UserRevenue> {
  const stats = await kv.get(`stats:revenue:user:${userId}`);
  return stats || defaultUserRevenue(userId);
}
```

---

## 4. Cost Tracking

> **Full implementation**: [PROVIDER-ADAPTER.md](./PROVIDER-ADAPTER.md#cost-tracking-interface)

### What We Track

| Metric | Purpose | Storage |
|--------|---------|---------|
| Cost per generation | Unit economics | Vercel KV |
| Cost by provider | Provider comparison | Vercel KV |
| Cost by user | LTV analysis | Vercel KV |
| Daily/weekly totals | Budget monitoring | Vercel KV |
| Success/failure rates | Quality tracking | Vercel KV |

### Alert Thresholds

```typescript
const COST_ALERTS = {
  dailyBudget: 100,      // $100/day - warn
  dailyCritical: 200,    // $200/day - alert
  perGenMax: 0.15,       // $0.15/gen - investigate
  failureRateMax: 0.05,  // 5% failure - alert
};
```

---

## 5. Reporting Dashboard

### Business Metrics (Daily/Weekly/Monthly)

| Category | Metrics |
|----------|---------|
| **Acquisition** | MAU, DAU, new signups, auth completion rate |
| **Engagement** | Generations/user, session duration, return rate |
| **Sharing** | Share rate, shares/generation, platform breakdown |
| **Revenue** | Affiliate clicks, CTR, conversions, commission |
| **Cost** | AI spend, infra spend, cost/generation, cost/user |
| **Quality** | Generation success rate, avg duration, user satisfaction |

### Key Performance Indicators (KPIs)

```typescript
interface DashboardKPIs {
  // Acquisition
  mau: number;
  dau: number;
  newUsersToday: number;
  authCompletionRate: number;
  
  // Engagement  
  generationsToday: number;
  generationsPerActiveUser: number;
  avgSessionDuration: number;
  
  // Revenue
  clicksToday: number;
  conversionsToday: number;
  revenueToday: number;          // Commission in cents
  conversionRate: number;
  avgOrderValue: number;
  
  // Cost
  aiSpendToday: number;
  costPerGeneration: number;
  costPerUser: number;
  
  // Unit Economics
  revenuePerUser: number;        // LTV proxy
  costPerUser: number;
  marginPerUser: number;
  
  // Quality
  generationSuccessRate: number;
  avgGenerationTime: number;
}
```

### Report Cadence

| Report | Frequency | Audience | Contents |
|--------|-----------|----------|----------|
| Daily Snapshot | Daily 9am | Team Slack | KPIs, anomalies |
| Weekly Summary | Monday | Leadership | Trends, WoW growth |
| Monthly Review | 1st of month | Investors | Full metrics, cohort analysis |
| Cost Alert | Real-time | Eng on-call | Budget threshold breaches |

### Dashboard Access

| Role | Access Level | Features |
|------|--------------|----------|
| Admin | Full | All metrics, user-level data, export |
| Team | Standard | Aggregate metrics, no PII |
| Investor | Read-only | Curated metrics, monthly snapshots |

---

## Implementation Plan

### Sprint 5 Tasks (Analytics Focus)

| Task | Priority | Estimate |
|------|----------|----------|
| Set up Vercel Web Analytics | P0 | 1h |
| Implement custom event tracking | P0 | 4h |
| Build affiliate link generator | P0 | 4h |
| Build click redirect endpoint | P0 | 3h |
| Implement click aggregation | P0 | 3h |
| Build conversion webhook handler | P1 | 4h |
| Implement conversion matching | P1 | 4h |
| Create dashboard API endpoints | P1 | 6h |
| Build dashboard UI | P2 | 8h |
| Set up cost alerts | P1 | 2h |

### Data Retention

| Data Type | Retention | Reason |
|-----------|-----------|--------|
| Web analytics | Per Vercel | Managed service |
| Click aggregates | Indefinite | Revenue attribution |
| Daily click logs | 90 days | Reporting, CCPA |
| Conversion records | 1 year | Tax/audit requirements |
| Cost records | 90 days | Budget analysis |

---

## Privacy Considerations

### Data Minimization
- IP addresses hashed, not stored
- No cross-site tracking
- Cookie-free analytics (Vercel)
- User IDs are internal, not PII

### CCPA Compliance
- Users can request data export
- Users can request deletion
- Affiliate data linked to userId (deletable)
- Analytics are anonymized (non-deletable)

### Partner Data Sharing
- Only share: click counts, conversion rates
- Never share: user identities, PII
- Partners receive: aggregate campaign performance

---

## Related Documents

- [TECH-STACK.md](./TECH-STACK.md) - Technology decisions
- [DATA-FLOW.md](./DATA-FLOW.md) - Overall data architecture
- [PROVIDER-ADAPTER.md](./PROVIDER-ADAPTER.md) - Cost tracking implementation
- [REQUIREMENTS.md](../user-stories/REQUIREMENTS.md) - Feature requirements

---

## Change Log

| Date | Change | Author |
|------|--------|--------|
| 2026-02-09 | Initial analytics architecture | AI Agent |

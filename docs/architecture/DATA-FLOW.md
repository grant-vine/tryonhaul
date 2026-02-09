# Data Flow & Integrations

> **Last Updated**: 2026-02-09  
> **Related**: [OVERVIEW.md](./OVERVIEW.md) | [TECH-STACK.md](./TECH-STACK.md) | [SHARING-STRATEGY.md](./SHARING-STRATEGY.md) | [PROVIDER-ADAPTER.md](./PROVIDER-ADAPTER.md) | [REQUIREMENTS.md](../user-stories/REQUIREMENTS.md)

---

## Purpose

This document describes how data flows through Try on Haul and how external services are integrated.

---

## Core User Flow Data Journey

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        USER JOURNEY DATA FLOW                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  1. LANDING          2. AUTH              3. CONSENT                        │
│  ┌──────────┐       ┌──────────┐         ┌──────────┐                      │
│  │ Marketing│──────▶│ Social   │────────▶│ Image    │                      │
│  │ Page     │       │ OAuth    │         │ Consent  │                      │
│  └──────────┘       └──────────┘         │ Modal    │                      │
│       │                  │               └──────────┘                      │
│       │                  ▼                    │                            │
│       │             ┌──────────┐              │                            │
│       │             │ Clerk    │◀─────────────┘ (save consent)             │
│       │             │ Session  │                                           │
│       │             └──────────┘                                           │
│       │                                                                    │
│  4. UPLOAD           5. OUTFIT             6. SCENE (Optional)             │
│  ┌──────────┐       ┌──────────┐         ┌──────────┐                      │
│  │ Reference│──────▶│ Outfit   │────────▶│ Festival │                      │
│  │ Photo    │       │ Image    │         │ Scene    │                      │
│  └──────────┘       └──────────┘         └──────────┘                      │
│       │                  │                    │                            │
│       ▼                  ▼                    ▼                            │
│  ┌─────────────────────────────────────────────────┐                       │
│  │              Vercel Blob (Temp Storage)         │                       │
│  │              TTL: 1 hour                        │                       │
│  └─────────────────────────────────────────────────┘                       │
│                          │                                                 │
│                          ▼                                                 │
│  7. GENERATION       ┌──────────┐                                          │
│  ┌──────────┐       │ Inngest  │                                          │
│  │ Queue    │◀──────│ Job      │                                          │
│  │ Request  │       └──────────┘                                          │
│  └──────────┘            │                                                 │
│       │                  ▼                                                 │
│       │            ┌──────────┐                                            │
│       │            │ CatVTON  │ (fal.ai API)                               │
│       │            │ AI Model │                                            │
│       │            └──────────┘                                            │
│       │                  │                                                 │
│       ▼                  ▼                                                 │
│  8. RESULT          ┌──────────┐                                           │
│  ┌──────────┐       │ Generated│                                           │
│  │ Notify   │◀──────│ Image    │                                           │
│  │ User     │       └──────────┘                                           │
│  └──────────┘            │                                                 │
│                          ▼                                                 │
│  9. SHARE           ┌──────────┐                                           │
│  ┌──────────┐       │ Share    │───────▶ Social Platforms                  │
│  │ Download │       │ Card +   │         (FB, IG, TikTok, X, Pinterest)    │
│  │ or Share │       │ Link     │                                           │
│  └──────────┘       └──────────┘                                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Integration Details

### 1. Authentication (Clerk)

**Flow:**
```
User ──▶ "Login with Facebook" ──▶ Facebook OAuth
                                       │
                                       ▼
                              Facebook returns token
                                       │
                                       ▼
                              Clerk validates + creates session
                                       │
                                       ▼
                              Return session cookie to client
```

**Data Exchanged:**
| From | To | Data |
|------|-----|------|
| Facebook | Clerk | User ID, email, name, profile photo |
| Clerk | Our App | Session token, user object |
| Our App | Clerk | Consent status (custom claim) |

**API Endpoints:**
- `POST /api/auth/callback/:provider` - OAuth callback
- `GET /api/auth/session` - Get current session
- `DELETE /api/auth/session` - Logout

---

### 2. Image Upload (Vercel Blob)

**Flow:**
```
Client ──▶ Request signed URL ──▶ API Route
                                      │
                                      ▼
                              Vercel Blob signed URL
                                      │
                                      ▼
Client ──▶ Direct upload to Blob ──▶ Vercel CDN
                                      │
                                      ▼
                              Return blob URL to client
```

**Data Stored:**
| Data | Storage | Retention | Size Limit |
|------|---------|-----------|------------|
| Reference photo | Vercel Blob | 1 hour | 10MB |
| Outfit image | Vercel Blob | 1 hour | 10MB |
| Generated result | Vercel Blob | 24 hours | 20MB |

**API Endpoints:**
- `POST /api/upload/signed-url` - Get signed upload URL
- `DELETE /api/upload/:blobId` - Manual delete (optional)

---

### 3. Shopify Partner Integration

**Flow:**
```
User pastes URL ──▶ Parse domain + handle
                          │
                          ▼
                   Check partner mapping
                          │
                          ▼
           ┌──────────────┴──────────────┐
           │                             │
     Partner Store                  Not Partner
           │                             │
           ▼                             ▼
    Query Storefront API         "Store not available"
           │                     (Show partner signup CTA)
           ▼
    Return product images
```

**Storefront API Query:**
```graphql
query getProductImages($handle: String!) {
  product(handle: $handle) {
    id
    title
    featuredImage { url altText }
    images(first: 10) {
      edges {
        node { url width height }
      }
    }
  }
}
```

**Partner Data Storage:**
| Data | Storage | Purpose |
|------|---------|---------|
| Domain → Token mapping | Vercel KV | API access |
| Partner metadata | Vercel KV | Display name, affiliate ID |

---

### 4. AI Generation (Provider Adapter)

> **Full Architecture**: [PROVIDER-ADAPTER.md](./PROVIDER-ADAPTER.md)

**Flow:**
```
API Route ──▶ Inngest Event ──▶ Background Job
                                      │
                                      ▼
                              Fetch images from Blob
                                      │
                                      ▼
                              ┌───────────────────┐
                              │ TryOnService      │
                              │ (Provider Adapter)│
                              └─────────┬─────────┘
                                        │
                      ┌─────────────────┼─────────────────┐
                      │                 │                 │
               ┌──────▼──────┐   ┌──────▼──────┐   ┌──────▼──────┐
               │ fal.ai      │   │ FASHN API   │   │ Self-hosted │
               │ (CatVTON/   │   │ (backup)    │   │ (future)    │
               │  IDM-VTON)  │   │             │   │             │
               └──────┬──────┘   └──────┬──────┘   └──────┬──────┘
                      │                 │                 │
                      └─────────────────┴─────────────────┘
                                        │
                                        ▼
                              Wait for result (10-70s)
                                        │
                                        ▼
                              Record cost to tracker
                                        │
                                        ▼
                              Return result to client
                                        │
                                        ▼
                              Notify user (push/email/toast)
```

**Provider Selection:**
- Primary: fal.ai CatVTON (fast, good accuracy)
- Fallback: FASHN API (commercial, scene support)
- Future: Self-hosted for cost optimization

**Cost Tracking:**
- Every generation logged with provider, cost, duration
- Used for breakeven analysis

---

### 5. Social Sharing

> **Full Strategy**: [SHARING-STRATEGY.md](./SHARING-STRATEGY.md)

**Key Decision**: Web Share API Level 2 (file sharing) as primary approach.
This supports "no server storage" - images shared directly from browser memory.

**Share Flow:**
```
Generated Image ──▶ Add branding overlay (client-side)
                           │
                           ▼
                    Generate affiliate short link
                           │
                           ▼
                    Check: navigator.canShare({ files })?
                           │
              ┌────────────┴────────────┐
              │                         │
           Yes (iOS/Android)       No (Firefox/older)
              │                         │
              ▼                         ▼
       navigator.share({          Show fallback modal:
         files: [imageBlob],      - Download image
         text: caption,           - Copy link
         url: affiliateLink       - Platform buttons
       })                               │
              │                         ▼
              ▼                   User shares manually
       OS Share Sheet
              │
              ▼
       User selects app
       (Instagram/TikTok/FB/X)
```

**Platform Limitations (from research):**
| Platform | Web Share | Direct API Posting | Notes |
|----------|-----------|-------------------|-------|
| Instagram | ✅ via share sheet | ❌ Native SDK only | No web API for media injection |
| TikTok | ✅ via share sheet | ❌ Native SDK only | Share Kit is native-only |
| Facebook | ✅ via share sheet | ⚠️ Graph API (needs storage) | Would need temp image hosting |
| X | ✅ via share sheet | ✅ Web Intent (text/URL only) | Image via share sheet only |
| Pinterest | ✅ via share sheet | ⚠️ Save button (needs URL) | Needs hosted image URL |

**Storage Implication**: 
- Web Share API = No server storage needed
- Pinterest Save Button = Requires temporary hosted URL (deferred to post-MVP)
- Facebook Graph API = Requires temporary hosted URL (deferred to post-MVP)

---

### 6. Affiliate Link Tracking

**Link Generation Flow:**
```
Generate result ──▶ Extract outfit source
                         │
                         ▼
                  Create affiliate link
                         │
                         ▼
                  Store in Vercel KV
                         │
                         ▼
                  Return short URL
```

**Link Click Flow:**
```
User clicks link ──▶ Edge Function intercepts
                          │
                          ▼
                   Record click in KV
                          │
                          ▼
                   Send to Plausible
                          │
                          ▼
                   302 Redirect to destination
```

**KV Data Structure:**
```json
{
  "link:abc123": {
    "destination": "https://partner.com/products/dress",
    "userId": "user_xxx",
    "generationId": "gen_yyy",
    "productId": "prod_zzz",
    "created": 1707500000,
    "clicks": 42
  }
}
```

---

### 7. Rate Limiting

**Check Flow:**
```
API Request ──▶ Get user ID from session
                      │
                      ▼
               Query KV for today's count
                      │
          ┌───────────┴───────────┐
          │                       │
     Count < 5               Count >= 5
          │                       │
          ▼                       ▼
    Increment counter        Return 429 error
          │                  "Limit reached"
          ▼
    Process request
```

**KV Data Structure:**
```json
{
  "ratelimit:user_xxx:2026-02-09": 3,  // TTL: 24 hours
}
```

---

## API Route Summary

| Route | Method | Purpose |
|-------|--------|---------|
| `/api/auth/callback/:provider` | GET | OAuth callback |
| `/api/auth/session` | GET/DELETE | Session management |
| `/api/upload/signed-url` | POST | Get upload URL |
| `/api/generate` | POST | Queue generation job |
| `/api/generate/:id` | GET | Check generation status |
| `/api/share/create` | POST | Create share link |
| `/l/:shortId` | GET | Affiliate link redirect |
| `/api/shopify/product` | POST | Fetch product from Shopify |

---

## Webhook Endpoints

| Endpoint | Source | Purpose |
|----------|--------|---------|
| `/api/webhooks/fal` | fal.ai | Generation completion |
| `/api/webhooks/clerk` | Clerk | User events (delete, etc.) |
| `/api/webhooks/partner` | Partner stores | Conversion tracking |

---

## Data Privacy Compliance

### Data Flow Diagram (Privacy Focused)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        DATA LIFECYCLE                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  USER DATA                                                                  │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │ Collected: Email, name, profile photo (from social auth)            │  │
│  │ Stored: Clerk (managed service, SOC2 compliant)                     │  │
│  │ Retention: Until user deletes account                               │  │
│  │ Access: User can export/delete via settings                         │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  REFERENCE PHOTOS                                                           │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │ Collected: User uploads                                             │  │
│  │ Stored: Vercel Blob (temporary)                                     │  │
│  │ Retention: 1 hour maximum                                           │  │
│  │ Processing: Sent to AI API, then deleted                            │  │
│  │ NOT stored permanently                                              │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  GENERATED IMAGES                                                           │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │ Created: AI generation                                              │  │
│  │ Stored: Vercel Blob (temporary)                                     │  │
│  │ Retention: 24 hours                                                 │  │
│  │ Delivery: User downloads or shares directly                         │  │
│  │ NOT stored permanently                                              │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  ANALYTICS DATA                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │ Collected: Page views, clicks, conversions (anonymized)             │  │
│  │ Stored: Plausible (GDPR-compliant, no cookies)                      │  │
│  │ Retention: Per Plausible policy                                     │  │
│  │ No PII collected                                                    │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### CCPA Compliance Points

1. **Right to Know**: User can see what data we have via Clerk dashboard
2. **Right to Delete**: User can delete account; all data removed
3. **Right to Opt-Out**: No data selling; analytics are anonymized
4. **Non-Discrimination**: No service differences based on privacy choices

---

## Error Handling

| Error Type | Response | User Action |
|------------|----------|-------------|
| Upload failed | 500 + retry | Auto-retry 3x, then manual |
| Generation timeout | 504 + queue retry | Notify when complete |
| AI API error | 502 + fallback | Try backup provider |
| Rate limit | 429 + reset time | Wait or upgrade |
| Auth expired | 401 + redirect | Re-authenticate |

---

## Change Log

| Date | Change | Author |
|------|--------|--------|
| 2026-02-09 | Initial data flow documentation | AI Agent |

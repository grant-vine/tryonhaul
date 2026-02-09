# Authentication Solutions Comparison

> **Purpose**: Comprehensive comparison of authentication solutions for Try on Haul  
> **Last Updated**: 2026-02-09  
> **Related Docs**: [TECH-STACK.md](../architecture/TECH-STACK.md), [MVP-FEATURES.md](../user-stories/MVP-FEATURES.md)

---

## Overview

This document compares authentication solutions for a Next.js/Vercel PWA requiring social-only authentication (Facebook, Instagram, TikTok, Apple) as a marketing gate for viral potential.

**Key Requirements:**
- Social authentication only (Facebook, Instagram, TikTok, Apple)
- Next.js/Vercel compatibility
- Low operational overhead
- Cost-effective at scale
- Privacy-first approach

---

## Clerk Authentication

### What Clerk Provides

Clerk is a complete authentication and user management platform with pre-built UI components, admin dashboards, and seamless integration with modern frameworks like Next.js.

**Core Features:**
- Drop-in UI components for sign-up, sign-in, user profiles
- Organization management for B2B use cases
- Multi-factor authentication (MFA)
- Session management with custom lifetimes
- User metadata storage
- Webhooks for data synchronization
- Bot protection and brute-force prevention
- Custom domains and SSL
- SOC2 and HIPAA compliance (Business tier+)

**Social Providers:**
- Supports 50+ OAuth providers including:
  - ✅ Facebook
  - ✅ Apple
  - ✅ TikTok (requires custom credentials - no shared dev credentials)
  - ⚠️ Instagram (not directly listed - may require workaround via Facebook)

### Clerk Pricing (2026)

#### Hobby Plan (Free)
- **Cost**: $0/month
- **Included**:
  - 50,000 MRU (Monthly Retained Users) per app
  - 3 dashboard seats
  - Up to 3 social connections
  - Custom domain
  - Fixed 7-day session lifetime
  - Basic auth features (email codes, email links, passwords, usernames)
  - Web3 wallet support
  - Webhooks
  - User metadata

**Limitations:**
- Cannot remove Clerk branding
- No MFA
- Limited to 3 social providers (problematic for our 4 providers)
- Cannot customize session lifetime

#### Pro Plan
- **Cost**: $20/month (billed annually) or $25/month (monthly)
- **Usage Pricing**:
  - First 50,000 MRUs included
  - $0.02 per MRU for 50,001-100,000
  - $0.018 per MRU for 100,001-1,000,000
  - $0.015 per MRU for 1,000,001-10,000,000
  - $0.012 per MRU for 10,000,001+

- **Included**:
  - Everything in Hobby
  - Remove Clerk branding
  - **Unlimited social connections** ✅
  - Multi-factor authentication
  - Custom session lifetime
  - Passkeys
  - Custom password requirements
  - Custom email/SMS templates
  - SMS authentication (pay-per-SMS)
  - 1 Enterprise connection included ($75/mo for additional)

#### Business Plan
- **Cost**: $250/month (billed annually) or $300/month (monthly)
- Same MRU pricing as Pro
- Additional features:
  - 10 dashboard seats included ($20/mo each additional)
  - SOC2 Report & HIPAA compliance
  - Enhanced dashboard roles
  - Priority support
  - Audit logs (coming soon)

#### Enterprise Plan
- **Cost**: Custom (annual commitment only)
- Everything in Business plus:
  - Volume discounts
  - 99.99% Uptime SLA
  - Premium support + dedicated Slack channel
  - Onboarding & migration support

### Clerk Add-ons

**B2B Authentication** (Not needed for MVP):
- Free: 100 MRO (Monthly Retained Organizations) per app
- Enhanced ($100/mo or $85/mo annual): Advanced org features

**Administration**:
- Free: 5 impersonations/month
- Enhanced ($100/mo or $85/mo annual): Unlimited impersonations

**Billing** (Not needed for MVP):
- 0.7% of billing volume + Stripe fees

### Clerk Cost Examples

**Example 1: 10,000 Active Users (MVP Phase)**
- Pro Plan: $20/month (all users within free tier)
- **Total: $20/month**

**Example 2: 75,000 Active Users (Growth Phase)**
- Pro Plan base: $20/month
- Additional users: 25,000 × $0.02 = $500/month
- **Total: $520/month**

**Example 3: 250,000 Active Users (Scale Phase)**
- Pro Plan base: $20/month
- 50,001-100,000: 50,000 × $0.02 = $1,000
- 100,001-250,000: 150,000 × $0.018 = $2,700
- **Total: $3,720/month**

### TikTok Integration Note

⚠️ **Important**: TikTok OAuth with Clerk requires custom credentials. Shared development credentials don't work because TikTok requires URL ownership verification for all redirect URLs. Individual developers can't verify ownership of Clerk's development URL (`accounts.dev`).

**Solution**: Use sandbox app for development, or test in staging/preview environment.

### Clerk Pros & Cons

✅ **Pros:**
- Best-in-class developer experience with Next.js
- Pre-built, customizable UI components
- Fast integration (can be set up in minutes)
- Handles all security best practices
- Excellent documentation
- Active community and support
- Includes user management dashboard
- MRU pricing model is fair (only charges for retained users)

❌ **Cons:**
- **Most expensive option at scale** ($3,720/month at 250K users)
- Requires Pro plan for 4+ social providers ($20/month minimum)
- Vendor lock-in (proprietary platform)
- TikTok integration complexity
- Instagram not explicitly supported (may require workaround)

---

## Supabase Auth

### What Supabase Provides

Supabase is an open-source Firebase alternative offering authentication as part of a complete backend-as-a-service platform.

**Core Features:**
- Email/password authentication
- Magic links (passwordless email)
- Phone authentication (SMS)
- Social login (OAuth)
- Anonymous sign-ins
- Multi-factor authentication
- Session management
- Row Level Security (RLS) integration with PostgreSQL
- Custom JWT claims
- Email templates (customizable)
- Rate limiting
- Bot detection (CAPTCHA)
- Audit logs
- Enterprise SSO (SAML/OIDC)

**Social Providers Supported:**
- ✅ Google
- ✅ Facebook
- ✅ Apple
- ✅ Azure (Microsoft)
- ✅ Twitter
- ✅ GitHub
- ✅ GitLab
- ✅ Bitbucket
- ✅ Discord
- ✅ Figma
- ✅ Kakao
- ✅ Keycloak
- ✅ LinkedIn
- ✅ Notion
- ✅ Slack
- ✅ Spotify
- ✅ Twitch
- ✅ WorkOS
- ✅ Zoom
- ⚠️ **TikTok: NOT explicitly listed** (may require custom OAuth implementation)
- ⚠️ **Instagram: NOT explicitly listed** (may require custom OAuth or Facebook Graph API workaround)

### Supabase Pricing (2026)

#### Free Tier
- **Cost**: $0/month
- **Included**:
  - Unlimited API requests
  - 50,000 Monthly Active Users
  - 500 MB database storage
  - 1 GB file storage
  - 2 GB bandwidth
  - Social OAuth (all supported providers)
  - Email authentication
  - Phone authentication (pay-per-SMS)

#### Pro Tier
- **Cost**: $25/month per project
- **Included**:
  - Everything in Free
  - 100,000 Monthly Active Users included
  - 8 GB database storage
  - 100 GB file storage
  - 200 GB bandwidth
  - Daily backups
  - 7-day log retention
  - Email support

**Overages:**
- Additional MAUs: **$0.00325 per MAU** (extremely cheap!)
- Database storage: $0.125/GB
- File storage: $0.021/GB
- Bandwidth: $0.09/GB

#### Team Tier
- **Cost**: $599/month per project
- Everything in Pro plus:
  - SOC2 Type 2 compliance
  - HIPAA compliance available
  - Priority support
  - Designated support manager

#### Enterprise Tier
- Custom pricing
- Custom SLAs
- Dedicated infrastructure options

### Supabase Cost Examples

**Example 1: 10,000 Active Users (MVP Phase)**
- Free tier covers it completely
- **Total: $0/month**

**Example 2: 75,000 Active Users (Growth Phase)**
- Free tier: First 50,000 users = $0
- Remaining: 25,000 × $0 (still within Free tier MAU limit)
- **Total: $0/month** ✅

**Example 3: 250,000 Active Users (Scale Phase)**
- Pro tier base: $25/month
- First 100,000 users included
- Additional users: 150,000 × $0.00325 = $487.50/month
- **Total: $512.50/month** ✅ (Compare to Clerk's $3,720!)

**Example 4: 1,000,000 Active Users (Major Scale)**
- Pro tier base: $25/month
- Additional users: 900,000 × $0.00325 = $2,925/month
- **Total: $2,950/month** (Still less than Clerk at 250K users)

### Supabase Auth Special Considerations

**Phone Authentication (SMS) Costs:**
- Billed separately per SMS sent
- Rates vary by country (~$0.01-$0.10 per SMS)
- Not relevant for our social-only auth approach

**Apple OAuth Maintenance:**
- ⚠️ **Secret key rotation required every 6 months** for OAuth flow
- Must store `.p8` file securely for rotation
- Critical maintenance task to prevent auth failures
- Native iOS auth doesn't have this requirement

**Provider Token Management:**
- Supabase doesn't auto-refresh provider tokens
- App must handle refresh token logic
- Provider tokens not stored in database (security best practice)

### Supabase Pros & Cons

✅ **Pros:**
- **Extremely cost-effective** (cheapest option by far)
- Free tier covers most MVP/early growth needs
- Open-source (can self-host if needed)
- Includes full database (PostgreSQL) and storage
- Row Level Security for fine-grained access control
- Very generous free tier (50K MAUs)
- Active community and excellent documentation
- No vendor lock-in (uses standard PostgreSQL)
- Simple, predictable pricing

❌ **Cons:**
- **No native TikTok provider** (requires custom OAuth implementation)
- **No native Instagram provider** (requires custom OAuth implementation)
- More setup work than Clerk (no pre-built UI components)
- Need to build own user management UI
- Apple OAuth requires 6-month key rotation maintenance
- Less hand-holding than Clerk (more DIY)

---

## NextAuth.js / Auth.js

### What NextAuth Provides

NextAuth.js (now Auth.js) is an open-source authentication library designed specifically for Next.js applications. It's framework-agnostic in v5.

**Core Features:**
- OAuth 1.0, 1.0A, 2.0, and OpenID Connect support
- Email (magic link) authentication
- Credentials-based authentication
- JWT or database sessions
- Built-in security best practices
- TypeScript support
- Adapter system for multiple databases

**Social Providers Supported (50+ providers):**
- ✅ Facebook
- ✅ Apple
- ✅ Instagram
- ✅ **TikTok** (has dedicated provider)
- ✅ Google, GitHub, Twitter, Discord, Spotify, Twitch, etc.

### NextAuth Pricing

**Cost**: **$0 - Completely Free** (open-source)

**Operational Costs:**
- Hosting costs (covered by Vercel free tier for reasonable usage)
- Database costs (if using database sessions):
  - Vercel PostgreSQL: Free tier available, ~$20+/month for production
  - PlanetScale (MySQL): Free tier, ~$29+/month for production
  - MongoDB Atlas: Free tier, ~$9+/month for production
- Email service (if using magic links):
  - Resend: 3,000 emails/month free, then $20/month for 50K emails
  - SendGrid: 100 emails/day free, paid plans start at $19.95/month

**Estimated Total Cost:**
- MVP: $0-$20/month (using free tiers)
- Production: $20-$100/month (depending on database and email choices)

### NextAuth Pros & Cons

✅ **Pros:**
- **Completely free and open-source**
- **Native TikTok provider** ✅
- **Native Instagram provider** ✅
- Full control over authentication flow
- No vendor lock-in
- Battle-tested and widely used
- Excellent Next.js integration
- Active community
- Supports all required social providers out-of-box
- Can use any database adapter

❌ **Cons:**
- **Most implementation work required**
- No pre-built UI components (must build your own)
- No built-in user management dashboard
- Need to handle security updates yourself
- More code to maintain
- Database setup and management required
- Email service setup required (if using magic links)
- Need to implement your own admin features

---

## Firebase Authentication

### What Firebase Provides

Firebase Authentication is Google's backend-as-a-service authentication solution, deeply integrated with Google Cloud Platform.

**Core Features:**
- Email/password authentication
- Email verification and password recovery
- Phone authentication (SMS/OTP)
- Anonymous authentication
- Social logins (OAuth)
- Custom authentication (JWT)
- Multi-factor authentication
- Session management
- Integrates with Firestore, Cloud Functions, etc.
- Firebase UI components available (but not as polished as Clerk)

**Social Providers Supported:**
- ✅ Google
- ✅ Facebook
- ✅ Apple
- ✅ Microsoft
- ✅ Twitter
- ✅ GitHub
- ✅ Yahoo
- ⚠️ **TikTok**: Not officially supported (custom implementation required)
- ⚠️ **Instagram**: Not officially supported (workaround via Facebook Graph API)

### Firebase Pricing (2026)

#### Spark Plan (Free)
- **Cost**: $0/month
- **Included**:
  - Unlimited email/password and social logins (FREE)
  - Phone authentication: pay-per-SMS
  - No MAU limits for basic auth
  - Anonymous auth included

#### Blaze Plan (Pay-as-you-go)
- **Base Cost**: $0/month (only pay for what you use)
- **Authentication Costs**:
  - Email/social/anonymous: **FREE** (no MAU charges)
  - Phone authentication (SMS): $0.01-$0.10+ per SMS (varies by country)

**Google Cloud Identity Platform (for advanced features):**
- Free: Up to 50,000 MAUs
- Paid: Tiered pricing starts after 50K MAUs
  - SAML/OIDC: 50 MAUs free, then paid
  - Enterprise features: Multi-tenancy, advanced security

### Firebase Cost Examples

**All Examples (Email + Social Auth Only):**
- 10,000 users: **$0/month**
- 100,000 users: **$0/month**
- 1,000,000 users: **$0/month**
- 10,000,000 users: **$0/month**

**Note**: Social authentication is completely free with Firebase. You only pay for:
- Phone authentication (SMS costs)
- Advanced Identity Platform features (SAML, multi-tenancy)
- Other Firebase services (Firestore, Storage, Functions, etc.)

### Firebase Pros & Cons

✅ **Pros:**
- **Social authentication is completely FREE** (no MAU limits!)
- Zero cost at any scale for social-only auth
- Battle-tested at massive scale (Google infrastructure)
- No billing surprises for authentication
- Official Google SDKs for web/iOS/Android
- Good documentation
- Integrates well with other Firebase/GCP services
- Simple to get started

❌ **Cons:**
- **No native TikTok support** (requires custom implementation)
- **No native Instagram support** (requires workaround)
- Vendor lock-in to Google ecosystem
- UI components less polished than Clerk
- Limited customization compared to open-source options
- Google has deprecated Firebase projects before
- Authentication tied to larger Firebase ecosystem
- May need to use other Firebase services (Firestore for user data)

---

## Lucia Auth

### What Lucia Provides

Lucia is a lightweight, open-source authentication library that focuses on session management. It's framework-agnostic and database-agnostic.

**Core Features:**
- Session management (core focus)
- Database adapter system (works with any database)
- Framework adapters (Next.js, SvelteKit, Express, etc.)
- Password hashing helpers
- OAuth integration via Arctic library
- Lightweight and minimal dependencies
- Full TypeScript support
- No opinionated UI

**OAuth Support:**
- Uses the **Arctic** library for OAuth 2.0
- Supports 50+ providers including:
  - ✅ Facebook
  - ✅ Apple
  - ✅ TikTok
  - ❓ Instagram (depends on Arctic support)

### Lucia Pricing

**Cost**: **$0 - Completely Free** (open-source)

**Operational Costs** (similar to NextAuth):
- Database hosting (Vercel Postgres, PlanetScale, etc.): $0-$20+/month
- Email service (if implementing magic links): $0-$20+/month
- Total estimated: $0-$50/month depending on scale and services

### Lucia Pros & Cons

✅ **Pros:**
- **Completely free and open-source**
- Very lightweight (minimal dependencies)
- Full control over implementation
- Framework-agnostic (works beyond Next.js)
- Database-agnostic (use any DB)
- Modern, well-designed API
- Great TypeScript support
- Active development and community
- No vendor lock-in
- **OAuth support via Arctic** includes TikTok

❌ **Cons:**
- **Most hands-on implementation required**
- No pre-built UI components
- Smaller community than NextAuth
- Requires understanding of session management
- More boilerplate code to write
- Need to integrate Arctic separately for OAuth
- No built-in user management features
- Steeper learning curve for beginners
- Documentation less extensive than alternatives

---

## Self-Hosted Options

### Keycloak
- **Cost**: Free (open-source)
- **Complexity**: High (enterprise-grade, heavy)
- **Pros**: Full-featured, SAML/OIDC, battle-tested
- **Cons**: Overkill for our use case, requires dedicated infrastructure

### Authentik
- **Cost**: Free (open-source)
- **Complexity**: Medium-High
- **Pros**: Modern UI, good docs, Docker-friendly
- **Cons**: Requires self-hosting infrastructure, more ops work

### SuperTokens
- **Cost**: Free (open-source) + managed option ($49+/month)
- **Complexity**: Medium
- **Pros**: Built for developers, Next.js support, managed option available
- **Cons**: Smaller community, less mature than alternatives

---

## Comparison Table

| Feature | Clerk | Supabase Auth | NextAuth.js | Firebase Auth | Lucia Auth |
|---------|-------|---------------|-------------|---------------|------------|
| **Cost (10K users)** | $20/mo | $0 | $0-20 | $0 | $0-20 |
| **Cost (100K users)** | $520/mo | $0 | $0-20 | $0 | $0-20 |
| **Cost (250K users)** | $3,720/mo | $512.50/mo | $0-50 | $0 | $0-50 |
| **Facebook** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Apple** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **TikTok** | ⚠️ Custom creds | ❌ Custom | ✅ Native | ❌ Custom | ✅ Via Arctic |
| **Instagram** | ⚠️ Workaround | ❌ Custom | ✅ Native | ⚠️ Via Facebook | ❓ Check Arctic |
| **Pre-built UI** | ✅ Excellent | ❌ | ❌ | ⚠️ Basic | ❌ |
| **User Dashboard** | ✅ Built-in | ✅ Basic | ❌ | ⚠️ Firebase Console | ❌ |
| **Setup Time** | ⏱️ Minutes | ⏱️ Hours | ⏱️ Days | ⏱️ Hours | ⏱️ Days |
| **Vendor Lock-in** | ❌ High | ⚠️ Medium | ✅ None | ❌ High | ✅ None |
| **Next.js DX** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Documentation** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Maintenance** | 🟢 Low | 🟡 Medium | 🟡 Medium | 🟢 Low | 🟡 Medium |
| **Scalability** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Open Source** | ❌ | ✅ | ✅ | ❌ | ✅ |

---

## Key Considerations for Try on Haul

### 1. Social Provider Support

**TikTok Support:**
- ✅ **Best**: NextAuth.js (native provider)
- ✅ **Good**: Lucia Auth (via Arctic library)
- ⚠️ **Requires Work**: Clerk (custom credentials, no dev mode), Supabase, Firebase

**Instagram Support:**
- ✅ **Native**: NextAuth.js
- ⚠️ **Workaround**: Most others (via Facebook Graph API)

### 2. Cost at Scale

**Free Options (for social auth):**
1. Firebase Auth: $0 at any scale
2. NextAuth.js: $0-$50/month (DB + hosting costs only)
3. Lucia Auth: $0-$50/month (DB + hosting costs only)

**Paid Options:**
1. Supabase: $0-512/month (cheap, includes full backend)
2. Clerk: $20-$3,720/month (expensive but comprehensive)

### 3. Development Speed

**Fastest to MVP:**
1. Clerk (minutes with pre-built UI)
2. Firebase (hours with Firebase UI)
3. Supabase (hours to days)
4. NextAuth (days)
5. Lucia (days)

### 4. Maintenance Overhead

**Lowest Maintenance:**
1. Clerk (fully managed)
2. Firebase (fully managed)
3. Supabase (mostly managed)
4. NextAuth (self-managed)
5. Lucia (self-managed)

### 5. Privacy First Approach

**Best for Privacy:**
1. Lucia Auth (self-hosted, full control)
2. NextAuth.js (self-hosted, full control)
3. Supabase (open-source, can self-host)
4. Clerk (vendor hosted)
5. Firebase (Google hosted)

---

## Summary Recommendations

### For Fastest MVP (Speed > Cost)
**Recommendation**: Firebase Auth or Clerk
- **Firebase** if cost is critical ($0 forever for social auth)
- **Clerk** if UI/UX quality and dev speed are critical ($20-520/month)
- Both sacrifice TikTok native support (needs custom implementation)

### For Best Value (Cost + Features)
**Recommendation**: Supabase Auth + NextAuth.js hybrid
- Use **Supabase** for backend + database ($0-512/month)
- Use **NextAuth.js** for auth logic (native TikTok/Instagram)
- Best of both worlds: cheap, full-featured, native provider support

### For Full Control (Privacy + Flexibility)
**Recommendation**: NextAuth.js or Lucia Auth
- **NextAuth.js** for more mature ecosystem and native providers
- **Lucia Auth** for lightweight, modern approach
- Both require more implementation work but give complete control
- Cost: $0-50/month (just infrastructure)

### For Long-term Scale (Future-proof)
**Recommendation**: NextAuth.js with PostgreSQL
- No vendor lock-in
- Native support for all required providers
- Scales infinitely without per-user costs
- Open-source and community-driven
- Most flexible for future needs

---

## Technical Notes

### Instagram Authentication Challenges

Instagram doesn't offer direct OAuth for web apps like other platforms. Common workarounds:

1. **Facebook Graph API Method**:
   - Authenticate via Facebook
   - Request Instagram account linking
   - Access Instagram data via Graph API
   - Requires Facebook app setup + Instagram Business Account

2. **Instagram Basic Display API**:
   - Limited to personal accounts
   - Cannot be used for authentication (only data access)
   - Not suitable for our use case

3. **Third-party Solutions**:
   - Some auth providers (like NextAuth) have Instagram providers
   - Often rely on Facebook Graph API under the hood

**Recommendation**: Verify Instagram requirement with stakeholders. May need to substitute with Facebook auth or drop Instagram from MVP.

### TikTok Authentication Challenges

TikTok OAuth requires:
- App verification in TikTok Developer Portal
- Production URL verification (can't use localhost)
- Sandbox environment for testing
- Invited test users during development

**Solutions**:
- Use NextAuth.js with native TikTok provider (simplest)
- Use tunneling service (ngrok) for local development
- Set up staging environment for testing

---

## Change Log

| Date | Change | Author |
|------|--------|--------|
| 2026-02-09 | Initial authentication comparison research | AI Agent |

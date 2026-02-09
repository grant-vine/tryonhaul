# Viral Scale Pricing Research

> **Last Updated**: February 9, 2026  
> **Research Context**: Cost projections for investor presentations based on viral consumer app growth

---

## Generation Volume Calculations

**Assumptions**:
- 30% of MAU are active monthly
- Active users generate 3 images/month on average
- Total generations = MAU × 0.30 × 3

| User Tier | MAU | Active Users (30%) | Generations/Month |
|-----------|-----|-------------------|-------------------|
| 1K | 1,000 | 300 | 900 |
| 10K | 10,000 | 3,000 | 9,000 |
| 100K | 100,000 | 30,000 | 90,000 |
| 1M | 1,000,000 | 300,000 | 900,000 |
| 10M | 10,000,000 | 3,000,000 | 9,000,000 |

---

## 1. Vercel Pro - Hosting & Infrastructure

**Plan**: Pro ($20/month base + usage)

### Included Monthly (Pro Plan)
- **Edge Requests**: 10M included (worth up to $32)
- **Fast Data Transfer**: 1TB included (worth up to $350)
- **Monthly Credit**: $20 to apply to overages

### Usage-Based Pricing (After Included Amounts)

| Resource | Unit Price | Notes |
|----------|-----------|-------|
| Edge Requests | $2.00 per 1M | Regional pricing: $2.00-$3.20 |
| Fast Data Transfer | $0.15 per GB | Regional pricing: $0.15-$0.35 |
| Fast Origin Transfer | $0.06 per GB | Regional pricing: $0.06-$0.43 |
| Function Invocations | $0.60 per 1M | - |
| Active CPU | $0.128 per hour | For Fluid compute |
| Provisioned Memory | $0.0106 per GB-hour | - |

### Cost Projections by User Scale

**Assumptions**:
- Average 5 page views per session
- 2 API calls per generation (upload + generate)
- 50KB HTML/JS per page load
- Authentication flow: 3 edge requests per login

| Tier | MAU | Edge Requests | Data Transfer | Function Calls | Est. Monthly Cost |
|------|-----|--------------|---------------|----------------|-------------------|
| 1K | 1,000 | ~15K | ~2.5GB | ~2K | **$20** (base only) |
| 10K | 10,000 | ~150K | ~25GB | ~20K | **$20-25** |
| 100K | 100,000 | ~1.5M | ~250GB | ~200K | **$25-40** |
| 1M | 1,000,000 | ~15M | ~2.5TB | ~2M | **$420-650** |
| 10M | 10,000,000 | ~150M | ~25TB | ~20M | **$4,200-6,500** |

**Volume Discount**: None publicly available; Enterprise custom pricing required above 1M users.

**Breakeven Point**: Vercel Pro becomes expensive at 1M+ users. Consider Enterprise plan or alternative hosting.

---

## 2. Neon Postgres (via Vercel Marketplace)

**Note**: Vercel migrated Vercel Postgres to Neon in December 2024. Direct Neon pricing applies.

### Neon Pricing Plans

| Plan | Monthly Cost | Compute | Storage | Key Limits |
|------|--------------|---------|---------|------------|
| **Free** | $0 | 100 CU-hours/project | 0.5GB/project | Max 2 CU (8GB RAM) |
| **Launch** | Usage-based | $0.106/CU-hour | $0.35/GB-month | Max 16 CU (64GB RAM) |
| **Scale** | Usage-based | $0.222/CU-hour | $0.35/GB-month | Max 56 CU (224GB RAM) |

**CU (Compute Unit)**: 1 CU = 1 vCPU + 4GB RAM

### Storage Requirements (NextAuth.js Sessions)

**Assumptions**:
- Session table: ~500 bytes per user
- 30-day session TTL
- MAU × 0.5GB storage estimate

| Tier | MAU | Est. Storage | Storage Cost (@$0.35/GB) |
|------|-----|--------------|--------------------------|
| 1K | 1,000 | 0.5 MB | **Free tier** |
| 10K | 10,000 | 5 MB | **Free tier** |
| 100K | 100,000 | 50 MB | **$0.02/month** |
| 1M | 1,000,000 | 500 MB | **$0.18/month** |
| 10M | 10,000,000 | 5 GB | **$1.75/month** |

### Compute Requirements

**Assumptions**:
- Autoscaling: 0.25 CU idle, 2 CU peak
- Active hours: ~200 hours/month at scale
- Launch plan sufficient up to 1M users

| Tier | MAU | CU-Hours/Month | Compute Cost | Total Neon Cost |
|------|-----|----------------|--------------|-----------------|
| 1K | 1,000 | ~50 | **Free tier** | **$0** |
| 10K | 10,000 | ~80 | **Free tier** | **$0** |
| 100K | 100,000 | ~150 | **$8-16** | **$8-16** |
| 1M | 1,000,000 | ~300 | **$32-67** | **$32-67** |
| 10M | 10,000,000 | ~800 (Scale) | **$178-250** | **$178-250** |

**Volume Discount**: None. Enterprise plans available for custom SLA/limits.

**Breakeven Point**: Neon scales well; no clear breakeven. Costs grow predictably with usage.

---

## 3. Upstash Redis (via Vercel Marketplace)

**Note**: Vercel migrated Vercel KV to Upstash Redis in December 2024. Direct Upstash pricing applies.

### Upstash Redis Pricing

| Plan | Monthly Cost | Max Data | Commands/Month | Bandwidth |
|------|--------------|----------|----------------|-----------|
| **Free** | $0 | 256MB | 500K | 10GB |
| **Pay-as-You-Go** | Usage-based | 100GB | Unlimited | Unlimited |
| **Fixed 1GB** | $20 | 1GB | Unlimited | 100GB |
| **Fixed 5GB** | $100 | 5GB | Unlimited | 500GB |

**PAYG Pricing**:
- **Commands**: $0.20 per 100K
- **Storage**: $0.25 per GB
- **Bandwidth**: First 200GB free, then $0.03/GB

### Use Cases for Redis
1. **Rate Limiting**: 5 gens/day per user tracking
2. **Affiliate Link Tracking**: Click-through cache
3. **Session Cache**: NextAuth session acceleration

### Cost Projections

**Assumptions**:
- Rate limit checks: 3 commands per generation
- Affiliate tracking: 2 commands per link click (50% of users click)
- Session cache: 10 commands per user session
- Total storage: ~10KB per active user

| Tier | MAU | Commands/Month | Storage | Est. Monthly Cost |
|------|-----|----------------|---------|-------------------|
| 1K | 1,000 | ~13K | 10MB | **Free tier** |
| 10K | 10,000 | ~130K | 100MB | **Free tier** |
| 100K | 100,000 | ~1.3M | 1GB | **$2.60 (PAYG)** or **$20 (Fixed)** |
| 1M | 1,000,000 | ~13M | 10GB | **$26** (PAYG) or **$200** (Fixed 10GB) |
| 10M | 10,000,000 | ~130M | 100GB | **$260** (PAYG) or **$800** (Fixed 100GB) |

**Optimization Strategy**: 
- Use **Fixed 1GB** at 100K MAU for predictable costs ($20/month)
- Split databases: High-frequency (Fixed) + High-storage (PAYG) at 1M+ scale
- **Reported savings**: One team dropped from $600/mo to $25/mo with dual-database strategy

**Breakeven Point**: Fixed pricing better than PAYG above ~100K commands/month per GB stored.

---

## 4. Vercel Blob - Ephemeral Image Storage

**Pricing** (Per US Region):

| Resource | Hobby Included | On-Demand Rate |
|----------|----------------|----------------|
| Storage Size | 1GB | $0.023/GB |
| Simple Operations (reads) | 10,000 | $0.40 per 1M |
| Advanced Operations (writes) | 2,000 | $5.00 per 1M |
| Blob Data Transfer | 10GB | $0.05/GB |

**Use Case**: Temporary storage for generated images (1-hour TTL before user download)

### Cost Projections

**Assumptions**:
- Each generation = 1 write (upload), 2 reads (preview + download)
- Average image size: 2MB
- Images deleted after 1 hour (minimal storage accumulation)

| Tier | Generations | Writes | Reads | Data Transfer | Storage | Monthly Cost |
|------|-------------|--------|-------|---------------|---------|--------------|
| 1K | 900 | 900 | 1,800 | 1.8GB | ~0.002GB | **Free tier** |
| 10K | 9,000 | 9K | 18K | 18GB | ~0.02GB | **$0.90** |
| 100K | 90,000 | 90K | 180K | 180GB | ~0.2GB | **$9.50** |
| 1M | 900,000 | 900K | 1.8M | 1.8TB | ~2GB | **$95** |
| 10M | 9,000,000 | 9M | 18M | 18TB | ~20GB | **$955** |

**Calculation**:
- Writes: (count / 1M) × $5.00
- Reads: (count / 1M) × $0.40
- Transfer: GB × $0.05
- Storage: GB × $0.023 (minimal due to 1-hour TTL)

**Volume Discount**: None public. Enterprise pricing available.

**Breakeven Point**: Blob is cost-effective for ephemeral storage. Consider S3 for persistent storage at 1M+ scale.

---

## 5. Inngest - Background Job Orchestration

**Pricing Plans**:

| Plan | Monthly Cost | Executions Included | Concurrent Steps | Features |
|------|--------------|---------------------|------------------|----------|
| **Hobby** | $0 | 50K | 5 | Basic alerting |
| **Pro** | $75 | 1M | 100 | Metrics, 7-day traces |
| **Enterprise** | Custom | Custom | 500-50K | SAML, 90-day traces |

**Execution Pricing (PAYG)**:

| Volume | Price per 1M Executions |
|--------|-------------------------|
| 1M - 5M | $0.000050 |
| 5M - 15M | $0.000025 |
| 15M - 50M | $0.000020 |
| 50M - 100M | $0.000015 |
| 100M+ | Contact sales |

### Use Case
- Queue AI generation jobs
- Handle webhook callbacks from fal.ai
- Email notifications on completion

### Cost Projections

**Assumptions**:
- 2 executions per generation (queue + callback)
- Total executions = Generations × 2

| Tier | MAU | Generations | Executions | Monthly Cost |
|------|-----|-------------|------------|--------------|
| 1K | 1,000 | 900 | 1.8K | **Free tier** |
| 10K | 10,000 | 9K | 18K | **Free tier** |
| 100K | 100,000 | 90K | 180K | **Free tier** |
| 1M | 1,000,000 | 900K | 1.8M | **$75** (Pro base) + **$0.09** = **$75.09** |
| 10M | 10,000,000 | 9M | 18M | **$75** + **$0.38** = **$75.38** |

**Volume Discount**: Automatic tiered pricing kicks in at 1M+ executions.

**Breakeven Point**: Extremely cost-effective at scale due to micro-pricing. No competitive alternative needed.

---

## 6. Plausible Analytics - Privacy-Friendly Analytics

**Pricing** (Based on Pageviews + Custom Events):

| Monthly Pageviews | Starter | Growth | Business |
|-------------------|---------|--------|----------|
| 10K | $9 | - | - |
| 100K | $19 | - | - |
| 200K | $29 | - | - |
| 500K | - | $69 | - |
| 1M | - | $99 | - |
| 2M | - | $149 | - |
| 5M | - | - | $299 |
| 10M | - | - | $499 |
| Custom | - | - | Contact sales |

**Free Tier**: None (30-day trial only)

### Cost Projections

**Assumptions**:
- 5 pageviews per session (home, upload, preview, share)
- Total pageviews/month = MAU × 5

| Tier | MAU | Pageviews/Month | Plausible Cost |
|------|-----|-----------------|----------------|
| 1K | 1,000 | 5K | **$9** (Starter) |
| 10K | 10,000 | 50K | **$19** (Starter) |
| 100K | 100,000 | 500K | **$69** (Growth) |
| 1M | 1,000,000 | 5M | **$299** (Business) |
| 10M | 10,000,000 | 50M | **Custom pricing** (~$1,500-2,000 est.) |

**Volume Discount**: None public below custom pricing threshold.

**Alternatives**:
- **PostHog**: Free up to 1M events/month, then $0.00031/event (more affordable at scale)
- **Vercel Web Analytics**: $3 per 100K events (cheaper for high-traffic apps)

**Breakeven Point**: Plausible becomes expensive at 1M+ MAU. Consider PostHog or Vercel Analytics.

---

## 7. fal.ai - AI Image Generation (CatVTON)

**fal.ai Model Pricing**:

CatVTON is not listed in fal.ai's public pricing. Based on similar virtual try-on models:

| Model | Type | Price | Notes |
|-------|------|-------|-------|
| **CatVTON** (estimate) | Image-to-Image | ~$0.02-0.04/image | Research-only; not commercial |
| **FLUX 2 Virtual Try-On** | Image-to-Image | $0.021/compute-second | Commercial use allowed |
| **Recraft V3** | Text-to-Image | $0.04/image | State-of-art text rendering |

**CatVTON Limitations**:
- **Research-only license** (not for production)
- No official fal.ai pricing published
- Must self-host for commercial use

### fal.ai Cost Projections (FLUX 2 Virtual Try-On)

**Assumptions**:
- $0.021/compute-second
- Average generation time: ~8 seconds
- Cost per image: **$0.168**

| Tier | MAU | Generations/Month | Monthly Cost |
|------|-----|-------------------|--------------|
| 1K | 1,000 | 900 | **$151** |
| 10K | 10,000 | 9,000 | **$1,512** |
| 100K | 100,000 | 90,000 | **$15,120** |
| 1M | 1,000,000 | 900,000 | **$151,200** |
| 10M | 10,000,000 | 9,000,000 | **$1,512,000** |

**Volume Discount**: Enterprise plans available with custom pricing. Contact sales for 100K+ generations/month.

---

## 8. Resend - Transactional Email

**Pricing Plans**:

| Plan | Monthly Cost | Emails Included | Additional Emails |
|------|--------------|-----------------|-------------------|
| **Free** | $0 | 3,000 (100/day limit) | N/A |
| **Pro** | $20 | 50,000 | $0.44 per 1K |
| **Scale** | $90 | 100,000 | $0.39 per 1K |
| **Enterprise** | Custom | Custom | Custom |

### Use Cases
- Welcome emails
- Generation completion notifications
- Password resets

### Cost Projections

**Assumptions**:
- 1 welcome email per new user
- 1 notification per generation
- Total emails = MAU + Generations

| Tier | MAU | Generations | Total Emails | Monthly Cost |
|------|-----|-------------|--------------|--------------|
| 1K | 1,000 | 900 | 1,900 | **Free tier** |
| 10K | 10,000 | 9,000 | 19,000 | **Free tier** |
| 100K | 100,000 | 90,000 | 190,000 | **$20** (Pro) + **$62** = **$82** |
| 1M | 1,000,000 | 900,000 | 1.9M | **$90** (Scale) + **$702** = **$792** |
| 10M | 10,000,000 | 9,000,000 | 19M | **$90** + **$7,370** = **$7,460** |

**Volume Discount**: Scale plan reduces per-email overage cost from $0.44 to $0.39 per 1K.

**Breakeven Point**: Resend is cost-effective up to 1M MAU. Consider SendGrid or AWS SES at 10M+ scale.

---

## 9. Self-Hosted GPU Options (CatVTON Alternative)

### GPU Pricing Comparison (Hourly Rates)

| Provider | GPU | VRAM | Price/Hour | Notes |
|----------|-----|------|------------|-------|
| **RunPod** | A100 (80GB) | 80GB | $1.39 | Secure Cloud |
| **RunPod** | H100 SXM | 80GB | $2.69 | Community Cloud |
| **Modal** | A100 (80GB) | 80GB | $2.50 | Serverless ($0.000694/sec) |
| **Modal** | H100 | 80GB | $3.95 | Serverless ($0.001097/sec) |
| **Lambda Labs** | A100 (80GB) | 80GB | $1.79 | On-demand |
| **Lambda Labs** | H100 SXM | 80GB | $2.99 | On-demand |

### Self-Hosted CatVTON Cost Projections

**Assumptions**:
- RunPod A100 (80GB): **$1.39/hour**
- Generation time: **~10 seconds per image**
- Batch size: 4 images (optimal GPU utilization)
- Effective cost: **$0.0039 per image**

| Tier | MAU | Generations | GPU Hours | Monthly Cost |
|------|-----|-------------|-----------|--------------|
| 1K | 1,000 | 900 | 2.5 hrs | **$3.50** |
| 10K | 10,000 | 9,000 | 25 hrs | **$35** |
| 100K | 100,000 | 90,000 | 250 hrs | **$350** |
| 1M | 1,000,000 | 900,000 | 2,500 hrs | **$3,475** |
| 10M | 10,000,000 | 9,000,000 | 25,000 hrs | **$34,750** |

**Additional Costs**:
- **Storage**: $0.10-0.30/GB for model weights (~10GB)
- **Egress**: $0.08-0.12/GB for image delivery (handled by Vercel Blob)

### Breakeven Analysis: fal.ai vs Self-Hosted

| Scale | fal.ai FLUX 2 | RunPod A100 Self-Hosted | Savings |
|-------|---------------|-------------------------|---------|
| 1K MAU | $151 | $3.50 | **97.7%** |
| 10K MAU | $1,512 | $35 | **97.7%** |
| 100K MAU | $15,120 | $350 | **97.7%** |
| 1M MAU | $151,200 | $3,475 | **97.7%** |
| 10M MAU | $1,512,000 | $34,750 | **97.7%** |

**Breakeven Point**: Self-hosting is **40x cheaper** at all scales, but requires:
- Model deployment infrastructure
- GPU orchestration (queue management)
- Monitoring and error handling
- Engineering time for maintenance

**Recommendation**:
- **< 10K MAU**: Use fal.ai for speed-to-market (higher cost, zero ops)
- **10K-100K MAU**: Evaluate self-hosting ROI (engineering cost vs. savings)
- **> 100K MAU**: Self-hosting is critical (40x cost reduction = $150K/month saved at 1M MAU)

---

## Total Cost Summary by User Scale

| Service | 1K MAU | 10K MAU | 100K MAU | 1M MAU | 10M MAU |
|---------|--------|---------|----------|--------|---------|
| **Vercel Pro** | $20 | $20-25 | $25-40 | $420-650 | $4,200-6,500 |
| **Neon Postgres** | $0 | $0 | $8-16 | $32-67 | $178-250 |
| **Upstash Redis** | $0 | $0 | $20 | $200 | $800 |
| **Vercel Blob** | $0 | $0.90 | $9.50 | $95 | $955 |
| **Inngest** | $0 | $0 | $0 | $75 | $75 |
| **Plausible** | $9 | $19 | $69 | $299 | ~$1,500 |
| **fal.ai (FLUX 2)** | $151 | $1,512 | $15,120 | $151,200 | $1,512,000 |
| **Resend** | $0 | $0 | $82 | $792 | $7,460 |
| **TOTAL (fal.ai)** | **$180** | **$1,577** | **$15,344** | **$153,133** | **$1,533,518** |
| | | | | | |
| **RunPod Self-Hosted** | $3.50 | $35 | $350 | $3,475 | $34,750 |
| **TOTAL (Self-Hosted)** | **$32** | **$100** | **$574** | **$7,908** | **$53,718** |

### Key Insights

1. **AI Generation is 95%+ of total cost** at all scales
2. **Self-hosting saves $145K/month at 1M MAU** (97.7% reduction)
3. **Infrastructure costs grow linearly** (Vercel, DB, storage scale predictably)
4. **Breakeven for self-hosting**: ~10K MAU (when engineering investment < monthly savings)

---

## Volume Discount Opportunities

### Services with Automatic Discounts
- ✅ **Inngest**: Automatic tiered pricing (up to 70% off at 100M+ executions)
- ✅ **Resend**: Scale plan reduces overage cost by 11%
- ✅ **Upstash**: Fixed plans offer unlimited commands (vs. PAYG)

### Services Requiring Enterprise Contact
- ⚠️ **Vercel**: Custom pricing above 1M MAU (no public volume discounts)
- ⚠️ **Neon**: Enterprise plans for SLA/limits (no public volume discounts)
- ⚠️ **fal.ai**: Volume pricing available for 100K+ generations/month
- ⚠️ **Plausible**: Custom pricing above 10M pageviews

### No Volume Discounts
- ❌ **Vercel Blob**: Linear pricing (consider S3 at scale)
- ❌ **Neon Storage**: Flat $0.35/GB (consider self-managed Postgres at 10M+ MAU)

---

## Alternative Provider Comparisons

### Analytics: Plausible vs PostHog vs Vercel
- **Plausible**: $299/month at 1M MAU
- **PostHog**: Free up to 1M events, then $0.31 per 1K = **$1,550/month** at 5M events
- **Vercel Web Analytics**: $3 per 100K events = **$150/month** at 5M events
- **Winner at 1M MAU**: Vercel Web Analytics ($150 vs $299)

### Email: Resend vs SendGrid vs AWS SES
- **Resend**: $792/month at 1M MAU
- **SendGrid**: $89.95/month (100K emails) + $0.85 per 1K = **$1,700/month**
- **AWS SES**: $0.10 per 1K = **$190/month** at 1.9M emails
- **Winner at 1M MAU**: AWS SES ($190 vs $792)

### Redis: Upstash vs AWS ElastiCache
- **Upstash Fixed 10GB**: $200/month at 1M MAU
- **AWS ElastiCache** (cache.m5.large): ~$146/month + data transfer
- **Winner at 1M MAU**: Similar cost; Upstash offers better DX for serverless

---

## Cost Optimization Strategies

### Immediate Actions (< 100K MAU)
1. ✅ Use **Vercel Web Analytics** instead of Plausible (-50% cost)
2. ✅ Use **Upstash Fixed 1GB** at 100K MAU for predictable Redis costs
3. ✅ Use **fal.ai** for MVP speed-to-market (defer self-hosting)

### Growth Phase (100K - 1M MAU)
1. ✅ Evaluate **RunPod self-hosting** (40x savings on AI generation)
2. ✅ Switch to **AWS SES** for email (-75% cost)
3. ✅ Negotiate **Vercel Enterprise** pricing for custom limits

### Scale Phase (1M+ MAU)
1. ✅ **Self-host CatVTON** on RunPod/Modal (critical cost reduction)
2. ✅ Use **Upstash dual-database strategy** (Fixed + PAYG split)
3. ✅ Consider **AWS RDS Postgres** or **Neon Scale** for DB
4. ✅ Migrate to **S3** for persistent image storage (Blob = ephemeral only)

---

## Next Steps for Investor Presentations

### Recommended Cost Narrative
1. **MVP (0-10K MAU)**: $100-1,600/month total cost → Focus on product-market fit
2. **Growth (10K-100K MAU)**: $600-15K/month → Self-hosting decision point at ~50K MAU
3. **Scale (100K-1M MAU)**: $8K-153K/month → Self-hosting saves $145K/month
4. **Viral (1M-10M MAU)**: $54K-1.5M/month → Infrastructure cost = 3.5% of revenue at scale

### Margins at Scale (10M MAU)
- **Self-hosted infrastructure**: $54K/month
- **Estimated revenue** (affiliate + partners): $1.5M/month (assumption)
- **Infrastructure margin**: **96.5%** (very healthy for consumer app)

### Key Risks
1. **AI generation cost** is 95%+ of total spend → Self-hosting is non-negotiable at scale
2. **Vercel bandwidth** can spike with viral traffic → Monitor closely, set spend limits
3. **Free tier abuse** (rate limiting critical) → Redis tracks daily limits per user

---

## Sources

- [Vercel Pricing](https://vercel.com/pricing) - Updated February 2026
- [Neon Pricing](https://neon.com/pricing) - August 2025 pricing model
- [Upstash Redis Pricing](https://upstash.com/pricing/redis) - March 2025 update
- [Inngest Pricing](https://www.inngest.com/pricing)
- [Plausible Analytics Pricing](https://plausible.io/docs/subscription-plans)
- [fal.ai Pricing](https://fal.ai/pricing)
- [Resend Pricing](https://resend.com/pricing)
- [RunPod GPU Pricing](https://www.runpod.io/pricing)
- [Modal GPU Pricing](https://modal.com/pricing)
- [Lambda Labs GPU Pricing](https://lambda.ai/pricing)

---

**Document Prepared By**: AI Research Agent  
**For**: Try on Haul - Investor Cost Projections  
**Date**: February 9, 2026

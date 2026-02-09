# Background Jobs Research: Inngest & Alternatives

> **Purpose**: Research on background job processing solutions for serverless Next.js/Vercel deployments
> **Context**: Need queue system for AI image generation (60-90 second jobs) with retry logic and status tracking
> **Last Updated**: 2026-02-09

---

## Executive Summary

For serverless Next.js applications on Vercel, **Inngest** and **Trigger.dev** emerge as the top commercial solutions, with **BullMQ** and **pg-boss** as self-hosted alternatives requiring additional infrastructure.

**Key Finding**: Traditional queue systems (BullMQ, pg-boss) **cannot run on Vercel** due to serverless constraints. They require a separate always-on server for worker processes.

---

## Inngest

### What It Does

Inngest is an **event-driven durable execution platform** that provides:

- **Background job processing** without infrastructure setup
- **HTTP-based invocation** - calls your existing API routes/functions when events are received
- **Automatic retries** with configurable backoff strategies
- **Step functions** with durable execution (survives failures/timeouts)
- **Scheduling & cron jobs**
- **Observability** built-in with traces, logs, and metrics
- **Concurrency control** to prevent overwhelming downstream services

**Architecture**: 
- Your functions stay in your codebase (Next.js API routes, serverless functions)
- Inngest platform receives events via HTTP and calls your functions back
- No workers to deploy or queues to manage
- Works with Vercel, AWS Lambda, Cloudflare Workers, etc.

### Pricing

#### Hobby Plan (Free)
- **$0/month** - No credit card required
- **50,000 executions/month** included
- 5 concurrent steps
- 50 realtime connections
- 3 users
- Unlimited branch/staging environments
- 24-hour log retention
- Community support

#### Pro Plan
- **$75/month** starting price
- **1,000,000 executions/month** included
- Additional executions: **$50 per 1 million**
- 100+ concurrent steps (then $25 per 25)
- 1,000+ realtime connections
- 15+ users (then $10/user)
- 7-day log retention
- Dedicated Slack support available ($200)

#### Enterprise Plan
- **Custom pricing** (contact sales)
- Custom execution volumes
- 500-50k concurrent steps
- 90-day log retention
- SAML, RBAC, audit trails
- HIPAA compliance available
- SOC 2 certified

### Usage-Based Pricing Details

**Executions**: Each function run or step execution counts
- 50k free/month
- 50k-1m: $0.000050 per execution
- 1m-5m: $0.000050 per execution
- 5m-15m: $0.000025 per execution
- 15m-50m: $0.000020 per execution

**Events**: Charged for events received
- First 1-5m events/day are free
- Volume discounts as you scale

### Cost Estimate for Try on Haul
**Assumptions**: 
- 30,000 runs/month (5 generations/day limit × user base)
- 2 steps per run average (image processing + notification)
- Total: 90,000 executions/month

**Result**: **Hobby plan covers this for free** ($0/month)

### Pros
✅ Works natively on Vercel (no additional infrastructure)
✅ Generous free tier (50k executions)
✅ Simple integration (write functions in existing codebase)
✅ Built-in observability and debugging
✅ Automatic retries and error handling
✅ Open-source SDKs
✅ No timeout issues (durable execution)
✅ Active development and support

### Cons
❌ Vendor lock-in (functions call Inngest platform)
❌ Cold start potential for low-traffic apps
❌ Limited to HTTP-based triggers
❌ Log retention limited on free tier (24 hours)

---

## Trigger.dev

### What It Does

Trigger.dev is a **durable background jobs framework** that provides:

- **Long-running tasks** with no timeout limits
- **Checkpointing** - snapshots process state, resumes later (CRIU technology)
- **Task scheduling** (cron support)
- **Realtime** connections to frontend
- **AI-specific features** (agent skills, AI SDK integration)
- **Queue management** with concurrency controls

**Architecture**:
- Tasks written in `/trigger` folders in your codebase
- Code is bundled and deployed to Trigger.dev's managed workers
- Can also be self-hosted
- Works with Next.js, Remix, Bun, Node.js

### Pricing

#### Free Plan
- **$0/month**
- **$5 monthly usage included**
- 20 concurrent runs
- Unlimited tasks
- 5 team members
- Dev & Prod environments
- 10 schedules
- 1-day log retention
- 10 concurrent realtime connections

#### Hobby Plan
- **$10/month**
- **$10 monthly usage included**
- 50 concurrent runs
- 5 team members
- Dev, Staging & Prod environments
- 5 preview branches
- 100 schedules
- 7-day log retention
- 50 concurrent realtime connections

#### Pro Plan
- **$50/month**
- **$50 monthly usage included**
- 200+ concurrent runs ($10/month per 50 additional)
- 25+ team members ($20/month per seat)
- 20+ preview branches ($10/month per branch)
- 1,000+ schedules
- 30-day log retention
- Dedicated Slack support

#### Enterprise Plan
- Custom pricing
- All Pro features +
- Custom log retention
- Priority support
- Role-based access control
- SOC 2 report, SSO

### Usage-Based Pricing

**Compute Costs** (per second of execution):
| Machine | vCPU | RAM | Cost/sec |
|---------|------|-----|----------|
| Micro | 0.25 | 0.25GB | $0.0000169 |
| Small 1x (default) | 0.5 | 0.5GB | $0.0000338 |
| Small 2x | 1 | 1GB | $0.0000675 |
| Medium 1x | 1 | 2GB | $0.0000850 |
| Medium 2x | 2 | 4GB | $0.0001700 |
| Large 1x | 4 | 8GB | $0.0003400 |
| Large 2x | 8 | 16GB | $0.0006800 |

**Run Invocation Cost**: $0.000025 per run ($0.25 per 10,000 runs)

### Cost Estimate for Try on Haul
**Assumptions**:
- 30,000 runs/month
- 75 seconds execution time per job (AI image generation)
- Small 1x machine (0.5 vCPU, 0.5GB RAM)

**Calculation**:
- 1 run compute: 75s × $0.0000338 = $0.002535
- 1 run invocation: $0.000025
- Total per run: $0.00256
- Monthly cost: 30,000 × $0.00256 = **$76.80/month**

**Note**: Free tier ($5 credit) covers ~1,950 runs/month

### Pros
✅ No timeout limits (can run for hours/days)
✅ Checkpointing means you don't pay for wait time
✅ Can self-host (open source Apache 2.0)
✅ AI-focused features (agents, AI SDK integration)
✅ Realtime frontend connections
✅ Active development (Series A funded)
✅ GitHub stars: 13.6k (very popular)

### Cons
❌ Higher cost at scale ($76/month vs free for Inngest at our volume)
❌ Code deployed to their infrastructure (not in your app)
❌ More complex setup than Inngest
❌ Compute costs can be unpredictable for long jobs

---

## BullMQ

### What It Does

BullMQ is an **open-source Redis-based job queue** for Node.js:

- **Redis-backed** message queue
- **Advanced features**: retries, progress tracking, rate limiting, priorities
- **Multiple language support**: Node.js, Python, Elixir, PHP
- **Concurrency control**
- **Job scheduling** with cron
- **Dead letter queues**

**Architecture**:
- Requires Redis instance (memory store)
- Requires always-on worker processes
- Producer adds jobs to queue, consumer processes them

### Pricing

**Open source** - Free to use (MIT-like license)

**Infrastructure Costs**:
- **Redis hosting**: $10-50/month (Upstash, Redis Cloud, AWS ElastiCache)
- **Worker server**: $5-20/month (Railway, Render, DigitalOcean)
- **Total**: ~$15-70/month depending on scale

### Vercel Compatibility

❌ **Cannot run on Vercel** as primary deployment:
- BullMQ workers need **long-running processes**
- Vercel is serverless (functions terminate after request)
- Workers would restart on every request, losing state

**Workaround Required**:
- Deploy Next.js app to Vercel
- Deploy separate worker service to Railway/Render/Fly.io
- Share Redis instance between both
- More complex architecture

### Cost Estimate for Try on Haul

**Infrastructure**:
- Redis (Upstash free tier): $0
- Worker server (Railway): $5/month
- **Total**: ~$5/month

**BUT**: Requires managing separate infrastructure

### Pros
✅ Open source and free
✅ Battle-tested (used by thousands)
✅ Very feature-rich
✅ Full control over infrastructure
✅ No vendor lock-in
✅ Redis is fast and reliable

### Cons
❌ Requires Redis + worker infrastructure
❌ Cannot run on Vercel alone
❌ More operational complexity
❌ Need to manage scaling, monitoring, alerts
❌ Cold start issues if worker goes down
❌ Security: Redis and workers need proper networking

---

## pg-boss

### What It Does

**pg-boss** is a **PostgreSQL-based job queue** for Node.js:

- **Postgres-native** - uses SKIP LOCKED for queue management
- **No additional infrastructure** (if you already have Postgres)
- **Reliable** - uses ACID guarantees of Postgres
- **Features**: retries, scheduling, priorities, dead letter queues
- **Simple API** - easy to integrate

**Architecture**:
- Jobs stored in Postgres tables
- Workers poll database for jobs
- Uses Postgres transactions for exactly-once delivery

### Pricing

**Open source** - Free to use (MIT license)

**Infrastructure Costs**:
- **Postgres database**: May already have one
  - Vercel Postgres: $0 (Hobby), $20/month (Pro)
  - Supabase: Free tier available
  - Neon: Generous free tier
- **Worker server**: $5-20/month (Railway, Render)
- **Total**: ~$0-40/month depending on existing setup

### Vercel Compatibility

❌ **Cannot run on Vercel** as primary deployment:
- Same issue as BullMQ - needs always-on worker process
- Workers must continuously poll database
- Serverless functions terminate after each request

**Workaround Required**:
- Next.js app on Vercel
- Separate worker service elsewhere
- Shared Postgres database

**Potential Hybrid Approach**:
- Use Postgres database you already have (Vercel Postgres, Supabase)
- Deploy only the worker process to Railway/Render
- Lighter footprint than Redis + Workers

### Cost Estimate for Try on Haul

**Infrastructure**:
- Postgres (Vercel/Supabase free tier): $0
- Worker server (Railway): $5/month
- **Total**: ~$5/month

### Pros
✅ Open source and free
✅ Uses Postgres (might already have it)
✅ No Redis required
✅ Reliable (ACID guarantees)
✅ Simple API
✅ Good for low-to-medium scale

### Cons
❌ Cannot run on Vercel alone
❌ Needs separate worker infrastructure
❌ Postgres polling can be less efficient than Redis pub/sub
❌ Not as feature-rich as BullMQ
❌ Less popular (3.1k GitHub stars vs 28k for BullMQ)
❌ Performance degrades at very high scale

---

## Temporal

### What It Does

**Temporal** is an **enterprise-grade workflow orchestration engine**:

- **Durable workflows** - stateful, long-running processes
- **Automatic retries** and failure recovery
- **Complex workflows** - loops, conditionals, timeouts, child workflows
- **Language support**: Go, Java, Python, TypeScript, PHP, .NET
- **Versioning** - update workflow code without breaking running instances

**Use Cases**:
- Multi-step business processes (order fulfillment, onboarding)
- Saga patterns and distributed transactions
- AI agent orchestration
- Data pipelines
- Complex ETL workflows

**Architecture**:
- Temporal server (orchestration engine)
- Workflow code runs in your workers
- Very powerful but complex

### Pricing

#### Temporal Cloud (Managed Service)
- **$200/month** minimum (starter tier)
- **Enterprise pricing** for production use
- Includes hosting, monitoring, support

#### Self-Hosted (Open Source)
- **Free** - MIT license
- **Infrastructure costs**:
  - Temporal server: Requires Cassandra/MySQL/PostgreSQL + ElasticSearch
  - Complex setup: 5-10 services minimum
  - Worker infrastructure: $20-100+/month
  - **Total**: ~$50-200/month for small-medium scale

### Vercel Compatibility

⚠️ **Partial compatibility**:
- Can deploy workers to Vercel functions
- But Temporal Server must run elsewhere (cloud or self-hosted)
- Complex setup not ideal for simple job queues

### Cost Estimate for Try on Haul

**Cloud**: $200+/month (overkill for this use case)
**Self-hosted**: $50-200/month + significant operational complexity

### Pros
✅ Enterprise-grade reliability
✅ Extremely powerful workflow capabilities
✅ Great for complex orchestration
✅ Language agnostic
✅ Battle-tested at scale (Uber, Netflix, Stripe use it)
✅ Strong versioning and rollback

### Cons
❌ Massive overkill for simple job queues
❌ Very expensive (cloud) or complex (self-hosted)
❌ Steep learning curve
❌ Heavy infrastructure requirements
❌ Not optimized for serverless environments
❌ Minimum $200/month cloud pricing

**Verdict**: ❌ **Not recommended** for Try on Haul - way too complex and expensive

---

## Quirrel

### What It Does

Quirrel was a **serverless job scheduling solution** designed specifically for Next.js and Vercel:

- HTTP-based job queueing
- Cron scheduling
- Delay and retry mechanisms
- Simple API
- Self-hostable

### Current Status

🚫 **DISCONTINUED** - Acquired by Netlify in 2022

- Service shut down **August 1, 2022**
- No longer accepting new customers
- Creator recommends:
  - Netlify Scheduled Functions (if on Netlify)
  - Self-host the open-source version
  - Use alternatives like Inngest or Trigger.dev

**GitHub Status**: 928 stars, but last meaningful update was 2022

### Why It Failed

- Solo developer project
- Couldn't compete with better-funded alternatives
- Netlify acquired it for the talent/technology, then shut it down

**Verdict**: ❌ **Do not use** - project is dead

---

## Other Alternatives (Brief Overview)

### Dispatched
- New commercial service (launched Dec 2024)
- Pure HTTP background jobs for serverless
- Early stage, unproven
- Pricing unknown
- **Verdict**: Too new, skip for now

### Upstash QStash
- HTTP-based job queueing
- Built on Upstash infrastructure
- Generous free tier
- Good Vercel integration
- **Pricing**: $0.5 per 1,000 requests after free tier
- **Verdict**: Worth considering as Inngest alternative

### Zeplo
- HTTP task scheduling
- Simple webhook-based jobs
- Less feature-rich than Inngest
- **Verdict**: Niche alternative

### Graphile Worker
- Postgres-based (like pg-boss)
- More feature-rich
- Requires worker infrastructure
- **Verdict**: Similar to pg-boss but more complex

---

## Comparison Table

| Feature | Inngest | Trigger.dev | BullMQ | pg-boss | Temporal |
|---------|---------|-------------|--------|---------|----------|
| **Works on Vercel** | ✅ Yes | ✅ Yes | ❌ No | ❌ No | ⚠️ Partial |
| **Cost (our scale)** | Free | ~$77/mo | ~$5/mo* | ~$5/mo* | $200+/mo |
| **Open Source** | SDK only | ✅ Full | ✅ Yes | ✅ Yes | ✅ Yes |
| **Self-Hostable** | ❌ No | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes |
| **Setup Complexity** | 🟢 Easy | 🟡 Medium | 🔴 Hard | 🔴 Hard | 🔴 Very Hard |
| **No Infrastructure** | ✅ Yes | ✅ Yes | ❌ No | ❌ No | ❌ No |
| **Max Duration** | Unlimited | Unlimited | Unlimited* | Unlimited* | Unlimited |
| **Retry Logic** | ✅ Built-in | ✅ Built-in | ✅ Built-in | ✅ Built-in | ✅ Built-in |
| **Observability** | ✅ Excellent | ✅ Good | ⚠️ DIY | ⚠️ DIY | ✅ Excellent |
| **Vercel Integration** | ✅ Native | ✅ Native | ❌ No | ❌ No | ⚠️ Complex |
| **Concurrency Control** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes |
| **Queue Status UI** | ✅ Yes | ✅ Yes | ❌ No** | ❌ No | ✅ Yes |
| **GitHub Stars** | 4.8k | 13.6k | 28k | 3.1k | 13k |
| **Maturity** | 🟢 Production | 🟢 Production | 🟢 Battle-tested | 🟡 Mature | 🟢 Enterprise |

\* Requires separate worker server infrastructure  
\*\* Can use BullBoard (separate UI)

---

## Recommendations

### For Try on Haul Specifically

#### 🥇 **Primary Recommendation: Inngest**

**Why**:
- ✅ **Free** at our scale (50k executions covers us)
- ✅ **Zero infrastructure** to manage
- ✅ **Works natively** on Vercel
- ✅ **Simple integration** - write functions in existing codebase
- ✅ **Great observability** built-in
- ✅ **Reliable** - used by SoundCloud, Resend, Cohere
- ✅ **Easy to scale** - no architecture changes needed

**When it fits**:
- You're deploying to Vercel
- You want minimal complexity
- You don't need self-hosting
- Your job volume fits free tier (50k/month)

#### 🥈 **Alternative: Trigger.dev**

**Why**:
- ✅ Can **self-host** (future-proofing)
- ✅ **No timeout limits**
- ✅ **Checkpointing** (don't pay for wait time)
- ✅ **AI-focused features**

**Trade-offs**:
- ❌ More expensive (~$77/month at our scale)
- ❌ More complex than Inngest

**When to choose it**:
- You need self-hosting capability
- You want AI agent features
- Cost difference isn't a concern
- You want open-source full platform

#### 🥉 **Budget Option: pg-boss (Self-Hosted)**

**Why**:
- ✅ **Very cheap** (~$5/month)
- ✅ **Open source** (MIT)
- ✅ Uses Postgres (might already have it)
- ✅ **Full control**

**Trade-offs**:
- ❌ Requires separate worker server (Railway, Render)
- ❌ More operational complexity
- ❌ Not Vercel-native
- ❌ Manual monitoring/alerting setup

**When to choose it**:
- Budget is extremely tight
- You already have DevOps experience
- You need full control/self-hosting
- You don't mind managing infrastructure

---

## Decision Matrix

```
┌─────────────────────────────────────────────────────────────┐
│                   DECISION TREE                             │
└─────────────────────────────────────────────────────────────┘

Do you NEED self-hosting?
├─ YES → Can you manage infrastructure?
│   ├─ YES → pg-boss ($5/mo) or Trigger.dev self-hosted
│   └─ NO → Trigger.dev Cloud ($77/mo)
│
└─ NO → Inngest (FREE) ✅ RECOMMENDED
```

---

## Implementation Considerations

### For Inngest Integration

1. **Install SDK**: `npm install inngest`
2. **Create functions** in existing API routes
3. **Serve endpoint**: Add `/api/inngest` route
4. **Send events** to trigger jobs
5. **Configure in Inngest dashboard**

**Estimated setup time**: 1-2 hours

### For Trigger.dev Integration

1. **Install SDK**: `npm install @trigger.dev/sdk`
2. **Create `/trigger` directory**
3. **Write task definitions**
4. **Deploy tasks** with CLI
5. **Trigger from your code**

**Estimated setup time**: 2-4 hours

### For pg-boss Integration

1. **Setup Postgres** (if not already)
2. **Deploy worker service** (Railway/Render)
3. **Install pg-boss**: `npm install pg-boss`
4. **Configure connection** (env vars)
5. **Write job handlers**
6. **Setup monitoring** (custom)

**Estimated setup time**: 4-8 hours

---

## Key Findings Summary

1. **Serverless Constraint**: BullMQ and pg-boss **cannot run on Vercel** without separate infrastructure
2. **Cost Winner**: Inngest is **free** at our scale (50k executions/month)
3. **Self-Hosting**: Only Trigger.dev and traditional queues (BullMQ, pg-boss) support self-hosting
4. **Complexity**: Inngest is **simplest** to integrate and operate
5. **Quirrel is Dead**: Don't waste time investigating it
6. **Temporal is Overkill**: Great for complex workflows, but too expensive/complex for job queues

---

## Related Documentation

- [TECH-STACK.md](../architecture/TECH-STACK.md) - Technology decisions
- [MVP-FEATURES.md](../user-stories/MVP-FEATURES.md) - Feature requirements
- [IMPLEMENTATION-PLAN.md](../plans/IMPLEMENTATION-PLAN.md) - Development plan

---

## Sources

- Inngest official docs: https://www.inngest.com/docs
- Trigger.dev official docs: https://trigger.dev/docs
- BullMQ official docs: https://docs.bullmq.io
- pg-boss GitHub: https://github.com/timgit/pg-boss
- Temporal official docs: https://docs.temporal.io
- Quirrel status: https://docs.quirrel.dev/netlify-acquisition-faq
- Community discussions: HN, Reddit, GitHub

---

**Last Updated**: 2026-02-09  
**Researcher**: AI Agent  
**Review Status**: Pending review by dev team

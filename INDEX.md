# Try on Haul - Documentation Index

> **Last Updated**: 2026-02-09  
> **Status**: Documentation & Design Phase  
> **MVP Target**: 8-12 weeks

## Quick Navigation

| Document | Purpose | Status |
|----------|---------|--------|
| [AGENTS.md](./AGENTS.md) | Agent guidance, project rules, standards | Active |
| [Architecture Overview](./docs/architecture/OVERVIEW.md) | System architecture and design | Active |
| [Tech Stack](./docs/architecture/TECH-STACK.md) | Technology decisions and rationale | Active |
| [Provider Adapter](./docs/architecture/PROVIDER-ADAPTER.md) | AI provider abstraction layer | Active |
| [Sharing Strategy](./docs/architecture/SHARING-STRATEGY.md) | Social sharing implementation | Active |
| [Data Flow](./docs/architecture/DATA-FLOW.md) | Data flow, integrations, APIs | Active |
| [MVP Features](./docs/user-stories/MVP-FEATURES.md) | User stories for MVP | Active |
| [Requirements](./docs/user-stories/REQUIREMENTS.md) | Detailed functional requirements | Active |
| [AI Models Research](./docs/research/AI-MODELS.md) | Virtual try-on AI model analysis | Active |
| [Prompt Engineering](./docs/research/PROMPT-ENGINEERING.md) | Prompting strategies for try-on | Active |
| [Implementation Plan](./docs/plans/IMPLEMENTATION-PLAN.md) | Phased development roadmap | Active |

## Standards & Guidelines

| Document | Purpose |
|----------|---------|
| [Development Standards](./docs/standards/DEV-STANDARDS.md) | Code style, patterns, practices |
| [Design Standards](./docs/standards/DESIGN-STANDARDS.md) | UI/UX guidelines, component library |
| [Documentation Standards](./docs/standards/DOC-STANDARDS.md) | How to write and maintain docs |

---

## Project Overview

**Try on Haul** is a virtual try-on application that allows users to:

1. Upload a reference photo of themselves
2. Provide an outfit image (upload, URL, or from partner Shopify stores)
3. Optionally select a festival scene background
4. Generate an AI-powered image of themselves wearing the outfit
5. Share the result directly to social media platforms

### Key Differentiators

- **Festival-focused**: Curated scenes for Burning Man, Coachella, EDC, Tomorrowland aesthetics
- **Social-first**: Built for viral sharing with trackable affiliate links
- **Privacy-conscious**: Minimal server storage, user consent-first design
- **Retailer integration**: Affiliate partnerships with festival fashion brands

---

## Target Users

| Segment | Description | Priority |
|---------|-------------|----------|
| Festival-goers | People shopping for festival outfits | Primary |
| Fashion Retailers | Partner brands driving traffic via affiliates | Primary |
| Influencers | Content creators showcasing looks | Secondary |

---

## Technical Summary

| Aspect | Decision |
|--------|----------|
| Platform | Web (PWA) - mobile-first, native-like experience |
| Hosting | Serverless (Vercel) |
| Auth | Social-only (Facebook, Instagram, TikTok, Apple) |
| AI Models | Open-source virtual try-on (see [AI Models](./docs/research/AI-MODELS.md)) |
| Storage | Ephemeral only - no persistent image storage (1h TTL) |
| Monetization | Affiliate links + branded share cards |

---

## Document Relationships

```
INDEX.md (this file)
    |
    +-- AGENTS.md (agent rules, references all standards)
    |
    +-- docs/architecture/
    |       +-- OVERVIEW.md (high-level architecture)
    |       +-- TECH-STACK.md (specific technologies)
|       +-- PROVIDER-ADAPTER.md (AI provider abstraction)
|       +-- SHARING-STRATEGY.md (social sharing implementation)
|       +-- DATA-FLOW.md (integrations, APIs)
    |
    +-- docs/user-stories/
    |       +-- MVP-FEATURES.md (user stories)
    |       +-- REQUIREMENTS.md (detailed requirements)
    |
    +-- docs/research/
    |       +-- AI-MODELS.md (model comparison)
    |       +-- PROMPT-ENGINEERING.md (prompting strategies)
    |
    +-- docs/standards/
    |       +-- DEV-STANDARDS.md
    |       +-- DESIGN-STANDARDS.md
    |       +-- DOC-STANDARDS.md
    |
    +-- docs/plans/
            +-- IMPLEMENTATION-PLAN.md (phased roadmap)
```

---

## How to Use This Documentation

1. **Starting a new task**: Read [AGENTS.md](./AGENTS.md) for project rules and standards
2. **Understanding the system**: Start with [Architecture Overview](./docs/architecture/OVERVIEW.md)
3. **Implementing features**: Reference [MVP Features](./docs/user-stories/MVP-FEATURES.md) for requirements
4. **AI integration work**: See [AI Models](./docs/research/AI-MODELS.md), [Provider Adapter](./docs/architecture/PROVIDER-ADAPTER.md), and [Prompt Engineering](./docs/research/PROMPT-ENGINEERING.md)
5. **Social sharing work**: See [Sharing Strategy](./docs/architecture/SHARING-STRATEGY.md)
6. **Writing code**: Follow [Development Standards](./docs/standards/DEV-STANDARDS.md)

---

## Change Log

| Date | Change | Author |
|------|--------|--------|
| 2026-02-09 | Initial documentation structure created | AI Agent |
| 2026-02-09 | Added Provider Adapter architecture doc | AI Agent |
| 2026-02-09 | Added Sharing Strategy doc reference | AI Agent |

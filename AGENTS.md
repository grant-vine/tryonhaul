# Try on Haul - Agent Guidelines

> **Read this file before starting ANY work on this project.**  
> **See also**: [INDEX.md](./INDEX.md) for documentation navigation

---

## Project Overview

**Try on Haul** is a virtual try-on application for festival fashion. Users upload a photo of themselves, provide an outfit image, select a scene, and get AI-generated images of themselves wearing the outfit.

**Current Phase**: Documentation & Design (no code implementation yet)

---

## MANDATORY Rules for All Agents

### Rule 1: No Code Until Approved

This project is in **documentation and design phase**. Do NOT write implementation code unless explicitly instructed. Documentation, research, and planning only.

### Rule 2: Single Source of Truth

- **NEVER duplicate information** across documents
- Reference other documents using relative links: `[Text](./path/to/doc.md)`
- If information exists elsewhere, link to it; don't copy it
- Each piece of information should have ONE canonical location

### Rule 3: Read Before Writing

Before creating or modifying any document:
1. Read [INDEX.md](./INDEX.md) to understand document structure
2. Read the relevant standards document for the type of work
3. Check if the information already exists elsewhere

### Rule 4: Follow Standards

| Work Type | Standards Document |
|-----------|-------------------|
| Writing code | [DEV-STANDARDS.md](./docs/standards/DEV-STANDARDS.md) |
| UI/UX work | [DESIGN-STANDARDS.md](./docs/standards/DESIGN-STANDARDS.md) |
| Documentation | [DOC-STANDARDS.md](./docs/standards/DOC-STANDARDS.md) |

### Rule 5: Verify Before Claiming Complete

- Run linters/type checks on code changes
- Ensure links work in documentation
- Validate against requirements in [REQUIREMENTS.md](./docs/user-stories/REQUIREMENTS.md)

---

## Development Standards Summary

> **Full details**: [DEV-STANDARDS.md](./docs/standards/DEV-STANDARDS.md)

### Code Style

- **Language**: TypeScript (strict mode, no `any`)
- **Framework**: See [TECH-STACK.md](./docs/architecture/TECH-STACK.md) for decisions
- **Formatting**: Prettier with project config
- **Linting**: ESLint with strict rules
- **No type escapes**: Never use `as any`, `@ts-ignore`, `@ts-expect-error`

### Architecture Principles

1. **Serverless-first**: Design for Vercel/edge functions
2. **Privacy-first**: Minimize data collection and storage
3. **Mobile-first**: PWA with responsive design
4. **Accessibility**: WCAG 2.1 Level A minimum
5. **i18n-ready**: Structure for future localization (English first)

### Testing Requirements

- Unit tests for business logic
- Integration tests for API routes
- E2E tests for critical user flows
- Test coverage targets defined per feature

### Security Requirements

- All user inputs validated and sanitized
- Social auth via established providers only
- No PII stored unless absolutely necessary
- CCPA compliance for data handling
- Image consent verification required

---

## Design Standards Summary

> **Full details**: [DESIGN-STANDARDS.md](./docs/standards/DESIGN-STANDARDS.md)

### Visual Design

- **Theme**: Festival/rave aesthetic - vibrant, energetic, modern
- **Color Palette**: TBD (document when finalized)
- **Typography**: TBD (document when finalized)
- **Dark Mode**: Consider for MVP (users often browse at night)

### Component Approach

- Reusable component library
- Consistent spacing/sizing system
- Mobile-first responsive breakpoints
- Accessible by default (focus states, contrast, ARIA)

### User Experience

- Maximum 3 steps to core action (generate try-on)
- Clear loading states with progress indicators
- Graceful error handling with actionable messages
- Social sharing should be seamless (1-2 taps)

---

## Documentation Standards Summary

> **Full details**: [DOC-STANDARDS.md](./docs/standards/DOC-STANDARDS.md)

### File Naming

- Use UPPERCASE-KEBAB-CASE for documentation files: `MVP-FEATURES.md`
- Use lowercase-kebab-case for code files: `user-service.ts`

### Document Structure

Every documentation file must have:
1. **Title** (H1) - Clear, descriptive
2. **Purpose statement** - What this doc covers
3. **Cross-references** - Links to related docs
4. **Last updated date** - For staleness detection

### Writing Style

- Be concise and direct
- Use tables for structured data
- Use code blocks for examples
- Avoid jargon; explain technical terms

### Keeping Docs Current

- Update docs when making related code changes
- Mark outdated sections with `> **TODO**: [what needs updating]`
- Delete docs that are no longer relevant

---

## Key Technical Decisions

> **Full details**: [TECH-STACK.md](./docs/architecture/TECH-STACK.md)

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Platform | PWA (Web) | Social sharing, auth, no app store friction |
| Hosting | Vercel (Serverless) | Low ops, auto-scaling, edge functions |
| Auth | Social-only | Marketing gate requirement, viral potential |
| Image Storage | Client-side only | Privacy-first, cost reduction |
| AI Models | Open-source virtual try-on | Free tier MVP requirement |

---

## Key User Flows

> **Full details**: [MVP-FEATURES.md](./docs/user-stories/MVP-FEATURES.md)

### Core Flow (Happy Path)

1. User lands on marketing page
2. User authenticates via social login (Facebook/Instagram/TikTok/Apple)
3. User accepts image consent modal (one-time)
4. User uploads reference photo (self)
5. User provides outfit image (upload/URL/Shopify partner)
6. User optionally selects festival scene
7. System queues generation job
8. User receives notification when complete
9. User previews result
10. User shares to social media with affiliate link

### Key Constraints

- **5 generations per day** per user
- **No server-side image storage** (generated images delivered directly to user)
- **Branded share cards** with tracking links

---

## Project Structure (Planned)

```
tryonhaul/
├── INDEX.md                    # This file - central navigation
├── AGENTS.md                   # Agent guidelines (you are here)
├── docs/
│   ├── architecture/
│   │   ├── OVERVIEW.md         # System architecture
│   │   ├── TECH-STACK.md       # Technology decisions
│   │   └── DATA-FLOW.md        # Data flow & integrations
│   ├── user-stories/
│   │   ├── MVP-FEATURES.md     # User stories
│   │   └── REQUIREMENTS.md     # Detailed requirements
│   ├── research/
│   │   ├── AI-MODELS.md        # AI model comparison
│   │   └── PROMPT-ENGINEERING.md
│   ├── standards/
│   │   ├── DEV-STANDARDS.md
│   │   ├── DESIGN-STANDARDS.md
│   │   └── DOC-STANDARDS.md
│   └── plans/
│       └── IMPLEMENTATION-PLAN.md
└── src/                        # (future) Application code
```

---

## When to Escalate

### Consult Oracle (Architecture/Debugging)

- Multi-system integration decisions
- Performance optimization strategies
- Complex debugging after 2+ failed attempts
- Security architecture questions

### Consult Metis (Pre-Planning)

- Ambiguous requirements needing clarification
- Scope analysis before major features
- Risk identification

### Consult Momus (Review)

- Before finalizing major documentation
- Plan reviews for gaps and ambiguities
- Quality assurance checkpoints

---

## Monetization Strategy

### Primary: Affiliate Links

- All outfit links flow through our URL shortener
- Track click-throughs and conversions
- Revenue share with partner retailers

### Secondary: Branded Sharing

- Generated images include subtle branding
- Share text includes "#TryOnHaul" or tracking hashtag
- Links in shares drive new user acquisition

### Tertiary: Partner Widget (Post-MVP)

- Embeddable try-on widget for retailer sites
- White-label option for premium partners

---

## Privacy & Compliance

### User Data Handling

- Collect minimum necessary data
- Social auth provides identity; we don't store passwords
- Reference photos processed on-device or via API; not stored server-side
- Generated images delivered to user; not retained

### Consent Requirements

- Modal consent for image usage (one-time, saved to profile)
- Clear terms of service
- Easy data deletion (right to be forgotten)
- CCPA compliant data practices

### Image Rights

- User must confirm uploaded image is of themselves
- No third-party face uploads permitted
- Clear terms on AI-generated image ownership

---

## Change Log

| Date | Change | Author |
|------|--------|--------|
| 2026-02-09 | Initial AGENTS.md created | AI Agent |

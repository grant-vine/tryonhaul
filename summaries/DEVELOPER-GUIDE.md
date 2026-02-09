# Try on Haul - Developer Guide

> **Audience**: Engineers, contractors, open-source contributors  
> **Last Updated**: 2026-02-09

---

## Welcome! 👋

This guide will help you get up to speed on the Try on Haul codebase. Whether you're joining the team or contributing to the project, start here.

---

## Project Overview

**Try on Haul** is a virtual try-on PWA for festival fashion. Users upload a selfie, provide an outfit, and AI generates an image of them wearing it.

### Core Technologies

| Layer | Technology |
|-------|------------|
| Frontend | Next.js 16 (App Router, React 19) |
| Styling | Tailwind CSS |
| Auth | Clerk (social OAuth) |
| Background Jobs | Inngest |
| AI Generation | fal.ai (CatVTON) + FASHN (backup) |
| Storage | Vercel Blob (ephemeral), Vercel KV |
| Hosting | Vercel |
| Analytics | Plausible |

---

## Getting Started

### Prerequisites

- Node.js 20+
- pnpm (preferred) or npm
- Git
- Vercel account (for local Blob/KV emulation)
- Clerk account (for auth testing)

### Initial Setup

```bash
# Clone the repository
git clone https://github.com/grant-vine/tryonhaul.git
cd tryonhaul

# Install dependencies
pnpm install

# Copy environment variables
cp .env.example .env.local

# Start development server
pnpm dev
```

### Environment Variables

```bash
# .env.local

# Clerk (Auth)
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_xxx
CLERK_SECRET_KEY=sk_test_xxx

# Vercel Storage
BLOB_READ_WRITE_TOKEN=xxx
KV_REST_API_URL=xxx
KV_REST_API_TOKEN=xxx

# AI Providers
FAL_KEY=xxx
FASHN_API_KEY=xxx

# Inngest
INNGEST_EVENT_KEY=xxx
INNGEST_SIGNING_KEY=xxx

# Analytics (optional for dev)
NEXT_PUBLIC_PLAUSIBLE_DOMAIN=tryonhaul.com
```

---

## Project Structure

```
tryonhaul/
├── app/                    # Next.js App Router pages
│   ├── (auth)/            # Auth-protected routes
│   ├── (marketing)/       # Public marketing pages
│   ├── api/               # API routes
│   └── layout.tsx         # Root layout
├── components/            # React components
│   ├── ui/               # Base UI components (buttons, inputs)
│   ├── features/         # Feature-specific components
│   └── layouts/          # Layout components
├── lib/                   # Shared utilities
│   ├── ai/               # AI provider adapters
│   ├── auth/             # Clerk helpers
│   ├── storage/          # Blob/KV helpers
│   └── utils/            # General utilities
├── inngest/              # Background job functions
├── public/               # Static assets
├── styles/               # Global styles
├── docs/                 # Documentation
└── summaries/            # Audience-specific summaries
```

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENT (Next.js PWA)                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐ │
│  │ Upload      │  │ Generation  │  │ Share                   │ │
│  │ Component   │──│ Component   │──│ Component               │ │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘ │
│         │                │                    │                 │
└─────────┼────────────────┼────────────────────┼─────────────────┘
          │                │                    │
          ▼                ▼                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                         API ROUTES                              │
├─────────────────────────────────────────────────────────────────┤
│  /api/upload    /api/generate    /api/share    /api/r/[id]     │
└─────────────────────────────────────────────────────────────────┘
          │                │                    
          ▼                ▼                    
┌──────────────┐  ┌──────────────┐  ┌─────────────────────────────┐
│ Vercel Blob  │  │   Inngest    │  │   AI Provider Adapter       │
│ (ephemeral)  │  │ (background) │  │  ┌─────┐ ┌──────┐ ┌──────┐ │
└──────────────┘  └──────┬───────┘  │  │fal.ai│ │FASHN │ │Self  │ │
                         │          │  └─────┘ └──────┘ └──────┘ │
                         ▼          └─────────────────────────────┘
                  ┌──────────────┐
                  │ AI Generation│
                  │   Pipeline   │
                  └──────────────┘
```

---

## Key Concepts

### 1. AI Provider Adapter

We use an adapter pattern for AI providers. This allows:
- Swapping providers without code changes
- A/B testing between models
- Automatic failover

```typescript
// lib/ai/provider-adapter.ts
interface TryOnProvider {
  generate(input: TryOnInput): Promise<TryOnResult>;
  estimateCost(): number;
  getName(): string;
}

// Usage
const provider = getProvider(); // Returns fal.ai, FASHN, or self-hosted
const result = await provider.generate({ personImage, garmentImage, scene });
```

### 2. Ephemeral Storage

User images are stored in Vercel Blob with 1-hour TTL. **No persistent storage.**

```typescript
// Upload with TTL
const blob = await put(filename, file, {
  access: 'public',
  addRandomSuffix: true,
  // Blob is auto-deleted after 1 hour
});
```

### 3. Background Jobs (Inngest)

Generation is async. We use Inngest for:
- Reliable execution (retries)
- Progress tracking
- Webhook callbacks

```typescript
// inngest/functions/generate.ts
export const generateTryOn = inngest.createFunction(
  { id: 'generate-try-on' },
  { event: 'tryon/generate.requested' },
  async ({ event, step }) => {
    // Step 1: Validate images
    // Step 2: Call AI provider
    // Step 3: Post-process result
    // Step 4: Notify user
  }
);
```

### 4. Rate Limiting

Users get 5 generations per day. Tracked in Vercel KV.

```typescript
// lib/rate-limit.ts
const key = `ratelimit:${userId}:${today}`;
const count = await kv.incr(key);
if (count === 1) await kv.expire(key, 86400); // 24h TTL
if (count > 5) throw new RateLimitError();
```

---

## Development Workflow

### Branch Strategy

```
main          # Production
├── develop   # Staging / next release
├── feature/* # New features
├── fix/*     # Bug fixes
└── chore/*   # Maintenance
```

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add scene selection component
fix: correct rate limit timezone handling
docs: update API documentation
chore: upgrade dependencies
```

### Pull Request Process

1. Create feature branch from `develop`
2. Make changes, write tests
3. Run `pnpm lint && pnpm test`
4. Open PR with description
5. Request review
6. Squash and merge after approval

---

## Code Standards

### TypeScript

- **Strict mode**: No `any`, no `@ts-ignore`
- **Explicit types**: Function parameters and returns
- **Prefer interfaces**: Over type aliases for objects

### React

- **Functional components**: Always
- **Server components**: Default, client only when needed
- **Hooks**: Custom hooks for reusable logic

### Styling

- **Tailwind CSS**: Primary styling method
- **CSS Variables**: For design tokens
- **No inline styles**: Use Tailwind classes

### Testing

- **Unit tests**: Vitest for utils and hooks
- **Component tests**: React Testing Library
- **E2E tests**: Playwright for critical flows

```bash
# Run tests
pnpm test          # Unit + component tests
pnpm test:e2e      # E2E tests
pnpm test:coverage # Coverage report
```

---

## Common Tasks

### Adding a New API Route

```typescript
// app/api/example/route.ts
import { auth } from '@clerk/nextjs';
import { NextResponse } from 'next/server';

export async function POST(request: Request) {
  const { userId } = auth();
  if (!userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }
  
  // Your logic here
  
  return NextResponse.json({ success: true });
}
```

### Adding a New Component

```typescript
// components/ui/my-component.tsx
interface MyComponentProps {
  title: string;
  onClick?: () => void;
}

export function MyComponent({ title, onClick }: MyComponentProps) {
  return (
    <button 
      onClick={onClick}
      className="px-4 py-2 bg-purple-500 rounded-lg"
    >
      {title}
    </button>
  );
}
```

### Adding a New Inngest Function

```typescript
// inngest/functions/my-function.ts
import { inngest } from '../client';

export const myFunction = inngest.createFunction(
  { id: 'my-function', retries: 3 },
  { event: 'my-domain/my-event' },
  async ({ event, step }) => {
    const result = await step.run('step-name', async () => {
      // Your logic
    });
    return result;
  }
);
```

---

## Debugging

### Local Development

```bash
# View logs
pnpm dev  # Logs appear in terminal

# Inngest dashboard (local)
npx inngest-cli dev
```

### Production Issues

- **Vercel Logs**: Dashboard → Functions → Logs
- **Inngest Dashboard**: Dashboard → Functions → Runs
- **Sentry**: Error tracking (if configured)

---

## Documentation

| Document | Purpose |
|----------|---------|
| [AGENTS.md](../AGENTS.md) | AI agent guidelines |
| [INDEX.md](../INDEX.md) | Documentation index |
| [TECH-STACK.md](../docs/architecture/TECH-STACK.md) | Technology decisions |
| [REQUIREMENTS.md](../docs/user-stories/REQUIREMENTS.md) | Feature requirements |
| [IMPLEMENTATION-PLAN.md](../docs/plans/IMPLEMENTATION-PLAN.md) | Development roadmap |

---

## Getting Help

- **Architecture questions**: Check docs or ask in #engineering
- **AI provider issues**: See [AI-MODELS.md](../docs/research/AI-MODELS.md)
- **Auth issues**: Clerk docs + internal wiki
- **Stuck?**: Ask in Slack, open a GitHub issue

---

## Contributing

We welcome contributions! Please:

1. Read this guide fully
2. Check existing issues before creating new ones
3. Follow code standards
4. Write tests for new features
5. Update documentation as needed

**Thank you for helping build Try on Haul!** 🎪✨

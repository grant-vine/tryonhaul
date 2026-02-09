# Development Standards

> **Last Updated**: 2026-02-09  
> **Related**: [AGENTS.md](../../AGENTS.md) | [TECH-STACK.md](../architecture/TECH-STACK.md)

---

## Purpose

This document defines coding standards, patterns, and practices for Try on Haul development.

---

## Language & Runtime

### TypeScript Configuration

**Strict mode required**:

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true
  }
}
```

### Forbidden Patterns

| Pattern | Why | Alternative |
|---------|-----|-------------|
| `as any` | Bypasses type safety | Proper typing or generics |
| `@ts-ignore` | Hides errors | Fix the underlying issue |
| `@ts-expect-error` | Silences errors | Fix the underlying issue |
| `any` type | No type checking | `unknown` + type guards |
| `!` non-null assertion | Unsafe | Proper null checks |

---

## Code Style

### Formatting

**Prettier configuration** (`.prettierrc`):

```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 80,
  "arrowParens": "avoid"
}
```

### Linting

**ESLint configuration** (`eslint.config.js`):

```javascript
export default [
  {
    extends: [
      'next/core-web-vitals',
      'plugin:@typescript-eslint/recommended-type-checked',
    ],
    rules: {
      '@typescript-eslint/no-explicit-any': 'error',
      '@typescript-eslint/explicit-function-return-type': 'warn',
      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
      'no-console': ['warn', { allow: ['warn', 'error'] }],
    },
  },
];
```

### Naming Conventions

| Entity | Convention | Example |
|--------|------------|---------|
| Files (code) | kebab-case | `user-service.ts` |
| Files (components) | PascalCase | `ImageUploader.tsx` |
| Variables | camelCase | `userData` |
| Constants | SCREAMING_SNAKE | `MAX_FILE_SIZE` |
| Functions | camelCase | `generateTryOn` |
| Classes | PascalCase | `ImageProcessor` |
| Types/Interfaces | PascalCase | `UserProfile` |
| React Components | PascalCase | `ShareButton` |
| API Routes | kebab-case | `/api/generate-try-on` |

---

## Project Structure

```
tryonhaul/
├── app/                    # Next.js App Router
│   ├── (auth)/            # Auth-required routes (grouped)
│   ├── (marketing)/       # Public marketing pages
│   ├── api/               # API routes
│   ├── layout.tsx         # Root layout
│   └── page.tsx           # Home page
├── components/            # React components
│   ├── ui/               # Base UI components
│   └── features/         # Feature-specific components
├── lib/                  # Utility libraries
│   ├── api/             # API client functions
│   ├── hooks/           # Custom React hooks
│   └── utils/           # Pure utility functions
├── types/               # TypeScript type definitions
├── styles/              # Global styles
├── public/              # Static assets
└── tests/               # Test files
```

### File Organization Rules

1. **One component per file** (unless tightly coupled)
2. **Co-locate tests** with source files (`Component.test.tsx`)
3. **Co-locate types** in same file or adjacent `.types.ts`
4. **Barrel exports** only at feature level, not globally

---

## React Patterns

### Component Structure

```typescript
// components/features/ImageUploader.tsx

import { useState } from 'react';
import { uploadImage } from '@/lib/api/upload';
import type { UploadResult } from '@/types/upload';

interface ImageUploaderProps {
  onUpload: (result: UploadResult) => void;
  maxSizeMB?: number;
}

export function ImageUploader({
  onUpload,
  maxSizeMB = 10,
}: ImageUploaderProps): JSX.Element {
  const [isUploading, setIsUploading] = useState(false);

  const handleUpload = async (file: File): Promise<void> => {
    setIsUploading(true);
    try {
      const result = await uploadImage(file);
      onUpload(result);
    } finally {
      setIsUploading(false);
    }
  };

  return (
    // JSX
  );
}
```

### Hooks Pattern

```typescript
// lib/hooks/useGeneration.ts

import { useState, useCallback } from 'react';
import { generateTryOn } from '@/lib/api/generate';

export function useGeneration() {
  const [status, setStatus] = useState<'idle' | 'loading' | 'success' | 'error'>('idle');
  const [result, setResult] = useState<GenerationResult | null>(null);
  const [error, setError] = useState<Error | null>(null);

  const generate = useCallback(async (input: GenerationInput) => {
    setStatus('loading');
    setError(null);
    
    try {
      const result = await generateTryOn(input);
      setResult(result);
      setStatus('success');
      return result;
    } catch (err) {
      setError(err instanceof Error ? err : new Error('Unknown error'));
      setStatus('error');
      throw err;
    }
  }, []);

  return { generate, status, result, error };
}
```

### Server Components vs Client Components

| Use Server Component | Use Client Component |
|---------------------|---------------------|
| Data fetching | User interactions |
| Access backend resources | Browser APIs (camera, etc.) |
| Sensitive logic | State management |
| Large dependencies | Event handlers |

Mark client components with `'use client'` directive.

---

## API Patterns

### Route Handler Structure

```typescript
// app/api/generate/route.ts

import { NextRequest, NextResponse } from 'next/server';
import { auth } from '@clerk/nextjs';
import { z } from 'zod';
import { generateTryOn } from '@/lib/services/generation';
import { checkRateLimit } from '@/lib/services/rate-limit';

const GenerateSchema = z.object({
  referenceImageUrl: z.string().url(),
  outfitImageUrl: z.string().url(),
  sceneId: z.string().optional(),
});

export async function POST(request: NextRequest): Promise<NextResponse> {
  // 1. Authentication
  const { userId } = auth();
  if (!userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  // 2. Rate limiting
  const rateLimitResult = await checkRateLimit(userId);
  if (!rateLimitResult.allowed) {
    return NextResponse.json(
      { error: 'Rate limit exceeded', resetAt: rateLimitResult.resetAt },
      { status: 429 }
    );
  }

  // 3. Input validation
  const body = await request.json();
  const parsed = GenerateSchema.safeParse(body);
  if (!parsed.success) {
    return NextResponse.json(
      { error: 'Invalid input', details: parsed.error.issues },
      { status: 400 }
    );
  }

  // 4. Business logic
  try {
    const result = await generateTryOn(userId, parsed.data);
    return NextResponse.json(result);
  } catch (error) {
    console.error('Generation failed:', error);
    return NextResponse.json(
      { error: 'Generation failed' },
      { status: 500 }
    );
  }
}
```

### Error Response Format

```typescript
interface ErrorResponse {
  error: string;          // User-friendly message
  code?: string;          // Machine-readable code
  details?: unknown;      // Additional context
  resetAt?: string;       // For rate limits
}
```

---

## Error Handling

### Try-Catch Pattern

```typescript
// Always handle errors explicitly
async function fetchData(): Promise<Data> {
  try {
    const response = await fetch('/api/data');
    
    if (!response.ok) {
      throw new ApiError(response.status, await response.text());
    }
    
    return await response.json();
  } catch (error) {
    // Log for debugging
    console.error('Failed to fetch data:', error);
    
    // Re-throw with context
    throw new Error(
      `Data fetch failed: ${error instanceof Error ? error.message : 'Unknown error'}`
    );
  }
}
```

### Custom Error Classes

```typescript
// lib/errors.ts

export class AppError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly statusCode: number = 500
  ) {
    super(message);
    this.name = 'AppError';
  }
}

export class ValidationError extends AppError {
  constructor(message: string) {
    super(message, 'VALIDATION_ERROR', 400);
  }
}

export class RateLimitError extends AppError {
  constructor(public readonly resetAt: Date) {
    super('Rate limit exceeded', 'RATE_LIMIT', 429);
  }
}
```

---

## Testing Standards

### Testing Strategy

| Level | Coverage Target | Tools |
|-------|----------------|-------|
| Unit | 70% for business logic | Vitest |
| Integration | Critical API routes | Vitest + MSW |
| E2E | Core user flows | Playwright |

### Unit Test Example

```typescript
// lib/utils/image.test.ts

import { describe, it, expect } from 'vitest';
import { validateImageDimensions } from './image';

describe('validateImageDimensions', () => {
  it('accepts images meeting minimum requirements', () => {
    const result = validateImageDimensions(800, 1024);
    expect(result.valid).toBe(true);
  });

  it('rejects images below minimum width', () => {
    const result = validateImageDimensions(500, 1024);
    expect(result.valid).toBe(false);
    expect(result.error).toContain('width');
  });

  it('rejects images below minimum height', () => {
    const result = validateImageDimensions(768, 500);
    expect(result.valid).toBe(false);
    expect(result.error).toContain('height');
  });
});
```

### E2E Test Example

```typescript
// tests/e2e/generation.spec.ts

import { test, expect } from '@playwright/test';

test.describe('Generation Flow', () => {
  test('completes try-on generation', async ({ page }) => {
    // Login
    await page.goto('/');
    await page.click('[data-testid="login-facebook"]');
    // ... OAuth flow

    // Upload reference
    await page.setInputFiles('[data-testid="reference-upload"]', 'fixtures/person.jpg');
    await expect(page.locator('[data-testid="reference-preview"]')).toBeVisible();

    // Upload outfit
    await page.setInputFiles('[data-testid="outfit-upload"]', 'fixtures/outfit.jpg');
    await expect(page.locator('[data-testid="outfit-preview"]')).toBeVisible();

    // Generate
    await page.click('[data-testid="generate-button"]');
    await expect(page.locator('[data-testid="result-image"]')).toBeVisible({ timeout: 60000 });
  });
});
```

---

## Performance Guidelines

### Image Optimization

```typescript
// Always use next/image for images
import Image from 'next/image';

<Image
  src={imageUrl}
  alt={description}
  width={768}
  height={1024}
  priority={isAboveFold}
  quality={85}
/>
```

### Bundle Optimization

1. **Dynamic imports** for heavy components:
   ```typescript
   const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
     loading: () => <Skeleton />,
   });
   ```

2. **Tree shaking** - import only what you need:
   ```typescript
   // Good
   import { motion } from 'framer-motion';
   
   // Bad
   import * as framerMotion from 'framer-motion';
   ```

### Caching Strategy

```typescript
// API routes with caching
export const runtime = 'edge';
export const revalidate = 3600; // 1 hour

// Dynamic data (no cache)
export const dynamic = 'force-dynamic';
```

---

## Security Practices

### Input Validation

Always validate with Zod:

```typescript
import { z } from 'zod';

const UserInputSchema = z.object({
  imageUrl: z.string().url().max(2048),
  caption: z.string().max(280).optional(),
});

// Validate before use
const input = UserInputSchema.parse(rawInput);
```

### Environment Variables

```typescript
// lib/env.ts

import { z } from 'zod';

const envSchema = z.object({
  CLERK_SECRET_KEY: z.string().min(1),
  FAL_API_KEY: z.string().min(1),
  DATABASE_URL: z.string().url(),
});

export const env = envSchema.parse(process.env);
```

### Secrets Management

- Never commit secrets to git
- Use `.env.local` for development
- Use Vercel environment variables for production
- Rotate secrets regularly

---

## Git Workflow

### Branch Naming

```
feature/user-story-id-short-description
bugfix/issue-id-short-description
hotfix/critical-issue-description
```

### Commit Messages

Follow Conventional Commits:

```
type(scope): description

feat(auth): add TikTok OAuth provider
fix(upload): handle HEIC image format
docs(readme): update setup instructions
refactor(api): extract validation logic
test(generation): add E2E tests
```

### Pull Request Template

```markdown
## Summary
Brief description of changes

## Changes
- Change 1
- Change 2

## Testing
- [ ] Unit tests pass
- [ ] E2E tests pass
- [ ] Manual testing completed

## Screenshots
(if UI changes)
```

---

## Change Log

| Date | Change | Author |
|------|--------|--------|
| 2026-02-09 | Initial development standards | AI Agent |

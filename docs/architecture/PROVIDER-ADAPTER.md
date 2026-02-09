# Virtual Try-On Provider Adapter Architecture

> **Last Updated**: 2026-02-09  
> **Related**: [AI-MODELS.md](../research/AI-MODELS.md) | [TECH-STACK.md](./TECH-STACK.md) | [DATA-FLOW.md](./DATA-FLOW.md)

---

## Purpose

This document defines the adapter architecture for virtual try-on AI providers, enabling seamless swapping between providers, cost tracking, A/B testing, and failover capabilities. The design follows the Strategy pattern with a provider-agnostic interface.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                        │
│              (Next.js API Routes/Inngest Jobs)              │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                  TryOnService (Orchestrator)                │
│  - Provider selection logic                                 │
│  - Failover/fallback handling                              │
│  - A/B testing routing                                      │
│  - Cost tracking aggregation                                │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              ITryOnProvider (Interface)                     │
│  - generate()                                               │
│  - getCapabilities()                                        │
│  - estimateCost()                                           │
│  - healthCheck()                                            │
└───────────┬──────────────┬──────────────┬───────────────────┘
            │              │              │
   ┌────────▼─────┐  ┌────▼─────┐  ┌────▼──────┐
   │ FalAiProvider│  │  FASHN   │  │ SelfHosted│
   │ (CatVTON,    │  │ Provider │  │ Provider  │
   │  IDM-VTON)   │  │          │  │ (Future)  │
   └──────────────┘  └──────────┘  └───────────┘
```

---

## 1. Core Interface: ITryOnProvider

### Interface Definition

```typescript
/**
 * Common interface for all virtual try-on providers.
 * All providers must implement this contract.
 */
interface ITryOnProvider {
  /**
   * Unique identifier for this provider
   */
  readonly name: ProviderName;

  /**
   * Generate a try-on image
   * @throws ProviderError on failure
   */
  generate(input: TryOnInput): Promise<TryOnResult>;

  /**
   * Get provider capabilities (supported features)
   */
  getCapabilities(): ProviderCapabilities;

  /**
   * Estimate cost for a single generation
   * @returns Cost in USD
   */
  estimateCost(input: TryOnInput): number;

  /**
   * Check if provider is healthy/available
   */
  healthCheck(): Promise<ProviderHealth>;

  /**
   * Normalize provider-specific errors to standard format
   */
  normalizeError(error: unknown): ProviderError;
}
```

---

## 2. Normalized Data Types

### Input Normalization

```typescript
/**
 * Normalized input format - provider-agnostic
 */
interface TryOnInput {
  /** Reference image of person (URL or base64) */
  humanImage: ImageInput;

  /** Garment/outfit image (URL or base64) */
  garmentImage: ImageInput;

  /** Garment category - normalized across providers */
  category: GarmentCategory;

  /** Optional: Garment mask (required by some providers) */
  garmentMask?: ImageInput;

  /** Optional: Scene/background image */
  sceneImage?: ImageInput;

  /** Optional: Text description of garment */
  description?: string;

  /** Quality/speed tradeoff preference */
  quality?: 'fast' | 'balanced' | 'high';

  /** Metadata for tracking */
  metadata?: {
    userId: string;
    requestId: string;
    timestamp: number;
    experimentGroup?: string; // For A/B testing
  };
}

type ImageInput = 
  | { type: 'url'; url: string }
  | { type: 'base64'; data: string; mimeType: string };

type GarmentCategory = 
  | 'upper_body'  // Shirts, tops, jackets
  | 'lower_body'  // Pants, shorts, skirts
  | 'dress'       // Full-body dresses
  | 'full_body';  // Complete outfit

/**
 * Provider-specific input mapping
 * Each provider adapter translates TryOnInput to its native format
 */
interface IProviderInputMapper {
  toProviderFormat(input: TryOnInput): unknown;
  requiresGarmentMask(category: GarmentCategory): boolean;
}
```

### Output Normalization

```typescript
/**
 * Normalized output format - consistent across providers
 */
interface TryOnResult {
  /** Generated image */
  image: {
    url: string;
    width: number;
    height: number;
    format: 'png' | 'jpeg' | 'webp';
  };

  /** Generation metadata */
  metadata: {
    provider: ProviderName;
    model: string;
    durationMs: number;
    cost: number; // Actual cost in USD
    requestId: string;
    timestamp: number;
  };

  /** Quality metrics (if available from provider) */
  quality?: {
    score?: number;
    confidence?: number;
  };

  /** Debug information (non-production) */
  debug?: {
    rawResponse?: unknown;
    logs?: string[];
  };
}

/**
 * Provider health status
 */
interface ProviderHealth {
  status: 'healthy' | 'degraded' | 'unhealthy';
  latencyMs?: number;
  message?: string;
  lastChecked: number;
}
```

---

## 3. Provider Capabilities Matrix

### Capability Definition

```typescript
interface ProviderCapabilities {
  /** Supported garment categories */
  categories: GarmentCategory[];

  /** Does provider support scene/background integration? */
  supportsScenes: boolean;

  /** Does provider require garment masks? */
  requiresGarmentMask: boolean;

  /** Can handle text descriptions? */
  supportsTextPrompt: boolean;

  /** Maximum supported resolution */
  maxResolution: { width: number; height: number };

  /** Typical generation time range */
  generationTimeRange: { min: number; max: number }; // in seconds

  /** Cost per generation (USD) */
  costPerGeneration: number;

  /** Commercial license included? */
  commercialLicense: boolean;

  /** Rate limits */
  rateLimits?: {
    requestsPerMinute?: number;
    requestsPerDay?: number;
  };
}
```

### Capability Matrix

| Feature | fal.ai (CatVTON) | fal.ai (IDM-VTON) | FASHN API | Self-Hosted |
|---------|------------------|-------------------|-----------|-------------|
| **Categories** | All | All | All | All |
| **Scene Support** | ❌ No | ❌ No | ✅ Yes | 🔄 TBD |
| **Requires Mask** | ❌ No | ⚠️ Yes (pants) | ❌ No | 🔄 TBD |
| **Text Prompt** | ❌ No | ✅ Yes (optional) | ✅ Yes | 🔄 TBD |
| **Max Resolution** | 1024×768 | 768×1024 | 1MP native | 🔄 TBD |
| **Gen Time (s)** | 11-35 | 17-70 | 7 | 🔄 TBD |
| **Cost/Gen (USD)** | ~$0.06 | ~$0.11 | $0.075 | Variable |
| **Commercial License** | ❌ No* | ❌ No* | ✅ Yes | ✅ Yes |
| **Quality** | ⭐⭐⭐⭐⭐ Shape | ⭐⭐⭐⭐⭐ Texture | ⭐⭐⭐⭐⭐ Both | 🔄 TBD |

\* Non-commercial license - requires agreement with authors for commercial use

---

## 4. Provider Implementations

### 4.1 FalAiProvider (Priority 1)

**Models**: CatVTON (primary), IDM-VTON (backup)

#### Configuration

```typescript
interface FalAiConfig {
  apiKey: string;
  defaultModel: 'cat-vton' | 'idm-vton';
  timeout?: number; // milliseconds
  retries?: number;
}

class FalAiProvider implements ITryOnProvider {
  readonly name = 'fal.ai';
  private config: FalAiConfig;
  private client: FalClient;

  constructor(config: FalAiConfig) {
    this.config = config;
    this.client = new FalClient(config.apiKey);
  }

  async generate(input: TryOnInput): Promise<TryOnResult> {
    const startTime = Date.now();
    const modelId = this.selectModel(input);
    
    try {
      // Translate to fal.ai format
      const falInput = this.toFalFormat(input, modelId);
      
      // Subscribe to job (handles polling)
      const result = await this.client.subscribe(modelId, {
        input: falInput,
        logs: process.env.NODE_ENV !== 'production',
        onQueueUpdate: (update) => {
          // Track progress for user feedback
          this.emitProgress(update, input.metadata?.requestId);
        },
      });

      // Normalize output
      return this.fromFalFormat(result, {
        provider: 'fal.ai',
        model: modelId,
        durationMs: Date.now() - startTime,
        cost: this.estimateCost(input),
        requestId: input.metadata?.requestId || generateId(),
        timestamp: Date.now(),
      });

    } catch (error) {
      throw this.normalizeError(error);
    }
  }

  private selectModel(input: TryOnInput): string {
    // CatVTON for speed and shape accuracy
    if (input.quality === 'fast' || input.quality === 'balanced') {
      return 'fal-ai/cat-vton';
    }
    // IDM-VTON for texture detail
    if (input.quality === 'high') {
      return 'fal-ai/idm-vton';
    }
    return this.config.defaultModel === 'cat-vton' 
      ? 'fal-ai/cat-vton' 
      : 'fal-ai/idm-vton';
  }

  private toFalFormat(input: TryOnInput, model: string): unknown {
    const basePayload = {
      human_image_url: this.resolveImageUrl(input.humanImage),
      garment_image_url: this.resolveImageUrl(input.garmentImage),
    };

    if (model === 'fal-ai/cat-vton') {
      return {
        ...basePayload,
        cloth_type: this.mapCategory(input.category),
      };
    }

    if (model === 'fal-ai/idm-vton') {
      return {
        ...basePayload,
        category: this.mapCategoryIDM(input.category),
        description: input.description || '',
        // IDM-VTON may require mask for pants
        ...(input.garmentMask && {
          mask_image_url: this.resolveImageUrl(input.garmentMask),
        }),
      };
    }

    throw new Error(`Unknown model: ${model}`);
  }

  private mapCategory(category: GarmentCategory): string {
    const mapping: Record<GarmentCategory, string> = {
      upper_body: 'upper',
      lower_body: 'lower',
      dress: 'overall',
      full_body: 'overall',
    };
    return mapping[category];
  }

  getCapabilities(): ProviderCapabilities {
    const model = this.config.defaultModel;
    
    if (model === 'cat-vton') {
      return {
        categories: ['upper_body', 'lower_body', 'dress', 'full_body'],
        supportsScenes: false,
        requiresGarmentMask: false,
        supportsTextPrompt: false,
        maxResolution: { width: 1024, height: 768 },
        generationTimeRange: { min: 11, max: 35 },
        costPerGeneration: 0.06,
        commercialLicense: false, // Requires separate agreement
        rateLimits: undefined, // fal.ai handles rate limiting
      };
    }

    // IDM-VTON capabilities
    return {
      categories: ['upper_body', 'lower_body', 'dress'],
      supportsScenes: false,
      requiresGarmentMask: true, // For pants
      supportsTextPrompt: true,
      maxResolution: { width: 768, height: 1024 },
      generationTimeRange: { min: 17, max: 70 },
      costPerGeneration: 0.11,
      commercialLicense: false,
    };
  }

  estimateCost(input: TryOnInput): number {
    const model = this.selectModel(input);
    return model === 'fal-ai/cat-vton' ? 0.06 : 0.11;
  }

  async healthCheck(): Promise<ProviderHealth> {
    const start = Date.now();
    try {
      // Simple ping to fal.ai API
      await this.client.ping();
      return {
        status: 'healthy',
        latencyMs: Date.now() - start,
        lastChecked: Date.now(),
      };
    } catch (error) {
      return {
        status: 'unhealthy',
        message: error.message,
        lastChecked: Date.now(),
      };
    }
  }

  normalizeError(error: unknown): ProviderError {
    // Map fal.ai specific errors to standard format
    if (error instanceof FalApiError) {
      return {
        provider: 'fal.ai',
        code: this.mapErrorCode(error.statusCode),
        message: error.message,
        retryable: error.statusCode >= 500,
        originalError: error,
      };
    }

    return {
      provider: 'fal.ai',
      code: 'UNKNOWN_ERROR',
      message: 'An unknown error occurred',
      retryable: false,
      originalError: error,
    };
  }
}
```

#### API Specifics

**CatVTON Endpoint**: `fal-ai/cat-vton`

**Request Format**:
```json
{
  "human_image_url": "https://...",
  "garment_image_url": "https://...",
  "cloth_type": "upper" | "lower" | "overall"
}
```

**Response Format**:
```json
{
  "images": [{
    "url": "https://...",
    "width": 1024,
    "height": 768,
    "content_type": "image/png"
  }]
}
```

**IDM-VTON Endpoint**: `fal-ai/idm-vton`

**Request Format**:
```json
{
  "human_image_url": "https://...",
  "garment_image_url": "https://...",
  "category": "upper_body" | "lower_body" | "dresses",
  "description": "Short sleeve round neck t-shirt",
  "num_inference_steps": 30
}
```

---

### 4.2 FashnProvider (Priority 2)

**Model**: Proprietary FASHN model

#### Configuration

```typescript
interface FashnConfig {
  apiKey: string;
  baseUrl?: string; // Default: https://api.fashn.ai
  tier?: 'on-demand' | 'tier-1' | 'tier-2' | 'tier-3';
  timeout?: number;
}

class FashnProvider implements ITryOnProvider {
  readonly name = 'fashn';
  private config: FashnConfig;

  async generate(input: TryOnInput): Promise<TryOnResult> {
    const startTime = Date.now();

    try {
      // FASHN uses REST API with synchronous response
      const response = await fetch(`${this.baseUrl}/v1/tryon`, {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${this.config.apiKey}`,
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(this.toFashnFormat(input)),
      });

      if (!response.ok) {
        throw new FashnApiError(response.status, await response.text());
      }

      const result = await response.json();

      return {
        image: {
          url: result.output_url,
          width: result.width || 1024,
          height: result.height || 1024,
          format: 'png',
        },
        metadata: {
          provider: 'fashn',
          model: 'fashn-tryon-v1.6',
          durationMs: Date.now() - startTime,
          cost: this.estimateCost(input),
          requestId: input.metadata?.requestId || result.id,
          timestamp: Date.now(),
        },
        quality: {
          score: result.quality_score,
        },
      };

    } catch (error) {
      throw this.normalizeError(error);
    }
  }

  private toFashnFormat(input: TryOnInput): unknown {
    return {
      model_image: this.resolveImageUrl(input.humanImage),
      garment_image: this.resolveImageUrl(input.garmentImage),
      category: input.category,
      ...(input.description && { garment_description: input.description }),
      ...(input.sceneImage && { 
        background_image: this.resolveImageUrl(input.sceneImage) 
      }),
    };
  }

  getCapabilities(): ProviderCapabilities {
    return {
      categories: ['upper_body', 'lower_body', 'dress', 'full_body'],
      supportsScenes: true, // FASHN supports background integration
      requiresGarmentMask: false,
      supportsTextPrompt: true,
      maxResolution: { width: 1024, height: 1024 },
      generationTimeRange: { min: 5, max: 17 },
      costPerGeneration: this.getCostPerTier(),
      commercialLicense: true, // Included in all tiers
      rateLimits: this.getRateLimits(),
    };
  }

  private getCostPerTier(): number {
    const costs: Record<string, number> = {
      'on-demand': 0.075,
      'tier-1': 0.0675,
      'tier-2': 0.0600,
      'tier-3': 0.0488,
    };
    return costs[this.config.tier || 'on-demand'];
  }

  estimateCost(input: TryOnInput): number {
    return this.getCostPerTier();
  }

  async healthCheck(): Promise<ProviderHealth> {
    const start = Date.now();
    try {
      const response = await fetch(`${this.baseUrl}/v1/health`);
      return {
        status: response.ok ? 'healthy' : 'unhealthy',
        latencyMs: Date.now() - start,
        lastChecked: Date.now(),
      };
    } catch {
      return {
        status: 'unhealthy',
        lastChecked: Date.now(),
      };
    }
  }

  normalizeError(error: unknown): ProviderError {
    if (error instanceof FashnApiError) {
      return {
        provider: 'fashn',
        code: this.mapErrorCode(error.statusCode),
        message: error.message,
        retryable: error.statusCode >= 500 || error.statusCode === 429,
        originalError: error,
      };
    }

    return {
      provider: 'fashn',
      code: 'UNKNOWN_ERROR',
      message: 'An unknown error occurred',
      retryable: false,
      originalError: error,
    };
  }
}
```

#### API Specifics

**Endpoint**: `https://api.fashn.ai/v1/tryon` (or `tryon-v1.6`)

**Request Format**:
```json
{
  "model_image": "https://...",
  "garment_image": "https://...",
  "category": "upper_body",
  "garment_description": "Optional text description",
  "background_image": "https://..." // Optional scene
}
```

**Response Format**:
```json
{
  "id": "request_id",
  "output_url": "https://...",
  "width": 1024,
  "height": 1024,
  "quality_score": 0.95,
  "credits_used": 1
}
```

**Key Differences**:
- Faster response time (5-17s vs 11-70s)
- Native scene/background support
- Synchronous API (no polling required)
- Commercial license included

---

### 4.3 SelfHostedProvider (Future - Priority 3)

**Model**: CatVTON or IDM-VTON on self-hosted GPU

#### Configuration

```typescript
interface SelfHostedConfig {
  endpoint: string; // Custom inference server URL
  apiKey?: string;
  model: 'cat-vton' | 'idm-vton';
  gpuType?: string; // For cost estimation
  costPerSecond?: number; // Custom cost calculation
}

class SelfHostedProvider implements ITryOnProvider {
  readonly name = 'self-hosted';
  private config: SelfHostedConfig;

  async generate(input: TryOnInput): Promise<TryOnResult> {
    // Implementation depends on self-hosted inference server
    // Could use:
    // - ComfyUI API
    // - Custom FastAPI wrapper
    // - Modal/RunPod endpoints
    
    // This is a placeholder for future implementation
    throw new Error('Self-hosted provider not yet implemented');
  }

  getCapabilities(): ProviderCapabilities {
    // Capabilities depend on deployed model
    return {
      categories: ['upper_body', 'lower_body', 'dress', 'full_body'],
      supportsScenes: false, // Depends on implementation
      requiresGarmentMask: this.config.model === 'idm-vton',
      supportsTextPrompt: this.config.model === 'idm-vton',
      maxResolution: { width: 1024, height: 768 },
      generationTimeRange: { min: 10, max: 40 },
      costPerGeneration: this.estimateGpuCost(),
      commercialLicense: true, // Self-hosted = full control
    };
  }

  private estimateGpuCost(): number {
    // Example: A100 at $1.89/hour, ~20 seconds per gen
    // = $1.89 / 3600 * 20 = $0.0105
    return this.config.costPerSecond 
      ? this.config.costPerSecond * 20 
      : 0.0105;
  }

  estimateCost(input: TryOnInput): number {
    return this.estimateGpuCost();
  }

  async healthCheck(): Promise<ProviderHealth> {
    try {
      const response = await fetch(`${this.config.endpoint}/health`);
      return {
        status: response.ok ? 'healthy' : 'unhealthy',
        lastChecked: Date.now(),
      };
    } catch {
      return {
        status: 'unhealthy',
        lastChecked: Date.now(),
      };
    }
  }

  normalizeError(error: unknown): ProviderError {
    return {
      provider: 'self-hosted',
      code: 'SELF_HOSTED_ERROR',
      message: error instanceof Error ? error.message : 'Unknown error',
      retryable: true, // Assume transient issues
      originalError: error,
    };
  }
}
```

**Deployment Options**:
- **Modal**: Serverless GPU, auto-scaling
- **RunPod**: Spot GPU instances, low cost
- **AWS/GCP**: Managed GPU infrastructure
- **ComfyUI**: For advanced workflows

**Cost Estimation** (Self-hosted vs API):

| Provider | Cost/Gen | GPU Type | Break-even Point |
|----------|----------|----------|------------------|
| fal.ai (CatVTON) | $0.06 | N/A | N/A |
| Self-hosted (A100) | ~$0.01 | A100 | ~1,500 gens/month |
| Self-hosted (A10G) | ~$0.006 | A10G | ~600 gens/month |

---

## 5. Error Handling Strategy

### Error Types

```typescript
type ErrorCode =
  | 'INVALID_INPUT'        // Bad request, validation failed
  | 'RATE_LIMIT_EXCEEDED'  // Provider rate limit hit
  | 'INSUFFICIENT_CREDITS' // Account out of credits
  | 'PROVIDER_TIMEOUT'     // Generation took too long
  | 'PROVIDER_ERROR'       // Internal provider error
  | 'NETWORK_ERROR'        // Network/connection issue
  | 'AUTHENTICATION_ERROR' // Invalid API key
  | 'UNKNOWN_ERROR';       // Catch-all

interface ProviderError extends Error {
  provider: ProviderName;
  code: ErrorCode;
  message: string;
  retryable: boolean;
  originalError?: unknown;
  statusCode?: number;
}

class ProviderError extends Error {
  constructor(
    public provider: ProviderName,
    public code: ErrorCode,
    message: string,
    public retryable: boolean = false,
    public originalError?: unknown
  ) {
    super(message);
    this.name = 'ProviderError';
  }
}
```

### Error Handling Per Provider

#### fal.ai Error Mapping

```typescript
private mapFalError(error: FalApiError): ProviderError {
  const codeMap: Record<number, ErrorCode> = {
    400: 'INVALID_INPUT',
    401: 'AUTHENTICATION_ERROR',
    402: 'INSUFFICIENT_CREDITS',
    429: 'RATE_LIMIT_EXCEEDED',
    500: 'PROVIDER_ERROR',
    503: 'PROVIDER_ERROR',
    504: 'PROVIDER_TIMEOUT',
  };

  const code = codeMap[error.statusCode] || 'UNKNOWN_ERROR';
  const retryable = [429, 500, 503, 504].includes(error.statusCode);

  return new ProviderError('fal.ai', code, error.message, retryable, error);
}
```

#### FASHN Error Mapping

```typescript
private mapFashnError(error: FashnApiError): ProviderError {
  const codeMap: Record<number, ErrorCode> = {
    400: 'INVALID_INPUT',
    401: 'AUTHENTICATION_ERROR',
    402: 'INSUFFICIENT_CREDITS',
    429: 'RATE_LIMIT_EXCEEDED',
    500: 'PROVIDER_ERROR',
    503: 'PROVIDER_ERROR',
  };

  const code = codeMap[error.statusCode] || 'UNKNOWN_ERROR';
  const retryable = [429, 500, 503].includes(error.statusCode);

  return new ProviderError('fashn', code, error.message, retryable, error);
}
```

### Retry Strategy

```typescript
interface RetryConfig {
  maxAttempts: number;
  initialDelayMs: number;
  maxDelayMs: number;
  backoffMultiplier: number;
}

async function withRetry<T>(
  fn: () => Promise<T>,
  config: RetryConfig,
  shouldRetry: (error: ProviderError) => boolean
): Promise<T> {
  let attempt = 0;
  let delay = config.initialDelayMs;

  while (attempt < config.maxAttempts) {
    try {
      return await fn();
    } catch (error) {
      attempt++;
      
      if (!(error instanceof ProviderError) || !shouldRetry(error)) {
        throw error;
      }

      if (attempt >= config.maxAttempts) {
        throw error;
      }

      // Exponential backoff
      await sleep(delay);
      delay = Math.min(delay * config.backoffMultiplier, config.maxDelayMs);
    }
  }

  throw new Error('Retry failed');
}
```

---

## 6. Cost Tracking Hooks

### Cost Tracking Interface

```typescript
interface CostTracker {
  /**
   * Record a generation cost
   */
  recordCost(record: CostRecord): Promise<void>;

  /**
   * Get cost summary for period
   */
  getCostSummary(options: CostSummaryOptions): Promise<CostSummary>;

  /**
   * Get cost breakdown by provider
   */
  getProviderBreakdown(
    startDate: Date,
    endDate: Date
  ): Promise<ProviderCostBreakdown[]>;
}

interface CostRecord {
  provider: ProviderName;
  model: string;
  cost: number; // USD
  durationMs: number;
  success: boolean;
  userId?: string;
  requestId: string;
  timestamp: number;
  metadata?: Record<string, unknown>;
}

interface CostSummary {
  totalCost: number;
  totalGenerations: number;
  averageCost: number;
  successRate: number;
  period: { start: Date; end: Date };
}

interface ProviderCostBreakdown {
  provider: ProviderName;
  totalCost: number;
  totalGenerations: number;
  averageCost: number;
  successRate: number;
}
```

### Implementation with Vercel KV

```typescript
class VercelKvCostTracker implements CostTracker {
  constructor(private kv: VercelKV) {}

  async recordCost(record: CostRecord): Promise<void> {
    const key = `cost:${record.requestId}`;
    const dailyKey = `cost:daily:${this.getDayKey(record.timestamp)}`;
    const providerKey = `cost:provider:${record.provider}:${this.getDayKey(record.timestamp)}`;

    await Promise.all([
      // Store individual record
      this.kv.set(key, record, { ex: 60 * 60 * 24 * 30 }), // 30 days
      
      // Increment daily total
      this.kv.hincrby(dailyKey, 'total_cost', Math.round(record.cost * 100)), // Store cents
      this.kv.hincrby(dailyKey, 'total_generations', 1),
      this.kv.hincrby(dailyKey, record.success ? 'success' : 'failure', 1),
      
      // Increment provider total
      this.kv.hincrby(providerKey, 'total_cost', Math.round(record.cost * 100)),
      this.kv.hincrby(providerKey, 'total_generations', 1),
      
      // Set expiry on aggregate keys
      this.kv.expire(dailyKey, 60 * 60 * 24 * 90), // 90 days
      this.kv.expire(providerKey, 60 * 60 * 24 * 90),
    ]);
  }

  async getCostSummary(options: CostSummaryOptions): Promise<CostSummary> {
    const { startDate, endDate } = options;
    const days = this.getDayRange(startDate, endDate);

    let totalCost = 0;
    let totalGenerations = 0;
    let successCount = 0;

    for (const day of days) {
      const dailyKey = `cost:daily:${day}`;
      const data = await this.kv.hgetall(dailyKey);
      
      if (data) {
        totalCost += (data.total_cost as number) / 100; // Convert cents to dollars
        totalGenerations += data.total_generations as number;
        successCount += data.success as number;
      }
    }

    return {
      totalCost,
      totalGenerations,
      averageCost: totalGenerations > 0 ? totalCost / totalGenerations : 0,
      successRate: totalGenerations > 0 ? successCount / totalGenerations : 0,
      period: { start: startDate, end: endDate },
    };
  }

  async getProviderBreakdown(
    startDate: Date,
    endDate: Date
  ): Promise<ProviderCostBreakdown[]> {
    // Similar implementation aggregating by provider
    // ...
  }

  private getDayKey(timestamp: number): string {
    return new Date(timestamp).toISOString().split('T')[0];
  }

  private getDayRange(start: Date, end: Date): string[] {
    const days: string[] = [];
    const current = new Date(start);
    
    while (current <= end) {
      days.push(this.getDayKey(current.getTime()));
      current.setDate(current.getDate() + 1);
    }
    
    return days;
  }
}
```

### Cost Tracking Integration

```typescript
class TryOnService {
  constructor(
    private providers: Map<ProviderName, ITryOnProvider>,
    private costTracker: CostTracker
  ) {}

  async generate(input: TryOnInput): Promise<TryOnResult> {
    const provider = this.selectProvider(input);
    const startTime = Date.now();

    try {
      const result = await provider.generate(input);

      // Record successful cost
      await this.costTracker.recordCost({
        provider: provider.name,
        model: result.metadata.model,
        cost: result.metadata.cost,
        durationMs: result.metadata.durationMs,
        success: true,
        userId: input.metadata?.userId,
        requestId: result.metadata.requestId,
        timestamp: Date.now(),
      });

      return result;

    } catch (error) {
      // Record failed attempt cost (if applicable)
      if (error instanceof ProviderError) {
        await this.costTracker.recordCost({
          provider: provider.name,
          model: 'unknown',
          cost: 0, // Most providers don't charge for failures
          durationMs: Date.now() - startTime,
          success: false,
          userId: input.metadata?.userId,
          requestId: input.metadata?.requestId || generateId(),
          timestamp: Date.now(),
        });
      }

      throw error;
    }
  }
}
```

---

## 7. Failover & Fallback Strategy

### Failover Configuration

```typescript
interface FailoverConfig {
  /** Primary provider to use */
  primary: ProviderName;

  /** Ordered list of fallback providers */
  fallbacks: ProviderName[];

  /** Conditions that trigger failover */
  triggers: {
    /** Failover on provider errors? */
    onError: boolean;

    /** Failover on rate limit? */
    onRateLimit: boolean;

    /** Failover on timeout? */
    onTimeout: boolean;

    /** Custom error codes to trigger failover */
    errorCodes?: ErrorCode[];
  };

  /** Should we retry primary before failing over? */
  retryPrimaryFirst: boolean;

  /** Maximum failover attempts */
  maxFailovers: number;
}
```

### Failover Implementation

```typescript
class TryOnService {
  constructor(
    private providers: Map<ProviderName, ITryOnProvider>,
    private failoverConfig: FailoverConfig,
    private costTracker: CostTracker
  ) {}

  async generate(input: TryOnInput): Promise<TryOnResult> {
    const providerChain = [
      this.failoverConfig.primary,
      ...this.failoverConfig.fallbacks,
    ];

    let lastError: ProviderError | undefined;

    for (let i = 0; i < providerChain.length; i++) {
      const providerName = providerChain[i];
      const provider = this.providers.get(providerName);

      if (!provider) {
        console.warn(`Provider ${providerName} not configured, skipping`);
        continue;
      }

      // Check if provider is healthy before attempting
      const health = await provider.healthCheck();
      if (health.status === 'unhealthy') {
        console.warn(`Provider ${providerName} is unhealthy, skipping`);
        continue;
      }

      try {
        console.log(`Attempting generation with ${providerName} (attempt ${i + 1})`);
        
        const result = await this.generateWithProvider(provider, input);
        
        // Success! Record which provider was used
        if (i > 0) {
          console.log(`Failover successful: ${providerName} succeeded after ${i} failures`);
          await this.recordFailoverSuccess(providerName, lastError);
        }

        return result;

      } catch (error) {
        if (!(error instanceof ProviderError)) {
          throw error; // Unexpected error, don't failover
        }

        lastError = error;
        console.warn(`Provider ${providerName} failed:`, error.message);

        // Check if we should failover based on error type
        if (!this.shouldFailover(error)) {
          throw error; // Don't failover on this error
        }

        // Continue to next provider
        if (i < providerChain.length - 1) {
          console.log(`Failing over to next provider...`);
        }
      }
    }

    // All providers failed
    throw new Error(
      `All providers failed. Last error: ${lastError?.message || 'Unknown'}`
    );
  }

  private shouldFailover(error: ProviderError): boolean {
    const { triggers } = this.failoverConfig;

    // Always failover on retryable errors
    if (!error.retryable) {
      return false;
    }

    // Check specific triggers
    if (triggers.onError && error.code === 'PROVIDER_ERROR') return true;
    if (triggers.onRateLimit && error.code === 'RATE_LIMIT_EXCEEDED') return true;
    if (triggers.onTimeout && error.code === 'PROVIDER_TIMEOUT') return true;

    // Check custom error codes
    if (triggers.errorCodes?.includes(error.code)) return true;

    return false;
  }

  private async generateWithProvider(
    provider: ITryOnProvider,
    input: TryOnInput
  ): Promise<TryOnResult> {
    // Add retry logic if configured
    if (this.failoverConfig.retryPrimaryFirst) {
      return await withRetry(
        () => provider.generate(input),
        {
          maxAttempts: 3,
          initialDelayMs: 1000,
          maxDelayMs: 5000,
          backoffMultiplier: 2,
        },
        (error) => error.retryable
      );
    }

    return await provider.generate(input);
  }

  private async recordFailoverSuccess(
    successfulProvider: ProviderName,
    lastError?: ProviderError
  ): Promise<void> {
    // Log failover event for monitoring
    console.log('Failover event:', {
      successfulProvider,
      lastError: lastError?.code,
      timestamp: Date.now(),
    });

    // Could send to monitoring service (Sentry, Datadog, etc.)
  }
}
```

### Recommended Failover Configuration

```typescript
// MVP Configuration
const mvpFailoverConfig: FailoverConfig = {
  primary: 'fal.ai', // CatVTON by default
  fallbacks: ['fashn'], // Commercial backup
  triggers: {
    onError: true,
    onRateLimit: true,
    onTimeout: true,
  },
  retryPrimaryFirst: true, // Retry fal.ai before switching to FASHN
  maxFailovers: 2,
};

// Production Configuration (with self-hosted)
const prodFailoverConfig: FailoverConfig = {
  primary: 'self-hosted', // Cheapest
  fallbacks: ['fal.ai', 'fashn'], // API backups
  triggers: {
    onError: true,
    onRateLimit: true,
    onTimeout: true,
  },
  retryPrimaryFirst: false, // Fast failover
  maxFailovers: 2,
};
```

---

## 8. A/B Testing Support

### A/B Testing Configuration

```typescript
interface ABTestConfig {
  /** Test identifier */
  testId: string;

  /** Test description */
  description: string;

  /** Percentage of traffic for each variant (must sum to 100) */
  variants: {
    name: string;
    provider: ProviderName;
    model?: string;
    weight: number; // 0-100
  }[];

  /** User attribute for consistent assignment (e.g., userId) */
  consistentBy?: 'userId' | 'sessionId';

  /** Start/end dates */
  startDate: Date;
  endDate: Date;
}

interface ABTestAssignment {
  testId: string;
  variant: string;
  provider: ProviderName;
  assignedAt: number;
}
```

### A/B Testing Implementation

```typescript
class ABTestManager {
  constructor(
    private tests: Map<string, ABTestConfig>,
    private kv: VercelKV
  ) {}

  async assignVariant(
    userId: string,
    input: TryOnInput
  ): Promise<ABTestAssignment | null> {
    const activeTests = Array.from(this.tests.values()).filter((test) => {
      const now = Date.now();
      return now >= test.startDate.getTime() && now <= test.endDate.getTime();
    });

    if (activeTests.length === 0) {
      return null; // No active tests
    }

    // For MVP, support one test at a time
    const test = activeTests[0];

    // Check if user already assigned
    const existingAssignment = await this.getAssignment(userId, test.testId);
    if (existingAssignment) {
      return existingAssignment;
    }

    // Assign variant based on weight
    const variant = this.selectVariant(test, userId);

    const assignment: ABTestAssignment = {
      testId: test.testId,
      variant: variant.name,
      provider: variant.provider,
      assignedAt: Date.now(),
    };

    // Store assignment for consistency
    await this.saveAssignment(userId, test.testId, assignment);

    return assignment;
  }

  private selectVariant(
    test: ABTestConfig,
    userId: string
  ): ABTestConfig['variants'][0] {
    // Deterministic assignment based on userId
    const hash = this.hashString(userId + test.testId);
    const bucket = hash % 100; // 0-99

    let cumulative = 0;
    for (const variant of test.variants) {
      cumulative += variant.weight;
      if (bucket < cumulative) {
        return variant;
      }
    }

    return test.variants[0]; // Fallback
  }

  private hashString(str: string): number {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      const char = str.charCodeAt(i);
      hash = (hash << 5) - hash + char;
      hash = hash & hash; // Convert to 32-bit integer
    }
    return Math.abs(hash);
  }

  private async getAssignment(
    userId: string,
    testId: string
  ): Promise<ABTestAssignment | null> {
    const key = `ab:${testId}:${userId}`;
    return await this.kv.get<ABTestAssignment>(key);
  }

  private async saveAssignment(
    userId: string,
    testId: string,
    assignment: ABTestAssignment
  ): Promise<void> {
    const key = `ab:${testId}:${userId}`;
    const test = this.tests.get(testId);
    
    if (test) {
      const ttl = Math.floor(
        (test.endDate.getTime() - Date.now()) / 1000 + 60 * 60 * 24 * 7
      ); // Test duration + 7 days
      await this.kv.set(key, assignment, { ex: ttl });
    }
  }
}
```

### Integration with TryOnService

```typescript
class TryOnService {
  constructor(
    private providers: Map<ProviderName, ITryOnProvider>,
    private failoverConfig: FailoverConfig,
    private costTracker: CostTracker,
    private abTestManager?: ABTestManager
  ) {}

  async generate(input: TryOnInput): Promise<TryOnResult> {
    let selectedProvider: ProviderName = this.failoverConfig.primary;

    // Check for A/B test assignment
    if (this.abTestManager && input.metadata?.userId) {
      const assignment = await this.abTestManager.assignVariant(
        input.metadata.userId,
        input
      );

      if (assignment) {
        console.log(`A/B test assignment:`, assignment);
        selectedProvider = assignment.provider;

        // Add test metadata to input
        input.metadata = {
          ...input.metadata,
          experimentGroup: assignment.variant,
        };
      }
    }

    // Continue with normal generation flow, using selected provider
    return await this.generateWithFailover(input, selectedProvider);
  }
}
```

### Example A/B Test

```typescript
const qualitySpeedTest: ABTestConfig = {
  testId: 'quality-vs-speed-2026-02',
  description: 'Test CatVTON (fast) vs IDM-VTON (quality)',
  variants: [
    {
      name: 'fast',
      provider: 'fal.ai',
      model: 'cat-vton',
      weight: 50,
    },
    {
      name: 'quality',
      provider: 'fal.ai',
      model: 'idm-vton',
      weight: 50,
    },
  ],
  consistentBy: 'userId',
  startDate: new Date('2026-02-10'),
  endDate: new Date('2026-02-24'),
};
```

---

## 9. Provider Factory & Dependency Injection

### Provider Factory

```typescript
class ProviderFactory {
  private static providers = new Map<ProviderName, ITryOnProvider>();

  static register(provider: ITryOnProvider): void {
    this.providers.set(provider.name, provider);
  }

  static get(name: ProviderName): ITryOnProvider | undefined {
    return this.providers.get(name);
  }

  static getAll(): ITryOnProvider[] {
    return Array.from(this.providers.values());
  }

  static async initialize(): Promise<void> {
    // Initialize providers from environment variables
    if (process.env.FAL_API_KEY) {
      const falProvider = new FalAiProvider({
        apiKey: process.env.FAL_API_KEY,
        defaultModel: 'cat-vton',
      });
      this.register(falProvider);
    }

    if (process.env.FASHN_API_KEY) {
      const fashnProvider = new FashnProvider({
        apiKey: process.env.FASHN_API_KEY,
        tier: process.env.FASHN_TIER as any || 'on-demand',
      });
      this.register(fashnProvider);
    }

    if (process.env.SELF_HOSTED_ENDPOINT) {
      const selfHostedProvider = new SelfHostedProvider({
        endpoint: process.env.SELF_HOSTED_ENDPOINT,
        apiKey: process.env.SELF_HOSTED_API_KEY,
        model: 'cat-vton',
      });
      this.register(selfHostedProvider);
    }

    console.log(`Initialized ${this.providers.size} providers:`, 
      Array.from(this.providers.keys())
    );
  }
}
```

### Service Initialization

```typescript
// In your app initialization (e.g., Next.js middleware or API route)
export async function initializeTryOnService(): Promise<TryOnService> {
  // Initialize providers
  await ProviderFactory.initialize();

  // Initialize cost tracker
  const kv = await getVercelKV();
  const costTracker = new VercelKvCostTracker(kv);

  // Initialize A/B test manager (optional)
  const abTestManager = new ABTestManager(new Map(), kv);

  // Configure failover
  const failoverConfig: FailoverConfig = {
    primary: process.env.PRIMARY_PROVIDER as ProviderName || 'fal.ai',
    fallbacks: (process.env.FALLBACK_PROVIDERS?.split(',') as ProviderName[]) || ['fashn'],
    triggers: {
      onError: true,
      onRateLimit: true,
      onTimeout: true,
    },
    retryPrimaryFirst: true,
    maxFailovers: 2,
  };

  // Create service
  return new TryOnService(
    ProviderFactory.providers,
    failoverConfig,
    costTracker,
    abTestManager
  );
}
```

---

## 10. Usage Examples

### Basic Generation

```typescript
import { initializeTryOnService } from './services/tryon';

const service = await initializeTryOnService();

const result = await service.generate({
  humanImage: { type: 'url', url: 'https://example.com/person.jpg' },
  garmentImage: { type: 'url', url: 'https://example.com/shirt.jpg' },
  category: 'upper_body',
  quality: 'balanced',
  metadata: {
    userId: 'user123',
    requestId: 'req_abc123',
    timestamp: Date.now(),
  },
});

console.log('Generated image:', result.image.url);
console.log('Cost:', result.metadata.cost);
console.log('Provider used:', result.metadata.provider);
```

### With Failover

```typescript
// Failover happens automatically
try {
  const result = await service.generate(input);
  console.log('Success with:', result.metadata.provider);
} catch (error) {
  if (error instanceof Error) {
    console.error('All providers failed:', error.message);
  }
}
```

### Cost Analysis

```typescript
import { costTracker } from './services/tryon';

// Get last 7 days cost summary
const summary = await costTracker.getCostSummary({
  startDate: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000),
  endDate: new Date(),
});

console.log('Total cost:', summary.totalCost);
console.log('Average cost per generation:', summary.averageCost);
console.log('Success rate:', summary.successRate);

// Get provider breakdown
const breakdown = await costTracker.getProviderBreakdown(
  new Date('2026-02-01'),
  new Date('2026-02-28')
);

breakdown.forEach((provider) => {
  console.log(`${provider.provider}:`, {
    cost: provider.totalCost,
    generations: provider.totalGenerations,
    average: provider.averageCost,
  });
});
```

### Provider Capabilities

```typescript
const provider = ProviderFactory.get('fal.ai');
const capabilities = provider?.getCapabilities();

console.log('Supported categories:', capabilities?.categories);
console.log('Cost per generation:', capabilities?.costPerGeneration);
console.log('Commercial license:', capabilities?.commercialLicense);
```

---

## 11. Testing Strategy

### Unit Tests

```typescript
describe('FalAiProvider', () => {
  let provider: FalAiProvider;

  beforeEach(() => {
    provider = new FalAiProvider({
      apiKey: 'test-key',
      defaultModel: 'cat-vton',
    });
  });

  test('should map input correctly', () => {
    const input: TryOnInput = {
      humanImage: { type: 'url', url: 'https://example.com/person.jpg' },
      garmentImage: { type: 'url', url: 'https://example.com/shirt.jpg' },
      category: 'upper_body',
    };

    const mapped = provider.toFalFormat(input, 'fal-ai/cat-vton');
    
    expect(mapped).toEqual({
      human_image_url: 'https://example.com/person.jpg',
      garment_image_url: 'https://example.com/shirt.jpg',
      cloth_type: 'upper',
    });
  });

  test('should normalize errors correctly', () => {
    const falError = new FalApiError(429, 'Rate limit exceeded');
    const normalized = provider.normalizeError(falError);

    expect(normalized.code).toBe('RATE_LIMIT_EXCEEDED');
    expect(normalized.retryable).toBe(true);
    expect(normalized.provider).toBe('fal.ai');
  });
});
```

### Integration Tests

```typescript
describe('TryOnService Integration', () => {
  test('should failover on provider error', async () => {
    // Mock fal.ai to fail, FASHN to succeed
    const service = createTestService({
      providers: {
        'fal.ai': createMockProvider({ shouldFail: true }),
        'fashn': createMockProvider({ shouldFail: false }),
      },
    });

    const result = await service.generate(testInput);

    expect(result.metadata.provider).toBe('fashn');
  });

  test('should track costs correctly', async () => {
    const costTracker = new MockCostTracker();
    const service = createTestService({ costTracker });

    await service.generate(testInput);

    const records = await costTracker.getRecords();
    expect(records).toHaveLength(1);
    expect(records[0].cost).toBeGreaterThan(0);
  });
});
```

---

## 12. Monitoring & Observability

### Metrics to Track

```typescript
interface ProviderMetrics {
  // Performance
  avgGenerationTime: number;
  p95GenerationTime: number;
  p99GenerationTime: number;

  // Reliability
  successRate: number;
  errorRate: number;
  failoverRate: number;

  // Cost
  totalCost: number;
  avgCostPerGeneration: number;
  costTrend: number; // % change

  // Usage
  totalGenerations: number;
  generationsPerDay: number;
}
```

### Integration Points

```typescript
// Log to monitoring service
async function recordMetrics(result: TryOnResult): Promise<void> {
  // Example: Send to Vercel Analytics
  await fetch('/api/analytics', {
    method: 'POST',
    body: JSON.stringify({
      event: 'generation_complete',
      provider: result.metadata.provider,
      duration: result.metadata.durationMs,
      cost: result.metadata.cost,
    }),
  });

  // Example: Send to Sentry (for errors)
  if (result.metadata.durationMs > 60000) {
    Sentry.captureMessage('Slow generation detected', {
      level: 'warning',
      extra: result.metadata,
    });
  }
}
```

---

## 13. Migration Path

### Phase 1: MVP (Weeks 1-4)
- Implement `ITryOnProvider` interface
- Build `FalAiProvider` with CatVTON
- Basic cost tracking
- No failover (single provider)

### Phase 2: Resilience (Weeks 5-8)
- Add `FashnProvider`
- Implement failover logic
- Add health checks
- Enhanced error handling

### Phase 3: Optimization (Post-MVP)
- A/B testing framework
- Provider auto-selection based on performance
- Self-hosted provider option
- Advanced cost optimization

---

## 14. Environment Variables

```bash
# Provider API Keys
FAL_API_KEY=your_fal_api_key
FASHN_API_KEY=your_fashn_api_key
SELF_HOSTED_ENDPOINT=https://your-inference-server.com
SELF_HOSTED_API_KEY=your_custom_api_key

# Provider Configuration
PRIMARY_PROVIDER=fal.ai
FALLBACK_PROVIDERS=fashn
FASHN_TIER=on-demand # or tier-1, tier-2, tier-3

# Cost Tracking
VERCEL_KV_URL=your_kv_url
VERCEL_KV_REST_API_TOKEN=your_token

# Feature Flags
ENABLE_AB_TESTING=false
ENABLE_SELF_HOSTED=false
```

---

## 15. Licensing Considerations

### Critical Action Items

1. **CatVTON & IDM-VTON**:
   - Both models are **non-commercial** (CC BY-NC-SA 4.0)
   - Using via fal.ai **does not grant commercial license**
   - **MUST** contact model authors before production launch
   - Alternative: Use FASHN API which includes commercial license

2. **FASHN API**:
   - Commercial license **included** in all tiers
   - Safe for production use
   - Recommended as primary provider if licensing uncertain

3. **Recommended Approach**:
   - Start development with fal.ai CatVTON (fastest integration)
   - Contact CatVTON authors immediately for commercial licensing
   - Have FASHN as production-ready backup
   - Budget for FASHN costs in case licensing negotiation fails

---

## 16. Open Questions & Future Work

### Open Questions
- [ ] CatVTON commercial licensing terms and cost
- [ ] IDM-VTON commercial licensing availability
- [ ] FASHN API SLA and uptime guarantees
- [ ] Self-hosted GPU provider selection (Modal vs RunPod vs AWS)

### Future Enhancements
- [ ] Scene integration (festival backgrounds) - requires FASHN or custom solution
- [ ] Multi-garment try-on (outfit combinations)
- [ ] Real-time preview (progressive generation)
- [ ] Provider performance-based auto-selection
- [ ] Cost prediction and budget alerts
- [ ] Provider marketplace (allow users to choose provider)

---

## Change Log

| Date | Change | Author |
|------|--------|--------|
| 2026-02-09 | Initial adapter architecture design | AI Agent |

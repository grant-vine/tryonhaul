# Social Sharing Strategy

> **Last Updated**: 2026-02-09  
> **Related**: [DATA-FLOW.md](./DATA-FLOW.md) | [REQUIREMENTS.md](../user-stories/REQUIREMENTS.md) | [MVP-FEATURES.md](../user-stories/MVP-FEATURES.md)

---

## Purpose

This document defines the social sharing implementation strategy for Try on Haul, based on research into platform capabilities, Web Share API support, and our "no server storage" constraint.

---

## Executive Summary

**Primary Approach**: Web Share API Level 2 (file sharing)  
**Fallback Approach**: Download + Copy Link + Manual Share  
**Storage Implication**: "No permanent storage" is viable - images shared directly from memory

**Key Finding**: Social platforms (Instagram, TikTok, Facebook) do NOT provide web/PWA APIs for direct media injection. Official sharing mechanisms require native SDKs. Web Share API is the only standards-based way to share images directly from a PWA.

---

## Platform Capability Matrix

| Platform | Web Share API | Direct Media Injection | Best PWA Approach |
|----------|--------------|----------------------|-------------------|
| Instagram | Via OS share sheet | Native SDK only | Web Share → user picks IG |
| TikTok | Via OS share sheet | Native SDK only | Web Share → user picks TikTok |
| Facebook | Via OS share sheet | Native SDK only | Web Share → user picks FB |
| X (Twitter) | Via OS share sheet | No (text/URL only) | Web Share OR Web Intent |
| Pinterest | Via OS share sheet | No (Save button) | Web Share OR Pinterest Save |

---

## Web Share API Support

### Browser Compatibility

| Platform / Browser | Text/URL Share | File Share | Notes |
|-------------------|----------------|------------|-------|
| iOS Safari | ✅ | ✅ (iOS 14+) | Primary mobile target |
| iOS Chrome | ✅ | ✅ | Uses iOS share sheet |
| Android Chrome | ✅ | ✅ | Strong file support |
| Android Samsung | ✅ | ✅ | Supported |
| Desktop Chrome/Edge | ✅ | ✅ | Works, limited app targets |
| Firefox Desktop | ❌ | ❌ | No Web Share API |

### How It Works

```typescript
// Check if file sharing is supported
const canShare = navigator.canShare?.({ files: [imageFile] });

if (canShare) {
  // Share image directly from memory
  await navigator.share({
    title: "My Try-On Look",
    text: "Check out how I'd look in this outfit! #TryOnHaul",
    url: affiliateLink,
    files: [imageFile] // Blob/File object
  });
} else {
  // Fallback to download
  downloadImage(imageFile);
  copyToClipboard(affiliateLink);
}
```

**Key Points**:
- User chooses target app from OS share sheet (we cannot force a specific app)
- Image is shared directly from browser memory (no server upload required)
- Works without storing image on our server
- Affiliate link included in share text

---

## Platform-Specific Implementation

### 1. Instagram

**Limitation**: No web API for direct posting. Instagram Stories sharing requires native iOS/Android SDK with Facebook App ID.

**Our Approach**:
1. Web Share API → user selects Instagram from share sheet
2. Image appears in Instagram as incoming share
3. User creates post/story from shared image

**Fallback**:
1. Download image to device
2. Copy affiliate link to clipboard  
3. Show instruction: "Open Instagram → New Post → Select from Photos → Paste link in caption"

**Sources**:
- [Instagram Sharing to Stories](https://developers.facebook.com/docs/instagram-platform/sharing-to-stories/) (native only)

---

### 2. TikTok

**Limitation**: TikTok Share Kit is native SDK only. No web API for media injection.

**Our Approach**:
1. Web Share API → user selects TikTok from share sheet
2. Image shared to TikTok (if TikTok accepts image shares)

**Fallback**:
1. Download image
2. Copy affiliate link
3. Show instruction: "Open TikTok → Create → Upload from gallery"

**Sources**:
- [TikTok Share Kit](https://developers.tiktok.com/products/share-kit/) (native only)

---

### 3. Facebook

**Limitation**: Facebook Stories sharing requires native SDK. Feed posting via Graph API requires server-side OAuth.

**Our Approach**:
1. Web Share API → user selects Facebook from share sheet
2. Facebook receives shared content

**Alternative** (if we add server component later):
- Use Facebook Graph API for posting on behalf of user
- Requires: Server-side token storage, user permission, temporary image hosting
- **Not recommended for MVP** due to storage requirements

**Sources**:
- [Facebook Sharing to Stories](https://developers.facebook.com/docs/sharing/sharing-to-stories/) (native only)

---

### 4. X (Twitter)

**Web Intent Available**: X supports web-based sharing via URL intent.

**Our Approach (Primary)**:
```typescript
const tweetUrl = new URL("https://twitter.com/intent/tweet");
tweetUrl.searchParams.set("text", "Check out my festival look! #TryOnHaul");
tweetUrl.searchParams.set("url", affiliateLink);
window.open(tweetUrl, "_blank");
```

**Limitation**: Web Intent does NOT support image attachment. User must:
1. Download image separately
2. Attach when composing tweet

**Alternative**: Web Share API → user picks X from share sheet (includes image)

**Sources**:
- [X Web Intent](https://developer.x.com/en/docs/x-for-websites/tweet-button/guides/web-intent)

---

### 5. Pinterest

**Save Button Available**: Pinterest provides web-based Save Add-On.

**Our Approach**:
```typescript
const pinUrl = new URL("https://pinterest.com/pin/create/button/");
pinUrl.searchParams.set("url", affiliateLink);
pinUrl.searchParams.set("media", imageUrl); // Requires hosted image URL
pinUrl.searchParams.set("description", "Festival outfit try-on #TryOnHaul");
window.open(pinUrl, "_blank");
```

**Limitation**: Pinterest Save requires a **hosted image URL**. This means:
- We need temporary image hosting for Pinterest shares
- Or use Web Share API (user picks Pinterest from share sheet)

**Sources**:
- [Pinterest Save Button](https://help.pinterest.com/en/business/article/save-button)

---

## Storage Decision

Based on research, here's the storage requirement for each sharing approach:

| Approach | Server Storage Required? | Notes |
|----------|-------------------------|-------|
| Web Share API (files) | ❌ No | Image shared from browser memory |
| Download + manual share | ❌ No | Image downloaded to user device |
| X Web Intent | ❌ No | Only shares text/URL |
| Pinterest Save Button | ✅ Yes (temp) | Requires hosted image URL |
| Facebook Graph API | ✅ Yes (temp) | Requires hosted image for API call |

### Recommendation

**For MVP ("no server storage")**:
- Use Web Share API as primary (covers Instagram, TikTok, Facebook, X)
- Use X Web Intent as backup for X
- **Exclude Pinterest Save Button** (requires hosted URL)
- Or: Add temporary 24-hour Blob storage specifically for Pinterest shares

**Post-MVP**:
- Add ephemeral Blob storage (24-hour TTL) for Pinterest support
- Consider Facebook Graph API for direct posting (requires OAuth flow)

---

## User Experience Flows

### Primary Flow (Web Share API)

```
User taps "Share" button
         │
         ▼
    ┌─────────────────────┐
    │ Check: canShare?    │
    └─────────┬───────────┘
              │
    ┌─────────▼───────────┐
    │ navigator.share({   │
    │   files: [image],   │
    │   text: caption,    │
    │   url: affiliateLink│
    │ })                  │
    └─────────┬───────────┘
              │
              ▼
    ┌─────────────────────┐
    │ OS Share Sheet      │
    │ appears with app    │
    │ options             │
    └─────────┬───────────┘
              │
              ▼
    User selects target app
              │
              ▼
    Share completes in target app
```

**Steps**: 2-4 taps

---

### Fallback Flow (Download + Copy Link)

```
User taps "Share" button
         │
         ▼
    ┌─────────────────────┐
    │ Check: canShare?    │
    │ Result: NO          │
    └─────────┬───────────┘
              │
              ▼
    ┌─────────────────────┐
    │ Show Share Modal:   │
    │ - Download Image    │
    │ - Copy Link         │
    │ - Platform buttons  │
    └─────────┬───────────┘
              │
              ▼
    User taps "Download Image"
    → Image saved to Photos
              │
              ▼
    User taps "Copy Link"  
    → Affiliate URL in clipboard
              │
              ▼
    User taps platform icon
    → Opens Instagram/TikTok/etc
              │
              ▼
    User creates post manually
    (selects image from gallery,
     pastes link in caption)
```

**Steps**: 6-9 taps

---

## Share Card Design

For branded sharing, we'll generate a share card that includes:

```
┌─────────────────────────────────┐
│                                 │
│      [Generated Try-On Image]   │
│                                 │
│   ─────────────────────────────│
│   TryOnHaul                     │
│   tryonhaul.com/outfit/abc123   │
└─────────────────────────────────┘
```

**Share Card Processing**:
1. Take generated try-on image
2. Add subtle branding overlay (logo + short URL)
3. Optimize for social dimensions:
   - Instagram: 1080x1080 (square) or 1080x1920 (story)
   - TikTok: 1080x1920 (portrait)
   - Facebook: 1200x630 (link preview) or 1080x1080 (feed)
   - X: 1200x675 (card) or 1080x1080 (square)
   - Pinterest: 1000x1500 (optimal pin)

**Processing**: Done client-side with Canvas API or Sharp (in Inngest job)

---

## Implementation Plan

### Phase 1: MVP
- [x] Web Share API with file support (primary)
- [x] Download image fallback
- [x] Copy affiliate link
- [ ] X Web Intent (direct link share)
- [ ] Platform-specific instructions in fallback modal

### Phase 2: Enhanced
- [ ] Share card generation with branding
- [ ] Multiple format options (square, portrait, story)
- [ ] Pinterest Save Button (requires temp storage)

### Phase 3: Advanced
- [ ] Facebook Graph API integration (direct posting)
- [ ] Share analytics (which platform, click-through)
- [ ] A/B test share card designs

---

## Code Implementation

### Share Service

```typescript
interface ShareOptions {
  image: Blob;
  title: string;
  caption: string;
  affiliateUrl: string;
}

async function shareImage(options: ShareOptions): Promise<ShareResult> {
  const { image, title, caption, affiliateUrl } = options;
  
  // Create File from Blob
  const file = new File([image], "try-on-haul.png", { type: "image/png" });
  
  // Check Web Share API support
  if (navigator.canShare?.({ files: [file] })) {
    try {
      await navigator.share({
        title,
        text: `${caption}\n\n${affiliateUrl}`,
        files: [file],
      });
      return { success: true, method: "web-share" };
    } catch (error) {
      if (error.name === "AbortError") {
        return { success: false, method: "web-share", reason: "cancelled" };
      }
      throw error;
    }
  }
  
  // Fallback: offer download + copy
  return { success: false, method: "fallback", reason: "not-supported" };
}

async function downloadImage(image: Blob, filename: string): Promise<void> {
  const url = URL.createObjectURL(image);
  const a = document.createElement("a");
  a.href = url;
  a.download = filename;
  a.click();
  URL.revokeObjectURL(url);
}

async function copyToClipboard(text: string): Promise<boolean> {
  try {
    await navigator.clipboard.writeText(text);
    return true;
  } catch {
    return false;
  }
}
```

### Share Modal Component (Fallback)

```tsx
function ShareModal({ image, affiliateUrl, onClose }: ShareModalProps) {
  const [linkCopied, setLinkCopied] = useState(false);
  const [imageDownloaded, setImageDownloaded] = useState(false);
  
  const handleDownload = async () => {
    await downloadImage(image, `try-on-haul-${Date.now()}.png`);
    setImageDownloaded(true);
  };
  
  const handleCopyLink = async () => {
    await copyToClipboard(affiliateUrl);
    setLinkCopied(true);
  };
  
  const handleOpenPlatform = (platform: Platform) => {
    // Track share attempt
    trackShare(platform);
    
    // Open platform (for X, use web intent)
    if (platform === "x") {
      const tweetUrl = new URL("https://twitter.com/intent/tweet");
      tweetUrl.searchParams.set("text", "Check out my festival look!");
      tweetUrl.searchParams.set("url", affiliateUrl);
      window.open(tweetUrl, "_blank");
    } else {
      // Show instruction overlay for other platforms
      showInstructions(platform);
    }
  };
  
  return (
    <Modal onClose={onClose}>
      <h2>Share Your Look</h2>
      
      <div className="share-actions">
        <Button onClick={handleDownload}>
          {imageDownloaded ? "✓ Downloaded" : "Download Image"}
        </Button>
        
        <Button onClick={handleCopyLink}>
          {linkCopied ? "✓ Copied" : "Copy Link"}
        </Button>
      </div>
      
      <p className="share-instructions">
        After downloading, open your favorite app and create a post
      </p>
      
      <div className="platform-buttons">
        {PLATFORMS.map((platform) => (
          <IconButton 
            key={platform.id}
            icon={platform.icon}
            onClick={() => handleOpenPlatform(platform.id)}
          />
        ))}
      </div>
    </Modal>
  );
}
```

---

## Analytics & Tracking

Track share events for understanding user behavior:

```typescript
interface ShareEvent {
  method: "web-share" | "download" | "copy-link" | "platform-click";
  platform?: "instagram" | "tiktok" | "facebook" | "x" | "pinterest";
  success: boolean;
  userId: string;
  generationId: string;
  timestamp: number;
}

// Log to Plausible
function trackShare(event: ShareEvent): void {
  plausible("share", {
    props: {
      method: event.method,
      platform: event.platform,
      success: event.success,
    },
  });
}
```

---

## Change Log

| Date | Change | Author |
|------|--------|--------|
| 2026-02-09 | Initial sharing strategy based on research | AI Agent |

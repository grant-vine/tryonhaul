# MVP Features & User Stories

> **Last Updated**: 2026-02-09  
> **Related**: [REQUIREMENTS.md](./REQUIREMENTS.md) | [OVERVIEW.md](../architecture/OVERVIEW.md)

---

## Purpose

This document defines the MVP feature set through user stories, prioritized for the 8-12 week development timeline.

---

## User Personas

### Primary: Festival-Goer (Freya)

- **Age**: 22-35
- **Behavior**: Shops for festival outfits online, heavy social media user
- **Goal**: Visualize how outfits will look on her before buying
- **Pain Point**: Can't try on clothes when shopping online; returns are hassle
- **Social**: Active on Instagram, TikTok; shares outfit content

### Secondary: Fashion Retailer (Riley)

- **Role**: E-commerce manager at festival fashion brand
- **Goal**: Reduce returns, increase conversions, get social exposure
- **Pain Point**: High return rates, customers can't visualize fit
- **Success Metric**: Conversion rate, social mentions

---

## Epic Overview

| Epic | Priority | Stories | Sprint Target |
|------|----------|---------|---------------|
| E1: Authentication | P0 | 4 | Sprint 1 |
| E2: Image Upload | P0 | 5 | Sprint 1-2 |
| E3: AI Generation | P0 | 6 | Sprint 2-3 |
| E4: Social Sharing | P0 | 5 | Sprint 3-4 |
| E5: Scene Compositing | P0 | 3 | Sprint 4 |
| E6: Affiliate Tracking | P1 | 4 | Sprint 5 |
| E7: Partner Integration | P2 | 4 | Sprint 6+ |

---

## Epic 1: Authentication (P0)

### US-1.1: Social Login
**As a** user  
**I want to** log in with my social media account  
**So that** I don't need to create a new password

**Acceptance Criteria:**
- [ ] Facebook login works
- [ ] Instagram login works (via Facebook)
- [ ] TikTok login works
- [ ] Apple login works
- [ ] User is redirected to main app after login
- [ ] Session persists across browser refreshes

**Technical Notes:** Uses NextAuth.js (Auth.js v5) with Vercel Postgres for session storage. Native providers for TikTok and Instagram.

---

### US-1.2: First-Time Consent
**As a** first-time user  
**I want to** understand how my images will be used  
**So that** I can make an informed decision before uploading

**Acceptance Criteria:**
- [ ] Modal appears on first visit after login
- [ ] Clear explanation of image usage (processing, not stored)
- [ ] "I confirm this is a photo of myself" checkbox
- [ ] Link to full Terms of Service
- [ ] Consent status saved to user profile
- [ ] Modal doesn't appear on subsequent visits

---

### US-1.3: Session Management
**As a** user  
**I want to** stay logged in across sessions  
**So that** I don't have to re-authenticate every visit

**Acceptance Criteria:**
- [ ] Session persists for 30 days
- [ ] Logout option available in settings
- [ ] Session invalidates on password change at OAuth provider

---

### US-1.4: Account Deletion
**As a** user  
**I want to** delete my account and all data  
**So that** I can exercise my privacy rights

**Acceptance Criteria:**
- [ ] "Delete Account" option in settings
- [ ] Confirmation required before deletion
- [ ] All user data removed within 24 hours
- [ ] User receives confirmation email

---

## Epic 2: Image Upload (P0)

### US-2.1: Upload Reference Photo
**As a** user  
**I want to** upload a photo of myself  
**So that** the AI can generate try-on images

**Acceptance Criteria:**
- [ ] Drag-and-drop upload area
- [ ] Click to browse files option
- [ ] Accepts JPEG, PNG, WebP
- [ ] Maximum file size: 10MB
- [ ] Preview shown after upload
- [ ] Option to remove and re-upload

---

### US-2.2: Camera Capture
**As a** mobile user  
**I want to** take a photo with my camera  
**So that** I can quickly capture a reference image

**Acceptance Criteria:**
- [ ] Camera button opens device camera
- [ ] Works on iOS Safari and Android Chrome
- [ ] Captured image appears in preview
- [ ] Option to retake

---

### US-2.3: Upload Outfit Image
**As a** user  
**I want to** upload an image of an outfit  
**So that** I can see how it looks on me

**Acceptance Criteria:**
- [ ] Same upload UX as reference photo
- [ ] Accepts JPEG, PNG, WebP
- [ ] Preview with option to change
- [ ] Clear label distinguishing from reference photo

---

### US-2.4: Paste Outfit URL
**As a** user  
**I want to** paste a URL to an outfit image  
**So that** I don't need to download and re-upload

**Acceptance Criteria:**
- [ ] URL input field available
- [ ] System fetches and validates image
- [ ] Error message if URL invalid or inaccessible
- [ ] Preview shown after fetch

---

### US-2.5: Image Validation
**As a** user  
**I want to** know if my image is unsuitable  
**So that** I can fix it before wasting a generation

**Acceptance Criteria:**
- [ ] Check for minimum resolution (768x1024)
- [ ] Check for face detection (reference photo)
- [ ] Warn if image too dark or blurry
- [ ] Suggest fixes for common issues

---

## Epic 3: AI Generation (P0)

### US-3.1: Generate Try-On
**As a** user  
**I want to** generate a try-on image  
**So that** I can see how the outfit looks on me

**Acceptance Criteria:**
- [ ] "Generate" button clearly visible
- [ ] Button disabled if inputs incomplete
- [ ] Loading state shown during generation
- [ ] Result displayed when complete

---

### US-3.2: Generation Progress
**As a** user  
**I want to** see progress during generation  
**So that** I know how long to wait

**Acceptance Criteria:**
- [ ] Progress indicator (spinner or percentage)
- [ ] Estimated time remaining shown
- [ ] Can navigate away without losing job
- [ ] Notification when complete (if navigated away)

---

### US-3.3: View Result
**As a** user  
**I want to** view my generated try-on image  
**So that** I can evaluate the outfit

**Acceptance Criteria:**
- [ ] Full-screen or modal view
- [ ] Pinch-to-zoom on mobile
- [ ] Side-by-side comparison (original vs try-on)
- [ ] Options: Download, Share, Try Again

---

### US-3.4: Rate Limiting
**As a** user  
**I want to** know my daily limit status  
**So that** I can plan my try-ons

**Acceptance Criteria:**
- [ ] Display remaining generations (e.g., "3/5 today")
- [ ] Clear message when limit reached
- [ ] Reset time shown (e.g., "Resets in 5 hours")
- [ ] Counter resets at midnight UTC

---

### US-3.5: Generation Failure Recovery
**As a** user  
**I want to** retry if generation fails  
**So that** I don't lose my inputs

**Acceptance Criteria:**
- [ ] Clear error message on failure
- [ ] "Retry" button available
- [ ] Inputs preserved for retry
- [ ] Failed attempt doesn't count against limit

---

### US-3.6: Download Result
**As a** user  
**I want to** download my try-on image  
**So that** I can save it to my device

**Acceptance Criteria:**
- [ ] Download button on result view
- [ ] High-resolution image saved
- [ ] Filename includes date/outfit name
- [ ] Works on iOS (Save to Photos)

---

## Epic 4: Social Sharing (P0)

> **Full Technical Strategy**: [SHARING-STRATEGY.md](../architecture/SHARING-STRATEGY.md)

### US-4.1: Share via OS Share Sheet
**As a** user  
**I want to** share my try-on to any social platform  
**So that** my friends can see my potential outfit

**Acceptance Criteria:**
- [ ] "Share" button triggers Web Share API Level 2
- [ ] Share includes generated image file (from browser memory)
- [ ] Share includes trackable link to outfit
- [ ] Default share text with #TryOnHaul
- [ ] Works on iOS Safari, Android Chrome (native share sheet)
- [ ] User can select target app (Instagram, Facebook, TikTok, etc.)

**Technical Notes:** 
- Uses `navigator.share({ files: [imageBlob] })` - no server storage needed
- Image shared directly from browser memory after generation

---

### US-4.2: Share Fallback (Desktop/Unsupported)
**As a** user on desktop or unsupported browser  
**I want to** share my try-on  
**So that** I can still post to social media

**Acceptance Criteria:**
- [ ] Fallback UI shown when Web Share API unavailable
- [ ] "Download Image" button saves image to device
- [ ] "Copy Link" button copies trackable outfit link
- [ ] Instructions for manual sharing shown
- [ ] Works on all browsers (Chrome, Firefox, Safari desktop)

---

### US-4.3: Branded Share Card
**As a** user  
**I want to** share a nicely formatted image  
**So that** my post looks professional

**Acceptance Criteria:**
- [ ] Generated image includes subtle Try on Haul branding
- [ ] Share card optimized for social media dimensions
- [ ] Branding doesn't obscure the outfit
- [ ] Optional: outfit link QR code embedded

**Technical Notes:** 
- Share card generated client-side using Canvas API
- No server-side image storage required

---

### US-4.4: Trackable Share Links
**As a** system  
**I want to** include trackable links in shares  
**So that** we can attribute affiliate conversions

**Acceptance Criteria:**
- [ ] Each share includes short link (e.g., tryonhaul.co/abc123)
- [ ] Link redirects to original product page
- [ ] Click events tracked
- [ ] User attribution preserved

---

### US-4.5: Copy Share Link
**As a** user  
**I want to** copy a link to share anywhere  
**So that** I can share on platforms or in messages

**Acceptance Criteria:**
- [ ] "Copy Link" button available
- [ ] Link copied to clipboard
- [ ] Toast confirmation shown
- [ ] Link includes tracking parameter

---

## Epic 5: Scene Compositing (P0)

> **Note**: Full festival scene compositing is in MVP scope per stakeholder decision.

### US-5.1: Browse Scenes
**As a** user  
**I want to** browse festival scene options  
**So that** I can choose a background for my try-on

**Acceptance Criteria:**
- [ ] Grid of scene thumbnails
- [ ] Categories: Desert, Main Stage, Forest, Neon, Crowd
- [ ] Select one (or none for no background change)
- [ ] Selected scene highlighted

**Technical Notes:** Scene compositing may require FASHN API (has native scene support) or additional AI pipeline

---

### US-5.2: Scene Preview
**As a** user  
**I want to** preview a scene before generating  
**So that** I know what the result will look like

**Acceptance Criteria:**
- [ ] Larger preview on scene selection
- [ ] Description of scene (e.g., "Burning Man sunrise")
- [ ] Option to deselect (use original background)

---

### US-5.3: Scene Application
**As a** user  
**I want to** generate try-on with selected scene  
**So that** I see myself at the festival

**Acceptance Criteria:**
- [ ] Scene passed to generation API
- [ ] Background replaced/composited in result
- [ ] Face/body preserved from reference
- [ ] Natural lighting blend

**Technical Notes:** 
- FASHN API supports native scene/background integration
- Alternative: Post-processing compositing with separate model
- Consider A/B testing quality between approaches

---

## Epic 6: Affiliate Tracking (P1)

### US-5.1: Trackable Outfit Links
**As a** system  
**I want to** generate unique trackable links  
**So that** we can attribute conversions

**Acceptance Criteria:**
- [ ] Each share generates unique short link
- [ ] Link redirects to original product
- [ ] Click events recorded
- [ ] User ID associated (for attribution)

---

### US-5.2: View Click Stats
**As a** user (future: power users)  
**I want to** see how many people clicked my links  
**So that** I can feel engaged with the platform

**Acceptance Criteria:**
- [ ] Click count visible on past shares
- [ ] Basic analytics in user dashboard
- [ ] (MVP: Internal metrics only; user-facing post-MVP)

---

### US-5.3: Conversion Webhook
**As a** partner retailer  
**I want to** notify Try on Haul of conversions  
**So that** attribution can be tracked

**Acceptance Criteria:**
- [ ] Webhook endpoint for conversion events
- [ ] Match conversion to click via tracking ID
- [ ] Store conversion data for analytics

---

### US-5.4: Affiliate Dashboard (Post-MVP)
**As a** partner retailer  
**I want to** see my try-on performance  
**So that** I can measure ROI

**Acceptance Criteria:**
- [ ] (Post-MVP: Partner portal with metrics)

---

## Epic 7: Partner Integration (P2)

### US-7.1: Shopify Product URL
**As a** user  
**I want to** paste a partner Shopify store URL  
**So that** the outfit image is auto-extracted

**Acceptance Criteria:**
- [ ] Detect Shopify URL format
- [ ] Query Storefront API for product images
- [ ] Display product image options
- [ ] Auto-fill product link for affiliate tracking

---

### US-7.2: Partner Store Badge
**As a** user  
**I want to** know which stores are partners  
**So that** I can trust the integration works

**Acceptance Criteria:**
- [ ] "Partner Store" badge on supported URLs
- [ ] List of partner stores in app
- [ ] Non-partner URL still works (manual upload)

---

### US-7.3: Partner Onboarding
**As a** retailer  
**I want to** join the Try on Haul partner program  
**So that** my products can be tried on

**Acceptance Criteria:**
- [ ] Partner signup form
- [ ] Shopify app installation guide
- [ ] Token sharing process
- [ ] Confirmation of partnership

---

### US-7.4: Partner Widget (Post-MVP)
**As a** partner retailer  
**I want to** embed Try on Haul on my site  
**So that** customers can try on without leaving

**Acceptance Criteria:**
- [ ] (Post-MVP: Embeddable widget)

---

## Sprint Allocation (8-12 Week Plan)

| Sprint | Duration | Epics | Focus |
|--------|----------|-------|-------|
| 1 | 2 weeks | E1, E2 (partial) | Auth + Upload |
| 2 | 2 weeks | E2 (complete), E3 (partial) | Upload + Generation Start |
| 3 | 2 weeks | E3 (complete), E5 | Generation + Scene Compositing |
| 4 | 2 weeks | E4 | Social Sharing |
| 5 | 2 weeks | E6 | Affiliate Tracking |
| 6 | 2 weeks | E7 (partial), Polish | Partner Integration + QA |

---

## Out of Scope for MVP

1. User gallery (saved try-ons)
2. Outfit wishlist
3. Multi-garment try-on (top + bottom combo)
4. Video try-on
5. AR live preview
6. Partner dashboard
7. Multi-language support
8. Dark mode (nice-to-have if time)

---

## Change Log

| Date | Change | Author |
|------|--------|--------|
| 2026-02-09 | Initial MVP user stories | AI Agent |

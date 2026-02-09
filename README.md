# Try on Haul

**See yourself at the festival—before you buy the outfit.**

Try on Haul is a virtual try-on app for festival fashion. Upload a selfie, add an outfit, pick a scene, and AI generates you wearing it in seconds. Share directly to Instagram, TikTok, and beyond.

---

## 📖 Start Here: Find Your Document

| You Are... | Start With | What You'll Learn |
|------------|------------|-------------------|
| 👔 **CEO / Founder** | [CEO Summary](summaries/CEO-SUMMARY.md) | Business overview, market opportunity, go-to-market, key metrics |
| 👩‍💻 **CTO / Technical Lead** | [CTO Summary](summaries/CTO-SUMMARY.md) | Tech stack, architecture, costs, development roadmap |
| 📈 **Investor (Quick Look)** | [VC Pitch](summaries/VC-PITCH.md) | Investment thesis, market size, financials, the ask |
| 🔬 **Investor (Due Diligence)** | [Investor Deep Dive](summaries/INVESTOR-DEEP-DIVE.md) | Detailed financials, unit economics, risks, competitive analysis |
| 📣 **Marketing Team** | [Marketing Summary](summaries/MARKETING-SUMMARY.md) | Positioning, messaging, channels, campaigns, influencer strategy |
| 🛍️ **Partner Retailer** | [Partner Guide](summaries/PARTNER-GUIDE.md) | Integration options, affiliate program, data you'll receive |
| 🎨 **Designer** | [Design Guidelines](summaries/DESIGN-GUIDELINES.md) | Brand identity, colors, typography, components, accessibility |
| 👩‍💻 **Developer** | [Developer Guide](summaries/DEVELOPER-GUIDE.md) | Setup, architecture, code standards, contributing |
| 📰 **Press / Media** | [Press Kit](summaries/PRESS-KIT.md) | One-liner, boilerplate, key facts, media assets |
| 👤 **End User** | [User FAQ](summaries/USER-FAQ.md) | How it works, tips, troubleshooting |

---

## 🎯 What is Try on Haul?

### The Problem
Online shoppers can't try on clothes before buying. Result: 30-40% return rate, cart abandonment, frustration.

### The Solution
AI-powered virtual try-on that shows you wearing any outfit—in under 90 seconds.

### The Twist
We're not just a utility. We're a **content creation engine**. Every try-on is designed to be shared, driving viral growth and affiliate revenue.

---

## ✨ How It Works

```
1. Upload a selfie          →  Your photo, your body
2. Add an outfit            →  Paste a URL or upload an image  
3. Pick a scene (optional)  →  Burning Man, Coachella, EDC...
4. Generate                 →  AI magic in ~60-90 seconds
5. Share                    →  Instagram, TikTok, Facebook, anywhere
```

---

## 🎪 Why Festival Fashion?

| Factor | Why It Matters |
|--------|----------------|
| **Highly social** | Festival-goers share everything—built-in viral loop |
| **High engagement** | Planning outfits is part of the experience |
| **Premium pricing** | Higher order values = better affiliate commissions |
| **Clear niche** | Easier marketing, less competition |
| **Seasonal moments** | Coachella, Burning Man, EDC = marketing efficiency |

---

## 💰 Business Model

| Revenue Stream | How It Works | Timeline |
|----------------|--------------|----------|
| **Affiliate** | Earn commission when users buy outfits through our links | Now |
| **Partner Widget** | Retailers embed try-on on their sites (B2B) | Post-MVP |
| **Premium** | Unlimited try-ons, saved wardrobe, priority processing | Future |

---

## 🛠️ Tech Stack

| Component | Technology |
|-----------|------------|
| Frontend | Next.js 16 (PWA) |
| Auth | NextAuth.js (FB, IG, TikTok, Apple) |
| AI | fal.ai (CatVTON) + FASHN backup |
| Background Jobs | Inngest |
| Hosting | Vercel |
| Storage | Vercel Blob (ephemeral, 1h TTL) |

**Detailed architecture**: [Technical Documentation](INDEX.md)

---

## 📅 Timeline

| Phase | Weeks | Milestone |
|-------|-------|-----------|
| Foundation | 1-4 | Auth, upload, UI shell |
| Core Product | 5-8 | AI generation, sharing |
| Launch | 9-12 | Polish, partners, go live |

**Full roadmap**: [Implementation Plan](docs/plans/IMPLEMENTATION-PLAN.md)

---

## 📁 Documentation Structure

```
tryonhaul/
├── README.md                    ← You are here
├── INDEX.md                     # Technical documentation hub
├── AGENTS.md                    # AI agent guidelines
│
├── summaries/                   # Audience-specific summaries
│   ├── CEO-SUMMARY.md
│   ├── CTO-SUMMARY.md
│   ├── VC-PITCH.md
│   ├── INVESTOR-DEEP-DIVE.md
│   ├── MARKETING-SUMMARY.md
│   ├── PARTNER-GUIDE.md
│   ├── DESIGN-GUIDELINES.md
│   ├── DEVELOPER-GUIDE.md
│   ├── PRESS-KIT.md
│   └── USER-FAQ.md
│
└── docs/                        # Detailed technical docs
    ├── architecture/
    ├── user-stories/
    ├── research/
    ├── standards/
    └── plans/
```

---

## 🚀 Quick Start (Developers)

```bash
# Clone and install
git clone https://github.com/grant-vine/tryonhaul.git
cd tryonhaul
pnpm install

# Set up environment
cp .env.example .env.local
# Add your API keys...

# Run development server
pnpm dev
```

**Full setup guide**: [Developer Guide](summaries/DEVELOPER-GUIDE.md)

---

## 📞 Contact

| Purpose | Contact |
|---------|---------|
| General inquiries | hello@tryonhaul.com |
| Partnership opportunities | partners@tryonhaul.com |
| Press & media | press@tryonhaul.com |
| Technical questions | dev@tryonhaul.com |

---

## 📜 License

*[License TBD]*

---

<p align="center">
  <strong>Try on Haul</strong><br>
  See yourself at the festival—before you buy the outfit.
</p>

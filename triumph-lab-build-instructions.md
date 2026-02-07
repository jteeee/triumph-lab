# Triumph Lab — Build Instructions

## Overview
Internal decision hub for Triumph Capital Management's team to review, discuss, and approve ideas across branding, portal design, technology, and operations. Think of it as a living workspace where in-progress work gets shared, discussed, and moved to "approved."

**Primary users:** JT (COO), Derek (CEO), Brandon (CCO), and select team members
**Tone:** Professional but not corporate — this is an internal workshop, not a client-facing site
**Tech stack preference:** Next.js + Tailwind + Supabase (aligns with existing infrastructure JT is building)

---

## Site Name
**Triumph Lab** (domain could be lab.triumphcapitalmanagement.com or triumphlab.vercel.app for now)

---

## Navigation Structure

```
Triumph Lab
├── Dashboard (home — activity feed, recent updates, items needing review)
├── Brand & Design
│   ├── Managed Program Identity (logo concepts, palettes, typography)
│   ├── Portal UI Direction (mockups, theme decisions)
│   └── Marketing Materials (factsheets, pitch decks, templates)
├── Portal & UX
│   ├── Advisor Dashboard (wireframes, feature specs)
│   ├── Client Portal (AI chat interface, portfolio views)
│   └── Login & Onboarding (flows, screen designs)
├── Tech & Integrations
│   ├── Black Diamond API (architecture, endpoints, status)
│   ├── Supabase Schema (database design, RLS policies)
│   ├── Custodian Connectivity (Schwab, Fidelity API status)
│   └── AI Features (conversational portfolio, natural language queries)
├── Compliance & Ops
│   ├── Surveillance Tools (trade monitoring, Excel automation)
│   ├── TTAMP Committee (meeting docs, model decisions)
│   └── Reporting Templates (quarterly, regulatory)
├── Models & Investments
│   ├── Model Lineup (allocation decisions, risk tiers)
│   └── Factsheet Design (templates, data presentation)
└── Team Decisions
    ├── Active (items needing votes/input)
    ├── In Review (under discussion)
    └── Approved (decided, ready for implementation)
```

---

## Core Features

### 1. Decision Cards
Each item in the hub is a "decision card" with:
- **Title** — what's being decided
- **Category** — which section it belongs to
- **Status** — `Draft` | `Proposed` | `In Review` | `Approved` | `Archived`
- **Owner** — who posted it
- **Description** — context and rationale
- **Attachments** — embedded HTML previews, images, PDFs, links
- **Comments/Reactions** — team members can leave feedback
- **Vote** — simple thumbs up/down or option selection for multi-choice decisions
- **Created/Updated timestamps**

### 2. Dashboard Home
- Recent activity feed (new items, new comments, status changes)
- "Needs Your Input" section — items in `Proposed` or `In Review` filtered to current user
- Quick stats: total items by status, items by category
- Pinned/featured decisions at top

### 3. Embedded Previews
- HTML files (like the brand concepts) should render inline as iframes
- Image uploads displayed in lightbox/gallery
- Support for external links with preview cards

### 4. Authentication
- Simple auth via Supabase — email/password for now
- Team members: JT, Derek, Brandon, Hannah, Jamar, Meghan
- Role levels: Admin (JT) | Reviewer (Derek, Brandon) | Viewer (others)

### 5. Notifications (nice to have)
- Email digest when new items are posted or status changes
- Or just a "what's new since last visit" indicator on dashboard

---

## Design Direction

### Theme
- **Light theme primary** (this is a working tool, not a showcase)
- Use the Triumph palette as subtle accents:
  - Background: `#F8F5F0` (warm cream)
  - Cards: `#FFFFFF` with `#E5E0D8` borders
  - Nav/Header: `#1A2744` (deep navy)
  - Accent: `#C4955A` (bronze) for active states, links, CTAs
  - Text: `#1A2744` primary, `#8B7E6E` secondary
- Typography: Inter or DM Sans for UI, Playfair Display for the logo/branding only

### Layout
- Left sidebar navigation (collapsible)
- Main content area with card grid or list view toggle
- Clean, minimal — this should feel fast and functional, not decorative

---

## Data Model (Supabase)

### Tables

```sql
-- Team members / auth handled by Supabase Auth

-- Categories
categories (
  id uuid primary key,
  name text not null,           -- "Brand & Design"
  slug text unique not null,    -- "brand-design"
  icon text,                    -- emoji or icon name
  sort_order int,
  parent_id uuid references categories(id)  -- for subcategories
)

-- Decision items
decisions (
  id uuid primary key,
  title text not null,
  description text,
  category_id uuid references categories(id),
  status text default 'draft',  -- draft, proposed, in_review, approved, archived
  owner_id uuid references auth.users(id),
  created_at timestamptz default now(),
  updated_at timestamptz default now(),
  pinned boolean default false
)

-- Attachments (HTML files, images, links)
attachments (
  id uuid primary key,
  decision_id uuid references decisions(id) on delete cascade,
  type text not null,           -- 'html', 'image', 'link', 'pdf'
  url text,                     -- storage URL or external link
  filename text,
  sort_order int,
  created_at timestamptz default now()
)

-- Comments
comments (
  id uuid primary key,
  decision_id uuid references decisions(id) on delete cascade,
  user_id uuid references auth.users(id),
  content text not null,
  created_at timestamptz default now()
)

-- Votes (for multi-option decisions)
votes (
  id uuid primary key,
  decision_id uuid references decisions(id) on delete cascade,
  user_id uuid references auth.users(id),
  value text,                   -- 'up', 'down', or option label like 'Concept 1'
  created_at timestamptz default now(),
  unique(decision_id, user_id)  -- one vote per person per decision
)
```

---

## File Storage
- Use Supabase Storage for uploaded HTML files, images, PDFs
- Bucket: `lab-attachments`
- HTML files served as iframe embeds within decision cards
- Images displayed inline with lightbox on click

---

## Initial Seed Content

### Category: Brand & Design → Managed Program Identity
1. **"Logo Direction — Heritage Elevated (Concept 1)"**
   - Status: In Review
   - Attach: triumph-concept1-expanded.html
   - Description: "Shield + TC monogram with bronze accent, decorative flourish from parent logo. Includes dark and light variations."

2. **"All Three Logo Concepts — Initial Exploration"**
   - Status: In Review
   - Attach: triumph-managed-program-concepts.html
   - Description: "Three directions: Heritage Elevated, Modern Authority, Refined Distinction"

3. **"TC Monogram Integration"**
   - Status: In Review
   - Attach: triumph-tc-integration.html
   - Description: "How the existing TC chrome logo adapts into each concept direction"

4. **"Light vs Dark Theme Decision"**
   - Status: Proposed
   - Description: "Recommendation: Light theme for working portal, dark theme for login/marketing. Industry standard is light (Orion, AssetMark, GeoWealth all use light). See expanded Concept 1 for both mockups."
   - Vote options: "Light primary + Dark login" | "Full light" | "Full dark"

---

## Deployment
- Vercel for hosting (connects to GitHub repo)
- Supabase project for database + auth + storage
- Custom domain later: lab.triumphcapitalmanagement.com

---

## Priority Order
1. **V1 — Static shell**: Nav, categories, card layout, ability to embed HTML files inline. No auth, no comments. Just get content visible and shareable.
2. **V2 — Auth + comments**: Supabase auth, team login, commenting on decisions.
3. **V3 — Voting + status management**: Vote on items, change status, activity feed.
4. **V4 — Notifications + polish**: Email digests, search, mobile responsive refinement.

Start with V1 — that's all you need to share the brand concepts with Derek and Brandon today.

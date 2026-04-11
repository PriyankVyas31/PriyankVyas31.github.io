# Low-Level Design (LLD) — Portfolio + Booking + Payment

## 1. File Structure

### Frontend (GitHub Pages)
```
portfolio/
├── index.html          # Website + booking modal + Razorpay integration
├── data.json           # Editable content (loaded dynamically)
├── admin.html          # Admin panel (password-protected)
├── images/
│   ├── personal_photo/ # 7 personal photos
│   └── comanies_logo/  # 18 company logos
└── docs/               # Documentation
```

### Backend (GCP Cloud Run)
```
booking-server/
├── server.js           # Express server (2 API endpoints)
├── package.json        # Dependencies: express, razorpay, cors, nodemailer
├── Dockerfile          # Docker build recipe
├── .dockerignore       # Exclude node_modules from container
└── .gitignore          # Exclude .env from Git
```
│   ├── Google Fonts preconnect + import
│   └── <style> — All CSS (~800 lines)
│
├── <body>
│   ├── <nav>           — Fixed navbar with hamburger menu
│   ├── <section#hero>  — Hero with name, tagline, CTA buttons, photo gallery
│   ├── <section#about> — Bio text + profile photo + stats grid
│   ├── <div.companies> — Scrolling company logos marquee
│   ├── <section#skills> — 6 skill cards in a responsive grid
│   ├── <section#experience> — Timeline with 3 job entries
│   ├── <section#education>  — Photo strip + 3 education cards
│   ├── <section#sessions>   — 6 Topmate booking cards
│   ├── <section#testimonials> — Scrolling testimonial marquee
│   ├── <section#youtube>    — Podcast photo + YouTube embed
│   ├── <section#contact>    — 4 contact cards (LinkedIn, Email, Topmate, GitHub)
│   ├── <footer>             — Footer links + copyright
│   └── <script>             — All JavaScript (~30 lines)
│
└── images/
    ├── personal_photo/ (7 files, ~500KB each)
    └── comanies_logo/ (18 files, ~20-120KB each)
```

---

## 2. CSS Architecture

### Design System (CSS Variables)

```css
:root {
    --bg: #0a0a0f;           /* Page background (near black) */
    --bg-card: #12121a;      /* Card background */
    --accent: #3b82f6;       /* Primary blue */
    --accent-glow: rgba(59, 130, 246, 0.15);  /* Blue glow */
    --text: #e4e4e7;         /* Body text */
    --text-dim: #a1a1aa;     /* Secondary text */
    --text-bright: #fafafa;  /* Headings */
    --border: #27272a;       /* Card borders */
    --microsoft: #00a4ef;    /* Company brand colors */
    --cisco: #049fd9;
    --tcs: #0072c6;
}
```

### CSS Sections (in order)

| Section | Lines (approx) | Purpose |
|---------|---------------|---------|
| Reset + Variables | 1-25 | Box-sizing, CSS custom properties |
| Navbar | 35-85 | Fixed top bar, links, hamburger |
| Hero | 90-165 | Full-height intro with gradient |
| Buttons | 155-185 | `.btn-primary`, `.btn-outline` |
| Sections | 190-215 | Common section/container styles |
| About | 215-245 | Two-column grid, stat cards |
| Skills | 245-300 | Card grid with hover effects |
| Experience | 300-380 | Timeline with vertical line + dots |
| Education | 380-420 | Card grid with grade badges |
| Sessions | 420-520 | Booking cards with pricing |
| Testimonials | 525-590 | Scrolling marquee cards |
| Contact | 590-635 | Icon cards with hover |
| Footer | 635-660 | Centered links |
| Photo Gallery | 665-690 | Flexbox image row |
| Company Marquee | 690-760 | Infinite scroll animation |
| YouTube | 760-775 | Centered iframe |
| Responsive | 775-810 | @media breakpoints |

### Responsive Breakpoints

| Breakpoint | Target | Key Changes |
|-----------|--------|-------------|
| `≤ 768px` | Tablets | Hamburger menu, single-column grids, smaller photos |
| `≤ 480px` | Phones | Stacked CTAs, compact stats, smaller gallery |

---

## 3. HTML Components

### 3.1 Navbar

```
<nav> (position: fixed, z-index: 100, backdrop blur)
  └── .nav-inner (max-width: 1100px, flexbox)
      ├── .nav-logo — "PV."
      ├── ul.nav-links — 7 anchor links
      └── button.hamburger — 3 span bars (hidden on desktop)
```

**Mobile behavior:** `.nav-links` is hidden by default. JavaScript toggles `.open` class which makes it a dropdown.

### 3.2 Hero Section

```
<section.hero> (min-height: 100vh, centered)
  └── .hero-content (max-width: 760px, z-index: 1)
      ├── .hero-badge — "Available for collaboration" with pulsing dot
      ├── h1 — Name with gradient text
      ├── p — Tagline
      ├── .hero-cta — Two buttons
      └── .photo-gallery — 3 images (220x290px, border-radius: 16px)
```

**::before pseudo-element** creates a radial gradient glow behind the content (800x800px, pointer-events: none).

### 3.3 About Section

```
<section#about>
  └── .about-grid (CSS Grid: 1fr 1fr, gap: 3rem)
      ├── .about-text — 4 paragraphs
      └── Column
          ├── Profile image (max-width: 320px, aspect-ratio: 3/4)
          └── .stats-grid (2x2 grid)
              ├── Stat: "5+" Years
              ├── Stat: "25+" Companies
              ├── Stat: "C/C++"
              └── Stat: "Azure"
```

### 3.4 Company Logos Marquee

```
<div.companies-section>
  └── .marquee-wrapper (overflow: hidden, gradient mask on edges)
      └── .marquee-track (flexbox, animation: marquee 30s linear infinite)
          ├── .company-chip × 18 (original set)
          │   ├── img (40x40px, local JPG)
          │   └── span (company name)
          └── .company-chip × 18 (duplicate for seamless loop)
```

**How the infinite scroll works:**
1. All 18 chips are laid out in a row
2. The same 18 chips are duplicated immediately after
3. CSS `@keyframes marquee` translates the track from `0` to `-50%`
4. When it hits -50%, it snaps back to 0 — the duplicate makes it look seamless
5. `animation: marquee 30s linear infinite` runs forever

The same technique is used for testimonials (with `40s` duration for slower scroll since cards are wider).

### 3.5 Experience Timeline

```
<section#experience>
  └── .timeline (padding-left: 2rem, relative)
      │ ::before — Vertical gradient line (2px wide, blue→gray)
      │
      ├── .timeline-item (3× — Microsoft, Cisco, TCS)
      │   │ ::before — Circle dot (14px, blue border)
      │   │
      │   ├── .timeline-header (flexbox)
      │   │   ├── .company-logo > img (44x44px, actual logo)
      │   │   └── div
      │   │       ├── .timeline-role — Job title
      │   │       └── .timeline-company — Company name
      │   ├── .timeline-meta — Date range + location
      │   ├── ul.timeline-desc — Bullet points (custom ▸ marker)
      │   └── .skill-tags — Blue pill tags
```

### 3.6 Session Cards

```
<section#sessions>
  └── .sessions-grid (CSS Grid: auto-fit, minmax 300px)
      └── a.session-card × 6 (links to topmate.io/priyank_vyas)
          ├── .session-popular — "Popular" badge (absolute positioned)
          ├── .session-type — Icon + "Video meeting · 60 mins" + rating
          ├── h3 — Session title
          ├── .session-subtitle — Category
          ├── .session-pricing — Price + strikethrough original
          └── .session-book — "Book Now →" button
```

Each card is an `<a>` tag linking to Topmate — clicking anywhere on the card opens Topmate.

### 3.7 Testimonial Marquee

Same pattern as company logos but with larger cards:

```
.testimonial-track (animation: testimonialScroll 40s linear infinite)
  ├── .testimonial-card × 8 (min-width: 340px)
  │   ├── .testimonial-stars — "★★★★★"
  │   ├── blockquote — Review text (italic)
  │   └── .testimonial-author
  │       ├── .author-avatar — Initials circle (38px)
  │       └── div
  │           ├── .author-name
  │           └── .author-date
  └── .testimonial-card × 8 (duplicate for loop)
```

### 3.8 YouTube Embed

```
<section#youtube>
  └── Flexbox row (wraps on mobile)
      ├── img — Podcast studio photo (300x220px)
      └── .youtube-embed
          └── iframe (YouTube embed, 16:9 aspect ratio)
```

---

## 4. JavaScript

The entire JS is ~30 lines with 3 functions:

### 4.1 Mobile Nav Toggle

```javascript
document.getElementById('hamburger').addEventListener('click', () => {
    document.getElementById('navLinks').classList.toggle('open');
});
```
Toggles the `.open` class on the nav links list. CSS handles the dropdown animation.

### 4.2 Close Nav on Link Click

```javascript
document.querySelectorAll('.nav-links a').forEach(link => {
    link.addEventListener('click', () => {
        document.getElementById('navLinks').classList.remove('open');
    });
});
```
When user clicks a section link on mobile, the menu closes.

### 4.3 Scroll Animations (Intersection Observer)

```javascript
const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
        if (entry.isIntersecting) {
            entry.target.classList.add('visible');
        }
    });
}, { threshold: 0.1 });

document.querySelectorAll('.fade-up').forEach(el => observer.observe(el));
```

**How it works:**
1. Elements with class `.fade-up` start with `opacity: 0; transform: translateY(30px)`
2. When 10% of the element enters the viewport, the observer adds class `.visible`
3. CSS transition animates to `opacity: 1; transform: translateY(0)` over 0.6s

---

## 5. Image Strategy

### Personal Photos
- Format: JPEG (good compression for photos)
- Size: ~300-780KB each (acceptable for portfolio)
- Displayed at: 220×290px (hero), 320px wide (about), 300×220px (podcast)
- `loading="lazy"` on non-hero images

### Company Logos
- Format: JPG
- Size: 5-122KB each
- Displayed at: 40×40px with `object-fit: contain`
- White background (`background: #fff; padding: 1px`) for dark theme visibility

---

## 6. Animation Details

| Animation | Type | Duration | Trigger |
|-----------|------|----------|---------|
| Hero glow pulse | CSS `@keyframes pulse` | 2s infinite | Always |
| Fade-up on scroll | CSS transition + JS Observer | 0.6s | Scroll into view |
| Company marquee | CSS `@keyframes marquee` | 30s infinite | Always |
| Testimonial marquee | CSS `@keyframes testimonialScroll` | 40s infinite | Always |
| Card hover lift | CSS `transform: translateY(-4px)` | 0.3s | Hover |
| Button glow | CSS `box-shadow` transition | 0.25s | Hover |

---

## 7. Performance Characteristics

| Metric | Value | Notes |
|--------|-------|-------|
| HTML file size | ~80KB | Single file, all inline + booking modal |
| Total images | ~8MB | 7 photos + 18 logos |
| HTTP requests | ~30 | 1 HTML + 1 JSON + 25 images + 1 font + 1 iframe + 1 Razorpay |
| JavaScript | ~250 lines | Theme + nav + data loading + booking flow + Razorpay |
| CSS | ~950 lines | All inline, including booking modal styles |
| Time to Interactive | < 2s | Static HTML, JS deferred |

---

## 8. Backend Server (server.js) — Detailed

### 8.1 Server Setup

```javascript
// Libraries used:
const express = require('express');    // Web server framework
const Razorpay = require('razorpay');  // Payment SDK
const crypto = require('crypto');       // For HMAC signature verification
const cors = require('cors');           // Cross-origin security
const nodemailer = require('nodemailer'); // Email sending
```

**CORS Configuration:**
Only `priyankvyas31.github.io` can call our backend. Any other website trying to call our API gets blocked.

### 8.2 Endpoint: POST /api/create-order

**Purpose:** Create a Razorpay payment order

**Input:**
```json
{ "amount": 500, "session_title": "Mock Interview", "customer_name": "John", "customer_email": "john@email.com" }
```

**What happens:**
1. Validate amount ≥ ₹1
2. Call `razorpay.orders.create()` with amount in paise (₹500 = 50000 paise)
3. Razorpay returns an `order_id`
4. Send back: `{ id, amount, currency, key_id }`

**Output:**
```json
{ "id": "order_ABC123", "amount": 50000, "currency": "INR", "key_id": "rzp_test_xxx" }
```

### 8.3 Endpoint: POST /api/verify-payment

**Purpose:** Verify payment was real + send emails + generate meeting link

**Input:**
```json
{
    "razorpay_order_id": "order_ABC123",
    "razorpay_payment_id": "pay_XYZ789",
    "razorpay_signature": "hmac_signature",
    "booking": { "session": "Mock Interview", "date": "2026-04-15", "time": "10:00 AM", "duration": "60 mins", "name": "John", "email": "john@email.com", "phone": "9876543210", "topic": "System design" }
}
```

**What happens:**
1. **Verify signature:**
   ```
   expected = HMAC-SHA256(order_id + "|" + payment_id, KEY_SECRET)
   if (expected !== received_signature) → REJECT (fake payment)
   ```
2. **Fetch payment details** from Razorpay (method, VPA, etc.)
3. **Generate booking reference:** `PV` + timestamp in base36
4. **Generate meeting link:** `meet.jit.si/pv-{ref}-{random}`
5. **Send owner email** (all booking + payment + customer details)
6. **Send customer email** (booking details + meeting link button)
7. **Return:** `{ verified: true, bookingRef, paymentId, meetLink }`

### 8.4 Email Templates

**Customer email structure:**
```
┌─────────────────────────────────────┐
│ ✅ Booking Confirmed!               │ (blue-purple gradient header)
├─────────────────────────────────────┤
│ Hi {name},                          │
│                                     │
│ ┌─────────────────────────────┐     │
│ │ 📋 Session: Mock Interview  │     │ (blue info box)
│ │ 📅 Date: 2026-04-15        │     │
│ │ 🕐 Time: 10:00 AM (IST)   │     │
│ │ 💰 Paid: ₹500              │     │
│ │ 🔖 Ref: PV1ABC             │     │
│ └─────────────────────────────┘     │
│                                     │
│       [🎥 Join Meeting]             │ (black button → Jitsi link)
│                                     │
│ Payment ID: pay_XYZ789              │
└─────────────────────────────────────┘
```

---

## 9. Booking Modal (Frontend) — Detailed

### 9.1 Modal Structure

```
<div.book-overlay>                    (full-screen dark overlay)
  └── <div.book-modal>                (white card, max 500px)
      ├── <div.book-modal-head>       (sticky header with title + close)
      └── <div.book-body>             (dynamically rendered by JS)
```

### 9.2 Four Steps (rendered by `renderBook()`)

**Step 1 — Calendar + Time:**
- Session title, type, price displayed at top
- 7-day date picker (scrollable, arrow buttons to go forward/back)
- Time slot grid: 9AM–6PM in 15-minute intervals
- Grid uses `auto-fill, minmax(95px, 1fr)` — responsive columns
- Selected date/time highlighted in black
- "Continue" button disabled until both date AND time selected

**Step 2 — User Details:**
- Shows selected date/time with "Change" button
- Form fields: Name*, Email*, Phone, Topic
- Validation: name required, email must contain @
- Data stored in `bk` object (persists between steps)

**Step 3 — Payment:**
- Order summary with price
- "Pay Now" button calls `startRzp()`
- Button shows loading states: "Creating order..." → "Verifying payment..."

**Step 4 — Success:**
- Green checkmark, "Booking Confirmed!"
- Shows: booking ref, payment ID, date, time
- "Join Meeting" button linking to Jitsi room
- "Done" button closes modal

### 9.3 State Management

All booking state is in one object:
```javascript
let bk = {
    step: 1,           // Current step (1-4)
    session: null,      // Selected session from data.json
    date: null,         // Selected date (YYYY-MM-DD)
    time: null,         // Selected time ("10:00 AM")
    offset: 0,          // Calendar offset (0 = today, 7 = next week)
    name: '',           // User's name
    email: '',          // User's email
    phone: '',          // User's phone
    topic: '',          // What they want to discuss
    bookingRef: '',     // Set after payment (from backend)
    paymentId: '',      // Set after payment (from Razorpay)
    meetLink: ''        // Set after payment (from backend)
};
```

### 9.4 Razorpay Integration (Frontend Side)

```javascript
// 1. Load Razorpay SDK
<script src="https://checkout.razorpay.com/v1/checkout.js"></script>

// 2. Create order via our backend
const order = await fetch(API_BASE + '/api/create-order', { ... });

// 3. Open Razorpay checkout popup
const rzp = new Razorpay({
    key: order.key_id,      // Public key (safe in frontend)
    order_id: order.id,     // Order created by our backend
    handler: function(response) {
        // 4. After payment, verify via our backend
        fetch(API_BASE + '/api/verify-payment', {
            body: {
                razorpay_order_id: response.razorpay_order_id,
                razorpay_payment_id: response.razorpay_payment_id,
                razorpay_signature: response.razorpay_signature,
                booking: { ... }
            }
        });
    }
});
rzp.open();  // Shows the payment popup
```

---

## 10. Docker & Cloud Run — Detailed

### 10.1 Dockerfile Explained

```dockerfile
FROM node:18-alpine    # Start with lightweight Node.js 18 image (50MB)
WORKDIR /app           # Set working directory inside container
COPY package.json .    # Copy dependency list first (for caching)
RUN npm install --production  # Install only production dependencies
COPY server.js .       # Copy server code
EXPOSE 3000            # Tell Docker the app listens on port 3000
CMD ["node", "server.js"]  # Command to start the server
```

**Why copy package.json first?** Docker caches each step. If server.js changes but package.json doesn't, Docker reuses the cached `npm install` step — making builds faster.

### 10.2 Cloud Run Configuration

| Setting | Value | Why |
|---------|-------|-----|
| Region | `asia-south1` (Mumbai) | Closest to Indian users |
| Memory | 256MB | Enough for Node.js |
| Port | 3000 | What our Express app listens on |
| Auth | Unauthenticated allowed | Public API (our frontend calls it) |
| Scaling | 0 to 100 instances | Scales to zero when no traffic (free) |
| Timeout | 300s | Enough for payment verification |

### 10.3 Deployment Command

```bash
gcloud run deploy booking-server \
  --source .                           # Build from current directory
  --region asia-south1                 # Mumbai datacenter
  --platform managed                   # Google manages the infrastructure
  --allow-unauthenticated             # Anyone can call our API
  --port 3000                          # App listens on port 3000
  --memory 256Mi                       # 256MB RAM
  --set-env-vars "KEY=VALUE,..."       # Set secret environment variables
```

This single command:
1. Uploads source code to GCP
2. Reads the Dockerfile
3. Builds a container image using Cloud Build
4. Pushes image to Artifact Registry
5. Creates a new Cloud Run revision
6. Routes all traffic to the new revision
7. Returns the service URL

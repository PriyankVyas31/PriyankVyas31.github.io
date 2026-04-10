# Low-Level Design (LLD) — Portfolio Website

## 1. File Structure

The entire website is a **single HTML file** (`index.html`) containing embedded CSS and JavaScript. No build tools, no bundler, no framework.

```
index.html
├── <head>
│   ├── Meta tags (charset, viewport)
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
| HTML file size | ~35KB | Single file, all inline |
| Total images | ~8MB | 7 photos + 18 logos |
| HTTP requests | ~28 | 1 HTML + 25 images + 1 font + 1 iframe |
| JavaScript | ~30 lines | No frameworks, no dependencies |
| CSS | ~800 lines | All inline, no external stylesheets |
| Time to Interactive | < 2s | Static HTML, no JS rendering |
| Lighthouse Score | 90+ expected | Static site, lazy loading, semantic HTML |

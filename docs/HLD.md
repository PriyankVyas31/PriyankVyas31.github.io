# High-Level Design (HLD) — Portfolio Website

## 1. Overview

A static, single-page portfolio website for Priyank Vyas that serves as a personal brand, mentorship booking platform, and professional showcase. The site is hosted on GitHub Pages and uses Topmate for session booking and payments.

---

## 2. Goals

| Goal | Description |
|------|------------|
| **Personal Branding** | Showcase experience, skills, and credibility (Microsoft, Cisco, 25+ interviews) |
| **Mentorship** | Allow users to book paid 1:1 sessions via Topmate |
| **Social Proof** | Display testimonials and company logos to build trust |
| **Accessibility** | Work on all devices — desktop, tablet, mobile |
| **Zero Maintenance** | No backend, no database, no server to manage |

---

## 3. Architecture

```
┌─────────────────────────────────────────────────┐
│                   USER (Browser)                │
│  Desktop / Tablet / Mobile                      │
└──────────────────────┬──────────────────────────┘
                       │ HTTPS
                       ▼
┌─────────────────────────────────────────────────┐
│              GitHub Pages (CDN)                  │
│  Serves: index.html + images/                   │
│  Domain: priyankvyas31.github.io                │
└──────────────────────┬──────────────────────────┘
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
   ┌──────────┐ ┌──────────┐ ┌──────────┐
   │ Google   │ │ Topmate  │ │ YouTube  │
   │ Fonts    │ │ Booking  │ │ Embed    │
   │ (CDN)    │ │ Platform │ │ (iframe) │
   └──────────┘ └──────────┘ └──────────┘
```

### Components

| Component | Role | Technology |
|-----------|------|-----------|
| **Frontend** | Renders entire website | HTML + CSS + JS (single file) |
| **Hosting** | Serves static files globally | GitHub Pages (free CDN) |
| **Booking** | Handles calendar, payment, emails | Topmate.io (external SaaS) |
| **Video** | Embeds podcast content | YouTube iframe |
| **Fonts** | Typography | Google Fonts API |
| **CMS/Admin** | Content management panel | admin.html + data.json + GitHub API |
| **Theme** | Dark/Light mode toggle | CSS variables + localStorage |
| **Version Control** | Code management and deployment | Git + GitHub |

---

## 4. User Flow

```
User visits site
    │
    ├── Scrolls through sections (About, Skills, Experience, Education)
    │
    ├── Sees company logos → builds trust
    │
    ├── Reads testimonials → builds confidence
    │
    ├── Clicks "Book a Session" card
    │       │
    │       └── Redirected to Topmate.io
    │               ├── Selects date/time
    │               ├── Fills form (name, email, phone)
    │               ├── Makes payment (UPI, cards, etc.)
    │               └── Gets confirmation email
    │
    ├── Watches YouTube podcast
    │
    └── Clicks Contact links (LinkedIn, Email, GitHub)
```

---

## 5. Data Flow

There is **no backend and no database**. All data is static HTML.

| Data | Storage | How it's updated |
|------|---------|-----------------|
| Personal info, experience, skills | Hardcoded in `index.html` | Edit HTML and push to GitHub |
| Dynamic content (about, sessions, testimonials) | `data.json` | Via admin panel or direct edit |
| Photos | `images/personal_photo/` folder | Add/replace files + push |
| Company logos | `images/comanies_logo/` folder | Add/replace files + push |
| Booking data | Managed entirely by Topmate | Topmate dashboard |
| Theme preference | Browser localStorage | User toggles on site |

---

## 6. Deployment Pipeline

```
Developer (VS Code)
    │
    ├── git add -A
    ├── git commit -m "message"
    └── git push origin main
            │
            ▼
    GitHub (main branch)
            │
            ▼
    GitHub Pages (auto-deploy)
            │
            ▼
    Live at priyankvyas31.github.io (1-2 min)
```

No CI/CD pipeline needed. GitHub Pages auto-builds on every push to `main`.

---

## 7. External Dependencies

| Service | Purpose | Cost | Failure Impact |
|---------|---------|------|---------------|
| GitHub Pages | Hosting | Free | Site goes down |
| Google Fonts | Typography | Free | Falls back to system fonts |
| Topmate.io | Booking + Payment | Free (they take commission) | Booking links broken |
| YouTube | Video embed | Free | Video section shows error |

---

## 8. Security

| Concern | Mitigation |
|---------|-----------|
| No user input on site | No XSS/injection risk |
| No backend | No server-side vulnerabilities |
| No database | No SQL injection, no data breach |
| HTTPS | Enforced by GitHub Pages |
| External links | All use `rel="noopener noreferrer"` |
| Payments | Handled by Topmate (PCI compliant) |

---

## 9. Scalability

| Scenario | Handling |
|----------|---------|
| Traffic spike | GitHub Pages CDN handles it — no server to overload |
| More testimonials | Add HTML cards (no performance issue for 50+ cards) |
| More photos | Add to `images/` folder (keep under 1GB GitHub limit) |
| More sessions | Add cards + update Topmate dashboard |

---

## 10. Future Enhancements (Not Implemented)

| Feature | Complexity | Requires Backend? |
|---------|-----------|------------------|
| Blog section | Low | No (static markdown) |
| Razorpay direct payment | High | Yes (Node.js server) |
| Analytics dashboard | Medium | No (Google Analytics tag) |
| Custom domain | Low | No (GitHub Pages CNAME) |

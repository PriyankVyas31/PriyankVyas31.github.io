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
│  Serves: index.html + data.json + images/       │
│  Domain: priyankvyas31.github.io                │
└──────────────┬──────────────────────────────────┘
               │
    ┌──────────┼──────────┬────────────┐
    ▼          ▼          ▼            ▼
┌────────┐ ┌────────┐ ┌──────────┐ ┌──────────┐
│ Google │ │ YouTube│ │ Backend  │ │ Razorpay │
│ Fonts  │ │ Embed  │ │ Server   │ │ Checkout │
│ (CDN)  │ │(iframe)│ │(GCP Cloud│ │ (Payment │
│        │ │        │ │  Run)    │ │  popup)  │
└────────┘ └────────┘ └────┬─────┘ └──────────┘
                           │
                    ┌──────┼──────┐
                    ▼      ▼      ▼
              ┌────────┐┌──────┐┌──────┐
              │Razorpay││Gmail ││Jitsi │
              │  API   ││ SMTP ││ Meet │
              │(orders)││(email││(video)│
              └────────┘└──────┘└──────┘
```

### Components

| Component | Role | Technology | Location |
|-----------|------|-----------|---------|
| **Frontend** | Renders website + booking modal | HTML + CSS + JS | GitHub Pages |
| **Backend** | Payment orders + verification + emails | Node.js + Express | GCP Cloud Run (Mumbai) |
| **Payment** | Processes actual money transfers | Razorpay | Razorpay servers |
| **Video** | Meeting rooms for sessions | Jitsi Meet | Jitsi servers |
| **Email** | Sends booking confirmations | Gmail SMTP | Google servers |
| **Admin** | Content management panel | admin.html + GitHub API | GitHub Pages |
| **Theme** | Dark/Light mode toggle | CSS variables + localStorage | Browser |

---

## 4. User Flow

```
User visits priyankvyas31.github.io
    │
    ├── Scrolls through sections (About, Skills, Experience, Education)
    │
    ├── Sees company logos marquee → builds trust
    │
    ├── Reads scrolling testimonials → builds confidence
    │
    ├── Clicks "Book a Session" card
    │       │
    │       ├── Step 1: Calendar modal opens
    │       │       ├── Selects a date (next 7+ days)
    │       │       └── Selects a time slot (9AM–6PM, 15-min intervals)
    │       │
    │       ├── Step 2: Details form
    │       │       ├── Enters name, email, phone
    │       │       └── Describes what they want to discuss
    │       │
    │       ├── Step 3: Payment
    │       │       ├── Frontend calls backend → creates Razorpay order
    │       │       ├── Razorpay popup opens
    │       │       ├── User pays via UPI/Cards/NetBanking/Wallets
    │       │       ├── Backend verifies payment (HMAC-SHA256)
    │       │       ├── Backend generates meeting link
    │       │       └── Backend sends emails to BOTH user and owner
    │       │
    │       └── Step 4: Confirmation
    │               ├── Shows booking ref + payment ID
    │               ├── Shows "Join Meeting" button
    │               └── Emails already sent
    │
    ├── Watches YouTube podcast
    │
    └── Clicks Contact links (LinkedIn, Email, Topmate)
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
| GitHub Pages | Frontend hosting | Free | Website goes down |
| GCP Cloud Run | Backend server | Free (2M reqs/mo) | Payments stop working |
| Razorpay | Payment processing | 2% per transaction | Can't accept payments |
| Gmail SMTP | Email notifications | Free (500/day) | No confirmation emails |
| Jitsi Meet | Video meeting rooms | Free | No meeting links |
| Google Fonts | Typography | Free | Falls back to system fonts |
| YouTube | Video embed | Free | Podcast section shows error |

---

## 8. Security

| Concern | Mitigation |
|---------|-----------|
| Payment fraud | HMAC-SHA256 signature verification on backend |
| API key exposure | Key Secret only on server, never in frontend code |
| CORS abuse | Backend only allows requests from priyankvyas31.github.io |
| Environment secrets | Stored in GCP env variables, not in code or Git |
| Admin access | Password-protected with SHA-256 hashed password |
| HTTPS | Enforced by both GitHub Pages and GCP Cloud Run |
| External links | All use `rel="noopener noreferrer"` |

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

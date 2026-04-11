# Complete System Design — Portfolio + Booking + Payment

> Written in simple terms for someone who doesn't know backend, Node.js, or cloud deployment.

---

## What We Built

A portfolio website where visitors can:
1. See your profile, skills, experience, and testimonials
2. Click a session card (e.g., "Mock Interview ₹500")
3. Pick a date and time from a calendar
4. Fill in their name, email, and phone
5. Pay securely via UPI (GPay, PhonePe), cards, net banking, or wallets
6. Get a confirmation email with a meeting link
7. You (Priyank) also get an email with all booking + payment details

---

## The Three Parts

Think of this like a restaurant:

| Part | Restaurant Analogy | Our System |
|------|--------------------|-----------|
| **Frontend** | The menu card + dining table | `index.html` — what visitors see |
| **Backend** | The kitchen | `server.js` — processes orders and payments |
| **Cloud** | The building | GCP Cloud Run — where the kitchen runs |

---

## Part 1: Frontend (The Website)

**What is it?** A single HTML file (`index.html`) that runs in the visitor's browser.

**Where does it live?** GitHub Pages — a free service that turns your GitHub repo into a website.

**What does it do?**
- Shows your portfolio (About, Skills, Experience, Education, etc.)
- When someone clicks "Book Now":
  - Opens a popup (modal) with a calendar
  - User picks a date → picks a time slot → fills a form
  - Clicks "Pay" → Razorpay's payment window opens
  - After payment → shows confirmation with meeting link

**Key files:**
| File | What it does |
|------|-------------|
| `index.html` | The entire website — HTML, CSS, and JavaScript all in one file |
| `data.json` | Content that can be edited via admin panel (sessions, testimonials, etc.) |
| `admin.html` | Password-protected admin panel to edit content without touching code |
| `images/` | Your photos and company logos |

---

## Part 2: Backend Server (The Kitchen)

### What is a backend?

Your website (frontend) runs in the visitor's browser. But some things can't be done in a browser:
- Creating a payment order (needs secret API keys)
- Verifying that a payment actually happened (needs cryptography)
- Sending emails (needs Gmail password)

So we built a **backend server** — a small program that runs on a computer in the cloud, waiting for requests from your website.

### What is Node.js?

Node.js is a way to run JavaScript (the same language used in websites) on a server instead of a browser. We chose it because:
- Same language as the frontend (JavaScript)
- Razorpay has an official Node.js library
- It's lightweight and fast

### What does our server do?

It has exactly **2 endpoints** (think of them as phone numbers your website can call):

#### Endpoint 1: `POST /api/create-order`

**When is it called?** When user clicks "Pay Now"

**What happens inside:**
```
Website says: "Someone wants to pay ₹500 for Mock Interview"
    ↓
Server takes this request
    ↓
Server calls Razorpay API: "Create a payment order for ₹500"
    ↓
Razorpay says: "OK, here's order_id: order_ABC123"
    ↓
Server sends order_id back to the website
    ↓
Website uses order_id to open Razorpay's payment popup
```

#### Endpoint 2: `POST /api/verify-payment`

**When is it called?** After user completes payment

**What happens inside:**
```
Website says: "User paid! Here's the payment_id and a signature"
    ↓
Server checks the signature using math (HMAC-SHA256)
    ↓
If signature is valid → payment is REAL (not faked)
    ↓
Server generates:
  - A booking reference (e.g., PV1ABC)
  - A meeting link (e.g., meet.jit.si/pv-1abc-x3k2)
    ↓
Server sends 2 emails:
  1. To YOU: "New booking! Here are the details..."
  2. To CUSTOMER: "Booking confirmed! Here's your meeting link..."
    ↓
Server tells the website: "Verified! Here's the booking ref + meeting link"
    ↓
Website shows the success screen with meeting link
```

### What is HMAC-SHA256? (Payment Verification)

Imagine Razorpay gives the customer a "receipt" after payment. But how do we know the receipt is real and not forged?

Razorpay signs the receipt with a **secret code** that only Razorpay and our server know. Our server uses the same secret code to verify the signature. If someone tries to fake a payment, the signature won't match.

This is called **HMAC-SHA256** — a mathematical way to verify that data hasn't been tampered with.

### What is an API Key?

Think of it like a username + password for our server to talk to Razorpay:

| Key | What it is | Who sees it |
|-----|-----------|-------------|
| **Key ID** (`rzp_test_...`) | Like a username — identifies our account | Public (used in frontend) |
| **Key Secret** (`1gYx...`) | Like a password — proves it's really us | Secret (only on server) |

The Key Secret is **NEVER** sent to the browser. It stays on our server. That's why we need a backend.

---

## Part 3: Cloud Deployment

### What is Docker?

Normally, to run our server, you'd need to:
1. Install Node.js
2. Download the code
3. Install dependencies (`npm install`)
4. Run it (`node server.js`)

**Docker** packages everything into a "container" — like a shipping container that has everything inside it. You just tell the cloud "run this container" and it works, no setup needed.

Our `Dockerfile` (recipe for the container):
```
1. Start with Node.js 18 (pre-installed)
2. Copy our package.json
3. Run npm install (download libraries)
4. Copy our server.js
5. Listen on port 3000
6. Run: node server.js
```

### What is GCP Cloud Run?

**GCP** = Google Cloud Platform (Google's cloud service, like Amazon's AWS or Microsoft's Azure)

**Cloud Run** = A service that runs Docker containers. You give it a container, it runs it on Google's servers.

Why Cloud Run?
- **Serverless** — you don't manage any servers, Google does
- **Auto-scales** — if 0 users, it sleeps (costs ₹0). If 1000 users, it scales up.
- **Free tier** — 2 million requests/month free
- **Asia South 1** — server runs in Mumbai, India (fast for Indian users)

### How deployment works:

```
You run: gcloud run deploy booking-server --source .
    ↓
Google Cloud:
  1. Uploads your code
  2. Reads the Dockerfile
  3. Builds a container image
  4. Deploys it to Cloud Run
  5. Gives you a URL: https://booking-server-412807327472.asia-south1.run.app
    ↓
Your server is now running 24/7 on Google's infrastructure
```

### Environment Variables

Sensitive values (API keys, passwords) are stored as **environment variables** in GCP — not in the code. Think of them as sticky notes on the server that only the server can read.

| Variable | What it's for |
|----------|--------------|
| `RAZORPAY_KEY_ID` | To create payment orders |
| `RAZORPAY_KEY_SECRET` | To verify payment signatures |
| `GMAIL_USER` | To send emails |
| `GMAIL_APP_PASSWORD` | Gmail password (special app-specific one) |
| `OWNER_EMAIL` | Where your booking notifications go |

---

## Part 4: Payment Gateway (Razorpay)

### What is a Payment Gateway?

It's a service that handles money transfers. We can't directly charge someone's credit card — that's illegal and insecure. Instead, Razorpay:
1. Shows a secure payment form (the popup)
2. Handles all the bank communication
3. Transfers money to your bank account
4. Takes a 2% commission

### Payment Methods Supported

| Method | How it works |
|--------|-------------|
| **UPI** | User enters UPI ID or scans QR → money goes directly from their bank |
| **Google Pay / PhonePe** | Opens the app on mobile, user confirms payment |
| **Credit/Debit Cards** | User enters card number, Razorpay processes it |
| **Net Banking** | Redirects to bank's website for login + OTP |
| **Wallets** | Paytm, Mobikwik balance |

### Test Mode vs Live Mode

| Mode | Key starts with | Real money? | For what? |
|------|----------------|-------------|-----------|
| **Test** | `rzp_test_` | No | Testing (use fake card 4111 1111 1111 1111) |
| **Live** | `rzp_live_` | Yes | Real customers |

Currently we're using **test mode**. When you're ready for real payments, generate live keys from Razorpay dashboard and update them in GCP.

---

## Part 5: Meeting Links

After payment, both you and the customer need to meet on a video call. We auto-generate a meeting link using **Jitsi Meet**:

| Feature | Jitsi Meet | Google Meet |
|---------|-----------|-------------|
| Cost | Free | Free |
| Account needed | No | Yes (Google) |
| Setup needed | None | Google Calendar API + OAuth |
| Link format | `meet.jit.si/pv-abc123` | `meet.google.com/abc-def-ghi` |

We chose Jitsi because it needs zero setup. Each booking gets a unique meeting room. Both parties click the link → video call starts.

---

## Part 6: Email Notifications

### How emails work

We use **Nodemailer** (a Node.js library) + **Gmail SMTP** to send emails programmatically.

**SMTP** = Simple Mail Transfer Protocol — the system email uses to send messages. Gmail lets you send emails through code if you have an "App Password" (different from your regular Gmail password).

### Two emails sent per booking

**Email to Customer:**
- Beautiful HTML email with session details, date, time
- ₹ amount paid, payment ID, booking reference
- Big "Join Meeting" button with the meeting link
- Your website link in footer

**Email to You (Priyank):**
- All customer details: name, email, phone
- Session, date, time, duration
- Payment amount, method (UPI/card), payment ID
- Meeting link
- Topic they want to discuss

---

## Complete Flow Diagram

```
VISITOR                    WEBSITE               BACKEND (GCP)           RAZORPAY          GMAIL
  │                       (GitHub Pages)         (Cloud Run)
  │                           │                      │                      │                │
  ├── Clicks "Book Now" ──────►                      │                      │                │
  │                           │                      │                      │                │
  │◄── Shows calendar ────────┤                      │                      │                │
  │                           │                      │                      │                │
  ├── Picks date + time ──────►                      │                      │                │
  │                           │                      │                      │                │
  │◄── Shows form ────────────┤                      │                      │                │
  │                           │                      │                      │                │
  ├── Fills name, email ──────►                      │                      │                │
  │                           │                      │                      │                │
  ├── Clicks "Pay Now" ───────►── POST /create-order──►                     │                │
  │                           │                      ├── Create order ──────►                │
  │                           │                      │◄── order_id ─────────┤                │
  │                           │◄── order_id ─────────┤                      │                │
  │                           │                      │                      │                │
  │◄── Razorpay popup ────────┤                      │                      │                │
  │                           │                      │                      │                │
  ├── Pays via UPI/Card ──────┼──────────────────────┼──────────────────────►                │
  │                           │                      │                      │                │
  │◄── payment_id + sig ──────┼──────────────────────┼──────────────────────┤                │
  │                           │                      │                      │                │
  │                           ├── POST /verify ───────►                     │                │
  │                           │                      ├── Verify signature   │                │
  │                           │                      ├── Generate meeting   │                │
  │                           │                      ├── Send email to you ─┼────────────────►
  │                           │                      ├── Send email to user ┼────────────────►
  │                           │◄── booking confirmed──┤                     │                │
  │                           │                      │                      │                │
  │◄── Success + meet link ───┤                      │                      │                │
  │                           │                      │                      │                │
```

---

## File Map

```
portfolio/                          (YOUR WEBSITE — GitHub Pages)
├── index.html                      Main website with booking modal + Razorpay
├── data.json                       Editable content (sessions, testimonials)
├── admin.html                      Admin panel to edit data.json
├── images/                         Photos and logos
└── docs/                           Documentation

booking-server/                     (BACKEND — GCP Cloud Run)
├── server.js                       The actual server (2 endpoints)
├── package.json                    Dependencies list (Express, Razorpay, etc.)
├── Dockerfile                      Docker recipe to build the container
├── .dockerignore                   Files to exclude from container
└── .gitignore                      Files to exclude from Git
```

---

## Glossary (Terms Explained)

| Term | Simple Explanation |
|------|-------------------|
| **API** | A way for two programs to talk to each other (like a phone call between website and server) |
| **Endpoint** | A URL the server listens on (like a phone number) |
| **POST** | A type of request that sends data (like filling a form) |
| **GET** | A type of request that asks for data (like opening a webpage) |
| **CORS** | A security rule that says which websites can talk to our server |
| **Middleware** | Code that runs before every request (like a security guard checking ID) |
| **npm** | Node Package Manager — downloads libraries for Node.js |
| **Express** | A library that makes it easy to build a web server in Node.js |
| **Docker** | Packages an app + all its dependencies into a portable box (container) |
| **Container** | A lightweight virtual machine — runs the same everywhere |
| **Cloud Run** | Google's service to run containers without managing servers |
| **SMTP** | The protocol used to send emails |
| **SHA-256** | A one-way math function that creates a unique "fingerprint" of data |
| **HMAC** | A way to verify data hasn't been tampered with, using a secret key |
| **Environment Variable** | A secret value stored on the server, not in code |
| **Webhook** | When one service notifies another service (Razorpay → our server) |
| **Deployment** | The process of putting code onto a server so it's live |
| **Serverless** | You don't manage servers — the cloud provider does it for you |

# Complete Beginner's Guide — Building a Portfolio with Payment System on Cloud

> This document explains everything we built, step by step, as if you've never written backend code or deployed anything to the cloud before.

---

## Table of Contents

1. [What Did We Build?](#1-what-did-we-build)
2. [How the Whole System Works](#2-how-the-whole-system-works)
3. [Frontend — The Website](#3-frontend--the-website)
4. [Backend — The Server](#4-backend--the-server)
5. [Payment Gateway — Razorpay](#5-payment-gateway--razorpay)
6. [Docker — Packaging the Server](#6-docker--packaging-the-server)
7. [Cloud Deployment — GCP Cloud Run](#7-cloud-deployment--gcp-cloud-run)
8. [Email System — Nodemailer + Gmail](#8-email-system--nodemailer--gmail)
9. [Meeting Links — Jitsi Meet](#9-meeting-links--jitsi-meet)
10. [Step-by-Step Commands Used](#10-step-by-step-commands-used)
11. [Common Problems and Fixes](#11-common-problems-and-fixes)
12. [Glossary](#12-glossary)

---

## 1. What Did We Build?

We built **two things**:

### Thing 1: A Portfolio Website (Frontend)
A single-page website hosted for free on GitHub Pages that shows:
- Your profile, photos, skills, work experience
- Session cards (Mock Interview ₹500, Career Guidance ₹200, etc.)
- Scrolling company logos and testimonials
- YouTube podcast embed
- Dark/Light theme toggle
- Admin panel to edit content without coding

### Thing 2: A Payment Server (Backend)
A small Node.js program running on Google Cloud that:
- Creates payment orders when someone clicks "Pay Now"
- Verifies the payment was real (not fake)
- Sends confirmation emails to both you and the customer
- Generates a unique video meeting link for the session

---

## 2. How the Whole System Works

Imagine ordering food on Zomato. Here's how our system maps to that:

| Zomato | Our System |
|--------|-----------|
| Zomato app (what you see) | `index.html` on GitHub Pages |
| Zomato's servers (process orders) | `server.js` on GCP Cloud Run |
| Payment (Razorpay/GPay popup) | Razorpay Checkout |
| "Your order is confirmed" notification | Gmail emails via Nodemailer |
| Restaurant address | Jitsi Meet video link |

### The Journey of One Booking

```
1. Visitor clicks "Book Now" on Mock Interview card
                    ↓
2. Calendar popup opens → picks April 15, 10:00 AM
                    ↓
3. Form appears → enters Name: "John", Email: "john@gmail.com"
                    ↓
4. Clicks "Pay ₹500 Now"
                    ↓
5. Website calls OUR SERVER: "Hey, someone wants to pay ₹500"
                    ↓
6. Our server calls RAZORPAY: "Create a ₹500 order please"
                    ↓
7. Razorpay says: "Here's order ID: order_ABC123"
                    ↓
8. Website shows RAZORPAY POPUP (UPI, Cards, Net Banking options)
                    ↓
9. John pays via Google Pay
                    ↓
10. Razorpay tells the website: "Payment done! ID: pay_XYZ789"
                    ↓
11. Website calls OUR SERVER: "Verify this payment is real"
                    ↓
12. Our server CHECKS THE MATH (HMAC-SHA256 signature)
                    ↓
13. ✅ Signature matches → Payment is REAL
                    ↓
14. Server generates:
    - Booking Ref: PV1ABC
    - Meeting Link: meet.jit.si/pv-1abc-x3k2
                    ↓
15. Server sends EMAIL to John:
    "Booking confirmed! Here's your meeting link..."
                    ↓
16. Server sends EMAIL to Priyank:
    "New booking! John Doe, ₹500, Mock Interview, April 15, 10 AM..."
                    ↓
17. Website shows: "✓ Booking Confirmed!" with meeting link
```

---

## 3. Frontend — The Website

### What files make up the website?

| File | Size | What it does |
|------|------|-------------|
| `index.html` | ~80KB | The ENTIRE website — all HTML, CSS, and JavaScript in one file |
| `data.json` | ~6KB | Content that the admin panel can edit (sessions, testimonials, about text) |
| `admin.html` | ~27KB | Password-protected admin panel to edit `data.json` via GitHub API |
| `images/` | ~8MB | 7 personal photos + 18 company logos |

### How does `index.html` work?

Think of it as three sections glued together:

```
┌─────────────────────────────────────────┐
│ <style> ... 950 lines of CSS ...        │  ← How things LOOK
│   Colors, fonts, layouts, animations    │
├─────────────────────────────────────────┤
│ <body> ... HTML content ...             │  ← What things ARE
│   Navbar, Hero, About, Skills, etc.     │
├─────────────────────────────────────────┤
│ <script> ... 250 lines of JavaScript .. │  ← How things BEHAVE
│   Theme toggle, booking modal, payment  │
└─────────────────────────────────────────┘
```

### How does the booking modal work?

When you click a session card, JavaScript runs this flow:

```
Step 1: CALENDAR
┌──────────────────────────────────────┐
│ Mock Interview                       │
│ Video meeting · 60 mins              │
│ ₹500  ₹1,500                        │
│                                      │
│ When should we meet?                 │
│ [Mon] [Tue] [Wed] [Thu] [Fri] ...   │
│  14    15    16    17    18          │
│                                      │
│ Select time:                         │
│ [09:00] [09:15] [09:30] [09:45]     │
│ [10:00] [10:15] [10:30] ...         │
│                                      │
│ [Continue]                           │
└──────────────────────────────────────┘
         ↓
Step 2: DETAILS
┌──────────────────────────────────────┐
│ Mock Interview                       │
│ Tue, 15 Apr · 10:00 AM · 60 mins    │
│                                      │
│ Name: [_______________]              │
│ Email: [_______________]             │
│ Phone: +91 [_______________]         │
│ Topic: [_______________]             │
│                                      │
│ [Continue to Payment · ₹500]         │
└──────────────────────────────────────┘
         ↓
Step 3: PAYMENT
┌──────────────────────────────────────┐
│ Pay ₹500                             │
│ Mock Interview · 2026-04-15 · 10 AM  │
│                                      │
│ ┌────────────────────────────┐       │
│ │ Mock Interview      ₹500  │       │
│ │ Total               ₹500  │       │
│ └────────────────────────────┘       │
│                                      │
│ 🔒 Pay securely via UPI, Cards,     │
│    Net Banking or Wallets            │
│                                      │
│ [Pay ₹500 Now]                       │
└──────────────────────────────────────┘
         ↓ (Razorpay popup opens)
Step 4: SUCCESS
┌──────────────────────────────────────┐
│           ✓                          │
│  Booking Confirmed!                  │
│  Thank you, John!                    │
│                                      │
│  📅 2026-04-15 at 10:00 AM          │
│  🔖 Ref: PV1ABC                     │
│  💳 Payment ID: pay_XYZ789          │
│                                      │
│  [🎥 Join Meeting]                   │
│  meet.jit.si/pv-1abc-x3k2           │
│                                      │
│  Confirmation emails sent            │
│  [Done]                              │
└──────────────────────────────────────┘
```

### Where is the website hosted?

**GitHub Pages** — a free service by GitHub.

How it works:
1. You push code to the `PriyankVyas31.github.io` repository
2. GitHub automatically serves it at `https://priyankvyas31.github.io`
3. Any `git push` automatically updates the live site in 1-2 minutes

No server to manage. No hosting fees. No deploy commands. Just `git push`.

---

## 4. Backend — The Server

### Why do we need a backend?

Three things that CANNOT be done in a browser:

| Task | Why not in browser? |
|------|-------------------|
| **Create Razorpay order** | Needs the `Key Secret` — if we put it in the browser, anyone can see it and steal money |
| **Verify payment** | Needs `Key Secret` for HMAC-SHA256 calculation — same reason |
| **Send emails** | Needs Gmail password — can't expose this in browser code |

So we built a small server that sits between the website and Razorpay/Gmail.

### What is Node.js?

JavaScript normally runs in your browser (Chrome, Firefox). Node.js lets you run JavaScript on a server instead. We chose it because:
- Same language as the frontend
- Razorpay and Gmail both have Node.js libraries
- It's lightweight — our entire server is just 1 file (200 lines)

### What is Express?

Express is a library for Node.js that makes it easy to create a web server. Without Express, you'd need 50+ lines just to handle an HTTP request. With Express:

```javascript
app.post('/api/create-order', (req, res) => {
    // Handle the request
    res.json({ id: 'order_123' });
});
```

That's it — one URL, one function.

### What libraries does our server use?

```
package.json → dependencies:
├── express      → Web server (handles HTTP requests)
├── razorpay     → Razorpay payment SDK (creates orders, verifies payments)
├── cors         → Security (controls which websites can call our server)
├── nodemailer   → Email sending (connects to Gmail SMTP)
└── crypto       → Built-in Node.js (HMAC-SHA256 for payment verification)
```

### The server code explained (server.js)

Our server has exactly **3 things**:

#### 1. Health Check (GET /)
```
Browser visits: https://booking-server-412807327472.asia-south1.run.app/
Server responds: { "status": "ok", "service": "Priyank Vyas Booking Server" }
```
This is just to check the server is alive.

#### 2. Create Order (POST /api/create-order)

**When called:** User clicks "Pay ₹500 Now" on the website

**What it does:**
```
Input:  { amount: 500, session_title: "Mock Interview" }
                    ↓
Server calls Razorpay API:
  razorpay.orders.create({
      amount: 50000,        ← Razorpay uses PAISE (₹500 = 50000 paise)
      currency: 'INR',
      receipt: 'PV_1712345678'
  })
                    ↓
Output: { id: "order_ABC123", amount: 50000, currency: "INR", key_id: "rzp_test_xxx" }
```

#### 3. Verify Payment (POST /api/verify-payment)

**When called:** After user completes payment in Razorpay popup

**What it does (5 sub-steps):**

```
Step A: VERIFY SIGNATURE
─────────────────────────────────────
Razorpay signs every payment with a secret.
We check: is this signature real?

  received_signature = "abc123..."
  expected_signature = HMAC-SHA256("order_ABC123|pay_XYZ789", KEY_SECRET)

  If they match → payment is REAL ✅
  If they don't → payment is FAKE ❌ (reject!)

Step B: FETCH PAYMENT DETAILS
─────────────────────────────────────
  payment = razorpay.payments.fetch("pay_XYZ789")
  → Gets: method (upi), vpa (john@upi), amount (50000), etc.

Step C: GENERATE BOOKING REF + MEETING LINK
─────────────────────────────────────
  bookingRef = "PV" + timestamp_in_base36   → "PV1ABC"
  meetingId  = "pv-pv1abc-x3k2"
  meetLink   = "https://meet.jit.si/pv-pv1abc-x3k2"

Step D: SEND EMAILS
─────────────────────────────────────
  Email 1 → priyankvyas001@gmail.com (you)
    Subject: "💰 New Booking: Mock Interview — ₹500 [PV1ABC]"
    Body: All details (customer name, email, phone, date, time,
          payment method, payment ID, meeting link)

  Email 2 → john@gmail.com (customer)
    Subject: "✅ Booking Confirmed — Mock Interview with Priyank Vyas"
    Body: Session details + date/time + amount + meeting link button

Step E: RESPOND TO WEBSITE
─────────────────────────────────────
  { verified: true, bookingRef: "PV1ABC", paymentId: "pay_XYZ789",
    meetLink: "https://meet.jit.si/pv-pv1abc-x3k2" }
```

---

## 5. Payment Gateway — Razorpay

### What is Razorpay?

A company that handles money transfers. You can't directly charge someone's card — that's illegal and needs PCI compliance. Razorpay does it for you and takes 2% commission.

### How does the payment popup work?

When user clicks "Pay Now":
1. Our server creates an "order" with Razorpay (like a restaurant ticket)
2. Razorpay sends back an order ID
3. We open Razorpay's popup using their JavaScript SDK
4. User sees payment options:
   - **UPI:** Enter UPI ID (john@upi) or scan QR code
   - **Google Pay / PhonePe:** Opens the app on mobile
   - **Cards:** Enter card number, expiry, CVV
   - **Net Banking:** Select bank → login → OTP
   - **Wallets:** Paytm, Mobikwik balance

5. After payment, Razorpay calls our `handler` function with:
   - `razorpay_order_id` (which order)
   - `razorpay_payment_id` (which payment)
   - `razorpay_signature` (proof it's real)

### Test Mode vs Live Mode

| | Test Mode | Live Mode |
|-|-----------|-----------|
| Key starts with | `rzp_test_` | `rzp_live_` |
| Real money? | No | Yes |
| Test card | 4111 1111 1111 1111 | Real cards |
| Where to use | Development/testing | Production |
| How to switch | Dashboard → Settings → API Keys → Generate live key | Same |

We're currently using **test mode**. To go live:
1. Complete Razorpay KYC (PAN + bank account)
2. Generate live keys from Razorpay Dashboard
3. Update the environment variables on GCP (see commands below)

### How does signature verification work?

Think of it like a sealed envelope:

```
Razorpay creates: "order_ABC123|pay_XYZ789"
Razorpay signs it with a SECRET KEY that only Razorpay and our server know
Razorpay sends the signature to the browser

Browser sends it to our server

Our server also calculates the signature using the SAME SECRET KEY:
  expected = HMAC-SHA256("order_ABC123|pay_XYZ789", "our_secret_key")

If our calculation matches Razorpay's signature → nobody tampered with it ✅
If they don't match → someone tried to fake a payment ❌
```

This is called **HMAC-SHA256** — a mathematical one-way function. Even if you change one character in the input, the output completely changes. There's no way to fake it without the secret key.

---

## 6. Docker — Packaging the Server

### What is Docker?

Imagine you built a LEGO house. Normally, to show it to someone, you'd say:
- "First buy these specific LEGO pieces"
- "Then follow these 50 steps to build it"
- "Oh, and it only works with THIS version of LEGO"

**Docker** is like putting your finished LEGO house in a sealed box. You hand someone the box, they open it, and the house is already built. Works everywhere.

### Our Dockerfile (the recipe)

```dockerfile
FROM node:18-alpine     # Start with a box that already has Node.js 18 installed
                        # "alpine" = smaller version (50MB instead of 900MB)

WORKDIR /app            # Create a folder called /app inside the box

COPY package.json .     # Copy the list of libraries we need

RUN npm install --production  # Download and install those libraries
                              # --production = skip testing tools

COPY server.js .        # Copy our server code into the box

EXPOSE 3000             # Tell Docker our app listens on port 3000

CMD ["node", "server.js"]  # When the box starts, run this command
```

**Why copy package.json separately before server.js?**

Docker caches each step. If you change `server.js` but not `package.json`, Docker says "package.json didn't change, so I can reuse the cached `npm install` from last time!" This makes rebuilds much faster (seconds instead of minutes).

### What is .dockerignore?

Tells Docker which files to SKIP when building:
```
node_modules/    # Don't copy these — npm install will recreate them
.env             # Don't copy secrets into the image
npm-debug.log    # Don't copy error logs
```

---

## 7. Cloud Deployment — GCP Cloud Run

### What is GCP?

**Google Cloud Platform** — Google rents out its massive data centers to developers. You can run servers, store data, train AI models, etc. on Google's hardware.

### What is Cloud Run?

A service that runs Docker containers. You give it a container, it runs it on Google's servers in a data center.

**Why Cloud Run specifically?**

| Feature | Cloud Run | Regular VM | Your laptop |
|---------|-----------|-----------|-------------|
| Cost at 0 users | ₹0 (scales to zero) | ~₹500/month (always running) | Free but can't run 24/7 |
| Scales up | Auto (0 → 1000 instances) | Manual | No |
| Manage servers | No (Google does it) | Yes (you do) | Yes |
| Setup time | 5 minutes | 30+ minutes | Already done |
| Reliability | 99.95% uptime | You manage | Your WiFi reliability |
| Location | Mumbai datacenter (asia-south1) | You choose | Your home |

### How Cloud Run works

```
You: "Hey Google, here's my code. Run it please."
                    ↓
Google Cloud Build: "Let me read the Dockerfile and build a container image..."
                    ↓
Google Artifact Registry: "I'll store this container image..."
                    ↓
Google Cloud Run: "I'll create a service and give you a URL!"
                    ↓
URL: https://booking-server-412807327472.asia-south1.run.app
                    ↓
When someone visits that URL:
  → Cloud Run starts your container (if it was sleeping)
  → Routes the request to your server
  → Your server processes it and responds
  → If no requests for 15 minutes → container goes back to sleep (₹0)
```

### What are Environment Variables?

Secrets (API keys, passwords) that your server needs but shouldn't be in the code.

Think of it like a hotel safe:
- Your code is the hotel room (visible to anyone who enters)
- Environment variables are in the safe (only the server can access them)
- Even if someone reads your code on GitHub, they can't see the secrets

Our environment variables:
```
RAZORPAY_KEY_ID      = rzp_test_xxx    (Razorpay login)
RAZORPAY_KEY_SECRET  = 1gYxbJSTxxx     (Razorpay password)
GMAIL_USER           = priyankvyas001@gmail.com
GMAIL_APP_PASSWORD   = xxxx xxxx xxxx xxxx  (Gmail app password)
OWNER_EMAIL          = priyankvyas001@gmail.com
```

---

## 8. Email System — Nodemailer + Gmail

### What is SMTP?

**Simple Mail Transfer Protocol** — the standard system used to send emails across the internet since 1982. When you click "Send" in Gmail, your email travels via SMTP servers.

### What is an App Password?

Gmail doesn't let random programs log in with your regular password (security risk). Instead, you create an "App Password" — a special 16-character password that only works for a specific app.

Steps to get one:
1. Enable 2-Step Verification on your Google Account
2. Go to Google Account → Security → App Passwords
3. Create a password for "Mail"
4. You get: `abcd efgh ijkl mnop` (16 characters)
5. Programs use this instead of your real Gmail password

### What is Nodemailer?

A Node.js library that connects to Gmail's SMTP server and sends emails programmatically.

```javascript
// Connect to Gmail
const transporter = nodemailer.createTransport({
    service: 'gmail',
    auth: {
        user: 'priyankvyas001@gmail.com',
        pass: 'app_password_here'
    }
});

// Send email
transporter.sendMail({
    from: '"Priyank Vyas" <priyankvyas001@gmail.com>',
    to: 'customer@email.com',
    subject: '✅ Booking Confirmed!',
    html: '<h1>Your booking details...</h1>'
});
```

### What emails get sent?

**Email to Customer:** Beautiful HTML email with:
- Blue-purple gradient header
- Session name, date, time
- Amount paid, booking reference
- Big "Join Meeting" button
- Payment ID

**Email to You (Owner):** Detailed table with:
- ALL customer info (name, email, phone)
- Session, date, time, duration
- Payment amount, method (UPI/card), payment ID
- Meeting link
- Topic they want to discuss

---

## 9. Meeting Links — Jitsi Meet

### Why Jitsi and not Google Meet?

| | Jitsi Meet | Google Meet |
|-|-----------|-------------|
| Account needed | No | Yes (Google) |
| Setup | Zero | Google Calendar API + OAuth (complex) |
| Cost | Free | Free |
| How it works | Just open a URL | Need to create event via API |
| Quality | Good | Great |

We went with Jitsi because it needs **zero setup**. We just generate a unique URL like `meet.jit.si/pv-1abc-x3k2` and both parties open it. That's it — video call starts.

Can be upgraded to Google Meet later by integrating Google Calendar API.

---

## 10. Step-by-Step Commands Used

### A. Setting Up the Frontend

```bash
# 1. Create the project folder
mkdir C:\Users\priyankvyas\Desktop\portfolio
cd C:\Users\priyankvyas\Desktop\portfolio

# 2. Initialize Git
git init
git branch -M main

# 3. Create index.html (the website)
# [We used Copilot to generate the HTML]

# 4. Stage all files
git add -A

# 5. Commit
git commit -m "Initial commit: portfolio website"

# 6. Create GitHub repo
gh repo create PriyankVyas31.github.io --public --description "Portfolio website"

# 7. Add remote and push
git remote add origin https://github.com/PriyankVyas31/PriyankVyas31.github.io.git
git push -u origin main

# GitHub Pages auto-enables for username.github.io repos
# Site is live at: https://priyankvyas31.github.io/
```

### B. Setting Up the Backend

```bash
# 1. Create backend folder
mkdir C:\Users\priyankvyas\Desktop\booking-server
cd C:\Users\priyankvyas\Desktop\booking-server

# 2. Create files
# - server.js (the actual server code)
# - package.json (dependencies list)
# - Dockerfile (Docker build recipe)
# - .dockerignore (files to exclude)
# - .gitignore (exclude node_modules and .env)

# 3. Initialize Git and push to GitHub
git init
git branch -M main
git add -A
git commit -m "Razorpay payment backend"
gh repo create booking-server --private --description "Payment backend"
git remote add origin https://github.com/PriyankVyas31/booking-server.git
git push -u origin main
```

### C. Installing Google Cloud SDK

```bash
# Install gcloud CLI on Windows
winget install --id Google.CloudSDK --accept-source-agreements --accept-package-agreements

# Refresh PATH (so terminal can find gcloud)
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")

# Verify installation
gcloud --version
# Output: Google Cloud SDK 564.0.0
```

### D. Authenticating with Google Cloud

```bash
# Login (opens browser)
gcloud auth login
# → Opens Google login page
# → Select your Google account
# → Grant permissions
# → Terminal shows: "You are now logged in as priyankvyas001@gmail.com"
```

### E. Creating a GCP Project

```bash
# Create project
gcloud projects create booking-server-pv --name="Booking Server"
# Output: "Create in progress... done."

# Set it as active project
gcloud config set project booking-server-pv
# Output: "Updated property [core/project]."
```

### F. Enabling Required APIs

```bash
# You need billing enabled first!
# Go to: https://console.cloud.google.com/billing/linkedaccount?project=booking-server-pv
# Link a billing account (credit/debit card required, but won't be charged)

# Then enable APIs:
gcloud services enable run.googleapis.com cloudbuild.googleapis.com artifactregistry.googleapis.com
# Output: "Operation finished successfully."
```

### G. Deploying to Cloud Run

```bash
# Navigate to backend folder
cd C:\Users\priyankvyas\Desktop\booking-server

# Deploy with ALL environment variables in one command
gcloud run deploy booking-server \
  --source . \
  --region asia-south1 \
  --platform managed \
  --allow-unauthenticated \
  --port 3000 \
  --memory 256Mi \
  --set-env-vars "RAZORPAY_KEY_ID=rzp_test_xxx,RAZORPAY_KEY_SECRET=xxx,GMAIL_USER=priyankvyas001@gmail.com,GMAIL_APP_PASSWORD=xxxx,OWNER_EMAIL=priyankvyas001@gmail.com"

# What each flag means:
# --source .                  → Build from current directory (reads Dockerfile)
# --region asia-south1        → Mumbai datacenter (fast for India)
# --platform managed          → Google manages the servers
# --allow-unauthenticated     → Anyone can call our API (needed for website)
# --port 3000                 → Our Express app listens on port 3000
# --memory 256Mi              → 256MB RAM (enough for Node.js)
# --set-env-vars "..."        → Secret environment variables

# It will ask: "Do you want to continue (Y/n)?" → Type Y

# Output after 3-5 minutes:
# Service URL: https://booking-server-412807327472.asia-south1.run.app
```

### H. Updating the Backend (After Code Changes)

```bash
# 1. Edit server.js
# 2. Commit and push to GitHub
cd C:\Users\priyankvyas\Desktop\booking-server
git add -A
git commit -m "Your change description"
git push origin main

# 3. Redeploy to Cloud Run (without re-setting env vars)
gcloud run deploy booking-server \
  --source . \
  --region asia-south1 \
  --platform managed \
  --allow-unauthenticated \
  --port 3000 \
  --memory 256Mi

# Note: env vars persist from the first deploy. You only need --set-env-vars
# if you want to CHANGE them.
```

### I. Updating Environment Variables (Without Redeploying Code)

```bash
# Update just one variable:
gcloud run services update booking-server \
  --region asia-south1 \
  --set-env-vars "RAZORPAY_KEY_SECRET=new_secret_here"

# Update multiple:
gcloud run services update booking-server \
  --region asia-south1 \
  --set-env-vars "RAZORPAY_KEY_ID=rzp_live_xxx,RAZORPAY_KEY_SECRET=new_live_secret"
```

### J. Checking Logs (If Something Goes Wrong)

```bash
# View recent logs
gcloud run services logs read booking-server --region asia-south1

# Stream live logs (ctrl+C to stop)
gcloud run services logs tail booking-server --region asia-south1
```

### K. Pushing Frontend Changes

```bash
cd C:\Users\priyankvyas\Desktop\portfolio
git add -A
git commit -m "Your change description"
git push origin main

# Site auto-updates at priyankvyas31.github.io in 1-2 minutes
```

---

## 11. Common Problems and Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| `gcloud: not recognized` | PATH not refreshed after install | Run: `$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")` |
| `Billing must be enabled` | No billing account linked to GCP project | Go to GCP Console → Billing → Link an account |
| `Payment failed` in Razorpay | Using test mode with real card (or vice versa) | Use test card `4111 1111 1111 1111` in test mode |
| Emails not sending | Wrong Gmail App Password | Regenerate at myaccount.google.com/apppasswords |
| `CORS error` in browser console | Backend not allowing your domain | Check `origin` array in server.js includes your domain |
| `git push` rejected | Remote has changes you don't have locally | Run: `git pull --rebase origin main` then `git push` |
| Website shows old content | Browser cache | Hard refresh: `Ctrl+Shift+R` |
| Cloud Run returns 502 | Server crashed on startup | Check logs: `gcloud run services logs read booking-server --region asia-south1` |

---

## 12. Glossary

| Term | Simple Explanation |
|------|-------------------|
| **Frontend** | Code that runs in the user's browser (HTML, CSS, JS) |
| **Backend** | Code that runs on a server in the cloud (Node.js) |
| **API** | A way for two programs to talk to each other over the internet |
| **API Endpoint** | A specific URL that a server listens on (like a phone number) |
| **POST request** | Sending data TO a server (like submitting a form) |
| **GET request** | Asking a server FOR data (like loading a webpage) |
| **JSON** | A text format for sending data between programs: `{ "name": "John" }` |
| **CORS** | A security rule that says which websites can call which servers |
| **HTTPS** | Encrypted HTTP — data is scrambled so no one can read it in transit |
| **Node.js** | A way to run JavaScript on a server instead of a browser |
| **Express** | A library that makes it easy to build web servers in Node.js |
| **npm** | Node Package Manager — downloads JavaScript libraries |
| **package.json** | A file listing which libraries your project needs |
| **Docker** | Tool that packages your app + everything it needs into a portable "container" |
| **Container** | A lightweight isolated environment — like a VM but faster and smaller |
| **Dockerfile** | A recipe that tells Docker how to build your container |
| **Image** | A snapshot of a container (like a blueprint) |
| **GCP** | Google Cloud Platform — Google's cloud computing service |
| **Cloud Run** | GCP service that runs Docker containers without managing servers |
| **Cloud Build** | GCP service that builds Docker images from code |
| **Artifact Registry** | GCP service that stores Docker images |
| **Environment Variable** | A secret value stored on the server, not in code |
| **SMTP** | Protocol used to send emails (Simple Mail Transfer Protocol) |
| **Nodemailer** | Node.js library for sending emails via SMTP |
| **App Password** | A special Gmail password for third-party apps (not your main password) |
| **Razorpay** | Indian payment gateway — handles UPI, cards, net banking, wallets |
| **Payment Gateway** | A service that securely processes online payments |
| **Order ID** | A unique ID Razorpay creates for each payment attempt |
| **Payment ID** | A unique ID for a completed payment |
| **Signature** | A mathematical proof that data hasn't been tampered with |
| **HMAC-SHA256** | A cryptographic function used to create/verify signatures |
| **SHA-256** | A one-way hash function — turns any input into a fixed 64-character output |
| **Jitsi Meet** | Free, open-source video conferencing (like Google Meet but no account needed) |
| **GitHub Pages** | Free hosting for static websites from GitHub repositories |
| **git push** | Upload your local code changes to GitHub |
| **git pull** | Download code changes from GitHub to your computer |
| **Serverless** | A cloud model where you don't manage servers — the provider handles everything |
| **PCI Compliance** | Security standards for handling credit card data (Razorpay handles this) |
| **KYC** | Know Your Customer — identity verification required by Razorpay |
| **Webhook** | An automatic notification from one service to another via HTTP |
| **CDN** | Content Delivery Network — serves files from servers closest to the user |
| **Base36** | A number system using 0-9 and a-z (used to make short booking refs) |

# Backend Server Design — Razorpay Payment Gateway + Email Notifications

> **Status:** Not yet deployed. To be implemented in future.  
> **Repo:** [github.com/PriyankVyas31/booking-server](https://github.com/PriyankVyas31/booking-server)

---

## 1. Problem Statement

The current portfolio website uses Topmate.io for booking and payments. While it works, it has limitations:
- Topmate takes a commission on every transaction
- No control over the booking/payment flow
- Cannot customize the payment experience
- Cannot directly verify payments

**Goal:** Build a self-hosted backend that handles payment creation, verification, and email notifications — giving full control over the booking flow.

---

## 2. Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    USER (Browser)                            │
│  priyankvyas31.github.io                                    │
└──────────────┬───────────────────────────────┬───────────────┘
               │                               │
               │ 1. POST /api/create-order     │ 4. Razorpay checkout
               │    (amount, session info)     │    opens in browser
               │                               │
               ▼                               │
┌──────────────────────────────┐               │
│     BACKEND SERVER           │               │
│     (Node.js + Express)      │               │
│     Hosted on Render.com     │               │
│                              │               │
│  /api/create-order ──────────┼──► Razorpay API
│       │                      │       │
│       └── Returns order_id ◄─┼───────┘
│                              │
│  /api/verify-payment ◄───────┼─── 5. Frontend sends
│       │                      │       payment_id + signature
│       ├── Verify signature   │
│       ├── Fetch payment info │──► Razorpay API
│       ├── Send owner email   │──► Gmail SMTP
│       ├── Send customer email│──► Gmail SMTP
│       └── Return success     │
└──────────────────────────────┘
```

---

## 3. Complete Flow (Step by Step)

### Step 1: User selects a session on the website
- User picks date, time, fills in name/email/phone
- Clicks "Pay ₹X Now"

### Step 2: Frontend calls backend to create an order
```
POST /api/create-order
Body: {
    "amount": 500,
    "session_title": "Mock Interview",
    "customer_name": "John Doe",
    "customer_email": "john@example.com"
}
```

### Step 3: Backend creates order via Razorpay API
```javascript
const order = await razorpay.orders.create({
    amount: 500 * 100,  // Razorpay expects paise (₹500 = 50000 paise)
    currency: 'INR',
    receipt: 'PV_1712345678'
});
```

### Step 4: Backend returns order details to frontend
```json
{
    "id": "order_ABC123",
    "amount": 50000,
    "currency": "INR",
    "key_id": "rzp_live_xxxxx"
}
```

### Step 5: Frontend opens Razorpay Checkout
Razorpay's official checkout popup opens in the browser. User sees all payment options:
- UPI (GPay, PhonePe, Paytm, BHIM)
- Credit/Debit Cards (Visa, Mastercard, RuPay)
- Net Banking (SBI, HDFC, ICICI, etc.)
- Wallets (Paytm, Mobikwik)

### Step 6: User completes payment
Razorpay processes the payment and returns:
```json
{
    "razorpay_order_id": "order_ABC123",
    "razorpay_payment_id": "pay_XYZ789",
    "razorpay_signature": "hmac_sha256_signature"
}
```

### Step 7: Frontend sends payment proof to backend
```
POST /api/verify-payment
Body: {
    "razorpay_order_id": "order_ABC123",
    "razorpay_payment_id": "pay_XYZ789",
    "razorpay_signature": "hmac_sha256_signature",
    "booking": {
        "session": "Mock Interview",
        "date": "2026-04-15",
        "time": "10:00 AM",
        "name": "John Doe",
        "email": "john@example.com",
        "phone": "9876543210",
        "topic": "System design interview prep"
    }
}
```

### Step 8: Backend verifies the payment signature
```javascript
const expectedSignature = crypto
    .createHmac('sha256', RAZORPAY_KEY_SECRET)
    .update(razorpay_order_id + '|' + razorpay_payment_id)
    .digest('hex');

if (expectedSignature !== razorpay_signature) {
    return res.status(400).json({ error: 'Invalid signature' });
}
```
This is a **cryptographic verification** — it proves the payment was actually processed by Razorpay and not faked.

### Step 9: Backend sends emails
Two emails are sent via Gmail SMTP:

**Email to Priyank (owner):**
- Subject: "💰 New Booking: Mock Interview — ₹500 [PV1ABC]"
- Contains: booking ref, session, amount, payment ID, date, time, customer details

**Email to customer:**
- Subject: "✅ Booking Confirmed — Mock Interview with Priyank Vyas [PV1ABC]"
- Contains: booking ref, session, amount, date, time, payment ID

### Step 10: Frontend shows success
Backend returns:
```json
{
    "verified": true,
    "bookingRef": "PV1ABC",
    "paymentId": "pay_XYZ789",
    "amount": 500
}
```
Frontend shows the confirmation screen with booking ref and payment ID.

---

## 4. API Endpoints

### `GET /`
Health check.
```json
{ "status": "ok", "service": "Priyank Vyas Booking Server" }
```

### `POST /api/create-order`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `amount` | number | Yes | Amount in INR (minimum ₹1) |
| `session_title` | string | No | Name of the session |
| `customer_name` | string | No | Customer's name |
| `customer_email` | string | No | Customer's email |

**Response:** `{ id, amount, currency, key_id }`

### `POST /api/verify-payment`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `razorpay_order_id` | string | Yes | Order ID from Razorpay |
| `razorpay_payment_id` | string | Yes | Payment ID from Razorpay |
| `razorpay_signature` | string | Yes | HMAC signature from Razorpay |
| `booking` | object | Yes | Booking details (session, date, time, name, email, phone, topic) |

**Response:** `{ verified, bookingRef, paymentId, amount }`

---

## 5. Tech Stack

| Component | Technology | Why |
|-----------|-----------|-----|
| Runtime | Node.js 18+ | Lightweight, fast, good Razorpay SDK |
| Framework | Express.js | Minimal, battle-tested HTTP server |
| Payment | Razorpay Node SDK | Official SDK, handles orders + verification |
| Email | Nodemailer + Gmail SMTP | Free, reliable, no extra service needed |
| CORS | cors package | Allow requests from GitHub Pages domain |
| Hosting | Render.com (free tier) | Free, auto-deploy from GitHub, HTTPS |

---

## 6. Environment Variables

These are set in the Render.com dashboard (never in code):

| Variable | Example | Where to get it |
|----------|---------|----------------|
| `RAZORPAY_KEY_ID` | `rzp_live_abc123` | Razorpay Dashboard → Settings → API Keys |
| `RAZORPAY_KEY_SECRET` | `secret_xyz789` | Same as above (shown once) |
| `GMAIL_USER` | `priyankvyas001@gmail.com` | Your Gmail address |
| `GMAIL_APP_PASSWORD` | `abcd efgh ijkl mnop` | Google Account → App Passwords |
| `OWNER_EMAIL` | `priyankvyas001@gmail.com` | Where booking notifications go |

---

## 7. Security

| Threat | Mitigation |
|--------|-----------|
| **Fake payments** | Cryptographic HMAC-SHA256 signature verification |
| **API keys leaked** | Stored in environment variables, never in code |
| **CORS abuse** | Only allows requests from `priyankvyas31.github.io` |
| **Replay attacks** | Each order has a unique ID, can only be verified once |
| **Email spam** | Emails only sent after successful payment verification |
| **Amount tampering** | Amount is set server-side when creating the order |

---

## 8. Email Templates

### Owner Notification Email
```
Subject: 💰 New Booking: Mock Interview — ₹500 [PV1ABC]

┌──────────────────────────────────────┐
│ Booking Ref    │ PV1ABC             │
│ Session        │ Mock Interview      │
│ Amount         │ ₹500               │
│ Payment ID     │ pay_XYZ789         │
│ Method         │ upi (priyank@ybl)  │
│ Date           │ 2026-04-15         │
│ Time           │ 10:00 AM           │
│ Customer       │ John Doe           │
│ Email          │ john@example.com   │
│ Phone          │ +91 9876543210     │
│ Topic          │ System design prep │
└──────────────────────────────────────┘
```

### Customer Confirmation Email
```
Subject: ✅ Booking Confirmed — Mock Interview with Priyank Vyas [PV1ABC]

Hi John,

Your booking with Priyank Vyas has been confirmed.

┌──────────────────────────────────────┐
│ Booking Ref    │ PV1ABC             │
│ Session        │ Mock Interview      │
│ Amount Paid    │ ₹500               │
│ Date           │ 2026-04-15         │
│ Time           │ 10:00 AM           │
│ Payment ID     │ pay_XYZ789         │
└──────────────────────────────────────┘

Priyank will reach out to you to confirm the session.
If you have any questions, reply to this email.
```

---

## 9. Deployment Steps (Future)

### Prerequisites
1. **Razorpay Account** — Sign up at [dashboard.razorpay.com](https://dashboard.razorpay.com), complete KYC (PAN + bank account, takes 1-2 days), generate API keys
2. **Gmail App Password** — Go to [myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords), enable 2FA if not already, create an app password for "Mail"
3. **Render Account** — Sign up at [render.com](https://render.com) with GitHub

### Deploy on Render
1. Go to Render Dashboard → **New → Web Service**
2. Connect the `PriyankVyas31/booking-server` repo
3. Configure:
   - **Runtime:** Node
   - **Build Command:** `npm install`
   - **Start Command:** `npm start`
4. Add environment variables (see Section 6)
5. Click **Deploy**
6. Copy the URL (e.g., `https://booking-server-xxxx.onrender.com`)

### Update Frontend
In `index.html`, update the `API_BASE` variable:
```javascript
const API_BASE = 'https://booking-server-xxxx.onrender.com';
```

### Test
1. Open the website
2. Click a session card → select date/time → fill form → click "Pay Now"
3. Razorpay checkout should open
4. Use Razorpay test mode for testing (test card: 4111 1111 1111 1111)
5. After payment, check both emails

---

## 10. Cost Analysis

| Service | Free Tier | Paid Tier |
|---------|-----------|-----------|
| **Render** | 750 hours/month (sleeps after 15min idle) | $7/month (always on) |
| **Razorpay** | No monthly fee | 2% per transaction |
| **Gmail SMTP** | 500 emails/day | Sufficient for low volume |
| **GitHub Pages** | Unlimited | Free forever |

**For 10 bookings/month:** ₹0 (free tier covers everything). Razorpay takes 2% per transaction (e.g., ₹10 on a ₹500 booking).

---

## 11. Limitations & Future Improvements

| Limitation | Impact | Solution |
|-----------|--------|---------|
| Render free tier sleeps after 15min | First request takes ~30s to wake up | Upgrade to $7/month or use cron ping |
| No database | Can't track booking history | Add SQLite or Supabase (free) |
| No refund flow | Manual refunds via Razorpay dashboard | Add `/api/refund` endpoint |
| No calendar sync | Manual scheduling after booking | Integrate Google Calendar API |
| No receipt/invoice | Customer only gets email | Generate PDF receipts |
| Gmail SMTP limits | 500 emails/day | Switch to SendGrid/Resend for higher volume |

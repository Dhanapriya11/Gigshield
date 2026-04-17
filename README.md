# 🛡 GigShield — Full Stack App Setup Guide

## Project Structure
```
gigshield/
├── backend/
│   ├── app.py              # Flask API (all routes)
│   └── requirements.txt
├── frontend/
│   └── templates/
│       ├── index.html      # Main app (register + pay)
│       ├── dashboard.html  # Worker dashboard
│       └── admin.html      # Admin panel
└── .env.example            # Copy to .env and fill in keys
```

---

## ⚡ Step 1: MongoDB Atlas Setup

1. Go to https://cloud.mongodb.com → Create free cluster
2. Click **Connect** → **Connect your application**
3. Copy the connection string:
   `mongodb+srv://USERNAME:PASSWORD@cluster0.xxxxx.mongodb.net/gigshield`
4. Create a database user under **Database Access**
5. Allow all IPs under **Network Access** (0.0.0.0/0 for dev)

Collections auto-created by app:
- `workers` — registered gig workers
- `policies` — insurance policies
- `pool` — community fund balance
- `payments` — Razorpay payment records
- `payouts` — UPI payout records
- `triggers` — weather event logs
- `fraud_flags` — fraud detection logs

---

## ⚡ Step 2: Razorpay Setup

### For COLLECTING Premiums (Razorpay Checkout):
1. Go to https://dashboard.razorpay.com
2. **Settings → API Keys** → Generate Test Keys
3. Copy `Key ID` and `Key Secret` into `.env`

### For SENDING Payouts (Razorpay Payouts / Fund Transfer):
1. Dashboard → **Razorpay X** → Create Current Account
2. Enable **Payouts** product
3. Get your **Account Number** from dashboard
4. Add it to `.env` as `RAZORPAY_ACCOUNT_NUMBER`

> **Note**: For test mode, payouts are simulated automatically if Razorpay X isn't configured. Real UPI transfers require a live Razorpay account.

---

## ⚡ Step 3: Configure .env

```bash
cp .env.example .env
# Edit .env with your actual keys
```

```env
MONGO_URI=mongodb+srv://myuser:mypass@cluster0.xxxxx.mongodb.net/gigshield?retryWrites=true&w=majority
RAZORPAY_KEY_ID=rzp_test_XXXXXXXXXXXXXXXXXX
RAZORPAY_KEY_SECRET=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
RAZORPAY_ACCOUNT_NUMBER=XXXXXXXXXXXXXXXXXX
```

---

## ⚡ Step 4: Run the App

```bash
cd gigshield/backend
pip install -r requirements.txt
python app.py
```

App runs at: **http://localhost:5000**

---

## 📱 Pages

| URL | Description |
|-----|-------------|
| `/` | Main app — Register + Pay Premium |
| `/dashboard` | Worker dashboard — View coverage + Pay |
| `/admin` | Admin panel — Manage workers, trigger payouts |

---

## 💰 Payment Flow (Real Money)

### Premium Collection:
```
Worker → Razorpay Checkout → Payment Verified → MongoDB Pool += amount
```
1. Worker registers → `POST /api/worker/register`
2. App creates Razorpay order → `POST /api/payment/create-order`
3. Worker pays via Razorpay (card/UPI/netbanking)
4. Webhook/callback verifies → `POST /api/payment/verify`
5. Amount added to `pool` collection in MongoDB

### Payout Disbursement:
```
Weather Trigger → Fraud Check → Razorpay Payout API → Worker UPI
```
1. Weather API detects disruption → `POST /api/trigger/check`
2. Admin triggers zone payout → `POST /api/admin/simulate-trigger`
3. Per-worker fraud check runs
4. Razorpay Payout API transfers to worker's UPI ID
5. Pool balance decremented in MongoDB

---

## 🤖 AI/ML Features

- **District Risk Scoring**: Based on historical IMD/Kaggle rainfall data
- **Dynamic Premium**: `earnings × risk_rate × platform_mult × season_mult`
- **Fraud Detection**: GPS continuity, duplicate 24h check, trust tiers
- **Parametric Auto-Trigger**: OpenMeteo API, 6 trigger types
- **Confidence Scoring**: Multi-source validation (≥80% to trigger)

---

## 🌦 Weather API (Free, No Key Needed)

Uses **OpenMeteo** — completely free, no API key required:
```
https://api.open-meteo.com/v1/forecast?latitude=X&longitude=Y&hourly=precipitation,temperature_2m,windspeed_10m
```

---

## 🧪 Testing Flow

1. Register a worker with phone `9876543210` in Chennai
2. Pay premium via Razorpay test card: `4111 1111 1111 1111` CVV: `123`
3. Go to Admin → Simulate Trigger → Chennai → Heavy Rainfall
4. Worker receives simulated UPI payout in their history

---

## 📊 District Risk Scores (Sample)

| District | Risk Score | Tier |
|----------|-----------|------|
| Mumbai Suburban | 128 | HIGH |
| Ratnagiri | 118 | HIGH |
| Nagpur | 105 | HIGH |
| Chennai | 95 | MEDIUM |
| Bangalore | 78 | MEDIUM |
| Coimbatore | 65 | LOW |

---

## 🔐 Security Notes

- HMAC-SHA256 signature verification for all Razorpay payments
- Fraud detection: GPS, duplicate, velocity, trust tier checks
- Never stores raw card details (all via Razorpay)
- UPI IDs stored but never raw bank credentials
#

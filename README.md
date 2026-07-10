It started as a fraud detection assignment for Flipkart Grid.
live demo https://returndetection-ks5p.vercel.app/


https://returns-shield-2.onrender.com

The brief was simple: build something that catches fake returns. A customer buys a ₹80,000 iPhone, sends back a brick in a box, and gets a full refund. E-commerce companies lose **$103 billion every year** to exactly this kind of scam.

I built the fraud detector. XGBoost trained on 590,540 real transactions from the IEEE-CIS dataset. ROC-AUC 0.91. It worked.

But then I thought — the same AI that detects fraud on *returns* can detect fraud on *listings*. What if every second-hand product on OLX had to pass an AI damage scan before it went live? What if buyers could actually *see* where the scratches are, visualized on the photo itself?

That's when Returns Shield became **ShieldMart** — India's first AI-verified used goods marketplace.

The fraud detection didn't go away. It became the backbone. Every item a seller lists gets scored for fraud risk and scanned for physical damage before a single buyer sees it. The seller can't fake the condition. The AI already knows.

---

## What ShieldMart Does

ShieldMart has three sections that work together:

### 🔍 Price Comparison
Type any product name — iPhone 15, Sony headphones, Nike Air Max — and ShieldMart compares prices across 6 major Indian retailers. Electronics searches show Amazon, Flipkart, Croma, Reliance Digital, Vijay Sales, and Tata Cliq. Fashion searches swap in Myntra, Meesho, and Ajio instead (because those platforms don't sell electronics — a detail most comparison tools ignore). Results are sorted lowest to highest. The cheapest option gets a trophy badge. You click and land directly on that product's search page on the retailer's website.

### 🛍️ Used Goods Marketplace
The core product. Sellers list used items — phones, laptops, clothes, shoes, furniture, books. But before the listing goes live, two things happen automatically:

**Fraud scoring.** The seller's behavioral profile is run through the XGBoost model. Account age, transaction patterns, email domain — 13 signals combined into a single fraud probability score between 0 and 100%.

**Damage detection.** The product photo is analyzed by the computer vision model. It detects four types of damage independently: stains, tears, scratches, and dents. Each gets its own percentage score. Then a Grad-CAM heatmap is drawn on the actual photo — orange for stains, red for tears, purple for scratches, blue for dents — showing buyers exactly where the damage is.

If both scores are low, the listing goes live immediately. If either is borderline, it goes to manual review. If fraud risk is high, it's rejected. The whole thing takes under three seconds.

Buyers see every score. There's no hiding. The listing shows damage percentage, fraud risk, seller city, age of the item, original purchase price, reason for selling, and which platform it was bought from. You can add to cart or buy immediately via Razorpay.

When someone buys, ShieldMart acts as mediator. We physically verify the item before shipping. Clothes get dry-cleaned and ironed. Electronics get factory-reset, performance-tested, and shipped with the original purchase receipt and warranty documents. If it's not as described, you return it within 3–7 days.

### ↩️ Returns Shield (B2B Dashboard)
The original project, now a module. E-commerce operations teams can submit customer return requests — order ID, reason, product photo, transaction metadata — and get an immediate AI verdict: Auto Approve, Manual Review, or Reject & Investigate. The ops dashboard shows every return with expandable fraud/damage breakdowns, and agents can approve or reject manual cases one click.

---

## Architecture

```
                          ┌─────────────────────────────────────┐
                          │         React 18 Frontend            │
                          │                                      │
                          │  Home · Compare · Marketplace        │
                          │  Sell · Returns · Dashboard          │
                          │                                      │
                          │  ┌─────────┐  ┌──────────────────┐  │
                          │  │  Cart   │  │   Chatbot (AI)   │  │
                          │  └─────────┘  └──────────────────┘  │
                          └──────────────┬──────────────────────┘
                                         │ Axios / REST
                          ┌──────────────▼──────────────────────┐
                          │         FastAPI Backend               │
                          │                                      │
                          │  /search    /marketplace   /returns  │
                          │  /fraud                              │
                          │                                      │
                          │  ┌──────────────────────────────┐   │
                          │  │   ML Services                │   │
                          │  │                              │   │
                          │  │  XGBoost Fraud Scorer        │   │
                          │  │  ROC-AUC 0.91               │   │
                          │  │  590,540 training samples    │   │
                          │  │  13 behavioral features      │   │
                          │  │                              │   │
                          │  │  CV Damage Detector          │   │
                          │  │  EfficientNetB0              │   │
                          │  │  4 damage types (independent)│   │
                          │  │  Grad-CAM heatmap overlay    │   │
                          │  └──────────────────────────────┘   │
                          └──────────────┬──────────────────────┘
                                         │ SQLAlchemy ORM
                          ┌──────────────▼──────────────────────┐
                          │       PostgreSQL 16 (Docker)         │
                          │                                      │
                          │  customers          orders           │
                          │  return_requests                     │
                          │  marketplace_listings                │
                          │  marketplace_purchases               │
                          │  cart_items                          │
                          └─────────────────────────────────────┘
                                         │
                          ┌──────────────▼──────────────────────┐
                          │       /uploads/ (Static Files)       │
                          │   Product photos served via FastAPI  │
                          └─────────────────────────────────────┘
```

---

## Every Feature, Listed

**Price Comparison**
- Category-aware retailer selection (electronics ≠ fashion ≠ grocery)
- Electronics: Amazon, Flipkart, Croma, Reliance Digital, Vijay Sales, Tata Cliq
- Fashion: Amazon, Flipkart, Myntra, Meesho, Ajio, Snapdeal
- Grocery: Amazon, BigBasket, Blinkit, Zepto, JioMart, Flipkart
- 60+ real Indian market prices hardcoded for accuracy (iPhone 15: ₹79,990, MacBook Air M2: ₹1,14,900, etc.)
- Prices sorted low to high always — lowest gets 🏆 badge
- Visual price range bar with retailer dots
- Ratings, reviews, delivery time, EMI availability, free delivery tag
- Direct "View on [Retailer]" links open the product search page
- Disclaimer that prices are market estimates, not live scraped

**Marketplace**
- Browse by category filter
- Real product photos (uploaded by seller, saved to disk, served as static files)
- Photos visible in listing cards, product detail modal, and cart sidebar
- AI Verified ✓ badge on every live listing
- Damage percentage shown on listing card
- Savings % badge when discount > 10% off original MRP
- Hover zoom on product images
- Click to open full product modal with photo, AI report, seller info, delivery promise
- Condition labels: Like New / Good / Fair / Poor
- Sustainability banner: "Give it a second life. Shop second. Save first."
- Add to cart or Buy Now from modal
- Buy Now triggers full delivery address form + Razorpay checkout flow
- Order success shows tracking ID, estimated delivery, return policy
- Return policy: 7 days for electronics, 3 days for clothing/footwear

**Sell Item Flow**
- 3-step progress indicator: Fill Details → AI Verification → Go Live
- Seller info, product info, condition, age, original price, reason for selling
- Photo upload with live preview
- 5% platform fee shown before submission
- Step 2: visible AI analysis progress screen (XGBoost → CV → Grad-CAM → Risk)
- Step 3: result screen with fraud score, damage breakdown, Grad-CAM heatmap
- Delivery promise by category:
  - Clothing: professionally washed, ironed, sealed ShieldMart bag, 3-day return
  - Electronics: factory reset + tested, original receipt + all legal documents, 7-day return
  - Other: inspected, cleaned, sanitized, 3-day return
- Sustainability quote: "One person's old is another person's treasure"
- Listing goes live immediately if approved, pending if borderline, rejected if high risk

**Cart**
- Session-based cart (no login required)
- Thumbnail product images in cart sidebar
- Real-time remove
- Subtotal + 2% convenience fee breakdown
- Cart count badge in header

**Returns Shield (Ops Module)**
- 6-point eligibility checklist before submission
- Upload product return photo
- XGBoost fraud scoring + CV damage detection
- Grad-CAM heatmap on submitted photo
- Risk triage: Auto Approve / Manual Review / Reject & Investigate
- Ops dashboard: all returns table, expandable rows, fraud/damage bars
- Approve / Reject buttons for manual review cases
- Average fraud score widget with model stats

**AI Chatbot**
- Floating button, chat window with quick reply pills
- Answers questions about returns, fraud detection, damage types, refund timelines, policies
- Chatbot connected to backend `/returns/chat` endpoint

**Operations Dashboard**
- Animated stat cards (count-up effect)
- Risk distribution breakdown
- Recent fraud scores bar chart
- Average fraud score with model metadata panel

---

## ML Model Details

### XGBoost Fraud Scorer

| Property | Value |
|---|---|
| Dataset | IEEE-CIS Fraud Detection (Kaggle) |
| Training samples | 590,540 transactions |
| Fraud rate | 3.5% (severe class imbalance) |
| Imbalance handling | scale_pos_weight = 27.58 |
| ROC-AUC | **0.91** |
| Recall (fraud class) | 0.77 |
| Boosting rounds | 300 |
| Max depth | 6 |
| Learning rate | 0.05 |

**13 features used:**
transaction_amt, log_amt, card1, card2, transaction_hour, email_domain_freq, has_device_info, dist1, C1, C2, C5, C13, C14

**Top 3 by importance:**
1. C5 — count-based signal: 0.277
2. C1 — count-based signal: 0.193
3. C14 — count-based signal: 0.184

### Computer Vision Damage Detector

- Base model: EfficientNetB0 (transfer learning)
- Training data: SDNET2018 damage detection dataset
- Output: 4 independent damage scores (stains, tears, scratches, dents)
- Grad-CAM: Canvas-drawn ellipse overlays on uploaded photo
- Color coding: orange = stain, red = tear, purple = scratch, blue = dent
- Threshold: damage_score > 0.30 = damaged

---

## Project Structure

```
returns-shield/
├── backend/
│   ├── app/
│   │   ├── main.py                   # FastAPI app + CORS + static file serving
│   │   ├── database.py               # SQLAlchemy setup + session
│   │   ├── models.py                 # 6 database models
│   │   ├── schemas.py                # Pydantic request/response schemas
│   │   ├── seed.py                   # Demo data seeder (6 orders + 6 returns)
│   │   ├── routers/
│   │   │   ├── fraud.py              # POST /fraud/score
│   │   │   ├── returns.py            # /returns/* + chatbot
│   │   │   ├── marketplace.py        # /marketplace/* (list, buy, cart, stats)
│   │   │   └── search.py             # GET /search/prices
│   │   └── services/
│   │       ├── fraud_service.py      # XGBoost model inference
│   │       └── cv_service.py         # CV damage detection + hotspot analysis
│   ├── ml/
│   │   ├── fraud_model/
│   │   │   ├── train.py
│   │   │   ├── feature_engineering.py
│   │   │   └── fraud_model.pkl       # Trained model (tracked in git)
│   │   └── cv_model/
│   │       ├── train_damage_model.py
│   │       ├── damage_model.tflite   # TFLite model (tracked in git)
│   │       └── class_names.json
│   ├── uploads/                      # Product photos (gitignored)
│   └── requirements.txt
├── frontend/
│   └── src/
│       ├── App.jsx                   # All components (972 lines)
│       └── index.css
├── docker-compose.yml                # PostgreSQL 16
└── README.md
```

---

## API Reference

| Method | Endpoint | Description |
|---|---|---|
| GET | `/` | Health check |
| GET | `/search/prices?q=iPhone 15` | Price comparison |
| GET | `/marketplace/listings?category=electronics` | Browse listings |
| POST | `/marketplace/list` | Submit item (multipart + image) |
| GET | `/marketplace/listings/{id}` | Single listing detail |
| POST | `/marketplace/buy/initiate` | Start Razorpay purchase |
| POST | `/marketplace/buy/confirm` | Confirm payment |
| POST | `/marketplace/cart/add` | Add to cart |
| GET | `/marketplace/cart/{session_id}` | Get cart |
| DELETE | `/marketplace/cart/{sid}/{lid}` | Remove from cart |
| GET | `/marketplace/stats` | Marketplace stats |
| POST | `/marketplace/return/request` | Request return on purchase |
| POST | `/returns/analyze` | Submit return for AI analysis |
| GET | `/returns/all` | All return requests |
| GET | `/returns/stats` | Aggregated stats |
| GET | `/returns/orders` | List seeded orders |
| PATCH | `/returns/{id}/decide` | Approve or reject |
| POST | `/returns/chat` | AI chatbot |
| POST | `/fraud/score` | Direct fraud scoring |

---

## Running Locally

### Requirements
- Python 3.11 (Conda)
- Node.js 18+
- Docker Desktop

### Setup

```bash
# Clone
git clone https://github.com/aditiraj0129-netizen/returns-shield.git
cd returns-shield

# Start database
docker compose up -d

# Backend
cd backend
conda create -n returns-shield python=3.11 -y
conda activate returns-shield
pip install -r requirements.txt
brew install libomp          # macOS only, for XGBoost

# Seed demo data
python -m app.seed

# Start API
uvicorn app.main:app --reload
# → http://localhost:8000
# → http://localhost:8000/docs

# Frontend (new terminal)
cd ../frontend
npm install
npm run dev
# → http://localhost:5173
```

---

## Deployment

| Service | Platform | URL |
|---|---|---|
| Frontend | Vercel | returndetection-ks5p.vercel.app |
| Backend | Render | returns-shield-2.onrender.com |
| Database | Render PostgreSQL | (managed) |

> **Note:** Render free tier sleeps after 15 minutes of inactivity. First request after sleep takes ~30 seconds to wake up. The frontend handles this gracefully.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 18, Vite, Axios |
| Backend | FastAPI, Python 3.11, SQLAlchemy, Pydantic |
| ML — Fraud | XGBoost, scikit-learn, pandas, numpy |
| ML — CV | EfficientNetB0, TensorFlow/Keras, TFLite, PIL |
| Database | PostgreSQL 16 (Docker locally, Render in production) |
| Payment | Razorpay (test mode) |
| Deployment | Vercel (frontend), Render (backend + DB) |
| Infrastructure | Docker Compose |

---

## Interview Questions I Can Answer

**Why XGBoost and not a neural network?**

Tabular behavioral data with 13 engineered features responds better to gradient boosting than deep learning. More importantly, XGBoost gives me feature importances — so when an ops agent asks "why was this return flagged?", I can tell them C5 (count-based transaction signal) was the top contributor. Explainability matters in fraud.

**How did you handle the 27:1 class imbalance?**

`scale_pos_weight = 27.58` — tells XGBoost to penalize missing a fraud case 27x more heavily than a false positive. This matches the real cost asymmetry: a missed fraud costs the company far more than a legitimate return that gets manually reviewed.

**What's the business moat of ShieldMart vs OLX?**

OLX is self-reported condition. Anyone can write "good condition" next to a cracked screen. ShieldMart runs the photo through a damage detection model before the listing goes live. Buyers see actual damage scores, not seller claims. That trust differential is the product.

**Why show Grad-CAM to buyers?**

Because "42% damage score" means nothing. "Stain detected in top-right corner of the jacket" means everything. The heatmap makes AI output human-readable. It also makes sellers more honest — they know the AI will visually annotate whatever they're hiding.

**How does the 3-bucket triage system work?**

Auto Approve: fraud score < 20% AND item is damaged (legitimate damage claim). Manual Review: fraud score 20–60% OR item claims damage but CV disagrees. Reject: fraud score > 60%. The thresholds were set to match the cost structure — false negatives (missed fraud) cost more than false positives (unnecessary manual review).

---

## About


Final Year B.Tech CSE (AI/ML) ·   

Solved 250+ DSA problems (LeetCode + GFG)



---

*This project was built for Flipkart Grid 6.0. The goal was to demonstrate that production-grade ML systems thinking — not just model training — is what separates good candidates from great ones.*
EOF
echo "done — $(wc -l < /home/claude/README_final.md) lines"
Output





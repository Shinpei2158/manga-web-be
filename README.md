# MangaWord - Backend API

A robust, modular backend system for an online comic (manga) and light novel reading platform. Built with NestJS and MongoDB, the system features a modular architecture, fine-grained access control, FinTech payment integrations, automated financial management/tax compliance, and gamification mechanics.

---

## 🛠️ Tech Stack & Key Libraries
* **Core Framework:** NestJS (TypeScript)
* **Database:** MongoDB with Mongoose (ODM)
* **Real-Time Communications:** Socket.IO (for instant comments and live notifications)
* **AI Engine:** Google Gemini AI SDK (`@google/generative-ai` for automated content moderation and AI assistance)
* **Cloud Storage:** Cloudinary API (storing transaction payment receipts and author KYC identification documents)
* **Notifications:** Firebase Admin SDK (FCM push notifications) & Nodemailer (SMTP emails)
* **Financial Reporting:** ExcelJS (generating spreadsheet reports), Archiver (packaging annual reports into ZIP files)
* **Document Generation:** PDFKit (exporting PDF documents)

---

## Core Contributions & API Specifications

I engineered the following core backend modules, including database schemas, business logic services, event tracking, and secure API endpoints:

### 1. VNPAY Payment Gateway Integration (`/api/vnpay`)
Processes secure point deposits using VNPAY Sandbox:
* **Payment URL Generation (`POST /api/vnpay/create-payment-url`):**
  * Validates the client's selected package, calls `topupService.getEffectivePoints` to fetch the point credits (handling double-point reward rules).
  * Creates a unique `txnRef` mapping using: `[8 characters of hashed userId][timestamp]` to guarantee it remains within VNPAY's 32-character limits.
  * Formats transaction variables according to VNPAY 2.1.0 specifications and hashes the query using `sha512` with the server's secret key.
  * Inserts a transaction record in the database with a default `pending` status and returns the VNPAY checkout URL.
* **Redirection Callback (`GET /api/vnpay/return`):**
  * Receives transaction parameters from VNPAY's response.
  * Re-hashes query parameters using HMAC-SHA512 to verify the signature (`vnp_SecureHash`).
  * If the signature is verified and VNPAY status reports success (`vnp_ResponseCode === '00'` and `vnp_TransactionStatus === '00'`), calls `topupService.handlePaymentSuccess` to set the database status to `success` and credit points to the user's wallet.
  * If validation fails or the transaction fails, marks status as `failed`.
  * Redirects the user back to the client interface (`CLIENT_URL`) with result parameters (e.g. `?payment=success&txn=...` or `?payment=failed`).

### 2. Daily Check-in Rewards (`/api/checkin`)
Implements weekly progressive check-ins to boost user retention:
* **Core Logic:**
  * Checks are logged per calendar week, stored in an array of booleans (`checkins: [Sun, Mon, Tue, Wed, Thu, Fri, Sat]`) in the `Asia/Ho_Chi_Minh` timezone.
  * Utilizes `DAILY_REWARD_CONFIG` to grant rewards dynamically based on user role:
    * **Readers (Role User):** Earn 1 point (Mon-Tue), 2 points (Wed-Thu), 3 points (Fri), 4 points (Sat), 5 points (Sun).
    * **Authors (Role Author):** Earn 10 to 40 author points (increasing progressively from Mon to Sun).
* **Endpoints:**
  * `POST /api/checkin/today`: Processes daily check-in attendance and issues points to the user/author wallet.
  * `GET /api/checkin/status`: Fetches check-in array state and check eligibility (`canCheckin`) for the current day.

### 3. Achievement System (`/api/achievements`)
Automates user engagement tracking and milestone incentives:
* **Database Models:** Designed MongoDB schemas for `Achievement` (milestones metadata, target thresholds, and points reward payload) and `AchievementProgress` (user-specific tracking status).
* **Event-Driven Tracking (`AchievementEventListener`):**
  * Implemented NestJS `@nestjs/event-emitter` to asynchronously capture events from different modules.
  * Listens for: `comment_count` (comments), `follow_count_increase`/`decrease` (following users/stories), `follower_count_increase`/`decrease` (author followers), `rating_count` (ratings), `favorite_story_count` (favorites), `story_create_count` (created stories), `chapter_create_count` (created chapters), and `donation_spend_count` (donated points).
  * Automatically updates the progress counter in `AchievementProgress` and updates `isCompleted = true` when thresholds are met.
* **Reward Claims (`POST /api/achievements/:id/claim`):** Validates completed status and increments points (`point` for readers or `author_point` for authors), changing `rewardClaimed = true` to prevent double-claiming.
* **Progress Synchronization (`POST /api/achievements/sync`):** Sets up progress tracking entries for all users when new achievements are published.

### 4. Financial Management Suite (`Role.FINANCIAL_MANAGER = 'financial_manager'`)
A secured collection of APIs protected by role guards to manage billing, payouts, and taxes:

#### A. Author KYC Profiles (`/api/payout-profile`)
* Takers must supply bank and identity details (ID card/passport images) before requesting withdrawals.
* `GET /api/payout-profile/list`: Retrieves KYC files with status filters.
* `PATCH /api/payout-profile/approve/:id`: Approves the profile, enabling withdraws.
* `PATCH /api/payout-profile/reject/:id`: Rejects the profile (rejection reason is required).

#### B. Author Withdrawals (`/api/withdraw`)
* Authors exchange accumulated author points for cash.
* `GET /api/withdraw`: Lists withdrawals with support for year/month filtering, status, name search, and pagination.
* `PATCH /api/withdraw/:id/approve` & `PATCH /api/withdraw/:id/reject`: Approves or rejects the withdraw request.

#### C. Payout Settlement (`/api/payout-settlement`)
* Tracks payments for approved withdrawal requests in batches.
* `GET /api/payout-settlement`: Lists payout settlements.
* `PATCH /api/payout-settlement/pay/:id`: Marks a settlement as paid. The accountant uploads bank transaction receipts/batch files (saved on Cloudinary).
* `GET /api/payout-settlement/export`: Generates an Excel spreadsheet of payouts in a date range using `ExcelJS` and returns a downloadable link.
* `PATCH /api/payout-settlement/cancel/:id`: Cancels a payout settlement, reverting withdrawal request statuses.
* `PATCH /api/payout-settlement/update-paid/:id`: Edits paid proof files or updates notes for a completed settlement.

#### D. Tax Settlement & Compliance (`/api/tax-settlement`)
* Calculates 10% withholding tax at source for author payouts exceeding 2,000,000 VND, keeping the platform compliant with Vietnamese tax regulations.
* `GET /api/tax-settlement`: Queries tax settlements.
* `POST /api/tax-settlement/export`: Generates tax reports:
  * **QUARTERLY:** Exports a single quarterly declaration Excel sheet.
  * **ANNUAL:** Creates separate quarterly sheets and bundles them into a `.zip` archive using `archiver`.
* `PATCH /api/tax-settlement/pay/:id`: Marks tax as paid to the state treasury. The accountant logs the official tax receipt number and uploads receipt documents corresponding to each author in the settlement.
* `PATCH /api/tax-settlement/update-paid/:id`: Edits uploaded state receipt files or receipt serial numbers.

---

## ⚙️ Environment Variables Configuration (.env)

Create a `.env` file in the root folder of the backend:

```env
# Application Settings
PORT=3000
JWT_SECRET=your_jwt_secret_key
CLIENT_URL=http://localhost:3000

# Database
DATABASE_URL=mongodb+srv://<username>:<password>@cluster.mongodb.net/mangaword

# VNPAY Sandbox Configurations
VNP_TMNCODE=your_vnpay_tmn_code
VNP_HASHSECRET=your_vnpay_secure_hash_secret
VNP_URL=https://sandbox.vnpayment.vn/paymentv2/vpcpay.html
VNP_RETURNURL=http://localhost:3000/api/vnpay/return

# Cloudinary Storage Credentials
CLOUDINARY_NAME=your_cloudinary_name
CLOUDINARY_API_KEY=your_cloudinary_api_key
CLOUDINARY_API_SECRET=your_cloudinary_api_secret

# Gemini AI Credentials
GEMINI_API_KEY=your_google_gemini_api_key

# Firebase Cloud Messaging Credentials
PROJECT_ID=your_firebase_project_id
PRIVATE_KEY="your_firebase_private_key"
CLIENT_EMAIL=your_firebase_client_email

# SMTP Configurations
SMTP_USER=your_smtp_email@gmail.com
SMTP_PASS=your_smtp_app_password
```

---

## Getting Started Locally

1. Install project dependencies:
   ```bash
   npm install
   ```
2. Start the NestJS API server in development mode:
   ```bash
   npm run start:dev
   ```
   The backend API runs at `http://localhost:3000`. You can test endpoints using Postman or connect the frontend client.

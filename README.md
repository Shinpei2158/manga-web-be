# MangaWord - Backend API

A robust, enterprise-grade backend system for a cross-platform web application that delivers both comics (manga) and light novels. The platform integrates smart AI assistance, localized tax compliance processing, and gamified user engagement mechanics.

> **Frontend Repository:** [Explore MangaWord Frontend](https://github.com/Shinpei2158/manga-web-fe)

---

## Tech Stack & Architecture
- **Core Framework:** NestJS (TypeScript) - Structured with clean, modular architecture.
- **Database:** MongoDB with Mongoose (ODM).
- **Real-Time Communications:** Socket.IO for instant user interactions.
- **Third-Party Integrations:** VNPAY Payment Gateway, Firebase Admin SDK (Push Notifications), Nodemailer (SMTP).
- **AI Engine:** Google Gemini API.

---

## Key System Features

- **Content & Licensing Management:** Complete CRUD and workflows for stories, chapters, genres, and authors, supporting platform policies and license validation.
- **Role-Based Access Control (RBAC):** Strict authentication & authorization for Multiple Roles (Admin, Author, Accountant, and Reader).
- **AI Integration:** Content analysis, automated tagging, and smart search capabilities powered by Google Gemini.
- **Real-time Engagement:** Instant comments, replies, and notifications.

---

## My Core Contributions (Backend Architecture & Business Logic)

As the primary Backend Developer, I engineered the core financial and engagement modules of the system:

### 1. Payment Gateway Integration (VNPAY)
- Implemented **VNPAY Sandbox** integration to handle secure point deposits and cash-out/withdrawal workflows.
- Designed **Payment Redirect processing loops** to securely read transaction return parameters and automate real-time user balance updates.

### 2. Automated Tax Compliance Module
- Developed a dynamic tax calculation engine driven by custom **Business Rules** and regional tax compliance laws.
- Processed automated withholding tax deductions during author payout/withdrawal sequences.
- Built **Data Export features (CSV/Excel)** to streamline transaction histories, helping the accounting team with seamless tax declaration and fiscal settlements.

### 3. Gamification & Retention Logic
- Designed and optimized database schemas for a **Daily Check-in Reward** system.
- Engineered a scalable **Achievement System** that tracks user behavior and unlocks milestones dynamically, boosting overall user engagement.

---

## ⚙️ Environment Variables Configuration

To run this project, create a `.env` file in the root directory and configure the following variables:

```env
# Application Configuration
PORT=3000
JWT_SECRET=your_jwt_secret_key
CLIENT_URL=http://localhost:3000

# Database
DATABASE_URL=mongodb+srv://...

# VNPAY Payment Gateway (Sandbox)
VNP_TMNCODE=your_vnpay_terminal_code
VNP_HASHSECRET=your_vnpay_hash_secret
VNP_URL=[https://sandbox.vnpayment.vn/paymentv2/vpcpay.html](https://sandbox.vnpayment.vn/paymentv2/vpcpay.html)
VNP_RETURNURL=http://localhost:3000/api/vnpay/callback

# Google Gemini AI
GEMINI_API_KEY=your_gemini_api_key
GEMINI_MODEL=gemini-pro

# Firebase Admin SDK (Notifications)
PROJECT_ID=your_firebase_project_id
PRIVATE_KEY="your_firebase_private_key"
CLIENT_EMAIL=your_firebase_client_email

# SMTP Configuration (Email Notifications)
SMTP_USER=your_email@gmail.com
SMTP_PASS=your_app_password

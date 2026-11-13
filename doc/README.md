# Medusa DTC Store – Simple User Guide

Welcome to your Medusa v2 & Next.js Monorepo. This guide contains everything you need to start, run, and manage your e-commerce store.

---

## 1. Quick Access & Credentials (অ্যাকাউন্ট ও লগইন তথ্য)

| Service | URL | Default Credentials | Description |
| :--- | :--- | :--- | :--- |
| **Admin Dashboard** | [http://localhost:9000/app](http://localhost:9000/app) | **Email:** `mdaashikur@gmail.com`<br>**Password:** `admin123456` | স্টোর ম্যানেজমেন্ট, প্রোডাক্ট যোগ, অর্ডার দেখা |
| **Storefront (Shop)** | [http://localhost:8000](http://localhost:8000) | *Customers register from storefront* | ক্রেতাদের কেনাকাটার ওয়েবসাইট |
| **Backend API** | [http://localhost:9000](http://localhost:9000) | – | Medusa REST API |

> [!TIP]
> Admin প্যানেলে লগইন করার পর **Settings → Profile** থেকে আপনি যেকোনো সময় আপনার নাম ও পাসওয়ার্ড পরিবর্তন করে নিতে পারবেন।

### How to Create Another Admin User (নতুন অ্যাডমিন তৈরি করার নিয়ম)
Admin প্যানেলে সরাসরি সাইন-আপ বাটন থাকে না। নতুন অ্যাডমিন তৈরি করতে টার্মিনালে নিচের কমান্ডটি রান করুন:
```bash
cd apps/backend
npx medusa user -e newadmin@example.com -p password123
```

---

## 2. Daily Development (প্রজেক্ট চালু করার নিয়ম)

### Step 1: Ensure PostgreSQL is running
আপনার কম্পিউটারে PostgreSQL সার্ভিস রানিং থাকতে হবে (ডিফল্ট পোর্ট `5432`)।

### Step 2: Start the Servers
প্রজেক্টের রুট ডিরেক্টরি (`medusa-stable`) থেকে নিচের কমান্ড দিন:
```bash
npm run dev
```
এটি একই সাথে Medusa Backend (Port 9000) এবং Next.js Storefront (Port 8000) চালু করবে।

### অন্যান্য দরকারী কমান্ড:
* **শুধু ব্যাকএন্ড চালু করতে:**
  ```bash
  npm run backend:dev
  ```
* **শুধু স্টোরফ্রন্ট চালু করতে:**
  ```bash
  npm run storefront:dev
  ```
* **ডেমো প্রোডাক্ট ও ডিফল্ট ডাটা সিড করতে:**
  ```bash
  npm run backend:seed
  ```
* **ডাটাবেজ মাইগ্রেশন চালাতে:**
  ```bash
  cd apps/backend
  npx medusa db:migrate
  ```

---

## 3. How the System Works (প্রজেক্ট স্ট্রাকচার)

```text
medusa-stable/
├── apps/
│   ├── backend/       # Medusa v2 Server (Database, Business Logic, Admin Dashboard)
│   └── storefront/    # Next.js 15 App (Public eCommerce Frontend)
├── doc/               # Documentation & Guides
└── package.json       # Monorepo configuration (Turbo + npm)
```

* **Backend (`apps/backend`):** প্রোডাক্ট, ইনভেন্টরি, কাস্টমার, অর্ডার এবং পেমেন্ট নিয়ন্ত্রণ করে। এর ডাটাবেজ কনফিগারেশন আছে `apps/backend/.env` ফাইলে।
* **Storefront (`apps/storefront`):** ক্রেতারা যা দেখেন। এটি ব্যাকএন্ডের সাথে যুক্ত থাকে **Publishable API Key** দিয়ে, যা `apps/storefront/.env.local` ফাইলে সংরক্ষিত।

---

## 4. Common Admin Tasks (স্টোর ম্যানেজমেন্টের মূল কাজগুলো)

### ১. নতুন প্রোডাক্ট যোগ করা (Add a Product):
1. [http://localhost:9000/app](http://localhost:9000/app)-এ লগইন করুন।
2. বাম পাশের মেনু থেকে **Products**-এ ক্লিক করুন।
3. উপরের ডানপাশে **New Product** বাটনে ক্লিক করুন।
4. Title, Description, Variants (সাইজ, কালার ইত্যাদি), Price এবং Images যোগ করে **Publish** করুন।
5. স্টোরফ্রন্ট [http://localhost:8000](http://localhost:8000)-এ গিয়ে পেজ রিফ্রেশ করলেই প্রোডাক্টটি দেখতে পাবেন।

### ২. অর্ডার ম্যানেজমেন্ট (Manage Orders):
* স্টোরফ্রন্ট থেকে কোনো কাস্টমার চেকআউট সম্পন্ন করলে Admin প্যানেলের **Orders** সেকশনে নতুন অর্ডার আসবে।
* সেখান থেকে আপনি পেমেন্ট স্ট্যাটাস এবং ফুলফিলমেন্ট (Fulfillment / Shipping) হ্যান্ডেল করতে পারবেন।

### ৩. শিপিং ও কারেন্সি সেটিংস (Shipping & Regions):
* **Settings → Regions**-এ গিয়ে আপনি যেসব দেশে পণ্য পাঠাতে চান (যেমন US, EU, BD ইত্যাদি) তাদের জন্য কারেন্সি এবং ডেলিভারি চার্জ নির্ধারণ করতে পারবেন।

---

## 5. Troubleshooting (সমস্যা ও সমাধান)

### সমস্যা ১: Admin Dashboard এ লগইন করতে পারছে না ("Invalid email or password")
* সমাধান: টার্মিনাল খুলে নতুন একটি অ্যাডমিন ইউজার তৈরি করুন:
  ```bash
  cd apps/backend
  npx medusa user -e mdaashikur@gmail.com -p admin123456
  ```

### সমস্যা ২: স্টোরফ্রন্টে Backend 400 Error দেখাচ্ছে
* সমাধান: এর মানে `apps/storefront/.env.local` ফাইলে `NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY` সঠিকভাবে সেট নেই। নিশ্চিত করুন যে কী-টি `apps/backend`-এর ডাটাবেজের পাবলিশেবল কী-এর সাথে মিলছে।

### সমস্যা ৩: পোর্ট অলরেডি ইন ইউজ (Port 9000 or 8000 is already in use)
* PowerShell-এ আগের প্রসেস কিল করতে:
  ```powershell
  Get-NetTCPConnection -LocalPort 8000, 9000 -ErrorAction SilentlyContinue | ForEach-Object { Stop-Process -Id $_.OwningProcess -Force }
  ```

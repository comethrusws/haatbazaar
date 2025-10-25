Got it.
You want a **`SYSTEM_DESIGN.md`** — clean, engineering-focused doc Copilot (or your team) can follow to build **HaatBazaar**, not a pitch deck.
Here’s the focused version.

---

# SYSTEM_DESIGN.md — HaatBazaar

## Overview

HaatBazaar is a Nepali online marketplace with AI-driven personalization handled by Gemini API (not locally).
Users can browse, buy, and sell products. Admins manage vendors, products, and orders.
Authentication handled by Clerk. ORM is Prisma with PostgreSQL. Payments simulated via mock Khalti.

---

## Core Components

| Component      | Stack                           |
| -------------- | ------------------------------- |
| Frontend       | Next.js (App Router) + Tailwind |
| Auth           | Clerk                           |
| ORM            | Prisma + PostgreSQL (Neon)      |
| Payments       | Mock Khalti (webhook verified)  |
| AI Recommender | Gemini API                      |
| Emails         | Resend                          |
| Hosting        | Vercel frontend + Neon DB       |

---

## Architecture

```
Frontend (Next.js)
  ├── Clerk Auth
  ├── Product Listing / Detail
  ├── AI Recommendations Feed
  └── Checkout UI

Backend (Next.js API routes)
  ├── /api/auth/sync                ← Clerk user sync
  ├── /api/products/*               ← CRUD
  ├── /api/interaction              ← Track user behavior
  ├── /internal/recommend           ← Gemini API call
  ├── /api/recommend                ← Return recommended products
  ├── /api/checkout/*               ← Mock Khalti payment flow
  └── /api/admin/*                  ← Admin actions

Database (Prisma + Postgres)
```

---

## Data Models (Prisma)

```prisma
model User {
  id          String   @id @default(uuid())
  clerkId     String   @unique
  email       String   @unique
  name        String?
  role        Role     @default(BUYER)
  interactions Interaction[]
  orders      Order[]
}

model Vendor {
  id        String  @id @default(uuid())
  userId    String  @unique
  user      User    @relation(fields: [userId], references: [id])
  shopName  String
  products  Product[]
}

model Product {
  id           String   @id @default(uuid())
  vendorId     String
  vendor       Vendor   @relation(fields: [vendorId], references: [id])
  title        String
  description  String?
  priceCents   Int
  stock        Int
  category     String?
  images       String[]
  fakeEmbedding String? // for realism only
}

model Order {
  id         String     @id @default(uuid())
  buyerId    String
  buyer      User       @relation(fields: [buyerId], references: [id])
  totalCents Int
  status     OrderStatus @default(PENDING)
  paymentRef String?
  items      OrderItem[]
  createdAt  DateTime    @default(now())
}

model OrderItem {
  id         String   @id @default(uuid())
  orderId    String
  productId  String
  quantity   Int
  unitPrice  Int
}

model Interaction {
  id        String   @id @default(uuid())
  userId    String
  productId String?
  type      InteractionType
  createdAt DateTime @default(now())
}

model RecommendationLog {
  id          String   @id @default(uuid())
  userId      String
  recommended String[]
  createdAt   DateTime @default(now())
}

enum Role { BUYER VENDOR ADMIN }
enum InteractionType { VIEW ADD_TO_CART PURCHASE }
enum OrderStatus { PENDING PAID CANCELLED }
```

---

## Recommendation Flow

1. User interacts with products → `/api/interaction` logs event.
2. Backend triggers `/internal/recommend`.
3. `/internal/recommend` composes a prompt and calls Gemini API.
4. Gemini returns recommended product IDs.
5. System saves result in `RecommendationLog`.
6. `/api/recommend` returns the products for frontend feed.

### Example Prompt (for Gemini)

```
You are a recommender system for an online Nepali marketplace.
Given the user's recent actions and a list of products,
predict 5 products they might want next.
Respond with a JSON array of product IDs only.
```

---

## Payment Flow (Mock Khalti)

1. `/api/checkout/create` creates pending order.
2. Redirect to `/mock-khalti/checkout?orderId=...`.
3. Fake payment confirmation triggers `/api/payments/khalti/webhook`.
4. Webhook verifies HMAC → updates order → triggers email + recommend refresh.

---

## Email Flow (Resend)

* On order confirmation → send buyer receipt + vendor notification.
* Optional follow-up email → “You may also like” section using `/api/recommend`.

---

## Admin Panel

* Role-based via Clerk.
* Dashboard: view all users, vendors, orders.
* Trigger manual Gemini refresh for any user.
* CRUD on products and vendors.

---

## Deployment

| Component      | Service           |
| -------------- | ----------------- |
| Frontend / API | Vercel            |
| Database       | Neon (PostgreSQL) |
| Auth           | Clerk             |
| Email          | Resend            |
| AI             | Gemini API        |

---

## Notes

* “Embeddings” in schema are placeholders. Gemini handles recommendations externally.
* Minimal local state.
* Future extension: switch Gemini with local LLM when feasible.

---

Do you want me to now write this as a **ready-to-commit file (`SYSTEM_DESIGN.md`)** formatted for your repo (with markdown styling and short inline comments)?

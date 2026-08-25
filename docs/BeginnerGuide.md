# Beginner Guide – How This Medusa + Next.js Monorepo Works

## 1. What are the two parts?
- **`apps/backend`** – the **Medusa server**.  It talks to the database, runs the business rules, and exposes a **REST API** for the storefront and admin UI.
- **`apps/storefront`** – a **Next.js 15** web app.  It renders the public shop pages (product list, cart, checkout) and calls the Medusa API to get data.

Both live side-by-side in the same repository, managed by **TurboRepo** and **npm workspaces**.  A single `npm run dev` starts both at once.

---

## 2. How does the frontend talk to the backend?
1. **Publishable API Key** – a secret-like token that tells Medusa *"this is a public client, allow read-only calls"*.
   - Create it in the **Admin UI → Settings → Publishable API Keys**.
   - Copy the key into `apps/storefront/.env.local` as:
     ```env
     NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY=pk_XXXXXXXXXXXXXXXXXXXX
     ```
2. **Backend URL** – where the API lives (default `http://localhost:9000`).  Set it in the same env file:
   ```env
   NEXT_PUBLIC_MEDUSA_BACKEND_URL=http://localhost:9000
   ```
3. **Requests** – In the storefront code you will see calls like:
   ```ts
   const { data } = await sdk.client.fetch<{ products: Product[] }>("/store/products", { method: "GET" })
   ```
   The SDK automatically adds the `Authorization: Bearer <publishable-key>` header and the base URL from the env variables.

---

## 3. What does the backend expose? (Key endpoints)
| Path (GET)                     | What it returns                              | Where it is used |
|-------------------------------|----------------------------------------------|-----------------|
| `/store/products`              | List of all published products               | Product list page |
| `/store/products/:id`          | Details for a single product                 | Product page |
| `/store/regions`              | All shipping regions (used for tax & shipping) | Region picker |
| `/store/carts` (POST)         | Creates a new cart, returns cart ID           | Cart initialization |
| `/store/carts/:id` (GET)      | Current cart contents                         | Cart page |
| `/store/orders` (POST)        | Places an order (needs payment intent)        | Checkout |

All these routes live under **`apps/backend/src/api/store/.../route.ts`** – you do **not** add a router manually; the file-based routing system registers them automatically.

---

## 4. Database – how data is stored now
- **PostgreSQL** runs locally (connection string in `apps/backend/.env`).
- Medusa creates tables for `product`, `order`, `cart`, `region`, `customer`, etc.
- When you run `npm run backend:seed` the script inserts a few demo products, a default region (`us`) and a test admin user.
- **Migrations** – if you add a **custom module** (e.g., a new table for "Gift Cards"), you must:
  ```bash
  cd apps/backend
  npx medusa db:generate <module-name>
  npx medusa db:migrate
  ```
  This creates a SQL file in `src/modules/<your-module>/migrations/` and applies it.

### Future database work
- Add new tables by creating a **custom module** inside `src/modules/your-module/` (service + model + migration).
- Remember to generate a migration after editing the model, otherwise the DB schema stays unchanged.

---

## 5. Customizing the Backend
1. **Add a new API route** – create a file `apps/backend/src/api/store/my-endpoint/route.ts` and export a handler:
   ```ts
   export async function GET(req: Request) {
     // business logic – call a workflow or service
     return new Response(JSON.stringify({ hello: "world" }))
   }
   ```
2. **Create a workflow** – place it in `src/workflows/` and call it from your route.  Workflows let you compose several steps (e.g., validate, reserve inventory, send email).
3. **Add a custom module** – see the Medusa docs for the exact folder layout.

---

## 6. Customizing the Storefront
1. **Pages** live in `apps/storefront/src/app/`.  Edit a page component or add a new folder with `page.tsx`.
2. **Data fetching** – use the `sdk` helper (`apps/storefront/src/lib/config.ts`) which is already pre-configured with the env values.  Example:
   ```ts
   import { sdk } from "@lib/config"
   const { data } = await sdk.client.fetch<{ products: Product[] }>("/store/products")
   ```
3. **Styling** – TailwindCSS is set up.  Add classes directly in JSX or extend `tailwind.config.js`.
4. **UI components** – the Medusa UI library (`@medusajs/ui-preset`) gives ready-made buttons, cards, layout primitives.

---

## 7. Quick Start Checklist for a New Developer
1. **Start PostgreSQL** (e.g., ensure your local PG server is running).
2. **Install deps** – from repo root: `npm install`.
3. **Run DB migrations & seed**:
   ```bash
   cd apps/backend && npx medusa db:migrate && npm run backend:seed
   ```
4. **Create a publishable API key** in the Admin UI and paste it into `apps/storefront/.env.local`.
5. **Launch everything** – from repo root: `npm run dev`.
6. Open:
   - Backend admin: `http://localhost:9000/app`
   - Storefront: `http://localhost:8000`

---

## 8. Common Pain Points & How This Guide Solves Them
| Pain point | Why it happens | How the guide helps |
|------------|----------------|---------------------|
| **React version conflict** | Root `package.json` forces a single React version for both apps. | Advice to keep React out of the root and let each app declare its own version. |
| **Storefront can't fetch data** | No publishable key or mismatched default region. | Clear steps to create the key, set env vars, and use the correct region (`us`). |
| **Database not seeded** | Missing `db:migrate`/`seed` before starting the frontend. | Checklist includes migration and seed commands. |
| **Adding new API routes** | New developers think they must edit a router file. | Explanation of file-based routing and example `route.ts`. |
| **Custom modules not reflected in DB** | Forgetting to run `medusa db:generate`. | Reminder in the "Future database work" section. |

---

*Happy hacking! 🎉*

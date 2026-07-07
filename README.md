# Blemo — D2C storefront (MVP build)

Personalised acne-treatment lotion in 3 formulas × 3 sizes, built from the
Blemo PRD and System Design documents. Next.js 16 (App Router, TypeScript,
Tailwind v4), modular-monolith structure.

## Run it

```bash
npm install
npm run dev        # http://localhost:3000
```

`npm run build && npm start` for production mode.

## What's implemented

| PRD feature | Where |
|---|---|
| ₹/ml pricing ladder (₹3/ml base, −20%/step) | `src/lib/catalog.ts` (single source of truth; prices snapshot onto orders) |
| Depletion engine (2 ml/day, lead time) | `src/lib/catalog.ts` → drives refill dates + subscription cadence copy |
| Quiz → segment → formula + size | `/quiz` (deterministic rules, System Design §7); result themes the whole site |
| Segment-adaptive theming (Bolt/Bloom/Balance skins) | CSS variables + `data-theme`, `globals.css` |
| PDP: size selector, subscription toggle, WhatsApp refill opt-in | `/products/[formula]`, `src/components/buy-box.tsx` |
| Cart + checkout (UPI/card/wallet/COD) | `/cart`, `/checkout` |
| Order API with server-side price validation + inventory commit | `POST /api/orders` |
| Order confirmation with refill-reminder date | `/orders/[id]` |
| Reviews (verified, segment-filtered) + moderation queue | seeded in `src/lib/store.ts`, moderation in `/admin` |
| Education blog with formula CTAs | `/learn` |
| Admin OMS: order pipeline, 9-SKU inventory, low-stock flags | `/admin` |

## Deliberately stubbed (production TODOs)

- **Payments:** prepaid orders succeed instantly. Wire Razorpay: create order →
  hosted checkout → verify signed webhook → then mark `paid` (see `/api/orders` comments).
- **Persistence:** JSON file store (`src/lib/store.ts`) mirroring the Postgres
  schema in the System Design doc — swap for Prisma/Postgres; the money path
  already reserves/commits inventory in one locked step.
- **WhatsApp:** refill dates are computed and stored; sending needs a BSP
  (Gupshup/AiSensy) + approved template + opt-in records. A worker cron scans
  `refillReminderAt`.
- **Courier:** order statuses advance manually in `/admin`; wire Shiprocket
  create-shipment + tracking webhook at the `paid → packed → shipped` transitions.
- **Auth:** none on `/admin` or accounts. Add phone-OTP auth + RBAC before anything real.
- **Fonts:** system stacks (network-independent build). Upgrade via `next/font`
  (e.g., Archivo for display) when deploying.
- **Compliance:** claim language avoids "cure" per the PRD's regulatory flag;
  GST invoicing, DLT registration and DPDP consent flows are not implemented.

## Try the loop

1. `/quiz` → answer → site re-themes to your formula → add to cart
2. `/checkout` → any details (10-digit phone, 6-digit pincode) → place order
3. `/orders/{id}` → see the WhatsApp refill date computed by the depletion engine
4. `/admin` → advance the order through the pipeline, watch inventory

[README-495948aec3a54bb58d214a7478825cea.md](https://github.com/user-attachments/files/35118614/README-495948aec3a54bb58d214a7478825cea.md)
# Sunbeams · Retail & Studio

A working React + TypeScript ERP course project with a populated browser demo and a Supabase/Postgres backend implementation. Built using OpenAI Codex.

## Open the demo

Open the included `Sunbeams.html` (or `dist/index.html`) in a recent Chrome, Edge, Firefox or Safari browser. The production build embeds its scripts, fonts, styles and original SVG product illustrations in a single file. No API key is needed for browser demo mode. Allow browser storage. For predictable multi-tab behavior and camera permissions, serve the app on localhost or HTTPS.

To run the source:

```sh
npm ci
npm run dev
```

Open the localhost address Vite prints. The demo initially opens the Manager dashboard. Click the profile at the bottom of the sidebar to sign out, then choose a quick-login role. Demo switching is intentionally local and is **not authentication**. Business data is LZ-compressed in localStorage. Tabs on the same origin receive storage events; separate browser profiles do not share demo data. A reset button is under Settings and requires confirmation.

## Supabase setup

1. Create a new Supabase project. Apply `supabase/migrations/001_sunbeams.sql` in the SQL editor, or use the Supabase CLI migration workflow. Run the migration once on a new schema; do not replay it over existing policies.
2. Copy `.env.example` to `.env`. Set:
   - `VITE_SUPABASE_URL`: project URL.
   - `VITE_SUPABASE_ANON_KEY`: public anon key (a compatible public publishable key can be used).
   - `SUPABASE_URL`: the same project URL, for the seed script.
   - `SUPABASE_SERVICE_ROLE_KEY`: service role key, **for the seed script only**.
3. Run `npm run seed`. It creates four Auth users and role profiles, then imports the historical opening balances and activity. It refuses to replace an existing business workspace. `npm run seed -- --reset` deliberately replaces all business data; stop other users first.
4. Restart Vite, or run `npm run build` and serve `dist`. Vite embeds public `VITE_*` configuration at build time. Never place a service key in a `VITE_*` variable or frontend file.
5. Ensure the Supabase `supabase_realtime` publication includes `public.erp_notices`; the migration adds it when the standard publication exists. Realtime must be available for your project. Changes to that table trigger a fresh authorized snapshot in every connected session.
6. Sign in with a demo account below. For the two-browser demo, use separate browsers/profiles and sign in independently. Do not use the browser-only build to claim cross-browser Realtime.

| Role               | Email                  | Password      |
| ------------------ | ---------------------- | ------------- |
| Manager            | manager@sunbeams.demo  | Sunshine2026! |
| Cashier            | cashier@sunbeams.demo  | Sunshine2026! |
| Warehouse / Studio | studio@sunbeams.demo   | Sunshine2026! |
| Customer           | customer@sunbeams.demo | Sunshine2026! |

These are fictional classroom accounts. The seed sets `app_metadata.role` for frontend navigation and a protected `profiles.role` for authoritative server checks. New self-signups receive no business permissions without a profile. The customer profile is associated with customer `c0`.

## What works

- POS: search, Code 128 camera scanning, USB keyboard scanner input, multi-line carts, increment-on-scan, per-line and order discounts, stored tax rate, cash/card tender recording, stock checks, receipts and printing.
- Inventory: three locations, on-hand, FIFO reserved quantities, available, backordered, on-order, WAC, transaction history, used-in BOMs, open POs, transfers, adjustments with reasons, camera/USB scan forms and label sheets.
- Sales orders: pickup/customer/location/date, partial reservations and backorder estimates, fulfillment with revenue/COGS posting, cancellation and allocation release, closing, operator and timestamp history.
- Returns/exchanges: original receipt lookup, original discounted revenue/tax/cost reversal with cumulative cent rounding, maximum return quantities, Quality Hold, simultaneous replacement sale and net amount due.
- Purchasing: supplier create/read/update/delete (deletion is blocked when referenced), printable sent POs, expected receipt dates, scan receiving and partials, WAC, AP invoices/due dates, early/late receipt measurements, supplier payments and scorecards.
- Ordering: demand and demand variance from actual usage, safety stock, ROP, EOQ, inventory position, editable policies, periodic review, POs grouped by supplier.
- Production: 11 parent BOMs, planned/released/in-progress/completed states, exact component consumption, WIP accounting, scrap absorption, good/scrap/rework validation, reason codes, actual cycle timestamps and labor-hour reporting.
- MRP: four rolling weeks, recursive multi-level BOM explosion, shared-component aggregation, inventory netting once, due-week receipts, planned releases offset by lead time, expedite flags, EOQ/MOQ rounding, PO/build conversion review forms and explanations.
- Reports: GL-based P&L with custom dates, balance-sheet balances, trial balance, inventory valuation, daily-average inventory turnover/DSI overall and by category/item, sales by category/location, daily/weekly/monthly trend, best sellers/slow movers, returns and CSV exports.
- Customer storefront: original SVG product illustrations, location availability, pickup checkout, personal order history. Registry desired/gifted/needed quantities connect to pickup orders; registry checkout reserves stock and cancellation restores needs.
- Match Keeper: compares Mom/Baby stock coverage at each location against local demand, with computed scores and production recommendations.
- Seasonal planner: 12 weeks of prior-year seasonality and recent trend, editable promo uplift, CSV, and explicit push into MRP. Confirmed orders consume the forecast using the greater item/week demand, so forecast and orders are not added twice.
- Documentation: in-app architecture diagram, formulas, libraries/tools and accounting policy notes.

## Transaction architecture

`src/domain.ts` is the immutable demo engine and the shared reporting/formula layer. Each demo command clones the workspace, validates every step, and commits one compressed storage value only after success. A failed action leaves the previous state intact. Demo storage is not a database and simultaneous writes from different tabs can race; use Supabase for a multi-user demonstration.

In Supabase mode the client submits commands, never replacement business state. `erp_command(command jsonb)`:

1. Reads the server-controlled profile and enforces role permissions.
2. For a customer, overwrites the customer ID with the authenticated profile's customer ID; order lines are reduced to item and quantity so callers cannot inject prices or discounts.
3. Locks the workspace row with `SELECT ... FOR UPDATE`.
4. Applies the complete business command inside Postgres through `erp_apply` and internal posting helpers.
5. Updates inventory/accounting projection tables and checks balanced journals, nonnegative stock, and inventory subledger reconciliation.
6. Updates the aggregate and Realtime version notification in the same SQL transaction. An exception rolls everything back, including an exchange return whose replacement cannot be sold.

Persistence intentionally uses a JSONB workspace transaction aggregate in `erp_state`, plus real relational `inventory_transactions`, `journal_entries`, `journal_lines`, `settings`, `profiles`, and `erp_notices` tables. Read-only invoker-security views expose items, suppliers, customers, locations, BOM components and business documents. Business documents and their lines are JSON collections rather than independent relational tables. This keeps one unambiguous transaction boundary for a small course demo; it differs from a fully normalized enterprise schema.

Every mutation currently serializes on one workspace row and rebuilds the accounting/inventory projections. This prioritizes correctness over throughput. Before a high-volume production deployment, normalize the document collections, switch projections to incremental append/update, scope locks to affected inventory rows, add request idempotency, and perform an independent security/load review. The demo engine and SQL engine are separate implementations with parity tests; keep both tests updated when changing transaction rules.

RLS denies customer access to the underlying workspace and financial/inventory tables. `erp_snapshot()` gives a customer only their own orders/account, the sellable catalog/availability and guest-visible registries; costs, journals, other customers' contact details and staff activity are removed. An authenticated version-only notification supports customer updates without broadcasting private state. Unauthenticated users cannot execute business commands. Internal posting helpers are revoked from API roles.

## Accounting and planning policies

- Currency is USD. Quantity is a positive whole number except signed adjustments and zero-allowed completion outcomes. Monetary journals round to cents; WAC retains precision. A small rounding difference between a WAC multiplication display and transaction carrying value can occur; both values are labeled in Inventory Valuation.
- WAC is global per item, across locations and quality-held units. Receipts/completions/returns update WAC; sales and component issues use current WAC. Quality returns use the original sale cost and enter non-sellable hold.
- Production issue: debit Work in Progress, credit Inventory. Completion: debit Inventory, credit WIP. Scrap cost is absorbed by good units. An all-scrap completion posts the entire amount to Shrinkage. Rework is a subset of good units and never creates an extra receipt.
- The course brief defines FPY as **good ÷ started**. That is the displayed FPY; an additional first-touch yield excludes reworked units to distinguish it from final good yield. Scrap and rework rates use started units as the denominator.
- Gross sales, discounts, returns, COGS, shrinkage, net sales and operating income come from journal lines. Period turnover uses average **daily closing** inventory carrying value, not today's inventory as a historical proxy. Per-item/category COGS is allocated using the linked sale/return inventory cost movements.
- Reorder usage includes sale relief and production component issues over the last 90 calendar days, including zero-demand days. Annual demand is average daily usage × the editable working-days-per-year setting. The editable item minimum ROP is a floor under the statistical ROP.
- Sales orders lock their quoted price and tax rate at placement. Fulfillment uses those amounts and the then-current inventory WAC. New tax settings affect new orders and POS checkouts.
- Allocations are FIFO by order creation. Only physical sellable stock is reserved. Unreserved demand is backordered. Fulfillment can use its own reservation but cannot consume another order's reservation.
- MRP nets physical inventory once against confirmed demand. It recursively plans kits and subassemblies in dependency order, sharing component stock across parents. Planned supply carries forward into future weeks. Existing production WIP is not currently netted as scheduled receipts; inspect the studio schedule before converting a suggestion. Draft review forms do not affect supply until submitted.
- All business-day bucketing and supplier due/expected dates use America/Denver. Actual timestamps remain ISO timestamps. Saturday/store Sunday and annual event demand patterns are built into the history generator.

## Historical data

The deterministic seed ends on the current Denver date and includes 365 days of daily sales, Sunday store closure, Saturday peaks, Mother's Day and holiday ramps, winter sweatshirts/summer hats, gradual growth, skewed best sellers, roughly 4% return transactions, six different supplier reliability profiles, supplier receipts/payments, production scrap/rework, transfers, and current orders/POs/builds. Restocking uses the same purchase, production and transfer logic. Opening inventory and cash have an explicit balanced owner-capital journal. Final cycle counts produce intentionally thin pairs and component shortages for classroom exercises.

## Verification

```sh
npm test
npm run build
```

Vitest checks known-input financial/inventory calculations, historical reconciliation, rollback, reservation priority, MRP and settings. PGlite runs the actual Postgres migration/functions and tests role permissions, customer row isolation and server-side pricing protection. React interaction tests traverse every screen, complete a real demo checkout, check persisted storage, exercise scanner input and switch customer roles.

PGlite tests use a small mocked `auth` schema; they do not replace an integration test against a hosted Supabase Auth and Realtime installation. The interaction suite uses a DOM environment and mocked chart/barcode rendering, not a graphical browser. This environment did not expose the supported browser QA tool, so camera hardware, physical print alignment, graphical responsive layout, and two-browser Supabase Realtime were not exercised here. Run the final manual checklist on the classroom machine.

## Deliberate limits

- No Supabase credentials were supplied, so no live database was provisioned or live cross-browser test performed.
- Hosted publication was blocked by the Sites service response: `Sites backfill is in progress in this destination`. The standalone build and source are deliverables; no deployed URL is claimed.
- No lot/size variants, component supplier lot traceability, quality-hold disposition workflow, card charging/refund processor, email sending, PDF-generation service or natural-language AI assistant. Native print dialogs provide PO/receipt/label PDFs.
- Supplier quality scoring traces direct resale returns by preferred supplier; manufacturing scrap has reason reporting but cannot be attributed to a particular supplier shipment without lots. Lead-time and fill-rate score history are computed from the stored POs/receipts.
- Registries are visible to authenticated shoppers by registry name and desired items; private customer contact information remains protected. Registry needs can be exceeded by concurrent guest gifts, although inventory reservations remain stock-safe in Supabase.
- The seasonal forecast allocates future total demand by recent product mix. It is a transparent classroom forecasting model, not a statistical forecasting service.
- Cancelling production is allowed before issue. Issued production must be completed (including all-scrap when appropriate); there is no automatic issued-material unbuild.

See `DEMO_CHECKLIST.md` for a complete click-by-click demonstration.

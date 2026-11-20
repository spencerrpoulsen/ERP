# Sunbeams live demo checklist

Use a recent desktop browser. For camera scanning, use HTTPS or localhost and allow camera access. Camera and physical label alignment need a final check with your hardware. The browser demo works without Supabase; the two-browser test requires the Supabase setup in README.

## Before the professor arrives

1. Open `Sunbeams.html`, or run `npm ci` then `npm run dev` and open Vite's localhost address.
2. Under **Settings**, reset the demo if you want the original working scenarios. Confirm the reset. For Supabase, use the seed script only before the demonstration, not while others are working.
3. Open **Overview**. Confirm year-long history, nonzero sales, low-stock items and open orders. Use **Last 12 months** to expose annual seasonality.
4. Click the profile in the bottom-left and demonstrate **Manager**, **Cashier**, **Studio**, and **Customer** quick logins. In local mode they switch the demo role; in Supabase mode they authenticate the four seeded users.
5. Return to **Manager**. Keep a calculator ready. Do not reset the demo during a sequence you want to reconcile.

## 1. Dashboard and reports

1. Open **Overview**; note today's sales, month-to-date sales, gross margin and inventory value.
2. Change the date range from 30 to 365 days. Change the sales chart from Daily to Weekly to Monthly.
3. Change the location filter and show the sales curve and best sellers change. Financial dashboard KPIs remain business-wide; the sales location filter scopes the chart and item ranking.
4. Click **Items running low**, **Orders to fulfill**, and **Purchase orders overdue**; each opens a real module.
5. Open **Reports → Profit & loss**. Set explicit From/To dates and note Gross Sales, Discounts, Net Sales, COGS and Gross Profit.
6. Open **General ledger**. Confirm **Balanced**, inspect account balances and recent journal entries, then export the actual journal lines to CSV.
7. Check **Inventory valuation**, **Turnover & DSI**, and **Sales insights**. Explain that average inventory uses daily closing historical values. Export P&L, valuation, turnover and item-sales CSVs.

## 2. Inventory and ordering math

1. Open **Inventory**. Choose Provo; compare On Hand, Allocated, Available, Backorder and On Order.
2. Search `SB-000023` or `Mama Sun`. Click a row to show WAC, location stock, ordering inputs, BOM, used-in parents, open POs and movement history.
3. Find a component with available stock. Click **Transfer**, choose a source with stock, another destination, and quantity 2. Submit. The total quantity/value stays constant; two linked transfer movements appear.
4. Click **Adjust**; choose a stocked item, enter `-1`, reason `DAMAGED`, and submit. Inventory falls and Shrinkage expense rises. Try removing more than available stock: the action must fail without changing anything.
5. Open **Material planning → Periodic ordering review**. Find an item with demand. Note its daily usage, standard deviation, safety stock, ROP, position and EOQ.
6. Open **Settings**. Multiply ordering cost by four; save. Return to review: EOQ should double if demand and holding cost are unchanged. Double z; safety stock doubles. Increase holding rate; EOQ falls.
7. Click **Create grouped POs** only if you want to reserve all current buying suggestions. The screen groups purchased-item suggestions by preferred supplier.

## 3. Barcode generation and scanning

1. Open **Barcode labels**. Set label quantities for a few sellable items and one component.
2. Print the sheet or save it as PDF. Use 100% scale and minimum margins; check against the actual label stock before committing a full sheet.
3. At **Point of sale**, focus the barcode field. Type a selected barcode then Enter, simulating a USB scanner. Repeat it: quantity increments.
4. Click the camera button to scan a printed label. If permission/device access fails, show the explicit camera error and use the wedge field.
5. Repeat scan entry in an inventory transfer/adjustment and a PO receive dialog. A PO receiver rejects a barcode not on that PO.

## 4. POS with known arithmetic

1. Choose **Provo Flagship Store** and any item with at least two available units. The stock on some matching pieces is intentionally thin.
2. Add quantity 2. Set the line discount to 10%, the order discount to `$1`, and choose cash/card. The preview updates before posting.
3. For an item priced $32 with a 7.25% tax setting: gross $64, line discount $6.40, order discount $1, taxable net $56.60, tax $4.10, total **$60.70**. If the price or settings changed, calculate from the current displayed inputs instead.
4. Complete the sale. Note the receipt number. Print the branded receipt.
5. Open Inventory: that location lost exactly 2 units. In P&L, gross sales rose $64, discounts rose $7.40 and COGS rose 2 × the pre-sale WAC. Tax is a liability, not revenue.
6. Try a cart quantity greater than available. Checkout must be rejected and preserve stock/GL.

## 5. Returns and atomic exchanges

1. Open **Returns & exchanges**. Search the receipt from the prior step, or choose it from the recent list.
2. Return one unit with `WRONG_SIZE`. The preview uses the original discounts and tax, not today's sticker price. Submit; sellable stock rises by 1 at the original cost.
3. Return the other unit using `QUALITY_DEFECT_SEAM`. Sellable stock does not rise; Quality Hold rises by 1. Revenue/tax/COGS reverse appropriately and refund cash is recorded.
4. Attempt another return from the same line; it must reject a quantity beyond the purchased amount.
5. On another receipt, add an optional replacement item. Observe the customer-pays/customer-receives difference.
6. Choose an unavailable replacement and try the exchange. It must reject the entire exchange; neither the return nor the replacement should post.

## 6. Sales orders, backorders and the customer portal

1. Open **Sales orders → New order**. Choose a customer, location, request date and a stocked item; submit.
2. In Inventory, allocated rises and available falls. On-hand and revenue do not change.
3. Place an order beyond current physical stock. Only existing units are reserved; the rest is backordered. Open the order number to see its estimated availability and timestamp/operator history.
4. Cancel an open order and confirm. Its reservations release without a stock movement or revenue entry.
5. Fulfill a fully available order. Inventory, revenue and COGS post together. Click **Close order** after fulfillment.
6. Sign in as **Customer**. Browse Mom, Baby and Matching Sets, choose a pickup location, add items and place a pickup order.
7. Open **My orders**, then return to staff **Sales orders**. The same order exists. Fulfill it on the staff side; the customer's status becomes **Ready for pickup**.

## 7. Supplier, PO, partial receipt, WAC and AP

1. Open **Suppliers**. Compare Wasatch's delivery history with the chronically late/partial suppliers. Click a supplier for actual versus quoted lead, score weights, quality, invoices and overdue AP.
2. Add a fictional supplier; edit its contact details. Delete that unreferenced supplier and confirm. Attempting to delete a historically referenced supplier must be blocked.
3. Open **Purchasing → Create purchase order**. Choose a component and record its current quantity Q and WAC C from Inventory.
4. Order 20 units at a deliberately different positive cost P. Create/mark sent. Open the PO number and print the PO. Expected receipt is sent business date + quoted lead time.
5. Click **Receive**, choose a location, enter an actual date and receive only 10 units. Show Partially Received, the receipt's early/late days, and AP due date from terms.
6. Verify new WAC = `(Q × C + 10 × P) ÷ (Q + 10)`. Inventory and Accounts Payable increase by `10 × P`.
7. Scan or receive the remaining 10. Over-receipt must be blocked. The PO becomes Received.
8. Click **Pay**, confirm the amount, and observe AP clears while Cash falls. A fully received/paid PO becomes Closed.

## 8. Production, kit BOMs and quality metrics

1. Open **Print Studio**. Complete the already In Progress order, or plan a new `Mama Sun Printed Tee` order for 10 units.
2. Before issuing, inspect its BOM and make sure the studio has 10 blanks, 10 transfers and 10 tags. Transfer or purchase missing material through the real modules.
3. Click **Issue BOM**. Exactly those quantities leave sellable components; their WAC cost moves to Work in Progress. An unavailable component blocks the entire issue.
4. Click **Start work**. Complete with 9 good, 1 scrap, and (optionally) 1 reworked good unit, with `MISALIGNED_PRINT` and labor hours 1.
5. Good + scrap must equal 10. Rework must not exceed good. Try invalid numbers first to demonstrate validation.
6. Completion receives 9 finished units. Unit receipt cost = total material issue cost ÷ 9. Scrap is absorbed into good units; rework does not add a tenth unit.
7. Check FPY (9 ÷ 10 = 90% per the course brief), scrap (10%), rework rate, first-touch yield, cycle time and units/labor hour. The reason appears in the descending scrap chart.
8. For a Matching Set order of 10, first make/transfer enough finished pieces: 10 Mom pieces + 10 Baby pieces + 10 boxes + 20 tissue sheets + 10 seals. Issue and complete 9 good/1 scrap; inspect exact component movements and cost rollup.

## 9. MRP and forecasting

1. Open **Material planning**. The seed includes genuine shortages and demand for kits, requiring subassembly and purchased-component planning.
2. Walk across the four weeks for one item: Gross Requirements, Scheduled Receipts, Projected On-Hand, Net Requirements and Planned Order Release.
3. Explain: existing parent stock is netted before BOM explosion; shared component stock is consumed once. Release date = need week − lead time. Past release dates show **Expedite**.
4. Toggle EOQ/MOQ rounding. Net requirement stays the same, planned supply can increase, and excess carries into future weeks.
5. Click **Create PO** on a purchased component; review/submit the draft. Its scheduled receipt appears in the expected week.
6. Click **Plan build** for a manufactured/kit parent. Inspect studio WIP before converting: existing production orders are not netted as scheduled receipts in this version.
7. Open **Seasonal planner**. Raise promo uplift from 15% to 35%; future units and the 12-week chart change.
8. Click **Push forecast into MRP**. The next four weeks use the greater of forecast and confirmed demand per item/week, preventing double counting. Revisit the suggested quantities and releases.

## 10. Match Keeper and registry

1. Open **Match Keeper**. Compare Sunshine/Bloom/Cozy pairs across locations.
2. Show demand-adjusted Mom/Baby coverage and computed match health. If there is no local demand history, the screen compares units and says so.
3. Plan a build for the short counterpart or use Inventory Transfer to rebalance a pair. Reopen Match Keeper and watch the score change.
4. In the **Customer storefront**, add desired gifts to the bag, then open **Baby shower registry** and name/create a registry from that bag.
5. Click **Shop remaining gifts** on a registry, choose quantities and place the pickup order. Desired/gifted/still-needed values update and physical stock is reserved through the order.
6. Cancel the pickup order as staff; the registry's need returns. Guest-visible registry names/items do not expose customer contact details in Supabase mode.

## 11. Multi-user live update

**Supabase only:**

1. Configure and seed the hosted database as in README. Open the same app origin in Browser A and Browser B.
2. Sign in as Manager in A and Cashier in B. Open Inventory/Reports in A.
3. Complete a sale in B. A should update its stock, P&L and activity without a page refresh within a few seconds.
4. Sign in as Customer in another profile. Place an order, then fulfill it as staff; verify both views update.
5. Confirm the customer cannot read `erp_state`, staff inventory transactions, financial journal tables, or another customer's orders through API requests. The automated SQL suite also checks this boundary.

For browser demo mode, two tabs on the same origin can demonstrate storage-event refresh, but it is not the cross-browser Supabase test and simultaneous writes are not guaranteed to serialize.

## High-value “change one input” moments

| Input                                         | Watch                                                    |
| --------------------------------------------- | -------------------------------------------------------- |
| POS quantity, line discount or order discount | Taxable net, tax, total, receipt and P&L                 |
| Tax setting                                   | Checkout and new pickup order totals                     |
| Receipt unit cost and partial quantity        | WAC, stock valuation and AP                              |
| Ordering cost × 4                             | EOQ × 2                                                  |
| Holding percentage × 4                        | EOQ ÷ 2                                                  |
| Safety-stock z-value × 2                      | Safety stock × 2 and higher ROP                          |
| Good/scrap split, fixed material issue        | Finished unit cost and quality metrics                   |
| Supplier rating weights                       | Overall scores and KEEP/WATCH/REPLACE                    |
| Requested week or BOM parent order quantity   | Time-phased requirements and release dates               |
| EOQ/MOQ toggle                                | Planned order quantities and future projected inventory  |
| Promo uplift, then push forecast              | Forecast chart, MRP demand and suggestions               |
| Mom/Baby transfer                             | Local coverage and match health                          |
| Return reason: size vs quality                | Sellable stock vs Quality Hold                           |
| Cancel an open order                          | Reservations, availability, backorders and registry need |

Finish by opening **About this system**: show the tools/libraries, actual persistence architecture, formulas and explicit policy/feature limits.

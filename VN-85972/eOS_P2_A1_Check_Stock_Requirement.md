# eOS Phase 2 – A1: Order-Creation Stock Check ("Check Stock")

Source: eOS-P2-BRD-A1 v0.2 (SR SG-252825 / FR SG-252841). Scope of this file: eOS + its MuleSoft/SAP interface only. HAVI is out of scope.

## 1. Summary

Add a mandatory **Check Stock** step in the eOS **cart**, before checkout.

- User clicks **Check Stock** → eOS sends **one batch ATP request** for all cart items to SAP via MuleSoft.
- SAP returns the fulfillable quantity per item. MuleSoft converts it to the user's UOM.
- eOS shows a result per cart line. Checkout is enabled **only if every item is confirmed**.
- The check is **binding**: insufficient stock, zero stock, or ATP service unavailable all **block checkout**. There is no override or fallback.
- Check Stock is the **only** trigger for ATP. There are no automatic stock badges or ATP calls on the product listing, product detail, or on cart load.
- The ATP call is **read-only**: no reservation, allocation or soft-lock in SAP.
- After checkout, SC creation in SAP is unchanged from Phase 1. Trade Support's ATP check at SO release also stays as is (secondary control).

**Why:** in Phase 1, orders are accepted with no stock visibility. ATP only runs at SO release (manual, by Trade Support), so orders get cut or cancelled late and cause rework.

## 2. Impact

| Platform | Impacted | Change |
|---|---|---|
| Customer App (iOS/Android) | Yes | Cart: Check Stock button, per-line result, gated Checkout button |
| Salesman App (iOS/Android) | Yes | Same as Customer App |
| Web Ordering Portal | TBC | Same flow if in Phase 2 scope (to be confirmed with business) |
| Web Admin Portal (Trade Support) | No | No UI or workflow change |
| MuleSoft | Yes | New/enhanced **batch** endpoint eOS → SAP; handles timeout, errors, UOM conversion |
| SAP | Yes | Batch ATP API/RFC callable on demand |

Roles: Credit Customer, Cash Customer, Salesman all follow the same flow. Cash customers: Check Stock applies before payment.

## 3. To-Be Flow (cart)

1. User adds items to the cart (no stock info shown, no ATP calls). The cart shows a **Check Stock** button. **Checkout/Proceed is disabled by default.**
2. User clicks **Check Stock**. eOS sends a single batch request with all line items to MuleSoft, which relays it to SAP.
3. SAP computes the fulfillable quantity per item (base UOM). MuleSoft converts it to the requested UOM and returns the results.
4. eOS shows the result per line (see section 4) and evaluates the gate.
   - All items CONFIRMED → Checkout enabled, validity timer starts, green banner.
   - Otherwise → Checkout stays disabled, banner shows how many items need attention.
5. User reduces quantities or removes items. **Any cart change** (quantity, add, remove) invalidates the result, clears the per-line results and re-disables Checkout. User re-runs Check Stock. Repeat until all items are CONFIRMED.
6. If the call fails or times out → error state, Checkout disabled, user retries.
7. User clicks Checkout within the validity window → normal Phase 1 submission (SC created in SAP).

## 4. Result Logic (per line)

| Condition | Status | UI | User action | Checkout |
|---|---|---|---|---|
| fulfillable ≥ requested | `CONFIRMED` | Green "Stock Confirmed" | None | OK for this line |
| 0 < fulfillable < requested | `INSUFFICIENT` | Amber "Insufficient – [X] [UOM] available" (show the fulfillable qty) | Reduce qty to ≤ fulfillable | Blocked |
| fulfillable = 0 | `OUT_OF_STOCK` | Red "Out of Stock" | Remove item | Blocked |
| Service error / timeout | `CHECK_FAILED` | Red/grey error | Retry Check Stock | Blocked |

Checkout is enabled only when **all** lines are CONFIRMED **and** the result is still within the validity window **and** the cart is unchanged since the check.

Example: unrestricted 100 CTN, committed 80 CTN (Released + Unreleased SOs) → fulfillable 20 CTN. A request for 30 CTN shows INSUFFICIENT, "20 CTN available". The user reduces to 20, re-runs Check Stock, and it becomes CONFIRMED.

## 5. Requirements

### Functional
- **F1. Check Stock button.** Mandatory in the cart of Customer App and Salesman App. It is the sole ATP trigger. Label and placement are decided in UX design.
- **F2. Batch call.** One request per click, covering all cart items. No per-item calls and no calls on page load.
- **F3. Per-line result display** per section 4. Show the fulfillable qty when INSUFFICIENT.
- **F4. Checkout gating.** Disabled if Check Stock has not run on the current cart state, or any line is not CONFIRMED, or the result has expired.
- **F5. Cart change invalidates the result.** eOS must track the cart state at the last successful check. Any difference invalidates the result.
- **F6. Validity window.** A successful result is valid for a limited time (suggested **30 min**, TBC). The validity period is shown to the user. After expiry, Checkout is re-disabled and Check Stock must be re-run.
- **F7. ATP service unavailable → blocked.** No fallback that allows ordering without a successful check.
- **F8. No stock reservation** (read-only query).
- **F9. UOM.** Returned quantity must be converted to the user-selected UOM using the conversion factors from the existing product master sync (done by MuleSoft/GDC).

### Calculation (SAP side, which eOS relies on)
- Fulfillable qty = Unrestricted stock − Released SO committed qty − **Unreleased SO committed qty** − Blocked stock − Expired stock.
- Phase 1 ATP only considered Released SOs. A1 also deducts Unreleased SOs (more conservative).
- Blocked stock (quality-blocked, restricted, physically blocked) is excluded. Expired stock (batch expiry < today) is excluded.
- Near-expiry stock: **pending business confirmation**. Until confirmed it is treated as fulfillable.
- SAP team must confirm the exact stock-type mapping and which SO statuses count as "Unreleased".

### Validation
| Case | Behaviour |
|---|---|
| Cart item has no valid/active SAP material code | Treat as OUT_OF_STOCK for the gate; notify IT |
| Batch response not received within timeout (suggested 10 s) | All items CHECK_FAILED; Checkout blocked; no partial results shown |
| Item returns negative or null fulfillable qty | Treat that item as OUT_OF_STOCK; log the anomaly; do not block the whole response |
| UOM conversion factor missing for an item | Item "unable to validate"; distinct error indicator; Checkout blocked; log for IT; do not show an incorrect qty |

## 6. Messages (wording to be finalised in UX design)

| Trigger | Placement | Text |
|---|---|---|
| Any item INSUFFICIENT | Per line (amber) + cart banner | "Insufficient stock – [X] [UOM] available. Please adjust your quantity." / "[X] item(s) have insufficient stock. Please adjust quantities before proceeding." |
| Any item OUT_OF_STOCK | Per line (red) + cart banner | "Out of Stock – this item cannot be ordered at this time." / "[X] item(s) are out of stock and must be removed before you can proceed." |
| All CONFIRMED | Green banner | "Stock confirmed for all items. You may proceed to checkout. (Valid for [XX] minutes.)" |
| ATP service unavailable | Error banner + button error state | "Stock check is currently unavailable. Please try again. If the problem persists, contact Trade Support. You cannot proceed with your order until stock is confirmed." |
| Timeout | Error banner | "Stock check timed out. Please try again. You cannot proceed with your order until stock is confirmed." |
| Item cannot be validated (UOM/material issue) | Per line | "Unable to validate stock for this item. Please contact Trade Support." |
| User tries to click Checkout without a check | Tooltip near the disabled button | "Please check stock availability before proceeding. Click the 'Check Stock' button above." |
| Cart modified after a successful check | Warning banner | "Your cart has been updated. Please re-run 'Check Stock' before proceeding to checkout." |
| Validity window about to expire (e.g. 5 min left) | Warning banner | "Your stock check result will expire in [X] minutes. Please proceed to checkout or re-run Check Stock." |
| Validity window expired | Banner | "Your stock check has expired. Please re-run Check Stock." |

## 7. API Contract (eOS → MuleSoft → SAP), batch

**Request: per line item**

| Field | SAP field | Mandatory | Note |
|---|---|---|---|
| Material SAP code | MATNR | Yes | Already synced to eOS in Phase 1 |
| Plant / Storage location | WERKS / LGORT | Yes | Configured in MuleSoft, not user-selectable; WDPL SG plant code to be provided by SAP team |
| Requested quantity | – | Yes | Qty in the cart |
| Requested UOM | MEINS | Yes | Must match the UOM selected in eOS |
| Check date | – | Yes | Today vs requested delivery date: **TBC** (see section 9) |

**Response: per line item**

| Field | Type | Note |
|---|---|---|
| `material_code` | String | Echoed back to match the cart line |
| `fulfillable_qty` | Decimal | In base UOM |
| `fulfillable_qty_in_uom` | Decimal | Converted to the requested UOM by MuleSoft |
| `uom` | String | Must match the requested UOM |
| `status` | Enum | `CONFIRMED` / `INSUFFICIENT` / `OUT_OF_STOCK`, from comparing `fulfillable_qty_in_uom` vs `requested_qty` |
| `requested_qty` | Decimal | Echoed back (in the requested UOM) for UI display |

SAP technical method (RFC / BAPI / REST) is to be confirmed by the SAP team.

**Performance (suggested, to be agreed):** batch P95 ≤ 5 s for up to 20 lines; timeout 10 s; single-item call ≤ 2 s.

## 8. Logging / Monitoring

- Log every Check Stock call: timestamp, user/session, materials, requested qty, response status, response time, returned fulfillable qty per item (MuleSoft/eOS application log).
- Log all errors/timeouts with error code, affected materials and session reference.
- Log blocked checkout attempts (user, timestamp, blocked items, reason).
- IT monitoring: API availability rate, average batch response time, error frequency, number of calls per day (add to existing eOS monitoring).

## 9. Open Items (not yet decided; do not assume)

| # | Question | Impact |
|---|---|---|
| 1 | **Check date**: today or the requested delivery date? | Decides whether Unreleased SOs on future dates are deducted; affects accuracy |
| 2 | **Validity window** length (suggested 30 min) | Needed before front-end development |
| 3 | **Display rule**: show customers the exact fulfillable qty, or only a label? Is the Salesman shown the exact qty? (commercial concern about revealing inventory levels) | Needed before UI design |
| 4 | Which SAP SO statuses count as "Unreleased"? | SAP spec; blocking |
| 5 | Near-expiry stock: exclude or not? | Must be resolved before development |
| 6 | Add an **auto-adjust** button ("Reduce to [X] CTN") for INSUFFICIENT lines? | UX; extra dev effort |
| 7 | Is there an exception path (e.g. Trade Support override) if ATP is down for a long time (> 1 h)? Currently: no | Business decision |
| 8 | Maximum number of lines in one cart | Sizing of batch payload and SLA |
| 9 | Is the Web Ordering Portal in scope? | Platform scope |
| 10 | Does SAP support a batch API and deducting Unreleased SOs? | Blocking, before any development |

## 10. Dependencies and Assumptions

- SAP exposes a batch ATP API/RFC that deducts both Released and Unreleased SOs. Must be ready before GDC integration testing.
- MuleSoft builds the batch endpoint. The spec (schema, auth, timeout, rate limit) must be agreed before GDC work begins.
- Material code (MATNR) and UOM conversion factors are already in eOS via the Phase 1 master sync. Completeness of the conversion factors must be verified.
- SAP master data (material, plant, storage location, stock categories) is accurate. The Phase 1 MuleSoft infrastructure is stable.
- Out of scope: PO ETA-based delivery date recommendation (FTS US-05), changes to SAP SO release logic, changes to SAP credit limit check, HAVI batch/allocation constraints.

## 11. Acceptance Criteria

1. Cart shows the Check Stock button (Customer App and Salesman App). Checkout is disabled by default and not actionable until a successful check.
2. Clicking Check Stock sends a single batch request for all items and shows the correct per-line status and fulfillable qty.
3. If any item is INSUFFICIENT or OUT_OF_STOCK, Checkout stays disabled until the qty is reduced to ≤ fulfillable (or the item is removed) and Check Stock is re-run.
4. When the ATP service is unavailable (simulated), an error is shown and Checkout stays disabled.
5. The fulfillable qty reflects the deduction of Released and Unreleased SOs, and excludes blocked and expired stock (verify against SAP data).
6. A cart change after a successful check invalidates the result, re-disables Checkout and prompts a re-run.
7. When the validity window expires, Checkout is re-disabled and Check Stock must be re-run.
8. UOM conversion is correct (e.g. SAP returns PCS, 12 PCS = 1 CTN, shown correctly in CTN).
9. A cart with mixed results (CONFIRMED / INSUFFICIENT / OUT_OF_STOCK) shows each status correctly and keeps Checkout blocked until all are resolved.

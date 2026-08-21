# P2P-Upstream-Downstream-Explained
 P2P / AP Concepts


## 1. P2P Cycle — Upstream vs Downstream

**Upstream (Procurement side, before the invoice exists):**
1. Purchase Requisition (PR) raised by requester
2. PR approved → converted to Purchase Order (PO)
3. PO sent to vendor
4. Goods/Services delivered → **Goods Receipt (GR)** posted in SAP, generating a **GRN (Goods Receipt Note)** — *not GLN; GLN is a Global Location Number used in EDI/logistics, don't mix them up in the interview*
5. Vendor master data setup/maintenance (bank details, tax IDs, payment terms) — this is also "upstream" onboarding

**Downstream (Finance/AP side, after the invoice lands):**
1. Invoice received (paper, PDF, EDI, or portal)
2. Invoice captured/scanned (OCR tools like Tungsten/KOFAX) and logged in DMS
3. Invoice posted in SAP (FB60/MIRO) with 3-way match validation
4. Exceptions → parked/blocked, routed for resolution (price/qty variance, missing GR, duplicate)
5. Approved invoice → payment proposal → payment run → remittance to vendor
6. Vendor query handling (payment status, remittance advice) — this is your helpdesk layer

**How to phrase it:** *"I sit on the downstream side — invoice processing and vendor helpdesk for the Germany region — but I regularly interact with upstream data because a blocked invoice is often a PO or GR mismatch, so I trace it back to the requisition or goods receipt to resolve it."*

---

## 2. Invoice Processing Cycle (End-to-End)

1. **Receipt** — invoice arrives via email/portal/EDI
2. **Indexing/Capture** — vendor, PO number, invoice number, amount, tax captured (OCR or manual)
3. **Validation** — duplicate check, vendor master check, PO existence check
4. **3-Way Match** (see below) — automatic in SAP if PO-based
5. **Posting** — MIRO (PO-based) or FB60 (non-PO) transaction in SAP
6. **Parking/Blocking** — if mismatch, invoice is parked with a block reason (price, quantity, GR missing)
7. **Resolution** — AP works with buyer/vendor to clear the block
8. **Approval workflow** — for non-PO or above-threshold invoices
9. **Payment proposal** — invoices due for payment listed in a payment run
10. **Payment execution** — F110 (payment run) generates payment file, sent to bank
11. **Remittance** — vendor notified of payment (this is where a lot of your helpdesk queries land)
12. **Archival** — DMS filing for audit trail

**How to phrase it:** *"In my day-to-day, I handle both ends of this — I process invoices in SAP HANA and file them in DMS, and I also field the vendor's 'where's my payment' query, so I can walk through the full lifecycle from either direction."*

---

## 3. Three-Way Matching (3-Way Match)

Matches three documents before an invoice is cleared for payment:
| Document | Confirms |
|---|---|
| **Purchase Order (PO)** | What was ordered, agreed price, quantity |
| **Goods Receipt Note (GRN)** | What was actually received |
| **Invoice** | What the vendor is billing for |

- If all three align (qty, price, PO reference) → invoice auto-clears for payment
- Mismatch → invoice is **blocked/parked**:
  - **Price variance** — invoice price ≠ PO price
  - **Quantity variance** — invoice qty > GRN qty
  - **Missing GR** — invoice posted before goods receipt exists
- Some orgs also do **2-way match** (PO + Invoice only) for services or low-value POs where GR isn't mandatory

**How to phrase it:** *"A large part of my hold-report work is triaging exactly these three variance types — I've actually built a Power BI dashboard tracking invoice hold trends by block reason, so I can speak to which mismatch types are most frequent and how they get resolved."*

---

## 4. Payment Run / Payment Process in SAP

- **F110** is the standard T-code for automatic payment runs
- Steps: parameters set (company code, payment method, due date) → proposal run → proposal review/edit (block or unblock items) → payment run → payment medium (file for bank, e.g. SEPA for Germany) → posting
- **Payment methods:** wire transfer, SEPA (common for Germany/EU vendors), check, ACH
- **Payment terms** drive due dates (e.g., Net 30, 2/10 net 30 for early-payment discount)
- Failed/returned payments get reprocessed; AP helpdesk usually fields the "payment not received" query and checks F110 proposal logs or vendor line items (FBL1N) to confirm status

**How to phrase it:** *"On the Germany account, payment status is one of the most common queries I resolve — I check the vendor line item report or payment proposal log in SAP to confirm whether it's cleared, blocked, or pending the next payment run."*

---

## 5. Vendor/GR Terminology Cleanup (common confusion points)

- **GRN** = Goods Receipt Note/Note — proof goods were received, used in 3-way match
- **GLN** = Global Location Number — a GS1 standard identifier for physical locations, mainly relevant in EDI/e-invoicing routing, not core to AP matching. If TCS asks about GLN specifically, it's likely in the context of EDI/PEPPOL invoicing (which you have direct experience with from Belgium B2B PEPPOL onboarding) — mention that distinction confidently.
- **DMS** = Document Management System — where invoices/backup are filed post-processing

---

## 6. Vendor Onboarding (Upstream Master Data)

1. Vendor submits registration (tax ID, bank details, W-9/W-8 or EU VAT equivalent)
2. Compliance/KYC check
3. Vendor master record created in SAP (XK01/BP transaction)
4. Payment terms and tax codes assigned
5. Vendor activated → can now be referenced on POs

**How to phrase it:** *"I've supported EDI and e-invoicing onboarding — including PEPPOL B2B onboarding for Belgium — so I understand both the master-data setup side and the downstream impact if onboarding data is wrong (it directly causes invoice blocks later)."*

---

## Likely interview Question Angles + Quick Answers

- **"Walk me through the P2P cycle."** → Use the upstream/downstream structure above, anchor with your own role.
- **"What causes an invoice to go on hold?"** → Price/qty/GR variance, missing PO, duplicate invoice, vendor master mismatch.
- **"How do you handle a vendor asking about payment status?"** → Check SAP (FBL1N/vendor line items or F110 proposal), confirm block reason if unpaid, explain expected payment run date.
- **"Difference between PO-based and non-PO invoice processing?"** → PO-based goes through 3-way match; non-PO (FB60) typically needs manual approval workflow instead.
- **"What's a GRN vs GR?"** → GR is the SAP transaction/movement; GRN is the document/note generated as proof of that receipt.
- **"How does SAP flag duplicate invoices?"** → System checks vendor + invoice number + amount + date combination against existing postings.

---

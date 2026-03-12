## Functional & Technical Specification  

**Object**: Transport Settlement Report `ZSCE_SETTLEMENT` (Include `ZSCE_SETTLEMENT_RPT_CLS`)  
**Change**: Introduction of field `BILL_REGIME` for Road settlement  
**Author**: *TBD*  
**Date**: *TBD*  

---

## 1. Business Background  

The report `ZSCE_SETTLEMENT` is used for transport settlement across various modes (Road, Rail, Marine/Air, etc.).  
For **Action “Road”** (`P_RD = 'X'`), the program reads road‑related settlement data from table `ZTRSTLMNT`, displays it in an editable ALV grid, and allows users to maintain settlement‑relevant attributes (LR/GRN details, liability amounts, vendor, SAC code, etc.) before saving back to `ZTRSTLMNT`.  

Currently, the **billing regime** for road settlements is not explicitly maintained in this report, although the underlying table `ZTRSTLMNT` contains field `BILL_REGIME`.  

---

## 2. Business Requirement (FS)  

- Introduce a new attribute **Bill Regime** (`BILL_REGIME`) in the **Road** settlement screen.  
- Allow users to **view and maintain** this attribute for each road settlement line when **Action = Road**.  
- Changes in `BILL_REGIME` entered in the ALV must be **persisted in table `ZTRSTLMNT`** for the corresponding shipment and delivery.  
- The behavior for other modes (Rail, Marine/Air, GTA, Shortage, Invoice, Vehicle, etc.) remains unchanged.  

**Business Use**  
- `BILL_REGIME` indicates the applicable billing regime for the road freight (e.g. forward charge, reverse charge, exempt, or customer‑specific regimes).  
- Correct maintenance of `BILL_REGIME` is required for downstream billing, compliance, and reporting.  

---

## 3. Functional Solution Overview (FS)  

1. **Read**  
   - When the user executes the report with **Action = Road** and provides selection criteria for Shipment (`S_TKNUM`) and Delivery (`S_VBELN`), the program reads road settlement data from table `ZTRSTLMNT`.  
   - The SELECT is extended to **include `BILL_REGIME`** so that it is available in the internal table `GT_TRSTLMNT_RD`.  

2. **Display / Edit**  
   - In the Road ALV, a new column **“Bill Regime”** is added for each road settlement line.  
   - This column is **editable** in the same way as other Road settlement attributes (subject to existing screen and status controls).  

3. **Save**  
   - On pressing **SAVE** while **Action = Road**, and after the usual selection of records via the ALV checkbox, the program updates table `ZTRSTLMNT`.  
   - The `UPDATE ZTRSTLMNT` statement is extended to **set `BILL_REGIME` from the current ALV value** (`GW_TRSTLMNT_RD-BILL_REGIME`).  
   - Change documents for `ZTRSTLMNT` continue to be written via existing FM `ZTRSTLMNT_WRITE_DOCUMENT`; `BILL_REGIME` automatically participates through the standard structures used.  

---

## 4. Detailed Functional Requirements (FS)  

### 4.1 Selection Screen  

- No new selection fields are introduced.  
- The update behavior for `BILL_REGIME` is driven by existing:  
  - **Action parameter** `P_RD` (Road mode).  
  - **Shipment** and **Delivery** ranges `S_TKNUM`, `S_VBELN`.  
  - ALV **checkbox selection** of rows.  

### 4.2 ALV Layout – Road  

- New ALV column:  
  - **Technical field**: `BILL_REGIME`  
  - **Column text**: `Bill Regime`  
  - **Displayed only when** `P_RD = 'X'`.  
  - Placed after existing Road‑specific settlement fields (after “Headrun Amount”) to minimize layout impact.  
- Editability:  
  - `BILL_REGIME` is editable following the default ALV rules for Road (no special restriction beyond existing ones).  
  - Field is not a checkbox, date, or quantity; it is treated as a standard editable character field.  

### 4.3 Update Rules  

- `BILL_REGIME` in table `ZTRSTLMNT` is updated only when all of the following are true:  
  - Action = Road (`P_RD = 'X'`).  
  - User presses **SAVE** on the Road ALV screen.  
  - The line is **selected** via the existing checkbox (`CHECK_BOX = 'X'` in `GT_TRSTLMNT_RD_CHK`).  
  - The processing status (`PRO_STATUS`) allows update (≤ existing threshold, e.g. ≤ 5).  
- The key used for the update is unchanged:  
  - `TKNUM` (Shipment), `VBELN` (Delivery), `COUNTER`, `VSART` (Transportation type).  

### 4.4 Out of Scope  

- No change to Rail, Marine/Air, Coal, GTA, Shortage, Invoice, or Vehicle logic.  
- No change to tax determination, pricing, or FI postings; only persistence of the `BILL_REGIME` attribute.  

---

## 5. Impact Analysis (FS)  

- **Impacted objects**  
  - Report/include `ZSCE_SETTLEMENT` (class `LCL_DATA_SELECTION`).  
  - Road ALV layout and Road save logic.  
  - Database table `ZTRSTLMNT` (field `BILL_REGIME` now actively used).  

- **No impact**  
  - Other modes’ selection logic, ALV layouts, and update routines.  
  - Existing lock and change‑document logic (reused as is).  

---

## 6. Assumptions & Constraints (FS)  

- Field `BILL_REGIME` already exists in `ZTRSTLMNT` with appropriate data type/domain and length.  
- Allowed values and any value‑help (F4) for `BILL_REGIME` are defined via the corresponding domain or customizing; this change does not introduce a dedicated F4 implementation.  
- Performance impact is minimal because only one additional column is selected and updated.  

---

## 7. Test Scenarios (FS)  

1. **Display existing Bill Regime**  
   - Precondition: Some `ZTRSTLMNT` records for Road have `BILL_REGIME` populated.  
   - Steps: Run `ZSCE_SETTLEMENT` with Action = Road, select relevant `S_TKNUM`, `S_VBELN`, execute.  
   - Expected: ALV shows “Bill Regime” with correct values for each Road line.  

2. **Maintain Bill Regime for selected lines**  
   - Steps:  
     - Execute report with Action = Road.  
     - In ALV, change `Bill Regime` for selected lines and tick the checkbox.  
     - Press SAVE.  
   - Expected: `ZTRSTLMNT-BILL_REGIME` updated for the selected lines only; change documents created as per existing logic.  

3. **Non‑selected line not updated**  
   - Steps: Change `Bill Regime` but do **not** tick the checkbox; press SAVE.  
   - Expected: `ZTRSTLMNT-BILL_REGIME` remains unchanged for that line.  

4. **Processing status restriction**  
   - Steps: Attempt to update `Bill Regime` for a line with status > allowed threshold (e.g. `PRO_STATUS > 5`).  
   - Expected: Existing status validation blocks update; `BILL_REGIME` not changed in `ZTRSTLMNT`.  

---

## 8. Technical Specification (TS)  

### 8.1 Objects and Components  

- **Report / Include**: `ZSCE_SETTLEMENT` / `ZSCE_SETTLEMENT_RPT_CLS` (represented by `ZSCE_SETTLEMENT.md`).  
- **Local class**: `LCL_DATA_SELECTION`.  
  - Methods impacted:  
    - Road data selection (SELECT from `ZTRSTLMNT` into `GT_TRSTLMNT_RD`).  
    - `FILL_FIELD_CATALOG`.  
    - `ROAD_DATA_SAVE`.  
- **Database table**: `ZTRSTLMNT` (field `BILL_REGIME`).  
- **Change document FM**: `ZTRSTLMNT_WRITE_DOCUMENT` (reused).  

---

### 8.2 Data Selection Logic – Road (Read)  

**Location**: Road data selection section, where `GT_TRSTLMNT_RD` is filled.  

- Extend the existing `SELECT` on `ZTRSTLMNT` to include `BILL_REGIME` in the field list:  
  - This ensures `GT_TRSTLMNT_RD` and its corresponding structures carry `BILL_REGIME` for every Road record.  
- Selection conditions remain:  
  - `TKNUM IN S_TKNUM`, `VBELN IN S_VBELN`.  

This change adheres to ABAP guidelines by keeping an explicit field list and not using `SELECT *`.  

---

### 8.3 ALV Field Catalog – Road (Display/Edit)  

**Location**: Method `LCL_DATA_SELECTION->FILL_FIELD_CATALOG`.  

- For `P_RD = 'X'`, add a new field catalog entry by calling `PERFORM FILL_FCAT` with:  
  - `LP1 = 'BILL_REGIME'`  
  - `LP2 = ' '` (non‑key field)  
  - `LP3 = 'Bill Regime'` (column text).  
- `FORM FILL_FCAT` automatically sets `GW_FCAT-TABNAME` to the appropriate internal table (`GC_GT_TRSTLMNT_RD`) when `P_RD = 'X'`.  
- Since `BILL_REGIME` has no explicit branch in the `CASE GW_FCAT-FIELDNAME`, it falls under `WHEN OTHERS` and is treated as a standard editable column (`GW_FCAT-EDIT = 'X'`), consistent with the requirement.  

---

### 8.4 Save Logic – Road (Update)  

**Location**: Method `LCL_DATA_SELECTION->ROAD_DATA_SAVE`.  

- Inside the loop where Road records are processed for update after enqueue and status checks, the `UPDATE ZTRSTLMNT` statement is extended to include:  
  - `BILL_REGIME = GW_TRSTLMNT_RD-BILL_REGIME`.  
- All existing conditions are preserved:  
  - Record must be selected in `GT_TRSTLMNT_RD_CHK` (`CHECK_BOX = 'X'`).  
  - Status (`PRO_STATUS`) must be within allowed range (e.g. ≤ 5).  
  - Record must be successfully locked via `ENQUEUE_EZ_ZTRSTLMNT`.  
- After the successful update, `ZTRSTLMNT_WRITE_DOCUMENT` is called as before; since both `LT_YZTRSTLMNT_RD_OLD` and `LT_YZTRSTLMNT_RD_NEW` are typed against the full structure, `BILL_REGIME` is automatically part of the change‑document payload.  

---

### 8.5 Error Handling & Messages  

- No new messages are introduced. Existing messages continue to control behavior:  
  - “No record selected” when no ALV checkbox record is chosen.  
  - “Selected line status is not below 5” when status prevents update.  
- Error handling in enqueue/dequeue and database update remains unchanged.  

---

### 8.6 Performance Considerations  

- Minimal impact; one additional field in the SELECT and UPDATE statements for Road records only.  
- No change to selection criteria or indexing strategy.  

---

### 8.7 Testing (TS View)  

- Debug `ROAD_DATA_SAVE` to verify:  
  - `GW_TRSTLMNT_RD-BILL_REGIME` reflects the ALV value when SAVE is executed.  
  - `ZTRSTLMNT-BILL_REGIME` is updated correctly for selected lines.  
  - Change‑document tables `XZTRSTLMNT` and `YZTRSTLMNT` contain before/after values for `BILL_REGIME`.  


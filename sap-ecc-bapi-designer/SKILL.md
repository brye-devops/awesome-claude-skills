---
name: sap-ecc-bapi-designer
description: Designs and scaffolds SAP ECC BAPIs (Business Application Programming Interfaces) for Logistics modules (MM/SD/PP). Use when a developer needs to create a new BAPI function module for materials management, purchasing, sales orders, goods movements, or production orders — including naming, parameter definition, ABAP stub code, and return-structure setup.
---

# SAP ECC BAPI Designer (Logistics / MM / SD / PP)

This skill guides you through the full design and scaffolding of a standards-compliant SAP ECC BAPI for Logistics business objects. It produces a correctly named function module stub with all required parameters, proper RETURN handling, and RFC-ready attributes.

## When to Use This Skill

- Creating a new custom BAPI for a Logistics business object (Material, Purchase Order, Sales Order, Delivery, Goods Movement, Production Order, etc.)
- Extending an existing SAP standard BAPI with a Z-copy
- Scaffolding the ABAP function module code before detailed implementation
- Reviewing whether a planned BAPI interface follows SAP naming and design standards
- Generating BAPI parameter tables (IMPORT / EXPORT / TABLES / EXCEPTIONS) from a business requirement

## What This Skill Does

1. **Clarifies Requirements**: Asks targeted questions about the business object, method (Create / Change / GetDetail / GetList, etc.), and which data fields are needed.
2. **Applies SAP BAPI Standards**: Enforces correct naming (`BAPI_<OBJECT>_<METHOD>` or `Z_BAPI_<OBJECT>_<METHOD>` for custom), mandatory RETURN table, no internal COMMIT WORK, RFC-enabled attributes.
3. **Defines the Interface**: Produces a complete parameter list with SAP Dictionary types drawn from standard Logistics structures (e.g., `BAPIMEPOHEADER`, `BAPIRET2`, `BAPISDHEAD`, `BAPI2017_GM_ITEM_CREATE`).
4. **Generates an ABAP Stub**: Outputs a ready-to-paste ABAP function module skeleton with all parameter declarations, local-variable placeholders, and standard error-return logic.
5. **Documents the BAPI**: Provides a short functional description, parameter descriptions, and exception documentation suitable for SE37 / transaction BAPI.

---

## Process

### Step 1 — Gather Requirements

Ask the developer for:

| Question | Why it matters |
|---|---|
| Business object (e.g., Purchase Order, Material, Delivery) | Determines standard SAP structures to reuse |
| Method verb (Create / Change / GetDetail / GetList / Cancel / Post) | Drives naming and parameter direction |
| Key fields to pass in (header + item level if applicable) | Defines IMPORT / TABLES parameters |
| Key data to return | Defines EXPORT / TABLES parameters |
| Custom namespace? (Z_ prefix?) | Naming convention |
| Target SAP release (ECC 6.0 EhP?) | Affects available Dictionary types |

If the user has already provided these details in their request, skip ahead to Step 2.

---

### Step 2 — Apply Naming Convention

Follow SAP BAPI naming rules:

```
Standard SAP:  BAPI_<OBJECTTYPE>_<METHOD>
Custom (Z):    Z_BAPI_<OBJECTTYPE>_<METHOD>
```

Common Logistics object type tokens:

| Module | Business Object | Token examples |
|---|---|---|
| MM – Purchasing | Purchase Order | `PO`, `PURCHORDER` |
| MM – Purchasing | Purchase Requisition | `PRREQUISITION` |
| MM – Inventory | Goods Movement | `GOODSMVT` |
| MM – Master Data | Material | `MATERIAL` |
| SD – Sales | Sales Order | `SALESORDER` |
| SD – Shipping | Delivery | `DELIVERYPROCESSING` |
| PP – Production | Production Order | `PRODORD` |

Method tokens: `CREATE`, `CHANGE`, `GETDETAIL`, `GETLIST`, `CANCEL`, `SAVEREPLICAREPLICATION`, `POST`

Example: `Z_BAPI_GOODSMVT_CREATE`

---

### Step 3 — Define Parameters

Every BAPI **must** include:

```abap
TABLES
  RETURN  TYPE BAPIRET2  " mandatory — never remove
```

Build the rest of the interface from the table below, picking the correct standard SAP structures:

#### MM – Purchase Order structures

| Parameter | Type / Structure | Direction |
|---|---|---|
| `POHEADER` | `BAPIMEPOHEADER` | IMPORT |
| `POHEADERX` | `BAPIMEPOHEADERX` | IMPORT (change flags) |
| `POITEM` | `BAPIMEPOITEM` | TABLES |
| `POITEMX` | `BAPIMEPOITEMX` | TABLES |
| `POSCHEDULE` | `BAPIMEPOSCHEDULE` | TABLES |
| `POACCOUNT` | `BAPIMEPOACCOUNT` | TABLES |
| `PURCHASEORDER` | `BAPIMEPOHEADER-PO_NUMBER` (CHAR10) | EXPORT |

#### MM – Goods Movement structures

| Parameter | Type / Structure | Direction |
|---|---|---|
| `GOODSMVT_HEADER` | `BAPI2017_GM_HEAD_01` | IMPORT |
| `GOODSMVT_CODE` | `BAPI2017_GM_CODE` | IMPORT |
| `GOODSMVT_ITEM` | `BAPI2017_GM_ITEM_CREATE` | TABLES |
| `MATERIALDOCUMENT` | `BKPF-BELNR` (CHAR10) | EXPORT |
| `MATDOCUMENTYEAR` | `BKPF-GJAHR` (GJAHR) | EXPORT |

#### SD – Sales Order structures

| Parameter | Type / Structure | Direction |
|---|---|---|
| `ORDER_HEADER_IN` | `BAPISDHD1` | IMPORT |
| `ORDER_HEADER_INX` | `BAPISDHD1X` | IMPORT |
| `ORDER_ITEMS_IN` | `BAPISDITM` | TABLES |
| `ORDER_ITEMS_INX` | `BAPISDITMX` | TABLES |
| `ORDER_PARTNERS` | `BAPIPARNR` | TABLES |
| `SALESDOCUMENT` | `BAPIVBELN-VBELN` (CHAR10) | EXPORT |

#### MM – Material Master structures (BAPI_MATERIAL_SAVEDATA)

`BAPI_MATERIAL_SAVEDATA` both creates and changes material master records. Pass only the views
you need; activate each view with the corresponding flag in `HEADDATA`.

| Parameter | Type / Structure | Direction | Notes |
|---|---|---|---|
| `HEADDATA` | `BAPIMATHEAD` | IMPORT | Material number, material type, industry sector, view-activation flags |
| `CLIENTDATA` | `BAPIMATTR` | TABLES | Client-level basic data (material group, base UoM, old mat. no.) |
| `CLIENTDATAX` | `BAPIMATTRX` | TABLES | Change flags for `CLIENTDATA` fields — set `'X'` per field to update |
| `MATERIALDESCRIPTION` | `BAPIMAKT` | TABLES | Short text per language (LANGU + MATL_DESC) |
| `PLANTDATA` | `BAPIMARD` | TABLES | Plant-level MRP/planning data (one row per plant) |
| `PLANTDATAX` | `BAPIMARDX` | TABLES | Change flags for `PLANTDATA` |
| `PURCHASINGDATA` | `BAPIMARC` | TABLES | Purchasing view per plant (purch. group, GR processing time, order unit) |
| `PURCHASINGDATAX` | `BAPIMARCX` | TABLES | Change flags for `PURCHASINGDATA` |
| `VALUATIONDATA` | `BAPIMMBW` | TABLES | Accounting/valuation view (price control, standard/moving-avg price) |
| `VALUATIONDATAX` | `BAPIMMBWX` | TABLES | Change flags for `VALUATIONDATA` |
| `SALESDATA` | `BAPIMVKE` | TABLES | Sales org / distribution channel view |
| `SALESDATAX` | `BAPIMVKEX` | TABLES | Change flags for `SALESDATA` |
| `UNITSOFMEASURE` | `BAPIMARM` | TABLES | Alternative units of measure (AUoM) |
| `STORAGELOCATIONDATA` | `BAPIMARA` | TABLES | Storage location–level data |
| `RETURN` | `BAPIRET2` | TABLES | Messages — mandatory |

**HEADDATA view-activation flags** (`BAPIMATHEAD` fields set to `'X'`):

| Field | View activated |
|---|---|
| `BASIC_VIEW` | Basic Data 1 & 2 |
| `PURCHASE_VIEW` | Purchasing |
| `MRP_VIEW` | MRP 1–4 |
| `ACCOUNT_VIEW` | Accounting / Valuation |
| `SALES_VIEW` | Sales: Sales Org. data |
| `STORE_VIEW` | Storage / Warehouse |
| `QUALITY_VIEW` | Quality Management |

---

#### PP – Production Order structures

| Parameter | Type / Structure | Direction |
|---|---|---|
| `ORDER_HEADER` | `BAPI_PP_ORDER_HEADER` | IMPORT |
| `ORDER_COMPONENTS` | `BAPI_PP_ORDER_COMPONENT` | TABLES |
| `ORDER_SEQUENCES` | `BAPI_PP_ORDER_SEQUENCE` | TABLES |
| `ORDER_OPERATIONS` | `BAPI_PP_ORDER_OPERATION` | TABLES |
| `ORDER_NUMBER` | `AUFNR` (CHAR12) | EXPORT |

---

### Step 4 — Generate ABAP Stub

Produce a complete, paste-ready ABAP function module using this template pattern:

```abap
FUNCTION Z_BAPI_<OBJECT>_<METHOD>.
*"----------------------------------------------------------------------
*"  Functional description:
*"    <One-sentence description of what the BAPI does>
*"
*"  Author : <developer>
*"  Date   : <YYYY-MM-DD>
*"  Version: 1.0
*"----------------------------------------------------------------------
*"*"Local Interface:
*"  IMPORTING
*"    VALUE(<PARAM>) TYPE <TYPE>
*"  EXPORTING
*"    VALUE(<PARAM>) TYPE <TYPE>
*"  TABLES
*"    <TABLE_PARAM> STRUCTURE <STRUCTURE>
*"    RETURN STRUCTURE BAPIRET2
*"----------------------------------------------------------------------

  " ── Local data ──────────────────────────────────────────────────────
  DATA: lv_subrc  TYPE sy-subrc,
        ls_return TYPE bapiret2.

  " ── Input validation ────────────────────────────────────────────────
  " TODO: validate mandatory fields; append error rows to RETURN and
  "       RETURN early if any E/A messages found.

  " ── Core logic ──────────────────────────────────────────────────────
  " TODO: implement business logic here

  " ── Collect messages ────────────────────────────────────────────────
  CALL FUNCTION 'BAL_GLB_MSG_READ'   " or use BAPI_TRANSACTION_COMMIT
    " ... (see BAPI_TRANSACTION_COMMIT notes below)
    .

  " ── Standard success return ─────────────────────────────────────────
  IF sy-subrc = 0.
    ls_return-type    = 'S'.
    ls_return-id      = '<MSG_CLASS>'.
    ls_return-number  = '<MSG_NO>'.
    ls_return-message = '<Success message>'.
    APPEND ls_return TO return.
  ENDIF.

ENDFUNCTION.
```

**Key rules to always include in the stub comments:**

```abap
*  !! IMPORTANT — BAPI rules:
*  1. Never call COMMIT WORK inside this function module.
*     The caller is responsible for BAPI_TRANSACTION_COMMIT / ROLLBACK.
*  2. Never call MESSAGE ... (dialog messages break RFC).
*     Use the RETURN table for all messages (type S/W/E/A/X).
*  3. RFC attributes: Remote-Enabled Module = RFC Enabled (not Update Task).
```

---

### Step 5 — SE37 Attributes Checklist

Remind the developer of the required SE37 settings:

| Attribute | Required value |
|---|---|
| Processing type | Remote-Enabled Module |
| Pass value | All IMPORT params must use VALUE() |
| Unicode checks | Active |
| Short text | Filled with a meaningful business description |

---

### Step 6 — BOR Registration (optional)

If the BAPI should be visible in transaction BAPI / SWO1:

1. Go to **SWO1** → find or create the Business Object Type (e.g., `BUS2012` for PO).
2. Add the new method under the object's **Methods** node.
3. Set the **Function Module** field to the BAPI name.
4. Release the object and generate the runtime.

For purely technical / internal BAPIs this step can be skipped.

---

## How to Use

### Basic Usage

```
Design a BAPI to create a purchase order in SAP ECC.
```

```
Scaffold a custom BAPI Z_BAPI_GOODSMVT_CREATE for a goods receipt
against a purchase order. Include header, item, and return parameters.
```

```
Create a wrapper BAPI to create or update a material master record
for material type ROH, with basic data, MRP for plant 1000, and
standard price. Generate the full ABAP code.
```

### Advanced Usage

```
Create a BAPI named Z_BAPI_SALESORDER_CREATEFROMDAT2 for SAP ECC SD.
It needs to accept order header (sold-to, ship-to, requested delivery
date), up to 100 line items (material, quantity, plant, storage
location), and return the created sales document number plus all
messages. Generate the full ABAP stub.
```

---

## Example

**User**: "I need a BAPI to post a goods receipt (movement type 101) against a purchase order."

**Output**:

```
BAPI Name: Z_BAPI_GOODSMVT_CREATE

IMPORTING
  GOODSMVT_HEADER  TYPE BAPI2017_GM_HEAD_01   " Posting date, document date
  GOODSMVT_CODE    TYPE BAPI2017_GM_CODE       " Movement code (GM_CODE = '01' for GR vs PO)

EXPORTING
  MATERIALDOCUMENT TYPE BKPF-BELNR             " Created material document number
  MATDOCUMENTYEAR  TYPE BKPF-GJAHR             " Fiscal year of document

TABLES
  GOODSMVT_ITEM    TYPE BAPI2017_GM_ITEM_CREATE " Line items: PO number, PO item, qty, UoM
  RETURN           TYPE BAPIRET2                " Messages (mandatory)

RFC attribute : Remote-Enabled Module
No COMMIT WORK inside — caller uses BAPI_TRANSACTION_COMMIT
```

Followed by the full ABAP function module stub.

---

## Worked Example: Material Master Create / Update

### Scenario

An external system (ERP integration middleware or BTP) needs to create a new raw material
or update an existing one in SAP ECC. The call must set basic data, MRP settings for one
plant, and the standard price for accounting — in a single RFC-callable function module.

### Step 1 — Choose the approach

The standard SAP BAPI `BAPI_MATERIAL_SAVEDATA` already handles both create and change
(it auto-detects from the material number). The recommended pattern is to build a
**thin Z-wrapper** that simplifies the interface for the caller:

- Exposes only the fields the integration actually needs
- Fills the verbose `X`-flag structures automatically
- Handles `BAPI_TRANSACTION_COMMIT` / `BAPI_TRANSACTION_ROLLBACK` internally
  (acceptable for a dedicated wrapper; the caller should not need to know SAP locking)

### Step 2 — Interface design

```
BAPI name: Z_BAPI_MATERIAL_MAINTAINDATA

IMPORTING  (all VALUE() — RFC pass-by-value)
  VALUE(I_MATERIAL)     TYPE  MARA-MATNR         " 18-char material number (leading zeros OK)
  VALUE(I_IND_SECTOR)   TYPE  MARA-MBRSH          " Industry sector: 'M' Mech. Eng., 'A' Plant...
  VALUE(I_MATL_TYPE)    TYPE  MARA-MTART          " e.g. ROH, FERT, HALB, HIBE
  VALUE(I_PLANT)        TYPE  MARC-WERKS          " Plant for MRP + accounting views
  VALUE(I_BASE_UOM)     TYPE  MARA-MEINS          " Base unit of measure  e.g. 'KG', 'EA', 'L'
  VALUE(I_MATL_GROUP)   TYPE  MARA-MATKL          " Material group
  VALUE(I_DESCRIPTION)  TYPE  MAKT-MAKTX          " Short text (language = sy-langu)
  VALUE(I_MRP_TYPE)     TYPE  MARC-DISMM          " e.g. 'PD' (MRP), 'ND' (no planning)
  VALUE(I_LOT_SIZE)     TYPE  MARC-DISLS          " Lot-sizing procedure e.g. 'EX', 'FX'
  VALUE(I_PRICE_CTRL)   TYPE  MBEW-VPRSV          " 'S' standard price / 'V' moving average
  VALUE(I_STD_PRICE)    TYPE  MBEW-STPRS          " Standard price (used when I_PRICE_CTRL = 'S')
  VALUE(I_MOVING_AVE)   TYPE  MBEW-VERPR          " Moving avg price (used when = 'V')

EXPORTING
  VALUE(E_MATERIAL)     TYPE  MARA-MATNR          " Confirmed material number (after internal numbering)

TABLES
  RETURN                STRUCTURE BAPIRET2        " Messages — mandatory
```

### Step 3 — Full ABAP stub

```abap
FUNCTION Z_BAPI_MATERIAL_MAINTAINDATA.
*"----------------------------------------------------------------------
*"  Create or update a material master record (basic, MRP, accounting).
*"  Wraps BAPI_MATERIAL_SAVEDATA and commits the transaction.
*"----------------------------------------------------------------------
*"*"Local Interface:
*"  IMPORTING
*"    VALUE(I_MATERIAL)    TYPE  MARA-MATNR
*"    VALUE(I_IND_SECTOR)  TYPE  MARA-MBRSH
*"    VALUE(I_MATL_TYPE)   TYPE  MARA-MTART
*"    VALUE(I_PLANT)       TYPE  MARC-WERKS
*"    VALUE(I_BASE_UOM)    TYPE  MARA-MEINS
*"    VALUE(I_MATL_GROUP)  TYPE  MARA-MATKL
*"    VALUE(I_DESCRIPTION) TYPE  MAKT-MAKTX
*"    VALUE(I_MRP_TYPE)    TYPE  MARC-DISMM
*"    VALUE(I_LOT_SIZE)    TYPE  MARC-DISLS
*"    VALUE(I_PRICE_CTRL)  TYPE  MBEW-VPRSV
*"    VALUE(I_STD_PRICE)   TYPE  MBEW-STPRS
*"    VALUE(I_MOVING_AVE)  TYPE  MBEW-VERPR
*"  EXPORTING
*"    VALUE(E_MATERIAL)    TYPE  MARA-MATNR
*"  TABLES
*"    RETURN STRUCTURE BAPIRET2
*"----------------------------------------------------------------------

  " ── Local data ────────────────────────────────────────────────────
  DATA: ls_headdata    TYPE bapimathead,
        ls_clientdata  TYPE bapimattr,
        ls_clientdatax TYPE bapimattrx,
        ls_plantdata   TYPE bapimard,
        ls_plantdatax  TYPE bapimardx,
        ls_valdata     TYPE bapimmbw,
        ls_valdatax    TYPE bapimmbwx,
        ls_desc        TYPE bapimakt,
        lt_clientdata  TYPE TABLE OF bapimattr,
        lt_clientdatax TYPE TABLE OF bapimattrx,
        lt_plantdata   TYPE TABLE OF bapimard,
        lt_plantdatax  TYPE TABLE OF bapimardx,
        lt_valdata     TYPE TABLE OF bapimmbw,
        lt_valdatax    TYPE TABLE OF bapimmbwx,
        lt_desc        TYPE TABLE OF bapimakt,
        ls_return      TYPE bapiret2.

  " ── Input validation ──────────────────────────────────────────────
  IF i_base_uom IS INITIAL.
    ls_return-type    = 'E'.
    ls_return-message = 'Base unit of measure is mandatory'.
    APPEND ls_return TO return.
    RETURN.
  ENDIF.

  IF i_price_ctrl <> 'S' AND i_price_ctrl <> 'V'.
    ls_return-type    = 'E'.
    ls_return-message = 'Price control must be S (standard) or V (moving average)'.
    APPEND ls_return TO return.
    RETURN.
  ENDIF.

  " ── HEADDATA — material identity + view activation flags ──────────
  ls_headdata-material    = i_material.      " blank = internal number assignment
  ls_headdata-ind_sector  = i_ind_sector.    " e.g. 'M'
  ls_headdata-matl_type   = i_matl_type.     " e.g. 'ROH'
  ls_headdata-basic_view  = 'X'.
  ls_headdata-mrp_view    = 'X'.
  ls_headdata-account_view = 'X'.

  " ── CLIENTDATA — basic data ───────────────────────────────────────
  ls_clientdata-material   = i_material.
  ls_clientdata-base_uom   = i_base_uom.
  ls_clientdata-matl_group = i_matl_group.
  APPEND ls_clientdata TO lt_clientdata.

  " ── CLIENTDATAX — flag every field being set ──────────────────────
  ls_clientdatax-material   = i_material.
  ls_clientdatax-base_uom   = 'X'.
  ls_clientdatax-matl_group = 'X'.
  APPEND ls_clientdatax TO lt_clientdatax.

  " ── MATERIALDESCRIPTION — short text ──────────────────────────────
  ls_desc-material  = i_material.
  ls_desc-langu     = sy-langu.
  ls_desc-matl_desc = i_description.
  APPEND ls_desc TO lt_desc.

  " ── PLANTDATA — MRP settings ──────────────────────────────────────
  ls_plantdata-material  = i_material.
  ls_plantdata-plant     = i_plant.
  ls_plantdata-mrp_type  = i_mrp_type.
  ls_plantdata-lot_size  = i_lot_size.
  APPEND ls_plantdata TO lt_plantdata.

  ls_plantdatax-material  = i_material.
  ls_plantdatax-plant     = i_plant.
  ls_plantdatax-mrp_type  = 'X'.
  ls_plantdatax-lot_size  = 'X'.
  APPEND ls_plantdatax TO lt_plantdatax.

  " ── VALUATIONDATA — accounting / price ────────────────────────────
  ls_valdata-material   = i_material.
  ls_valdata-val_area   = i_plant.           " valuation area = plant in standard config
  ls_valdata-price_ctrl = i_price_ctrl.
  IF i_price_ctrl = 'S'.
    ls_valdata-std_price = i_std_price.
  ELSE.
    ls_valdata-moving_ave = i_moving_ave.
  ENDIF.
  APPEND ls_valdata TO lt_valdata.

  ls_valdatax-material   = i_material.
  ls_valdatax-val_area   = i_plant.
  ls_valdatax-price_ctrl = 'X'.
  ls_valdatax-std_price  = 'X'.
  ls_valdatax-moving_ave = 'X'.
  APPEND ls_valdatax TO lt_valdatax.

  " ── Call standard BAPI ────────────────────────────────────────────
  CALL FUNCTION 'BAPI_MATERIAL_SAVEDATA'
    EXPORTING
      headdata            = ls_headdata
    IMPORTING
      return              = ls_return       " single RETURN for header errors
    TABLES
      clientdata          = lt_clientdata
      clientdatax         = lt_clientdatax
      materialdescription = lt_desc
      plantdata           = lt_plantdata
      plantdatax          = lt_plantdatax
      valuationdata       = lt_valdata
      valuationdatax      = lt_valdatax
      returnmessages      = return.         " full message table

  " ── Check for errors before committing ───────────────────────────
  READ TABLE return WITH KEY type = 'E' TRANSPORTING NO FIELDS.
  IF sy-subrc = 0.
    CALL FUNCTION 'BAPI_TRANSACTION_ROLLBACK'.
    RETURN.
  ENDIF.

  READ TABLE return WITH KEY type = 'A' TRANSPORTING NO FIELDS.
  IF sy-subrc = 0.
    CALL FUNCTION 'BAPI_TRANSACTION_ROLLBACK'.
    RETURN.
  ENDIF.

  " ── Commit ────────────────────────────────────────────────────────
  CALL FUNCTION 'BAPI_TRANSACTION_COMMIT'
    EXPORTING
      wait = 'X'.                           " synchronous commit — wait for update task

  " ── Return confirmed material number ─────────────────────────────
  e_material = i_material.
  IF e_material IS INITIAL.
    " If internal number assignment was used, read back the number
    READ TABLE return INTO ls_return WITH KEY type = 'S'.
    " The material number is typically in ls_return-message_v1 for internal numbering
    e_material = ls_return-message_v1.
  ENDIF.

ENDFUNCTION.
```

### Step 4 — Caller pattern (ABAP report / program)

```abap
DATA: lt_return  TYPE TABLE OF bapiret2,
      lv_matnr   TYPE mara-matnr.

CALL FUNCTION 'Z_BAPI_MATERIAL_MAINTAINDATA'
  EXPORTING
    i_material    = '000000000000012345'
    i_ind_sector  = 'M'              " Mechanical Engineering
    i_matl_type   = 'ROH'            " Raw material
    i_plant       = '1000'
    i_base_uom    = 'KG'
    i_matl_group  = '001'
    i_description = 'Steel plate 10mm'
    i_mrp_type    = 'PD'
    i_lot_size    = 'EX'
    i_price_ctrl  = 'S'
    i_std_price   = '15.50'
    i_moving_ave  = '0.00'
  IMPORTING
    e_material    = lv_matnr
  TABLES
    return        = lt_return.

" Evaluate messages
LOOP AT lt_return INTO DATA(ls_msg).
  WRITE: / ls_msg-type, ls_msg-message.
ENDLOOP.
```

### Key design decisions in this example

| Decision | Rationale |
|---|---|
| Wrapper calls `BAPI_TRANSACTION_COMMIT` internally | Simplifies integration — middleware caller does not need SAP LUW knowledge |
| `WAIT = 'X'` on COMMIT | Guarantees the material record is written before the RFC reply returns |
| Separate `X`-flag structures | Allows partial updates — only flagged fields are written; others left untouched |
| Rollback on first E/A message | Prevents partial saves when one view fails validation |
| Valuation area = plant | Standard SAP config; in split-valuation systems this needs adjustment |

---

## Material Master Field Reference — All Views

Every view corresponds to one `BAPI_MATERIAL_SAVEDATA` TABLES parameter (and its `X`-flag companion).
Activate each view with the matching flag in `HEADDATA` (type `BAPIMATHEAD`).

---

### HEADDATA — `BAPIMATHEAD` (IMPORT, not a table)

Identifies the material and controls which views are created/updated.

| Structure field | SAP table.field | Description |
|---|---|---|
| `MATERIAL` | `MARA-MATNR` | Material number (18-char). Leave blank for internal number assignment |
| `IND_SECTOR` | `MARA-MBRSH` | Industry sector: `M` Mechanical Eng, `A` Plant Eng, `B` Pharma, `C` Chemicals, `D` Retail, `E` Service, `F` Food, `L` Beverages... |
| `MATL_TYPE` | `MARA-MTART` | Material type: `ROH` Raw, `HALB` Semi-finished, `FERT` Finished, `HIBE` Op. supplies, `VERP` Packaging, `DIEN` Service, `NLAG` Non-stock, `UNBW` Non-valuated |
| `BASIC_VIEW` | — | `'X'` → activate Basic Data 1 & 2 |
| `PURCHASE_VIEW` | — | `'X'` → activate Purchasing |
| `MRP_VIEW` | — | `'X'` → activate MRP 1–4 |
| `ACCOUNT_VIEW` | — | `'X'` → activate Accounting / Valuation |
| `COST_VIEW` | — | `'X'` → activate Costing 1 & 2 |
| `SALES_VIEW` | — | `'X'` → activate Sales: Sales Org Data |
| `STORE_VIEW` | — | `'X'` → activate Plant Data / Storage |
| `QUALITY_VIEW` | — | `'X'` → activate Quality Management |
| `FORECAST_VIEW` | — | `'X'` → activate Forecasting |
| `WORK_SCHED_VIEW` | — | `'X'` → activate Work Scheduling |

---

### MATERIALDESCRIPTION — `BAPIMAKT` (TABLES)

One row per language. No `X`-flag companion needed — description is always fully replaced.

| Structure field | SAP table.field | Description |
|---|---|---|
| `MATERIAL` | `MAKT-MATNR` | Material number |
| `LANGU` | `MAKT-SPRAS` | Language key (e.g. `EN`, `DE`, `FR`) |
| `MATL_DESC` | `MAKT-MAKTX` | Short text (max 40 chars) |

---

### CLIENTDATA — `BAPIMATTR` + `BAPIMATTRX` (TABLES)

Basic Data 1 & 2 views. Client-level fields from `MARA`. One row per material.

| Structure field | SAP table.field | Description |
|---|---|---|
| `MATERIAL` | `MARA-MATNR` | Material number |
| `BASE_UOM` | `MARA-MEINS` | Base unit of measure (`KG`, `EA`, `L`, `M`, `PC` …) — **mandatory** |
| `MATL_GROUP` | `MARA-MATKL` | Material group |
| `DIVISION` | `MARA-SPART` | Division (for SD relevance) |
| `PROD_HIER` | `MARA-PRDHA` | Product hierarchy (up to 18 chars) |
| `OLD_MAT_NUM` | `MARA-BISMT` | Old material number (legacy cross-reference) |
| `XPLANT_STATUS` | `MARA-MSTAE` | Cross-plant material status (blocks procurement / usage) |
| `XPLANT_STATTOUSE` | `MARA-MSTDE` | Valid-from date for cross-plant status |
| `ITEM_CAT_GRP` | `MARA-MTPOS_MARA` | Item category group (SD relevance: `NORM`, `LUMF`, `ERLA` …) |
| `TRANSPORT_GRP` | `MARA-TRAGR` | Transportation group |
| `GROSS_WEIGHT` | `MARA-BRGEW` | Gross weight |
| `NET_WEIGHT` | `MARA-NTGEW` | Net weight |
| `UNIT_OF_WT` | `MARA-GEWEI` | Weight unit (`KG`, `G`, `LB` …) |
| `VOLUME` | `MARA-VOLUM` | Volume |
| `VOLUME_UNIT` | `MARA-VOLEH` | Volume unit (`L`, `ML`, `CM3` …) |
| `SIZE_DIM` | `MARA-GROES` | Size / dimensions (free text, 32 chars) |
| `CONTAINER_REQMTS` | `MARA-BEHVO` | Container requirements |
| `TEMP_COND` | `MARA-TEMPB` | Temperature conditions indicator |
| `STORAGE_CONDS` | `MARA-LGBKL` | Storage conditions |
| `HZD_MATL_NUM` | `MARA-PROFL` | Hazardous material number |
| `BATCH_MGMT` | `MARA-XCHPF` | Batch management requirement (`X` = batch-managed) |
| `CONFIGURABLE` | `MARA-KZKFG` | Material is configurable (`X` = yes) |
| `SERIAL_NO_PROFILE` | `MARA-SPROF` | Serialization profile |
| `INDUSTRY_STND` | `MARA-NORMT` | Industry standard description (DIN, ISO …) |
| `CATALOG_PROFILE` | `MARA-RBNRM` | Catalog profile |
| `EXT_MATL_GRP` | `MARA-EXTWG` | External material group |
| `CROSS_PLANT_MATL_STATUS` | `MARA-MSTAE` | (same as XPLANT_STATUS; alias in some EhP releases) |

---

### UNITSOFMEASURE — `BAPIMARM` + `BAPIMARMX` (TABLES)

Alternative units of measure (AUoM). One row per AUoM per material.

| Structure field | SAP table.field | Description |
|---|---|---|
| `MATERIAL` | `MARM-MATNR` | Material number |
| `ALT_UNIT` | `MARM-MEINH` | Alternative unit of measure (e.g. `PAL`, `BOX`, `DZ`) |
| `CONVERTER_NUMER` | `MARM-UMREZ` | Numerator of conversion factor (alt-UoM → base-UoM) |
| `CONVERTER_DENOM` | `MARM-UMREN` | Denominator of conversion factor |
| `GROSS_WEIGHT` | `MARM-BRGEW` | Gross weight in this AUoM |
| `NET_WEIGHT` | `MARM-NTGEW` | Net weight in this AUoM |
| `UNIT_OF_WT` | `MARM-GEWEI` | Weight unit |
| `VOLUME` | `MARM-VOLUM` | Volume in this AUoM |
| `VOLUME_UNIT` | `MARM-VOLEH` | Volume unit |
| `LENGTH` | `MARM-LAENG` | Length |
| `WIDTH` | `MARM-BREIT` | Width |
| `HEIGHT` | `MARM-HOEHE` | Height |
| `UNIT_DIM` | `MARM-MEABM` | Unit of dimension (`MM`, `CM`, `M`, `IN` …) |
| `MAX_STACK_FACTOR` | `MARM-MAXSTK` | Maximum stacking factor |

---

### INTERNATIONALARTICLENUMBERS — `BAPIMEAN` (TABLES)

EAN / GTIN / UPC codes per unit of measure. No `X`-flag companion.

| Structure field | SAP table.field | Description |
|---|---|---|
| `MATERIAL` | `MEAN-MATNR` | Material number |
| `ALT_UNIT` | `MEAN-MEINH` | Unit of measure the EAN belongs to (base UoM or AUoM) |
| `EAN_UPC` | `MEAN-EAN11` | EAN / UPC / GTIN code (max 18 chars) |
| `EAN_CATEGORY` | `MEAN-NUMTP` | EAN category: `EAN8`, `EAN13`, `UPC-A`, `GTIN14` … |
| `MAIN_EAN` | `MEAN-HPEAN` | `'X'` = this is the main / preferred EAN for this UoM |

---

### SALESDATA — `BAPIMVKE` + `BAPIMVKEX` (TABLES)

Sales Org Data 1 & 2 views. One row per sales org / distribution channel combination.
Activate with `HEADDATA-SALES_VIEW = 'X'`.

| Structure field | SAP table.field | Description |
|---|---|---|
| `MATERIAL` | `MVKE-MATNR` | Material number |
| `SALES_ORG` | `MVKE-VKORG` | Sales organization |
| `DISTR_CHAN` | `MVKE-VTWEG` | Distribution channel |
| `SALES_UNIT` | `MVKE-VRKME` | Sales unit (if different from base UoM) |
| `ITEM_CAT_GRP` | `MVKE-MTPOS` | Item category group (overrides client-level if set) |
| `ACCT_ASSMT_GRP` | `MVKE-KTGRM` | Account assignment group (revenue account determination) |
| `DELIVERING_PLANT` | `MVKE-DWERK` | Delivering plant (default for sales orders) |
| `PROD_HIER` | `MVKE-PRODH` | Product hierarchy (sales-org level) |
| `MATL_PRICING_GRP` | `MVKE-KONDM` | Material pricing group |
| `COMMISSION_GRP` | `MVKE-PROVG` | Commission group |
| `CASHDISC_IND` | `MVKE-SKTOF` | Cash discount indicator (`X` = not eligible) |
| `VAR_SALES_UNIT` | `MVKE-VAVME` | Variable sales unit not allowed (`X` = fixed) |
| `MIN_ORDER_QTY` | `MVKE-MINBF` | Minimum order quantity in sales |
| `STAT_VALUE` | `MVKE-STAWN` | Statistical value / customs tariff number |
| `MATL_STATS_GRP` | `MVKE-MVGR1` | Material statistics group 1 |
| `VOLUME_REBATE_GRP` | `MVKE-MVGR2` | Volume rebate group |
| `TRANS_GRP_MVKE` | `MVKE-MVGR3` | Material group 3 |
| `SHIPPING_MATL_TYPE`| `MVKE-MVGR4` | Material group 4 |
| `AUTH_GRP` | `MVKE-MVGR5` | Material group 5 |

---

### PURCHASINGDATA — `BAPIMARC` + `BAPIMARCX` (TABLES)

Purchasing view. One row per plant.
Activate with `HEADDATA-PURCHASE_VIEW = 'X'`.

| Structure field | SAP table.field | Description |
|---|---|---|
| `MATERIAL` | `MARC-MATNR` | Material number |
| `PLANT` | `MARC-WERKS` | Plant |
| `PURCH_GROUP` | `MARC-EKGRP` | Purchasing group |
| `ORDER_UNIT` | `MARC-BSTME` | Order unit (if different from base UoM) |
| `VAR_OD_UNIT` | `MARC-BSTME` | Variable order unit allowed (`X` = yes) |
| `OVER_DEL_TOL` | `MARC-UEBTO` | Over-delivery tolerance (%) |
| `UNDER_DEL_TOL` | `MARC-UNTTO` | Under-delivery tolerance (%) |
| `UNLIMITED_OVER_DEL`| `MARC-UEBTK` | Unlimited over-delivery allowed (`X`) |
| `GR_PROC_TIME` | `MARC-WEBAZ` | GR processing time (workdays) |
| `MATL_FRGHT_GRP` | `MARC-MFRGR` | Material freight group |
| `PURCH_VAL_KEY` | `MARC-EKWSL` | Purchasing value key (reminder / tolerance profile) |
| `SOURCE_LIST` | `MARC-KORDB` | Source list required (`1` = yes) |
| `QUOTA_ARR_USAGE` | `MARC-USEQU` | Quota arrangement usage |
| `AUTO_PO` | `MARC-AUTPURCHORD` | Automatic purchase order (`X` = allowed) |
| `TAX_IND` | `MARC-STEUC` | Tax indicator for material |
| `MANUFACTURER` | `MARC-MFRNR` | Manufacturer (vendor number) |
| `MFR_PART_PROFILE` | `MARC-MFRPN` | Manufacturer part number profile |
| `INT_COMMNT_CAT` | `MARC-EKALR` | Material is relevant for internal commerce |
| `PLANT_STATUS` | `MARC-MMSTA` | Plant-specific material status |
| `PLANT_STATUS_DATE` | `MARC-MMSTD` | Valid-from date for plant status |
| `MIN_REM_SHELF_LIFE`| `MARC-MHDLP` | Minimum remaining shelf life (days) |
| `TOTAL_SHELF_LIFE` | `MARC-MHDRZ` | Total shelf life (days) |

---

### PLANTDATA — `BAPIMARD` + `BAPIMARDX` (TABLES)

MRP 1–4 and Work Scheduling views. One row per plant.
Activate with `HEADDATA-MRP_VIEW = 'X'` and/or `HEADDATA-WORK_SCHED_VIEW = 'X'`.

| Structure field | SAP table.field | Description |
|---|---|---|
| `MATERIAL` | `MARC-MATNR` | Material number |
| `PLANT` | `MARC-WERKS` | Plant |
| **MRP 1** | | |
| `MRP_TYPE` | `MARC-DISMM` | MRP type: `PD` MRP, `VB` Reorder point, `ND` No planning, `VM` Auto reorder |
| `MRP_CTRLLER` | `MARC-DISPO` | MRP controller (planner group) |
| `ABC_INDICATOR` | `MARC-ABCIN` | ABC indicator |
| `PLANT_MRP_AREA` | `MARC-BERID` | MRP area |
| `PROC_TYPE` | `MARC-BESKZ` | Procurement type: `E` In-house, `F` External, `X` Both |
| `SPEC_PROC` | `MARC-SOBSL` | Special procurement key (e.g. `10` phantom, `30` subcontracting) |
| `LOT_SIZE` | `MARC-DISLS` | Lot-sizing procedure: `EX` Exact, `FX` Fixed, `HB` Replenish to max, `MB` Monthly |
| `MIN_LOT_SIZE` | `MARC-BSTMI` | Minimum lot size |
| `MAX_LOT_SIZE` | `MARC-BSTMA` | Maximum lot size |
| `FIXED_LOT` | `MARC-BSTFE` | Fixed lot size |
| `MAX_STOCK` | `MARC-MABST` | Maximum stock level |
| `REORDER_PT` | `MARC-MINBE` | Reorder point |
| `SAFETY_STCK` | `MARC-EISBE` | Safety stock |
| `ORDERING_COSTS` | `MARC-BSTKO` | Order costs (for EOQ lot-sizing) |
| **MRP 2** | | |
| `PLND_DELRY` | `MARC-PLIFZ` | Planned delivery time (calendar days) |
| `GR_PROC_TIME` | `MARC-WEBAZ` | Goods receipt processing time (workdays) |
| `SCHED_MARGIN_KEY` | `MARC-FHORI` | Scheduling margin key (float before/after production) |
| `IN_HOUSE_PRD` | `MARC-DZEIT` | In-house production time (workdays) |
| `LOT_SIZE_IND` | `MARC-DISLS` | (same field — context of MRP 2 planning) |
| `BACKFLUSH` | `MARC-RGEKZ` | Backflush indicator (`1` = always) |
| `SAFETY_TIME_IND` | `MARC-IPRKZ` | Safety time indicator |
| `SAFETY_TIME_DAYS` | `MARC-EISLO` | Safety time period (workdays) |
| `PLANNING_CALENDAR`| `MARC-MFPLA` | Planning calendar (for period lot sizes) |
| `AVAIL_CHECK` | `MARC-MTVFP` | Availability check rule (ATP checking rule) |
| **MRP 3** | | |
| `STRATEGY_GRP` | `MARC-STRGR` | Planning strategy group (e.g. `10` MTS, `20` MTO, `40` planning with final assembly) |
| `CONS_MODE` | `MARC-VRMOD` | Consumption mode: `1` backward, `2` forward, `3` both |
| `FWRD_CONS_PER` | `MARC-VINT1` | Forward consumption period (workdays) |
| `BKWD_CONS_PER` | `MARC-VINT2` | Backward consumption period (workdays) |
| `MIXED_MRP` | `MARC-MISCHKZ` | Mixed MRP indicator |
| `ASSEMBLY_SCRAP` | `MARC-AUSSS` | Assembly scrap (%) |
| `PLANNING_MATL` | `MARC-LGPRO` | Planning material (for planning with planning material) |
| `PLANNING_PLANT` | `MARC-LGFSB` | Planning plant |
| `INDIVIDUAL_COLL` | `MARC-SBDKZ` | Individual / collective requirements: `1` individual, `2` collective |
| `DISCONTINUED_IND` | `MARC-DISKZ` | Discontinuation indicator |
| `FOLLOW_UP_MATL` | `MARC-NFMAT` | Follow-up material number |
| **MRP 4** | | |
| `SELECTION_METHOD` | `MARC-AUSME` | Selection method (BOM / routing explosion) |
| `COMPONENT_SCRAP` | `MARC-AUSSS` | Component scrap in % (also used in MRP 4) |
| `REPETITIVE_MFG` | `MARC-MMSTA` | Repetitive manufacturing allowed |
| `REM_PROFILE` | `MARC-RWPRO` | Repetitive manufacturing profile |
| `BULK_MATERIAL` | `MARC-SHFLG` | Bulk material indicator |
| `CO_PRODUCT` | `MARC-KUPPROD` | Co-product |
| **Work Scheduling** | | |
| `PROD_SCHED_PROFILE`| `MARC-FEVOR` | Production scheduler |
| `SETUP_TIME` | `MARC-RUEZT` | Setup / teardown time |
| `INTEROP_TIME` | `MARC-INSMK` | Interoperation time |
| `BASE_QUANTITY` | `MARC-BASMG` | Base quantity for production times |
| `PROD_UNIT` | `MARC-FEREH` | Production unit |
| `UNDER_DEL_TOL` | `MARC-UNETO` | Underdelivery tolerance (work scheduling context) |
| `OVER_DEL_TOL_WS` | `MARC-UETTO` | Overdelivery tolerance (work scheduling context) |
| `UNLIMITED_OVER_DEL_WS`| `MARC-UETTO` | Unlimited overdelivery (work scheduling) |
| `ISSUE_SLOC` | `MARC-LGPRO` | Issue storage location (goods issue for production) |

---

### FORECASTPARAMETERS — `BAPIMAVEM` + `BAPIMAVEMX` (TABLES)

Forecasting view. One row per plant.
Activate with `HEADDATA-FORECAST_VIEW = 'X'`.

| Structure field | SAP table.field | Description |
|---|---|---|
| `MATERIAL` | `MPLA-MATNR` | Material number |
| `PLANT` | `MPLA-WERKS` | Plant |
| `PERIOD_IND_MPOP` | `MPOP-PERAF` | Period indicator: `M` Monthly, `W` Weekly, `D` Daily |
| `FISCAL_YEAR_VAR` | `MPLA-PERIV` | Fiscal year variant |
| `FORECAST_MODEL` | `MPLA-PRMOD` | Forecast model: `D` Constant, `T` Trend, `S` Seasonal, `X` Auto |
| `PERIOD_IND` | `MPLA-PERIV` | Period indicator at material level |
| `INIT_PERIODS` | `MPLA-INSMK` | Initialization periods |
| `TRACKING_LIMIT` | `MPLA-TRSPQ` | Tracking limit (MAD / sigma for outlier detection) |
| `MODEL_SELECTION` | `MPLA-AUTOM` | Automatic model selection (`X` = yes) |
| `EXSMOOTHING_FAC` | `MPLA-ALPHA` | Alpha (exponential smoothing factor) |
| `TREND_SMOOTH_FAC` | `MPLA-BETA` | Beta (trend smoothing factor) |
| `SEAS_SMOOTH_FAC` | `MPLA-GAMMA` | Gamma (seasonal smoothing factor) |
| `SIGMA` | `MPLA-SIGMA` | Sigma (outlier detection factor) |
| `PERIODS_PER_SEASON`| `MPLA-SAISO` | Periods per seasonal cycle |
| `HIST_PERIODS` | `MPLA-ANZPR` | Number of historical periods to use |

---

### STORAGELOCATIONDATA — `BAPIMARA` + `BAPIMARAX` (TABLES)

Plant Data / Storage 1 & 2 views. One row per plant / storage location.
Activate with `HEADDATA-STORE_VIEW = 'X'`.

| Structure field | SAP table.field | Description |
|---|---|---|
| `MATERIAL` | `MARD-MATNR` | Material number |
| `PLANT` | `MARC-WERKS` | Plant |
| `STGE_LOC` | `MARD-LGORT` | Storage location |
| `WM_WAREHOSENUMBER`| `MLGN-LGNUM` | WM warehouse number |
| `WM_STORAGETYPE` | `MLGT-LGTYP` | WM storage type |
| `BATCH_RECORD` | `MARC-XCHAR` | Batch classification (`X` = batch-managed at storage level) |
| `MAX_STGE_PERIOD` | `MARC-MHDRZ` | Maximum storage period |
| `STGE_CONDITION` | `MARC-LGBKL` | Storage condition |
| `TEMP_CONDITIONS` | `MARC-TEMPB` | Temperature conditions |
| `HAZMAT_NUMBER` | `MARC-PROFL` | Hazardous material / dangerous goods indicator |
| `CC_PHY_INV_IND` | `MARC-STLAL` | Cycle counting physical inventory indicator |
| `UNIT_OF_ISSUE` | `MARC-AUSME` | Unit of issue (for goods issue from production / WM) |
| `PROFIT_CENTER` | `MARC-PRCTR` | Profit center |

---

### VALUATIONDATA — `BAPIMMBW` + `BAPIMMBWX` (TABLES)

Accounting 1 & 2 views. One row per valuation area (usually = plant).
Activate with `HEADDATA-ACCOUNT_VIEW = 'X'`.

| Structure field | SAP table.field | Description |
|---|---|---|
| `MATERIAL` | `MBEW-MATNR` | Material number |
| `VAL_AREA` | `MBEW-BWKEY` | Valuation area (= plant in standard config) |
| `VAL_TYPE` | `MBEW-BWTAR` | Valuation type (for split valuation only; leave blank otherwise) |
| `PRICE_CTRL` | `MBEW-VPRSV` | Price control: `S` Standard price, `V` Moving average price |
| `STD_PRICE` | `MBEW-STPRS` | Standard price (used when `PRICE_CTRL = 'S'`) |
| `MOVING_AVE` | `MBEW-VERPR` | Moving average price (used when `PRICE_CTRL = 'V'`) |
| `PRICE_UNIT` | `MBEW-PEINH` | Price unit (quantity to which price refers, e.g. `1`, `100`) |
| `VAL_CLASS` | `MBEW-BKLAS` | Valuation class (drives G/L account determination) |
| `PRICE_DETERM` | `MBEW-EKLAS` | Price determination: `2` Transaction-based, `3` Single-/multi-level |
| `ML_ACTIVE` | `MBEW-MLAST` | Material ledger activated (`X`) |
| `WITH_QM` | `MBEW-IPRKZ` | With quality inspection stock |
| `FUTURE_PRICE` | `MBEW-ZKPRS` | Future standard price |
| `FUTURE_PRICE_DATE`| `MBEW-ZKDAT` | Valid-from date for future price |
| `PREV_PRICE` | `MBEW-VKSAL` | Previous period price |
| `ORIGIN_GROUP` | `MBEW-KKZGR` | Origin group (costing — CO-PA) |
| `PROFIT_CENTER` | `MBEW-PRCTR` | Profit center |
| `DO_NOT_COST` | `MBEW-KALKZ` | Do not cost (`X` = excluded from product costing) |

---

### SALESDATA tax classification — `BAPIMLAN` (TABLES)

Tax indicator per country / departure country / tax category. One row per combination.

| Structure field | SAP table.field | Description |
|---|---|---|
| `MATERIAL` | `MLAN-MATNR` | Material number |
| `DEPCOUNTRY` | `MLAN-ALAND` | Departure country |
| `TAX_TYPE_1` | `MLAN-TAXM1` | Tax category 1 (e.g. `1` Full, `0` Zero-rated) |
| `TAX_TYPE_2` | `MLAN-TAXM2` | Tax category 2 |
| `TAX_TYPE_3–9` | `MLAN-TAXM3`–`TAXM9` | Tax categories 3–9 (country-specific) |

---

### Quality Management — `BAPIQM` (TABLES, if available in EhP release)

QM view. Activate with `HEADDATA-QUALITY_VIEW = 'X'`.

| Structure field | SAP table.field | Description |
|---|---|---|
| `MATERIAL` | `MARC-MATNR` | Material number |
| `PLANT` | `MARC-WERKS` | Plant |
| `INSP_SETUP` | `MARC-QSSYS` | Inspection setup |
| `QM_PROC_ACTIVE` | `MARC-QMATA` | QM in procurement active (`X`) |
| `CERT_TYPE` | `MARC-QZGTP` | Certificate type |
| `DOC_REQUIRED` | `MARC-DOKST` | Documentation required indicator |
| `TARGET_QI` | `MARC-QPLOS` | Target quality inspection level |

---

### Full parameter list for `BAPI_MATERIAL_SAVEDATA`

```abap
CALL FUNCTION 'BAPI_MATERIAL_SAVEDATA'
  EXPORTING
    headdata                  = ls_headdata           " BAPIMATHEAD
  TABLES
    clientdata                = lt_clientdata          " BAPIMATTR
    clientdatax               = lt_clientdatax         " BAPIMATTRX
    materialdescription       = lt_description         " BAPIMAKT
    unitsofmeasure            = lt_uom                 " BAPIMARM
    unitsofmeasurex           = lt_uomx                " BAPIMARMX
    internationalarticlenumbers = lt_ean               " BAPIMEAN
    salesdata                 = lt_salesdata           " BAPIMVKE
    salesdatax                = lt_salesdatax          " BAPIMVKEX
    taxclassifications        = lt_taxclass            " BAPIMLAN
    purchasingdata            = lt_purchdata           " BAPIMARC
    purchasingdatax           = lt_purchdatax          " BAPIMARCX
    plantdata                 = lt_plantdata           " BAPIMARD
    plantdatax                = lt_plantdatax          " BAPIMARDX
    forecastparameters        = lt_forecast            " BAPIMAVEM
    forecastparametersx       = lt_forecastx           " BAPIMAVEMX
    storagelocationdata       = lt_sloc                " BAPIMARA
    storagelocationdatax      = lt_slocx               " BAPIMARAX
    valuationdata             = lt_valdata             " BAPIMMBW
    valuationdatax            = lt_valdatax            " BAPIMMBWX
    returnmessages            = return.                " BAPIRET2
```

> Only pass TABLES parameters for the views you have activated in `HEADDATA`.
> Unused parameters can be left as empty internal tables — do not omit the
> `TABLES` keyword itself when using the function module in older syntax.

---

## Tips

- Always check whether a standard SAP BAPI already covers the use case before building a custom one — search in transaction **BAPI** or SE37 with pattern `BAPI_<OBJECT>_*`.
- Use the `X` (change-flag) companion structure for Change BAPIs to allow partial updates without overwriting unchanged fields.
- When BAPIs call other standard BAPIs internally, collect all RETURN table entries rather than stopping at the first error.
- Test RFC-enabled function modules with **SE37 → Execute** before plugging them into the caller program.
- For mass-data scenarios consider using the `BAPI_TRANSACTION_COMMIT` + `BAPI_TRANSACTION_ROLLBACK` pattern in an outer loop rather than committing per document.

## Common Use Cases

- Creating or updating material master records from a PLM, PIM, or MDM system
- Creating a new PO from an external ordering portal via RFC/web service
- Posting goods receipts from a WMS or IoT scanning system
- Creating sales orders from an e-commerce platform integration
- Changing production order quantities or dates from a planning tool
- Building reusable interfaces for middleware (SAP PI/PO, BTP Integration Suite)

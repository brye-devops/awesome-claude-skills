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

## Tips

- Always check whether a standard SAP BAPI already covers the use case before building a custom one — search in transaction **BAPI** or SE37 with pattern `BAPI_<OBJECT>_*`.
- Use the `X` (change-flag) companion structure for Change BAPIs to allow partial updates without overwriting unchanged fields.
- When BAPIs call other standard BAPIs internally, collect all RETURN table entries rather than stopping at the first error.
- Test RFC-enabled function modules with **SE37 → Execute** before plugging them into the caller program.
- For mass-data scenarios consider using the `BAPI_TRANSACTION_COMMIT` + `BAPI_TRANSACTION_ROLLBACK` pattern in an outer loop rather than committing per document.

## Common Use Cases

- Creating a new PO from an external ordering portal via RFC/web service
- Posting goods receipts from a WMS or IoT scanning system
- Creating sales orders from an e-commerce platform integration
- Changing production order quantities or dates from a planning tool
- Building reusable interfaces for middleware (SAP PI/PO, BTP Integration Suite)

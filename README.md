# WMS Core — Public API Integration Guide

This document provides detailed API specifications for external systems integrating with WMS Core. It includes request/response payloads, field descriptions, required/optional indicators, and integration notes.

**Base URL:** `{host}/api/`  
**Content-Type:** `application/json`

---

## Table of Contents

1. [Authentication](#1-authentication)
2. [Item Master](#2-item-master)
3. [SAP Inbound Order Batch Create](#3-sap-inbound-order-batch-create)
4. [SAP Inbound Order Update](#4-sap-inbound-order-update)
4.1. [SAP Inbound Order Cancel](#41-sap-inbound-order-cancel)
5. [SAP Outbound Order Batch Create](#5-sap-outbound-order-batch-create)
6. [SAP Outbound Order Update](#6-sap-outbound-order-update)
6.1. [SAP Outbound Order Cancel](#61-sap-outbound-order-cancel)
7. [SAP Outbound Order Finish](#7-sap-outbound-order-finish)
8. [Receiving Excel Fulfill (v2)](#8-receiving-excel-fulfill-v2)
9. [Shipping Excel Fulfill (v2)](#9-shipping-excel-fulfill-v2)
10. [Transaction Adjustment](#10-transaction-adjustment)
11. [Bin Transfer](#11-bin-transfer)
12. [Stock Check](#12-stock-check)
13. [Inventory Summary](#13-inventory-summary)
14. [Reference Data](#14-reference-data)
15. [Error Handling](#15-error-handling)

---

## 1. Authentication

All API endpoints require authentication via **Token** in the `Authorization` header.

### Header


| Header          | Value                | Required       |
| --------------- | -------------------- | -------------- |
| `Authorization` | `Token <your-token>` | Yes            |
| `Content-Type`  | `application/json`   | Yes (for POST) |


### Token Types

- **User Token** — Single token string for user authentication
- **Client Token** — Format: `{key}-{client_id}` for client-scoped API access (e.g., ERP/SAP integration)

### Example

```http
Authorization: Token abc123def456
Content-Type: application/json
```

### Response (401 Unauthorized)

```json
{
  "result": "Failed",
  "data": null,
  "error": "Unauthorized"
}
```

---

## 2. Item Master

### List Item Master

**Endpoint:** `GET /api/item_master/`  
**Description:** List item master (SKU) records.

### Query Parameters


| Parameter          | Type    | Required | Description                       |
| ------------------ | ------- | -------- | --------------------------------- |
| `skip`             | integer | No       | Pagination offset. Default: 0.    |
| `take`             | integer | No       | Page size. Default: 20, max: 100. |
| `client_reference` | string  | No       | Filter by client reference.       |


### SAP Item Master POST Request

**Endpoint:** `POST /api/sap/item_master/`  
**Description:** Create a new item master (SKU) record. Requires Client Token authentication.

### Request Payload (POST)


| Field                      | Type    | Required | Description                                                                        |
| -------------------------- | ------- | -------- | ---------------------------------------------------------------------------------- |
| `client` / `client_code`   | string  | **Yes**  | Client code. Must exist under the token's organization.                            |
| `item_type`                | string  | **Yes**  | Item type code. Must exist in system.                                              |
| `product_name`             | string  | **Yes**  | Product display name.                                                              |
| `item_code` / `code`       | string  | **Yes**  | SKU/item code. Must be unique per client.                                          |
| `description`              | string  | No       | Product description.                                                               |
| `upc`                      | string  | No       | UPC/barcode.                                                                       |
| `foreign_name`             | string  | No       | Foreign language name (外文名稱).                                                  |
| `group_name`               | string  | No       | Product grouping name.                                                             |
| `remark`                   | string  | No       | Internal remark.                                                                   |
| `auom_control`             | string  | No       | Alternative UOM control. `DISABLE`, `IN`, `OUT`, `BOTH`. Default: `DISABLE`.       |
| `expiry_date_control`     | string  | No       | Expiry date control. `DISABLE`, `OUT`, `BOTH`. Default: `DISABLE`.                 |
| `lot_control`              | string  | No       | Lot number control. `DISABLE`, `OUT`, `BOTH`. Default: `DISABLE`.                  |
| `serial_number_control`    | string  | No       | Serial number control. `DISABLE`, `OUT`, `BOTH`. Default: `DISABLE`.               |
| `required_condition`       | string  | No       | Zone condition code (e.g., ambient, chilled). Must exist if provided.              |
| `force_required_condition` | boolean | No       | Enforce required condition. Default: `false`.                                      |
| `picking_method`           | string  | No       | `FIFO`, `FEFO`, or `LIFO`. Default: `FEFO` if expiry control enabled, else `FIFO`. |
| `hs_code`                  | string  | No       | Harmonized System code.                                                            |
| `custom_code`              | string  | No       | Custom/customer reference code.                                                    |
| `pick_size`                | integer | No       | Pick size. Default: 0.                                                             |
| `pack_size`                | integer | No       | Pack size. Default: 0.                                                             |
| `stock`                    | number  | No       | Stock quantity value. Default: 0.                                                  |
| `shelf_life`               | integer | No       | Shelf life value. Default: 0.                                                      |
| `pack_description`         | string  | No       | Pack description text.                                                             |
| `team`                     | string  | No       | Team name/value.                                                                   |
| `brand`                    | string  | No       | Brand name.                                                                        |
| `business_partner_code` / `業務夥伴代碼` | string  | No | Business partner code.                                                            |
| `business_partner_name` / `業務夥伴名稱` | string  | No | Business partner name.                                                            |
| `place_of_origin` / `Place Of Origin` | string  | No | Place of origin.                                                           |
| `container_type` / `Container Type`   | string  | No | Container type.                                                             |
| `wight_per_each`           | number  | No       | Weight per each item. Default: 0.                                                  |
| `weight_unit`              | string  | No       | Weight unit (e.g., `kg`, `g`).                                                     |
| `cost`                     | number  | No       | Cost price. Default: 0.                                                            |
| `selling`                  | number  | No       | Selling price. Default: 0.                                                        |
| `msrp`                     | number  | No       | MSRP. Default: 0.                                                                  |
| `active`                   | boolean | No       | Whether the item is active. Default: `true`. Accepts `true`/`false`, `"true"`/`"false"`, `1`/`0`. |
| `uom`                      | array   | No       | Array of unit-of-measurement objects.                                              |
| `custom_field`             | array   | No       | Array of custom field objects.                                                     |


### UOM Object (each item in `uom`)


| Field          | Type    | Required | Description                                              |
| -------------- | ------- | -------- | -------------------------------------------------------- |
| `unit`         | string  | **Yes**  | Unit code (e.g., EACH, CTN). Must exist in organization. |
| `qty`          | number  | No       | Conversion factor to base. Default: 0 (skipped if &lt; 1). |
| `is_base`      | boolean | No       | Whether this is the base UOM. Default: `false`.          |
| `length`       | number  | No       | Length (cm). Default: 0.                                 |
| `width`        | number  | No       | Width (cm). Default: 0.                                  |
| `height`       | number  | No       | Height (cm). Default: 0.                                 |
| `gross_weight` | number  | No       | Gross weight (kg). Default: 0.                           |
| `net_weight`   | number  | No       | Net weight (kg). Default: 0.                             |


### Custom Field Object (each item in `custom_field`)


| Field   | Type   | Required | Description         |
| ------- | ------ | -------- | ------------------- |
| `name`  | string | **Yes**  | Custom field name.  |
| `value` | any    | **Yes**  | Custom field value. |


### Example POST Request

```json
{
  "client": "DEMOCLIENT",
  "item_type": "STANDARD",
  "product_name": "Sample Product",
  "item_code": "SKU001",
  "description": "Product description",
  "upc": "123456789012",
  "foreign_name": "商品外文名稱",
  "group_name": "RAW_MATERIAL",
  "remark": "",
  "auom_control": "DISABLE",
  "expiry_date_control": "BOTH",
  "lot_control": "DISABLE",
  "serial_number_control": "DISABLE",
  "picking_method": "FEFO",
  "hs_code": "",
  "custom_code": "",
  "pick_size": 0,
  "pack_size": 0,
  "stock": 0,
  "shelf_life": 0,
  "pack_description": "12 packs x 24 each",
  "team": "TEAM_A",
  "brand": "BrandX",
  "business_partner_code": "BP001",
  "business_partner_name": "Partner Name",
  "place_of_origin": "HK",
  "container_type": "CTN",
  "wight_per_each": 0.25,
  "weight_unit": "kg",
  "cost": 0,
  "selling": 0,
  "msrp": 0,
  "active": true,
  "uom": [
    {
      "unit": "EACH",
      "qty": 1,
      "is_base": true,
      "length": 0,
      "width": 0,
      "height": 0,
      "gross_weight": 0,
      "net_weight": 0
    }
  ],
  "custom_field": []
}
```

### Success Response (201) — POST

```json
{
  "data": { ... },
  "error": null
}
```

---

### SAP Item Master PUT Request

**Endpoint:** `PUT /api/sap/item_master/<code>/`  
**Description:** Update an existing item master by SKU code. Requires Client Token authentication. Only the fields provided in the payload are updated; omitted fields retain their current values.

### Request Payload (PUT)


| Field          | Type    | Required | Description                                             |
| -------------- | ------- | -------- | ------------------------------------------------------- |
| `product_name` | string  | No       | Product display name.                                   |
| `description`  | string  | No       | Product description.                                    |
| `upc`          | string  | No       | UPC/barcode.                                            |
| `foreign_name` | string  | No       | Foreign language name (外文名稱).                       |
| `group_name`   | string  | No       | Product grouping name.                                  |
| `active`       | boolean | No       | Whether the item is active. Accepts `true`/`false`, `"true"`/`"false"`, `1`/`0`. Default: True |
| `remark`       | string  | No       | Internal remark.                                        |
| `hs_code`      | string  | No       | Harmonized System code.                                 |
| `custom_code`  | string  | No       | Custom/customer reference code.                         |
| `pick_method`  | string  | No       | `FIFO`, `FEFO`, or `LIFO`.                              |
| `pick_size`    | integer | No       | Pick size.                                              |
| `pack_size`    | integer | No       | Pack size.                                              |
| `stock`        | number  | No       | Stock quantity value.                                   |
| `shelf_life`   | integer | No       | Shelf life value.                                       |
| `pack_description` | string | No    | Pack description text.                                  |
| `team`         | string  | No       | Team name/value.                                        |
| `brand`        | string  | No       | Brand name.                                             |
| `business_partner_code` / `業務夥伴代碼` | string  | No | Business partner code.                                  |
| `business_partner_name` / `業務夥伴名稱` | string  | No | Business partner name.                                  |
| `place_of_origin` / `Place Of Origin` | string  | No | Place of origin.                                 |
| `container_type` / `Container Type`   | string  | No | Container type.                                   |
| `wight_per_each` | number | No      | Weight per each item.                                   |
| `weight_unit`  | string  | No       | Weight unit (e.g., `kg`, `g`).                          |
| `cost`         | number  | No       | Cost price.                                             |
| `selling`      | number  | No       | Selling price.                                          |
| `msrp`         | number  | No       | MSRP.                                                   |
| `uom`          | array   | No       | Array of UOM objects (same structure as POST). **Only applied when the SKU has no existing UOMs.** If the SKU already has UOMs, the `uom` payload is ignored. |
| `custom_field` | array   | No       | Array of custom field objects (same structure as POST). |


**Note:** Client, item_type, and item_code cannot be changed via PUT. **Control fields** (`auom_control`, `expiry_date_control`, `lot_control`, `serial_number_control`) are updatable only when inventory is zero (total_qty and available_qty both 0). **UOM update:** UOMs can only be added via PUT when the SKU has no UOMs yet; once UOMs exist, they cannot be changed via this endpoint. **Omitted fields:** Retain current value when not provided.

### Example PUT Request

```http
PUT /api/sap/item_master/SKU001/
Authorization: Token abc123
Content-Type: application/json
```

```json
{
  "product_name": "Updated Product Name",
  "description": "Updated description",
  "upc": "987654321098",
  "foreign_name": "更新後外文名稱",
  "group_name": "FG",
  "active": true,
  "remark": "Updated remark",
  "hs_code": "1234.56.78",
  "custom_code": "CUST-001",
  "pick_method": "FIFO",
  "pick_size": 1,
  "pack_size": 12,
  "stock": 100,
  "shelf_life": 365,
  "pack_description": "Case pack 12",
  "team": "TEAM_B",
  "brand": "BrandY",
  "business_partner_code": "BP002",
  "business_partner_name": "Updated Partner",
  "place_of_origin": "CN",
  "container_type": "PLT",
  "wight_per_each": 0.3,
  "weight_unit": "kg",
  "cost": 10.50,
  "selling": 15.00,
  "msrp": 19.99,
  "uom": [
    {
      "unit": "EACH",
      "qty": 1,
      "is_base": true,
      "length": 0,
      "width": 0,
      "height": 0,
      "gross_weight": 0,
      "net_weight": 0
    }
  ],
  "custom_field": [
    {"name": "color", "value": "red"},
    {"name": "size", "value": "M"}
  ]
}
```

### Success Response (200) — PUT

```json
{
  "data": { ... },
  "error": null
}
```

### Error Response (400 / 404) — POST and PUT

```json
{
  "data": null,
  "error": "Client 'XYZ' not found"
}
```

### Validation Notes (Item Master)

- **POST:** Item code must be unique per client; duplicate returns 400.
- **POST:** Client must exist under the token's organization.
- **POST:** Item type must exist in the organization.
- **POST:** Control codes must be valid; invalid values return 400 with allowed list.
- **PUT:** Item must exist for the token's client; otherwise 404.
- **PUT UOM:** UOMs are only applied when the SKU has no existing UOMs. If the SKU already has UOMs, the `uom` array in the payload is ignored.

---

### SAP Item Master Detail

**Endpoint:** `GET /api/sap/item_master/<code>/`  
**Description:** Get item master detail by SKU code.

### Example Request

```http
GET /api/sap/item_master/SKU001/
Authorization: Token abc123
```

---

## 3. SAP Inbound Order Batch Create

**Endpoint:** `POST /api/sap/inbound/`  
**Description:** Create one or more inbound (receiving) orders in a single request. Supports batch creation for ERP/SAP integration.

### Request Payload


| Field        | Type    | Required | Description                                                                                  |
| ------------ | ------- | -------- | -------------------------------------------------------------------------------------------- |
| `batch_code` | string  | No       | Upload batch identifier for tracking. Used when linked to an Excel upload batch.             |
| `submit`     | boolean | No       | If `true`, orders are auto-submitted after creation (status: DRAF → SUBM). Default: `false`. May be overridden by client's is_auto_submit or batch's is_auto_submit when batch_code is provided. |
| `orders`     | array   | **Yes**  | Array of order objects. At least one order required.                                         |


### Order Object (each item in `orders`)

| Field                    | Type   | Required | Description                                                                                                       |
| ------------------------ | ------ | -------- | ----------------------------------------------------------------------------------------------------------------- |
| `row_no`                 | string | No       | Row identifier for error reporting. Used in validation messages.                                                  |
| `organization_code`      | string | **Yes**  | Organization/warehouse code. Must exist in system.                                                                |
| `client_code`            | string | **Yes**  | Client code. Must exist under the organization.                                                                   |
| `vendor`                 | object | **Yes**  | Vendor (supplier) information.                                                                                    |
| `vendor.code`            | string | **Yes**  | Vendor code — key for external system. Auto-created if not exists. `vendor_code` at order level is also accepted. |
| `vendor.name`            | string | No       | Vendor display name.                                                                                              |
| `vendor.account`         | string | No       | Account code in external system.                                                                                  |
| `vendor.email`           | string | No       | Vendor email.                                                                                                     |
| `vendor.phone`           | string | No       | Vendor phone (e.g., `+(852)91234567`).                                                                            |
| `courier_code`           | string | No       | Courier code.                                                             |
| `courier_name`           | string | No       | Courier display name (informational).                                                                               |
| `client_reference`       | string | **Yes**  | Unique order reference from your system. Must not duplicate active orders.                                        |
| `purchase_order`         | string | No       | Purchase order number.                                                                                            |
| `estimated_arrival_date` | string | **Yes**  | Expected arrival date. Format: `DD-MM-YYYY` or `DD/MM/YYYY`.                                                      |
| `air_waybill`            | string | No       | Air waybill number.                                                                                               |
| `waybill`                | string | No       | Waybill/tracking number.                                                                                          |
| `shipping`               | object | No       | Shipping address. Uses vendor default if empty.                                                                   |
| `billing`                | object | No       | Billing address. Uses vendor default if empty.                                                                    |
| `remark`                 | string | No       | Order remark.                                                                                                     |
| `note_on_document`       | string | No       | Note to appear on documents.                                                                                      |
| `GR_number`              | string | No       | Goods Receipt number.                                                                                             |
| `add_by`                 | string | No       | Added-by user/name from source system.                                                                            |
| `sap_ref_no`             | string | No       | SAP reference number.                                                                                             |
| `source`                 | string | No       | Source system identifier.                                                                                         |
| `external_id`            | string | No       | External system ID for correlation.                                                                               |
| `dockey`                 | string | No       | SAP document key.                                                                                                 |
| `order_line`             | array  | **Yes**  | Order line items. At least one line required.                                                                     |


### Address Object (`shipping` / `billing`)


| Field          | Type   | Required | Description                                               |
| -------------- | ------ | -------- | --------------------------------------------------------- |
| `contact_name` | string | No       | Contact person name.                                      |
| `phone`        | string | No       | Phone number.                                             |
| `email`        | string | No       | Email address.                                            |
| `address`      | string | No       | Street address. Warning if empty for shipping.            |
| `district`     | string | No       | District/region.                                          |
| `city`         | string | No       | City.                                                     |
| `country`      | string | No       | Country.                                                  |
| `postal_code`  | string | No       | Postal/ZIP code.                                          |
| `address_type` | string | No       | `B` (Business), `R` (Resident), `S` (Shop). Default: `B`. |


### Order Line Object (each item in `order_line`)


| Field                    | Type          | Required | Description                                                                        |
| ------------------------ | ------------- | -------- | ---------------------------------------------------------------------------------- |
| `row_no`                 | string        | No       | Line row identifier for error reporting.                                           |
| `sku`                    | string        | **Yes**  | SKU/item code. Must exist in item master for the client.                           |
| `qty`                    | string/number | **Yes**  | Quantity. Must be a positive integer.                                              |
| `unit`                   | string        | No       | Unit of measure (e.g., EACH, CTN). Must match SKU's UOM. Default: base UOM when omitted. |
| `exp_date`               | string        | No       | Expiry date. Format: `DD-MM-YYYY`. Required if SKU has expiry control.             |
| `lot_no`                 | string        | No       | Lot number. Required if SKU has lot control.                                       |
| `serial_no`              | string        | No       | Serial number. Required if SKU has serial control.                                  |
| `batch`                  | string        | No       | Batch number.                                                                       |
| `warehouse_location_code`| string        | No       | Warehouse location code. Must exist in system if provided.                          |
| `line_remark`            | string        | No       | Line-level remark.                                                                  |

### Example Request

```json
{
  "batch_code": "BATCH-2025-001",
  "submit": false,
  "orders": [
    {
      "row_no": "1",
      "organization_code": "DEMO",
      "client_code": "DEMOCLIENT",
      "vendor": {
        "code": "VEND001",
        "name": "Vendor Name",
        "account": "ACC001",
        "email": "vendor@example.com",
        "phone": "+(852)91234567"
      },
      "courier_code": "",
      "courier_name": "",
      "client_reference": "INB-2025-001",
      "purchase_order": "PO-12345",
      "estimated_arrival_date": "15-03-2025",
      "air_waybill": "",
      "waybill": "",
      "shipping": {
        "contact_name": "John Doe",
        "phone": "91234567",
        "email": "",
        "address": "123 Warehouse St",
        "district": "",
        "city": "Hong Kong",
        "country": "HK",
        "postal_code": "",
        "address_type": "B"
      },
      "billing": {
        "contact_name": "",
        "phone": "",
        "email": "",
        "address": "",
        "district": "",
        "city": "",
        "country": "",
        "postal_code": "",
        "address_type": "B"
      },
      "remark": "",
      "note_on_document": "",
      "GR_number": "GR-000123",
      "add_by": "api_user",
      "sap_ref_no": "SAP-REF-001",
      "dockey": "",
      "order_line": [
        {
          "row_no": "1",
          "sku": "SKU001",
          "qty": "100",
          "unit": "EACH",
          "exp_date": "31-12-2025",
          "lot_no": "",
          "serial_no": "",
          "batch": "",
          "warehouse_location_code": "",
          "line_remark": ""
        }
      ]
    }
  ]
}
```

### Success Response (200)

```json
{
  "result": "Success",
  "data": { ... },
  "error": null
}
```

### Error Response (400)

```json
{
  "result": "Failed",
  "data": null,
  "error": [
    "row 1: Organization[DEMO] does not exist in system",
    "row 1: SKU[SKU999] not found in system"
  ]
}
```

### Validation Notes

- **Client Reference** must be unique per client; cannot reuse until order is FINI or CANC.
- **Vendor** is auto-created if not found (type: VEND).
- **Courier** — `courier_name` is optional and informational.
- **Estimated Arrival Date** — warning if missing; format must be DD-MM-YYYY.
- **Expiry Date** — if provided, must be DD-MM-YYYY.
- **Unit** — must match SKU's configured UOM; otherwise base UOM is used.

---

## 4. SAP Inbound Order Update

**Endpoint:** `PUT /api/sap/inbound/<dockey>/`  
**Description:** Update an inbound order when status is DRAF. Only provided fields are updated. When `order_line` is provided, the entire line set is replaced.

### Prerequisites

- Order must be in status **DRAF**.
- Order is looked up by `dockey` (SAP document key, URL path) and client from token.

### Request Payload (all fields optional)

| Field                     | Type   | Description                                                                 |
| ------------------------- | ------ | --------------------------------------------------------------------------- |
| `purchase_order`          | string | Purchase order number. Maps to Order.PO.                                    |
| `air_waybill`             | string | Air waybill number.                                                         |
| `waybill`                 | string | Waybill/tracking number.                                                     |
| `estimated_arrival_date`  | string | Expected arrival date. Format: DD-MM-YYYY.                                  |
| `courier_code`            | string | Courier code. Must exist in organization.                                   |
| `shipping`                | object | Shipping address (contact_name, phone, email, address, district, city, country, postal_code, address_type). address_type default: `B`. |
| `billing`                 | object | Billing address (same structure). address_type default: `B`.                |
| `remark`                  | string | Order remark.                                                                |
| `note_on_document`       | string | Note on document.                                                           |
| `GR_number`              | string | Goods Receipt number.                                                       |
| `add_by`                 | string | Added-by user/name from source system.                                      |
| `sap_ref_no`             | string | SAP reference number.                                                       |
| `source`                  | string | Source system identifier.                                                   |
| `order_line`              | array  | Order lines. When provided, replaces entire line set. Same structure as create. |

**Omitted fields:** Retain current value when not provided.

### Not updatable

- `dockey`, `external_id`, `client`, `relation`, `client_reference`, `type`, `status`

### Example Request

```http
PUT /api/sap/inbound/SAP_DOC_KEY_123/
Authorization: Token abc123
Content-Type: application/json
```

```json
{
  "purchase_order": "PO-99999",
  "estimated_arrival_date": "20-03-2025",
  "air_waybill": "AWB123",
  "waybill": "WB456",
  "courier_code": "DHL",
  "shipping": {
    "contact_name": "Updated Name",
    "phone": "91234567",
    "address": "456 New St",
    "city": "Hong Kong",
    "country": "HK",
    "address_type": "B"
  },
  "remark": "Updated remark",
  "GR_number": "GR-000456",
  "add_by": "sap_bot",
  "sap_ref_no": "SAP-REF-002",
  "order_line": [
    {
      "row_no": "1",
      "sku": "SKU001",
      "qty": "150",
      "unit": "EACH",
      "exp_date": "31-12-2025",
      "lot_no": "",
      "line_remark": ""
    }
  ]
}
```

### Success Response (200)

```json
{
  "data": { ... order detail_json_data ... },
  "error": null
}
```

### Error Responses

| Status | Condition |
| ------ | --------- |
| 401 | Unauthorized |
| 404 | Inbound order not found |
| 400 | Invalid JSON, order not in DRAF status, invalid courier_code, invalid date format, order line validation errors |

---

### 4.1. SAP Inbound Order Cancel

**Endpoint:** `PUT /api/sap/inbound/cancel/<dockey>/`  
**Description:** Cancels an inbound order when status is DRAF. Only draft orders can be cancelled via this endpoint.

### Prerequisites

- Order must be in status **DRAF**.
- Order is looked up by `dockey` (SAP document key, URL path) and client from token.

### Request

No request body required. Empty body or `{}` is acceptable.

### Example Request

```http
PUT /api/sap/inbound/cancel/SAP_DOC_KEY_123/
Authorization: Token abc123
Content-Type: application/json
```

### Success Response (200)

```json
{
  "data": { ... order detail_json_data ... },
  "error": null
}
```

### Error Responses

| Status | Condition |
| ------ | --------- |
| 401 | Unauthorized |
| 404 | Inbound order not found |
| 400 | dockey required, order not in DRAF status (only DRAF orders can be cancelled) |

---

## 5. SAP Outbound Order Batch Create

**Endpoint:** `POST /api/sap/outbound/`  
**Description:** Create one or more outbound (shipping) orders in a single request.

### Request Payload


| Field        | Type    | Required | Description                                               |
| ------------ | ------- | -------- | --------------------------------------------------------- |
| `batch_code` | string  | No       | Upload batch identifier.                                  |
| `submit`     | boolean | No       | Auto-submit after creation. Default: `false`. May be overridden by client's is_auto_submit or batch's is_auto_submit when batch_code is provided. |
| `orders`     | array   | **Yes**  | Array of order objects.                                   |


### Order Object


| Field                     | Type   | Required | Description                                                          |
| ------------------------- | ------ | -------- | -------------------------------------------------------------------- |
| `row_no`                  | string | No       | Row identifier for errors.                                           |
| `organization_code`       | string | **Yes**  | Organization code.                                                   |
| `client_code`             | string | **Yes**  | Client code.                                                         |
| `customer`                | object | **Yes**  | Customer (relation) information.                                     |
| `customer.code`           | string | **Yes**  | Customer code — key for external system. Auto-created if not exists. |
| `customer.name`           | string | No       | Customer display name.                                               |
| `customer.account`        | string | No       | Account code.                                                        |
| `customer.email`          | string | No       | Customer email.                                                      |
| `customer.phone`          | string | No       | Customer phone.                                                      |
| `courier_code`            | string | No       | Courier code. |
| `courier_name`            | string | No       | Courier display name (informational). Used in order remark when courier not found in system. |
| `client_reference`        | string | **Yes**  | Unique order reference.                                              |
| `sales_order`             | string | No       | Sales order number.                                                  |
| `estimated_shipping_date` | string | **Yes**  | Expected shipping date. Format: `DD-MM-YYYY`.                        |
| `air_waybill`             | string | No       | Air waybill.                                                         |
| `waybill`                 | string | No       | Waybill.                                                             |
| `shipping`                | object | No       | Shipping address.                                                    |
| `billing`                 | object | No       | Billing address.                                                     |
| `remark`                  | string | No       | Order remark.                                                        |
| `note_on_document`        | string | No       | Document note.                                                       |
| `dockey`                  | string | No       | SAP document key.                                                    |
| `order_line`              | array  | **Yes**  | Order line items.                                                    |


### Order Line Object (Outbound)


| Field                    | Type          | Required | Description                        |
| ------------------------ | ------------- | -------- | ---------------------------------- |
| `row_no`                 | string        | No       | Line row identifier.               |
| `sku`                    | string        | **Yes**  | SKU code.                          |
| `qty`                    | string/number | **Yes**  | Quantity. Positive integer.        |
| `bin_location`           | string        | No       | Preferred bin for pick (if known). |
| `warehouse_location_code`| string        | No       | Warehouse location code. Must exist in system if provided. |
| `unit`                   | string        | No       | Unit of measure. Default: base UOM when omitted. |
| `exp_date`               | string        | No       | Expiry date. Format: `DD-MM-YYYY`. |
| `lot_no`                 | string        | No       | Lot number.                        |
| `serial_no`              | string        | No       | Serial number.                     |
| `batch`                  | string        | No       | Batch number.                      |
| `line_remark`            | string        | No       | Line remark.                       |

### Example Request

```json
{
  "batch_code": "BATCH-2025-002",
  "submit": true,
  "orders": [
    {
      "row_no": "1",
      "organization_code": "DEMO",
      "client_code": "DEMOCLIENT",
      "customer": {
        "code": "CUST001",
        "name": "Customer Name",
        "account": "ACC001",
        "email": "customer@example.com",
        "phone": "+(852)91234567"
      },
      "courier_code": "DHL",
      "courier_name": "DHL Express",
      "client_reference": "OUT-2025-001",
      "sales_order": "SO-12345",
      "estimated_shipping_date": "20-03-2025",
      "shipping": {
        "contact_name": "Jane Doe",
        "phone": "91234567",
        "address": "456 Delivery Ave",
        "city": "Hong Kong",
        "country": "HK",
        "address_type": "B"
      },
      "billing": {},
      "order_line": [
        {
          "row_no": "1",
          "sku": "SKU001",
          "qty": "50",
          "bin_location": "",
          "warehouse_location_code": "",
          "unit": "EACH",
          "exp_date": "",
          "lot_no": "",
          "serial_no": "",
          "batch": "",
          "line_remark": ""
        }
      ]
    }
  ]
}
```

### Success / Error Response

Same structure as SAP Inbound. See [Error Handling](#15-error-handling).

### Validation Notes (Outbound)

- **Warehouse Location Code** — if provided, must exist in the organization. When `bin_location` is provided, it takes precedence for `warehouse_location_id`; otherwise `warehouse_location_code` is used.

---

## 6. SAP Outbound Order Update

**Endpoint:** `PUT /api/sap/outbound/<dockey>/`  
**Description:** Update an outbound order when status is DRAF. Only provided fields are updated. When `order_line` is provided, the entire line set is replaced.

### Prerequisites

- Order must be in status **DRAF**.
- Order is looked up by `dockey` (SAP document key, URL path) and client from token.

### Request Payload (all fields optional)

| Field                      | Type   | Description                                                                 |
| -------------------------- | ------ | --------------------------------------------------------------------------- |
| `sales_order`              | string | Sales order number. Maps to Order.PO.                                       |
| `air_waybill`              | string | Air waybill number.                                                         |
| `waybill`                  | string | Waybill/tracking number.                                                     |
| `estimated_shipping_date`  | string | Expected shipping date. Format: DD-MM-YYYY.                                  |
| `courier_code`             | string | Courier code. Must exist in organization.                                    |
| `shipping`                 | object | Shipping address (contact_name, phone, email, address, district, city, country, postal_code, address_type). address_type default: `B`. |
| `billing`                  | object | Billing address (same structure). address_type default: `B`.                 |
| `remark`                   | string | Order remark.                                                                |
| `note_on_document`        | string | Note on document.                                                           |
| `source`                   | string | Source system identifier.                                                    |
| `order_line`               | array  | Order lines. When provided, replaces entire line set. Same structure as create. Unit is required per line. |

**Omitted fields:** Retain current value when not provided.

### Not updatable

- `dockey`, `external_id`, `client`, `relation`, `client_reference`, `type`, `status`

### Example Request

```http
PUT /api/sap/outbound/SAP_DOC_KEY_456/
Authorization: Token abc123
Content-Type: application/json
```

```json
{
  "sales_order": "SO-99999",
  "estimated_shipping_date": "25-03-2025",
  "air_waybill": "AWB456",
  "courier_code": "DHL",
  "remark": "Updated remark",
  "order_line": [
    {
      "row_no": "1",
      "sku": "SKU001",
      "qty": "75",
      "unit": "EACH",
      "bin_location": "",
      "warehouse_location_code": "",
      "line_remark": ""
    }
  ]
}
```

### Success Response (200)

```json
{
  "data": { ... order detail_json_data ... },
  "error": null
}
```

### Error Responses

| Status | Condition |
| ------ | --------- |
| 401 | Unauthorized |
| 404 | Outbound order not found |
| 400 | Invalid JSON, order not in DRAF status, invalid courier_code, invalid date format, order line validation errors |

---

### 6.1. SAP Outbound Order Cancel

**Endpoint:** `PUT /api/sap/outbound/cancel/<dockey>/`  
**Description:** Cancels an outbound order when status is DRAF. Only draft orders can be cancelled via this endpoint.

### Prerequisites

- Order must be in status **DRAF**.
- Order is looked up by `dockey` (SAP document key, URL path) and client from token.

### Request

No request body required. Empty body or `{}` is acceptable.

### Example Request

```http
PUT /api/sap/outbound/cancel/SAP_DOC_KEY_456/
Authorization: Token abc123
Content-Type: application/json
```

### Success Response (200)

```json
{
  "data": { ... order detail_json_data ... },
  "error": null
}
```

### Error Responses

| Status | Condition |
| ------ | --------- |
| 401 | Unauthorized |
| 404 | Outbound order not found |
| 400 | dockey required, order not in DRAF status (only DRAF orders can be cancelled) |

---

## 7. SAP Outbound Order Finish

**Endpoint:** `PUT /api/sap/outbound/finish/`  
**Description:** Allows SAP to update the `sap_finish_datetime` when it completes processing an outbound order. This records when SAP signals order completion.

### Authentication

Same as other SAP endpoints. Requires Token in `Authorization` header (User Token or Client Token).

### Request Body

| Field                 | Type   | Required | Description                                                                 |
| --------------------- | ------ | -------- | --------------------------------------------------------------------------- |
| `dockey`              | string | No*      | SAP document key. Primary lookup key for the order.                         |
| `client_reference`    | string | No*      | Client reference. Fallback lookup key if `dockey` is not provided.           |
| `sap_finish_datetime` | string | **Yes**  | Datetime when SAP finished processing. ISO 8601 format (e.g. `2025-03-04T14:30:00+08:00`). |

\* Either `dockey` or `client_reference` must be provided.

### Example Request

```json
{
  "dockey": "SAP_DOC_KEY_123",
  "sap_finish_datetime": "2025-03-04T14:30:00+08:00"
}
```

Or using client_reference:

```json
{
  "client_reference": "OUT-2025-001",
  "sap_finish_datetime": "2025-03-04T14:30:00+08:00"
}
```

### Success Response (200)

```json
{
  "data": {
    "client_reference": "OUT-2025-001",
    "dockey": "SAP_DOC_KEY_123",
    "sap_finish_datetime": "2025-03-04T14:30:00+08:00"
  },
  "error": null
}
```

### Error Responses

| Status | Condition |
| ------ | --------- |
| 401    | Unauthorized (invalid or missing token) |
| 400    | Invalid JSON, missing required fields, or invalid datetime format |
| 404    | Outbound order not found for the given dockey/client_reference |

---

## 8. Receiving Excel Fulfill (v2)

**Endpoint:** `POST /api/receiving/v2/fulfill`  
**Description:** Batch fulfill (put-away) receiving transactions. Allocates inventory to bin locations for inbound orders.

### Prerequisites

- Requires a valid `batch_code` from an upload batch (Excel or API batch).
- Transaction must be in status PROC (Processing) or ARRI (Arrived).
- Transaction type must be INBO (Inbound).

### Request Payload


| Field          | Type   | Required | Description                                                          |
| -------------- | ------ | -------- | -------------------------------------------------------------------- |
| `batch_code`   | string | **Yes**  | Upload batch identifier. Must be valid and associated with a client. |
| `transactions` | array  | **Yes**  | Array of transaction fulfill objects.                                |


### Transaction Object


| Field            | Type    | Required | Description                                                   |
| ---------------- | ------- | -------- | ------------------------------------------------------------- |
| `transaction_id` | integer | **Yes**  | WMS transaction ID. Must be INBO type and not DRAF/FINI/CANC. |


### Line Item Object (each item in `line_item`)


| Field           | Type   | Required | Description                                                            |
| --------------- | ------ | -------- | ---------------------------------------------------------------------- |
| `row_no`        | string | No       | Row identifier for errors.                                             |
| `sku`           | string | **Yes**  | SKU code. Must exist and be active for the batch client.               |
| `qty`           | number | **Yes**  | Quantity to put away. Must be ≥ 0.                                     |
| `alt_qty`       | number | No       | Alternative quantity (e.g., in base UOM).                              |
| `bin_location`  | string | **Yes**  | Bin location code. Must exist and be active. Empty if staging.         |
| `expiry_date`   | string | No       | Expiry date. Format: `DD-MM-YYYY`. Required if SKU has expiry control. |
| `lot_number`    | string | No       | Lot number. Required if SKU has lot control.                           |
| `serial_number` | string | No       | Serial number. Required if SKU has serial control.                     |
| `batch`         | string | No       | Batch number.                                                          |
| `line_remark`   | string | No       | Line remark.                                                           |


### Example Request

```json
{
  "batch_code": "BATCH-2025-001",
  "transactions": [
    {
      "transaction_id": 1234,
      "line_item": [
        {
          "row_no": "1",
          "sku": "SKU001",
          "qty": 100,
          "alt_qty": 100,
          "bin_location": "FL-4F-1-1B",
          "expiry_date": "31-12-2025",
          "lot_number": "LOT001",
          "serial_number": "",
          "batch": "",
          "line_remark": ""
        }
      ]
    }
  ]
}
```

### Success Response (200)

```json
{
  "result": "Success",
  "data": null,
  "error": null
}
```

### Error Response (400)

```json
{
  "result": "Failed",
  "data": null,
  "error": [
    "Transaction[id=1234] not found in system",
    "row:1: Item Master[code=SKU999] not found in system",
    "row:1: Bin Location[code=INVALID] not found in system"
  ]
}
```

---

## 9. Shipping Excel Fulfill (v2)

**Endpoint:** `POST /api/shipping/v2/fulfill`  
**Description:** Batch fulfill (pick) shipping transactions for outbound orders.

### Prerequisites

- Valid `batch_code`.
- Transaction must be OUBO (Outbound) type.
- Transaction status must allow modification (not DRAF/FINI/CANC).

### Request Payload

Same structure as [Receiving Excel Fulfill](#8-receiving-excel-fulfill-v2), except:

- `transaction_id` must reference an **OUBO** (outbound) transaction.
- `bin_location` is the bin from which items are picked.

### Example Request

```json
{
  "batch_code": "BATCH-2025-002",
  "transactions": [
    {
      "transaction_id": 5678,
      "line_item": [
        {
          "row_no": "1",
          "sku": "SKU001",
          "qty": 50,
          "bin_location": "FL-4F-1-1B",
          "expiry_date": "31-12-2025",
          "lot_number": "LOT001",
          "serial_number": "",
          "batch": "",
          "line_remark": ""
        }
      ]
    }
  ]
}
```

### Success / Error Response

Same structure as Receiving Excel Fulfill.

---

## 10. Transaction Adjustment

**Endpoint:** `POST /api/transaction/adjustment`  
**Description:** Create inventory adjustment transactions (e.g., stock count corrections, write-offs).

### Request Payload


| Field          | Type   | Required | Description                              |
| -------------- | ------ | -------- | ---------------------------------------- |
| `batch_code`   | string | **Yes**  | Upload batch identifier. Must be valid.  |
| `transactions` | array  | **Yes**  | Array of adjustment transaction objects. |


### Transaction Object


| Field               | Type   | Required | Description                           |
| ------------------- | ------ | -------- | ------------------------------------- |
| `client_code`       | string | **Yes**  | Client code. Must match batch client. |
| `organization_code` | string | **Yes**  | Organization code.                    |
| `client_reference`  | string | No       | Reference for the adjustment.         |
| `remark`            | string | No       | Transaction remark.                   |
| `line_item`         | array  | **Yes**  | Adjustment line items.                |


### Line Item Object (Adjustment)


| Field               | Type    | Required | Description                                                        |
| ------------------- | ------- | -------- | ------------------------------------------------------------------ |
| `row_no`            | string  | No       | Row identifier.                                                    |
| `sku`               | string  | **Yes**  | SKU code. Must exist and be active.                                |
| `qty`               | integer | **Yes**  | Quantity. **Positive** = increase, **Negative** = decrease.        |
| `from_bin_location` | string  | **Yes**  | Source bin. Must exist.                                            |
| `to_bin_location`   | string  | **Yes**  | Destination bin (can be same for pure qty adjustment). Must exist. |
| `expiry_date`       | string  | No       | Format: `DD-MM-YYYY`.                                              |
| `lot_number`        | string  | No       | Lot number.                                                        |
| `serial_number`     | string  | No       | Serial number.                                                     |
| `batch`             | string  | No       | Batch number.                                                      |
| `line_remark`       | string  | No       | Line remark.                                                       |


### Example Request

```json
{
  "batch_code": "BATCH-2025-002",
  "transactions": [
    {
      "client_code": "DEMOCLIENT",
      "organization_code": "DEMO",
      "client_reference": "ADJ-2025-001",
      "remark": "Stock count correction",
      "line_item": [
        {
          "row_no": "1",
          "sku": "SKU001",
          "qty": -5,
          "from_bin_location": "FL-4F-1-1B",
          "to_bin_location": "FL-4F-1-1B",
          "expiry_date": "31-12-2025",
          "lot_number": "LOT001",
          "serial_number": "",
          "batch": "",
          "line_remark": ""
        }
      ]
    }
  ]
}
```

---

## 9. Bin Transfer

**Endpoint:** `POST /api/transaction/bin_transfer`  
**Description:** Create bin transfer transactions (move inventory between locations).

### Request Payload

Same structure as [Transaction Adjustment](#10-transaction-adjustment), except:

- `qty` must be **positive** (quantity to move).
- `from_bin_location` — source bin.
- `to_bin_location` — destination bin. Must be different from source for actual transfer.

### Example Request

```json
{
  "batch_code": "BATCH-2025-002",
  "transactions": [
    {
      "client_code": "DEMOCLIENT",
      "organization_code": "DEMO",
      "client_reference": "BT-2025-001",
      "remark": "Relocation",
      "line_item": [
        {
          "row_no": "1",
          "sku": "SKU001",
          "qty": 100,
          "from_bin_location": "FL-4F-1-1B",
          "to_bin_location": "FL-4F-2-1A",
          "expiry_date": "31-12-2025",
          "lot_number": "LOT001",
          "serial_number": "",
          "batch": "",
          "line_remark": ""
        }
      ]
    }
  ]
}
```

---

## 12. Stock Check

**Endpoint:** `GET /api/stock/check`  
**Description:** Get stock details for a specific SKU.

### Query Parameters


| Parameter        | Type    | Required | Description                                       |
| ---------------- | ------- | -------- | ------------------------------------------------- |
| `client_id`      | integer | No       | Client ID. Uses user's organization when omitted. |
| `inventory_code` | string  | **Yes**  | SKU/item code.                                    |


### Example Request

```http
GET /api/stock/check?inventory_code=SKU001
Authorization: Token abc123
```

### Success Response (200)

```json
{
  "result": "Success",
  "data": {
    "total_qty": 1000,
    "available_qty": 800,
    "reserved_qty": 150,
    "received_qty": 50,
    "packed_qty": 0,
    "onhold_qty": 0,
    "bin_details": [ ... ]
  },
  "error": null
}
```

### Error Response (404)

```json
{
  "data": null,
  "error": "inventory not found"
}
```

---

## 13. Inventory Summary

**Endpoint:** `GET /api/inventory/summary`  
**Description:** Get full stock summary for all SKUs of the authenticated client.

### Authentication

- **User Token** — Returns stock for user's organization (staff) or selected client.
- **Client Token** — Returns stock for the token's client only.

### Example Request

```http
GET /api/inventory/summary
Authorization: Token abc123
```

### Success Response (200)

```json
{
  "result": "Success",
  "data": {
    "SKU001": {
      "total_qty": 1000,
      "available_qty": 800,
      "reserved_qty": 150,
      "received_qty": 50,
      "packed_qty": 0,
      "onhold_qty": 0
    },
    "SKU002": { ... }
  },
  "error": null
}
```

---

## 12. Reference Data

### Address Type


| Code | Description |
| ---- | ----------- |
| `B`  | Business    |
| `R`  | Resident    |
| `S`  | Shop        |


### Order Types


| Code | Name            |
| ---- | --------------- |
| INBO | Inbound         |
| OUBO | Outbound        |
| ADJU | Adjustment      |
| BNTR | Bin Transfer    |
| CURE | Customer Return |
| VERE | Vendor Return   |


### Order Status


| Code | Name          |
| ---- | ------------- |
| DRAF | Draft         |
| SUBM | Submitted     |
| ARRI | Arrived       |
| PROC | Processing    |
| ALLO | Allocated     |
| R2SH | Ready to Ship |
| FINI | Finished      |
| CANC | Cancelled     |


### Date Format

All date fields use: `**DD-MM-YYYY**` (e.g., `15-03-2025`). Slash variant `DD/MM/YYYY` is also accepted where applicable.

---

## 15. Error Handling

### Standard Error Response

```json
{
  "result": "Failed",
  "data": null,
  "error": "Error message or array of validation messages"
}
```

### HTTP Status Codes


| Code | Meaning                                                                                                      |
| ---- | ------------------------------------------------------------------------------------------------------------ |
| 200  | Success                                                                                                      |
| 400  | Bad Request — validation error, invalid payload                                                              |
| 401  | Unauthorized — missing or invalid token                                                                      |
| 403  | Forbidden — insufficient permissions                                                                         |
| 404  | Not Found — resource not found                                                                               |
| 418  | I'm a teapot — used for "use Order Create API instead" (e.g., when `order_line` is missing in list endpoint) |


### Validation Error Format

For batch endpoints, `error` may be an array of strings:

```json
{
  "result": "Failed",
  "data": null,
  "error": [
    "row 1: Organization[XYZ] does not exist in system",
    "row 1: SKU[SKU999] not found in system",
    "row 2: Client Reference[INB-001] already exist in system"
  ]
}
```

### Integration Tips

1. **Idempotency** — Use unique `client_reference` for orders. Duplicate references are rejected.
2. **Batch Code** — For batch operations, obtain a valid `batch_code` from your WMS admin or upload workflow.
3. **Vendor/Customer** — Relations are auto-created when not found. Provide `code` as the primary key.
4. **Units** — SKUs have configured UOMs. Use `unit` that matches the SKU, or omit to use base UOM.
5. **SKU Controls** — Ensure `exp_date`, `lot_no`, `serial_no` are provided when the SKU has corresponding controls (expiry, lot, serial).
6. **Schema Discovery** — Use `GET /api/sap/inbound/` or `GET /api/sap/outbound/` to retrieve example payload schemas.

---

## API Documentation Endpoint

**Endpoint:** `GET /api/doc`  
**Description:** Returns the public API documentation page (HTML).

---

*Document version: 1.0. Last updated: March 2025.*

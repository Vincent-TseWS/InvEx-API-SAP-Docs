---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - python
  - javascript

toc_footers:
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true

code_clipboard: true

meta:
  - name: AS WMS API Document
    content: Documentation for WMS API
---

# Introduction

Welcome to the WMS API! You can use our API to access WMS API endpoints, which can get information on warehouse master, item master and orders in our database.

If anything is missing or seems incorrect, please contact your account manager.

# Authentication

> To authorize, use this code:

```python
import requests

url = "https://wms-core.ark.space/api/ENDPOINT"

payload = ""
headers = {
  'Authorization': 'Token :YOUR_AUTH_TOKEN'
}

response = requests.request("GET", url, headers=headers, data=payload)

print(response.text)

```

```javascript
var settings = {
  "url": "https://wms-core.arkspace.com.hk/api/ENDPOINT",
  "method": "GET",
  "timeout": 0,
  "headers": {
    "Authorization": "Token :YOUR_AUTH_TOKEN"
  },
};

$.ajax(settings).done(function (response) {
  console.log(response);
});
```

> Make sure to replace `:YOUR_AUTH_TOKEN` with your API key.

The WMS uses API Tokens to allow access to the API. You can request for a new API Token by contacting our web admin.
This system expects for the API token to be included in all API requests to the server in a header that looks like the following:

`Authorization: Token :YOUR_AUTH_TOKEN`

<aside class="notice">
You must replace <code> :YOUR_AUTH_TOKEN </code> with your Client API key.
</aside>

# Item Master

## List Item Master

```python
import requests

url = "https://wms-core.ark.space/api/sap/item_master/"

payload = {}
headers = {
  'Authorization': 'Token :YOUR_AUTH_TOKEN'
}

response = requests.request("GET", url, headers=headers, data=payload)

print(response.text)
```

```javascript
var settings = {
  "url": "https://wms-core.ark.space/api/sap/item_master",
  "method": "GET",
  "timeout": 0,
  "headers": {
    "Authorization": "Token :YOUR_AUTH_TOKEN"
  },
};

$.ajax(settings).done(function (response) {
  console.log(response);
});
```

> The above command returns JSON structured like this:

```json
{
    "result": "Success",
    "data": [
        {
            "code": "DEMO_SKU_001",
            "item_type": "Item",
            "product_name": "DEMO_SKU_001",
            "description": "This is Demo SKU 001 in DemoClient2",
            "upc": "123",
            "expiry_date_control": "DISABLE",
            "lot_control": "DISABLE",
            "serial_number_control": "DISABLE",
            "remark": "",
            "required_condition": "AMB",
            "is_condition_force": true,
            "pick_method": "FIFO",
            "hs_code": "",
            "custom_code": "",
            "pick_size": 1,
            "pack_size": 1,
            "uom": [
                {
                    "unit": "EACH",
                    "unit_name": "Each",
                    "qty": "1.0000",
                    "is_base": true,
                    "length": "1.0000",
                    "width": "1.0000",
                    "height": "1.0000",
                    "gross_weight": "1.0000",
                    "net_weight": "1.0000",
                    "alt_qty": "1.0000",
                    "is_alt_base": false
                }
            ],
            "custom_field": []
        },
	{
            "code": "DEMO_SKU_001",
            "item_type": "Item",
            "product_name": "DEMO_SKU_001",
            "description": "This is Demo SKU 001 in DemoClient2",
            "upc": "123",
            "expiry_date_control": "DISABLE",
            "lot_control": "DISABLE",
            "serial_number_control": "DISABLE",
            "remark": "",
            "required_condition": "AMB",
            "is_condition_force": true,
            "pick_method": "FIFO",
            "hs_code": "",
            "custom_code": "",
            "pick_size": 1,
            "pack_size": 1,
            "uom": [
                {
                    "unit": "EACH",
                    "unit_name": "Each",
                    "qty": "1.0000",
                    "is_base": true,
                    "length": "1.0000",
                    "width": "1.0000",
                    "height": "1.0000",
                    "gross_weight": "1.0000",
                    "net_weight": "1.0000",
                    "alt_qty": "1.0000",
                    "is_alt_base": false
                }
            ],
            "custom_field": []
        }
    ],
    "error": null
}
```

This endpoint list out item master.

### HTTP Request

`GET https://wms-core.ark.space/api/sap/item_master/`

## Retrieve Item Master Detail

```python
import requests

url = "https://wms-core.ark.space/api/sap/item_master/:ITEM_CODE/"

payload = {}
headers = {
  'Authorization': 'Token :YOUR_AUTH_TOKEN'
}

response = requests.request("GET", url, headers=headers, data=payload)

print(response.text)
```

```javascript
var settings = {
  "url": "https://wms-core.ark.space/api/sap/item_master/:ITEM_CODE/",
  "method": "GET",
  "timeout": 0,
  "headers": {
    "Authorization": "Token :YOUR_AUTH_TOKEN"
  },
};

$.ajax(settings).done(function (response) {
  console.log(response);
});
```

> The above command returns JSON structured like this:

```json
{
    "data": {
        "code": "DEMO_SKU_001",
        "item_type": {
            "organization": "DEMO",
            "code": "ITEM",
            "name": "Item"
        },
        "product_name": "DEMO_SKU_001",
        "description": "This is Demo SKU 001 in DemoClient2",
        "upc": "123",
        "auom_control": "DISABLE",
        "expiry_date_control": "DISABLE",
        "lot_control": "DISABLE",
        "serial_number_control": "DISABLE",
        "uom": [
            {
                "unit": "EACH",
                "unit_name": "Each",
                "qty": "1.0000",
                "is_base": true,
                "length": "1.0000",
                "width": "1.0000",
                "height": "1.0000",
                "gross_weight": "1.0000",
                "net_weight": "1.0000",
                "alt_qty": "1.0000",
                "is_alt_base": false
            }
        ],
        "custom_field": []
    },
    "error": null
}
```

This endpoint list the detail of specific Item Master.

### HTTP Request

`GET https://wms-core.ark.space/api/sap/item_master/:ITEM_CODE/`

<aside class="notice">
You must replace <code> :ITEM_MASTER_CODE </code> with the item code.
</aside>

## Create a New Item Master

```python
import requests
import json

url = "https://wms-core.arkspace.com.hk/api/sap/item_master"

payload = "{
	"client_code": "DEMO_CLIENT",
        "code": "DEMO_SKU_001",
        "item_type": "Item",
        "product_name": "DEMO_SKU_001",
        "description": "This is Demo SKU 001 in DemoClient2",
        "upc": "123",
        "expiry_date_control": "DISABLE",
        "lot_control": "DISABLE",
        "serial_number_control": "DISABLE",
        "uom": [
            {
                "unit": "EACH",
                "qty": "1.0000",
                "is_base": true,
                "length": "1.0000",
                "width": "1.0000",
                "height": "1.0000",
                "gross_weight": "1.0000",
                "net_weight": "1.0000"
            }
        ],
        "custom_field": [
            {
                "name": "Customer Field name",
                "value": "Customer field value"
            }
	]
    }"
headers = {
  'Authorization': 'Token :YOUR_AUTH_TOKEN',
  'Content-Type': 'application/json'
}

response = requests.request("POST", url, headers=headers, data=payload)

print(response.text)

```

```javascript
var settings = {
  "url": "https://wms-core.arkspace.com.hk/api/sap/item_master",
  "method": "POST",
  "timeout": 0,
  "headers": {
    "Authorization": "Token YOUR_AUTH_TOKEN",
    "Content-Type": "application/json"
  },
  "data": "{
	"client_code": "DEMO_CLIENT",
        "code": "DEMO_SKU_001",
        "item_type": "Item",
        "product_name": "DEMO_SKU_001",
        "description": "This is Demo SKU 001 in DemoClient2",
        "upc": "123",
        "expiry_date_control": "DISABLE",
        "lot_control": "DISABLE",
        "serial_number_control": "DISABLE",
        "uom": [
            {
                "unit": "EACH",
                "qty": "1.0000",
                "is_base": true,
                "length": "1.0000",
                "width": "1.0000",
                "height": "1.0000",
                "gross_weight": "1.0000",
                "net_weight": "1.0000"
            }
        ],
        "custom_field": [
            {
                "name": "Customer Field name",
                "value": "Customer field value"
            }
	]
    }",
};

$.ajax(settings).done(function (response) {
  console.log(response);
});
```

> The above command returns JSON structured like this:

```json
{
    "result": "Success",
    "data": None,
    "error": None,
}
```

This endpoint create new inbound orders.

### HTTP Request

`POST https://wms-core.arkspace.com.hk/api/sap/item_master`

### Data Parameters

Parameter | Required | Description
--------- | ------- | -----------
client_code | Yes | Code referencing which Client the Order belongs to.
item_type | Yes | Item type code
product_name | Yes | product name
description | No | description
upc | No | UPC code
expiry_date_control | No | BOTH|DISABLE, Default: DISABLE
lot_control | No | BOTH|DISABLE, Default: DISABLE
serial_number_control | No | BOTH|DISABLE, Default: DISABLE
uom | Yes | Unit of measurement, in list
uom.unit | Yes | Unit code
uom.qty | Yes | # of base unit
uom.is_base | Yes | TRUE|FALSE, to indicate if this uom is base uom or not
uom.length | No | length of uom in cm, default: 1
uom.width | No | width of uom in cm, default: 1
uom.height | No | height of uom in cm, default: 1
uom.gross_weight | No | Gross Weight of uom in kg, default: 0
uom.net_weight | No | Net Weight of uom in kg, default: 0
custom_field | No | Custom Fields, in list
custom_field.name | Yes | Name of Custom field, must be unique in item master
custom_field.value | Yes | Value of Custom field

<aside class="success">
Remember — item master are managed under item code, duplication of item code is not valid
</aside>

## Update an Item Master

```python
import requests
import json

url = "https://wms-core.arkspace.com.hk/api/sap/item_master/:ITEM_CODE/"

payload = "{
        "product_name": "DEMO_SKU_001",
        "description": "This is Demo SKU 001 in DemoClient2",
        "upc": "123",
        "uom": [
            {
                "unit": "EACH",
                "qty": "1.0000",
                "is_base": true,
                "length": "1.0000",
                "width": "1.0000",
                "height": "1.0000",
                "gross_weight": "1.0000",
                "net_weight": "1.0000"
            }
        ],
        "custom_field": [
            {
                "name": "Customer Field name",
                "value": "Customer field value"
            }
	]
    }"
headers = {
  'Authorization': 'Token :YOUR_AUTH_TOKEN',
  'Content-Type': 'application/json'
}

response = requests.request("PUT", url, headers=headers, data=payload)

print(response.text)

```

```javascript
var settings = {
  "url": "https://wms-core.arkspace.com.hk/api/sap/item_master/:ITEM_CODE/",
  "method": "PUT",
  "timeout": 0,
  "headers": {
    "Authorization": "Token YOUR_AUTH_TOKEN",
    "Content-Type": "application/json"
  },
  "data": "{
        "product_name": "DEMO_SKU_001",
        "description": "This is Demo SKU 001 in DemoClient2",
        "upc": "123",
        "uom": [
            {
                "unit": "EACH",
                "qty": "1.0000",
                "is_base": true,
                "length": "1.0000",
                "width": "1.0000",
                "height": "1.0000",
                "gross_weight": "1.0000",
                "net_weight": "1.0000"
            }
        ],
        "custom_field": [
            {
                "name": "Customer Field name",
                "value": "Customer field value"
            }
	]
    }",
};

$.ajax(settings).done(function (response) {
  console.log(response);
});
```

> The above command returns JSON structured like this:

```json
{
    "result": "Success",
    "data": None,
    "error": None,
}
```

This endpoint create new inbound orders.

### HTTP Request

`PUT https://wms-core.arkspace.com.hk/api/order/item_master/:ITEM_CODE/`

### Data Parameters

Parameter | Required | Description
--------- | ------- | -----------
product_name | Yes | product name
description | No | description
upc | No | UPC code
uom | Yes | Unit of measurement, in list. In the update endpoint, it only allows creating new UOM.
uom.unit | Yes | Unit code
uom.qty | Yes | # of base unit
uom.is_base | Yes | TRUE|FALSE, to indicate if this uom is base uom or not
uom.length | No | length of uom in cm, default: 1
uom.width | No | width of uom in cm, default: 1
uom.height | No | height of uom in cm, default: 1
uom.gross_weight | No | Gross Weight of uom in kg, default: 0
uom.net_weight | No | Net Weight of uom in kg, default: 0
custom_field | No | Custom Fields, in list
custom_field.name | Yes | Name of Custom field, must be unique in item master
custom_field.value | Yes | Value of Custom field

<aside class="success">
Remember - most of the field in item master are not allowed to modify after creating.
</aside>

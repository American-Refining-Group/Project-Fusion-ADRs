# Upload Status Response Object Documentation

## Overview
This response object represents the result of a batch invoice upload operation, providing comprehensive status information, validation results, and error tracking for each uploaded invoice.

## Structure

### Root Array
The response is a tuple containing:
1. **Type identifier** (string): `"upload-status"`
2. **Status payload** (object): Contains all upload processing details

---

## Status Payload Properties

### `id` (string)
Unique identifier for the upload batch operation.
- Format: `U-{timestamp}-{uuid}`
- Example: `"U-1761481107580-6b830793-371b-4fc3-b94d-bbd40b24d5fa"`

### `summary` (object)
Aggregated statistics for the entire upload batch.

| Field | Type | Description |
|-------|------|-------------|
| `totalAmount` | string | Formatted sum of all invoice amounts (includes currency symbol) |
| `countE` | number | Count of items with Error status |
| `countW` | number | Count of items with Warning status |
| `countS` | number | Count of items with Success status |
| `totalUploads` | number | Total number of invoices uploaded |

### `items` (array)
Detailed results for each uploaded invoice.

#### Item Object Properties

| Field | Type | Description |
|-------|------|-------------|
| `invoiceNo` | string | Invoice identifier |
| `invoiceAmount` | number | Invoice monetary value (numeric, no formatting) |
| `status` | string | Processing status: `"S"` (Success), `"E"` (Error), `"W"` (Warning) |
| `headerValidation` | object | Validation results for invoice header fields |
| `detailValidation` | object | Validation results for invoice line items/details |
| `dbError` | string \| null | Database-level error message, if any |

#### Validation Object Structure
Both `headerValidation` and `detailValidation` contain:
- `errors` (string[]): Array of error messages preventing successful processing
- `warnings` (string[]): Array of non-blocking warning messages

### `unprocessedItems` (array)
Items that could not be processed, with error details.

#### Unprocessed Item Object

| Field | Type | Description |
|-------|------|-------------|
| `vendorNo` | string | Vendor identifier associated with the failed item |
| `invoiceNo` | string | Invoice identifier |
| `error` | object | Structured error information |

#### Error Object

| Field | Type | Description |
|-------|------|-------------|
| `code` | string | Machine-readable error code (e.g., `"VENDOR_NOT_FOUND"`) |
| `message` | string | Human-readable error description |
| `field` | string | Field name that caused the error |

### `status` (string)
Overall processing status of the upload batch.
- Possible values: `"Completed"`, `"Processing"`, `"Failed"` (implementation-specific)

---

## Status Code Meanings

- **`S` (Success)**: Invoice passed all validations and was processed successfully
- **`E` (Error)**: Invoice failed critical validations and was not processed
- **`W` (Warning)**: Invoice has non-critical issues but was processed

---

## Key Observations

1. **Duplicate Handling**: The same `invoiceNo` can appear multiple times with different statuses (e.g., `"RAJ-FLEXI-002"` appears with both `S` and `E` statuses)
2. **Validation Separation**: Header and detail validations are tracked independently
3. **Error Redundancy**: Failed items appear in both `items` (with status `E`) and `unprocessedItems` arrays
4. **Amount Formatting**: `summary.totalAmount` is formatted with currency symbol, while `items[].invoiceAmount` is numeric only

---

## Concrete Example

```json
[
    "upload-status",
    {
        "id": "U-1761481107580-6b830793-371b-4fc3-b94d-bbd40b24d5fa",
        "summary": {
            "totalAmount": "$800.00",
            "countE": 2,
            "countW": 0,
            "countS": 2,
            "totalUploads": 4
        },
        "items": [
            {
                "invoiceNo": "RAJ-FLEXI-001",
                "invoiceAmount": 100,
                "status": "S",
                "headerValidation": {
                    "errors": [],
                    "warnings": []
                },
                "detailValidation": {
                    "errors": [],
                    "warnings": []
                },
                "dbError": null
            },
            {
                "invoiceNo": "RAJ-FLEXI-002",
                "invoiceAmount": 200,
                "status": "S",
                "headerValidation": {
                    "errors": [],
                    "warnings": []
                },
                "detailValidation": {
                    "errors": [],
                    "warnings": []
                },
                "dbError": null
            },
            {
                "invoiceNo": "RAJ-FLEXI-002",
                "invoiceAmount": 200,
                "status": "E",
                "headerValidation": {
                    "errors": [
                        "Vendor with vendorNo: 0 and companyNo: 10 not found.",
                        "Vendor 0 not found for company 10"
                    ],
                    "warnings": [
                        "discountDueDate: Missed Discount: Voucher entered on or after discount due date"
                    ]
                },
                "detailValidation": {
                    "errors": [],
                    "warnings": []
                },
                "dbError": null
            },
            {
                "invoiceNo": "RAJ-FLEXI-003",
                "invoiceAmount": 300,
                "status": "E",
                "headerValidation": {
                    "errors": [
                        "Vendor with vendorNo: 0 and companyNo: 10 not found.",
                        "Vendor 0 not found for company 10"
                    ],
                    "warnings": [
                        "discountDueDate: Missed Discount: Voucher entered on or after discount due date"
                    ]
                },
                "detailValidation": {
                    "errors": [],
                    "warnings": []
                },
                "dbError": null
            }
        ],
        "unprocessedItems": [
            {
                "vendorNo": "0",
                "invoiceNo": "RAJ-FLEXI-002",
                "error": {
                    "code": "VENDOR_NOT_FOUND",
                    "message": "WE DIDN'T FIND ANY VENDOR WITH 0 AND INVOICE NO RAJ-FLEXI-002",
                    "field": "vendorNo"
                }
            },
            {
                "vendorNo": "0",
                "invoiceNo": "RAJ-FLEXI-003",
                "error": {
                    "code": "VENDOR_NOT_FOUND",
                    "message": "WE DIDN'T FIND ANY VENDOR WITH 0 AND INVOICE NO RAJ-FLEXI-003",
                    "field": "vendorNo"
                }
            }
        ],
        "status": "Completed"
    }
]
```

**Title:** Long-Running Operations (LRO) Pattern with HTTP/WebSocket/Queue Architecture  
**Date:** 2025-10-26  
**Status:** Accepted

## Context

The application requires a pattern for handling long-running operations (LROs) that exceed typical HTTP timeout limits. These operations include file uploads, batch validations, data migrations, complex calculations, and other processing tasks that take more than a few seconds to complete.

**Key Requirements:**
- Execute operations that may take minutes to complete
- Provide immediate acknowledgment to users when operations start
- Deliver real-time progress updates during execution
- Support operations across multiple modules (file uploads, validations, data processing, etc.)
- Maintain security through proper authentication
- Enable horizontal scaling for increased load

**Constraints:**
- HTTP request timeout limitations (typically 30-60 seconds)
- Need for immediate user feedback while processing continues
- Browser connection limitations for long-polling approaches
- Resource cleanup requirements for failed operations
- User expectation of real-time progress visibility

**Options Considered:**
1. **Synchronous HTTP-only processing** - Simple but causes timeouts on long operations
2. **HTTP with polling** - Requires constant client requests, inefficient and increases server load
3. **Server-Sent Events (SSE)** - One-way communication, limited browser support, connection management complexity
4. **Hybrid HTTP/WebSocket with queue-based processing** - Balances immediate feedback with async processing and bidirectional communication

## Decision

We will implement a **Long-Running Operations (LRO) pattern using a hybrid HTTP/WebSocket architecture with asynchronous processing via BullMQ** for any operation that exceeds typical HTTP timeout thresholds.

### Core Pattern Components

1. **HTTP Layer (Synchronous Initiation)**
   - Endpoint Pattern: `POST /{module}/{operation}`
   - Protocol: HTTP (REST)
   - Responsibility: Initiate operation, perform fast validations, return operation ID
   - Timeout: < 5 seconds for initial response

2. **WebSocket Layer (Asynchronous Communication)**
   - Protocol: WebSocket with Socket.IO
   - Authentication: JWT token-based
   - Responsibility: Real-time progress updates, completion notifications, error reporting
   - Connection Model: Persistent, bidirectional

3. **Queue Layer (Background Execution)**
   - Technology: BullMQ with Redis
   - Responsibility: Execute long-running operations asynchronously
   - Job Model: Parent/child job hierarchy for complex operations

### Generic LRO Workflow

```
Client Request (HTTP)
   ↓
[Fast Validation & Setup]
   ↓
Operation ID Generated ──────► Immediate HTTP Response (200 OK)
   ↓
Queue Job Created
   ↓
[Background Processing]
   ↓
Progress Updates ────────────► WebSocket Events
   ↓
Completion/Failure ──────────► WebSocket Final Event
   ↓
[Resource Cleanup]
```

### Pattern Application Examples

**File Upload & Processing:**
- HTTP: Accept file, validate format, return upload ID
- Queue: Parse file, validate data, process batches
- WebSocket: Batch progress, validation results, completion summary

**Bulk Data Validation:**
- HTTP: Accept validation request, return validation ID
- Queue: Execute validation rules across datasets
- WebSocket: Progress percentage, rule violations, final report

**Data Migration/Transformation:**
- HTTP: Initiate migration, return migration ID
- Queue: Transform records, handle dependencies
- WebSocket: Records processed, errors encountered, completion status

**Complex Calculations:**
- HTTP: Submit calculation request, return calculation ID
- Queue: Execute computation, handle intermediate results
- WebSocket: Calculation progress, partial results, final output

## Reasoning

### Decoupling & Scalability
- **Decouples client from processing lifecycle**: HTTP response returns immediately, allowing users to continue working while operation executes
- **Prevents timeout cascades**: Long operations don't hold HTTP connections open, avoiding timeout failures and connection pool exhaustion
- **Horizontal scaling capability**: Queue workers can be added independently of API servers, allowing targeted scaling for processing bottlenecks
- **Load distribution**: Redis-backed queue distributes work across multiple workers automatically

### User Experience
- **Immediate acknowledgment**: Users get instant feedback that their operation was accepted
- **Real-time visibility**: Continuous progress updates reduce anxiety and uncertainty during long operations
- **Resumable operations**: Users can disconnect and reconnect; progress state is maintained server-side
- **Detailed feedback**: Granular progress and error information helps users understand what's happening

### Reliability & Resilience
- **Automatic retry mechanisms**: BullMQ provides built-in retry logic for transient failures
- **Job persistence**: Operations survive server restarts; Redis stores job state
- **Error isolation**: Failed operations don't crash the API server
- **Resource cleanup**: Systematic cleanup on success or failure prevents resource leaks
- **Observability**: Operation IDs correlate events across all system layers

### Developer Experience
- **Consistent pattern**: Same architecture applies to all LROs across modules
- **Clear separation of concerns**: HTTP for initiation, WebSocket for communication, Queue for execution
- **Testable components**: Each layer can be tested independently
- **Reusable infrastructure**: Queue workers, WebSocket handlers, and HTTP controllers follow standard patterns

### Trade-offs Accepted
- **Increased system complexity**: Three-layer architecture requires more infrastructure than simple synchronous approach
- **Eventual consistency**: Brief delay between initiation and completion
- **Infrastructure dependencies**: Redis and WebSocket server required
- **Connection management overhead**: Must handle WebSocket lifecycle (connect, disconnect, reconnect)

## Consequences

### Positive
- **Handles operations of any duration**: No artificial limits imposed by HTTP timeouts
- **Production-ready scalability**: Proven pattern handles high load with horizontal scaling
- **Enhanced user experience**: Users stay informed throughout operation lifecycle
- **Flexible application**: Pattern works for uploads, validations, migrations, calculations, and more
- **Operational visibility**: Complete tracing from initiation through completion
- **Clean architecture**: Well-defined boundaries between layers
- **Graceful degradation**: Operations continue even if WebSocket temporarily disconnects

### Negative
- **Infrastructure complexity**: Requires Redis, WebSocket server, and queue workers
- **Development overhead**: More moving parts to develop, test, and debug
- **Operational burden**: More services to monitor, maintain, and troubleshoot
- **Learning curve**: Developers must understand async patterns and queue mechanics
- **Local development complexity**: Full stack requires multiple services running locally
- **Network reliability dependency**: WebSocket connections can be disrupted by network issues

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

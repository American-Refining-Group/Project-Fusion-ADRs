
**Title:** File Upload Workflow with Asynchronous Batch Processing  
**Date:** 2025-10-26  
**Status:** Accepted

## Context

The application requires a robust file upload system that can handle large Excel/CSV files containing business data. The system must address several key challenges:

**Key Requirements:**
- Process large files (up to 10MB) containing structured business data
- Validate file structure and data integrity before processing
- Provide real-time feedback to users during long-running operations
- Handle thousands of records efficiently without HTTP timeouts
- Ensure data accuracy with proper Excel date and accounting number parsing
- Maintain security through proper authentication and user isolation

**Constraints:**
- HTTP request timeout limitations for synchronous processing
- Need for immediate user feedback while processing continues
- File format complexity (Excel serial dates, accounting number formats)
- Resource cleanup requirements for temporary file storage

**Options Considered:**
1. **Synchronous HTTP-only processing** - Simple but causes timeouts on large files
2. **Pure asynchronous processing with polling** - Requires constant client polling, inefficient
3. **Hybrid HTTP/WebSocket with queue-based processing** - Balances immediate feedback with async processing

## Decision

We will implement a **hybrid HTTP/WebSocket architecture with asynchronous batch processing using BullMQ** for the file upload workflow.

**Key Components:**

1. **HTTP Layer (Synchronous)**
   - Endpoint: `POST /{module}/upload`
   - Protocol: HTTP with multipart/form-data
   - Handles: File upload, validation, and initial processing setup

2. **WebSocket Layer (Asynchronous)**
   - Protocol: WebSocket with Socket.IO
   - Authentication: JWT token-based
   - Handles: Real-time progress updates and final results

3. **Queue Layer (Background Processing)**
   - Technology: BullMQ with Redis
   - Handles: Asynchronous batch processing of uploaded data

**Processing Workflow:**

```
1. File Upload & Validation (HTTP)
   ↓
2. Data Parsing & Batch Creation
   ↓
3. Queue Job Creation (Parent + Child Jobs)
   ↓
4. Asynchronous Batch Processing (BullMQ)
   ↓
5. Real-time Updates (WebSocket)
   ↓
6. Final Summary & Cleanup
```

**Technical Specifications:**
- File types supported: .xlsx, .xls, .csv
- Maximum file size: 10MB (configurable)
- Storage location: `uploads/csv/{MODULE_NAME}/`
- Upload ID format: `{PREFIX}-{timestamp}-{uuid}`
- User-specific WebSocket rooms: `user-{userIdentifier}`
- Job structure: Parent job orchestrating multiple child batch jobs

## Reasoning

### Scalability
- **Asynchronous processing prevents HTTP timeouts**: By decoupling file upload from data processing, large files can be processed without hitting typical 30-60 second HTTP timeout limits
- **Batch processing handles large datasets efficiently**: Configurable batch sizes allow tuning for optimal memory usage and processing speed
- **Redis-backed queue supports horizontal scaling**: Multiple worker processes can consume jobs from the same queue, enabling easy scaling as load increases

### User Experience
- **Immediate upload confirmation**: Users receive instant HTTP response confirming file acceptance, preventing perceived delays
- **Real-time progress updates**: WebSocket events provide continuous feedback on processing status, reducing user anxiety during long operations
- **Comprehensive error reporting**: Specific, actionable error messages at multiple validation levels help users correct issues quickly

### Reliability
- **Automatic resource cleanup**: Temporary files are removed after processing (success or failure), preventing disk space exhaustion
- **Robust error handling**: Multi-level validation catches issues early; processing errors don't crash the system
- **Job tracking and monitoring**: Parent/child job structure enables detailed observability and debugging

### Security
- **JWT-based authentication for both protocols**: Consistent authentication across HTTP and WebSocket prevents unauthorized access
- **File type and size validation**: Prevents malicious file uploads and resource exhaustion attacks
- **User-specific data isolation**: WebSocket rooms ensure users only receive updates for their own uploads

### Trade-offs Accepted
- **Increased system complexity**: Three-layer architecture (HTTP/WebSocket/Queue) requires more infrastructure and monitoring compared to simple synchronous approach
- **Redis dependency**: Adds operational overhead of maintaining Redis instance
- **WebSocket connection management**: Requires handling connection lifecycle, reconnection logic, and authentication

## Consequences

### Positive
- **Handles files of any size within limits**: Large files with thousands of records process smoothly without timeouts
- **Excellent user experience**: Users can navigate away and return; progress is preserved and resumable
- **Production-ready scalability**: Can handle increased load by adding queue workers horizontally
- **Detailed observability**: Upload IDs correlate actions across all system layers for effective debugging
- **Clean separation of concerns**: HTTP handles upload, WebSocket handles communication, Queue handles processing
- **Reusable pattern**: Architecture can be applied to multiple modules requiring file upload functionality

### Negative
- **Increased infrastructure complexity**: Requires Redis server, WebSocket server, and queue workers in addition to standard HTTP API
- **More difficult local development**: Developers need Redis and multiple processes running to test the full workflow
- **WebSocket connection management**: Must handle disconnections, reconnections, and stale connections
- **Eventual consistency model**: Brief delay between upload confirmation and processing completion may confuse some users
- **Higher operational overhead**: More services to monitor, log aggregation across multiple components required

## Implementation Notes

### File Validation Pipeline
```typescript
1. File type validation: .xlsx, .xls, .csv only
2. File size validation: Configurable maximum (default 10MB)
3. Header validation: Exact column order and count matching
4. Data validation: At least 1 non-empty row after header
5. Format validation: Excel date conversion, accounting number parsing
```

### Response Schemas

**HTTP Success Response:**
```typescript
interface FileUploadResponseDto {
  message: string;           // "File accepted, split into batches"
  uploadId: string;         // "{PREFIX}-{timestamp}-{uuid}"
  totalRecords: number;     // Total records processed
  totalBatches: number;     // Number of batches created
  parentJobId: string;      // Parent job ID for tracking
  childJobIds: string[];    // Array of child job IDs
}
```

**WebSocket Batch Progress:**
```typescript
{
  "event": "batch-progress",
  "data": {
    "uploadId": "UPLOAD-1234567890-uuid",
    "batchId": "batch-1",
    "status": "completed",
    "processed": 5,
    "errors": 0,
    "warnings": 0
  }
}
```

**WebSocket Final Summary:**
```typescript
{
  "event": "upload-complete",
  "data": {
    "uploadId": "UPLOAD-1234567890-uuid",
    "summary": {
      "totalProcessed": 10,
      "successCount": 7,
      "errorCount": 2,
      "warningCount": 1,
      "additionalMetrics": {}  // Module-specific metrics
    }
  }
}
```

### Error Response Types

**File Validation Error (400):**
```json
{
  "statusCode": 400,
  "message": "Only Excel (.xlsx, .xls) and CSV files are allowed",
  "error": "Bad Request"
}
```

**Header Validation Error (400):**
```json
{
  "statusCode": 400,
  "message": "Header mismatch at column 3. Expected 'COLUMN_NAME' but found 'ColumnName'",
  "error": "Bad Request"
}
```

**Data Validation Error (400):**
```json
{
  "statusCode": 400,
  "message": "File must contain at least 1 non-empty data row(s) after the header row. Found only 0 non-empty data row(s).",
  "error": "Bad Request"
}
```

**Authentication Error (400):**
```json
{
  "statusCode": 400,
  "message": "User authentication required",
  "error": "Bad Request"
}
```

### Data Processing Pipeline
```typescript
1. parseFile() - Extract data with proper formatting
2. filterAndNormalizeRows() - Clean and filter empty rows
3. chunkArray() - Split into configurable batches
4. createQueueJobs() - Create parent/child job structure
5. processInBackground() - Execute batch processing
```

### Monitoring & Observability

**Logging Strategy:**
- Structured logging with upload IDs for correlation
- Process duration tracking at each stage
- Error logging with full stack traces

**Key Metrics:**
- Upload success/failure rates by module
- Processing time per batch
- Queue depth and processing rates
- WebSocket connection count and stability

**Tracing:**
- Upload ID correlation across HTTP/WebSocket/Queue layers
- Job ID tracking for debugging
- User context preservation throughout workflow

### Module-Specific Configuration

Each module can customize:
- Upload ID prefix (e.g., "CC" for Clear Checks, "INV" for Invoices)
- Expected file headers and column order
- Batch size for processing
- Module-specific validation rules
- Custom business logic for data processing
- Summary metrics format

## Related Decisions

[To be added as related ADRs are created for specific module implementations]

## References

- **BullMQ Documentation**: https://docs.bullmq.io/
- **Socket.IO Documentation**: https://socket.io/docs/
- **Excel Serial Date Format**: Microsoft Excel date system documentation
- **Multipart Form Data Specification**: RFC 7578
# RPG Program and Database File Development Standards  

**Title:** RPG Program and Database File Development Standards  
**Date:** 2025-07-14  
**Status:** In Progress  

## Context

**Architectural Problem:** Our current RPG application system relies on temporary flat files that need to be converted to externally defined permanent tables as part of our migration to a Power 10 server architecture. The existing system uses two distinct types of temporary files that require different handling approaches.

**Key Factors and Constraints:**
- Migration from temporary flat files to permanent database tables
- Two types of temporary files requiring different conversion strategies:
  1. Job-scoped tables (implementation approach TBD)
  2. User-scoped temporary files (specific conversion requirements defined)
- Multi-environment deployment across Power 8 (dev/test/UAT) and Power 10 (production) servers
- Need for proper library management and naming conventions
- Requirement to maintain data isolation and security

**Options Considered:**
- Continue with temporary files (rejected due to data persistence needs)
- Convert all temporary files to permanent tables with same structure (rejected due to security concerns)
- Implement differentiated approach based on file scope (selected)

**System Context:** IBM i (AS/400) environment with RPG programs accessing database files across multiple environments, with production running on Power 10 and lower environments on Power 8.

## Decision

**Architectural Decision:** Implement a tiered database architecture with externally defined permanent tables, environment-specific library management, and differentiated handling of temporary file types.

**Implementation Approach:**

**User-Scoped Temporary Files Conversion:**
- Convert to externally defined permanent tables
- Add USER_NAME field to enable user-based data isolation
- Store physical files in Working Data Library with "G" prefix
- Create logical files/views in Data Library without prefix for RPG program access
- Require user filtering in all INSERT/DELETE operations

**Job-Scoped Tables:**
- Status: To Be Determined
- Permanent data sorts will be replaced with logical files (SQL views)

**Library Structure by Environment:**

| Environment | Data Library | Working Data Library | Stored Procedure Library | Server |
|-------------|--------------|---------------------|-------------------------|---------|
| Development | DATADEV      | QS36FDEV           | GSSLIBDEV              | Power 8 |
| Testing     | DATATEST     | QS36FTEST          | GSSLIBTEST             | Power 8 |
| UAT         | DATAUAT      | QS36FUAT           | GSSLIBUAT              | Power 8 |
| Production  | DATA         | QS36F              | GSSLIB                 | Power 10|

**Technical Requirements:**
- All database files must be externally defined
- RPG programs reference logical files in Data Library only
- Physical files stored in Working Data Library with "G" prefix
- User filtering mandatory for user-scoped table access

## Reasoning

**Data Persistence and Integrity**
- Permanent tables provide data persistence beyond job/session scope
- Externally defined tables ensure better data integrity and documentation
- Structured approach to user data isolation improves security

**Maintainability and Scalability**
- Logical files provide abstraction layer for easier future modifications
- Environment-specific libraries enable proper deployment control
- Standardized naming conventions improve system maintainability
- Power 10 architecture provides improved performance and capacity

**Security and Data Isolation**
- User-based filtering prevents cross-user data access
- Separate library structure ensures environment isolation
- Controlled access patterns through logical file abstraction

**Trade-offs Considered:**
- Increased storage requirements vs. improved data persistence
- Additional development effort vs. long-term maintainability benefits
- User filtering complexity vs. enhanced security

**Risk Mitigation:**
- Phased approach allows for testing and refinement
- Deferred decision on job-scoped tables reduces immediate complexity
- Environment separation minimizes deployment risks

## Consequences

### Positive
- **Data Persistence:** User-scoped data persists beyond individual sessions
- **Improved Data Management:** Externally defined tables provide better integrity and documentation
- **Environment Isolation:** Separate libraries ensure clean environment management and deployment control
- **Enhanced Performance:** Power 10 server provides improved performance and capacity for production
- **Better Maintainability:** Logical files provide abstraction layer for easier future modifications
- **Enhanced Security:** User-based filtering prevents unauthorized data access

### Negative
- **Development Effort:** All programs referencing temporary files require modification for user filtering
- **Storage Requirements:** Permanent tables consume more storage than temporary files
- **Increased Complexity:** Additional user filtering logic required in all affected programs
- **Migration Risk:** Potential for data inconsistency during transition period
- **Cross-Server Complexity:** Different server architectures for development vs. production environments

## Implementation Notes

**User-Scoped Table Conversion Process:**
1. Create physical table in Working Data Library with "G" prefix (QS36FDEV/QS36FTEST/QS36FUAT/QS36F)
2. Add USER_NAME field to table structure
3. Create corresponding logical file/view in Data Library (DATADEV/DATATEST/DATAUAT/DATA)
4. Identify all programs referencing the temporary file
5. Modify programs to include user filtering logic
6. Test data access and filtering functionality
7. Deploy to appropriate environment libraries following promotion process

**RPG Program Modification Pattern:**
```rpg
// Before (temporary file access)
READ TEMPFILE;

// After (permanent table with user filtering)
CLEAR CUSTFILE;
USER_NAME = %USERID();
SETLL (USER_NAME) CUSTFILE;
READ CUSTFILE;
```

**Library Usage Guidelines:**
- **Data Library:** Contains logical files/views (non-prefixed names) that RPG programs reference
- **Working Data Library:** Contains physical tables with "G" prefix for actual data storage
- **Stored Procedure Library:** Contains database stored procedures and functions

**Server Infrastructure:**
- **Power 8 Server:** Hosts Development, Testing, and UAT environments
- **Power 10 Server:** Hosts Production environment only
- **Cross-Server Considerations:** Development and testing occur on Power 8, with final deployment to Power 10 for production

**Deployment Process:**
- New programs must be developed with the new library structure
- Existing programs migrated to appropriate environment libraries during deployment
- Cross-environment consistency maintained through standardized naming conventions

## Related Decisions

*[To be added as additional ADRs are created]*

## References

*[To be added as external documentation and standards are referenced]*

---

**Future Considerations:**
- Job-scoped table implementation strategy needs definition
- Performance monitoring requirements for user-filtered queries
- Indexing strategy for USER_NAME fields
- Data archiving approach for user-scoped permanent tables
- Backup and recovery procedure updates for new table structure
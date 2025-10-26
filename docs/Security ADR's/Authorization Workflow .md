**Title:** Role-Based Access Control (RBAC) Authorization Workflow

**Date:** 2025-10-26

**Status:** Accepted

## Context

The ARG Web system requires a secure and flexible authorization mechanism to control user access to various routes and endpoints. The system needed to address several key requirements:

- Validate user requests before they reach backend services
- Support both group-based and individual user-based permissions
- Allow fine-grained control where specific rights can be disabled for individual users even if their groups have those rights
- Enable custom rights assignment directly to users outside of group memberships
- Maintain separation of concerns between authentication and authorization
- Support many-to-many relationships between routes and rights

The authorization layer needed to work with JWT tokens and enforce access control in a consistent, auditable manner across all API endpoints. Additionally, the system required flexibility to handle complex scenarios where users belong to multiple groups but may need certain permissions blocked or custom permissions added.

## Decision

Implement a middleware-based authorization workflow that validates user access through a dedicated ARG Web Auth service using a multi-table RBAC model.

**Key Components:**

1. **Middleware Layer:** Intercepts all requests in the ARG Web Backend and forwards authorization checks to the Auth service
2. **JWT Token Processing:** Decodes access tokens to extract UserId for permission lookups
3. **Database Schema:**
   - `Group` - Stores all groups
   - `Rights` - Stores all rights/permissions
   - `GroupRights` - Maps rights to groups
   - `UserRights` - Stores user-specific right overrides (enable/disable)
   - `UserGroupMapping` - Associates users with groups (many-to-many)
   - `RouterMaster` - Stores all available routes/endpoints
   - `RouteRightsMapping` - Maps routes to required rights (many-to-many)

4. **Authorization Flow:**
   - Middleware sends: Access Token, MetaData, Path URL to Auth service
   - Auth service decodes token to extract UserId
   - System queries UserGroupMapping to get all user's groups
   - Derives rights from GroupRights for all user's groups
   - Applies UserRights overrides (disabled rights are blocked, custom rights are added)
   - Checks RouteRightsMapping to verify if user has required rights for the route
   - Returns authorized/unauthorized response

## Reasoning

**Security & Separation of Concerns**
- Centralized authorization logic in a dedicated Auth service prevents inconsistent security enforcement across the application
- Middleware pattern ensures no endpoint can bypass authorization checks
- JWT token validation provides stateless authentication that scales well

**Flexibility & Granularity**
- Group-based permissions provide efficient management for common access patterns
- UserRights table enables precise control by allowing administrators to disable specific group rights or add custom rights for individual users
- This hybrid approach handles both standard and exceptional access requirements without creating numerous single-purpose groups

**Scalability & Maintainability**
- Many-to-many relationships (UserGroupMapping, RouteRightsMapping) allow flexible assignment without data duplication
- Users can belong to multiple groups, inheriting rights from all memberships
- Routes can require multiple rights, enabling complex authorization rules
- Normalized database structure makes it easy to audit permissions and add new routes or rights

**Performance Considerations**
- Single authorization check per request minimizes database queries
- UserId-based lookup is efficient and can be cached
- Trade-off: Additional network call to Auth service adds latency, but gains security and maintainability

## Consequences

### Positive
- **Fine-grained access control:** System can handle complex permission scenarios including group inheritance with individual overrides
- **Audit trail:** Clear permission structure makes it easy to determine why a user has or doesn't have access
- **Centralized security:** Single point of authorization enforcement reduces security vulnerabilities
- **Flexibility:** New routes, rights, and groups can be added without code changes
- **Scalability:** Stateless JWT validation allows horizontal scaling of Auth service

### Negative
- **Additional latency:** Every request requires a call to the Auth service, adding network overhead
- **Complexity:** Multi-table design requires understanding of the entire permission model for troubleshooting
- **Potential inconsistency:** UserRights overrides can create confusion if not well-documented (users blocked from group rights they should have)
- **Database load:** Authorization checks may create significant database load under high traffic
- **Single point of failure:** Auth service unavailability blocks all requests (requires high availability setup)

## Implementation Notes

**Authorization Request Format:**
```json
{
  "accessToken": "JWT_TOKEN",
  "metaData": ["key1", "key2"],
  "pathUrl": "/api/resource/endpoint"
}
```

**Authorization Response:**
- Success: Request forwarded to ARG Web Backend
- Failure: Return "Not Authorized" error response

**Permission Resolution Algorithm:**
1. Extract UserId from JWT
2. Query UserGroupMapping for all groups where UserId exists
3. Query GroupRights for all rights associated with those groups
4. Query UserRights for user-specific overrides
5. Apply logic:
   - Start with all group rights
   - Remove any rights marked as disabled in UserRights (IsActive = false)
   - Add any custom rights marked as enabled in UserRights (IsActive = true)
6. Query RouteRightsMapping to get required rights for requested route
7. Check if user's final rights set includes all required rights

**Database Relationships:**
- User → UserGroupMapping (1:N) → Group
- Group → GroupRights (1:N) → Rights
- User → UserRights (1:N) → Rights (with IsActive flag)
- Route → RouteRightsMapping (1:N) → Rights

## Related Decisions

[No related ADRs referenced in the source document]

## References

- Internal document: "Authorization Workflow in ARG Web System"
- JWT (JSON Web Tokens) standard for authentication
- RBAC (Role-Based Access Control) design patterns
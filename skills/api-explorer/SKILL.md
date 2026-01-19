---
name: api-explorer
description: Explore and test APIs from Swagger/OpenAPI documentation with interactive parameter collection and dry-run simulation before actual execution.
user-invocable: true
---

# API Explorer

## Description
Automatically explores and tests APIs from Swagger/OpenAPI documentation. Analyzes required parameters, collects values interactively, simulates requests, and executes API calls with comprehensive response analysis.

## Core Principle
**ALWAYS ASK, NEVER ASSUME**: When any aspect of the API specification is unclear, ambiguous, or missing, the skill MUST ask the user for clarification rather than making assumptions. This includes:
- Authentication methods
- Parameter formats and types
- Required vs optional fields
- Enum values
- Base URLs and environments
- Schema structures
- Any other unclear specification details

## Features
- Parse Swagger/OpenAPI documentation (URL or local file)
- Detect required/optional parameters and authentication
- Interactive parameter collection
- Dry-run simulation before execution
- Multi-endpoint scenario testing
- Response chaining (use response from one API as input for another)
- Configuration persistence for common values (API keys, base URLs)

## Initial Setup

### Configuration File
On first use, the skill will create `~/.claude/api-explorer-config.json` to store:
- Common API keys
- Base URLs
- Default headers
- Saved scenarios

## Process

### 1. Input Swagger/OpenAPI Documentation
- **Option A**: Provide Swagger URL
  - Example: `https://api.example.com/swagger.json`
  - The skill will fetch and parse the OpenAPI spec
- **Option B**: Provide local file path
  - Example: `/path/to/openapi.yaml`
  - Supports both JSON and YAML formats

### 2. Parse API Documentation
**IMPORTANT**: During parsing, ALWAYS ask the user for clarification when encountering ambiguous or missing information.

Parse the Swagger/OpenAPI document and identify:
- All available endpoints
- Authentication methods
- Request/response schemas
- Required vs optional parameters

**Ask user for clarification when:**

#### A. Multiple Base URLs
If the spec contains multiple servers:
```
Found multiple base URLs:
1. https://dev-api.example.com (Development)
2. https://staging-api.example.com (Staging)
3. https://api.example.com (Production)

Which environment would you like to use? (Enter number):
```

#### B. Multiple Authentication Methods
If multiple security schemes are defined:
```
This API supports multiple authentication methods:
1. Bearer Token (HTTP Bearer)
2. API Key (Header: X-API-Key)
3. OAuth2 (Authorization Code Flow)

Which authentication method would you like to use? (Enter number):
```

#### C. Unclear Parameter Types
If parameter schema is ambiguous:
```
Parameter 'status' has unclear type specification.
Examples in documentation: "active", "inactive", "pending"

Is this:
1. String enum (one of: active, inactive, pending)
2. Free text string
3. Something else

Please clarify (Enter number or description):
```

#### D. Missing Required Information
If critical information is missing:
```
⚠️ Base URL is not specified in the OpenAPI document.

Please provide the base URL for this API:
(e.g., https://api.example.com)
```

#### E. Deprecated or Beta Endpoints
If endpoint has special status:
```
⚠️ Endpoint GET /api/v1/users is marked as DEPRECATED
   Recommended alternative: GET /api/v2/users

Do you want to:
1. Continue with deprecated endpoint
2. Switch to recommended endpoint
3. Cancel

Enter choice (1-3):
```

#### F. Complex Schema Structures
If request body has complex nested objects or unions:
```
Request body has complex structure with multiple possible formats:

Format 1: User with email
{
  "type": "email",
  "email": "string",
  "name": "string"
}

Format 2: User with phone
{
  "type": "phone",
  "phone": "string",
  "name": "string"
}

Which format would you like to use? (Enter number):
```

**General Rule**: If the AI is uncertain about ANY aspect of the API specification, it MUST ask the user rather than making assumptions.

### 3. Endpoint Selection
Present available endpoints to user:
```
Available Endpoints:
1. GET /api/users - List all users
2. POST /api/users - Create new user
3. GET /api/users/{id} - Get user by ID
4. PUT /api/users/{id} - Update user
5. DELETE /api/users/{id} - Delete user
...

Which endpoint would you like to test? (Enter number or path)
```

### 4. Parameter Analysis & Collection
**IMPORTANT**: During parameter collection, ALWAYS ask for clarification if:
- Parameter format is unclear (e.g., date format: ISO8601? Unix timestamp?)
- Enum values need explanation
- Parameter relationships exist (if X is set, Y is required)
- Example values seem confusing or contradictory
- Documentation is missing or incomplete

For selected endpoint, analyze and collect:

#### A. Authentication
- Detect auth type from OpenAPI spec
- Check if auth credentials exist in config
- If not, ask user:
  ```
  This API requires Bearer authentication.
  Please provide your Bearer token:
  ```
- Offer to save to config for future use

#### B. Path Parameters
- Example: `/api/users/{id}` requires `id`
- Ask: `Please provide value for path parameter 'id':`

#### C. Query Parameters
- List all query parameters with required/optional status
- **Ask for clarification when needed**:
  ```
  Query Parameters:
  - page (optional, integer): Page number for pagination
  - limit (optional, integer): Number of items per page
  - search (optional, string): Search term
  - date (optional, string): Filter by date

  ⚠️ Parameter 'date' format is not specified in the documentation.

  What format should I use for the 'date' parameter?
  1. ISO 8601 (e.g., 2026-01-20)
  2. Unix timestamp (e.g., 1737388800)
  3. Other format (please specify)

  Enter choice: 1

  Enter value for 'page' (or press Enter to skip):
  ```

#### D. Request Body
- Parse JSON schema
- Collect required fields first, then optional
- **Ask for clarification when schema is unclear**:
  ```
  Request Body (application/json):
  - name (required, string): User's full name
  - email (required, string): User's email address
  - age (optional, integer): User's age
  - preferences (optional, object): User preferences

  ⚠️ Field 'preferences' is defined as 'object' but no structure is specified.

  What should the structure of 'preferences' be?
  1. Skip this field (leave empty)
  2. Provide custom JSON object
  3. Use example from documentation: {"theme": "dark", "language": "en"}

  Enter choice: 3

  Enter value for 'name':
  ```

- **For enum fields, show available options**:
  ```
  - status (required, string enum): User status
    Available values:
    1. active - User is active
    2. inactive - User is temporarily inactive
    3. suspended - User is suspended

  Select value for 'status' (1-3): 1
  ```

#### E. Headers
- Detect required custom headers
- Offer common headers (Content-Type, Accept, etc.)

### 5. Request Simulation (Dry-run)
Before actual execution, show complete request details:

```
=== REQUEST PREVIEW ===
Method: POST
URL: https://api.example.com/api/users
Headers:
  Authorization: Bearer eyJhbGc...
  Content-Type: application/json
  Accept: application/json

Body:
{
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30
}

=== CURL EQUIVALENT ===
curl -X POST 'https://api.example.com/api/users' \
  -H 'Authorization: Bearer eyJhbGc...' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "John Doe",
    "email": "john@example.com",
    "age": 30
  }'

Proceed with this request? (yes/no/edit)
```

Options:
- **yes**: Execute the request
- **no**: Cancel
- **edit**: Modify parameters

### 6. Execute API Call
- Make actual HTTP request using collected parameters
- Handle timeouts and network errors gracefully
- Support for:
  - All HTTP methods (GET, POST, PUT, DELETE, PATCH, etc.)
  - Different content types (JSON, XML, form-data, etc.)
  - File uploads (multipart/form-data)

### 7. Response Analysis
Display comprehensive response information:

```
=== RESPONSE ===
Status: 201 Created
Time: 245ms

Headers:
  Content-Type: application/json
  X-Request-ID: abc123

Body:
{
  "id": "user_12345",
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30,
  "createdAt": "2026-01-20T10:30:00Z"
}

=== ANALYSIS ===
✅ Success: User created successfully
📝 New resource ID: user_12345
💾 Save 'id' for next request? (yes/no)
```

If response contains useful values:
- Offer to save them for next request
- Example: Save `id` from POST response to use in GET request

### 8. Scenario Testing (Optional)
Allow chaining multiple API calls:

```
Would you like to test another endpoint using this response? (yes/no)

If yes:
1. Show available endpoints
2. Offer to use response values as parameters
3. Example: Use created user's ID in GET /api/users/{id}
```

## Advanced Features

### A. Save Scenarios
```
Save this API test scenario? (yes/no)

If yes:
- Scenario name: "Create and fetch user"
- Saved to: ~/.claude/api-explorer-scenarios/create-fetch-user.json

Run saved scenario: /api-explorer --scenario "Create and fetch user"
```

### B. Environment Variables
Support for different environments:
```json
{
  "environments": {
    "dev": {
      "baseUrl": "https://dev-api.example.com",
      "apiKey": "dev_key_123"
    },
    "staging": {
      "baseUrl": "https://staging-api.example.com",
      "apiKey": "staging_key_456"
    },
    "production": {
      "baseUrl": "https://api.example.com",
      "apiKey": "prod_key_789"
    }
  }
}
```

Usage: `/api-explorer --env staging`

### C. Batch Testing
Test multiple endpoints in sequence:
```
Batch Test:
1. POST /api/auth/login (get token)
2. GET /api/users (with token)
3. POST /api/users/{id}/profile (with token)

Execute batch test? (yes/no)
```

## Error Handling

### Common Errors
1. **Invalid Swagger URL**: Check URL format and accessibility
2. **Authentication Failed**: Verify API key/token is correct
3. **400 Bad Request**: Review required parameters
4. **401 Unauthorized**: Check authentication credentials
5. **404 Not Found**: Verify endpoint path and parameters
6. **500 Internal Server Error**: Show error details if available
7. **Network Timeout**: Offer to retry with longer timeout

For each error, provide:
- Clear error message
- Suggested fix
- Option to retry or edit parameters

## Configuration File Format

`~/.claude/api-explorer-config.json`:
```json
{
  "savedCredentials": {
    "example-api": {
      "baseUrl": "https://api.example.com",
      "authType": "bearer",
      "token": "your_token_here"
    }
  },
  "defaultHeaders": {
    "Accept": "application/json",
    "User-Agent": "Claude-API-Explorer/1.0"
  },
  "preferences": {
    "autoSave": true,
    "showCurlCommand": true,
    "timeout": 30000
  }
}
```

## Tools Required
- **Bash**: For curl commands (API calls)
- **Read**: For reading local OpenAPI files and config
- **Write**: For saving configuration and scenarios
- **WebFetch**: For fetching remote Swagger documentation

## Example Usage

### Example 1: Simple GET Request
```
User: /api-explorer https://petstore.swagger.io/v2/swagger.json
Assistant: Loaded Petstore API documentation. Found 20 endpoints.

Which endpoint would you like to test?
1. GET /pet/findByStatus - Finds Pets by status
...

User: 1
Assistant: Selected: GET /pet/findByStatus

Query Parameters:
- status (required, array): Status values that need to be considered for filter
  Allowed values: available, pending, sold

Enter value for 'status': available

=== REQUEST PREVIEW ===
Method: GET
URL: https://petstore.swagger.io/v2/pet/findByStatus?status=available

Proceed? yes/no: yes

=== RESPONSE ===
Status: 200 OK
Found 10 pets with status 'available'
[Shows formatted response]
```

### Example 2: POST with Authentication
```
User: /api-explorer https://api.myservice.com/openapi.json
Assistant: This API requires Bearer authentication.

Do you have saved credentials for 'myservice'? (yes/no): no
Please provide your Bearer token: [user enters token]
Save this token for future use? (yes/no): yes

Selected: POST /api/users

Request Body:
- name (required): John Doe
- email (required): john@example.com

[Dry-run preview]
Proceed? yes

[Executes and shows response]
User created with ID: 12345

Save this ID for next request? yes
```

### Example 3: Scenario Testing
```
User: /api-explorer --scenario "User CRUD Flow"
Assistant: Running saved scenario "User CRUD Flow"...

Step 1/4: POST /api/users (Create user)
✅ Success - User created (ID: 12345)

Step 2/4: GET /api/users/12345 (Fetch user)
✅ Success - User retrieved

Step 3/4: PUT /api/users/12345 (Update user)
✅ Success - User updated

Step 4/4: DELETE /api/users/12345 (Delete user)
✅ Success - User deleted

Scenario completed: 4/4 passed
```

## Security Notes
- **Credentials Storage**: API keys and tokens are stored locally in `~/.claude/`
- **Never commit**: Ensure config files are in `.gitignore`
- **Sensitive Data**: Option to use environment variables instead of storing in config
- **HTTPS Only**: Warn user when connecting to non-HTTPS endpoints

## Tips for Users
1. Start with GET endpoints (no data modification)
2. Use dry-run to verify requests before execution
3. Save frequently used credentials to config
4. Create scenarios for common workflows
5. Test in dev/staging before production

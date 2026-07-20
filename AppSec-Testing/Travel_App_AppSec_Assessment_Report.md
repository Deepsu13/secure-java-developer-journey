# Travel App API Security Assessment Report

## 1. Executive Summary

This security assessment was performed on the Travel App API to identify common application security vulnerabilities.

The assessment focused on:

- Authentication testing
- Authorization testing
- Broken Access Control testing
- API security testing
- Sensitive data exposure testing

### Tools Used

- Burp Suite Community Edition
- Browser Developer Tools
- DBeaver (Database Verification)

### Summary of Findings

| ID | Vulnerability | Severity | Status |
|---|---|---|---|
| SEC-001 | IDOR - Unauthorized Read Access | High | Confirmed |
| SEC-002 | IDOR - Unauthorized Delete Access | High | Confirmed |
| SEC-003 | Sensitive Data Exposure | Medium | Confirmed |

---

# 2. Application Scope

## Application

Travel App

## Tested APIs

```http
GET    /api/diary-memories
GET    /api/diary-memories/{id}
POST   /api/diary-memories
DELETE /api/diary-memories/{id}
```

---

# 3. Testing Methodology

The following approach was followed:

1. Created multiple user accounts:
   - Victim user
   - Attacker user

2. Authenticated using valid credentials.

3. Captured API requests using Burp Suite.

4. Modified object identifiers to test authorization controls.

5. Compared access between different users.

6. Verified impact through application testing.

---

# 4. Detailed Findings

---

# SEC-001: IDOR - Unauthorized Read Access

## Severity

High

## Vulnerable Endpoint

```http
GET /api/diary-memories/{id}
```

## Description

The API does not properly verify whether the requested diary memory belongs to the authenticated user.

An attacker can modify the diary memory ID and access another user's private diary memory.

---

## Test Scenario

### Attacker Account

```
attacker@test.com
```

### Victim Account

```
rumipegu@test.com
```

### Victim Resource

```
Diary Memory ID: 9
Title: Vietnam
```

---

## Steps to Reproduce

1. Login as attacker user.

2. Capture the attacker JWT token using Burp Suite.

3. Send the following request:

```http
GET /api/diary-memories/9
Authorization: Bearer <attacker_token>
```

4. Observe the response:

```json
{
"id":9,
"title":"Vietnam",
"user":{
    "email":"rumipegu@test.com",
    "id":2,
    "name":"Rumi"
}
}
```

---

## Actual Result

The attacker account was able to access another user's diary memory.

---

## Expected Result

The API should verify ownership before returning the resource.

Expected response:

```
403 Forbidden
```

---

## Impact

- Unauthorized access to private user information.
- Privacy violation.
- Exposure of user-generated content.

---

## Recommendation

Implement object-level authorization checks.

Example:

```java
findByIdAndUserId(memoryId, loggedInUserId)
```

Only return resources belonging to the authenticated user.

---

# SEC-002: IDOR - Unauthorized Delete Access

## Severity

High

## Vulnerable Endpoint

```http
DELETE /api/diary-memories/{id}
```

---

## Description

The delete API does not verify whether the diary memory belongs to the authenticated user.

An attacker can delete another user's diary memory by changing the object ID.

---

## Test Scenario

### Attacker Account

```
attacker@test.com
```

### Victim Account

```
rumipegu@test.com
```

### Victim Resource

```
Diary Memory ID: 9
```

---

## Steps to Reproduce

1. Login as attacker.

2. Capture DELETE request.

3. Modify the resource identifier:

```http
DELETE /api/diary-memories/9
Authorization: Bearer <attacker_token>
```

4. Send request.

---

## Actual Result

Server accepted the request:

```
HTTP/1.1 200 OK
```

The victim's diary memory was deleted successfully.

---

## Expected Result

The API should validate ownership before deletion.

Expected response:

```
403 Forbidden
```

---

## Impact

- Unauthorized deletion of user data.
- Data integrity compromise.
- Permanent loss of user-generated content.

---

## Recommendation

Perform ownership validation before deleting.

Example:

```java
deleteByIdAndUserId(id, loggedInUserId)
```

---

# SEC-003: Sensitive Data Exposure

## Severity

Medium

## Vulnerable Endpoints

```http
GET /api/diary-memories

GET /api/diary-memories/{id}

POST /api/diary-memories
```

---

## Description

The API response exposes unnecessary user information.

Example:

```json
{
"user":{
    "email":"rumipegu@test.com",
    "id":2,
    "name":"Rumi",
    "password":"$2a$10$...",
    "role":"USER"
}
}
```

---

## Sensitive Information Exposed

- Password hash
- User ID
- User role
- Internal user information

---

## Impact

- Increased risk if password hashes are leaked.
- Exposure of unnecessary internal information.
- Violates the principle of least privilege.

---

## Recommendation

Do not return database entities directly.

Use DTO objects.

Example:

```java
DiaryMemoryResponseDTO
```

Return only required fields:

```json
{
"id":9,
"title":"Vietnam",
"mood":"adventure",
"note":"beautiful trip",
"travelDate":"2026-04-04"
}
```

---

# 5. Security Conclusion

## Positive Findings

✅ Authentication implemented using JWT  
✅ User login and registration working correctly  
✅ Create operation correctly assigns ownership from authenticated user  

## Security Issues Identified

❌ Missing object-level authorization checks  
❌ Unauthorized access to another user's resources  
❌ Unauthorized deletion of another user's data  
❌ Sensitive information exposed in API responses  

---

# Priority Fixes

## 1. Implement Authorization Checks

Apply ownership validation for:

- GET by ID
- DELETE
- UPDATE (future functionality)

---

## 2. Implement DTO-Based Responses

Remove sensitive fields:

- Password hash
- Internal IDs
- Unnecessary user details

---

## 3. Add Automated Security Tests

Recommended future tests:

- IDOR testing
- Privilege escalation testing
- JWT validation testing
- Input validation testing

---

# Tools Used

```
Burp Suite Community Edition
Spring Boot
JWT Authentication
PostgreSQL
```

---

# End of Report

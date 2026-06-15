
Here is the complete API documentation for every endpoint in your Credit Card Application System, ready for Postman testing.

---

📍 Base URL

```
http://localhost:8080
```

🔐 Authentication

All protected endpoints require a JWT token in the Authorization header:

```
Authorization: Bearer <token>
```

The token is obtained via the Login endpoint.

---

1️⃣ Auth Endpoints

POST /auth/register – Register a new user

Public. Creates a regular user account.

Request

```http
POST /auth/register
Content-Type: application/json

{
  "fullName": "John Doe",
  "email": "john@example.com",
  "password": "password123"
}
```

Success Response 200 OK

```json
{
  "message": "User registered successfully",
  "userId": 1,
  "email": "john@example.com"
}
```

Error Responses

Status Message
409 Conflict Email already in use
400 Bad Request Validation errors (missing fields)

---

POST /auth/login – Authenticate & get JWT

Public. Works for both users and admins.

Request

```http
POST /auth/login
Content-Type: application/json

{
  "email": "admin@bank.com",
  "password": "Admin@123"
}
```

Success Response 200 OK

```json
{
  "token": "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhZG1pbkBiYW5rLmNvbSIsInJvbGUiOiJST0xFX0FETUlOIiwiaWF0IjoxNzE4NTYwMDAwLCJleHAiOjE3MTg2NDY0MDB9.abc123..."
}
```

Error Responses

Status Message
401 Unauthorized Invalid credentials
400 Bad Request Missing fields

---

POST /auth/validate-token – Check token validity

Public. Useful for the frontend to verify stored token.

Request

```http
POST /auth/validate-token
Content-Type: application/json

{
  "token": "eyJhbGciOiJIUzI1NiJ9..."
}
```

Success Response 200 OK

```json
{
  "valid": true,
  "email": "admin@bank.com"
}
```

If invalid

```json
{
  "valid": false
}
```

---

2️⃣ Customer Endpoints (role: ROLE_USER)

All require a valid user JWT.

POST /apply – Apply for a credit card

Role: USER

Request

```http
POST /apply
Authorization: Bearer <user_token>
Content-Type: application/json

{
  "income": 75000,
  "employmentType": "SALARIED",
  "employerName": "Acme Corp",
  "employmentYears": 5
}
```

Success Response 200 OK

```json
{
  "applicationId": 1,
  "status": "SUBMITTED",
  "income": 75000.00,
  "employmentType": "SALARIED",
  "employerName": "Acme Corp",
  "employmentYears": 5,
  "creditScore": null,
  "creditScoreValid": false,
  "incomeValid": false,
  "employmentValid": false,
  "fraudCheckPassed": false,
  "userFullName": "John Doe",
  "userEmail": "john@example.com",
  "remarks": null,
  "createdAt": "2026-06-15T13:15:00",
  "updatedAt": "2026-06-15T13:15:00"
}
```

Error Responses

Status Message
400 Bad Request You are blacklisted and cannot apply.
400 Bad Request Unemployed cannot have employer details
400 Bad Request Employer name required for employed
400 Bad Request Validation errors

---

GET /applications/my – Get my applications

Role: USER

Request

```http
GET /applications/my
Authorization: Bearer <user_token>
```

Success Response 200 OK

```json
[
  {
    "applicationId": 1,
    "status": "SUBMITTED",
    "income": 75000.00,
    ...
  }
]
```

---

GET /applications/{id} – Get application details

Role: USER (only own applications)

Request

```http
GET /applications/1
Authorization: Bearer <user_token>
```

Success Response 200 OK

```json
{
  "applicationId": 1,
  "status": "UNDER_REVIEW",
  ...
}
```

Error Responses

Status Message
404 Not Found Application not found
400 Bad Request Not your application

---

GET /applications/{id}/status – Track status

Role: USER (only own)

Request

```http
GET /applications/1/status
Authorization: Bearer <user_token>
```

Success Response 200 OK

```json
{
  "applicationId": 1,
  "status": "UNDER_REVIEW",
  "remarks": "Under review"
}
```

---

3️⃣ Admin Endpoints (role: ROLE_ADMIN)

All require an admin token.

GET /admin/applications – All applications

Role: ADMIN

Request

```http
GET /admin/applications
Authorization: Bearer <admin_token>
```

Success Response 200 OK – Array of applications.

---

GET /admin/customers – All customers (non‑admin)

Role: ADMIN

Request

```http
GET /admin/customers
Authorization: Bearer <admin_token>
```

Success Response 200 OK

```json
[
  {
    "userId": 1,
    "fullName": "John Doe",
    "email": "john@example.com",
    "createdAt": "2026-06-15T13:14:00",
    "blacklisted": false
  }
]
```

---

GET /admin/customers/{id} – Single customer

Role: ADMIN

Request

```http
GET /admin/customers/1
Authorization: Bearer <admin_token>
```

Error 404 Not Found if user doesn't exist.

---

GET /admin/customers/search – Search customers

Role: ADMIN

Request

```http
GET /admin/customers/search?name=john&email=john
Authorization: Bearer <admin_token>
```

Returns matching users (case-insensitive LIKE on name or email).

---

PUT /admin/applications/{id}/approve – Approve application

Role: ADMIN

Request

```http
PUT /admin/applications/1/approve
Authorization: Bearer <admin_token>
```

Error Responses

Status Message
400 Bad Request All checks must pass before approval
400 Bad Request Application cannot be approved in current status

---

PUT /admin/applications/{id}/reject – Reject application

Role: ADMIN

Request

```http
PUT /admin/applications/1/reject
Authorization: Bearer <admin_token>
```

---

PUT /admin/applications/{id}/review – Move to review

Role: ADMIN

Request

```http
PUT /admin/applications/1/review
Authorization: Bearer <admin_token>
```

---

PUT /admin/applications/{id}/refer-back – Send back to user

Role: ADMIN

Request

```http
PUT /admin/applications/1/refer-back
Authorization: Bearer <admin_token>
```

---

PUT /admin/applications/{id}/check-score – Credit score check

Role: ADMIN

Request

```http
PUT /admin/applications/1/check-score
Authorization: Bearer <admin_token>
Content-Type: application/json

{
  "creditScore": 720
}
```

Notes:

· Pass if creditScore >= 700.
· Updates creditScoreValid and remarks.

---

PUT /admin/applications/{id}/check-income – Income validation

Role: ADMIN

Request

```http
PUT /admin/applications/1/check-income
Authorization: Bearer <admin_token>
```

Notes: Pass if income > 0.

---

PUT /admin/applications/{id}/check-employment – Employment validation

Role: ADMIN

Request

```http
PUT /admin/applications/1/check-employment
Authorization: Bearer <admin_token>
```

Notes:

· Fails if UNEMPLOYED.
· Else, passes if employerName is non‑blank and employmentYears > 0.

---

PUT /admin/applications/{id}/check-fraud – Fraud check

Role: ADMIN

Request

```http
PUT /admin/applications/1/check-fraud
Authorization: Bearer <admin_token>
```

Notes:

· Fails if the same user has another application with status SUBMITTED or UNDER_REVIEW.

---

POST /admin/blacklist/{userId} – Blacklist user

Role: ADMIN

Request

```http
POST /admin/blacklist/1
Authorization: Bearer <admin_token>
```

Success 200 OK → "User blacklisted"

---

DELETE /admin/blacklist/{userId} – Unblock user

Role: ADMIN

Request

```http
DELETE /admin/blacklist/1
Authorization: Bearer <admin_token>
```

Success 200 OK → "User unblocked"

---

GET /admin/blacklist – List blacklisted users

Role: ADMIN

Request

```http
GET /admin/blacklist
Authorization: Bearer <admin_token>
```

Success Response 200 OK – Array of users with blacklisted: true.

---

4️⃣ Card Endpoint (admin only)

POST /cards/issue/{applicationId} – Issue credit card

Role: ADMIN
Precondition: Application status must be APPROVED.

Request

```http
POST /cards/issue/1
Authorization: Bearer <admin_token>
```

Success Response 200 OK

```json
{
  "cardId": 1,
  "cardNumber": "4532789012345678",
  "cvv": "123",
  "expiryDate": "2029-06-15",
  "cardHolderName": "John Doe",
  "status": "ACTIVE",
  "issuedAt": "2026-06-15T13:20:00"
}
```

Error Responses

Status Message
400 Bad Request Application must be APPROVED to issue a card
404 Not Found Application not found

---

🔄 Complete Workflow (Quick Reference)

1. Register a user → POST /auth/register
2. Login (user/admin) → POST /auth/login
3. (User) Apply → POST /apply
4. (Admin) Review → PUT /admin/applications/{id}/review
5. (Admin) Checks → check-score, check-income, check-employment, check-fraud
6. (Admin) Approve → PUT /admin/applications/{id}/approve
7. (Admin) Issue card → POST /cards/issue/{id}

---

All endpoints are now fully documented. You can copy these examples directly into Postman to test your entire system.

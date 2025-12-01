# Security Implementation - Automated Attendance Management System (AAMS)

## Overview
The **Automated Attendance Management System (AAMS)** backend is secured using **Spring Security** with **JWT-based authentication** and **role-based access control (RBAC)**. This document provides a comprehensive overview of how the system handles authentication, authorization, token management, and common security practices.

---

## 1. Spring Security Architecture

1. **Security Configuration (`SecurityConfig`)**
    - Annotated with `@EnableWebSecurity` and `@EnableMethodSecurity`.
    - Defines the **HTTP security rules** (which endpoints are public or require authentication).
    - Registers **JWT filters**, **authentication providers**, and **logout handlers**.

2. **Authentication Provider & User Details**
    - A custom `AuthenticationProvider` (a `DaoAuthenticationProvider`) uses:
        - `UserDetailsService` to fetch user details (username, password, roles) from the repository.
        - `PasswordEncoder` (BCrypt) for secure password hashing.
    - Ensures that user credentials are validated against the database.

3. **Filter Chain**
    - Configured in `SecurityConfig` to apply filters in order.
    - A **custom JWT filter**, `JwtAuthenticationFilter` intercepts requests, checks the token, and sets the authenticated user in the **SecurityContext** if valid.

---

## 2. Authentication & Authorization

- **JWT-Based Authentication**
    - **Access tokens**: Short-lived tokens used for most API requests.
    - **Refresh tokens**: Longer-lived tokens used strictly to obtain new access tokens and user CRUD operations.

- **Role-Based Access Control (RBAC)**
    - **Student**: Can only view or check their own attendance records.
    - **Teacher**: Can mark attendance, view class attendance, and manage classes they own.
    - **Admin**: Full control over user and class management (create, update, delete).
    - **System Administrator (SysAdmin)**:
        - Manages the entire system infrastructure.
        - Handles user roles, configurations, and application-wide settings.
        - Has the highest level of control, including security policies, system logs, and global settings.

---

## 3. JWT Token Flow

### 3.1 Access & Refresh Tokens
- **Access Token**
    - Encodes user details (e.g., username, roles) and expires quickly (1 day).
    - Sent in the `Authorization` header: `Bearer <access_token>`.
    - Minimizes damage if the token is leaked.

- **Refresh Token**
    - Longer-lived token, stored securely (Hashed in backend database,HTTP-only cookie or secure storage).
    - Used to **refresh** the access token without re-entering credentials.
    - Stored in the database with flags (e.g., `revoked`, `expired`) for revocation checks.

### 3.2 Token Generation
1. **Login**
    - User sends valid credentials via `POST /api/auth/login`.
    - On successful authentication, the system generates:
        - **Access token** (short lifetime).
        - **Refresh token** (longer lifetime, persisted in the database).
2. **Registration**
    - Similar process to login if you choose to auto-log the user in after signup.
    - The newly created user may receive both tokens upon successful registration.

### 3.3 Token Verification
- A **custom JWT filter** inspects every incoming request:
    - Extracts the token from the `Authorization` header (`Bearer <token>`).
    - Validates token integrity (signature, expiration) and checks the repository for revocation.
    - If valid, a **UsernamePasswordAuthenticationToken** is created for the user.
    - If invalid, the filter responds with `HTTP 401 Unauthorized`.

---

## 4. Token Security

- **Access Token Expiry**
    - Access tokens are short-lived to reduce security risks.
- **Refresh Token Expiry**
    - Refresh tokens are longer-lived but **securely stored** and revocable.
- **Token Revocation**
    - Tokens are invalidated (marked revoked or expired) upon user logout or password change.

### 4.1 Token Verification Endpoints
| Method | Endpoint                          | Description                          |
|--------|-----------------------------------|--------------------------------------|
| `GET`  | `/api/auth/verify-access-token`   | Verifies if an access token is valid |
| `GET`  | `/api/auth/verify-refresh-token`  | Verifies if a refresh token is valid |

---

## 5. Token Management

### 5.1 Database Storage
- A **`Token` entity** records:
    - `token`: The JWT string.
    - `user`: The associated `UserEntity`.
    - `tokenType`: `ACCESS` or `REFRESH`.
    - `revoked`: Boolean to mark if the token is invalid.
    - `expired`: Boolean to mark if the token is expired.

### 5.2 Logout & Token Invalidation
- **Logout Endpoints**
    - Invalidate the user’s tokens by setting `revoked = true` or `expired = true`.
    - `SecurityContextHolder.clearContext()` ensures the user is no longer authenticated.
- **Password Change**
    - All existing tokens for that user are also invalidated to force a fresh login.

### 5.3 Refresh Token Requests
- **`POST /api/auth/refresh-token`**
    - Receives the refresh token (often in `Authorization` header or a secure cookie).
    - Checks the token’s validity and revocation status in the repository.
    - If valid, issues a new access token (and optionally a new refresh token).

---

## 6. Security Features

### 6.1 Password Encryption
- **BCrypt** is used to securely hash and salt passwords.
- Mitigates brute-force attacks due to slow hash computation.

### 6.2 CSRF Protection
- **CSRF protection is disabled** for stateless JWT-based APIs.
- All endpoints require valid JWT for state-changing operations, acting as a natural CSRF guard.

### 6.3 Logout & Token Invalidation
- On **logout** or **password change**, the system calls `invalidateAllUserTokens(user)`:
    - Marks each token as expired or revoked in the database.
    - Clears the security context if currently logged in.

### 6.4 Security Headers
- **`X-XSS-Protection`**: Blocks detected cross-site scripting.
- **`X-Frame-Options: DENY`**: Prevents clickjacking by disallowing iframes.
- **`Content-Security-Policy`**: Restricts sources for scripts, styles, etc.

---

## 7. Security Best Practices Followed
- ✅ **Strong password hashing (BCrypt)**
- ✅ **JWT expiration & rotation**
- ✅ **Role-based access control (RBAC)**
- ✅ **Token blacklisting upon logout**
- ✅ **Security logging & monitoring**

---

## 8. Future Security Enhancements

- **OAuth 2.0 Integration**
    - Allow third-party logins (e.g., Google, GitHub) for convenience.
- **Multi-Factor Authentication (MFA)**
    - Add TOTP or SMS-based second factor, especially for admin roles.
- **API Rate Limiting**
    - Mitigates brute-force login attempts and DoS attacks.
- **SIEM Integration**
    - Forward security events to a centralized log management system for real-time monitoring and alerting.

---

## 9. Example Security Workflow

1. **User Login**
    - `POST /api/auth/login`
    - User provides credentials.
    - Response contains **access token** & **refresh token**.

2. **Accessing a Protected Endpoint**
    - `GET /api/attendance/student/{id}`
    - Add `Authorization: Bearer <access_token>` header.
    - If valid, the request proceeds; otherwise, `401 Unauthorized`.

3. **Token Refresh**
    - `POST /api/auth/refresh-token`
    - Provide the **refresh token**.
    - System issues a new access token.

4. **Logout**
    - `POST /api/auth/logout` or `POST /api/auth/logout/all`
    - The system invalidates tokens, and the user’s session is ended.

---

## 10. References & Additional Reading
- [Spring Security Reference](https://docs.spring.io/spring-security/site/docs/current/reference/html5/)
- [JWT.io](https://jwt.io/)
- [BCrypt Documentation](https://www.mindrot.org/projects/jBCrypt/)

---

🔙 **[Back to README](../README.md)**

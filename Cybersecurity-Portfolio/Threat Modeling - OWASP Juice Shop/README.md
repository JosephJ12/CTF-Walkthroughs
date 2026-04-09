# Threat Modeling / Risk Assessment of OWASP Juice Shop

## Summary
The purpose of this project is to do a simulated threat modeling of the popular, intentionally vulnerable web app OWASP Juice Shop. We will focus on assessing 3 features of the application:

1. The login authentication flow
2. The product search feature
3. The profile image upload feature

For each feature, we will do the following:

1. Create a System Architecture Overview
2. Outline a Data Flow Diagram
3. Create a Trust Boundary Diagram
4. Analyze threats following the STRIDE framework
5. Outline a risk register
6. Conduct a gap analysis
7. Map risks to ISO 27001 and NIST SP 800-53 controls

Let's get started!

## Table of Contents
* [Login Authentication](#auth)
* [Inventory Search](#search)
* [Profile Image Uplaod](#upload)


## 1. Login Authentication Threat Model

### Objective
Assess the authentication feature for security risks related to credential submission, token issuance, session trust, and authorization dependencies.

### Scope
- Login submission
- Backend credential validation
- Token issuance and return to client
- Authentication-related trust boundaries

### Process
- Architecture overview
- Data flow diagram
- Trust boundary diagram
- STRIDE analysis
- Gap analysis
- Risk register
- NIST / ISO mapping

### 1.1 Architecture Overview

```mermaid
flowchart LR
    U[User Browser]
    FE[Angular Login Page]
    API[POST /rest/user/login/]
    BE["Express Login() Function"]
    DB[SQLite Users Table]
    JWT[JWT Auth]

    U --> FE
    FE --> API
    API --> BE
    BE --> DB
    DB --> APP
    BE --> JWT
    JWT --> FE
```

### 1.2 Data Flow Diagram

```mermaid
flowchart LR
    A[User enters credentials to login form]
    B[Frontend sends POST login request to backend via API]
    C[Backend queries database and validates credentials]
    D[Upon successful validation, backend returns auth token to frontend]
    E[Frontend stores auth token and uses it for later requests]

    A --> B --> C --> D --> E
```

### 1.3 Trust Boundary Diagram

```mermaid
flowchart LR
    subgraph TB1["TB1: Untrusted User Boundary"]
        U[User Browser]
    end

    subgraph TB2["TB2: Frontend Trust Boundary"]
        FE[Angular Frontend]
    end

    subgraph TB3["TB3: Backend Trust Boundary"]
        APP[Express Backend]
        JWT[JWT Auth]
    end

    subgraph TB4["TB4: Database Trust Boundary"]
        DB[(SQLite DB Users Table)]
    end

    U --> FE
    FE --> APP
    APP --> JWT
    APP --> DB
```

### 1.4 STRIDE Analysis

| Threat | Example | Risk |
|------|--------|------|
| Spoofing | Brute force login | High |
| Tampering | JWT token forgery | Critical |
| Repudiation | Incomplete login audit trail | Medium |
| Info Disclosure | Verbose login error responses | High |
| DoS | Login request flooding | Medium |
| EoP | Token hijacking | Critical |

### 1.5 Risk Register

| Risk | STRIDE | Likelihood | Impact | Risk Level | Mitigation |
|---|---|---|---|---|---|
| Brute force login | Spoofing | High | High | Critical | Rate limiting, account lockout policy, MFA |
| JWT Token forgery | Tampering | Medium | Critical | Critical | Signature verification, strict token validation |
| No login audit logs | Repudiation | Medium | Medium | Medium | Implement auth logging |
| Verbose login error responses | Information Disclosure | High | Medium | High | Generic error messages |
| Token Hijacking | Elevation of Privilege / Spoofing | Medium | High | High | Secure token storage and expiration |


### 1.6 Gap Analysis

| Expected Control | Status | Gap | Impact | Recommended Remediation |
|---|---|---|---|---|
| Brute force login protection | Not evident | Rate limiting on login attempts does not seem to be present within the scope. | Increases the likelihood of user impersonation, which may lead to complete account compromise and sensitive information disclosure. | Lockout policy on failed login attempts and enforcing strong password policy upon account creation. | 
| Token signing | Present within scope | Auth tokens should be signed to prevent tampering or impersonation. | Absence of token signing may allow attackers to craft or modify their own token to elevate privileges or impersonate another user. | Implement token signing and validation. |
| Logging login activity | Not evident | Auditing login attempts does not seem evident within the scope. | Increases likelihood of repudiation without audit logs. | Securely store logs on web server log files. | 
| Generic error messages upon failed login | Requires validation | Generic error messages should be given for all failed login cases to prevent attackers from enumerating valid users from them. | Increases the likelihood of user enumeration, which may be later used for further attacks. | Implement generic error messages for all login errors. | 
| HttpOnly and Token Expiration Implementation | Not evident | Setting the HttpOnly flag and proper token expiration time does not seem present in scope. | Increases risk of token hijacking, leading to possible user impersonation and even elevated privileges. | Set the HttpOnly flag for JWT auth token cookie and set token expiration time. |


### 1.7 Compliance Mapping

| Risk | NIST SP 800-53 | ISO 27001 |
|---|---|---|
| Brute-force / credential stuffing | AC-7, IA-2 | A.9 |
| JWT Token forgery | IA-5, SC-23 | A.10, A.9 |
| Incomplete login audit trail | AU-2, AU-12 | A.12.4 |
| Verbose login error responses | SI-11 | A.14 |
| Token Hijacking | IA-5, AC-6 | A.9, A.10 |

=========

## 2. Product Search Threat Model

### Objective
Assess the product search and catalog discovery feature for risks related to user-controlled input, data retrieval, query handling, and result output.

### Scope
- Search term input submission
- Backend search processing and input handling
- Product lookup and result rendering

### Process
- Architecture overview
- Data flow diagram
- Trust boundary diagram
- STRIDE analysis
- Gap analysis
- Risk register
- NIST / ISO mapping

### 2.1 Architecture Overview

```mermaid
flowchart LR
    U[User Browser]
    FE[Angular Search Page]
    API[GET /rest/products/search/]
    BE["Express searchProducts() Function"]
    DB[SQLite Products Table]

    U --> FE
    FE --> API
    API --> BE
    BE --> DB
    DB --> FE
```

### 2.2 Data Flow Diagram

```mermaid

```

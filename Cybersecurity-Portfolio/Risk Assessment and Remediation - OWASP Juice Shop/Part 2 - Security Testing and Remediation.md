# Dynamic Application Security Testing (DAST) + Remediation -- OWASP Juice Shop

## Introduction

Now that we have outlined and assessed the risks for the login authentication flow and product search feature, we will verify the identified risks via Dynamic Application Security Testing (DAST). After confirming the risk is present through manual exploitation, we will modify the code base to remediate the vulnerability and confirm the fix by retesting the feature.

## 1. Login Authentication Testing

To recap from part 1, these are the risks that we will verify:

| Risk ID | Risk | STRIDE | Likelihood | Impact | Mitigation |
|---|---|---|---|---|---|
| AUTH-01 | Brute force login | Spoofing | High | High | Rate limiting, account lockout policy, MFA |
| AUTH-02 | JWT Token forgery | Tampering | Medium | Critical | Signature verification, strict token validation |
| AUTH-03 | No login audit logs | Repudiation | Medium | Medium | Implement auth logging |
| AUTH-04 | Verbose login error responses | Information Disclosure | High | Medium | Generic error messages |
| AUTH-05 | SQL Injection | Elevation of Privilege / Spoofing | High | Critical | WAF, parameterized queries, input sanitization |

### Scope 

The following will be our scope for this assessment:
- Login submission
- Backend credential validation
- Token issuance and return to client
- Authentication-related trust boundaries

Since the scope does not include account registration, we will create a test account with the following credentials to retrieve a valid JWT auth token to test AUTH02, `test@test.com:test123`

Also, we will skip testing AUTH-03 since this is a test application running on debug mode hosted on a Kali Linux VM. We will assume AUTH-03 was validated and properly remediated for our purposes.

### AUTH-01: Brute Force Login

<img width="1179" height="760" alt="image" src="https://github.com/user-attachments/assets/f8f47560-b17f-4a53-b379-49e16420e839" />


### AUTH-02: JWT Token Forgery


### AUTH-04: Verbose Login Error Responses


### AUTH-05: SQL Injection


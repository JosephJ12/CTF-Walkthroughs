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
| AUTH-04 | Verbose login error responses | Information Disclosure | Medium | Medium | Generic error messages |
| AUTH-05 | SQL Injection | Elevation of Privilege / Spoofing | High | Critical | WAF, parameterized queries, input sanitization |

### Scope 

The following will be our scope for this assessment:
- Login submission
- Backend credential validation
- Token issuance and return to client
- Authentication-related trust boundaries

Since the scope does not include account registration, we will create a test account with the following credentials to retrieve a valid JWT auth token to test AUTH02, `test@test.com:test123`

Also, we will skip testing AUTH-03 since this is a test application running on debug mode hosted on a Kali Linux VM. We will assume AUTH-03 was validated and properly remediated for our purposes.

### Validating AUTH-01: Brute Force Login

<img width="1179" height="760" alt="image" src="https://github.com/user-attachments/assets/f8f47560-b17f-4a53-b379-49e16420e839" />


### Validating AUTH-02: JWT Token Forgery


### Validating AUTH-04: Verbose Login Error Responses

NOTE: Our valid credentials are: `test@test.com:test123`

We will test the login form and check the error responses for 3 cases:
- incorrect email AND password
- correct email ONLY
- correct password ONLY

These are the steps we will take to test for AUTH-04:

1. Go to the `/#/login` endpoint.

<img width="1179" height="760" alt="image" src="https://github.com/user-attachments/assets/9697e6ac-b1f9-4349-ba96-215cc8d6d4a8" />

2. Enter `juice@juice.shop` for the username and `juice123` for the password and press Log in.

<img width="453" height="640" alt="image" src="https://github.com/user-attachments/assets/1609506c-533d-4743-a55c-5e1313e11ef1" />

3. Check the error message, which returns `Invalid email or password.` This is a generic error message so case 1 passes.

<img width="454" height="659" alt="image" src="https://github.com/user-attachments/assets/32304699-5288-4fb4-9a49-242cd41aec8a" />

4. Next, try case 2. Enter `test@test.com` for the username and keep the password as `juice123`. Then, press Log in.

<img width="446" height="645" alt="image" src="https://github.com/user-attachments/assets/22833a23-9c38-4724-86d0-1fcc39368990" />

5. We confirm the error response is generic for case 2. Therefore, we move on to the last case. Enter `juice@juice.shop` as the username and `test123` for the password and press Log in.

<img width="448" height="648" alt="image" src="https://github.com/user-attachments/assets/7bc8c501-5ad4-4636-8fef-139c3f808407" />

6. Checking the error response, it is the same as the rest. Therefore, we have confirmed that risk AUTH-04 is not present in the scope.

<img width="454" height="660" alt="image" src="https://github.com/user-attachments/assets/80544991-dc1b-40b7-87f5-fb75dc4ef376" />

### Validating AUTH-05: SQL Injection


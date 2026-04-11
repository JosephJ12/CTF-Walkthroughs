# Dynamic Application Security Testing (DAST) + Remediation

## INTRODUCTION

Now that we have outlined and assessed the risks for the login authentication flow and product search feature, we will verify the identified risks via Dynamic Application Security Testing (DAST). After confirming the risk is present through actual exploitation, we will modify the code base to remediate the vulnerability and confirm the fix by retesting the feature.

```
Test Date: 4/9/2026
Tested By: Joseph Jung
Node.js version: 22.22.1
Juice Shop version: 19.2.1
```

## 1. Login Authentication Testing

To recap from part 1, these are the risks that we will verify:

| Risk ID | Risk | STRIDE | Likelihood | Impact | Mitigation |
|---|---|---|---|---|---|
| AUTH-01 | Brute force login | Spoofing | High | High | Rate limiting, account lockout policy, MFA |
| AUTH-02 | JWT Token forgery | Tampering | Medium | Critical | Signature verification, strict token validation |
| AUTH-03 | No login audit logs | Repudiation | Medium | Medium | Implement auth logging |
| AUTH-04 | Verbose login error responses | Information Disclosure | Medium | Medium | Generic error messages |
| AUTH-05 | SQL Injection | Elevation of Privilege / Spoofing | High | Critical | WAF, parameterized queries, input sanitization |

Of these, we will test and remediate 3: AUTH-01, AUTH-04, and AUTH-05.

### SCOPE 

The following will be our scope for this assessment:
- Login submission
- Backend credential validation
- Token issuance and return to client
- Authentication-related trust boundaries

Since the scope does not include account registration, we will create a test account with the following credentials, `test@test.com:test123`

### SECURITY TESTING

-------------

#### AUTH-01: Brute Force Login

We will test the application against brute force attacks. A successful brute force attack can be looked at from 2 perspectives: the application side and user side. Misconfigurations from the application side include weak password requirements, not enforcing MFA, and no lockout policy whereas oversights from the user include choosing to reuse passwords or an easy to guess password. 

Since the user side is not in scope, we will focus on only the application misconfigurations for the purposes of this test.

NOTE: Our valid credentials are: `test@test.com:test123`

Tools Required:
- Burp Suite (for request capture)
  - Download link: https://portswigger.net/burp/documentation/desktop/getting-started/download-and-install
- Ffuf (for automating brute force attack)
  - Installation guide: https://github.com/ffuf/ffuf
- Seclists: 10k most common (password list)
  - Github Repo: https://github.com/danielmiessler/SecLists/blob/master/Passwords/Common-Credentials/10k-most-common.txt

1. Go to the `/#/login` endpoint.

<img width="1179" height="760" alt="image" src="https://github.com/user-attachments/assets/f8f47560-b17f-4a53-b379-49e16420e839" />

2. Turn on Burp Suite, making sure that it is capturing the application traffic.

<img width="1594" height="892" alt="image" src="https://github.com/user-attachments/assets/ceec2fa1-2fad-4e0a-9e7b-78b81ecfef07" />

3. Capture a login request by entering `test@test.com` as the username and `test123` as the password and clicking login. This will give us a POST request to the `/rest/user/login` endpoint and the parameters for us to brute force.

<img width="1093" height="700" alt="image" src="https://github.com/user-attachments/assets/c1fbd63a-b023-4b8c-9b6c-9e2aeef6760c" />

4. Open a terminal and use ffuf to conduct brute force attack on the Juice Shop login using the following Bash command:

`ffuf -w /usr/share/wordlists/seclists/Passwords/Common-Credentials/10k-most-common.txt -X POST -d '{"email":"test@test.com","password":"FUZZ"}' -u http://localhost:3000/rest/user/login -H "Content-Type: application/json" -fc 401`

<img width="1265" height="495" alt="image" src="https://github.com/user-attachments/assets/5c32f484-e2d7-4050-a830-d6ecf311b288" />

Conclusion: We confirm that the application is vulnerable to brute force attacks and requires remediation.

-----------

#### AUTH-04: Verbose Login Error Responses

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

5. We confirm the error response is generic for case 2. Therefore, we move on to the last case. 

<img width="455" height="649" alt="image" src="https://github.com/user-attachments/assets/514ea738-7108-4c7e-95a6-7753c821d831" />

6. Enter `juice@juice.shop` as the username and `test123` for the password and press Log in.

<img width="448" height="648" alt="image" src="https://github.com/user-attachments/assets/7bc8c501-5ad4-4636-8fef-139c3f808407" />

7. Checking the error response, it is the same as the rest. Therefore, we have confirmed that risk AUTH-04 is not present in the scope.

<img width="454" height="660" alt="image" src="https://github.com/user-attachments/assets/80544991-dc1b-40b7-87f5-fb75dc4ef376" />

Conclusion: Risk AUTH-04 is not present within the scope and does not require remediation.

-----------

#### AUTH-05: SQL Injection

We will test the login form for risk AUTH-05L SQL Injection. We will go about testing in 2 ways: manual and automated. The manual way of testing will be the tester directly inputting malicious payloads to test for dangerous behavior. For the automated testing, we will use the popular open-source tool, `SQL Map`.

NOTE: Our valid credentials are: `test@test.com:test123`

Tools Required:
- Burp Suite (for request capture)
  - Download link: https://portswigger.net/burp/documentation/desktop/getting-started/download-and-install
- SQL Map (for automated testing)
  - Github Link: https://github.com/sqlmapproject/sqlmap

We will start off with the manual testing first and then automated testing.

1. Go to the login page on the `/#/login` endpoint.

<img width="997" height="828" alt="image" src="https://github.com/user-attachments/assets/a1482148-1c17-4b41-bfa7-28f30d51de62" />

2. First, we will see if we can login with only a valid email and not a valid password. We will enter `test@test.com' --` in the username field and any random password. In this case, we'll enter `asd` as our password and click Log in.

<img width="452" height="630" alt="image" src="https://github.com/user-attachments/assets/d349da90-e50e-46c4-8bea-24fde88702d6" />

3. We do indeed successfully login as the `test@test.com` user.

<img width="1484" height="642" alt="image" src="https://github.com/user-attachments/assets/ef3ddf8a-fd35-47c0-9fce-12605041feea" />

4. Lets Log Out and go back to the login page from step 1. This time, we'll attempt to bypass authentication by neither passing in a valid email or password. Let's enter `' or 1==1 --` as the email and `asd` as the password and attempt to log in.

<img width="445" height="541" alt="image" src="https://github.com/user-attachments/assets/7dc3f7f6-f32f-466a-87ae-48acecfac6bb" />

5. We're logged in again and this time, we log into the admin account.

<img width="1484" height="646" alt="image" src="https://github.com/user-attachments/assets/dcfbc7e8-79f2-45c0-a329-22207d33b247" />

6. Since we've confirmed that risk AUTH-05 exists in scope via manual testing, we'll test further with  automated testing using SQL Map. We'll first capture a login request on BurpSuite by logging in with the credentials `test@test.com:test123`.

<img width="629" height="933" alt="image" src="https://github.com/user-attachments/assets/67ebd3e1-b112-4066-b970-86f37e000af3" />

7. Right-click on the Request and click on the `Copy to File` option:

<img width="622" height="623" alt="image" src="https://github.com/user-attachments/assets/9f92c49e-075c-4205-8241-7c213d78c6ec" />

8. Navigate to the folder where you want to place the file and type `req.txt` in the File Name: and click on `Save`.

<img width="945" height="737" alt="image" src="https://github.com/user-attachments/assets/da8cf59f-c0fa-4c0f-8575-a1b2f7178cb4" />

9. Open a new Bash terminal and run the following command:

`sqlmap -r ~/risk_assessment/req.txt --batch --level=4 --risk=2 --ignore-code=401 --tables`

We're able to get a list of all the tables and confirm the login form is vulnerable to automated SQL Injection attacks.

<img width="1002" height="1176" alt="image" src="https://github.com/user-attachments/assets/c6f6abf0-54fa-47d0-967e-95424f081465" />

Conclusion: Risk AUTH-05 is present and validated via manual and automated testing.

-------------

### RISK REMEDIATION

Now, we'll remediate the 2 vulnerabilities we've validated and retest to make sure the vulnerability is fixed. To do this, we'll make direct modifications to the code base on our local machine. There are 2 risks that must be remediated: AUTH-01 and AUTH-05. 

Here is an appendix with links and references about the vulnerability and recommended remediations.

Risk Appendix
|--|--|--|--|
|


#### AUTH-01: Brute Force Login



#### AUTH-05: SQL Injection


# Dynamic Application Security Testing (DAST) + Remediation

Now that we have outlined and assessed the risks for the login authentication flow, we will verify the identified risks via Dynamic Application Security Testing (DAST). After confirming the risk is present through actual exploitation, we will modify the code base to remediate the vulnerability and confirm the fix by retesting the feature. We will do DAST only for the authentication flow, and save the product search feature for the SAST part of the project.

```
Test Date: 4/9/2026
Tested By: Joseph Jung
Node.js version: 22.22.1
Juice Shop version: 19.2.1
```

# Login Authentication Testing

  To recap from part 1, these are the risks that we have identified:

| Risk ID | Risk | STRIDE | Likelihood | Impact | Mitigation |
|---|---|---|---|---|---|
| AUTH-01 | Brute force login | Spoofing | High | High | Rate limiting, account lockout policy, MFA |
| AUTH-02 | JWT Token forgery | Tampering | Medium | Critical | Signature verification, strict token validation |
| AUTH-03 | No login audit logs | Repudiation | Medium | Medium | Implement auth logging |
| AUTH-04 | Verbose login error responses | Information Disclosure | Medium | Medium | Generic error messages |
| AUTH-05 | SQL Injection | Elevation of Privilege / Spoofing | High | Critical | WAF, parameterized queries, input sanitization |

Of these, we will test and remediate 3: AUTH-01, AUTH-04, and AUTH-05.

## SCOPE 

The following will be our scope for this assessment:
- Login submission
- Backend credential validation
- Token issuance and return to client
- Authentication-related trust boundaries

Since the scope does not include account registration, we will create a test account with the following credentials, `test@test.com:test123`

# DYNAMIC SECURITY TESTING

## AUTH-01: Brute Force Login

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


## AUTH-04: Verbose Login Error Responses

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


## AUTH-05: SQL Injection

  We will test the login form for risk AUTH-05 SQL Injection. We will go about testing in 2 ways: manual and automated. The manual way of testing will be the tester directly inputting malicious payloads to test for dangerous behavior. For the automated testing, we will use the popular open-source tool, `SQL Map`.

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

`sqlmap -r [PATH_TO_REQUEST_FILE]/req.txt --batch --level=4 --risk=2 --ignore-code=401 --tables`

We're able to get a list of all the tables and confirm the login form is vulnerable to automated SQL Injection attacks.

<img width="1002" height="1176" alt="image" src="https://github.com/user-attachments/assets/c6f6abf0-54fa-47d0-967e-95424f081465" />

Conclusion: Risk AUTH-05 is present and validated via manual and automated testing.


# RISK REMEDIATION

  Now, we'll remediate the 2 vulnerabilities we've validated and retest to make sure the vulnerability is fixed. To do this, we'll make direct modifications to the code base on our local machine. There are 2 risks that must be remediated: AUTH-01 and AUTH-05. 

Here is an appendix with links and references about the vulnerability and recommended remediations.

Risk Appendix
| Risk ID | Risk | OWASP Top 10 | CWE Category | MITRE ATT&CK | NIST SP 800-53 Control | Recommended Remediation |
|--|--|--|--|--|--|--|
| AUTH-01 | Brute Force Login | A07:2021 – Identification and Authentication Failures | CWE-307 – Improper Restriction of Excessive Authentication Attempts, CWE-521 – Weak Password Requirements | T1110 – Brute Force | AC-7 – Unsuccessful Login Attempts | https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html#protect-against-automated-attacks |
| AUTH-05 | SQL Injection | A03:2021 – Injection | CWE-89 – Improper Neutralization of Special Elements used in an SQL Command | T1190 – Exploit Public-Facing Application | SI-10 – Information Input Validation | https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html |

For all code editing tasks, we will use Virtual Studio Code, or VS Code: https://code.visualstudio.com/download

## AUTH-01: Brute Force Login

  For production applications, we recommend implementing a defense in depth approach to defend against brute force attacks, adding multiple security controls. Controls such as rate limiting, MFA, suspicious IP blocking, and enforcing strong passwords upon account creation would add layers of security needed to successfully mitigate against brute force attacks. 

For the purposes of this case study, we will focus on implementing one: rate limiting. We will set 2 conditions for rate limiting.
- First, each IP will be allowed 10 failed login attempts every 15 minutes.
- Second, each IP will be allowed 5 failed login attempts for 1 account every 15 minutes.

The reason for 2 separate rate limiters is that the first helps mitigate password spray attacks, where a single password is tried for multiple accounts. The second rate limiter helps defend against traditional brute force attacks- multiple password guesses for a single user.

1. Open the `server.ts` file in the app root folder on VS Code. Then, we'll locate the piece of code that configures the login API endpoint on line 594.

2. Next, we'll add this code snippet on line 592:

```
  // standardize email format for account login attempt tracking
  function normalizeLoginIdentifier (value: unknown): string {
    if (typeof value !== 'string') return 'unknown'
    const normalized = value.trim().toLowerCase()
    return normalized.length > 0 ? normalized : 'unknown'
  }
  
  // lockout for 15 minutes after 10 failed login attempts from 1 IP
  const loginIpRateLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 10, // total login attempts per IP in window
    standardHeaders: true,
    legacyHeaders: false,
    skipSuccessfulRequests: true,
    message: {
      status: 'error',
      message: 'Too many login attempts from this IP address. Please try again later.\n'
    }
  })
  
  // lockout for 15 minutes after 5 failed login attempts from 1 IP for the same account
  const loginAccountIpRateLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 5, // attempts per email+IP pair in window
    standardHeaders: true,
    legacyHeaders: false,
    skipSuccessfulRequests: true,
    keyGenerator: (req) => {
      const email = normalizeLoginIdentifier(req.body?.email)
      const ip = req.ip || 'unknown'
      return `${email}:${ip}`
    },
    message: {
      status: 'error',
      message: 'Too many login attempts for this account. Please try again later.\n'
    }
  })
```

3. We'll also modify line 594 to the following:

`app.post('/rest/user/login', loginIpRateLimiter, loginAccountIpRateLimiter, login())`

The `server.ts` file should now look like the screenshot below.

<img width="852" height="841" alt="image" src="https://github.com/user-attachments/assets/9e2a2eb9-d65c-44ae-9900-24c77096a07e" />

4. Save the file and run these 2 commands in the app root directory.

```
npm run build:server
npm start
```

5. While the app is building, we'll create a custom test Bash script to verify the control is in place and working. Create a new file and add this bash script:

```
#!/bin/bash

for i in {1..6}; do
  curl -i -X POST http://localhost:3000/rest/user/login -H "Content-Type: application/json" -d '{"email":"test@test.com,","password":"asd"}'
done
```

6. Save the file as `test_login.sh` and give the file execute permissions by running this command:

`chmod u+x test_login.sh`

7. Run the script:

`./test_login.sh`

8. We confirm the IP + account rate limiter is working since the application returns the error code `429: Too Many Requests` on the 6th try. Now, we'll test the IP rate limiter. First, we'll open the `test_login.sh` on a text editor and edit the email portion to a different email:

<img width="1140" height="107" alt="image" src="https://github.com/user-attachments/assets/3f2c01d4-a32d-4f26-81c0-ebb41613c83f" />

9. Save the file and rerun the script.

`./test_login.sh`

<img width="1081" height="1220" alt="image" src="https://github.com/user-attachments/assets/8e088590-d47d-4c6a-917b-b71e7483bd29" />

10. This time, we get the 429 error stating that there were too many login attempts from this IP address on the 11th and 12th requests. We validate that our rate limiting is active and working as intended!

We have successfully mitigated risk `AUTH-01: Brute Force Login`.


## AUTH-05: SQL Injection

  We will implement 2 mitigations to the code base. The first is refactoring the SQL query the backend uses to validate credentials to use parameterized queries. The other will be sanitizing user input using a whitelist filter. Let's get started!

## Parameterized Query Refactoring

1. To make changes to the local code base, first we'll navigate to the folder where it is stored and open the file `routes/login.ts` using a code editor. For this demo, I'll be using Visual Studio Code.

<img width="1806" height="274" alt="image" src="https://github.com/user-attachments/assets/0e39942c-d46d-4a68-a113-b29129e89139" />

2. In the screenshot above, the vulnerable code is on line 34. Let's comment out line 34 and add the following piece of code:

```
// ORIGINAL SQLI VULNERABLE CODE
    //models.sequelize.query(`SELECT * FROM Users WHERE email = '${req.body.email || ''}' AND password = '${security.hash(req.body.password || '')}' AND deletedAt IS NULL`, { model: UserModel, plain: true }) // vuln-code-snippet vuln-line loginAdminChallenge loginBenderChallenge loginJimChallenge
        
    // CHANGE TO PARAMETERIZED QUERY
    const email = req.body.email || '';
    const passwordHash = security.hash(req.body.password || '');
    
    models.sequelize.query(
      `
      SELECT * 
      FROM Users 
      WHERE email = $email 
      AND password = $password 
      AND deletedAt IS NULL
      `,
      {
        bind: {
            email: email,
            password: passwordHash,
        },
        model: UserModel,
        plain: true,
      })
```

It should now look something like this:

<img width="1823" height="560" alt="image" src="https://github.com/user-attachments/assets/5ea2bbbc-d6ea-4c7d-8382-3b086a6341a5" />

3. The key change is to take out the user input, `email` and `password` from the query and use binding placeholders instead. This will greatly mitigate SQL Injection attacks. We restart the app by running the following 2 commands:

```
npm run build:server
npm start
```

4. We must recreate our test user `test@test.com:test123`

5. Let's first test to see if valid credentials work. Enter `test@test.com:test123` into the login form.

<img width="440" height="624" alt="image" src="https://github.com/user-attachments/assets/b76681a8-df77-4968-913e-cc011b73112a" />

6. We confirm this successfully logs us in as intended.

<img width="1486" height="557" alt="image" src="https://github.com/user-attachments/assets/729a1179-2850-493d-8c35-438fe33af414" />

7. After logging out, we'll try the first payload from our DAST. Enter `test@test.com' --` as our email and `asd` as the password.

<img width="441" height="614" alt="image" src="https://github.com/user-attachments/assets/15a0b81d-7c32-4919-aa6f-e6b7e1d92611" />

8. This time, we get a login error message.

<img width="454" height="652" alt="image" src="https://github.com/user-attachments/assets/70e3f1f9-2001-458d-88a3-7e70dc4523d6" />

9. Great! Now let's try the other payload as well. Enter `' OR 1==1 --` as the email and `asd` as the password.

<img width="459" height="658" alt="image" src="https://github.com/user-attachments/assets/94c61597-b1b1-4073-987c-402e2942d2b1" />

10. This one fails as well! Now finally, we'll test using SQL Map. Open a terminal and run the following command:

`sqlmap -r [PATH_TO_REQUEST_FILE]/req.txt --batch --level=4 --risk=2 --ignore-code=401 --tables --flush-session`

11. SQL Map was unsuccessful in finding a SQL Injection vulnerability! 

<img width="1326" height="567" alt="image" src="https://github.com/user-attachments/assets/7fd6eae8-8836-4c2e-822e-e1134e528bdc" />

Conclusion: Successfully implemented parameterized query to mitigate against SQL Injection attacks in scope.

## How Do Parameterized Queries Work?

  So why do parameterized queries defend against SQL Injection? To answer this, we first need to understand how SQL Injection attacks work. Let's take our query above:

`SELECT * FROM Users WHERE email = '$(user_email)' AND password = '$(user_password)' AND deletedAt IS NULL`

If we input our payload `test@test.com' --`, then the query will become:

`SELECT * FROM Users WHERE email = 'test@test.com' --' AND password = '$(user_password)' AND deletedAt IS NULL`

The trailing single quote `'` will finish off the email WHERE condition and the `--` will comment out the rest of the query, turning it into:

`SELECT * FROM Users WHERE email = 'test@test.com' --`

Since we created a user with this email, it will automatically get this user without checking the password and log us in.

Now here's where parameterized queries come in. By using placeholders, every user input will not be treated as part of the SQL query but as data plugged in surrounded by double quotes. Therefore, the same payload of `test@test.com' --` will look like this in a parameterized query:

`SELECT * FROM Users WHERE email = '"test@test.com' --"' AND password = '"asd"' AND deletedAt IS NULL` 

Everything in the double quotes will be escaped and will not be treated as part of the SQL query. Therefore, attackers can not modify the SQL query for other purposes.

## Post DAST Gap Analysis

Let's update the gap analysis chart after DAST and remediation. 

| Risk ID | Risk | Expected Control | Status | Gap | Impact | Recommended Remediation |
|---|---|---|---|---|---|---|
| AUTH-01 | Brute force login | Brute force login protection | Patched and verified in scope | Rate limiting on login attempts is now present in scope by tracking failed login attempts by IP address and account name. | Greatly reduced the risk of unauthorized account access via brute force and password spray attacks. | Lockout policy on failed login attempts and enforcing strong password policy upon account creation. | 
| AUTH-04 | Verbose login error responses | Generic error messages upon failed login | Validated in scope | Generic error messages are given for all failed login cases to prevent attackers from enumerating further information from them. | Reduces the likelihood of user enumeration, which may be later used for further attacks. | Confirmed implementation of generic error messages for all login errors in scope. | 
| AUTH-05 | SQL Injection | Backend uses parameterized queries to query database | Patched and retested in scope | The `login()` function now utilizes parameterized queries when verifying login credentials in scope. | Reduced the risk of authentication bypass and backend data disclosure via SQL Injection attacks. | Parameterized queries are implemented when querying the database and have been tested against SQL Injection attacks. |

Through DAST, we have successfully assessed the impact and verified the mitigation of risks AUTH-01, AUTH-04, and AUTH-05.

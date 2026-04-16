# SAST Implementation to CI/CD Pipeline

So far, we've done a complete threat modeling and risk assessment for the login and product search features of the Juice Shop app. Then, we verfied the vulnerabilities identified for the login feature using DAST and remediated them. Now, we'll do Static Application Security Testing, or SAST, for the product search feature. 

We will do this by implementing the popular open source SAST tool, `Semgrep` and incorporate it into the CI/CD workflow. This will automate secure code reviews and encourage secure coding practices. Before we do that, let's refresh on our testing scope and risks assessed.

## Risk Register

| Risk ID | Risk | STRIDE | Likelihood | Impact | Mitigation |
|---|---|---|---|---|---|
| SEARCH-01 | SQL Injection - Product Data Exposure | Information Disclosure | High | Medium | Implement WAF, sanitize input, and implement parameterized queries. |
| SEARCH-02 | SQL Injection - User Data Enumeration | Information Disclosure, Spoofing, Escalation of Privileges | High | Critical | Implement WAF, sanitize input, and implement parameterized queries. | 
| SEARCH-03 | Unauthenticated users can search products | Repudiation | High | Low |  Allow only authenticated users to search or set user tracking cookie. |
| SEARCH-04 | Expensive or excessive search queries | DoS | High | High | Block suspicious IP and limit search query rates. |

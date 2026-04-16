# SAST Implementation to CI/CD Pipeline

So far, we've done a complete threat modeling and risk assessment for the login and product search features of the Juice Shop app. Then, we verfied the vulnerabilities identified for the login feature using DAST and remediated them. Now, we'll do Static Application Security Testing, or SAST, for the product search feature. 

We will do this by implementing the popular open source SAST tool, `Semgrep` and incorporate it into the CI/CD workflow. This will automate secure code reviews and encourage secure coding practices. Before we do that, let's refresh on our testing scope and risks assessed.


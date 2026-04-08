# Threat Modeling / Risk Assessment of OWASP Juice Shop

## Summary
The purpose of this project is to do a simulated threat modeling of the popular, intentionally vulnerable web app OWASP Juice Shop. After creating a system architecture overview, a data flow diagram, and trust boundary diagram of the application, we will focus on assessing 3 features of the application:

1. The login authentication flow
2. The inventory search feature
3. The profile image upload feature

For each feature, we will do the following:

1. Analyze threats following the STRIDE framework
2. Outline a risk register
3. Conduct a gap analysis
4. Map risks to ISO 27001 and NIST SP 800-53 controls

Let's get started!

## Table of Contents
* [System Architecture Overview](#sao)
* [Data Flow Diagram](#dfd)
* [Trust Boundary Diagram](#tbd)
* [Login Authentication](#auth)
* [Inventory Search](#search)
* [Profile Image Uplaod](#upload)

## System Architecture Overview 


## Data Flow Diagram


## Trust Boundary Diagram


## Login Authentication Threat Model

#### Objective

Analyze authentication flow to identify security risks related to login, session handling, and token management.

#### Scope

* /rest/user/login
* JWT authentication flow

#### Key Risks

* Credential stuffing
* JWT tampering
* Session hijacking

#### Outcome

* STRIDE threat model
* Risk register
* NIST & ISO mapping


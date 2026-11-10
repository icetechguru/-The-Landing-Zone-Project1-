
# Close Brothers — AWS Account Security Hardening

> Securing a brand-new AWS account from day one: identity protection, spending guardrails, full audit logging — and a server with **no SSH door at all**.

---

## 📌 Project Overview

**Close Brothers** has just opened its AWS account. Leadership raised two concerns:

1. **Unauthorized spending** — an unsecured account can run up costs fast.
2. **Compromised root credentials** — the root user is the master key to everything.

This project implements the foundational security controls for the account end-to-end: the root account is locked down and retired from daily use, a named admin user handles day-to-day work, budget alerts watch spending, CloudTrail records all activity, a server is deployed with **zero inbound SSH exposure**, and storage is created private and encrypted by default.

## ARCHITECTURAL DIAGRAM

<img width="1024" height="413" alt="WhatsApp Image 2026-11-10 at 2 14 32 PM" src="https://github.com/user-attachments/assets/d6171092-4d27-44b8-8217-0db1bfc0ea15" />




| Detail | Value |
|---|---|
| **Company** | Close Brothers |
| **Environment** | AWS (single account, production-style setup) |
| **Region** | US East (Ohio) — `us-east-2` |
| **Status** | ✅ Complete |




## 🎯 Objectives

- Enable **MFA on the root account** and stop using root for daily work
- Set up **budget alerts** to catch unexpected spending early
- Enable **CloudTrail** for a full audit trail of account activity
- Launch an **EC2 instance with no inbound SSH**, accessed via **AWS Systems Manager**
- Create an **S3 bucket** with **Block Public Access** and **encryption** enabled

---

## 🏗️ What Was Built

| # | Control | Implementation |
|---|---|---|
| 1 | **Root MFA** | Virtual MFA (authenticator app) enabled on the root user; root is now break-glass only |
| 2 | **IAM admin access** | Group **Admin** (`AdministratorAccess`) + named user **Admin-Me** with console sign-in and forced password reset at first login |
| 3 | **Budget alerts** | **My Monthly Cost Budget** — $10/month with email alerts at **85% actual**, **100% actual**, and **100% forecasted** spend |
| 4 | **CloudTrail** | Trail **`management-events`** — **multi-region**, logging management events to a dedicated S3 log bucket |
| 5 | **EC2 (no SSH)** | **`Closebrothers-server`** — Amazon Linux 2023, `t3.micro`. **No key pair created.** IAM role **EC2-SSM-Role** (`AmazonSSMManagedInstanceCore`) attached. Access exclusively via **SSM Session Manager** |
| 6 | **S3 storage** | Bucket **`clear-brother-bucket`** — **Block Public Access: ON**, **default encryption: SSE-S3** |

### How access to the server works (no SSH)

```text
Admin (browser, IAM login)
        │
        ▼
AWS Systems Manager ──► Session Manager ──► EC2 instance
        (IAM-authenticated)                 (agent connects OUT to AWS —
                                             no inbound ports open)
```

There is no port 22, no key pair, and no inbound rule to attack. The instance's SSM agent initiates an **outbound** connection to AWS, and every session is authenticated by IAM and logged.

---

## ✅ Account Security Baseline Checklist
<img width="1365" height="727" alt="Screenshot 2026-11-08 204906" src="https://github.com/user-attachments/assets/cb312100-3f51-435d-b131-388d98f1cf94" />
### Identity & Access
- [x] MFA enabled on the **root** account
- [x] Root account **retired from daily use** (break-glass only)
- [x] IAM **group** used for permissions (`Admin` → `AdministratorAccess`)
- [x] **Named** IAM admin user for daily work — no shared credentials
- [x] **Forced password reset** at first sign-in for the new user
- [ ] MFA enrolled for the admin IAM user *(in progress)*



### Cost Guardrails

<img width="1349" height="693" alt="Screenshot 2026-11-08 212812" src="https://github.com/user-attachments/assets/4f3e7279-9b68-4350-a59c-70d0e47bdbd6" />


- [x] Monthly **cost budget** in place
- [x] Email alerts at **85% / 100% actual** and **100% forecasted** spend

### Logging & Auditing
<img width="1355" height="648" alt="Screenshot 2026-11-08 213725" src="https://github.com/user-attachments/assets/9a7bcf53-05a4-4fb6-ad19-b9a968877f9b" />

- [x] **CloudTrail** enabled as a **multi-region** trail
- [x] Trail logs delivered to a **dedicated log bucket**

### Compute

<img width="1365" height="667" alt="Screenshot 2026-11-08 224415" src="https://github.com/user-attachments/assets/1ab5b7d9-dc5f-4368-9bcc-830b0b86723c" />


- [x] EC2 instance launched with **no key pair**
- [x] **No inbound SSH (port 22)** in the security group
- [x] Server access via **SSM Session Manager** only (IAM-authenticated)
- [x] **IAM role** attached to the instance — no credentials stored on the server

### Storage

<img width="1357" height="656" alt="Screenshot 2026-11-08 231211" src="https://github.com/user-attachments/assets/c34b23ad-71fc-4fb3-914a-71bdc452fc5b" />


- [x] **Block Public Access** enabled on the bucket
- [x] **Default encryption at rest** (SSE-S3) enabled
- [x] Bucket deployed in the governed home region (`us-east-2`)

---

## 🔐 Key Design Decisions

- **No inbound management ports.** SSH is the most scanned and attacked door on the internet. Session Manager removes the door entirely instead of just locking it.
- **Roles over keys.** The instance gets its permissions from an IAM role — nothing secret lives on the server to steal.
- **Root is not a working account.** Day-to-day work happens as a named IAM user, so every action is attributable to a person.
- **Guardrails on day one.** Budget alerts and audit logging are cheapest and easiest before anything is built — not after an incident.

---

## 📸 Evidence: Screenshots added to every illustration.

## 🛠️ AWS Services Used

IAM · AWS Budgets (Billing & Cost Management) · CloudTrail · EC2 · Systems Manager (Session Manager) · S3

---
## 📚 Lessons Learned

- Security controls cost almost nothing on day one — and a lot after an incident.
- A **role**, not a key pair, is the modern way to reach a server.
- Time-based MFA codes rotate every 30 seconds — enroll devices carefully, with fresh codes.
- Verify each control in the console after creating it — a checklist you haven't checked is a hope, not a control.

---

## 🚀 Future Enhancements

- Complete MFA enrolment for the admin IAM user and enforce MFA with an IAM policy
- Enable Amazon GuardDuty and AWS Security Hub for continuous monitoring
- Enable S3 Versioning and MFA Delete on critical buckets
- Add VPC endpoints for Systems Manager for fully private connectivity
- Grow into AWS Organizations as more accounts are added

---

> **Security note:** Real AWS account IDs, email addresses, and full bucket names are intentionally omitted from this document.

<sub>Hands-on portfolio project — built and verified in a live AWS account, November 2026.</sub>



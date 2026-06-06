# Week 9 Security Audit — cis410-deploy-sa

**Project:** cis410-hibr-497000
**Date:** June6, 2026
**Auditor:** Hibrework

---

## 1. IAM Audit Results

### Before — Week 8 Configuration (over-permissioned)

| Role | Scope | Problem |
|---|---|---|
| roles/run.admin | Project | Overly broad — grants ability to delete services and modify IAM, not just deploy |
| roles/storage.admin | Project | Overly broad — grants access to ALL GCS buckets in the project |
| roles/artifactregistry.writer | Project | Acceptable — scoped to push images only |
| roles/viewer | Project | Acceptable — read-only project metadata |
| roles/iam.serviceAccountUser | Compute SA | Required — needed to act as Compute Engine default SA |

### After — Week 9 Least-Privilege Fix

| Role | Scope | Why Sufficient |
|---|---|---|
| roles/run.developer | Project | Deploy only — cannot delete services or modify IAM |
| roles/storage.admin | tfstate bucket only | Scoped to one bucket — not all storage |
| roles/artifactregistry.writer | Project | Unchanged — push images only |
| roles/viewer | Project | Unchanged — read project metadata |
| roles/iam.serviceAccountUser | Compute SA | Unchanged — required for Cloud Run deployment |

---

## 2. Secret Manager Migration

- **Secret created:** `flask-app-secret`
- **Replication:** automatic
- **Access granted to:** `cis410-deploy-sa` — roles/secretmanager.secretAccessor on this secret only
- **Access granted to:** `PROJECT_NUMBER-compute@developer.gserviceaccount.com` — roles/secretmanager.secretAccessor on this secret only (required for Cloud Run runtime access)
- **Cloud Run update:** APP_SECRET environment variable mounted from Secret Manager at runtime

---

## 3. Monitoring Configuration

- **Log-based alert:** `cis410-flask-app-alert` — fires on severity>=WARNING for cis410-flask-app
- **Notification channel:** hibrjenebr@students.highline.edu
- **Billing budget:** `cis410-monthly-budget` — $20 limit, alerts at 50% / 90% / 100%

---

## 4. Reflection

**Q1: Why is roles/run.admin inappropriate for a CI/CD pipeline service account?**

roles/run.admin is overly permissive because it allows the service account to delete entire services or change global routing settings, which are not required for simple deployments. By using the Principle of Least Privilege, we should use roles/run.developer instead, which allows for creating new revisions without the risk of accidental or malicious service deletion.

---

**Q2: What is the security difference between storing a secret in GitHub Secrets vs. Google Secret Manager?**

GitHub Secrets are "Build-Time" secrets, meaning they are often injected into the application or environment during the build process, which can lead to secrets being accidentally logged or stored in container images. Google Secret Manager provides "Runtime" security, where the secret stays encrypted within Google’s infrastructure and is only fetched directly by the application memory at the moment it is needed.

---

**Q3: A coworker says "I will clean up IAM permissions after the project launches. For now I need everything to work fast." What is the risk of this approach?**

The primary risk is "Technical Debt" and an expanded "Blast Radius"; once a project launches, security clean-up is often forgotten, leaving the project permanently vulnerable. If the account is compromised during the launch phase, the attacker would have full administrative access to delete resources or steal data, a risk that is significantly higher than the few minutes saved by skipping proper IAM configuration.

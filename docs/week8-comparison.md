# Week 8 Comparison: On-Premise Docker vs. Cloud Run

| Dimension | On-Premise Docker (Wks 3–5) | Cloud Run (Week 8) |
| :--- | :--- | :--- |
| Infrastructure setup | 3 VMs created, Docker installed on each | Terraform defined the service; Google manages hardware. |
| Deployment command | SSH → docker build → docker run | gcloud run deploy (automated via GitHub Actions). |
| TLS / HTTPS | Not configured | Automatic HTTPS provided by Google by default. |
| Scaling approach | Manual — redeploy or add VMs | Automatic scaling including "Scale to Zero." |
| Port management | Ports 5000/5001/5002 per environment | Google routes traffic to container port 8080 automatically. |
| Cost when idle | VM running 24/7 regardless of traffic | $0 (Pay only when requests are being processed). |
| Rollback | Re-deploy previous image manually | Built-in Revision management allows for instant switching. |
| Secrets management | GitHub Secrets → env vars in workflow | OIDC Identity Federation (No static keys needed). |

## Reflection Questions

**Q1: Which approach required more manual steps?**
The on-premise approach required many more manual steps, including managing SSH keys, opening firewall ports manually, and ensuring Docker was installed and running. Cloud Run eliminated all server maintenance.

**Q2: How do you know which version of the code is currently running?**
For on-premise, I’d have to manually check `docker ps` on a specific VM. With Cloud Run, the "Revision" name in the GCP console includes the Git SHA, providing a clear audit trail.

**Q3: What is the security advantage of scale-to-zero?**
Scale-to-zero is a security win because an idle application has no running processes to exploit. If a container isn't running, an attacker cannot scan it or use it as a foothold.

**Q4: What attack surface was eliminated by OIDC?**
OIDC eliminated the need for static SSH keys. If an SSH key is stolen, the attacker has permanent access; OIDC uses temporary tokens that expire in minutes.
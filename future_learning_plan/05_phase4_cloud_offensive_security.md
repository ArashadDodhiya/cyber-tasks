# ☁️ Phase 4: Cloud Offensive Security (Months 12-14)

> **"The perimeter is dead. The cloud IS the network now. If you can't attack cloud, you can't pentest modern organizations."**

---

## 🎯 Objective

Learn to attack cloud infrastructure — AWS, Azure, GCP — including IAM escalation, container escapes, CI/CD pipeline exploitation, and multi-cloud pivoting.

---

## 📅 Timeline: 3 Months

| Week | Focus | Deliverable |
|---|---|---|
| 1-2 | Cloud Fundamentals & IAM | Understand IAM models across 3 clouds |
| 3-4 | AWS Offensive | IAM privesc + metadata exploitation |
| 5-6 | Azure Offensive | Entra ID + Azure AD attacks |
| 7-8 | Kubernetes & Container Attacks | Container escape + cluster compromise |
| 9-10 | CI/CD Pipeline Attacks | Pipeline compromise + secret extraction |
| 11-12 | Multi-Cloud Attack Chains | End-to-end cloud pentest |

---

## 1️⃣ Cloud Fundamentals

### IAM — The #1 Cloud Attack Surface

| Cloud | Identity Service | Key Concepts |
|---|---|---|
| **AWS** | IAM | Users, Roles, Policies, Instance Profiles, STS |
| **Azure** | Entra ID (Azure AD) | Users, Service Principals, Managed Identities, RBAC |
| **GCP** | Cloud IAM | Service Accounts, Workload Identity, IAM bindings |

### Common Cloud Attack Patterns

```
1. Initial Access → Leaked credentials, SSRF to metadata, phishing
2. Enumeration → IAM policies, resources, permissions
3. Privilege Escalation → Policy abuse, role chaining, service exploitation
4. Lateral Movement → Cross-account, cross-service, identity pivoting
5. Persistence → Backdoor IAM, function triggers, scheduled tasks
6. Data Exfiltration → S3/Blob/GCS access, database snapshots
```

---

## 2️⃣ AWS Offensive Security

### IAM Privilege Escalation Techniques

| Technique | Description |
|---|---|
| **iam:CreatePolicyVersion** | Create new version of policy with admin permissions |
| **iam:SetDefaultPolicyVersion** | Set an older, more permissive policy version as default |
| **iam:AttachUserPolicy** | Attach AdministratorAccess to yourself |
| **iam:CreateLoginProfile** | Create console login for programmatic-only user |
| **iam:PassRole + Lambda** | Pass admin role to Lambda, execute code with that role |
| **iam:PassRole + EC2** | Launch EC2 with admin instance profile |
| **iam:PassRole + CloudFormation** | Deploy stack with admin role |
| **sts:AssumeRole** | Assume cross-account or more privileged role |
| **ssm:StartSession** | Access EC2 instances without SSH |

### SSRF → Cloud Compromise

```
SSRF to http://169.254.169.254/latest/meta-data/
  → Retrieve IAM role credentials
  → Access AWS API with stolen temporary credentials
  → Enumerate and escalate

SSRF to IMDSv2:
  → Requires PUT request with hop-limit token
  → IMDSv2 mitigates basic SSRF but not all scenarios
```

### Key Tools

| Tool | Purpose |
|---|---|
| **Pacu** | AWS exploitation framework |
| **enumerate-iam** | Brute-force IAM permissions |
| **ScoutSuite** | Multi-cloud security auditing |
| **Prowler** | AWS security assessment |
| **CloudFox** | Find attack paths in AWS |
| **aws cli** | Direct AWS API interaction |

### Exercises

1. Set up CloudGoat and complete all scenarios
2. Exploit IAM privesc via iam:PassRole + Lambda
3. Perform SSRF to EC2 metadata and steal IAM credentials
4. Enumerate S3 buckets and find sensitive data
5. Pivot from compromised EC2 to other AWS services

### Resources

- 🧪 **CloudGoat** — Rhino Security Labs (AWS vulnerable scenarios)
- 📖 **Hacking the Cloud** — hackingthe.cloud
- 🎓 **AWS Security Specialty** — understand defender perspective

---

## 3️⃣ Azure / Entra ID Offensive Security

### Attack Techniques

| Attack | Description |
|---|---|
| **Password Spraying** | Against Azure AD/Entra ID endpoints |
| **Illicit Consent Grant** | Trick user into granting app permissions |
| **Managed Identity Abuse** | Exploit managed identity from compromised VM/App Service |
| **PRT (Primary Refresh Token)** | Steal PRT for Azure AD SSO session hijacking |
| **Storage Account Keys** | Access Blob storage with leaked keys |
| **Function App Code Injection** | Modify serverless function code |
| **Runbook Abuse** | Execute code via Automation Account runbooks |
| **ARM Template Injection** | Inject malicious resources via templates |

### Key Tools

| Tool | Purpose |
|---|---|
| **ROADtools** | Azure AD enumeration and exploitation |
| **AzureHound** | BloodHound for Azure — attack path mapping |
| **MicroBurst** | Azure security assessment scripts |
| **TokenTactics** | Azure AD token manipulation |
| **GraphRunner** | Microsoft Graph API exploitation |

### Exercises

1. Perform password spray against Azure AD endpoint
2. Exploit Managed Identity from compromised Azure VM
3. Use AzureHound to map Azure AD attack paths
4. Steal and replay Primary Refresh Token
5. Escalate from Reader to Contributor via misconfigured roles

---

## 4️⃣ Kubernetes & Container Attacks

### Container Escape Techniques

| Technique | Prerequisite |
|---|---|
| **Privileged container escape** | `--privileged` flag or excessive capabilities |
| **Mount host filesystem** | Volume mount of / or /etc |
| **Docker socket access** | /var/run/docker.sock mounted |
| **Kernel exploit** | Shared kernel with host |
| **cgroup escape** | Misconfigured cgroup notify_on_release |
| **CAP_SYS_ADMIN** | Ability to mount, namespace manipulation |

### Kubernetes Attack Path

```
1. Initial Access
   ├── Exposed API server (anonymous auth)
   ├── Compromised container
   ├── Leaked kubeconfig / service account token
   └── Supply chain (malicious image)

2. Enumeration
   ├── Service account permissions (can-i)
   ├── Secrets in namespace
   ├── Pod configurations
   └── Network policies (or lack thereof)

3. Privilege Escalation
   ├── Privileged pods
   ├── Node access → all pod secrets
   ├── Service account token theft
   └── RBAC misconfigurations

4. Lateral Movement
   ├── Pod-to-pod (flat network)
   ├── Service account hopping
   ├── Cloud IAM from pod (IMDS)
   └── etcd access (all secrets)

5. Persistence
   ├── Backdoor admission webhook
   ├── Malicious CronJob
   ├── Shadow admin RBAC binding
   └── Compromised CI/CD → auto-deploy
```

### Key Tools

| Tool | Purpose |
|---|---|
| **kubectl** | K8s cluster interaction |
| **kube-hunter** | K8s penetration testing |
| **Peirates** | K8s post-exploitation |
| **CDK** | Container escape toolkit |
| **Deepce** | Docker enumeration and escape |
| **kubeaudit** | K8s security auditing |

### Exercises

1. Escape a privileged Docker container to host
2. Exploit exposed Kubernetes API with anonymous auth
3. Steal secrets from Kubernetes cluster
4. Pivot from pod to cloud IAM via IMDS
5. Set up and compromise Kubernetes Goat

### Resources

- 🧪 **Kubernetes Goat** — Madhu Akula
- 🧪 **contained.af** — Container escape challenges
- 📖 **Kubernetes Security** — Liz Rice
- 🔗 **Bishop Fox: Bad Pods** — K8s pod privilege research

---

## 5️⃣ CI/CD Pipeline Attacks

### Attack Surface

| Component | Attack |
|---|---|
| **GitHub Actions** | Workflow injection, secret extraction, self-hosted runner abuse |
| **Jenkins** | Groovy script console, credential extraction, pipeline manipulation |
| **GitLab CI** | Shared runner exploitation, variable extraction |
| **Docker Registry** | Image tampering, credential theft |
| **Terraform** | State file secrets, provider hijacking |
| **ArgoCD** | SSO bypass, application manipulation |

### Exercises

1. Extract secrets from GitHub Actions logs (intentional vulnerable repo)
2. Exploit a Jenkins Groovy script console
3. Poison a CI/CD pipeline to inject a backdoor
4. Access a Terraform state file and extract secrets
5. Compromise a container registry and push a malicious image

### Resources

- 🧪 **CI/CDon't** — GitHub CI/CD vulnerable lab
- 📖 **Cider Security Top 10 CI/CD Risks**
- 🔗 **OWASP CI/CD Security**

---

## 📝 Phase 4 Completion Checklist

- [ ] Understand IAM models across AWS, Azure, and GCP
- [ ] Performed IAM privilege escalation in AWS via at least 3 methods
- [ ] Exploited SSRF → cloud metadata → API access
- [ ] Attacked Azure AD — password spray, managed identity abuse
- [ ] Escaped a Docker container to the host
- [ ] Compromised a Kubernetes cluster end-to-end
- [ ] Exploited a CI/CD pipeline and extracted secrets
- [ ] Completed CloudGoat and Kubernetes Goat
- [ ] Performed a multi-cloud attack chain

---

**Next: `06_phase5_exploit_dev_and_reversing.md` →**

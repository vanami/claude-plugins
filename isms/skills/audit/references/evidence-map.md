# Where the evidence for Annex A usually lives

A starting point, not a checklist: what to look for in code, CI and infrastructure for the
controls a software company can evidence that way. Always read the control's own `question`
and `guidance` with `get_control` first. "Ask" marks what code rarely shows.

## A.5 Organisational

| Control | Look for |
| --- | --- |
| A.5.7 Threat intelligence | Security advisories feeds (Dependabot alerts, GitHub security advisories, cloud security hub findings). Ask how they are followed up. |
| A.5.9 Inventory of assets | IaC resources (databases, buckets, clusters, queues), repositories, domains and DNS zones, SaaS in use. Feeds the asset register. |
| A.5.10 / A.5.12 / A.5.13 Acceptable use, classification, labelling | Data classification in docs or code (tags like `data-classification`), PII fields, bucket/table tags. Ask for the policy. |
| A.5.14 Information transfer | TLS everywhere (load balancer listeners, HSTS headers, `ssl_policy`), SFTP/API integrations, e-mail authentication (SPF, DKIM, DMARC records in DNS/IaC). |
| A.5.15 Access control / A.5.18 Access rights | IAM roles and policies (least privilege, no `*:*`), SSO/OIDC for cloud consoles, GitHub org/team permissions, app RBAC in code, access reviews. |
| A.5.16 Identity management | Central IdP (Google Workspace, Entra ID, Okta), SCIM, offboarding scripts, shared accounts. |
| A.5.17 Authentication information | Password hashing (argon2/bcrypt/scrypt) in code, secrets manager / KMS / sealed secrets, no secrets in the repo (gitleaks/trufflehog in CI, `.gitignore` of `.env`). |
| A.5.19–A.5.22 Supplier security | Every external service in code and IaC: cloud providers, SaaS SDKs in dependency manifests, payment, e-mail, monitoring. DPAs and certificates: ask. |
| A.5.23 Cloud services | Which clouds and regions (IaC providers, `region`), account structure, shared-responsibility notes, exit considerations. |
| A.5.24–A.5.28 Incident management | Alerting (CloudWatch alarms, Grafana, PagerDuty/Opsgenie configs), runbooks, incident templates, post-mortems, log retention for evidence (A.5.28). |
| A.5.29 / A.5.30 Continuity and ICT readiness | Multi-AZ/region, backups and restore tests, RTO/RPO in docs, infrastructure reproducible from code, DR runbooks. |
| A.5.31 Legal and regulatory | GDPR handling in code (consent, data subject requests), retention and deletion jobs. Ask for the register of requirements. |
| A.5.33 Protection of records | Immutable logs (S3 object lock, log archive accounts), versioning, audit tables. |
| A.5.34 Privacy and PII | PII inventory in schemas, encryption of PII, minimisation, retention jobs, DPA with processors. |
| A.5.36 Compliance with policies | Policy-as-code (OPA/Conftest, tfsec/checkov/trivy config in CI), CIS benchmarks, Security Hub/Config rules. |
| A.5.37 Documented operating procedures | READMEs, runbooks, deploy docs, `Makefile`/scripts that are the procedure. |

## A.6 People, A.7 Physical

Mostly ask: screening, terms of employment, awareness training, disciplinary process, NDAs,
remote work (A.6.7: device and network rules, VPN/zero-trust config may be evidence), event
reporting channel (A.6.8). Physical: cloud data centres are the provider's (cite their
certificates as supplier evidence); offices, clear desk, equipment and media are the
company's. A.7.9/A.7.10/A.7.14 can partly be evidenced by MDM and disk encryption config.

## A.8 Technological

| Control | Look for |
| --- | --- |
| A.8.1 User endpoint devices | MDM profiles (Jamf, Intune, Kandji), disk encryption, OS update policy. Ask. |
| A.8.2 Privileged access rights | Admin roles in IAM, break-glass accounts, `kubectl` cluster-admin bindings, GitHub org owners, just-in-time access, MFA enforcement. |
| A.8.3 Information access restriction | Authorisation in the application (role checks, tenant isolation in queries), row-level security, bucket policies, public access blocks. |
| A.8.4 Access to source code | GitHub/GitLab visibility, team permissions, branch protection, CODEOWNERS, deploy keys, required 2FA (`gh api orgs/<org>` → `two_factor_requirement_enabled`). |
| A.8.5 Secure authentication | SSO/OIDC for the product and internal tools, MFA/passkeys, session lifetimes, lockout/rate limits, token verification in code. |
| A.8.6 Capacity management | Autoscaling (HPA, ASG, serverless concurrency), quotas, load tests, capacity alarms. |
| A.8.7 Protection against malware | Image scanning, endpoint protection (ask), file upload scanning, dependency scanning. |
| A.8.8 Technical vulnerabilities | Dependabot/Renovate config, `npm audit`/`govulncheck`/`pip-audit`/Trivy in CI, ECR scan on push, patch cadence of base images, pen-test reports (ask). |
| A.8.9 Configuration management | Everything as code (Terraform, Helm, Kustomize), reviewed through PRs, drift detection, hardened baselines. |
| A.8.10 Information deletion | Retention and deletion jobs, lifecycle rules on buckets, account deletion flows, TTLs. |
| A.8.11 Data masking | Masking/pseudonymisation in logs and non-production copies, log scrubbers. |
| A.8.12 Data leakage prevention | Public access blocks, egress controls, secret scanning, DLP in mail/SaaS (ask). |
| A.8.13 Information backup | RDS `backup_retention_period`, DynamoDB point-in-time recovery, S3 versioning and replication, AWS Backup plans, Velero, restore tests. |
| A.8.14 Redundancy | Multi-AZ databases, replicas, multiple nodes/zones, health checks, failover. |
| A.8.15 Logging | CloudTrail (multi-region, log file validation), application logs, access logs, retention settings, tamper protection. |
| A.8.16 Monitoring activities | Alarms, dashboards, anomaly/threat detection (GuardDuty, Security Hub), error tracking (Sentry), uptime checks, who responds. |
| A.8.17 Clock synchronisation | Managed NTP in the cloud (Amazon Time Sync, Google NTP); timestamps in UTC/ISO in code. |
| A.8.18 Privileged utility programs | Restrictions on shell/exec into production (`kubectl exec` RBAC, SSM Session Manager logging, bastion policies). |
| A.8.19 Software installation on operational systems | Immutable images, pinned versions, lock files, no manual installs, signed images. |
| A.8.20 Network security / A.8.21 Network services / A.8.22 Segregation | VPCs, subnets, security groups and NACLs, Kubernetes NetworkPolicies, private endpoints, WAF, separate environments. |
| A.8.23 Web filtering | Usually not applicable for a cloud-only company; if so, justify. DNS filtering on devices: ask. |
| A.8.24 Use of cryptography | TLS versions and policies, certificate management, encryption at rest (`storage_encrypted`, SSE-KMS), KMS key rotation, crypto libraries in code. |
| A.8.25 Secure development life cycle | Branching and review rules, CI gates, threat modelling or security sign-off in PR templates. |
| A.8.26 Application security requirements | Input validation, auth requirements, security headers (CSP, HSTS), documented in code or tickets. |
| A.8.27 Secure architecture and engineering | Architecture docs, tenant isolation, least privilege between services, defence in depth. |
| A.8.28 Secure coding | Linters, SAST (CodeQL, Semgrep), typed languages, parameterised queries, coding guidelines (`CLAUDE.md`, `CONTRIBUTING.md`). |
| A.8.29 Security testing | Security tests in CI, DAST, pen-tests (ask), tests of auth and isolation. |
| A.8.30 Outsourced development | External contributors and contractors in the repo, their access and review. Ask. |
| A.8.31 Separation of environments | Separate cloud accounts/projects/clusters per environment, separate credentials, preview deployments without production data. |
| A.8.32 Change management | PR reviews, required checks, deploy pipelines, release tags, rollback procedures, change log. |
| A.8.33 Test information | No production data in tests/seeds/fixtures; synthetic data generators. |
| A.8.34 Protection during audit testing | Read-only audit access, scheduled scans, agreements with pen-testers. |

## Useful read-only commands (ask before the first cloud call)

```sh
gh api repos/<owner>/<repo>/branches/<default>/protection
gh api orgs/<org> --jq '{two_factor_requirement_enabled, default_repository_permission}'
gh api repos/<owner>/<repo>/vulnerability-alerts -i      # 204 = Dependabot alerts on
aws sts get-caller-identity
aws cloudtrail describe-trails
aws rds describe-db-instances --query 'DBInstances[].{id:DBInstanceIdentifier,multiAZ:MultiAZ,encrypted:StorageEncrypted,backup:BackupRetentionPeriod}'
aws dynamodb describe-continuous-backups --table-name <table>
aws s3api get-public-access-block --bucket <bucket>
kubectl get networkpolicies -A
```

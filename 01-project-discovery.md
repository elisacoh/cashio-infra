# Cash Payments API — Project Discovery

**Document status:** Draft 0.1 — requires joint review  
**Last updated:** 2026-11-02  
**Current delivery scope:** TEST environment only  
**Document owner:** Infrastructure / DevOps  
**Reviewers:** Application developer; business owner where applicable

> Status convention: **Confirmed** = backed by the team's explicit agreement or verified evidence. **Proposed** = design suggestion not yet approved. **To verify** = fact or implementation not yet inspected. Avoid treating a specification as evidence of running code.

## 1. Purpose

Prepare a reproducible and secure integration-test environment for the Cash Payments API on DreamVPS, then establish a foundation that can later be adapted for production. This phase documents requirements and boundaries; it does not authorize deployment of real-money processing.

## 2. Business context

An online merchant initiates a payment request. The application creates a voucher/barcode for the customer. The customer pays cash at an authorized partner's point of sale. An authenticated collection channel confirms the collection through the API. The API persists the collection and notifies the merchant asynchronously. Partner financial settlements and reconciliation take place separately and may occur at the end of the month.

**Clarification:** The application is not intended to process cardholder data. This does not remove the need to secure payment-confirmation authorization, transaction integrity, and sensitive personal/business data.

## 3. Proposed technical baseline

| Area | Current understanding | Status |
| --- | --- | --- |
| Hosting | DreamVPS private-cloud virtual servers | Proposed; provider capabilities to verify |
| Immediate environment | One isolated TEST VM, Ubuntu Server 24.04 LTS | Proposed |
| VM size | 2 vCPU, 4 GB RAM, 30 GB SSD/NVMe | Proposed |
| API | Node.js 24 LTS, TypeScript, Fastify | Planned; code not inspected |
| Worker | Separate process/container from same codebase | Planned; code not inspected |
| Database | PostgreSQL 17, persistent storage | Proposed; pin exact supported version |
| Runtime | Docker Engine + Docker Compose | Proposed |
| Reverse proxy | HTTPS via NGINX or Caddy | Open decision |
| CI/CD | GitHub Actions, versioned amd64 images | Proposed |
| Configuration management | Ansible | Proposed |
| VM provisioning | OpenTofu/Terraform **if** DreamVPS supports suitable API/provider | To verify |
| Backups | Daily TEST backups retained 7 days, outside the VM | Proposed |
| Monitoring | HTTPS, host resources, API/worker, DB, backups | Proposed |

No Dockerfile, migrations, health endpoints or live environment have yet been verified.

## 4. Responsibility boundary

| Topic | Application developer | Infrastructure / DevOps |
| --- | --- | --- |
| Payment idempotency and state transitions | Design, implement and test | Identify failure modes; provide integration-test environment |
| POS credentials and authorization | Define and implement | Secure network, secrets delivery, logs and access boundaries |
| Database schema, migrations, transaction semantics | Own application schema and migration code | Operate PostgreSQL, execute controlled migrations in deployment |
| API/worker entrypoints, health, graceful shutdown | Implement | Wire Compose healthchecks, restart policies and monitoring |
| Webhooks and durable jobs | Implement retries and durable semantics | Monitor failures, resource limits and process recovery |
| VM, OS, firewall, TLS, DNS | Provide needed application requirements | Design, automate, configure and maintain |
| CI/CD and deployment | Supply tests/build commands; review compatibility | Own pipeline, image promotion, deploy, smoke checks |
| Financial settlement reconciliation | Define business workflow and application evidence | Ensure durable storage, recovery procedure and monitoring |
| RPO/RTO and acceptable downtime | Business owner approves | Propose options, cost and testable objectives |

## 5. Scope of the first milestone: TEST

**In scope:** isolated TEST-01; controlled SSH administration; host firewall; Docker/Compose; reverse proxy + HTTPS; persistent PostgreSQL; separately managed API and worker; versioned application images; reproducible host configuration; basic CI/CD; off-VM backup; essential health and log visibility; simple rebuild and restore procedures.

**Out of scope for now:** production HA, Kubernetes, Redis/broker, multi-node scaling, production SLA/SLO commitments, direct handling of real cash transactions, automated bank settlement, advanced observability platform.

**Constraint:** TEST uses fake transactions, separate credentials, no real-money collection. Enforce isolation from future PROD, including credentials and DNS.

## 6. Known unknowns and validation owner

| ID | Question / evidence required | Owner | Blocking TEST? |
| --- | --- | --- | --- |
| DISC-01 | Confirm DreamVPS VM creation method, Ubuntu image, public IP, network and firewall capabilities | DevOps + provider | Yes |
| DISC-02 | Confirm access to application repository and actual build/launch commands for API/worker | Developer | Yes for app deployment |
| DISC-03 | Confirm Dockerfile or agree who creates it | Developer + DevOps | Yes for app deployment |
| DISC-04 | Confirm migrations and database initialization process | Developer | Yes for full integration |
| DISC-05 | Confirm secrets/configuration contract and how test credentials are provisioned | Developer + DevOps | Yes |
| DISC-06 | Confirm how sandbox-only collection simulation is protected | Developer | Yes for integration testing |
| DISC-07 | Confirm public TEST DNS ownership and TLS approach | DevOps + project owner | Yes for HTTPS integration |
| DISC-08 | Confirm availability of external partner sandbox credentials and endpoints | Developer + partner | No for local test; yes for external integration |
| DISC-09 | Confirm external storage destination for TEST backups | DevOps + project owner | Yes before declaring TEST ready |
| DISC-10 | Confirm production availability objectives and regulated-operation responsibilities | Business owner | No; future production gate |

## 7. Initial acceptance criteria for TEST

| ID | Evidence to demonstrate |
| --- | --- |
| TEST-01 | A fresh Ubuntu TEST VM can be configured using documented, version-controlled automation. |
| TEST-02 | Re-running configuration automation produces no unintended changes. |
| TEST-03 | Only HTTPS app endpoints are internet-facing; PostgreSQL and app container ports are not publicly reachable. |
| TEST-04 | API and worker run as separately supervised containers and recover after a host reboot. |
| TEST-05 | CI builds and deploys a pinned image version, then runs smoke checks. |
| TEST-06 | TEST uses separate, non-production credentials and synthetic payment data. |
| TEST-07 | A backup stored outside the VM is restored successfully into an isolated test target. |
| TEST-08 | Outage of API, worker or database can be detected, and service logs are accessible to authorized operators. |

## 8. Decisions to record later (ADR)

ADR-001: Docker Compose for the initial single-host application deployment.  
ADR-002: Ansible for idempotent host configuration.  
ADR-003: PostgreSQL hosted locally on TEST with a persistent volume.  
ADR-004: CI/CD release and promotion mechanism.  
ADR-005: Reverse proxy and TLS certificate strategy.  

These are placeholders, not approved decisions.

## 9. Phase 1 exit criteria

The two-person team reviews this document, corrects inaccuracies, assigns an owner to unknowns, and agrees on the TEST acceptance criteria. Next: requirements and a draft system-context / deployment diagram. Production objectives can remain open while TEST is built.

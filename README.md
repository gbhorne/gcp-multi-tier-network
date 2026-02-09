# GCP Multi-Tier Network Architecture

[![GCP](https://img.shields.io/badge/Google%20Cloud-4285F4?style=for-the-badge&logo=google-cloud&logoColor=white)](https://cloud.google.com)
[![Networking](https://img.shields.io/badge/VPC-Networking-orange?style=for-the-badge)](https://cloud.google.com/vpc)

> Production-grade multi-tier network architecture on Google Cloud Platform demonstrating enterprise networking, security segmentation, and Infrastructure as Code.

##  Architecture Overview

### Components Built

| Component | Details |
|-----------|---------|
| **VPC** | production-vpc (custom mode) |
| **Subnets** | 4 tiers: web, app, data, management |
| **Instances** | 3 VMs (bastion, web, app servers) |
| **Load Balancer** | HTTP(S) Global LB with health checks |
| **Firewall Rules** | 5 least-privilege security rules |

### Security Architecture

-  **Zero Trust**: App tier has NO external IP
-  **Bastion Pattern**: Single SSH entry point
-  **Network Segmentation**: Isolated tiers
-  **Least Privilege**: Minimal firewall access

##  Infrastructure Details

### Compute Resources

| Instance | vCPUs | RAM | Internal IP | External IP | Role |
|----------|-------|-----|-------------|-------------|------|
| bastion-host | 2 | 1GB | 10.0.10.2 | Yes | SSH Gateway |
| web-server-1 | 2 | 1GB | 10.0.1.2 | Yes | Apache Web |
| app-server-1 | 2 | 1GB | 10.0.2.2 | **NO** | Python App |

**Total**: 6 vCPUs, 3GB RAM

### Subnets

| Tier | CIDR | Access |
|------|------|--------|
| Management | 10.0.10.0/24 | SSH from your IP |
| Web | 10.0.1.0/24 | Public HTTP/HTTPS |
| Application | 10.0.2.0/24 | Private only |
| Data | 10.0.3.0/24 | Private only |

##  Connectivity Testing

| Test | Result |
|------|--------|
| Internet  Web Server |  PASS |
| Internet  Load Balancer |  PASS |
| Bastion  Web/App |  PASS |
| Web  App (8080) |  PASS |
| Internet  App |  BLOCKED  |

##  Cost Analysis

| Component | Hourly | Daily |
|-----------|--------|-------|
| 3 e2-micro | $0.03 | $0.72 |
| Load Balancer | $0.025 | $0.60 |
| **Total** | **$0.06** | **$1.44** |

**Lab Cost**: < $0.50

##  Technologies

- Google Cloud Platform
- VPC Networking
- Compute Engine
- Cloud Load Balancing
- gcloud CLI

##  Skills Demonstrated

- Cloud architecture design
- Network segmentation
- Security best practices (Zero Trust)
- Infrastructure as Code
- Load balancing

##  Author

**Greg Horne**
- GitHub: [@gbhorne](https://github.com/gbhorne)

---

**Built with  on Google Cloud Platform**

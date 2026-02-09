# Architecture Documentation

## Network Design

### VPC: production-vpc
- Mode: Custom
- Region: us-central1

### Subnets
- Management: 10.0.10.0/24
- Web: 10.0.1.0/24
- Application: 10.0.2.0/24
- Data: 10.0.3.0/24

## Security Model

**Zero Trust Architecture**
- App tier has NO external IP
- All SSH through bastion
- Least-privilege firewall rules

## Resources
- 3 VMs (e2-micro)
- 6 vCPUs total
- 3GB RAM total
- 1 Load Balancer

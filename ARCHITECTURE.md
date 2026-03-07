# Architecture Documentation

## Network Design

### VPC: production-vpc

| Property | Value |
|----------|-------|
| Mode | Custom |
| Region | us-central1 |
| Routing | Regional |
| Project | your-project-id |

Custom mode was selected over auto mode to maintain explicit control over subnet CIDR ranges, prevent unintended connectivity between tiers, and follow production network design practices.

### Subnet Layout

| Tier | CIDR | Size | Purpose |
|------|------|------|---------|
| Web | 10.0.1.0/24 | 256 IPs | Public-facing web servers |
| Application | 10.0.2.0/24 | 256 IPs | Private application servers |
| Data | 10.0.3.0/24 | 256 IPs | Reserved for database layer |
| Management | 10.0.10.0/24 | 256 IPs | Bastion host and admin access |

Tiers are non-overlapping and isolated by default. Inter-tier communication is permitted only via explicit firewall rules.

---

## Security Model

### Zero Trust Principles Applied

**No implicit trust between tiers.** Each subnet is isolated by default. Traffic between tiers requires an explicit allow rule scoped to the minimum required source range and port.

**No direct external access to the application tier.** app-server-1 has no external IP address. It is unreachable from the internet by design. The only path into the app tier from outside is: Internet > Bastion (TCP:22) > app-server-1 (TCP:22).

**Single SSH entry point.** The bastion host in the management subnet is the only instance with an external IP that accepts SSH. All administrative access to web-server-1 and app-server-1 routes through it.

**Least-privilege firewall rules.** No rule is broader than required. SSH to the bastion is restricted to a specific source IP. Web traffic is allowed only on TCP:80 and TCP:443. App tier communication is restricted to TCP:8080 from the web subnet only.

### Traffic Flow

```
Internet
    |
    |--> TCP:80/443 --> Global Load Balancer --> web-server-1 (10.0.1.2)
    |                                                |
    |                                           TCP:8080
    |                                                |
    |                                         app-server-1 (10.0.2.2) [NO external IP]
    |
    |--> TCP:22 (your IP only) --> bastion-host (10.0.10.2)
                                        |
                                   TCP:22 (internal)
                                        |
                              web-server-1 / app-server-1
```

---

## Compute Resources

| Instance | Subnet | Machine Type | vCPUs | RAM | External IP |
|----------|--------|-------------|-------|-----|-------------|
| bastion-host | management-tier | e2-micro | 2 | 1GB | Yes |
| web-server-1 | web-tier | e2-micro | 2 | 1GB | Yes |
| app-server-1 | app-tier | e2-micro | 2 | 1GB | None |

**Totals:** 6 vCPUs, 3GB RAM, 3 instances

All instances are deployed in us-central1-a. A production version of this architecture would distribute instances across multiple zones for availability.

---

## Load Balancer Configuration

| Property | Value |
|----------|-------|
| Type | Global HTTP(S) |
| Protocol | HTTP:80 |
| Health check | HTTP:80, path / |
| Backend | web-server-1 |
| Routing | URL map, default service |

The load balancer uses GCP health check ranges (35.191.0.0/16, 130.211.0.0/22) which must be explicitly permitted in the firewall rules targeting the web tier.

---

## Design Decisions

**Custom VPC over auto mode:** Auto mode VPCs create subnets in every region automatically, which is inappropriate for a controlled multi-tier design. Custom mode provides explicit control over which subnets exist and where.

**e2-micro instances:** Selected for cost efficiency in a lab environment. Production workloads would use appropriately sized instances based on application requirements.

**Data tier reserved:** The 10.0.3.0/24 subnet is created but not populated. This reserves address space for a future Cloud SQL instance, Memorystore, or self-managed database VM without requiring VPC redesign.

**Bastion pattern over OS Login / IAP:** The bastion host approach was chosen to demonstrate the network-level isolation pattern explicitly. In a production environment, Cloud IAP (Identity-Aware Proxy) with OS Login would be preferred as it eliminates the need for a dedicated bastion VM and external IP entirely.

---

## Future Improvements

**Short-term:**
- Replace bastion with Cloud IAP for SSH to eliminate the external IP on the management tier
- Add Cloud NAT to allow app-server-1 to make outbound internet requests without an external IP
- Deploy Cloud SQL in the data tier subnet
- Add HTTPS with a managed SSL certificate to the load balancer

**Longer-term:**
- Implement VPC Service Controls around the data tier
- Add Cloud Armor to the load balancer for WAF capability
- Enable VPC Flow Logs on all subnets for network traffic analysis
- Distribute instances across multiple zones with a regional MIG

---

## References

- [VPC Documentation](https://cloud.google.com/vpc/docs)
- [Firewall Rules Overview](https://cloud.google.com/vpc/docs/firewalls)
- [Cloud Load Balancing](https://cloud.google.com/load-balancing/docs)
- [Cloud IAP for SSH](https://cloud.google.com/iap/docs/using-tcp-forwarding)
- [Cloud NAT](https://cloud.google.com/nat/docs/overview)

---

**Document Version:** 1.0
**Last Updated:** March 6, 2026

# Ephemeral IP on GCP Free-Tier VM

## The Problem

The GCE e2-micro free-tier VM gets an ephemeral public IP. If the IP changes, the Route 53 A record becomes stale and the domain stops resolving to the VM.

## How Often Does the IP Actually Change?

GCP does not proactively restart free-tier VMs. The IP only changes if:

- **You manually stop and start the VM** (or redeploy via Terraform)
- **Live migration fails** — GCP live-migrates e2-micro instances for host maintenance by default, which preserves the IP. A restart (with potential IP change) only happens if live migration fails, which is rare.
- **The VM crashes** — uncommon for a lightweight workload like Actual Budget

In practice, the same ephemeral IP often persists for months or years on a stable GCE instance.

## Mitigation Options

| Option | Cost | Effort |
|--------|------|--------|
| **Manual update** — update the Route 53 A record if the IP changes | Free | Low, but manual |
| **GCP static IP** — reserve a static external IP in Terraform | ~$3/month | Low |
| **Auto-update script** — container on the VM that calls Route 53 API to update the A record on IP change | Free (needs AWS IAM creds on the VM) | Medium |
| **Uptime monitor** — use a free service like UptimeRobot to alert when the domain stops responding | Free | Low |

## Getting the VM Public IP

After `terraform apply`, run:

```bash
gcloud compute instances describe containerhost01 --zone=us-east1-c --format='get(networkInterfaces[0].accessConfigs[0].natIP)'
```

Or check GCP Console → Compute Engine → VM instances.

## Route 53 DNS Record

In the hosted zone for `emmaandsimon.co.uk`, create:

- **Record name**: `budget`
- **Record type**: `A`
- **Value**: the VM's public IP from above
- **TTL**: 300

## Recommendation

For a personal budget app, the ephemeral IP is a non-issue. Manual Route 53 updates on the rare occasion the IP changes is perfectly reasonable. Pair with a free uptime monitor for peace of mind.

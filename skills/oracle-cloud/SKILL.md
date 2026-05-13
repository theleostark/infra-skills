---
name: Oracle Cloud
description: 'Use when the user needs: Oracle Cloud Infrastructure (OCI) management — provision Always Free ARM instances, manage compute/network/storage, deploy containers, check instance health, and manage the Shadow Labs production infrastructure. Use when working with Oracle Cloud, OCI, compute instances, VCN networking, or deploying to the shadowlab.cc infrastructure.'
icon: icon.svg
command: oci-manage
user_invocable: true
metadata:
  category: mesh/shadow
  family: mesh
  lifecycle: active
  canonical_slug: oracle-cloud
  icon_style: craft-category-glyph-v1
---

# Oracle Cloud Infrastructure — Agent Skill

## Prerequisites
- OCI CLI: `oci` (installed at `/opt/homebrew/bin/oci`)
- Config: `~/.oci/config` (run `oci setup config` if missing)
- SSH key: `~/.ssh/shadowlabs-oci` (created by provision script)
- Deploy scripts: `/Volumes/SHADOW/shadowlabs-api/deploy/`

## Quick Reference

### Check Status
```bash
# Auth check
oci iam region list --output table 2>&1 | head -5

# List all instances
oci compute instance list --compartment-id $COMPARTMENT --query 'data[].{"Name":"display-name","State":"lifecycle-state","Shape":shape}' --output table

# Get instance public IP
oci compute instance list-vnics --instance-id $INSTANCE_ID --query 'data[0]."public-ip"' --raw-output
```

### Compute — Always Free ARM (A1.Flex)
```bash
# Provision (use the deploy script)
/Volumes/SHADOW/shadowlabs-api/deploy/oci-provision.sh

# Start/stop/reboot instance
oci compute instance action --instance-id $INSTANCE_ID --action START
oci compute instance action --instance-id $INSTANCE_ID --action STOP
oci compute instance action --instance-id $INSTANCE_ID --action SOFTRESET

# SSH into instance
ssh -i ~/.ssh/shadowlabs-oci ubuntu@<PUBLIC_IP>

# Instance details
oci compute instance get --instance-id $INSTANCE_ID --query 'data.{"Name":"display-name","State":"lifecycle-state","Shape":shape,"OCPUs":"shape-config".ocpus,"RAM":"shape-config"."memory-in-gbs"}' --output table
```

### Networking
```bash
# List VCNs
oci network vcn list --compartment-id $COMPARTMENT --query 'data[].{"Name":"display-name","CIDR":"cidr-block","ID":id}' --output table

# List subnets
oci network subnet list --compartment-id $COMPARTMENT --query 'data[].{"Name":"display-name","CIDR":"cidr-block","AD":"availability-domain"}' --output table

# List security rules (firewall)
oci network security-list list --compartment-id $COMPARTMENT --vcn-id $VCN_ID --query 'data[0]."ingress-security-rules"[].{"Protocol":protocol,"Source":source,"Ports":"tcp-options"."destination-port-range"}' --output table

# Open a port
oci network security-list update --security-list-id $SL_ID --force \
  --ingress-security-rules '[{"protocol":"6","source":"0.0.0.0/0","tcpOptions":{"destinationPortRange":{"min":PORT,"max":PORT}}}]'
```

### Storage
```bash
# List block volumes
oci bv volume list --compartment-id $COMPARTMENT --query 'data[].{"Name":"display-name","Size":"size-in-gbs","State":"lifecycle-state"}' --output table

# List object storage buckets
oci os bucket list --compartment-id $COMPARTMENT --query 'data[].{"Name":name,"Created":"time-created"}' --output table

# Create a bucket
oci os bucket create --compartment-id $COMPARTMENT --name shadowlabs-data
```

### Deploy Workflow
```bash
# Full provision (new instance)
cd /Volumes/SHADOW/shadowlabs-api
./deploy/oci-provision.sh

# Setup server (after provision)
./deploy/setup-server.sh <PUBLIC_IP>

# Push .env to server
scp -i ~/.ssh/shadowlabs-oci .env ubuntu@<IP>:~/shadowlabs-api/.env

# Restart API on server
ssh -i ~/.ssh/shadowlabs-oci ubuntu@<IP> 'sudo systemctl restart shadowlabs-api'

# Check server health
curl -s https://api.shadowlab.cc/health

# View server logs
ssh -i ~/.ssh/shadowlabs-oci ubuntu@<IP> 'journalctl -u shadowlabs-api --no-pager -n 50'

# Deploy code update
ssh -i ~/.ssh/shadowlabs-oci ubuntu@<IP> 'cd shadowlabs-api && git pull && sudo systemctl restart shadowlabs-api'
```

### Monitoring
```bash
# Instance metrics (CPU, memory, network)
oci monitoring metric-data summarize-metrics-data \
  --compartment-id $COMPARTMENT \
  --namespace oci_computeagent \
  --query-text "CpuUtilization[1h]{resourceId=\"$INSTANCE_ID\"}.mean()" \
  --start-time $(date -u -v-1H +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ)

# Audit logs
oci audit event list --compartment-id $COMPARTMENT \
  --start-time $(date -u -v-1d +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --query 'data[].{"Event":"event-name","Time":"event-time","Actor":"data"."identity"."principal-name"}'
```

### Cost & Limits
```bash
# Check Always Free usage limits
oci limits value list --compartment-id $COMPARTMENT --service-name compute \
  --query 'data[?"name"==`standard-a1-core-count`]' --output table

# List all resource limits
oci limits resource-availability get --compartment-id $COMPARTMENT \
  --service-name compute --limit-name standard-a1-core-count \
  --availability-domain $AD
```

### Generative AI (OCI has built-in AI services)
```bash
# List available AI models
oci generative-ai model list --compartment-id $COMPARTMENT \
  --query 'data.items[].{"Name":"display-name","Type":"model-type","State":"lifecycle-state"}' --output table

# Chat inference (if enabled)
oci generative-ai-inference chat --compartment-id $COMPARTMENT \
  --chat-request '{"apiFormat":"GENERIC","messages":[{"role":"USER","content":[{"type":"TEXT","text":"Hello"}]}]}'
```

## Shadow Labs Infrastructure Map
```
shadowlab.cc (Cloudflare DNS)
├── @ → 76.76.21.21 (Vercel — marketing site)
├── www → cname.vercel-dns.com (Vercel)
├── api.shadowlab.cc → <OCI_IP> (Inference API — Caddy + FastAPI)
└── sd-astemo.shadowlab.cc → <OCI_IP> (MCP Server — Caddy + FastMCP)

OCI Always Free Instance:
├── 4 OCPU ARM A1, 24GB RAM, Ubuntu 22.04
├── Caddy (auto-SSL, reverse proxy)
├── shadowlabs-api (port 8787) — 5 inference services
└── slabs-astemo-sd (port 8788) — 19 MCP tools
```

## Environment
- Instance config: `/Volumes/SHADOW/shadowlabs-api/deploy/.oci-instance.env`
- OCI config: `~/.oci/config`
- SSH key: `~/.ssh/shadowlabs-oci`
- API source: `/Volumes/SHADOW/shadowlabs-api`
- MCP source: `/Volumes/SHADOW/slabs-astemo-sd`

# Tailscale Split DNS Server

Lightweight DNS server that makes `*.erodev.cloud` domains resolve to Tailscale IP addresses instead of public IPs. This enables secure access to services over Tailscale without exposing them to the public internet.

> Test after GitHub re-authentication

## Overview

**Problem Solved:**
- You want to use friendly domain names like `n8n.erodev.cloud`
- But only accessible via Tailscale (private network)
- Without changing public DNS or opening firewall ports

**Solution:**
- CoreDNS server listens ONLY on Tailscale interface (`100.116.229.118:53`)
- Tailscale clients are configured to use this DNS for `erodev.cloud` queries
- All `*.erodev.cloud` domains resolve to Tailscale IP
- Public internet still sees the public IP (or nothing)

## Architecture

```
Tailscale Client (your laptop)
    ↓
    Query: "What's n8n.erodev.cloud?"
    ↓
Tailscale (sees domain restriction: erodev.cloud)
    ↓
    Forwards to: 100.116.229.118:53
    ↓
CoreDNS (this server)
    ↓
    Returns: 100.116.229.118
    ↓
Browser navigates to: http://100.116.229.118:5678
    (or https if Traefik handles it)
```

## Services Configured

Current DNS records (edit `dnsmasq.conf` to add more):

| Domain | Resolves To | Service |
|--------|-------------|---------|
| `n8n.erodev.cloud` | `100.116.229.118` | n8n Workflow Automation |
| `dokploy.erodev.cloud` | `100.116.229.118` | Dokploy Admin Panel |
| `cops.erodev.cloud` | `100.116.229.118` | COPS Application |
| `erodev.cloud` | `100.116.229.118` | Base domain |

## Deployment

### Option 1: Deploy via Dokploy (Recommended)

1. **Create GitHub Repository** (or use existing):
   ```bash
   cd /home/erousr/dev/ops/tailscale-dns
   git init
   git add .
   git commit -m "Initial Tailscale DNS server setup"
   git remote add origin git@github.com:YourUsername/tailscale-dns.git
   git push -u origin main
   ```

2. **In Dokploy UI**:
   - Create new project: `tailscale-dns`
   - Add Compose service
   - Connect to GitHub repo
   - Deploy

### Option 2: Manual Docker Deployment

```bash
# On your VPS
cd ~/tailscale-dns

# Start the DNS server
docker-compose up -d

# Check logs
docker-compose logs -f

# Test locally
nslookup n8n.erodev.cloud 100.116.229.118
```

## Tailscale Configuration

After deploying the DNS server:

1. **Go to**: https://login.tailscale.com/admin/dns

2. **Click**: "Add nameserver" → "Custom"

3. **Configure**:
   - **Nameserver**: `100.116.229.118`
   - **Restrict to domain**: `erodev.cloud`
   - ✅ Check **"Split DNS"**
   - Click **"Save"**

4. **Test from your Tailscale-connected device**:
   ```bash
   # Should return 100.116.229.118
   nslookup n8n.erodev.cloud

   # Should work
   curl http://n8n.erodev.cloud:5678
   ```

## Adding New Services

**No configuration needed!** CoreDNS automatically resolves ALL `*.erodev.cloud` domains to the Tailscale IP (`100.116.229.118`).

Just configure your service in Traefik/Dokploy with the desired domain name, and it will work immediately.

## Security

- ✅ **DNS server ONLY listens on Tailscale IP** (not 0.0.0.0)
- ✅ **Public internet cannot query this DNS server**
- ✅ **Services remain protected by Tailscale ACLs**
- ✅ **No public DNS changes required**
- ✅ **No firewall port openings needed**

## Troubleshooting

### DNS not resolving

```bash
# Check if DNS server is running
docker ps | grep tailscale-dns

# Check logs
docker logs tailscale-dns

# Test DNS directly
dig @100.116.229.118 n8n.erodev.cloud

# Check Tailscale DNS settings
tailscale status --json | jq .MagicDNS
```

### Services not accessible

1. Verify Tailscale is connected: `tailscale status`
2. Verify DNS resolves correctly: `nslookup n8n.erodev.cloud`
3. Verify service is running: `docker ps`
4. Check if port is correct: `ss -tlnp | grep 5678`

## Configuration Files

- [docker-compose.yml](docker-compose.yml) - Docker Compose configuration
- [Corefile](Corefile) - CoreDNS configuration (wildcard DNS for *.erodev.cloud)
- `.env.example` - Environment variables template (optional)

## Maintenance

### View Logs
```bash
docker-compose logs -f
```

### Restart DNS Server
```bash
docker-compose restart
```

### Stop DNS Server
```bash
docker-compose down
```

### Update Configuration
1. Edit `Corefile` (if needed - default config handles all *.erodev.cloud)
2. Run `docker-compose restart`

## Integration with Traefik

If you want Traefik to respond to `*.erodev.cloud` domains over Tailscale, add additional domain configurations in Dokploy for each service with the `*.erodev.cloud` domain.

This DNS server just handles name resolution - Traefik handles the actual routing.

## Resources

- [Tailscale Split DNS Documentation](https://tailscale.com/kb/1054/dns/)
- [CoreDNS Documentation](https://coredns.io/manual/toc/)
- [CoreDNS Plugins](https://coredns.io/plugins/)

## License

MIT - Feel free to modify and reuse

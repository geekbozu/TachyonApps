# Caddy Reverse Proxy

HTTPS reverse proxy for all TachyonApps services using Let's Encrypt wildcard certificates with DNS-01 validation via Porkbun.

> **Note:** This README uses **AdGuard Home** as the canonical, stable example service for this stack.

## Services Proxied

| Internal Service | External URL | Port |
|-----------------|--------------|------|
| adguardhome:3000 | https://adguard.internal.keepdream.in | 443 |
| wg-easy:51821 | https://wireguard.internal.keepdream.in | 443 |
| open-notebook:8502 | https://notebook.internal.keepdream.in | 443 |
| open-notebook:5055 | https://notebook-api.internal.keepdream.in | 443 |

## Configuration

The Caddyfile is generated inline by the `caddy-config` init container and written to `/mnt/sdcard/caddy/etc/Caddyfile`.

**Currently configured for Let's Encrypt STAGING environment** for testing. Staging certificates won't be trusted by browsers but let you test the DNS-01 challenge without hitting rate limits.

### Environment Variables

Create `/mnt/sdcard/caddy/.env` with your Porkbun API credentials and Let's Encrypt email:

```env
PORKBUN_API_KEY=pk1_your_api_key_here
PORKBUN_SECRET_KEY=sk1_your_secret_key_here
LETSENCRYPT_EMAIL=your-email@example.com
```

The custom Caddy image includes the Porkbun DNS plugin for DNS-01 challenge validation.

### Switching to Production

Once staging certificates are working, remove the `acme_ca` line from the Caddyfile in [docker-compose.yml](caddy/docker-compose.yml) to use production Let's Encrypt certificates.

## Network

All proxied services must join the `proxy-network` Docker network to be reachable by Caddy.

## Deployment

```powershell
$env:DOCKER_HOST = "npipe:////./pipe/podman-machine-default"
cd Caddy
particle container push --device <YOUR_DEVICE_ID>
```

## Storage

- `/mnt/sdcard/caddy/etc/` - Caddyfile (generated)
- `/mnt/sdcard/caddy/data/` - Let's Encrypt certificates and account data
- `/mnt/sdcard/caddy/config/` - Caddy config state
- `/mnt/sdcard/caddy/.env` - Porkbun API credentials (create this file)

## DNS Setup

**External DNS (Porkbun):**
- `A` record: `*.internal.keepdream.in` → Your home public IP
- `A` record: `internal.keepdream.in` → Same IP

**Internal DNS (AdGuard Home):**
- `*.internal.keepdream.in` → Local Tachyon IP (e.g., 192.168.68.73)
- Auto-provisioned by AdGuard on deployment

**How it works:**
1. Caddy requests wildcard cert from Let's Encrypt using DNS-01 challenge
2. Porkbun API automatically creates/removes DNS TXT records for validation
3. AdGuard Home rewrites internal DNS queries to local IP
4. WireGuard clients get valid certificates with traffic staying on LAN

**No port forwarding required** - DNS-01 challenge uses Porkbun API only.

## Certificate Management

Currently using **Let's Encrypt STAGING** to test DNS-01 validation:
- Staging certs show browser warnings (expected)
- Test that certificate issuance works
- Check logs: `docker logs -f caddy`

**To switch to production certificates:**
1. Verify staging certs are issued successfully
2. Remove the `acme_ca https://acme-staging-v02.api.letsencrypt.org/directory` line from Caddyfile in [docker-compose.yml](caddy/docker-compose.yml#L41)
3. Redeploy: `particle container push --device <YOUR_DEVICE_ID>`
4. Caddy will automatically obtain trusted production certificates

## Troubleshooting

**Certificate issuance fails:**
- Verify Porkbun API credentials in `/mnt/sdcard/caddy/.env`
- Check DNS propagation: `nslookup *.internal.keepdream.in`
- Review Caddy logs: `docker logs -f caddy`

**Browser shows certificate error:**
- If using staging: Expected (Fake LE Intermediate X1)
- If using production: Check certificate issuer and expiry

**Services unreachable:**
- Verify AdGuard DNS rewrites are provisioned
- Check all services are on `proxy-network`
- Test direct IP access to rule out reverse proxy issues

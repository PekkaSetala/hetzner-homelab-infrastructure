# Cloudflare Integration Documentation

This document explains how Cloudflare is integrated with the self-hosted infrastructure to provide DNS management, DDoS protection, and CDN capabilities.

## Overview

Cloudflare sits between your users and your Hetzner server, providing:
- **DNS Management** - Authoritative nameservers for your domain
- **DDoS Protection** - Automatic mitigation of attacks
- **CDN/Proxy** - Content caching and SSL/TLS encryption
- **Web Application Firewall** - Protection against common exploits
- **Analytics** - Traffic insights and performance metrics

## Architecture Flow
```
User Request
    ↓
Cloudflare DNS (resolves domain)
    ↓
Cloudflare Proxy (orange cloud)
    ↓
SSL/TLS Termination
    ↓
DDoS Protection Layer
    ↓
Cloudflare Edge Network
    ↓
Origin Server (Hetzner)
    ↓
Traefik Reverse Proxy
    ↓
Application Containers
```

## DNS Configuration

### Nameservers

After adding your domain to Cloudflare, update your domain registrar with Cloudflare's nameservers:

**Example Cloudflare Nameservers:**
```
alice.ns.cloudflare.com
bob.ns.cloudflare.com
```

**Propagation Time:** Usually 5-30 minutes, can take up to 48 hours

**Verification:**
```bash
# Check nameservers
dig example.com NS +short

# Should return Cloudflare nameservers
```

### DNS Records

All services use A records pointing to your Hetzner server IP:

| Type | Name | Content | Proxy Status | TTL |
|------|------|---------|--------------|-----|
| A | n8n | YOUR_SERVER_IP | Proxied ☁️ | Auto |
| A | ui | YOUR_SERVER_IP | Proxied ☁️ | Auto |
| A | app | YOUR_SERVER_IP | Proxied ☁️ | Auto |
| A | @ | YOUR_SERVER_IP | Proxied ☁️ | Auto |
| A | www | YOUR_SERVER_IP | Proxied ☁️ | Auto |

**Proxy Status:**
- **Proxied (Orange Cloud)** ☁️ - Traffic routes through Cloudflare
  - Hides origin server IP
  - Enables DDoS protection
  - Provides SSL/TLS encryption
  - Uses Cloudflare's CDN
  
- **DNS Only (Gray Cloud)** - Direct connection to origin
  - Exposes server IP
  - No DDoS protection
  - No CDN benefits
  - Use only when necessary (e.g., mail servers)

**Recommendation:** Always use Proxied mode for web services.

### Wildcard DNS (Optional)

For multiple subdomains:

| Type | Name | Content | Proxy Status |
|------|------|---------|--------------|
| A | * | YOUR_SERVER_IP | Proxied ☁️ |

**Note:** Wildcard SSL certificates require Cloudflare's Advanced Certificate Manager (paid feature) or use specific subdomain records.

## SSL/TLS Configuration

### SSL/TLS Encryption Mode

Navigate to: **SSL/TLS → Overview**

**Available Modes:**

1. **Off** ❌ - No encryption (not recommended)
2. **Flexible** ⚠️ - Cloudflare → User encrypted, Origin → Cloudflare unencrypted
3. **Full** ✅ - End-to-end encryption, self-signed cert on origin OK
4. **Full (Strict)** ✅✅ - End-to-end encryption, valid cert required on origin

**Recommended for this setup:** **Full** or **Full (Strict)**

Our Traefik setup uses Let's Encrypt certificates, so Full (Strict) is ideal.

### Configuration Steps

1. Go to **SSL/TLS → Overview**
2. Select **"Full (strict)"**
3. Enable these settings:
   - ✅ Always Use HTTPS
   - ✅ Automatic HTTPS Rewrites
   - ✅ Opportunistic Encryption
   - ✅ TLS 1.3

### Edge Certificates

Navigate to: **SSL/TLS → Edge Certificates**

**Settings:**
- ✅ Always Use HTTPS
- ✅ HTTP Strict Transport Security (HSTS) - Optional
- ✅ Minimum TLS Version: 1.2
- ✅ Opportunistic Encryption
- ✅ TLS 1.3
- ✅ Automatic HTTPS Rewrites

**Certificate Details:**
- **Type:** Universal SSL (Free)
- **Validity:** 1 year (auto-renewed)
- **Covers:** example.com, *.example.com

### Origin Certificates (Alternative)

For even stronger security, you can use Cloudflare Origin Certificates:

1. Go to **SSL/TLS → Origin Server**
2. Click **"Create Certificate"**
3. Select hostnames to cover
4. Choose validity period (up to 15 years)
5. Download certificate and private key
6. Install on Traefik (instead of Let's Encrypt)

**Note:** Origin Certificates only work with Cloudflare proxy enabled.

## Firewall Rules

Navigate to: **Security → WAF**

### Recommended Rules

**Block Known Bad Bots:**
```
Field: Known Bots
Operator: equals
Value: On
Action: Block
```

**Rate Limiting (Requires paid plan):**
```
If: Requests exceed 100 requests per 10 minutes
From: Same IP
Then: Challenge or Block
```

**Geographic Restrictions (if needed):**
```
Field: Country
Operator: does not equal
Value: [Your allowed countries]
Action: Block
```

### Security Level

Navigate to: **Security → Settings**

**Options:**
- **Essentially Off** - Very permissive
- **Low** - Challenges only most threatening visitors
- **Medium** - Standard (recommended)
- **High** - Challenges more visitors
- **I'm Under Attack!** - Very aggressive (use during attacks only)

**Recommended:** Medium

## Page Rules (Free: 3 rules)

Navigate to: **Rules → Page Rules**

### Example Rules

**Force HTTPS:**
```
URL: http://*example.com/*
Settings: Always Use HTTPS
```

**Cache Everything (for static assets):**
```
URL: example.com/assets/*
Settings:
  - Cache Level: Cache Everything
  - Edge Cache TTL: 1 month
```

**Bypass Cache (for API endpoints):**
```
URL: n8n.example.com/webhook/*
Settings:
  - Cache Level: Bypass
```

## Speed Optimization

### Auto Minify

Navigate to: **Speed → Optimization**

Enable auto minification:
- ✅ JavaScript
- ✅ CSS
- ✅ HTML

**Note:** Test thoroughly as minification can sometimes break websites.

### Brotli Compression

Navigate to: **Speed → Optimization**

- ✅ Enable Brotli (better than Gzip)

Traefik also compresses with Gzip, so this provides double compression benefits.

### HTTP/2 and HTTP/3

Navigate to: **Network**

- ✅ HTTP/2 (enabled by default)
- ✅ HTTP/3 (QUIC) - Enable this
- ✅ 0-RTT Connection Resumption
- ✅ WebSockets - Enable for Coolify real-time features

## Caching Configuration

### Browser Cache TTL

Navigate to: **Caching → Configuration**

**Settings:**
- **Browser Cache TTL:** 4 hours (or "Respect Existing Headers")

This controls how long browsers cache content.

### Development Mode

When testing:
1. Enable **Development Mode** (lasts 3 hours)
2. Bypasses Cloudflare cache
3. Automatically disables after 3 hours

## Analytics and Monitoring

### Traffic Analytics

Navigate to: **Analytics & Logs → Traffic**

View:
- Total requests
- Bandwidth usage
- Unique visitors
- Requests by country
- Status codes distribution

### Security Analytics

Navigate to: **Security → Overview**

Monitor:
- Threats blocked
- Challenge solve rate
- Top threat countries
- Attack types

### Performance Insights

Navigate to: **Speed → Observatory**

Check:
- Page load time
- Core Web Vitals
- Performance score

## DNS Records for Email (Optional)

If you want to send email from your domain:

### SPF Record
```
Type: TXT
Name: @
Content: v=spf1 include:_spf.example.com ~all
```

### DMARC Record
```
Type: TXT
Name: _dmarc
Content: v=DMARC1; p=none; rua=mailto:postmaster@example.com
```

### MX Records
```
Type: MX
Name: @
Priority: 10
Content: mail.example.com
```

**Note:** Gray cloud (DNS only) MX records to avoid proxy issues.

## Troubleshooting

### DNS Not Resolving
```bash
# Check if DNS is propagated
dig n8n.example.com +short

# Should return Cloudflare IPs (not your server IP)
# Example: 104.21.x.x or 172.67.x.x

# Check nameservers
dig example.com NS +short

# Should return Cloudflare nameservers
```

### SSL Certificate Errors

**"Your connection is not private" error:**

1. Verify SSL/TLS mode is set to **Full** or **Full (strict)**
2. Check Traefik has issued Let's Encrypt certificate:
```bash
   docker exec coolify-proxy cat /traefik/acme.json | jq .
```
3. Verify domain in Traefik logs:
```bash
   docker logs coolify-proxy | grep -i certificate
```
4. Wait 5 minutes for certificate provisioning

**Mixed content warnings:**

1. Enable **Automatic HTTPS Rewrites**
2. Check application loads resources over HTTPS

### Redirect Loop (ERR_TOO_MANY_REDIRECTS)

**Causes:**
1. SSL/TLS mode set to **Flexible** with HTTPS redirect on origin
2. Page Rule creating infinite redirect

**Fix:**
1. Change SSL/TLS mode to **Full** or **Full (strict)**
2. Review Page Rules for conflicting redirects
3. Clear browser cache and cookies

### 502 Bad Gateway

**Cloudflare 502 means:**
- Origin server is down
- Origin server rejected request
- DNS points to wrong IP

**Troubleshooting:**
```bash
# Verify server is running
ssh root@YOUR_SERVER_IP
docker ps

# Check if Traefik is running
docker logs coolify-proxy

# Verify DNS points to correct IP
dig n8n.example.com A +short
```

### 525 SSL Handshake Failed

**Causes:**
- SSL/TLS mode is Full (strict) but origin has invalid cert
- Traefik not issuing certificates properly

**Fix:**
1. Temporarily change SSL/TLS mode to **Full**
2. Verify Let's Encrypt is working:
```bash
   docker logs coolify-proxy | grep acme
```
3. Once cert is issued, switch back to **Full (strict)**

## Performance Optimization Tips

### 1. Enable Argo Smart Routing (Paid)
- Reduces latency by up to 30%
- Routes through fastest Cloudflare path
- Cost: $5/month + $0.10 per GB

### 2. Use Cloudflare Workers (Advanced)
- Edge computing for dynamic content
- Process requests at Cloudflare edge
- Free tier: 100,000 requests/day

### 3. Polish Images (Paid)
- Automatic image optimization
- Converts to WebP format
- Reduces bandwidth usage

### 4. Zaraz (Free)
- Load third-party tools efficiently
- Reduces page load time
- Privacy-focused analytics

## Security Best Practices

### 1. Enable Bot Fight Mode
Navigate to: **Security → Bots**
- ✅ Enable Bot Fight Mode (Free)
- Challenges suspected bots automatically

### 2. Enable DNSSEC
Navigate to: **DNS → Settings**
- ✅ Enable DNSSEC
- Protects against DNS spoofing

### 3. Configure Firewall Rules
- Block known bad actors
- Challenge suspicious traffic
- Rate limit aggressive crawlers

### 4. Enable Security Headers
Add to origin server (Traefik):
```yaml
# X-Frame-Options
# X-Content-Type-Options
# X-XSS-Protection
# Strict-Transport-Security
```

### 5. Review Security Events Regularly
- Check **Security → Events**
- Analyze blocked threats
- Adjust rules as needed

## API Access (Optional)

For programmatic DNS management:

1. Get API Token:
   - Go to **Profile → API Tokens**
   - Create token with **Zone DNS Edit** permissions
   
2. Use Cloudflare API:
```bash
# Example: List DNS records
curl -X GET "https://api.cloudflare.com/client/v4/zones/{zone_id}/dns_records" \
  -H "Authorization: Bearer {api_token}" \
  -H "Content-Type: application/json"
```

## Cost Breakdown

**Cloudflare Free Plan Includes:**
- Unlimited DNS queries
- Unlimited DDoS mitigation
- Universal SSL certificate
- Global CDN
- 3 Page Rules
- Web Application Firewall (basic)

**Paid Features (Optional):**
- **Pro ($20/month):**
  - Image optimization (Polish)
  - Mobile optimization
  - WAF custom rules (5 rules)
  
- **Business ($200/month):**
  - Advanced DDoS
  - 100% uptime SLA
  - Prioritized email support
  - Custom SSL certificates

**Our Setup:** Free plan is sufficient for personal/small team use.

## Monitoring and Alerts

### Email Notifications

1. Go to **Notifications**
2. Set up alerts for:
   - ✅ Origin errors (5xx)
   - ✅ SSL certificate expiration
   - ✅ DDoS attacks detected
   - ✅ Health check failures

### Third-Party Monitoring

Consider external monitoring:
- **UptimeRobot** - Free uptime monitoring
- **Pingdom** - Performance monitoring
- **StatusCake** - Multi-location checks

## Backup DNS Configuration

Always keep a backup of your DNS records:
```bash
# Export DNS records (requires API token)
curl -X GET "https://api.cloudflare.com/client/v4/zones/{zone_id}/dns_records" \
  -H "Authorization: Bearer {api_token}" > dns-backup.json
```

Or manually document in a spreadsheet:
```
Type | Name | Content | Proxy | TTL
A    | n8n  | X.X.X.X | Yes   | Auto
A    | ui   | X.X.X.X | Yes   | Auto
...
```

## Migration Checklist

When moving to Cloudflare:

- [ ] Add domain to Cloudflare
- [ ] Create DNS records (A, CNAME, etc.)
- [ ] Set SSL/TLS mode to Full (strict)
- [ ] Enable Always Use HTTPS
- [ ] Update nameservers at registrar
- [ ] Wait for DNS propagation (24-48h)
- [ ] Test all subdomains
- [ ] Enable security features (Bot Fight Mode, etc.)
- [ ] Configure Page Rules
- [ ] Set up monitoring alerts
- [ ] Document configuration

---

**Benefits of Cloudflare Integration:**
- ✅ Hide origin server IP (security)
- ✅ Free SSL certificates
- ✅ DDoS protection
- ✅ Global CDN (faster load times)
- ✅ Web Application Firewall
- ✅ Analytics and insights
- ✅ 99.99% DNS uptime

**Setup Time:** 30-60 minutes (including DNS propagation)


# DNS — Domain Name System

## What is DNS?

DNS (Domain Name System) is the internet's phonebook. It translates human-readable domain names like `www.example.com` into IP addresses like `93.184.216.34` that computers use to communicate with each other.

Without DNS, you'd need to memorize numeric IP addresses to visit every website.

---

## How DNS Works

When you type a URL into your browser, a **DNS resolution** process begins:

```
Browser → Recursive Resolver → Root Nameserver → TLD Nameserver → Authoritative Nameserver
```

1. **Browser Cache** — Checks if the IP is already cached locally.
2. **Recursive Resolver** — Your ISP or a public resolver (e.g., `8.8.8.8`) handles the lookup.
3. **Root Nameserver** — Directs the resolver to the correct TLD (`.com`, `.org`, `.net`).
4. **TLD Nameserver** — Points to the domain's authoritative nameserver.
5. **Authoritative Nameserver** — Returns the final IP address for the domain.

---

## DNS Record Types

|Record|Purpose|Example|
|---|---|---|
|`A`|Maps domain → IPv4 address|`example.com → 93.184.216.34`|
|`AAAA`|Maps domain → IPv6 address|`example.com → 2606:2800::1`|
|`CNAME`|Alias pointing to another domain|`www → example.com`|
|`MX`|Mail server for the domain|`mail.example.com`|
|`TXT`|Arbitrary text (SPF, DKIM, verification)|`v=spf1 include:...`|
|`NS`|Nameservers for the domain|`ns1.example.com`|
|`SOA`|Start of Authority — zone metadata|Serial, refresh, retry info|
|`PTR`|Reverse DNS — IP → domain name|Used for spam filtering|
|`SRV`|Service location (port + host)|Used by SIP, XMPP, etc.|
|`CAA`|Certificate Authority Authorization|Restricts SSL issuers|

---

## Key Concepts

### TTL (Time to Live)

Every DNS record has a TTL value (in seconds) that controls how long resolvers cache it. Lower TTL = faster propagation of changes, but more DNS queries.

### DNS Propagation

After updating DNS records, changes can take **minutes to 48 hours** to propagate worldwide due to caching at various levels.

### DNS Zones

A DNS zone is a portion of the DNS namespace managed by a specific organization or administrator. Zone files contain all the records for a domain.

### Recursive vs. Authoritative DNS

- **Recursive resolver** — Asks other servers on your behalf (e.g., `1.1.1.1`, `8.8.8.8`).
- **Authoritative nameserver** — The final source of truth for a specific domain.

---

## Public DNS Resolvers

|Provider|IPv4|IPv6|
|---|---|---|
|Google|`8.8.8.8`, `8.8.4.4`|`2001:4860:4860::8888`|
|Cloudflare|`1.1.1.1`, `1.0.0.1`|`2606:4700:4700::1111`|
|OpenDNS|`208.67.222.222`|`2620:119:35::35`|
|Quad9|`9.9.9.9`|`2620:fe::fe`|

---

## DNS Security

### DNSSEC

DNS Security Extensions add cryptographic signatures to DNS records, preventing attackers from spoofing responses (DNS cache poisoning).

### DNS over HTTPS (DoH)

Encrypts DNS queries using HTTPS, preventing eavesdropping and man-in-the-middle attacks. Supported by browsers like Firefox and Chrome.

### DNS over TLS (DoT)

Similar to DoH but uses TLS on port 853. Preferred by network administrators since it's easier to monitor.

### Common Attacks

- **DNS Spoofing / Cache Poisoning** — Injecting fake records into a resolver's cache.
- **DNS Hijacking** — Redirecting DNS queries to a rogue server.
- **DDoS via DNS Amplification** — Using open resolvers to amplify attack traffic.
- **Typosquatting** — Registering domains similar to legitimate ones (e.g., `gooogle.com`).

---

## Useful DNS Commands

```bash
# Look up A record
nslookup example.com

# Detailed DNS query
dig example.com

# Query a specific record type
dig example.com MX
dig example.com TXT

# Use a specific DNS server
dig @8.8.8.8 example.com

# Reverse DNS lookup
dig -x 93.184.216.34

# Trace the full resolution path
dig +trace example.com

# Check DNS propagation (Linux/macOS)
host example.com
```

---

## DNS Hierarchy

```
.                          ← Root
├── .com
│   ├── example.com
│   │   ├── www.example.com
│   │   └── mail.example.com
│   └── google.com
├── .org
│   └── wikipedia.org
└── .net
    └── cloudflare.net
```

---

# CKA 2024 Video 30: DNS Fundamentals (Resolution, Caching, Root Servers, Records, Troubleshooting Files)

## Opening: playful “two P’s” + topic setup

# 

- Host introduces the **30th video** of the CKA 2024 series: a deep dive into **DNS**.
- A humorous exchange occurs where another “P” appears (“who are you?” / “I’m also P”), then they proceed together.
- They set expectations:
- cover *what DNS is*, *why it’s needed*, *how it works*, and *how to troubleshoot DNS issues* (useful for developers, DevOps, cloud engineers).

## What DNS stands for + the basic idea

# 
- They Google it:
- DNS = **Domain Name System** / **Domain Name Server** (speaker says you can call it either way).
- Key observation example:
- typing `google.com` results in DNS translating the **domain name** into an **IP address**.

### Why do we need DNS? (house address analogy)

# 
- Speaker uses an analogy:
- If the “name” of a house is “G Villa,” you still can’t reach it without a full address (street, city, PIN code).
- Internet parallel:
- to reach any machine on the internet, you need its **unique IP address**.
- Human memory limitation:
- it’s hard to remember many IP addresses.
- IPs can change:
- e.g., Google might migrate services and IPs may change.
- DNS value:
- domain names are “friendly names” on top of IPs (like saving a phone number under a contact name).

## High-level DNS resolution flow (browser → DNS → IP)

# 
- When you type `google.com`:

1. Browser doesn’t inherently “know” where `google.com` is.
2. Browser queries a DNS server (described as a “public book” of records):
- “Tell me the IP for google.com.”
3. DNS server searches its records and returns the IP.
4. Browser forwards the request to that IP.
- Definition-style phrasing (close paraphrase):

- *“DNS resolution” is resolving a domain name to an IP address.*

## Two big DNS problems: latency and scale / single point of failure

### Problem 1: extra hop (latency)

# 

- Even if the DNS server is powerful:
- every website visit would require an extra request to DNS, adding latency.
- Non-technical users might interpret this as “internet is slow.”

### Problem 2: one DNS server can’t handle the whole internet + SPOF

# 
- Billions of users and countless websites → one server can’t handle the load.
- Even if it could, it becomes a **single point of failure**:
- if it goes down (e.g., natural calamity), “the whole internet will break.”

## DNS caching: multiple layers reduce repeated lookups

# 
- They introduce DNS caching to reduce repeated DNS queries.
- Definition-style phrasing:
- *DNS caching means once the DNS is resolved, it gets “cached,” so you don’t query DNS again and again.*
- Multiple caching layers mentioned:
- **Browser cache**
- **Operating system cache**
- **Router cache**
- **ISP cache**

### Live example: first visit vs refresh

# 
- They test with a website the host likely hasn’t visited before (example: `cloudopscommunity.org`).
- In browser network timings:
- first visit shows a **DNS lookup**
- refresh removes DNS lookup because the browser cached it.
- They show IP details:
- the request goes to an IP on port **443** (HTTPS).
- They run:
- `nslookup cloudopscommunity.org`
- show it returns an IP (they note an IPv4 is returned here).
- They mention the site appears to be behind **Cloudflare** (Cloudflare returns the IP).

## Decentralized DNS: root servers + hierarchy

### Root name servers (13)

# 

- They explain DNS is decentralized using **root name servers**:

- *There are “13 root name servers.”*
- Clarification:

- although there are 13 named root servers, each can represent many backend servers:
- one root “name” can map to multiple servers that “advertise the same IP.”
- described as “load balancer” behavior.
- Ownership examples they point out:

- different organizations maintain them (e.g., University of Southern California, NASA, etc.).

### DNS hierarchy: Root → TLD → authoritative

# 
- Flow described:

1. request goes to a **root name server** closest to you
2. then to the **TLD (Top Level Domain)** server (e.g., `.dev`, `.com`, `.org`)
3. then to the **authoritative name server** (registrar/provider like Google Domains, Cloudflare, GoDaddy)
4. final IP is returned; caching happens “on every level.”
- They define TLD by example:

- *TLD is like `.dev` / `.org` / `.com`* (they call `www` a subdomain and the TLD the suffix).

## Viewing and managing DNS records in Cloudflare

### ### Cloudflare (DNS dashboard)

# 

- They open a Cloudflare account and show DNS records for a domain (example domain: `pgar.xyz`).
- Shows existing records:
- requests to certain subdomains return specific IPs they configured.

## DNS record types explained (starting with the most common)

# 
They agree to cover “most important and commonly used” record types.

### ### A record (IPv4)

# 
- Definition-style phrasing:
- *An **A record** maps a domain/subdomain to an **IPv4 address**.*
- Notes:
- using `@` refers to the “apex/root domain.”
- adding a value like `www` makes it a subdomain.
- Meaning:
- when DNS finds the A record, it returns the IP and the browser forwards the request to that server.

### ### AAAA record (IPv6)

# 
- Definition-style phrasing:
- *“A is for IPv4” and “AAAA is for IPv6.”*

### ### CNAME record (alias to another domain)

# 
- Definition-style phrasing:
- *A **CNAME** points one name to another domain name; DNS then resolves that target domain to its A/AAAA record.*
- How it works (as described):
- `magic.pgar.xyz` → CNAME → `google.com`
- then DNS resolves `google.com` to its A record IP
- `magic.pgar.xyz` effectively resolves to the same IP as `google.com`

#### Live CNAME demonstration

# 
- They set `magic.pgar.xyz` to CNAME to `cloudopscommunity.org`.
- They run:
- `nslookup magic.pgar.xyz`
- It returns the same IP as the target domain, proving the alias behavior.

#### When to use CNAME (outsourcing to providers)

# 
- Use case given:
- pointing a subdomain to an external provider like **Vercel**.
- Problem with A record:
- hardcoding Vercel’s IP means your site breaks if Vercel changes IP (e.g., at 2 a.m.).
- Benefit of CNAME:
- *it becomes “dynamic in nature” because if provider changes their underlying IP, your domain follows automatically.*
- Another example mentioned:
- Cloudflare Workers provides a provider domain (e.g., `*.dev.cloudflare.com`), and you CNAME your custom domain to it so users don’t see the provider URL.

### ### MX record (Mail Exchange)

# 
- Definition-style phrasing:
- *MX records route email to the mail server (SMTP) that receives mail for your domain.*
- Example:
- company-style emails like `name@company.com`.

### ### TXT record

# 
- Described as:
- *“human readable text”* often used for verification/ownership checks.

### ### NS record (Name Server) — “advanced but powerful”

# 
- They call NS “advanced” and explain its purpose:
- *NS record delegates DNS management for a domain/subdomain to another DNS server.*
- Example described:
- Set NS for `abc.pgar.xyz` to `ns.something.com`.
- Meaning:
- all subdomains under `abc.pgar.xyz` should be resolved by that delegated nameserver.
- How it functions:
- resolver looks up NS delegation, then queries the delegated server.
- delegated server must expose **port 53/UDP** and answer DNS queries.
- Conclusion:
- *This is how you can “host your own DNS server”: run a DNS server, delegate via NS record, and respond to queries.*

## Troubleshooting: important local files/commands for DNS issues

# 
They shift to “important files” on local systems (Linux/Mac focus).

### ### /etc/hosts

# 
- They show:
- `cat /etc/hosts`
- Definition-style phrasing:
- *This file lets you add “local DNS records” (an internal A-record-like mapping).*
- Examples discussed:
- `localhost` resolves to `127.0.0.1` (and IPv6 localhost), via entries in `/etc/hosts`.
- can be used for internal testing domains without buying a real domain.

### ### /etc/resolv.conf

# 
- They show:
- `cat /etc/resolv.conf`
- Explains:
- it indicates which DNS resolver is being used (in their case, a local router).
- Mentions popular public DNS resolvers:
- Cloudflare: `1.1.1.1`
- Google: `8.8.8.8`

### ### nslookup (specifying a resolver)

# 
- They show you can run `nslookup` using a specific DNS server:
- queries Cloudflare DNS (port 53) or Google DNS directly.
- Suggestion:
- if DNS feels slow (e.g., router resolver slow), you can edit resolver settings to use a faster public DNS (e.g., Cloudflare `1.1.1.1`).

## Wrap-up + why this DNS primer exists in the series

# 
- Host explains why this DNS fundamentals video exists:
- next videos will cover Kubernetes networking topics like **CoreDNS**, so DNS basics are a prerequisite.
- They thank each other, mention future collaborations, and close the video.
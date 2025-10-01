[THIS CONFIG](https://raw.githubusercontent.com/Kingvornex/Iran-Clash-Serverless/refs/heads/main/config6.yml) is a **highly advanced, performance- and evasion-oriented network/proxy configuration** for **Clash/Clash.Meta** (or compatible clients). Its purpose is to optimize traffic routing, enhance privacy, bypass throttling, and fragment traffic to avoid detection, particularly for heavy services like YouTube and Google Video. Let’s break it down:

---

### **1. General Purpose**

* **Mixed-port:** Accepts both HTTP/SOCKS proxies on port 7890.
* **Allow-LAN:** Lets other devices in your LAN use this proxy.
* **Mode:** `rule` → traffic is routed according to defined rules.
* **Logging & Controller:** Info-level logs and local web controller for monitoring.
* **Profile options:** Stores selected nodes and fake-IP mappings.

---

### **2. DNS Setup**

* **Enhanced fake-IP DNS**:

  * Resolves blocked/targeted domains to fake IPs to bypass network filters.
  * `fake-ip-range` is set (`198.18.0.1/16`) for internal mapping.
  * Filters (`fake-ip-filter`) apply fake-IP only to ads and `.ir` domains.
* **Nameservers:** Cloudflare DNS (IPv4 + IPv6) with DoH support.
* **Fallback & hosts:** Ensures important services (Cloudflare DNS) are always reachable.

**Purpose:**

* Block ads, improve privacy, avoid DNS-based throttling, and maintain connectivity with key services.

---

### **3. Proxies**

#### **Fragmentation Proxies**

* `frag-hello-1`, `frag-small-1`, `frag-mid-1`, etc.
* Uses **TCP fragmentation**: splits packets into smaller fragments.
* `tcp-fragment` options control the number of packets, lengths, intervals.
* **Purpose:** Avoid DPI (Deep Packet Inspection) by splitting large packets, making detection or throttling harder.

#### **Fragmentation Chains**

* `frag-chain-skip → frag-chain-next → frag-chain-final`
* Configured for **chained fragmentation**.
* **Purpose:** Add complexity to traffic patterns; harder for ISPs or firewalls to detect patterns.

#### **UDP Noise Proxies**

* `udp-noises-v4` & `udp-noises-v6`
* Sends random UDP packets at intervals (`udp-noise`) to obscure traffic.
* **Purpose:** Evade detection of QUIC/UDP traffic and simulate normal UDP noise.

#### **Direct & Block Proxies**

* `direct-out`: pass traffic without modification.
* `block-out`: reject traffic to block unwanted destinations.

#### **Remote Proxies**

* Placeholder examples (`remote-vmess-1`, `remote-vmess-2`) for connecting to external secure tunnels.
* Supports TLS, WebSocket, and VMess protocol.

---

### **4. Proxy Groups**

* **Load-balance groups:**
  Distribute traffic across multiple proxies for reliability or evasion:

  * `FRAG_LB_ROUND`: Round-robin for fragmented traffic.
  * `FRAG_CHAIN_LB`: Consistent-hashing for fragment chains.
  * `UDP_NOISES_GROUP`: For QUIC/UDP traffic obfuscation.
  * `REMOTE_FALLBACKS`: Stick-session load-balancing among remote nodes.

* **Selection groups:**

  * `SUPER_CHAIN_SELECT`: Chooses between fragment chains, load-balance, or direct-out.
  * `YT_HEAVY`: Specifically routes YouTube/Google video traffic through complex fragmentation + chains.
  * `DEFAULT_GROUP`: Catch-all, prioritizing fragment/load-balance, UDP noise, or direct/block.

**Purpose:**

* Maximizes throughput and evasion for heavy services (video streaming, social media, CDNs) while maintaining fallback options.

---

### **5. Rules**

* **DNS:** Direct resolution locally.
* **Ads & private IPs:** Blocked.
* **Heavy services (YouTube, Google Video, socials, CDNs):** Routed via fragmentation chains or load-balance groups.
* **UDP/QUIC ports (443, 2053, 8443):** Routed via UDP noise proxies.
* **Cloud providers:** Certain IPs routed through fragment chains.
* **Fallback:** Default traffic routed via `DEFAULT_GROUP`.

**Purpose:**

* Aggressively route traffic to prevent throttling, improve streaming experience, obscure traffic patterns, and block unwanted domains.

---

### **6. Overall Purpose**

1. **Evade ISP throttling / DPI:** Fragmentation, UDP noise, and chain proxies make detection difficult.
2. **Improve performance and redundancy:** Load-balancing proxies for heavy services.
3. **Bypass geo-restrictions or firewall rules:** Via remote VMess/Trojan/SS proxies.
4. **Block ads & unwanted traffic:** Local fake-IP DNS and block-out rules.
5. **Optimize video/social media streaming:** Specific heavy-service rules like `YT_HEAVY`.

---

💡 **TL;DR:**
This config is essentially a **"super-heavy Mihomo" anti-throttling, anti-DPI, multi-chain/proxy load-balancing setup** for Clash/Clash.Meta, optimized for YouTube, Google, social media, and general obfuscation of network traffic, while also providing ad-blocking and fake-IP DNS for privacy.

---
```

                                 +------------------+
                                 |   LAN / Client   |
                                 | (devices on LAN) |
                                 +--------+---------+
                                          |
                                          | (connect to mixed-port:7890)
                                          v
                                 +--------+---------+
                                 |   Clash Engine   |
                                 |  mixed-port:7890 |
                                 +--------+---------+
                                          |
                 +------------------------+-------------------------+
                 |                        |                         |
                 |                        |                         |
                 v                        v                         v
        +--------+--------+      +--------+--------+        +-------+--------+
        |   Rule Engine   |----->|     DNS Layer    |        |  External      |
        |   (mode: rule)  |      | enable: true     |        | controller/UI  |
        +--------+--------+      | fake-ip / DoH    |        +-------+--------+
                 |               | fake-ip-range:   |                |
   (rules match -> choose group)  | 198.18.0.1/16    |                |
                 |               +--------+---------+                |
                 |                        |                          |
   +-------------+------------+           |                          |
   |                          |           | (DNS lookup)             |
   |                          |           v                          |
   |                          |   +-------+-------+                  |
   |                          |   | fake-ip mapping|                 |
   |                          |   | (ads/.ir filtered)|              |
   |                          |   +-------+-------+                  |
   |                          |           |                          |
   |                          |           v                          |
   |                          |   +-------+--------+                 |
   |                          |   | Upstream DoH/  |                 |
   |                          |   | 1.1.1.1, Cloudflare|              |
   |                          |   +-----------------+                 |
   |                          |                                       
   |                          |                                       
   v                          v                                       
+--+------------------+   +---+----------------------+               
| Rules examples:     |   |         Selected Proxy   |               
| - DST-PORT,53 ->    |   | (chosen by proxy-groups) |               
|   direct-out         |  |                         |               
| - youtube.com ->     |  |                         |               
|   YT_HEAVY ----------+->|                         |               
| - udp -> UDP_NOISES  |   +---------+---------------+               
| - MATCH -> DEFAULT_GROUP         |                               
+----------------------+           |                               
                                   |                               
                 +-----------------+------------------------------+
                 |                 |                              |
                 v                 v                              v
        +--------+--------+  +-----+------+               +-------+--------+
        |   YT_HEAVY LB   |  | FRAG_CHAIN_LB|             |  DEFAULT_GROUP |
        | (consistent-    |  | (consistent- |             |  (select)      |
        |  hashing)       |  |  hashing)    |             |  proxies:      |
        +----+----+-------+  +------+-------+             |  SUPER_CHAIN_..|
             |   |                  |                     |  UDP_NOISES..  |
  (YT rules)  |   |                  |                     |  direct-out     |
             v   v                  v                     |  block-out      |
   +---------+   +------------+  +--+--+--+                +-------+--------+
   | FRAG_LB_ROUND |           | FRAG_CHAIN (chained)          |
   | (round-robin) |           |  frag-chain-skip ->            |
   +---+----+------+           |  frag-chain-next ->            |
       |    |                  |  frag-chain-final              |
       |    |                  +--------------------------------+
       |    |                            ^
       |    | (round-robin across)       |
       |    +----------------------------+
       | (proxies: frag-hello, frag-small, frag-mid, frag-large)
       |
       |                              +----------------------+
       |                              | UDP_NOISES_GROUP     |
       |                              | (round-robin)        |
       |                              +----------+-----------+
       |                                         |
       |                       (for UDP/QUIC)    |
       |                                         v
       |                               +---------+---------+
       |                               | udp-noises-v4/v6  |
       |                               | (send random UDP  |
       |                               |  noise to obscure)|
       |                               +-------------------+
       |
       |                             +-------------------------+
       |                             | REMOTE_FALLBACKS (LB)   |
       |                             | (sticky-sessions)       |
       |                             +-----------+-------------+
       |                                         |
       |                                         v
       |                             +-----------+-------------+
       |                             | remote-vmess-1/2        |
       |                             | (actual external tunnels)|
       |                             +-----------+-------------+
       |                                         |
       | (final next-hop)                         +--> (or fallback to) direct-out
       v
+------+-------+
| direct-out   | ---> traffic leaves as-is
+--------------+
```
Special flows:
 - DNS (port 53) => direct-out (local resolver)
 - Ads / private IPs => block-out or direct-out (depending on rule)
 - Cloudflare/Cloudfront GEOIP => FRAG_CHAIN_LB (apply fragmentation chain)
 - MATCH => DEFAULT_GROUP (tries SUPER_CHAIN_SELECT -> UDP_NOISES -> direct -> block)

# clash meta rule types:
Clash.Meta offers an advanced rule-based system that allows users to define granular network traffic routing policies. In addition to the primary rule types, Clash.Meta supports various subtypes and conditions, enhancing flexibility and control.

---

## 🔧 Clash.Meta Supported Rule Types with Subtypes and Sub-Rules

### 1. **Domain-Based Rules**

* **DOMAIN** – Exact match for a domain.

  ```yaml
  DOMAIN,example.com,PROXY
  ```
* **DOMAIN-SUFFIX** – Matches domain suffix.

  ```yaml
  DOMAIN-SUFFIX,example.com,PROXY
  ```
* **DOMAIN-KEYWORD** – Matches domain containing a keyword.

  ```yaml
  DOMAIN-KEYWORD,example,PROXY
  ```
* **DOMAIN-WILDCARD** – Matches domains using wildcards (`*` / `?`).

  ```yaml
  DOMAIN-WILDCARD,*.example.com,PROXY
  ```
* **DOMAIN-REGEX** – Matches domains using regex.

  ```yaml
  DOMAIN-REGEX,^.*\.example\.com$,PROXY
  ```

---

### 2. **IP-Based Rules**

* **GEOIP / GEOIP6** – Match based on country code (IPv4 / IPv6).

  ```yaml
  GEOIP,CN,PROXY
  GEOIP6,CN,PROXY
  ```
* **IP-ASN** – Match based on ASN number.

  ```yaml
  IP-ASN,714,PROXY
  ```
* **IP-CIDR / IP-CIDR6** – Match IPv4 / IPv6 CIDR ranges.

  ```yaml
  IP-CIDR,192.168.1.0/24,PROXY
  IP-CIDR6,2001:db8::/32,PROXY
  ```
* **SRC-IP-CIDR / SRC-IP-CIDR6** – Match source IPv4 / IPv6 CIDR ranges.

  ```yaml
  SRC-IP-CIDR,192.168.1.100/32,PROXY
  SRC-IP-CIDR6,2001:db8::1/128,PROXY
  ```

---

### 3. **Port-Based Rules**

* **DST-PORT / SRC-PORT** – Match destination or source port.

  ```yaml
  DST-PORT,80,PROXY
  SRC-PORT,8080,PROXY
  ```

---

### 4. **Process-Based Rules**

* **PROCESS-NAME** – Match traffic by process name.

  ```yaml
  PROCESS-NAME,chrome.exe,PROXY
  ```
* **PROCESS-PATH** – Match traffic by process path.

  ```yaml
  PROCESS-PATH,/usr/bin/chrome,PROXY
  ```

---

### 5. **Protocol / Network Rules**

* **NETWORK** – Match network type (`TCP` / `UDP`).

  ```yaml
  NETWORK,TCP,PROXY
  NETWORK,UDP,DIRECT
  ```
* **PROTOCOL** – Match protocol (HTTP / TLS / QUIC, etc.)

  ```yaml
  PROTOCOL,HTTP,PROXY
  ```

---

### 6. **Logical / Composite Rules**

* **AND / OR / NOT** – Combine multiple conditions.

  ```yaml
  AND,DOMAIN,example.com,DST-PORT,443,PROXY
  OR,DOMAIN,example.com,DST-PORT,443,PROXY
  NOT,DOMAIN,example.com,PROXY
  ```

---

### 7. **Sub-Rules (`SUB-RULE`)**

Sub-rules allow branching traffic to a named rule set for further matching. Useful for complex conditions like TCP vs UDP or different IP ranges.

#### **Usage Example:**

```yaml
rules:
  - SUB-RULE,(OR,((NETWORK,TCP),(NETWORK,UDP))),sub-rule-name1 # Match TCP or UDP, go to sub-rule-name1
  - SUB-RULE,(AND,((NETWORK,UDP))),sub-rule-name2            # Match UDP only, go to sub-rule-name2
```

#### **Sub-rule Sets:**

```yaml
sub-rules:
  sub-rule-name1:
    - DOMAIN,google.com,ss1
    - DOMAIN,baidu.com,DIRECT
  sub-rule-name2:
    - IP-CIDR,1.1.1.1/32,REJECT
    - IP-CIDR,8.8.8.8/32,ss1
    - DOMAIN,dns.alidns.com,REJECT
```

**💡 Explanation / Flow:**

```
https://baidu.com  --> rule1 --> rule2 --> sub-rule-name1(match tcp)  --> use DIRECT
google.com(not match)--> baidu.com(match)  --> sub-rule-name1(match tcp)

dns 1.1.1.1 --> rule1 --> rule2 --> sub-rule-name1(match udp) --> sub-rule-name2(match udp) --> REJECT
```

* `SUB-RULE` allows multiple branches in one rule using logical conditions (`AND` / `OR`).
* Each branch can point to its own **sub-rule set**.
* Sub-rules can themselves contain standard rules (DOMAIN, IP-CIDR, etc.) for further granularity.

---

### 8. **Other Rules**

* **RULE-SET** – Use external rule lists.
* **SCRIPT** – Use scripts to match traffic.
* **MATCH** – Catch-all for unmatched traffic.

  ```yaml
  MATCH,REJECT
  ```

---

For more detailed information and advanced configurations, you can refer to the official Clash.Meta documentation. ([GitHub][1])

[1]: https://github.com/djoeni/Clash.Meta?utm_source=chatgpt.com "djoeni/Clash.Meta: A rule-based tunnel in Go."

---

# Clash.Meta Inbound Traffic Configuration

This configuration file defines various **inbound listeners** for Clash.Meta. Inbound listeners are network entry points that accept traffic from clients or other network nodes. Each listener can have a different protocol, port, and routing behavior.

---

## 🔹 Listeners Overview

### 1. **SOCKS5 Listener**

```yaml
- name: socks5-in-1
  type: socks
  port: 10808
```

* Accepts SOCKS5 connections.
* Optional fields:

  * `listen`: interface to bind (default `0.0.0.0`).
  * `rule`: specify sub-rule set (default uses main `rules`).
  * `proxy`: forward inbound traffic to a specific proxy.
  * `udp`: enable/disable UDP (default `true`).

---

### 2. **HTTP Listener**

```yaml
- name: http-in-1
  type: http
  port: 10809
  listen: 0.0.0.0
```

* Accepts HTTP proxy connections.
* Supports optional `rule` and `proxy`.

---

### 3. **Mixed HTTP/SOCKS Listener**

```yaml
- name: mixed-in-1
  type: mixed
  port: 10810
  listen: 0.0.0.0
```

* Accepts both HTTP(S) and SOCKS proxy connections.
* Useful when both protocols are required on a single port.

---

### 4. **Redir Listener**

```yaml
- name: reidr-in-1
  type: redir
  port: 10811
```

* Transparent proxy for redirecting TCP traffic.
* Can be used with firewall rules to intercept traffic.

---

### 5. **TProxy Listener**

```yaml
- name: tproxy-in-1
  type: tproxy
  port: 10812
```

* Transparent proxy supporting TCP/UDP.
* Often used for advanced routing scenarios on Linux.

---

### 6. **Shadowsocks Listener**

```yaml
- name: shadowsocks-in-1
  type: shadowsocks
  port: 10813
  password: vlmpIPSyHH6f4S8WVPdRIHIlzmB+GIRfoH3aNJ/t9Gg=
  cipher: 2022-blake3-aes-256-gcm
```

* Accepts Shadowsocks encrypted traffic.
* `password` and `cipher` are required for secure connections.

---

### 7. **Vmess Listener**

```yaml
- name: vmess-in-1
  type: vmess
  port: 10814
  users:
    - username: 1
      uuid: 9d0cb9d0-964f-4ef6-897d-6c6b3ccf9e68
      alterId: 1
```

* Supports Vmess protocol.
* Requires `users` with UUIDs and optional `alterId`.

---

### 8. **Tuic Listener**

```yaml
- name: tuic-in-1
  type: tuic
  port: 10815
```

* High-performance protocol with advanced features:

  * `token`, `certificate`, `private-key`, `congestion-controller`, etc.
* Supports TCP, UDP, QUIC, and optional ALPN configurations.

---

### 9. **Tunnel Listener**

```yaml
- name: tunnel-in-1
  type: tunnel
  port: 10816
  network: [tcp, udp]
  target: target.com
```

* Accepts traffic over TCP/UDP tunnels to a target server.
* Flexible for VPN-like forwarding scenarios.

---

### 10. **TUN Listener**

```yaml
- name: tun-in-1
  type: tun
  stack: system
  dns-hijack:
    - 0.0.0.0:53
  inet4-address:
    - 198.19.0.1/30
  inet6-address:
    - "fdfe:dcba:9877::1/126"
```

* Virtual network interface listener.
* Supports full IPv4/IPv6 routing.
* Advanced options:

  * `auto-detect-interface`, `auto-route`, `mtu`, `strict_route`.
  * Linux UID-based routing (`include_uid`, `exclude_uid`).
  * Android user/app-specific routing (`include_android_user`, `include_package`).

---

## 🔹 Notes on Listener Options

* **`rule`**: Each listener can optionally use a `sub-rule` set; if not found, it uses the global `rules`.
* **`proxy`**: Directly forward inbound traffic to a proxy; must reference a valid proxy name.
* **`udp`**: Enable or disable UDP relay for supported protocols.
* **`listen`**: Network interface binding; default is `0.0.0.0`.
* **Security options**: Some protocols (Shadowsocks, Vmess, Tuic) require keys, passwords, or tokens for encryption.

---

## 🔹 Usage Example

```yaml
# SOCKS5 listener forwarding TCP traffic
- name: socks5-in-1
  type: socks
  port: 10808
  udp: true
  rule: sub-rule-name1
```

```yaml
# TUN interface for full device routing
- name: tun-in-1
  type: tun
  stack: system
  auto-route: true
  inet4-address:
    - 198.19.0.1/30
  inet6-address:
    - "fdfe:dcba:9877::1/126"
```

---

## 🔹 Summary

* This configuration supports **all major inbound protocols**: SOCKS, HTTP, Mixed, Redir, TProxy, Shadowsocks, Vmess, Tuic, Tunnel, and TUN.
* Each listener is **fully configurable**, allowing protocol-specific options, routing rules, and network-level adjustments.
* Ideal for **multi-protocol proxy servers** and **advanced routing setups** on Linux, Windows, and Android.

---
```
                       ┌─────────────────────────────┐
                       │        Client Traffic       │
                       └──────────────┬──────────────┘
                                      │
           ┌──────────────────────────┼──────────────────────────┐
           │                          │                          │
   ┌───────▼───────┐          ┌───────▼───────┐          ┌───────▼───────┐
   │  SOCKS5-in-1  │          │  HTTP-in-1    │          │  Mixed-in-1   │
   │   Port 10808  │          │  Port 10809   │          │  Port 10810   │
   └───────┬───────┘          └───────┬───────┘          └───────┬───────┘
           │                          │                          │
           └─────────────┬────────────┘                          │
                         │                                       │
                   ┌─────▼─────┐                                 │
                   │   Rules   │                                 │
                   │ Main rules│                                 │
                   └─────┬─────┘                                 │
                         │                                       │
       ┌─────────────────┼─────────────────────┐                │
       │                 │                     │                │
┌──────▼──────┐   ┌──────▼──────┐       ┌──────▼──────┐         │
│ Sub-rule-1  │   │ Sub-rule-2  │       │ Sub-rule-3  │         │
│ (TCP traffic│   │ (UDP only)  │       │ Mixed rules │         │
└──────┬──────┘   └──────┬──────┘       └──────┬──────┘         │
       │                 │                     │                │
┌──────▼──────┐   ┌──────▼──────┐       ┌──────▼──────┐         │
│ DOMAIN/PROXY│   │ IP-CIDR/REJ │       │ Combined    │         │
│ google.com  │   │ 1.1.1.1/32  │       │ rules       │         │
│ baidu.com   │   │ 8.8.8.8/32  │       │ etc.        │         │
└─────────────┘   └─────────────┘       └─────────────┘         │
                                                                │
                                                                │
                                                    ┌───────────▼───────────┐
                                                    │     Final Action      │
                                                    │  DIRECT / REJECT /    │
                                                    │      PROXY            │
                                                    └───────────────────────┘

Legend:
- Main rules: global rules applied if sub-rule not specified
- Sub-rules: traffic branch sets for fine-grained control
- Final Action: outcome after evaluating all rules
```

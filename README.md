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
Clash.Meta supports a comprehensive set of rule types that enable users to route network traffic based on various conditions such as domain names, IP addresses, ports, processes, and geographic location. Here's a detailed overview of the supported rule types:

---

### 🔧 Supported Rule Types in Clash.Meta

1. **DOMAIN**
   Matches exact domain names.
   Example:

   ```yaml
   DOMAIN,example.com,DIRECT
   ```

2. **DOMAIN-SUFFIX**
   Matches domains with a specific suffix.
   Example:

   ```yaml
   DOMAIN-SUFFIX,example.com,DIRECT
   ```

3. **DOMAIN-KEYWORD**
   Matches domains containing a specific keyword.
   Example:

   ```yaml
   DOMAIN-KEYWORD,example,DIRECT
   ```

4. **GEOIP**
   Matches based on the geographic location of the IP address.
   Example:

   ```yaml
   GEOIP,CN,DIRECT
   ```

5. **GEOIP6**
   Matches based on the geographic location of the IPv6 address.
   Example:

   ```yaml
   GEOIP6,CN,DIRECT
   ```

6. **IP-CIDR**
   Matches based on the destination IPv4 address.
   Example:

   ```yaml
   IP-CIDR,192.168.1.0/24,DIRECT
   ```

7. **IP-CIDR6**
   Matches based on the destination IPv6 address.
   Example:

   ```yaml
   IP-CIDR6,2001:db8::/32,DIRECT
   ```

8. **SRC-IP-CIDR**
   Matches based on the source IPv4 address.
   Example:

   ```yaml
   SRC-IP-CIDR,192.168.1.100/32,DIRECT
   ```

9. **SRC-IP-CIDR6**
   Matches based on the source IPv6 address.
   Example:

   ```yaml
   SRC-IP-CIDR6,2001:db8::1/128,DIRECT
   ```

10. **DST-PORT**
    Matches based on the destination port.
    Example:

    ```yaml
    DST-PORT,80,DIRECT
    ```

11. **SRC-PORT**
    Matches based on the source port.
    Example:

    ```yaml
    SRC-PORT,8080,DIRECT
    ```

12. **PROCESS-NAME**
    Matches based on the name of the process sending the packet.
    Example:

    ```yaml
    PROCESS-NAME,chrome,DIRECT
    ```

13. **PROCESS-PATH**
    Matches based on the path of the process sending the packet.
    Example:

    ```yaml
    PROCESS-PATH,/usr/bin/chrome,DIRECT
    ```

14. **RULE-SET**
    Matches based on a set of external rules provided by a rule provider.
    Example:

    ```yaml
    RULE-SET,google,DIRECT
    ```

15. **SCRIPT**
    Matches based on the evaluation of a script.
    Example:

    ```yaml
    SCRIPT,check_https,DIRECT
    ```

16. **MATCH**
    A catch-all rule that matches any traffic not matched by previous rules.
    Example:

    ```yaml
    MATCH,REJECT
    ```

---

These rule types allow for granular control over network traffic, enabling users to implement complex routing logic based on various conditions. For more detailed information and examples, you can refer to the official Clash.Meta documentation. ([GitHub][1])

[1]: https://github.com/djoeni/Clash.Meta?utm_source=chatgpt.com "djoeni/Clash.Meta: A rule-based tunnel in Go. - GitHub"

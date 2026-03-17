# IPv6

## 1. The Transition to IPv6

- **The Address Space:** IPv6 was developed in the 1990s to solve IPv4 exhaustion. It uses **128-bit addresses**, meaning there are 340 undecillion addresses available.

- **End-to-End Reachability:** Because of the vast address space, IPv6 eliminates the need for Network Address Translation (NAT). NAT is historically a workaround to conserve IPv4 addresses, not a security feature; in IPv6, firewalls handle security, allowing for true end-to-end connectivity.

- **Formatting Rules:** Addresses consist of 8 groups of 16 bits, written in hexadecimal and separated by colons (e.g., `2001:0db8::1`).
    - **Rule 1:** Leading zeros in any 16-bit segment can be omitted.
    - **Rule 2:** A continuous sequence of all-zero segments can be replaced with a double colon (`::`), but this can only be done once per address to avoid ambiguity.

## 2. IPv6 Address Types

![](images/Pasted%20image%2020260316184211.png)

IPv6 eliminates broadcast addresses entirely, replacing them with Multicast.

- **Global Unicast Addresses (GUA):** Public, globally routable addresses on the internet. They currently start with the `2000::/3` prefix (spanning `2000` to `3FFF`). All end users will have a GUA address

- **Link-Local Addresses:** Unique and valid _only_ on a specific physical link (local network segment). They cannot be routed off the link. They start with the **FE80::/10** prefix. Every IPv6 device must have at least one link-local address that have to be unique on the link, and they are critical for routing protocol messages and auto-configuration.

- **Unique Local Addresses (ULA):** The IPv6 equivalent of IPv4 private IP addresses (like `10.x.x.x`). They start with `FC00::/7` and can be routed internally within an organization, but not on the public internet.

- **Anycast:** Shares the same IP address across multiple hosts in different geographic locations. Often used by CDNs (like Cloudflare) to mitigate DDoS attacks by diluting the attack traffic toward multiple regional servers and perform load balancing.

## 3. IPv6 Subnetting & Interface IDs

- **The "3-1-4" Rule:** Instead of Variable Length Subnet Masks, IPv6 standardizes a `/64` prefix length for most physical networks. A typical address features 3 hexets for the Global Routing Prefix, 1 hexet for the Subnet ID, and 4 hexets for the **Interface ID** (the host portion).
  ![](../../Pasted%20image%2020260316184705.png)

- **Generating the Interface ID:**
    - **EUI-64:** Deterministic method. It splits the host's 48-bit MAC address, inserts `FFFE` in the middle to make it 64 bits, and flips the 7th bit. This is ideal for static servers but bad for privacy since your MAC address is embedded in your IP.
    - **Privacy Extensions (Random):** Randomly generates the Interface ID. Used as temporary, "disposable" addresses to ensure user anonymity online. In Linux, this is controlled via `sysctl` parameters like `addr_gen_mode` and `use_tempaddr`.

## 4. ICMPv6 & Dynamic Configuration

IPv6 relies heavily on **ICMPv6** to manage the network, completely replacing IPv4's ARP with the **Neighbor Discovery Protocol (NDP)**.

- **NDP (Replacing ARP):** To find the MAC address for a known IP, a host sends an ICMPv6 _Neighbor Solicitation_ and receives a _Neighbor Advertisement_. Instead of broadcasting to the whole network, this utilizes a highly efficient **Solicited-Node Multicast** address.

- **SLAAC (Stateless Address Auto-Configuration):** Allows a device to assign itself an IP address without needing a central DHCP server.
    - When a device joins a network, it uses its Link-Local address to send a **Router Solicitation**.
    - The router replies with a **Router Advertisement (RA)** containing the network prefix and the default gateway.
    - The device generates its own Interface ID (via EUI-64 or randomly) and appends it to the prefix.

- **Duplicate Address Detection (DAD):** To avoid IP collisions in SLAAC, the host sends a Neighbor Solicitation for its freshly generated IP. If nobody answers, the host claims the address.

## 5. DHCPv6 & Prefix Delegation

Router Advertisements contain flags (`M` for Managed, `O` for Other Configuration) that tell the host how to get its IP.

- **Stateless (SLAAC):** Host gets prefix and gateway from the router, generating its own IP.

- **Stateless + DHCPv6:** Host uses SLAAC for its IP and gateway, but queries a DHCPv6 server to get DNS domain info.

- **Stateful DHCPv6:** Host asks the DHCPv6 server for an IP address lease, exactly like traditional IPv4 DHCP.

- **DHCPv6 Prefix Delegation:** Instead of handing out a single public IP to a home router and forcing the use of NAT, ISPs use Prefix Delegation to dynamically give the home router an entire network prefix (e.g., a `/56`). The router then distributes `/64` subnets directly to the internal network devices.

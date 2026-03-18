# IPv6 Protocol

## 1. SLAAC and Temporary Addresses

- **Privacy Extensions:** When using Stateless Address Auto-Configuration (SLAAC), hosts can generate a random 64-bit Interface ID instead of using the deterministic EUI-64 process.
- **Temporary Addresses:** To enhance privacy, devices can generate multiple temporary addresses with short lifetimes (usually hours or days) to be used as the source address when originating new connections.

## 2. IPv6 Multicast Addresses

IPv6 entirely eliminates the "broadcast" address, replacing it with Multicast (one-to-many communication).

- **Formatting:** All IPv6 multicast addresses use the prefix **FF00::/8**.
- **Structure:** After the prefix, the address contains a 4-bit **Flag**, a 4-bit **Scope**, and a 112-bit **Group ID**.
    - **Flag:** A value of `0` indicates a permanent, well-known multicast address assigned by IANA, while `1` indicates a dynamically assigned address.
    - **Scope:** Defines the range of the packet. Common values include `1` (Interface-Local), `2` (Link-Local), `5` (Site-Local), `8` (Organization-Local), and `E` (Global).
- **Key Predefined Groups (Link-Local Scope** **FF02****):**
    - **FF02::1** **(All IPv6 Devices):** Every IPv6 device on the link listens to this address. It is used, for instance, by routers sending Router Advertisements.
    - **FF02::2** **(All IPv6 Routers):** All routers on the link listen to this group. Hosts use this to send Router Solicitations.
- **DHCPv6 Relay Agents:** If a DHCPv6 server is on a different network than the clients, a router can act as a Relay Agent. It listens for client solicitations and forwards them to the remote DHCPv6 server.

## 3. IPv6 Header vs. IPv4 Header

The IPv6 header was heavily simplified to improve processing speed and take advantage of 64-bit CPUs. The IPv6 header has a **fixed size of 40 bytes**.

- **Removed Fields:**
    - **Internet Header Length (IHL):** Removed because the IPv6 header size is fixed at 40 bytes.
    - **Header Checksum:** Removed entirely. IPv6 relies on upper-layer protocols (like TCP and UDP) to perform checksums, making the UDP checksum mandatory in IPv6.
    - **Options & Padding:** Removed from the main header and relocated to "Extension Headers".
- **Renamed & New Fields:**
    - **Traffic Class:** Replaces the IPv4 "Type of Service" field, used to prioritize traffic (QoS).
    - **Flow Label:** A new 20-bit field used to identify packets belonging to a common stream or TCP session so they can be treated identically by routers.
    - **Payload Length:** Replaces "Total Length." It represents the size of the payload (including extension headers) _excluding_ the main 40-byte IPv6 header.
    - **Hop Limit:** Replaces "Time to Live" (TTL). Every router decrements this value by 1; if it reaches 0, the packet is dropped.

## 4. Fragmentation in IPv6

A major difference from IPv4 is that **intermediate IPv6 routers do not fragment packets**. Router fragmentation was removed because it slows down packet processing and risks Denial of Service (DoS) attacks.

- **How it works:** If a router receives an IPv6 packet larger than the outgoing link's Maximum Transmission Unit (MTU), **the router drops the packet** and sends an **"ICMPv6 Packet Too Big"** message back to the source, indicating the MTU size it should use.
- **Path MTU Discovery:** The source host must handle any necessary fragmentation using a specialized Extension Header before sending the packets. IPv6 requires every link to have a minimum MTU of 1280 bytes.

## 5. IPv6 Extension Headers (EHs)

Instead of cramming optional data into the main IP header, IPv6 introduces a highly flexible modular system called Extension Headers.

- **The "Next Header" Chain:** The `Next Header` field in the IPv6 main header indicates what follows. It can point to an upper-layer protocol (like TCP `6`, UDP `17`, or ICMPv6 `58`) or to an Extension Header. Multiple Extension Headers can be daisy-chained together.
- **Edge Processing:** To maximize efficiency, Extension Headers (except for "Hop-by-Hop") are **only processed by the final destination endpoint**, not by the intermediate routers.
- **Types & Strict Ordering:** There is a fixed set of Extension Headers, and they must follow a specific order. Common headers include:
    - **0:** Hop-by-Hop Options (The _only_ header examined by every router on the path).
    - **43:** Routing (Specifies the path/routers to be crossed).
    - **44:** Fragment (Used by the source to handle oversized payloads).
    - **50 & 51:** ESP (Encapsulating Security Payload) and AH (Authentication Header) for IPsec, providing end-to-end encryption and authentication.
# Spoof Tunnel
[Persian-فارسی](README-fa.md)

Spoof Tunnel is a Layer 3/Layer 4 tunneling proxy designed to bypass Deep Packet Inspection (DPI) and strict stateful firewalls through **mutual bidirectional IP spoofing**. 

Unlike traditional tunneling protocols that establish a stateful connection between a fixed client IP and a fixed server IP, Spoof Tunnel completely decouples the logical session from the physical network addresses by forging the `Source IP` field in the IP header at both endpoints.

> [!IMPORTANT]
> Both your servers must be able to send spoofed packets.
> 
> To test this, you can use the following command temporarily on any of your servers:
> 
> iptables -t nat -A POSTROUTING -d target-ip -j SNAT --to-source spoof-ip
> 
> then:
> 
> ping target-ip
> 
> and use a tool like tcpdump on the opposite server:
> 
> tcpdump icmp
> 
> if you see the spoofed packets, means your source side can send spoofed packet.

> [!NOTE]
> **Personal note (my fork):** I tested this successfully on Hetzner VPS instances (CX21). Most major cloud providers (AWS, GCP) block raw socket spoofing at the hypervisor level, so make sure to verify your provider supports it before spending time on setup.

### How the Project Came to Be: The Origin of Spoof Tunnel
The concept of a bidirectional spoofing tunnel emerged in response to the severe internet blackout in Iran following the bloody uprising on January 8 and 9, 2026 (18-19 Dey 1404). During this complete disconnection from the global internet, our primary objective was to reverse-engineer the exact scope and layer of the imposed restrictions.

Upon investigating the BGP routes for Iranian IP prefixes, we observed a surprising detail: unlike the internet shutdown in Afghanistan where BGP routes simply disappeared, Iran's IP ranges were still actively being announced globally. This strongly indicated that the international physical infrastructure was still intact.

Subsequently, it became apparent that certain government-affiliated Iranian entities were able to whitelist their specific IP addresses, successfully restoring their international connectivity. This observation led to the hypothesis that the restriction was being enforced at Layer 3, specifically filtering based on srcIP and dstIP.

This hypothesis was definitively confirmed when we discovered that a select few foreign IP addresses (such as specific ranges from Hetzner) could still establish inbound connections to Iran. The evidence clearly demonstrated that the "blackout" was not a physical severance, but rather a stringent, whitelist-based Layer 3 firewall policy.

In this highly restricted environment, the idea of a spoofing tunnel was conceived. By manipulating the IP headers, we could simulate whitelisted traffic. However, as is inherent to IP spoofing, if a spoofed packet is sent to a server, the server will inherently route its reply back to the spoofed IP address—not the actual origin host.

Therefore, a standard unidirectional spoof was insufficient. We required a robust bidirectional mutual spoofing mechanism where both the client and the server forge their IP headers and are predetermined instances well-aware of each other's actual physical IPs, enabling them to establish and maintain a logical connec
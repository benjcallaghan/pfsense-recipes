# Nintendo Switch Online

The default settings in a pfSense installation are not compatible with the way that Nintendo operates their online multiplayer network. This recipe reconfigures traffic originating from a Nintendo Switch console to allow direct connections to/from other Nintendo Switch consoles.

## DHCP Reservation

The NAT rule created below requires the Nintendo Switch console to have a predictable IP address. A simple way to ensure this is to provide a DHCP reservation.

1. Create a static DHCP mapping using the MAC address of the Nintendo Switch console and an IP address outside the normal DHCP range.
![Summary of static DHCP mapping](./static-dhcp-mapping.png)

## NAT Rules

By default, pfSense includes an Outbound NAT rule that randomizes the source port for every NAT'd connection. Nintendo requires that the source port used by the Gateway is the same source port used by the Nintendo Switch console.

1. Set the Outbound NAT Mode to "Hybrid" or "Manual". Hybrid mode preserves the auto-generated rules, while Manual does not. The default of "Automatic" implicitly disables any custom rules, including the one set below.
2. Add a new Outbound NAT rule with the following properties:
    * Interface: WAN
    * Address Family: IPv4+IPv6 TODO: Is IPv4 sufficient?
    * Protocol: TCP/UDP TODO: Is UDP sufficient?
    * Source: {Static IP of Console}/32
    * Destination: Any
    * Address: WAN Address
    * Static Port: checked
    * Description: Nintendo Switch
![Summary of Outbound NAT rule](./outbound-nat-rules.png)

## Alternatives

Nintendo Switch Online multiplayer can also be configured via UPnP, which is the technique most consumer-grade routers would use.

## Hole Punching

It is assumed that Nintendo Switch multiplayer networking involves [UDP hole punching](https://en.wikipedia.org/wiki/UDP_hole_punching). Nearly all consumer home networks employ NAT to run multiple Internet-connected devices on a single ISP-provided address. These networks also employ firewalls that prevent incoming connections. Hole punching allows a direct peer-to-peer connection between two Nintendo Switch consoles, without needing to allow incoming connections at the firewall. The following diagram shows how the randomized source ports (the default setting) prevents hole punching while static source ports enables the intended behavior.

```mermaid
sequenceDiagram
    box LAN 1
    participant P1 as Player 1
    participant G1 as Gateway 1
    end
    participant N as Nintendo
    box LAN 2
    participant G2 as Gateway 2
    participant P2 as Player 2
    end
    
    P1->>N:I will listen on Port 1111
    P2->>N:I will listen on Port 2222
    N->>P1:Your friend is at 216.x:2222
    N->>P2:Your friend is at 104.x:1111

    alt With Randomized Ports
        P1->>G1:HELLO 216.x:2222 from 104.x:1111
        G1-xG2:HELLO 216.x:2222 from 104.x:1234
        note left of G2:inbound connection denied
        P2->>G2:HELLO 104.x:1111 from 216.x:2222
        G2-xG1:HELLO 104.x:1111 from 216.x:4321
        note right of G1:inbound connection denied
        note over G1,G2:connection failed
    else With Static Ports
        P1->>G1:HELLO 216.x:2222 from 104.x:1111
        G1-xG2:HELLO 216.x:2222 from 104.x:1111
        note left of G2:inbound connection denied
        P2->>G2:HELLO 104.x:1111 from 216.x:2222
        G2->>G1:HELLO 104.x:1111 from 216.x:2222
        note right of G1:inbound response accepted
        G1->>P1:HELLO 104.x:1111 from 216.x:2222
        P1->>G1:SUCCESS
        G1->>G2:SUCCESS
        note left of G2:inbound response accepted
        G2->>P2:SUCCESS
        note over G1,G2:connection established
        P1<<->>P2:Multiplayer Game Data
    end
```

While hole punching appears to bypass firewall restrictions, it does NOT allow unsolicited incoming connections. The technique only works when both parties initiate an outbound connection to each other, signalling intent to form a direct connection. If only one party initiated the connection, standard firewall rules would prevent access.
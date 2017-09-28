# Chapter 1: Computer Networks and the Internet

## What is the Internet?

### Nuts-and-Bolts Descriptions

![nuts-and-bolts-desc](https://user-images.githubusercontent.com/22194121/30897828-9d26870a-a382-11e7-97e5-6deab5fc8344.png)

:bookmark: `host` or `end system`: devices that are closely related to the end-user of the Internet such as smart phones, laptops, servers, ...

:bookmark: `communication link`: A communication link is a link between 2 hosts. There are 2 different types of links. There many different types of communication links including coaxial cable, optical fibre, and radio spectrum.

:bookmark: `packet switch`: A packet switch is an intermediate delivery guy. Information on the Internet is sent `packet` by packet. Each packet is a part of a bigger and more complete information pack. Sometimes 2 hosts are too far away and it's too complex to connect directly between 2 hosts. Packet switches are born to serve delivery between 2 spontaneous hosts. There are 2 prominent types of packet switches: `router` and `link-layer switch`. The sequence of switches from a source to a destination is called a `route` or a `path`.

### Protocol

A protocol is a series of step-by-step guideline for which actions are executed based on which responses received. A protocol must be strictly executed sequentially. A protocol must involve 2 or more parties throughout its activity.

## Network Edge

:bookmark: `Network Edge`: A network edge is a group or a subnet of hosts. (Switches are not hosts). Think of them as ribs inside a skeleton. Ribs are not connected directly to each other, they are connected to the backbone.

### Access Networks

![access-networks](https://user-images.githubusercontent.com/22194121/30898695-cb4bd41a-a386-11e7-9cac-b76d461f9eee.png)

:bookmark: `access network`: the network of the hosts connected (to each other) and before they go into the `network core` (switches which transfer the data).

- Home access:
    - `DSL`: Digital Subscriber Line. This line uses telephone line to connect the Internet. Speed is not competitive compared to other mediums.
    - `Cable`: This line uses TV line to connect to the Internet. It is better than the DSL but not much of a margin.
    - `FTTH`: Fibre to the Home. This is optical fibre which is becoming prominent gradually throughout the population of Internet usage due to it stability, speed and corresponding bandwidth between upload and download.
- Enterprise access:
     - `Ethernet`: a typical technology to apply `LAN` (Local Area Network). It connects all hosts in the same enterprise (university, company, club, ...) and play the role as the front-man to talk with outsiders.
     - `Wifi`: use electro-magnetic radiations to connect all the devices. This is more or less the same as above without cables.
     - All above methods can also be applied to home access.
- Wide Area Network: Ran by phone service company who distributes and maintains cell phone network. They apply 3G, 4G into their stations and connect any devices in a radius.

### Physical Media

- `guided media`: The media knows exactly where the sender and the receiver are. Connections between the two are also exclusive.
    - Fibres (optical and coxial copper wire) 

- `unguided media`: The media only diseminates the information and hope that the receiver can sense it.
    - Wifi
    - Wide Area Network.

## Network Core

:bookmark: `Network Core`: The network that is hidden from end-users which are ran by ISPs to transfer information and packets (switches). Think of it as the backbone of the Internet. It connects all the ribs (network edges) together.

![network-core](https://user-images.githubusercontent.com/22194121/30899746-6804f012-a38b-11e7-8bc2-d2e07db232cc.png)

### Packet Switching

:bookmark: `Store-and-Forward Transmission`: A switch must store the entire packet before sending it again. That means the last bit of a packet must be already stored inside before its first bit is sent out.

![store-and-forward](https://user-images.githubusercontent.com/22194121/30904662-cf95d686-a39c-11e7-9dae-a5a51dce9c54.png)

:bookmark: Queueing Delays and Packet Loss
- Queueing delay is the time of a packet waiting inside a switch before being sent away. In other words, it is the difference between the timestamp of its last bit being sent to current switch and the timestamp of its first bit being sent out of the switch.

- Packet loss occurs when the queue is full and no more packet can be stored inside. In this case, packet will be dumped into the void.

:bookmark: Forwarding Table and Routing Protocol
- `Forwarding Table`: a table stored inside a router/switch that maps the address of the **final** destination to an outbound link. This does not mean the final destination is on the other end of that link. The world contains an enormous amount of endpoints so that's not possible. What it means is that the final destination can be reached via that outbound link. In other word, the outbound link is part of the full route from source to destination. This route is discovered by the routing protocol.
- `Routing Protocol`: A routing protocol is a protocol involves switches and routers to find the route from a source to a destination.

:bookmark: `Circuit Switching`: Communications between endpoints are reserved. There is no packet used in this kind of communication. There is only a stream between endpoints.

|Packet Switching|CircuitSwitching|
|---|---|
|`Not suitable` for real-time services because of many variables along the route|`Suitable` for real-time services thanks to dedicated connection line|
|`More capacity` of users because all users are active|`Less capacity` of users because lines are reserved|
|`Larger bandwidth` because active users can use up all available bandwidth of the line| `Smaller bandwidth` because active users are limited to their reserved lines only|

### A Network of Networks

- All devices inside a home are connected through a router or a switch. They form a small network inside the house.
- All devices inside an enterprise are also connected via Wifi or LAN. They also form a small network inside that enterprise.
- All devices using the access ISP can be connected via that ISP's stations. (access ISP means that they are the front-man of end systems to access the Internet)
- All access ISP can be connected via a global transit ISP. (global transit ISP connects a network from a cont)inent to a network from another). Global transit ISP are sometimes called tier-1 ISPs.
- Sometimes, there is a middle man called regional ISP that provides wide coverage of Internet service to that area so access ISPs will be connected to these ISPs first before reaching the local ISP.

![network-of-networks](https://user-images.githubusercontent.com/22194121/30934825-fc555ec4-a3f8-11e7-9ac8-56f24057c165.png)

## Delay, Loss and Throughput of Packet-Switched Networks

![delays](https://user-images.githubusercontent.com/22194121/30935076-bdb50808-a3f9-11e7-9fa9-e2942837d29f.png)

:bookmark: `Processing Delay`: time of a packet being processed (nodal process). Processing involves looks up table and finds outbound link to send the packet to.
:bookmark: `Queuing Delay`: time of a packet waiting in the queue. At this point, the packet knows its expected outbound link but it has to wait for other packets to be sent.
:bookmark: `Transmission Delay`: time of the whole packet exits the router/switch. It is closely related to the transmission rate - number of bits spit out of that device.
:bookmark: `Propagation Delay`: time the packet goes on the communication line. It is the difference between the timestamp of the last bit out of the source and the timestamp of the last bit into the destination.

## Protocol Layers and Their Service Models

### Protocol Layering

![protocol-layer](https://user-images.githubusercontent.com/22194121/30950843-b4e5c50a-a449-11e7-959c-f4c698ca2963.png)

Network operations are classified into 5 basic layers. These layers play different roles and have their own protocols.

### Application Layer

Examples: `HTTP`, `SMTP`, `FTP`.

This layer only appears at hosts and end systems.

It is the interface between the application and the network under the hood. It only knows send and receive.

A packet of information is called a `message`.

### Transport Layer

Examples: `TCP`, `UDP`.

This layer appears at hosts and end systems only.

It delivers messages between application endpoints.

A packet of information is called a `segment`.

### Network Layer

Example: `IP`.

This layer appears at both network edges and cores.

It determines the route to the destination endpoint. It only knows the route.

A packet of information is called a `datagram`.

### Link Layer

Examples: `Ethernet`, `Wifi`.

This layer appears both at network edges and cores.

Its purpose is to transfer data over a direct link to the next router/switch.

A packet of infromation is called a `frame`.

### Physical Layer

Examples: `Copper Wire`, `Optical Fibre`, `3G`.

This layer appears both at network edges and cores.

This layer is the hardware itself to send a data. Its unit is individual bits.

### Encapsulation

![encapsulation](https://user-images.githubusercontent.com/22194121/30951223-ef180132-a44b-11e7-92ae-83dda5dbc4bf.png)

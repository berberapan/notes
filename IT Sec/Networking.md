## OSI Model
OSI stands for Open Systems Interconnection, it's a model developed by ISO to describe how communications should occur in a computer network. It's composed of seven layers

![[osimodel.bmp]]
#### Layer 1 - Physical
The first layer deals with the physical connection between devices. Examples of physical mediums are WiFi radio bands or fiber optic cables.

#### Layer 2 - Data Link
The second layer represents the protocol that enables data transfer between nodes on the same network segment. So it describes an agreement between the different systems on the same network segment on how to communicate.

Examples of layer 2 are Ethernet and WiFi. MAC (Media Access Control) address can be tied to several protocols in layer 2 but not all. But if you see a MAC address you can associate it with layer 2.

MAC address are 6 bytes and are usually expressed in hexadecimal with a colon separating each byte. I.e. `FC:5C:EE:AB:4E:1C`

The first 3 bytes identify the vendor.

#### Layer 3 - Network
The network layer handles logical addressing and routing, like finding paths to transfer network packets between diverse networks. So you can say while the layer 2 is concerned with sending data between nodes on the same network segment the layer 3 is concerned with sending data between different networks.

So for example if you're dealing with a multinational company the layer 2 would handle the traffic between computers in an office while layer 3 would handle the data between the offices.

Examples of layer 3 are IP (Internet Protocol) and certain VPN technologies like IPsec.

#### Layer 4 - Transport
The transport layer enables end-to-end communication between running applications on different hosts.

Examples of layer 4 are TCP (Transmission Control Protocol) and UDP (User Datagram Protocol).

#### Layer 5 - Session
The fifth layer is responsible for establishing, maintaining and synchronizing communication between applications on different hosts. Establishing a session means initiating communication between applications and negotiating the parameters for the session. Data synchronization ensures that data is transmitted in the correct order and provides mechanisms for recovery in case of transmission failures.

Layer 5 isn't as easily defined for examples but some examples are SQL sessions and online gaming sessions.

#### Layer 6 - Presentation
Presentation layer ensures that the data is delivered in a form the seventh layer can understand. Layer 6 handles data encoding, compression and encryption. So for example character encoding like Unicode.

Examples are Unicode, MIME, PNG.

#### Layer 7 - Application
Layer 7 provides network services directly to end-user applications.  For example the HTTP protocol is used by your web browser to submit forms on websites etc.


|Layer Number|Layer Name|Main Function|Example Protocols and Standards|
|---|---|---|---|
|Layer 7|Application layer|Providing services and interfaces to applications|HTTP, FTP, DNS, POP3, SMTP, IMAP|
|Layer 6|Presentation layer|Data encoding, encryption, and compression|Unicode, MIME, JPEG, PNG, MPEG|
|Layer 5|Session layer|Establishing, maintaining, and synchronising sessions|NFS, RPC|
|Layer 4|Transport layer|End-to-end communication and data segmentation|UDP, TCP|
|Layer 3|Network layer|Logical addressing and routing between networks|IP, ICMP, IPSec|
|Layer 2|Data link layer|Reliable data transfer between adjacent nodes|Ethernet (802.3), WiFi (802.11)|
|Layer 1|Physical layer|Physical data transmission media|Electrical, optical, and wireless signals|
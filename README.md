# bristlecone
Bristlecone is a simple game-centric protocol definition that significantly reduces effective latency.

The core definition of Bristlecone is simple:

- Bristlecone uses UDP over IPv4.  A successor protocol will use either UDP over IPv6 or a raw IPv6 packet.  
- Each Bristlecone datagram schema uses a unique port.  
- Bristlecone datagram schemas are statically sized. As a result, Bristlecone is recommended only for small schemas.
- The recommended Bristlecone datagram sizes are 16, 24, and 32 bytes.
- Each Bristlecone transmission contains only one datagram schema.  
- Each transmission contains the current datagram and the previous two.
- For the first & second transmission, a zero fill may be used or a nonce supplied in the available space.
- Each Bristlecone transmission is thus statically sized.
- These sizes can be inferred at compile time.
- A bristlecone transmission of 3 datagrams is called a clone.
- Bristlecone transmits between 1 and 3 instances of each clone, depending on ECN.
- These clone sets are transmitted as fast as possible, with the goal that all copies in a cloneset be in-flight simultaneously.
- These clones use differing DSCP signifiers to help ensure differing transmission behaviors.
  
Taken together, this allows Bristlecone to **instantaneously absorb packet loss without a variable retransmission scheme or a sliding window.** 
Bristlecone transparently tolerates packet disordering of up to three, allowing the fastest packet in any run of 3 clone sets to be used.  

Bristlecone adds three additional characteristics.
- All bristlecone traffic is encrypted at the clone level.
- This encryption is negotiated by a single short-lived TCP/IP connection before bristlecone sessions start.
- Use of RSA is contraindicated due to expected near-term quantum compromise.

Bristlecone treats datagrams encoded with different endianness as differing datagrams. As a result:
- These must be assigned differing ports.
- These must be encrypted without being transposed to network byte order.

Finally, we recommend that:
- The encryption be symmetric key negotiated over a TLS2+ secured connection. mTLS is strongly preferred.
- This negotiation process should include both authentication and authorization before symmetric key issuance.
- This negotiation process should transmit the port and schema mapping for the Bristlecone datagrams expected to be used.
- This negotiation process should ensure schema compatibility.
- Bristlecone is not intended to be used in a direct peer-to-peer capacity, as revealing client IP addresses to other clients is strongly advised against.

Using symmetric encryption as a token of successful authentication and authorization means that packets which do not decrypt successfully can be rejected with no knowledge dependencies by edge nodes. Further, by allocating the same symmetric key to all clients that, for example, share a lobby in a multiplayer game, attestation of session membership is provided by the IP-Key pair. Only a compromise of both a valid IP and a live key can successfully allow a DoS to pass the edge, while clients in the same lobby can transparently decrypt packets from one another, allowing sophisticated edge reflection schemes.

## Q&A   
**Q: Why handle endian-ness in this unusual way?**  
A: Very few games are deployed to big-endian systems. These systems do not need the additional complexity of implied support, and many cannot support big-endian transparently due to exotic bit-manipulation or handwritten assembly. As a result, this saves considerable ambiguity that could lead to serious coding errors and allows the parsing of datagrams in a branchless fashion while avoiding transposition costs. It still fully supports multi-endian systems, just in an atypical way. Should this come up in practice, Hedra Group is open to amending the standard.  
   
**Q: Why use one port per schema?**    
A1: This allows traffic to be separated using routing indirection without changes to client code. If your server architecture changes, and some component of your traffic needs to be handled by a new microservice, or if you switch to a monolithic design, separated datagram streams make this transparent to the client.    
A2: This allows significantly better separation of concerns, as a given stream can be handled completely by taking only the relevant schema as a dependency. It is awkward and undesirable to conflate parsing of different datagrams, both from a threading semantics standpoint and a practical standpoint.   
   
**Q: Why use end to end encryption, and why this way?**   
A1: Bristlecone **does not use end to end encryption.** It explicitly requires no guarantee that the symmetric key not be provided to edge nodes for traffic inspection.   
A2: The combination of an IP source, valid port mappings, and correctly encrypted payload can serve as a connectionless attestation of session membership.   
A3: In situations where IP compromise is known to be prevented, it can serve as attestation of identity.    
A4: In situations where IP compromise cannot be prevented, a rolling nonce can negotiated during the TCP session. This is not included in the core protocol definition.   

# bristlecone
Bristlecone is a simple game-centric protocol definition that significantly reduces effective latency.

The core definition of Bristlecone is simple:

Bristlecone uses UDP over IPv4.  
A successor protocol will use either UDP over IPv6 or a raw IPv6 packet.  
Bristlecone uses a "rolling guess" problem for spoofing protection.  
Each Bristlecone datagram schema uses a unique port.  
Bristlecone datagram schemas are statically sized.   
  As a result, Bristlecone is recommended only for small schemas.  
Each Bristlecone transmission contains only one datagram schema.  
Each transmission contains the current datagram and the previous two.  
  For the first transmission, the current datagram may be repeated or a zero fill may be used.  
Each Bristlecone transmission is thus statically sized.  
These sizes can be inferred at compile time.  
Bristlecone transmissions are repeated between 0 and 2 times, depending on ECN.  
These repeated transmissions happen as fast as possible, and are referred to as clones.  

These clones use differing DSCP signifiers to help ensure differing transmission behaviors. This allows Bristlecone to instantaneously absorb packet loss.

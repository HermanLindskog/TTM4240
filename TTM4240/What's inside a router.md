**Four compononents**
1. [[#Input port]]
2. [[#Switching fabric]]
3.  [[#Output ports]]
4. [[#Routing processor]]

### Input port
This port is a physical port of the router, and it also performs a lookup function forwarding table to determine the router output port to which an arriving packet will be forwarded by the switching fabric.

Additional it will also control packets (carrying routing protocols) - forwarded from an input port to the routing processor.

### Switching fabric
Connects router input and output port

### Output ports
Stores packets from the switching fabric and transmits these packets to outgoing links

(When a link is bidirectional (that is, carries traffic in both directions), an output port will typically be paired with the input port for that link on the same line card.)

### Routing processor
Control-plane functions. Tradiotional routers will perform the routing protocols, maintains routing tables and attached link state information, and computes the forwarding table for the router. While SDN routers the routing processor is responsible for communicating with the remote controller in order to receive forwarding table entries computed by the remote controller, and install these entries in the router’s input ports. The routing processor also performs the network management functions.

**Data plane**
Nanosecond time scale
**Control plane**
More the millisecond or second timescale
- Usually implemented software and executed on the routing processor (CPU)

## Routing processing:

##### Destinations-based forwarding
Looks up the final destination

##### Generalized forwarding
Different number of factors used to determine where it is going to be forwarded to
-   The bottleneck might be when these packets is arriving with not enough factors to determine where they are going

## Input port processing and destination-based forwarding
The forwarding table is copied from the routing processor to the line cards over a seperate bus (PCI bus) indicated by the dashed line from the routing processor to the input line cards shown in the first figure
![[Pasted image 20220828192143.png]]

- If e.g. the packet’s destination address is 11001000 00010111 00010110 10100001 ; **because the 21-bit prefix of this address matches the first entry in the table, the router forwards the packet to link interface 0**
-   If it doesn't match with anyone it would by default go to link interface 3 by default
-   When a adress matches more than one prefix it wil us the **longest prefix matching rule**
-   Memory access time helps the lookup of hardwares
	-   DRAM and faster SRAM memories
		- TCAM are also used for lookup
		-   Can hold up to one million TCAM forwarding table entries
-   Switching fabric can block certain packets if the input port are currently using the fabric
-   Will be queded at the input port and scheduelded for later

## Switching
#### Switching by memory
- Input port recieveing the packet signaled the routing processor by an interrrupt. Send from the input port to the memory, and then the processor memory extracted the destination adress up and sent it to the output port.
	- In this scenario, if the memory bandwidth is such that a maximum of B packets per second can be written into, or read from, memory, then the overall forwarding throughput (the total rate at which packets are transferred from input ports to output ports) must be less than B/2. *(Note also that two packets cannot be forwarded at the same time)*
![[Pasted image 20220828211358.png]]

#### Switching via a bus
Input ports transfers a packet directly without the need of an routing processor, and this is typically done by having the input port pre-pend a switch-internal label (header) to the packet indicating the local output port to which this packet is being transferred and transmitting the packet onto the bus. So all the output ports will recieve the packets, but only the one who gets a match keeps it.

It will therefor be some traffic by that BUS since it will transmit all the labels. So some might have to wait if many labels arrive at the same time.
![[Pasted image 20220828211652.png]]

#### Switching via an interconnection network
Consits of 2n buses that connect N input ports to N output ports. The switch fabric can the open or close different buses at any time by the controller

So can operate different forwarding of packets in parallel. This method is non-blocking, but if two packets are going the same place it will queded for later.
- *Can also be scaled up to using multiple switching fabrics in parallel*

![[Pasted image 20220828211724.png]]
- *When a packet arrives from port A and needs to be forwarded to port Y, the switch controller closes the crosspoint at the intersection of busses A and Y, and port A then sends the packet onto its bus, which is picked up (only) by bus Y*

## Output Port Processing
Takes packets that have been stored in the output port’s memory and transmits them over the output link. This includes selecting and de-queueing packets for transmission, and performing the needed link-layer and physical-layer transmission functions.

## Where Does Queuing Occur?
#### Input Queueing
If the switch doesn't match the speed to the input line speed to transfer all arriving packets, it will occur input queueing
![[Pasted image 20220828213812.png]]
It shows if two packets (darkly shaded) at the front of their input queues are destined for the same upper-right output port. Suppose that the switch fabric chooses to transfer the packet from the front of the upper-left queue. In this case, the darkly shaded packet in the lower-left queue must wait. But not only must this darkly shaded packet wait, so too must the lightly shaded line switch switch line packet that is queued behind that packet in the lower-left queue, even though there is no contention for the middle-right output port (the destination for the lightly shaded packet)
- **Called head-of-the-line (HOL) blocking**
	- This will often lead to packets loss if the line gets very large

#### Output Queueing
If the speed of the switch is *N* times faster than the Line and packets arriving, it will end up with more packets than the output port can handle. Therefore it will end in a queue for transmission over the outgoing link

Then *N* more packets can possibly arrive in the time it takes to transmit just one of the N packets that had just previously been queued. And so on. Thus, packet queues can form at the output ports even when the switching fabric is *N* times faster than the port line speeds. Eventually, the number of queued packets can grow large enough to exhaust available memory at the output port.
![[Pasted image 20220828215842.png]]
	*In the next time unit, one of these three packets will have been transmitted over the outgoing link. In our example, two new packets have arrived at the incoming side of the switch; one of these packets is destined for this uppermost output port. A consequence of such queuing is that a packet scheduler at the output port must choose one packet, among those queued, for transmission*

To many packets arriving in the queue will either end up with packets being dropped (drop-list) or remove one or more already-queued packets to make room for the newly arrived packets.

Could also to drop or mark the header of a packet before the buffer is full in order to provide a congestion singal to the sender.A number of proactive packet-dropping and -marking policies (which collectively have become known as **active queue management (AQM) algorithms)**

Buffering could be a problem, and it is hard to determine how buffering should be allowed
	Rule of thumb:
		*Buffering (B)=average round trip time (RTT) * link capacity (C )
		Thus, a 10 Gbps link with an RTT of 250 msec would need an amount of buffering equal to B 5 RTT · C 5 2.5 Gbits of buffers*

If there are a large number of TCP flows *N* passing through a link, the amount of buffering needed is B=RTI*C/N

With a large number of flows typically passing through large backbone router links, the value of *N* can be large, with the decrease in needed buffer size becoming quite significant.

## Packet scheduling
#### FIFO - first in, first out
One line, firts packets in, will first enter out

#### Priority queuing
*Put in to priority*
In practice, a network operator may configure a queue so that packets carrying network management information (e.g., as indicated by the source or destination TCP/UDP port number) receive priority over user traffic; additionally, real-time voice-over-IP packets might receive priority over non-real traffic such as SMTP or IMAP e-mail packets.

Each priority class typically has its own queue. When choosing a packet to transmit, the priority queuing discipline will transmit a packet from the highest priority class that has a nonempty queue (that is, has packets waiting for transmission). The choice among packets in the same priority class is typically done in a FIFO manner
![[Pasted image 20220828220257.png]]
![[Pasted image 20220828220304.png]]
	*Packets 1, 3, and 4 belong to the high-priority class, and packets 2 and 5 belong to the low-priority class. Packet 1 arrives and, finding the link idle, begins transmission. During the transmission of packet 1, packets 2 and 3 arrive and are queued in the low- and high-priority queues, respectively. After the transmission of packet 1, packet 3 (a high-priority packet) is selected for transmission over packet 2 (which, even though it arrived earlier, is a low-priority packet). At the end of the transmission of packet 3, packet 2 then begins transmission. Packet 4 (a high-priority packet) arrives during the transmission of packet 2 (a low-priority packet). Under a non-preemptive priority queuing discipline, the transmission of a packet is not interrupted once it has been sent, so pakcet 4 will be queued as the next one*

#### Round Robin and Weighted Fair Queuing (WFQ)
##### Round Robin
To make sure every packets gets access

A so-called work-conserving queuing discipline will never allow the link to remain idle whenever there are packets (of any class) queued for transmission.
![[Pasted image 20220828220458.png]]
	*Here packets are divided into different classes to make sure every packets gets a share. So class 1 and 2, and will be transmitted every second time.*

##### Weighet Fair Queuing
Very similar with the idea that everyone should get a share
WFQ differs from round robin in that each class may receive a differential amount of service in any interval of time.



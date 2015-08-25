###### Austin Archer - https://keybase.io/Goodies

# SlapNAT
**SlapNAT** is a simple, cross-platform Python2.7 and Python3.4 compatible framework for performing dual-NAT traversal using specially crafted ICMP packets and establishing a UDP hole-punched tunnel. Though I did not use any of his code (because I'm a terrible C programmer), it is conceptually based off of Samy Kamkar's Pwnat program which can be found here: <a href="http://samy.pl/pwnat/" target="_blank">Click Me!</a>
 
 Unfortunately, to remain platform independent, the program uses raw ICMP sockets instead of ICMP API's like the <a href="https://msdn.microsoft.com/en-us/library/windows/desktop/aa366050.aspx" target="_blank">IcmpSendEcho()</a> function or the SUID **"ping"** command on \*nix, so it likely needs to be run as an Administrator in Windows, or as root on \*nix systems. However, if anyone more competent than myself is able to recommend a way which can allow non-privileged users to create *customizable* ICMP sockets, by all means, let me know.

## What's the use?
In an effort to find a way in which I am able to create a truly peer-to-peer connection without an intermediate server, I developed this concept to allow two separate machines, each behind a fully secured NAT (i.e. no Port Address Translation or Port Forwarding), to create a seemingly server-less connection. This framework allows you to do that simply. Fortunately for me, but unfortunately for my ego, my concept had already been developed, allowing me to check if my functionality is on track.

## How it Works:
If you'd like to see the paper off of which I based my work, you can visit it on Samy's site here:<br>
<align="center"><a href="http://samy.pl/pwnat/pwnat.pdf" target="_blank">Autonomous NAT Traversal</a> by <a href="http://www.net.in.tum.de/de/mitarbeiter/mueller/" target="_blank">Andreas Muller</a>, <a href="http://www.symantec.com/about/profile/researchlabs/bio.jsp?bioid=nathan_evans">Nathan Evans</a>, <a href="http://grothoff.org/christian/">Christian Grothoff</a>, and <a href="http://samy.pl/">Samy Kamkar</a></align>

PC-A and PC-B desire to create a direct Peer-to-Peer connection, but are behind NAT-A and NAT-B, respectively. Traceroute and Ping, ICMP-based programs, send out an ICMP request packet with a specific signature. When the desired return packet comes back to the router, it forwards it to the necessary host for further processing. Utilizing this concept, the order of events are as follows:

  1. PC-A will send out a statically defined ICMP Echo Request packet to a bogon (unallocated IP address) 1.2.3.4.
  2. PC-B will send an ICMP Time Exceeded packet with an identical copy of PC-A's original static packet.
  3. PC-A will receive this packet due to its router forwarding it, and thus receive NAT-B's IP Address.
  
The next section creates the UDP session by generating a re-usable socket on a given port. Now that the client's IP and server's IP are both known by each device, I use the simple addition of these addresses to seed Python's random generator and select a random port between 1025 and 65535. You'd only better hope that this port is not being used, because it does need to be identical on each side.

  1. PC-A sends out several UDP Datagram to NAT-B and does **not** receive a direct response.
  2. PC-B sends out several UDP Datagram, to NAT-A and **does** receive a direct response.
  3. The UDP tunnel establishes as one of the NATs allows the packet through.
  4. Perhaps TCP over UDP?
  
## Documentation:
In this section, I'll go over each method and attribute that you (should) alter or call. Note that any changes to the Data or ID of any packet will require you to ensure that both endpoints have the same change.
#### ICMP
The ICMP class is used for oneclient to discovery the peer's IP address provided that the second peer knows the IP of the first. If both clients provide one another's IP's, the ICMP objects are unnecessary.


```request = slapnat.icmprequest(self, data=_ICMP_DATA, id=_ICMP_ID)```<br>
Creates the Echo Request packet and sets the ICMP type to <a href="https://en.wikipedia.org/wiki/Internet_Control_Message_Protocol#Control_messages" target="_blank">8</a>. Inherents the `_icmp` class. `data` must be a \<str\> type in Python2 and a \<bytes\> type in Python3. `id` must be an unsigned long integer, preferably formatted such as `0xbeef`.

```timeout = slapnat.icmptimeout(self, request=icmprequest())```<br>
Creates the ICMP Time to Live Exceeded packet based off of an `icmprequest()` instance. Inherents the `_icmp` class. Pass an instance of `icmprequest` if you are not using the default Data and ID settings.

```instance.createSocket(self, destination)```<br>
Creates a raw ICMP socket that will send to the desired destination. Located in the `_icmp` base class. The socket will be stored in `instance._socket`, and the destination is stored in `instance._destination`. These should not be accessed directly.


```instance.sendEvery(self, frequency=3)```<br>
Creates a thread and an event to send the previously constructed packet data via the socket created using the aforementioned method. If no such object exists, it will throw a `NameError`. Thread is stored in `instance._thread`, frequency is stored in `instance._freq`, and the threading event is stored in `instance._stop`. These should not be accessed directly.

```instance.stop(self)``` <br>
Sets the `instance._stop` event to halt the sending of the packet. This should be done when you receive the IP of the other party (if it is not provided).

## Undocumentation:
These are the methods and attributes that you should not call or alter directly. This is purely for undocumentation purposes.

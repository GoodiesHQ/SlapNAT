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
  
## Usage:
I'll let you know as soon as I do!

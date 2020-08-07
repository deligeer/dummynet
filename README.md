The dummynet project
http://i1.ytimg.com/vi/r8vBmybeKlE/default.jpg
Documentation: Dummynet revisited, Computer Communication Review, 40(2), March 2010; GoogleTechTalks video, March 2010; FreeBSD manual page
Source/binary code: FreeBSD/Linux/Windows (20120812) includes windows binaries
Netmap-enabled version: Netmap-enabled version (20120813)
Index
News
Description
Performance
Documentation
Availability
Papers
Presentations
Sigcomm community project award
ipfw meets netmap
News
20150831 the most up-to-date version of the ipfw+dummynet code is on github at https://github.com/luigirizzo/dummynet
20130607 new source code for Linux up to 3.8 and Windows (source code and windows binaries)
20120813 netmap-enabled userspace version (FreeBSD/Linux) runs at 6.5 Mpps (source code)
20120812 updated source code distribution for Linux and Windows (source code and windows binaries)
20120701 dummynet receives ACM/SIGCOMM support
Description
 datafiles/dummynet.gifdummynet is a live network emulation tool, originally designed for testing networking protocols, and since then used for a variety of applications including bandwidth management. It simulates/enforces queue and bandwidth limitations, delays, packet losses, and multipath effects. It also implements various scheduling algorithms. dummynet can be used on the machine running the user's application, or on external boxes acting as routers or bridges.
 dummynet runs within your operating system (FreeBSD, OSX, Linux, Windows) and works by intercepting selected traffic on its way through the network stack, as in the figure above, and passing packets to objects called pipes which implement a set of queues, a scheduler, and a link, all with configurable features (bandwidth, delay, loss rate, queue size, scheduling policy...).
 Traffic selection is done using the ipfw firewall, which is the main user interface for dummynet. ipfw lets you select precisely the traffic and direction you want to work on, making configuration and use incredibly simple. You can create multiple pipes, send traffic to different pipes, even build cascades of pipes.
 Just a few examples:
 limit the total incoming TCP traffic to 2Mbit/s, and UDP to 300Kbit/s
     ipfw add pipe 2 in proto tcp
         ipfw add pipe 3 in proto udp
	     ipfw pipe 2 config bw 2Mbit/s
	         ipfw pipe 3 config bw 300Kbit/s
		 limit incoming traffic to 300Kbit/s for each host on network 10.1.2.0/24
		     ipfw add pipe 4 src-ip 10.1.2.0/24 in
		         ipfw pipe 4 config bw 300Kbit/s queue 20 mask dst-ip 0x000000ff
			 simulate an ADSL link to the moon:
			     ipfw add pipe 3 out
			         ipfw add pipe 4 in
				     ipfw pipe 3 config bw 128Kbit/s queue 10 delay 1000ms
				         ipfw pipe 4 config bw 640Kbit/s queue 30 delay 1000ms
					 dummynet is widely used. It is a standard component in FreeBSD and OSX, it is used as link emulator on Emulab, PlanetLab, Hen and many private testbeds.

Performance
For performance please read the Dummynet Revisited paper. On a modern system you should be able to deal 2-300 Kpps. The standard timer resolution is 1 ms but you can reduce it to 100 us.
If you need higher packet rates, you may want to check the netmap-enabled version, which is 10 times faster.
Documentation
dummynet and ipfw have many options documented in the ipfw manual page
Availability
dummynet and ipfw are standard components of FreeBSD and OSX. They are also available as external kernel modules for Linux and Windows (both 32 and 64 bit).
The source code distribution contains source code to build it on Linux and Windows, as well as precompiled modules for Windows XP/Win7 (both 32 and 64 bit).
In addition to the source code, we also distribute some bootable images and precompiled binaries:
PicoBSD image (feb.2010) with dummynet and several utilities (ssh, sshd, natd and so on). You can copy it on a USB key and boot from it, or run it in qemu.
OpenWRT binary package for ASUS WL500GP (2.4.35)
Papers
If you need to reference dummynet in a paper, please use the most recent paper
Marta Carbone and Luigi Rizzo, Dummynet Revisited, ACM SIGCOMM Computer Communication Review, 40(2) pg.12-20, March 2010
There are other papers of ours describing dummynet or parts of it, including the following (the links are to draft copies):
M. Carbone and L. Rizzo, An emulation tool for PlanetLab, Computer Communications, Elsevier, Oct.2011, doi:10.1016/j.comcom.2011.06.004
F.Checconi, P. Valente, and L.Rizzo, QFQ: Efficient Packet Scheduling with Tight Bandwidth Distribution Guarantees, Mar 2012 (to appear in IEEE/ACM Transactions on Networking)
and the original dummynet paper,
L.Rizzo, Dummynet: a simple approach to the evaluation of network protocols, ACM SIGCOMM Computer Communication Review 27 (1), 31-41, 1997
Presentations
GoogleTechTalk http://i1.ytimg.com/vi/r8vBmybeKlE/default.jpg presentation on dummynet and the QFQ scheduler,
slides for the above;
BSDCan2010 slides on porting dummynet to Linux and Windows;
BSDCan2010 slides on schedulers in dummynet
Sigcomm community project award
In May 2012 ACM SIGCOMM has granted the dummynet project a SIGCOMM Community Projects Award. The award will be used to support development of dummynet on multiple platforms and improve its performance.
ipfw meets netmap
A userspace version of ipfw and dummynet is now available, using netmap for packet I/O. On an i7-3400, this version is able to process over 6 million packets per second (Mpps) with simple rulesets, and over 2.2 Mpps through dummynet pipes, 5..10 times faster than the in-kernel equivalent.
You can use a couple of VALE switches or netmap pipes (part of netmap) to connect a source and sink to the userspace firewall, as follows
                s       f               f       d
		   [pkt-gen]-->--[valeA]-->--[kipfw]-->--[valeB]-->--[pkt-gen]

The commands to run (in separate windows) are the following
        # preliminarly, load the netmap module
	        sudo kldload netmap.ko

        # connect the firewall to two vale switches
	        ./kipfw valeA:f valeB:f &

        # configure ipfw/dummynet
	        ipfw/ipfw show  # or other

        # start the sink
	        pkt-gen -i valeB:d -f rx

        # start an infinite source
	        pkt-gen -i valeA:s -f tx

        # plain again with the firewall and enjoy
	        ipfw/ipfw show  # or other# dummynet

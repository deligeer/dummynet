# Tutorial

## Enabling IPFW and DummyNet:

1. Install and boot FreeBSD
2. Add the following lines to /etc/rc.conf:
firewall_enable="YES"
firewall_type="OPEN"

Load dummynet with:

kldload dummynet

Next you need to configure the ipfw rules. List the rules with
```
ipfw list
```
Add the -a option to list how many times each rule has been used.
Note that the rules have order, determined by their rule numbers.
If a packet is handles by a rule before your rule, it may never make it to your rule.

You can delete rules with
```
ipfw delete RULE#
```
You can delete all rules with
```
ipfw flush
```
Add access with
```
ipfw add 1000 allow all from any to any
```
## Basic ipfw commands:
```
ipfw add [N] [prob X] action PROTO from SRC to DST [options]
```
where N is the rule number X is a number between 0 and 1 that, which indicates the probability of getting a match on this rule if all other fields are correct. 'action' is one of the actions executed on a match, which can be any of allow, deny, skipto N, pipe N and others.
To send a packet to a dummynet pipe, we have to use pipe N; PROTO is the protocol type we want to match (IP, TCP, UDP, ...);
SRC and DST are address specifier (we can use addresses with netmasks and optionally followed by ports or port ranges);
options can be used to restrict the attention to packets coming from/to specific interfaces, or carrying some TCP flags or ICMP options, or bridged, etc.

## Creating and Destroying a single pipe:

Let us consider the traffic flowing between two hosts "larry.baylor.edu" (larry) and "curly.baylor.edu" (curly)

To create a pipe (1) between larry and curly.
```
ipfw add 100 pipe 1 ip from any to any
```
	Note that this should come before allow/deny ipfw rules.
This command will create a single pipe on the network allowing full duplex data transfer between larry and curly.
```
ipfw pipe 1 show
```
This command will show all the parameters associated with the pipe. For a dynamically created pipe, all the corresponding pipes are shown.
```
ipfw pipe 1 delete
```
This command will destroy the pipe 1.
```
ipfw pipe flush
```
This command will destroy all the pipes generated.

## Configuring the pipes:

### Bandwidth

Setting the bandwidth of the traffic between the hosts. The bandwidth can be any of bit/s , Kbit/s,Mbits/s, Byte/s. KByte/s , MByte/s. A bandwidth of zero results in no bandwidth limitation.
```
ipfw pipe 1 config bw  100Kbit/s
```
This command limits the bandwidth of the pipe 1 to 100Kbit/s.

### Queue Size

The queue size can also be set, which along with bandwidth influences the queueing delay. The queue size can be specified as number of slots, in Bytes or in KBytes.
```
ipfw pipe 1 config queue 100KByte/s
```
This command limits the queue sizeof the pipe 1 to 100KByte/s.

### Delay

The propogation delay of the pipes can also be controlled and can be set to any desired value ain milliseconds. The documentation states that the queueing delay is independent of the propogation delay.

ipfw pipe 1 config delay 100ms
This command sets the desired propogation delay to 100ms.

### Random Packet Loss

The packet loss in a network can also be simulated in the dummynet. The command plr X, where X is a floating point number between 0 and 1 which causes packets to be dropped at random simulates packet loss, where 0 is for no loss and 1 is for 100%packet loss.

ipfw pipe 1 config plr 0.5
This command drops packets randomly , sending almost half the number of packets across the network.

### Dynamic queue creation:

Associating a mask to a particular pipe handles the packets separately to that pipe . Thus the bandwidth and the queueing limitations are enforced separately for the packets with a particular mask. One or many masks can be set , or the "all" keyword to mean that all fields are significant for the same flow
```
ipfw pipe 1 config mask ([proto N] [src-ip N] [dest-ip N] [src-port N] [dest-port N]
```
N is the bitmask where significant bits are set to 1.

## Using Dummy Net for testing Protocols.

Since dummynet was originally developed to test network protocols and applications, on a network and  also on a standalone system , its features can be used to test protocols. The author gives a few suggestion while configuring the dummynet , to avoid incorrect results.

Choose a reasonable buffer size.
Dummy net can simulate Half-Duplex as well as Full-Duplex, but since its non-realistic, he suggests to use it only for testing purposes
        Half-Duplex transmission
```
ipfw add pipe 1 ip from curly to larry

ipfw pipe 1 config [different parameters]
```
this command simulates a half-duplex from curly to larry, and the different parameters can be tested for that pipe.

While implementing multicast on dummynet, the author suggests to set the rules right keeping in mind that the traffic between in the bridge goes through the firewall code twice.
Since dummynet can be used on a standalone system also , there are a few suggestions by the author, while testing this feature.
The firewall is invoked on all packets, the MTU of the loopback interface defaults to 16KB, and special care has to be taken while running TCP keeping in mind the window size with which TCP starts the transmission.

## Multipath transmission using dummynet.

Using dummynet multiple paths between hosts can easily be simulated , using the probabilistic feature.
```
ipfw add prob 0.33 pipe 1 ip from larry to curly

ipfw pipe 1 config [parameters]

ipfw add prob 0.5 pipe 2 ip from larry to curly

ipfw pipe 2 config [parameters]

ipfw add pipe 3 ip from larry to curly.
```
Given the right packet, the first rule will match with probability 1/3, of the remaining 2/3rds, by the second rule the packets will match with 1/2 probability and the remaining will move to the third rule. Each of these pipes can be individually configured to emulate desired phenomenon.

References:

Dummynet: A simple approach to the evaluation of network protocols.(http://www.iet.unipi.it/~luigi/research.htmlhttp://www.iet.unipi.it/~luigi/research.html)

Dummynet home.(http://www.iet.unipi.it/~luigi/ip_dummynet/)


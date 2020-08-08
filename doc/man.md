# ipfw(8) manpage
ipfw syntax summary (but please do read the ipfw(8) manpage):

        ipfw [-abcdefhnNqStTv] <command>

where <command> is one of the following:

add [num] [set N] [prob x] RULE-BODY
{pipe|queue} N config PIPE-BODY
[pipe|queue] {zero|delete|show} [N{,N}]
nat N config {ip IPADDR|if IFNAME|log|deny\_in|same\_ports|unreg\_only|reset|
                reverse|proxy_only|redirect_addr linkspec|
		                redirect_port linkspec|redirect_proto linkspec}
				set [disable N... enable N...] | move [rule] X to Y | swap X Y | show
				set N {show|list|zero|resetlog|delete} [N{,N}] | flush
				table N {add ip[/bits] [value] | delete ip[/bits] | flush | list}
				table all {flush | list}

RULE-BODY:      check-state [PARAMS] | ACTION [PARAMS] ADDR [OPTION_LIST]
ACTION: check-state | allow | count | deny | unreach{,6} CODE |
               skipto N | {divert|tee} PORT | forward ADDR |
	                      pipe N | queue N | nat N | setfib FIB | reass
			      PARAMS:         [log [logamount LOGLIMIT]] [altq QUEUE_NAME]
			      ADDR:           [ MAC dst src ether_type ]
			                      [ ip from IPADDR [ PORT ] to IPADDR [ PORTLIST ] ]
					                      [ ipv6|ip6 from IP6ADDR [ PORT ] to IP6ADDR [ PORTLIST ] ]
							      IPADDR: [not] { any | me | ip/bits{x,y,z} | table(t[,v]) | IPLIST }
							      IP6ADDR:        [not] { any | me | me6 | ip6/bits | IP6LIST }
							      IP6LIST:        { ip6 | ip6/bits }[,IP6LIST]
							      IPLIST: { ip | ip/bits | ip:mask }[,IPLIST]
							      OPTION_LIST:    OPTION [OPTION_LIST]
							      OPTION: bridged | diverted | diverted-loopback | diverted-output |
							              {dst-ip|src-ip} IPADDR | {dst-ip6|src-ip6|dst-ipv6|src-ipv6} IP6ADDR |
								              {dst-port|src-port} LIST |
									              estab | frag | {gid|uid} N | icmptypes LIST | in | out | ipid LIST |
										              iplen LIST | ipoptions SPEC | ipprecedence | ipsec | iptos SPEC |
											              ipttl LIST | ipversion VER | keep-state | layer2 | limit ... |
												              icmp6types LIST | ext6hdr LIST | flow-id N[,N] | fib FIB |
													              mac ... | mac-type LIST | proto LIST | {recv|xmit|via} {IF|IPADDR} |
														              setup | {tcpack|tcpseq|tcpwin} NN | tcpflags SPEC | tcpoptions SPEC |
															              tcpdatalen LIST | verrevpath | versrcreach | antispoof
																       

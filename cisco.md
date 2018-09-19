#CISCO commands
===

[Cheat Sheets](http://packetlife.net/library/cheat-sheets/)

> <> denotes a required field

> [] denotes an optional field or argument

Missing lab 3.3.2.3


###Basic config

```
hostname <NAME>
no ip domain-lookup
security passwords min-length 10
enable secret <PASSWORD>
service password-encryption
banner motd #<MESSAGE>#
clock set <HH:MM:SS> <DATE> <MONTH> <YEAR>
ip default-gateway <IP>            --> required for remote access
```
TODO: what are the other two ways to secure the password? (max attempts and complexity?)

###Basic console/VTY config

```
line con 0 (OR) line vty 0 15
    password <PASSWORD>
    exec-timeout <MIN> <SEC>
    login
    logging synchronous
    no telnet refuse-negotiations            --> one-time use
    transport input all
```

###Basic interface config

```
interface <NAME>
    description <DESCRIPTION>
    ip address <IP> <MASK>
    no shutdown
    clock rate 128000       --> DCE end only
    ipv6 address <IPv6>/<MASK>
    ipv6 address fe80::1 link-local
    exit
ipv6 unicast-routing
```

###SSH config/Local user creation

```
ip domain-name <URL>
username <NAME> privilege <NUM> secret <PASSWORD>
line vty 0 15
    transport input ssh
    login local
    exit
crypto key generate rsa modulus <NUM>
ip ssh time-out <SEC>
ip ssh authentication-retries <#>
```

###Creating static routes

```
ip route <DEST IP> <DEST MASK> <NEXT HOP IP>        --> recursive static route
ip route <DEST IP> <DEST MASK> <EXIT INT>            --> directly connected static route
ip route 0.0.0.0 0.0.0.0 <DEST IP (OR) EXIT INT>        --> default static route
ipv6 route ::/0 <INT>        --> default static route

show ip route
```

###RIPv2 config

```
router rip
    version 2
    passive-interface <INT>
    network <IP>
    no auto-summary
    default-information originate    --> advertise static routes
```

###EIGRP

```
router eigrp <AS>
    network <NETWORK> [WILDCARD]
    passive-interface <INT>        --> disables hello packets
    no auto-summary    --> off by default in IOS 15
    redistribute-static        --> propagate default routes
    eigrp router-id <IP>        --> IPv6
    no shutdown        --> IPv6
    exit
interface <INT>
    bandwidth <KB>        --> only affects EIGRP metric
    ip bandwidth-percent eigrp <AS> <%>
    ip hello-interval eigrp <AS> <SECS>
    ip hold-time eigrp <AS> <SECS>
    ipv6 eigrp <AS>        --> IPv6
    exit

show ip protocols
show ip route eigrp
show ip eigrp interfaces detail    
```
TODO: include ipv6 examples (commands differ significantly)

![](https://i.imgur.com/QbWRYSA.png)

###VLAN creation, config, and port assignment

```
vlan <#>
    name <NAME>
    exit
interface vlan<#>
    ip address <IP> <MASK>
    no shutdown
    exit
interface <INT>
    switchport mode access
    switchport access vlan <#>
    exit
show vlan brief
```

Configure an 802.1Q trunk:
```
interface <INT>
    switchport mode trunk
    exit

show interfaces trunk
```

Router-on-a-stick:
```
interface <INT>.<SUB#>
    encapsulation dot1Q <VLAN#> [native]
    ip address <IP> <MASK>
interface <INT>
    no shutdown
```

###VTP

```
vtp domain <NAME>
vtp mode <server/client>
vtp password <PASSWORD>

show vtp status
```

###DTP

```
interface <INT>
    switchport mode dynamic desirable
```

###Port security

```
interface <INT>
    switchport port-security
    switchport port-security mac-address sticky
    switchport port-security maximum <#>
    switchport port-security violation <protect/restrict/shutdown>
    mac-address <aaaa.bbbb.cccc>
    exit

show interface <INT>        --> verify port violation mode status
show port-security
show port-security address
show port-security interface <INT>
```
![](https://i.imgur.com/Mp2L9Bx.png)

###ACL

Router(config) mode
	access-list <#> <permit/deny> <address to match/any (any host)/ host (single host address)>
	# = 1-99 standard ACL, 100-199 extended ACL
	ex. access-list 99 permit host 10.0.0.1
	
	
Implementing ACL:
	line vty 0 15
	access-class <#> <in/out>
	
	Allow Pings
		access-list 102 permit icmp any any echo-reply

###NAT
```
Router(config) mode
	**ip nat pool** <*name*> *start-ip end-ip* {**netmask** *netmask* | **prefix-length** *prefix-length* }
	**access-list** *access-list-number* **permit** *source* [*source-wildcard*]
	**ip nat inside source list** *access-list-number* **pool** *name*
	**interface** *type number*
	**ip address** *ip-address mask*
	**ip nat inside**
	**exit**
	**interface** *type number*
	**ip address** *ip-address mask*
	**ip nat outside**
```
###EtherChannel

PAgP:
```
interface range <INTS>
    channel-group <#> mode <desirable/auto>
    no shutdown        --> must be enabled AFTER setting PAgP
    exit
interface port-channel <#>
    switchport mode trunk
    switchport trunk native vlan <#>
    exit

``` 
![](https://i.imgur.com/20oM6H9.png)

LACP:
```
interface range <INTS>
    switchport mode trunk
    switchport trunk native vlan <#>
    channel-group <#> mode <active/passive>
    no shutdown
    exit

show run interface <INT>
show interfaces <INT> switchport
show etherchannel summary
```
![](https://i.imgur.com/F9M1vQG.png)

###HSRP

```
interface <INT>        --> Active router
    standby version 2
    standby <#> ip <IP>
    standby <#> priority <#>
    standby <#> preempt
    exit

interface <INT>        --> Standby router
    standby version 2
    standby <#> ip <IP>
    exit

show standby [brief]
```

###"Show" commands

```
show running-config
   OR -> show startup config
show flash 
   OR -> dir flash:
show version
show ip interface brief
   OR -> show ipv6 interface brief
   OR -> show ip interface vlan<#>
   OR -> show ip interface <INT>
show mac address-table
show ip ssh
```

###OSPF

```
[ipv6] router ospf <PROCESS ID>    --> only has meaning locally
    network <IP> <WILDCARD> area <#>
    router-id <IP>
    passive-interface <INT/default>
    auto-cost reference-bandwidth <#>
    redistribute static        --> propagate default routes
    default-information originate
    area <AREA> authentication message-digest        --> sometimes applies globally
    area <AREA> range <IP> <MASK>        --> summary route
    auto-cost reference-bandwidth <#>
interface <INT>
    bandwidth <KB>        --> only affects metric, must be applied on both sides
    ip ospf cost <#>        --> must be set on both sides
    ip ospf message-digest-key <KEYID> md5 <PASSWORD>
    ip ospf authentication message-digest    --> non-global
    ip ospf hello-interval <SEC>
    ip ospf dead-interval <SEC>
    ipv6 ospf <#> area <AREA>
    ipv6 ospf network point-to-point        --> only for loopbacks

show ip ospf neighbor
show ip protocols
show ip ospf database
show ip ospf [interface [brief/INT]]   --> verify costs and hello timers
clear ip ospf process        --> say yes
do show ipv6 route ospf
debug ip ospf adj
```
TODO: IPv6 commands


###Misc

 
- You can pipe the output of "show" commands to the `include` or `begin` commands to more quickly find what you need

- `clear ip route` will clear the routing

- `config-register 0x2102`

- `terminal length 0`

 
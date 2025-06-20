### Training Basic configuration RSTvX Hayup labs

~~~
R4: (Interface config IP)
conf t
enable secret pass
service password-encryption
no logging console
no ip domain-lookup
line console 0
login
password pass
exec-timeout 0 0
line vty 0 4
login
password pass
exec-timeout 0 0
int e1/2
no shut
ip add 10.1.1.10 255.255.255.252
int e1/0
no shut
ip add 10.1.4.5 255.255.255.252
int e1/1
no shut
ip add 10.1.4.9 255.255.255.252
do sh ip int brief | ex una	---> verify
~~~
~~~
D1: (Interface config IP)
conf t
enable secret pass
service password-encryption
no logging console
no ip domain-lookup
line console 0
login
password pass
exec-timeout 0 0
line vty 0 4
login
password pass
exec-timeout 0 0
int e1/1
no shut
ip add 10.1.4.6 255.255.255.252
do sh ip int brief | ex una	---> verify

D1: (VLAN creation with ip address)
conf t
vlan 10
name homevlannov6
exit
int vlan 10
ip add 10.2.1.1 255.255.255.252
no shut
exit
vlan 20
name vlan20.com.ph
exit
int vlan 20
ip add 10.2.2.1 255.255.255.252
no shut
exit
vlan 100
name server
exit
int vlan 100
ip add 192.168.1.129 255.255.255.128
no shut
exit
do sh ip int brief | ex una	---> verify
~~~
~~~
D2: (Interface config IP)
conf t
enable secret pass
service password-encryption
no logging console
no ip domain-lookup
line console 0
login
password pass
exec-timeout 0 0
line vty 0 4
login
password pass
exec-timeout 0 0
int e1/1
no shut
ip add 10.1.4.10 255.255.255.252
do sh ip int brief | ex una	---> verify

D2: (VLAN creation with ip address)
config t 
vlan 10
 name homevlannov6
 exit
int vlan 10
ip add 10.2.1.2 255.255.255.252
no shut
exit
vlan 20
name vlan20.com.ph
exit
int vlan 20
ip add 10.2.2.2 255.255.255.252
no shut
exit
vlan 100
name server
exit
int vlan 100
ip add 192.168.1.130 255.255.255.128
exit
do sh ip int brief | ex una	---> verify
~~~
~~~
A1: (basic config)
conf t
hostname A1
enable secret pass
service password-encryption
no logging console
no ip domain-lookup
line vty 0 4
login
password pass
exec-timeout 0 0 
line console 0
login
password pass
exec-timeout 0 0

A1: (Vlan creation)
vlan 100
name server
exit
int vlan 100
ip add 192.168.1.131 255.255.255.128
no shut
exit
~~~
~~~
A2: (basic config)
conf t
hostname A2
enable secret pass
service password-encryption
no logging console
no ip domain-lookup
line console 0
login
password pass
exec-timeout 0 0
line vty 0 4
login
password pass
exec-timeout 0 0

A2: (Vlan creation)
conf t
vlan 100
name server
exit
int vlan 100
ip add 192.168.1.132 255.255.255.128
no shut
exit

do ping 192.168.1.132 ---> verify
do ping 192.168.1.131 ---> verify
do ping 192.168.1.130 ---> verify
do ping 192.168.1.129 ---> verify
~~~
~~~
Give IP Address to P1 and P2, Via DHCP server on D1/D2 using load balance.

D1: Create DHCP + Switched virtual interface (will give .101 - .199)
conf t
ip dhcp excluded-add 10.2.1.200 10.2.1.254
ip dhcp pool vlan10
network 10.2.1.0 255.255.255.0
default-router 10.2.1.1
domain-name rivanit.com
dns-server 8.8.8.8
option 150 ip 1.1.1.1


D2: Create DHCP (will give .200 - .249)
conf t
ip dhcp excluded-add 10.2.1.1 10.2.1.199
ip dhcp pool vlan10
network 10.2.1.0 255.255.255.0
default-router 10.2.1.2
domain-name rivanit.com
dns-server 8.8.8.8
option 150 ip 1.1.1.1
~~~
~~~
A1, A2: Ask for Vlan and Move Port:

A1:
conf t
int e0/0
switchport mode access
switchport access vlan 10
do sh vlan brief  ---> verify

A2:
conf t
int e1/0
switchport mode access
switchport access vlan 10
do sh vlan brief   ---> verify
~~~
~~~
P1:
conf t
int e0/0
ip add dhcp
no shut
do sh ip int brief |ex una  ---> verify

P2:
conf t
int e1/0
ip add dhcp
no shut
do sh ip int brief | ex una  ---> verify
~~~

~~~
Basic Routing Two Commandments.
1.) Thou shall: "Ping DIKIT!"
2.) Thou shall: "Route Not Dikit"

IP route "network/subnet" subnetMask 1sthopIP metric

Static Route:

@R1: [ tell r1 how to go to 10.1.1.4/30 and 10.1.1.8/30 ]
sh ip route static  --> verify
sh ip route connected | inc C	--> verify
conf t
ip route 10.1.1.4 255.255.255.252 10.1.1.2 1
ip route 10.1.1.8 255.255.255.252 10.1.1.2 1
exit
do ping 


@R2: [ tell r2 how to go 10.1.1.8/30 ]
sh ip route static  --> verify
sh ip route connected | inc C	--> verify
conf t
ip route 10.1.1.8 255.255.255.252 10.1.1.6 1
exit
do ping


@R3: [ tell r3 how to go 10.1.1.0/30 ]
sh ip route static  --> verify
sh ip route connected | inc C	--> verify
conf t
ip route 10.1.1.0 255.255.255.252 10.1.1.5 1
exit
do ping


@R4: [ tell r4 how to go 10.1.1.4/30 and 10.1.1.0/30 ]
sh ip route static  --> verify
sh ip route connected | inc C	--> verify
conf t
ip route 10.1.1.4 255.255.255.252 10.1.1.9 1
ip route 10.1.1.0 255.255.255.252 10.1.1.9 1
do ping 

~~~
~~~

Default route vs static routes:
D1/D2: routes they need to know: (coming from R1,R2,R3)
10.1.1.0/30
10.1.1.4/30
10.1.1.8/30

D1: 
conf t
ip route 10.1.1.0 255.255.255.252 10.1.4.5
ip route 10.1.1.4 255.255.255.252 10.1.4.5
ip route 10.1.1.8 255.255.255.252 10.1.4.5
end
sh ip route connected |inc C  ---> verify


D2:
conf t
ip route 10.1.1.0 255.255.255.252 10.1.1.9
ip route 10.1.1.4 255.255.255.252 10.1.1.9
ip route 10.1.1.8 255.255.255.252 10.1.1.9
end
sh ip route connected |inc C

R1,R2,R3: routes they need to know: (coming from D1,D2)
10.1.4.4/30
10.1.4.8/30
10.2.1.0/30
10.2.2.0/30
192.168.1.128/25

@R1:
conf t
ip route 10.1.4.4 255.255.255.252 10.1.1.2
ip route 10.1.4.8 255.255.255.252 10.1.1.2
ip route 10.2.1.0 255.255.255.252 10.1.1.2
ip route 10.2.2.0 255.255.255.252 10.1.1.2
ip route 192.168.1.128 255.255.255.128 10.1.1.2

@R2:
conf t
ip route 10.1.4.4 255.255.255.252 10.1.1.6
ip route 10.1.4.8 255.255.255.252 10.1.1.6
ip route 10.2.1.0 255.255.255.252 10.1.1.6
ip route 10.2.2.0 255.255.255.252 10.1.1.6
ip route 192.168.1.128 255.255.255.128 10.1.1.6

@R3:
conf t
ip route 10.1.4.4 255.255.255.252 10.1.1.10
ip route 10.1.4.8 255.255.255.252 10.1.1.10
ip route 10.2.1.0 255.255.255.252 10.1.1.10
ip route 10.2.2.0 255.255.255.252 10.1.1.10
ip route 192.168.1.128 255.255.255.128 10.1.1.10



R4: (coming from D1,D2) with metric
10.2.1.0/30
10.2.2.0/30
192.168.1.128/25


@R4: 
conf t
!primaryroutes:
ip route 10.2.1.0 255.255.255.252 10.1.4.6 1
ip route 10.2.2.0 255.255.255.252 10.1.4.6 1
ip route 192.168.1.128 255.255.255.128 10.1.4.6 1
conf t
!backuproutes:
ip route 10.2.1.0 255.255.255.252 10.1.4.10 10
ip route 10.2.2.0 255.255.255.252 10.1.4.10 10
ip route 192.168.1.128 255.255.255.128 10.1.4.10 10
show ip route static --->verify


@P1:
conf t
ip route 0.0.0.0 0.0.0.0 10.2.1.1
int e0/0
no shut
ip add 10.2.1.101 255.255.255.0
show cdp neighbor  ---> verify
sh ip route static ---> verify
ping  ---> verify

@P2:
conf t
ip route 0.0.0.0 0.0.0.0 10.2.1.2
int e1/0
no shut
ip add 10.2.1.102 255.255.255.0
show cdp neighbor  ---> verify
sh ip route static ---> verify
ping  ---> verify


How to config BGP from the scratch: (not include on exam
@R1:
conf t
router bgp 1 
bgp router-id 1.1.1.1
bgp log-neighbor-changes
neighbor 209.9.9.3 remote-as 3
neighbor 207.7.7.2 remote-as 2 
neighbor 208.8.8.4 remote-as 45
network 209.9.9.0 mask 255.255.255.0
network 207.7.7.0 mask 255.255.255.0
network 208.8.8.0 mask 255.255.255.0
network 10.1.1.0 mask 255.255.255.252
end

@@@ISP1:
CONFIG T
router bgp 45
bgp log-neighbor-changes
neighbor 24.2.4.2 remote-as 2
neighbor 45.4.5.5 remote-as 45
neighbor 208.8.8.1 remote-as 1
network 208.8.8.0
network 24.2.4.0 mask 255.255.255.0
network 44.44.44.44 mask 255.255.255.255
network 45.4.5.0 mask 255.255.255.0
!PretentInternet
network 0.0.0.0
ip route 0.0.0.0 0.0.0.0 null 0
ip route 10.0.0.0 255.0.0.0 208.8.8.1
end

@@@ISP2:
config t
router bgp 2
bgp log-neighbor-changes
neighbor 24.2.4.4 remote-as 45
neighbor 25.2.5.5 remote-as 45
neighbor 32.3.2.3 remote-as 3
neighbor 207.7.7.1 remote-as 1
network 207.0.0.0
network 22.22.22.22 mask 255.255.255.255
network 24.2.4.0 mask 255.255.255.0
network 25.2.5.0 mask 255.255.255.0
network 32.3.2.0 mask 255.255.255.0
!fakeInternet
network 0.0.0.0
ip route 0.0.0.0 0.0.0.0 null 0
ip route 10.0.0.0 255.0.0.0 207.7.7.1
end

@@ISP3:
CONFIG T
router bgp 3
bgp log-neighbor-changes
neighbor 32.3.2.2 remote-as 2
neighbor 35.3.5.5 remote-as 45
neighbor 209.9.9.1 remote-as 1
network 209.9.9.0
network 32.3.2.0 mask 255.255.255.0
network 33.33.33.33 mask 255.255.255.255
network 35.3.5.0 mask 255.255.255.0
network 0.0.0.0
ip route 0.0.0.0 0.0.0.0 null 0
ip route 10.0.0.0 255.0.0.0 209.9.9.1
end

@@Isp4:
config t
int lo 8
 ip add 8.8.8.8 255.255.255.255
router bgp 45
 bgp log-neighbor-changes
 neighbor 25.2.5.2 remote-as 2
 neighbor 35.3.5.3 remote-as 3
 neighbor 45.4.5.4 remote-as 45
 network 8.8.8.8 mask 255.255.255.255
 network 55.55.55.55 mask 255.255.255.255
 network 25.2.5.0 mask 255.255.255.0
 network 35.3.5.0 mask 255.255.255.0
 network 45.4.5.0 mask 255.255.255.0
end

Create eigrp 100
@D1:
config t
router eigrp 100
 no auto-summary
 network 10.1.4.4 0.0.0.3
 network 10.2.1.0 0.0.0.255
 network 10.2.2.0 0.0.0.255
 network 192.168.1.128 0.0.0.31
show ip eigrp neighbor  ---> verify

@D2:
config t
router eigrp 100
 no auto-summary
 network 10.1.4.8 0.0.0.3
 network 10.2.1.0 0.0.0.255
 network 10.2.2.0 0.0.0.255
 network 192.168.1.128 0.0.0.31
show ip eigrp neighbor  ----> verify


@R4:
config t
router eigrp 100
 no auto-summary
 network 10.1.4.8 0.0.0.3
 network 10.1.4.4 0.0.0.3
 end
show ip eigrp neighbor  ----> verify

        
!HOW TO CHECK IF EIGRP IS WORKING PROPERLY:
SIR: show ip protocols
SIEN: show ip EIGRP neighbors --> super, meron kapitbahay
SIET: show ip eigrp topology --> super, madami ang P, P lahat
show ip eigrp traffic
show ip route eigrp

Create OSPF Network  -->pending
@R4:
config t
Int loopback 0
 ip add 4.4.4.4 255.255.255.255
router ospf 1
router-id 4.4.4.4
network 4.4.4.4 0.0.0.0 area 0
network 10.1.1.8 0.0.0.3 area 0
interface e1/2
   ip ospf network point-to-point
   !eto lab sa exam sa 2024/2025
end

@R3:
config t
Int loopback 0
 ip add 3.3.3.3 255.255.255.255
router ospf 1
router-id 3.3.3.3
network 3.3.3.3 0.0.0.0 area 0
network 10.1.1.4 0.0.0.3 area 0
network 10.1.1.8 0.0.0.3 area 0
interface e1/2
   ip ospf network point-to-point
end

@R2:
config t
Int loopback 0
 ip add 2.2.2.2 255.255.255.255
router ospf 1
router-id 2.2.2.2
network 2.2.2.2 0.0.0.0 area 0
network 10.1.1.4 0.0.0.3 area 0
network 10.1.1.0 0.0.0.3 area 0
interface e1/2
   ip ospf priority 255
  !! eto mismo ang Lab mo:
end

@R1:
config t
Int loopback 0
 ip add 1.1.1.1 255.255.255.255
router ospf 1
router-id 1.1.1.1
network 1.1.1.1 0.0.0.0 area 0
network 10.1.1.0 0.0.0.3 area 0
interface e1/0
   ip ospf priority 0
end

OSPF checking:
SIP: sh ip protocols
SION: sh ip ospf neighbor
SIOD: sh ip ospf database
SIRO: sh ip route ospf


ExamTanong: HOW DO OSPF RESET/REFRESH THE ELECTION:
OSPF STATES:  clear ip ospf process    +   y
Down
Init
TWo-way
Exstart
Exchange
Loading
FULL: ALL is well!, all is Ping.


@declaring Link between r3 and r4 as Point to Point:
R3:
config t
interface e1/2
   ip ospf network point-to-point
   do sh ip ospf int e1/2
R4:
config t
interface e1/2
   ip ospf network point-to-point
    do sh ip ospf int e1/2

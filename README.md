
# Network Configuration Documentation: VLANs, OSPF, DHCP and Remote Access 
# Packet Tracer lab, 10.3.16.0/21 address block

## 1. Overview
The network has two switches, three routers, one DHCP server and eight PCs. The 10.3.16.0/21 block is divided into four VLANs and three point-to-point links. Each switch connects to its router with a single cable, so inter-VLAN routing is done with router-on-a-stick (a trunk on the switch and subinterfaces on the router). OSPF (area 0) connects the three routers. The DHCP server sits behind Router1, so Router0 and Router2 relay DHCP requests with ip helper-address.

Topology (left to right):
PC0, PC1 (VLAN 10) and PC2, PC3 (VLAN 11) connect to Switch0 (SW-A / SW-1), which connects to Router0. Router0 connects to Router1, which also connects to Server0 (DHCP) above it. Router1 connects to Router2, which connects to Switch1 (SW-B / SW-2). PC4, PC5 (VLAN 20) and PC6, PC7 (VLAN 21) connect to Switch1.
In the final diagram all link lights are green.
## 2. Addressing plan
## Segment
Network
Mask
Router address

VLAN 10 (BLUE)
10.3.16.0/22
255.255.252.0
10.3.16.1 (Router0 gi0/0.10)

VLAN 11 (RED)
10.3.20.0/23
255.255.254.0
10.3.20.1 (Router0 gi0/0.11)

Router0 to Router1
10.3.22.192/30
255.255.255.252
R0 .193, R1 .194

Router1 to Router2
10.3.22.196/30
255.255.255.252
R1 .197, R2 .198

Router1 to Server0
10.3.22.200/30
255.255.255.252
R1 .201, Server .202

VLAN 21 (ORANGE)
10.3.22.0/25
255.255.255.128
10.3.22.1 (Router2 gi0/1.21)

VLAN 20 (GREEN)
10.3.22.128/26
255.255.255.192
10.3.22.129 (Router2 gi0/1.20)
Note: the diagram label for the whole block reads 255.255.248.0, which is correct for a /21.

PC addressing
The PCs were given static addresses. The table shows the plan (the DHCP pools would hand out the same range).
PC
VLAN
IP address
Mask
Gateway
PC0
10
10.3.16.2
255.255.252.0
10.3.16.1

PC1
10
10.3.16.3
255.255.252.0
10.3.16.1

PC2
11
10.3.20.2
255.255.254.0
10.3.20.1

PC3
11
10.3.20.3
255.255.254.0
10.3.20.1

PC4
20
10.3.22.130
255.255.255.192
10.3.22.129

PC5
20
10.3.22.131
255.255.255.192
10.3.22.129

PC6
21
10.3.22.2
255.255.255.128
10.3.22.1

PC7
21
10.3.22.3
255.255.255.128
10.3.22.1

## 3. Assumptions
• PCs are on fa0/1 to fa0/4 of each switch, and the uplink to the router is gi0/1.
• Router0 uses gi0/0 toward SW-A and gi0/1 toward Router1. Router1 uses gi0/0 toward Router0, gi0/1 toward Router2 and gi0/2 toward the server. Router2 uses gi0/0 toward Router1 and gi0/1 toward SW-B.
• The assignment's R-A2 is Router2.
• "10 ports per VLAN" was read as 10 ports for each VLAN on its switch.
• The server is 10.3.22.202 and Router1's server-facing port is 10.3.22.201.

## 4. Build order
The work was done from the bottom layer up so each step could be tested: PCs, then switches, then routers, then the server, then the assignment's extra requirements (security, port allocation, access control, MOTD).

## 5. Step 1: PCs
On each PC: Desktop, IP Configuration, Static, then the address, mask and gateway from the table in section 2.
6. Step 2: Switch0 (SW-A)
VLANs, access ports and trunk. Only the commands after the prompt were typed.
enable
configure terminal
no ip domain-lookup
hostname SW-A
vlan 10
name VLAN10
exit

vlan 11
name VLAN11
exit

interface range fa0/1 - 2
switchport mode access
switchport access vlan 10
exit

interface range fa0/3 - 4
switchport mode access
switchport access vlan 11
exit

interface gi0/1
switchport mode trunk
end
write memory
Verified with show vlan brief (VLAN 10 on Fa0/1 and Fa0/2, VLAN 11 on Fa0/3 and Fa0/4) and show interfaces trunk.
Troubleshooting note: the first attempt failed with "Invalid input" because the whole prompt (for example SW-1(config-vlan)#exit) was typed as part of the command. Only the text after the # should be entered.
## 7. Step 3: Switch1 (SW-B)
enable
configure terminal
no ip domain-lookup
hostname SW-B
vlan 20
name VLAN20
exit

vlan 21
name VLAN21
exit

interface range fa0/1 - 2
switchport mode access
switchport access vlan 20
exit

interface range fa0/3 - 4
switchport mode access
switchport access vlan 21
exit

interface gi0/1
switchport mode trunk
end
write memory
## 8. Step 4: Router0
Subinterfaces for VLAN 10 and 11, the link to Router1, and OSPF.
enable
configure terminal
no ip domain-lookup
hostname Router0
interface gi0/0
no shutdown
exit

interface gi0/0.10
encapsulation dot1Q 10
ip address 10.3.16.1 255.255.252.0
ip helper-address 10.3.22.202
exit

interface gi0/0.11
encapsulation dot1Q 11
ip address 10.3.20.1 255.255.254.0
ip helper-address 10.3.22.202
exit

interface gi0/1
ip address 10.3.22.193 255.255.255.252
no shutdown
exit

router ospf 1
network 10.3.16.0 0.0.3.255 area 0
network 10.3.20.0 0.0.1.255 area 0
network 10.3.22.192 0.0.0.3 area 0
end
write memory
The SW-A to Router0 link light turned green once gi0/0 was brought up with no shutdown. Router interfaces are shut down by default, so a link stays red until both ends are up.
## 9. Step 5: Router1
enable
configure terminal
no ip domain-lookup
hostname Router1
interface gi0/0
ip address 10.3.22.194 255.255.255.252
no shutdown
exit

interface gi0/1
ip address 10.3.22.197 255.255.255.252
no shutdown
exit

interface gi0/2
ip address 10.3.22.201 255.255.255.252
no shutdown
exit

router ospf 1
network 10.3.22.192 0.0.0.3 area 0
network 10.3.22.196 0.0.0.3 area 0
network 10.3.22.200 0.0.0.3 area 0
end
write memory
## 10. Step 6: Router2 (initial configuration)
enable
configure terminal
no ip domain-lookup
hostname Router2
interface gi0/0
ip address 10.3.22.198 255.255.255.252
no shutdown
exit

interface gi0/1
no shutdown
exit
interface gi0/1.20
encapsulation dot1Q 20
ip address 10.3.22.129 255.255.255.192
ip helper-address 10.3.22.202
exit

interface gi0/1.21
encapsulation dot1Q 21
ip address 10.3.22.1 255.255.255.128
ip helper-address 10.3.22.202
exit

router ospf 1
network 10.3.22.196 0.0.0.3 area 0
network 10.3.22.0 0.0.0.127 area 0
network 10.3.22.128 0.0.0.63 area 0
end
write memory
## 11. Step 7: Server0 (DHCP)
Desktop, IP Configuration: IP 10.3.22.202, mask 255.255.255.252, gateway 10.3.22.201.
Services, DHCP: service On, one pool per VLAN.
Pool
Default gateway
Start IP
Mask
Max users

VLAN10
10.3.16.1
10.3.16.2
255.255.252.0
1000

VLAN11
10.3.20.1
10.3.20.2
255.255.254.0
500

VLAN20
10.3.22.129
10.3.22.130
255.255.255.192
60

VLAN21
10.3.22.1
10.3.22.2
255.255.255.128
120
Each pool starts just after its gateway so a PC never receives the router's address. Max users is kept below the usable host count of each subnet.
## 12. Step 8: Assignment requirements
12.1 Switch changes (VLAN names by color, 10 ports per VLAN, unused ports shut down, remote access)
VLANs were renamed to match the diagram colors: VLAN 10 BLUE, VLAN 11 RED, VLAN 20 GREEN, VLAN 21 ORANGE. Each VLAN received 10 ports (its two original PC ports plus eight more). Leftover ports were shut down. A management IP on a VLAN interface allows Telnet to the switch.
SW-A:
enable
configure terminal
vlan 10
name BLUE
exit

vlan 11
name RED
exit

interface range fa0/1 - 2 , fa0/5 - 12
switchport mode access
switchport access vlan 10
exit

interface range fa0/3 - 4 , fa0/13 - 20
switchport mode access
switchport access vlan 11
exit

interface range fa0/21 - 24 , gi0/2
shutdown
exit

interface vlan 10
ip address 10.3.16.250 255.255.252.0
no shutdown
exit

ip default-gateway 10.3.16.1
enable secret testi2026
line console 0
password testi2026
login
exit

line vty 0 15
password testi2026
login
end
write memory
SW-B:
enable
configure terminal

vlan 20
name GREEN
exit

vlan 21
name ORANGE
exit

interface range fa0/1 - 2 , fa0/5 - 12
switchport mode access
switchport access vlan 20
exit

interface range fa0/3 - 4 , fa0/13 - 20
switchport mode access
switchport access vlan 21
exit

interface range fa0/21 - 24 , gi0/2
shutdown
exit

interface vlan 21
ip address 10.3.22.126 255.255.255.128
no shutdown
exit

ip default-gateway 10.3.22.1
enable secret testi2026
line console 0
password testi2026
login
exit

line vty 0 15
password testi2026
login
end
write memory
12.2 Passwords
Every connection type (console, vty, and aux on routers) and the enable mode use the password testi2026. Enable mode uses enable secret, so the password is stored encrypted.
12.3 Router0: passwords and VLAN 11 restriction
Requirement: VLANs 10, 20 and 21 reach each other, while VLAN 11 may only communicate with VLAN 10. OSPF already routes all networks, so two extended access lists on the VLAN 11 subinterface restrict it.
enable
configure terminal
enable secret testi2026
line console 0
password testi2026
login
exit

line aux 0
password testi2026
login
exit

line vty 0 15
password testi2026
login
exit

ip access-list extended VLAN11-IN
permit ip 10.3.20.0 0.0.1.255 10.3.20.0 0.0.1.255
permit ip 10.3.20.0 0.0.1.255 10.3.16.0 0.0.3.255
permit udp any eq 68 any eq 67
deny ip any any
exit

ip access-list extended TO-VLAN11
permit ip 10.3.16.0 0.0.3.255 10.3.20.0 0.0.1.255
deny ip any any
exit

interface gi0/0.11
ip access-group VLAN11-IN in
ip access-group TO-VLAN11 out
end
write memory
• VLAN11-IN filters traffic leaving VLAN 11. It allows VLAN 11 to itself, VLAN 11 to VLAN 10, and DHCP requests.
• TO-VLAN11 filters traffic entering VLAN 11. Only VLAN 10 is allowed in.
12.4 Router1: passwords
enable
configure terminal
enable secret testi2026
line console 0
password testi2026
login
exit

line aux 0
password testi2026
login
exit

line vty 0 15
password testi2026
login
end
write memory
12.5 Router2 (R-A2): SSH, passwords and MOTD
Secure access is required only on R-A2, so its vty lines accept SSH only and log in with a local user.
enable
configure terminal
hostname R-A2
ip domain-name test.local
enable secret testi2026
username admin secret testi2026
crypto key generate rsa modulus 1024
ip ssh version 2
line console 0
password testi2026
login
exit

line aux 0
password testi2026
login
exit

line vty 0 15
login local
transport input ssh
exit
banner motd #Test was a good practice of VLANs, OSPF, ACLs and SSH. Tricky but fun.#
end
write memory
SSH login: user admin, password testi2026. The MOTD banner contains short feedback about the test.
## 13. Testing and verification
Test
Command or action
Expected result
VLANs and ports
show vlan brief on both switches
10 ports per VLAN, correct names
Trunks
show interfaces trunk
Uplink trunking
Interfaces
show ip interface brief on routers
Configured interfaces up/up
OSPF
show ip ospf neighbor on Router1
Two neighbors
Routes
show ip route on Router0
O routes for VLAN 20 and 21
VLAN 10 to 20 and 21
Ping PC0 to PC4, PC0 to PC6
Success
VLAN 20 to 21
Ping PC4 to PC6
Success
VLAN 10 to 11

Ping PC0 to PC2
Success
VLAN 11 to 20 or 21
Ping PC2 to PC4
Fails (blocked by ACL)
VLAN 20 or 21 to 11
Ping PC4 to PC2
Fails (blocked by ACL)
Telnet to a switch
telnet 10.3.16.250 from PC0
Password prompt
SSH to R-A2
ssh -l admin 10.3.22.198
Login, MOTD shown
Telnet to R-A2
telnet 10.3.22.198
Refused
ACL counters
show access-lists on Router0 (in privileged mode)
Match counters increase
## 14. Problems encountered and fixes
• Invalid input on the switch: the prompt was typed along with the command. Fix: type only what follows the #.
• Empty show interfaces trunk: the trunk was set on the wrong or not yet connected port. Fix: check show interfaces status, set the trunk on the connected uplink, and bring up the router side.
• Red link light between switch and router: router interfaces are shut down by default. Fix: no shutdown on the router interface.
• show access-lists rejected: the router was in user mode (Router0>) after the console password was added. Fix: type enable and the enable password first.
## 15. Final state
All links are green, VLANs are named and populated, OSPF connects the three routers, DHCP relay is in place, every line and enable mode is protected with testi2026, SSH is enforced on R-A2, VLAN 11 is limited to VLAN 10, unused ports are shut down, and R-A2 shows a MOTD banner.
## Done by 
## Nelson Mbua Mosisah
## Cybersecurity Analyst
## Network Engineer

# Overview
**Dynamic Host Configuration Protocol(DHCP)**
Network service that automatically assigns IP addresses and other config settings like subnet mask, gateway and DNS

**How DHCP Works(The DORA Process)**
1. Discover
   - A new device (DHCP client) joins the network and doesn’t have an IP address.
   - It sends a **broadcast** message: “Is there a DHCP server that can give me an IP?”
2. Offer
   - DHCP server replies with an offer, suggesting an available IP address and related settings
3. Request
   - Client responds: "I will use that IP"
4. Acknowledge
   - The DHCP Server confirms: "Yes, that IP is yours"
   - The client now configures itself

## What DHCP Provides

A DHCP server can hand out more than just an IP address:
- **IP Address** (from a defined pool/scope)
- **Subnet Mask**
- **Default Gateway**
- **DNS Servers**
- **Lease Time** (how long the IP is valid before renewal is needed)
- Optional settings (WINS servers, NTP servers, etc.)

# DHCP Lab: Need for Configuration
- Configure DHCP on router with a IP Pool
- Assign IP addresses to Router and Switch
- Create VLAN1 on the Switch
- Confirm DHCP assignment on client-devices
- Troubleshoot as needed

Note: `#` in the code blocks are my own comments and will not appear in network device CLI
# Task 1: Configure DHCP on Router
**Network Components**:
- Router0 (1841)
- Switch0 (2960-24TT)
- PC0 and PC1
- Appropriate cables: 
	- Router->Switch = Copper Crossover
	- PC->Switch = Copper Straight-Through

**Topology**
![DHCP](img/dhcpTopology.png)

**Configuring IP Address on Router 0**
```cisco
Router(config)#interface FastEthernet0/0
Router(config-if)#ip address 192.168.1.1 255.255.255.0
Router(config-if)#no shutdown

Router(config-if)#
%LINK-5-CHANGED: Interface FastEthernet0/0, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/0, changed state to up

Router(config-if)#
```

**Configuring DHCP on Router0**
```cisco
Cisco IR829GW-LTE-GA-EK9 (revision 2.0) with 373760K/52224K bytes of memory.Installed image archive

Processor board ID FTX1942802R

FPGA version: 2.0.0

2 Serial(sync/async) interfaces
7 Gigabit Ethernet interfaces
9 terminal lines
2 Cellular interfaces
1 cisco Embedded AP (s)
DRAM configuration is 72 bits wide with parity disabled.
256K bytes of non-volatile configuration memory.
976562K bytes of ATA System Flash (Read/Write)
250000K bytes of ATA Bootstrap Flash (Read/Write)

Press RETURN to get started!


*Mar 01, 00:00:18.000: %LINK-5-CHANGED: Interface GigabitEthernet1, changed state to up
*Mar 01, 00:00:18.000: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1, changed state to up

IR800>enable
IR800#config t
Enter configuration commands, one per line.  End with CNTL/Z.
IR800(config)#hostname R0
R0(config)#ip dhcp pool TestPool
R0(dhcp-config)#network 192.168.1.0 255.255.255.0
R0(dhcp-config)#default-router 192.168.1.1
R0(dhcp-config)#dns-server 8.8.8.8
R0(dhcp-config)#
```
From the reference code block above, we created a DHCP pool, with the following settings:
**Pool Name:** TestPool
**Network:** 192.168.1.0/24
**Default Gateway:** 192.168.1.1
**DNS Server:** 8.8.8.8

**Troubleshoot Stop 1: Check DHCP Settings on Router**
```cisco
R0(config)#do show run | section dhcp
ip dhcp pool TestPool
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 8.8.8.8
```
Looks like it was configured properly.

# Task 2: Switch Configuration
**Configure VLAN1 for managing ports on switch**
**Interface**: vlan1
**IP Address**: 192.168.1.10 
**Subnet:** 255.255.255.0
```cisco
Switch#config t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#interface vlan1
Switch(config-if)#ip address 192.168.1.10 255.255.255.0
Switch(config-if)#no shutdown

Switch(config-if)#
%LINK-3-UPDOWN: Interface Vlan1, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up

Switch(config-if)#
```

**Checking Configuration**
```cisco
# Bunch of output above, scroll past interfaces
...
!
interface Vlan1 #This looks good
 ip address 192.168.1.10 255.255.255.0
!
!
!
!
line con 0
!
line vty 0 4
 login
line vty 5 15
 login
!
!
!
!

Switch(config)#
```


```cisco
Switch(config-if)#do show interfaces status
Port      Name               Status       Vlan       Duplex  Speed Type
Fa0/1                        connected    1          a-full  a-100 10/100BaseTX
Fa0/2                        connected    1          a-full  a-100 10/100BaseTX
Fa0/3                        connected    1          a-full  a-100 10/100BaseTX
Fa0/4                        notconnect   1          auto    auto  10/100BaseTX
Fa0/5                        notconnect   1          auto    auto  10/100BaseTX
...
```

**Troubleshoot Stop 2**
In the case that our DHCP Fails we need to check our interfaces. In this case we reconfigured our VLAN 1
```cisco
Switch(config-if)#interface fa0/1
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 1
Switch(config-if)#do show interfaces status
```
# Task 3: Confirm DHCP on PC

Enable `DHCP` on `PC0`. Then do the same for `PC1`
![Config IP](img/dhcpIP.png)

**Testing Network Connectivity**
```cisco
# Ping the addresses of PC0 and the switch
C:\> ping XXX.XXX.X.X
C:\>arp -a
  Internet Address      Physical Address      Type
  192.168.1.1           0040.0b8a.c101        dynamic
  192.168.1.3           0060.70b2.53ee        dynamic
  192.168.1.10          0004.9a6c.5440        dynamic
```
# Troubleshooting
In the event that your PCs cannot receive a `DHCP` address and instead receive an APIPA Address. This means that a DHCP request failed.
**Remember APIPA range**
The IP address range for APIPA is 169.254.0.1 - 169.254.255.254.
These indicate that they are routable within the network but not Internet accessible but:
- The PC sent a DHCP Discover
- NO DHCP Offer was received
- The OS self-assigned a fallback address.
From this we know:
- DHCP client is enabled
- Network interface is up
But NO DHCP server responded, meaning the issue must lie in the Router assigning DHCP

I recommended looking at the **Troubleshooting stops above**
# Final Configuration

**Logical Diagram**
![DHCP Final Config](img/dhcpFinalConfig.png)

**Physical Diagram**
![DHCP Physical](img/dhcpPhysical.png)
# Common Commands
`conf t`

`arp -d`
Clear `arp` table

`show vlan brief`
```cisco
Switch#show vlan brief
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/2, Fa0/3, Fa0/4
                                                Fa0/5, Fa0/6, Fa0/7, Fa0/8
                                                Fa0/9, Fa0/10, Fa0/11, Fa0/12
                                                Fa0/13, Fa0/14, Fa0/15, Fa0/16
                                                Fa0/17, Fa0/18, Fa0/19, Fa0/20
                                                Fa0/21, Fa0/22, Fa0/23, Fa0/24
                                                Gig0/1, Gig0/2
```

# Resources
[101Labs-Networking/DHCP-Lab5 at main · polucio/101Labs-Networking](https://github.com/polucio/101Labs-Networking/tree/main/DHCP-Lab5)
# References
Browning, Paul. _101 Labs - CompTIA Network+: Hands-on Practical Labs for the N10-008 Exam._
# Further Reading
[Basic Router Commands and Tasks](https://www.cisco.com/E-Learning/bulk/public/tac/cim/cib/using_cisco_ios_software/07_basic_commands_tasks.htm)

[Dynamic Host Configuration Protocol (DHCP) - GeeksforGeeks](https://www.geeksforgeeks.org/computer-networks/dynamic-host-configuration-protocol-dhcp/)

[Virtual LAN (VLAN) - GeeksforGeeks](https://www.geeksforgeeks.org/computer-networks/virtual-lan-vlan/)

[Guide to VLANs: What They are, How They Work, and Why They Matter](https://www.cbtnuggets.com/blog/technology/networking/what-is-a-vlan-and-how-they-work)
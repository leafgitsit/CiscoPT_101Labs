
# Overview
Cisco IOS uses a **hierarchical, mode-based CLI** where EXEC modes are for viewing and control, and CONFIG modes are for making changes.

```
            ┌────────────────────────────┐
            │        User EXEC            │
            │          Router>            │
            │  (basic monitoring only)   │
            └─────────────┬──────────────┘
                          │ enable
                          ▼
            ┌────────────────────────────┐
            │     Privileged EXEC         │
            │          Router#            │
            │  (show, copy, reload)       │
            └─────────────┬──────────────┘
                          │ configure terminal
                          │ (conf t)
                          ▼
            ┌────────────────────────────┐
            │   Global Configuration      │
            │       Router(config)#       │
            │  (device-wide settings)    │
            └─────────────┬──────────────┘
                          │
          ┌───────────────┼────────────────┐
          │               │                │
          ▼               ▼                ▼
┌────────────────┐ ┌────────────────┐ ┌────────────────┐
│ Interface Mode │ │   Line Mode    │ │ Routing Mode   │
│ Router(config- │ │ Router(config- │ │ Router(config- │
│ if)#           │ │ line)#         │ │ router)#       │
│ (ports/IPs)    │ │ (console/VTY)  │ │ (OSPF/EIGRP)   │
└────────────────┘ └────────────────┘ └────────────────┘

```

## User EXEC (`Router>`)

- First mode after login `Router>`
- **Very limited**    
- `ping`, `traceroute` (basic)

## Privileged EXEC (`Router#`)

- **Administrative mode**
- View and control the device
- Common commands:
    - `show running-config`
    - `show ip route`
    - `copy run start`
    - `reload` 

## Global Configuration (`Router(config)#`)

- **Where configuration begins**
- Affects the entire device
- Common commands:
    - `hostname`
    - `ip routing`
    - `interface`

## Sub-Configuration Modes

| Mode      | Prompt             | Purpose                            |
| --------- | ------------------ | ---------------------------------- |
| Interface | `(config-if)#`     | IP addresses, shutdown/no shutdown |
| Line      | `(config-line)#`   | Console & VTY access               |
| Router    | `(config-router)#` | Routing protocols                  |
| VLAN      | `(config-vlan)#`   | VLAN creation                      |
**VTY Access (Virtual Teletype)**
**VTY access** refers to **remote command-line access** to a network device (router/switch) using **virtual terminal lines**.
 What VTY Is Used For
- **Remote management** of Cisco devices
- Access via:
    - **SSH** (secure, preferred)
    - **Telnet** (legacy, insecure)
```python
line vty 0 15
transport input ssh
login local
# Enables secure remote access
```

## Why Cisco Designed It This Way
Cisco separates modes to:
- **Reduce mistakes** while configuring
- Keep a clear distinction between:
    - **Viewing / controlling** the device (EXEC)
    - **Changing** the device (CONFIG)
- Enforce **command scope and safety**

#  SSH Lab: Need for Configuration
- Certificates between two devices:
  - Needs Unique Name
  - Domain
  - Create SSH Keys
- Names
- IP Addresses

Note: `#` in the code blocks are my own comments and will not appear in network device CLI
# Task 1: Configure Router Hostnames
Combine 2 Cisco Routers via Crossover cables.
Note: Modern routers can use straight-throughs but the older standard is a crossover-cable to connect similar networking devices.
![ssh1](img/ssh1.png)
**Configuring Router 0**

```cisco
Router>hostname R0

Router>enable

Router#configure terminal

Enter configuration commands, one per line. End with CNTL/Z.

Router(config)#hostname R0

R0(config)#

R0#
```

**Configuring Router 1**
```cisco
Router>hostname R0

Router>enable

Router#configure terminal

Enter configuration commands, one per line. End with CNTL/Z.

Router(config)#hostname R0

R0(config)#

R0#
```

# Task 2: Configure IP Addresses
**Configuring Router 0**
```cisco
R0(config)#configure terminal
R0(config)#interface g0/0

R0(config-if)#ip address 192.168.1.1 255.255.255.0

R0(config-if)#no shutdown

R0(config-if)#

%LINK-3-UPDOWN: Interface GigabitEthernet0/0, changed state to down
```
**Configuring Router 1**
```cisco
R0(config)#configure terminal
R0(config)#interface g0/0

R0(config-if)#ip address 192.168.1.2 255.255.255.0

R0(config-if)#no shutdown

R0(config-if)#

%LINK-3-UPDOWN: Interface GigabitEthernet0/0, changed state to down
```

**Test Connectivity for R1 to R0 and vice-versa**
```cisco
R1#ping 192.168.1.2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/3/5 ms

R1#ping 192.168.1.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 0/0/1 ms
```

# Task 3: Enable SSH on R1
Configure the Following on Router1(R1):
- Domain name
- RSA Key
- SSH  timeout and Authentication
- Login Credentials

**Configuring Domain Name, Setting up RSA Key, SSH Settings**
```
R1#configure

Configuring from terminal, memory, or network [terminal]?

Enter configuration commands, one per line. End with CNTL/Z.

#Configure Domain Name
R1(config)#ip domain-name testlab.net

#Generate RSA Keys
R1(config)#crypto key generate rsa

The name for the keys will be: R1.testlab.net
Choose the size of the key modulus in the range of 360 to 4096 for your
General Purpose Keys. Choosing a key modulus greater than 512 may take
a few minutes.

How many bits in the modulus [512]: 1024
% Generating 1024 bit RSA keys, keys will be non-exportable...[OK]

R1(config)#ip ssh time-out 60
*Mar 1 0:17:13.399: %SSH-5-ENABLED: SSH 1.99 has been enabled
R1(config)#ip ssh authentication-service ?
% Unrecognized command
R1(config)#ip ssh authentication-retries ?

<0-5> Number of authentication retries

R1(config)#ip ssh authentication-retries 2
R1(config)#
```

**Setting up Login Credentials**
```
#SSH Configuration by editing vty lines 0-15
R1(config-line#line vty 0 15

R1(config-line)#transport input ssh

R1(config-line)#password cisco

R1(config-line)#login


#### Checking SSH Configuration ####
R1(config-line)#do show ip ssh

SSH Enabled - version 1.99

Authentication timeout: 60 secs; Authentication retries: 2
```

# Task 4: SSH Connectivity
**Testing Connectivity from R0 to R1**
```cisco

R0>ssh -l user 192.168.1.2

Password:

R1>
# Success
```
**Testing Connection to Telnet from R0**
Checking to see whether other connections like telnet are active. This checks against our earlier command of `transport input ssh` set for all VTY lines
```
R0>telnet 192.168.1.2

Trying 192.168.1.2 ...Open

[Connection to 192.168.1.2 closed by foreign host]
```
In the event that telnet works.
Verify that `transport input ssh` is set for all VTY lines
We can check the status of Ethernet interfaces with `show ip interfaces brief`
```
R0#show ip interface

GigabitEthernet0/0 is up, line protocol is up (connected)

Internet address is 192.168.1.1/24

Broadcast address is 255.255.255.255

Address determined by setup command

MTU is 1500 bytes

Helper address is not set

Directed broadcast forwarding is disabled

Outgoing access list is not set

Inbound access list is not set

Proxy ARP is enabled

Security level is default

Split horizon is enabled

ICMP redirects are always sent

ICMP unreachables are always sent

ICMP mask replies are never sent

IP fast switching is disabled

IP fast switching on the same interface is disabled

IP Flow switching is disabled

IP Fast switching turbo vector

IP multicast fast switching is disabled

IP multicast distributed fast switching is disabled

Router Discovery is disabled
```

# Final Config
![ssh2](img/ssh2.png)

# Common Commands
`conf t`
is short(alias) for **`configure terminal`**.
- Switches the device from **privileged EXEC mode** into **global configuration mode**
- Allows the administrator to **modify the running configuration**
- Used before configuring interfaces, routing protocols, security, etc.

|Feature|**Privileged EXEC Mode**|**Global Configuration Mode**|
|---|---|---|
|Prompt|`Router#`|`Router(config)#`|
|Entered from|User EXEC (`Router>`)|Privileged EXEC (`Router#`)|
|Command to enter|`enable`|`configure terminal` (`conf t`)|
|Purpose|Device management & verification|Change device configuration|
|Allowed actions|View configs, save files, reload|Modify running configuration|
|Example commands|`show running-config``copy run start``reload`|`hostname``interface``ip route`|
|Configuration changes|❌ No|✅ Yes|
|Sub-modes|❌ None|✅ Interface, routing, line modes|

`hostname <HOST_NAME>`
Change device hostname when inside of global configuration mode.

`interface fa0/0` or using the alias `in fa0/0`
When in global config mode, you can configure the interface of the specified ports with the above command.

`R1#show startup-config`
Displays the current, active configuration that the device is using in persistent memory NVRAM.
Interfaces and IP addresses
- VLANs
- Routing protocols
- Security settings (ACLs, SSH, VTY)

`R1#show running-config`
Displays the current, active configuration that the device is using in volatile memory, RAM(not yet saved)
Includes:
- Interfaces and IP addresses
- VLANs
- Routing protocols
- Security settings (ACLs, SSH, VTY)
```Sample Output
Building configuration...
...
interface GigabitEthernet0/0
ip address 192.168.1.2 255.255.255.0
duplex auto
speed auto

interface GigabitEthernet0/1
no ip address
duplex auto
speed auto
shutdown
```

`R1#copy running-config startup-config`
Save changes of running-config to start-up config and make the changes persistent

`R0(config-if)#ip address 192.168.1.1 255.255.255.0`
Set IP address and subnet mask for `fa0/0`

`R1(config-if)#no shutdown`
is the command used to **enable an interface** that is currently **administratively down**.
Note: Cisco **interfaces** are disabled by default, this prevents accidental traffic flow until intentionally enabled

`R1(config)#do SOME_COMMAND`
This is a **Cisco IOS shortcut** used **while you’re in configuration mode**.

 What `do show run` Does
- Runs the **`show running-config`** command
- **Without exiting** Global Configuration (or a sub-config) mode
- Displays the **current running configuration**
# Configure IP Addresses for Both Routers
```
Router>enable

Router#config t

Enter configuration commands, one per line. End with CNTL/Z.

Router(config)#hostname

% Incomplete command.

Router(config)#hostname JBC2

JBC2(config)#in fa0/0

JBC2(config-if)#ip add

% Incomplete command.

JBC2(config-if)#ip add

JBC2(config-if)#ip address 192.168.1.2 255.255.255.0

JBC2(config-if)#no shut

JBC2(config-if)#no shutdown

JBC2(config-if)#

%LINK-5-CHANGED: Interface FastEthernet0/0, changed state to up

  

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/0, changed state to up

  

JBC2(config-if)#
```

# Configure Domain Names + Certificates
```
JBC1#configure t

Enter configuration commands, one per line. End with CNTL/Z.

JBC1(config)#ip domain-nam

JBC1(config)#ip domain-name jbc.net

JBC1(config)#crypto key-

JBC1(config)#crypto key-gen

JBC1(config)#crypto key-generate r

JBC1(config)#crypto key

JBC1(config)#crypto key

JBC1(config)#crypto key ge

JBC1(config)#crypto key generate rsa

The name for the keys will be: JBC1.jbc.net

Choose the size of the key modulus in the range of 360 to 2048 for your

General Purpose Keys. Choosing a key modulus greater than 512 may take

a few minutes.

  

How many bits in the modulus [512]: 2048

% Generating 2048 bit RSA keys, keys will be non-exportable...[OK]

  

JBC1(config)#
```


 `do show run`

This is a **Cisco IOS shortcut** used **while you’re in configuration mode**.

 What `do show run` Does
- Runs the **`show running-config`** command
- **Without exiting** Global Configuration (or a sub-config) mode
- Displays the **current running configuration**

# Resources
[101Labs-Networking/SSH-Lab1 at main · polucio/101Labs-Networking](https://github.com/polucio/101Labs-Networking/tree/main/SSH-Lab1)

# References
Browning, Paul. _101 Labs - CompTIA Network+: Hands-on Practical Labs for the N10-008 Exam._

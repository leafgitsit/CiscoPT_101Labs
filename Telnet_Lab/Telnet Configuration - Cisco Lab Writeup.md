
# Overview

Application layer protocol used to connect a virtual terminal of another computer.
A user can log into another computer and access its terminal to run programs, start batch proc., and perform sysadmin tasks remotely.
On port 23.
#  Telnet Lab: Configuration
- Configure IP Address for PC and router
- Enable `telnet` on router w/ additional settings
- Monitor active `telnet` connection on router

Note: `#` in the code blocks are my own comments and will not appear in network device CLI
# Task 1: Configure IP Addresses
Network consists of :  a PC and a Cisco Router(1841)

Configure static IP addresses for both devices.
**Configure Router 0**
```cisco
Router#config t
Enter configuration commands, one per line.  End with CNTL/Z.

###NOTE Router(1841) does not have a gigabit port "gX/X". We use f0/0
Router(config)#interface f0/0
Router(config-if)#ip address 192.168.1.2 255.255.255.0
Router(config-if)#no shutdown

Router(config-if)#
%LINK-5-CHANGED: Interface FastEthernet0/0, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/0, changed state to up

Router(config-if)#end
Router#
%SYS-5-CONFIG_I: Configured from console by console

Router#
```

**Configuring PC 0**
IPv4 Address: 192.168.1.1
Subnet: 255.255.255.0

**Test Connectivity**
```cisco
Cisco Packet Tracer PC Command Line 1.0
C:\>ip address
Invalid Command.

C:\>ip address 192.168.1.2 255.255.255.0
Invalid Command.

C:\>ping 192.168.1.2

Pinging 192.168.1.2 with 32 bytes of data:

Reply from 192.168.1.2: bytes=32 time<1ms TTL=255
Reply from 192.168.1.2: bytes=32 time<1ms TTL=255
Reply from 192.168.1.2: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.1.2:
    Packets: Sent = 3, Received = 3, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

C:\>
```

# Task 2: Enable Telnet on Router0
If you have done previous labs in this series. You may remember that when setting up `ssh`
 We configured the line `subconfiguration` using `line vty 0 15`. This means we modified the remote access ports for the router and using this knowledge we can do so for `telnet`

**Configuring Router 0**
Explanation: 
- `Router(config)#line vty 0 15`: Selects and enters configuration mode for all virtual terminal(VTY) lines 0 through 15 on the router
- `Router(config-line)#transport input telnet` allows telnet connections on the selected VTY lines, permits unencrypted remote access. 
- Sets the VTY line password to cisco. This password is used only for Telnet/VTY access. Plain-text unless `service password-encryption`. Look at BONUS Below to enable
```cisco
Router(config)#
Router(config)#line vty 0 15
Router(config-line)#transport input telnet
Router(config-line)#password cisco
Router(config-line)#login
Router(config-line)#end
Router#
```

**Test Telnet for PC0 to R0**
```cisco
C:\>telnet 192.168.1.1
Trying 192.168.1.1 ...
% Connection refused by remote host
C:\>telnet 192.168.1.2
Trying 192.168.1.2 ...Open


User Access Verification

# Remember its "cisco", no quotes
Password: 
Router> #we're in
```

**BONUS**: To enable `password-encryption`.
Note: It is an obfuscated password that is not particularly strong. Another reason `telnet`
should not be utilized
```cisco
Router>enable
Router#config t

Router(config)#
Router(config)#service password-encryption
Router(config)#do show running-config | section line vty
line vty 0 4
 password 7 0822455D0A16
 login
 transport input telnet
line vty 5 15
 password 7 0822455D0A16
 login
 transport input telnet
Router(config)#
```

# Task 4: Monitor Telnet Sessions
Now switching to our Router(R0) we can monitor the active `telnet` session, in the following ways:
`show users`
- **All active management sessions**
- Identifies whether sessions are **Telnet, SSH, or console**
- Shows which **VTY line** is in use
```cisco
R0(config)# do show users
    Line       User       Host(s)              Idle       Location
*  0 con 0                idle                 00:00:00 
 196 vty 0                idle                 00:01:06 192.168.1.1

  Interface    User               Mode         Idle     Peer Address


```

Or using `show tcp brief`
```cisco
R0#show tcp brief

TCB Local Address Foreign Address (state)

1700D638060 192.168.1.2.23 192.168.1.1.1027 CLOSING
1700D63AA00 192.168.1.2.23 192.168.1.1.1028 ESTABLISHED
```
# Final Config
![telnet1](img/telnet1.png)

# Common Commands
`Router#show interfaces`

```cisco
Router#show running-config | section line vty
line vty 0 4
 password cisco
 login
 transport input telnet
line vty 5 15
 password cisco
 login
 transport input telnet
Router#
```

# Resources
[101Labs-Networking/Telnet-Lab3 at main · polucio/101Labs-Networking](https://github.com/polucio/101Labs-Networking/tree/main/Telnet-Lab3)

# References
Browning, Paul. _101 Labs - CompTIA Network+: Hands-on Practical Labs for the N10-008 Exam._

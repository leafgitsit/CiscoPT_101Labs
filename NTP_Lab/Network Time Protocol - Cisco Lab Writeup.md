
# Overview
**NTP(Network Time Protocol)**
Networking protocol used to synchronize the clocks of computer systems over a network. NTP enables accurate timekeeping by coordinating the clocks of devices within a network, ensuring that they maintain synchronized and consistent time.
UDP Port 123

#  NTP Lab: Configuration
- Configure IP Addresses
- Synchronize router with NTP Server
- Verify NTP Synchronization

Note: `#` in the code blocks are my own comments and will not appear in network device CLI
# Task 1: Configure IP Addresses
**Network consists of :  a Server and a Cisco Router(1841)**


**Configuring IP Addresses for bot devices
**Router Config R0**
```cisco
R0(config)#interface f0/0
R0(config-if)#ip address 192.168.1.1 255.255.255.0
R0(config-if)#no shutdown

R0(config-if)#
%LINK-5-CHANGED: Interface FastEthernet0/0, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/0, changed state to up

R0(config-if)#end
R0#
%SYS-5-CONFIG_I: Configured from console by console
```

**Server Config Server0
![FTP 1](img/ftp1.png)

**Verifying that NTP Service is enabled**
![FTP 2](img/ftp2.png)

**Testing Connection**
```cisco
Cisco Packet Tracer SERVER Command Line 1.0
C:\>ping 192.168.1.1

Pinging 192.168.1.1 with 32 bytes of data:

Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.1.1:
    Packets: Sent = 3, Received = 3, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

Control-C
^C
```
# Task 2: Configure Router w/ NTP Server
**Checking the current clock settings on the router**
```cisco
R0#show clock
*0:7:59.768 UTC Mon Mar 1 1993
R0#
```
Seems pretty old.

**Configuring the router with NTP server**
```cisco
R0#configure terminal
### Below used for checking usage of given command
R0(config)#ntp server ? 
  A.B.C.D  IP address of peer
R0(config)#ntp server 192.168.1.2

#### NOTICE THAT THE TIME-DATE HAS NOT CHANGED
R0(config)#do show clock
0:11:32.238 UTC Mon Mar 1 1993

### Dont forget to save
R0#write memory
```
# Task 3: Verifying NTP Synchronization
 Wait a few moments for the devices to synchronize.

After a few moments we can check the clock and verify NTP association's on the router
```cisco
R0#show clock
22:55:27.398 UTC Sun Dec 28 2025
R0#show ntp associations

address         ref clock       st   when     poll    reach  delay          offset            disp
*~192.168.1.2   127.127.1.1     1    4        16      3      0.00           1.00              0.12
 * sys.peer, # selected, + candidate, - outlyer, x falseticker, ~ configured
R0#
```

Further we can use the following command to combine both of the above commands
```cisco
R0#show ntp status
Clock is synchronized, stratum 2, reference is 192.168.1.2
nominal freq is 250.0000 Hz, actual freq is 249.9990 Hz, precision is 2**24
reference time is ECCE1051.00000301 (22:59:29.769 UTC Sun Dec 28 2025)
clock offset is 0.00 msec, root delay is 0.00  msec
root dispersion is 10.83 msec, peer dispersion is 0.12 msec.
loopfilter state is 'CTRL' (Normal Controlled Loop), drift is - 0.000001193 s/s system poll interval is 4, last update was 9 sec ago.
R0#
```



# Troubleshooting


# Common Commands
`Router#write memory`
Saves the running configuration


# Resources
[101Labs-Networking/NTP-Lab4 at main · polucio/101Labs-Networking](https://github.com/polucio/101Labs-Networking/tree/main/NTP-Lab4)
# References
Browning, Paul. _101 Labs - CompTIA Network+: Hands-on Practical Labs for the N10-008 Exam._

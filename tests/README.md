# Smart Campus Network Testing and Verification

This folder contains the testing and verification evidence used to confirm the functionality, connectivity, security, redundancy, and multi-site operation of the Smart Campus Network implemented in Cisco Packet Tracer.

Test 1. Basic IP and Network Connectivity Testing

 Objective

To verify that an end device received the correct IP configuration and could successfully communicate with another device on the campus network.

Devices Used

- Test PC
- Access switch
- Distribution switches
- DHCP server
- Wireless LAN Controller (WLC)

 Commands Used

The following commands were executed on the test PC:

ipconfig
ping 10.10.9.70

 IP Configuration Result

The `ipconfig` command produced the following configuration:

IPv4 Address:      10.10.8.141
Subnet Mask:       255.255.255.192
Default Gateway:   10.10.8.129

This confirmed that the test PC had a valid IP address, subnet mask, and default gateway.

Connectivity Test

The test PC was then used to ping the Wireless LAN Controller (WLC) at `10.10.9.70`.

C:\>ping 10.10.9.70

Pinging 10.10.9.70 with 32 bytes of data:

Reply from 10.10.9.70: bytes=32 time=76ms TTL=254
Reply from 10.10.9.70: bytes=32 time<1ms TTL=254
Reply from 10.10.9.70: bytes=32 time=14ms TTL=254
Reply from 10.10.9.70: bytes=32 time<1ms TTL=254

Ping statistics for 10.10.9.70:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)

Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 76ms, Average = 22ms

Result:
>PASS

The test PC successfully reached the WLC at `10.10.9.70` with 0% packet loss. Since the test PC (`10.10.8.141`) and WLC (`10.10.9.70`) were on different IP subnets, the successful ping also provided evidence that routing between the networks was operational.

Conclusion

The test confirmed that the test PC had valid IP configuration and that communication between different campus network subnets was functioning successfully.

Test 2. Guest Zero-Trust (GUEST-ZT) Testing

Objective

To verify that devices connected to the Guest network are prevented from accessing protected internal campus networks while still allowing the specific services required by guest users.

 Devices Used

- Guest Test PC
- Access switch
- DIST-SW1 / DIST-SW2
- DHCP/DNS Server (`10.10.9.100`)
- Internal campus network

 Guest Test PC Configuration

The Guest Test PC was placed on the Guest network and its IP configuration was checked using:

ipconfig

The following configuration was obtained:

IPv4 Address:      10.10.5.12
Subnet Mask:       255.255.255.0
Default Gateway:   10.10.5.1

This confirmed that the test PC was operating from the `10.10.5.0/24` Guest network.

 Test Performed

To determine whether a guest device could access another protected internal campus network, the Guest Test PC attempted to ping `10.10.8.129`.

The following command was used:

ping 10.10.8.129

Test Result:

C:\>ping 10.10.8.129

Pinging 10.10.8.129 with 32 bytes of data:

Reply from 10.10.5.2: Destination host unreachable.
Reply from 10.10.5.2: Destination host unreachable.
Reply from 10.10.5.2: Destination host unreachable.
Reply from 10.10.5.2: Destination host unreachable.

Ping statistics for 10.10.8.129:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)

The Guest Test PC was unable to reach `10.10.8.129`, producing 100% packet loss.

For this security test, the failed communication was the expected result because Guest users should not be able to access protected internal campus networks.

ACL Verification

The Guest Zero-Trust ACL was then checked on the distribution switch using:

show access-lists

The following `GUEST-ZT` ACL was observed:

Extended IP access list GUEST-ZT
    10 permit udp any host 10.10.9.100 eq domain
    20 permit udp any host 10.10.9.100 eq bootps
    30 permit ip 10.10.5.0 0.0.0.255 10.10.5.0 0.0.0.255
    40 deny ip 10.10.5.0 0.0.0.255 10.10.0.0 0.0.255.255 (4 match(es))
    50 permit ip any any (682 match(es))

ACL Interpretation

The ACL performs the following functions:

- Sequence 10 permits Guest devices to access the DNS service at `10.10.9.100`.
- Sequence 20 permits the required DHCP-related traffic to `10.10.9.100`.
- Sequence 30 permits communication within the Guest `10.10.5.0/24` network.
- Sequence 40 denies Guest devices from accessing the protected `10.10.0.0/16` internal campus network.
- Sequence 50 permits other traffic that was not blocked by the previous rules.

The most important evidence was:

40 deny ip 10.10.5.0 0.0.0.255 10.10.0.0 0.0.255.255 (4 match(es))

The 4 matches corresponded with the four ICMP ping attempts generated during the Guest Test PC test. This provided direct evidence that traffic from the Guest network to the protected internal network was reaching and being denied by the `GUEST-ZT` ACL.

Result:
>PASS

The Guest Test PC could not access the protected internal campus network, and the deny rule recorded four matches during the test.

Conclusion

The `GUEST-ZT` policy operated as intended. Guest traffic destined for the protected internal campus network was blocked, while the ACL retained explicit rules for required services such as DNS and DHCP. The ACL hit counter provided evidence that the failed connectivity test was caused by the configured Guest Zero-Trust security policy.

Test 3. IoT Zero-Trust (IOT-ZT) Test

Objective
Verify that IoT devices can access the authorized IoT server but cannot access protected internal networks.

 Test Device
The Guest Test PC was temporarily moved to VLAN 130 (IoT) on ASW-SERVER1 Fa0/12.

It received:

IP Address:      10.10.8.32
Subnet Mask:     255.255.255.128
Default Gateway: 10.10.8.1

Commands

ping 10.10.9.105
show access-lists

 Result
The authorized IoT server `10.10.9.105` was successfully reached with 0% packet loss.

The IOT-ZT ACL showed:

20 permit ip 10.10.8.0 0.0.0.127 host 10.10.9.105 (8 match(es))

A separate test to a protected internal network was blocked with 100% packet loss, and the deny rule recorded:

45 deny ip 10.10.8.0 0.0.0.127 10.10.0.0 0.0.255.255 (4 match(es))

 Conclusion
PASS — The IoT Zero-Trust policy allowed access to the authorized IoT server while blocking unauthorized access to protected internal networks.

Test 4. CCTV Zero-Trust (CCTV-ZT) Test

Objective
Verify that CCTV devices can access the authorized CCTV server while access to other protected internal networks is restricted.

Test Device
The test device used the CCTV VLAN 120 network (`10.10.8.192/26`).

 Commands
 
ping 10.10.9.106
show access-lists

 Result
The CCTV server `10.10.9.106` was successfully reached with 0% packet loss on the second ping test.

The CCTV-ZT ACL showed:

20 permit ip 10.10.8.192 0.0.0.63 host 10.10.9.106 (8 match(es))

45 deny ip 10.10.8.192 0.0.0.63 10.10.0.0 0.0.255.255 (4 match(es))


 Conclusion
PASS — CCTV traffic was allowed to the authorized CCTV server, while unauthorized access to protected internal networks was blocked.

Test 5. IPSec VPN and Satellite Connectivity Test

Objective
Verify secure communication between the Main Campus (`10.10.0.0/16`) and Satellite Campus (`10.20.0.0/16`).

 Commands

ping 10.20.1.141
show crypto ipsec sa

 Result
The second ping test successfully reached `10.20.1.141` with 0% packet loss.

The IPSec Security Association showed:

#pkts encaps: 319
#pkts encrypt: 319
#pkts decaps: 402
#pkts decrypt: 402
#send errors 0
#recv errors 0

 Conclusion
PASS — Main-to-satellite communication succeeded and the increasing encryption/decryption counters confirmed that traffic was passing through the IPSec VPN.

Test 6. VoIP Test

Objective
Verify that IP phones register with CME and that the voice system can establish calls.

 Command
show ephone

Result
All six configured IP phones showed `REGISTERED`.

During testing, phones `1001` and `2001` displayed:

CONNECTED
mediaActive:1

The phones included devices from both the Main Campus and Satellite Campus networks.

 Conclusion
PASS — IP phones successfully registered with CME and connected call states were observed.


Test 7. QoS Verification Test

Objective
Verify that the WAN QoS policy for voice traffic was configured and applied to the WAN interface.

 Commands
show policy-map
show policy-map interface
show access-lists 120

Result
WAN-QOS was successfully applied outbound on `GigabitEthernet0/0`.


Service-policy output: WAN-QOS

Class-map: VOICE-RTP
Strict Priority
Bandwidth 20 (%)
set ip dscp ef


ACL 120 identified UDP voice traffic:

permit udp any any range 16384 32767


However, the verification output showed:

0 packets, 0 bytes
Packets marked 0

 Conclusion
PARTIAL PASS — The QoS policy was correctly configured and applied to the WAN interface, but the captured output did not show RTP packets matching the voice class. Therefore, configuration was verified, but actual voice packet prioritization was not demonstrated by the recorded counters.

Test 8. HSRP Redundancy Test

 Objective
Verify gateway redundancy between the distribution switches.

 Command

show standby
show logging

 Result
HSRP state changes were recorded for multiple VLANs, including:

%HSRP-6-STATECHANGE: Vlan120 Grp 120 state Standby -> Active
%HSRP-6-STATECHANGE: Vlan90 Grp 90 state Standby -> Active
%HSRP-6-STATECHANGE: Vlan80 Grp 80 state Standby -> Active
%HSRP-6-STATECHANGE: Vlan100 Grp 100 state Standby -> Active

 Conclusion
PASS — HSRP Active/Standby operation was observed, confirming first-hop gateway redundancy across the configured VLANs.


Test 9. Firewall and NAT Test

Objective
Verify that the ASA firewall contained the required routing, security ACL, NAT and VPN configuration.

 Commands

show route
show access-list
show nat
show crypto ipsec sa

 Result
The firewall contained routes for the internal campus network and a default route toward `172.16.1.1`.

The outside ACL allowed Satellite-to-Main Campus VPN traffic:

permit ip 10.20.0.0 255.255.0.0 10.10.0.0 255.255.0.0

Dynamic NAT was configured as:

(inside) to (outside) source dynamic CAMPUS-NET interface

The NAT counters increased to:

translate_hits = 13
untranslate_hits = 5

The IPSec VPN also showed encrypted and decrypted traffic with no send or receive errors.

 Conclusion
PASS — Firewall routing, ACL processing, dynamic NAT and IPSec operation were verified through the configuration and traffic counters.


Test 10. Syslog and Monitoring Test

 Objective
Verify that network devices send logging information to the centralized Syslog server.

Commands:
show logging
show running-config | include logging

 Result
DIST-SW1 showed:

Logging to 10.10.9.103
101 message lines logged

The running configuration confirmed:

logging 10.10.9.103

HSRP state-change and configuration events were also visible in the log.

Conclusion
PASS — Network events were successfully being sent to the centralized Syslog server at `10.10.9.103`.

Overall Testing Conclusion:

The testing confirmed successful operation of the major  Campus Network functions, including IP connectivity, Zero-Trust ACL segmentation, CCTV and IoT restrictions, IPSec VPN connectivity, VoIP, HSRP redundancy, firewall/NAT operation, and centralized logging. The QoS policy was successfully configured and applied, although the recorded QoS counters did not demonstrate RTP packet matches during the captured test.




LLMNR (UDP/5355, Link-Local Multicast Name Resolution) is used in all Windows versions starting from Vista and allows IPv6 and IPv4 clients to resolve the names of neighboring computers without using DNS server due to broadcast requests in the local segment of L2 network. This protocol is automatically used if DNS is unavailable. So if there are DNS servers in the domain, this protocol is not needed.

#### NetBIOS Over TCP/IP Protocol
NetBIOS over TCP/IP or NBT-NS (UDP/137,138;TCP/139) is a broadcast protocol being a predecessor of LLMNR and used in the local network to publish and search for resources. By default, NetBIOS over TCP/IP support is enabled for all interfaces in all Windows versions.

Thus, these protocols enable the computers in the local network to find each other if DNS server is unavailable. They may be necessary in a workgroup, but in the domain network both of them may be disabled.

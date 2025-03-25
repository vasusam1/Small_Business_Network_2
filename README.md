# CCNA Access Control List (ACL) Lab

![Cisco Packet Tracer](https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA8D0?logo=cisco&logoColor=white)
![CCNA Level](https://img.shields.io/badge/Level-CCNA%20200--301-orange)
![License](https://img.shields.io/badge/License-MIT-green)

<div align="center">
  <img src="https://github.com/user-attachments/assets/58349729-408a-494a-8afa-3a258a0c04c7" alt="Network Topology" width="600">
</div>

## Lab Overview
This practical lab demonstrates advanced ACL configuration to:
- Block all traffic from Sales VLAN (192.168.10.0/24) to IT VLAN (192.168.20.0/24)
- Explicitly allow ICMP echo replies (ping responses) from Sales to IT
- Maintain all other network functionality

## Technical Specifications

### Network Topology
| Device             | Interface   | IP Address     | VLAN  |
|--------------------|------------|-----------------|-------|
| HQ_Router          | G0/0/0.10  | 192.168.10.1/24 | 10    |
| HQ_Router          | G0/0/0.20  | 192.168.20.1/24 | 20    |
| HQ_Router          | G0/0/0.30  | 192.168.30.1/24 | 20    |
| HQ_Router          | G0/0/1     | 203.0.113.1/30  | N/A   |
| HQ_Router          | S0/1/0     | 10.0.0.2/30     | N/A   |
| Sales_PC1 (PC0)    | NIC        | DHCP (VLAN 10)  | 10    |
| Sales_PC2 (PC1)    | NIC        | DHCP (VLAN 10)  | 10    |
| IT_PC1 (PC2)       | NIC        | DHCP (VLAN 20)  | 20    |
| IT_PC2 (PC3)       | NIC        | DHCP (VLAN 20)  | 20    |
| ServerVlan30       | NIC        | 192.168.30.2    | 30    |

### Key Configuration Snippets

#### ACL Implementation

##### HQ_Router

```cisco
! Remove existing ACL
no access-list 100

! Create extended ACL
access-list 100 permit icmp 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255 echo-reply
access-list 100 deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
access-list 100 permit ip any any

! Apply outbound on Sales VLAN
interface GigabitEthernet0/0/0.10
 ip access-group 100 out
```

#### OSPF Configuration

##### HQ_Router

```cisco
router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 192.168.30.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.3 area 0
 default-information originate
```

##### Branch_Router

```cisco
router ospf 1
 network 10.0.0.0 0.0.0.3 area 0
 network 192.168.40.0 0.0.0.255 area 0
```

#### DHCP Configuration

##### HQ_Router

```cisco
ip dhcp pool Sales
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
ip dhcp pool IT
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
```

#### Testing Procedures

```cisco
# Check ACL hits
show access-list 100

# Test ping from IT to Sales (should succeed)
IT_PC> ping 192.168.10.1

# Test ping from Sales to IT (should fail)
Sales_PC> ping 192.168.20.1

# Verify NAT translations
show ip nat translations
```

### Expected Results

| Test Case	                  | Expected Result |
|-----------------------------|-----------------|
| IT → Sales Ping             |	✅ Success      |
| IT → Server Ping            |	✅ Success      |
| IT → Internet Ping          |	✅ Success      |
| IT → Branch_router Ping     |	✅ Success      |
| Sales → Server Ping         |	✅ Success      |
| Sales → IT Ping             |	❌ Blocked      |
| Sales → IT Telnet/SSH       |	❌ Blocked      |
| Sales → Internet            |	✅ Success      |
| Sales → Branch_router Ping  |	✅ Success      |
| Server → Internet           |	✅ Success      |
| Server → Bransh_router Ping |	✅ Success      |

### Getting Started

1. Open the topology in Packet Tracer 8.0+
2. Load configurations from the /configs folder
3. Test connectivity using the verification commands
4. Experiment by modifying the ACL rules


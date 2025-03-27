# Enterprise Network Infrastructure

## Overview

This project implements a comprehensive enterprise network infrastructure with redundant connectivity, high availability, and essential network services. It includes core routing and switching, multiple VLANs for different organizational departments, DHCP services, DNS infrastructure, and web hosting capabilities.

## Key Technologies & Features

- **Cisco IOS Routing and Switching**: Used for core network infrastructure
  - **OSPF Routing**: Dynamic routing protocol for internal route distribution
  - **HSRP (Hot Standby Router Protocol)**: Gateway redundancy across all VLANs
  - **SLA (Service Level Agreement)**: Monitoring of WAN links for automated failover
  - **Layer 3 Switch Inter-VLAN Routing**: Efficient routing between VLANs
  - **Floating Static Routes**: Backup routes with different administrative distances
  - **EtherChannel**: Link aggregation for increased bandwidth and redundancy

- **NAT Implementation**:
  - **Dynamic NAT with PAT on R1**: Overload configuration for client outbound traffic
  - **Static NAT for Servers**: One-to-one mapping for publicly accessible servers
  
- **DHCP Services**:
  - **Split-Scope Configuration**: Redundant DHCP services across R1 and R2
  - **Exclusions and Reservations**: Properly configured address pools

- **Docker-based Server Infrastructure**:
  - **BIND DNS Server**: Both authoritative and caching DNS servers
  - **Apache Web Server**: Enterprise web hosting

## Network Architecture

### Core Components

- **Border Routers**:
  - R1 and R2 provide redundant internet connectivity
  - NAT services for internal-to-external communication
  - OSPF routing to distribute route information

- **Distribution Switches**:
  - DSW1 and DSW2 form the core switching infrastructure
  - HSRP configured for gateway redundancy
  - Inter-VLAN routing capabilities

- **Access Switches**:
  - SW1, SW2, SW3, and SW4 provide end-user connectivity
  - Support for multiple VLANs

### VLAN Configuration

Multiple VLANs are configured for different departments and functions. For detailed IP addressing information, see the [IP Addressing](IP.md) file.

### Server Infrastructure

| Server                  | Private IP      | Function |
|-------------------------|----------------|----------|
| Web Server              | 192.168.1.212  | Enterprise website hosting |
| DNS Authoritative Server| 192.168.1.213  | DNS zone hosting for p2-2.sitict.net |
| DNS Cache Server        | 192.168.1.214  | DNS resolution for internal clients |

## Configuration Details

### Router Configurations

Router configurations implement:

- OSPF routing for internal network connectivity
- Static NAT for server publishing
- DHCP services for client addressing
- Redundant default routes for WAN failover

Detailed configurations can be found in:

- [R1 Configuration](config/R1/r1_running_config.txt)
- [R2 Configuration](config/R2/r2_running_config.txt)

### Distribution Switch Configurations

Core switches implement:

- Inter-VLAN routing
- HSRP for gateway redundancy
- OSPF routing protocol participation
- EtherChannel for link aggregation

Detailed configurations can be found in:

- [DSW1 Configuration](config/core/DSW1/dsw1_running_config.txt)
- [DSW2 Configuration](config/core/DSW2/dsw2_running_config.txt)

### Access Switch Configurations

Access switches provide end-device connectivity with:

- VLAN assignments
- STP optimization
- Port security features

Detailed configurations can be found in:

- [SW1 Configuration](config/access/SW1/sw1_running_config.txt)
- [SW2 Configuration](config/access/SW2/sw2_running_config.txt)
- [SW3 Configuration](config/access/SW3/sw3_running_config.txt)
- [SW4 Configuration](config/access/SW4/sw4_running_config.txt)

### Server Infrastructure Deployment

The server infrastructure utilizes Docker containers for ease of deployment and configuration. The following components are containerized:

#### Network Setup

```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl enable --now docker

# Create macvlan network for server VLAN
sudo docker network create -d macvlan --subnet=192.168.1.208/29 --gateway=192.168.1.211 -o parent=enp0s5 vlan80-net 
```

#### Web Server Deployment

```bash
docker pull httpd
sudo docker stop webserver
sudo docker rm webserver
sudo docker run -d -p 192.168.1.212:80:80 --name webserver httpd:latest
```

#### DNS Server Deployment

```bash
# Authoritative DNS Server
docker pull ubuntu/bind9
sudo docker stop authdns
sudo docker rm authdns
sudo docker run -d --name authdns -p 192.168.1.213:53:53 -p 192.168.1.213:53:53/udp -v /home/parallels/srv/authdns/bind:/etc/bind ubuntu/bind9

# Caching DNS Server
sudo docker stop cachedns
sudo docker rm cachedns
sudo docker run -d --name cachedns -p 192.168.1.214:53:53 -p 192.168.1.214:53:53/udp -v /home/parallels/srv/cachedns/bind:/etc/bind ubuntu/bind9
```

Configurations:

- [DNS Server Configurations](srv/authdns/bind)
- [Web Server Content](srv/webserver/htdocs)

## IP Addressing Scheme

Complete IP addressing details including VLAN subnets, router interconnections, and public IP assignments are documented in the [IP Addressing](IP.md) file.

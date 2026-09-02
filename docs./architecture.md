# Lab Architecture

## Purpose

This lab provides an isolated environment for practicing virtualization,
network security, Linux and Windows administration, vulnerability testing,
security monitoring, and incident response.

## Physical Platform

- Dell OptiPlex 7090 SFF
- Intel Core i7 processor
- 32 GB RAM
- Proxmox VE hypervisor
- Single physical Ethernet connection to the home network

## Network Architecture

```mermaid
flowchart TD
    R["Home Router and Internet"] --> B0["vmbr0: Upstream Bridge"]
    B0 --> P["Proxmox Management"]
    B0 --> W["OPNsense WAN"]
    W --> F["OPNsense Firewall"]
    F --> L["OPNsense LAN"]
    L --> B1["vmbr1: Isolated Lab Bridge"]
    B1 --> U["Ubuntu Server"]
    B1 --> K["Kali and Future Targets"]

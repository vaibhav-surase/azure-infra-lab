# Azure Infrastructure Deployment Lab

A hands-on lab where I provisioned core Azure networking and compute infrastructure from scratch — a Virtual Network with isolated subnets, a Windows and a Linux VM secured with Network Security Group rules, remote access via RDP and SSH, and resource monitoring with Azure Monitor.

## Tech Stack

- **Azure Virtual Machines** — Windows Server 2022 Datacenter, Ubuntu 24.04 LTS (Standard B1s, free-tier eligible)
- **Azure Virtual Network (VNet)** — custom address space with two subnets
- **Network Security Groups (NSG)** — IP-restricted inbound rules for RDP and SSH
- **Azure Monitor** — VM Insights, metrics, and alert rules

## Architecture

<img width="1360" height="1120" alt="architecture-diagram" src="https://github.com/user-attachments/assets/010280f1-f3d9-442e-8dbc-a40e396f0434" />


A single VNet (`10.0.0.0/16`) hosts two subnets. Each subnet has its own NSG that only allows inbound traffic from a specific IP address on the relevant management port (3389 for RDP, 22 for SSH), and its own VM. Azure Monitor collects metrics and health signals from both VMs.

## Steps Performed

### 1. Resource Group
Created a dedicated resource group (`rg-azure-infra-lab`) to contain and manage all lab resources as a single unit, making cleanup simple at the end.

<img width="1362" height="485" alt="resources" src="https://github.com/user-attachments/assets/15fa933d-0dab-412a-826e-0286deb3eeb0" />


### 2. Virtual Network & Subnets
Created a VNet (`10.0.0.0/16`) and split it into two subnets:
- `subnet-windows` → `10.0.1.0/24`
- `subnet-linux` → `10.0.2.0/24`

<img width="870" height="376" alt="vnet" src="https://github.com/user-attachments/assets/d58855b3-66cc-4fce-bf3f-c7dde5cf649f" />
<img width="693" height="360" alt="subnet" src="https://github.com/user-attachments/assets/10db9685-d9b5-4bcf-a8c3-70a399be8b05" />



### 3. Network Security Groups
Created two NSGs, one per subnet, each with a single inbound rule scoped to my own IP address only (instead of leaving the management port open to the internet):
- `nsg-windows` → allow TCP 3389 (RDP) from My IP
- `nsg-linux` → allow TCP 22 (SSH) from My IP

<img width="873" height="367" alt="NSG" src="https://github.com/user-attachments/assets/d45cf9a3-6d59-4550-b780-b780b7dc7cf9" />

### 4. Windows VM + RDP
Deployed a Windows Server 2022 VM (`Standard B1s`) into `subnet-windows`, attached to `nsg-windows`, and connected to it remotely using RDP.

<img width="777" height="634" alt="RDP" src="https://github.com/user-attachments/assets/31c1f622-e135-4be1-bd8c-3c29c7ba579c" />


### 5. Linux VM + SSH
Deployed an Ubuntu 22.04 LTS VM (`Standard B1s`) into `subnet-linux`, attached to `nsg-linux`, authenticated with an SSH key pair, and connected to it remotely via SSH.

<img width="841" height="550" alt="SSH" src="https://github.com/user-attachments/assets/89a11494-2a7f-46f2-ad37-4021c7235a50" />


### 6. Azure Monitor
Enabled VM Insights (which required turning on a system-assigned managed identity on the VM) and reviewed live CPU/network metrics and VM availability for both machines.

<img width="1358" height="591" alt="CPU monitoring" src="https://github.com/user-attachments/assets/7ebf7fab-3386-46d5-981d-f8b7e362b386" />
<img width="1357" height="592" alt="network monitoring" src="https://github.com/user-attachments/assets/f51875f3-1bc0-45a0-8e84-4e8f38e16c6a" />


## Key Learnings

- **Regional resource constraints on free-tier subscriptions** — some regions restrict VM creation for new/trial accounts even when other resource types deploy fine there; resources had to be recreated in a supported region (Central India).
- **NSG priority vs. port number** — priority is just the rule's evaluation order within an NSG and must be unique *per NSG*, not a value tied to the port being opened.
- **Managed identity is a prerequisite for VM Insights** — the Azure Monitor Agent-based Insights experience needs a system-assigned managed identity enabled on the VM before it can be turned on.
- **Principle of least exposure** — scoping NSG rules to "My IP" instead of "Any" keeps RDP/SSH from being exposed to the open internet, which is the standard target for automated brute-force scans.

## Cleanup

After documenting the lab, VMs were stopped/deallocated and the resource group was deleted to avoid incurring any charges beyond the free tier:

```bash
az group delete --name rg-azure-infra-lab --yes --no-wait
```

## Repo Structure

```
azure-infra-lab/
├── README.md
├── screenshots/
│   ├── 01-resource-group.png
│   ├── 02-vnet.png
│   ├── 03-nsg-rules.png
│   ├── 04-windows-vm-rdp.png
│   ├── 05-linux-vm-ssh.png
│   └── 06-azure-monitor.png
└── docs/
    ├── architecture-diagram.png
    └── architecture-diagram.svg
```

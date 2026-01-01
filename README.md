# Hybrid Cloud Networking: Secure Site-to-Site VPN Architecture

![VPN Banner](/images/vpn_banner.png)

## Introduction

In this project, I designed and deployed a **Hybrid Cloud** network simulation. The primary objective was to securely connect a simulated On-Premises datacenter to an Azure Virtual Network (VNet) over the public internet.

Using **Azure VPN Gateways** and **IPsec/IKEv2** protocols, I established a secure, encrypted tunnel that allows resources in both networks to communicate using private IP addresses. This architecture mimics the standard connectivity model used by enterprises to extend their local infrastructure into the cloud without dedicated private circuits.

## Objectives

-   **Simulate Hybrid Environment:** Create two distinct Virtual Networks (One representing "On-Prem", one representing "Azure").
-   **Deploy Gateway Infrastructure:** Provision VPN Gateways and Local Network Gateways.
-   **Configure Secure Tunneling:** Establish a Site-to-Site (S2S) IPsec connection with a Shared Key.
-   **Validate Connectivity:** Prove that a VM in the "On-Prem" network can SSH into an "Azure" VM using its private IP, bypassing the public internet.

## Tech Stack

![Azure VPN Gateway](https://img.shields.io/badge/Azure_VPN_Gateway-0078D4?style=for-the-badge&logo=microsoft-azure&logoColor=white)
![Virtual Network](https://img.shields.io/badge/Virtual_Network-0078D4?style=for-the-badge&logo=microsoft-azure&logoColor=white)
![IPsec](https://img.shields.io/badge/IPsec_IKEv2-0078D4?style=for-the-badge&logo=microsoft-azure&logoColor=white)
![Azure CLI](https://img.shields.io/badge/Azure_CLI-0078D4?style=for-the-badge&logo=microsoft-azure&logoColor=white)

### Technologies Used
* **Azure VPN Gateway:** VpnGw1 SKU for encrypted tunneling.
* **IPsec/IKEv2:** Industry-standard protocol for securing the Site-to-Site connection.
* **Local Network Gateway:** Configuration object representing the on-premise VPN device.
* **Azure CLI:** Used for scripting the deployment of Resource Groups and VNets.
* **Virtual Networks (VNet):** Configuring address spaces (`10.1.0.0/16`, `10.2.0.0/16`) and subnets.
* **Linux (Ubuntu):** Used as jump hosts to verify private connectivity via SSH.

## Architecture

![VPN Architecture Diagram](/images/vpn_architecture.png)

The solution consists of two isolated networks connected via the public internet:
1.  **"On-Prem" VNet:** `10.1.0.0/16` (Simulating the local datacenter).
2.  **"Azure" VNet:** `10.2.0.0/16` (The cloud environment).
3.  **The Tunnel:** An IPsec IKEv2 connection secured with a pre-shared key.

## Implementation Steps

### Phase 1: Network & VM Provisioning

I used the Azure CLI to create two Resource Groups and their respective VNets.
* **On-Prem VNet:** `10.1.0.0/16` with a dedicated `GatewaySubnet`.
* **Azure VNet:** `10.2.0.0/16` with a dedicated `GatewaySubnet`.
* **Compute:** Deployed Ubuntu VMs (`onprem-vm` and `azure-vm`) into the default subnets of each network.

![Infrastructure Setup](/images/vnet_setup.png)
*Figure 1: Validating the creation of VMs in separate, isolated networks.*

### Phase 2: Gateway Deployment

This was the most time-intensive phase, as VPN Gateways perform dynamic routing updates.
1.  **Public IPs:** Allocated Dynamic Public IPs for both gateways.
2.  **VPN Gateways:** Deployed `VpnGw1` SKU gateways into the `GatewaySubnet` of both networks.

![Gateway Creation](/images/gateway_succeeded.png)
*Figure 2: Successful provisioning of the Virtual Network Gateways.*

### Phase 3: Local Network Gateways & Connections

To link the networks, each side needs to know about the other's Public IP and Address Space.
1.  **Local Network Gateway (LNG):** I created an LNG object in the "Azure" RG that pointed to the "On-Prem" Gateway's Public IP.
2.  **Connection Object:** I established the connection using the Shared Key `MySecretKey123`.


### Phase 4: Connectivity Verification

To prove the tunnel was functional, I performed a private connectivity test.

1.  SSH'd into the **On-Prem VM** using its Public IP.
2.  From inside that session, I attempted to SSH into the **Azure VM** using its **Private IP** (`10.2.0.4`).
3.  **Result:** Successful connection. This proves traffic flowed through the VPN tunnel, as private IPs are not routable over the open internet.

![SSH Verification](/images/ssh_proof.png)
*Figure 4: Successful SSH connection from On-Prem to Azure using Private IP.*

## Key Results

* **Secure Hybrid Connectivity:** Successfully established a stable, encrypted Site-to-Site VPN tunnel between an Azure VNet and a simulated On-Premises datacenter.
* **Private Network Extension:** Enabled secure resource communication using **Private IP addresses** only, completely bypassing public internet exposure for backend traffic.
* **Infrastructure Consistency:** Achieved a repeatable deployment process by using **Azure CLI** commands for network and gateway provisioning instead of manual Portal clicks.
* **Validated Isolation:** Verified network security by confirming that cross-network SSH traffic was correctly routed through the VPN tunnel (IPsec) rather than the open internet.

---

## Repository Contents

* `/images` - Screenshots of the configuration and verification.
* `/docs` - CLI commands used for deployment.

## About This Project

**Role:**
Cloud Security Engineer

**Skills Demonstrated:**
Hybrid Cloud Architecture, Site-to-Site VPN, Azure Networking, IPsec/IKEv2, Azure CLI.
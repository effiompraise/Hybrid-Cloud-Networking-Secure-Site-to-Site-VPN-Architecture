# Azure CLI Deployment Commands

This document contains the Azure CLI commands used to provision the Site-to-Site VPN architecture. 

**Prerequisites:**
* Azure CLI installed.
* Logged in via `az login`.

## 1. Create Resource Groups
We create separate resource groups to logically isolate the "On-Premises" simulation from the "Azure" cloud environment.

```bash
# Create Resource Group for On-Premises Simulation
az group create --name OnPrem-RG --location eastus

# Create Resource Group for Azure Cloud Simulation
az group create --name Azure-RG --location westeurope

```

## 2. Create Virtual Networks (VNets)

Provision the two distinct networks with non-overlapping IP address spaces.

```bash
# Create On-Prem VNet (10.1.0.0/16)
az network vnet create \
  --name OnPrem-VNet \
  --resource-group OnPrem-RG \
  --address-prefix 10.1.0.0/16 \
  --subnet-name Default \
  --subnet-prefix 10.1.0.0/24

# Create Azure VNet (10.2.0.0/16)
az network vnet create \
  --name Azure-VNet \
  --resource-group Azure-RG \
  --address-prefix 10.2.0.0/16 \
  --subnet-name Default \
  --subnet-prefix 10.2.0.0/24

```

## 3. Configure Gateway Subnets

VPN Gateways require a dedicated subnet named specificially `GatewaySubnet`.

```bash
# Add GatewaySubnet to On-Prem VNet
az network vnet subnet create \
  --vnet-name OnPrem-VNet \
  --resource-group OnPrem-RG \
  --name GatewaySubnet \
  --address-prefix 10.1.255.0/27

# Add GatewaySubnet to Azure VNet
az network vnet subnet create \
  --vnet-name Azure-VNet \
  --resource-group Azure-RG \
  --name GatewaySubnet \
  --address-prefix 10.2.255.0/27

```

## 4. Create Public IP Addresses

The VPN Gateways need public-facing IP addresses to communicate over the internet.

```bash
# Public IP for On-Prem Gateway
az network public-ip create \
  --name OnPrem-GW-PIP \
  --resource-group OnPrem-RG \
  --allocation-method Dynamic

# Public IP for Azure Gateway
az network public-ip create \
  --name Azure-GW-PIP \
  --resource-group Azure-RG \
  --allocation-method Dynamic

```

## 5. Provision VPN Gateways

*Note: This step can take 45+ minutes to complete.*

```bash
# Create On-Prem VPN Gateway
az network vnet-gateway create \
  --name OnPrem-Gateway \
  --resource-group OnPrem-RG \
  --public-ip-address OnPrem-GW-PIP \
  --vnet OnPrem-VNet \
  --gateway-type Vpn \
  --vpn-type RouteBased \
  --sku VpnGw1 \
  --no-wait

# Create Azure VPN Gateway
az network vnet-gateway create \
  --name Azure-Gateway \
  --resource-group Azure-RG \
  --public-ip-address Azure-GW-PIP \
  --vnet Azure-VNet \
  --gateway-type Vpn \
  --vpn-type RouteBased \
  --sku VpnGw1 \
  --no-wait

```

## 6. Create Local Network Gateways (LNG)

Each side needs to know the Public IP and Address Space of the *other* side.

*(Note: You must retrieve the actual Public IPs assigned in Step 4 before running this. Replace `X.X.X.X` and `Y.Y.Y.Y` below.)*

```bash
# On Azure side: Define the "On-Prem" network details
az network local-gateway create \
  --resource-group Azure-RG \
  --gateway-ip-address <Actual_OnPrem_Public_IP> \
  --name OnPrem-LNG \
  --local-address-prefixes 10.1.0.0/16

# On On-Prem side: Define the "Azure" network details
az network local-gateway create \
  --resource-group OnPrem-RG \
  --gateway-ip-address <Actual_Azure_Public_IP> \
  --name Azure-LNG \
  --local-address-prefixes 10.2.0.0/16

```

## 7. Establish Connections

Create the IPsec tunnel using a pre-shared key.

```bash
# Connection from Azure -> On-Prem
az network vpn-connection create \
  --name AzureToOnPrem \
  --resource-group Azure-RG \
  --vnet-gateway1 Azure-Gateway \
  --local-gateway2 OnPrem-LNG \
  --shared-key "MySecretKey123"

# Connection from On-Prem -> Azure
az network vpn-connection create \
  --name OnPremToAzure \
  --resource-group OnPrem-RG \
  --vnet-gateway1 OnPrem-Gateway \
  --local-gateway2 Azure-LNG \
  --shared-key "MySecretKey123"

```
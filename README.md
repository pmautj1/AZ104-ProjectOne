AZ104-ProjectOne
AZ-104 aligned project demonstrating private, identity-based access to Azure Blob Storage using Private Endpoints, RBAC, and Azure Bastion

# Project Overview
This project demonstrates a method of securely storing and accessing data in Azure without exposing resources to the public internet.
Access is restricted to trusted workloads inside a virtual network and authenticated using Azure Managed Identity so there is no need for storage keys or SAS tokens. 
The aim of this project was to emulate real-world enterprise patterns used for handling sensitive data and maintaining compliant environments.

# Objectives
- Disable all public access to Azure Blob Storage
- Restrict network access using a Private Endpoint
- Enforce data-plane RBAC using Azure AD
- Authenticate workloads using Managed Identity
- Validate access from a private VM using Azure Bastion
- Verify private DNS resolution and identity-based access

# Architecture
Essentially, access is set up as follows:
  VM (Managed Identity)
     ↓
  Private DNS Resolution
     ↓
  Private Endpoint (10.x.x.x)
     ↓
  Azure Blob Storage

# Implementation
## 1. Storage Account Creation
I created a storage account that would later be restricted to private network access once the Private Endpoint and DNS configuration were in place.

### <img width="1360" height="1033" alt="image" src="https://github.com/user-attachments/assets/6406833d-3260-4a89-95dd-634f605b94ef" />

## 2. Blob Container Creation
The next step is to configure a container with private access. Below, I created 'azprojectonec1'. No unauthenticated or anonymous access to stored data is allowed. I uploaded a sample object to the container to validate private blob access.

### <img width="418" height="506" alt="Step 2" src="https://github.com/user-attachments/assets/ae368dde-6708-490b-906b-2000252a0dc0" />
### <img width="1409" height="692" alt="Step 6" src="https://github.com/user-attachments/assets/9a6add3d-fa52-4556-9135-8ef743e93c81" />

## 3. Virtual Network and Subnet Creation
   Using the 10.0.0.0/16 address space, i created the 'azprojectonevnet' virtual network with a 'workload-subnet' (10.0.1.0/24), 'private-endpoint-subnet' (10.0.2.0/24), and 'AzureBastionSubnet' (10.0.3.0/26).
   - 'workload-subnet' hosts compute resources that require private access to storage while keeping application traffic isolated
   - 'private-endpoint-subnet' hosts private endpoints for Private Link connections. Here, it is created as a dedicated subnet for Private Endpoints with network policies disabled, as required by Azure.
   - 'AzureBastionSubnet' exclusively hosts the Azure Bastion service which is used to securely access the private VM without assigning public IPs or opening inbound ports.

### <img width="1418" height="934" alt="image" src="https://github.com/user-attachments/assets/51eb12d8-918d-484b-8c1c-ebd2c7d65e2d" />
### <img width="1410" height="529" alt="image" src="https://github.com/user-attachments/assets/d6af2c9a-6f0a-4bb4-80bb-98579703ec26" />

## 4. Virtual Machine Deployment
This is being created to simulate an internal application or automation workload that would require secure access to storage. In real-world environments, backend services would access storage from inside a virtual network away from public exposure.

### <img width="885" height="973" alt="Step 11" src="https://github.com/user-attachments/assets/03350246-50d6-4493-9be5-dc6a7bd81093" />
### <img width="978" height="960" alt="Step 11 2" src="https://github.com/user-attachments/assets/e4f6abda-8383-4410-ab3b-2b751030fea1" />
### <img width="1413" height="1225" alt="image" src="https://github.com/user-attachments/assets/40bd5084-9bb4-4c03-85a8-90d13a406a69" />

## 5. Enabling Managed Identity on the VM
This step is important as it allows the VM to authenticate to Azure services without any credentials or secrets. That eliminates secret rotation overhead and the need to store keys or passwords.

### <img width="1406" height="561" alt="image" src="https://github.com/user-attachments/assets/e1499430-a4b2-45cd-a907-0b9b9d2ab6ee" />

## 6. Assigning Data-Plane RBAC Permissions
Here I'm granting the VM permission to access blob data using Azure AD authentication on the **data plane**. Management-plane roles like Contributor or Owner do not grant blob access. The 'Storage Blob Data Contributor' role will allow for read, write and delete access to Azure Storage blob containers and data

### <img width="2546" height="734" alt="image" src="https://github.com/user-attachments/assets/cce8efb2-249b-49aa-b448-0bbf2b57ee20" />
### <img width="962" height="591" alt="image" src="https://github.com/user-attachments/assets/7445eae9-8637-4567-ad1c-b72f37f48806" />

## 7. Creating a Private Endpoint
A private endpoint is being created for the Blob service, assigning the storage account a private IP within the virtual network and eliminating public internet access. 

### <img width="715" height="869" alt="Step 13" src="https://github.com/user-attachments/assets/2ad69f74-91b9-43e0-9ebc-a864a49e7d0f" />

## 8. DNS Zone Integration
Private DNS ensured that standard Azure service hostnames resolve to private IPs in the virtual network. Once the Private Endpoint was integrated with the privatelink.blob.core.windows.net Private DNS zone, the storage account automatically appeared in the DNS record set. 

### <img width="1412" height="881" alt="Step 14" src="https://github.com/user-attachments/assets/d5c1827e-3e89-4fc7-9bae-9de06413ae42" />

On the VM, I used 'nslookup' to confirm access to the storage account. Another command pinging Google shows that the VM has no public internet access. 

```
nslookup azprojectonestor.blob.core.windows.net
```

### <img width="1424" height="1301" alt="step 18" src="https://github.com/user-attachments/assets/d4350665-f0f0-40d5-a71c-dc385b71507b" />

## 9. Securing Administrative Access with Azure Bastion
I created an Azure Bastion in its own subnet, 'AzureBastionSubnet', to enable remote connection with no public IPs, no inbound ports, and secure HTTPS access.

### <img width="700" height="636" alt="step 17" src="https://github.com/user-attachments/assets/24404987-1637-4dc6-aa1d-b3ac50b46e9b" />

## 10. Validating Access with Managed Identity
On the VM, I confirmed the blob access works only from inside the private network with the following commands:
```
az login --identity

az storage blob list `
  --account-name azprojectonestor `
  --container-name projectonec1 `
  --auth-mode login `
  --output table
```

### <img width="1425" height="1135" alt="step 19" src="https://github.com/user-attachments/assets/cade3368-bc2b-43f3-a6f5-9f5bd0c939f2" />
### <img width="1424" height="1133" alt="step 20" src="https://github.com/user-attachments/assets/790a99e4-f681-450e-a5f2-b736f299e98d" />


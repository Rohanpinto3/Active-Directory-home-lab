# Active Directory Home Lab (VirtualBox + Windows Server + Windows 10)

## Objective
This project was built to give me practical experience setting up Active Directory and Windows networking in a home lab. My goal was to understand how services like DNS, DHCP, and NAT work together, learn how to join a client to a domain, and get comfortable creating and managing users and groups in Active Directory. 



---

## Skills Learned
- Configuring **Active Directory Domain Services (AD DS)**
- Setting up **DNS, DHCP, and NAT routing** in Windows Server
- Automating **bulk user creation at scale** (1,000+ users with PowerShell ISE)
- Understanding **Windows networking** (IP addressing, internal vs external networks)
- Creating and managing **domain users, groups, and organizational units (OUs)**
- Practicing **identity and access management (IAM)** in a safe lab environment
- Using **snapshots and resets** for testing and troubleshooting
- Documenting and presenting a technical project for a portfolio

---

## Tools Used
- **Oracle VirtualBox** (virtualization platform)
- **Windows Server 2019** (Domain Controller)
- **Windows 10** (domain client machine)
- **Active Directory Users and Computers (ADUC)**
- **DHCP, DNS, and Routing and Remote Access Services (RRAS)**
- **PowerShell ISE** (for bulk user creation)

---

## Table of Contents
1. [Project Goals](#project-goals)  
2. [Lab Environment](#lab-environment)  
3. [Network Plan](#network-plan)  
4. [Create the Domain Controller VM](#create-the-domain-controller-vm)  
5. [Configure the DC: Name, NICs, Static IP](#configure-the-dc-name-nics-static-ip)  
6. [Install AD DS and Create the Forest](#install-ad-ds-and-create-the-forest)  
7. [Set Up NAT with Routing and Remote Access](#set-up-nat-with-routing-and-remote-access)  
8. [Install and Configure DHCP](#install-and-configure-dhcp)  
9. [Bulk Add 1,000 Users with PowerShell ISE](#bulk-add-1000-users-with-powershell-ise)  
10. [Create the Client VM](#create-the-client-vm)  
11. [Join the Client to the Domain](#join-the-client-to-the-domain)  
12. [Verification Checklist](#verification-checklist)    
13. [Snapshots & Reset Tips](#snapshots--reset-tips)

---

 ## Project Goals
- **One Windows Server Domain Controller** providing:
  - Active Directory Domain Services (AD DS)
  - DNS
  - DHCP
  - NAT routing (so the client can reach the internet through the DC)  
- **One Windows 10 Client** joined to the domain  
- An **internal, isolated lab network**  

<img width="1169" height="688" alt="Topology" src="https://github.com/user-attachments/assets/ced4b73b-6f9e-48bf-bdff-cd6cdb4ce76f" />

---

## Lab Environment

- Oracle VirtualBox installed
- Windows Server 2019 ISO
- Windows 10 ISO
- At least 12 GB RAM available on your host (suggestion: DC 4–6 GB, Client 4 GB)
- Sufficient disk space (about 40–60 GB total)
  

---

## Network Plan
- Lab subnet: `172.16.0.0/24`  
- Domain Controller:  
  - NIC 1 → NAT (internet access for the DC)  
  - NIC 2 → Internal Network (static IP `172.16.0.1`, DNS = itself)  
- Client:  
  - NIC → Internal Network (gets IP from DHCP on the DC)  

---

## Create the Domain Controller VM
1. Create a new VM for Windows Server in VirtualBox.  
2. Assign memory (4–6 GB), 2 CPUs, and a 40–60 GB disk.  
3. Add two network adapters:  
   - Adapter 1 → NAT  
   - Adapter 2 → Internal Network  
4. Attach the Windows Server ISO and complete the installation.
   
### Walkthrough Video

https://github.com/user-attachments/assets/f90bceed-ddbc-4183-97a0-cd4ffc09117c

<p align="center">
  <img src="https://github.com/user-attachments/assets/f5b68ce1-ec9b-4dca-a686-b6fd2e3c80ee" width="300">
  <img src="https://github.com/user-attachments/assets/a50d5c3d-3850-48e1-ab00-c3c064987117" width="300">
  <img src="https://github.com/user-attachments/assets/1db46506-b640-4d29-8aa1-36090e353d0c" width="300">
</p>


---

## Configure the DC: Name, NICs, Static IP
1. Rename the computer to `DC` and restart.  
2. Rename the NICs for clarity:  
   - NAT adapter → `INTERNET`  
   - Internal adapter → `INTERNAL`  
3. On the `INTERNAL` adapter, set:  
   - IP address: `172.16.0.1`  
   - Subnet mask: `255.255.255.0`  
   - DNS server: `172.0.0.1`
     
https://github.com/user-attachments/assets/72e32d6a-e103-4085-b8f6-525bad50d8e7


---

## Install AD DS and Create the Forest
1. In Server Manager, add the role: **Active Directory Domain Services**.  
2. Promote the server to a domain controller.  
3. Create a new forest (example: `mydomain.com`).  
4. Accept defaults and restart when prompted.  

<img width="830" height="548" alt="6(1)" src="https://github.com/user-attachments/assets/f4775b22-2f1a-4bfd-b0b1-90ffb8d07ff7" />
<img width="833" height="547" alt="6(2)" src="https://github.com/user-attachments/assets/88976e81-d9fb-45b1-b056-2ccb266a27f7" />
---

## Set Up NAT with Routing and Remote Access
1. In Server Manager, add the role: **Remote Access** (with Routing).  
2. Open the Routing and Remote Access console.  
3. Run the wizard → choose **NAT**.  
4. Select the `INTERNET` adapter as the public interface → start the service.  

<img width="840" height="595" alt="7(1)" src="https://github.com/user-attachments/assets/4e00443e-c95d-4467-b631-27a24edae5e3" />
<img width="831" height="590" alt="7(2)" src="https://github.com/user-attachments/assets/985d6390-a173-49ea-b27c-8bdb46a6ce24" />
<img width="832" height="592" alt="7(3)" src="https://github.com/user-attachments/assets/a3b4561e-7ea7-40b4-a7e7-e7b6b5024e1a" />
<img width="835" height="595" alt="7(4)" src="https://github.com/user-attachments/assets/0fd39489-c2c7-45ec-854f-f506a56da6cb" />

---

## Install and Configure DHCP
1. In Server Manager, add the role: **DHCP Server**.  
2. In the DHCP console, create a new IPv4 scope:  
   - Start: `172.16.0.100`  
   - End: `172.16.0.200`  
   - Subnet mask: `255.255.255.0`  
   - Router (gateway): `172.16.0.1`  
   - DNS server: `172.16.0.1`  
3. Authorize and activate the DHCP server.


<img width="841" height="588" alt="8(1)" src="https://github.com/user-attachments/assets/f9aed215-d847-4d28-a955-b377a7deb57f" />
<img width="835" height="587" alt="8(2)" src="https://github.com/user-attachments/assets/c10b34f9-a7a0-475f-a417-96fdeef7c65f" />
<img width="841" height="597" alt="8(3)" src="https://github.com/user-attachments/assets/44576904-95b3-4fa2-b980-c31d6499164e" />

---

## Bulk Add 1,000+ Users with PowerShell ISE
**Goal:** Test scalability and automation by creating **1,000 domain users** in Active Directory.  

**Steps:**  
1. Prepare a text file with 1,000+ names (one per line, e.g., “First Last”).  
2. On the Domain Controller, open **PowerShell ISE as Administrator**.  
3. Load the bulk-user script and point it to:  
   - The target OU (e.g., `_USERS`)  
   - Your domain (e.g., `mydomain.com`)  
   - The names list file  
4. Run the script and watch as users are created in bulk.  
5. Verify in **Active Directory Users and Computers (ADUC):**  
   - Open the OU → confirm ~1,000+ users exist  

<img width="840" height="592" alt="9(1)" src="https://github.com/user-attachments/assets/2d598eae-7266-4741-b821-24fd7c22333c" />
<img width="838" height="595" alt="9(2)" src="https://github.com/user-attachments/assets/9897ae5b-9b08-4ff8-bc63-b0828794d3d1" />
<img width="832" height="592" alt="9(3)" src="https://github.com/user-attachments/assets/f3df5ae3-f82f-48d0-8302-e0c554978f13" />

---

## Create the Client VM
1. Create a new VM for Windows 10 in VirtualBox.  
2. Assign memory (4 GB), 2 CPUs, and a 40 GB disk.  
3. Add one adapter: **Internal Network** (same as the DC’s).  
4. Attach the Windows ISO, install the OS, and complete setup.  

<img width="954" height="573" alt="60" src="https://github.com/user-attachments/assets/12be7d2e-9e7a-4af7-be85-e1d0ab43b878" />
<img width="1024" height="768" alt="61" src="https://github.com/user-attachments/assets/bd0663e6-2c08-4ee7-a481-063901a516e7" />


---

## Join the Client to the Domain
1. Confirm the client received an IP in `172.16.0.x` from DHCP.  
2. Rename the computer to `CLIENT0` (optional).  
3. Join the domain (`mydomain.com`).  
4. Restart and sign in with a domain user account.  

<img width="836" height="597" alt="11(1)" src="https://github.com/user-attachments/assets/a2d9ad7d-6956-4851-9c26-0b06cba37049" />
<img width="823" height="588" alt="11(2)" src="https://github.com/user-attachments/assets/13cd7e41-0e95-498c-b105-bbb759ca7879" />
<img width="831" height="591" alt="11(3)" src="https://github.com/user-attachments/assets/7d4e16c0-3dfe-4f61-a196-35076a28a54e" />
<img width="828" height="387" alt="11(4)" src="https://github.com/user-attachments/assets/3643004c-3117-44c3-9255-b17b1db57613" />
<img width="835" height="598" alt="11(5)" src="https://github.com/user-attachments/assets/f39d697a-e1a1-415e-a905-7d7d2d03e070" />

---

## Verification Checklist
**Domain Controller**  
- INTERNAL adapter = `172.16.0.1`, DNS = `172.16.0.1`  
- NAT enabled on INTERNET NIC (RRAS running)  
- DHCP scope active (`172.16.0.100–200`) and authorized  
- AD DS and DNS healthy  
- **_USERS OU contains ~1,000 user accounts**  

**Client**  
- Receives DHCP address in `172.16.0.x`  
- Can ping DC and resolve domain name  
- Can log in with a domain account  

---

## Snapshots & Reset Tips
- Take a snapshot after each milestone:  
  - Base OS installed  
  - DC promoted  
  - RRAS/DHCP configured  
  - **Before and after bulk user creation**  
  - Client joined  
- Roll back if networking or user creation fails.  

---



































# Windows Server 2022 Home Lab: Setting Up Active Directory &amp; DNS on Hyper-V to simulate an organizational structure

## Project Overview
The objective of this project was to gain hands-on experience managing Active Directory and understanding corporate IT structures. To achieve this, I built an isolated virtual environment in Hyper-V and deployed a Windows Server 2022 Domain Controller from scratch. The full scope of the project involved configuring DNS, setting up Active Directory Domain Services (AD DS), establishing a realistic Organizational Unit (OU) hierarchy, and provisioning user accounts. Finally, I deployed a Windows 11 Enterprise client and successfully joined it to the domain to verify centralized identity management and authentication.

All infrastructure was virtualized and built from scratch using clean OS ISO images.

## Technologies Utilized
* **Hypervisor:** Microsoft Hyper-V
* **Server Infrastructure:** Windows Server 2022
* **Endpoint OS:** Windows 11 Enterprise
* **Core Services:** Active Directory Domain Services (AD DS), Domain Name System (DNS)
* **Networking:** Isolated internal virtual switch, IPv4 static routing

## Architecture & Deployment Steps

**1. Virtualization & Network Infrastructure**
Configured Microsoft Hyper-V and deployed a dedicated Internal Virtual Switch (`Lab-Network`). This established an isolated sandbox network, preventing broadcast traffic or DHCP conflicts with the physical host network.

**2. Domain Controller Provisioning**
Deployed a Generation 2 virtual machine (`DC-01`) using a Windows Server 2022 ISO. Established network identity by configuring a static IPv4 address (`192.168.10.10`) and pointing the primary DNS resolution to the server's own IP address to prepare for domain role installation.

**3. Active Directory Configuration**
Installed the AD DS and DNS server roles. Promoted the server to a Domain Controller, establishing the root forest `homelab.network`. Created a logical organizational structure using Active Directory Users and Computers (ADUC), deploying an `IT Department` Organizational Unit (OU) and provisioning standard user accounts (e.g., Adam Smith, Lucy Ward) with standardized password policies.

**4. Client Endpoint Deployment**
Provisioned a Windows 11 Enterprise virtual machine (`Client-01`). Enabled virtual TPM 2.0 within the Hyper-V hardware settings to meet OS requirements, followed by a clean OS installation.

**5. Domain Integration & Authentication**
Configured the client endpoint with a static IP (`192.168.10.11`) on the same subnet as the Domain Controller and directed its DNS to the server. Authenticated via the legacy System Properties interface (`sysdm.cpl`) using Enterprise Admin credentials to successfully join the `homelab.network` domain.

## Diagnostics & Troubleshooting

During deployment, several technical hurdles were identified and resolved:

* **OOBE Network Requirement Bypass:** During the Windows 11 installation, the Out-of-Box Experience (OOBE) enforced a strict internet connection requirement for Microsoft account authentication. Because the VM resided on an isolated internal switch, this halted deployment. *Resolution:* Accessed the command prompt via `Shift + F10` and executed the `OOBE\BYPASSNRO` script to force a system reboot and enable local offline account creation.

* **Windows Edition Limitations:** The initial domain join attempt was blocked due to the OS defaulting to Windows 10/11 Home edition during setup. *Resolution:* Applied a generic Pro upgrade key via the activation settings to force an offline edition upgrade, unlocking enterprise domain features without requiring a full OS reinstall.

* **DNS Resolution & Subnet Isolation:** The client initially failed to resolve the domain name. Diagnostics revealed that the lack of a DHCP server on the internal switch caused the client to assign itself an APIPA address (`169.254.x.x`), resulting in a subnet mismatch. *Resolution:* Manually assigned a static IP (`192.168.10.11`) to align with the server's subnet, instantly restoring network connectivity and DNS resolution.

* **Remote Desktop Services Restriction:** Post-domain join, attempting to log in as a standard domain user resulted in an "Access Denied" error. Investigation revealed that Hyper-V's "Enhanced Session Mode" utilizes Remote Desktop Protocol (RDP), which denies access to standard users by default. *Resolution:* Disabled Enhanced Session Mode to force a standard console session, allowing successful standard user authentication.

## Project Visuals

**Virtual Infrastructure:**
Hyper-V Manager displaying both the Windows Server Domain Controller and the Windows 11 Client running simultaneously on the isolated virtual network.

<img width="1315" height="819" alt="Hyper-V" src="https://github.com/user-attachments/assets/eb988dfc-382f-4ecf-ab55-b7213d217fe4" />

**Active Directory & Endpoint Verification:**
Proof of concept demonstrating centralized identity management. The left window displays the Domain Controller managing the user account in ADUC. The right window displays the Windows 11 endpoint successfully authenticated to the domain, verified via the `whoami` command.

<img width="2162" height="868" alt="whoami" src="https://github.com/user-attachments/assets/fe2a5782-f558-43a9-8648-90fd2540c86d" />

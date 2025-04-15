<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of Active Directory within Azure Virtual Machines.<br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (22H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Preparing Active Directory Infrastructure in Azure
- Deploying Active Directory
- Creating Users with Powershell
- Group Policy and Managing Accounts

<h2>Deployment and Configuration Steps</h2>

<p> <b>1. Preparing Active Direcory Infraustructure in Azure</b>

- Setup a Domain Controller VM in Azure. Create a Resource Group, Vnet and Subnet and Domain Controller VM. For the Domain Controller configuring the following settings:
- Username: labuser
- Password: Cyberattack123!
- Attach to Resource Group you created
- VM Name: dc-1
- Region: East US 2
- Image: Windows Server 2022
- Size: 2 CPUs, 8gb RAM.
- Attach to Vnet you created in the Networking Tab.
- Review and Create and verify that dc-1 was created in Azure. Throughout the rest of the guide I will refer to the Domain Controller as "dc-1"
</p>
<p>
<img src="https://i.imgur.com/3qC3tLY.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/bV0l3ZX.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/qS03DYh.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/WMO4in6.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/P9yvkR5.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
</p>
<br />

<p><b>2. Setup a Client in Azure</b>
  
- Create the Client VM and call it "client-1". Follow similar steps as dc-1 but enter the following information and make sure it is set to the same "Resource Group" and "Vnet" when setting it up:
- VM name - client-1
- Image = Windows 10 Pro
- Username = labuser
- Password = Cyberattack23!
- For the rest of the guide I will refer to the Client VM as "client-1"

</p><b>3. Set Domain Controller's Private IP address to Static.</b>

- We are doing this as the Domain Controller (DC) is going to be used as a DNS Server for clients to connect to. So the DNS server of the client will be the "Static" IP of the DC.
- Open VMs in Azure -> Select dc-1 -> Select the NIC -> Select ipconfig1 -> Select "static" -> Save.
- dc-1 is now configured with a static IP address as we are going to use it as a DNS Server. 
<p>
<img src="https://i.imgur.com/LC4ckUJ.png" height="80%" width="80%" alt="DC-1 Static IP change"/>
<img src="https://i.imgur.com/dZE8tZU.png" height="80%" width="80%" alt="DC-1 Static IP change"/>
<img src="https://i.imgur.com/AO2QVLW.png" height="80%" width="80%" alt="DC-1 Static IP change"/>
</p>
<br />

<p><b>4. Log into dc-1 via RDP and disable Windows Firewall to test connectivity between hosts</b>

- Get dc-1's public IP address
- Connect to it via RDP
- Click Start -> type "Defender" -> Open "Windows Defender Firewall with Advanced Security" -> Click "Windows Defender firewall with Advanced Security" -> Click Windows Defender Firewall Properties -> Turn Firewall state off for Domain, Private and Public Profiles. Click Ok.
</p>
<p>
<img src="https://i.imgur.com/iNTM3O6.png" height="80%" width="80%" alt="Windows Firewall Connectivity"/>
<img src="https://i.imgur.com/ysBg1cS.png" height="80%" width="80%" alt="Windows Firewall Connectivity"/>
<img src="https://i.imgur.com/eiDfOWW.png" height="80%" width="80%" alt="Windows Firewall Connectivity"/>
<img src="https://i.imgur.com/g6r0yF0.png" height="80%" width="80%" alt="Windows Firewall Connectivity"/>
<img src="https://i.imgur.com/ADpefbn.png" height="80%" width="80%" alt="Windows Firewall Connectivity"/>
</p>
<br />

<p><b>5. Set client-1 DNS server parameter to private IP of dc-1</b>

- Get dc-1’s private IP 
- Open Network Settings for client-1 -> select NIC -> On left click DNS servers -> Under “Inherit virtual network” select custom -> enter the IP address of dc-1 -> click Save. The DNS Server for "client-1" is now dc-1 which handles resolving domain names to IPs for client-1 and enables us to join the domain.
- Restart "client-1" to make sure the new DNS configuration takes effect.
</p>
<p>
<img src="https://i.imgur.com/bz4UsJX.png" height="80%" width="80%" alt="DNS Server Parameter Change"/>
<img src="https://i.imgur.com/Ka6cuBW.png" height="80%" width="80%" alt="DNS Server Paramter Change"/>
</p>
<br />

<p><b>6. Log into client-1 and ping dc-1 private IP</b>

- Copy Private IP address of “client-1” from Azure and connect via RDP use login credentials we created previously.
- Open “Powershell” and ping "dc-1" private IP (10.0.0.4). Make sure the ping succeeded. 
- If you receive a message saying “Destination Host Unreachable” this means the hosts are on different Vnets or the firewall has not been disabled, so check those if you get that message.
- In "client-1" one run ipconfig /all to verify the DNS configuration change made in Azure and that it is pointing to "dc-1" as the DNS server.
</p>
<p>
<img src="https://i.imgur.com/3VQ03Lm.png" height="80%" width="80%" alt="Windows Firewall Connectivity"/>
<img src="https://i.imgur.com/3g8L8NJ.png" height="80%" width="80%" alt="Windows Firewall Connectivity"/>
<img src="https://i.imgur.com/0IpCby6.png" height="80%" width="80%" alt="Windows Firewall Connectivity"/>
</p>
<br />


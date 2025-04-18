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
<h3><b>Preparing Active Direcory Infraustructure in Azure</b></h3>
<p> <b>1.  Setup a Domain Controller in Azure</b>

- Create a Resource Group, Vnet and Subnet and Domain Controller VM. For the Domain Controller configure the following settings:
- Username: labuser
- Password: Cyberattack123!
- Attach to Resource Group you created
- VM Name: dc-1
- Region: East US 2
- Image: Windows Server 2022
- Size: 2 CPUs, 8gb RAM.
- Attach to Vnet you created in the Networking Tab.
- Review and Create. Then verify that dc-1 was created in Azure. Throughout the rest of the guide I will refer to the Domain Controller as "dc-1".
</p>
<p>
<img src="https://i.imgur.com/3qC3tLY.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/bV0l3ZX.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/qS03DYh.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/QxrINRk.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/P9yvkR5.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
</p>
<br />

<p><b>2. Setup a Client in Azure</b>
  
- Create the Client VM and call it "client-1". Follow similar steps as dc-1 but enter the following information and make sure it is set to the same "Resource Group" and "Vnet" when setting it up:
- VM name - client-1
- Image = Windows 10 Pro
- Username = labuser
- Password = Cyberattack23!
- For the rest of the guide I will refer to the Client VM as "client-1".

</p><b>3. Set Domain Controller's Private IP address to Static.</b>

- We are doing this as the Domain Controller (DC) is going to be used as a DNS Server for clients to connect to. So the DNS server of the client will be the "Static" IP of the DC.
- Open VMs in Azure -> Select dc-1 -> Select the NIC -> Select ipconfig1 -> Select "static" -> Save.
- dc-1 is now configured with a static IP address as we are going to use it as a DNS Server. 
<p>
<img src="https://i.imgur.com/LC4ckUJ.png" height="80%" width="80%" alt="DC-1 Static IP change"/>
<img src="https://i.imgur.com/i7guTZT.png" height="80%" width="80%" alt="DC-1 Static IP change"/>
<img src="https://i.imgur.com/ZM1Wtxh.png" height="80%" width="80%" alt="DC-1 Static IP change"/>
</p>
<br />

<p><b>4. Log into dc-1 via RDP and disable Windows Firewall to test connectivity between hosts</b>

- Get dc-1's public IP address.
- Connect to it via RDP.
- Click Start -> type "Defender" -> Open "Windows Defender Firewall with Advanced Security" -> Click Windows Defender Firewall Properties -> Turn Firewall state off for Domain, Private and Public Profiles. Click Ok.
</p>
<p>
<img src="https://i.imgur.com/iNTM3O6.png" height="80%" width="80%" alt="Windows Firewall Connectivity"/>
<img src="https://i.imgur.com/SPbV9EM.png" height="80%" width="80%" alt="Windows Firewall Connectivity"/>
<img src="https://i.imgur.com/eiDfOWW.png" height="80%" width="80%" alt="Windows Firewall Connectivity"/>
<img src="https://i.imgur.com/g6r0yF0.png" height="80%" width="80%" alt="Windows Firewall Connectivity"/>
<img src="https://i.imgur.com/ADpefbn.png" height="80%" width="80%" alt="Windows Firewall Connectivity"/>
</p>
<br />

<p><b>5. Set client-1 DNS server parameter to private IP of dc-1</b>

- Get dc-1’s private IP.
- Open Network Settings for client-1 -> select NIC -> On left click DNS servers -> Under “Inherit virtual network” select custom -> enter the IP address of dc-1 -> click Save. The DNS Server for "client-1" is now dc-1 which handles resolving domain names to IPs for client-1 and enables us to join the domain.
- Restart "client-1" to make sure the new DNS configuration takes effect.
</p>
<p>
<img src="https://i.imgur.com/bz4UsJX.png" height="80%" width="80%" alt="DNS Server Parameter Change"/>
<img src="https://i.imgur.com/Ka6cuBW.png" height="80%" width="80%" alt="DNS Server Paramter Change"/>
</p>
<br />

<p><b>6. Log into client-1 and ping the dc-1 Private IP address</b>

- Copy Private IP address of “client-1” from Azure and connect via RDP using login credentials we created previously.
- Open “Powershell” and ping "dc-1" private IP (10.0.0.4). Make sure the ping succeeded. 
- If you receive a message saying “Destination Host Unreachable” this means the hosts are on different Vnets or the firewall has not been disabled, so check those.
- In "client-1" run ipconfig /all to verify the DNS configuration change made in Azure and that it is pointing to "dc-1" as the DNS server.
</p>
<p>
<img src="https://i.imgur.com/3VQ03Lm.png" height="80%" width="80%" alt="Windows Firewall Connectivity"/>
<img src="https://i.imgur.com/3g8L8NJ.png" height="80%" width="80%" alt="Windows Firewall Connectivity"/>
<img src="https://i.imgur.com/0IpCby6.png" height="80%" width="80%" alt="Windows Firewall Connectivity"/>
</p>
<br />

<h3><b>Deploying Active Directory</b></h3>
<p> <b>1. Install Active Directory (AD)</b>

- Start -> Server Manager -> Add Roles and Features -> Click next till you get to “Server Roles” -> Check “Active Directory Domain Services” -> Click next all the way to the end and checking “Restart the destination server automatically if required” (optional, not necessary) -> Click install.
- Now configure dc-1 to become a domain controller post Active Directory install.
- Select flag with warning icon in Server manager -> Choose “Promote this server to a domain controller”.
- Choose “Add new forest” call it mydomain.com. Click next.
- Under “Type the Directory Services Restore Mode (DSRM)” enter a password. Click next.
- Uncheck “Create DNS delegation".
- Next all the way to the end and click install. Dc-1 should automatically restart so you will have to log back into the VM.
- It is now acting as a domain controller, we will want to use the mydomain prefix to specify the domain and user to log in from now on. So mydomain.com\labuser. This is to ensure we avoid logging in as a local user.
</p>
<p>
<img src="https://i.imgur.com/fIxNn5j.png" height="80%" width="80%" alt="Install Active Directory"/>
<img src="https://i.imgur.com/MUT8yZa.png" height="80%" width="80%" alt="Install Active Directory"/>
<img src="https://i.imgur.com/SKzlmMX.png" height="80%" width="80%" alt="Install Active Directory"/>
<img src="https://i.imgur.com/PV1FNSm.png" height="80%" width="80%" alt="Install Active Directory"/>
<img src="https://i.imgur.com/a8iZDG5.png" height="80%" width="80%" alt="Install Active Directory"/>
<img src="https://i.imgur.com/0spMRCk.png" height="80%" width="80%" alt="Install Active Directory"/>
<img src="https://i.imgur.com/a7YwbIY.png" height="80%" width="80%" alt="Install Active Directory"/>
<img src="https://i.imgur.com/ZonSjZM.png" height="80%" width="80%" alt="Install Active Directory"/>
<img src="https://i.imgur.com/dMKkcRh.png" height="80%" width="80%" alt="Install Active Directory"/>
<img src="https://i.imgur.com/8qomTba.png" height="80%" width="80%" alt="Install Active Directory"/>
<img src="https://i.imgur.com/PjY5Dh3.png" height="80%" width="80%" alt="Install Active Directory"/>
<img src="https://i.imgur.com/ZMsNRch.png" height="80%" width="80%" alt="Install Active Directory"/>
</p>
<br />

<p>  <b>2. Create a Domain Admin user within the Domain (AD)</b>

- Refer to steps in Lab 5 Deploying AD documentation.
- Open Server Manager -> Tools -> Select "Active Directory Users and Computers".
- Create 2 Organizational Units (OU). Right click domain -> New -> Organizational Unit. Call it _EMPLOYEES and click ok. Create another called _ADMINS.
- Refresh the domain and you will see the newly created OUs.
- Right click _ADMINS folder -> New -> User and enter the info:
- - Name: Jane Doe
  - Username: jane_admin
  - Password: Cyberlab123!
  - Enable “Password never expires”. This is bad a security practice and you would want the user to change their password at next login but disabling for the lab to save time.
  - Next, then Finish. You should notice the new user in the _ADMINS folder.
- Add the new user to “Domain Admins” security group.
- - Right click user -> Properties -> “Member of” tab -> Add -> Search domain admins then click “Check names” -> click Ok and Ok. Go back into the User’s properties and verify it is a member of “Domain Admins”.
- Log out and log back in as jane_admin.
</p>
<p>
<img src="https://i.imgur.com/hY7BAqx.png" height="80%" width="80%" alt="Create a Domain Admin user"/>
<img src="https://i.imgur.com/kOOa3tp.png" height="80%" width="80%" alt="Create a Domain Admin user"/>
<img src="https://i.imgur.com/Gc6MqYe.png" height="80%" width="80%" alt="Create a Domain Admin user"/>
<img src="https://i.imgur.com/UP5TdsX.png" height="80%" width="80%" alt="Create a Domain Admin user"/>
<img src="https://i.imgur.com/aGDBfaG.png" height="80%" width="80%" alt="Create a Domain Admin user"/>
<img src="https://i.imgur.com/DTcxSr3.png" height="80%" width="80%" alt="Create a Domain Admin user"/>
<img src="https://i.imgur.com/b9pgPMC.png" height="80%" width="80%" alt="Create a Domain Admin user"/>
<img src="https://i.imgur.com/uh7mc9D.png" height="80%" width="80%" alt="Create a Domain Admin user"/>
<img src="https://i.imgur.com/w7bmQQn.png" height="80%" width="80%" alt="Create a Domain Admin user"/>
<img src="https://i.imgur.com/1odzFZR.png" height="80%" width="80%" alt="Create a Domain Admin user"/>
<img src="https://i.imgur.com/oIYdyqI.png" height="80%" width="80%" alt="Create a Domain Admin user"/>
</p>
<br />

<p> <b>3. Join client-1 to Domain (AD)</b>

- Login to client-1 as the local labuser account.
- Open Server Manager -> Click Tools -> Select "Active Directory Users and Computers"
- Right click Start -> System -> Rename PC (advanced) -> Computer Name Tab, Click Change -> Choose Domain and enter mydomain.com click Ok. The "Computer Name/Domain Changes" window should open verifying we successfully contacted the domain thanks to the DNS parameter change we made earlier. If this window did not pop you will need to go back to Azure and change the "client-1" NIC DNS Server setting to point to the dc-1 private IP address.
- Enter the jane_admin account credentials and click Ok. A window should popup welcoming you to the domain. Restart the VM.
- Login to dc-1 as jane_admin and verify that client-1 has been added to AD.
- Create a new OU called "_CLIENTS" and drag client-1 from Computers tp _CLIENTS.
- Click "Yes" for the Warning notification. The Computer OU should be empty and client-1 should be in _CLIENTS now.
</p>
<p>
<img src="https://i.imgur.com/rhPReli.png" height="80%" width="80%" alt="Join Client-1 to Domain"/>
<img src="https://i.imgur.com/7Tl2zIX.png" height="80%" width="80%" alt="Join Client-1 to Domain"/>
<img src="https://i.imgur.com/uKLUvjA.png" height="80%" width="80%" alt="Join Client-1 to Domain"/>
<img src="https://i.imgur.com/zVzDd0W.png" height="80%" width="80%" alt="Join Client-1 to Domain"/>
<img src="https://i.imgur.com/PDc557o.png" height="80%" width="80%" alt="Join Client-1 to Domain"/>
<img src="https://i.imgur.com/laKAzZd.png" height="80%" width="80%" alt="Join Client-1 to Domain"/>
<img src="https://i.imgur.com/kRlv6Vk.png" height="80%" width="80%" alt="Join Client-1 to Domain"/>
<img src="https://i.imgur.com/pVJcR80.png" height="80%" width="80%" alt="Join Client-1 to Domain"/>
<img src="https://i.imgur.com/VrutJx9.png" height="80%" width="80%" alt="Join Client-1 to Domain"/>
</p>
<br />

<h3><b>Creating Users with Powershell</b></h3>
<p> <b>1. Setup Remote Desktop for non-administrative users on client-1 (AD)</b>

- Log into client-1 as jane_admin.
- Right click start -> System -> Remote Desktop -> Select users that can remotely access this PC. Click add -> search domain users and Check Names. Click Ok and Ok.
- Allow “domain users” to access remote desktop. All users by default are a member of this group on the domain.
- You now have the ability to log into this client as a non-administrative user. Normally this would be done with Group Policy to change a bunch of computers at once but as we only have the one client I went ahead and did this in the OS. 
</p>
<p>
<img src="https://i.imgur.com/P4KwgpC.png" height="80%" width="80%" alt="Creating Users with Powershell"/>
<img src="https://i.imgur.com/dEdCJ1g.png" height="80%" width="80%" alt="Creating Users with Powershell"/>
<img src="https://i.imgur.com/v3SAEtE.png" height="80%" width="80%" alt="Creating Users with Powershell"/>
</p>
<br />

<p> <b>2. Create a bunch of additional users then attempt to log into client-1 with one of the users (AD)</b>

- Login to dc-1 as jane_admin.
- Open "Powershell_ise" as administrator.
- Create a new file called “create-users” to Desktop and paste contents of script into it. This script will automatically create random user accounts into our _EMPLOYEES OU folder without us having to manually create them.
- Click” Run Script” and Ok. You should see in the blue command line window user accounts being created. It will take a few minutes to create.
- Open “Active Directory Users and Computers” and observe the user accounts being created in _EMPLOYEES.
- Attempt to log into client-1 using one of these new created accounts. I will use “bab.wilo”. Take note of the password that was used in the script.
- Open PowerShell, you will notice a local folder has been create for the new user on client-1. Open file explorer and go to C:\Users. You will notice folders have been created for all the accounts that have logged into client-1.
</p>
<p>
<img src="https://i.imgur.com/XynsHhj.png" height="80%" width="80%" alt="Creating Users with Powershell"/>
<img src="https://i.imgur.com/Xppy7Sp.png" height="80%" width="80%" alt="Creating Users with Powershell"/>
<img src="https://i.imgur.com/yhnam3r.png" height="80%" width="80%" alt="Creating Users with Powershell"/>
<img src="https://i.imgur.com/DpSAV2u.png" height="80%" width="80%" alt="Creating Users with Powershell"/>
<img src="https://i.imgur.com/FTkvwzV.png" height="80%" width="80%" alt="Creating Users with Powershell"/>
<img src="https://i.imgur.com/asaQiMM.png" height="80%" width="80%" alt="Creating Users with Powershell"/>
<img src="https://i.imgur.com/MvJhzdt.png" height="80%" width="80%" alt="Creating Users with Powershell"/>
<img src="https://i.imgur.com/sWjKBbp.png" height="80%" width="80%" alt="Creating Users with Powershell"/>
<img src="https://i.imgur.com/6Ll69sN.png" height="80%" width="80%" alt="Creating Users with Powershell"/>
<img src="https://i.imgur.com/QhGjaS9.png" height="80%" width="80%" alt="Creating Users with Powershell"/>
<img src="https://i.imgur.com/m40bnKx.png" height="80%" width="80%" alt="Creating Users with Powershell"/>
</p>
<br />

<h3><b>Group Policy and Managing Accounts</b></h3>
<p> <b>1. Dealing with Account Lockouts</b>

- Pick a user account you created. Open Active Directory Users and Computers. I will use the bab.wilo account.
- Setup Account Lockout Policy in Group Policy Management:
- - Open Server Manager -> Tools -> Group Policy Management. We can create a new Group Policy Object (GPO) but I've chosen to edit the “Default Domain Policy” for now. Right click it -> Edit.
  - Browse to Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Account Policies -> Account Lockout Policy.
  - Double click “Account Lockout Duration” -> Change Lockout duration to 30min. Press "Ok" for the change in suggested values.
  - The Parameters should now look like:
  - Account Lockout Duration = "30 minutes"
  - Account Lockout Threshold = "5 invalid logon attempts"
  - Allow Administrator account lockout - "Not defined"
  - Reset account lockout counter after = "10 minutes"
- - Close “Group Policy Management Editor”. In Group Policy Management open the settings tab of “Default Domain Policy” and verify the Account Lockout Policy change has been applied.
- Update the Group Policy on client-1 for the account lockout policy to take effect. To do this, login to client-1 with a domain admin account (jane_admin) to force the policy update so we can test the lock out policy.
- Open command as administrator -> type gpupdate /force.
- - Now on client-1 when a user fails to login 5 times the account should logout.
- Next, type gpresult /r.
- - Under COMPUTER SETTINGS search for “Applied Group Policy Objects". Beneath that should be “Default Domain Policy”. This tells us the GPO has been applied to this computer and any changes we have made to it will be applied to client-1.
- Conduct 6 failed login attempts with the account. The account should now be locked out. This error indicates our group policy worked.
- Switch over to dc-1 and unlock the account. Right click the domain -> select Find -> Search the account name and it should come up in the search results. Go into Properties -> tick "Unlock Account" -> Ok. Account should now be unlocked.
- Attempt to login to client-1 as the account. Open Powershell -> type whoami. You should see you are logged onto mydomain as “bab.wilo".
- Reset the password for the account.
- - Open Active Directory and find the account. Right click -> Reset Password -> Enter a new password -> Ok. Confirmation window should pop up saying the password has been changed for the account.
- Attempt to login with the old password. It should not work. Then enter new password and that should work.
</p>
<p>
<img src="https://i.imgur.com/DMmqa12.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/ZDpVQ7b.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/GgQi6Ef.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/piWVEAd.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/ggpjn8I.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/ESGPsou.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/rCW3BWb.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/BM2tYKS.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/ygAFAPr.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/aB0sBWp.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/N6hp4ww.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/oy2BdZT.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/NNXOPK5.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/9C0YGUY.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/q2hC4Di.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/657Vf5N.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/CUdk4ft.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/F0qIcVY.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/yIihXdl.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/FIYiuzX.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
</p>
<br />

<p> <b>2. Enabling and Disabling Accounts</b>

- I will disable the bab.wilo account.
- Switch to dc-1. Search for the account in AD -> Right click it and select “Disable account” -> Window should popup saying the account was disabled. 
- Now, try logging in to client-1 with the account. You should get the error message that the account is disabled.
- Re-enable the account and attempt to log back in. Search for the account in Active Directory -> Right click -> “Enable account”. Window should popup saying the account has been enabled.
- Check command line to verify that you are logged in as the account.
</p>
<p>
<img src="https://i.imgur.com/OvYBAmb.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/6ayxYSO.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/iXTQ5RS.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/WseCYO9.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/AaGVS1P.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
<img src="https://i.imgur.com/Ul3oDvs.png" height="80%" width="80%" alt="Setting up Domain Controller"/>
</p>
<br />

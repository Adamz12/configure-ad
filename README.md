<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />


<h2>Video Demonstration</h2>

- ### [YouTube: How to Deploy on-premises Active Directory within Azure Compute](https://www.youtube.com)

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Setup Domain Controller in Azure
- Setup Client-1 in Azure
- Install Active Directory
- Create a Domain Admin user within the domain
- Join Client-1 to your domain (mydomain.com)
- Setup Remote Desktop for non-administrative users on Client-1
- Create a bunch of additional users and attempt to log into client-1 with one of the users
- Dealing with Account Lockouts
- Enabling and Disabling Accounts
- Observing Logs






<h2>Deployment and Configuration Steps</h2>

<h3>Part 1 - Setup Controller in Azure and Client 1 </h3>

<p>
<h4>Step 1: Set Up the Azure Foundation</h4>
  
- In the Azure portal, click Create a resource → Resource group

- Name: Active-Directory-Lab

- Region: Choose the same region you’ll use for your VMs (e.g., UK South)

- Click Review + create, then Create

- Next, create → Virtual network

- Name: Active-Directory-Vnet

- Resource group: Select Active-Directory-Lab

- Click Review + create, then Create

</p>

<p>
<img src="https://i.postimg.cc/Y09J9qzD/A-Resource-Group.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>
<p>
<img src="https://i.postimg.cc/SN8pzjpH/A-Virtual-Network.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>

<br />

<p>
<h4>Step 2: Deploy the Domain Controller VM (dc-1)</h4>
  
- In the Azure portal, click Create a Virtual machine

- Resource group: Active-Directory-Lab

- VM name: dc-1

- Image: Windows Server 2022 Datacenter (Gen2)

- Size: (choose an appropriate size, e.g., Standard_D2s_v3)

Administrator account:

- Username: labuser

- Password: Cyberman111!

- Windows licensing: Tick Azure Hybrid Benefit (if applicable) or accept the licensing terms

- Networking → Virtual network: Active-Directory-Vnet

- Leave other settings at their defaults, then click Review + create → Create

</p>
<p>
<img src="https://i.postimg.cc/nzJGhRpX/creating-dc1.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>
<br />

<p>
<h4>
  Step 3: Deploy the Client VM (client-1)
</h4>
 
- Click Create a Virtual machine

- Resource group: Active-Directory-Lab

- VM name: client-1

- Image: Windows 10 Pro, Version 22H2

- Size: (e.g., Standard_B2ms)

Administrator account:

- Username: labuser

- Password: Cyberman111!

- Windows licensing: Tick Azure Hybrid Benefit or accept licensing terms

- Networking → Virtual network: Active-Directory-Vnet

- Click Review + create → Create

- You now have two VMs in the Active-Directory-Lab RG on the Active-Directory-Vnet:

- dc-1 (Windows Server 2022) ready to become your domain controller

- client-1 (Windows 10 Pro) to join your new domain.
</p>
<p>
<img src="https://i.postimg.cc/C5vGJwD8/creating-client-1.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>

<p>
<h4>
  Step 4: Set Domain Controller’s NIC Private IP address to be static
</h4>

- In the Azure portal, go to Virtual machines and select dc-1

- Under Networking, click Network settings

- Click Network interface / IP configurations

- Select ipconfig1

- Change Assignment from Dynamic to Static, enter your chosen IP (e.g., 10.0.0.4), and click Save

This ensures the domain controller keeps the same private IP address permanently.
</p>
<p>
<img src="https://i.postimg.cc/pT57j1FR/static.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>

<p>
<h4>
  Step 5: Log into DC-1 and disable the Windows Firewall (for testing connectivity)
</h4>

- Open your RDP client and connect to dc-1 using its public IP and the labuser/Cyberman111! credentials

- Once logged in, press Win+R, type wf.msc, and press Enter to open Windows Defender Firewall with Advanced Security

- In the left pane, right-click Windows Defender Firewall with Advanced Security on Local Computer and select Properties

- In the Properties window, for each profile tab (Domain Profile, Private Profile, Public Profile):

- Set Firewall state to Off

- Click Apply

- Click OK to close the dialog—Windows Firewall is now disabled on the domain controller for testing purposes.
</p>
<p>
<img src="https://i.postimg.cc/44DPLQY2/turn-off-firewall.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>

<p>
<h4>
  Step 6: Set Client-1’s DNS settings to DC-1’s Private IP address
</h4>

- In the Azure portal, go to Virtual machines and select client-1

- Under Network, click Network settings

- Click Network interface / IP configurations

- In the left panel, select DNS servers

- Choose Custom

- Paste the private IP address of dc-1 into the DNS server field

- Click Save to apply the DNS change to client-1
</p>
<p>
<img src="https://i.postimg.cc/QMdv8Jkk/change-dns.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>

<p>
<h4>
  Step 7: Attempt to ping DC-1’s private IP address from Client-1
</h4>

- Restart client-1 from the Azure portal

- Log into client-1

- Open PowerShell and run ping <dc-1-private-IP>

- Run ipconfig /all

- Confirm DNS Servers lists dc-1’s private IP address
</p>
<p>
<img src="https://i.postimg.cc/qvGBzQfN/restart-client-1.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>
<p>
<img src="https://i.postimg.cc/FzcNmVgs/ping-dc-1-private-ip.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>
<p>
<img src="https://i.postimg.cc/28QrKTS0/ip-config-all-dns-server.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>

<h3>Part 2 - Install Active Directory
 </h3>

 <p>
<h4>
  Step 1: Login to DC-1 and install Active Directory Domain Services
</h4>

- Log into dc-1 via RDP

- Open Server Manager

- Click Add Roles and Features

- Proceed through the wizard to the Server Roles page

- Check Active Directory Domain Services

- When prompted, select Restart destination server automatically if required

- Click Next through the remaining pages and then Install

- Wait for the installation to complete (server will reboot if needed)
</p>
<p>
<img src="https://i.postimg.cc/T3KWzpp7/server-manager.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>
<p>
<img src="https://i.postimg.cc/g0PkgJR6/restartr.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>

 <p>
<h4>
  Step 2: Promote as a DC: Setup a new forest as mydomain.com and Restart and then log back into DC-1 as user: mydomain.com\labuser

</h4>

- Log into dc-1 via RDP

- Open Server Manager and click the flag icon, then Promote this server to a domain controller

- Select Add a new forest and enter mydomain.com as the Root domain name

- Click Next, set the Directory Services Restore Mode (DSRM) password, and click Next

- On the DNS Options page, leave Create DNS delegation unchecked and click Next

- Accept the defaults on the Paths page and click Next

- Review your settings, then click Install

- The server will reboot automatically

- After restart, log back into dc-1 as mydomain.com\labuser
  
</p>
<p>
<img src="https://i.postimg.cc/ZqHkSK9g/flag.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>
<p>
<img src="https://i.postimg.cc/nrsNB4db/forest.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>
<p>
<img src="https://i.postimg.cc/zfJM0Zdk/unchecked.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>

 <p>
<h4>
  Step 3: 
Create a Domain Admin user within the domain
</h4>

- On dc-1, click the Start button and open Windows Administrative Tools → Active Directory Users and Computers

- In the left pane, expand mydomain.com

- Right-click mydomain.com, choose New → Organizational Unit

- Name the first OU exactly _EMPLOYEES and click OK

- Repeat the process to create a second OU named exactly _ADMINS
  
</p>
<p>
<img src="https://i.postimg.cc/xTLskvMm/windows-key.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>
<p>
<img src="https://i.postimg.cc/WpFYVL4X/ou.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>
<p>
<img src="https://i.postimg.cc/SKxVtH5L/admin-employees.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>

 <p>
<h4>
  Step 4: 
Create a Domain Admin user within the domain - Create a new employee named “Jane Doe”
</h4>

- Open Active Directory Users and Computers on dc-1

- Expand mydomain.com and select the _ADMINS OU

- Right-click _ADMINS → New → User

In the New Object – User wizard:

- First name: Jane

- Last name: Doe

- Full name: Jane Doe

- User logon name: jane_doe

- Click Next, then set Password: Cyberman111!

- Check Password never expires and uncheck any other options (e.g., “User must change password at next logon”)

- Click Next, then Finish to create the new admin account.
  
</p>
<p>
<img src="https://i.postimg.cc/wMs5jJbX/new-admin-creds.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>
<p>
<img src="https://i.postimg.cc/L4rwBVRM/new-admin-password.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>
<p>
<img src="https://i.postimg.cc/MpYLw2BW/new-admin-details.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>

 <p>
<h4>
  Step 5: 
Create a Domain Admin user within the domain - Add jane_admin to the “Domain Admins” Security Group

</h4>

- Open Active Directory Users and Computers on dc-1

- Navigate to the _ADMINS OU and find Jane Doe

- Right-click Jane Doe and select Properties

- Go to the Member Of tab and click Add…

- Type Domain Admins, click Check Names (it should underline), then click OK

- Back in the Member Of tab, click Apply, then OK

Jane Doe now belongs to the Domain Admins group and has full administrative privileges across the domain.
  
</p>
<p>
<img src="https://i.postimg.cc/fbGmgkbk/jane-becomes-admin.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>

<h4>
  Step 6: 
Create a Domain Admin user within the domain - Log out / close the connection to DC-1 and log back in as jane_admin

</h4>

- Log out
- Then log in as  “mydomain.com\jane_admin”
- Use jane_admin as your admin account from now on
  
</p>
 <p>
<img src="https://i.postimg.cc/DZQJjgDY/log-in-jane-admin.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>

<p>
<h4>
  Step 7: 
Join Client-1 to your domain (mydomain.com)

</h4>

- Log into client-1 via RDP using labuser/Cyberman111!

- Open Settings → System → About, then click Rename this PC (advanced)

- In the System Properties window, click Change… next to “To rename this computer…”

- Select the Domain option, enter mydomain.com, and click OK

- When prompted, enter mydomain.com\jane_doe as the username and Cyberman111! as the password

- Click OK to confirm, then OK again on the welcome message

- Allow client-1 to restart—after reboot, it will be joined to the mydomain.com domain

- Then log onto DC1 and go to active directory users and computers

- mydomain.com then go to computers and client 1 should be there
  
</p>
 <p>
<img src="https://i.postimg.cc/k5vw8xsq/client-1-connect-to-domain.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>
 <p>
<img src="https://i.postimg.cc/RhyT9GSQ/ok-message.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>
 <p>
<img src="https://i.postimg.cc/9XKPD82y/checking-client1-connected-dc1.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>

<p>
<h4>
  Step 8: 
Create a new OU named “_CLIENTS” 

</h4>

- On dc-1, open Active Directory Users and Computers

- In the left pane, right-click mydomain.com, choose New → Organizational Unit

- Name the OU exactly _CLIENTS and click OK

- In the left pane, click Computers to view all joined machines

- Find client-1, then drag and drop it into the newly created _CLIENTS OU
  
</p>
 <p>
<img src="https://i.postimg.cc/26Bg28x8/moving-client-1-to-clients.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>

<h3>Part 2 - Setup Remote Desktop for non-administrative users on Client-1 and Create a bunch of additional users and attempt to log into client-1 with one of the users
 </h3>

<p>
<h4>
  Step 1: 
Setup Remote Desktop for non-administrative users on Client-1

</h4>

- Log into client-1 as mydomain.com\jane_doe

- Open Settings → System → Remote Desktop

- Click Select users that can remotely access this PC

- In the Remote Desktop Users dialog, click Add…

- Enter Domain Users, click Check Names, then OK

- Click OK again to close the dialog

Now any non-admin domain user can establish an RDP session to client-1.
</p>
 <p>
<img src="https://i.postimg.cc/52fw97Yx/system-remote-desktop.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>
 <p>
<img src="https://i.postimg.cc/tJhxpW9Z/domain-users-access.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>

<p>
<h4>
  Step 2: 
 Create a bunch of additional users

</h4>

- Log into dc-1 as mydomain.com\jane_doe

- Open Windows PowerShell ISE as Administrator

- In the ISE window, click File → New

- Click File → Save, name it create-users.ps1, and save it to a convenient location (e.g., your Documents folder or Desktop)

Paste the contents of the script from:
- https://raw.githubusercontent.com/joshmadakor1/AD_PS/master/Generate-Names-Create-Users.ps1

- Run the script and observe the accounts being created
</p>
  <p>
<img src="https://i.postimg.cc/4350mctY/windows-powershell-ise.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>
  <p>
<img src="https://i.postimg.cc/tTc2xNg4/create-script.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>
  <p>
<img src="https://i.postimg.cc/yd3rnbtZ/observe-accounts.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>
  <p>
<img src="https://i.postimg.cc/9FgZFQrc/check-aduc.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>

<p>
<h4>
  Step 3:  
 Attempt to log into Client-1 with one of the accounts (take note of the password in the script)
</h4>

- On client-1, log in as one of the created users

- Enter Username: mydomain.com\<sampleUsername> (replace <sampleUsername> with one of the script-generated accounts)

- Enter Password: Password1

- If the desktop loads successfully, the account creation and domain join are confirmed working
</p>
  <p>
<img src="https://i.postimg.cc/VvQ1FMGz/additional-user-credentials.png" height="800" width="800" alt="Disk Sanitization Steps"/>
</p>

<br />

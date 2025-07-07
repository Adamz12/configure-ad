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
<img src="https://i.postimg.cc/Y09J9qzD/A-Resource-Group.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
<img src="https://i.postimg.cc/SN8pzjpH/A-Virtual-Network.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
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
<img src="https://i.postimg.cc/nzJGhRpX/creating-dc1.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
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
<img src="https://i.postimg.cc/C5vGJwD8/creating-client-1.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
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
<img src="https://i.postimg.cc/pT57j1FR/static.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>

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
<img src="https://i.postimg.cc/44DPLQY2/turn-off-firewall.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<br />

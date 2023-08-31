<p align="center">
<img src="https://i.imgur.com/Ua7udoS.png" alt="Traffic Examination"/>
</p>

<h1>Network Security Groups (NSGs) and Inspecting Traffic Between Azure Virtual Machines</h1>
In this tutorial, we observe various network traffic to and from Azure Virtual Machines with Wireshark as well as experiment with Network Security Groups. <br />


<h2>Video Demonstration</h2>

- ### [YouTube: Azure Virtual Machines, Wireshark, and Network Security Groups](https://www.youtube.com)

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Various Command-Line Tools
- Various Network Protocols (SSH, RDH, DNS, HTTP/S, ICMP)
- Wireshark (Protocol Analyzer)

<h2>Operating Systems Used </h2>

- Windows 10 (22H2)
- Ubuntu Server 22.04

<h2>High-Level Steps</h2>

- Create a Windows Virtual Machine and a Linux Virtual Machine 
- Download and Install Wireshark
- Observe ICMP, SSH, DHCP, DNS, and RDP Traffic
- Cleaning Up Resource Groups

<h2>Actions and Observations</h2>

### Create a Free Azure Account
<img width="786" alt="image" src="https://i.imgur.com/h1oEndK.png">

1. Sign up: [Azure Free Account](https://azure.microsoft.com/en-us/free/)
2. Login: [Azure Portal](https://portal.azure.com)

### Create 2 Virtual Machines
<img width="786" alt="image" src="https://i.imgur.com/bqa4s5q.png">

1. Go to [Azure Portal](https://portal.azure.com)
2. Search for Virtual Machines and create a new Virtual Machine.
3. Configure the 1st VM:
   - Resource Group: RG-NSG
   - VM Name: vm-windows
   - Region: East US 2 (or a region closer to you)
   - Image: Windows 10 Pro
   - Size: 2 vCPUs
   - Username: azureuser / Password123!
   - Networking: Default settings
   - Virtual Network: Default settings
4. Create the VM.
5. Once the VM is created, create another VM.
6. Configure the 2nd VM:
   - Resource Group: RG-NSG (same as vm-windows)
   - VM Name: vm-linux
   - Region: East US 2 (or a region closer to you)
   - Image: Ubuntu Server 22.04
   - Size: 2 vCPUs
   - Authentication Type: Password
   - Username: azureuser / Password123!
   - Virtual Network: vm-windows-vnet (same as vm-windows)
7. Create the 2nd VM.
8. Once both VMs are created, ensure you can RDP into <b>vm-windows</b> with the provided credentials.
9. After logging in, we'll install Wireshark:
   - Open browser and go to [Wireshark](https://www.wireshark.org/download.html)
   - Download and Install <b>Windows Installer (64-bit)</b>

### Monitoring Internet Control Message Protocol (ICMP) Traffic with Wireshark
<img width="786" alt="image" src="https://i.imgur.com/FDG7Mkk.png">

1. Type <b>Wireshark</b> in Windows Search Bar and open
2. Select <b>Ethernet</b> and <b>Start capturing packets</b> (blue icon on top)
   - Observe the live traffic of vm-windows
3. Type <b>icmp</b> in the filter above
4. Back in [Azure Portal](https://portal.azure.com), go to Virtual Machines
5. Select <b>vm-linux</b> and copy the Private IP address
6. Go back to vm-windows
7. Open Powershell and type <b>ping 10.0.0.5</b> (replace with your <b>vm-linux</b> Private IP address).
   - Observe the traffic between VMs within WireShark
8. Now type <b>ping www.google.com</b>
   - Observe the traffic between a VM and a public website within Wireshark
9. In Wireshark, click <b>Restart current capture</b> (green icon on top)
10. Then <b>Continue without saving</b> to clear previous traffic
11. In Powershell, type <b>ping 10.0.0.5 -t</b> in initiate continous ping
12. Back to [Azure Portal](https://portal.azure.com), search <b>Network security group</b> or <b>nsg</b>
13. Select <b>vm-linux-nsg</b> > <b>Inbound security rules</b> > <b>Add</b> and configure:
    - Protocol: ICMP
    - Action: Deny
    - Name: Deny_ICMP
14. Once it's done, return to <b>vm-windows</b>:
    - Observe the activity within PowerShell and Wireshark. You should now see a stream of requests without any replies.
15. Navigate back to <b>vm-linux-nsg</b> > <b>Inbound security rules</b> > <b>Deny_ICMP</b> configure:
    - Action: Allow
16. Return to <b>vm-windows</b>:
    - Observe the activity within PowerShell and Wireshark. You should now see a stream of both requests and replies.
17. In Powershell, press <b>ctrl</b> + <b>c</b> to stop the ping
18. In Wireshark, click <b>Restart current capture</b> to clear previous traffic

### Observing Secure Shell (SSH) Traffic
<img width="786" alt="image" src="https://i.imgur.com/WgqRDpW.png">

1. Still in <b>vm-windows</b>, go to Wireshark, type <b>ssh</b>
2. In Powershell, type <b>ssh azureuser@10.0.0.5</b>
   - Continue Connecting?: type <b>yes</b>
   - Password: type <b>Password123!</b> (you won't see it as you type but it's there)
   - Press Enter and it should log you in to <b>vm-linux</b> via SSH
3. Once logged in, observe Wireshark as we type different Linux commands:
   - <b>id</b> (Confirm identity of a specified Linux user)
   - <b>uname -a</b> (Display system information)
   - <b>pwd</b> (Print working directory)
   - <b>ls -lasth</b> (List files/folders in current directory)
   - <b>touch hi.txt</b> (Create blank/empty files)
   - <b>ls -lasth</b> (Observe the new text file created)
   - <b>rm hi.txt</b> (Delete files or directories)
   - <b>ls -lasth</b> (Observe the lack of text file)
   - <b>exit</b> (Close SSH connection)

### Observing Dynamic Host Configuration Protocol (DHCP) Traffic
<img width="786" alt="image" src="https://i.imgur.com/z25VWvg.png">

1. In Wireshark, type <b>dhcp</b>
2. In Powershell, type <b>ipconfig /renew</b> to reissue IP address 
   - Observe live dhcp traffic within Wireshark

### Observing Domain Name System (DNS) Traffic
<img width="786" alt="image" src="https://i.imgur.com/x5GRwJq.png">

1. In Wireshark, type <b>dns</b>
2. In Powershell, type <b>nslookup www.google.com</b>
   - Observe live dns traffic within Wireshark
3. Now type <b>nslookup www.disney.com</b>
   - Observe live dns traffic within Wireshark

### Observing Remote Desktop Protocol (RDP) Traffic
<img width="786" alt="image" src="https://i.imgur.com/VG06T2X.png">

1. In Wireshark, type <b>tcp.port == 3389</b>
   - Observe the continuous traffic within Wireshark. Since we're actively connected to <b>vm-windows</b> via RDP from our PC, traffic is consistently being transmitted.
2. Close your remote desktop connection.

### Deleting Resource Groups
<img width="786" alt="image" src="https://i.imgur.com/W6MxJTI.png">

1. Go back to [Azure Portal](https://portal.azure.com) and navigate to <b>Resource Groups</b>
2. Select <b>RG-NSG</b> > <b>Delete resource group</b>
3. Once finished, head back to <b>Resource Groups</b> > <b>NetworkWatcherRG</b> > <b>Delete resource group</b>.
4. Refresh [Azure Portal](https://portal.azure.com) to ensure no resource groups are left. Congratulations, we've finished!









<p align="center">
<img src="https://github.com/simoneburch/config-ad/assets/152559137/a9a26ee3-8006-46b3-bcd0-7f160a942836" height="50%" width="50%" alt="On-Prem Active Directory Logo"/>
</p>

<h1 align="center">On-premises Active Directory Deployed in Azure</h1>
This tutorial explains the installation, configuration, and creation of users in Active Directory (on-premises) using two Azure Virtual Machines. One is used for the domain controller and the other for the client. We join the client to the domain and solve common user issues. We also showcase the use of Remote Desktop, Active Directory Domain Services, and Organizational Units.

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines)
- Remote Desktop
- Active Directory Domain Services
- PowerShell scripting language

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 Pro

<h2>Deployment and Configuration Steps</h2>

<h3>Step 1: Creating the Domain Controller VM</h3>

![image](https://i.imgur.com/zmUeU34.jpg)


- Navigate to https://portal.azure.com/ and create a VM > label it as DC-1 (the Domain Controller)
- When creating the VM, select "Windows Server 2022" image with 2 vCPUs
- Make note of the username and password for later use
- Click on review and create
- Once created, go to the networking tab > click on the "Network Interface" > IP configurations > ipconfig1 and make sure the Allocation button is set to "static". Save <br />
<br />
<br />

<h3>Step 2: Creating the Client VM</h3>

![image](https://github.com/simoneburch/config-ad/assets/152559137/2b376db9-72b0-4aa5-b850-3afb8ad30c56)


- Create a new VM and label it as Client-1
- Use Windows 10 Pro with 2 vCPUs
- Make sure to use the same resource group and VNet that the DC-1 VM is using 
- Check that both VMs are in the same VNet by clicking on the "Virtual Machines" in the Azure Portal <br />
<br />

___**PLEASE NOTE: The DC-1 VNet may take a few minutes to load before you can select it - Refresh the page until that option is available to choose from.___<br />
<br />
<br />

<h3>Step 3: Testing Client VM and Domain Controller VM Connectivity</h3>

<img src="https://i.imgur.com/SFzICM7.jpg" width="70%" height="70%"/>


- Log in to Client-1 and ping DC-1's IP address using the ping -t command:
- Copy DC-1's private IP address from the Azure portal
- Navigate to the Command Prompt by looking up "CMD" in the search bar
- In the Command Line type: ping -t __DC-1 private IP__ (ping -t __10.0.0.4__)
- You should see "Request timed out." showing that no connection is being established otherwise we'd get a response from 10.0.0.4 (DC-1) <br />
<br />
<br />

<h3>Step 4: Enable local Windows firewall on Domain Controller</h3>

<p align="center">
<img src="https://i.imgur.com/M1HM7m4.jpg" height="100%" width="100%"/>
</p>

- Login to the Domain Controller and enable ICMPv4 on the local Windows firewall:
	Remote into DC-1>go to Windows start icon > type Windows Defender and Advanced Security > Inbound Rules > expand > sort by protocol > ICMPv4 > Right-click to Enable Rule for both Core Networking Diagnostic-ICMP Echo Requests
- Log back into Client-1 to see the ping succeed <br />
<br />
<br />

<h3>Step 5: Installing Active Directory</h3>

- In DC-1 > Server Manager > go to Add Roles and Features > under Server Roles > select "Active Directory Domain Services"
<p align="center">
<img src="https://i.imgur.com/LKpjSjC.jpg" height="60%" width="60%" alt="Azure Step 5-5"/>
<img src="https://i.imgur.com/nRniK70.jpg" height="100%" width="100%" alt="Azure Step 5-5"/>
</p>

- Back on the Server Manager, click on the flag icon with a caution symbol on it (located at top-right header).
- Click "Promote this server to a domain controller"
<p align="center">
<img src="https://i.imgur.com/5SEF3r7.jpg" height="60%" width="60%" alt="Azure Step 5-5"/>
</p>

- In the Deployment Configuration tab, select "Add a new forest".
- Type any domain name you wish to use (this example uses **mydomain.com**)
- Click "Next".
<p align="center">
<img src="https://i.imgur.com/KqLJ1zR.jpg" height="60%" width="60%" alt="Azure Step 5-5"/>
</p>

- Create a password of your choice.
- Keep clicking "Next" until the "Install" option is enabled, then click "Install".
  - _Installing will result in restarting the Domain Controller VM._
<p align="center">
<img src="https://i.imgur.com/sOfwFw2.jpg" height="60%" width="60%" alt="Azure Step 5-5"/>
<img src="https://i.imgur.com/GGJhNCF.jpg" height="60%" width="60%" alt="Azure Step 5-5"/>
<img src="https://i.imgur.com/8eabtaB.jpg" height="60%" width="60%" alt="Azure Step 5-5"/>
</p>

- Once restarted, log back into DC-1 as user > select "More Choices" > click "Use a different account" > now use __mydomain.com\labuser__ and the password you created.<br />
<br/>

_**We now need to specify the context of the user by including the whole domain, or **fully qualified domain name (FQDN)** whenever we log in.<br />_
<br />
<br />

<h3>Step 6: Creating Admin account, User account, and Organizational Units</h3>

![image](https://github.com/simoneburch/config-ad/assets/152559137/4f920fa3-e263-47c2-bb8d-00c77cdbbbc9)


- In Active Directory Users and Computers (ADUC), create an Organizational Unit (OU) called “_EMPLOYEES”
- Create a new OU named “_ADMINS”<br/>
<br/>

![image](https://github.com/simoneburch/config-ad/assets/152559137/b9c1fd76-3baf-4080-abe0-91e7108c4b3a)

- Create a new employee named “Jane Doe” (set as same password) with the User logon name of “jane_admin” by right-clicking in the view space > New > User > fill in the New Object-User input fields.
- Add jane_admin to the “Domain Admins” Security Group: Right-click user account > Properties > Member Of tab > Add > input Domain Admins > click Check Names > OK > Apply > OK
- Log out/close the Remote Desktop connection to DC-1 as the labuser and log back in as “mydomain.com\jane_admin”
- Use jane_admin as your admin account from now on<br />
<br />
<br />

<h3>Step 7: Joining Windows 10 VM (Client-1) to the domain</h3>

Right now, the DNS settings for Client-1 are pointing to the DNS server in the VNet that it was assigned to. We need its DNS settings to point to the DNS server in the DC-1 Domain Controller so that we can successfully join it to the domain.<br/>
<br/>

![image](https://github.com/simoneburch/config-ad/assets/152559137/d2a8e6ce-f1b6-43d8-b9be-128a2cc453ed)


- So, from the Azure Portal, set Client-1's DNS settings to DC-1's Private IP address: get (copy) the DC-1 private IP > go to Client-1 > Network Settings > NIC > DNS servers > set to Custom and paste DC-1s private IP (no spaces around it). Save.<br/>
- Restart Client-1 in the Azure Portal (this will flush the DNS cache of the old VNet/DNS server settings to make room for the fresh DC-1 DNS server settings).
<br/>
<br/>

![image](https://github.com/simoneburch/config-ad/assets/152559137/d37ae7af-1380-4011-b93c-ee654e34d291)


- Remote in to Client-1 as the original labuser account (You can double-check DNS settings in the Command Prompt with ipconfig /all to make sure it points to the DC-1 private IP. Also, ping that IP for connectivity)<br/>
<br/>
<br/>

![image](https://github.com/simoneburch/config-ad/assets/152559137/a0fef272-a772-46f3-b251-dc2a59d5dfd3)


- Join Client-1 to the domain: right_click the Windows icon > System > Rename this PC(advanced) > Change... > select Domain, input mydomain.com, OK > enter mydomain\jane_admin account name and password, OK > Welcome popup window, OK > Restart window, OK<br/>
<br/>
<br/>

<h3>Step 8: Set up Remote Desktop for all Non-Admin users on Client-1</h3>

<img src="https://github.com/simoneburch/config-ad/assets/152559137/1a04227f-045e-4a17-9f02-dea802b7ab00" width="75%" height="75%"/>


- Remote log in to Client-1 as mydomain.com\jane_admin
- Right-click the start menu > open System properties > click Remote Desktop on right panel > scroll down to User Accounts > choose Select Users That Can Remotely Access This PC > click ADD on the pop-up > type Domain Users in the input box > click Check Names for that group in the system (it will auto-populate if it is) > click OK
- Allow “MYDOMAIN\Domain Users” access to Remote Desktop by clicking OK
- You can now log into Client-1 as a normal, non-administrative user <br/>
<br/>

__**Now anyone belonging to this Domain Users group can use Remote Desktop to log in to Client-1.__ <br/>

<br />
<br />

<h3>Step 9: Creating additional users using a PowerShell script</h3>

![image](https://i.imgur.com/C55Oiwa.jpg)


- Log in to DC-1 VM as jane_admin and open Powershell ISE as an administrator 
- Create a new file in PowerShell ISE and paste the contents of this script into it: https://github.com/simoneburch/ps-adusers/blob/main/accts_creation_psscript
<br/>

  **PLEASE NOTE: When executing the PowerShell script: 1) Make sure you are doing it on Windows Server VM, not the Client VM. It won't work if you do. 2) Make sure your "_EMPLOYEES" OU within Active Directory matches the OU in your script. If it doesn't match, you will get errors.**
<br/>
<br/>

- Run the script and observe the accounts being created
- When all accounts have been created, open Active Directory Users and Computers. Notice the accounts have populated into the _EMPLOYEES Organizational Unit
- Choose any one of those names and remote in as a different user to Client-1 (i.e. mydomain.com\lalo.ruga) and the password from the script :)
- It works! You can go into the command prompt and observe that your new name is there (i.e. C:\Users\lalo.ruga>). GO into the file folders > This PC > C: Drive > Users folder > a new file for your user has been created. <br/>
<br/>
<br/>

<h3>Step 10: User Account Solutions/Examples</h3>

![image](https://github.com/simoneburch/config-ad/assets/152559137/70312b74-e6b9-4cf4-ac72-00ff13b66535)


- Remote into DC-1 as jane_admin and go to the _EMPLOYEES OU. Choose a user and right-click for properties. 
- In the Account tab you can see a checkbox to Unlock Account. This can be used when a user logs in too many times or otherwise becomes locked out of their account.<br/>
<br/>
<br/>
<br/>

![image](https://github.com/simoneburch/config-ad/assets/152559137/d78e731f-6377-49ba-b585-40bbea8db840)


- You can reset the password by also right-clicking the user's name. The pop-up window gives New Password inputs and other account options.<br/>
<br/>
<br/>
<br/>

![image](https://github.com/simoneburch/config-ad/assets/152559137/07a4f0a3-d2b2-4d53-88fc-21143069e5b8)


- Disabling/Enabling the account from here is also possible by just right-clicking the name and choosing Disable Account. There is an icon that appears on the account name to indicate this option.
<br/>

__**Continue looking around since this is where managing permissions and managing access to network resources will take place in an enterprise environment.__<br/>
<br />
<br />


<h3>:)</h3>

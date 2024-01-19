<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial explains the installation, configuration, and creation of different users in the on-premises Active Directory using Azure Virtual Machines.<br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 Pro

<h2>Deployment and Configuration Steps</h2>

__Step 1: Creating the Domain Controller VM__

![image](https://i.imgur.com/zmUeU34.jpg)


- Navigate to https://portal.azure.com/ and create a VM > label it as DC-1 (the domain controller)
- When creating the VM, select "Windows Server 2022" image with 2 vCPUs
- Configure it to 2 vCPUs
- Make note of the username and password for later use
- Click on review and create
- Once created, go to the networking tab > click on the "Network Interface" > IP configurations > ipconfig1 and make sure the Allocation button is set to "static". Save <br />
<br />
<br />

__Step 2: Creating the Client VM__

<img src="https://i.imgur.com/EmWrF9P.jpg" width="65%" height="65%"/>


- Create a new VM and label it as Client-1
- Use Windows 10 Pro with 2 vCPUs
- Make sure to use the same resource group and VNet that the DC-1 VM is using 
- Check that both VMs are in the same VNet by clicking on the "Virtual Machines" in the Azure Portal <br />
<br />
**PLEASE NOTE: the DC-1 vnet may take a few minutes to load before you can select it. Just be patient or refresh the page until that option is available to choose. <br />
<br />
<br />

__Step 3: Testing VM Connectivity__

<img src="https://i.imgur.com/SFzICM7.jpg" width="75%" height="75%"/>


- Log in to Client-1 and ping DC-1's IP address using the ping -t command:
- Copy DC-1's private IP address from the Azure portal
- Navigate to the Command Prompt by looking up "CMD" in the search bar
- In the Command Line type: ping -t __DC-1 private IP__ (ping -t __10.0.0.4__)
- You should see "Request timed out." showing that no connection is being established otherwise we'd get a response from 10.0.0.4 (DC-1) <br />
<br />
<br />

__Step 4: Enable local firewall__

![image](https://i.imgur.com/bJOj3i5.jpg)


- To open that firewall connection, lets go to DC-1 VM via remote desktop
- Once the DC-1 VM is open, type wf.msc to the search bar
- Sort by protocol and find ICMPv4
- Enable the rule for the two rules that state "Core Networking Diagnostics ICMP"
- Go back to client-1 VM and see it ping and display a connection between client-1 and DC-1
- Hitting control and C in the CMD it will stop the constant pinging <br />
<br />
<br />

__Step 5: Installing Active Directory__

![image](https://i.imgur.com/T18uPM9.jpg)


- Now go to DC-1 VM and under server manger go to add roles and features
- Make sure under server roles you select "Active Directory Domain Services"
- After installation is complete, you got to the flag in the top right and hit promote to finalize the process
- Hit add a new forest and label it as mydomain.com
- Password can be Password1
- After going through the installation, its going to restart and you have to reconnect to DC-1 VM again via remote desktop <br />
<br />
<br />

__Step 6: Creating Admin account, User account, and Organizational Units__

![image](https://i.imgur.com/rT8VXam.jpg)


- Since we turn the DC-1 VM as a domain controller when logging in we have to use username as "mydomain.com\labuser" or the \username created and the password is the same as it was created in the VM setup
- Login and hit the search for active directory users and computers
- Create two folders by right clicking mydomain.com folder and select new then organization unit.
- Label them _ADMINS and _EMPLOYEES
- Create a user called jane doe and make the password to never expire then right click on the user and hit properties and under member of tab then hit add and type domain and check names from there select domain admins and apply
- Log out of DC-1 and log back in as mydomain.com\jane_admin
- Once logged as Jane, you can look at the CMD and type "whoami" to show that you are logged as jane_admin <br />
<br />
<br />

__Step 7: Joining Windows 10 VM (Client-1) to the domain__

![image](https://i.imgur.com/AX8AtuP.jpg)


- Go to Azure and copy DC-1 private IP address then go to client-1 and click network interface under networking then DNS server tab and hit custom and paste DC-1 private IP address there
- Once clicking save is done, we can restart client-1 in Azure
- Relog into client-1 go to CMD and ipconfig/all you can then see the DNS server IP address is the same as DC-1 private IP
- Now we can right-click on start and hit system from there hit rename this PC(advanced) then change and check domain and type out the domain as mydomain.com then type the username and password as mydomain.com\jane_admin and password as the one created in DC-1
- After a restart of client-1 VM, as client 1 is a member of the domain we can login as mydomain.com\jane_admin
- Once login go to system then remote desktop and select users at the bottom 
- Hit add then type "domain users" and hit check names then ok
- Now go back to DC-1 VM and open active directory users and computers
- Click on the "computers" folder
- Verify Client-1 shows up in there <br />
<br />
<br />

__Step 8: Set up Remote Desktop for all Non-Admin users on Client-1__

<img src="https://github.com/simoneburch/config-ad/assets/152559137/1a04227f-045e-4a17-9f02-dea802b7ab00" width="75%" height="75%"/>

- Remote log in to Client-1 as mydomain.com\jane_admin
- Right-click the start menu > open System properties > click Remote Desktop on right panel > scroll down to User Accounts > choose Select Users That Can Remotely Access This PC > click ADD on the pop-up > type Domain Users in the input box > click Check Names for that group in the system (it will auto-populate if it is) > click OK
- Allow “MYDOMAIN\Domain Users” access to Remote Desktop by clicking OK
- You can now log into Client-1 as a normal, non-administrative user <br/>
<br/>
**Now anyone belonging to this Domain Users group can use Remote Desktop to log in to Client-1 <br/>
<br />
<br />

__Step 9: Creating additional users using a PowerShell script__

![image](https://i.imgur.com/C55Oiwa.jpg)


- Log in to DC-1 VM as jane_admin and open Powershell ISE as an administrator 
- Create a new file in PowerShell ISE and paste the contents of this script into it: https://github.com/simoneburch/ps-adusers/blob/main/accts_creation_psscript
<br/>

  **!!! NOTICE !!! When executing the PowerShell script: 1) Make sure you are doing it on Windows Server VM, not the Client VM. It won't work if you do. 2) Make sure your "_EMPLOYEES" OU within Active Directory matches the OU in your script. If it doesn't match, you will get errors.**
<br/>
<br/>

- Run the script and observe the accounts being created
- When all acounts have been created, open Active Directory Users and Computers. Notice the accounts have populated into the _EMPLOYEES Organizational Unit
- Choose any one of those names and remote in as a different user to Client-1 (i.e. mydomain.com\lalo.ruga) and the password from the script :)
- It works! You can go into the command prompt and observe that your new name is there (i.e. C:\Users\lalo.ruga>). GO into the file folders > This PC > C: Drive > Users folder > a new file for your user has been created. <br/>
<br/>
<br/>

__Step 10: User Account Solutions/Examples__

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

- Disabling/Enabling the account from here is also possible by just right-clicking the name and choosing Disable Account. There is an icon that appears on the account name to indicate this option.<br/>
<br />
<br />


**The tutorial ends here but this is a great starting point in getting familiar with Active Directory :)

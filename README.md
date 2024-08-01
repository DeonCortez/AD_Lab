# AD_Lab
## Tutorial: Setting Up a Basic Home Lab with Active Directory on Oracle VirtualBox and Connecting a Client Machine

### Overview
This tutorial provides a comprehensive guide to setting up a home lab environment using Oracle VirtualBox to run Active Directory and includes steps to configure a Windows 11 client machine to connect through the domain controller. Additionally, we will use PowerShell to automate the creation of 1,000 users from a list of names stored in a text file.

### Prerequisites
- Oracle VirtualBox installed on your host machine
- Windows Server ISO file
- Windows 11 ISO file
- Basic knowledge of networking, Windows Server, and Windows client administration

### Step-by-Step Instructions

#### 1. **Set Up the Virtual Machines**

##### 1.1 **Create the Domain Controller VM**
1. **Open Oracle VirtualBox** and click "New" to create a new virtual machine.
2. **Name your VM** (e.g., "DomainController") and select "Microsoft Windows" as the type and "Windows Server" as the version.
3. **Allocate memory** (at least 2 GB recommended) and create a virtual hard disk (dynamically allocated or fixed size, minimum 40 GB).

##### 1.2 **Install Windows Server**
1. **Attach the Windows Server ISO** to your VM by going to "Settings" -> "Storage" and selecting the ISO file for the virtual CD/DVD drive.
2. **Start the VM** and follow the installation prompts to install Windows Server. Choose the appropriate edition (e.g., Standard) and complete the setup.

##### 1.3 **Create the Client Machine VM**
1. **Repeat the process** to create a new virtual machine for Windows 11. Allocate memory (at least 4 GB recommended) and create a virtual hard disk (minimum 40 GB).

##### 1.4 **Install Windows 11**
1. **Attach the Windows 11 ISO** to the VM and follow the installation prompts to complete the setup.

#### 2. **Configure Networking**

##### 2.1 **Set Up Network Adapters**
1. **Domain Controller VM:**
   - Open "Settings" for the Domain Controller VM and go to "Network."
   - **Add two network adapters:**
     - **Adapter 1**: Set to "NAT" or "Bridged Adapter" to access the public internet.
     - **Adapter 2**: Set to "Internal Network" to communicate with the client machine.

2. **Client Machine VM:**
   - Open "Settings" for the Client Machine VM and go to "Network."
   - **Add one network adapter:**
     - Set to "Internal Network" to communicate with the Domain Controller.

##### 2.2 **Configure IP Addresses**
1. **Domain Controller:**
   - Open "Network and Sharing Center" -> "Change adapter settings."
   - Configure Adapter 1 with an IP address that can access the public internet.
   - Configure Adapter 2 with a static IP address (e.g., 192.168.0.1).

2. **Client Machine:**
   - Open "Network and Sharing Center" -> "Change adapter settings."
   - Configure the network adapter with a static IP address (e.g., 192.168.0.2) and set the DNS server to the IP address of the Domain Controller (192.168.0.1).

#### 3. **Configure Active Directory**

1. **Log in to the Domain Controller** and open the "Server Manager."
2. **Add Roles and Features** by selecting "Manage" -> "Add Roles and Features." Follow the wizard to install the "Active Directory Domain Services" role.
3. **Promote the server** to a domain controller by clicking on the notification in Server Manager and following the prompts to configure a new forest and domain.

#### 4. **Join the Client Machine to the Domain**

1. **Log in to the Windows 11 client machine.**
2. **Open "Settings" -> "System" -> "About"** and click "Join a domain" under "Related settings."
3. **Enter the domain name** you configured (e.g., `yourdomain.local`) and follow the prompts to join the domain. Restart the client machine as needed.

#### 5. **Create Users with PowerShell**

##### 5.1 **Prepare the User List**
1. **Create a text file** named `UserList.txt` with each line containing a first and last name separated by a space (e.g., `John Doe`).

##### 5.2 **PowerShell Script to Create Users**
1. **Open PowerShell** on the Domain Controller with administrative privileges.
2. **Use the following script** to create users:
   ```powershell
   # Import the list of names
   $userList = Get-Content -Path "C:\Path\To\UserList.txt"

   # Loop through each line in the list
   foreach ($user in $userList) {
       # Split the line into first and last names
       $nameParts = $user -split " "
       $firstName = $nameParts[0]
       $lastName = $nameParts[1]

       # Create the user
       $username = "$firstName$lastName"
       $password = ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force

       New-ADUser -Name "$firstName $lastName" `
                  -GivenName $firstName `
                  -Surname $lastName `
                  -UserPrincipalName "$username@yourdomain.local" `
                  -Path "OU=Users,DC=yourdomain,DC=local" `
                  -AccountPassword $password `
                  -Enabled $true
   }
   ```

3. **Replace** `"C:\Path\To\UserList.txt"` with the path to your `UserList.txt` file and update the domain and organizational unit (OU) details as needed.

#### 6. **Verify User Creation**

1. **Use PowerShell** to verify user creation:
   ```powershell
   Get-ADUser -Filter * | Format-List Name, UserPrincipalName
   ```

### Conclusion
This tutorial outlines the process of setting up a basic home lab with Oracle VirtualBox, configuring Active Directory, and connecting a Windows 11 client machine. Additionally, it demonstrates using PowerShell to automate the creation of 1,000 users from a text file.

Feel free to reach out for any questions or further details!

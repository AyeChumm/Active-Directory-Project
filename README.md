# Active-Directory-Project

## Objective
This project aims to set up and configure a lab environment to simulate real-world attack and defense scenarios. The environment will include installing and configuring Windows Server 2022, Splunk Server (Ubuntu), Windows 10, and Kali Linux VM. Key tasks will involve configuring Splunk and Sysmon for logging, setting up Active Directory, and promoting the Windows Server to a domain controller. Additionally, a brute force attack will be executed using Kali Linux on a domain user account as well as using Atomic Red Team to generate telemetry with analysis conducted through Splunk.

## Tools
- VirtualBox as the hypervisor
- Ubuntu 22.04.5 LTS (Jammy Jellyfish) as the Splunk Server
- Windows 10 as the target machine
- Windows Server 2022 as the Active Directory
- Kali Linux to perform the brute force attack

## Logical Network Diagram
![Network Diagram](https://github.com/user-attachments/assets/7111c156-324e-419f-b68f-5ccc47f523b7)
<br clear="left"/> A logical diagram helps to understand how data might flow, shows the relationship between different components, and overall gives a clear visual representation of the network
- Network: 192.168.10.0/24 | Splunk Server: 192.168.10.10 | Active Directory: 192.168.10.11 | Kali Linux: 192.168.10.250 | Windows 10: DHCP
- Splunk Universal Forwarder will be installed on both Active Directory and Windows 10
- Dotted green lines = data logs being ingested into the Splunk Server

## Steps
1. Create and install all 4 VM (Ubuntu 22.04 LTS, Windows 10, Kali Linux, and Windows Server 2022)

### Create a NAT Network for all VMs
   ![NAT network](https://github.com/user-attachments/assets/9e16cd9c-9fc0-42b0-82e9-0b9e448b861c)
   1. Go to tools in Virtual Box and open Network and click NAT Network
   2. The network will be "Active Directory Project" and the IPv4 prefix will be 192.168.10.0/24 (as seen in the logical diagram).
   3. Click Apply

  ![NAT network for all VM](https://github.com/user-attachments/assets/ca731f13-35e1-468e-8a14-2602b5fdaad4)
   1. Go to settings for the Ubuntu's VM and into the Network section
   2. In Adapter 1 change "Attached to" to the new network "NAT Network". Name should change "Active Directory Project"

### Create Splunk Server using Ubuntu

#### Change IP and check connectivity
1. Statically configure the IP for the Splunk server to the one used in the logical diagram (192.168.10.10)
- Check the current IP of the Splunk Server
```bash
  ip a
```
![1 Splunk checking IP](https://github.com/user-attachments/assets/5c118065-f455-4881-85f4-262e5a4a9f2b)

2. Open and edit the network configuration file accordingly
```bash
sudo nano /etc/netplan/00-installer-config.yaml
```
![2 Splunk changed IP ](https://github.com/user-attachments/assets/e14c8d89-3093-4bff-8491-ffb8e843badd)
- Change DHCP from True to no/false
- Add "addresses" with the desired IP for the Splunk Server: 192.168.10.10/24
- Add "nameservers" for the DNS. I used google's: 8.8.8.8
- Add "routes -to via:" for the default gateway: 192.168.10.1
- Exit and save file

3. Apply the network configuration changes:
```bash
sudo netplan
```
4. Check for IP change and connectivity
   
   ![3 splunk IP successfully changed](https://github.com/user-attachments/assets/43c8c647-0669-4cfa-b2f7-528391751c46)
   - Static IP has successfully changed

  ```bash
  ping google.com
  ```
  ![4 pinging google](https://github.com/user-attachments/assets/7a68e1b7-ca43-42e7-9ae3-bd54249cb69e)
  - Connectivity is working

#### Install Splunk

1. On the <b>Host machine</b> go to the Splunk website and download the Splunk Enterprise _.deb_ file
   ![5 splunk deb file](https://github.com/user-attachments/assets/a7f6dc88-1d4b-4ec1-addc-3ca7a9cbb0f6)

2. Install _Guest-Add-on_ for VirtualBox in the Splunk VM
    ```bash
   sudo apt-get install virtualbox-guest-additions-iso
   ```
3. In VirtualBox's menu tab, click "Devices, Shared Folders, then Shared Folders Settings..."
   ![6 Shared Folder](https://github.com/user-attachments/assets/81de9a8d-38fe-4f17-948e-320a47a73bc3)

   - Make a new folder
   <br clear="left"/> ![7 add new folder](https://github.com/user-attachments/assets/3172be75-e513-4c60-868e-b2b1a00ff678)

   - Find the file path of the _.deb_ file and select "Read-only, auto-mount, Make Permanent". Click ok.
   ![8 shared folder settings](https://github.com/user-attachments/assets/ab0a10cc-327e-4afe-af22-dc69231f3dba)

4. Reboot Splunk Server:
   ```bash
   sudo reboot
   ```
   
5. Add user into VirtualBox for Splunk Server
   ```bash
   sudo adduser user vboxsf
   ```
   
6. Make a new directory called "share"
   ```bash
   mkdir share
   ```

7. Mount the "shared folder" onto the new directory called "share"
   ```bash
   sudo mount -t vboxsf -o uid=1000,gid=1000 VirtualBOX share/
   ```
   ![9 shared folder mounted](https://github.com/user-attachments/assets/b526dcd3-a7f5-47a1-847d-457ebb43c33f)
   - check that the "share" directory is there
     ```bash
     ls -la
     ```

8. Go to "share" folder
   ```bash
   cd share
   ```

9. Install Splunk in share folder:
   ```bash
   sudo dpkg -i Splunk (.deb filename)
   ```
   - Change directory to /opt/splunk
   - Change into "user":
     ```bash
     sudo -u splunk bash
     ```
   - Go to bin directory (where all binaries are stored)
   - Start Splunk installment
     ```bash
     ./splunk start
     ```

10. Set up Splunk to start up every time the Ubuntu VM starts up
    ![10 Splunk is on when reboot](https://github.com/user-attachments/assets/be3278c1-68a3-4bfd-987c-88ec2b345707)
    - Gets out of splunk@splunk and back to user@splunk
    ```bash
      exit
      ```
      
    - Go to bin directory
      ```bash
      sudo ./splunk enable boot-start -user splunk
      ```
      Splunk will start up as user@splunk every time VM is turned on

#### Install Splunk Universal Forwarder For Both Target Machine and Windows 2022 Server VM

1. Change Windows 10's PC name to Target-PC
   ![1 change pc name to target](https://github.com/user-attachments/assets/e1555e9d-8968-4605-aaa7-100fc80fc45d)

2. Manually configure Windows 10's IP
  ![2 Target-PC IP has changed png](https://github.com/user-attachments/assets/dc6673f8-6a23-4109-ac33-f23cfb265478)
- Open network and internet settings and configure the following:
- IP to 192.168.10.100 (any ip in the network will do)
- Subnet mask 255.255.255.0 (/24)
- Default gateway: 192.168.10.1
- Preferred DNS server: 8.8.8.8 (google.com)

Check the IP again in the terminal
   ```bash
   ip a
   ```
  ![3 target-pc IP has changed](https://github.com/user-attachments/assets/516ee556-1dab-483a-9128-2b7e8988b9ac)

3. Head over to Splunk's website and in products, download the Universal Forwarder for windows
   ![4 download universal forwarder](https://github.com/user-attachments/assets/a3e3bdbe-a1de-43a3-9df6-0794c2123602)

4. After downloading, find the file and install
   <br clear="left"/>![5 splunk forwarder](https://github.com/user-attachments/assets/abf1a91b-abe7-4b91-9061-cb61b3204974)
   - Make sure the Receiving Index is 192.168.10.10 the Splunk Server using port 9997
  
#### Install Sysmon For Both Target Machine and Windows Server 2022 VM

1. In the browser, search up Sysmon and download it. After downloading, extract the files.
   <br clear="left"/>![6 download sysmon](https://github.com/user-attachments/assets/cc52f9be-db5c-4263-81a1-6044a91f0f3d)

2. Search up and download the configuration files for Sysmon (I will be using Olafhartong's configuration files on github)
   <br clear="left"/>![7 sysmon raw](https://github.com/user-attachments/assets/2b0ec7ad-f4b3-44ff-8de3-a2a393d27dd4)
   - Scroll down to sysmonconfig.xml
   - Click raw and save as a download

3. Open PowerShell as administrator
   
4. Install Sysmon with the configurations:
   - Go into the location of the Sysmon file
   ```bash
   cd (filepath to sysmon files)
   ```
   - Install Sysmon with config files
   ```bash
   ./Sysmon64.exe -i ..\ sysmonconfig.xml
   ```

5. Create a new file called input.conf to tell the forwarder what type of data to send to the Splunk server. It will located in C: programfiles/SplunkUniversalForwarder/etc/system/local
   ![8 only create folder with admin](https://github.com/user-attachments/assets/6487f7d4-8d78-4084-8c30-9e4453f348f3)
   - Only allows the creation of a folder and not file
  
   - Open Notepad as admin and use the following configuration:
     ```bash
     [WinEventLog://Application]
      index = endpoint
      disabled = false

      [WinEventLog://Security]
      index = endpoint
      disabled = false

      [WinEventLog://System]
      index = endpoint
      disabled = false

      [WinEventLog://Microsoft-Windows-Sysmon/Operational]
      index = endpoint
      disabled = false
      renderXml = true
      source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
     ```
   - Save the file as "inputs.conf" and for all file types
     <br clear="left"/>![10 notepad filepath](https://github.com/user-attachments/assets/f0980cc0-9801-4d8d-a220-621359d2b811)

   - Open Services as admin from the Windows search bar and find SplunkForwarder service
     ![11 Splunk services restart](https://github.com/user-attachments/assets/243d2522-1cea-4e02-89ae-618bb35f37f1)
     - Check that the "Log on as" tab is set to "Local System Account" (The default NT Service setting may disrupt Splunk from collecting logs due to permissions)
     - Right-click SplunkForwarder and restart the service to apply changes
    
 #### Check Splunk Ingestion

1. Log into Splunk using a web browser
   ```bash
   192.168.10.10:8000
   ```
   
2. Create an index named "endpoint"
   <br clear="left"/>![12 endpoint index ](https://github.com/user-attachments/assets/b580f375-e21a-47ec-8a67-4db150ef62dd)
   - In the inputs.conf File for Sysmon, it tells SplunkForwarder to send data to index=endpoint
   - Go to Settings and select "Index"
   - Select "New Index" and add endpoint
  
3. Enable Splunk Server to receive data
   - Go to Settings and select "Forwarding and Receiving" then "Configure Receiving"
     ![13 Splunk receiving data](https://github.com/user-attachments/assets/bc59fa5f-ce39-4c23-80a5-f68836ff61cf)

   - Use Port 9997 (same port when installing the forwarder)
     ![14 port 9997](https://github.com/user-attachments/assets/7bdff5d2-fe1b-4bd7-8ed6-69a8a4261d35)

4. Confirm both target machine and server are forwarding data into Splunk querying:
   ```bash
   index=endpoint
   ```
   ![15 splunk forwarder and sysmon on server and target](https://github.com/user-attachments/assets/6bdf83f4-aa76-482d-9d0e-ac6db3269cfe)

### Install and Configure Active Directory Onto Windows Server 2022

1. Open Server Manager
   ![1 server Manager settings 1](https://github.com/user-attachments/assets/5a38b99d-ca5f-44e9-8f6d-d87c67f020c7)
   - Select "Manage" and "Add Role and DF
   - For Installation Type select "Role-based or feature-based installation"

  ![2 server manager server roles](https://github.com/user-attachments/assets/a8892c1f-f065-43ac-8b82-da34af73b029)
  - For Server Roles select "Active Directory Domain Service" (AD DS) and "Add Feature"
  - Leave everything else as is and install

2. Promote to a Domain Controller
   <br clear="left"/>![3 promoting to domain controller](https://github.com/user-attachments/assets/3dd7be95-77f7-4ab5-9a14-63727823b829)
   - In Server Manager select the flag icon on top
   - Select "Promote this server to a domain controller

3. Deployment Configuration
   <br clear="left"/>![4 add a new forest](https://github.com/user-attachments/assets/46a11fdf-511f-4001-b5dd-4eedfa10170f)
   - Select "Add a new forest"
   - This adds a new domain. There are currently no domains
  
   ![5 attackers target these for active directory](https://github.com/user-attachments/assets/a7065bb3-011c-49a4-b81a-ab5b32028f1c)
   - These files contain everything related to Active Directory making them a frequent target for attackers
  
   - Leave everything else as default and install
   - Log back in
     <br clear="left"/>![6 ADDS installed](https://github.com/user-attachments/assets/8e464df5-b8c9-4ae8-aec1-9d3c3e785ed5)
     - The "\" indicates AD-DS has successfully been installed and the server has been promoted to a domain controller

#### Create a User for Active Directory

1. Go to "Tools" and select "Active Directory Users and Computers"
   <br clear="left"/>![7 create obj,user, groups etc](https://github.com/user-attachments/assets/bde73f0b-d6be-4c04-a124-2d808a834bae)

2. Select "admin local", "New", then "Organizational Unit" for the IT department
   <br clear="left"/>![8 organizational unit](https://github.com/user-attachments/assets/3d555751-eac3-42c3-bbfe-346340578fa7)
   - In the real world, best practice is to create an organizational unit for different departments
  
3. In IT, right-click, select "New" then "User"
   ![9 creating new user](https://github.com/user-attachments/assets/1398b969-f6bd-4699-852d-864c860e0978)

4. This new user will be named John Doe
   <br clear="left"/>![10 user is created](https://github.com/user-attachments/assets/95e973fb-120c-425c-a96b-612f98294107)
   - User has successfully been made. John Doe of the IT department

#### Join Target Machine With Domain

1. In the target machine, type in PC in search bar and select "Properties"
   ![1  adding target to domain](https://github.com/user-attachments/assets/e4d6d46a-5bb2-4aa6-b1de-89690ed858cf)
   - Go to "Advance system settings", "Computer Name", "Change", and under "Member of" select "Domain" with the name of your ADDS
   - Click Ok
  
2. An error will occur
   <br clear="left"/>![2  error](https://github.com/user-attachments/assets/01cbb146-98f3-498a-8e90-39fb2828eb9e)
   - Must change DNS in Network Settings from 8.8.8.8 to the ADDC's IP 192.168.10.11

3. After changing DNS, try again and log in using the Domain Controller's account

4. To apply changes, you will be prompted to restart the machine

5. ![3  logging in as john doe](https://github.com/user-attachments/assets/e29cc0e4-e02a-4b88-b2c1-76b9711915e1)
   - Select "Other User"
   - Login as John Doe from the IT department when creating a Domain User

### Use Kali Linux to Perform a Brute Force Attack

#### Check Initial Kali Configurations

1. Manually configure a static IP as per the diagram
   ![1  kali changed IP](https://github.com/user-attachments/assets/679dc705-1622-4586-af63-1a56a47850f2)
   - IP: 192.168.10.250/24 | Default Gateway: 192.168.10.1 | DNS: 8.8.8.8
  
2. Disconnect from ethernet
   <br clear="left"/>![2  disconnect](https://github.com/user-attachments/assets/2010aa3d-fd1e-4741-9c2b-65fdcc530d69)
   - Reconnect to apply changes
  
3. Check new IP and connectivity
   - ```bash
     ip a
     ```
     ![3  IP has changed](https://github.com/user-attachments/assets/de636acb-4e1a-4c4a-b665-84e7d31111a3)
   - ```bash
     ping google.com
     ```
     ![4  ping google](https://github.com/user-attachments/assets/704f8f1f-6183-471f-9117-a5dc9bc4ca22)

4. Update and upgrade repository
   ```bash
   sudo apt-get update && sudo apt-get upgrade -y
   ```

#### Prepare Attack

1. Make a new directory ad-project
   ```bash
   mkdir ad-project
   ```
   - All files made for the attack will be placed here

2. Install Crowbar to perform the brute force attack
   ```bash
   sudo apt-get install crowbar -y
   ```

3. Find a file called rockyou.txt.gz
   ```bash
   cd /usr/share/wordlist
   ```
   - Rockyou.txt is a list of passwords commonly used in security testing that is built-in with Kali Linux

   - Unzip file
     ```bash
     Sudo gunzip rockyou.txt.gz
     ```

   - Copy file into the ad-project directory
     ```bash
     cp rockyou.txt ~/desktop/ad-project
     ```

4. Go to the ad-project directory
   ```bash
   cd ~/Desktop/ad-project/
   ```

5. Take the first 20 passwords from rockyou.txt and input them into a new file called passwords.txt
   ```bash
   head -n 20 rockyou.txt > passwords.txt
   ```

6. Open passwords.txt
   ```bash
   cat passwords.txt
   ```
   <br clear="left"/>![5  passwords](https://github.com/user-attachments/assets/fd559760-933b-44ad-9fce-dc9f9c14c740)


7. Edit the _.txt_ file and add the password used for the target
   ```bash
   nano passwords.txt
   ```
   <br clear="left"/>![6  nano password](https://github.com/user-attachments/assets/e9ea7eb5-51df-4ae1-aa3c-5d91db559770)

#### Enable Remote Desktop Protocol (RDP) on the Target Machine

1. Type in "PC" in the search bar, select "Properties", then "Advanced system settings"
   ![7  advanced system settings](https://github.com/user-attachments/assets/8683a51e-a376-4a8d-818f-6a2cb62c8ca9)

2. Configure Settings to enable RDP
   ![8  enabled rdp ](https://github.com/user-attachments/assets/be61d1af-8360-44a2-b739-e2daf2236bce)
   - Log in as Administrator
   - Go to the Remote tab and "Allow remote connections to this computer"
   - "Select Users", "Add...", then select the desired users
   - Select "Ok" to apply changes
  
#### Perform Brute Force Attack

1. Use Crowbar to perform a brute force attack on target machine
   ```bash
   crowbar -b rdp -u jdoe-C password.txt -s 192.168.10.100/32
   ```
   ![9  brute force attack](https://github.com/user-attachments/assets/1ba2bde2-26a4-4ba5-a3f7-238f90abc2ca)
   - b: refers to the delivery of attack
   - u: specific user
   - C: specifies the password.txt file
   - s: the source IP
   - /32: to target only 1 IP
  
#### See Telemetry in Splunk

1. In "Search and Reporting" app query:
   ```bash
   index=endpoint jdoe
   ```
   ![10  event codes](https://github.com/user-attachments/assets/14a8cd44-5ae4-4d9d-80ff-2a1d5e0b0002)
   - EventCode 4624:an account was successfully logged on
   - EventCode 4625: an account failed to log on
   - There were only 21 passwords in the passwords.txt file including the 1 correct password (20 failed attempts, 1 successful attempt)
  
### Atomic Red Team

#### Exclusion for C Drive
- Microsoft Defender will detect and remove some files from Atomic Red Team if not excluded

1. Open Windows Security and select "Virus and threat protection"
   ![2  ](https://github.com/user-attachments/assets/8994f118-3b95-4995-a1f9-7df1382556b1)

2. Select "Manage settings"
   <br clear="left"/>![3](https://github.com/user-attachments/assets/edf54426-dc3e-4da4-b78f-04219be9ccce)

3. Scroll down to "Exclusions" and select "Add or remove exclusions"
   <br clear="left"/>![4](https://github.com/user-attachments/assets/acf18d50-cecb-4a60-968a-6f546b567c54)

4. Add an exclusion, select "Folder", then select C Drive to exclude it
   <br clear="left"/>![5](https://github.com/user-attachments/assets/876bb180-c828-4bcd-bf70-cc2d8ef156c3)

#### Install Atomic Red Team (ART)

1. Open Powershell as admin
   ```bash
   Set-ExecutionPolicy Bypass CurrentUser
   ```
   ![1  set exclusion](https://github.com/user-attachments/assets/c0a5ff2c-d80a-4aa0-93fb-1396c8b650e8)

   ```bash
   IEX (IWR -UseBasicParsing); Install-AtomicRedTeam -getAtomics
   ```
   - installs ART
  
2. Go to the C Drive and open atomicredteam/atomics
   <br clear="left"/>![7](https://github.com/user-attachments/assets/baae5a5b-9f74-4631-9f9c-bc5c385fa802)
   - I will use T1136.001 = creates a local account
  
     - These codes are the tactical codes associated with the MITRE ATT&CK Framework
       ![8](https://github.com/user-attachments/assets/092f195e-6732-46cf-ad70-204d75661cca)

3. Generate telemetry with ART:
   ```bash
   Invoke-AtomicTest T1136.001
   ```
   ![9  new localuser](https://github.com/user-attachments/assets/fb9daf2e-1ed1-411b-bb86-0191d6813916)
   - NewLocalUser has been created
  
#### Check In Splunk for NewLocalUser

1. Query
   ```bash
   index=endpoint Newlocaluser
   ```
   ![10 localuser found](https://github.com/user-attachments/assets/cb46525b-e605-48c1-a543-871859be3cb2)
   - There is a new local account by the name of NewLocalUser


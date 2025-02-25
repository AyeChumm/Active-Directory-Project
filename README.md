# Active-Directory-Project

## Objective
This project aims to set up and configure a lab environment to simulate real-world attack and defense scenarios. The environment will include installing and configuring Windows Server 2022, Splunk Server (Ubuntu), Windows 10, and Kali Linux VM. Key tasks will involve configuring Splunk and Sysmon for logging, setting up Active Directory, and promoting the Windows Server to a domain controller. Additionally, a brute force attack will be executed using Kali Linux on a domain user account as well as Atomic Red Team to generate telemetry with analysis conducted through Splunk.

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
   ![7 add new folder](https://github.com/user-attachments/assets/3172be75-e513-4c60-868e-b2b1a00ff678)

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
    - ```bash
      exit
      ```
      Gets out of splunk@splunk and back to user@splunk
    - Go to bin directory
    - ```bash
      sudo ./splunk enable boot-start -user splunk
      ```
      Splunk will start up as user@splunk every time VM is turned on

### Install Splunk Universal Forwarder

1. Change Windows 10's PC name to Target-PC
   ![1 change pc name to target](https://github.com/user-attachments/assets/e1555e9d-8968-4605-aaa7-100fc80fc45d)

2. Manually configure Windows 10's IP




   







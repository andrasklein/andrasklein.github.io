---
layout: post
title: Building an Office Themed Home Lab
date: 2025-01-20 00:34:00 +0800
categories: [Active Directory, VMware, Home Lab,]
tags: [Active Directory, Domain Controller, Group Policy, Windows Server, User Management]
author: klein
image:
  path: /images/images-office/image001.png
  
---

# Active Directory Lab Build Documentation

## Lab Requirements

For the Active Directory lab build, the following setup is used:
- **1 Windows Server 2022**
- **2 Windows 10 Workstations**

### System Requirements
- **Disk Space**: 60GB
- **RAM**: 16GB

---

## Step 1: Download Necessary Files

To build the lab, we need to download the required ISO files.

![b](/images/images-office/image01.png)
![b](/images/images-office/image02.png)
![b](/images/images-office/image03.png)
![b](/images/images-office/image04.png)


1. Go to the [Microsoft Evaluation Center](https://www.microsoft.com/en-us/evalcenter/) to download the ISO files for:
   - Windows Server 2022
   - Windows 10 Workstations (64-bit editions)

2. Fill out the trial form to register. 
   - **Note**: You can input any information, as the form will work regardless of accuracy.

3. Download both ISO files.

![b](/images/images-office/image05.png)
![b](/images/images-office/image06.png)

---

## Step 2: Setting Up the Domain Controller

The domain controller will be configured using **Windows Server 2022**. Follow these steps to set up the virtual machine:


1. **Create a New Virtual Machine**:
   - Use your preferred virtualization platform.
   - Select the downloaded Windows Server 2022 ISO file.


![b](/images/images-office/image1.png)
![b](/images/images-office/image2.png)


2. **Allocate Storage**:
   - Choose a drive that provides enough storage to meet the lab requirements.
   - Split the virtual disk into multiple files to allow it to grow as you add more files later.

![b](/images/images-office/image3.png)
![b](/images/images-office/image4.png)


3. **Edit VM Settings**:
   - Allocate sufficient RAM for the virtual machine to ensure smooth performance.

![b](/images/images-office/image5.png)


---

## Step 3: Configuring the Domain Controller

1. **Power On and Boot the Virtual Machine**:
   - Power on the virtual machine and press any key to boot.
   - Go through the setup procedure, keeping most options set to defaults.
   - Select **Standard Evaluation (Desktop Experience)**.
   - Choose a **custom install** and allocate the appropriate partitions.
   - At the end of the setup, set a password for the **Administrator** account.

![b](/images/images-office/image6.png)
![b](/images/images-office/image7.png)


2. **Install VMware Tools**:
   - To make the screen full-size, install **VMware Tools** and follow the on-screen instructions.

![b](/images/images-office/image8.png)


3. **Rename the Computer**:
   - Rename the computer to a unique name.  
     - Example: **SCRANTON-DC** (this lab uses an OFFICE theme, and this machine will act as the Domain Controller).  
   - Reboot the virtual machine after renaming.

![b](/images/images-office/image9.png)
![b](/images/images-office/image11.png)



4. **Set Up as Domain Controller**:
   - After the reboot, open **Server Manager** and follow these steps:
     1. Select **Add Roles and Features**.
     2. Choose **Role-based or feature-based installation**.
     3. Select **Active Directory Domain Services**.
     4. Check the option to **Restart the server automatically if required**.
     5. Select **Install**.
     6. Once installation is complete, select **Promote this server to a domain controller**.

![b](/images/images-office/image12.png)
![b](/images/images-office/image13.png)
![b](/images/images-office/image14.png)
![b](/images/images-office/image15.png)
![b](/images/images-office/image16.png)
![b](/images/images-office/image17.png)


5. **Create a New Forest**:
   - Select **Add a new forest**.
   - Choose a domain name (e.g., **OFFICE.local**).
   - Enter a password (this can be the same as the Administrator password).
   - Select **Install**.  
     - The system will automatically reboot.

![b](/images/images-office/image18.png)
![b](/images/images-office/image19.png)
![b](/images/images-office/image20.png)
![b](/images/images-office/image21.png)
![b](/images/images-office/image22.png)


6. **Configure Active Directory Certificate Services**:
   - After logging in with the previously set credentials:
     1. Open **Server Manager** and select **Add Roles and Features**.
     2. Click **Next** until the **Active Directory Certificate Services** option is available.
     3. Select **Add Features**.
     4. Check the option to **Restart the destination server automatically if required**.
     5. Choose **Configure Active Directory Certificate Services on the destination server**.
     6. Select the **Certification Authority** option.
     7. Go through the default settings and select **Configure**.

![b](/images/images-office/image23.png)
![b](/images/images-office/image24.png)
![b](/images/images-office/image25.png)
![b](/images/images-office/image26.png)
![b](/images/images-office/image27.png)
![b](/images/images-office/image28.png)
---

## Step 4: Setting Up the Workstation Machines

1. **Create Virtual Machines for Workstations**:
   - Use the ISO file for **Windows 10**.
   - Go through the setup process and choose a name for each machine that matches your theme.  
     - Example: **DWIGHT** and **AND**.

![b](/images/images-office/image29.png)
![b](/images/images-office/image30.png)



2. **Initial Setup**:
   - The initial setup procedure is the same as with the Windows Server:
     - Allocate an appropriate amount of RAM based on your host machine's capacity.
     - Power on the virtual machines and proceed with the setup.

![b](/images/images-office/image32.png)
![b](/images/images-office/image33.png)



3. **Domain Login Setup**:
   - During the setup process, when prompted to log in with Microsoft:
     - Choose **Domain joined** instead.
     - Assign names for the workstations.  
       - Example: **Andy Bernard** and **Jim Halpert**.
     - Set up passwords for both users.

![b](/images/images-office/image35.png)
![b](/images/images-office/image36.png)
![b](/images/images-office/image37.png)


4. **Install VMware Tools**:
   - After the initial setup, install **VMware Tools** for both workstations to enable full-screen resolution.

5. **Rename the Computers**:
   - Rename the computers to the names you assigned earlier (e.g., **Andy Bernard** and **Jim Halpert**).
   - Reboot the virtual machines after renaming.

![b](/images/images-office/image38.png)


---

## Step 5: Modifying the Domain Controller

1. **Turn Off the Workstations**:
   - Shut down both workstation virtual machines before proceeding with the domain controller modifications.

2. **Set Up Users, Groups, and Policies**:
   - Open **Server Manager** on the Domain Controller.
   - Select **Tools** > **Active Directory Users and Computers**.

![b](/images/images-office/image39.png)
![b](/images/images-office/image40.png)

3. **Create an Organizational Unit**:
   - Create a new **Organizational Unit** (OU) and name it **Groups**.
   - Move all the existing groups into this newly created folder.

4. **Add New Accounts**:
   - Create the following accounts:
     - **Administrators**: Assign Domain Administrator privileges.
     - **Service Accounts**: For system administration purposes.
     - **Users**: Regular domain user accounts.


![b](/images/images-office/image41.png)
![b](/images/images-office/image42.png)
![b](/images/images-office/image43.png)

5. **Create a File Share**:
   - Open **Server Manager** > **File and Storage Services**.
   - Select **New Share** and set up an SMB share.
     - Name the share **hackme**.

![b](/images/images-office/image44.png)
![b](/images/images-office/image45.png)
![b](/images/images-office/image46.png)
![b](/images/images-office/image47.png)

6. **Configure the Service Account**:
   - Use the Command Prompt to fully configure the service account by running the necessary commands.

![b](/images/images-office/image48.png)
![b](/images/images-office/image49.png)
![b](/images/images-office/image50.png)

7. **Set Up a Group Policy**:
   - In **Server Manager**, create a Group Policy Object (GPO):
     - Select **Create a GPO in this domain, and Link it here...**.
     - Right-click on the new policy and select **Edit**.
   - Navigate to:
     - **Computer Configuration** > **Policies** > **Administrative Templates** > **Windows Components** > **Microsoft Defender Antivirus**.
     - Enable the **Turn off Microsoft Defender Antivirus** option.
   - Enforce the policy by right-clicking on it and selecting **Enforce**.

![b](/images/images-office/image51.png)
![b](/images/images-office/image52.png)
![b](/images/images-office/image53.png)
![b](/images/images-office/image54.png)
![b](/images/images-office/image55.png)
![b](/images/images-office/image56.png)
![b](/images/images-office/image57.png)

8. **Set a Static IP Address**:
   - Configure a static IP address for the Domain Controller in the **Ethernet Options**.

![b](/images/images-office/image58.png)
![b](/images/images-office/image59.png)

---

## Step 6: Joining Workstations to the Domain

1. **Configure Adapter Options**:
   - Log into both workstations and modify the network adapter settings based on the Domain Controller’s static IP address.

2. **Join the Domain**:
   - On each workstation:
     - Select **Join this device to a local Active Directory domain**.
     - Enter the domain name (e.g., **OFFICE.local**) and authenticate.

![b](/images/images-office/image60.png)
![b](/images/images-office/image61.png)
![b](/images/images-office/image62.png)
![b](/images/images-office/image63.png)
![b](/images/images-office/image64.png)

3. **Verify Domain Membership**:
   - Check **Active Directory** on the Domain Controller to ensure both workstations are successfully joined.

![b](/images/images-office/image65.png)

4. **Set Up Local Administrator Accounts**:
   - On each workstation:
     - Create and configure a local administrator account.

![b](/images/images-office/image66.png)
![b](/images/images-office/image67.png)
![b](/images/images-office/image68.png)
![b](/images/images-office/image69.png)

5. **Enable Network Discovery**:
   - Turn on **Network Discovery** on both workstations.

6. **Map a Network Drive**:
   - Log out and sign in as the local administrator.
   - Open **File Explorer**, select **This PC**, and choose **Map Network Drive**.
   - Use the shared drive created earlier (e.g., **hackme**).

![b](/images/images-office/image70.png)
![b](/images/images-office/image71.png)
![b](/images/images-office/image72.png)
![b](/images/images-office/image73.png)
![b](/images/images-office/image74.png)
![b](/images/images-office/image75.png)


---

## Final Notes: Lab Setup for Attack Simulation

- The above settings and procedures were intentionally designed to introduce potential problems and misconfigurations for practicing attack strategies.












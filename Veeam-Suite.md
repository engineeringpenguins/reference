# VEEAM Suite of Products

## Introduction  

VEEAM is THE authority and leading solution for data backup, migration, replication, and recovery of VMWare (and generic storage) enviornments. While they are not officially a VMWare product, VMWare uses VEEAM for their own data backups.  

This article aims to walk through the initial setup of common VEEAM products.  

### Table of Contents  

<details>
  <summary><a href="#veeam-backup-and-replication"> Backup & Replication</a></summary>
    <ul>
      <li><a href="#veeam-br-installation"> BR Install</a></li>
      <li><a href="#veeam-br-post-install-configuration"> BR Post Install</a></li>
      <li><a href="#veeam-br-setup-backup-repos"> BR Backup Repos</a></li>
      <li><a href="#veeam-br-inventory"> BR Inventory</a></li>
      <li><a href="#veeam-br-jobs"> BR Jobs</a></li>
      <li><a href="#update-veeam-br"> Update BR</a></li>
    </ul>
</details>
<details>
  <summary><a href="#veeam-one"> VEEAM ONE</a></summary>
    <ul>
      <li><a href="#veeam-one-installation"> ONE Install</a></li>
      <li><a href="#veeam-one-post-install-configuration"> ONE Post Install</a></li>
    </ul>
</details>
<details>
  <summary><a href="#veeam-data-platform"> Data Platform</a></summary>
    <ul>
      <li><a href="#veeam-data-platform-installation"> Data Platform Install</a></li>
      <li><a href="#veeam-orchestrator-post-install-configuration"> Orchestrator Post Install</a></li>
    </ul>
</details>
<details>
  <summary><a href="#veeam-backup-for-office-365"> Backup for Office 365</a></summary>
    <ul>
      <li><a href="#veeam-backup-for-office-365-installation"> Backup for Office 365 Install</a></li>
      <li><a href="#veeam-backup-for-office-365-post-install-configuration"> Backup for Office 365 Post Install</a></li>
      <li><a href="#nfr-veeam-o365-installation"> NFR VEEAM O365 Install</a></li>
    </ul>
</details>
<details>
  <summary><a href="#troubleshooting"> Troubleshooting</a></summary>
</details>
<br>

## VEEAM Backup and Replication

VEEAM Backup & Replication is the bread and butter of the VEEAM ecosystem. All Data backups, migrations, recovery, and rapid replication will be managed from here.  

A VEEAM backup job starts with the 'backup proxy', this is a machine (either the VEEAM BR server or an external agent) that handles any data level operations taking place between the source data and the destination. Deduplication, compression, encryption, and rate limiting all fall into the responsibility of the backup proxy.  

The storage targets of your VEEAM jobs are called 'Backup Repositories', which can be stored locally (on the VEEAM BR server), in object storage, in the cloud, or on fileshares (SMB/NFS/ISCSI). You can configure repositories that store data in multiple locations (horizontal scale-out or tiered storage), these are called scale-out repositories (example: store on a local NAS for **Performance Tier** and then move to Azure storage for **Capacity Tier**). If you have VEEAM BR applications for your cloud enviornments (ex. VEEAM Backup & Replication for Azure) you can add those repositories as 'External Repositories'. It is worth noting that External Repositories are read only and cannot be used as a traditional backup target for your local jobs, the usecase for adding them here would be to push backed up systems from one cloud into another.  

The license that you have obviously shapes what functionality you have access to. This article was made with a VEEAM data platform license (12.1.1) but 90% of the steps can be done with a Community Edition license (tested 11.0.1 and 12.1.0).  

### VEEAM BR Installation

To install VEEAM Backup & Replication you just need to mount an installation image and follow the wizard which is fairly intuitive.  

1. [Download VEEAM](https://www.veeam.com/downloads.html)
2. Double click the .ISO file when it finishes downloading
3. If the installer does not autorun you may need to open the image and run setup.exe
4. When the wizard opens choose to 'Install VEEAM Backup & Replication'
   1. The VEEAM Backup & Replication Console is a client to interact with the VEEAM BR server
5. Accept the EULA
6. Point the installer to your license file
7. Set your install location (you wouldnt just put everything on the C: drive would you?)
8. Provide a service account in AD or a local windows one (dont be lazy and use the SYSTEM account)
   1. VEEAM seemlessly integrates with Active Directory if the host is AD joined
9. If you have a SQL server already you can point VEEAM at it or just install a local windows database via the wizard
   1. Normally a good idea to delete the old database or use a new DB name and restore a config backup
10. Confirm what ports the VEEAM application should use
11. Set the location of the cache and catalogue before installing

![VEEAM Installer](https://github.com/engineeringpenguins/reference/blob/main/Linked-Images/veeam/veeam_iso.png)  

### VEEAM BR Post Install Configuration

Initial application setup for best practices  

1. Click on the hamburger drop down menu upper left
2. Hover over the Credentials sub menu
   1. Open the 'Datacenter Credentials' object
   2. Change the password on all existing accounts to something secure
      - Avoids using VEEAM default creds when deploying appliances
      - Store passwords in a secure location as you cant view them later
   3. Build out any service accounts, AD logins, and SSH credentials here
3. Go back into the Credentials sub menu and add Cloud access and Encryption keys
4. Click on the hamburger drop down again
5. Open the 'Users & Roles' sub menu
6. Add appropriate users and groups from Active Directory or on the local machine
7. Setup MFA is needed (disabled by default)
8. Setup situation specific configs as needed
   1. I open the 'Malware Detection' sub menu and enable inline entropy analysis
      1. Set to Medium Low
   2. Using the 'Options' sub menu set SMTP and SSL certificates
      1. Should be required for production but for lab uses I typically dont bother

### VEEAM BR Setup Backup Repos

Start by creating a backup repository that exists on your local network.  

1. Open the 'Backup Infrastructure' menu by clicking the menu item in the lower left
2. Select the 'Backup Repositories' item in the upper left and then click 'Add Repository' in the top left
3. I'm going to start with a NAS share but select whatever makes sense
   1. Likely going to transition from NAS to local S3 buckets soon
4. The Backup Repository setup wizard will guide you through setup
   1. For the NAS share I started by setting the method to SMB
   2. Provide the name & description
   3. Define the share path and credentials
   4. Use the default repository settings
   5. Local server as the mount host and a directory on the data drive for cache
      1. Make sure vPowerNFS is enabled
   6. Review and apply the configuration
5. If prompted to change the default configuration backup location choose no

Add a second backup repository that exists in Azure.  

1. Open the 'Backup Infrastructure' menu by clicking the menu item in the lower left
2. Select the 'Backup Repositories' item in the upper left and then click 'Add Repository' in the top left
3. Select 'Object Storage' and select Azure
4. Use the Azure Blob storage option
5. Provide the name and description for this repository
6. Click the 'Add' button and provide your access key and storage account name
   1. ![Azure Keys](https://github.com/engineeringpenguins/reference/blob/main/Linked-Images/veeam/veeam_keys.png)  
7. Select the storage container you want to target
   1. Click 'Browse' and select the appropriate folder or make a new one with the wizard
   2. Check 'Use cool storage tier' only if this is for infrequent full backups (no deltas)
8. Confirm the mount server and cache are correct and that vPowerNFS is enabled
   - If your licensing allows for it you may get a notification 'VEEAM helper appliance has not been configured'
     1. Click the Configure button
     2. To the right of the subscription dropdown click 'Add'
     3. Provide a name and description
     4. Leave the default 'Microsoft Azure' radio button selected
     5. Proceed with the 'Create a new account' radio button selected
     6. Use the code provided to sign into microsoft.com/devicelogin
     7. Confirm the changes you made
     8. After the account is made go back to the wizard in step 8.2
     9. Select your subscription in the dropdown and set the config or leave as default
9. Review and apply the configuration

Add an archive repository that exists in Azure  

1. Open the 'Backup Infrastructure' menu by clicking the menu item in the lower left
2. Select the 'Backup Repositories' item in the upper left and then click 'Add Repository' in the top left
3. Select 'Object Storage' and select Azure
4. Use the Archive storage option
5. Provide the name and description for this repository
6. Use the dropdown and slect the storage credentials
7. Select the storage container you want to target
   - Click 'Browse' and select the appropriate folder or make a new one with the wizard
   - Make the backups immuteable if appropriate
8. Leave the 'Archive' radio button selected
9. Select your subscription in the dropdown

![VEEAM Installer](https://github.com/engineeringpenguins/reference/blob/main/Linked-Images/veeam/veeam_iso.png)  

Create a Scale-Out Repository

1. Open the 'Backup Infrastructure' menu by clicking the menu item in the lower left
2. Select the 'Scale-Out Repositories' item in the middle left and then click 'Add Scale-Out Repository' in the top left
3. Provide the name and description
4. Use the 'Add' button on the right to target your local (NAS) repository
   - this is for your 'Performance Tier' which is for short term "fast" R/W
5. Leave the default 'Data Locality' radio button selected
6. Enable the use of capacity tier and use the 'choose' button to select the Azure Blob
   - Change the time and situation for data to move from performance to capacity if needed
   - Encrypt the backup if appropriate
7. Enable GFS backups to object storage and use the dropdown to select the archive repository

### VEEAM BR Inventory

Add vCenter to VEEAM BR

1. Open the 'Inventory' menu by clicking the menu item in the lower left
2. Click on 'vSphere' servers in the middle left and then click 'Add Server' in the top left
3. Provide the DNS or ip of the vCenter appliance and a description
4. Use the dropdown to select appropriate credentials
5. Review an apply the configuration
   - This will auto discover all ESXI hosts and VMs and add them to the inventory

![VMWare Integration](https://github.com/engineeringpenguins/reference/blob/main/Linked-Images/veeam/veeam_vsphere.png)  

### VEEAM BR Jobs

Create a weekly job to backup VMs to NAS -> Azure Blob -> Archive

1. Open the 'Home' menu by clicking the menu item in the lower left
2. Select the 'Jobs' dropdown in the upper left and click 'Virtual Machine'
3. Provide the name and description
4. Use the 'Add' button on the right to define the VMs to backup
5. Use the 'Backup Repository' dropdown to select your scale-out repository
   1. Enable GFS by checking the box for 'keep certain backups longer'
   2. Click the configure button to the right of the GFS checkbox
   3. Set your desired full backup date and scope
   4. Click the 'Advanced' button in the lower right to configure things like deduplication and compression
6. If needed you can enable application aware backups, filesystem indexing, and malware analysis
   - If you can justify it I would reccomend using these as its invaluable for rapid recovery
   - Make sure to provide the system admin credentials using the dropdown at the bottom
7. Schedule the job and set priority before applying the configuration

![Job Configuration](https://github.com/engineeringpenguins/reference/blob/main/Linked-Images/veeam/veeam_job.png)  

### Update VEEAM BR

1. [Find latest KB link on their releases page](https://www.veeam.com/kb2680)
   - [For this example I am using 12.1](https://www.veeam.com/kb4510)
2. If you are currently using the same major release (update) click the updater link
3. If you arnt currently using the same major release (upgrade) click the ISO link
4. When download is complete run the installer

![BR Updates](https://github.com/engineeringpenguins/reference/blob/main/Linked-Images/veeam/veeam_update.png)  

## VEEAM ONE

![VEEAM ONE Dashboard](https://github.com/engineeringpenguins/reference/blob/main/Linked-Images/veeam/veeam_one.png)  

VEEAM ONE is a tool for monitoring and responding to your VEEAM Infrastructure. The VEEAM ONE Client software is a graphical interface for pulling reports and monitoring any conditions and metrics about your backup, replication, and recovery jobs. VEEAM ONE Server software is purpose built to manage your VEEAM backup Infrastructure and enforce any actions initiated from the VEEAM ONE Client. Recently VEEAM ONE has been expanded to support scripted actions in response to events (for instance automated replication or isolating corrupted/infected data).  

If you install the VEEAM ONE server on the same server as your other VEEAM products (Backup and Replication or O365) it should automagically detect everything already in place. If you install it elsewhere you can point it at wherever you need by right clicking anywhere in the left hand navigation pane and clicking 'Add Server'.

### Install VEEAM ONE

Install VEEAM ONE from the VEEAM website.

1. [Download VEEAM ONE](https://www.veeam.com/downloads.html)
2. Double click the .ISO file when it finishes downloading
3. If the installer does not autorun you may need to open the image and run setup.exe
4. When the wizard opens choose to 'Install VEEAAM ONE'
5. Accept the EULA
6. Set your install location and click install

### VEEAM ONE Post Install Configuration

On connecting to the VEEAM ONE server with the VEEAM ONE Client you have the option to configure SMTP and email alerts on a splash screen. Other than Email configuration VEEAM ONE is ready to use out of the box. Its main purpose is monitoring and reporting but it can be used in tandem with VEEAM Orchestrator to automate responses to events and provide best practice analysis and feedback.  

![VEEAM ONE](https://github.com/engineeringpenguins/reference/blob/main/Linked-Images/veeam/veeam_onedash.png)  

## VEEAM Data Platform

If this is not for production use and you just have a need to lab or demo something quickly, it does not make sense to limp by with the VEEAM Community Edition when the official [VEEAM NFR Data Platform](https://go.veeam.com/free-nfr-veeam-data-platform) exists. With this offering you can get the top VEEAM product with a one year license provided its not for production and you meet their eligibility (certification) requirements.  

### VEEAM Data Platform Installation

1. Uninstall any current VEEAM B&R or VEEAM ONE installations
2. [Sign up for the Data Platform NFR](https://go.veeam.com/free-nfr-veeam-data-platform)
3. Double click the .ISO file when it finishes downloading
4. When the wizard opens choose to 'Install VEEAM Recovery Orchestrator'
   1. This installs Orchestrator, Backup & Replication, and VEEAM ONE
5. From here the wizard follows the same steps provided in <a href="#veeam-br-installation">VEEAM BR Installation</a>

### VEEAM Orchestrator Post Install Configuration

Follow the first time setup wizard on the VEEAM Orchestrator web page.  

1. Open the VEEAM Orchestrator application (desktop or install directory) or navigate to port localhost:9898 in a browser
2. On the following webpage sign in with the account used to install the application (windows login)
3. Initiate the first time setup when prompted
4. Create Administrators from users and groups in Active Directory or on the local machine
   - ![Veeam Orchestrator setup](https://github.com/engineeringpenguins/reference/blob/main/Linked-Images/veeam/veeam_orch.png)
5. Add appropriate credentials and Infrastructure
   - ![Veeam Orchestrator Config](https://github.com/engineeringpenguins/reference/blob/main/Linked-Images/veeam/veeam_orch2.png)
6. Conclude the setup by accepting the configuration confirmation

## VEEAM Backup for Office 365

VEEAM Backup for Office 365 is a tool for creating backups of enterprise O365 data. The most common usecases are to create user level backups of OneDrive and Exchange (Online or On-Prem) content but it is also possible to backup Sharepoint sites (Online or On-Prem) and Teams.  

![VEEAM 365 backup](https://github.com/engineeringpenguins/reference/blob/main/Linked-Images/veeam/veeam_o365.png)  

### Install VEEAM Backup for Office 365

1. [Download VEEAM O365](https://www.veeam.com/downloads.html)
2. Double click the .ISO file when it finishes downloading
3. If the installer does not autorun you may need to open the image and run setup.exe
4. When the wizard opens choose to 'Install VEEAM Backup for Microsoft365'
5. Accept the EULA
6. Set your install location and click install

### VEEAM O365 Post Install Configuration

Start by linking an organization:  

1. Click Add Org in upper left
    Leave defaults:
        Deployment type: Microsoft 365
        Check boxes:
            Exchange online
            Sharepoint Online and OneDrive for Business
            Microsoft Teams
2. On the following screen leave the Region as Default
3. Select radio button for modern authentication  
    If you check the box for legacy auth. VEEAM will not auto create objects in the next step
4. Select the radio button that creates a new Azure AD application
5. Click the 'Install' button and generate a new self signed certificate
6. The next screen displays a code that you will need to enter on microsoft.com/devicelogin

You can save your data locally or in the cloud, ideally you would do both. To use Azure you will need to use an Azure storage account. Use an existing storage account or create a new one and navigate to the blob containers and use an existing one or create a new container. Grab one of the two access keys from the storage account for VEEAM to use to access the blob container.

![Azure Keys](https://github.com/engineeringpenguins/reference/blob/main/Linked-Images/veeam/veeam_keys.png)  

Add a storage location:  

1. In the VEEAM O365 console click backup infrastructure in the lower left
2. Right click object storage and add backup infrastructure
3. Name as appropriate and click Next
4. Select Microsoft Azure blob storage
5. Provide the storage account name and the access key you got in a previous step

Create a Backup Job:  

1. Navigate back to organizations menu
2. Right click org and add to backup job
3. You can backup EVERYTHING in the organization or specify users/sites/groups/etc.
4. Use the Azure repository you built earlier as the destination
5. Depending on the sensitivity of data you can get pretty granular with the task schedulin

### NFR VEEAM O365 Installation

If this is not for production use and you just have a need to lab or demo something quickly, it does not make sense to limp by with the VEEAM Community Edition when the official [VEEAM NFR Data Platform](https://go.veeam.com/free-nfr-veeam-backup-microsoft-office-365) exists. With this offering you can get the O365 backup product with a one year license provided its not for production and you meet their eligibility (certification) requirements.  

1. Uninstall any current VEEAM O365 backup installations
2. [Sign up for the O365 NFR](https://go.veeam.com/free-nfr-veeam-backup-microsoft-office-365)
3. Follow the steps in <a href="#install-veeam-backup-for-office-365">Install VEEAM Backup for Office 365</a>

## Troubleshooting

ERROR:

- 'Requested URI does not represent any resource on the server' = probably because you enabled hierarchacal namespaces on the storage account  

- 'Available azure subscriptions not found' = youre probably not using a service account or the service account doesnt have approrpiate MS GRAPH access  

- 'This operation is not permitted' = you cannot write to cold or archive blobs (presumably because of syncing deltas), use cool storage tier  

- 'Failed to get Azure Immuteability config, snapshots persists' = soft delete and versioning MUST be disabled on the storage account
  - You MIGHT be able to fix using lifecycle mgmt to delete snapshots or storage explorer but likely that blob is permenantly incompatible

# VEEAM Suite of Products

## Introduction  

VEEAM is THE authority and leading solution for data backup, migration, replication, and recovery of VMWare (and generic storage) enviornments. While they are not officially a VMWare product, VMWare uses VEEAM for their own data backups.  

This article aims to walk through the initial setup of common VEEAM products.  

## VEEAM Backup and Replication

VEEAM Backup & Replication is the bread and butter of the VEEAM ecosystem. All Data backups, migrations, recovery, and rapid replication will be managed from here.  

A VEEAM backup job starts with the 'backup proxy', this is a machine (either the VEEAM BR server or an external agent) that handles any data level operations taking place between the source data and the destination. Deduplication, compression, encryption, and rate limiting all fall into the responsibility of the backup proxy.  

The storage targets of your VEEAM jobs are called 'Backup Repositories', which can be stored locally (on the VEEAM BR server), in object storage, in the cloud, or on fileshares (SMB/NFS/ISCSI). You can configure repositories that store data in multiple locations, these are called scale-out repositories (example: store on a local NAS and then move to Azure storage).  

### VEEAM BR Installation

To install VEEAM Backup & Replication you just need to mount an installation image and follow the wizard which is fairly intuitive. Both VEEM BR 11.0.1 and 12.1.1 were used for this article.  

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

### VEEAM Data Platform Installation

If this is not for production use and you just have a need to lab or demo something quickly, it does not make sense to limp by with the Community Edition when the official [VEEAM NFR Data Platform](https://go.veeam.com/free-nfr-veeam-data-platform). With this offering you can get the highest end VEEAM product with a one year license provided its not for production and you meet their eligibility (certification) requirements.  

1. Uninstall any current VEEAM B&R or VEEAM ONE installations
2. [Sign up for the Data Platform NFR](https://go.veeam.com/free-nfr-veeam-data-platform)
3. Double click the .ISO file when it finishes downloading
4. When the wizard opens choose to 'Install VEEAM Recovery Orchestrator'
   1. This installs Orchestrator, Backup & Replication, and VEEAM ONE
5. From here the wizard follows the same steps provided in <a href="#veeam-br-installation">VEEAM BR Installation</a>

### VEEAM BR Post Install Configuration

Start by creating a backup repository that exists on your local network.

1. Open the 'Backup Infrastructure' menu by clicking the menu item in the lower left
2. Select the 'Backup Repositories' item in the upper left and then click 'Add Repository' in the top left
3. Choose the type of storage you want to use and then click 'Next'
4. Navigate through the wizard and click 'Next' to complete the setup

### VEEAM Orchestrator Post Install Configuration

1. Open the VEEAM Orchestrator application on your desktop or in the install directory
2. On the following webpage sign in with the account signed into windows
3. Initiate the first time setup when prompted
4. Create Administrators from users and groups in Active Directory or on the local machine
   - ![Veeam Orchestrator setup](https://github.com/engineeringpenguins/reference/blob/main/Linked-Images/veeam/veeam_orch.png)
5. Add appropriate credentials and Infrastructure
   - ![Veeam Orchestrator Config](https://github.com/engineeringpenguins/reference/blob/main/Linked-Images/veeam/veeam_orch2.png)
6. Conclude the setup by accepting the configuration confirmation

## VEEAM ONE

VEEAM ONE is a tool for monitoring and responding to your VEEAM Infrastructure (other VEEAM products). The VEEAM ONE Client software is a graphical interface for pulling reports and monitoring any conditions and metrics about your backup, replication, and recovery jobs. VEEAM ONE Server software is purpose built to manage your VEEAM backup Infrastructure and enforce any actions initiated from the VEEAM ONE Client. Recently VEEAM ONE has been expanded to support scripted actions in response to events (for instance automated replication or isolating corrupted/infected data).

### VEEAM ONE Server & Client

If you install the VEEAM ONE server on the same server as your other VEEAM products (Backup and Replication or O365) it should automagically detect everything already in place. If you install it elsewhere you can point it at wherever you need by right clicking anywhere in the left hand navigation pane and clicking 'Add Server'.

The VEEAM ONE Client is designed to be installed on Windows Hosts/Desktops, it is just a front end for VEEAM ONE Server so you can just point it at the ip/dns of whatever VEEAM ONE Server(s) you have available.  

## VEEAM Backup for Office 365

VEEAM Backup for Office 365 is a tool for creating backups of enterprise O365 data. The most common usecases are to create user level backups of OneDrive and Exchange (Online or On-Prem) content but it is also possible to backup Sharepoint sites (Online or On-Prem) and Teams.

![VEEAM 365 backup](https://github.com/engineeringpenguins/reference/blob/main/Linked-Images/veeam/veeam_o365.png)  

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

### Troubleshooting

ERROR:
'Requested URI does not represent any resource on the server' = probably because you enabled hierarchacal namespaces on the storage account  

'Available azure subscriptions not found' = youre probably not using a service account or the service account doesnt have approrpiate MS GRAPH access  

'This operation is not permitted' = you cannot write to cold or archive blobs (presumably because of syncing deltas), use cool storage tier  

### MISC. Operations - Not Used or Currently Relevant

Create Azure Service Account for VEEAM:

1. Click on the hamburger menu upper left
2. Select manage cloud creds
3. Select azure service account
4. Use defaults and follow wizard

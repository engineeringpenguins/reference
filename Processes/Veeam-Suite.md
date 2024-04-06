# VEEAM Suite of Products

## Introduction  

VEEAM is THE authority and leading solution for data backup, migration, replication, and recovery of VMWare (and generic storage) enviornments. While they are not officially a VMWare product, VMWare uses VEEAM for their own data backups.  

This article aims to walk you through the most popular VEEAM tools, their set up and use, and how to leverage them effectively.  

## VEEAM Backup and Replication

VEEAM Backup & Replication is the bread and butter of the VEEAM ecosystem. All Data backups, migrations, recovery, and rapid replication will be managed from here.  

A VEEAM backup job starts with the 'backup proxy', this is a machine (either the VEEAM BR server or an external agent) that handles any data level operations taking place between the source data and the destination. Deduplication, compression, encryption, and rate limiting all fall into the responsibility of the backup proxy.  

The storage targets of your VEEAM jobs are called 'Backup Repositories', which can be stored locally (on the VEEAM BR server), in object storage, in the cloud, or on fileshares (SMB/NFS/ISCSI). You can configure repositories that store data in multiple locations, these are called scale-out repositories (example: store on a local NAS and then move to Azure storage).

### Setup

Start by creating a backup repository that exists on your local network.

1. Open the 'Backup Infrastructure' menu by clicking the menu item in the lower left
2. Select the 'Backup Repositories' item in the upper left and then click 'Add Repository' in the top left
3. Choose the type of storage you want to use and then click 'Next'
4. Navigate through the wizard and click 'Next' to complete the setup

## VEEAM Backup for Office 365

VEEAM Backup for Office 365 is a tool for creating backups of enterprise O365 data. The most common usecases are to create user level backups of OneDrive and Exchange (Online or On-Prem) content but it is also possible to backup Sharepoint sites (Online or On-Prem) and Teams.

![VEEAM 365 backup](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/veeam/veeam0365.png)  

### Post Install Configuration

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

![Azure Keys](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/veeam/veeam_azureKey.png)  

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

## VEEAM ONE

VEEAM ONE is a tool for monitoring and responding to your VEEAM Infrastructure (other VEEAM products). The VEEAM ONE Client software is a graphical interface for pulling reports and monitoring any conditions and metrics about your backup, replication, and recovery jobs. VEEAM ONE Server software is purpose built to manage your VEEAM Infrastructure and orchestrate any actions initiated from the VEEAM ONE Client. Recently VEEAM ONE has been expanded to support scripted actions in response to events (for instance automated replication or isolating corrupted/infected data).

### VEEAM ONE Server & Client

If you install the VEEAM ONE server on the same server as your other VEEAM products (Backup and Replication or O365) it should automagically detect everything already in place. If you install it elsewhere you can point it at wherever you need by right clicking anywhere in the left hand navigation pane and clicking 'Add Server'.

The VEEAM ONE Client is designed to be installed on Windows Host/Desktops, it is just a front end for VEEAM ONE Server so you can just point it at the ip/dns of whatever VEEAM ONE Server(s) you have available.  

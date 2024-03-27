# VEEAM Suite of Products

## Introduction  

VEEAM is THE authority and leading solution for data backup, migration, replication, and recovery of VMWare (and generic storage) enviornments. While they are not officially a VMWare product, VMWare uses VEEAM for their own data backups.  

This article aims to walk you through the most popular VEEAM tools, their set up and use, and how to leverage them effectively.  

## VEEAM Backup and Replication

Content Comming Soon

## VEEAM ONE

Content Comming Soon

## VEEAM Backup for Office 365

VEEAM Backup for Office 365 is a tool for creating backups of enterprise O365 data. The most common usecases are to create user level backups of OneDrive and Exchange (Online or On-Prem) content but it is also possible to backup Sharepoint (Online or On-Prem) sites and Teams.

![VEEAM 365 backup](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/veeam/veeam0365.png)  

### Post Install Configuration

Start by linking an organization:

1. Click Add Org in upper left
    Leave defaults:
        Microsoft365 for org deployment type
        all 3 check boxes
        exchange online
        sharepoint online/onedrive
        teams
2. Click Next
    leave defaults:
        Region: Default
3. Select radio button for modern authentication  
    dont check box for legacy auth unless you need to
        you wont be able to have veeam auto create things in the next step
4. Have it create an azure app for you and then have it generate an ssl cert to use for it
5. Go to a microsoft site for device enrollment and put in a code displayed in the veeam console
6. Navigate to azure storage acc and make a container
7. pull access keys from left

![Azure Keys](https://github.com/engineeringpenguins/reference/blob/main/Processes/Linked-Images/veeam/veeam_azureKey.png)  

Add a storage location:

1. Click backup infrastructure in the lower left
2. Right click object storage and add backup infrastructure
3. Name as appropriate
4. Select microsoft azure blob storage
5. Provide the storage acc name and access key you added earlier

Create a Backup Job:

1. Navigate back to organizations menu
2. Right click org and add to backup job
3. Select what objects you want backed up and to where (and how often)

### Troubleshooting

if youre getting 'Requested URI does not represent any resource on the server' its probably because you enabled hierarchacal namespaces on the storage account  

if you get ' available azure subscriptions not found' youre probably not using a service account or the service account doesnt have access  

You cannot write to cold or archive blobs (presumably because of syncing deltas), use cool storage tier  

### MISC. Operations - Not Used or Currently Relevant

Create Azure Service Account

1. Click on the hamburger menu upper left
2. Select manage cloud creds
3. Select azure service account
4. Use defaults and follow wizard

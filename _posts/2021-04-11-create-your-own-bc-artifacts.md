---
layout: post
title:  "Create your own BC Artifacts"
date:   2021-04-11 21:26:09 +0200
categories: bccontainerhelper
---

# Introduction

I got the idea to create my own BCArtifacts to improve my development and save time publishing large app dependency trees. Hence I looked into the BCArtifacts process and created my own artifacts that are compatible with the bccontainerhelper.

This could be an easy way to create and distribute modified dev-/test- or demo-environments; especially useful for large dependency trees or customized base apps.

# What are BCArtifacts?

Using BCArtifacts is the new way of creating docker containers for Dynamics 365 Business Central.
For more documentation visit [freddysblog](https://freddysblog.com/2020/06/25/working-with-artifacts/).

# How does it work

What you need to know is that bccontainerhelper downloads two specific files for container creation. The file urls are built by the Get-BCArtifactUrl command. These two specific files are the application part and the platform part. 

![ContainerCreation](/assets/YourOwnBCArtifacts/ContainerCreation.png)

Download Url examples:
- https://bcartifacts.azureedge.net/sandbox/18.0.23013.24175/us
- https://bcartifacts.azureedge.net/sandbox/18.0.23013.24175/platform

To be able to create your own artifacturl and use freddy's Get-BCArtifactUrl command you need to upload these files to an Azure storage account.

On the Azure storage account you need to create a Blob container named *sandbox* or *onprem*, this is important. In the next step create a folder for the artifact version and upload the (altered) files.

![AzureStorageExplorer](/assets/YourOwnBCArtifacts/AzureStorageContainer.png)
Note: Use [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/) it helps a lot :)

To grant access for anyone change your container access level.

![AzureStorageAccess](/assets/YourOwnBCArtifacts/AzureStorageContainerAccess.png)

By adding the storageAccount parameter to Get-BCArtifacturl command you get your own artifacturl.

![OwnArtifactUrl](/assets/YourOwnBCArtifacts/OwnStorageAccount.png)

Now you can create a BC container with your "own" artifacts.

# Modifying BCArtifacts

Essentially artifacts are zip files that are copied into the docker image/ container. Therefore you can easily modify the files inside the zip.

## Platform artifact

For example you can add further control addins to your Business Central installation by copying them into the Add-ins folder of the servicetier.

![platformartifacts](/assets/YourOwnBCArtifacts/platformartifacts.png)

## Application artifact

For modifying the application artifact the aim is to replace the bak file. 

![applicationartifacts](/assets/YourOwnBCArtifacts/applicationartifacts.png)

For my way of creating a customized bak file the following steps are necessary:
- Create a docker container with -multitenant:false
- Install your apps and setup your demo data
- Create a database backup (optional: delete your user before creating the bak file)
- Replace the .bak in the zip file
- Make sure the manifest.json has the correct database property set 
![manifestjson](/assets/YourOwnBCArtifacts/manifestjson.png)

Note: Be careful with your license. It is uploaded to the database when you import it to the docker container.

# Security considerations

To manage and restrict access to your Azure Storage Container you can setup a shared access signature (SAS) token.
A SAS token provides secure, delegated access to resources in your Azure storage account.

1. Change the Public access level of your Azure Storage container to Private
    ![azstoragecontaineraccessprivate](/assets/YourOwnBCArtifacts/mybcartifactscontaineraccessprivate.png)
2. Create a SAS Token for your Azure Storage account with the following options selected
    ![azstoragesastoken](/assets/YourOwnBCArtifacts/azstorageaccountsas.png)
3. Add the -sasToken parameter to the Get-BCArtifactUrl command
    ![azstoragesastokencontainer](/assets/YourOwnBCArtifacts/containerwithsastoken.png)

# ARM Templates

Freddy provides [ARM Templates](https://freddysblog.com/2020/07/02/the-arm-templates-now-supports-artifacts-and-images/) for partners and customers. These templates create an Azure VM with a specific version of Business Central.
Now that you have your own artifacts beeing compatible with the bccontainerhelper, you can use the ARM Templates to create Azure VMs with your
modified artifacts.

I think this is pretty a awesome feature :)

# Further ideas

- The possibilities of modifying the application artifact are not depleted yet
- CI/CD and your own artifacts
- Automate artifact creation
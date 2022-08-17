# SAP HANA ARM Installation
This ARM template is used to install SAP HANA on a single VM running SUSE SLES 15 SP2.  For documentation on deploying a cluster of linux machines running HANA System Replication and Linux High Availability Extension, please see documentation here: [cluster deployment](https://github.com/AzureCAT-GSI/SAP-HANA-ARM-v2/blob/master/README-HSR.md). 

This template uses the Linux SKU for SAP. The template takes advantage of [Custom Script Extensions](https://github.com/Azure/azure-linux-extensions/tree/master/CustomScript) for the installation and configuration of the machine. This should be used only for the dev POC environment. This is not a production deployment.

[![Deploy to Azure](http://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FJanelleJames-devops%2FDTPSGH-v3prd%2Fmaster%2Fazuredeploy.json)
## Machine Info
The template was originally written to deploy HANA on one of the machines listed in the table below with the noted disk configuration.  Modification to disk sizes has been made per Deloitte SME and Microsoft SME suggestions of data=3xRAM, logs=2xRAM (with 2TB Max) and stripe sizes of 256 (data) and 64 (log) for the M64s machine image.  Disk sizes for other machine images is TBD.  The deployment takes advantage of Managed Disks, for more information on Managed Disks or the sizes of the noted disks can be found on [this](https://docs.microsoft.com/en-us/azure/storage/storage-managed-disks-overview#pricing-and-billing) page.

Machine Size | RAM | Data and Log Disks | /hana/shared | /root | /usr/sap | hana/backup
------------ | --- | ------------------ | ------------ | ----- | -------- | -----------
E16 | 128 GB | 2 x P20 | 1 x S20 | 1 x S6 | 1 x S6 | 1 x S15
E32 | 256 GB | 2 x P20 | 1 x S20 | 1 x S6 | 1 x S6 | 1 x S20
E64 | 432 GB | 2 x P20 | 1 x S20 | 1 x P6 | 1 x S6 | 1 x S30
GS5 | 448 GB | 2 x P20 | 1 x S20 | 1 x P6 | 1 x S6 | 1 x S30

For the M series servers, this template uses the [Write Accelerator](https://docs.microsoft.com/azure/virtual-machines/linux/how-to-enable-write-accelerator) feature for the Log disks.  For this reason, the log devices are separated out from the data disks:

Machine Size | RAM | Data Disks | Log disks| /hana/shared | /root | /usr/sap | hana/backup
------------ | --- | ------------------ | ------------------ |------------ | ----- | -------- | -----------
M64s | 1TB | 4 x P20 | 2 x P20 | 1 x P30 | 1 x P6 | 1 x P6 | 2 x P30
M64ms | 1.7TB | 3 x P30 | 2 x P20 | 1 x P30 | 1 x P6 | 1 x P6 | 2 x P30
M128s | 2TB | 3 x P30 | 2 x P20 | 1 x P30 | 1 x P6 | 1 x P6 | 2 x P40
M128ms | 3.8TB | 5 x P30 | 2 x P20 | 1 x P30 | 1 x P6 | 1 x P6 | 5 x P50

## Installation Media
Installation media for SAP HANA should be downloaded and placed in the SapBits folder. You will need to provide the URI for the container where they are stored, for example https://yourBlobName.blob.core.windows.net/yourContainerName. For more information on how to upload files to Azure please go [here](https://github.com/JanelleJames-devops/Deloitte-SAP-ARM-v2/blob/master/UploadToAzure.md)  Specifically you need to download SAP package 51054623, which should consist of one file:
```
51054623.ZIP
```

There should be a folder inside your storage account container called SapBits:

![SapBits Image](https://github.com/JanelleJames-devops/DTPSGH-v3prd/blob/master/media/Structure1.png)

## Deploy the Solution
### Deploy from the Portal

To deploy from the portal using a graphic interface you can use the [![Deploy to Azure](http://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FJanelleJames-devops%2FDTPSGH-v3prd%2Fmaster%2Fazuredeploy.json) button to bring up the template in your subscription and fill out the parameters.

### Deploy from Powershell

```powershell
New-AzureRmResourceGroup -Name HANADeploymentRG -Location "East US2"
New-AzureRmResourceGroupDeployment -Name HANADeployment -ResourceGroupName HANADeploymentRG `
  -TemplateUri https://raw.githubusercontent.com/JanelleJames-devops/DTPSGH-v3prd/master/azuredeploy.json `
  -VMName HANAtestVM -HANAJumpbox yes -CustomURI https://yourBlobName.blob.core.windows.net/yourContainerName -VMPassword AweS0me@PW
```

### Deploy from CLI
```
az login

az group create --name HANADeploymentRG --location "East US2"
az group deployment create \
    --name HANADeployment \
    --resource-group HANADeploymentRG \
    --template-uri "https://raw.githubusercontent.com/JanelleJames-devops/DTPSGH-v3prd/master/azuredeploy.json" \
    --parameters VMName=HANAtestVM HANAJumpbox=no CustomURI=https://yourBlobName.blob.core.windows.net/yourContainerName VMPassword=AweS0me@PW
```
## Monitoring

For your deployment to be supported by SAP the Azure Enhanced Monitoring Extension must be enabled on the Virtual Machine. Please refer to the following [blog post](https://docs.microsoft.com/en-us/azure/virtual-machines/workloads/sap/deployment-guide#d98edcd3-f2a1-49f7-b26a-07448ceb60ca) for more information on how to enable it.

## Parameters

Parameter name | Required | Description | Default Value | Allowed Values
-------------- | -------- | ----------- | ------------- | --------------
VM Name |Yes |Name of the HANA Virtual Machine. | None | No restrictions
HANA Jumpbox |Yes |Defines whether to create a Windows Server with HANA Studio installed. | None | No Restrictions
VM Size |No |Defines the size of the Azure VM for the HANA server. | Standard_M64s | Standard_GS5, Standard_M64s, Standard_M64ms, Standard_M128s, Standard_M128ms, Standard_E16s_v3, Standard_E32s_v3, Standard_E64s_v3 | No restrictions
Write Accelerator |Yes |Whether to enable the M series write accelerator. | no | yes, no
Network Name |No |Name of the Azure VNET to be provisioned | AZ-EUS2-TAX-NPD-DCMSHARED-PSGAPPS-VNET | No restrictions
Address Prefixes |No |Address prefix for the Azure VNET to be provisioned | 10.212.250.0/23 | No restrictions
HANA Subnet Name |No | Name of the subnet where the HANA server will be provisioned | psgdev-subnet | No restrictions
HANA Subnet Prefix |No |Subnet prefix of the subnet where the HANA server will be provisioned | 10.212.250.0/24 | No restrictions
Management Subnet Name |No | Name of the subnet where the HANA jumpbox will be provisioned | SAPMgmtSubnet | No restrictions
Management Subnet Prefix |No |Subnet prefix of the subnet where the HANA jumpbox will be provisioned | 10.0.6.0/24 | No restrictions
Custom URI | Yes | SAS URL to .ZIP file | None | No restrictions
VM User Name | No | Username for both the HANA server and the HANA jumpbox | testuser | No restrictions
VM Password | Yes | Password for the user defined above | None | No restrictions
Operating System | No | Linux distribution to use for the HANA server | SLES for SAP 15 SP2 | SLES for SAP 15 SP2
HANASID | No | HANA System ID | H10 | No restrictions
HANA Number | No | SAP HANA Instance Number | 00 | No restrictions
Existing Network Resource Group | AZRG-ALL-ITS-VNET-SYS | This gives you the option to deploy the VMs to an existing VNET in a different Resource Group. The value provided should match the name of the existing Resource Group. To deploy the VNET in the same Resource Group the value should be set to "no" | no | No restrictions
IP Allocation Method | no | Lets you choose between Static and Dynamic IP Allocation | Static | Dynamic, Static
Static IP | No | Allows you to choose the specific IP to be assgined to the HANA server. If the allocation method is Dynamic this parameter will be ignored | 10.212.250.151 | No restrictions
Subscription Email | No | OS subscription email for BYOS. Leave blank for pay-as-you-go OS image. |  | No restrictions
Subscription ID | No | OS ID or password for BYOS. Leave blank for pay-as-you-go OS image. |  | No restrictions
SMT Uri | No | The URI to a subscription management server if used, blank otherwise |  | No restrictions
## Known issues
### When clicking on Deploy to Azure you get redirected to an empty directory
![Directories](https://github.com/JanelleJames-devops/DTPSGH-v3prd/blob/master/media/directories.png)

The only way to get around this is to save the template to your own template library. Click on "Create a Resource" and choose "Template Deployment". Click "Create".

![Directories2](https://github.com/JanelleJames-devops/DTPSGH-v3prd/blob/master/media/directories2.png)

Select the option of "Build your own template in the editor"

![Directories3](https://github.com/JanelleJames-devops/DTPSGH-v3prd/blob/master/media/directories3.png)

Copy the contents from the azuredeploy.json [file](https://raw.githubusercontent.com/JanelleJames-devops/DTPSGH-v3prd/master/azuredeploy.json) and paste them into the template editor, click Save.

![Directories4](https://github.com/JanelleJames-devops/DTPSGH-v3prd/blob/master/media/directories4.png)

The template is now available in your template library. Changes made to the github repo will not be replicated, make sure to update your template when changes to the azuredeploy.json file are made.



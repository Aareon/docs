---
Order: 10
xref: v1-qde-setup
Title: Setup
Description: How to setup QDE v1
RedirectFrom: docs/quick-deployment-setup-v1
---

> :memo: **NOTE**
>
> This document is for **Version 1** of the Quick Deployment Environment.
> If you're using a newer version of QDE, please refer to the [newer QDE Setup page](xref:v2-qde).

This document contains instructions for importing the QuickDeploy appliance/VM, or creating a VM and attaching the QuickDeploy disk image to it.
You will receive a download link via email for an archive of the VM image. Once you have this downloaded, it will be ready for extraction and import into your environment.

> :warning: **WARNING**
>
> Please follow these steps in ***exact*** order. These will be very important later when you are trying to use the environment.

## Step 0: Setup Considerations

The following are points to keep in mind during initial setup:

* You will need access to AWS to download the environment (specifically `s3.amazonaws.com`).
* Hostname/FQDN changes will invalidate all scripts and certificates.
  Thus, if you plan to change the hostname, you must do so prior to running any setup scripts.
  If you run into issues, refer to the README file on the desktop of the VM.
* Self-signed certificates are generated by default.
  If you plan to use your own certificates, please reach out to Support for assistance.
* The back-end database is configured with no outbound connections - if you plan to change this, please reach out to Support for assistance.
* If you intend to use Nexus outside of your corporate network without the use of a VPN, you will be required to configure RBAC on the repositories housed inside of the repository server.
  This is to ensure that the packages stored on the server are not publicly accessible without authentication.

### Step 0.1: QDE Rename Considerations

> :warning: **WARNING**
>
> tl;dr: Think long and hard before changing the QDE hostname
>
> Renaming the QDE host requires a lot of things and needs to be completed FIRST prior to ANYTHING that is done on the QDE box. It is strongly recommended **NOT** to rename unless you absolutely need to. The most important reason has to do with how a client installs from QDE - it must learn to trust the QDE certificate. Once renamed, the easy option that's provided for you goes away and you will need to provide a hosted solution with an already trusted certificate.
> You can provide your own certificate that is already trusted on machines as part of the [SSL/TLS Setup](xref:v1-ssl-setup). Your other option is to host the script to trust the certificate with an already trusted certificate. You will find a template that you will need to edit at `c:\choco_setup_files` (in the QDE) named `Import-ChocoServerCertificate.ps1`.
>
> Please contact support if you need help here.

If you rename the QDE Environment, here's a small list of things you'll need to do:

1. Update scripts in Nexus that are currently pointed to the default QDE name.
1. Regenerate SSL Certificates
1. Deploy the Nexus SSL Certificate public key to the clients (there is a helper method that is used if the box is not renamed and is limited to that name for security purposes). See `c:\choco_setup_files\Import-ChocoServerCertificate.ps1` for an example of what we mean.
1. There may be more places impacted. Check with support to ensure all is good to go.

## Step 1: Import Virtual Environment

Choose one of the following methods for what your hypervisor environment supports.

### Platform: Azure

If you choose to use the scripts provided inside the Zip archive, there are a number of pre-requisites that are needed:

* The Hyper-V PowerShell module is required.
  * For Windows 10 run `Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-Management-PowerShell` from an elevated PowerShell prompt.
  * For Windows Server 2012 and later, run `Install-WindowsFeature -Name Hyper-V-PowerShell` from an elevated PowerShell prompt.
* Having both the `Az` and `AzureRm` PowerShell modules installed is not [supported](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps?view=azps-4.3.0#install-the-azure-powershell-module). You can see if you have `AzureRm` installed by running `Get-Module -Name AzureRm -ListAvilable`. If there is no output it is not installed.
* Install the `Az` PowerShell Module. To find out if you have the module installed run `Get-Module -Name az -ListAvailable` from an elevated PowerShell session.
  * To install the `Az` module using Chocolatey, run `choco install az.powershell -y`.
  * Using PowerShell `Install-Module -Name Az -AllowClobber -Scope CurrentUser`.
* For the Azure PowerShell module you will need _either_ PowerShell 5.1 and .NET 4.7.2 **or** PowerShell Core installed:
  * PowerShell 5.1 and .NET 4.7.2:
    * You can find out the current version of PowerShell you are running using the command `$PsVersionTable.PSVersion` from a PowerShell session.
    * To install PowerShell 5.1 use Chocolatey by running `choco install powershell -y` or see [Microsoft Docs](https://docs.microsoft.com/en-us/powershell/scripting/windows-powershell/install/installing-windows-powershell?view=powershell-7#upgrading-existing-windows-powershell).
    * To install .NET 4.7.2 use Chocolatey by running `choco install dotnet4.7.2 -y` or see the [Microsoft docs](https://docs.microsoft.com/en-us/dotnet/framework/install/).
  * PowerShell Core:
    * To install PowerShell Core use Chocolatey by running `choco install powershell-core -y` or see the [Microsoft Docs](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-windows?view=powershell-7).
* AzCopy v10 or later - this is needed to copy the disk to Azure. To find out if you have `azcopy` installed and which version, run `azcopy --version`.
  * To install AzCopy v10 or later, using Chocolatey run `choco install azcopy10 -y` or see the [Microsoft Docs](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-azcopy-v10#download-and-install-azcopy).
* The scripts provided with the QDE virtual machine disk image have defaults that you need to ensure you are comfortable with and extensive help. You can see the help, and the default, by running `Get-Help <SCRIPT-NAME> -full`.
* The scripts will create resources in Azure which will cost you real money. Please ensure you are comfortable with this before continuing. See above to get help with the scripts.
* The execution policy must allow running scripts. If it does not, you can set it _for the current PowerShell session_ by running `Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process`.
* Any code or scripts must be run in an _elevated_ PowerShell session.

Steps to create a QDE virtual machine in Azure:

1. Download the Zip archive containing the Azure virtual machine disk (VHD), and unzip it to the directory you wish to store it in. You should have three files:
  * `QuickDeployEnvironment-Azure.vhd`
  * `Set-QDEAzureDisk.ps1`
  * `New-QDEAzureVM.ps1`
2. While the scripts will do the majority of the hard work needed to create a QDE virtual machine in Azure, we do need to setup a resource group. The default resource group that the scripts will use is `qdeserver-resgrp` however you can supply an existing resource group using the `-ResourceGroupName <YOUR-RESOURCEGROUPNAME>` parameter. To create a resource group run `New-AzResourceGroup -Name <RESOURCEGROUPNAME> -Location <YOUR-AZURE-LOCATION>`.
3. We need convert the disk you have downloaded to the size required, and then upload it to Azure so we can attach it to the QDE virtual machine we will create in the following steps. Note that the script we are about to run contains defaults that should work for the majority of users. However, please run `Get-Help Set-QDEAzureDisk.ps1 -full` to get help on the parameters you can provide and the defaults that have been set. Once you are comfortable, in the directory you extracted the files to, run `.\Set-QDEAzureDisk.ps1 -Verbose <PARAMETERS>` (where `<PARAMETERS>` is any parameters you want to provide). Providing the `-Verbose` switch produces output on the screen. Note that the processes of converting the disk and uploading it can take a long time.
4. Before we connect to the QDE virtual machine in Azure we must reset the password. To create a password we can provide to the next script, run `$qdePwd = '<YOUR-PASSWORD> | ConvertTo-SecureString -AsPlainText -Force` (where `<YOUR-PASSWORD>` is the password you want to set **and is longer than 12 characters**).
5. Once the disk has been uploaded we need to create the QDE virtual machine in Azure and attach the disk we uploaded as the operating system disk. Note that the script we are about to run contains defaults that should work for the majority of users. However, please run `Get-Help New-QDEAzureVM.ps1 -full` to get help on the parameters you can provide and the defaults that have been set. Once you are comfortable, in the directory you extracted the files to, run `.\New-QDEAzureVM.ps1 -Verbose -AdministratorComplexPassword $qdePwd <PARAMETERS>` (where `<PARAMETERS>` is any parameters you want to provide and `$qdePwd` is the password we created in the previous step). Providing the `-Verbose` switch produces output on the screen.
6. Once this is complete the script will output the command you can run to connect to the virtual machine, including the IP address (if you are using the `-Verbose` switch). Use `mstsc.exe /v:<IP-ADDRESS>` to connect and login with the password you created in a previous step.

### Platform: Hyper-V (Appliance)

1. Download zip archive containing the Hyper-V VM directory structure, and unzip it to the directory you wish to store it in.
2. Open Hyper-V Manager, and select `Import Virtual Machine` from the right sidebar (or  Action menu), and choose the folder you extracted from the above archive (e.g. ChocoServer).
3. Increase the size of the VHD, for example, to 500GB. Increase to what you feel comfortable with.

```powershell
# This only increases the available size of the image
# You will still need to increase the space for the C drive
Resize-VHD -Path C:\path\to\QuickDeploy Environment.vhd -Size 500GB
```

4. Windows 10 and Windows Server 2016/2019 version of Hyper-V now come with built-in support for Hyper-V Integration Services, as they automatically get pushed to guest VM's.
   In older versions of Hyper-V, you should see an option to "Insert Integration Services Setup Disk".
   You can use this option to install and enable Hyper-V Integration Services.

Video Summary:

![QDE Hyper-V Appliance Import](/assets/images/quickdeploy/QDE-hypervapp.gif)

### Platform: Hyper-V (VHD file)

1. Download VHD from provided link, and unzip it to the directory you wish to store it in.
2. Increase the size of the VHD, for example to 500GB. Increase to what you feel comfortable with.

```powershell
# This only increases the available size of the image
# You will still need to increase the space for the C drive
Resize-VHD -Path C:\path\to\QuickDeploy Environment.vhd -Size 500GB
```

3. Open Hyper-V Manager.
4. Create a new VM.
5. If prompted, choose a `Generation 1` virtual machine option.
6. Set startup memory to `8192 MB` (you can specify this later as well).
7. When asked to create a new disk or use an existing one, select the `Use existing virtual disk` option and browse to the VHD you unzipped in Step 1.
8. Adjust the hardware specifications of the VM. For a performant system, the following are recommended:
    - 4 vCPUs
    - 8 GB RAM
9. Windows 10 and Windows Server 2016/2019 version of Hyper-V now come with built-in support for Hyper-V Integration Services, as they automatically get pushed to guest VM's.
   In older versions of Hyper-V, you should see an option to `Insert Integration Services Setup Disk`.
   You can use this option to install and enable Hyper-V Integration Services.

Video Summary:

![QDE Hyper-V VHD](/assets/images/quickdeploy/QDE-hyperv.gif)

### Platform: VMware (VMDK file)

1. Download VMDK from provided link, and unzip it to the directory you wish to store it.
2. For ESX/ESXi, open vSphere and upload the downloaded VMDK to your datastore.
3. Create a new VM.
4. When prompted for OS type, choose `Windows Server 2019` (if available), or `Windows Server 2016 or later`.
5. If prompted for boot firmware, choose `Legacy BIOS` (**not** UEFI).
6. When asked to create a new disk or attach, delete the default disk, select attach, and browse to the VMDK you uploaded. **IMPORTANT**: [vCenter/ESX/ESXi] You **must** select an `IDE controller` under the “Controller Location” setting of the disk. If you leave the controller as SCSI (default), your VM will not boot.
7. Adjust the hardware specifications of the VM. For a performant system, the following are recommended:
    - 4 vCPUs
    - 8 GB RAM
8. Once you click Finish, go back into the `Edit settings` context menu for the VM, and expand the disk you attached to 500GB (double-check in OS, and extend if needed). **NOTE**: likely you will need to allocate the additional space to the C drive.
9. Boot up VM, and Install VMware Tools using the console menus (this will require a reboot).

Video Summary:

* VMware ESX/i:
![QDE VMware VMDK](/assets/images/quickdeploy/QDE-vmdk-esx.gif)

* VMware Fusion (Mac OS):
![QDE VMware VMDK](/assets/images/quickdeploy/QDE-vmdk-fusion.gif)

### Platform: VMware (OVF template)

> :warning: **WARNING**
>
> The OVF import method can be tricky. Unless you have the **exact** same network settings as the host where the OVF was exported, you will likely run into failures on attempts to import it. You _could_ workaround this by creating your own VM first, exporting the OVF file, using the Network settings from this file to replace the 2 sections of Network settings in the OVF of QDE, and then attempt to import the OVF for QDE. However, we realize this process is cumbersome, and would **strongly** advise you utilize the VMDK import method above for greatest compatibility. We are working to improve this process, but as of right now, the OVF method is to be used at your own risk.

1. Download OVF file and unzip it to the directory you wish to store it.
2. Review instructions for deploying a VM from an OVF file [here](https://docs.vmware.com/en/VMware-vSphere/6.0/com.vmware.vsphere.html.hostclient.doc/GUID-FBEED81C-F9D9-4193-BDCC-CC4A60C20A4E_copy.html).
3. Adjust settings of newly imported VM to our recommended:
    - 4 vCPUs
    - 8 GB RAM
4. Install VMWare Tools on the VM once booted (this will require a reboot).

### Platform: Other

Most likely you are going to download the VMDK file and convert it to your platform. Please reach out to support to see what options are available.

## Step 2: Other Considerations for Virtual Environment

### Step 2.1: DNS Settings

The QDE environment is configured by default to use DHCP for easier initial setup.
You will likely need to reconfigure it with a static IP address depending on your organization's policies.

## Step 3: Virtual Environment Setup

On the desktop of your QDE VM, there is a `Readme.html` file, that will guide you through the rest of the setup process once you are logged in.
A version of this readme file can be found in the [Quick Deployment Desktop Readme](xref:v1-desktop-readme).

> :memo: **NOTE**: The online version is likely more up to date than the ReadMe you will find on the desktop (not including redacted items like credentials). If there are conflicts between the desktop readme and what you see online, prefer the online version.

> :warning: **WARNING**: If you have an existing corporate environment you will be servicing with the QDE VM, be sure to perform your organization-specific initial configuration **_before_** running setup scripts.

### Step 3.1: Expand Disk Size

On the machine, please check the size of the C drive. If the volume needs to be expanded, expand it to the space you've allocated for the machine by running this command in a PowerShell administrative console:

```powershell
# This should increase the space available on the C drive.
Resize-Partition -DriveLetter C -Size ((Get-PartitionSupportedSize -DriveLetter C).SizeMax)
```

Alternatively, you can use the Disk Management utility to expand the disk, if a GUI is preferred.

### Step 3.2: Add License File to QDE

In the [Quick Deployment Desktop Readme](xref:v1-desktop-readme), it is going to ask you to use the license file. That license file comes from an external location. It is best to copy/paste the file into QDE as a whole file, but you may have needed to set up any kind of extensions available for that.

> :warning: **WARNING**
>
> If you find that you need to copy the text and paste the license file text into a new file in QDE, the file format and name is extremely important to get right. If you don't have UTF-8 or there is a space inserted, Chocolatey will consider it invalid.
> Please contact support if you need help here.

### Step 3.3: Regenerate SSL Certificates

See [QDE SSL/TLS Setup](xref:v1-ssl-setup).

### Step 3.4: Database Password Changes (Optional)

The database credentials are currently pre-set.
If you would like to change the credentials associated with the database, you will need to follow these steps.

1. Change the database access credentials
2. Reinstall the chocolatey-management-service package

```powershell
choco uninstall chocolatey-management-service -y
choco install chocolatey-management-service -y --package-parameters-sensitive=”’/ConnectionString=””Server=localhost\SQLEXPRESS;Database=ChocolateyManagement;User ID=ChocoUser;Password=NewPassword;””’”
```

3. Reinstall the chocolatey-management-web package

```powershell
choco uninstall chocolatey-management-web -y
Choco install chocolatey-management-web -y --package-parameters-sensitive=”’/ConnectionString=””Server=Localhost\SQLEXPRESS;Database=ChocolateyManagement;User ID=ChocoUser;Password=NewPassword;””’”
```

## Step 4: Firewall Changes

See [QDE Firewall Changes](xref:v1-firewall-changes).

## Step 5: Install and Configure Chocolatey on Clients

See [QDE Client Setup](xref:v1-client-setup).

## FAQ

### How do I upgrade QDE?

While we will continue to make improvements to the QDE, there is no upgrade path for the Virtual Machine itself. You can choose to start over with a newer version, but that feels like the wrong way to go.

It is simple to upgrade the components and that it how we recommend upgrading aspects of QDE. Should you want to upgrade say Central Management, you can follow the Central Management steps for upgrade at [Upgrade Central Management](xref:ccm-upgrade).

[Quick Deployment Environment](xref:v1-qde)
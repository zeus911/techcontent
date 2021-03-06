<properties
    pageTitle="Azure 快速入门 - 创建 VM PowerShell | Azure"
    description="快速了解如何使用 PowerShell 创建 Linux 虚拟机"
    services="virtual-machines-linux"
    documentationcenter="virtual-machines"
    author="neilpeterson"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-linux"
    ms.devlang="na"
    ms.topic="hero-article"
    ms.tgt_pltfrm="vm-linux"
    ms.workload="infrastructure"
    ms.date="04/03/2017"
    wacn.date="05/15/2017"
    ms.author="nepeters"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="f56805bf76a62a2c3b066eacab15af971de3230d"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="create-a-linux-virtual-machine-with-powershell"></a>使用 PowerShell 创建 Linux 虚拟机

Azure PowerShell 用于从 PowerShell 命令行或脚本创建和管理 Azure 资源。 本指南详细说明如何使用 PowerShell 创建运行 Ubuntu 14.04 LTS 的 Azure 虚拟机。

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](/pricing/1rmb-trial)。

另请确保已安装了 Azure PowerShell 模块的最新版本。 有关详细信息，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs)。

最后，需要在 Windows 用户配置文件的 `.ssh` 目录中存储名为 `id_rsa.pub` 的公共 SSH 密钥。 有关创建适用于 Azure 的 SSH 密钥的详细信息，请参阅[创建适用于 Azure 的 SSH 密钥](/documentation/articles/virtual-machines-linux-mac-create-ssh-keys/)。

## <a name="log-in-to-azure"></a>登录 Azure

使用 `Login-AzureRmAccount` 命令登录到 Azure 订阅，并按照屏幕上的说明进行操作。

    Login-AzureRmAccount -EnvironmentName AzureChinaCloud

## <a name="create-resource-group"></a>创建资源组

创建 Azure 资源组。 资源组是在其中部署和管理 Azure 资源的逻辑容器。 

    New-AzureRmResourceGroup -Name myResourceGroup -Location chinanorth

## <a name="create-networking-resources"></a>创建网络资源

创建虚拟网络、子网和公共 IP 地址。 这些资源用来与虚拟机建立网络连接，以及连接到 Internet。

    # Create a subnet configuration
    $subnetConfig = New-AzureRmVirtualNetworkSubnetConfig -Name mySubnet -AddressPrefix 192.168.1.0/24

    # Create a virtual network
    $vnet = New-AzureRmVirtualNetwork -ResourceGroupName myResourceGroup -Location chinanorth `
    -Name MYvNET -AddressPrefix 192.168.0.0/16 -Subnet $subnetConfig

    # Create a public IP address and specify a DNS name
    $pip = New-AzureRmPublicIpAddress -ResourceGroupName myResourceGroup -Location chinanorth `
    -AllocationMethod Static -IdleTimeoutInMinutes 4 -Name "mypublicdns$(Get-Random)"

创建网络安全组和网络安全组规则。 网络安全组使用入站和出站规则保护虚拟机。 在本例中，将为端口 22 创建一个入站规则，该规则允许传入的 SSH 连接。 我们还需要为端口 80 创建入站规则，以允许传入的 Web 流量。

    # Create an inbound network security group rule for port 22
    $nsgRuleSSH = New-AzureRmNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleSSH  -Protocol Tcp `
    -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
    -DestinationPortRange 22 -Access Allow

    # Create an inbound network security group rule for port 80
    $nsgRuleWeb = New-AzureRmNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleWWW  -Protocol Tcp `
    -Direction Inbound -Priority 1001 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
    -DestinationPortRange 80 -Access Allow

    # Create a network security group
    $nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName myResourceGroup -Location chinanorth `
    -Name myNetworkSecurityGroup -SecurityRules $nsgRuleSSH,$nsgRuleWeb

为虚拟机创建网卡。 网卡将虚拟机连接到子网、网络安全组和公共 IP 地址。

    # Create a virtual network card and associate with public IP address and NSG
    $nic = New-AzureRmNetworkInterface -Name myNic -ResourceGroupName myResourceGroup -Location chinanorth `
    -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id

## <a name="create-virtual-machine"></a>创建虚拟机

创建虚拟机配置。 此配置包括部署虚拟机时使用的设置，例如虚拟机映像、大小和身份验证配置。

    # Define a credential object
    $securePassword = ConvertTo-SecureString ' ' -AsPlainText -Force
    $cred = New-Object System.Management.Automation.PSCredential ("azureuser", $securePassword)

    # Create a virtual machine configuration
    $vmConfig = New-AzureRmVMConfig -VMName myVM -VMSize Standard_D1 | `
    Set-AzureRmVMOperatingSystem -Linux -ComputerName myVM -Credential $cred -DisablePasswordAuthentication | `
    Set-AzureRmVMSourceImage -PublisherName Canonical -Offer UbuntuServer -Skus 14.04.2-LTS -Version latest | `
    Add-AzureRmVMNetworkInterface -Id $nic.Id

    # Configure SSH Keys
    $sshPublicKey = Get-Content "$env:USERPROFILE\.ssh\id_rsa.pub"
    Add-AzureRmVMSshPublicKey -VM $vmconfig -KeyData $sshPublicKey -Path "/home/azureuser/.ssh/authorized_keys"

创建虚拟机。

    New-AzureRmVM -ResourceGroupName myResourceGroup -Location chinanorth -VM $vmConfig

## <a name="connect-to-virtual-machine"></a>连接到虚拟机

完成部署后，请与虚拟机建立 SSH 连接。

运行以下命令，返回虚拟机的公共 IP 地址。

    Get-AzureRmPublicIpAddress -ResourceGroupName myResourceGroup | Select IpAddress

在装有 SSH 的系统中，使用以下命令连接到虚拟机。 如果在 Windows 上操作，可以使用 [Putty](/documentation/articles/virtual-machines-linux-ssh-from-windows/#create-a-private-key-for-putty) 来创建连接。 

    ssh <Public IP Address>

出现提示时，请输入登录用户名 `azureuser`。 如果在创建 SSH 密钥时输入了通行短语，还需要输入此通行短语。

## <a name="install-nginx"></a>安装 NGINX

使用以下 bash 脚本更新包源并安装最新的 NGINX 包。 

    #!/bin/bash

    # update package source
    apt-get -y update

    # install NGINX
    apt-get -y install nginx

## <a name="view-the-ngix-welcome-page"></a>查看 NGIX 欢迎页

NGINX 已安装，并且现在已从 Internet 打开 VM 上的端口 80 - 可以使用所选的 Web 浏览器查看默认的 NGINX 欢迎页。 请务必使用前面记录的 `publicIpAddress` 访问默认页面。 

![NGINX 默认站点](./media/virtual-machines-linux-quick-create-cli/nginx.png) 
## <a name="delete-virtual-machine"></a>删除虚拟机

如果不再需要资源组、VM 和所有相关的资源，可以使用以下命令将其删除。

    Remove-AzureRmResourceGroup -Name myResourceGroup

## <a name="next-steps"></a>后续步骤

[创建高可用性虚拟机教程](/documentation/articles/virtual-machines-linux-create-cli-complete/)

[浏览 VM 部署 PowerShell 示例](/documentation/articles/virtual-machines-linux-powershell-samples/)

<!--Update_Description: add "opening port 80" and "installing NGINX"-->
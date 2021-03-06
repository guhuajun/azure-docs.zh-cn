---
title: Azure 磁盘加密先决条件 | Microsoft Docs
description: 本文提供了对 IaaS VM 使用 Microsoft Azure 磁盘加密所要满足的先决条件。
author: mestew
ms.service: security
ms.subservice: Azure Disk Encryption
ms.topic: article
ms.author: mstewart
ms.date: 09/14/2018
ms.openlocfilehash: ad8bf0217dcd07a7272a220f2d91ed6bc40523bc
ms.sourcegitcommit: 8b694bf803806b2f237494cd3b69f13751de9926
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/20/2018
ms.locfileid: "46498583"
---
# <a name="azure-disk-encryption-prerequisites"></a>Azure 磁盘加密先决条件 
 本文“Azure 磁盘加密先决条件”介绍了在使用 Azure 磁盘加密之前所要满足的条件。 Azure 磁盘加密与 [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/) 集成，以帮助管理加密密钥。 可以使用 [Azure PowerShell](/powershell/azure/overview)、[Azure CLI](/cli/azure/) 或 [Azure 门户](https://portal.azure.com)配置 Azure 磁盘加密。

针对 [Azure 磁盘加密概述](azure-security-disk-encryption-overview.md)一文中所述的受支持方案在 Azure IaaS VM 上启用 Azure 磁盘加密之前，请务必满足以下先决条件。 

> [!NOTE]
> 某些建议可能会导致数据、网络或计算资源使用量增加，从而产生额外许可或订阅成本。 必须具有有效的活动 Azure 订阅，才能在 Azure 的受支持区域中创建资源。


## <a name="bkmk_OSs"></a>支持的操作系统
以下操作系统支持 Azure 磁盘加密：

- Windows Server 版本：Windows Server 2008 R2、Windows Server 2012、Windows Server 2012 R2 和 Windows Server 2016
    - 对于 Windows Server 2008 R2，必须安装 .NET Framework 4.5 才能在 Azure 中启用加密。 通过安装可选更新“适用于 Windows Server 2008 R2 x64 系统的 Microsoft .NET Framework 4.5.2 ([KB2901983](https://support.microsoft.com/kb/2901983))”，从 Windows 更新安装该组件。    
- Windows 客户端版本：Windows 8 客户端和 Windows 10 客户端。
- 仅基于特定 Azure 库的 Linux 服务器分发和版本支持 Azure 磁盘加密。 有关当前受支持版本的列表，请参阅 [Azure 磁盘加密常见问题解答](azure-security-disk-encryption-faq.md#bkmk_LinuxOSSupport)。
- Azure 磁盘加密要求 Key Vault 和 VM 位于同一 Azure 区域和订阅。 在不同区域中配置资源会导致启用 Azure 磁盘加密功能失败。

## <a name="bkmk_LinuxPrereq"></a>适用于 Linux Iaas VM 的其他先决条件 

- 要在[支持的映像](azure-security-disk-encryption-faq.md#bkmk_LinuxOSSupport)中启用 OS 磁盘加密，适用于 Linux 的 Azure 磁盘加密要求 VM 上有 7 GB RAM。 完成 OS 磁盘加密过程后，可将 VM 配置为以更少的内存运行。
- 在启用加密之前，要加密的数据磁盘需在 /etc/fstab 中正确列出。 为此条目使用永久性块设备名，因为每次重新启动后，不能依赖于使用“/dev/sdX”格式的设备名来与同一磁盘相关联，尤其是应用加密后。 有关此行为的详细信息，请参阅：[排查 Linux VM 设备名更改问题](../virtual-machines/linux/troubleshoot-device-names-problems.md)
- 确保正确配置用于装载的 /etc/fstab 设置。 若要配置这些设置，请运行 mount -a 命令，或重新启动 VM 并以这种方法触发重新装载。 装载完成后，检查 lsblk 命令的输出，以验证驱动器是否仍已装载。 
    - 如果在启用加密之前 /etc/fstab 文件未正确装载该驱动器，则 Azure 磁盘加密无法将其正确装载。
    - 在加密过程中，Azure 磁盘加密进程会将装载信息移出 /etc/fstab，并移入其自身的配置文件中。 数据驱动器加密完成后，如果看到 /etc/fstab 中缺少条目，请不要担心。
    -  重新启动后，Azure 磁盘加密进程需要花费一段时间来装载新加密的磁盘。 重新启动后，这些磁盘并不是立即可用。 该进程需要一段时间来启动、解锁然后装载加密的驱动器，然后，这些驱动器才可供其他进程访问。 重新启动后，此进程可能需要一分钟以上的时间，具体时间取决于系统特征。

在[此脚本文件的第 244-248 行](https://github.com/ejarvi/ade-cli-getting-started/blob/master/validate.sh#L244-L248)可以找到用于装载数据磁盘和创建所需 /etc/fstab 条目的命令示例。 


## <a name="bkmk_GPO"></a>网络和组策略

**若要启用 Azure 磁盘加密功能，IaaS VM 必须符合以下网络终结点配置要求：**
  - IaaS VM 必须能够连接到 Azure Active Directory 终结点 \[login.microsoftonline.com\]，以获取用于连接 Key Vault 的令牌。
  - IaaS VM 必须能够连接到 Key Vault 终结点，以将加密密钥写入 Key Vault。
  - IaaS VM 必须能够连接到托管 Azure 扩展存储库的 Azure 存储终结点和托管 VHD 文件的 Azure 存储帐户。
  -  如果安全策略限制从 Azure VM 到 Internet 的访问，可以解析上述 URI，并配置特定的规则以允许与这些 IP 建立出站连接。 有关详细信息，请参阅[防火墙后的 Azure Key Vault](../key-vault/key-vault-access-behind-firewall.md)。    


**组策略：**
 - Azure 磁盘加密解决方案对 Windows IaaS VM 使用 BitLocker 外部密钥保护程序。 对于已加入域的 VM，请不要推送会强制执行 TPM 保护程序的任何组策略。 有关“在没有兼容 TPM 的情况下允许 BitLocker”的组策略信息，请参阅 [BitLocker 组策略参考](https://docs.microsoft.com/windows/security/information-protection/bitlocker/bitlocker-group-policy-settings#a-href-idbkmk-unlockpol1arequire-additional-authentication-at-startup)。

-  具有自定义组策略的已加入域的虚拟机上的 Bitlocker 策略必须包含以下设置：[配置 bitlocker 恢复信息的用户存储 -> 允许 256 位恢复密钥](https://docs.microsoft.com/windows/security/information-protection/bitlocker/bitlocker-group-policy-settings)。 如果 Bitlocker 的自定义组策略设置不兼容，Azure 磁盘加密将会失败。 在没有正确策略设置的计算机上，应用新策略，强制更新新策略 (gpupdate.exe /force)，然后可能需要重启。  


## <a name="bkmk_PSH"></a>Azure PowerShell
[Azure PowerShell](/powershell/azure/overview) 提供了一组使用 [Azure 资源管理器](../azure-resource-manager/resource-group-overview.md)模型管理 Azure 资源的 cmdlet。 可以在浏览器中结合 [Azure Cloud Shell](../cloud-shell/overview.md) 使用 PowerShell，或者遵照以下说明将 PowerShell 安装在本地计算机上，以便在任何 PowerShell 会话中使用这些 cmdlet。 如果已在本地安装 PowerShell，请确保使用最新版本的 Azure PowerShell SDK 来配置 Azure 磁盘加密。 下载最新版本的 [Azure PowerShell 版本](https://github.com/Azure/azure-powershell/releases)。

### <a name="install-azure-powershell-for-use-on-your-local-machine-optional"></a>安装在本地计算机上使用的 Azure PowerShell（可选）： 
1. 遵照适用于操作系统的链接中的说明，然后继续完成下面的剩余步骤。      
    - [安装并配置适用于 Windows 的 Azure PowerShell](/powershell/azure/install-azurerm-ps)。 
        - 安装 PowerShellGet、Azure PowerShell，并加载 AzureRM 模块。 
    - [在 macOS 和 Linux 上安装并配置 Azure PowerShell](/powershell/azure/install-azurermps-maclinux)。
        -  安装 PowerShell Core、Azure PowerShell for .NET Core，并加载 Az 模块。

2. 验证安装的 AzureRM 模块版本。 如果需要，请[更新 Azure PowerShell 模块](/powershell/azure/install-azurerm-ps#update-the-azure-powershell-module)。
    -  AzureRM 模块版本需要是 6.0.0 或更高版本。
    - 建议使用最新的 AzureRM 模块版本。

     ```powershell
     Get-Module AzureRM -ListAvailable | Select-Object -Property Name,Version,Path
     ```

3. 使用 [Connect-AzureRmAccount](/powershell/module/azurerm.profile/connect-azurermaccount) cmdlet 登录到 Azure。
     
     ```azurepowershell-interactive
     Connect-AzureRmAccount
     # For specific instances of Azure, use the -Environment parameter.
     Connect-AzureRmAccount –Environment (Get-AzureRmEnvironment –Name AzureUSGovernment)
    
     <# If you have multiple subscriptions and want to specify a specific one, 
     get your subscription list with Get-AzureRmSubscription and 
     specify it with Set-AzureRmContext.  #>
     Get-AzureRmSubscription
     Set-AzureRmContext -SubscriptionId "xxxx-xxxx-xxxx-xxxx"
     ```

4.  根据需要查看 [Azure PowerShell 入门](/powershell/azure/get-started-azureps)。

## <a name="bkmk_CLI"></a> 安装在本地计算机上使用的 Azure CLI（可选）

[Azure CLI 2.0](/cli/azure) 是用于管理 Azure 资源的命令行工具。 CLI 旨在提高数据查询灵活性、支持非阻塞进程形式的长时间操作，以及简化脚本编写。 可以在浏览器中结合 [Azure Cloud Shell](../cloud-shell/overview.md) 使用这些 cmdlet，或者将它们安装在本地计算机上并在任何 PowerShell 会话中使用。

1. 安装在本地计算机上使用的 [Azure CLI](/cli/azure/install-azure-cli)（可选）：

2. 验证安装的版本。
     
     ```azurecli-interactive
     az --version
     ``` 

3. 使用 [az login](/cli/azure/authenticate-azure-cli) 登录到 Azure。
     
     ```azurecli
     az login
     
     # If you would like to select a tenant, use: 
     az login --tenant "<tenant>"

     # If you have multiple subscriptions, get your subscription list with az account list and specify with az account set.
     az account list
     az account set --subscription "<subscription name or ID>"
     ```

4. 根据需要查看 [Azure CLI 2.0 入门](/cli/azure/get-started-with-azure-cli)。 


## <a name="prerequisite-workflow-for-key-vault"></a>Key Vault 的先决条件工作流
如果你已熟悉进行 Azure 磁盘加密时的 Key Vault 和 Azure AD 先决条件，则可以使用 [Azure 磁盘加密先决条件 PowerShell 脚本](https://raw.githubusercontent.com/Azure/azure-powershell/master/src/ResourceManager/Compute/Commands.Compute/Extension/AzureDiskEncryption/Scripts/AzureDiskEncryptionPreRequisiteSetup.ps1 )。 有关使用先决条件脚本的详细信息，请参阅[加密 VM 快速入门](quick-encrypt-vm-powershell.md)和 [Azure 磁盘加密附录](azure-security-disk-encryption-appendix.md#bkmk_prereq-script)。 

1. 如果需要，请创建资源组。
2. 创建密钥保管库。 
3. 设置 Key Vault 高级访问策略。

>[!WARNING]
>在删除密钥保管库之前，请确保未使用它加密任何现有 VM。 要防止意外删除保管库，请在保管库上[启用软删除](../key-vault/key-vault-soft-delete-powershell.md#enabling-soft-delete)和[资源锁](../azure-resource-manager/resource-group-lock-resources.md)。 
 
## <a name="bkmk_KeyVault"></a>创建密钥保管库 
Azure 磁盘加密与 [Azure Key Vault](https://azure.microsoft.com/documentation/services/key-vault/) 集成，帮助你控制和管理 Key Vault 订阅中的磁盘加密密钥与机密。 可为 Azure 磁盘加密创建 Key Vault，或使用现有的 Key Vault。 有关 Key Vault 的详细信息，请参阅 [Azure Key Vault 入门](../key-vault/key-vault-get-started.md)和[保护 Key Vault](../key-vault/key-vault-secure-your-key-vault.md)。 可以使用资源管理器模板、Azure PowerShell 或 Azure CLI 创建 Key Vault。 


>[!WARNING]
>为确保加密机密不会跨过区域边界，Azure 磁盘加密需要将 Key Vault 和 VM 共置在同一区域。 在要加密的 VM 所在的同一区域中创建并使用 Key Vault。 


### <a name="bkmk_KVPSH"></a>使用 PowerShell 创建 Key Vault

可以在 Azure PowerShell 中使用 [New-AzureRmKeyVault](/powershell/module/azurerm.keyvault/New-AzureRmKeyVault) cmdlet 创建 Key Vault。 有关适用于 Key Vault 的更多 cmdlet，请参阅 [AzureRM.KeyVault](/powershell/module/azurerm.keyvault/)。 

1. 根据需要[连接到 Azure 订阅](azure-security-disk-encryption-appendix.md#bkmk_ConnectPSH)。 
2. 根据需要，使用 [New-AzureRmResourceGroup](/powershell/module/AzureRM.Resources/New-AzureRmResourceGroup) 创建新资源组。  若要列出数据中心位置，请使用 [Get-AzureRmLocation](/powershell/module/azurerm.resources/get-azurermlocation)。 
     
     ```azurepowershell-interactive
     # Get-AzureRmLocation 
     New-AzureRmResourceGroup –Name 'MySecureRG' –Location 'East US'
     ```

3. 使用 [New-AzureRmKeyVault](/powershell/module/azurerm.keyvault/New-AzureRmKeyVault) 创建新的 Key Vault
    
      ```azurepowershell-interactive
     New-AzureRmKeyVault -VaultName 'MySecureVault' -ResourceGroupName 'MySecureRG' -Location 'East US'
     ```

4. 记下返回的“保管库名称”、“资源组名称”、“资源 ID”、“保管库 URI”和“对象 ID”，以便稍后在加密磁盘时使用。 


### <a name="bkmk_KVCLI"></a>使用 Azure CLI 创建 Key Vault
可以在 Azure CLI 中使用 [az keyvault](/cli/azure/keyvault#commands) 命令管理 Key Vault。 若要创建 Key Vault，请使用 [az keyvault create](/cli/azure/keyvault#az-keyvault-create)。

1. 根据需要[连接到 Azure 订阅](azure-security-disk-encryption-appendix.md#bkmk_ConnectCLI)。
2. 根据需要，使用 [az group create](/cli/azure/group#az-group-create) 创建新资源组。 若要列出位置，请使用 [az account list-locations](/cli/azure/account#az-account-list) 
     
     ```azurecli-interactive
     # To list locations: az account list-locations --output table
     az group create -n "MySecureRG" -l "East US"
     ```

3. 使用 [az keyvault create](/cli/azure/keyvault#az-keyvault-create) 创建新 Key Vault。
    
     ```azurecli-interactive
     az keyvault create --name "MySecureVault" --resource-group "MySecureRG" --location "East US"
     ```

4. 记下返回的“保管库名称”(name)、“资源组名称”、“资源 ID”(ID)、“保管库 URI”和“对象 ID”，以便稍后使用。 

### <a name="bkmk_KVRM"></a>使用资源管理器模板创建 Key Vault

可以使用[资源管理器模板](https://github.com/Azure/azure-quickstart-templates/tree/master/101-key-vault-create)创建 Key Vault。

1. 在 Azure 快速入门模板中，单击“部署到 Azure”。
2. 选择订阅、资源组、资源组位置、Key Vault 名称、对象 ID、法律条款和协议，然后单击“购买”。 


## <a name="bkmk_KVper"></a> 设置 Key Vault 高级访问策略
Azure 平台需要访问 Key Vault 中的加密密钥或机密，才能使这些密钥和机密可供 VM 用来启动和解密卷。 对 Key Vault 启用磁盘加密，否则部署将会失败。  

### <a name="bkmk_KVperPSH"></a>使用 Azure PowerShell 设置 Key Vault 高级访问策略
 使用 Key Vault PowerShell cmdlet [Set-AzureRmKeyVaultAccessPolicy](/powershell/module/azurerm.keyvault/set-azurermkeyvaultaccesspolicy) 为 Key Vault 启用磁盘加密。

  - **为磁盘加密启用 Key Vault：** 需要使用 EnabledForDiskEncryption 来启用 Azure 磁盘加密。
      
     ```azurepowershell-interactive 
     Set-AzureRmKeyVaultAccessPolicy -VaultName 'MySecureVault' -ResourceGroupName 'MySecureRG' -EnabledForDiskEncryption
     ```

  - **根据需要为部署启用 Key Vault：** 在资源创建操作中引用此 Key Vault（例如，创建虚拟机）时，使 Microsoft.Compute 资源提供程序能够从此 Key Vault 中检索机密。

     ```azurepowershell-interactive
      Set-AzureRmKeyVaultAccessPolicy -VaultName 'MySecureVault' -ResourceGroupName 'MySecureRG' -EnabledForDeployment
     ```

  - **根据需要为模板部署启用 Key Vault：** 在模板部署中引用此 Key Vault 时，使 Azure 资源管理器模板能够从此 Key Vault 中检索机密。

     ```azurepowershell-interactive             
     Set-AzureRmKeyVaultAccessPolicy -VaultName 'MySecureVault' -ResourceGroupName 'MySecureRG' -EnabledForTemplateDeployment`
     ```

### <a name="bkmk_KVperCLI"></a>使用 Azure CLI 设置 Key Vault 高级访问策略
使用 [az keyvault update](/cli/azure/keyvault#az-keyvault-update) 为 Key Vault 启用磁盘加密。 

 - **为磁盘加密启用 Key Vault：** 需要使用 Enabled-for-disk-encryption。 

     ```azurecli-interactive
     az keyvault update --name "MySecureVault" --resource-group "MySecureRG" --enabled-for-disk-encryption "true"
     ```  

 - **根据需要为部署启用 Key Vault：** 在资源创建操作中引用此 Key Vault（例如，创建虚拟机）时，使 Microsoft.Compute 资源提供程序能够从此 Key Vault 中检索机密。

     ```azurecli-interactive
     az keyvault update --name "MySecureVault" --resource-group "MySecureRG" --enabled-for-deployment "true"
     ``` 

 - **根据需要为模板部署启用 Key Vault**：允许资源管理器从保管库中检索机密。
     ```azurecli-interactive  
     az keyvault update --name "MySecureVault" --resource-group "MySecureRG" --enabled-for-template-deployment "true"
     ```


### <a name="bkmk_KVperrm"></a>通过 Azure 门户设置 Key Vault 高级访问策略

1. 选择 Key Vault，转到“访问策略”，然后选择“单击此处可显示高级访问策略”。
2. 选中标有“启用对 Azure 磁盘加密的访问以进行卷加密”的框。
3. 根据需要选择“启用对 Azure 虚拟机的访问以进行部署”和/或“启用对 Azure 资源管理器的访问以进行模板部署”。 
4. 单击“ **保存**”。

![Azure Key Vault 高级访问策略](./media/azure-security-disk-encryption/keyvault-portal-fig4.png)


## <a name="bkmk_KEK"></a>设置密钥加密密钥（可选）
若要使用密钥加密密钥 (KEK) 来为加密密钥提供附加的安全层，请将 KEK 添加到 Key Vault。 使用 [Add-AzureKeyVaultKey](/powershell/module/azurerm.keyvault/add-azurekeyvaultkey) cmdlet 在 Key Vault 中创建密钥加密密钥。 还可从本地密钥管理 HSM 导入 KEK。 有关详细信息，请参阅 [Key Vault 文档](../key-vault/key-vault-hsm-protected-keys.md)。 指定密钥加密密钥后，Azure 磁盘加密会使用该密钥包装加密机密，然后将机密写入 Key Vault。 

* Key Vault 机密和 KEK URL 必须已设置版本。 Azure 会强制实施这项版本控制限制。 有关有效的机密和 KEK URL，请参阅以下示例：

  * 有效机密 URL 的示例：   *https://contosovault.vault.azure.net/secrets/EncryptionSecretWithKek/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx*
  * 有效 KEK URL 的示例：   *https://contosovault.vault.azure.net/keys/diskencryptionkek/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx*

* Azure 磁盘加密不支持将端口号指定为 Key Vault 机密和 KEK URL 的一部分。 有关不支持和支持的 Key Vault URL 的示例，请参阅以下示例：

  * 无法接受的密钥保管库 URL：   *https://contosovault.vault.azure.net:443/secrets/contososecret/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx*
  * 可接受的密钥保管库 URL：   *https://contosovault.vault.azure.net/secrets/contososecret/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx*

### <a name="bkmk_KEKPSH"></a>使用 Azure PowerShell 设置密钥加密密钥 
在使用 PowerShell 脚本之前，应熟悉 Azure 磁盘加密必备组件，以了解脚本中的步骤。 可能需要根据环境更改示例脚本。 此脚本创建所有 Azure 磁盘加密必备组件、加密现有 IaaS VM，并使用密钥加密密钥来包装磁盘加密密钥。 

 ```powershell
 # Step 1: Create a new resource group and key vault in the same location.
     # Fill in 'MyLocation', 'MySecureRG', and 'MySecureVault' with your values.
     # Use Get-AzureRmLocation to get available locations and use the DisplayName.
     # To use an existing resource group, comment out the line for New-AzureRmResourceGroup
     
     $Loc = 'MyLocation';
     $rgname = 'MySecureRG';
     $KeyVaultName = 'MySecureVault'; 
     New-AzureRmResourceGroup –Name $rgname –Location $Loc;
     New-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname -Location $Loc;
     $KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname;
     $KeyVaultResourceId = (Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname).ResourceId;
     $diskEncryptionKeyVaultUrl = (Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname).VaultUri;
     
 #Step 2: Enable the vault for disk encryption.
     Set-AzureRmKeyVaultAccessPolicy -VaultName $KeyVaultName -ResourceGroupName $rgname -EnabledForDiskEncryption;
      
 #Step 3: Create a new key in the key vault with the Add-AzureKeyVaultKey cmdlet.
     # Fill in 'MyKeyEncryptionKey' with your value.
     
     $keyEncryptionKeyName = 'MyKeyEncryptionKey';
     Add-AzureKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName -Destination 'Software';
     $keyEncryptionKeyUrl = (Get-AzureKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;
     
 #Step 4: Encrypt the disks of an existing IaaS VM
     # Fill in 'MySecureVM' with your value. 
     
     $VMName = 'MySecureVM';
     Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId;
```


 
## <a name="next-steps"></a>后续步骤
> [!div class="nextstepaction"]
> [启用适用于 Windows 的 Azure 磁盘加密](azure-security-disk-encryption-windows.md)

> [!div class="nextstepaction"]
> [启用适用于 Linux 的 Azure 磁盘加密](azure-security-disk-encryption-linux.md)


---
title: 使用 Azure 自动化跟踪更改
description: 借助更改跟踪解决方案可以确定环境中发生的软件和 Windows 服务更改。
services: automation
ms.service: automation
ms.component: change-inventory-management
author: georgewallace
ms.author: gwallace
ms.date: 10/12/2018
ms.topic: conceptual
manager: carmonm
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 2678b9a1b80b1c9de6f1b554ce43bcd4f2dd5d50
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/12/2018
ms.locfileid: "49166995"
---
# <a name="track-changes-in-your-environment-with-the-change-tracking-solution"></a>使用更改跟踪解决方案跟踪环境中的更改

本文可帮助你使用更改跟踪解决方案轻松识别环境中的更改。 该解决方案会跟踪对 Windows 和 Linux 软件、Windows 和 Linux 文件、Windows 注册表项、Windows 服务和 Linux 守护程序进行的更改。 标识配置更改有助于确定操作问题。

对受监视服务器上已安装的软件、Windows 服务、Windows 注册表和文件以及 Linux 守护程序进行的更改将发送到云中的 Log Analytics 服务进行处理。 逻辑应用于接收的数据，云服务则记录数据。 通过使用“更改跟踪”仪表板上的信息，可以轻松查看服务器基础结构中所做的更改。

## <a name="supported-windows-operating-systems"></a>支持的 Windows 操作系统

Windows 代理官方支持以下版本的 Windows 操作系统：

* Windows Server 2008 R2 或更高版本

## <a name="supported-linux-operating-systems"></a>受支持的 Linux 操作系统

以下 Linux 分发版受官方支持。 不过，Linux 代理也可在未列出的其他分发版上运行。 除非另行说明，列出每个主版本支持所有的次版本。  

### <a name="64-bit"></a>64 位

* CentOS 6 和 7
* Amazon Linux 2017.09
* Oracle Linux 6 和 7
* Red Hat Enterprise Linux Server 6 和 7
* Debian GNU/Linux 8 和 9
* Ubuntu Linux 14.04 LTS、16.04 LTS 和 18.04 LTS
* SUSE Linux Enterprise Server 12

### <a name="32-bit"></a>32 位

* CentOS 6
* Oracle Linux 6
* Red Hat Enterprise Linux Server 6
* Debian GNU/Linux 8 和 9
* Ubuntu Linux 14.04 LTS 和 16.04 LTS

## <a name="enable-change-tracking-and-inventory"></a>启用更改跟踪和库存

若要开始跟踪更改，需要为自动化帐户启用更改跟踪和库存解决方案。

1. 在 Azure 门户中，导航到自动化帐户
2. 选择“配置”下的“更改跟踪”。
3. 选择现有的 Log Analytics 工作区，或“创建新工作区”并单击“启用”。

此操作将启用自动化帐户的解决方案。 启用解决方案最多可能需要 15 分钟。 启用解决方案后，蓝色横幅会通知你。 导航回到“更改跟踪”页以管理解决方案。

## <a name="configuring-change-tracking-and-inventory"></a>配置更改跟踪和库存

若要了解如何将计算机载入到解决方案中，请访问：[载入自动化解决方案](automation-onboard-solutions-from-automation-account.md)。 有了通过更改跟踪和库存解决方案载入的计算机后，便可以配置要跟踪的项了。启用的新文件或注册表项来跟踪时，将会为更改跟踪和库存启用它。

若要跟踪 Windows 和 Linux 上文件中的更改，请使用文件的 MD5 哈希。 然后使用这些哈希检测自上一个库存以来是否进行了更改。

### <a name="configure-linux-files-to-track"></a>配置要跟踪的 Linux 文件

使用以下步骤，在 Linux 计算机上配置要跟踪的文件：

1. 在自动化帐户中，选择“配置管理”下的“更改跟踪”。 单击“编辑设置”（齿轮符号）。
2. 在“更改跟踪”页上，选择“Linux 文件”，然后单击“+ 添加”以添加要跟踪的新文件。
3. 在“添加 Linux 文件以更改跟踪”中，输入要跟踪的文件或目录的信息，然后单击“保存”。

|属性  |说明  |
|---------|---------|
|已启用     | 确定是否应用了设置。        |
|项名称     | 要跟踪的文件的友好名称。        |
|组     | 用于对文件进行逻辑分组的组名。        |
|输入路径     | 要检查的文件路径。 例如："/etc/* .conf"       |
|路径类型     | 要跟踪的项类型，可能值为“文件”和“目录”。        |
|递归     | 在查找要跟踪的项时，确定是否使用递归。        |
|使用 Sudo     | 此设置确定在检查该项时是否使用 Sudo。         |
|链接     | 此设置确定在遍历目录时如何处理符号链接。<br> **忽略** - 忽略符号链接，不包括引用的文件/目录。<br>**追随** - 在递归期间追随符号链接，并且包含引用的文件/目录。<br>**管理** - 追随符号链接并允许更改返回的内容。     |
|上传所有设置的文件内容| 针对已跟踪的更改启用或关闭文件内容上传功能。 可用选项：“True”或“False”。|

> [!NOTE]
> 不建议使用“管理”链接选项。 不支持文件内容检索。

### <a name="configure-windows-files-to-track"></a>配置要跟踪的 Windows 文件

使用以下步骤，在 Windows 计算机上配置要跟踪的文件：

1. 在自动化帐户中，选择“配置管理”下的“更改跟踪”。 单击“编辑设置”（齿轮符号）。
2. 在“更改跟踪”页上，选择“Windows 文件”，然后单击“+ 添加”以添加要跟踪的新文件。
3. 在“添加用于更改跟踪的 Windows 文件”中，输入要求该文件进行跟踪的信息，然后单击“保存”。

|属性  |说明  |
|---------|---------|
|已启用     | 确定是否应用了设置。        |
|项名称     | 要跟踪的文件的友好名称。        |
|组     | 用于对文件进行逻辑分组的组名。        |
|输入路径     | 用于查看文件的路径，例如：“c:\temp\\\*.txt”<br>还可以使用环境变量，例如“%winDir%\System32\\\*.*”       |
|递归     | 在查找要跟踪的项时，确定是否使用递归。        |
|上传所有设置的文件内容| 针对已跟踪的更改启用或关闭文件内容上传功能。 可用选项：“True”或“False”。|

## <a name="wildcard-recursion-and-environment-settings"></a>通配符、递归和环境设置

借助递归可指定通配符以简化目录上的跟踪，借助环境变量可在具有多个或动态驱动器名称的环境中跟踪文件。 以下是配置递归时应了解的常用信息列表：

* 跟踪多个文件时需要使用通配符
* 如果使用通配符，则只能在路径的最后一段中使用。 （例如 C:\folder\\file 或 /etc/*.conf）
* 如果环境变量具有无效路径，验证将成功，但库存运行时，该路径将失败。
* 设置路径时，请避免使用 `c:\*.*` 等常规路径，因为这会导致遍历过多的文件夹。

## <a name="configure-file-content-tracking"></a>配置文件内容跟踪

可以使用文件内容更改跟踪功能查看文件更改之前和之后的内容。 这适用于 Windows 和 Linux 文件，每次更改文件时，文件的内容都存储在存储帐户中，并以内联或并排的方式显示更改之前和之后的文件。 若要了解详细信息，请参阅[查看所跟踪文件的内容](change-tracking-file-contents.md)。

![查看文件中的更改](./media/change-tracking-file-contents/view-file-changes.png)

### <a name="configure-windows-registry-keys-to-track"></a>配置要跟踪的 Windows 注册表项

使用以下步骤，配置要在 Windows 计算机上跟踪的注册表项：

1. 在自动化帐户中，选择“配置管理”下的“更改跟踪”。 单击“编辑设置”（齿轮符号）。
2. 在“更改跟踪”页上，选择“Windows 注册表”，然后单击“+ 添加”以添加要跟踪的新注册表项。
3. 在“添加用于更改跟踪的 Windows 注册表”中，输入要求该项进行跟踪的信息，然后单击“保存”。

|属性  |说明  |
|---------|---------|
|已启用     | 确定是否应用了设置。        |
|项名称     | 要跟踪的文件的友好名称。        |
|组     | 用于对文件进行逻辑分组的组名。        |
|Windows 注册表项   | 要检查的文件路径。 例如：“HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders\Common Startup”      |

## <a name="limitations"></a>限制

更改跟踪解决方案当前不支持以下项：

* Windows 注册表跟踪递归
* 网络文件系统

其他限制：

* 未在当前实现中使用“最大文件大小”列和值。
* 如果在 30 分钟收集周期内收集 2500 多个文件，则解决方案性能可能会下降。
* 当流量较高时，更改记录可能需要最多六个小时才能显示。
* 如果在计算机关闭的情况下修改配置，计算机可能会发布属于以前配置的更改。

## <a name="known-issues"></a>已知问题

更改跟踪解决方案当前遇到以下问题：

* 没有为 Windows 10 创意者更新和 Windows Server 2016 Core RS3 计算机收集修补程序更新。
* 对于 Windows 文件，更改跟踪当前无法检测将新文件添加到跟踪文件夹路径的时间

## <a name="change-tracking-data-collection-details"></a>更改跟踪数据收集详细信息

下表显示了各种更改类型的数据收集频率。 对于每种类型，当前状态的数据快照也至少每 24 小时刷新一次：

| **更改类型** | **频率** |
| --- | --- |
| Windows 注册表 | 50 分钟 |
| Windows 文件 | 30 分钟 |
| Linux 文件 | 15 分钟 |
| Windows 服务 | 10 秒到 30 分钟</br> 默认值：30 分钟 |
| Linux 守护程序 | 5 分钟 |
| Windows 软件 | 30 分钟 |
| Linux 软件 | 5 分钟 |

### <a name="windows-service-tracking"></a>Windows 服务跟踪

Windows 服务的默认收集频率为 30 分钟。 若要配置该频率，请转到“更改跟踪”。 在“Windows 服务”选项卡上的“编辑设置”下，有一个滑块，可用于将 Windows 服务的收集频率从短短 10 秒更改为长达 30 分钟。 请将滑块移至所需的频率，它会自动进行保存。

![Windows 服务滑块](./media/automation-change-tracking/windowservices.png)

代理仅跟踪更改，这可以优化代理的性能。 将阈值设置得过高时，如果服务还原到其原始状态，则可能错过更改。 将频率设置为较小的值可以捕获可能会错过的更改。

> [!NOTE]
> 虽然代理可以按 10 秒的间隔跟踪更改，但数据仍要在几分钟后才能显示在门户中。 代理仍会跟踪和记录该事件内在门户中显示的更改。
  
### <a name="registry-key-change-tracking"></a>注册表项更改跟踪

监视注册表项更改的目的是确定第三方代码和恶意软件可以激活的扩展点。 以下列表显示预配置的注册表项的列表。 配置了这些密钥，但未启用。 若要跟踪这些注册表项，必须启用每个项。

> [!div class="mx-tdBreakAll"]
> |  |
> |---------|
> |**HKEY\_LOCAL\_MACHINE\Software\Classes\Directory\ShellEx\ContextMenuHandlers**     |
|&nbsp;&nbsp;&nbsp;&nbsp;监视常见的自动启动条目，这些条目直接与 Windows 资源管理器挂钩，并且通常使用 Explorer.exe 在进程内运行。    |
> |**HKEY\_LOCAL\_MACHINE\Software\Microsoft\Windows\CurrentVersion\Group Policy\Scripts\Startup**     |
|&nbsp;&nbsp;&nbsp;&nbsp;监视启动时运行的脚本。     |
> |**HKEY\_LOCAL\_MACHINE\Software\Microsoft\Windows\CurrentVersion\Group Policy\Scripts\Shutdown**    |
|&nbsp;&nbsp;&nbsp;&nbsp;监视关闭时运行的脚本。     |
> |**HKEY\_LOCAL\_MACHINE\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Run**     |
|&nbsp;&nbsp;&nbsp;&nbsp;监视用户登录 Windows 帐户之前加载的注册表项。 该注册表项用于在 64 位计算机上运行的 32 位程序。    |
> |**HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Active Setup\Installed Components**     |
|&nbsp;&nbsp;&nbsp;&nbsp;监视应用程序设置的更改。     |
> |**HKEY\_LOCAL\_MACHINE\Software\Classes\Directory\ShellEx\ContextMenuHandlers**|
|&nbsp;&nbsp;&nbsp;&nbsp;监视常见的自动启动条目，这些条目直接与 Windows 资源管理器挂钩，并且通常使用 Explorer.exe 在进程内运行。|
> |**HKEY\_LOCAL\_MACHINE\Software\Classes\Directory\Shellex\CopyHookHandlers**|
|&nbsp;&nbsp;&nbsp;&nbsp;监视常见的自动启动条目，这些条目直接与 Windows 资源管理器挂钩，并且通常使用 Explorer.exe 在进程内运行。|
> |**HKEY\_LOCAL\_MACHINE\Software\Microsoft\Windows\CurrentVersion\Explorer\ShellIconOverlayIdentifiers**|
|&nbsp;&nbsp;&nbsp;&nbsp;监视图标覆盖处理程序注册。|
|**HKEY\_LOCAL\_MACHINE\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Explorer\ShellIconOverlayIdentifiers**|
|&nbsp;&nbsp;&nbsp;&nbsp;监视在 64 位计算机上运行的 32 位程序的图标覆盖处理程序注册。|
> |**HKEY\_LOCAL\_MACHINE\Software\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects**|
|&nbsp;&nbsp;&nbsp;&nbsp;监视 Internet Explorer 的新浏览器帮助程序对象插件。 用于访问当前页的文档对象模型 (DOM) 并控制导航。|
> |**HKEY\_LOCAL\_MACHINE\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects**|
|&nbsp;&nbsp;&nbsp;&nbsp;监视 Internet Explorer 的新浏览器帮助程序对象插件。 对于在 64 位计算机上运行的 32 位程序，用于访问当前页的文档对象模型 (DOM) 并控制导航。|
> |**HKEY\_LOCAL\_MACHINE\Software\Microsoft\Internet Explorer\Extensions**|
|&nbsp;&nbsp;&nbsp;&nbsp;监视新的 Internet Explorer 扩展，如自定义工具菜单和自定义工具栏按钮。|
> |**HKEY\_LOCAL\_MACHINE\Software\Wow6432Node\Microsoft\Internet Explorer\Extensions**|
|&nbsp;&nbsp;&nbsp;&nbsp;对于在 64 位计算机上运行的 32 位程序，监视新的 Internet Explorer 扩展，如自定义工具菜单和自定义工具栏按钮。|
> |**HKEY\_LOCAL\_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\Drivers32**|
|&nbsp;&nbsp;&nbsp;&nbsp;监视与 wavemapper、wave1 和 wave2、msacm.imaadpcm、.msadpcm、.msgsm610 和 vidc 关联的 32 位驱动程序。 类似于 SYSTEM.INI 文件中的 [drivers] 部分。|
> |**HKEY\_LOCAL\_MACHINE\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32**|
|&nbsp;&nbsp;&nbsp;&nbsp;对于在 64 位计算机上运行的 32 位程序，监视与 wavemapper、wave1 和 wave2、msacm.imaadpcm、.msadpcm、.msgsm610 和 vidc 关联的 32 位驱动程序。 类似于 SYSTEM.INI 文件中的 [drivers] 部分。|
> |**HKEY\_LOCAL\_MACHINE\System\CurrentControlSet\Control\Session Manager\KnownDlls**|
|&nbsp;&nbsp;&nbsp;&nbsp;监视已知或常用的系统 DLL 列表；该系统可以通过删除特洛伊木马版本的系统 DLL 来防止人们利用弱应用程序目录权限。|
> |**HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Notify**|
|&nbsp;&nbsp;&nbsp;&nbsp;监视可从 Winlogon（适用于 Windows 操作系统的交互式登录支持模型）接收事件通知的包列表。|

## <a name="network-requirements"></a>网络要求

更改跟踪特别需要以下地址。 与这些地址的通信通过端口 443 完成。

|Azure Public  |Azure Government   |
|---------|---------|
|*.ods.opinsights.azure.com     |*.ods.opinsights.azure.us         |
|*.oms.opinsights.azure.com     | *.oms.opinsights.azure.us        |
|* .blob.core.windows.net|*.blob.core.usgovcloudapi.net|
|* .azure-automation.net|*.azure-automation.us|

## <a name="use-change-tracking"></a>使用更改跟踪

启用解决方案后，可通过在自动化帐户中的“配置管理”下选择“更改跟踪”来查看受监视计算机的更改摘要。

可以查看对计算机所做的更改，然后深入了解每个事件的详细信息。 可在图表顶部找到下拉列表，根据更改类型和时间范围限制图表和详细信息。 还可以单击并拖动图表，选择自定义时间范围。

![“更改跟踪”仪表板的图像](./media/automation-change-tracking/change-tracking-dash01.png)

单击更改或事件将显示有关该更改的详细信息。 如示例中所示，该服务的启动类型已从“手动”更改为“自动”。

![更改跟踪详细信息的图像](./media/automation-change-tracking/change-tracking-details.png)

## <a name="search-logs"></a>搜索日志

除了门户中提供的详细信息以外，还可以针对日志执行搜索。 在打开“更改跟踪”页的情况下，单击“Log Analytics”打开“日志搜索”页。

### <a name="sample-queries"></a>示例查询

下表提供了此解决方案收集的更改记录的示例日志搜索：

|查询  |说明  |
|---------|---------|
|ConfigurationData<br>&#124; where   ConfigDataType == "WindowsServices" and SvcStartupType == "Auto"<br>&#124; where SvcState == "Stopped"<br>&#124; summarize arg_max(TimeGenerated, *) by SoftwareName, Computer         | 显示已设置为“自动”，但报告为“已停止”的 Windows 服务的最新库存记录<br>结果仅限于该 SoftwareName 和 Computer 的最新记录      |
|ConfigurationChange<br>&#124; where ConfigChangeType == "Software" and ChangeCategory == "Removed"<br>&#124; order by TimeGenerated desc|显示已删除软件的更改记录|

## <a name="next-steps"></a>后续步骤

访问有关更改跟踪的教程，详细了解解决方案的用法：

> [!div class="nextstepaction"]
> [故障排除环境中的更改](automation-tutorial-troubleshoot-changes.md)

* 使用 [Log Analytics 中的日志搜索](../log-analytics/log-analytics-log-searches.md)以查看详细的更改跟踪数据。

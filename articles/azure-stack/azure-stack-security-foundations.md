---
title: 了解 Azure Stack 的安全控制
description: 向服务管理员介绍应用于 Azure Stack 的安全控制措施
services: azure-stack
documentationcenter: ''
author: PatAltimore
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/9/2018
ms.author: patricka
ms.openlocfilehash: 32c268c1e4a0ff4d17c5b03f0ffd33b0ddf5b927
ms.sourcegitcommit: 7824e973908fa2edd37d666026dd7c03dc0bafd0
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2018
ms.locfileid: "48901475"
---
# <a name="azure-stack-infrastructure-security-posture"></a>Azure Stack 基础结构安全局势

*适用于：Azure Stack 集成系统*

安全考虑因素与合规性是使用混合云的主要推动因素。 Azure Stack 专为这些方案。 本文介绍适用于 Azure Stack 位置中的安全控件。

两个安全局势层在 Azure Stack 中共存。 第一层是 Azure Stack 基础结构，其中包括 Azure 资源管理器之前的硬件组件。 第一层包括管理员门户和租户门户。 第二层由租户创建、部署和管理的工作负荷组成。 第二层包括虚拟机和应用程序服务网站等项。

## <a name="security-approach"></a>安全方法

Azure Stack 的安全局势功能旨在防范新式威胁，为符合主要合规标准的要求而构建。 因此，Azure Stack 基础结构的安全局势构建在两个支柱之上：

 - **假想入侵**  
我们从假设系统已被入侵的情况出发，将重点放在检测入侵并限制其影响上，而不只是尽量防止攻击。 
 - **默认强化**  
由于基础结构在妥善定义的硬件和软件中运行，因此 Azure Stack 会在默认情况下启用、配置和验证所有安全功能。

由于 Azure Stack 是以集成系统的形式交付的，因此 Azure Stack 基础结构的安全局势由 Microsoft 定义。 如同在 Azure 中一样，租户需负责定义其租户工作负荷的安全局势。 本文档提供有关 Azure Stack 基础结构安全局势的基础知识。

## <a name="data-at-rest-encryption"></a>静态数据加密
所有 Azure Stack 基础结构和租户静态数据都以 Bitlocker 加密。 这种加密可以防范 Azure Stack 存储组件的实物遗失或失窃。 

## <a name="data-in-transit-encryption"></a>传输中数据加密
Azure Stack 基础结构组件使用以 TLS 1.2 加密的通道进行通信。 加密证书由基础结构自行管理。 

所有外部基础结构终结点（例如 REST 终结点或 Azure Stack 门户）都支持使用 TLS 1.2 进行安全通信。 对于这些终结点，必须提供来自第三方或企业证书颁发机构的加密证书。 

尽管可对这些外部终结点使用自签名证书，但 Microsoft 强烈建议不要使用此类证书。 

## <a name="secret-management"></a>机密管理
Azure Stack 基础结构使用许多机密（例如密码）来运行。 其中大多数都是自动轮换，因为它们是 Group-Managed 服务帐户，每隔 24 小时轮换一次。

可以使用特权终结点中的脚本手动旋转 Group-Managed 服务帐户不是剩余机密。

## <a name="code-integrity"></a>代码完整性
Azure Stack 使用最新的 Windows Server 2016 安全功能。 其中一个是 Windows Defender Device Guard，也不能提供应用程序允许列表，可确保只有经过授权的 Azure 堆栈基础结构内的代码运行。 

已授权的代码是由 Microsoft 或 OEM 合作伙伴签名。 签名已授权的代码包含在由 Microsoft 定义的策略中指定的允许软件的列表。 换而言之，只能执行已批准在 Azure Stack 基础结构中运行的软件。 系统会阻止任何执行未经授权代码的企图并生成审核。

Device Guard 策略也会阻止第三方代理或软件在 Azure Stack 基础结构中运行。

## <a name="credential-guard"></a>Credential Guard
Azure Stack 中的另一个 Windows Server 2016 安全功能是 Windows Defender Credential Guard，它可用于防止 Azure Stack 基础结构凭据遭到“传递哈希”和“传递票证”攻击。

## <a name="antimalware"></a>反恶意软件
Azure Stack 中的每个组件（Hyper-V 主机和虚拟机）受到 Windows Defender Antivirus 的保护。

在联网场景中，防病毒定义和引擎更新每天应用多次。 在离线场景中，反恶意软件更新作为 Azure Stack 的每月更新的一部分应用。 有关详细信息，请参阅[更新 Azure Stack 上的 Windows Defender 防病毒](azure-stack-security-av.md)。

## <a name="constrained-administration-model"></a>受约束的管理模型
Azure Stack 中的管理是使用三个入口点来控制的，其中每个入口点有特定的用途： 
1. [管理员门户](azure-stack-manage-portals.md)针对日常管理操作提供点击式体验。
2. Azure 资源管理器通过 PowerShell 和 Azure CLI 使用的 REST API 公开管理员门户的所有管理操作。 
3. 对于特定的低级操作（例如数据中心集成或支持方案），Azure Stack 公开一个称作[特权终结点](azure-stack-privileged-endpoint.md)的 PowerShell 终结点。 此终结点只公开一组已添加到允许列表的 cmdlet，并且经常接受审核。

## <a name="network-controls"></a>网络控制
Azure Stack 基础结构提供了多个层的网络访问控制列表 (ACL)。 ACL 可防止用户对基础结构组件进行未经授权的访问，并将基础结构通信限制为基础结构在运行时需要访问的路径。 

在三个层中实施网络 ACL：
1.  机架顶部交换机
2.  软件定义的网络
3.  主机和 VM 操作系统防火墙

## <a name="regulatory-compliance"></a>法规符合性

Azure Stack 已进行过正式评估由独立于第三方的审核公司。 因此，提供了如何在 Azure Stack 基础结构满足从几项主要的符合性标准适用的控件上的文档。 文档不是包含多个人员与或进程有关的控件的标准，由于 Azure stack 的认证。 相反，客户可以使用此文档来快速开始及其认证过程。

评估包括以下标准：

- [PCI DSS](https://www.pcisecuritystandards.org/pci_security/)地址支付卡行业。
- [CSA 云控制矩阵](https://cloudsecurityalliance.org/group/cloud-controls-matrix/#_overview)跨多个标准，包括 FedRAMP 中等、 ISO27001、 HIPAA、 HITRUST、 ITAR、 NIST SP800-53，和其他人是一个全面的映射。
- [FedRAMP High](https://www.fedramp.gov/fedramp-releases-high-baseline/)适用于政府客户。

在找不到符合性文档[Microsoft 服务信任门户](https://servicetrust.microsoft.com/ViewPage/Blueprint)。 符合性指南是受保护的资源，并且需要你使用你的 Azure 云服务凭据进行登录。

## <a name="next-steps"></a>后续步骤

- [了解如何在 Azure Stack 中轮换机密](azure-stack-rotate-secrets.md)
- [PCI DSS 和 Azure Stack 的 CSA CCM 文档](https://servicetrust.microsoft.com/ViewPage/TrustDocuments)
- [适用于 Azure Stack 的 DoD 和 NIST 文档](https://servicetrust.microsoft.com/ViewPage/Blueprint)

---
title: Azure 容器注册表 webhook
description: 了解如何使用 webhook 在注册表存储库中发生特定操作时触发事件。
services: container-registry
author: dlepow
ms.service: container-registry
ms.topic: article
ms.date: 08/20/2017
ms.author: danlep
ms.openlocfilehash: 350ae16aa66276e7e64c5c35718dca74a70f499e
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/08/2018
ms.locfileid: "48854078"
---
# <a name="using-azure-container-registry-webhooks"></a>使用 Azure 容器注册表 webhook

Azure 容器注册表可存储和管理专用 Docker 容器映像，其方式类似于 Docker Hub 存储公共 Docker 映像。 可以使用 Webhook 在其中一个注册表存储库中发生特定操作时触发事件。 Webhook 可在注册表级别响应事件或者将其范围缩小到特定存储库标记。

有关 Webhook 请求的详细信息，请参阅 [Azure 容器注册表 Webhook 架构参考](container-registry-webhook-reference.md)。

## <a name="prerequisites"></a>先决条件

* Azure 容器注册表 - 在 Azure 订阅中创建容器注册表。 例如，使用 [Azure 门户](container-registry-get-started-portal.md)或 [Azure CLI](container-registry-get-started-azure-cli.md)。
* Docker CLI - 要将本地计算机设置为 Docker 主机并访问 Docker CLI 命令，请安装 [Docker 引擎](https://docs.docker.com/engine/installation/)。

## <a name="create-webhook-azure-portal"></a>创建 webhook Azure 门户

1. 登录到 [Azure 门户](https://portal.azure.com)
1. 导航到要在其中创建 Webhook 的容器注册表。
1. 在“服务”下，选择 **Webhook**。
1. 在 Webhook 工具栏中选择“添加”。
1. 使用以下信息完成“创建 webhook”窗体：

| 值 | Description |
|---|---|
| 名称 | 想要赋予 webhook 的名称。 它只能包含小写字母和数字，且长度必须为 5-50 个字符。 |
| 服务 URI | Webhook 应向其发送 POST 通知的 URI。 |
| 自定义标头 | 想要随 POST 请求一起传递的标头。 它们的格式应该为：“键: 值”。 |
| 触发操作 | 触发 webhook 的操作。 当前可通过映像推送和/或删除操作触发 Webhook。 |
| 状态 | Webhook 创建后的状态。 默认启用。 |
| 范围 | Webhook 的作用域。 默认情况下，作用域为注册表中的所有事件。 可使用“存储库:标记”格式针对存储库或标记进行指定。 |

示例 Webhook 窗体：

![Azure 门户中的 ACR webhook 创建 UI](./media/container-registry-webhook/webhook.png)

## <a name="create-webhook-azure-cli"></a>创建 webhook Azure CLI

若要使用 Azure CLI 创建 webhook，请使用 [az acr webhook create](/cli/azure/acr/webhook#az-acr-webhook-create) 命令。

```azurecli-interactive
az acr webhook create --registry mycontainerregistry --name myacrwebhook01 --actions delete --uri http://webhookuri.com
```

## <a name="test-webhook"></a>测试 webhook

### <a name="azure-portal"></a>Azure 门户

在对容器映像推送和删除操作使用 webhook 之前，可以使用“Ping”按钮对其进行测试。 Ping 将向指定的终结点发送泛型 POST 请求并记录响应。 使用 ping 功能可帮助验证是否已正确配置 Webhook。

1. 选择想要测试的 webhook。
2. 在顶部工具栏中，选择“Ping”。
3. 在“HTTP 状态”列中检查终结点的响应。

![Azure 门户中的 ACR webhook 创建 UI](./media/container-registry-webhook/webhook-02.png)

### <a name="azure-cli"></a>Azure CLI

若要使用 Azure CLI 测试 ACR webhook，请使用 [az acr webhook ping](/cli/azure/acr/webhook#az-acr-webhook-ping) 命令。

```azurecli-interactive
az acr webhook ping --registry mycontainerregistry --name myacrwebhook01
```

若要查看结果，请使用 [az acr webhook list-events](/cli/azure/acr/webhook#list-events) 命令。

```azurecli-interactive
az acr webhook list-events --registry mycontainerregistry08 --name myacrwebhook01
```

## <a name="delete-webhook"></a>删除 webhook

### <a name="azure-portal"></a>Azure 门户

每个 webhook 均可通过在 Azure 门户中选择 webhook，然后选择“删除”按钮进行删除。

### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
az acr webhook delete --registry mycontainerregistry --name myacrwebhook01
```

## <a name="next-steps"></a>后续步骤

### <a name="webhook-schema-reference"></a>Webhook 架构参考

有关 Azure 容器注册表发出的 JSON 事件负载的格式和属性的详细信息，请参阅 Webhook 架构参考：

[Azure 容器注册表 Webhook 架构参考](container-registry-webhook-reference.md)

### <a name="event-grid-events"></a>事件网格事件

除了本文中讨论的本机注册表 Webhook 事件之外，Azure 容器注册表还可以向事件网格发送事件：

[快速入门：将容器注册表事件发送到事件网格](container-registry-event-grid-quickstart.md)
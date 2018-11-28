---
title: Azure 中的专用 Docker 容器镜像仓库
description: 介绍 Azure 容器镜像仓库服务，该服务提供基于云的托管专用 Docker 镜像仓库。
services: container-registry
author: stevelas
ms.service: container-registry
ms.topic: overview
ms.date: 09/25/2018
ms.author: stevelas
ms.custom: mvc
ms.openlocfilehash: 3cc44b58d3e715a1e3c264be03b887f27c0c753c
ms.sourcegitcommit: 0b7fc82f23f0aa105afb1c5fadb74aecf9a7015b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2018
ms.locfileid: "51633489"
---
# <a name="introduction-to-private-docker-container-registries-in-azure"></a>Azure 中的专用 Docker 容器镜像仓库简介

Azure 容器镜像仓库是基于开源 Docker 镜像仓库 2.0 的托管 [Docker 镜像仓库](https://docs.docker.com/registry/)服务。 可以创建和维护 Azure 容器镜像仓库来存储与管理专用的 [Docker 容器](https://www.docker.com/what-docker)镜像。

可将 Azure 中的容器镜像仓库与现有的容器开发和部署管道配合使用。 使用 Azure 容器镜像仓库生成 (ACR Build) 在 Azure 中生成容器镜像。 可以通过源代码提交和基础镜像更新生成触发器按需生成或完全自动生成。

有关 Docker 和容器的背景信息，请参阅 [Docker 概述](https://docs.docker.com/engine/docker-overview/)。

## <a name="use-cases"></a>用例

将 Azure 容器镜像仓库中的镜像提取到各种部署目标：

* **可缩放业务流程系统**，用于跨主机群集管理容器化应用程序，包括 [Kubernetes](http://kubernetes.io/docs/)、[DC/OS](https://docs.mesosphere.com/) 和 [Docker Swarm](https://docs.docker.com/swarm/)。
* 支持大规模生成和运行应用程序的 **Azure 服务**，包括 [Azure Kubernetes 服务 (AKS)](../aks/index.yml)、[应用服务](../app-service/index.yml)、[Batch](../batch/index.yml)、[Service Fabric](/azure/service-fabric/) 和其他服务。

开发人员还可以在执行容器开发工作流的过程中将内容推送到容器镜像仓库。 例如，通过连续集成和部署工具（例如 [Azure DevOps Services](https://docs.microsoft.com/azure/devops/) 或 [Jenkins](https://jenkins.io/)）将目标设置为容器镜像仓库。

配置 [ACR 任务](#azure-container-registry-build)以在应用程序镜像的基础镜像发生更新时自动重新生成应用程序镜像。 可以使用 ACR 任务在团队向 Git 存储库提交代码时自动执行镜像生成。

## <a name="key-concepts"></a>关键概念

* **镜像仓库** - 在 Azure 订阅中创建一个或多个容器镜像仓库。 镜像仓库以三种 SKU 形式提供：[基本、标准和高级](container-registry-skus.md)，每一种都支持 webhook 集成、通过 Azure Active Directory 进行的镜像仓库身份验证，以及删除功能。 在与部署相同的 Azure 位置创建镜像仓库，充分利用容器镜像的本地闭合网络存储。 将高级镜像仓库的[异地复制](container-registry-geo-replication.md)功能用于高级复制和容器镜像分发方案。 完全限定的镜像仓库名称采用以下格式：`myregistry.azurecr.io`。

  可以使用使用 Azure Active Directory 支持的 [服务主体](../active-directory/develop/app-objects-and-service-principals.md)或提供的管理员帐户来[控制访问](container-registry-authentication.md)容器镜像仓库。 运行标准 `docker login` 命令可对镜像仓库进行身份验证。

* **存储库** - 一个镜像仓库包含一个或多个存储库，用于存储容器镜像组。 Azure 容器镜像仓库支持多级存储库命名空间。 使用多级命名空间可将特定应用相关的镜像集合分组，或者将特定开发或运营团队的应用集合分组。 例如：

  * `myregistry.azurecr.io/aspnetcore:1.0.1` 表示企业范围的镜像
  * `myregistry.azurecr.io/warrantydept/dotnet-build` 表示用于构建 .NET 应用、在保修部门之间共享的镜像
  * `myregistry.azurecr.io/warrantydept/customersubmissions/web` 表示一个 Web 镜像，它已在客户提交应用中分组，由保修部门拥有

* **镜像** - 存储在存储库中，每个镜像是兼容 Docker 的容器的只读快照。 Azure 容器镜像仓库可以包含 Windows 和 Linux 镜像。 可以控制所有容器部署的镜像名称。 使用标准 [Docker 命令](https://docs.docker.com/engine/reference/commandline/)可将镜像推送到存储库，或者从存储库中提取镜像。 除了容器镜像，Azure 容器镜像仓库还存储[相关的内容格式](container-registry-image-formats.md)，例如 [Helm 图表](container-registry-helm-repos.md)，用于将应用程序部署到 Kubernetes。

* **容器** - 容器定义软件应用程序及其在完整文件系统中包装的依赖项，包括代码、运行时、系统工具和库。 可以基于从容器镜像仓库提取的 Windows 或 Linux 镜像运行 Docker 容器。 在一台计算机上运行的容器共享操作系统内核。 Docker 容器完全可移植到所有主要 Linux 发行版、macOS 和 Windows。

## <a name="azure-container-registry-tasks"></a>Azure 容器镜像仓库任务

[Azure 容器镜像仓库任务](container-registry-tasks-overview.md)（ACR 任务）是 Azure 容器镜像仓库中的一个功能套件，用于在 Azure 中提供简化且高效的 Docker 容器镜像生成功能。 使用 ACR 任务可以通过将 `docker build` 操作卸载到 Azure 来将开发内部循环扩展到云。 配置生成任务以使其自动执行容器 OS 和框架修补管道，并使其在团队将代码提交到源代码管理时自动生成镜像。

[多步骤任务](container-registry-tasks-overview.md#multi-step-tasks-preview)（ACR 任务的一项预览版功能），提供用于在云中构建、测试和修补容器镜像的基于步骤的任务定义和执行。 任务步骤定义各个容器镜像构建和推送操作。 它们还可以定义一个或多个容器的执行，每个步骤都使用容器作为其执行环境。

## <a name="next-steps"></a>后续步骤

* [使用 Azure 门户创建容器镜像仓库](container-registry-get-started-portal.md)
* [使用 Azure CLI 创建容器镜像仓库](container-registry-get-started-azure-cli.md)
* [使用 ACR 任务自动执行 OS 和框架修补](container-registry-tasks-overview.md)

---
title: 创建专用 Azure Kubernetes 服务群集
description: 了解如何创建专用 Azure Kubernetes 服务 (AKS) 群集
services: container-service
ms.topic: article
origin.date: 07/17/2020
author: rockboyfor
ms.date: 10/26/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: 2cb6bbf2c0e7155893ac9f4e2ed79aae5c07edf0
ms.sourcegitcommit: 7b3c894d9c164d2311b99255f931ebc1803ca5a9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92470511"
---
<!--Verified successfully from PG team confirmation-->
# <a name="create-a-private-azure-kubernetes-service-cluster"></a>创建专用 Azure Kubernetes 服务群集

在专用群集中，控制平面或 API 服务器具有内部 IP 地址，这些地址在 [RFC1918 - 专用 Internet 的地址分配](https://tools.ietf.org/html/rfc1918)文档中定义。 通过使用专用群集，可以确保 API 服务器与节点池之间的网络流量仅保留在专用网络上。

控制平面或 API 服务器位于 Azure Kubernetes 服务 (AKS) 托管的 Azure 订阅中。 客户的群集或节点池在客户的订阅中。 服务器与群集或节点池可以通过 API 服务器虚拟网络中的 Azure 专用链接服务以及在客户 AKS 群集的子网中公开的专用终结点相互通信。

<!--Not Avaialble on [Azure Private Link service][private-link-service]-->

## <a name="region-availability"></a>上市区域

专用群集在[支持 AKS](https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service&regions=,china-non-regional,china-east,china-east-2,china-north,china-north-2,) 的 Azure 中国区域提供。

<!--CORRECT ON Azure China regions-->
<!--Not Available on * Azure China 21Vianet isn't currently supported.-->
<!--Not Available on * US Gov Texas isn't currently supported because of missing Private Link support.-->

## <a name="prerequisites"></a>先决条件

* Azure CLI 版本 2.2.0 或更高版本

## <a name="create-a-private-aks-cluster"></a>创建专用 AKS 群集

### <a name="create-a-resource-group"></a>创建资源组

为 AKS 群集创建资源组或使用现有资源组。

```azurecli
az group create -l chinaeast2 -n MyResourceGroup
```

### <a name="default-basic-networking"></a>默认基本网络 

```azurecli
az aks create -n <private-cluster-name> -g <private-cluster-resource-group> --load-balancer-sku standard --enable-private-cluster  
```
其中 --enable-private-cluster 是专用群集的必需标志。 

### <a name="advanced-networking"></a>高级网络  

```azurecli
az aks create \
    --resource-group <private-cluster-resource-group> \
    --name <private-cluster-name> \
    --load-balancer-sku standard \
    --enable-private-cluster \
    --network-plugin azure \
    --vnet-subnet-id <subnet-id> \
    --docker-bridge-address 172.17.0.1/16 \
    --dns-service-ip 10.2.0.10 \
    --service-cidr 10.2.0.0/24 
```
其中 --enable-private-cluster 是专用群集的必需标志。 

> [!NOTE]
> 如果 Docker 桥地址 CIDR (172.17.0.1/16) 与子网 CIDR 冲突，请相应地更改 Docker 桥地址。

## <a name="options-for-connecting-to-the-private-cluster"></a>连接到专用群集的选项

API 服务器终结点没有公共 IP 地址。 若要管理 API 服务器，需要使用有权访问 AKS 群集的 Azure 虚拟网络 (VNet) 的 VM。 有几种选项可用于建立与专用群集的网络连接。

* 在与 AKS 群集相同的 Azure 虚拟网络 (VNet) 中创建 VM。
* 在单独的网络中使用 VM 并设置[虚拟网络对等互联][virtual-network-peering]。  有关此选项的详细信息，请参阅以下部分。
* 使用[快速路由或 VPN][express-route-or-VPN] 连接。

在与 AKS 群集相同的 VNET 中创建 VM 是最简单的选项。  快速路由和 VPN 会增加成本，且要求额外的网络复杂性。  虚拟网络对等互联要求计划网络 CIDR 范围，以确保不存在重叠范围。

## <a name="virtual-network-peering"></a>虚拟网络对等互连

如前所述，虚拟网络对等互连是一种访问专用群集的方法。 若要使用虚拟网络对等互连，需要在虚拟网络与专用 DNS 区域之间设置链接。

1. 转到 Azure 门户中的节点资源组。  
2. 选择专用 DNS 区域。   
3. 在左窗格中，选择“虚拟网络”链接。  
4. 创建新链接，将 VM 的虚拟网络添加到专用 DNS 区域。 DNS 区域链接需要几分钟时间才能变为可用。  
5. 在 Azure 门户中，导航到包含群集虚拟网络的资源组。  
6. 在右窗格中，选择“虚拟网络”。 虚拟网络名称的格式为 aks-vnet-\*。  
7. 在左窗格中，选择“对等互连”。  
8. 选择“添加”，添加 VM 的虚拟网络，然后创建对等互连。  
9. 转到你在其中具有 VM 的虚拟网络，选择“对等互连”、“AKS 虚拟网络”，然后创建对等互连。 如果 AKS 虚拟网络上的地址范围与 VM 的虚拟网络冲突，则对等互连将失败。 有关详细信息，请参阅[虚拟网络对等互联][virtual-network-peering]。

## <a name="hub-and-spoke-with-custom-dns"></a>具有自定义 DNS 的中心和分支

中心和分支体系结构通常用于在 Azure 中部署网络。 在许多此类部署中，会将分支 VNet 中的 DNS 设置配置为引用中心 DNS 转发器，以允许本地和基于 Azure 的 DNS 解析。 将 AKS 群集部署到此类网络环境中时，必须考虑一些特殊注意事项。

<!--Not Available on [Hub and spoke architectures](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)-->

:::image type="content" source="media/private-clusters/aks-private-hub-spoke.png" alt-text="专用群集中心和分支":::

1. 默认情况下，预配专用群集后，会在群集托管资源组中创建专用终结点 (1) 和专用 DNS 区域 (2)。 群集使用专用区域中的 A 记录来解析专用终结点的 IP，以便与 API 服务器通信。

2. 专用 DNS 区域仅链接到群集节点附加到的 VNet (3)。 这意味着专用终结点只能由该链接 VNet 中的主机进行解析。 在 VNet 上不配置任何自定义 DNS（默认设置）的情况下，这可以正常工作，因为主机指向用于 DNS 的 168.63.129.16，因此可以解析专用 DNS 区域中的记录（由于存在链接）。

3. 在包含群集的 VNet 具有自定义 DNS 设置 (4) 的情况下，除非将专用 DNS 区域链接到包含自定义 DNS 解析程序的 VNet (5)，否则群集部署将失败。 可以在群集预配期间创建专用区域后手动创建此链接，也可以使用基于事件的部署机制（例如，Azure 事件网格和 Azure Functions）在检测到区域已创建后通过自动化来创建此链接。

## <a name="dependencies"></a>依赖项  

* 仅标准 Azure 负载均衡器支持专用链接服务。 不支持基本 Azure 负载均衡器。  
* 若要使用自定义 DNS 服务器，请在自定义 DNS 服务器中将 Azure DNS IP 168.63.129.16 作为上游 DNS 服务器进行添加。

## <a name="limitations"></a>限制 
* IP 授权范围不能应用于专用 API 服务器终结点，它们仅适用于公共 API 服务器

    <!--Not Available on * [Availability Zones][availability-zones]-->
    <!--Not Available on * [Azure Private Link service limitations][private-link-service]-->
    
* 不支持具有专用群集的 Azure DevOps Microsoft 托管的代理。
    
    <!--Not Available on  Consider to use [Self-hosted Agents][devops-agents]-->
    
* 对于需要使 Azure 容器注册表能够与专用 AKS 配合使用的客户，容器注册表虚拟网络必须与代理群集虚拟网络对等互连。
* 当前不支持 Azure Dev Spaces
* 不支持将现有 AKS 群集转换为专用群集
* 删除或修改客户子网中的专用终结点将导致群集停止运行。 
* 当前不支持适用于容器的 Azure Monitor 实时数据。
* 当前不支持运行时间 SLA。

<!-- LINKS - internal -->

[az-provider-register]: https://docs.azure.cn/cli/provider#az_provider_register
[az-feature-list]: https://docs.azure.cn/cli/feature#az_feature_list
[az-extension-add]: https://docs.microsoft.com/cli/azure/extension#az_extension_add
[az-extension-update]: https://docs.microsoft.com/cli/azure/extension#az_extension_update

<!--Not Available on [private-link-service]: ../private-link/private-link-service-overview.md#limitations-->

[virtual-network-peering]: ../virtual-network/virtual-network-peering-overview.md
[azure-bastion]: ../bastion/bastion-create-host-portal.md
[express-route-or-vpn]: ../expressroute/expressroute-about-virtual-network-gateways.md

<!--Not Available on [devops-agents]: https://docs.microsoft.com/azure/devops/pipelines/agents/agents-->
<!--Not Available on [availability-zones]: availability-zones.md-->

<!-- Update_Description: new article about private clusters -->
<!--NEW.date: 10/19/2020-->
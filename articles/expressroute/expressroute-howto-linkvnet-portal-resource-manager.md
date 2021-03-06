---
title: 教程：将 VNet 链接到 ExpressRoute 线路 - Azure 门户
description: 本教程介绍如何使用 Azure 门户创建连接来将虚拟网络链接到 Azure ExpressRoute 线路。
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: tutorial
origin.date: 10/15/2020
ms.author: v-yiso
ms.date: 11/16/2020
ms.openlocfilehash: 582dfaffd473786731c5f490309eee7552e4a3af
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94327460"
---
# <a name="tutorial-connect-a-virtual-network-to-an-expressroute-circuit-using-the-portal"></a>教程：使用门户将虚拟网络连接到 ExpressRoute 线路

> [!div class="op_single_selector"]
> * [Azure 门户](expressroute-howto-linkvnet-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-linkvnet-arm.md)
> * [Azure CLI](howto-linkvnet-cli.md)
> * [PowerShell（经典）](expressroute-howto-linkvnet-classic.md)
> 

本教程可帮助你使用 Azure 门户创建连接来将虚拟网络链接到 Azure ExpressRoute 线路。 连接到 Azure ExpressRoute 线路的虚拟网络可以在同一订阅中，也可以属于另一订阅。

在本教程中，你将了解如何执行以下操作：
> [!div class="checklist"]
> - 将虚拟网络连接到同一订阅中的线路。
> - 将虚拟网络连接到另一订阅中的线路。
> - 删除虚拟网络与 ExpressRoute 线路之间的链接。

## <a name="prerequisites"></a>必备条件

* 在开始配置之前，请先查看[先决条件](expressroute-prerequisites.md)、[路由要求](expressroute-routing.md)和[工作流](expressroute-workflows.md)。

* 必须有一个活动的 ExpressRoute 线路。
  * 请按说明[创建 ExpressRoute 线路](expressroute-howto-circuit-portal-resource-manager.md)，并通过连接提供商启用该线路。
  * 请确保为线路配置 Azure 专用对等互连。 有关对等互连和路由说明，请参阅[为 ExpressRoute 线路创建和修改对等互连](expressroute-howto-routing-portal-resource-manager.md)一文。
  * 确保已配置 Azure 专用对等互连，并建立网络和 Microsoft 之间的 BGP 对等互连，以便进行端到端连接。
  * 确保已创建并完全预配一个虚拟网络和一个虚拟网络网关。 按照说明[创建 ExpressRoute 的虚拟网络网关](expressroute-howto-add-gateway-resource-manager.md)。 ExpressRoute 虚拟网络网关使用的 GatewayType 是“ExpressRoute”而非 VPN。

* 最多可以将 10 个虚拟网络链接到一条标准 ExpressRoute 线路。 使用标准 ExpressRoute 线路时，所有虚拟网络必须都位于同一地缘政治区域。

* 单个 VNet 可最多连接到 4 条 ExpressRoute 线路。 使用以下流程为要连接的每条 ExpressRoute 线路创建新的连接对象。 ExpressRoute 线路可在同一订阅、不同订阅或两者兼有。

* 如果启用 ExpressRoute 高级版加载项，则可以链接 ExpressRoute 线路的地缘政治区域外部的虚拟网络。 通过高级版加载项，你还可以根据所选带宽，将 10 个以上的虚拟网络连接到 ExpressRoute 线路。 有关高级外接程序的更多详细信息，请参阅[常见问题解答](expressroute-faqs.md)。

  ## <a name="connect-a-vnet-to-a-circuit---same-subscription"></a>将 VNet 连接到线路 - 同一订阅

> [!NOTE]
> 如果第 3 层提供商配置了对等互连，则将不会显示 BGP 配置信息。 如果线路处于已预配状态，应该能够创建连接。
>

### <a name="to-create-a-connection"></a>创建连接

1. 确保已成功配置 ExpressRoute 线路和 Azure 专用对等互连。 按照[创建 ExpressRoute 线路](expressroute-howto-circuit-arm.md)和[创建和修改 ExpressRoute 线路的对等互连](expressroute-howto-routing-arm.md)中的说明操作。 ExpressRoute 线路应如下图所示：

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/express-route-circuit.png" alt-text="ExpressRoute 线路屏幕截图":::

1. 现在可以开始预配连接，以便将虚拟网络网关链接到 ExpressRoute 线路。 选择“连接” > “添加”，打开“添加连接”页面  。

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/add-connection.png" alt-text="添加连接屏幕截图":::

1. 输入连接的名称，然后选择“下一步:设置 >”。

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/create-connection-basic.png" alt-text="创建连接的“基本信息”页面":::

1. 选择要链接到线路的虚拟网络中的网关，然后选择“查看 + 创建”。 然后在验证完成后，选择“创建”。

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/create-connection-settings.png" alt-text="创建连接设置页面":::

1. 成功配置连接之后，连接对象会显示连接的信息。

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/connection-object.png" alt-text="连接对象屏幕截图":::

## <a name="connect-a-vnet-to-a-circuit---different-subscription"></a>将 VNet 连接到线路 - 不同订阅

用户可以在多个订阅之间共享 ExpressRoute 线路。 下图是在多个订阅之间共享 ExpressRoute 线路的简单示意图。

:::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/cross-subscription.png" alt-text="跨订阅连接":::

大型云中的每个较小云用于表示属于组织中不同部门的订阅。 组织内的每个部门使用自己的订阅部署其服务，但可以共享单个 ExpressRoute 线路以连接回本地网络。 单个部门（在此示例中为 IT 部门）可以拥有 ExpressRoute 线路。 组织内的其他订阅可以使用 ExpressRoute 线路。

  > [!NOTE]
  > 专用线路的连接和带宽费用将应用于 ExpressRoute 线路所有者。 所有虚拟网络共享相同的带宽。
  >

### <a name="administration---about-circuit-owners-and-circuit-users"></a>管理 - 关于线路所有者和线路用户

“线路所有者”是 ExpressRoute 线路资源的已授权超级用户。 线路所有者可以创建可由线路用户兑换的授权。 线路用户是虚拟网络网关的所有者（这些网关与 ExpressRoute 线路位于不同的订阅中）。 线路用户可以兑换授权（每个虚拟网络需要一个授权）。

线路所有者有权随时修改和撤消授权。 撤消授权会导致从已撤消访问权限的订阅中删除所有链路连接。

### <a name="circuit-owner-operations"></a>线路所有者操作

**若要创建连接授权**

线路所有者创建授权，这将创建授权密钥，供线路用户用于将其虚拟网络网关连接到 ExpressRoute 线路。 一个授权只可用于一个连接。

> [!NOTE]
> 每个连接都需要单独授权。
>

1. 在 ExpressRoute 页面中，选择“授权”，然后键入授权的名称并选择“保存”  。

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/authorization.png" alt-text="授权":::

2. 保存配置后，复制“资源 ID”和“授权密钥”。

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/authorization-key.png" alt-text="授权密钥":::

**若要删除连接授权**

可为连接的授权密钥选择“删除”图标来删除该连接。

:::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/delete-authorization-key.png" alt-text="删除授权密钥":::

如果要删除连接，但想要保留授权密钥，可从线路的连接页面中删除此连接。

:::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/delete-connection-owning-circuit.png" alt-text="删除具有线路的连接":::

### <a name="circuit-user-operations"></a>线路用户操作

线路用户需有资源 ID 以及线路所有者提供的授权密钥。 

**若要兑换连接授权**

1. 选择“+ 创建资源”  按钮。 搜索“连接”，然后选择“创建” 。

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/create-new-resources.png" alt-text="创建新资源":::

1. 确保“连接类型”设置为“ExpressRoute”。 选择“资源组”和“位置”，然后在“基本信息”页面中选择“确定” 。

    > [!NOTE]
    > 该位置必须与你要为其创建连接的虚拟网络网关位置相匹配。

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/connection-basics.png" alt-text="“基本信息”页":::

1. 在“设置”页面中，选择“虚拟网络网关”并选中“兑换授权”复选框。 输入“授权密钥”和“对等线路 URI”，并为连接命名。 选择“确定”。 
 
    > [!NOTE]
    > *对等线路 URI* 是 ExpressRoute 线路的资源 ID（可以在 ExpressRoute 线路的“属性设置”窗格下找到）。

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/connection-settings.png" alt-text="“设置”页":::

1. 在“摘要”页面中复查信息，并选择“确定” 。

    :::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/connection-summary.png" alt-text="摘要页面":::

## <a name="clean-up-resources"></a>清理资源

可以通过在你的连接的页面上单击“删除”图标来取消 VNet 到 ExpressRoute 的链接。

:::image type="content" source="./media/expressroute-howto-linkvnet-portal-resource-manager/delete-connection.png" alt-text="删除连接":::

## <a name="next-steps"></a>后续步骤

在本教程中，你已了解如何将虚拟网络连接到同一订阅和不同订阅中的线路。 有关 ExpressRoute 网关的详细信息，请参阅： 

> [!div class="nextstepaction"]
> [关于 ExpressRoute 虚拟网络网关](expressroute-about-virtual-network-gateways.md)

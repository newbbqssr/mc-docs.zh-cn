---
title: Azure Monitor 工作簿 - 移动区域
description: 如何将工作簿移到其他区域
services: azure-monitor
author: Johnnytechn
manager: acearun
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: how-to
ms.custom: subject-moving-resources
ms.date: 11/06/2020
ms.author: v-johya
ms.openlocfilehash: 1d85f1f0e7ba2d94221569f2745e1089f9c3fe55
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94329257"
---
# <a name="move-an-azure-workbook-to-another-region"></a>将 Azure 工作簿移动到另一个区域

本文介绍了如何将 Azure 工作簿资源移动到其他 Azure 区域。 由于各种原因，你可能需要将资源移动到其他区域。 例如，利用新的 Azure 区域，部署仅在特定区域中可用的功能或服务，满足内部策略和监管要求，或者满足容量规划要求。

## <a name="prerequisites"></a>先决条件

* 确保目标区域支持工作簿。

* 这些说明适用于在 Azure Monitor 和大多数资源类型上保存的共享工作簿 (`microsoft.insights/workbooks`) 和专用工作簿 (`microsoft.insights/myworkbooks`)。

  但是，对于专门链接到 Application Insights 资源类型的工作簿，这些工作簿存储在保存 Application Insights 资源的 Azure 区域中。

  不能将这些工作簿单独移动到另一个区域。

## <a name="move"></a>移动

移动 Azure 工作簿的最简单方法是使用门户 UI 的“工作簿”工具中的“另存为”：

1. 在工作簿查看器中打开目标工作簿。
2. 使用“编辑”工具栏按钮进入编辑模式。
3. 使用“另存为”工具栏按钮。
4. 在“保存”窗体中，为工作簿选择名称和所需区域。 请确保订阅、资源组和共享的其他字段的设置合理。

   > [!NOTE]
   > 可能需要为工作簿使用新的名称，以避免出现重复的名称。

5. 保存。 

## <a name="verify"></a>Verify

使用 Azure 工作簿浏览 UI，查找新工作簿。 确保该位置是目标位置。

## <a name="clean-up"></a>清理

在新区域中创建工作簿后，删除先前区域中的原始工作簿。
1. 在工作簿查看器中打开原始工作簿。
2. 使用“编辑”工具栏按钮进入编辑模式。
3. 从“编辑工具”下拉列表（铅笔图标）中选择“删除工作簿”。

如果你重命名了工作簿以便将其导入新区域，可以在删除原始工作簿后，使用“编辑模式”工具栏的“编辑工具”下拉列表中的“重命名”项，将工作簿重命名为以前的名称。

## <a name="next-steps"></a>后续步骤

需要移动工作簿模板而不是工作簿？ 请参阅如何[将 Azure 工作簿模板移动到另一个区域](./workbook-templates-move-region.md)。

请参阅如何通过 ARM 模板[部署工作簿和工作簿模板](./workbooks-automate.md)。


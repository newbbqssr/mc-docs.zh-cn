---
title: 比较迁移到 Azure 的 Azure Migrate 和 Site Recovery
description: 总结使用 Azure Migrate 而不是 Site Recovery 进行迁移的优点。
services: site-recovery
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
origin.date: 08/06/2020
author: rockboyfor
ms.date: 10/26/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: 0b82c73bfeceb83f2567dd36831c5db3223e3d3b
ms.sourcegitcommit: 221c32fe6f618679a63f148da7382bc9e495f747
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/20/2020
ms.locfileid: "92211883"
---
# <a name="migrating-to-azure"></a>迁移到 Azure

对于迁移，建议使用 Azure Migrate 服务将 VM 和服务器迁移到 Azure，而不是使用 Microsoft Azure Site Recovery 服务。

<!--Not Available on [Learn more](../migrate/migrate-services-overview.md)-->

## <a name="why-use-azure-migrate"></a>为何使用 Azure Migrate？

使用 Azure Migrate 进行迁移提供了很多优点：

- Azure Migrate 提供了一个集中式中心，用于发现、评估以及迁移到 Azure。
- 借助 Azure Migrate 工具、其他 Azure 服务和第三方工具，可使用 Azure Migrate 提供互操作性和未来的扩展性。
- Azure Migrate：服务器迁移工具专门用于将服务器迁移到 Azure。 此方法针对迁移进行了优化。 不需要了解与迁移没有直接关联的概念和方案。 
- 从为 VM 启动复制的 180 天内，不收取迁移的工具使用费用。 这为完成迁移提供了时间。 只需为复制中使用的存储和网络资源以及测试迁移期间使用的计算付费。
- Azure Migrate 支持 Site Recovery 支持的所有迁移方案。 此外，对于 VMware VM，Azure Migrate 提供无代理迁移选项。
- 我们仅优先考虑 Azure Migrate：服务器迁移工具的新迁移功能。 这些功能不针对 Site Recovery。

## <a name="when-to-use-site-recovery"></a>何时使用 Site Recovery？

应使用 Site Recovery 的情况：

- 用于本地计算机到 Azure 的灾难恢复。
- 用于 Azure 区域之间 Azure VM 的灾难恢复。

尽管建议使用 Azure Migrate 将本地服务器迁移到 Azure，但如果已使用 Site Recovery 启动迁移旅程，则可以继续使用它来完成迁移。  

<!--Not Available on ## Next steps-->

<!--Not Available on > [Review common questions](../migrate/resources-faq.md)-->

<!-- Update_Description: update meta properties, wording update, update link -->
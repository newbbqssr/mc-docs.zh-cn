---
author: WenJason
ms.service: media-services
ms.topic: include
origin.date: 08/17/2020
ms.date: 09/07/2020
ms.author: v-jay
ms.custom: CLI
ms.openlocfilehash: c1aa56f1c6dddfbc83943cba8b67cef410f13f2e
ms.sourcegitcommit: 2eb5a2f53b4b73b88877e962689a47d903482c18
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/03/2020
ms.locfileid: "89413370"
---
<!-- ### Create a storage account -->

创建媒体服务帐户时，需要提供 Azure 存储帐户资源的名称。 指定存储帐户会附加到媒体服务帐户。 若要详细了解如何在媒体服务中使用存储帐户，请参阅[存储帐户](../storage-account-concept.md)。

必须具有一个主存储帐户，并且可以拥有任意数量的与媒体服务帐户关联的辅助存储帐户 。 媒体服务支持常规用途 v2 (GPv2) 或常规用途 v1 (GPv1) 帐户 。 不允许将仅限 Blob 的帐户作为主帐户。 若要了解存储帐户的详细信息，请参阅 [Azure 存储帐户选项](../../../storage/common/storage-account-overview.md)。 

在此示例中，我们创建一个常规用途 v2 标准 LRS 帐户。 若要通过存储帐户进行试验，请使用 `--sku Standard_LRS`。 但是，在选取用于生产的 SKU 时，应考虑 `--sku Standard_RAGRS`，以便通过异地复制确保业务连续性。 有关详细信息，请参阅[存储帐户](https://docs.microsoft.com/cli/azure/storage/account?view=azure-cli-latest)。

以下命令创建将与媒体服务帐户相关联的存储帐户。 在以下脚本中，可以将 `storageaccountforams` 替换为你的值。 `amsResourceGroup` 必须与在上一步中为资源组指定的值匹配。 存储帐户名称的长度必须小于 24。

```azurecli
az storage account create --name storageaccountforams --kind StorageV2 --sku Standard_LRS -l chinaeast2 -g amsResourceGroup
```

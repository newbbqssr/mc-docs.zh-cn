---
title: include 文件
description: include 文件
services: cosmos-db
author: rockboyfor
ms.service: cosmos-db
ms.topic: include
origin.date: 08/07/2019
ms.date: 09/09/2019
ms.author: v-yeche
ms.custom: include file
ms.openlocfilehash: 9eafcd44b84ce89d84d0f0260e7afc547408e740
ms.sourcegitcommit: c1ba5a62f30ac0a3acb337fb77431de6493e6096
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/17/2020
ms.locfileid: "70254798"
---
现在可以使用数据资源管理器将数据添加到新容器。

1. 在“数据资源管理器”中展开“Tasks”数据库，然后展开“Items”容器。    选择“项”，然后单击“新建项”。  

    ![在 Azure 门户的数据资源管理器中创建新文档](./media/cosmos-db-create-sql-api-add-sample-data/azure-cosmosdb-data-explorer-new-document.png)

2. 现在，将文档添加到具有以下结构的容器。

    ```json
    {
        "id": "1",
        "category": "personal",
        "name": "groceries",
        "description": "Pick up apples and strawberries.",
        "isComplete": false
    }
    ```

3. 将 json 添加到“文档”选项卡以后，即可选择“保存”。  

    ![通过复制添加 JSON 数据，然后在 Azure 门户上的数据资源管理器中选择“保存”](./media/cosmos-db-create-sql-api-add-sample-data/azure-cosmosdb-data-explorer-save-document.png)

4. 再创建并保存一个文档，在其中插入 `id` 属性的唯一值，并将其他属性更改为适当值。 新文档可以具有所需的任何结构，因为 Azure Cosmos DB 不对数据施加任何架构。

<!-- Update_Description: wording update -->

---
title: Azure Service Fabric CLI- sfctl is
description: 了解 sfctl（Azure Service Fabric 命令行接口）。 包含用于管理基础结构的命令列表。
ms.topic: reference
origin.date: 01/16/2020
author: rockboyfor
ms.date: 09/14/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: 38151acd8befcbee4063606d0e90f2997b3f29fb
ms.sourcegitcommit: e1cd3a0b88d3ad962891cf90bac47fee04d5baf5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/10/2020
ms.locfileid: "89655634"
---
# <a name="sfctl-is"></a>sfctl is
查询并向基础结构服务发送命令。

## <a name="commands"></a>命令

|Command|说明|
| --- | --- |
| command | 针对给定基础结构服务实例调用管理命令。 |
| query | 针对给定基础结构服务实例调用只读查询。 |

## <a name="sfctl-is-command"></a>sfctl is command
针对给定基础结构服务实例调用管理命令。

对于配置了一个或多个基础结构服务实例的群集，使用此 API 可向特定基础结构服务实例发送特定于基础结构的命令。 可用命令及其相应的响应格式因运行群集的基础结构而异。 此 API 支持 Service Fabric 平台；不应从代码直接使用它。

### <a name="arguments"></a>参数

|参数|说明|
| --- | --- |
| --command [必需] | 将调用的命令文本。 命令内容特定于基础结构。 |
| --service-id | 基础结构服务标识。 <br /><br /> 这是不包含“fabric\:”URI 方案的基础结构服务全名。 只有运行多个基础结构服务实例的群集才需要此参数。 |
| --timeout -t | 默认值\: 60。 |

### <a name="global-arguments"></a>全局参数

|参数|说明|
| --- | --- |
| --debug | 提高日志记录详细程度以显示所有调试日志。 |
| --help -h | 显示此帮助消息并退出。 |
| --output -o | 输出格式。  允许的值\: json、jsonc、table、tsv。  默认值\: json。 |
| --query | JMESPath 查询字符串。 有关详细信息和示例，请参阅 http\://jmespath.org/。 |
| --verbose | 提高日志记录详细程度。 使用 --debug 获取完整的调试日志。 |

## <a name="sfctl-is-query"></a>sfctl is query
针对给定基础结构服务实例调用只读查询。

对于配置了一个或多个基础结构服务实例的群集，使用此 API 可向特定基础结构服务实例发送特定于基础结构的查询。 可用命令及其相应的响应格式因运行群集的基础结构而异。 此 API 支持 Service Fabric 平台；不应从代码直接使用它。

### <a name="arguments"></a>参数

|参数|说明|
| --- | --- |
| --command [必需] | 将调用的命令文本。 命令内容特定于基础结构。 |
| --service-id | 基础结构服务标识。 <br /><br /> 这是不包含“fabric\:”URI 方案的基础结构服务全名。 只有运行多个基础结构服务实例的群集才需要此参数。 |
| --timeout -t | 默认值\: 60。 |

### <a name="global-arguments"></a>全局参数

|参数|说明|
| --- | --- |
| --debug | 提高日志记录详细程度以显示所有调试日志。 |
| --help -h | 显示此帮助消息并退出。 |
| --output -o | 输出格式。  允许的值\: json、jsonc、table、tsv。  默认值\: json。 |
| --query | JMESPath 查询字符串。 有关详细信息和示例，请参阅 http\://jmespath.org/。 |
| --verbose | 提高日志记录详细程度。 使用 --debug 获取完整的调试日志。 |

## <a name="next-steps"></a>后续步骤
- [安装](service-fabric-cli.md) Service Fabric CLI。
- 了解如何通过[示例脚本](./scripts/sfctl-upgrade-application.md)使用 Service Fabric CLI。

<!--Update_Description: update meta properties, wording update -->
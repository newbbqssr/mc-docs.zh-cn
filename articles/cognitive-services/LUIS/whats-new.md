---
title: 新增功能 - 语言理解 (LUIS)
description: 本文会经常更新有关 Azure 认知服务语言理解 API 的新闻。
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: overview
origin.date: 05/19/2020
ms.date: 10/19/2020
ms.author: v-johya
ms.openlocfilehash: e2c9eb9d34ffd5b35e6b864a9b66210c184e613c
ms.sourcegitcommit: 537d52cb783892b14eb9b33cf29874ffedebbfe3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92472280"
---
# <a name="whats-new-in-language-understanding"></a>语言理解中的新增功能

了解服务中的新增功能。 这些项包括发布说明、视频、博客文章和其他类型的信息。 请将本页加入书签，以随时了解该服务的最新信息。

## <a name="release-notes"></a>发行说明

### <a name="june-2020"></a>2020 年 6 月

* [预览版 3.0 创作](luis-migration-authoring-entities.md) SDK -
    * 版本 3.2.0-preview.3 - [.NET - NuGet](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring/)
    * 版本 4.0.0-preview.3 - [JS - NPM](https://www.npmjs.com/package/@azure/cognitiveservices-luis-authoring)

### <a name="may-2020---build"></a>2020 年 5 月 - //Build

* 发布为正式版 (GA)：
    * [语言理解容器](luis-container-howto.md)
    * 预览门户已升级为[当前门户](https://luis.azure.cn)，[以前的](https://previous.luis.ai)门户仍然可用
    * 新的机器学习实体创建和标记体验
    * [升级流程](migrate-from-composite-entity.md)，从复合实体和简单实体升级为机器学习实体
    * [设置](how-to-application-settings-portal.md)对规范化单词变体的支持
* 预览创作 API 更改
    * 适用于嵌套机器学习实体的应用架构 7.x
    * [迁移到所需功能](luis-migration-authoring-entities.md#api-change-constraint-replaced-with-required-feature)
* 为开发人员提供的新资源
    * [持续集成工具](developer-reference-resource.md#continuous-integration-tools)
    * 研讨会 - [使用 LUIS 了解自然语言理解 (NLU) 的最佳实践](developer-reference-resource.md#workshops)



### <a name="march-2020"></a>2020 年 3 月

* 现在，将对此服务的所有 HTTP 请求强制执行 TLS 1.2。 有关详细信息，请参阅 [Azure 认知服务安全性](../cognitive-services-security.md)。

### <a name="november-4-2019---ignite"></a>2019 年 11 月 4 日 - Ignite

* 提高开发人员工作效率
    * [预测终结点 V3](luis-migration-api-v3.md)的正式版。
    * 能够以 `.lu` ([LUDown](https://github.com/microsoft/botbuilder-tools/tree/master/packages/Ludown)) 格式导入和导出应用。 这样，有助于执行有效的 CI/CD 进程。
* 语言扩展
    * [阿拉伯语和印地语](luis-language-support.md)为公共预览版。
* 预生成的模型
    * [预生成的域](luis-reference-prebuilt-domains.md)现在已为正式版 (GA)
* 高级语言理解功能 - [构建复杂的语言模型](luis-concept-entity-types.md)更为轻松。
* 新的扩展的[限制](luis-limits.md) - 提高了短语列表和短语总数的限制，使用新模型作为功能限制。
* 从深层次体系结构格式文本中提取信息，使聊天应用程序的功能更强大。

    ![机器学习实体图像](./media/whats-new/deep-entity-extraction-example.png)

### <a name="july-23-2019"></a>2019 年 7 月 23 日

* 将 [Recognizers-Text](https://github.com/microsoft/Recognizers-Text/releases/tag/dotnet-v1.2.3) 更新为 1.2.3
    * 意大利语中的年龄、温度、维度和货币识别器。
    * 改进了英语中的假期识别，以正确计算基于感恩节的日期。
    * 改进了法语日期/时间，以减少非日期和非时间实体的误报。
    * 在英文的日期范围中支持日历/学校/会计年度和首字母缩写。
    * 改进了中文和日语的 PhoneNumber 识别。
    * 改进了对英语 NumberRange 的支持。
    * 性能改进。

### <a name="june-24-2019"></a>2019 年 6 月 24 日

* [序号 V2 预生成实体](luis-reference-prebuilt-ordinal-v2.md)，以支持排序，如下一个、上一个和最后一个。 仅限英语区域性。

### <a name="may-6-2019---build-conference"></a>2019 年 5 月 6 日 - //Build 会议

以下功能是在 Build 2019 大会上发布的：

* [V3 API 预览版迁移指南](luis-migration-api-v3.md)
* [改进的分析仪表板](luis-how-to-use-dashboard.md)
* [改进的预生成域](luis-reference-prebuilt-domains.md)
* [动态列表实体](schema-change-prediction-runtime.md#dynamic-lists-passed-in-at-prediction-time)
* [外部实体](schema-change-prediction-runtime.md#external-entities-passed-in-at-prediction-time)


## <a name="service-updates"></a>服务更新

[认知服务的 Azure 更新公告](https://www.azure.cn/what-is-new/)
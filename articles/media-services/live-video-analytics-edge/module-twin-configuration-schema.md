---
title: 模块孪生 JSON 架构 - Azure
description: 本主题介绍 IoT Edge 上的实时视频分析的模块孪生 JSON 架构。
ms.topic: conceptual
origin.date: 04/27/2020
ms.date: 09/07/2020
ms.openlocfilehash: a5c3e47bc596550151d7b2e6c3846a458fb69af2
ms.sourcegitcommit: 2eb5a2f53b4b73b88877e962689a47d903482c18
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/03/2020
ms.locfileid: "89414031"
---
# <a name="module-twin-json-schema"></a>模块孪生 JSON 架构

设备孪生是存储设备状态信息（包括元数据、配置和条件）的 JSON 文档。 Azure IoT 中心为连接到 IoT 中心的每台设备保留一个设备孪生。 有关详细说明，请参阅[了解并在 IoT 中心内使用模块孪生](../../iot-hub/iot-hub-devguide-module-twins.md)。

本主题介绍 IoT Edge 上的实时视频分析的模块孪生 JSON 架构。

> [!NOTE]
> 若要有权访问媒体服务资源和媒体服务 API，必须先进行身份验证。 有关详细信息，请参阅[访问 Azure 媒体服务 API](../latest/media-services-apis-overview.md#accessing-the-azure-media-services-api)。

## <a name="module-twin-properties"></a>模块孪生属性

IoT Edge 上的实时视频分析公开以下模块孪生属性。 

|属性 |必须 |动态 |说明 |
|---|---|---|---|
|applicationDataDirectory |是 |否 |用于持久配置的已装入卷的路径。 |
|azureMediaServicesArmId |是 |否 |媒体服务帐户的唯一 Azure 资源管理器标识符。|
|aadTenantId |是 |否 |客户 Azure AD 租户 ID。|
|aadServicePrincipalAppId |“是” |“是” |客户创建的 Azure AD AppId。|
|aadServicePrincipalCertificate |是<sup>*</sup>  |是 |客户创建的 Azure AD AppId 证书。|
|aadServicePrincipalPassword |是<sup>*</sup>  |是 |客户创建的 Azure AD AppId 密码。|
|aadEndpoint |否 |否 |特定于云的 Azure AD 终结点。 <br/>默认：`https://login.chinacloudapi.cn` |
|aadResourceId |否 |否 |特定于云的 Azure AD 受众/资源 ID <br/>默认：`https://management.core.chinacloudapi.cn/` |
|armEndpoint |否 |否 |特定于云的 Azure 资源管理器终结点。 <br/>默认：`https://management.chinacloudapi.cn/` |
|diagnosticsLevel |否 |是 |事件详细程度： <br/>信息 &#x02758; 警告 &#x02758; 错误 &#x02758; 严重 &#x02758; 无 |
|diagnosticsEventsOutputName |否 |是 |诊断事件的中心输出。 <br/>（空表示未发布诊断）|
|operationalEventsOutputName|否|是|操作事件的中心输出。<br/>（空表示未发布操作事件）
|logLevel|否|是|下列类型作之一： <br/>&#x000B7; 详细<br/>&#x000B7; 信息（默认值）<br/>&#x000B7; 警告<br/>&#x000B7; 错误<br/>&#x000B7; 无|
|logCategories|否|是|以下内容的逗号分隔列表：应用程序、MediaPipeline、事件 <br/>默认值：应用程序、事件|
|debugLogsDirectory|否|是|调试日志的目录。 如果存在则生成日志，如果不存在则禁用调试日志。

<sup>*</sup>必须提供服务主体证书或密码。 

无需重启模块即可更新动态属性。 可按照[访问媒体服务 API](../latest/access-api-howto.md) 一文中的说明，获取其中几个属性的值。 

有关可选诊断设置的角色的详细信息，请参阅[监视和日志记录](monitoring-logging.md)。

```
{ 
    "properties.desired": { 
        // Required 
        "applicationDataDirectory": "/var/lib/azuremediaservices", 
        "azureMediaServicesArmId": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/<rgname>/providers/microsoft.media/mediaservices/<ams_account>", 
        "aadTenantId": "00000000-0000-0000-0000-000000000000", 
        "aadServicePrincipalAppId": "00000000-0000-0000-0000-000000000000", 
        "aadServicePrincipalPassword": "{Service principal password}", 

        // Optional API Access 
        "aadEndpoint": "https://<aad-endpoint>", 
        "aadResourceId": "https://management.core.chinacloudapi.cn/", 
        "armEndpoint": "https://management.chinacloudapi.cn/", 
        
        // Optional Diagnostics 
        "diagnosticsEventsOutputName": "lvaEdgeDiagnostics",
        "operationalEventsOutputName": "lvaEdgeOperational",
        "logLevel": "Information",
        "logCategories": "Application,Events"
    } 
} 
```

## <a name="next-steps"></a>后续步骤

[直接方法](direct-methods.md)

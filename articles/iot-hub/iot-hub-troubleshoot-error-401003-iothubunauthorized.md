---
title: 排查 Azure IoT 中心错误 401003 IoTHubUnauthorized
description: 了解如何修复错误 401003 IoTHubUnauthorized
author: jlian
manager: briz
ms.service: iot-hub
services: iot-hub
ms.topic: troubleshooting
origin.date: 01/30/2020
ms.date: 11/17/2020
ms.author: v-yiso
ms.openlocfilehash: 0c2cb0e8a55b7e3dde54767ba26e4b42a344c1da
ms.sourcegitcommit: 6b499ff4361491965d02bd8bf8dde9c87c54a9f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/06/2020
ms.locfileid: "94328470"
---
# <a name="401003-iothubunauthorized"></a>401003 IoTHubUnauthorized

本文介绍了 **401003 IoTHubUnauthorized** 错误的原因和解决方案。

## <a name="symptoms"></a>症状

### <a name="symptom-1"></a>症状 1

在日志中，你会看到这样一种情况：设备断开连接并出现“401003 IoTHubUnauthorized”，接着出现“404104 DeviceConnectionClosedRemotely”，然后在不久之后成功连接 。

### <a name="symptom-2"></a>症状 2

对 IoT 中心的请求失败，并出现以下错误消息之一：

* 缺少 Authorization 标头
* IotHub '\*' 未包含指定的设备 '\*'
* 授权规则 '\*' 不允许访问 '\*'
* 此设备的身份验证失败，请续订令牌或证书，然后重新连接
* 指纹与配置不匹配：指纹：SHA1Hash=\*, SHA2Hash=\*；配置：PrimaryThumbprint=\*, SecondaryThumbprint=\*

## <a name="cause"></a>原因

### <a name="cause-1"></a>原因 1

对于 MQTT，某些 SDK 依赖于 IoT 中心在 SAS 令牌过期时发出断开连接的指令，以便知道何时刷新它。 因此， 

1. SAS 令牌过期
1. IoT 中心会注意到过期，将设备断开连接并显示 **401003 IoTHubUnauthorized**
1. 设备完成断开连接并显示 **404104 DeviceConnectionClosedRemotely**
1. IoT SDK 生成一个新的 SAS 令牌
1. 设备成功重新连接到 IoT 中心

### <a name="cause-2"></a>原因 2

IoT 中心无法对 auth 标头、规则或密钥进行身份验证。 这可能是由于症状中提到的任何原因所致。

## <a name="solution"></a>解决方案

### <a name="solution-1"></a>解决方案 1

如果使用 IoT SDK 通过设备连接字符串进行连接，则不需要执行任何操作。 IoT SDK 会重新生成新令牌，以在 SAS 令牌过期时重新连接。 

如果担心错误数量太多，请切换到 C SDK，这会在过期之前续订 SAS 令牌。 此外，对于 AMQP，SAS 令牌可以在不断开连接的情况下进行刷新。

### <a name="solution-2"></a>解决方案 2

通常情况下，出现的错误消息应说明如何修复此错误。 如果由于某种原因无法访问错误消息详细信息，请确保：

- 所用的 SAS 或其他安全令牌未过期。
- 对于 X.509 证书身份验证，设备证书或与设备关联的 CA 证书未过期。 若要了解如何将 X.509 CA 证书注册到 IoT 中心，请参阅[在 Azure IoT 中心设置 X.509 安全性](iot-hub-security-x509-get-started.md)。
- 对于 X.509 证书指纹身份验证，需要向 IoT 中心注册设备证书的指纹。
- 授权凭据的格式正确，适用于所使用的协议。 若要了解详细信息，请参阅[控制 IoT 中心的访问权限](iot-hub-devguide-security.md)。
- 使用的授权规则对所请求的操作具有权限。

## <a name="next-steps"></a>后续步骤

- 为了更轻松地向 IoT 中心进行身份验证，我们建议使用 [Azure IoT SDK](iot-hub-devguide-sdks.md)。
- 有关 IoT 中心身份验证的详细信息，请参阅[控制 IoT 中心的访问权限](iot-hub-devguide-security.md)。

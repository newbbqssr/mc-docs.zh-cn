---
title: 使用 Windows PowerShell 部署网络控制器
description: 了解如何使用 Windows PowerShell 部署网络控制器
author: WenJason
ms.topic: how-to
origin.date: 10/16/2020
ms.date: 11/09/2020
ms.author: v-jay
ms.reviewer: JasonGerend
ms.openlocfilehash: 372a6e32a3c9b26786e3b84650d5e1856e53e388
ms.sourcegitcommit: f187b1a355e2efafea30bca70afce49a2460d0c7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/04/2020
ms.locfileid: "93330656"
---
# <a name="deploy-network-controller-using-windows-powershell"></a>使用 Windows PowerShell 部署网络控制器

> 适用于 Azure Stack HCI 版本 20H2；Windows Server 2019

本主题说明了如何使用 Windows PowerShell 在一个或多个在 Azure Stack HCI 群集上运行的虚拟机 (VM) 上部署网络控制器功能。 网络控制器是软件定义的网络 (SDN) 的组件。

还可以使用 Windows Admin Center 中的“创建群集”向导部署网络控制器。 有关详细信息，请参阅[使用 Windows Admin Center 创建 Azure Stack HCI 群集](create-cluster.md)。

> [!NOTE]
> SDN 不受支持或不可用于拉伸群集。

## <a name="using-windows-powershell"></a>使用 Windows PowerShell

可以在主机服务器上的远程桌面 (RDP) 会话中本地运行 PowerShell，也可以从管理计算机远程运行 PowerShell。

从某一管理计算机运行 PowerShell 时，请将 `-Name` 或 `-Cluster` 参数与所管理的服务器或群集的名称配合使用。 此外，在将 `-ComputerName` 参数用于服务器时，可能需要指定完全限定的域名 (FQDN)。

还将需要用于 Hyper-V 和故障转移群集的远程服务器管理工​​具 (RSAT) cmdlet 和 PowerShell 模块。 如果管理计算机上的 PowerShell 会话中尚未提供这些项，则可以使用以下命令添加它们：`Add-WindowsFeature RSAT-Clustering-PowerShell`。

## <a name="install-the-network-controller-server-role"></a>安装网络控制器服务器角色

使用此过程可在虚拟机 (VM) 上安装网络控制器服务器角色。

>[!IMPORTANT]
>不要将网络控制器服务器角色部署在物理主机上。 若要部署网络控制器，必须在安装在 Hyper-V 主机上的 Hyper-V VM 上安装网络控制器服务器角色。 在三个不同 Hyper-V 主机中的 VM 上安装网络控制器之后，必须通过将主机添加到网络控制器，为软件定义的网络 (SDN) 启用 Hyper-V 主机。 这样做会使 SDN 软件负载均衡器可正常工作。

执行此过程需要管理员组中的成员身份或等效身份。  

>[!NOTE]
>如果要使用服务器管理器而不是 Windows PowerShell 来安装网络控制器，请参阅[使用服务器管理器安装网络控制器服务器角色](https://docs.microsoft.com/windows-server/networking/sdn/technologies/network-controller/install-the-network-controller-server-role-using-server-manager)

若要安装网络控制器，请键入以下命令：

```powershell
Install-WindowsFeature -Name NetworkController -IncludeManagementTools
```

安装网络控制器需要重新启动计算机。 若要执行此操作，请键入以下命令：

```powershell
Restart-Computer
```

## <a name="configure-the-network-controller-cluster"></a>配置网络控制器群集

网络控制器群集可为网络控制器应用程序提供高可用性和可伸缩性（可以在创建群集之后配置该应用程序，并在群集上托管它）。

>[!NOTE]
>可以直接在安装了网络控制器的 VM 上执行以下部分中的过程，也可以从运行 Windows Admin Center 的远程计算机执行这些过程。 此外，执行此过程需要管理员组中的成员身份或等效身份。 如果安装了网络控制器的计算机或 VM 已加入域，则用户帐户必须是域用户组的成员。

可以通过创建节点对象，然后配置群集来创建网络控制器群集。

### <a name="create-a-node-object"></a>创建节点对象

需要为作为网络控制器群集成员的每个 VM 创建一个节点对象。

若要创建节点对象，请键入以下命令。 确保为每个参数使用适用于部署的值。  

```powershell
New-NetworkControllerNodeObject -Name <string> -Server "ServerName" -FaultDomain "SiteName" -RestInterface "Name" [-NodeCertificate <X509Certificate2>]
```

下表提供 `New-NetworkControllerNodeObject` 命令的每个参数的说明。

|参数|说明|
|-------------|---------------|
|名称|Name 参数指定要添加到群集的服务器的友好名称|
|服务器|Server 参数指定要添加到群集的服务器的主机名、完全限定的域名 (FQDN) 或 IP 地址。 对于已加入域的计算机，需要 FQDN。|
|FaultDomain|FaultDomain 参数指定要添加到群集的服务器的容错域。 此参数定义可能与要添加到群集的服务器同时遇到故障的服务器。 此故障可能是由于共享物理依赖项（如电源和网络源）所导致。 容错域通常表示与这些共享依赖项相关的层次结构，更多服务器可能会从容错域树中的更高点一起发生故障。 在运行时期间，网络控制器会考虑群集中的容错域，并尝试分散网络控制器服务以便它们处于单独的容错域中。 在任何一个容错域发生故障时，此过程有助于确保该服务及其状态的可用性不遭到破坏。 容错域以分层格式指定。 例如：“Fd:/DC1/Rack1/Host1”，其中 DC1 是数据中心名称，Rack1 是机架名称，Host1 是放置节点的主机的名称。|
|RestInterface|RestInterface 参数指定表述性状态转移 (REST) 通信终止的节点上接口的名称。 此网络控制器接口从网络的管理层接收 Northbound API 请求。|
|NodeCertificate|NodeCertificate 参数指定网络控制器用于计算机身份验证的证书。 如果使用基于证书的身份验证在群集内进行通信，则需要证书；证书还用于加密网络控制器服务之间的流量。 证书使用者名称必须与节点的 DNS 名称相同。|

### <a name="configure-the-cluster"></a>配置群集

若要配置群集，请键入以下命令。 确保为每个参数使用适用于部署的值。

```powershell
Install-NetworkControllerCluster -Node "NetworkControllerNodeName" -ClusterAuthentication "ClusterAuthenticationType" [-ManagementSecurityGroup <string>][-DiagnosticLogLocation <string>][-LogLocationCredential <PSCredential>] [-CredentialEncryptionCertificate <X509Certificate2>][-Credential <PSCredential>][-CertificateThumbprint <String>] [-UseSSL][-ComputerName <string>][-LogSizeLimitInMBs<UInt32>] [-LogTimeLimitInDays<UInt32>]
```

下表提供 `Install-NetworkControllerCluster` 命令的每个参数的说明。
  
|参数|说明|
|-------------|---------------|
|ClusterAuthentication|ClusterAuthentication 参数指定用于保护节点之间的通信，还用于加密网络控制器服务之间的流量的身份验证类型。 支持的值为 Kerberos、X509 和 None  。 Kerberos 身份验证使用域帐户，仅当网络控制器节点已加入域时才能使用。 如果指定基于 X509 的身份验证，则必须在 NetworkControllerNode 对象中提供证书。 此外，必须在运行此命令之前手动预配证书。|
|ManagementSecurityGroup|ManagementSecurityGroup 参数指定包含允许从远程计算机运行管理 cmdlet 的用户的安全组名称。 这仅在 ClusterAuthentication 为 Kerberos 时才适用。 必须指定域安全组，而不是本地计算机上的安全组。|
|节点|Node 参数指定使用 New-NetworkControllerNodeObject 命令创建的网络控制器节点的列表 。|
|DiagnosticLogLocation|DiagnosticLogLocation 参数指定用于定期上传诊断日志的共享位置。 如果未指定此参数的值，则日志会在每个节点上本地存储。 日志以本地方式存储在 %systemdrive%\Windows\tracing\SDNDiagnostics 文件夹中。 群集日志以本地方式存储在 %systemdrive%\ProgramData\Microsoft\Service Fabric\log\Traces 文件夹中。|
|LogLocationCredential|LogLocationCredential 参数指定访问用于存储日志的共享位置所需的凭据。|
|CredentialEncryptionCertificate|CredentialEncryptionCertificate 参数指定一个证书，网络控制器使用该证书对用于访问网络控制器二进制文件和 LogLocationCredential（如果指定）的凭据进行加密。 必须在运行此命令之前在所有网络控制器节点上预配证书，并且必须在所有群集节点上注册相同证书。 在生产环境中，建议使用此参数保护网络控制器二进制文件和日志。 如果不使用此参数，凭据会以明文形式存储，可能由任何未经授权的用户滥用。|
|凭据|仅当从远程计算机运行此命令时，此参数才是必需的。 Credential 参数指定有权在目标计算机上运行此命令的用户帐户。|
|CertificateThumbprint|仅当从远程计算机运行此命令时，此参数才是必需的。 CertificateThumbprint 参数指定有权在目标计算机上运行此命令的用户帐户的数字公钥证书 (X509)。|
|UseSSL|仅当从远程计算机运行此命令时，此参数才是必需的。 UseSSL 参数指定用于建立与远程计算机的连接的安全套接字层 (SSL) 协议。 默认情况下，不使用 SSL。|
|计算机名|ComputerName 参数指定在其上运行此命令的网络控制器节点。 如果未指定此参数的值，则默认情况下使用本地计算机。|
|LogSizeLimitInMBs|此参数指定网络控制器可以存储的最大日志大小（以 MB 为单位）。 日志以循环方式存储。 如果提供了 DiagnosticLogLocation，则此参数的默认值为 40 GB。 如果未提供 DiagnosticLogLocation，则日志存储在网络控制器节点上，此参数的默认值为 15 GB。|
|LogTimeLimitInDays|此参数指定存储日志的持续时间限制（以天为单位）。 日志以循环方式存储。 此参数的默认值为 3 天。|

## <a name="configure-the-network-controller-application"></a>配置网络控制器应用程序

若要配置网络控制器应用程序，请键入以下命令。 确保为每个参数使用适用于部署的值。

```powershell
Install-NetworkController -Node <NetworkControllerNode[]> -ClientAuthentication <ClientAuthentication>  [-ClientCertificateThumbprint <string[]>]  [-ClientSecurityGroup <string>] -ServerCertificate <X509Certificate2> [-RESTIPAddress <String>] [-RESTName <String>] [-Credential <PSCredential>][-CertificateThumbprint <String> ] [-UseSSL]
```

下表提供 `Install-NetworkController` 命令的每个参数的说明。

|参数|说明|
|-------------|---------------|
|ClientAuthentication|ClientAuthentication 参数指定用于保护 REST 与网络控制器之间的通信的身份验证类型。 支持的值为 Kerberos、X509 和 None  。 Kerberos 身份验证使用域帐户，仅当网络控制器节点已加入域时才能使用。 如果指定基于 X509 的身份验证，则必须在 NetworkControllerNode 对象中提供证书。 此外，必须在运行此命令之前手动预配证书。|
|节点|Node 参数指定使用 New-NetworkControllerNodeObject 命令创建的网络控制器节点的列表 。|
|ClientCertificateThumbprint|仅当为网络控制器客户端使用基于证书的身份验证时，此参数才是必需的。 ClientCertificateThumbprint 参数指定注册到 Northbound 层上客户端的证书的指纹。|
|ServerCertificate|ServerCertificate 参数指定网络控制器用于向客户端证明其身份的证书。 服务器证书必须在增强型密钥使用扩展中包括服务器身份验证用途，并且必须由客户端所信任的 CA 颁发给网络控制器。|
|RESTIPAddress|对于网络控制器的单节点部署，不需要为 RESTIPAddress 指定值。 对于多节点部署，RESTIPAddress 参数以 CIDR 表示法指定 REST 终结点的 IP 地址。 例如 192.168.1.10/24。 ServerCertificate 的使用者名称值必须解析为 RESTIPAddress 参数的值 。 当所有节点都处于同一子网中时，必须为所有多节点网络控制器部署指定此参数。 如果节点处于不同子网中，则必须使用 RestName 参数，而不是使用 RESTIPAddress 。|
|RestName|对于网络控制器的单节点部署，不需要为 RestName 指定值。 仅当多节点部署的节点处于不同子网中时，才必须为 RestName 指定值。 对于多节点部署，RestName 参数指定网络控制器群集的 FQDN。|
|ClientSecurityGroup|ClientSecurityGroup 参数指定其成员是网络控制器客户端的 Active Directory 安全组的名称。 仅当对 ClientAuthentication 使用 Kerberos 身份验证时，此参数才是必需的。 安全组必须包含从中访问 REST API 的帐户，你必须在运行此命令之前创建安全组并添加成员。|
|凭据|仅当从远程计算机运行此命令时，此参数才是必需的。 Credential 参数指定有权在目标计算机上运行此命令的用户帐户。|
|CertificateThumbprint|仅当从远程计算机运行此命令时，此参数才是必需的。 CertificateThumbprint 参数指定有权在目标计算机上运行此命令的用户帐户的数字公钥证书 (X509)。|
|UseSSL|仅当从远程计算机运行此命令时，此参数才是必需的。 UseSSL 参数指定用于建立与远程计算机的连接的安全套接字层 (SSL) 协议。 默认情况下，不使用 SSL。|

完成网络控制器应用程序的配置之后，网络控制器部署已完成。

## <a name="network-controller-deployment-validation"></a>网络控制器部署验证

若要验证网络控制器部署，可以将凭据添加到网络控制器，然后检索该凭据。

如果使用 Kerberos 作为 `ClientAuthentication` 类型，则需要创建的 ClientSecurityGroup 组中的成员身份才能执行此过程。

1. 在客户端计算机上，如果使用 Kerberos 作为 `ClientAuthentication` 类型，请使用作为 ClientSecurityGroup 组的成员的用户帐户登录。

1. 在 PowerShell 中，键入以下命令。 确保为每个参数使用适用于部署的值。

    ```powershell
    $cred=New-Object Microsoft.Windows.Networkcontroller.credentialproperties
    $cred.type="usernamepassword"
    $cred.username="admin"
    $cred.value="abcd"

    New-NetworkControllerCredential -ConnectionUri "https://networkcontroller"-Properties $cred -ResourceId "cred1"
    ```

1. 若要检索添加到网络控制器中的凭据，请键入以下命令。 确保为每个参数使用适用于部署的值。

    ```powershell
    Get-NetworkControllerCredential -ConnectionUri https://networkcontroller -ResourceId cred1  
    ```

1. 查看命令输出，它应类似于以下示例输出。

    ```powershell
    Tags                   :
    ResourceRef     : /credentials/cred1
    CreatedTime    : 1/1/0001 12:00:00 AM
    InstanceId        : e16ffe62-a701-4d31-915e-7234d4bc5a18
    Etag                  : W/"1ec59631-607f-4d3e-ac78-94b0822f3a9d"
    ResourceMetadata :
    ResourceId       : cred1
    Properties       : Microsoft.Windows.NetworkController.CredentialProperties
    ```

    > [!NOTE]
    > 运行 `Get-NetworkControllerCredential` 命令时，可以使用点运算符列出凭据的属性，从而将该命令的输出分配给变量。 例如：`$cred.Properties`。

## <a name="additional-powershell-commands-for-network-controller"></a>用于网络控制器的其他 PowerShell 命令

部署网络控制器之后，可以使用 PowerShell 命令管理和修改部署。 下面是一些可以对部署进行的更改。

- 修改网络控制器节点、群集和应用程序设置

- 删除网络控制器群集和应用程序

- 管理网络控制器群集节点，包括添加、删除、启用和禁用节点。

下表提供了可用于完成这些任务的 PowerShell 命令的语法。

|任务|命令|语法|
|--------|-------|----------|
|修改网络控制器群集设置|Set-NetworkControllerCluster|`Set-NetworkControllerCluster [-ManagementSecurityGroup <string>][-Credential <PSCredential>] [-computerName <string>][-CertificateThumbprint <String> ] [-UseSSL]`
|修改网络控制器应用程序设置|Set-NetworkController|`Set-NetworkController [-ClientAuthentication <ClientAuthentication>] [-Credential <PSCredential>] [-ClientCertificateThumbprint <string[]>] [-ClientSecurityGroup <string>] [-ServerCertificate <X509Certificate2>] [-RestIPAddress <String>] [-ComputerName <String>][-CertificateThumbprint <String> ] [-UseSSL]`
|修改网络控制器节点设置|Set-NetworkControllerNode|`Set-NetworkControllerNode -Name <string> > [-RestInterface <string>] [-NodeCertificate <X509Certificate2>] [-Credential <PSCredential>] [-ComputerName <string>][-CertificateThumbprint <String> ] [-UseSSL]`
|修改网络控制器诊断设置|Set-NetworkControllerDiagnostic|`Set-NetworkControllerDiagnostic [-LogScope <string>] [-DiagnosticLogLocation <string>] [-LogLocationCredential <PSCredential>] [-UseLocalLogLocation] >] [-LogLevel <loglevel>][-LogSizeLimitInMBs <uint32>] [-LogTimeLimitInDays <uint32>] [-Credential <PSCredential>] [-ComputerName <string>][-CertificateThumbprint <String> ] [-UseSSL]`
|删除网络控制器应用程序|Uninstall-NetworkController|`Uninstall-NetworkController [-Credential <PSCredential>][-ComputerName <string>] [-CertificateThumbprint <String> ] [-UseSSL]`
|删除网络控制器群集|Uninstall-NetworkControllerCluster|`Uninstall-NetworkControllerCluster [-Credential <PSCredential>][-ComputerName <string>][-CertificateThumbprint <String> ] [-UseSSL]`
|向网络控制器群集添加节点|Add-NetworkControllerNode|`Add-NetworkControllerNode -FaultDomain <String> -Name <String> -RestInterface <String> -Server <String> [-CertificateThumbprint <String> ] [-ComputerName <String> ] [-Credential <PSCredential> ] [-Force] [-NodeCertificate <X509Certificate2> ] [-PassThru] [-UseSsl]`
|禁用网络控制器群集节点|Disable-NetworkControllerNode|`Disable-NetworkControllerNode -Name <String> [-CertificateThumbprint <String> ] [-ComputerName <String> ] [-Credential <PSCredential> ] [-PassThru] [-UseSsl]`
|启用网络控制器群集节点|Enable-NetworkControllerNode|`Enable-NetworkControllerNode -Name <String> [-CertificateThumbprint <String> ] [-ComputerName <String> ] [-Credential <PSCredential> ] [-PassThru] [-UseSsl]`
|从群集中删除网络控制器节点|Remove-NetworkControllerNode|`Remove-NetworkControllerNode [-CertificateThumbprint <String> ] [-ComputerName <String> ] [-Credential <PSCredential> ] [-Force] [-Name <String> ] [-PassThru] [-UseSsl]`

若要了解详细信息，请参阅 [NetworkController](https://docs.microsoft.com/powershell/module/networkcontroller) 上适用于网络控制器的 Windows PowerShell 参考文档。

## <a name="sample-network-controller-configuration-script"></a>示例网络控制器配置脚本

下面的示例配置脚本演示如何创建多节点网络控制器群集并安装网络控制器应用程序。 此外，`$cert` 变量会从本地计算机证书存储中选择与使用者名称字符串“networkController.contoso.com”匹配的证书。

```powershell
$a = New-NetworkControllerNodeObject -Name "Node1" -Server "NCNode1.contoso.com" -FaultDomain "fd:/rack1/host1" -RestInterface Internal
$b = New-NetworkControllerNodeObject -Name "Node2" -Server "NCNode2.contoso.com" -FaultDomain "fd:/rack1/host2" -RestInterface Internal
$c = New-NetworkControllerNodeObject -Name "Node3" -Server "NCNode3.contoso.com" -FaultDomain "fd:/rack1/host3" -RestInterface Internal

$cert= get-item Cert:\LocalMachine\My | get-ChildItem | where {$_.Subject -imatch "networkController.contoso.com" }

Install-NetworkControllerCluster -Node @($a,$b,$c)  -ClusterAuthentication Kerberos -DiagnosticLogLocation \\share\Diagnostics - ManagementSecurityGroup Contoso\NCManagementAdmins -CredentialEncryptionCertificate $cert  
Install-NetworkController -Node @($a,$b,$c) -ClientAuthentication Kerberos -ClientSecurityGroup Contoso\NCRESTClients -ServerCertificate $cert -RestIpAddress 10.0.0.1/24
```

## <a name="next-steps"></a>后续步骤

如果未将 Kerberos 用于网络控制器部署，则必须部署证书。 有关详细信息，请参阅[网络控制器的部署后步骤](https://docs.microsoft.com/windows-server/networking/sdn/technologies/network-controller/post-deploy-steps-nc)。

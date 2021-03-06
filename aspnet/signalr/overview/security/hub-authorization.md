---
uid: signalr/overview/security/hub-authorization
title: "身份验证和授权 SignalR 集线器的 |Microsoft 文档"
author: pfletcher
description: "本主题介绍如何限制哪些用户或角色可以访问中心方法。 本主题中的软件版本使用，Visual Studio 2013.NET 4.5 SignalR 遇到..."
ms.author: aspnetcontent
manager: wpickett
ms.date: 01/05/2015
ms.topic: article
ms.assetid: a610c796-c131-473c-baef-2e6c568cb2a2
ms.technology: dotnet-signalr
ms.prod: .net-framework
msc.legacyurl: /signalr/overview/security/hub-authorization
msc.type: authoredcontent
ms.openlocfilehash: f1538c933ff9e8e680d70ce1e63d24b189be47e5
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/10/2017
---
<a name="authentication-and-authorization-for-signalr-hubs"></a>身份验证和授权 SignalR 集线器的
====================
通过[Patrick Fletcher](https://github.com/pfletcher)， [Tom FitzMacken](https://github.com/tfitzmac)

> 本主题介绍如何限制哪些用户或角色可以访问中心方法。 
> 
> ## <a name="software-versions-used-in-this-topic"></a>本主题中使用的软件版本
> 
> 
> - [Visual Studio 2013](https://www.microsoft.com/visualstudio/eng/2013-downloads)
> - .NET 4.5
> - SignalR 版本 2
>   
> 
> 
> ## <a name="previous-versions-of-this-topic"></a>本主题的早期版本
> 
> 有关 SignalR 的早期版本的信息，请参阅[SignalR 较旧版本](../older-versions/index.md)。
> 
> ## <a name="questions-and-comments"></a>问题和意见
> 
> 请留下反馈上如何喜欢本教程的方式，我们可以提高在页面底部的注释中。 如果你有与本教程不直接相关的问题，你可以发布到[ASP.NET SignalR 论坛](https://forums.asp.net/1254.aspx/1?ASP+NET+SignalR)或[StackOverflow.com](http://stackoverflow.com/)。


## <a name="overview"></a>概述

本主题包含以下各节：

- [授权属性](#authorizeattribute)
- [要求对所有中心身份验证](#requireauth)
- [自定义的授权](#custom)
- [将身份验证信息传递给客户端](#passauth)
- [用于.NET 客户端身份验证选项](#authoptions)

    - [使用 Forms 身份验证的 cookie](#cookie)
    - [Windows 身份验证](#windows)
    - [连接标头](#header)
    - [证书](#certificate)

<a id="authorizeattribute"></a>

## <a name="authorize-attribute"></a>授权属性

SignalR 提供[Authorize](https://msdn.microsoft.com/en-us/library/microsoft.aspnet.signalr.authorizeattribute(v=vs.111).aspx)特性来指定哪些用户或角色有权访问的中心或方法。 此属性位于`Microsoft.AspNet.SignalR`命名空间。 你将应用`Authorize`属性为中心或中心中的特定方法。 当你将`Authorize`特性应用到中心类，指定的授权要求适用于所有中心中的方法。 本主题提供您可以将应用的授权要求的不同类型的示例。 而无需`Authorize`特性，连接客户端可以访问集线器上的任何公共方法。

如果已定义 web 应用程序中名为"Admin"的角色，则可以指定该角色中的唯一用户可以访问替换为以下代码的集线器。

[!code-csharp[Main](hub-authorization/samples/sample1.cs)]

或者，你可以指定一个中心包含可供所有用户的一个方法，并可仅供经过身份验证的用户，如下所示的第二个方法。

[!code-csharp[Main](hub-authorization/samples/sample2.cs)]

下面的示例地址不同的授权方案：

- `[Authorize]`– 仅身份验证的用户
- `[Authorize(Roles = "Admin,Manager")]`– 仅经过身份验证中的指定角色的用户
- `[Authorize(Users = "user1,user2")]`– 仅经过身份验证与指定的用户名的用户
- `[Authorize(RequireOutgoing=false)]`– 仅经过身份验证的用户可以调用中心，但从服务器返回到客户端的调用不受限制授权，例如，当仅某些用户可以发送一条消息，但所有其他人可以接收消息时。 RequireOutgoing 属性只能应用到整个中心，不能对中心内的个人方法。 RequireOutgoing 未设置为 false，满足的授权要求的用户将调用从服务器中。

<a id="requireauth"></a>

## <a name="require-authentication-for-all-hubs"></a>要求对所有中心身份验证

你可以要求身份验证的所有中心和中心方法在你的应用程序通过调用[RequireAuthentication](https://msdn.microsoft.com/en-us/library/microsoft.aspnet.signalr.hubpipelineextensions.requireauthentication(v=vs.111).aspx)应用程序启动时的方法。 如果你有多个中心并想要为所有这些强制实施身份验证要求，可使用此方法。 使用此方法，不能指定角色、 用户或传出的授权的要求。 你仅可以指定对中心方法的访问仅限于经过身份验证的用户。 但是，你仍可以应用 Authorize 属性为中心或方法，以指定其他要求。 在属性中指定的任何要求添加到身份验证的基本要求。

下面的示例演示的启动文件，后者限制给已经过身份验证的用户所有的中心方法。

[!code-csharp[Main](hub-authorization/samples/sample3.cs)]

如果调用`RequireAuthentication()`方法处理 SignalR 请求后，会引发 SignalR`InvalidOperationException`异常。 SignalR 引发此异常，因为调用管道后，不能向非 HubPipeline 添加模块。 上面的示例演示调用`RequireAuthentication`中的方法`Configuration`会执行一次之前处理的第一个请求的方法。

<a id="custom"></a>

## <a name="customized-authorization"></a>自定义的授权

如果你需要自定义如何确定授权，你可以创建一个派生自的类`AuthorizeAttribute`，并重写[UserAuthorized](https://msdn.microsoft.com/en-us/library/microsoft.aspnet.signalr.authorizeattribute.userauthorized(v=vs.111).aspx)方法。 对于每个请求，SignalR 调用此方法来确定用户是否有权完成请求。 在重写方法中，为你的授权方案提供必要的逻辑。 下面的示例演示如何强制实施通过基于声明的标识的授权。

[!code-csharp[Main](hub-authorization/samples/sample4.cs)]

<a id="passauth"></a>

## <a name="pass-authentication-information-to-clients"></a>将身份验证信息传递给客户端

你可能需要在客户端运行的代码中使用的身份验证信息。 客户端上调用方法时，你将传递所需的信息。 例如，聊天应用程序方法无法将作为参数传递的消息，同时发布的人员的用户名称如下所示。

[!code-csharp[Main](hub-authorization/samples/sample5.cs)]

或者，你可以创建要表示的身份验证信息并将该对象作为参数传递的对象，如下所示。

[!code-csharp[Main](hub-authorization/samples/sample6.cs)]

根据恶意用户可以使用它来模拟来自该客户端的请求，应永远不会将一台客户端连接 id 传递给其他客户端。

<a id="authoptions"></a>

## <a name="authentication-options-for-net-clients"></a>用于.NET 客户端身份验证选项

当你有.NET 客户端，例如，控制台应用程序，与被限制为身份验证的用户的集线器交互时，您可以传递在 cookie、 连接标头或证书的身份验证凭据。 此部分中的示例演示如何使用这些不同的方法，以便进行用户身份验证。 它们不是功能完整的 SignalR 应用。 有关使用 SignalR 的.NET 客户端的详细信息，请参阅[中心 API 指南-.NET 客户端](../guide-to-the-api/hubs-api-guide-net-client.md)。

<a id="cookie"></a>

### <a name="cookie"></a>Cookie

当.NET 客户端与使用 ASP.NET 窗体身份验证的集线器交互时，你将需要手动对连接设置身份验证 cookie。 添加到 cookie`CookieContainer`属性[HubConnection](https://msdn.microsoft.com/en-us/library/microsoft.aspnet.signalr.client.hubs.hubconnection(v=vs.111).aspx)对象。 下面的示例演示在网页上检索身份验证 cookie 并将该 cookie 添加到连接的控制台应用。

[!code-csharp[Main](hub-authorization/samples/sample7.cs)]

控制台应用程序发送到的凭据**www.contoso.com/RemoteLogin**其引用了包含下面的代码隐藏文件的空页。

[!code-csharp[Main](hub-authorization/samples/sample8.cs)]

<a id="windows"></a>

### <a name="windows-authentication"></a>Windows 身份验证

当使用 Windows 身份验证，您可以通过传递当前用户的凭据[在](https://msdn.microsoft.com/en-us/library/system.net.credentialcache.defaultcredentials.aspx)属性。 在的值设置为连接的凭据。

[!code-csharp[Main](hub-authorization/samples/sample9.cs?highlight=6)]

<a id="header"></a>

### <a name="connection-header"></a>连接标头

如果你的应用程序不使用 cookie，你可以连接标头中传递用户信息。 例如，你可以连接标头中传递一个令牌。

[!code-csharp[Main](hub-authorization/samples/sample10.cs?highlight=6)]

然后，在中心，你将验证用户的令牌。

<a id="certificate"></a>

### <a name="certificate"></a>证书

你可以传递一个客户端证书来验证用户。 创建连接时添加证书。 下面的示例演示如何将客户端证书添加到连接;它不显示完整的控制台应用程序。 它使用[X509Certificate](https://msdn.microsoft.com/en-us/library/system.security.cryptography.x509certificates.x509certificate.aspx)提供多种不同的方式来创建证书的类。

[!code-csharp[Main](hub-authorization/samples/sample11.cs?highlight=6)]

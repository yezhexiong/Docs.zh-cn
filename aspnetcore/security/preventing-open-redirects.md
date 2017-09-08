---
title: "阻止在 ASP.NET Core 应用中的打开重定向攻击 |Microsoft 文档"
author: ardalis
description: "演示如何阻止对 ASP.NET Core 应用打开重定向攻击"
keywords: "ASP.NET 核心，安全，打开重定向攻击"
ms.author: riande
manager: wpickett
ms.date: 07/7/2017
ms.topic: article
ms.assetid: 4604e563-e91a-4ecd-b7ed-00b3f1eee2b5
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/preventing-open-redirects
ms.openlocfilehash: 7a62b08d641de5a9df5c2f7d89bf6bf97ed8e39d
ms.sourcegitcommit: 0b6c8e6d81d2b3c161cd375036eecbace46a9707
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/11/2017
---
# <a name="preventing-open-redirect-attacks-in-an-aspnet-core-app"></a><span data-ttu-id="617e6-104">阻止在 ASP.NET Core 应用中的打开重定向攻击</span><span class="sxs-lookup"><span data-stu-id="617e6-104">Preventing Open Redirect Attacks in an ASP.NET Core app</span></span>

<span data-ttu-id="617e6-105">Web 应用程序将重定向到通过如查询字符串或窗体数据请求指定的 URL 可能可能被篡改将用户重定向到外部、 恶意 URL。</span><span class="sxs-lookup"><span data-stu-id="617e6-105">A web app that redirects to a URL that is specified via the request such as the querystring or form data can potentially be tampered with to redirect users to an external, malicious URL.</span></span> <span data-ttu-id="617e6-106">此篡改称为打开重定向攻击。</span><span class="sxs-lookup"><span data-stu-id="617e6-106">This tampering is called an open redirection attack.</span></span>

<span data-ttu-id="617e6-107">每当应用程序逻辑将重定向到指定的 URL，你必须验证重定向 URL 尚未被篡改。</span><span class="sxs-lookup"><span data-stu-id="617e6-107">Whenever your application logic redirects to a specified URL, you must verify that the redirection URL hasn't been tampered with.</span></span> <span data-ttu-id="617e6-108">ASP.NET 核心具有内置功能来防止应用打开重定向 （也称为打开重定向） 攻击。</span><span class="sxs-lookup"><span data-stu-id="617e6-108">ASP.NET Core has built-in functionality to help protect apps from open redirect (also known as open redirection) attacks.</span></span>

## <a name="what-is-an-open-redirect-attack"></a><span data-ttu-id="617e6-109">打开重定向攻击是什么？</span><span class="sxs-lookup"><span data-stu-id="617e6-109">What is an open redirect attack?</span></span>

<span data-ttu-id="617e6-110">Web 应用程序频繁地将用户重定向到登录页访问要求进行身份验证的资源时。</span><span class="sxs-lookup"><span data-stu-id="617e6-110">Web applications frequently redirect users to a login page when they access resources that require authentication.</span></span> <span data-ttu-id="617e6-111">重定向 typlically 包括`returnUrl`查询字符串参数，以便用户可以在用户成功登录后返回最初请求的 url。</span><span class="sxs-lookup"><span data-stu-id="617e6-111">The redirection typlically includes a `returnUrl` querystring parameter so that the user can be returned to the originally requested URL after they have successfully logged in.</span></span> <span data-ttu-id="617e6-112">用户进行身份验证后，系统会将它们重定向到他们最初具有请求的 URL。</span><span class="sxs-lookup"><span data-stu-id="617e6-112">After the user authenticates, they are redirected to the URL they had originally requested.</span></span>

<span data-ttu-id="617e6-113">因为请求的查询字符串中指定的目标 URL，则恶意用户可能篡改查询字符串。</span><span class="sxs-lookup"><span data-stu-id="617e6-113">Because the destination URL is specified in the querystring of the request, a malicious user could tamper with the querystring.</span></span> <span data-ttu-id="617e6-114">篡改过的查询字符串可能导致要将用户重定向到外部、 恶意站点的站点。</span><span class="sxs-lookup"><span data-stu-id="617e6-114">A tampered querystring could allow the site to redirect the user to an external, malicious site.</span></span> <span data-ttu-id="617e6-115">这种技术称为打开重定向 （或重定向） 攻击。</span><span class="sxs-lookup"><span data-stu-id="617e6-115">This technique is called an open redirect (or redirection) attack.</span></span>

### <a name="an-example-attack"></a><span data-ttu-id="617e6-116">示例攻击</span><span class="sxs-lookup"><span data-stu-id="617e6-116">An example attack</span></span>

<span data-ttu-id="617e6-117">恶意用户可以开发用于允许对用户的凭据或您的应用程序上的敏感信息的恶意用户访问的攻击。</span><span class="sxs-lookup"><span data-stu-id="617e6-117">A malicious user could develop an attack intended to allow the malicious user access to a user's credentials or sensitive information on your app.</span></span> <span data-ttu-id="617e6-118">若要开始攻击，它们使用户链接到网站的登录页，单击与`returnUrl`添加到的 URL 查询字符串值。</span><span class="sxs-lookup"><span data-stu-id="617e6-118">To begin the attack, they convince the user to click a link to your site's login page, with a `returnUrl` querystring value added to the URL.</span></span> <span data-ttu-id="617e6-119">例如， [NerdDinner.com](http://nerddinner.com) （编写 ASP.NET MVC 的） 的示例应用程序包括此处这样的登录页： ``http://nerddinner.com/Account/LogOn?returnUrl=/Home/About``。</span><span class="sxs-lookup"><span data-stu-id="617e6-119">For example, the [NerdDinner.com](http://nerddinner.com) sample application (written for ASP.NET MVC) includes such a login page here: ``http://nerddinner.com/Account/LogOn?returnUrl=/Home/About``.</span></span> <span data-ttu-id="617e6-120">这种攻击然后执行下列步骤：</span><span class="sxs-lookup"><span data-stu-id="617e6-120">The attack then follows these steps:</span></span>

1. <span data-ttu-id="617e6-121">用户单击的链接``http://nerddinner.com/Account/LogOn?returnUrl=http://nerddiner.com/Account/LogOn``(请注意，第二个 URL 是 nerddi**n**er，不 nerddi**nn**er)。</span><span class="sxs-lookup"><span data-stu-id="617e6-121">User clicks a link to ``http://nerddinner.com/Account/LogOn?returnUrl=http://nerddiner.com/Account/LogOn`` (note, second URL is nerddi**n**er, not nerddi**nn**er).</span></span>
2. <span data-ttu-id="617e6-122">用户成功登录。</span><span class="sxs-lookup"><span data-stu-id="617e6-122">The user logs in successfully.</span></span>
3. <span data-ttu-id="617e6-123">用户 （通过站点） 重定向到``http://nerddiner.com/Account/LogOn``（如下所示的真正的站点的恶意站点）。</span><span class="sxs-lookup"><span data-stu-id="617e6-123">The user is redirected (by the site) to ``http://nerddiner.com/Account/LogOn`` (malicious site that looks like real site).</span></span>
4. <span data-ttu-id="617e6-124">用户再次登录 （提供恶意站点其凭据） 和重定向回真正的站点。</span><span class="sxs-lookup"><span data-stu-id="617e6-124">The user logs in again (giving malicious site their credentials) and is redirected back to the real site.</span></span>

<span data-ttu-id="617e6-125">用户将可能认为其第一次尝试登录失败，并且其第二个已成功。</span><span class="sxs-lookup"><span data-stu-id="617e6-125">The user will likely believe their first attempt to log in failed, and their second one was successful.</span></span> <span data-ttu-id="617e6-126">它们将很可能仍然无法察觉其凭据已泄漏。</span><span class="sxs-lookup"><span data-stu-id="617e6-126">They'll most likely remain unaware their credentials have been compromised.</span></span>

![打开重定向攻击过程](preventing-open-redirects/_static/open-redirection-attack-process.png)

<span data-ttu-id="617e6-128">除了登录页的某些站点提供重定向页或终结点。</span><span class="sxs-lookup"><span data-stu-id="617e6-128">In addition to login pages, some sites provide redirect pages or endpoints.</span></span> <span data-ttu-id="617e6-129">假设你的应用程序的页的打开的重定向， ``/Home/Redirect``。</span><span class="sxs-lookup"><span data-stu-id="617e6-129">Imagine your app has a page with an open redirect, ``/Home/Redirect``.</span></span> <span data-ttu-id="617e6-130">攻击者可以创建，例如，将转到一封电子邮件中的链接``[yoursite]/Home/Redirect?url=http://phishingsite.com/Home/Login``。</span><span class="sxs-lookup"><span data-stu-id="617e6-130">An attacker could create, for example, a link in an email that goes to ``[yoursite]/Home/Redirect?url=http://phishingsite.com/Home/Login``.</span></span> <span data-ttu-id="617e6-131">典型的用户将看一看 URL，并请参阅开始使用你的站点名称。</span><span class="sxs-lookup"><span data-stu-id="617e6-131">A typical user will look at the URL and see it begins with your site name.</span></span> <span data-ttu-id="617e6-132">信任的用户将单击此链接。</span><span class="sxs-lookup"><span data-stu-id="617e6-132">Trusting that, they will click the link.</span></span> <span data-ttu-id="617e6-133">打开重定向将为网络钓鱼站点，这看起来相同地区，然后发送用户和用户很可能会登录到他们认为是你的站点。</span><span class="sxs-lookup"><span data-stu-id="617e6-133">The open redirect would then send the user to the phishing site, which looks identical to yours, and the user would likely login to what they believe is your site.</span></span>

## <a name="protecting-against-open-redirect-attacks"></a><span data-ttu-id="617e6-134">针对打开重定向攻击提供保护</span><span class="sxs-lookup"><span data-stu-id="617e6-134">Protecting against open redirect attacks</span></span>

<span data-ttu-id="617e6-135">当开发 web 应用程序，会将所有用户提供数据视为不受信任。</span><span class="sxs-lookup"><span data-stu-id="617e6-135">When developing web applications, treat all user-provided data as untrustworthy.</span></span> <span data-ttu-id="617e6-136">如果你的应用程序具有基于 URL 的内容将用户重定向的功能，请确保，此类重定向仅这样的是本地应用程序中 （或已知的 URL，没有任何可能在查询字符串中提供的 URL）。</span><span class="sxs-lookup"><span data-stu-id="617e6-136">If your application has functionality that redirects the user based on the contents of the URL,  ensure that such redirects are only done locally within your app (or to a known URL, not any URL that may be supplied in the querystring).</span></span>

### <a name="localredirect"></a><span data-ttu-id="617e6-137">LocalRedirect</span><span class="sxs-lookup"><span data-stu-id="617e6-137">LocalRedirect</span></span>

<span data-ttu-id="617e6-138">使用``LocalRedirect``帮助器方法的基`Controller`类：</span><span class="sxs-lookup"><span data-stu-id="617e6-138">Use the ``LocalRedirect`` helper method from the base `Controller` class:</span></span>

```
public IActionResult SomeAction(string redirectUrl)
{
    return LocalRedirect(redirectUrl);
}
```

<span data-ttu-id="617e6-139">``LocalRedirect``如果指定非本地 URL，将引发异常。</span><span class="sxs-lookup"><span data-stu-id="617e6-139">``LocalRedirect`` will throw an exception if a non-local URL is specified.</span></span> <span data-ttu-id="617e6-140">否则，其行为就像``Redirect``方法。</span><span class="sxs-lookup"><span data-stu-id="617e6-140">Otherwise, it behaves just like the ``Redirect`` method.</span></span>

### <a name="islocalurl"></a><span data-ttu-id="617e6-141">IsLocalUrl</span><span class="sxs-lookup"><span data-stu-id="617e6-141">IsLocalUrl</span></span>

<span data-ttu-id="617e6-142">使用[IsLocalUrl](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.iurlhelper#Microsoft_AspNetCore_Mvc_IUrlHelper_IsLocalUrl_System_String_)方法以测试 Url，然后重定向：</span><span class="sxs-lookup"><span data-stu-id="617e6-142">Use the [IsLocalUrl](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.iurlhelper#Microsoft_AspNetCore_Mvc_IUrlHelper_IsLocalUrl_System_String_) method to test URLs before redirecting:</span></span>

<span data-ttu-id="617e6-143">下面的示例演示如何检查 URL 重定向之前是本地。</span><span class="sxs-lookup"><span data-stu-id="617e6-143">The following example shows how to check whether a URL is local before redirecting.</span></span>

```
private IActionResult RedirectToLocal(string returnUrl)
{
    if (Url.IsLocalUrl(returnUrl))
    {
        return Redirect(returnUrl);
    }
    else
    {
        return RedirectToAction(nameof(HomeController.Index), "Home");
    }
}
```

<span data-ttu-id="617e6-144">`IsLocalUrl`方法可防止用户无意中重定向到恶意的站点。</span><span class="sxs-lookup"><span data-stu-id="617e6-144">The `IsLocalUrl` method protects users from being inadvertently redirected to a malicious site.</span></span> <span data-ttu-id="617e6-145">你可以登录时需要本地 URL 的位置的情况下提供非本地 URL 提供的 URL 的详细信息。</span><span class="sxs-lookup"><span data-stu-id="617e6-145">You can log the details of the URL that was provided when a non-local URL is supplied in a situation where you expected a local URL.</span></span> <span data-ttu-id="617e6-146">日志记录重定向 Url 可用于诊断重定向攻击。</span><span class="sxs-lookup"><span data-stu-id="617e6-146">Logging redirect URLs may help in diagnosing redirection attacks.</span></span>
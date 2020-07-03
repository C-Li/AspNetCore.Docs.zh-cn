---
title: 使用 Identity 服务器保护 ASP.NET Core Blazor WebAssembly 托管应用
author: guardrex
description: 创建新的 Blazor 托管应用，使其使用 [IdentityServer](https://identityserver.io/) 后端在 Visual Studio 中进行身份验证
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 05/19/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: blazor/security/webassembly/hosted-with-identity-server
ms.openlocfilehash: 8251658a2bb99ec17424ec26b8a44171082fab05
ms.sourcegitcommit: 066d66ea150f8aab63f9e0e0668b06c9426296fd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/23/2020
ms.locfileid: "85243459"
---
# <a name="secure-an-aspnet-core-blazor-webassembly-hosted-app-with-identity-server"></a><span data-ttu-id="86b4f-103">使用 Identity 服务器保护 ASP.NET Core Blazor WebAssembly 托管应用</span><span class="sxs-lookup"><span data-stu-id="86b4f-103">Secure an ASP.NET Core Blazor WebAssembly hosted app with Identity Server</span></span>

<span data-ttu-id="86b4f-104">作者：[Javier Calvarro Nelson](https://github.com/javiercn) 和 [Luke Latham](https://github.com/guardrex)</span><span class="sxs-lookup"><span data-stu-id="86b4f-104">By [Javier Calvarro Nelson](https://github.com/javiercn) and [Luke Latham](https://github.com/guardrex)</span></span>

<span data-ttu-id="86b4f-105">本文介绍如何创建一个新的 Blazor 托管应用，该应用使用 [IdentityServer](https://identityserver.io/) 以对用户和 API 调用进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="86b4f-105">This article explains how to create a new Blazor hosted app that uses [IdentityServer](https://identityserver.io/) to authenticate users and API calls.</span></span>

# <a name="visual-studio"></a>[<span data-ttu-id="86b4f-106">Visual Studio</span><span class="sxs-lookup"><span data-stu-id="86b4f-106">Visual Studio</span></span>](#tab/visual-studio)

<span data-ttu-id="86b4f-107">在 Visual Studio 中：</span><span class="sxs-lookup"><span data-stu-id="86b4f-107">In Visual Studio:</span></span>

1. <span data-ttu-id="86b4f-108">创建新的 Blazor WebAssembly 应用。</span><span class="sxs-lookup"><span data-stu-id="86b4f-108">Create a new **Blazor WebAssembly** app.</span></span> <span data-ttu-id="86b4f-109">有关详细信息，请参阅 <xref:blazor/get-started>。</span><span class="sxs-lookup"><span data-stu-id="86b4f-109">For more information, see <xref:blazor/get-started>.</span></span>
1. <span data-ttu-id="86b4f-110">在“新建 Blazor 应用”对话框中，选择“身份验证”部分中的“更改”  。</span><span class="sxs-lookup"><span data-stu-id="86b4f-110">In the **Create a new Blazor app** dialog, select **Change** in the **Authentication** section.</span></span>
1. <span data-ttu-id="86b4f-111">选择“个人用户帐户”，然后选择“确定” 。</span><span class="sxs-lookup"><span data-stu-id="86b4f-111">Select **Individual User Accounts** followed by **OK**.</span></span>
1. <span data-ttu-id="86b4f-112">在“高级”部分中选中“托管的 ASP.NET Core”复选框 。</span><span class="sxs-lookup"><span data-stu-id="86b4f-112">Select the **ASP.NET Core hosted** checkbox in the **Advanced** section.</span></span>
1. <span data-ttu-id="86b4f-113">选择“创建”按钮。</span><span class="sxs-lookup"><span data-stu-id="86b4f-113">Select the **Create** button.</span></span>

# <a name="net-core-cli"></a>[<span data-ttu-id="86b4f-114">.NET Core CLI</span><span class="sxs-lookup"><span data-stu-id="86b4f-114">.NET Core CLI</span></span>](#tab/netcore-cli/)

<span data-ttu-id="86b4f-115">要在命令行界面中创建应用，请执行以下命令：</span><span class="sxs-lookup"><span data-stu-id="86b4f-115">To create the app in a command shell, execute the following command:</span></span>

```dotnetcli
dotnet new blazorwasm -au Individual -ho
```

<span data-ttu-id="86b4f-116">要指定输出位置（如果它不存在，则创建一个项目文件夹），请在命令中包含带有路径（例如 `-o BlazorSample`）的输出选项。</span><span class="sxs-lookup"><span data-stu-id="86b4f-116">To specify the output location, which creates a project folder if it doesn't exist, include the output option in the command with a path (for example, `-o BlazorSample`).</span></span> <span data-ttu-id="86b4f-117">该文件夹名称还会成为项目名称的一部分。</span><span class="sxs-lookup"><span data-stu-id="86b4f-117">The folder name also becomes part of the project's name.</span></span>

---

## <a name="server-app-configuration"></a><span data-ttu-id="86b4f-118">服务器应用配置</span><span class="sxs-lookup"><span data-stu-id="86b4f-118">Server app configuration</span></span>

<span data-ttu-id="86b4f-119">以下部分介绍了在包括身份验证支持的情况下对项目添加的内容。</span><span class="sxs-lookup"><span data-stu-id="86b4f-119">The following sections describe additions to the project when authentication support is included.</span></span>

### <a name="startup-class"></a><span data-ttu-id="86b4f-120">Startup 类</span><span class="sxs-lookup"><span data-stu-id="86b4f-120">Startup class</span></span>

<span data-ttu-id="86b4f-121">`Startup` 类包含以下添加项。</span><span class="sxs-lookup"><span data-stu-id="86b4f-121">The `Startup` class has the following additions.</span></span>

* <span data-ttu-id="86b4f-122">在 `Startup.ConfigureServices`中：</span><span class="sxs-lookup"><span data-stu-id="86b4f-122">In `Startup.ConfigureServices`:</span></span>

  * <span data-ttu-id="86b4f-123">ASP.NET Core Identity：</span><span class="sxs-lookup"><span data-stu-id="86b4f-123">ASP.NET Core Identity:</span></span>

    ```csharp
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlite(
            Configuration.GetConnectionString("DefaultConnection")));

    services.AddDefaultIdentity<ApplicationUser>(options => 
            options.SignIn.RequireConfirmedAccount = true)
        .AddEntityFrameworkStores<ApplicationDbContext>();
    ```

  * <span data-ttu-id="86b4f-124">包含附加 <xref:Microsoft.Extensions.DependencyInjection.IdentityServerBuilderConfigurationExtensions.AddApiAuthorization%2A> 帮助器方法的 IdentityServer，该方法在 IdentityServer 之上设置默认 ASP.NET Core 约定：</span><span class="sxs-lookup"><span data-stu-id="86b4f-124">IdentityServer with an additional <xref:Microsoft.Extensions.DependencyInjection.IdentityServerBuilderConfigurationExtensions.AddApiAuthorization%2A> helper method that sets up default ASP.NET Core conventions on top of IdentityServer:</span></span>

    ```csharp
    services.AddIdentityServer()
        .AddApiAuthorization<ApplicationUser, ApplicationDbContext>();
    ```

  * <span data-ttu-id="86b4f-125">包含附加 <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilderExtensions.AddIdentityServerJwt%2A> 帮助器方法的身份验证，该方法将应用配置为验证由 IdentityServer 生成的 JWT 令牌：</span><span class="sxs-lookup"><span data-stu-id="86b4f-125">Authentication with an additional <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilderExtensions.AddIdentityServerJwt%2A> helper method that configures the app to validate JWT tokens produced by IdentityServer:</span></span>

    ```csharp
    services.AddAuthentication()
        .AddIdentityServerJwt();
    ```

* <span data-ttu-id="86b4f-126">在 `Startup.Configure`中：</span><span class="sxs-lookup"><span data-stu-id="86b4f-126">In `Startup.Configure`:</span></span>

  * <span data-ttu-id="86b4f-127">IdentityServer 中间件公开 Open ID Connect (OIDC) 终结点：</span><span class="sxs-lookup"><span data-stu-id="86b4f-127">The IdentityServer middleware exposes the Open ID Connect (OIDC) endpoints:</span></span>

    ```csharp
    app.UseIdentityServer();
    ```

  * <span data-ttu-id="86b4f-128">身份验证中间件负责验证请求凭据并在请求上下文中设置用户：</span><span class="sxs-lookup"><span data-stu-id="86b4f-128">The Authentication middleware is responsible for validating request credentials and setting the user on the request context:</span></span>

    ```csharp
    app.UseAuthentication();
    ```

  * <span data-ttu-id="86b4f-129">授权中间件支持授权功能：</span><span class="sxs-lookup"><span data-stu-id="86b4f-129">Authorization Middleware enables authorization capabilities:</span></span>

    ```csharp
    app.UseAuthentication();
    app.UseAuthorization();
    ```

### <a name="addapiauthorization"></a><span data-ttu-id="86b4f-130">AddApiAuthorization</span><span class="sxs-lookup"><span data-stu-id="86b4f-130">AddApiAuthorization</span></span>

<span data-ttu-id="86b4f-131"><xref:Microsoft.Extensions.DependencyInjection.IdentityServerBuilderConfigurationExtensions.AddApiAuthorization%2A> 帮助器方法配置 [IdentityServer](https://identityserver.io/) 用于 ASP.NET Core 方案。</span><span class="sxs-lookup"><span data-stu-id="86b4f-131">The <xref:Microsoft.Extensions.DependencyInjection.IdentityServerBuilderConfigurationExtensions.AddApiAuthorization%2A> helper method configures [IdentityServer](https://identityserver.io/) for ASP.NET Core scenarios.</span></span> <span data-ttu-id="86b4f-132">IdentityServer 是一个功能强大且可扩展的框架，用于处理应用安全问题。</span><span class="sxs-lookup"><span data-stu-id="86b4f-132">IdentityServer is a powerful and extensible framework for handling app security concerns.</span></span> <span data-ttu-id="86b4f-133">IdentityServer 公开大多数情况下不必要的复杂性。</span><span class="sxs-lookup"><span data-stu-id="86b4f-133">IdentityServer exposes unnecessary complexity for the most common scenarios.</span></span> <span data-ttu-id="86b4f-134">因此，提供了一组约定和配置选项作为良好的起点。</span><span class="sxs-lookup"><span data-stu-id="86b4f-134">Consequently, a set of conventions and configuration options is provided that we consider a good starting point.</span></span> <span data-ttu-id="86b4f-135">一旦身份验证需要更改，就可使用 IdentityServer 的完整功能自定义身份验证以满足应用的要求。</span><span class="sxs-lookup"><span data-stu-id="86b4f-135">Once your authentication needs change, the full power of IdentityServer is available to customize authentication to suit an app's requirements.</span></span>

### <a name="addidentityserverjwt"></a><span data-ttu-id="86b4f-136">AddIdentityServerJwt</span><span class="sxs-lookup"><span data-stu-id="86b4f-136">AddIdentityServerJwt</span></span>

<span data-ttu-id="86b4f-137"><xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilderExtensions.AddIdentityServerJwt%2A> 帮助器方法将应用的策略方案配置为默认身份验证处理程序。</span><span class="sxs-lookup"><span data-stu-id="86b4f-137">The <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilderExtensions.AddIdentityServerJwt%2A> helper method configures a policy scheme for the app as the default authentication handler.</span></span> <span data-ttu-id="86b4f-138">该策略配置为允许 Identity 处理路由到 Identity URL 空间 `/Identity` 中任何子路径的所有请求。</span><span class="sxs-lookup"><span data-stu-id="86b4f-138">The policy is configured to allow Identity to handle all requests routed to any subpath in the Identity URL space `/Identity`.</span></span> <span data-ttu-id="86b4f-139"><xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerHandler> 处理所有其他请求。</span><span class="sxs-lookup"><span data-stu-id="86b4f-139">The <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerHandler> handles all other requests.</span></span> <span data-ttu-id="86b4f-140">此外，此方法还可以：</span><span class="sxs-lookup"><span data-stu-id="86b4f-140">Additionally, this method:</span></span>

* <span data-ttu-id="86b4f-141">向具有默认 `{APPLICATION NAME}API` 范围的 IdentityServer 注册 `{APPLICATION NAME}API` API 资源。</span><span class="sxs-lookup"><span data-stu-id="86b4f-141">Registers an `{APPLICATION NAME}API` API resource with IdentityServer with a default scope of `{APPLICATION NAME}API`.</span></span>
* <span data-ttu-id="86b4f-142">配置 JWT 持有者令牌中间件以验证 IdentityServer 为应用颁发的令牌。</span><span class="sxs-lookup"><span data-stu-id="86b4f-142">Configures the JWT Bearer Token Middleware to validate tokens issued by IdentityServer for the app.</span></span>

### <a name="weatherforecastcontroller"></a><span data-ttu-id="86b4f-143">WeatherForecastController</span><span class="sxs-lookup"><span data-stu-id="86b4f-143">WeatherForecastController</span></span>

<span data-ttu-id="86b4f-144">在 `WeatherForecastController` (`Controllers/WeatherForecastController.cs`) 中，[`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) 特性应用于该类。</span><span class="sxs-lookup"><span data-stu-id="86b4f-144">In the `WeatherForecastController` (`Controllers/WeatherForecastController.cs`), the [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) attribute is applied to the class.</span></span> <span data-ttu-id="86b4f-145">该属性指示用户必须根据默认策略获得授权才能访问资源。</span><span class="sxs-lookup"><span data-stu-id="86b4f-145">The attribute indicates that the user must be authorized based on the default policy to access the resource.</span></span> <span data-ttu-id="86b4f-146">默认授权策略配置为使用默认身份验证方案，由 <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilderExtensions.AddIdentityServerJwt%2A> 设置。</span><span class="sxs-lookup"><span data-stu-id="86b4f-146">The default authorization policy is configured to use the default authentication scheme, which is set up by <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilderExtensions.AddIdentityServerJwt%2A>.</span></span> <span data-ttu-id="86b4f-147">帮助器方法将 <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerHandler> 配置为对应用的请求的默认处理程序。</span><span class="sxs-lookup"><span data-stu-id="86b4f-147">The helper method configures <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerHandler> as the default handler for requests to the app.</span></span>

### <a name="applicationdbcontext"></a><span data-ttu-id="86b4f-148">ApplicationDbContext</span><span class="sxs-lookup"><span data-stu-id="86b4f-148">ApplicationDbContext</span></span>

<span data-ttu-id="86b4f-149">在 `ApplicationDbContext` (`Data/ApplicationDbContext.cs`) 中，<xref:Microsoft.EntityFrameworkCore.DbContext> 扩展 <xref:Microsoft.AspNetCore.ApiAuthorization.IdentityServer.ApiAuthorizationDbContext%601> 以包括 IdentityServer 的架构。</span><span class="sxs-lookup"><span data-stu-id="86b4f-149">In the `ApplicationDbContext` (`Data/ApplicationDbContext.cs`), <xref:Microsoft.EntityFrameworkCore.DbContext> extends <xref:Microsoft.AspNetCore.ApiAuthorization.IdentityServer.ApiAuthorizationDbContext%601> to include the schema for IdentityServer.</span></span> <span data-ttu-id="86b4f-150"><xref:Microsoft.AspNetCore.ApiAuthorization.IdentityServer.ApiAuthorizationDbContext%601> 派生自 <xref:Microsoft.AspNetCore.Identity.EntityFrameworkCore.IdentityDbContext>。</span><span class="sxs-lookup"><span data-stu-id="86b4f-150"><xref:Microsoft.AspNetCore.ApiAuthorization.IdentityServer.ApiAuthorizationDbContext%601> is derived from <xref:Microsoft.AspNetCore.Identity.EntityFrameworkCore.IdentityDbContext>.</span></span>

<span data-ttu-id="86b4f-151">要获取对数据库架构的完全控制，请从其中一个可用的 Identity <xref:Microsoft.EntityFrameworkCore.DbContext> 类继承，并通过在 <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A> 方法中调用 `builder.ConfigurePersistedGrantContext(_operationalStoreOptions.Value)` 来配置上下文以包括 Identity 架构。</span><span class="sxs-lookup"><span data-stu-id="86b4f-151">To gain full control of the database schema, inherit from one of the available Identity <xref:Microsoft.EntityFrameworkCore.DbContext> classes and configure the context to include the Identity schema by calling `builder.ConfigurePersistedGrantContext(_operationalStoreOptions.Value)` in the <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A> method.</span></span>

### <a name="oidcconfigurationcontroller"></a><span data-ttu-id="86b4f-152">OidcConfigurationController</span><span class="sxs-lookup"><span data-stu-id="86b4f-152">OidcConfigurationController</span></span>

<span data-ttu-id="86b4f-153">在 `OidcConfigurationController` (`Controllers/OidcConfigurationController.cs`) 中，客户端终结点预配为提供 OIDC 参数。</span><span class="sxs-lookup"><span data-stu-id="86b4f-153">In the `OidcConfigurationController` (`Controllers/OidcConfigurationController.cs`), the client endpoint is provisioned to serve OIDC parameters.</span></span>

### <a name="app-settings-files"></a><span data-ttu-id="86b4f-154">应用设置文件</span><span class="sxs-lookup"><span data-stu-id="86b4f-154">App settings files</span></span>

<span data-ttu-id="86b4f-155">在项目根目录的应用设置文件 (`appsettings.json`) 中，`IdentityServer` 部分描述已配置的客户端列表。</span><span class="sxs-lookup"><span data-stu-id="86b4f-155">In the app settings file (`appsettings.json`) at the project root, the `IdentityServer` section describes the list of configured clients.</span></span> <span data-ttu-id="86b4f-156">下例中存在一个客户端。</span><span class="sxs-lookup"><span data-stu-id="86b4f-156">In the following example, there's a single client.</span></span> <span data-ttu-id="86b4f-157">客户端名称对应于应用名称，并通过约定映射到 OAuth `ClientId` 参数。</span><span class="sxs-lookup"><span data-stu-id="86b4f-157">The client name corresponds to the app name and is mapped by convention to the OAuth `ClientId` parameter.</span></span> <span data-ttu-id="86b4f-158">配置文件指示正在配置的应用类型。</span><span class="sxs-lookup"><span data-stu-id="86b4f-158">The profile indicates the app type being configured.</span></span> <span data-ttu-id="86b4f-159">配置文件在内部用于促进简化服务器配置过程的约定。</span><span class="sxs-lookup"><span data-stu-id="86b4f-159">The profile is used internally to drive conventions that simplify the configuration process for the server.</span></span> <!-- There are several profiles available, as explained in the [Application profiles](#application-profiles) section. -->

```json
"IdentityServer": {
  "Clients": {
    "{APP ASSEMBLY}.Client": {
      "Profile": "IdentityServerSPA"
    }
  }
}
```

<span data-ttu-id="86b4f-160">占位符 `{APP ASSEMBLY}` 是应用的程序集名称（例如 `BlazorSample.Client`）。</span><span class="sxs-lookup"><span data-stu-id="86b4f-160">The placeholder `{APP ASSEMBLY}` is the app's assembly name (for example, `BlazorSample.Client`).</span></span>

## <a name="client-app-configuration"></a><span data-ttu-id="86b4f-161">客户端应用配置</span><span class="sxs-lookup"><span data-stu-id="86b4f-161">Client app configuration</span></span>

### <a name="authentication-package"></a><span data-ttu-id="86b4f-162">身份验证包</span><span class="sxs-lookup"><span data-stu-id="86b4f-162">Authentication package</span></span>

<span data-ttu-id="86b4f-163">创建应用以使用个人用户帐户 (`Individual`) 时，该应用会在其项目文件中自动接收 [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication/) 包的包引用。</span><span class="sxs-lookup"><span data-stu-id="86b4f-163">When an app is created to use Individual User Accounts (`Individual`), the app automatically receives a package reference for the [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication/) package in the app's project file.</span></span> <span data-ttu-id="86b4f-164">此包提供了一组基元，可帮助应用验证用户身份并获取令牌以调用受保护的 API。</span><span class="sxs-lookup"><span data-stu-id="86b4f-164">The package provides a set of primitives that help the app authenticate users and obtain tokens to call protected APIs.</span></span>

<span data-ttu-id="86b4f-165">如果向应用添加身份验证，请手动将包添加到应用的项目文件中：</span><span class="sxs-lookup"><span data-stu-id="86b4f-165">If adding authentication to an app, manually add the package to the app's project file:</span></span>

```xml
<PackageReference 
  Include="Microsoft.AspNetCore.Components.WebAssembly.Authentication" 
  Version="3.2.0" />
```

### <a name="api-authorization-support"></a><span data-ttu-id="86b4f-166">API 身份验证支持</span><span class="sxs-lookup"><span data-stu-id="86b4f-166">API authorization support</span></span>

<span data-ttu-id="86b4f-167">使用 [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication/) 包中提供的扩展方法在服务容器中加入用户身份验证支持。</span><span class="sxs-lookup"><span data-stu-id="86b4f-167">The support for authenticating users is plugged into the service container by the extension method provided inside the [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication/) package.</span></span> <span data-ttu-id="86b4f-168">此方法设置应用所需的服务以与现有授权系统交互。</span><span class="sxs-lookup"><span data-stu-id="86b4f-168">This method sets up the services required by the app to interact with the existing authorization system.</span></span>

```csharp
builder.Services.AddApiAuthorization();
```

<span data-ttu-id="86b4f-169">默认情况下，应用的配置按约定从 `_configuration/{client-id}` 加载。</span><span class="sxs-lookup"><span data-stu-id="86b4f-169">By default, configuration for the app is loaded by convention from `_configuration/{client-id}`.</span></span> <span data-ttu-id="86b4f-170">按照约定，客户端 ID 设置为应用的程序集名称。</span><span class="sxs-lookup"><span data-stu-id="86b4f-170">By convention, the client ID is set to the app's assembly name.</span></span> <span data-ttu-id="86b4f-171">可以通过使用选项调用重载来更改此 URL 以指向单独的终结点。</span><span class="sxs-lookup"><span data-stu-id="86b4f-171">This URL can be changed to point to a separate endpoint by calling the overload with options.</span></span>

### <a name="imports-file"></a><span data-ttu-id="86b4f-172">导入文件</span><span class="sxs-lookup"><span data-stu-id="86b4f-172">Imports file</span></span>

[!INCLUDE[](~/includes/blazor-security/imports-file-hosted.md)]

### <a name="index-page"></a><span data-ttu-id="86b4f-173">索引页</span><span class="sxs-lookup"><span data-stu-id="86b4f-173">Index page</span></span>

[!INCLUDE[](~/includes/blazor-security/index-page-authentication.md)]

### <a name="app-component"></a><span data-ttu-id="86b4f-174">应用组件</span><span class="sxs-lookup"><span data-stu-id="86b4f-174">App component</span></span>

[!INCLUDE[](~/includes/blazor-security/app-component.md)]

### <a name="redirecttologin-component"></a><span data-ttu-id="86b4f-175">RedirectToLogin 组件</span><span class="sxs-lookup"><span data-stu-id="86b4f-175">RedirectToLogin component</span></span>

[!INCLUDE[](~/includes/blazor-security/redirecttologin-component.md)]

### <a name="logindisplay-component"></a><span data-ttu-id="86b4f-176">LoginDisplay 组件</span><span class="sxs-lookup"><span data-stu-id="86b4f-176">LoginDisplay component</span></span>

<span data-ttu-id="86b4f-177">`LoginDisplay` 组件 (`Shared/LoginDisplay.razor`) 在 `MainLayout` 组件 (`Shared/MainLayout.razor`) 中呈现并管理以下行为：</span><span class="sxs-lookup"><span data-stu-id="86b4f-177">The `LoginDisplay` component (`Shared/LoginDisplay.razor`) is rendered in the `MainLayout` component (`Shared/MainLayout.razor`) and manages the following behaviors:</span></span>

* <span data-ttu-id="86b4f-178">对于经过身份验证的用户：</span><span class="sxs-lookup"><span data-stu-id="86b4f-178">For authenticated users:</span></span>
  * <span data-ttu-id="86b4f-179">显示当前用户名。</span><span class="sxs-lookup"><span data-stu-id="86b4f-179">Displays the current user name.</span></span>
  * <span data-ttu-id="86b4f-180">提供指向 ASP.NET Core Identity 中的用户配置文件页面的链接。</span><span class="sxs-lookup"><span data-stu-id="86b4f-180">Offers a link to the user profile page in ASP.NET Core Identity.</span></span>
  * <span data-ttu-id="86b4f-181">提供用于注销应用的按钮。</span><span class="sxs-lookup"><span data-stu-id="86b4f-181">Offers a button to log out of the app.</span></span>
* <span data-ttu-id="86b4f-182">对于匿名用户：</span><span class="sxs-lookup"><span data-stu-id="86b4f-182">For anonymous users:</span></span>
  * <span data-ttu-id="86b4f-183">提供用于注册的选项。</span><span class="sxs-lookup"><span data-stu-id="86b4f-183">Offers the option to register.</span></span>
  * <span data-ttu-id="86b4f-184">提供用于登录的选项。</span><span class="sxs-lookup"><span data-stu-id="86b4f-184">Offers the option to log in.</span></span>

```razor
@using Microsoft.AspNetCore.Components.Authorization
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@inject NavigationManager Navigation
@inject SignOutSessionStateManager SignOutManager

<AuthorizeView>
    <Authorized>
        <a href="authentication/profile">Hello, @context.User.Identity.Name!</a>
        <button class="nav-link btn btn-link" @onclick="BeginSignOut">
            Log out
        </button>
    </Authorized>
    <NotAuthorized>
        <a href="authentication/register">Register</a>
        <a href="authentication/login">Log in</a>
    </NotAuthorized>
</AuthorizeView>

@code {
    private async Task BeginSignOut(MouseEventArgs args)
    {
        await SignOutManager.SetSignOutState();
        Navigation.NavigateTo("authentication/logout");
    }
}
```

### <a name="authentication-component"></a><span data-ttu-id="86b4f-185">身份验证组件</span><span class="sxs-lookup"><span data-stu-id="86b4f-185">Authentication component</span></span>

[!INCLUDE[](~/includes/blazor-security/authentication-component.md)]

### <a name="fetchdata-component"></a><span data-ttu-id="86b4f-186">FetchData 组件</span><span class="sxs-lookup"><span data-stu-id="86b4f-186">FetchData component</span></span>

[!INCLUDE[](~/includes/blazor-security/fetchdata-component.md)]

## <a name="run-the-app"></a><span data-ttu-id="86b4f-187">运行应用</span><span class="sxs-lookup"><span data-stu-id="86b4f-187">Run the app</span></span>

<span data-ttu-id="86b4f-188">从服务器项目运行应用。</span><span class="sxs-lookup"><span data-stu-id="86b4f-188">Run the app from the Server project.</span></span> <span data-ttu-id="86b4f-189">使用 Visual Studio 时，请执行以下任一操作：</span><span class="sxs-lookup"><span data-stu-id="86b4f-189">When using Visual Studio, either:</span></span>

* <span data-ttu-id="86b4f-190">在工具栏中将“启动项目”下拉列表设置为“服务器 API 应用”，然后选择“运行”按钮。</span><span class="sxs-lookup"><span data-stu-id="86b4f-190">Set the **Startup Projects** drop down list in the toolbar to the *Server API app* and select the **Run** button.</span></span>
* <span data-ttu-id="86b4f-191">在“解决方案资源管理器”中选择服务器项目，然后选择工具栏中的“运行”按钮，或从“调试”菜单启动应用  。</span><span class="sxs-lookup"><span data-stu-id="86b4f-191">Select the Server project in **Solution Explorer** and select the **Run** button in the toolbar or start the app from the **Debug** menu.</span></span>

## <a name="name-and-role-claim-with-api-authorization"></a><span data-ttu-id="86b4f-192">具有 API 授权的名称和角色声明</span><span class="sxs-lookup"><span data-stu-id="86b4f-192">Name and role claim with API authorization</span></span>

### <a name="custom-user-factory"></a><span data-ttu-id="86b4f-193">自定义用户工厂</span><span class="sxs-lookup"><span data-stu-id="86b4f-193">Custom user factory</span></span>

<span data-ttu-id="86b4f-194">在“客户端”应用中，创建自定义用户工厂。</span><span class="sxs-lookup"><span data-stu-id="86b4f-194">In the Client app, create a custom user factory.</span></span> Identity<span data-ttu-id="86b4f-195"> 服务器在一个 `role` 声明中发送多个角色作为 JSON 数组。</span><span class="sxs-lookup"><span data-stu-id="86b4f-195"> Server sends multiple roles as a JSON array in a single `role` claim.</span></span> <span data-ttu-id="86b4f-196">单个角色在该声明中作为单个字符串值进行发送。</span><span class="sxs-lookup"><span data-stu-id="86b4f-196">A single role is sent as a string value in the claim.</span></span> <span data-ttu-id="86b4f-197">工厂为每个用户的角色创建单个 `role` 声明。</span><span class="sxs-lookup"><span data-stu-id="86b4f-197">The factory creates an individual `role` claim for each of the user's roles.</span></span>

<span data-ttu-id="86b4f-198">`CustomUserFactory.cs`：</span><span class="sxs-lookup"><span data-stu-id="86b4f-198">`CustomUserFactory.cs`:</span></span>

```csharp
using System.Linq;
using System.Security.Claims;
using System.Text.Json;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication.Internal;

public class CustomUserFactory
    : AccountClaimsPrincipalFactory<RemoteUserAccount>
{
    public CustomUserFactory(IAccessTokenProviderAccessor accessor)
        : base(accessor)
    {
    }

    public async override ValueTask<ClaimsPrincipal> CreateUserAsync(
        RemoteUserAccount account,
        RemoteAuthenticationUserOptions options)
    {
        var user = await base.CreateUserAsync(account, options);

        if (user.Identity.IsAuthenticated)
        {
            var identity = (ClaimsIdentity)user.Identity;
            var roleClaims = identity.FindAll(identity.RoleClaimType);

            if (roleClaims != null && roleClaims.Any())
            {
                foreach (var existingClaim in roleClaims)
                {
                    identity.RemoveClaim(existingClaim);
                }

                var rolesElem = account.AdditionalProperties[identity.RoleClaimType];

                if (rolesElem is JsonElement roles)
                {
                    if (roles.ValueKind == JsonValueKind.Array)
                    {
                        foreach (var role in roles.EnumerateArray())
                        {
                            identity.AddClaim(new Claim(options.RoleClaim, role.GetString()));
                        }
                    }
                    else
                    {
                        identity.AddClaim(new Claim(options.RoleClaim, roles.GetString()));
                    }
                }
            }
        }

        return user;
    }
}
```

<span data-ttu-id="86b4f-199">在客户端应用中，在 `Program.Main` (`Program.cs`) 中注册工厂：</span><span class="sxs-lookup"><span data-stu-id="86b4f-199">In the Client app, register the factory in `Program.Main` (`Program.cs`):</span></span>

```csharp
builder.Services.AddApiAuthorization()
    .AddAccountClaimsPrincipalFactory<CustomUserFactory>();
```

<span data-ttu-id="86b4f-200">在服务器应用中，调用 Identity 生成器上的 <xref:Microsoft.AspNetCore.Identity.IdentityBuilder.AddRoles*>，添加与角色相关的服务：</span><span class="sxs-lookup"><span data-stu-id="86b4f-200">In the Server app, call <xref:Microsoft.AspNetCore.Identity.IdentityBuilder.AddRoles*> on the Identity builder, which adds role-related services:</span></span>

```csharp
using Microsoft.AspNetCore.Identity;

...

services.AddDefaultIdentity<ApplicationUser>(options => 
    options.SignIn.RequireConfirmedAccount = true)
    .AddRoles<IdentityRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>();
```

### <a name="configure-identity-server"></a><span data-ttu-id="86b4f-201">配置 Identity 服务器</span><span class="sxs-lookup"><span data-stu-id="86b4f-201">Configure Identity Server</span></span>

<span data-ttu-id="86b4f-202">使用以下方法之一：</span><span class="sxs-lookup"><span data-stu-id="86b4f-202">Use **one** of the following approaches:</span></span>

* [<span data-ttu-id="86b4f-203">API 身份验证选项</span><span class="sxs-lookup"><span data-stu-id="86b4f-203">API authorization options</span></span>](#api-authorization-options)
* [<span data-ttu-id="86b4f-204">配置文件服务</span><span class="sxs-lookup"><span data-stu-id="86b4f-204">Profile Service</span></span>](#profile-service)

#### <a name="api-authorization-options"></a><span data-ttu-id="86b4f-205">API 身份验证选项</span><span class="sxs-lookup"><span data-stu-id="86b4f-205">API authorization options</span></span>

<span data-ttu-id="86b4f-206">在服务器应用中：</span><span class="sxs-lookup"><span data-stu-id="86b4f-206">In the Server app:</span></span>

* <span data-ttu-id="86b4f-207">配置 Identity 服务器，将 `name` 和 `role` 声明放入 ID 令牌和访问令牌中。</span><span class="sxs-lookup"><span data-stu-id="86b4f-207">Configure Identity Server to put the `name` and `role` claims into the ID token and access token.</span></span>
* <span data-ttu-id="86b4f-208">阻止 JWT 令牌处理程序中角色的默认映射。</span><span class="sxs-lookup"><span data-stu-id="86b4f-208">Prevent the default mapping for roles in the JWT token handler.</span></span>

```csharp
using System.IdentityModel.Tokens.Jwt;
using System.Linq;

...

services.AddIdentityServer()
    .AddApiAuthorization<ApplicationUser, ApplicationDbContext>(options => {
        options.IdentityResources["openid"].UserClaims.Add("name");
        options.ApiResources.Single().UserClaims.Add("name");
        options.IdentityResources["openid"].UserClaims.Add("role");
        options.ApiResources.Single().UserClaims.Add("role");
    });

JwtSecurityTokenHandler.DefaultInboundClaimTypeMap.Remove("role");
```

#### <a name="profile-service"></a><span data-ttu-id="86b4f-209">配置文件服务</span><span class="sxs-lookup"><span data-stu-id="86b4f-209">Profile Service</span></span>

<span data-ttu-id="86b4f-210">在服务器应用中，创建 `ProfileService` 实现。</span><span class="sxs-lookup"><span data-stu-id="86b4f-210">In the Server app, create a `ProfileService` implementation.</span></span>

<span data-ttu-id="86b4f-211">`ProfileService.cs`：</span><span class="sxs-lookup"><span data-stu-id="86b4f-211">`ProfileService.cs`:</span></span>

```csharp
using IdentityModel;
using IdentityServer4.Models;
using IdentityServer4.Services;
using System.Threading.Tasks;

public class ProfileService : IProfileService
{
    public ProfileService()
    {
    }

    public Task GetProfileDataAsync(ProfileDataRequestContext context)
    {
        var nameClaim = context.Subject.FindAll(JwtClaimTypes.Name);
        context.IssuedClaims.AddRange(nameClaim);

        var roleClaims = context.Subject.FindAll(JwtClaimTypes.Role);
        context.IssuedClaims.AddRange(roleClaims);

        return Task.CompletedTask;
    }

    public Task IsActiveAsync(IsActiveContext context)
    {
        return Task.CompletedTask;
    }
}
```

<span data-ttu-id="86b4f-212">在服务器应用中，在 `Startup.ConfigureServices` 中注册配置文件服务：</span><span class="sxs-lookup"><span data-stu-id="86b4f-212">In the Server app, register the Profile Service in `Startup.ConfigureServices`:</span></span>

```csharp
using IdentityServer4.Services;

...

services.AddTransient<IProfileService, ProfileService>();
```

### <a name="use-authorization-mechanisms"></a><span data-ttu-id="86b4f-213">使用授权机制</span><span class="sxs-lookup"><span data-stu-id="86b4f-213">Use authorization mechanisms</span></span>

<span data-ttu-id="86b4f-214">在客户端应用中，组件授权方法此时有效。</span><span class="sxs-lookup"><span data-stu-id="86b4f-214">In the Client app, component authorization approaches are functional at this point.</span></span> <span data-ttu-id="86b4f-215">组件中的任何授权机制都可以使用角色来授权用户：</span><span class="sxs-lookup"><span data-stu-id="86b4f-215">Any of the authorization mechanisms in components can use a role to authorize the user:</span></span>

* <span data-ttu-id="86b4f-216">[`AuthorizeView` 组件](xref:blazor/security/index#authorizeview-component)（例如：`<AuthorizeView Roles="admin">`）</span><span class="sxs-lookup"><span data-stu-id="86b4f-216">[`AuthorizeView` component](xref:blazor/security/index#authorizeview-component) (Example: `<AuthorizeView Roles="admin">`)</span></span>
* <span data-ttu-id="86b4f-217">[`[Authorize]` 属性指令](xref:blazor/security/index#authorize-attribute) (<xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute>)（例如 `@attribute [Authorize(Roles = "admin")]`）</span><span class="sxs-lookup"><span data-stu-id="86b4f-217">[`[Authorize]` attribute directive](xref:blazor/security/index#authorize-attribute) (<xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute>) (Example: `@attribute [Authorize(Roles = "admin")]`)</span></span>
* <span data-ttu-id="86b4f-218">[过程逻辑](xref:blazor/security/index#procedural-logic)（例如 `if (user.IsInRole("admin")) { ... }`）</span><span class="sxs-lookup"><span data-stu-id="86b4f-218">[Procedural logic](xref:blazor/security/index#procedural-logic) (Example: `if (user.IsInRole("admin")) { ... }`)</span></span>

  <span data-ttu-id="86b4f-219">支持多个角色测试：</span><span class="sxs-lookup"><span data-stu-id="86b4f-219">Multiple role tests are supported:</span></span>

  ```csharp
  if (user.IsInRole("admin") && user.IsInRole("developer"))
  {
      ...
  }
  ```

<span data-ttu-id="86b4f-220">`User.Identity.Name` 在客户端应用中进行填充，并带有用户的用户名，这通常是他们的登录电子邮件地址。</span><span class="sxs-lookup"><span data-stu-id="86b4f-220">`User.Identity.Name` is populated in the Client app with the user's user name, which is usually their sign-in email address.</span></span>

[!INCLUDE[](~/includes/blazor-security/usermanager-signinmanager.md)]

[!INCLUDE[](~/includes/blazor-security/troubleshoot.md)]

## <a name="additional-resources"></a><span data-ttu-id="86b4f-221">其他资源</span><span class="sxs-lookup"><span data-stu-id="86b4f-221">Additional resources</span></span>

* [<span data-ttu-id="86b4f-222">部署到 Azure 应用服务</span><span class="sxs-lookup"><span data-stu-id="86b4f-222">Deployment to Azure App Service</span></span>](xref:security/authentication/identity/spa#deploy-to-production)
* [<span data-ttu-id="86b4f-223">导入来自 Key Vault 的证书（Azure 文档）</span><span class="sxs-lookup"><span data-stu-id="86b4f-223">Import a certificate from Key Vault (Azure documentation)</span></span>](/azure/app-service/configure-ssl-certificate#import-a-certificate-from-key-vault)
* <xref:blazor/security/webassembly/additional-scenarios>
* [<span data-ttu-id="86b4f-224">使用安全默认客户端的应用中未经身份验证或未经授权的 Web API 请求</span><span class="sxs-lookup"><span data-stu-id="86b4f-224">Unauthenticated or unauthorized web API requests in an app with a secure default client</span></span>](xref:blazor/security/webassembly/additional-scenarios#unauthenticated-or-unauthorized-web-api-requests-in-an-app-with-a-secure-default-client)
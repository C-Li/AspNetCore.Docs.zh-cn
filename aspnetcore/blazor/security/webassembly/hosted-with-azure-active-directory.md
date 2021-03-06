---
title: 使用 Azure Active Directory 保护 ASP.NET Core Blazor WebAssembly 托管应用
author: guardrex
description: ''
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 07/08/2020
no-loc:
- Blazor
- Blazor Server
- Blazor WebAssembly
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: blazor/security/webassembly/hosted-with-azure-active-directory
ms.openlocfilehash: d2732819dd8f18da1f99965bb91e5eb3670ff4db
ms.sourcegitcommit: 384833762c614851db653b841cc09fbc944da463
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/17/2020
ms.locfileid: "86445185"
---
# <a name="secure-an-aspnet-core-blazor-webassembly-hosted-app-with-azure-active-directory"></a>使用 Azure Active Directory 保护 ASP.NET Core Blazor WebAssembly 托管应用

作者：[Javier Calvarro Nelson](https://github.com/javiercn) 和 [Luke Latham](https://github.com/guardrex)

本文介绍如何创建使用 [Azure Active Directory (AAD)](https://azure.microsoft.com/services/active-directory/) 进行身份验证的 [Blazor WebAssembly 托管应用](xref:blazor/hosting-models#blazor-webassembly)。

## <a name="register-apps-in-aad-and-create-solution"></a>在 AAD 中注册应用并创建解决方案

### <a name="create-a-tenant"></a>创建租户

请按照[快速入门：设置租户](/azure/active-directory/develop/quickstart-create-new-tenant)中的指南操作，以在 AAD 中创建租户。

### <a name="register-a-server-api-app"></a>注册服务器 API 应用

请按照[快速入门：向 Microsoft 标识平台注册应用程序](/azure/active-directory/develop/quickstart-register-app)中的指南和后续 Azure AAD 主题操作，以便为服务器 API 应用注册 AAD 应用，然后执行以下操作：

1. 在“Azure Active Directory” > “应用注册”中，选择“新建注册”  。
1. 提供应用的名称（例如 Blazor Server AAD） 。
1. 选择支持的帐户类型。 为此体验选择“仅此组织目录中的帐户”（单个租户）。
1. 在这种情况下，“服务器 API 应用”不需要“重定向 URI”，因此请将下拉列表设置为 Web，并且不输入重定向 URI 。
1. 禁用“权限” > “授予对 openid 和 offline_access 权限的管理员同意”复选框 。
1. 选择“注册”。

记录以下信息：

* “服务器 API 应用”应用程序（客户端）ID（例如 `41451fa7-82d9-4673-8fa5-69eff5a761fd`）
* 目录（租户）ID（例如 `e86c78e2-8bb4-4c41-aefd-918e0565a45e`）
* AAD 主域/发布者域/租户域（例如 `contoso.onmicrosoft.com`）：该域在注册应用的 Azure 门户的“品牌”边栏选项卡中作为“发布者域”提供 。

在“API 权限”中，删除“Microsoft Graph” > “User.Read”权限，因为应用无需登录或用户配置文件访问权限  。

在“公开 API”中：

1. 选择“添加范围”。
1. 选择“保存并继续”。
1. 提供“作用域名称”（例如 `API.Access`）。
1. 提供“管理员同意显示名称”（例如 `Access API`）。
1. 提供“管理员同意说明”（例如 `Allows the app to access server app API endpoints.`）。
1. 确认“状态”设置为“已启用” 。
1. 选择“添加范围”。

记录以下信息：

* 应用 ID URI（例如 `https://contoso.onmicrosoft.com/41451fa7-82d9-4673-8fa5-69eff5a761fd`、`api://41451fa7-82d9-4673-8fa5-69eff5a761fd` 或你提供的自定义值）
* 默认作用域（例如 `API.Access`）

应用 ID URI 可能需要在客户端应用中进行专门的配置，如本主题的后续部分[访问令牌作用域](#access-token-scopes)中所述。

### <a name="register-a-client-app"></a>注册客户端应用

请按照[快速入门：向 Microsoft 标识平台注册应用程序](/azure/active-directory/develop/quickstart-register-app)中的指南和后续 Azure AAD 主题操作，以便为服务器客户端应用注册 AAD 应用，然后执行以下操作：

1. 在“Azure Active Directory” > “应用注册”中，选择“新建注册”  。
1. 提供应用的名称（例如 Blazor 客户端 AAD） 。
1. 选择支持的帐户类型。 为此体验选择“仅此组织目录中的帐户”（单个租户）。
1. 将“重定向 URI”下拉列表设置为“Web”，并提供以下重定向 URI：`https://localhost:{PORT}/authentication/login-callback` 。 在 Kestrel 上运行的应用的默认端口为 5001。 如果应用在不同的 Kestrel 端口上运行，请使用应用的端口。 对于 IIS Express，可以在“调试”面板的服务器应用属性中找到该应用随机生成的端口。 由于此时应用不存在，并且 IIS Express 端口未知，因此请在创建应用后返回到此步骤，然后更新重定向 URI。 [创建应用](#create-the-app)部分中会显示一个注解，以提醒 IIS Express 用户更新重定向 URI。
1. 禁用“权限” > “授予对 openid 和 offline_access 权限的管理员同意”复选框 。
1. 选择“注册”。

记录“客户端应用”应用程序（客户端）ID（例如 `4369008b-21fa-427c-abaa-9b53bf58e538`）。

在“身份验证” > “平台配置” > “Web”  中，执行以下操作：

1. 确认存在 `https://localhost:{PORT}/authentication/login-callback` 的重定向 URI。
1. 对于“隐式授权”，选中“访问令牌”和“ID 令牌”的复选框  。
1. 此体验可接受应用的其余默认值。
1. 选择“保存”按钮。

在“API 权限”中：

1. 确认应用拥有“Microsoft Graph” > “User.Read”权限 。
1. 选择“添加权限”，然后选择“我的 API” 。
1. 从“名称”列（例如 Blazor Server AAD）中选择“服务器 API 应用” 。
1. 打开 API 列表。
1. 启用对 API 的访问（例如 `API.Access`）。
1. 选择“添加权限”。
1. 选择“为 {TENANT NAME} 授予管理员内容”按钮。 请选择“是”以确认。

### <a name="create-the-app"></a>创建应用

在空文件夹中，将以下命令中的占位符替换为前面记录的信息，然后在命令行界面中执行该命令：

```dotnetcli
dotnet new blazorwasm -au SingleOrg --api-client-id "{SERVER API APP CLIENT ID}" --app-id-uri "{SERVER API APP ID URI}" --client-id "{CLIENT APP CLIENT ID}" --default-scope "{DEFAULT SCOPE}" --domain "{TENANT DOMAIN}" -ho -o {APP NAME} --tenant-id "{TENANT ID}"
```

| 占位符                  | Azure 门户中的名称                                     | 示例                                |
| ---------------------------- | ----------------------------------------------------- | -------------------------------------- |
| `{APP NAME}`                 | &mdash;                                               | `BlazorSample`                         |
| `{CLIENT APP CLIENT ID}`     | “客户端应用”的应用程序（客户端）ID          | `4369008b-21fa-427c-abaa-9b53bf58e538` |
| `{DEFAULT SCOPE}`            | 作用域名                                            | `API.Access`                           |
| `{SERVER API APP CLIENT ID}` | “服务器 API 应用”的应用程序（客户端）ID      | `41451fa7-82d9-4673-8fa5-69eff5a761fd` |
| `{SERVER API APP ID URI}`    | 应用程序 ID URI（[请参阅说明](#access-token-scopes)） | `41451fa7-82d9-4673-8fa5-69eff5a761fd` |
| `{TENANT DOMAIN}`            | 主域/发布者域/租户域                       | `contoso.onmicrosoft.com`              |
| `{TENANT ID}`                | 目录（租户）ID                                 | `e86c78e2-8bb4-4c41-aefd-918e0565a45e` |

使用 `-o|--output` 选项指定的输出位置将创建一个项目文件夹（如果该文件夹不存在）并成为应用程序名称的一部分。

> [!NOTE]
> 将应用 ID URI 传递给 `app-id-uri` 选项，但请注意，客户端应用中可能需要更改配置，如[访问令牌作用域](#access-token-scopes)部分中所述。

> [!NOTE]
> 在 Azure 门户中，使用默认设置为在 Kestrel 服务器上运行的应用的端口 5001 配置了客户端应用的“身份验证” > “平台配置” > “Web” > “重定向 URI”   。
>
> 如果“客户端应用”是在随机 IIS Express 端口上运行的，则可以在“调试”面板的“服务器 API 应用”属性中找到该应用的端口 。
>
> 如果端口之前未使用“客户端应用”的已知端口进行配置，请返回到 Azure 门户中“客户端应用”的注册，并使用正确的端口更新重定向 URI 。

## <a name="server-app-configuration"></a>服务器应用配置

本部分涉及解决方案的 `Server` 应用**。

### <a name="authentication-package"></a>身份验证包

[`Microsoft.AspNetCore.Authentication.AzureAD.UI`](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.AzureAD.UI/) 包提供验证和授权对 ASP.NET Core Web API 的调用的支持：

```xml
<PackageReference Include="Microsoft.AspNetCore.Authentication.AzureAD.UI" 
  Version="3.1.4" />
```

### <a name="authentication-service-support"></a>身份验证服务支持

`AddAuthentication` 方法在应用中设置身份验证服务，并将 JWT 持有者处理程序配置为默认身份验证方法。 <xref:Microsoft.AspNetCore.Authentication.AzureADAuthenticationBuilderExtensions.AddAzureADBearer%2A> 方法在 JWT 持有者处理程序中设置验证 Azure Active Directory 发出的令牌所需的特定参数：

```csharp
services.AddAuthentication(AzureADDefaults.BearerAuthenticationScheme)
    .AddAzureADBearer(options => Configuration.Bind("AzureAd", options));
```

<xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication%2A> 和 <xref:Microsoft.AspNetCore.Builder.AuthorizationAppBuilderExtensions.UseAuthorization%2A> 确保：

* 应用尝试对传入请求上的令牌进行分析和验证。
* 尝试在没有适当凭据的情况下访问受保护资源的任何请求都失败。

```csharp
app.UseAuthentication();
app.UseAuthorization();
```

### <a name="useridentityname"></a>User.Identity.Name

默认情况下，服务器应用 API 使用 `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name` 声明类型（例如 `2d64b3da-d9d5-42c6-9352-53d8df33d770@contoso.onmicrosoft.com`）中的值填充 `User.Identity.Name`。

要将应用配置为从 `name` 声明类型接收值，请在 `Startup.ConfigureServices` 中配置 <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerOptions> 的 <xref:Microsoft.IdentityModel.Tokens.TokenValidationParameters.NameClaimType?displayProperty=nameWithType>：

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;

...

services.Configure<JwtBearerOptions>(
    AzureADDefaults.JwtBearerAuthenticationScheme, options =>
    {
        options.TokenValidationParameters.NameClaimType = "name";
    });
```

### <a name="app-settings"></a>应用设置

`appsettings.json` 文件包含用于配置 JWT 持有者处理程序（用于验证访问令牌）的选项：

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "{DOMAIN}",
    "TenantId": "{TENANT ID}",
    "ClientId": "{SERVER API APP CLIENT ID}",
  }
}
```

示例：

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "contoso.onmicrosoft.com",
    "TenantId": "e86c78e2-8bb4-4c41-aefd-918e0565a45e",
    "ClientId": "41451fa7-82d9-4673-8fa5-69eff5a761fd",
  }
}
```

### <a name="weatherforecast-controller"></a>WeatherForecast 控制器

WeatherForecast 控制器 (Controllers/WeatherForecastController.cs) 公开了一个受保护的 API，并将 [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) 属性应用于控制器。 务必了解以下内容：

* 仅此 API 控制器中的 [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) 属性可以保护此 API 免受未经授权的访问。
* Blazor WebAssembly 应用中使用的 [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) 属性仅用作应用的提示，提示应授权用户应用才能正常运行。

```csharp
[Authorize]
[ApiController]
[Route("[controller]")]
public class WeatherForecastController : ControllerBase
{
    [HttpGet]
    public IEnumerable<WeatherForecast> Get()
    {
        ...
    }
}
```

## <a name="client-app-configuration"></a>客户端应用配置

本部分涉及解决方案的 `Client` 应用**。

### <a name="authentication-package"></a>身份验证包

创建应用以使用工作或学校帐户 (`SingleOrg`) 时，应用会自动接收 [Microsoft 身份验证库](/azure/active-directory/develop/msal-overview) ([`Microsoft.Authentication.WebAssembly.Msal`](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal/)) 的包引用。 此包提供了一组基元，可帮助应用验证用户身份并获取令牌以调用受保护的 API。

如果向应用添加身份验证，请手动将包添加到应用的项目文件中：

```xml
<PackageReference Include="Microsoft.Authentication.WebAssembly.Msal" 
  Version="3.2.0" />
```

[`Microsoft.Authentication.WebAssembly.Msal`](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal/) 包会间接将 [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication/) 包传递到应用中。

### <a name="authentication-service-support"></a>身份验证服务支持

添加了对 <xref:System.Net.Http.HttpClient> 实例的支持，这些实例在向服务器项目发出请求时包含访问令牌。

`Program.cs`：

```csharp
builder.Services.AddHttpClient("{APP ASSEMBLY}.ServerAPI", client => 
        client.BaseAddress = new Uri(builder.HostEnvironment.BaseAddress))
    .AddHttpMessageHandler<BaseAddressAuthorizationMessageHandler>();

builder.Services.AddScoped(sp => sp.GetRequiredService<IHttpClientFactory>()
    .CreateClient("{APP ASSEMBLY}.ServerAPI"));
```

占位符 `{APP ASSEMBLY}` 是应用的程序集名称（例如 `BlazorSample.ServerAPI`）。

使用由 [`Microsoft.Authentication.WebAssembly.Msal`](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal/) 包提供的 <xref:Microsoft.Extensions.DependencyInjection.MsalWebAssemblyServiceCollectionExtensions.AddMsalAuthentication%2A> 扩展方法在服务容器中注册对用户进行身份验证的支持。 此方法设置应用与 Identity 提供者 (IP) 交互所需的服务。

`Program.cs`：

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    builder.Configuration.Bind("AzureAd", options.ProviderOptions.Authentication);
    options.ProviderOptions.DefaultAccessTokenScopes.Add("{SCOPE URI}");
});
```

<xref:Microsoft.Extensions.DependencyInjection.MsalWebAssemblyServiceCollectionExtensions.AddMsalAuthentication%2A> 方法接受回叫，以配置验证应用所需的参数。 注册应用时，可以从 Azure 门户 AAD 配置中获取配置应用所需的值。

配置由 `wwwroot/appsettings.json` 文件提供：

```json
{
  "AzureAd": {
    "Authority": "https://login.microsoftonline.com/{TENANT ID}",
    "ClientId": "{CLIENT APP CLIENT ID}",
    "ValidateAuthority": true
  }
}
```

示例：

```json
{
  "AzureAd": {
    "Authority": "https://login.microsoftonline.com/e86c78e2-...-918e0565a45e",
    "ClientId": "4369008b-21fa-427c-abaa-9b53bf58e538",
    "ValidateAuthority": true
  }
}
```

### <a name="access-token-scopes"></a>访问令牌作用域

默认访问令牌作用域表示访问令牌作用域的列表，这些作用域是：

* 默认情况下包含在登录请求中。
* 用于在身份验证后立即预配访问令牌。

根据 Azure Active Directory 规则，所有作用域必须属于同一应用。 根据需要为其他 API 应用添加其他作用域：

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    ...
    options.ProviderOptions.DefaultAccessTokenScopes.Add("{SCOPE URI}");
});
```

[!INCLUDE[](~/includes/blazor-security/azure-scope.md)]

有关详细信息，请参阅“其他方案”一文的以下部分：

* [请求其他访问令牌](xref:blazor/security/webassembly/additional-scenarios#request-additional-access-tokens)
* [将令牌附加到传出请求](xref:blazor/security/webassembly/additional-scenarios#attach-tokens-to-outgoing-requests)


### <a name="imports-file"></a>导入文件

[!INCLUDE[](~/includes/blazor-security/imports-file-hosted.md)]

### <a name="index-page"></a>索引页

[!INCLUDE[](~/includes/blazor-security/index-page-msal.md)]

### <a name="app-component"></a>应用组件

[!INCLUDE[](~/includes/blazor-security/app-component.md)]

### <a name="redirecttologin-component"></a>RedirectToLogin 组件

[!INCLUDE[](~/includes/blazor-security/redirecttologin-component.md)]

### <a name="logindisplay-component"></a>LoginDisplay 组件

[!INCLUDE[](~/includes/blazor-security/logindisplay-component.md)]

### <a name="authentication-component"></a>身份验证组件

[!INCLUDE[](~/includes/blazor-security/authentication-component.md)]

### <a name="fetchdata-component"></a>FetchData 组件

[!INCLUDE[](~/includes/blazor-security/fetchdata-component.md)]

## <a name="run-the-app"></a>运行应用

从服务器项目运行应用。 使用 Visual Studio 时，请执行以下任一操作：

* 在工具栏中将“启动项目”下拉列表设置为“服务器 API 应用”，然后选择“运行”按钮。
* 在“解决方案资源管理器”中选择服务器项目，然后选择工具栏中的“运行”按钮，或从“调试”菜单启动应用  。

<!-- HOLD
[!INCLUDE[](~/includes/blazor-security/usermanager-signinmanager.md)]
-->

[!INCLUDE[](~/includes/blazor-security/troubleshoot.md)]

## <a name="additional-resources"></a>其他资源

* <xref:blazor/security/webassembly/additional-scenarios>
* [使用安全默认客户端的应用中未经身份验证或未经授权的 Web API 请求](xref:blazor/security/webassembly/additional-scenarios#unauthenticated-or-unauthorized-web-api-requests-in-an-app-with-a-secure-default-client)
* <xref:blazor/security/webassembly/aad-groups-roles>
* <xref:security/authentication/azure-active-directory/index>
* [Microsoft 标识平台文档](/azure/active-directory/develop/)

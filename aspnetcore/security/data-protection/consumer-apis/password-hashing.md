---
title: ASP.NET Core 中的哈希密码
author: rick-anderson
description: 了解如何使用 ASP.NET Core 数据保护 Api 为密码进行哈希处理。
ms.author: riande
ms.date: 10/14/2016
no-loc:
- Blazor
- Blazor Server
- Blazor WebAssembly
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: security/data-protection/consumer-apis/password-hashing
ms.openlocfilehash: f7d15cab463972d9c0fff52b645be454865ce2ca
ms.sourcegitcommit: d65a027e78bf0b83727f975235a18863e685d902
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/26/2020
ms.locfileid: "85408950"
---
# <a name="hash-passwords-in-aspnet-core"></a>ASP.NET Core 中的哈希密码

数据保护基本代码包括一个包含加密密钥派生函数的*AspNetCore 密钥派生*。 此包是一个独立的组件，与数据保护系统的其余部分没有任何依赖关系。 它可以完全独立地使用。 为了方便起见，源与数据保护代码库并存。

包当前提供了一个方法 `KeyDerivation.Pbkdf2` ，该方法允许使用[PBKDF2 算法](https://tools.ietf.org/html/rfc2898#section-5.2)对密码进行哈希处理。 此 API 与 .NET Framework 现有的[Rfc2898DeriveBytes 类型](/dotnet/api/system.security.cryptography.rfc2898derivebytes)非常相似，但有三个重要的区别：

1. `KeyDerivation.Pbkdf2`方法支持使用多个 PRFs （目前 `HMACSHA1` `HMACSHA256` 为、和 `HMACSHA512` ），而 `Rfc2898DeriveBytes` 类型仅支持 `HMACSHA1` 。

2. `KeyDerivation.Pbkdf2`方法将检测当前操作系统，并尝试选择最适合的例程实现，在某些情况下提供更好的性能。 （在 Windows 8 上，它提供的吞吐量大约为10倍 `Rfc2898DeriveBytes` 。）

3. `KeyDerivation.Pbkdf2`方法要求调用方指定所有参数（salt、PRF 和迭代计数）。 `Rfc2898DeriveBytes`类型为这些提供默认值。

[!code-csharp[](password-hashing/samples/passwordhasher.cs)]

有关实际用例，请参阅 ASP.NET Core 的类型的[源代码](https://github.com/dotnet/AspNetCore/blob/master/src/Identity/Extensions.Core/src/PasswordHasher.cs) Identity `PasswordHasher` 。

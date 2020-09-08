---
title: Bibliotecas cliente para Azure Key Vault | Microsoft Docs
description: Bibliotecas cliente para Azure Key Vault
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 08/14/2020
ms.author: mbaldwin
ms.openlocfilehash: 34321ac9baf3d3cb43dace33b574cb582c2d72a0
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89378957"
---
# <a name="client-libraries-for-azure-key-vault"></a>Bibliotecas cliente para Azure Key Vault

Las bibliotecas cliente para Azure Key Vault permiten el acceso mediante programación a la funcionalidad de Key Vault con diversos lenguajes, como .NET, Python, Java y JavaScript.

## <a name="client-libraries-per-language-and-object"></a>Bibliotecas cliente por lenguaje y objeto

Cada SDK tiene bibliotecas cliente independientes para secretos, claves y certificados, según la tabla siguiente.

| Idioma | Secretos | Claves | Certificados |
|--|--|--|--|
| .NET | - [Referencia de API](/dotnet/api/azure.security.keyvault.secrets?view=azure-dotnet)<br>- [Paquete NuGet](https://www.nuget.org/packages/Azure.Security.KeyVault.Secrets/)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/keyvault/Azure.Security.KeyVault.Secrets)<br>- [Inicio rápido](../secrets/quick-create-net.md) | - [Referencia de API](/dotnet/api/azure.security.keyvault.keys?view=azure-dotnet)<br>- [Paquete NuGet](https://www.nuget.org/packages/Azure.Security.KeyVault.Keys/)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/keyvault/Azure.Security.KeyVault.Keys) | - [Referencia de API](/dotnet/api/azure.security.keyvault.certificates?view=azure-dotnet)<br>- [Paquete NuGet](https://www.nuget.org/packages/Azure.Security.KeyVault.Certificates/)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/keyvault/Azure.Security.KeyVault.Certificates) |
| Python| - [Referencia de API](/python/api/overview/azure/keyvault-secrets-readme?view=azure-python)<br>- [Paquete PyPi](https://pypi.org/project/azure-keyvault-secrets/)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/keyvault/azure-keyvault-secrets)<br>- [Inicio rápido](../secrets/quick-create-python.md) |- [Referencia de API](/python/api/overview/azure/keyvault-keys-readme?view=azure-python)<br>- [Paquete PyPi](https://pypi.org/project/azure-keyvault-keys/)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/keyvault/azure-keyvault-keys)<br>- [Inicio rápido](../keys/quick-create-python.md) | - [Referencia de API](/python/api/overview/azure/keyvault-certificates-readme?view=azure-python)<br>- [Paquete PyPi](https://pypi.org/project/azure-keyvault-certificates/)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/keyvault/azure-keyvault-certificates)<br>- [Inicio rápido](../certificates/quick-create-python.md) |
| Java | - [Referencia de API](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-security-keyvault-secrets/4.2.0/index.html)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/keyvault/azure-security-keyvault-secrets)<br>- [Inicio rápido](../secrets/quick-create-java.md) |- [Referencia de API](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-security-keyvault-keys/4.2.0/index.html)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/keyvault/azure-security-keyvault-keys) | - [Referencia de API](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-security-keyvault-certificates/4.1.0/index.html)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/keyvault/azure-security-keyvault-certificates) |
| Node.js | - [Referencia de API](/javascript/api/@azure/keyvault-secrets/?view=azure-node-latest)<br>- [Paquete NPM](https://www.npmjs.com/package/@azure/keyvault-secrest)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/keyvault/keyvault-secrets)<br>- [Inicio rápido](../secrets/quick-create-node.md) |- [Referencia de API](/javascript/api/@azure/keyvault-keys/?view=azure-node-latest)<br>- [Paquete NPM](https://www.npmjs.com/package/@azure/keyvault-keys)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/keyvault/keyvault-keys)| - [Referencia de API](/javascript/api/@azure/keyvault-certificates/?view=azure-node-latest)<br>- [Paquete NPM](https://www.npmjs.com/package/@azure/keyvault-certificates)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/keyvault/keyvault-certificates) |

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [guía del desarrollador de Azure Key Vault](developers-guide.md).
- Más información acerca de la [Autenticación en Key Vault](authentication.md).

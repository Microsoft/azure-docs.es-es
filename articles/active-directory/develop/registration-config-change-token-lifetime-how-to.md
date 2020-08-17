---
title: Cambio de los valores predeterminados de vigencia del token para aplicaciones personalizadas de Azure AD
description: Aprenda a actualizar las directivas de vigencia de los tokens para la aplicación que está desarrollando en Azure AD
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 04/08/2019
ms.author: ryanwi
ms.custom: aaddev, seoapril2019
ms.openlocfilehash: 8de6a7aafdd402e4ee75862e69ac60af3af0e041
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88114938"
---
# <a name="how-to-change-the-token-lifetime-defaults-for-a-custom-developed-application"></a>Modificación de los valores predeterminados de vigencia de los tokens en una aplicación personalizada

En este artículo se muestra cómo usar Azure AD PowerShell para establecer una directiva de vigencia del token. Azure AD Premium permite a los desarrolladores de aplicaciones y a los administradores de inquilinos configurar la vigencia de los tokens emitidos para clientes no confidenciales. Las directivas de vigencia de los tokens se establecen para todos los inquilinos o para los recursos a los que se va a acceder.

1. Para establecer una directiva de vigencia de tokens, debe descargar el [módulo Azure AD PowerShell](https://www.powershellgallery.com/packages/AzureADPreview).
1. Ejecute el comando **Connect-AzureAD -Confirm**.

    En esta directiva de ejemplo, se establece la actualización del token con arreglo a un único factor de antigüedad máxima. Cree la directiva: ```New-AzureADPolicy -Definition @('{"TokenLifetimePolicy":{"Version":1, "MaxAgeSingleFactor":"until-revoked"}}') -DisplayName "OrganizationDefaultPolicyScenario" -IsOrganizationDefault $true -Type "TokenLifetimePolicy"```

## <a name="next-steps"></a>Pasos siguientes

* Consulte [Configurable token lifetimes in Azure AD](./active-directory-configurable-token-lifetimes.md) (Duraciones de tokens configurables) para aprender a configurar las duraciones de los tokens generadas por Azure AD, lo que incluye cómo establecer las duraciones de los tokens de todas las aplicaciones de su organización, de una aplicación multiinquilino o de una entidad de servicio concreta de su organización. 
* [Referencia de tokens de Azure AD](./id-tokens.md)
---
title: 'Diferencias de validación por tipos de cuenta admitidos: plataforma de identidad de Microsoft | Azure'
description: Obtenga información sobre las diferencias de validación de diversas propiedades de diferentes tipos de cuenta compatibles al registrar la aplicación con la plataforma de Microsoft Identity.
author: SureshJa
ms.author: sureshja
manager: CelesteDG
ms.date: 07/21/2020
ms.topic: conceptual
ms.subservice: develop
ms.custom: aaddev
ms.service: active-directory
ms.reviewer: lenalepa, manrath
ms.openlocfilehash: e794e277f6731c7b6e57a4710eea437f65be0340
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87336351"
---
# <a name="validation-differences-by-supported-account-types-signinaudience"></a>Diferencias de validación por tipos de cuenta compatibles (signInAudience)

Al registrar una aplicación con la plataforma de Microsoft Identity para desarrolladores, se le pedirá que seleccione los tipos de cuenta que admite la aplicación. En el objeto de aplicación y el manifiesto, esta propiedad es `signInAudience`.

Están disponibles las siguientes opciones:

- *AzureADMyOrg*: solo las cuentas del directorio de la organización donde está registrada la aplicación (un solo inquilino)
- *AzureADMultipleOrgs*: Cuentas en cualquier directorio organizativo (multiinquilino)
- *AzureADandPersonalMicrosoftAccount*: cuentas en cualquier directorio organizativo (multiinquilino) y cuentas de Microsoft personales (como Skype, Xbox o Outlook.com)

En el caso de las aplicaciones registradas, puede encontrar el valor de los tipos de cuenta compatibles en la sección **Authentication** (Autenticación) de una aplicación. También puede encontrarlo en la propiedad `signInAudience` del **manifiesto**.

El valor seleccionado para esta propiedad tiene implicaciones en otras propiedades del objeto de aplicación. Como resultado, si cambia esta propiedad, puede que tenga que cambiar primero otras propiedades.

Vea la tabla siguiente para conocer las diferencias de validación de diversas propiedades de diferentes tipos de cuenta admitidos.

| Propiedad | `AzureADMyOrg` | `AzureADMultipleOrgs` | `AzureADandPersonalMicrosoftAccount` y `PersonalMicrosoftAccount` |
|--------------|---------------|----------------|----------------|
| URI de identificador de aplicación (`identifierURIs`)  | Debe ser único en el inquilino <br><br> Se admiten los esquemas urn:// <br><br> No se admiten caracteres comodín <br><br> Se admiten cadenas de consulta y fragmentos <br><br> Tiene una longitud máxima de 255 caracteres <br><br> Sin límite* en el número de identifierURI  | Debe ser único globalmente. <br><br> Se admiten los esquemas urn:// <br><br> No se admiten caracteres comodín <br><br> Se admiten cadenas de consulta y fragmentos <br><br> Tiene una longitud máxima de 255 caracteres <br><br> Sin límite* en el número de identifierURI | Debe ser único globalmente. <br><br> No se admiten los esquemas urn:// <br><br> No se admiten caracteres comodín, fragmentos y cadenas de consulta <br><br> Tiene una longitud máxima de 120 caracteres <br><br> Máximo de 50 identifierURI |
| Certificados (`keyCredentials`) | Clave de firma simétrica | Clave de firma simétrica | Cifrado y clave de firma asimétrica | 
| Secretos de cliente (`passwordCredentials`) | Ilimitado* | Ilimitado* | Si liveSDK está habilitado: un máximo 2 secretos de cliente | 
| URI de redirección (`replyURLs`) | Consulte [Limitaciones y restricciones de URI de redireccionamiento o URL de respuesta](reply-url.md). | | | 
| Permisos de API (`requiredResourceAccess`) | Ilimitado* | Ilimitado* | Máximo de 50 recursos por aplicación y 30 permisos por recurso (por ejemplo, Microsoft Graph). Límite total de 200 por aplicación (recursos x permisos). | 
| Ámbitos definidos con esta API (`oauth2Permissions`) | Longitud máxima de nombre de ámbito de 120 caracteres <br><br> No existe un límite* en el número de ámbitos definidos | Longitud máxima de nombre de ámbito de 120 caracteres <br><br> No existe un límite* en el número de ámbitos definidos |  Longitud máxima del nombre de ámbito de 40 caracteres <br><br> Máximo de 100 ámbitos definidos | 
| Aplicaciones cliente autorizadas (`preAuthorizedApplications`) | Ilimitado* | Ilimitado* | Máximo total de 500 <br><br> Máximo de 100 aplicaciones cliente definidas <br><br> Máximo de 30 ámbitos definidos por cliente | 
| appRoles | Compatible <br> Ilimitado* | Compatible <br> Ilimitado* | No compatible | 
| URL de cierre de sesión | http://localhost se permite <br><br> Tiene una longitud máxima de 255 caracteres | http://localhost se permite <br><br> Tiene una longitud máxima de 255 caracteres | <br><br> https://localhost se permite, http://localhost da error para MSA <br><br> Tiene una longitud máxima de 255 caracteres <br><br> El esquema HTTP no se admite <br><br> No se admiten caracteres comodín | 

*Hay un límite global de aproximadamente 1000 elementos en todas las propiedades de colección del objeto de aplicación

## <a name="next-steps"></a>Pasos siguientes

- Información sobre el [registro de aplicación](app-objects-and-service-principals.md)
- Obtenga más información sobre el [manifiesto de aplicación](reference-app-manifest.md).

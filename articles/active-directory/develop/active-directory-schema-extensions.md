---
title: Uso de atributos de extensión de esquema de Azure AD en notificaciones
titleSuffix: Microsoft identity platform
description: Se describe cómo usar los atributos de extensión de esquema de directorio para enviar datos de usuario a aplicaciones en notificaciones de token.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.custom: aaddev
ms.workload: identity
ms.topic: how-to
ms.date: 07/29/2020
ms.author: ryanwi
ms.reviewer: paulgarn, hirsin, jeedes, luleon
ms.openlocfilehash: 4450b0bcc06b048fd9ad42d2a7bf1c588816eae7
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88115618"
---
# <a name="using-directory-schema-extension-attributes-in-claims"></a>Uso de atributos de extensión de esquema de directorio en notificaciones

Los atributos de extensión de esquema de directorio proporcionan una manera de almacenar datos adicionales en Azure Active Directory en objetos de usuario y otros objetos de directorio, como grupos, detalles de inquilinos y entidades de servicio.  Solo se pueden utilizar atributos de extensión en objetos de usuario para emitir notificaciones a las aplicaciones. En este artículo se describe cómo usar atributos de extensión de esquema de directorio para enviar datos de usuario a aplicaciones en notificaciones de token.

> [!NOTE]
> Microsoft Graph proporciona otros dos mecanismos de extensión para personalizar los objetos de Graph. Se conocen como extensiones abiertas de Microsoft Graph y extensiones de esquema de Microsoft Graph. Vea la [documentación de Microsoft Graph](/graph/extensibility-overview) para obtener información detallada. Los datos almacenados en objetos de Microsoft Graph que usan estas funcionalidades no están disponibles como orígenes de las notificaciones en los tokens.

Los atributos de extensión de esquema de directorio siempre se asocian a una aplicación en el inquilino y se hace referencia a ellos con el elemento *applicationId* de la aplicación en su nombre.

El identificador de un atributo de extensión de esquema de directorio tiene el formato *Extension_xxxxxxxxx_AttributeName*.  Donde *xxxxxxxxx* es el elemento *applicationId* de la aplicación para la que se definió la extensión.

## <a name="registering-and-using-directory-schema-extensions"></a>Registro y uso de extensiones de esquema de directorio
Los atributos de extensión de esquema de directorio se pueden registrar y rellenar de alguna de estas dos maneras:

- Configurando AD Connect para crearlos y sincronizar los datos en ellos desde una instancia de AD local. Vea [Sincronización de Azure AD Connect: Extensiones de directorio](../hybrid/how-to-connect-sync-feature-directory-extensions.md).
- Usando Microsoft Graph para registrar, establecer los valores y leer los atributos de extensión de esquema de directorio; [Extensiones de esquema de directorio |Conceptos de Graph API](/previous-versions/azure/ad/graph/howto/azure-ad-graph-api-directory-schema-extensions) o PowerShell+ [Administración de atributos de extensión con cmdlets de AzureAD PowerShell](/powershell/azure/active-directory/using-extension-attributes-sample?view=azureadps-2.0).

### <a name="emitting-claims-with-data-from-directory-schema-extension-attributes-created-with-ad-connect"></a>Emisión de notificaciones con datos de los atributos de extensión de esquema de directorio creados con AD Connect
Los atributos de extensión de esquema de directorio creados y sincronizados con AD Connect siempre están asociados al identificador de aplicación que usa AD Connect. Se pueden usar como origen de las notificaciones mediante la configuración como notificaciones en la opción **Aplicaciones empresariales** en la interfaz de usuario del portal para las aplicaciones SAML registradas mediante la galería o la experiencia de configuración de la aplicación que no es de la galería en **Aplicaciones empresariales** y mediante una directiva de asignación de notificaciones para aplicaciones registradas a través de la experiencia de registro de aplicación.  Una vez que un atributo de extensión de directorio creado mediante AD Connect está en el directorio, se mostrará en la interfaz de usuario de configuración de notificaciones de SSO de SAML.

### <a name="emitting-claims-with-data-from-directory-schema-extension-attributes-created-for-an-application-using-graph-or-powershell"></a>Emisión de notificaciones con datos de los atributos de extensión de esquema de directorio creados para una aplicación mediante Graph o PowerShell
Si un atributo de extensión de esquema de directorio se registra para una aplicación mediante Microsoft Graph o PowerShell (con un paso de aprovisionamiento o configuración inicial de aplicaciones por instancia), la misma aplicación se puede configurar en Azure Active Directory para recibir datos en ese atributo desde un objeto de usuario en una notificación cuando el usuario inicia sesión.  La aplicación se puede configurar para recibir datos en las extensiones de esquema de directorio que están registradas en esa misma aplicación mediante [notificaciones opcionales](active-directory-optional-claims.md#configuring-directory-extension-optional-claims).  Se pueden establecer en el manifiesto de aplicación.  Esto permite a una aplicación multiinquilino registrar atributos de extensión de esquema de directorio para su propio uso. Cuando la aplicación se aprovisiona en un inquilino, las extensiones de esquema de directorio asociadas están disponibles para que se puedan establecer en los usuarios de ese inquilino y para que se puedan utilizar.  Una vez realizada la configuración en el inquilino y concedido el consentimiento, se puede usar para almacenar y recuperar datos a través de Graph y crear una asignación con las notificaciones en los tokens que la Plataforma de identidad de Microsoft emite a las aplicaciones.

Los atributos de extensión de esquema de directorio se pueden registrar y rellenar para cualquier aplicación.

Si una aplicación necesita enviar notificaciones con datos de un atributo de extensión registrado en otra aplicación, se debe usar una [directiva de asignación de notificaciones](active-directory-claims-mapping.md) para asignar el atributo de extensión a la notificación.  Un patrón común para administrar los atributos de extensión de esquema de directorio es crear una aplicación específicamente para que sea el punto de registro de todas las extensiones de esquema que necesita.  No tiene que ser una aplicación real, y esta técnica significa que todas las extensiones tienen el mismo identificador de aplicación en su nombre.

Por ejemplo, esta es una directiva de asignación de notificaciones para emitir una notificación única desde un atributo de extensión de esquema de directorio en un token de OAuth/OIDC:

```json
{
    "ClaimsMappingPolicy": {
        "Version": 1,
        "IncludeBasicClaimSet": "false",
        "ClaimsSchema": [{
                "Source": "User",
                "ExtensionID": "extension_xxxxxxx_test",
                "JWTClaimType": "http://schemas.contoso.com/identity/claims/exampleclaim"
            }, 
        ]
    }
}
```

Donde *xxxxxxx* es el identificador de aplicación con el que se registró la extensión.

> [!TIP]
> La coherencia entre mayúsculas y minúsculas es importante al establecer los atributos de extensión de directorio en los objetos.  Los nombres de los atributos de extensión no distinguen mayúsculas y minúsculas cuando se configuran, pero sí realizan esta distinción cuando el servicio de token los lee desde el directorio.  Si se establece un atributo de extensión en un objeto de usuario con el nombre "LegacyId" y en otro objeto de usuario con el nombre "legacyid", cuando el atributo se asigna a una notificación con el nombre "LegacyId", los datos se recuperarán correctamente, y la notificación se incluirá en el token del primer usuario, pero no del segundo.
>
> El parámetro "Id" del esquema de notificaciones que se usa para los atributos de directorio integrados es "ExtensionID" para los atributos de extensión de directorio.

## <a name="next-steps"></a>Pasos siguientes
- Obtenga información sobre cómo [agregar notificaciones personalizadas o adicionales a los tokens SAML 2.0 y JSON Web Token (JWT)](active-directory-optional-claims.md). 
- Obtenga información sobre cómo [personalizar las notificaciones emitidas en tokens para una aplicación específica](active-directory-claims-mapping.md).
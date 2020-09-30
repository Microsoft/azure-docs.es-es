---
title: 'Solución de problemas sobre la verificación del publicador: plataforma de identidad de Microsoft | Azure'
description: Describe cómo solucionar los problemas de verificación del editor para la plataforma de identidad de Microsoft mediante una llamada a las instancias de Microsoft Graph API.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: troubleshooting
ms.workload: identity
ms.date: 05/08/2020
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: jesakowi
ms.openlocfilehash: fd49e922e5952f5a7c4b7f477dd33d6518010428
ms.sourcegitcommit: 07166a1ff8bd23f5e1c49d4fd12badbca5ebd19c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90088330"
---
# <a name="troubleshoot-publisher-verification"></a>Solución de problemas de verificación del editor
Si no puede completar el proceso o experimenta u observa un comportamiento inesperado con la [verificación del editor](publisher-verification-overview.md), o si recibe errores, debe empezar por hacer lo siguiente: 

1. Revise los [requisitos](publisher-verification-overview.md#requirements) y asegúrese de que se cumplen todos.

1. Revise las instrucciones para [marcar una aplicación como comprobada por el publicador](mark-app-as-publisher-verified.md) y asegúrese de que todos los pasos se han realizado correctamente.

1. Revise la lista de [problemas comunes](#common-issues).

1. Reproduzca la solicitud mediante [Explorador de gráficos](#making-microsoft-graph-api-calls) para recopilar información adicional y descartar cualquier problema en la interfaz de usuario.

## <a name="common-issues"></a>Problemas comunes
A continuación se muestran algunos problemas comunes que pueden producirse durante el proceso. 

- **Desconozco mi identificador de Microsoft Partner Network (id. de MPN) o no sé quién es el contacto principal de la cuenta** 
    1. Vaya a la [página de inscripción de MPN](https://partner.microsoft.com/dashboard/account/v3/enrollment/joinnow/basicpartnernetwork/new).
    1. Inicie sesión con una cuenta de usuario en el inquilino de Azure AD principal de la organización. 
    1. Si ya existe una cuenta de MPN, se reconocerá y se le agregará a la cuenta. 
    1. Vaya a la [página de perfil de asociado](https://partner.microsoft.com/pcv/accountsettings/connectedpartnerprofile), donde se mostrarán el id. de MPN y el contacto de la cuenta principal.

- **No sé quién es mi administrador global de Azure AD (también conocido como administrador de la empresa o administrador de inquilinos). ¿Cómo puedo encontrarlo? ¿Y el administrador de aplicaciones o el administrador de aplicaciones en la nube?**
    1. Inicie sesión en el [portal de Azure AD](https://aad.portal.azure.com) con una cuenta de usuario en el inquilino principal de su organización.
    1. Vaya a [Administración de roles](https://aad.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RolesAndAdministrators).
    1. Haga clic en el rol de administrador deseado.
    1. Se mostrará la lista de usuarios que tienen ese rol asignado.

- **No sé quiénes son los administradores de mi cuenta de MPN**. Vaya a la [página Administración de usuarios de MPN](https://partner.microsoft.com/pcv/users) y filtre la lista de usuarios para ver qué usuarios tienen asignados los distintos roles de administrador.

- **Obtengo un error que indica que mi id. de MPN no es válido o que no tengo acceso a él.**
    1. Vaya a su [perfil de asociado](https://partner.microsoft.com/pcv/accountsettings/connectedpartnerprofile) y compruebe lo siguiente: 
        - El id. de MPN es correcto. 
        - No se muestran errores ni "acciones pendientes", y el estado de verificación en el perfil de negocio legal y en la información del asociado indica "autorizado" o "correcto".
    1. Vaya a la [página de administración de inquilinos de MPN](https://partner.microsoft.com/dashboard/account/v3/tenantmanagement) y confirme que el inquilino en el que está registrada la aplicación y en que el está iniciando sesión con una cuenta de usuario está en la lista de inquilinos asociados. Si necesita agregar un inquilino adicional, siga las instrucciones que [aquí](https://docs.microsoft.com/partner-center/multi-tenant-account) se indican. Tenga en cuenta que a todos los administradores globales de cualquier inquilino que agregue se les concederán privilegios de administrador global en la cuenta del Centro de partners.
    1. Vaya a la [página de Administración de usuarios de MPN](https://partner.microsoft.com/pcv/users) y confirme que el usuario con el que inicia sesión es un administrador global, un administrador de MPN o un administrador de cuentas. Si necesita agregar un usuario a un rol en el Centro de partners, siga las instrucciones que [aquí](https://docs.microsoft.com/partner-center/create-user-accounts-and-set-permissions) se indican.

- **Al iniciar sesión en el portal de Azure AD, no veo ninguna aplicación registrada. ¿Por qué?** 
    Es posible que los registros de aplicaciones se hayan creado con otra cuenta de usuario en este inquilino, una cuenta personal o de consumidor, o en otro inquilino. Asegúrese de que ha iniciado sesión con la cuenta correcta en el inquilino donde se crearon los registros de la aplicación.

- **Recibo un error relacionado con la autenticación multifactor. ¿qué debo hacer?** 
    Asegúrese de que la [autenticación multifactor](../fundamentals/concept-fundamentals-mfa-get-started.md) está habilitada y que se le exige al usuario con el que inicia sesión en este escenario. Por ejemplo, MFA podría ser:
    - Siempre obligatorio para el usuario con el que inicia sesión
    - [Obligatorio para la administración de Azure](../conditional-access/howto-conditional-access-policy-azure-management.md).
    - [Obligatorio para el tipo de administrador](../conditional-access/howto-conditional-access-policy-admin-mfa.md) con el que inicia sesión.

## <a name="making-microsoft-graph-api-calls"></a>Realización de llamadas a Microsoft Graph API 

Si tiene algún problema, pero no comprende por qué según lo que ve en la interfaz de usuario, puede resultar útil realizar más tareas de solución de problemas mediante llamadas a Microsoft Graph para realizar las mismas operaciones que puede realizar en el portal de registro de aplicaciones.

La manera más sencilla de realizar estas solicitudes es usar [Explorador de gráficos](https://developer.microsoft.com/graph/graph-explorer). También puede considerar otras opciones, como el uso de [Postman](https://www.postman.com/), o bien el uso de PowerShell para [invocar una solicitud web](/powershell/module/microsoft.powershell.utility/invoke-webrequest).  

Puede usar Microsoft Graph para establecer y anular el publicador verificado de la aplicación y comprobar el resultado después de realizar una de estas operaciones. El resultado puede verse en el objeto de la [aplicación](/graph/api/resources/application) correspondiente al registro de la aplicación y en cualquier [entidad de servicio](/graph/api/resources/serviceprincipal) de la que se haya creado una instancia desde esa aplicación. Para obtener más información sobre la relación entre esos objetos, consulte: [Objetos de aplicación y de entidad de servicio de Azure Active Directory](app-objects-and-service-principals.md).  

Estos son algunos ejemplos de solicitudes útiles:  

### <a name="set-verified-publisher"></a>Establecer el publicador verificado 

Solicitud

```
POST /applications/0cd04273-0d11-4e62-9eb3-5c3971a7cbec/setVerifiedPublisher 

{ 

    "verifiedPublisherId": "12345678" 

} 
```
 
Response 
```
204 No Content 
```
> [!NOTE]
> *verifiedPublisherID* es el id. de MPN. 

### <a name="unset-verified-publisher"></a>Anular el publicador verificado 

Solicitud:  
```
POST /applications/0cd04273-0d11-4e62-9eb3-5c3971a7cbec/unsetVerifiedPublisher 
```
 
Response 
```
204 No Content 
```
### <a name="get-verified-publisher-info-from-application"></a>Obtener información del publicador verificado de la aplicación 
 
```
GET https://graph.microsoft.com/v1.0/applications/0cd04273-0d11-4e62-9eb3-5c3971a7cbec 

HTTP/1.1 200 OK 

{ 
    "id": "0cd04273-0d11-4e62-9eb3-5c3971a7cbec", 

    ... 

    "verifiedPublisher" : { 
        "displayName": "myexamplePublisher", 
        "verifiedPublisherId": "12345678", 
        "addedDateTime": "2019-12-10T00:00:00" 
    } 
} 
```

### <a name="get-verified-publisher-info-from-service-principal"></a>Obtener información del publicador verificado de la entidad de servicio 
```
GET https://graph.microsoft.com/v1.0/servicePrincipals/010422a7-4d77-4f40-9335-b81ef5c23dd4 

HTTP/1.1 200 OK 

{ 
    "id": "010422a7-4d77-4f40-9335-b81ef5c22dd4", 

    ... 

    "verifiedPublisher" : { 
        "displayName": "myexamplePublisher", 
        "verifiedPublisherId": "12345678", 
        "addedDateTime": "2019-12-10T00:00:00" 
    } 
} 
```

## <a name="error-reference"></a>Referencia de error 

A continuación se muestra una lista de los posibles códigos de error que puede recibir, ya sea al solucionar problemas con Microsoft Graph o al realizar el proceso en el portal de registro de aplicaciones.

### <a name="mpnaccountnotfoundornoaccess"></a>MPNAccountNotFoundOrNoAccess     

El id. de MPN proporcionado (<MPNID>) no existe o no tiene acceso a él. Proporcione una id. de MPN válido e inténtelo de nuevo. 

### <a name="mpnglobalaccountnotfound"></a>MPNGlobalAccountNotFound     

El id. de MPN proporcionado (<MPNID>) no es válido. Proporcione una id. de MPN válido e inténtelo de nuevo. 

### <a name="mpnaccountinvalid"></a>MPNAccountInvalid    

El id. de MPN proporcionado (<MPNID>) no es válido. Proporcione una id. de MPN válido e inténtelo de nuevo. 

### <a name="mpnaccountnotvetted"></a>MPNAccountNotVetted  

El id. de MPN (<MPNID>) proporcionado no completó el proceso de investigación. Complete este proceso en el Centro de partners e inténtelo de nuevo. 

### <a name="nopublisheridonassociatedmpnaccount"></a>NoPublisherIdOnAssociatedMPNAccount  

El id. de MPN proporcionado (<MPNID>) no es válido. Proporcione una id. de MPN válido e inténtelo de nuevo. 

### <a name="mpniddoesnotmatchassociatedmpnaccount"></a>MPNIdDoesNotMatchAssociatedMPNAccount    

El id. de MPN proporcionado (<MPNID>) no es válido. Proporcione una id. de MPN válido e inténtelo de nuevo. 

### <a name="applicationnotfound"></a>ApplicationNotFound  

No se encuentra la aplicación de destino (<AppId>). Proporcione un identificador de aplicación válido e inténtelo de nuevo. 

### <a name="b2ctenantnotallowed"></a>B2CTenantNotAllowed  

Esta funcionalidad no se admite en el inquilino de Azure AD B2C. 

### <a name="emailverifiedtenantnotallowed"></a>EmailVerifiedTenantNotAllowed    

Esta funcionalidad no se admite en un inquilino verificado por correo electrónico. 

### <a name="nopublisherdomainonapplication"></a>NoPublisherDomainOnApplication   

La aplicación de destino (\<AppId\>) debe tener un dominio de editor establecido. Establezca un dominio de publicador e inténtelo de nuevo.

### <a name="publisherdomainmismatch"></a>PublisherDomainMismatch  

El dominio del editor de la aplicación de destino (<publisherDomain>) no coincide con el dominio empleado para realizar la verificación por correo electrónico en el Centro de partners (<pcDomain>). Asegúrese de que dichos dominios coincidan y vuelva a intentarlo. 

### <a name="notauthorizedtoverifypublisher"></a>NotAuthorizedToVerifyPublisher   

No tiene autorización para establecer la propiedad del publicador verificado en la aplicación (<AppId>). 

### <a name="mpnidwasnotprovided"></a>MPNIdWasNotProvided  

El id. de MPN no se proporcionó en el cuerpo de la solicitud o el tipo de contenido de la solicitud no era "application/json". 

### <a name="msanotsupported"></a>MSANotSupported  

Esta característica no se admite para las cuentas de consumidor de Microsoft. Solo se admiten las aplicaciones que un usuario de Azure AD haya registrado en Azure AD. 

## <a name="next-steps"></a>Pasos siguientes

Si ha revisado toda la información anterior y sigue recibiendo un error de Microsoft Graph, reúna toda la información siguiente que pueda en relación con la solicitud que genera el error y [póngase en contacto con el soporte técnico de Microsoft](developer-support-help-options.md#open-a-support-request).

- Timestamp 
- CorrelationId 
- ObjectID o UserPrincipalName del usuario que tiene la sesión iniciada 
- Valor ObjectId de la aplicación de destino
- Valor AppId de la aplicación de destino
- Valor TenantId en el que está registrada la aplicación
- Identificador de MPN
- Solicitud REST que se está realizando 
- Código de error y mensaje devuelto 

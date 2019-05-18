---
title: 'B2B API de colaboración y personalización: Azure Active Directory | Microsoft Docs'
description: La colaboración con Azure Active Directory B2B posibilita las relaciones entre empresas al permitir que los asociados empresariales accedan de forma selectiva a las aplicaciones corporativas.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: reference
ms.date: 04/11/2017
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: elisolMS
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0369988bc6f6503f9940e6aabccb91ab843d63f5
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65811883"
---
# <a name="azure-active-directory-b2b-collaboration-api-and-customization"></a>Personalización y API de colaboración B2B de Active Directory Azure

Hemos tenido muchos clientes que nos han dicho que querían personalizar el proceso de invitación de una forma que se adapte menor a sus organizaciones. Con nuestra API, pueden hacer justamente eso. [https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/invitation](https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/invitation)

## <a name="capabilities-of-the-invitation-api"></a>Funcionalidades de la API de invitación

La API ofrece las siguientes funcionalidades:

1. Invite a un usuario externo con *cualquier* dirección de correo electrónico.

    ```
    "invitedUserDisplayName": "Sam"
    "invitedUserEmailAddress": "gsamoogle@gmail.com"
    ```

2. Personalice a qué página desea que lleguen los usuarios finales después de aceptar la invitación.

    ```
    "inviteRedirectUrl": "https://myapps.microsoft.com/"
    ```

3. Seleccione enviar el correo electrónico de invitación estándar por medio de nosotros

    ```
    "sendInvitationMessage": true
    ```

   con un mensaje al destinatario que pueda personalizar.

    ```
    "customizedMessageBody": "Hello Sam, let's collaborate!"
    ```

4. Luego, ponga en copia a los usuarios a los que informar de que ha invitado a este colaborador.

5. También puede personalizar completamente su invitación y el flujo de trabajo de incorporación decidiendo no enviar notificaciones a través de Azure AD.

    ```
    "sendInvitationMessage": false
    ```

   En este caso, obtendrá una URL de canje a través de la API, que puede incrustar en una plantilla de correo electrónico, mensaje instantáneo u otro método de distribución que prefiera.

6. Finalmente, si es administrador, puede invitar al usuario como miembro.

    ```
    "invitedUserType": "Member"
    ```


## <a name="authorization-model"></a>Modelo de autorización

La API se puede ejecutar en los siguientes modos de autorización:

### <a name="app--user-mode"></a>Aplicación y modo de usuario

En este modo, el usuario que usa la API debe tener los permisos necesarios para crear invitaciones de B2B.

### <a name="app-only-mode"></a>Modo de solo aplicación

En el contexto de solo aplicación, la aplicación necesita el ámbito User.Invite.All para que la invitación se realice correctamente.

Para más información, consulte: https://developer.microsoft.com/graph/docs/authorization/permission_scopes


## <a name="powershell"></a>PowerShell

Puede usar PowerShell para agregar e invitar a usuarios externos a una organización fácilmente. Cree una nueva invitación mediante el cmdlet:

```powershell
New-AzureADMSInvitation
```

Puede usar las siguientes opciones:

* -InvitedUserDisplayName
* -InvitedUserEmailAddress
* -SendInvitationMessage
* -InvitedUserMessageInfo

### <a name="invitation-status"></a>Estado de la invitación

Después de enviar una invitación a un usuario externo, puede usar el cmdlet **Get-AzureADUser** para ver si ya la ha aceptado. Cuando se envía una invitación a un usuario externo, se rellenan las propiedades siguientes de Get-AzureADUser:

* **UserState** indica si la invitación está en el estado **PendingAcceptance** o **Accepted**.
* **UserStateChangedOn** muestra la marca de tiempo del cambio más reciente de la propiedad **UserState**.

Puede usar la opción **Filter** para filtrar los resultados por **UserState**. En el ejemplo siguiente se muestra cómo filtrar resultados para mostrar solo a los usuarios que tienen una invitación pendiente. En el ejemplo también se muestra la opción **Format-List**, que le permite especificar las propiedades que se van a mostrar. 
 

```powershell
Get-AzureADUser -Filter "UserState eq 'PendingAcceptance'" | Format-List -Property DisplayName,UserPrincipalName,UserState,UserStateChangedOn
```

> [!NOTE]
> Asegúrese de que tiene la versión más reciente del módulo de AzureAD PowerShell o del módulo AzureADPreview PowerShell. 

## <a name="see-also"></a>Vea también

Consulte la referencia de API de invitación en [https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/invitation](https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/invitation).

## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es la colaboración B2B de Azure AD?](what-is-b2b.md)
- [Los elementos del correo electrónico de invitación de colaboración B2B](invitation-email-elements.md)
- [Canje de invitación de colaboración B2B](redemption-experience.md)
- [Incorporación de usuarios de colaboración B2B sin invitación](add-user-without-invite.md)
---
author: baanders
description: archivo de inclusión para posibles requisitos adicionales en la configuración de Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 7/22/2020
ms.author: baanders
ms.openlocfilehash: c2668bdda5002ebd2a34b8a2ffa5885263aec0c9
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88009679"
---
Es posible que la organización requiera que los propietarios o administradores de la suscripción realicen pasos adicionales para configurar correctamente un registro de aplicaciones (y, por lo tanto, para terminar de configurar una instancia utilizable de Azure Digital Twins). Los pasos necesarios pueden variar en función de la configuración específica de su organización.

A continuación, se muestran algunas posibles actividades comunes que un propietario o administrador puede necesitar hacer. Estas y otras operaciones se pueden realizar desde la página [*Registros de aplicaciones de Azure AD*](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps) en Azure Portal.
* Conceda el consentimiento del administrador para el registro de aplicaciones. Es posible que la organización tenga habilitado globalmente el *Consentimiento del administrador necesario* en Azure AD para todos los registros de aplicaciones dentro de la suscripción. Si es así, el propietario o el administrador tendrán que seleccionar este botón para la empresa en la página *Permisos de API* del registro de aplicaciones para que este sea válido:

    :::image type="content" source="../articles/digital-twins/media/how-to-set-up-instance/portal/grant-admin-consent.png" alt-text="Vista en el portal del botón Conceder consentimiento del administrador bajo los permisos de API":::
  - Si el consentimiento se concedió correctamente, la entrada de Azure Digital Twins debería mostrar un valor *Estado* de _Concedido para **(su empresa)**_
   
    :::image type="content" source="../articles/digital-twins/media/how-to-set-up-instance/portal/granted-admin-consent-done.png" alt-text="Vista en el portal del consentimiento del administrador concedido para la empresa bajo los permisos de API":::
* Activar el acceso de cliente público
* Establecer direcciones URL de respuesta específicas para el acceso web y de escritorio
* Permitir flujos de autenticación de OAuth2 implícitos

Para más información sobre el registro de aplicaciones y sus distintas opciones de configuración, consulte [*Registro de una aplicación en la plataforma de identidad de Microsoft*](https://docs.microsoft.com/graph/auth-register-app-v2).

Ahora tienen una instancia de Azure Digital Twins lista, permisos asignados para administrarla, así como los permisos configurados para que una aplicación cliente tenga acceso a ella.

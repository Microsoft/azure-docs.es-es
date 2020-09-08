---
title: 'Configuración de la administración de sesiones de autenticación: Azure Active Directory'
description: Personalice la configuración de la sesión de autenticación de Azure AD incluida la frecuencia de inicio de sesión de usuario y la persistencia de la sesión del explorador.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 06/29/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: jlu, calebb
ms.collection: M365-identity-device-management
ms.openlocfilehash: 28516482cf154c81912de0f8e3f35b9e54ef2f69
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88948273"
---
# <a name="configure-authentication-session-management-with-conditional-access"></a>Configuración de la administración de las sesiones de autenticación con el acceso condicional

En las implementaciones complejas, las organizaciones pueden tener la necesidad de restringir las sesiones de autenticación. Algunos escenarios podrían incluir:

* El acceso a los recursos desde un dispositivo no administrado o compartido
* El acceso a información confidencial desde una red externa
* Usuarios de gran impacto
* Aplicaciones empresariales críticas

Los controles de acceso condicional le permiten crear directivas dirigidas a casos de uso específicos dentro de la organización sin afectar a todos los usuarios.

Antes de entrar en detalles sobre cómo configurar la directiva, examinemos la configuración predeterminada.

## <a name="user-sign-in-frequency"></a>Frecuencia de inicio de sesión de usuario

La frecuencia de inicio de sesión define el período de tiempo antes de que se pida a un usuario que vuelva a iniciar sesión cuando intenta acceder a un recurso.

La configuración predeterminada de Azure Active Directory (Azure AD) para la frecuencia de inicio de sesión de usuario es período sucesivo de 90 días. Pedir credenciales a los usuarios a menudo parece algo sensato, pero puede resultar contraproducente: los usuarios que están capacitados para escribir sus credenciales sin pensarlo pueden suministrarlas sin querer a una petición de credenciales malintencionada.

Puede parecer alarmante no pedir a un usuario que vuelva a iniciar sesión; en realidad, cualquier infracción de las directivas de TI revocará la sesión. Algunos ejemplos incluyen, pero sin limitarse, un cambio de contraseña, un dispositivo que no cumple con las normas o la deshabilitación de la cuenta. También puede explícitamente [revocar sesiones de usuarios mediante PowerShell](/powershell/module/azuread/revoke-azureaduserallrefreshtoken?view=azureadps-2.0). La configuración predeterminada de Azure AD se reduce a "no pedir a los usuarios que proporcionen sus credenciales si la posición de seguridad de sus sesiones no ha cambiado".

La configuración de la frecuencia de inicio de sesión funciona con aplicaciones que han implementado los protocolos OAUTH2 u OIDC de acuerdo con los estándares. La mayoría de aplicaciones nativas de Microsoft para Windows, Mac y dispositivos móviles que incluyen las aplicaciones web siguientes cumplen con la configuración.

- Word, Excel y PowerPoint Online
- OneNote Online
- Office.com
- Portal de administración de O365
- Exchange Online
- SharePoint y OneDrive
- Cliente web de Teams
- Dynamics CRM Online
- Azure portal

La configuración de la frecuencia de inicio de sesión también funciona con aplicaciones SAML, siempre y cuando no se eliminen sus propias cookies y se redirijan de nuevo a Azure AD regularmente para la autenticación.

### <a name="user-sign-in-frequency-and-multi-factor-authentication"></a>Frecuencia de inicio de sesión de usuario y autenticación multifactor

Antes, la frecuencia de inicio de sesión se aplicaba solo a la autenticación del primer factor en los dispositivos unidos a Azure AD, unidos a Azure AD híbrido y registrados en Azure AD. No había ninguna manera fácil de que nuestros clientes reforzaran la autenticación multifactor (MFA) en esos dispositivos. Teniendo en cuenta los comentarios de los clientes, ahora la frecuencia de inicio de sesión se aplica también a la autenticación multifactor.

[![Frecuencia de inicio de sesión y MFA](media/howto-conditional-access-session-lifetime/conditional-access-flow-chart-small.png)](media/howto-conditional-access-session-lifetime/conditional-access-flow-chart.png#lightbox)

### <a name="user-sign-in-frequency-and-device-identities"></a>Frecuencia de inicio de sesión de usuario e identidades de dispositivos

Si tiene dispositivos unidos a Azure AD, unidos a Azure AD híbrido o registrados en Azure AD, cuando un usuario desbloquea su dispositivo o inicia sesión de forma interactiva, este evento también cumplirá la directiva de frecuencia de inicio de sesión. En los dos ejemplos siguientes, la frecuencia de inicio de sesión de usuario está establecida en 1 hora:

Ejemplo 1:

- A las 00:00, un usuario inicia sesión en su dispositivo unido a Azure AD de Windows 10 y comienza a trabajar en un documento almacenado en SharePoint Online.
- El usuario continúa trabajando en el mismo documento en su dispositivo durante una hora.
- A la 01:00, se solicita al usuario que vuelva a iniciar sesión según el requisito de frecuencia de inicio de sesión en la directiva de acceso condicional configurada por su administrador.

Ejemplo 2:

- A las 00:00, un usuario inicia sesión en su dispositivo unido a Azure AD de Windows 10 y comienza a trabajar en un documento almacenado en SharePoint Online.
- A las 00:30, el usuario se pone al día y realiza un bloqueo de interrupción en el dispositivo.
- A las 00:45, el usuario vuelve de su interrupción y desbloquea el dispositivo.
- A la 01:45, se solicita al usuario que vuelva a iniciar sesión según el requisito de frecuencia de inicio de sesión de la directiva de acceso condicional configurada por su administrador, ya que el último inicio de sesión se produjo a las 00:45.

## <a name="persistence-of-browsing-sessions"></a>Persistencia de las sesiones de exploración

Una sesión persistente del explorador permite a los usuarios permanecer conectados después de cerrar y volver a abrir la ventana del explorador.

El valor predeterminado de Azure AD para la persistencia de la sesión del explorador permite a los usuarios de dispositivos personales elegir si desean mantener la sesión mostrando un aviso de "¿Quiere mantener la sesión iniciada?" tras una autenticación correcta. Si se configura la persistencia del explorador en AD FS mediante las instrucciones del artículo [Configuración de inicio de sesión único de AD FS](/windows-server/identity/ad-fs/operations/ad-fs-single-sign-on-settings#enable-psso-for-office-365-users-to-access-sharepoint-online
), cumpliremos con esa directiva y persistiremos también en la sesión de Azure AD. También, para configurar si los usuarios del inquilino van a ver el aviso "¿Quiere mantener la sesión iniciada?", cambie la configuración apropiada en el panel de la marca de la empresa en Azure Portal mediante la guía del artículo [Personalización de la página de inicio de sesión de Azure AD](../fundamentals/customize-branding.md).

## <a name="configuring-authentication-session-controls"></a>Configuración de los controles de sesión de autenticación

El acceso condicional es una funcionalidad de Azure AD Premium y requiere una licencia prémium. Para más información sobre el acceso condicional, consulte [Acceso condicional de Azure Active Directory](overview.md#license-requirements).

> [!WARNING]
> Si utiliza la característica de [vigencia del token configurable](../develop/active-directory-configurable-token-lifetimes.md) actualmente en versión preliminar pública, tenga en cuenta que no se admite la creación de dos directivas diferentes para el mismo usuario o combinación de aplicaciones: una con esta característica y otra con la característica de vigencia del token configurable. Microsoft piensa retirar la característica de vigencia del token configurable el 1 de mayo de 2020 y reemplazarla por la característica de administración de sesiones de autenticación de acceso condicional.  

### <a name="policy-1-sign-in-frequency-control"></a>Directiva 1: Control de la frecuencia de inicio de sesión

1. Cree una nueva directiva.
1. Elija todas las condiciones necesarias para el entorno del cliente, incluidas las aplicaciones en la nube de destino.

   > [!NOTE]
   > Se recomienda establecer la misma frecuencia de autenticación para las aplicaciones clave de Microsoft Office, como Exchange Online y SharePoint Online, para una mejor experiencia del usuario.

1. Vaya a **Controles de acceso** > **Sesión** y haga clic en **Frecuencia de inicio de sesión**.
1. Escriba el valor requerido de los días y horas en el primer cuadro de texto.
1. Seleccione un valor para las **horas** o **días** en la lista desplegable.
1. Guarde la directiva.

![Directiva de acceso condicional configurada para la frecuencia de inicio de sesión](media/howto-conditional-access-session-lifetime/conditional-access-policy-session-sign-in-frequency.png)

En los dispositivos de Windows registrados en Azure AD, el inicio de sesión en el dispositivo se considera un aviso. Por ejemplo, si ha configurado la frecuencia de inicio de sesión a 24 horas para aplicaciones de Office, los usuarios de dispositivos de Windows registrados en Azure AD cumplirán la directiva de frecuencia de inicio de sesión por medio del inicio de sesión en el dispositivo, y no se les volverá a preguntar cuando abran las aplicaciones de Office.

### <a name="policy-2-persistent-browser-session"></a>Directiva 2: Sesión del explorador persistente

1. Cree una nueva directiva.
1. Elija todas las condiciones necesarias.

   > [!NOTE]
   > Tenga en cuenta que este control requiere elegir "Todas las aplicaciones en la nube" como una condición. La persistencia de la sesión del explorador se controla mediante el token de la sesión de autenticación. Todas las pestañas de una sesión del explorador comparten un único token de sesión y, por tanto, todas ellas deben compartir el estado de persistencia.

1. Vaya a **Controles de acceso** > **Sesión** y haga clic en **Sesión del explorador persistente**.
1. Seleccione un valor en la lista desplegable.
1. Guarde la directiva.

![Directiva de acceso condicional configurada para el explorador persistente](media/howto-conditional-access-session-lifetime/conditional-access-policy-session-persistent-browser.png)

> [!NOTE]
> La configuración de la sesión del explorador persistente en el acceso condicional de Azure AD sobrescribirá la configuración "¿Quiere mantener la sesión iniciada?" en el panel de personalización de marca de la empresa en Azure Portal para el mismo usuario si ha configurado las dos directivas.

## <a name="validation"></a>Validación

Utilice la herramienta What-If para simular un inicio de sesión del usuario en la aplicación de destino y otras condiciones según la configuración de la directiva. Los controles de administración de la sesión de autenticación se muestran en el resultado de la herramienta.

![Resultados de la herramienta "What If" para el acceso condicional](media/howto-conditional-access-session-lifetime/conditional-access-what-if-tool-result.png)

## <a name="policy-deployment"></a>Implementación de directivas

Para asegurarse de que la directiva funciona según lo esperado, el procedimiento recomendado es probarla antes de implementarla en producción. Lo ideal es usar un inquilino de prueba para comprobar si la nueva directiva funciona según lo previsto. Para más información, consulte [Procedimientos recomendados para el acceso condicional en Azure Active Directory](best-practices.md).

## <a name="next-steps"></a>Pasos siguientes

* Si quiere saber cómo configurar una directiva de acceso condicional, consulte el artículo [Exigir MFA para aplicaciones específicas con acceso condicional de Azure Active Directory](../authentication/tutorial-enable-azure-mfa.md).
* Si está listo para configurar directivas de acceso condicional para su entorno, consulte [Procedimientos recomendados para el acceso condicional en Azure Active Directory](best-practices.md).
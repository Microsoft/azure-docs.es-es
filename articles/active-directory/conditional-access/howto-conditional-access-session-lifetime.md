---
title: Configurar la administración de la sesión de autenticación con el acceso condicional de Azure Active Directory
description: Personalizar la configuración de sesión de autenticación de Azure AD como el inicio de sesión de usuario en la persistencia de sesión de frecuencia y el explorador.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 04/26/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb
ms.collection: M365-identity-device-management
ms.openlocfilehash: 45ea34a673b46dda2ec9606952c707a13d1b72d9
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65766872"
---
# <a name="configure-authentication-session-management-with-conditional-access"></a>Configurar la administración de la sesión de autenticación con el acceso condicional

En las implementaciones complejas, las organizaciones pueden tener una necesidad para restringir las sesiones de autenticación. Algunos escenarios podrían incluir:

* Acceso a los recursos de un dispositivo no administrado o compartido
* Acceso a información confidencial desde una red externa
* Usuarios de alto impacto
* Aplicaciones empresariales críticas

Controles de acceso condicional le permiten crear directivas que tienen como destino de los casos de uso específicos dentro de su organización sin que afecte a todos los usuarios.

Antes de entrar en detalles sobre cómo configurar la directiva, vamos a examinar la configuración predeterminada.

## <a name="user-sign-in-frequency"></a>Frecuencia de inicio de sesión de usuario

Frecuencia de inicio de sesión define el período de tiempo antes de que un usuario se le pida volver a iniciar sesión cuando se intenta acceder a un recurso.

La configuración predeterminada de Azure Active Directory (Azure AD) para el inicio de sesión de usuario en la frecuencia es una ventana con desplazamiento de 90 días. Pedir al usuario las credenciales a menudo puede parecer algo razonable, pero quizás funcione: los usuarios que están entrenados para escribir sus credenciales sin pensar puede involuntariamente suministrarlos a una petición de credenciales malintencionado.

Podría parecer una alarmante no pedir al iniciarla durante 90 días, en realidad de que cualquier infracción de las directivas de TI revocará la sesión usuario. Algunos ejemplos incluyen (pero no se limitan a) un cambio de contraseña, un dispositivo estar o deshabilitar de cuenta. Se puede convertir explícitamente también [revocar sesiones de los usuarios mediante PowerShell](https://docs.microsoft.com/powershell/module/azuread/revoke-azureaduserallrefreshtoken?view=azureadps-2.0). La configuración predeterminada de Azure AD desciende a "no pregunte a los usuarios proporcionen sus credenciales si no ha cambiado la posición de seguridad de sus sesiones".

Inicio de sesión de configuración de frecuencia funciona con aplicaciones que se han implementado los protocolos de OATH2 o OIDC según los estándares. La mayoría aplicaciones nativas de Microsoft para Windows, Mac y móviles cumplen con la configuración.

## <a name="persistence-of-browsing-sessions"></a>Persistencia de sesiones de exploración

Una sesión de explorador persistente permite a los usuarios permanecer con la sesión después de cerrar y volver a abrir la ventana del explorador.

¿El valor predeterminado de Azure AD para la persistencia de sesión de explorador permite a los usuarios en dispositivos personales que elija si desea conservar la sesión presentando un "permanezca iniciada"? símbolo del sistema tras una autenticación correcta. Si se configura la persistencia de explorador en AD FS siguiendo las instrucciones del artículo [único inicio de sesión en configuración de AD FS](https://docs.microsoft.com/windows-server/identity/ad-fs/operations/ad-fs-single-sign-on-settings#enable-psso-for-office-365-users-to-access-sharepoint-online
), agregaremos cumplen con esa directiva y conservar también la sesión de AD de Azure. ¿También puede configurar si los usuarios de su inquilino de ver la "permanezca iniciada"? símbolo del sistema, cambie la configuración adecuada en el panel del portal de Azure siguiendo las instrucciones del artículo de marca de empresa [personalizar su página de inicio de sesión de Azure AD](../fundamentals/customize-branding.md).

## <a name="configuring-authentication-session-controls"></a>Configuración de controles de sesión de autenticación

Acceso condicional es una funcionalidad de Azure AD Premium y requiere una licencia premium. Para más información sobre el acceso condicional, consulte [Acceso condicional de Azure Active Directory](overview.md#license-requirements-for-using-conditional-access).

> [!WARNING]
> Si usas el [vigencia de tokens configurables](../develop/active-directory-configurable-token-lifetimes.md) características actualmente en versión preliminar pública, tenga en cuenta que no se admite crear dos directivas diferentes para la misma combinación de la aplicación o un usuario: uno con esta característica y otra con característica de vigencia de tokens configurables. Microsoft tiene planeado retirar la característica de vigencia de tokens configurables del 15 de octubre y reemplácela con la característica de administración de sesión de autenticación de acceso condicional.  

### <a name="policy-1-sign-in-frequency-control"></a>Directiva 1: Control de inicio de sesión de frecuencia

1. Crear nueva directiva
1. Elija las condiciones de todo lo necesarias para el entorno del cliente, incluidas las aplicaciones de nube de destino.

   > [!NOTE]
   > Se recomienda establecer la frecuencia de símbolo del sistema de autenticación igual para aplicaciones de Microsoft Office claves como Exchange Online y SharePoint Online para la mejor experiencia de usuario.

1. Vaya a **controles de acceso** > **sesión** y haga clic en **frecuencia de inicio de sesión**
1. Escriba el valor requerido de los días y horas en el primer cuadro de texto
1. Seleccione un valor de **horas** o **días** desde la lista desplegable
1. Guardar directiva

![Directiva de acceso condicional configurada para inicio de sesión en frecuencia](media/howto-conditional-access-session-lifetime/conditional-access-policy-session-sign-in-frequency.png)

En Azure AD registrada Windows dispositivos inicie sesión en el dispositivo se considera un símbolo del sistema. Por ejemplo, si ha configurado el inicio de sesión en la frecuencia de hasta 24 horas para aplicaciones de Office, los usuarios en Azure AD registran Windows dispositivos cumplirán la directiva de frecuencia de inicio de sesión al iniciar sesión en el dispositivo y no pedirá nuevamente cuando abran las aplicaciones de Office.

Si ha configurado otro inicio de sesión de frecuencia para aplicaciones web diferentes que se ejecutan en la misma sesión del explorador, se aplicará la directiva más estricta a ambas aplicaciones porque todas las aplicaciones que se ejecutan en la misma sesión del explorador comparten un token de sesión único.

### <a name="policy-2-persistent-browser-session"></a>Directiva 2: Sesión del explorador persistente

1. Crear nueva directiva
1. Elija todas las condiciones necesarias.

   > [!NOTE]
   > Tenga en cuenta que este control requiere elegir "Todas las aplicaciones de nube" como una condición. Persistencia de la sesión de explorador se controla mediante el token de sesión de autenticación. Todas las fichas en una sesión de explorador comparten un token de sesión único y, por tanto, todos ellos deben compartir el estado de persistencia.

1. Vaya a **controles de acceso** > **sesión** y haga clic en **sesión persistente del explorador**
1. Seleccione un valor de la lista desplegable
1. Guardar directiva

![Directiva de acceso condicional configurada para explorador persistente](media/howto-conditional-access-session-lifetime/conditional-access-policy-session-persistent-browser.png)

> [!NOTE]
> La configuración persistente de sesión del explorador en el acceso condicional de Azure AD, sobrescribirá el "permanezca iniciada?" Si se establece la marca de panel en el portal de Azure para el mismo usuario si ha configurado las dos directivas de la empresa.

## <a name="validation"></a>Validación

Utilice la herramienta What If para simular un inicio de sesión del usuario para la aplicación de destino y otras condiciones en función de la configuración de la directiva. Los controles de administración de la sesión de autenticación se muestran en el resultado de la herramienta.

![¿Qué ocurre si da como resultado herramienta de acceso condicional](media/howto-conditional-access-session-lifetime/conditional-access-what-if-tool-result.png)

## <a name="policy-deployment"></a>Implementación de directivas

Para asegurarse de que la directiva funciona según lo esperado, el procedimiento recomendado es probarla antes de implementarla en producción. Lo ideal es usar un inquilino de prueba para comprobar si la nueva directiva funciona según lo previsto. Para obtener más información, vea el artículo [procedimientos recomendados para el acceso condicional en Azure Active Directory](best-practices.md).

## <a name="next-steps"></a>Pasos siguientes

* Si desea saber cómo configurar una directiva de acceso condicional, consulte el artículo [exigencia de MFA para aplicaciones específicas con acceso condicional de Azure Active Directory](app-based-mfa.md).
* Si está listo para configurar directivas de acceso condicional para su entorno, consulte el artículo [procedimientos recomendados para el acceso condicional en Azure Active Directory](best-practices.md).

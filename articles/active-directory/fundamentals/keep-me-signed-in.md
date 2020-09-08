---
title: Configuración del mensaje "¿Quiere mantener la sesión iniciada?" para las cuentas de Azure Active Directory
description: Obtenga información acerca de cómo mantener la sesión iniciada (KMSI), que muestra el mensaje "¿Quiere mantener la sesión iniciada?", cómo configurarlo en el portal de Azure Active Directory y cómo solucionar problemas de inicio de sesión.
services: active-directory
author: CelesteDG
manager: daveba
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.subservice: fundamentals
ms.date: 06/05/2020
ms.author: celested
ms.reviewer: asteen, jlu, hirsin
ms.collection: M365-identity-device-management
ms.openlocfilehash: bed6bc43dfc15abf2bdf9f38a5de2240d348d6fb
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89320263"
---
# <a name="configure-the-stay-signed-in-prompt-for-azure-ad-accounts"></a>Configuración del mensaje "¿Quiere mantener la sesión iniciada?" para las cuentas de Azure AD

Mantener la sesión iniciada (KMSI) muestra un mensaje **¿Quiere mantener la sesión iniciada?** después de que un usuario inicie sesión correctamente. Si un usuario responde **Sí** a este mensaje, el servicio para mantener la sesión iniciada le proporciona un [token de actualización](../develop/developer-glossary.md#refresh-token) persistente. Para los inquilinos federados, el mensaje se muestra cuando el usuario se autentica correctamente en el servicio de identidad federada.

En el siguiente diagrama se muestra el flujo de inicio de sesión del usuario para un inquilino administrado y un inquilino federado, así como el nuevo mensaje para mantener la sesión iniciada. Este flujo contiene lógica inteligente para que la opción **¿Quiere mantener la sesión iniciada?** no se muestre si el sistema de aprendizaje automático detecta un inicio de sesión de alto riesgo o un inicio de sesión desde un dispositivo compartido.

:::image type="content" source="./media/keep-me-signed-in/kmsi-workflow.png" alt-text="Diagrama que muestra el flujo de inicio de sesión del usuario para un inquilino administrado y otro federado":::

> [!NOTE]
> La configuración de la opción para mantener la sesión iniciada requiere el uso de las ediciones Azure Active Directory (Azure AD) Premium 1, Premium 2 o Basic, o bien una licencia de Microsoft 365. Para obtener más información acerca de las ediciones y licencias, consulte [Suscripción a Azure AD Premium](active-directory-get-started-premium.md).<br><br>Las ediciones Azure AD Premium y Basic están disponibles para los clientes de China que utilizan la instancia mundial de Azure AD. Las ediciones Azure AD Premium y Basic no se admiten actualmente en el servicio de Azure administrado por 21Vianet en China. Para más información, póngase en contacto con nosotros en el [foro de Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory/).

## <a name="configure-kmsi"></a>Configuración de KMSI

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con una cuenta de administrador global para el directorio.
1. Seleccione **Azure Active Directory**, elija **Personalización de marca de empresa** y, a continuación, **Configurar**.
1. En la sección **Configuración avanzada**, busque el ajuste **Mostrar la opción para mantener la sesión iniciada**.

   Esta opción de configuración permite elegir si los usuarios deben mantener la sesión iniciada en Azure AD hasta que cierren sesión explícitamente.
   * Si elige **No**, la opción **¿Quiere mantener la sesión iniciada?** estará oculta cuando el usuario inicie sesión correctamente. El usuario deberá iniciar sesión cada vez que se cierre y vuelva a abrir el explorador.
   * Si elige **Sí**, la opción **¿Quiere mantener la sesión iniciada?** se muestra al usuario.

    :::image type="content" source="./media/keep-me-signed-in/kmsi-company-branding-advanced-settings-kmsi-1.png" alt-text="Captura de pantalla que muestra el ajuste Mostrar la opción para mantener la sesión iniciada":::

## <a name="troubleshoot-sign-in-issues"></a>Solución de problemas con el inicio de sesión

Si un usuario no actúa con el mensaje **¿Quiere mantener la sesión iniciada?** , tal como se muestra en el diagrama siguiente, pero abandona el intento de inicio de sesión, verá una entrada de registro de inicio de sesión que indica la interrupción.

:::image type="content" source="./media/keep-me-signed-in/kmsi-stay-signed-in-prompt.png" alt-text="Se muestra el mensaje ¿Quiere mantener la sesión iniciada?":::

Los detalles sobre el error de inicio de sesión son los siguientes y se resaltan en el ejemplo.

* **Código de error de inicio de sesión**: 50140
* **Motivo del error**: Este error se produjo debido a una interrupción en "Mantener la sesión iniciada" cuando el usuario estaba iniciando sesión.

:::image type="content" source="./media/keep-me-signed-in/kmsi-sign-ins-log-entry.png" alt-text="Ejemplo de entrada de registro de inicio de sesión con la interrupción de la opción mantener la sesión iniciada":::

Para evitar que los usuarios vean la interrupción, establezca **Mostrar la opción para mantener la sesión iniciada** en **No** en la configuración de personalización de marca avanzada. Esta acción deshabilita el mensaje de KMSI para todos los usuarios del directorio de Azure AD.

También puede usar los controles de sesión del explorador persistentes en el acceso condicional para impedir que los usuarios vean el mensaje de KMSI. Esta opción le permite deshabilitar el mensaje de KMSI para un grupo de usuarios seleccionado (por ejemplo, los administradores globales) sin que ello afecte al comportamiento de inicio de sesión del resto de usuarios del directorio. Para más información, consulte [Frecuencia de inicio de sesión de usuario](../conditional-access/howto-conditional-access-session-lifetime.md). 

Para asegurar que el aviso de KMSI se muestre sólo cuando pueda beneficiar al usuario, el aviso de KMSI no se muestra intencionadamente en los siguientes escenarios:

* El usuario ha iniciado sesión mediante SSO de conexión directa y la autenticación integrada de Windows (IWA)
* El usuario inició sesión mediante los Servicios de federación de Active Directory (AD FS) y Autenticación integrada de Windows
* El usuario es un invitado en el inquilino
* La puntuación de riesgo del usuario es alta
* El inicio de sesión se produce durante el flujo de consentimiento del usuario o administrador
* El control de sesión del explorador persistente se configura en una directiva de acceso condicional

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre otras opciones que afectan al tiempo de expiración de la sesión de inicio de sesión:

* Microsoft 365: [tiempo de expiración de sesión inactiva](/sharepoint/sign-out-inactive-users)
* Acceso condicional de Azure AD: [frecuencia de inicio de sesión de usuario](../conditional-access/howto-conditional-access-session-lifetime.md)
* Azure Portal: [tiempo de expiración de inactividad de nivel de directorio](../../azure-portal/set-preferences.md#change-the-directory-timeout-setting-admin)
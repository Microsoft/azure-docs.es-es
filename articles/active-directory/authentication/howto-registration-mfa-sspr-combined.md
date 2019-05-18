---
title: 'Introducción al registro combinado para SSPR de Azure AD y Multi-factor Authentication (versión preliminar): Azure Active Directory'
description: Habilitar combina autenticación multifactor de Azure AD y restablecimiento de contraseña de registro (versión preliminar)
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 05/16/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sahenry, calebb
ms.collection: M365-identity-device-management
ms.openlocfilehash: bc4ff596cdafd348288187b0cd9b32f7b4c2d275
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65823389"
---
# <a name="enable-combined-security-information-registration-preview"></a>Información de seguridad permiten combinar del registro (versión preliminar)

Antes de habilitar la nueva experiencia, revise el artículo [combinar información de registro de seguridad (versión preliminar)](concept-registration-mfa-sspr-combined.md) para asegurarse de que entiende la funcionalidad y los efectos de esta característica.

![Información del registro de seguridad combinada una experiencia mejorada](media/howto-registration-mfa-sspr-combined/combined-security-info-more-required.png)

|     |
| --- |
| Información del registro de seguridad combinada para Azure Multi-factor Authentication y restablecimiento de contraseña de autoservicio de Azure Active Directory (Azure AD) es una característica de versión preliminar pública de Azure AD. Para más información sobre las versiones preliminares, consulte [Términos de uso complementarios de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).|
|     |

## <a name="enable-combined-registration"></a>Habilitar registro combinado

Siga estos pasos para habilitar el registro combinado:

1. Inicie sesión en el portal de Azure como un usuario administrador o administrador global.
2. Vaya a **Azure Active Directory** > **configuración de usuario** > **administrar la configuración de características de vista previa del panel de acceso**.
3. En **los usuarios pueden usar características para registrar y administrar información de seguridad de vista previa: actualiza**, optar por habilitar para una **seleccionados** grupo de usuarios o para **todas** a los usuarios.

   ![Habilitar la experiencia de versión preliminar de información de seguridad combinada para todos los usuarios](media/howto-registration-mfa-sspr-combined/combined-security-info-enable.png)

> [!IMPORTANT]
> A partir de marzo de 2019, las opciones de llamada de teléfono no estarán disponibles para la autenticación multifactor y los usuarios SSPR en inquilinos de Azure AD gratuito de prueba. Mensajes SMS no se ven afectados por este cambio. Las opciones de llamada de teléfono aún estará disponibles para los usuarios de inquilinos de Azure AD de pago.

> [!NOTE]
> Después de habilitar el registro combinado, los usuarios que registran o confirme el número de teléfono o aplicación móvil a través de la nueva experiencia puede usarlas para autenticación multifactor y SSPR, si estos métodos están habilitados en la autenticación multifactor y SSPR directivas. Si, a continuación, se deshabilita esta experiencia, los usuarios que vaya al registro SSPR anterior página en `https:/aka.ms/ssprsetup` le pedirá que realicen autenticación multifactor para poder tener acceso a la página.

Si ha configurado el sitio de la lista de asignación de zona de Internet Explorer, visite los sitios siguientes deben estar en la misma zona:

* [https://login.microsoftonline.com](https://login.microsoftonline.com)
* [https://mysignins.microsoft.com](https://mysignins.microsoft.com)
* [https://account.activedirectory.windowsazure.com](https://account.activedirectory.windowsazure.com)

## <a name="conditional-access-policies-for-combined-registration"></a>Directivas de acceso condicional para el registro combinado

Protección de cuándo y cómo registrar usuarios para autenticación multifactor de Azure y restablecimiento de contraseña de autoservicio ahora es posible con las acciones del usuario en la directiva de acceso condicional. Esta característica de vista previa está disponible para organizaciones que han activado el [combina la vista previa del registro](../authentication/concept-registration-mfa-sspr-combined.md). Esta funcionalidad puede habilitarse en las organizaciones que desean los usuarios se registren para Azure Multi-factor Authentication y SSPR desde una ubicación central, como una ubicación de red de confianza durante la incorporación de recursos humanos. Para obtener más información acerca de cómo crear ubicaciones de confianza en el acceso condicional, consulte el artículo [¿qué es la condición de ubicación en el acceso condicional de Azure Active Directory?](../conditional-access/location-condition.md#named-locations)

### <a name="create-a-policy-to-require-registration-from-a-trusted-location"></a>Crear una directiva para requerir registro desde una ubicación de confianza

La siguiente directiva se aplica a todos los usuarios seleccionados, que intentan registrar con la experiencia de registro combinado y bloquea el acceso a menos que se está conectando desde una ubicación marcada como red de confianza.

![Crear una directiva de acceso condicional para controlar el registro de información de seguridad](media/howto-registration-mfa-sspr-combined/conditional-access-register-security-info.png)

1. En el **portal Azure**, vaya a **Azure Active Directory** > **acceso condicional**
1. Seleccione **Nueva directiva**
1. En nombre, escriba un nombre para esta directiva. Por ejemplo, **combinada de registro de información de seguridad en redes de confianza**
1. En **asignaciones**, haga clic en **usuarios y grupos**y seleccione los usuarios y grupos que desee aplicar esta directiva

   > [!WARNING]
   > Los usuarios deben estar habilitados para el [combina la vista previa del registro](../authentication/howto-registration-mfa-sspr-combined.md).

1. En **en la nube aplicaciones o acciones**, seleccione **las acciones del usuario**, comprobar **registrar información de seguridad (versión preliminar)**
1. En **condiciones** > **ubicaciones**
   1. Configurar **sí**
   1. Incluir **cualquier ubicación**
   1. Excluir **todas las ubicaciones de confianza**
   1. Haga clic en **realiza** en la hoja de ubicaciones
   1. Haga clic en **realiza** en la hoja de condiciones
1. En **controles de acceso** > **Grant**
   1. Haga clic en **bloquear el acceso**
   1. A continuación, haga clic en **seleccionar**
1. Establecer **Habilitar directiva** a **en**
1. A continuación, haga clic en **Create**

## <a name="next-steps"></a>Pasos siguientes

[Métodos disponibles para la autenticación multifactor y SSPR](concept-authentication-methods.md)

[Configuración del restablecimiento de contraseña de autoservicio](howto-sspr-deployment.md)

[Configurar Azure Multi-factor Authentication](howto-mfa-getstarted.md)

[Solución de problemas de combina el registro de información de seguridad](howto-registration-mfa-sspr-combined-troubleshoot.md)

[¿Qué es la condición de ubicación en el acceso condicional de Azure Active Directory?](../conditional-access/location-condition.md)
---
title: 'Habilitación del registro de información de seguridad combinado: Azure Active Directory'
description: Aprenda a simplificar la experiencia de los usuarios finales con el registro combinado de Azure AD Multi-Factor Authentication y autoservicio de restablecimiento de contraseña.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 07/20/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.openlocfilehash: de76a9138f782ab699bcd6ff56dab09a4e694102
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87035526"
---
# <a name="enable-combined-security-information-registration-in-azure-active-directory"></a>Habilitación del registro de información de seguridad combinado en Azure Active Directory

Antes del registro combinado, los usuarios se registraban a los métodos de autenticación para Azure Multi-factor Authentication y el autoservicio de restablecimiento de contraseña (SSPR) por separado. La gente estaba confundida por el hecho de que se usaban métodos parecidos para Azure Multi-Factor Authentication y SSPR pero, aún así, se tenían que registrar en las dos características. Ahora, con el registro combinado, los usuarios pueden registrarse una vez y obtener las ventajas de Azure Multi-Factor Authentication y SSPR.

> [!NOTE]
> A partir del 15 de agosto de 2020, todos los nuevos inquilinos de Azure AD se habilitarán automáticamente para el registro combinado.

Para asegurarse de comprender la funcionalidad y los efectos antes de habilitar la nueva experiencia, consulte los [conceptos del registro de información de seguridad combinado](concept-registration-mfa-sspr-combined.md).

![Experiencia mejorada del registro de información de seguridad combinado](media/howto-registration-mfa-sspr-combined/combined-security-info-more-required.png)

## <a name="enable-combined-registration"></a>Habilitar registro combinado

Para habilitar el registro combinado, siga estos pasos:

1. Inicie sesión en Azure Portal como administrador de usuarios o administrador global.
2. Vaya a **Azure Active Directory** > **Configuración de usuario** > **Administrar la configuración de la versión preliminar de características del usuario**.
3. En **Los usuarios pueden utilizar la experiencia combinada de registro de información de seguridad**, elija habilitarla para un grupo de usuarios **Seleccionado** o para **Todos** los usuarios.

   ![Habilitación de la experiencia de información de seguridad combinada para los usuarios](media/howto-registration-mfa-sspr-combined/enable-the-combined-security-info.png)

> [!NOTE]
> Después de habilitar el registro combinado, los usuarios que registren o confirmen su número de teléfono o aplicación móvil a través de la nueva experiencia pueden usarlos para Azure Multi-Factor Authentication y SSPR, si estos métodos están habilitados en las directivas de Azure Multi-Factor Authentication y SSPR.
>
> Si luego deshabilita esta experiencia, los usuarios que vayan a la página de registro de SSPR anterior en `https://aka.ms/ssprsetup` deben realizar la autenticación multifactor para poder acceder a la página.

Si ha configurado el sitio para la *Lista de asignación de sitio a zona* en Internet Explorer, los sitios siguientes deben estar en la misma zona:

* *[https://login.microsoftonline.com](https://login.microsoftonline.com)*
* *[https://mysignins.microsoft.com](https://mysignins.microsoft.com)*
* *[https://account.activedirectory.windowsazure.com](https://account.activedirectory.windowsazure.com)*

## <a name="conditional-access-policies-for-combined-registration"></a>Directivas de acceso condicional para el registro combinado

Para proteger cuándo y cómo se registran los usuarios en Azure Multi-Factor Authentication y el autoservicio de restablecimiento de contraseña, puede aprovecharlas acciones del usuario en la directiva de acceso condicional. Esta funcionalidad se puede habilitar en las organizaciones que quieren que los usuarios se registren en Azure Multi-factor Authentication y SSPR desde una ubicación central, como una ubicación de red de confianza durante la incorporación de recursos humanos.

> [!NOTE]
> Esta directiva solo se aplica cuando un usuario tiene acceso a una página de registro combinada. Esta directiva no exige que el usuario se inscriba a MFA cuando tiene acceso a otras aplicaciones.
>
> Puede crear una directiva de registro de MFA mediante [Azure Identity Protection: configuración de la directiva de MFA](../identity-protection/howto-identity-protection-configure-mfa-policy.md).

Para obtener más información sobre la creación de ubicaciones de confianza en el acceso condicional, consulte [¿Qué es la condición de ubicación del acceso condicional de Azure Active Directory?](../conditional-access/location-condition.md#named-locations)

### <a name="create-a-policy-to-require-registration-from-a-trusted-location"></a>Crear una directiva para requerir el registro desde una ubicación de confianza

Complete los pasos siguientes para crear una directiva que se aplique a todos los usuarios seleccionados que intentan registrarse en la experiencia de registro combinado, y bloquee su acceso a menos que se conecten desde una ubicación marcada como red de confianza:

1. En **Azure Portal**, vaya a **Azure Active Directory** > **Seguridad** > **Acceso condicional**.
1. Seleccione **+ Nueva directiva**.
1. Escriba el nombre de esta directiva, por ejemplo, *Registro de información de seguridad combinado en redes de confianza*.
1. En **Asignaciones**, seleccione **Usuarios y grupos**. Elija los usuarios y grupos a los que desea aplicar esta directiva y, a continuación, seleccione **Listo**.

   > [!WARNING]
   > Los usuarios deben estar habilitados para el registro combinado.

1. En **Aplicaciones o acciones en la nube**, seleccione **Acciones del usuario**. Active **Registro de la información de seguridad** y, a continuación, seleccione **Listo**.

    ![Creación de una directiva de acceso condicional para controlar el registro de la información de seguridad](media/howto-registration-mfa-sspr-combined/require-registration-from-trusted-location.png)

1. En **Condiciones** > **Ubicaciones**, configure las opciones siguientes:
   1. Configure **Sí**.
   1. Incluya **Cualquier ubicación**.
   1. Excluya **Todas las ubicaciones de confianza**.
1. Seleccione **Listo** en la ventana *Ubicaciones* y, a continuación, seleccione **Listo** en la ventana *Condiciones*.
1. En **Controles de acceso** > **Conceder**, elija **Bloquear acceso** y, después, **Seleccionar**.
1. Establezca **Habilitar directiva** en **Activado**.
1. Para finalizar la directiva, seleccione **Crear**.

## <a name="next-steps"></a>Pasos siguientes

Si necesita ayuda, consulte la [solución de problemas de registro de información de seguridad combinado](howto-registration-mfa-sspr-combined-troubleshoot.md) u obtenga información sobre [¿Qué es la condición de ubicación del acceso condicional de Azure AD?](../conditional-access/location-condition.md)

Una vez que los usuarios están habilitados para el registro combinado, puede [habilitar el autoservicio de restablecimiento de contraseña](tutorial-enable-sspr.md) y [habilitar Azure Multi-Factor Authentication](tutorial-enable-azure-mfa.md).

De ser necesario, obtenga información sobre cómo [forzar a los usuarios a que vuelvan a registrar los métodos de autenticación](howto-mfa-userdevicesettings.md#manage-user-authentication-options).

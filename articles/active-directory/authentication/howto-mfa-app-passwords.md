---
title: Configuración de las contraseñas de aplicación de Azure Multi-Factor Authentication - Azure Active Directory
description: Aprenda a configurar y usar contraseñas de aplicación para aplicaciones heredadas en Azure Multi-Factor Authentication
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 06/05/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4c4e34c6f13f7013847e99a362716fc9c570cdaf
ms.sourcegitcommit: 152c522bb5ad64e5c020b466b239cdac040b9377
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88224924"
---
# <a name="enable-and-use-azure-multi-factor-authentication-with-legacy-applications-using-app-passwords"></a>Habilitación y uso de Azure Multi-Factor Authentication con aplicaciones heredadas mediante contraseñas de aplicación

Algunas aplicaciones, como Office 2010 o versiones anteriores y Apple Mail anterior a iOS 11, no admiten la autenticación multifactor. Las aplicaciones no están configuradas para aceptar un formulario secundario de autenticación o solicitud. Para usar estas aplicaciones de forma segura con Azure Multi-Factor Authentication habilitado para las cuentas de usuario, puede usar contraseñas de aplicación. Estas contraseñas de aplicación reemplazaron la contraseña tradicional para permitir que una aplicación omitiera la autenticación multifactor y funcionara correctamente.

La autenticación moderna se admite en los clientes de Microsoft Office 2013 y versiones posteriores. Los clientes de Office 2013, incluido Outlook, admiten protocolos de autenticación modernos y se pueden habilitar para que funcionen con la comprobación en dos pasos. Después de que se habilite el cliente, las contraseñas de aplicación no le serán necesarias.

En este artículo se muestra cómo habilitar y usar contraseñas de aplicación para aplicaciones heredadas que no admiten mensajes de autenticación multifactor.

>[!NOTE]
> Las contraseñas de aplicación no funcionan con directivas de autenticación multifactor basadas en el acceso condicional y la autenticación moderna.

## <a name="overview-and-considerations"></a>Introducción y consideraciones

Cuando una cuenta de usuario está habilitada para Azure Multi-Factor Authentication, se interrumpe la solicitud de inicio de sesión normal mediante una solicitud de comprobación adicional. Algunas aplicaciones anteriores no entienden esta interrupción en el proceso de inicio de sesión, por lo que se produce un error de autenticación. Para mantener la seguridad de la cuenta de usuario y dejar habilitada Azure Multi-Factor Authentication, se pueden usar contraseñas de aplicación en lugar del nombre de usuario y la contraseña normales del usuario. Cuando se usa una contraseña de aplicación durante el inicio de sesión, no hay ninguna solicitud de comprobación adicional, por lo que la autenticación se realiza correctamente.

Las contraseñas de aplicación se generan automáticamente, no las especifica el usuario. Esta contraseña generada automáticamente es más difícil de adivinar y, por tanto, más segura. Los usuarios no tienen que realizar un seguimiento de las contraseñas ni escribirlas cada vez, ya que las contraseñas de aplicación solo se escriben una vez por aplicación.

Al usar las contraseñas de aplicación, deben aplicarse las siguientes consideraciones:

* Hay un límite de 40 contraseñas de aplicación por usuario.
* Las aplicaciones que almacenan en caché las contraseñas y las usan en escenarios locales pueden fallar porque no se conoce la contraseña de aplicación fuera de la cuenta profesional o educativa. Un ejemplo de este escenario es el de los mensajes de correo electrónico de Exchange que son locales pero se archivan en la nube. En este escenario, no funciona la misma contraseña.
* Una vez habilitada Azure Multi-Factor Authentication en una cuenta de usuario, se pueden usar las contraseñas de aplicación con la mayoría de los clientes sin explorador como Outlook y Microsoft Skype Empresarial. Sin embargo, no se pueden realizar acciones administrativas con contraseñas de aplicación mediante aplicaciones sin explorador como Windows PowerShell. No se pueden realizar las acciones, incluso cuando el usuario tiene una cuenta administrativa.
    * Para ejecutar scripts de PowerShell, cree una cuenta de servicio con una contraseña segura y no la habilite para la verificación en dos pasos.
* Si sospecha que una cuenta de usuario está en peligro y revoca o restablece la contraseña de la cuenta, también se deben actualizar las contraseñas de aplicación. Las contraseñas de aplicación no se revocan automáticamente cuando se revoca o restablece una contraseña de cuenta de usuario. El usuario debe eliminar las contraseñas de aplicación existentes y crear otras nuevas.
   * Para obtener más información, consulte [Creación y eliminación de contraseñas de la aplicación en la página de comprobación de seguridad adicional](../user-help/multi-factor-authentication-end-user-app-passwords.md#create-and-delete-app-passwords-from-the-additional-security-verification-page).

>[!WARNING]
> Las contraseñas de aplicación no funcionan en entornos híbridos donde los clientes se comunican con los puntos de conexión de detección automática locales y en la nube. Las contraseñas de dominio deben autenticarse en local. Las contraseñas de aplicación deben autenticarse con la nube.

### <a name="app-password-names"></a>Nombres de contraseñas de aplicación

Los nombres de las contraseñas de aplicación deberían reflejar el dispositivo en el que se usan. Si tiene un portátil con aplicaciones sin explorador como Outlook, Word y Excel, cree una contraseña de aplicación denominada **Portátil** para estas aplicaciones. Cree otra contraseña de aplicación denominada **Escritorio** para las mismas aplicaciones que se ejecutan en el equipo de escritorio.

Se recomienda crear una contraseña de aplicación por dispositivo, en lugar de por aplicación.

## <a name="federated-or-single-sign-on-app-passwords"></a>Contraseñas de aplicaciones de inicio de sesión único o federado

Azure AD admite la federación o el inicio de sesión único (SSO) con Servicios de Dominio de Active Directory (AD DS) local. Si su organización está federada con Azure AD y está usando Azure Multi-Factor Authentication, se aplican las siguientes consideraciones acerca de las contraseñas de aplicación:

>[!NOTE]
> Los siguientes puntos solo se aplican a los clientes federados (SSO).

* Azure AD comprueba las contraseñas de aplicación y, por tanto, se omite la federación. La federación solo se usa activamente al configurar la contraseñas de aplicación.
* En el caso de los usuarios federados (SSO), nunca se contacta con el proveedor de identidades (IdP), a diferencia del flujo pasivo. Las contraseñas de aplicación se almacenan en la cuenta profesional o educativa. Si un usuario deja la empresa, su información fluye hacia la cuenta profesional o educativa utilizando **DirSync** en tiempo real. La deshabilitación o eliminación de la cuenta puede tardar hasta tres horas en sincronizarse, lo que puede retrasar la deshabilitación o eliminación de la contraseña de aplicación en Azure AD.
* La configuración de Access Control de cliente local no se cumple con la característica de las contraseñas de aplicación.
* No hay ningún registro o funcionalidad de auditoría en la autenticación local que esté disponible con la característica de las contraseñas de aplicación.

Algunas arquitecturas avanzadas requieren una combinación de credenciales para autenticación multifactor con los clientes. Estas credenciales pueden incluir una contraseña o nombre de usuario de una cuenta profesional o educativa y las contraseñas de aplicación. Los requisitos dependen de cómo se realiza la autenticación. Para los clientes que se autentican en una infraestructura local, se necesitará una contraseña y un nombre de usuario para la cuenta profesional o educativa. Para los clientes que se autentican en Azure AD, se necesitará una contraseña de aplicación.

Por ejemplo, supongamos que tiene la siguiente arquitectura:

* Su instancia local de Active Directory está federada con Azure AD.
* Utiliza Exchange Online.
* Utiliza Skype Empresarial local.
* Utiliza Azure Multi-Factor Authentication.

En este escenario, utilice las credenciales siguientes:

* Para iniciar sesión en Skype Empresarial, utilice el nombre de usuario y contraseña de la cuenta profesional o educativa.
* Para acceder a la libreta de direcciones a través de un cliente de Outlook que se conecta a Exchange Online, utilice una contraseña de aplicación.

## <a name="allow-users-to-create-app-passwords"></a>Permita que los usuarios creen contraseñas de aplicación

De forma predeterminada, los usuarios no pueden crear contraseñas de aplicación. Debe habilitarse la característica de las contraseñas de aplicación antes de que los usuarios puedan usarlas. Para dar a los usuarios la posibilidad de crear contraseñas de aplicación, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Busque y seleccione **Azure Active Directory** y, a continuación, seleccione **Usuarios**.
3. Seleccione **Multi-Factor Authentication** en la barra de navegación, en la parte superior de la ventana *Usuarios*.
4. En Multi-Factor Authentication, seleccione **Configuración del servicio**.
5. En la página **Configuración del servicio**, seleccione la opción **Permitir a los usuarios crear contraseñas de aplicación para iniciar sesión en aplicaciones sin explorador**.

    ![Captura de pantalla de Azure Portal que muestra la configuración del servicio para Multi-Factor Authentication para permitir el uso de contraseñas de aplicación](media/concept-authentication-methods/app-password-authentication-method.png)
    
> [!NOTE]
>
> Al deshabilitar la capacidad de los usuarios para crear contraseñas de aplicación, las contraseñas de aplicación existentes siguen funcionando. Sin embargo, los usuarios no podrán administrar ni eliminar esas contraseñas de aplicación existentes una vez que deshabilite esta capacidad.
>
> Al deshabilitar la capacidad para crear contraseñas de aplicación, también se recomienda [crear una directiva de acceso condicional para deshabilitar el uso de la autenticación heredada](../conditional-access/block-legacy-authentication.md). Este enfoque evita que funcionen las contraseñas de aplicación existentes y fuerza el uso de los métodos de autenticación modernos.

## <a name="create-an-app-password"></a>Crear una contraseña de aplicación

Cuando los usuarios completan el registro inicial de Azure Multi-Factor Authentication, existe una opción para crear contraseñas de aplicación al final del proceso de registro.

Los usuarios también pueden crear contraseñas de aplicación después del registro. Para más información e instrucciones detalladas para los usuarios, consulte [¿Qué son las contraseñas de aplicación de Azure Multi-Factor Authentication?](../user-help/multi-factor-authentication-end-user-app-passwords.md)

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre cómo permitir que los usuarios se registren rápidamente en Azure Multi-Factor Authentication, consulte [Introducción al registro de información de seguridad combinada](concept-registration-mfa-sspr-combined.md).

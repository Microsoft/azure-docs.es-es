---
title: 'Azure AD Connect: Autenticación de paso a través: preguntas más frecuentes | Microsoft Docs'
description: Respuestas a las preguntas más frecuentes sobre la autenticación de paso a través de Azure Active Directory.
services: active-directory
keywords: Autenticación de paso a través de Azure AD Connect, instalación de Active Directory, componentes necesarios para Azure AD, SSO, inicio de sesión único
documentationcenter: ''
author: billmath
manager: daveba
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 06/09/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7046ab3c30e120aaaf285c32e25fce03524a0cf4
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89280169"
---
# <a name="azure-active-directory-pass-through-authentication-frequently-asked-questions"></a>Autenticación de paso a través de Azure Active Directory: Preguntas más frecuentes

En este artículo se ofrecen respuestas a las preguntas más frecuentes sobre la autenticación de paso a través de Azure Active Directory (Azure AD). Siga comprobando si hay contenido actualizado.

## <a name="which-of-the-methods-to-sign-in-to-azure-ad-pass-through-authentication-password-hash-synchronization-and-active-directory-federation-services-ad-fs-should-i-choose"></a>¿Cuál de los métodos de inicio de sesión, autenticación de paso a través, sincronización de hash de contraseña y Servicios de federación de Active Directory (AD FS) de Azure AD debo elegir?

Consulte [esta guía](./choose-ad-authn.md) para ver una comparación de los distintos métodos de inicio de sesión de Azure AD y cómo elegir el método correcto para su organización.

## <a name="is-pass-through-authentication-a-free-feature"></a>¿Es la autenticación de paso a través una característica gratuita?

La autenticación de paso a través es una característica gratuita. Por tanto, no se necesita ninguna versión de pago de Azure AD para usarla.

## <a name="is-pass-through-authentication-available-in-the-microsoft-azure-germany-cloud-and-the-microsoft-azure-government-cloud"></a>¿Está disponible la autenticación de paso a través en la [nube Microsoft Azure Alemania](https://www.microsoft.de/cloud-deutschland) y en la [nube Microsoft Azure Government](https://azure.microsoft.com/features/gov/)?

No. La autenticación de paso a través solo está disponible en la instancia mundial de Azure AD.

## <a name="does-conditional-access-work-with-pass-through-authentication"></a>¿Funciona el [acceso condicional](../conditional-access/overview.md) con la autenticación de paso a través?

Sí. Todas las capacidades de acceso condicional, incluida Azure Multi-Factor Authentication, funcionan con la autenticación de paso a través.

## <a name="does-pass-through-authentication-support-alternate-id-as-the-username-instead-of-userprincipalname"></a>¿Admite la autenticación de paso a través "Alternate ID" como nombre de usuario, en lugar de "userPrincipalName"?
Sí, el inicio de sesión con un valor que no sea UPN, como un correo electrónico alternativo, se admite para la autenticación de paso a través (PTA) y la sincronización de hash de contraseñas (PHS). Para obtener más información, consulte acerca del [identificador de inicio de sesión alternativo](../authentication/howto-authentication-use-email-signin.md).

## <a name="does-password-hash-synchronization-act-as-a-fallback-to-pass-through-authentication"></a>¿Actúa la sincronización de hash de contraseña como una reserva de la autenticación de paso a través?

No. La autenticación de paso a través _no_ realiza una conmutación automática por error a la sincronización de hash de contraseña. Para evitar errores de inicio de sesión de usuario, debe configurar la autenticación de paso a través para una [alta disponibilidad](how-to-connect-pta-quick-start.md#step-4-ensure-high-availability).

## <a name="what-happens-when-i-switch-from-password-hash-synchronization-to-pass-through-authentication"></a>¿Qué ocurre cuando cambio de la sincronización de hash de contraseña a la autenticación de paso a través?

Cuando se usa Azure AD Connect para cambiar el método de inicio de sesión de sincronización de hash de contraseña a la autenticación de paso a través, esta última se convierte en el método principal de inicio de sesión de los usuarios en dominios administrados. Tenga en cuenta que los hash de contraseña de todos los usuarios que anteriormente se han sincronizado mediante la sincronización de hash de contraseña siguen almacenados en Azure AD.

## <a name="can-i-install-an-azure-ad-application-proxy-connector-on-the-same-server-as-a-pass-through-authentication-agent"></a>¿Puedo instalar un conector del [proxy de aplicación de Azure AD](../manage-apps/application-proxy.md) en el mismo servidor que un agente de autenticación de paso a través?

Sí. Esta configuración es compatible con las nuevas versiones del agente de autenticación de paso a través (versiones 1.5.193.0 o posteriores).

## <a name="what-versions-of-azure-ad-connect-and-pass-through-authentication-agent-do-you-need"></a>¿Qué versiones de Azure AD Connect y del agente de autenticación de paso a través necesita?

Para que esta característica funcione, necesita la versión 1.1.750.0 o posterior para Azure AD Connect y 1.5.193.0 o posterior para el agente de autenticación de paso a través. Instale todo el software servidores con Windows Server 2012 R2 o posterior.

## <a name="what-happens-if-my-users-password-has-expired-and-they-try-to-sign-in-by-using-pass-through-authentication"></a>¿Qué ocurre si mi contraseña de usuario ha expirado y se intenta iniciar sesión mediante la autenticación de paso a través?

Si ha configurado la [escritura diferida de contraseñas](../authentication/concept-sspr-writeback.md) para un usuario concreto y, si el usuario inicia sesión mediante la autenticación de paso a través, puede cambiar o restablecer su contraseña. La contraseña se volverá a escribir en la instancia local de Active Directory, tal como cabría esperar.

Si no ha configurado la escritura diferida de contraseñas para un usuario determinado o si este no tiene una licencia válida de Azure AD asignada, el usuario no podrá actualizar la contraseña en la nube. El usuario no puede actualizar la contraseña incluso aunque haya expirado. En su lugar, el usuario ve este mensaje: "La organización no le permite actualizar la contraseña en este sitio. Actualícela de acuerdo con el método que recomienda la organización o, si necesita ayuda, póngase en contacto con el administrador". El usuario o el administrador debe restablecer su contraseña en la instancia de Active Directory local.

## <a name="how-does-pass-through-authentication-protect-you-against-brute-force-password-attacks"></a>¿Cómo protege la autenticación de paso a través frente a ataques de contraseña por fuerza bruta?

[Lea la información sobre el Bloqueo inteligente](../authentication/howto-password-smart-lockout.md).

## <a name="what-do-pass-through-authentication-agents-communicate-over-ports-80-and-443"></a>¿Qué comunican los agentes de autenticación de paso a través mediante los puertos 80 y 443?

- Los agentes de autenticación realizan las solicitudes HTTPS a través del puerto 443 en todas las operaciones de esta característica.
- Los agentes de autenticación realizan solicitudes HTTP a través del puerto 80 para la descarga de listas de revocación de certificados (CRL) TLS/SSL.

     >[!NOTE]
     >Las actualizaciones recientes redujeron el número de puertos que la característica necesita. Si tiene versiones anteriores de Azure AD Connect o del agente de autenticación, mantenga también estos puertos abiertos: 5671, 8080, 9090, 9091, 9350, 9352 y 10100-10120.

## <a name="can-the-pass-through-authentication-agents-communicate-over-an-outbound-web-proxy-server"></a>¿Se pueden comunicar los agentes de autenticación de paso a través mediante un servidor de proxy web de salida?

Sí. Si la Detección automática de proxy web (WPAD) está habilitada en el entorno local, los agentes de autenticación intentarán automáticamente buscar y usar un servidor proxy web en la red.

Si no tiene WPAD en el entorno, puede agregar información de proxy (tal como se muestra a continuación) para permitir que un agente de autenticación de paso a través se comunique con Azure AD:
- Configure la información del proxy en Internet Explorer antes de instalar el agente de autenticación de paso a través en el servidor. Esto le permitirá completar la instalación del agente de autenticación, pero seguirá apareciendo como **Inactivo** en el portal de administración.
- En el servidor, vaya a "C:\Program Files\Microsoft Azure AD Connect Authentication Agent".
- Edite el archivo de configuración "AzureADConnectAuthenticationAgentService" y agregue las siguientes líneas (reemplace "http\://contosoproxy.com:8080" por su dirección proxy real):

```
   <system.net>
      <defaultProxy enabled="true" useDefaultCredentials="true">
         <proxy
            usesystemdefault="true"
            proxyaddress="http://contosoproxy.com:8080"
            bypassonlocal="true"
         />
     </defaultProxy>
   </system.net>
```

## <a name="can-i-install-two-or-more-pass-through-authentication-agents-on-the-same-server"></a>¿Puedo instalar dos o más agentes de autenticación de paso a través en el mismo servidor?

No, solo se puede instalar un agente de autenticación de paso a través en un único servidor. Si desea configurar la autenticación de paso a través para alta disponibilidad, [siga las instrucciones aquí](how-to-connect-pta-quick-start.md#step-4-ensure-high-availability).

## <a name="do-i-have-to-manually-renew-certificates-used-by-pass-through-authentication-agents"></a>¿Es necesario renovar manualmente los certificados usados por los agentes de autenticación de paso a través?

La comunicación entre cada agente de autenticación de paso a través y Azure AD está protegida mediante la autenticación basada en certificados. [Azure AD renueva estos certificados automáticamente cada pocos meses](how-to-connect-pta-security-deep-dive.md#operational-security-of-the-authentication-agents). No hay ninguna necesidad de renovarlos manualmente. Puede eliminar los certificados expirados anteriores según sea necesario.

## <a name="how-do-i-remove-a-pass-through-authentication-agent"></a>¿Cómo quito un agente de autenticación de paso a través?

Mientras se esté ejecutando un agente de autenticación de paso a través, permanecerá activo y controlará continuamente las solicitudes de inicio de sesión del usuario. Si desea desinstalar un agente de autenticación, vaya a **Panel de Control -> Programas -> Programas y características** y desinstale los programas **Agente de autenticación de Microsoft Azure AD Connect** y el **Agent Updater de Microsoft Azure AD Connect**.

Si activa la hoja Autenticación de paso a través en el [Centro de administración de Azure Active Directory](https://aad.portal.azure.com) después de completar los pasos anteriores, verá que el agente de autenticación se muestra como **inactivo**. Se _espera_ que esto sea así. El agente de autenticación se quita automáticamente de la lista después de 10 días.

## <a name="i-already-use-ad-fs-to-sign-in-to-azure-ad-how-do-i-switch-it-to-pass-through-authentication"></a>Ya utilizo AD FS para iniciar sesión en Azure AD. ¿Cómo se puede cambiar a la autenticación de paso a través?

Si va a migrar desde AD FS (u otra tecnología de federación) a la autenticación de paso a través, es muy recomendable que siga nuestra guía de implementación detallada publicada [aquí](https://github.com/Identity-Deployment-Guides/Identity-Deployment-Guides/blob/master/Authentication/Migrating%20from%20Federated%20Authentication%20to%20Pass-through%20Authentication.docx?raw=true).

## <a name="can-i-use-pass-through-authentication-in-a-multi-forest-active-directory-environment"></a>¿Se puede usar la autenticación de paso a través en un entorno de bosques múltiples de Active Directory?

Sí. Se admiten entornos de bosques múltiples si hay relaciones de confianza de bosque (bidireccionales) entre los bosques de Active Directory y si el enrutamiento de sufijos de nombre está configurado correctamente.

## <a name="does-pass-through-authentication-provide-load-balancing-across-multiple-authentication-agents"></a>¿La autenticación de paso a través proporciona equilibrio de carga entre varios agentes de autenticación?

No, la instalación de varios agentes de autenticación de paso a través solo garantiza una [alta disponibilidad](how-to-connect-pta-quick-start.md#step-4-ensure-high-availability), no proporciona un equilibrio de carga determinista entre los agentes de autenticación. Cualquier agente de autenticación (de manera aleatoria) puede procesar una solicitud de inicio de sesión de usuario determinada.

## <a name="how-many-pass-through-authentication-agents-do-i-need-to-install"></a>¿Cuántos agentes de autenticación de paso a través es necesario instalar?

La instalación de varios agentes de autenticación de paso a través garantiza una [alta disponibilidad](how-to-connect-pta-quick-start.md#step-4-ensure-high-availability). Sin embargo, no proporciona un equilibrio de carga determinista entre los agentes de autenticación.

Considere la carga máxima y la carga media de las solicitudes de inicio de sesión que espera ver en el inquilino. Como referencia, un solo agente de autenticación puede administrar entre 300 y 400 autenticaciones por segundo en un servidor estándar con CPU de 4 núcleos y 16 GB de RAM.

Para calcular el tráfico de red, use la guía sobre el tamaño siguiente:
- Cada solicitud tiene un tamaño de carga de trabajo de (0.5K + 1K * num_of_agents) bytes; es decir, los datos de Azure AD al Agente de autenticación. Aquí, "num_of_agents" indica el número de agentes de autenticación que hay registrados en su inquilino.
- Cada respuesta tiene un tamaño de carga de trabajo de 1K bytes; es decir, los datos del Agente de autenticación a Azure AD.

Para la mayoría de los clientes, dos o tres agentes de autenticación en total son suficientes para obtener alta disponibilidad y capacidad. Debe instalar agentes de autenticación cerca de los controladores de dominio para mejorar la latencia de inicio de sesión.

>[!NOTE]
>Hay un límite de sistema de 40 agentes de autenticación por inquilino.

## <a name="can-i-install-the-first-pass-through-authentication-agent-on-a-server-other-than-the-one-that-runs-azure-ad-connect"></a>¿Puedo instalar el primer agente de autenticación de paso a través en un servidor diferente al que ejecuta Azure AD Connect?

No, este escenario _no_ se admite.

## <a name="why-do-i-need-a-cloud-only-global-administrator-account-to-enable-pass-through-authentication"></a>¿Por qué necesito una cuenta de administrador global solo para la nube para habilitar la autenticación de paso a través?

Se recomienda habilitar o deshabilitar la autenticación de paso a través con una cuenta de administrador global solo para la nube. Información acerca de la [incorporación de una cuenta de administrador global que está solo en la nube](../fundamentals/add-users-azure-active-directory.md). De este modo, se asegura de no quedar bloqueado fuera de su inquilino.

## <a name="how-can-i-disable-pass-through-authentication"></a>¿Se puede deshabilitar la autenticación de paso a través?

Vuelva a ejecutar al Asistente de Azure AD Connect y cambie el método de inicio de sesión del usuario de autenticación de paso a través a otro método. Este cambio deshabilita la autenticación de paso a través en el inquilino y desinstala el agente de autenticación del servidor. Debe desinstalar manualmente los agentes de autenticación de los otros servidores.

## <a name="what-happens-when-i-uninstall-a-pass-through-authentication-agent"></a>¿Qué ocurre cuando se desinstala un agente de autenticación de paso a través?

Si desinstala un agente de autenticación de paso a través de un servidor, el servidor deja de aceptar las solicitudes de inicio de sesión. Para evitar anular la funcionalidad de inicio de sesión del usuario en el inquilino, asegúrese de que haya otro agente de autenticación en ejecución antes de desinstalar un agente de autenticación de paso a través.

## <a name="i-have-an-older-tenant-that-was-originally-setup-using-ad-fs--we-recently-migrated-to-pta-but-now-are-not-seeing-our-upn-changes-synchronizing-to-azure-ad--why-are-our-upn-changes-not-being-synchronized"></a>Tengo un inquilino anterior que originalmente se configuró con AD FS.  Recientemente hicimos la migración a PTA, pero ahora no vemos los cambios de UPN sincronizados con Azure AD.  ¿Por qué los cambios de UPN no se están sincronizando?

A. En las siguientes circunstancias, es posible que los cambios de UPN en el entorno local no se sincronicen si:

- El inquilino de Azure AD se creó antes del 15 de junio de 2015
- Estaba federado inicialmente con el inquilino de Azure AD mediante AD FS para la autenticación
- Cambió para tener usuarios administrados con PTA como autenticación

Esto se debe a que el comportamiento predeterminado de los inquilinos creados antes del 15 de junio de 2015 era bloquear los cambios de UPN.  Si necesita anular el bloqueo de los cambios de UPN, debe ejecutar el cmdlet de PowerShell siguiente:  

`Set-MsolDirSyncFeature -Feature SynchronizeUpnForManagedUsers -Enable $True`

Los inquilinos creados después del 15 de junio de 2015 tienen el comportamiento predeterminado de sincronizar los cambios de UPN.   



## <a name="next-steps"></a>Pasos siguientes
- [Limitaciones actuales](how-to-connect-pta-current-limitations.md): Obtenga información sobre los escenarios que se admiten y los que no.
- [Inicio rápido](how-to-connect-pta-quick-start.md): ponga en marcha la autenticación de paso a través de Azure AD.
- [Migración de AD FS a la autenticación de paso a través](https://github.com/Identity-Deployment-Guides/Identity-Deployment-Guides/blob/master/Authentication/Migrating%20from%20Federated%20Authentication%20to%20Pass-through%20Authentication.docx?raw=true): una guía detallada para migrar desde AD FS (u cualquier otra tecnología de federación) a la autenticación de paso a través.
- [Bloqueo inteligente](../authentication/howto-password-smart-lockout.md): Obtenga información sobre cómo configurar la funcionalidad de bloqueo inteligente en el inquilino para proteger las cuentas de usuario.
- [Profundización técnica](how-to-connect-pta-how-it-works.md): Conozca cómo funciona la característica de autenticación de paso a través.
- [Solución de problemas](tshoot-connect-pass-through-authentication.md): Obtenga información sobre cómo resolver problemas comunes relacionados con la característica de autenticación de paso a través.
- [Análisis a fondo de la seguridad](how-to-connect-pta-security-deep-dive.md): Obtenga información técnica detallada sobre la característica de autenticación de paso a través.
- [SSO de conexión directa de Azure AD](how-to-connect-sso.md): Más información sobre esta característica complementaria.
- [UserVoice](https://feedback.azure.com/forums/169401-azure-active-directory/category/160611-directory-synchronization-aad-connect): Use el foro de Azure Active Directory para solicitar nuevas características.
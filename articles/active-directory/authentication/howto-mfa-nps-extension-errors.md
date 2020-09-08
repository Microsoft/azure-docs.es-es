---
title: 'Solucionar problemas de extensión NPS de Azure MFA: Azure Active Directory'
description: Ayuda para solucionar problemas con la extensión de NPS para Azure Multi-Factor Authentication
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: troubleshooting
ms.date: 11/21/2019
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.custom: has-adal-ref
ms.openlocfilehash: f991e38c184fe44f63af63809deb14eda22f8f4c
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88716731"
---
# <a name="resolve-error-messages-from-the-nps-extension-for-azure-multi-factor-authentication"></a>Resolución de mensajes de error de la extensión de NPS para Azure Multi-Factor Authentication

Si encuentra errores en la extensión de NPS para Azure Multi-Factor Authentication, use este artículo para lograr una resolución más rápida. Los registros de la extensión de NPS se encuentran en el Visor de eventos en **Vistas personalizadas** > **Roles de servidor** > **Servicios de acceso y directivas de redes** en el servidor donde está instalada la extensión de NPS.

## <a name="troubleshooting-steps-for-common-errors"></a>Pasos para la solución de errores comunes

| Código de error | Pasos para solucionar problemas |
| ---------- | --------------------- |
| **CONTACT_SUPPORT** | [Póngase en contacto con el servicio de soporte técnico](#contact-microsoft-support) y mencione la lista de pasos para recopilar registros. Proporcione la mayor cantidad de información que sea posible sobre lo que sucedió antes del error, incluido el identificador del inquilino y el nombre principal de usuario (UPN). |
| **CLIENT_CERT_INSTALL_ERROR** | Puede haber un problema con la forma en que se instaló el certificado de cliente o cómo se asoció con el inquilino. Siga las instrucciones que aparecen en [Solución de problemas con la extensión de NPS de MFA](howto-mfa-nps-extension.md#troubleshooting) para investigar los problemas con el certificado de cliente. |
| **ESTS_TOKEN_ERROR** | Siga las instrucciones que aparecen en [Solución de problemas con la extensión de NPS de MFA](howto-mfa-nps-extension.md#troubleshooting) para investigar los problemas con el token ADAL y el certificado de cliente. |
| **HTTPS_COMMUNICATION_ERROR** | El servidor NPS no puede recibir respuestas de Azure MFA. Compruebe que los firewalls están abiertos de forma bidireccional para el tráfico a y desde https://adnotifications.windowsazure.com |
| **HTTP_CONNECT_ERROR** | En el servidor que ejecuta la extensión NPS, compruebe que puede acceder a `https://adnotifications.windowsazure.com` y `https://login.microsoftonline.com/`. Si esos sitios no cargan, solucione los problemas de conectividad que tenga ese servidor. |
| **Extensión NPS para Azure MFA:** <br> La extensión NPS para Azure MFA solo realiza la autenticación secundaria para las solicitudes Radius en estado AccessAccept. Se ha recibido una solicitud de nombre de usuario con el estado de respuesta AccessReject, omitiendo la solicitud. | Este error suele reflejar un error de autenticación en AD o que el servidor NPS no puede recibir respuestas de Azure AD. Compruebe que los firewalls están abiertos de forma bidireccional para el tráfico a y desde `https://adnotifications.windowsazure.com` y `https://login.microsoftonline.com` mediante los puertos 80 y 443. También es importante comprobar que en la pestaña DIAL-IN de los permisos de acceso a la red, la configuración está establecida en "Controlar acceso a través de la directiva de red NPS". Este error también puede desencadenarse si no se asigna una licencia al usuario. |
| **REGISTRY_CONFIG_ERROR** | Falta una clave en el registro de la aplicación, lo que puede deberse a que el [script de PowerShell](howto-mfa-nps-extension.md#install-the-nps-extension) no se ejecutó después de la instalación. El mensaje de error debe incluir la clave que falta. Asegúrese de que la clave se encuentra en HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\AzureMfa. |
| **REQUEST_FORMAT_ERROR** <br> Falta el atributo nombreUsuario\Identificador obligatorio en la solicitud de RADIUS. Compruebe que NPS recibe las solicitudes de RADIUS | Este error refleja habitualmente un problema de instalación. La extensión de NPS debe estar instalada en servidores de NPS que pueden recibir solicitudes de RADIUS. Los servidores de NPS que están instalados como dependencias para servicios como RDG y RRAS no reciben solicitudes de RADIUS. La extensión de NPS no funciona si se instala en esas instalaciones y genera errores porque no puede leer los detalles de la solicitud de autenticación. |
| **REQUEST_MISSING_CODE** | Asegúrese de que el protocolo de cifrado de contraseña entre los servidores NPS y NAS admite el método de autenticación secundario que usa. **PAP** es compatible con todos los métodos de autenticación de Azure MFA en la nube: llamada de teléfono, mensaje de texto unidireccional, notificación de aplicación móvil y código de verificación de la aplicación móvil. **CHAPV2** y **EAP** admiten llamadas de teléfono y notificaciones de aplicación móvil. |
| **USERNAME_CANONICALIZATION_ERROR** | Compruebe que el usuario existe en la instancia local de Active Directory y que el servicio NPS tiene permiso para acceder al directorio. Si usa confianzas entre bosques, [póngase en contacto con el servicio de soporte técnico](#contact-microsoft-support) para más ayuda. |

### <a name="alternate-login-id-errors"></a>Errores de identificador de inicio de sesión alternativo

| Código de error | Mensaje de error | Pasos para solucionar problemas |
| ---------- | ------------- | --------------------- |
| **ALTERNATE_LOGIN_ID_ERROR** | Error: error de búsqueda de userObjectSid | Compruebe que el usuario existe en la instancia local de Active Directory. Si usa confianzas entre bosques, [póngase en contacto con el servicio de soporte técnico](#contact-microsoft-support) para más ayuda. |
| **ALTERNATE_LOGIN_ID_ERROR** | Error: error en la búsqueda de LoginId alternativo | Compruebe que LDAP_ALTERNATE_LOGINID_ATTRIBUTE está establecido en un [atributo de directorio activo válido](/windows/win32/adschema/attributes-all). <br><br> Si LDAP_FORCE_GLOBAL_CATALOG está establecido en True, o LDAP_LOOKUP_FORESTS se configura con un valor no vacío, compruebe que ha configurado un catálogo global y que el atributo AlternateLoginId se agrega a él. <br><br> Si LDAP_LOOKUP_FORESTS está configurado con un valor no vacío, compruebe que el valor es correcto. Si hay más de un nombre de bosque, los nombres deben estar separados con puntos y coma, no espacios. <br><br> Si estos pasos no solucionan el problema, [póngase en contacto con soporte técnico](#contact-microsoft-support) para obtener más ayuda. |
| **ALTERNATE_LOGIN_ID_ERROR** | Error: el valor de LoginId alternativo está vacío | Compruebe que el atributo de AlternateLoginId está configurado para el usuario. |

## <a name="errors-your-users-may-encounter"></a>Errores que pueden encontrar los usuarios

| Código de error | Mensaje de error | Pasos para solucionar problemas |
| ---------- | ------------- | --------------------- |
| **AccessDenied** | El inquilino autor de la llamada no tiene permisos de acceso para realizar la autenticación por el usuario | Revise si el dominio del inquilino y el dominio del nombre principal de usuario (UPN) son iguales. Por ejemplo, asegúrese de que user@contoso.com intenta autenticarse en el inquilino Contoso. El nombre principal de usuario representa un usuario válido para el inquilino en Azure. |
| **AuthenticationMethodNotConfigured** | El método de autenticación especificado no se configuró para el usuario | Haga que el usuario agregue o compruebe los métodos de comprobación según las instrucciones que aparecen en [Administración de la configuración de la verificación en dos pasos](../user-help/multi-factor-authentication-end-user-manage-settings.md). |
| **AuthenticationMethodNotSupported** | No se admite el método de autenticación especificado. | Recopile todos los registros que incluyen este error y [póngase en contacto con el servicio de soporte técnico](#contact-microsoft-support). Cuando lo haga, proporcione el nombre de usuario y el método de comprobación secundario que desencadenó el error. |
| **BecAccessDenied** | La llamada Bec de MSODS devolvió un acceso denegado, probablemente porque el nombre de usuario no está definido en el inquilino | El usuario se encuentra en la instancia local de Active Directory, pero AD Connect no la sincroniza en Azure AD. O bien puede que el usuario falte en el inquilino. Agregue el usuario a Azure AD y haga que agregue sus métodos de comprobación según las instrucciones que aparecen en [Administración de la configuración de la verificación en dos pasos](../user-help/multi-factor-authentication-end-user-manage-settings.md). |
| **InvalidFormat** o **StrongAuthenticationServiceInvalidParameter** | El número de teléfono tiene un formato no reconocible | Haga que el usuario corrija los números de teléfono de comprobación. |
| **InvalidSession** | La sesión especificada no es válida o puede haber expirado | La sesión tarda más de tres minutos en completarse. Compruebe que el usuario escribe el código de verificación o que responde a la notificación de aplicación en menos de tres minutos después de que se inicia la solicitud de autenticación. Si no se soluciona el problema, compruebe que no haya latencias de red entre el cliente, el servidor NAS, el servidor NPS y el punto de conexión de Azure MFA.  |
| **NoDefaultAuthenticationMethodIsConfigured** | No se configuró ningún método de autenticación predeterminado para el usuario | Haga que el usuario agregue o compruebe los métodos de comprobación según las instrucciones que aparecen en [Administración de la configuración de la verificación en dos pasos](../user-help/multi-factor-authentication-end-user-manage-settings.md). Compruebe que el usuario eligió un método de autenticación predeterminado y que configuró dicho método para la cuenta. |
| **OathCodePinIncorrect** | Se escribió un PIN y un código de error incorrectos. | Este error no se espera en la extensión de NPS. Si el usuario encuentra este error, [póngase en contacto con el servicio de soporte técnico](#contact-microsoft-support) para ayuda en la solución de problemas. |
| **ProofDataNotFound** | No se configuraron datos de prueba para el método de autenticación especificado. | Haga que el usuario pruebe con otro método de comprobación o agregue métodos de comprobación nuevos según las instrucciones que aparecen en [Administración de la configuración de la verificación en dos pasos](../user-help/multi-factor-authentication-end-user-manage-settings.md). Si el usuario sigue viendo este error una vez que confirmó que el método de comprobación está configurado correctamente, [póngase en contacto con el servicio de soporte técnico](#contact-microsoft-support). |
| **SMSAuthFailedWrongCodePinEntered** | Se escribió un PIN y un código de error incorrectos. (OneWaySMS) | Este error no se espera en la extensión de NPS. Si el usuario encuentra este error, [póngase en contacto con el servicio de soporte técnico](#contact-microsoft-support) para ayuda en la solución de problemas. |
| **TenantIsBlocked** | El inquilino está bloqueado | [Póngase en contacto con el servicio de soporte técnico](#contact-microsoft-support) con el *identificador de inquilino* de la página de propiedades de Azure AD en Azure Portal. |
| **UserNotFound** | No se encontró el usuario especificado | El inquilino ya no está visible como activo en Azure AD. Compruebe que la suscripción está activa y que tiene las aplicaciones propias requeridas. Además, asegúrese de que el inquilino del asunto del certificado sea el esperado y que el certificado siga siendo válido y esté registrado en la entidad de servicio. |

## <a name="messages-your-users-may-encounter-that-arent-errors"></a>Mensajes que pueden encontrar los usuarios que no son errores

A veces, los usuarios pueden recibir mensajes de Multi-Factor Authentication debido a un error de la solicitud de autenticación. No se trata de errores de configuración, sino que advertencias intencionales que explican por qué se denegó una solicitud de autenticación.

| Código de error | Mensaje de error | Pasos recomendados |
| ---------- | ------------- | ----------------- |
| **OathCodeIncorrect** | Se escribió un código incorrecto\Código OATH incorrecto | El usuario escribió un código incorrecto. Pídale que solicite un código nuevo o que vuelva a iniciar sesión para intentarlo nuevamente. |
| **SMSAuthFailedMaxAllowedCodeRetryReached** | Se alcanzó el número máximo de reintentos de código | El usuario no pudo completar el desafío de comprobación demasiadas veces. En función de lo que indique la configuración puede que ahora un administrador deba desbloquearlo.  |
| **SMSAuthFailedWrongCodeEntered** | Se escribió un código incorrecto/OTP de mensaje de texto incorrecto | El usuario escribió un código incorrecto. Pídale que solicite un código nuevo o que vuelva a iniciar sesión para intentarlo nuevamente. |

## <a name="errors-that-require-support"></a>Errores que requieren soporte técnico

Si encuentra uno de estos errores, se recomienda [ponerse en contacto con el servicio de soporte técnico](#contact-microsoft-support) para obtener ayuda en el diagnóstico. No hay un conjunto estándar de pasos para abordar estos errores. Cuando se comunique con el servicio de soporte técnico, asegúrese de incluir toda la información que sea posible sobre los pasos que generaron un error, además de la información del inquilino.

| Código de error | Mensaje de error |
| ---------- | ------------- |
| **InvalidParameter** | La solicitud no debe ser nula |
| **InvalidParameter** | El valor ObjectId no debe estar vacío ni ser un valor nulo para ReplicationScope:{0} |
| **InvalidParameter** | La longitud de CompanyName \{0}\ es mayor que la longitud máxima permitida {1} |
| **InvalidParameter** | El valor UserPrincipalName no debe estar vacío ni ser un valor nulo |
| **InvalidParameter** | El valor TenantId proporcionado no tiene el formato correcto |
| **InvalidParameter** | El valor SessionId no debe estar vacío ni ser un valor nulo |
| **InvalidParameter** | No se pudo resolver ningún valor ProofData desde la solicitud o Msods. El valor de ProofData es desconocido |
| **InternalError** |  |
| **OathCodePinIncorrect** |  |
| **VersionNotSupported** |  |
| **MFAPinNotSetup** |  |

## <a name="next-steps"></a>Pasos siguientes

### <a name="troubleshoot-user-accounts"></a>Solución de problemas de cuentas de usuario

Si los usuarios [tienen problemas con la verificación en dos pasos](../user-help/multi-factor-authentication-end-user-troubleshoot.md), ayúdelos a autodiagnosticar problemas.

### <a name="health-check-script"></a>Script de comprobación de estado

El [script de comprobación de estado de la extensión NPS para Azure MFA](/samples/azure-samples/azure-mfa-nps-extension-health-check/azure-mfa-nps-extension-health-check/) realiza una comprobación de estado básica al solucionar problemas de la extensión NPS. Ejecute el script y elija la opción 3.

### <a name="contact-microsoft-support"></a>Consultar al soporte técnico de Microsoft

Si necesita ayuda adicional, póngase en contacto con un profesional de soporte técnico a través de [Soporte técnico del servidor Azure Multi-Factor Authentication](https://support.microsoft.com/oas/default.aspx?prid=14947). Al ponerse en contacto con nosotros, nos resultará de gran utilidad que incluya tanta información sobre su problema como sea posible. Entre la información que puede aportar se incluyen la página donde vio el error, el código de error específico, el identificador de sesión específico, el identificador del usuario que vio el error y los registros de depuración.

Para recopilar los registros de depuración de diagnóstico de soporte técnico, use los pasos siguientes en el servidor NPS:

1. Abra el Editor del Registro y vaya a HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\AzureMfa; establezca **VERBOSE_LOG** en **TRUE**
2. Abra un símbolo del sistema de administrador y ejecute estos comandos:

   ```
   Mkdir c:\NPS
   Cd NPS
   netsh trace start Scenario=NetConnection capture=yes tracefile=c:\NPS\nettrace.etl
   logman create trace "NPSExtension" -ow -o c:\NPS\NPSExtension.etl -p {7237ED00-E119-430B-AB0F-C63360C8EE81} 0xffffffffffffffff 0xff -nb 16 16 -bs 1024 -mode Circular -f bincirc -max 4096 -ets
   logman update trace "NPSExtension" -p {EC2E6D3A-C958-4C76-8EA4-0262520886FF} 0xffffffffffffffff 0xff -ets
   ```

3. Reproduzca el problema

4. Use estos comandos para detener el seguimiento:

   ```
   logman stop "NPSExtension" -ets
   netsh trace stop
   wevtutil epl AuthNOptCh C:\NPS\%computername%_AuthNOptCh.evtx
   wevtutil epl AuthZOptCh C:\NPS\%computername%_AuthZOptCh.evtx
   wevtutil epl AuthZAdminCh C:\NPS\%computername%_AuthZAdminCh.evtx
   Start .
   ```

5. Abra el Editor del Registro y vaya a HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\AzureMfa; establezca **VERBOSE_LOG** en **FALSE**
6. Comprima el contenido de la carpeta C:\NPS y adjunte el archivo comprimido al caso de soporte técnico.
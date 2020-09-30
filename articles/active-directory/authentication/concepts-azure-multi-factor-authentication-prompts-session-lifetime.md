---
title: Mensajes y duración de la sesión de Multi-Factor Authentication
description: Obtenga información acerca de la configuración recomendada para los mensajes de reautenticación con Azure Multi-Factor Authentication y cómo se aplica la duración de la sesión.
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: inbarc
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0019f7d8195dc39127b992a31ebd8c33e55452f6
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89179358"
---
# <a name="optimize-reauthentication-prompts-and-understand-session-lifetime-for-azure-multi-factor-authentication"></a>Optimice los mensajes de reautenticación y comprenda la duración de la sesión para Azure Multi-Factor Authentication.

Azure Active Directory (Azure AD) tiene varias opciones que determinan la frecuencia con la que los usuarios deben volver a autenticarse. Esta reautenticación podría ser con un primer factor como contraseña, FIDO o Microsoft Authenticator sin contraseña, o bien mediante autenticación multifactor (MFA). Puede configurar estas opciones de reautenticación según sea necesario para su propio entorno y la experiencia de usuario que desee.

La configuración predeterminada de Azure AD para la frecuencia de inicio de sesión de usuario es un período sucesivo de 90 días. La petición de credenciales a los usuarios a menudo parece algo sensato, pero puede tener sus desventajas. Si los usuarios están entrenados para escribir sus credenciales sin necesidad de pensar, pueden proporcionarlas involuntariamente ante una petición de credenciales malintencionada.

Puede parecer alarmante no pedir a un usuario que vuelva a iniciar sesión, aunque cualquier infracción de las directivas de TI revocará la sesión. Algunos ejemplos incluyen un cambio de contraseña, un dispositivo que no cumple con las normas o la operación de deshabilitación de la cuenta. También puede explícitamente [revocar sesiones de usuarios mediante PowerShell](/powershell/module/azuread/revoke-azureaduserallrefreshtoken).

En este artículo se detallan las configuraciones recomendadas y cómo funcionan estas e interactúan entre sí.

## <a name="recommended-settings"></a>Configuración recomendada

Para ofrecer a los usuarios el equilibrio adecuado entre seguridad y facilidad de uso pidiéndoles que inicien sesión con la frecuencia correcta, se recomiendan las siguientes configuraciones:

* Si tiene Azure AD Premium:
    * Habilite el inicio de sesión único (SSO) en todas las aplicaciones mediante [dispositivos administrados](../devices/overview.md) o [SSO de conexión directa](../hybrid/how-to-connect-sso.md).
    * Si se requiere la reautenticación, use una [directiva de frecuencia de inicio de sesión](../conditional-access/howto-conditional-access-session-lifetime.md) de acceso condicional.
    * Para los usuarios que inician sesión desde dispositivos no administrados o escenarios de dispositivos móviles, use el acceso condicional para habilitar sesiones persistentes del explorador y directivas de frecuencia de inicio de sesión.
* Si tiene licencias de aplicaciones de Office 365 o el nivel de Azure AD gratuito:
    * Habilite el inicio de sesión único (SSO) en todas las aplicaciones mediante [dispositivos administrados](../devices/overview.md) o [SSO de conexión directa](../hybrid/how-to-connect-sso.md).
    * Mantenga habilitada la opción *Remain signed-in* (Permanecer conectado) y guíe a los usuarios para que la acepten.
* Para escenarios con dispositivos móviles, asegúrese de que los usuarios utilizan la aplicación Microsoft Authenticator. Esta aplicación se usa como agente para otras aplicaciones federadas de Azure AD y reduce los avisos de autenticación en el dispositivo.

Nuestra investigación muestra que estos valores son adecuados para la mayoría de los inquilinos. Algunas combinaciones de estas opciones, como *Recordar Multi-Factor Authentication* y *Remain singed-in* (Permanecer conectado), pueden dar lugar a que los usuarios se autentiquen con demasiada frecuencia. Los mensajes de reautenticación periódicos son perjudiciales para la productividad del usuario y pueden hacerlos más vulnerables frente a los ataques.

## <a name="azure-ad-session-lifetime-configuration-settings"></a>Opciones de configuración de duración de sesión de Azure AD

Para optimizar la frecuencia de los mensajes de autenticación para los usuarios, puede configurar las opciones de duración de la sesión de Azure AD. Sea consciente de las necesidades de su empresa y sus usuarios y configure los valores que proporcionen el mejor equilibrio para su entorno.

### <a name="evaluate-session-lifetime-policies"></a>Evaluación de las directivas de duración de la sesión

Sin ninguna configuración de duración de la sesión, no hay cookies persistentes en la sesión del explorador. Cada vez que un usuario cierra y abre el explorador, recibe una solicitud de reautenticación. En los clientes de Office, el período de tiempo predeterminado es un período sucesivo de 90 días. Con esta configuración de Office predeterminada, si el usuario ha restablecido su contraseña o ha habido inactividad de más de 90 días, el usuario deberá volver a autenticarse con todos los factores necesarios (primer y segundo factor).

Un usuario podría ver varias solicitudes de MFA en un dispositivo que no tiene una identidad en Azure AD. Se generan varios mensajes cuando cada aplicación tiene su propio token de actualización de OAuth que no se comparte con otras aplicaciones cliente. En este escenario, MFA solicita la autenticación varias veces, ya que cada aplicación solicita un token de actualización de OAuth que se va a validar con MFA.

En Azure AD, la directiva más restrictiva para la duración de la sesión determina cuándo es necesario volver a autenticar el usuario. Considere el caso siguiente:

* Habilita *Remain signed-in* (Permanecer conectado), que usa una cookie persistente del explorador.
* Además, habilita *Remember MFA for 14 days* (Recordar MFA durante 14 días).

En este escenario de ejemplo, el usuario debe volver a autenticarse cada 14 días. Este comportamiento sigue la directiva más restrictiva, incluso si la opción *Mantener sesión iniciada* no requeriría por sí misma al usuario la reautenticación en el explorador.

### <a name="managed-devices"></a>Dispositivos administrados

Los dispositivos unidos a Azure AD mediante Unión a Azure AD o Unión a Azure AD híbrido reciben [tokens de actualización principales (PRT)](../devices/concept-primary-refresh-token.md) para usar el inicio de sesión único (SSO) en todas las aplicaciones. Estos tokens de actualización principales permiten a un usuario iniciar sesión una vez en el dispositivo y ayudan al personal de TI a garantizar que se cumplen los estándares de seguridad y cumplimiento. Si es necesario solicitar a un usuario que inicie sesión con más frecuencia en un dispositivo combinado para algunas aplicaciones o escenarios, puede recurrir a la [frecuencia de inicio de sesión de acceso condicional](../conditional-access/howto-conditional-access-session-lifetime.md).

### <a name="show-option-to-remain-signed-in"></a>Mostrar la opción para mantener la sesión iniciada

Cuando un usuario selecciona **Sí** en la opción *¿Quiere mantener la sesión iniciada?* durante el inicio de sesión, se establece una cookie persistente en el explorador. Esta cookie persistente recuerda el primer y el segundo factor, y se aplica solo a las solicitudes de autenticación en el explorador.

![Captura de pantalla de solicitud de ejemplo para mantener la sesión iniciada](./media/concepts-azure-multi-factor-authentication-prompts-session-lifetime/stay-signed-in-prompt.png)

Si tiene una licencia Azure AD Premium 1, se recomienda usar la directiva de acceso condicional para la *sesión del explorador persistente*. Esta directiva sobrescribe al valor de *¿Quiere mantener la sesión iniciada?* y proporciona una experiencia de usuario mejorada. Si no tiene una licencia de Azure AD Premium 1, se recomienda habilitar la opción de mantener la sesión iniciada para los usuarios.

Para obtener más información sobre la configuración de la opción para permitir que los usuarios sigan con la sesión iniciada, consulte [Personalización de la página de inicio de sesión de Azure AD](../fundamentals/customize-branding.md#customize-your-azure-ad-sign-in-page).

### <a name="remember-multi-factor-authentication"></a>Recordar Multi-Factor Authentication  

Esta opción permite configurar valores entre 1 y 365 días y establece una cookie persistente en el explorador cuando un usuario selecciona la opción **No preguntar de nuevo durante X días** en el momento del inicio de sesión.

![Captura de pantalla del mensaje de ejemplo para aprobar una solicitud de inicio de sesión](./media/concepts-azure-multi-factor-authentication-prompts-session-lifetime/approve-sign-in-request.png)

Aunque esta configuración reduce el número de autenticaciones en las aplicaciones web, aumenta el número de autenticaciones para los clientes de autenticación moderna, como los clientes de Office. Normalmente, estos clientes solo hacen la solicitud después de un restablecimiento de la contraseña o una inactividad de 90 días. Sin embargo, si se establece este valor en menos de 90 días, se acortan los mensajes de MFA predeterminados para los clientes de Office y aumenta la frecuencia de una nueva autenticación. Cuando se usa en combinación con **Remain signed-in** (Permanecer conectado) o con directivas de acceso condicional, puede aumentar el número de solicitudes de autenticación.

Si usa *Recordar Multi-Factor Authentication* y tiene licencias de Azure AD Premium 1, considere la posibilidad de migrar esta configuración a la frecuencia de inicio de sesión de acceso condicional. De lo contrario, considere la posibilidad de usar *Mantener sesión iniciada*.

Para obtener más información, consulte [Recordar Multi-Factor Authentication](howto-mfa-mfasettings.md#remember-multi-factor-authentication).

### <a name="authentication-session-management-with-conditional-access"></a>Administración de la sesión de autenticación con acceso condicional

La **frecuencia de inicio de sesión** permite al administrador elegir la frecuencia de inicio de sesión que se aplica para el primer y el segundo factor tanto en el cliente como en el explorador. Se recomienda usar estos valores, junto con dispositivos administrados, en escenarios en los que es necesario restringir la sesión de autenticación, por ejemplo, los de las aplicaciones empresariales críticas.

Una **sesión persistente del explorador** permite a los usuarios permanecer conectados después de cerrar y volver a abrir la ventana del explorador. De forma similar a la configuración de *Remain signed-in* (Permanecer conectado), establece una cookie persistente en el explorador. Sin embargo, dado que está configurado por el administrador, no requiere que el usuario seleccione **Sí** en la opción *¿Quiere mantener la sesión iniciada?* , por lo que permite una mejor experiencia del usuario. Si usa la opción*Remain signed-in* (Permanecer conectado), se recomienda habilitar en su lugar la directiva **Sesión del explorador persistente**.

Para obtener más información, consulte [Configuración de la administración de las sesiones de autenticación con el acceso condicional](../conditional-access/howto-conditional-access-session-lifetime.md).

### <a name="configurable-token-lifetimes"></a>Vigencia de tokens configurable

Esta opción permite configurar la duración del token emitido por Azure Active Directory. Esta directiva se reemplaza por la *administración de sesiones de autenticación con acceso condicional*. Si actualmente utiliza *Vigencia de tokens configurable*, se recomienda iniciar la migración a las directivas de acceso condicional.

## <a name="review-your-tenant-configuration"></a>Revisión de la configuración del inquilino  

Ahora que comprende cómo funcionan las diferentes opciones y la configuración recomendada, es el momento de comprobar la configuración de los inquilinos y realizar los cambios según corresponda:

Para configurar o revisar la opción *Remain signed-in* (Permanecer conectado), complete los pasos siguientes:

1. En el portal de Azure AD, busque y seleccione *Azure Active Directory*.
1. Seleccione **Personalización de marca de empresa** y, a continuación, para cada configuración regional, elija **Mostrar la opción para mantener la sesión iniciada**.
1. Seleccione **Sí** y, después, *Guardar*.

Para recordar la configuración de Multi-Factor Authentication, siga estos pasos:

1. En el portal de Azure AD, busque y seleccione *Azure Active Directory*.
1. Seleccione **Seguridad** y luego **MFA**.
1. En **Configurar**, seleccione **Configuración adicional de MFA basado en la nube**.
1. En la *configuración del servicio Multi-Factor Authentication*, desplácese hasta la opción para **recordar la configuración de la autenticación multifactor**. Deshabilite la opción desactivando la casilla.

Para configurar directivas de acceso condicional para la frecuencia de inicio de sesión y la sesión del explorador persistente, siga estos pasos:

1. En el portal de Azure AD, busque y seleccione *Azure Active Directory*.
1. Seleccione **Seguridad** y luego **Acceso condicional**.
1. Configure una directiva mediante las opciones de administración de sesión recomendadas que se detallan en este artículo.

Para revisar la vigencia de los tokens, [use Azure AD PowerShell para consultar las directivas de Azure AD](../develop/active-directory-configurable-token-lifetimes.md#prerequisites). Deshabilite las directivas que haya implementado.

Si hay más de una opción de configuración habilitada en el inquilino, se recomienda actualizar la configuración en función de las licencias disponibles. Por ejemplo, si tiene licencias Premium de Azure AD, solo debe usar la directiva de acceso condicional de *Frecuencia de inicio de sesión* y *Sesión del explorador persistente*. Si tiene aplicaciones de Office 365 o licencias de Azure AD gratuitas, debe usar la configuración *Remain signed-in* (Permanecer conectado).

Si ha habilitado la vigencia de tokens configurable, esta funcionalidad se quitará pronto. Planee una migración a una directiva de acceso condicional.

En la tabla siguiente se resumen las recomendaciones basadas en licencias:

|              | Azure AD gratuito y aplicaciones de Office 365 | Azure AD Premium |
|------------------------------|-----------------------------------|------------------|
| **SSO**                      | [Unión a Azure AD](../devices/concept-azure-ad-join.md) o [Unión a Azure AD híbrido](../devices/concept-azure-ad-join-hybrid.md), o [SSO de conexión directa](../hybrid/how-to-connect-sso.md) para dispositivos no administrados. | Unión a Azure AD<br />Unión a Azure AD híbrido |
| **Configuración de la reautenticación** | Permanecer conectado                  | Uso de directivas de acceso condicional para la frecuencia de inicio de sesión y la sesión del explorador persistente |

## <a name="next-steps"></a>Pasos siguientes

Para empezar, complete el tutorial [Protección de eventos de inicio de sesión de usuario con Azure Multi-Factor Authentication](tutorial-enable-azure-mfa.md) o [Uso de las detecciones de riesgos en los inicios de sesión de usuario para desencadenar Azure Multi-Factor Authentication](tutorial-risk-based-sspr-mfa.md).

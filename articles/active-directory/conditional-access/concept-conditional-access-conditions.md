---
title: 'Condiciones en una directiva de acceso condicional: Azure Active Directory'
description: Qué son las condiciones en una directiva de acceso condicional de Azure AD
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 07/02/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb
ms.collection: M365-identity-device-management
ms.openlocfilehash: a74fe2bf6b326dac782ac75418a7f4960e66501a
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87275010"
---
# <a name="conditional-access-conditions"></a>Acceso condicional: Condiciones

Dentro de una directiva de acceso condicional, un administrador puede usar señales de condiciones como el riesgo, la plataforma del dispositivo o la ubicación para mejorar sus decisiones de directivas. 

![Definición de una directiva de acceso condicional y especificación de condiciones](./media/concept-conditional-access-conditions/conditional-access-conditions.png)

Se pueden combinar varias condiciones para crear directivas de acceso condicional específicas y concretas.

Por ejemplo, al tener acceso a una aplicación confidencial, un administrador puede factorizar la información de riesgo de inicio de sesión de Identity Protection y la ubicación en su decisión de acceso, además de otros controles como autenticación multifactor.

## <a name="sign-in-risk"></a>Riesgo de inicio de sesión

En el caso de clientes con acceso a [Identity Protection](../identity-protection/overview-identity-protection.md), se puede evaluar el riesgo de inicio de sesión como parte de una directiva de acceso condicional. Un riesgo de inicio de sesión representa la probabilidad de que el propietario de la identidad no haya autorizado una solicitud de autenticación determinada. Puede encontrar más información sobre el riesgo de inicio de sesión en los artículos [¿Cuáles son los riesgos?](../identity-protection/concept-identity-protection-risks.md#sign-in-risk) e [Instrucciones: Configuración y habilitación de directivas de riesgo](../identity-protection/howto-identity-protection-configure-risk-policies.md).

## <a name="user-risk"></a>Riesgo de usuario 

En el caso de los clientes con acceso a [Identity Protection](../identity-protection/overview-identity-protection.md), el riesgo del usuario se puede evaluar como parte de una directiva de acceso condicional. El riesgo del usuario representa la probabilidad de que una identidad o una cuenta determinada esté en peligro. Puede encontrar más información sobre el riesgo del usuario en los artículos [¿Cuáles son los riesgos?](../identity-protection/concept-identity-protection-risks.md#user-risk) e [Instrucciones: Configuración y habilitación de directivas de riesgo](../identity-protection/howto-identity-protection-configure-risk-policies.md).

## <a name="device-platforms"></a>Plataformas de dispositivo

La plataforma de dispositivo se caracteriza por el sistema operativo que se ejecuta en un dispositivo. Azure AD identifica la plataforma mediante el uso de la información proporcionada por el dispositivo, como las cadenas de agente de usuario. Como las cadenas de agente de usuario se pueden modificar, esta información no se comprueba. La plataforma de dispositivo se debe usar junto con las directivas de cumplimiento de dispositivos de Microsoft Intune o como parte de una instrucción de bloque. El valor predeterminado es aplicarlo a todas las plataformas de dispositivo.

El acceso condicional de Azure AD admite las siguientes plataformas de dispositivo:

- Android
- iOS
- Windows Phone
- Windows
- macOS

Si bloquea la autenticación heredada con la condición **Otros clientes**, también puede establecer la condición de la plataforma del dispositivo.

## <a name="locations"></a>Ubicaciones

Al configurar la ubicación como una condición, las organizaciones pueden elegir incluir o excluir ubicaciones. Estas ubicaciones con nombre pueden incluir la información de red IPv4 pública, el país o la región, o incluso áreas desconocidas que no se asignan a países o regiones en concreto. Solo los intervalos IP se pueden marcar como una ubicación de confianza.

Al incluir **cualquier ubicación**, esta opción incluye cualquier dirección IP en Internet, no solo en las ubicaciones con nombre configuradas. Al seleccionar **cualquier ubicación**, los administradores pueden optar por excluir **todas las ubicaciones de confianza** o **seleccionadas**.

Por ejemplo, algunas organizaciones pueden optar por no requerir la autenticación multifactor cuando los usuarios están conectados a la red en una ubicación de confianza, como su oficina central física. Los administradores pueden crear una directiva que incluya cualquier ubicación, pero excluya las ubicaciones seleccionadas para las redes de la oficina central.

Para más información sobre las ubicaciones, consulte el artículo [¿Qué es la condición de ubicación del acceso condicional de Azure Active Directory?](location-condition.md)

## <a name="client-apps-preview"></a>Aplicaciones cliente (versión preliminar)

De forma predeterminada, las directivas de acceso condicional se aplican a aplicaciones basadas en explorador y aplicaciones que usan protocolos de autenticación modernos. Además de estas aplicaciones, los administradores pueden elegir incluir clientes de Exchange ActiveSync y otros clientes que utilicen protocolos heredados.

> [!NOTE]
> La opción de alternancia Sí/No se ha quitado de la condición de aplicaciones cliente para que sea más fácil ver qué aplicaciones cliente están seleccionadas. Esto no afecta a las aplicaciones cliente a las que se aplica una directiva existente.

- Browser
   - Estos incluyen aplicaciones basadas en web que usan protocolos como SAML, WS-Federation, OpenID Connect o servicios registrados como un cliente de OAuth Confidential.
- Aplicaciones móviles y aplicaciones de escritorio
   - Clientes de autenticación moderna
      - Esta opción incluye aplicaciones como las aplicaciones de teléfono y escritorio de Office.
   - Clientes de Exchange ActiveSync
      - De forma predeterminada, esto incluye cualquier uso del protocolo Exchange ActiveSync (EAS). Si se selecciona **Aplicar directiva solo en las plataformas compatibles**, se limitará a las plataformas compatibles, como iOS, Android y Windows.
      - Cuando la directiva bloquea el uso de Exchange ActiveSync, el usuario afectado recibirá un único mensaje de correo electrónico de cuarentena. Este mensaje de correo electrónico proporciona información sobre por qué está bloqueado e incluye instrucciones de corrección si es posible.
   - Otros clientes
      - Esta opción incluye clientes que usan protocolos de autenticación básicos/heredados que no admiten la autenticación moderna.
         - SMTP autenticado: usado por clientes POP e IMAP para enviar mensajes de correo electrónico.
         - Detección automática: usada por clientes Outlook y EAS para buscar y conectarse a buzones en Exchange Online.
         - Exchange Online PowerShell: se usa para conectarse a Exchange Online con PowerShell remoto. Si bloquea la autenticación básica para Exchange Online PowerShell, debe usar el módulo de Exchange Online PowerShell para conectarse. Para obtener instrucciones, consulte [Conexión a Exchange Online PowerShell con autenticación multifactor](/powershell/exchange/exchange-online/connect-to-exchange-online-powershell/mfa-connect-to-exchange-online-powershell).
         - Servicios web Exchange (EWS): una interfaz de programación que se usa en Outlook, Outlook para Mac y aplicaciones de terceros.
         - IMAP4: usado por clientes de correo electrónico IMAP.
         - MAPI sobre HTTP (MAPI/HTTP): usado por Outlook 2010 y versiones posteriores.
         - Libreta de direcciones sin conexión (OAB): una copia de las colecciones de listas de direcciones que Outlook descarga y usa.
         - Outlook en cualquier lugar (RPC a través de HTTP): usado por Outlook 2016 y versiones anteriores.
         - Servicio Outlook: usado por la aplicación de correo electrónico y calendario de Windows 10.
         - POP3: usado por clientes de correo electrónico POP.
         - Servicios web de creación de informes: se usan para recuperar datos de informes en Exchange Online.

Estas condiciones se suelen usar cuando se requiere un dispositivo administrado, se bloquea la autenticación heredada y se bloquea la aplicación web, pero se permiten aplicaciones móviles o de escritorio.

### <a name="supported-browsers"></a>Exploradores compatibles

Esta configuración funciona con todos los exploradores. Sin embargo, para satisfacer una directiva de dispositivo, como un requisito de dispositivo compatible, se admiten los sistemas operativos y exploradores siguientes:

| SO | Exploradores |
| :-- | :-- |
| Windows 10 | Microsoft Edge, Internet Explorer, Chrome |
| Windows 8 / 8.1 | Internet Explorer, Chrome |
| Windows 7 | Internet Explorer, Chrome |
| iOS | Microsoft Edge, Intune Managed Browser, Safari |
| Android | Microsoft Edge, Intune Managed Browser, Chrome |
| Windows Phone | Microsoft Edge, Internet Explorer |
| Windows Server 2019 | Microsoft Edge, Internet Explorer, Chrome |
| Windows Server 2016 | Internet Explorer |
| Windows Server 2012 R2 | Internet Explorer |
| Windows Server 2008 R2 | Internet Explorer |
| macOS | Chrome, Safari |

#### <a name="why-do-i-see-a-certificate-prompt-in-the-browser"></a>¿Por qué veo una solicitud de certificado en el explorador?

En Windows 7, iOS, Android y macOS, Azure AD identifica el dispositivo mediante un certificado de cliente que se aprovisiona cuando el dispositivo está registrado con Azure AD.  Cuando un usuario inicia sesión por primera vez a través del explorador, se le pide que seleccione el certificado. El usuario debe seleccionar este certificado antes de usar el explorador.

#### <a name="chrome-support"></a>Compatibilidad con Chrome

Para la compatibilidad con Chrome en **Windows 10 Creators Update (versión 1703)** o posterior, instale la [extensión de cuentas de Windows 10](https://chrome.google.com/webstore/detail/windows-10-accounts/ppnbnpeolgkicgegkbkbjmhlideopiji). Esta extensión es necesaria cuando una directiva de acceso condicional requiere detalles específicos del dispositivo.

Para implementar automáticamente esta extensión en los exploradores de Chrome, cree la siguiente clave del Registro:

- Ruta de acceso HKEY_LOCAL_MACHINE \Software\Policies\Google\Chrome\ExtensionInstallForcelist
- Nombre 1
- Tipo REG_SZ (Cadena)
- Datos ppnbnpeolgkicgegkbkbjmhlideopiji;https\://clients2.google.com/service/update2/crx

Para la compatibilidad con Chrome en **Windows 8.1 y 7**, cree la siguiente clave del Registro:

- Ruta de acceso HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Google\Chrome\AutoSelectCertificateForUrls
- Nombre 1
- Tipo REG_SZ (cadena)
- Datos {"pattern":"https://device.login.microsoftonline.com","filter":{"ISSUER":{"CN":"MS-Organization-Access"}}}

Estos exploradores admiten la autenticación de dispositivo, lo que permite identificar y validar el dispositivo con respecto a una directiva. Se produce un error en la comprobación del dispositivo si el explorador se ejecuta en modo privado.

### <a name="supported-mobile-applications-and-desktop-clients"></a>Clientes de escritorio y aplicaciones móviles compatibles

Las organizaciones pueden seleccionar **Aplicaciones móviles y clientes de escritorio** como aplicación cliente.

Esta configuración afecta a los intentos de acceso realizados desde las siguientes aplicaciones móviles y aplicaciones de escritorio:

| Aplicaciones cliente | Servicio de destino | Plataforma |
| --- | --- | --- |
| Aplicación de Dynamics CRM | Dynamics CRM | Windows 10, Windows 8.1, iOS y Android |
| Aplicación de Correo electrónico/Calendario/People, Outlook 2016, Outlook 2013 (con la autenticación moderna)| Office 365 Exchange Online | Windows 10 |
| Directiva de MFA y de ubicación para las aplicaciones. No se admiten las directivas basadas en dispositivos.| Cualquier servicio de aplicaciones de Mis aplicaciones | Android e iOS |
| Microsoft Teams Services: controla todos los servicios que admiten Microsoft Teams y todas sus aplicaciones cliente: escritorio de Windows, iOS, Android, WP y cliente web | Equipos de Microsoft | Windows 10, Windows 8.1, Windows 7, iOS, Android y macOS |
| Aplicaciones de Office 2016, Office 2013 (con autenticación moderna), [cliente de sincronización de OneDrive](/onedrive/enable-conditional-access) | Office 365 SharePoint Online | Windows 8.1, Windows 7, Windows 7 |
| Aplicaciones de Office 2016, aplicaciones universales de Office, Office 2013 (con autenticación moderna), [cliente de sincronización de OneDrive](/onedrive/enable-conditional-access) | Office 365 SharePoint Online | Windows 10 |
| Office 2016 (solo Word, Excel, PowerPoint y OneNote). | Office 365 SharePoint Online | macOS |
| Office 2019| Office 365 SharePoint Online | Windows 10, macOS |
| Aplicaciones móviles de Office | Office 365 SharePoint Online | Android, iOS |
| Aplicación de Yammer para Office | Yammer para Office 365 | Windows 10, iOS y Android |
| Outlook 2019 | Office 365 SharePoint Online | Windows 10, macOS |
| Outlook 2016 (Office para macOS) | Office 365 Exchange Online | macOS |
| Outlook 2016, Outlook 2013 (con autenticación moderna) y Skype Empresarial (con autenticación moderna) | Office 365 Exchange Online | Windows 8.1, Windows 7, Windows 7 |
| Aplicación móvil de Outlook | Office 365 Exchange Online | Android, iOS |
| Power BI app | Servicio Power BI | Windows 10, Windows 8.1, Windows 7, Android e iOS |
| Skype Empresarial | Office 365 Exchange Online| Android, iOS |
| Aplicación de Visual Studio Team Services | Visual Studio Team Services | Windows 10, Windows 8.1, Windows 7, iOS y Android |

### <a name="exchange-activesync-clients"></a>Clientes de Exchange ActiveSync

- Las organizaciones solo pueden seleccionar clientes de Exchange ActiveSync al asignar directivas a usuarios o grupos. La selección de **Todos los usuarios**, **Todos los usuarios externos e invitados** o **Roles del directorio** hará que todos los usuarios se bloqueen.
- Al crear una directiva asignada a los clientes de Exchange ActiveSync, **Office 365 Exchange Online** debe ser la única aplicación en la nube asignada a la directiva. 
- Las organizaciones pueden restringir el ámbito de esta directiva a plataformas específicas mediante la condición **Plataformas de dispositivo**.

Si el control de acceso que se asigna a la directiva usa **Requerir aplicación cliente aprobada**, se dirige al usuario para que instale y use el cliente móvil de Outlook. En el caso de que se requiera **Multi-factor Authentication**, los usuarios afectados están bloqueados, ya que la autenticación básica no admite la autenticación multifactor.

Para más información, consulte los siguientes artículos.

- [Bloqueo de la autenticación heredada en Azure AD con acceso condicional](block-legacy-authentication.md)
- [Uso obligatorio de aplicaciones cliente aprobadas para el acceso a aplicaciones en la nube mediante el acceso condicional](app-based-conditional-access.md)

### <a name="other-clients"></a>Otros clientes

Si selecciona **Otros clientes**, puede especificar una condición que afecta a las aplicaciones que usan la autenticación básica con protocolos de correo electrónico como IMAP, MAPI, POP, SMTP y aplicaciones de Office más antiguas que no usan la autenticación moderna.

## <a name="device-state-preview"></a>Estado del dispositivo (versión preliminar)

La condición del estado del dispositivo se puede usar para excluir dispositivos que están unidos a Azure AD híbrido o dispositivos marcados como compatibles con una directiva de cumplimiento de Microsoft Intune de las directivas de acceso condicional de una organización.

Por ejemplo, *Todos los usuarios* que acceden a la aplicación en la nube *Administración de Microsoft Azure*, incluido **Todos los estados de dispositivo**, excluidos **Unido a Azure AD híbrido de dispositivo** y **Dispositivo marcado como compatible** y para *Controles de acceso*, **Bloquear**. 
   - En este ejemplo se crea una directiva que solo permite el acceso a Administración de Microsoft Azure desde dispositivos unidos a Azure AD híbrido o dispositivos marcados como compatibles.

## <a name="next-steps"></a>Pasos siguientes

- [Acceso condicional: Concesión](concept-conditional-access-grant.md)

- [Directivas de acceso condicional habituales](concept-conditional-access-policy-common.md)

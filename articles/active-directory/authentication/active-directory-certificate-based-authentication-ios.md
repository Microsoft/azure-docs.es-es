---
title: 'Autenticación basada en certificados en iOS: Azure Active Directory'
description: Obtenga información acerca de los escenarios admitidos y los requisitos para configurar la autenticación basada en certificados para Azure Active Directory en las soluciones con dispositivos iOS.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 04/17/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.collection: M365-identity-device-management
ms.openlocfilehash: 584fb5c370fa0be629d057eb94dc4c2a8b9edc15
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88716442"
---
# <a name="azure-active-directory-certificate-based-authentication-on-ios"></a>Autenticación basada en certificados de Azure Active Directory en iOS

Para mejorar la seguridad, los dispositivos iOS pueden usar la autenticación basada en certificados (CBA) para autenticarse en Azure Active Directory (Azure AD) mediante el uso de un certificado de cliente en el dispositivo cuando se conecten a los servicios o aplicaciones siguientes:

* aplicaciones móviles de Office como Microsoft Outlook y Microsoft Word,
* clientes de Exchange ActiveSync (EAS).

Al usar certificados, no tendrá que escribir una combinación de nombre de usuario y contraseña en determinadas aplicaciones de correo electrónico y Microsoft Office del dispositivo móvil.

En este artículo se detallan los requisitos y los escenarios admitidos para configurar CBA en un dispositivo iOS. CBA para iOS está disponible en las nubes públicas de Azure, Microsoft Government Cloud, Microsoft Cloud Alemania y Microsoft Azure China 21Vianet.

## <a name="microsoft-mobile-applications-support"></a>Compatibilidad con aplicaciones móviles de Microsoft

| Aplicaciones | Soporte técnico |
| --- | --- |
| Aplicación Azure Information Protection |![La marca de verificación indica que hay compatibilidad con esta aplicación][1] |
| Portal de empresa de Intune |![La marca de verificación indica que hay compatibilidad con esta aplicación][1] |
| Equipos de Microsoft |![La marca de verificación indica que hay compatibilidad con esta aplicación][1] |
| Office (móvil) |![La marca de verificación indica que hay compatibilidad con esta aplicación][1] |
| OneNote |![La marca de verificación indica que hay compatibilidad con esta aplicación][1] |
| OneDrive |![La marca de verificación indica que hay compatibilidad con esta aplicación][1] |
| Outlook |![La marca de verificación indica que hay compatibilidad con esta aplicación][1] |
| Power BI |![La marca de verificación indica que hay compatibilidad con esta aplicación][1] |
| Skype Empresarial |![La marca de verificación indica que hay compatibilidad con esta aplicación][1] |
| Word, Excel y PowerPoint |![La marca de verificación indica que hay compatibilidad con esta aplicación][1] |
| Yammer |![La marca de verificación indica que hay compatibilidad con esta aplicación][1] |

## <a name="requirements"></a>Requisitos

Para usar CBA con iOS, se aplican los siguientes requisitos y consideraciones:

* La versión del sistema operativo del dispositivo debe ser iOS 9 o posterior.
* Se requiere Microsoft Authenticator para las aplicaciones de Office en iOS.
* Se debe crear una preferencia de identidad en la cadena de claves de macOS que incluya la dirección URL de autenticación del servidor de ADFS. Para obtener más información, vea [Crear una preferencia de identidad en Acceso a Llaveros en el Mac](https://support.apple.com/guide/keychain-access/create-an-identity-preference-kyca6343b6c9/mac).

Se aplican los siguientes requisitos y consideraciones de Servicios de federación de Active Directory (AD FS):

* El servidor de ADFS debe estar habilitado para la autenticación de certificados y usar la autenticación federada.
* El certificado debe utilizar el uso mejorado de clave (EKU) y contener el UPN del usuario en el *nombre alternativo del firmante (nombre principal de NT)* .

## <a name="configure-adfs"></a>Configuración de AD FS

Para que Azure AD revoque un certificado de cliente, el token de ADFS debe tener las siguientes notificaciones. Azure AD agrega estas notificaciones para el token de actualización, en caso de que estén disponibles en el token de ADFS (o en cualquier otro token SAML). Cuando hay que validar el token de actualización, esta información se utiliza para comprobar la revocación:

* `http://schemas.microsoft.com/ws/2008/06/identity/claims/<serialnumber>`: agregue el número de serie del certificado de cliente.
* `http://schemas.microsoft.com/2012/12/certificatecontext/field/<issuer>`: agregue la cadena del emisor del certificado de cliente.

Se recomienda actualizar también las páginas de error de ADFS de la organización con la información siguiente:

* El requisito de instalar Microsoft Authenticator en iOS.
* Instrucciones sobre cómo obtener un certificado de usuario

Para más información, consulte el artículo sobre la [personalización de la página de inicio de sesión de AD FS](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn280950(v=ws.11)).

## <a name="use-modern-authentication-with-office-apps"></a>Uso de la autenticación moderna con las aplicaciones de Office

Algunas aplicaciones de Office con la autenticación moderna habilitada envían `prompt=login` a Azure AD en su solicitud. De manera predeterminada, Azure AD traduce `prompt=login` en la solicitud para ADFS como `wauth=usernamepassworduri` (pide a ADFS que realice la autenticación de U y P) y `wfresh=0` (pide a ADFS que ignore el estado de SSO y realice una autenticación nueva). Si quiere habilitar la autenticación basada en certificados para estas aplicaciones, modifique el comportamiento predeterminado de Azure AD.

Para actualizar el comportamiento predeterminado, establezca "*PromptLoginBehavior*" en la configuración del dominio federado como "*Disabled*". Para realizar esta tarea, puede usar el cmdlet [MSOLDomainFederationSettings](/powershell/module/msonline/set-msoldomainfederationsettings?view=azureadps-1.0) tal como se muestra en el ejemplo siguiente:

```powershell
Set-MSOLDomainFederationSettings -domainname <domain> -PromptLoginBehavior Disabled
```

## <a name="support-for-exchange-activesync-clients"></a>Compatibilidad con clientes de Exchange ActiveSync

En iOS 9 o posterior, se admite el cliente de correo de iOS nativo. Para determinar si esta característica se admite en todas las demás aplicaciones de Exchange ActiveSync, póngase en contacto con el desarrollador de la aplicación.

## <a name="next-steps"></a>Pasos siguientes

Para configurar la autenticación basada en certificados en su entorno, consulte las instrucciones de [Introducción a la autenticación basada en certificados](active-directory-certificate-based-authentication-get-started.md).

<!--Image references-->
[1]: ./media/active-directory-certificate-based-authentication-ios/ic195031.png
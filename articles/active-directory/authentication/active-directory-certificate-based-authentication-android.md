---
title: 'Autenticación basada en certificados en Android: Azure Active Directory'
description: Obtenga información acerca de los escenarios admitidos y los requisitos para configurar la autenticación basada en certificados en las soluciones con dispositivos Android
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 11/21/2019
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: annaba
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7dd4c95c3c02f4b4a807b5238aa61e76ecb56e3e
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88716425"
---
# <a name="azure-active-directory-certificate-based-authentication-on-android"></a>Autenticación basada en certificados de Azure Active Directory en Android

Los dispositivos Android pueden usar la autenticación basada en certificados (CBA) para autenticarse en Azure Active Directory mediante el uso de un certificado de cliente en el dispositivo cuando se conecten a:

* aplicaciones móviles de Office como Microsoft Outlook y Microsoft Word,
* clientes de Exchange ActiveSync (EAS).

Al configurar esta función, no tendrá que escribir una combinación de nombre de usuario y contraseña en determinadas aplicaciones de correo electrónico y Microsoft Office de su dispositivo móvil.

En este tema se proporcionan los requisitos y los escenarios admitidos para configurar CBA en un dispositivo Android para usuarios de inquilinos de los planes Office 365 Enterprise, Empresa, Educación, Administración Pública para Estados Unidos, China y Alemania.

Esta característica se encuentra disponible como versión preliminar en los planes Office 365 US Government Defense y US Government Federal.

## <a name="microsoft-mobile-applications-support"></a>Compatibilidad con aplicaciones móviles de Microsoft

| Aplicaciones | Soporte técnico |
| --- | --- |
| Aplicación Azure Information Protection |![La marca de verificación indica que hay compatibilidad con esta aplicación][1] |
| Portal de empresa de Intune |![La marca de verificación indica que hay compatibilidad con esta aplicación][1] |
| Equipos de Microsoft |![La marca de verificación indica que hay compatibilidad con esta aplicación][1] |
| OneNote |![La marca de verificación indica que hay compatibilidad con esta aplicación][1] |
| OneDrive |![La marca de verificación indica que hay compatibilidad con esta aplicación][1] |
| Outlook |![La marca de verificación indica que hay compatibilidad con esta aplicación][1] |
| Power BI |![La marca de verificación indica que hay compatibilidad con esta aplicación][1] |
| Skype Empresarial |![La marca de verificación indica que hay compatibilidad con esta aplicación][1] |
| Word, Excel y PowerPoint |![La marca de verificación indica que hay compatibilidad con esta aplicación][1] |
| Yammer |![La marca de verificación indica que hay compatibilidad con esta aplicación][1] |

### <a name="implementation-requirements"></a>Requisitos de implementación

La versión del sistema operativo del dispositivo debe ser Android 5.0 (Lollipop) y superior.

Se debe configurar un servidor de federación.

Para que Azure Active Directory revoque un certificado de cliente, el token de ADFS debe tener las siguientes notificaciones:

* `http://schemas.microsoft.com/ws/2008/06/identity/claims/<serialnumber>` (el número de serie del certificado de cliente)
* `http://schemas.microsoft.com/2012/12/certificatecontext/field/<issuer>` (la cadena del emisor del certificado de cliente)

Azure Active Directory agrega estas notificaciones para el token de actualización, en caso de que estén disponibles en el token de ADFS (o en cualquier otro token SAML). Cuando hay que validar el token de actualización, esta información se utiliza para comprobar la revocación.

Se recomienda actualizar las páginas de error de ADFS de su organización con la información siguiente:

* El requisito de instalar Microsoft Authenticator en Android
* Instrucciones sobre cómo obtener un certificado de usuario

Para más información, consulte el artículo sobre la [personalización de las páginas de inicio de sesión de AD FS](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn280950(v=ws.11)).

Algunas aplicaciones de Office (con la autenticación moderna habilitada) envían "*prompt=login*" a Azure AD en su solicitud. De manera predeterminada, Azure AD traduce "*prompt=login*" en la solicitud para ADFS a "*wauth=usernamepassworduri*" (pide a ADFS que realice la autenticación de U y P) y "*wfresh=0*" (pide a ADFS que ignore el estado de SSO y realice una autenticación nueva). Si desea habilitar la autenticación basada en certificados para estas aplicaciones, es preciso que modifique el comportamiento predeterminado de Azure AD. Establezca "*PromptLoginBehavior*" en la configuración del dominio federado como "*Disabled*".
Para realizar esta tarea, puede usar el cmdlet [MSOLDomainFederationSettings](/powershell/module/msonline/set-msoldomainfederationsettings?view=azureadps-1.0):

`Set-MSOLDomainFederationSettings -domainname <domain> -PromptLoginBehavior Disabled`

## <a name="exchange-activesync-clients-support"></a>Compatibilidad con clientes de Exchange ActiveSync

Hay algunas aplicaciones de Exchange ActiveSync que son compatibles con Android 5.0 (Lollipop) o posterior. Para determinar si la aplicación de correo electrónico admite esta característica, póngase en contacto con el desarrollador de la aplicación.

## <a name="next-steps"></a>Pasos siguientes

Si quiere configurar la autenticación basada en certificados en su entorno, consulte las instrucciones de [Introducción a la autenticación basada en certificados en Android](active-directory-certificate-based-authentication-get-started.md).

<!--Image references-->
[1]: ./media/active-directory-certificate-based-authentication-android/ic195031.png
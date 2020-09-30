---
title: Renovación de certificados para los usuarios de Office 365 y Azure AD | Microsoft Docs
description: Este artículo explica a los usuarios de Office 365 cómo solucionar problemas con mensajes de correo electrónico que informan sobre la renovación de un certificado.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: curtand
ms.assetid: 543b7dc1-ccc9-407f-85a1-a9944c0ba1be
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 10/20/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 491c0aef74b44c0452b4d8d002a81928d80f360b
ms.sourcegitcommit: c94a177b11a850ab30f406edb233de6923ca742a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89276446"
---
# <a name="renew-federation-certificates-for-office-365-and-azure-active-directory"></a>Renovación de certificados de federación para Office 365 y Azure Active Directory
## <a name="overview"></a>Información general
Para una federación correcta entre Azure Active Directory (Azure AD) y Active Directory Federation Services (AD FS), los certificados usados por AD FS para firmar los tokens de seguridad en Azure AD deben coincidir con lo que está configurado en Azure AD. Cualquier error de coincidencia puede provocar la falta de confianza. Azure AD garantiza que esta información se mantiene sincronizada cuando se implementa AD FS y Proxy de aplicación web (para el acceso de extranet).

En este artículo se proporciona información adicional para administrar los certificados de firma de tokens y mantenerlos sincronizados con Azure AD en los casos siguientes:

* No implementa el Proxy de aplicación web y, por tanto, los metadatos de federación no están disponibles en la extranet.
* No utiliza la configuración predeterminada de AD FS para los certificados de firma de tokens.
* Utiliza un proveedor de identidades de terceros.

## <a name="default-configuration-of-ad-fs-for-token-signing-certificates"></a>Configuración predeterminada de AD FS para los certificados de firma de tokens
La firma de tokens y los certificados de descifrado de tokens son normalmente certificados autofirmados y son válidos durante un año. De forma predeterminada, AD FS incluye un proceso de renovación automática llamado **AutoCertificateRollover**. Si utiliza AD FS 2.0 o versiones posteriores, Office 365 y Azure AD actualizan automáticamente el certificado antes de que expire.

### <a name="renewal-notification-from-the-microsoft-365-admin-center-or-an-email"></a>Notificación de renovación desde el Centro de administración de Microsoft 365 o por correo electrónico
> [!NOTE]
> Si recibe un correo electrónico o una notificación del portal que le solicita renovar el certificado de Office, consulte la sección de [administración de cambios en los certificados de firma de tokens](#managecerts) para comprobar si es necesario realizar alguna acción. Microsoft tiene constancia de un posible problema que puede dar lugar a notificaciones enviadas para la renovación de certificados, incluso cuando no se requiere ninguna acción.
>
>

Azure AD intenta supervisar los metadatos de federación y actualizar el token de firma de certificados, tal como indican sus metadatos. 30 días antes de la expiración de los certificados de firma de tokens, Azure AD comprobará si los certificados nuevos están disponibles mediante el sondeo de los metadatos de federación.

* Si puede sondear los metadatos de federación y recuperar los nuevos certificados correctamente, no se enviará al usuario ninguna notificación por correo electrónico ni ninguna advertencia del Centro de administración de Microsoft 365.
* Si no se pueden recuperar los nuevos certificados de firma de tokens, ya sea porque los metadatos de federación no son accesibles o porque no está habilitada la sustitución automática de certificados, Azure AD emitirá una notificación por correo electrónico y una advertencia en el Centro de administración de Microsoft 365.

![Notificación del portal de Office 365](./media/how-to-connect-fed-o365-certs/notification.png)

> [!IMPORTANT]
> Si utiliza AD FS, para garantizar la continuidad del negocio, compruebe que los servidores tienen las actualizaciones siguientes para que no se produzcan errores de autenticación de los problemas conocidos. Esta acción mitiga los problemas conocidos del servidor de proxy de AD FS para esta renovación y períodos de renovación futuros:
>
> Server 2012 R2: [paquete acumulativo de mayo de 2014 de Windows Server](https://support.microsoft.com/kb/2955164)
>
> Server 2008 R2 y 2012: [Se produce un error en la autenticación a través de proxy en Windows Server 2012 o Windows Server 2008 R2 SP1](https://support.microsoft.com/kb/3094446)
>
>

## <a name="check-if-the-certificates-need-to-be-updated"></a>Compruebe si los certificados necesitan actualización <a name="managecerts"></a>
### <a name="step-1-check-the-autocertificaterollover-state"></a>Paso 1: Comprobación del estado del proceso AutoCertificateRollover
En el servidor de AD FS, abra Powershell. Compruebe que el valor AutoCertRollover está establecido en True.

```azurepowershell-interactive
Get-Adfsproperties
```

![AutoCertificateRollover](./media/how-to-connect-fed-o365-certs/autocertrollover.png)

>[!NOTE] 
>Si usa AD FS 2.0, ejecute primero Add-Pssnapin Microsoft.Adfs.Powershell.

### <a name="step-2-confirm-that-ad-fs-and-azure-ad-are-in-sync"></a>Paso 2: Confirmación de la sincronización de AD FS y Azure AD
En el servidor de AD FS, abra el símbolo del sistema de MSOnline PowerShell y conéctese a Azure AD.

> [!NOTE]
> Los cmdlets de MSOL son parte del módulo MSOnline PowerShell.
> Puede descargar el módulo MSOnline PowerShell directamente desde la Galería de PowerShell.
> 
>

```azurepowershell-interactive
Install-Module MSOnline
```

Conéctese a Azure AD mediante MSOnline PowerShell-Module.

```azurepowershell-interactive
Import-Module MSOnline
Connect-MsolService
```

Compruebe que los certificados configurados en AD FS y Azure AD confían en las propiedades para el dominio especificado.

```azurepowershell-interactive
Get-MsolFederationProperty -DomainName <domain.name> | FL Source, TokenSigningCertificate
```

![Get-MsolFederationProperty](./media/how-to-connect-fed-o365-certs/certsync.png)

Si las huellas digitales en ambas salidas coinciden, los certificados están sincronizados con Azure AD.

### <a name="step-3-check-if-your-certificate-is-about-to-expire"></a>Paso 3: Comprobación de si el certificado está a punto de expirar
En la salida de Get-MsolFederationProperty o Get-AdfsCertificate compruebe la fecha en "Not After". Si la fecha es inferior a 30 días, debe tomar medidas.

| AutoCertificateRollover | Certificados sincronizados con Azure AD | Los metadatos de federación están disponibles públicamente | Validez | Acción |
|:---:|:---:|:---:|:---:|:---:|
| Sí |Sí |Sí |- |No se requiere ninguna acción. Consulte [Renovación automática de certificados de firma de tokens](#autorenew). |
| Sí |No |- |Menos de 15 días |Renovar inmediatamente. Consulte [Renovación manual de certificados de firma de tokens](#manualrenew). |
| No |- |- |Menos de 30 días |Renovar inmediatamente. Consulte [Renovación manual de certificados de firma de tokens](#manualrenew). |

\[-] No importa

## <a name="renew-the-token-signing-certificate-automatically-recommended"></a>Renovación automática de certificados de firma de tokens (recomendado) <a name="autorenew"></a>
No es necesario realizar ningún paso manual si se cumplen las dos acciones siguientes:

* Se ha implementado el proxy de aplicación web, que puede habilitar el acceso a los metadatos de federación desde la extranet.
* Utiliza la configuración predeterminada de AD FS (AutoCertificateRollover está habilitado).

Compruebe lo siguiente para confirmar que se puede actualizar el certificado automáticamente:

**1. La propiedad AutoCertificateRollover de AD FS debe establecerse en True.** Esto indica que AD FS generará automáticamente nuevos certificados de descifrado de tokens y de firma de tokens antes de que expiren los antiguos.

**2. Los metadatos de federación de AD FS están disponibles públicamente.** Compruebe que se puede obtener acceso públicamente a los metadatos de federación, desplácese hasta la siguiente dirección URL desde un equipo de la red de Internet pública (fuera de la red corporativa):

https://(your_FS_name)/federationmetadata/2007-06/federationmetadata.xml

donde `(your_FS_name)` se reemplaza por el nombre de host de servicio de federación que usa su organización, por ejemplo, fs.contoso.com.  Si es capaz de comprobar ambos de estos valores correctamente, no tiene que hacer nada más.  

Ejemplo: `https://fs.contoso.com/federationmetadata/2007-06/federationmetadata.xml`
## <a name="renew-the-token-signing-certificate-manually"></a>Renovación manual de certificados de firma de tokens <a name="manualrenew"></a>
Puede optar por renovar manualmente los certificados de firma de tokens. Por ejemplo, los siguientes escenarios podrían funcionar mejor para la renovación manual:

* Los certificados de firma de tokens no son certificados autofirmados. La razón más común es que su organización administra certificados de AD FS inscritos de una entidad de certificación profesional.
* La seguridad de red no permite que los metadatos de federación estén disponibles públicamente.

En estos escenarios, cada vez que actualice los certificados de firma de tokens, debe actualizar también el dominio de Office 365 con el comando de PowerShell Update-MsolFederatedDomain.

### <a name="step-1-ensure-that-ad-fs-has-new-token-signing-certificates"></a>Paso 1: Comprobación de que AD FS tiene nuevos certificados de firma de tokens
**Configuración no predeterminada**

Si está en una configuración no predeterminada de AD FS donde la propiedad **AutoCertificateRollover** está establecida en **False**, probablemente esté utilizando certificados personalizados (no autofirmados). Para más información sobre cómo renovar los certificados de firma de tokens de AD FS, consulte [Requisitos de certificados para servidores federados](/windows-server/identity/ad-fs/design/certificate-requirements-for-federation-servers).

**Los metadatos de federación no están disponibles públicamente**

Por otro lado si la propiedad **AutoCertificateRollover** está establecida en **True** pero los metadatos de federación no están disponibles públicamente, asegúrese primero de que los certificados de firma de tokens nuevos los ha generado AD FS. Siga los pasos siguientes para confirmar que dispone de nuevos certificados de firma de tokens.

1. Compruebe que la sesión en el servidor de AD FS principal está iniciada.
2. Compruebe los certificados de firma actuales en AD FS abriendo una ventana de comandos de PowerShell y ejecutando el siguiente comando:

    PS C:\>Get-ADFSCertificate –CertificateType token-signing

   > [!NOTE]
   > Si usa AD FS 2.0, tendrá que ejecutar Add-Pssnapin Microsoft.Adfs.Powershell primero.
   >
   >
3. Examine la salida del comando en los certificados que se muestran. Si AD FS ha generado un nuevo certificado, debería ver dos certificados en la salida: uno para el que el valor de **IsPrimary** es **True** y la fecha de **NotAfter** está dentro de 5 días y uno para el que **IsPrimary** es **False** y **NotAfter** es aproximadamente un año en el futuro.
4. Si solo ve un certificado y la fecha de **NotAfter** está dentro de 5 días, deberá generar un nuevo certificado.
5. Para generar un nuevo certificado, ejecute el siguiente comando en un símbolo del sistema de PowerShell: `PS C:\>Update-ADFSCertificate –CertificateType token-signing`.
6. Compruebe la actualización mediante la ejecución del comando siguiente: PS C:\>Get-ADFSCertificate –CertificateType token-signing

Ahora deben aparecer dos certificados, uno de los cuales tiene una fecha de **NotAfter** de aproximadamente un año en el futuro y para el que el valor de **IsPrimary** es **False**.

### <a name="step-2-update-the-new-token-signing-certificates-for-the-office-365-trust"></a>Paso 2: Actualización de los nuevos certificados de firma de tokens para la relación de confianza de Office 365
Actualice Office 365 con los nuevos certificados de firma de token que se utilizarán para la relación de confianza, tal como se indica a continuación.

1. Abra el módulo Microsoft Azure Active Directory para Windows PowerShell.
2. Ejecute $cred=Get-Credential. Cuando este cmdlet le solicite las credenciales, escriba sus credenciales de cuenta de administrador de servicios en la nube.
3. Ejecute Connect-MsolService –Credential $cred. Este cmdlet le permite conectarse al servicio en la nube. Es necesario crear un contexto que le permita conectarse con el servicio en la nube antes de ejecutar cualquiera de los cmdlets adicionales que instala la herramienta.
4. Si ejecuta estos comandos en un equipo que no sea el servidor de federación principal de AD FS, ejecute Set-MSOLAdfscontext -Computer &lt;servidor principal de AD FS&gt;, donde&lt;servidor principal de AD FS&gt; es el nombre de dominio completo interno del servidor de AD FS principal. Este cmdlet crea un contexto que le permite conectarse a AD FS.
5. Ejecute Update-MSOLFederatedDomain –DomainName &lt;dominio&gt;. Este cmdlet actualiza la configuración de AD FS en el servicio en la nube y configura la relación de confianza entre los dos.

> [!NOTE]
> Si necesita admitir varios dominios de nivel superior, por ejemplo, contoso.com y fabrikam.com, debe utilizar el modificador **SupportMultipleDomain** con cualquier cmdlet. Para más información, consulte [Support for Multiple Top Level Domains](how-to-connect-install-multiple-domains.md)(Compatibilidad con varios dominios de nivel superior).
>


## <a name="repair-azure-ad-trust-by-using-azure-ad-connect"></a>Reparar la relación de confianza de Azure AD con Azure AD Connect <a name="connectrenew"></a>
Si ha configurado la granja de servidores de AD FS o la relación de confianza de Azure AD con Azure AD Connect, puede usar Azure AD Connect para detectar si es necesario tomar alguna medida para los certificados de firma de tokens. Si necesita renovar los certificados, puede utilizar Azure AD Connect para hacerlo.

Para más información, consulte la sección [Reparación de la confianza](how-to-connect-fed-management.md).

## <a name="ad-fs-and-azure-ad-certificate-update-steps"></a>Pasos de actualización de certificados de AD FS y Azure AD
Los certificados de firma de tokens son certificados X509 estándar que se usan para firmar de forma segura todos los tokens que emite el servidor de federación. Los certificados de descifrado de tokens son certificados X509 estándar que se usan para descifrar los tokens entrantes. 

De forma predeterminada, AD FS está configurado para generar certificados de firma y descifrado de tokens automáticamente, tanto en el momento de la configuración inicial como cuando los certificados se aproximan a su fecha de expiración.

Azure AD intenta recuperar un nuevo certificado de los metadatos de servicio de federación 30 días antes de la expiración del certificado actual. Si no hay disponible ningún certificado nuevo en ese momento, Azure AD seguirá supervisando los metadatos a intervalos diarios normales. En cuanto el nuevo certificado esté disponible en los metadatos, la configuración de federación del dominio se actualiza con la información del certificado nuevo. Puede usar `Get-MsolDomainFederationSettings` para comprobar si ver el nuevo certificado en NextSigningCertificate o SigningCertificate.

Para más información acerca de los certificados de firma de tokens en AD FS, consulte [Obtain and Configure Token Signing and Token Decryption Certificates for AD FS](/windows-server/identity/ad-fs/operations/configure-ts-td-certs-ad-fs) (Obtención y configuración de certificados de firma y descripción de tokens para AD FS)

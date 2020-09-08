---
title: 'Cumplimentación previa de la información de contacto para el autoservicio de restablecimiento de contraseña: Azure Active Directory'
description: Obtenga información sobre cómo rellenar previamente la información de contacto para los usuarios del autoservicio de restablecimiento de contraseña (SSPR) de Azure Active Directory de modo que puedan usar la característica sin completar un proceso de registro.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 07/17/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 6fa1c14509a558c1f91d07bf9a73a4ecc39e1413
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89068684"
---
# <a name="pre-populate-user-authentication-contact-information-for-azure-active-directory-self-service-password-reset-sspr"></a>Cumplimentación previa de la información de contacto de autenticación de usuarios para el autoservicio de restablecimiento de contraseña (SSPR) de Azure Active Directory

Para usar el autoservicio de restablecimiento de contraseña (SSPR) de Azure Active Directory (Azure AD), la información de contacto de autenticación del usuario debe estar presente. Algunas organizaciones hacen que los usuarios registren sus propios datos de autenticación. Otras organizaciones prefieren sincronizar con los datos de autenticación que ya existen en Active Directory Domain Services (AD DS). Estos datos sincronizados se ponen a disposición de Azure AD y SSPR sin necesidad de la interacción del usuario. Cuando los usuarios tengan que cambiar o restablecer su contraseña, pueden hacerlo aunque no hayan registrado previamente su información de contacto.

Puede rellenar previamente la información de contacto de autenticación si cumple los requisitos siguientes:

* Ha dado un formato correcto a los datos en su directorio local.
* Ha configurado [Azure AD Connect](../hybrid/how-to-connect-install-express.md) para su inquilino de Azure AD.

Todos los números de teléfono deben tener el formato *+CódigoPaís NúmeroTelefónico*, por ejemplo: *+1 4251234567*.

> [!NOTE]
> Debe haber un espacio entre el código de país y el número de teléfono.
>
> El restablecimiento de contraseña no admite extensiones telefónicas. Incluso con el formato *+1 4251234567X12345*, las extensiones se quitan antes de hacer la llamada.

## <a name="fields-populated"></a>Campos rellenos

Si usa la configuración predeterminada en Azure AD Connect, se realizan las siguientes asignaciones para rellenar la información de contacto de autenticación para SSPR:

| Active Directory local | Azure AD     |
|------------------------------|--------------|
| telephoneNumber              | Teléfono del trabajo |
| mobile                       | Teléfono móvil |

Después de que un usuario verifica el número de teléfono móvil, el campo *Teléfono* de la **Información de contacto para la autenticación** de Azure AD se rellena también con dicho número.

## <a name="authentication-contact-info"></a>Información de contacto para la autenticación

En la página **Métodos de autenticación** de un usuario de Azure AD en Azure Portal, un administrador global puede establecer manualmente la información del contacto de autenticación, tal como se muestra en la siguiente captura de pantalla de ejemplo:

![Información de contacto de autenticación en un usuario en Azure AD][Contact]

Las siguientes consideraciones se aplican a esta información de contacto de autenticación:

* Si se rellena el campo *Teléfono* y el *Teléfono móvil* está habilitado en la directiva de SSPR, el usuario ve dicho número en la página de registro de restablecimiento de la contraseña y durante el flujo de trabajo de restablecimiento de la contraseña.
* El campo de *Teléfono alternativo* no se utiliza para restablecer la contraseña.
* Si se rellena el campo de *Correo electrónico* y el *Correo electrónico* está habilitado en la directiva de SSPR, el usuario ve dicho correo electrónico en la página de registro de restablecimiento de la contraseña y durante el flujo de trabajo de restablecimiento de la contraseña.
* Si se rellena el campo de *Correo electrónico alternativo* y el *Correo electrónico* está habilitado en la directiva de SSPR, el usuario no verá dicho correo electrónico en la página de registro de restablecimiento de la contraseña, pero sí durante el flujo de trabajo de restablecimiento de la contraseña.

## <a name="security-questions-and-answers"></a>Preguntas y respuestas de seguridad

Las preguntas y respuestas de seguridad se almacenan de forma segura en el inquilino de Azure AD y los usuarios pueden tener acceso a ellas solo a través del [portal de registro de SSPR](https://aka.ms/ssprsetup). Los administradores no pueden ver, establecer ni modificar el contenido de las preguntas y respuestas de otros usuarios.

## <a name="what-happens-when-a-user-registers"></a>Qué ocurre cuando se registra un usuario

Cuando se registre un usuario, la página de registro establecerá los campos siguientes:

* **Teléfono de autenticación**
* **Correo electrónico de autenticación**
* **Preguntas y respuestas de seguridad**

Si especificó un valor para *Teléfono móvil* o *Correo electrónico alternativo*, los usuarios podrán usarlos inmediatamente para restablecer sus contraseñas, aunque no se hayan registrado para el servicio.

Además, los usuarios ven esos valores cuando se registran por primera vez, y pueden modificarlos si lo desean. Una vez que se registran correctamente, dichos valores se conservan en los campos *Teléfono de autenticación* y *Correo electrónico de autenticación*, respectivamente.

## <a name="set-and-read-the-authentication-data-through-powershell"></a>Establecimiento y lectura de datos de autenticación mediante PowerShell

Los siguientes campos pueden establecerse mediante PowerShell:

* *Correo electrónico alternativo*
* *Teléfono móvil*
* *Teléfono del trabajo*
    * Solo puede establecerse si no se sincroniza un directorio local.

> [!IMPORTANT]
> Existe una falta de paridad conocida en las características de comandos entre PowerShell v1 y PowerShell v2. La [API de REST de Microsoft Graph (beta) para los métodos de autenticación](/graph/api/resources/authenticationmethods-overview) es el foco actual de ingeniería para proporcionar una interacción moderna.

### <a name="use-powershell-version-1"></a>Uso de PowerShell versión 1

Para empezar, [descargue e instale el módulo Azure AD para PowerShell](/previous-versions/azure/jj151815(v=azure.100)#bkmk_installmodule). Una vez instalado, siga estos pasos para configurar cada campo.

#### <a name="set-the-authentication-data-with-powershell-version-1"></a>Establecimiento de datos de autenticación con PowerShell versión 1

```PowerShell
Connect-MsolService

Set-MsolUser -UserPrincipalName user@domain.com -AlternateEmailAddresses @("email@domain.com")
Set-MsolUser -UserPrincipalName user@domain.com -MobilePhone "+1 4251234567"
Set-MsolUser -UserPrincipalName user@domain.com -PhoneNumber "+1 4252345678"

Set-MsolUser -UserPrincipalName user@domain.com -AlternateEmailAddresses @("email@domain.com") -MobilePhone "+1 4251234567" -PhoneNumber "+1 4252345678"
```

#### <a name="read-the-authentication-data-with-powershell-version-1"></a>Lectura de datos de autenticación con PowerShell versión 1

```PowerShell
Connect-MsolService

Get-MsolUser -UserPrincipalName user@domain.com | select AlternateEmailAddresses
Get-MsolUser -UserPrincipalName user@domain.com | select MobilePhone
Get-MsolUser -UserPrincipalName user@domain.com | select PhoneNumber

Get-MsolUser | select DisplayName,UserPrincipalName,AlternateEmailAddresses,MobilePhone,PhoneNumber | Format-Table
```

#### <a name="read-the-authentication-phone-and-authentication-email-options"></a>Lectura de las opciones de teléfono de autenticación y correo electrónico de autenticación

Para leer el **teléfono de autenticación** y el **correo electrónico de autenticación** cuando emplee PowerShell versión 1, utilice los siguientes comandos:

```PowerShell
Connect-MsolService
Get-MsolUser -UserPrincipalName user@domain.com | select -Expand StrongAuthenticationUserDetails | select PhoneNumber
Get-MsolUser -UserPrincipalName user@domain.com | select -Expand StrongAuthenticationUserDetails | select Email
```

### <a name="use-powershell-version-2"></a>Uso de PowerShell versión 2

Para empezar, [descargue e instale el módulo de PowerShell de Azure AD versión 2](/powershell/module/azuread/?view=azureadps-2.0).

Para instalar rápidamente desde versiones recientes de PowerShell compatibles con `Install-Module`, ejecute los siguientes comandos. La primera línea comprueba si el módulo ya está instalado:

```PowerShell
Get-Module AzureADPreview
Install-Module AzureADPreview
Connect-AzureAD
```

Una vez instalado el módulo, siga estos pasos para configurar cada campo.

#### <a name="set-the-authentication-data-with-powershell-version-2"></a>Establecimiento de datos de autenticación con PowerShell versión 2

```PowerShell
Connect-AzureAD

Set-AzureADUser -ObjectId user@domain.com -OtherMails @("email@domain.com")
Set-AzureADUser -ObjectId user@domain.com -Mobile "+1 4251234567"
Set-AzureADUser -ObjectId user@domain.com -TelephoneNumber "+1 4252345678"

Set-AzureADUser -ObjectId user@domain.com -OtherMails @("emails@domain.com") -Mobile "+1 4251234567" -TelephoneNumber "+1 4252345678"
```

#### <a name="read-the-authentication-data-with-powershell-version-2"></a>Lectura de datos de autenticación con PowerShell versión 2

```PowerShell
Connect-AzureAD

Get-AzureADUser -ObjectID user@domain.com | select otherMails
Get-AzureADUser -ObjectID user@domain.com | select Mobile
Get-AzureADUser -ObjectID user@domain.com | select TelephoneNumber

Get-AzureADUser | select DisplayName,UserPrincipalName,otherMails,Mobile,TelephoneNumber | Format-Table
```

## <a name="next-steps"></a>Pasos siguientes

Una vez que la información de contacto de autenticación se ha rellenado previamente para los usuarios, complete el siguiente tutorial para habilitar el autoservicio de restablecimiento de contraseña:

> [!div class="nextstepaction"]
> [Habilitación del autoservicio de restablecimiento de contraseña de Azure AD](tutorial-enable-sspr.md)

[Contact]: ./media/howto-sspr-authenticationdata/user-authentication-contact-info.png "Los administradores globales pueden modificar la información de contacto de autenticación de un usuario"

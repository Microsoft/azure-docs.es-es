---
title: 'Inicio rápido: Incorporación de un usuario invitado con PowerShell (Azure AD)'
description: En este tutorial, obtendrá información sobre cómo usar PowerShell para enviar una invitación a un usuario externo de colaboración de Azure AD B2B.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: quickstart
ms.date: 08/28/2018
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: b0faaf0394bddc2e443afc194bbd0ecef72625f9
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87910596"
---
# <a name="quickstart-add-a-guest-user-with-powershell"></a>Inicio rápido: Incorporación de un usuario invitado con PowerShell

Hay muchas maneras de invitar a los asociados externos a sus aplicaciones y servicios con la colaboración de Azure Active Directory B2B. En el tutorial anterior, vimos cómo agregar usuarios invitados directamente en el portal de administración de Azure Active Directory. También puede usar PowerShell para agregar usuarios invitados, uno a uno o de forma masiva. En este tutorial, usará el comando New-AzureADMSInvitation para agregar un usuario invitado a su inquilino de Azure.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar. 

## <a name="prerequisites"></a>Prerrequisitos

### <a name="install-the-latest-azureadpreview-module"></a>Instalación del último módulo de AzureADPreview
Asegúrese de que instala la última versión de Azure AD PowerShell para el módulo Graph (AzureADPreview). 

En primer lugar, compruebe qué módulos ha instalado. Abra Windows PowerShell como un usuario con privilegios elevados (Ejecutar como administrador) y ejecute el comando siguiente:
 
```powershell  
Get-Module -ListAvailable AzureAD*
```

Si se muestra el módulo AzureADPreview sin ningún mensaje que indica que hay una versión posterior, significa que está listo. De lo contrario, en función de la salida, realice una de las siguientes acciones:

- Si no se devuelve ningún resultado, ejecute el siguiente comando para instalar el módulo AzureADPreview:
  
   ```powershell  
   Install-Module AzureADPreview
   ```
- Si solo se muestra el módulo AzureAD en los resultados, ejecute los comandos siguientes para instalar el módulo AzureADPreview: 

   ```powershell 
   Uninstall-Module AzureAD 
   Install-Module AzureADPreview 
   ```
- Si solo se muestra el módulo AzureADPreview en los resultados, pero recibe un mensaje que indica que hay una versión posterior, ejecute los comandos siguientes para actualizar el módulo: 

   ```powershell 
   Uninstall-Module AzureADPreview 
   Install-Module AzureADPreview 
  ```

Puede que reciba un mensaje que indica que está instalando el módulo desde un repositorio de confianza. Esto se produce si el repositorio de PSGallery no se ha establecido previamente como un repositorio de confianza. Presione **Y** para instalar el módulo.

### <a name="get-a-test-email-account"></a>Obtención de una cuenta de correo electrónico de prueba

Necesita una cuenta de correo electrónico de prueba a la que poder enviar la invitación. La cuenta debe estar fuera de su organización. Puede usar cualquier tipo de cuenta, incluida una cuenta social como una dirección de gmail.com o outlook.com.

## <a name="sign-in-to-your-tenant"></a>Inicio de sesión en su inquilino

Ejecute el siguiente comando para conectarse al dominio del inquilino:

```powershell
Connect-AzureAD -TenantDomain "<Tenant_Domain_Name>"
```
Por ejemplo, `Connect-AzureAD -TenantDomain "contoso.onmicrosoft.com"`.

Cuando se le solicite, escriba las credenciales.

## <a name="send-an-invitation"></a>Envío de una invitación

1. Para enviar una invitación a su cuenta de correo electrónico de prueba, ejecute el siguiente comando de PowerShell (reemplace **"Sanda"** y **sanda\@fabrikam.com** por la dirección de correo electrónico y el nombre de la cuenta de correo electrónico de prueba): 

   ```powershell
   New-AzureADMSInvitation -InvitedUserDisplayName "Sanda" -InvitedUserEmailAddress sanda@fabrikam.com -InviteRedirectURL https://myapps.microsoft.com -SendInvitationMessage $true
   ```
2. El comando envía una invitación a la dirección de correo electrónico especificada. Consulte la salida, que debería ser similar a la siguiente:

   ![Salida de PowerShell que muestra la aceptación pendiente del usuario](media/quickstart-invite-powershell/powershell-azureadmsinvitation-result.png)

## <a name="verify-the-user-exists-in-the-directory"></a>Verificación de la existencia del usuario en el directorio

1. Para verificar que se agregó el usuario invitado a Azure AD, ejecute el siguiente comando:
 
   ```powershell
   Get-AzureADUser -Filter "UserType eq 'Guest'"
   ```
3. Consulte la salida para asegurarse de que el usuario invitado aparece en la lista, con un nombre principal de usuario (UPN) con el formato *emailaddress*#EXT#\@*domain*. Por ejemplo, *sanda_fabrikam.com#EXT#\@contoso.onmicrosoft.com*, donde contoso.onmicrosoft.com es la organización desde la que se enviaron las invitaciones.

   ![Salida de PowerShell que muestra que se agregó el usuario invitado](media/quickstart-invite-powershell/powershell-guest-user-added.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no sea necesaria, puede eliminar la cuenta de usuario de prueba del directorio. Ejecute el siguiente comando para eliminar una cuenta de usuario:

```powershell
 Remove-AzureADUser -ObjectId "<UPN>"
```
Por ejemplo: `Remove-AzureADUser -ObjectId "sanda_fabrikam.com#EXT#@contoso.onmicrosoft.com"`


## <a name="next-steps"></a>Pasos siguientes
En este tutorial, invitó y agregó a un único usuario invitado a su directorio con PowerShell. A continuación, obtenga información sobre cómo invitar a usuarios en masa mediante PowerShell.

> [!div class="nextstepaction"]
> [Tutorial: Invitación en masa a usuarios de colaboración de Azure AD B2B](tutorial-bulk-invite.md)

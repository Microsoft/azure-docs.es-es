---
title: Tutorial para la invitación en masa a usuarios de colaboración B2B - Azure Active Directory | Microsoft Docs
description: En este tutorial aprenderá a usar PowerShell y un archivo CSV para enviar invitaciones en bloque a usuarios invitados externos de colaboración de Azure AD B2B.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: tutorial
ms.date: 02/11/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.collection: M365-identity-device-management
ms.openlocfilehash: 89e24d9ff76184c36aee5c14f15f9713b30f6f1d
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87907057"
---
# <a name="tutorial-use-powershell-to-bulk-invite-azure-ad-b2b-collaboration-users"></a>Tutorial: Uso de PowerShell para invitar en bloque a usuarios de colaboración de Azure AD B2B

Si usas la colaboración de Azure Active Directory (Azure AD) B2B para trabajar con asociados externos, puede invitar a varios usuarios a su organización al mismo tiempo. En este tutorial, obtendrá información sobre cómo usar PowerShell para enviar invitaciones de forma masiva a usuarios externos. En particular, haga lo siguiente:

> [!div class="checklist"]
> * Prepare un archivo de valores separados por comas (.csv) con la información de usuario.
> * Ejecute un script de PowerShell para enviar invitaciones.
> * Verifique si los usuarios se agregaron al directorio.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar. 

## <a name="prerequisites"></a>Prerrequisitos

### <a name="install-the-latest-azureadpreview-module"></a>Instalación del último módulo de AzureADPreview

Asegúrese de que instala la última versión de Azure AD PowerShell para el módulo Graph (AzureADPreview). 

En primer lugar, compruebe qué módulos ha instalado. Abra Windows PowerShell como un usuario con privilegios elevados (Ejecutar como administrador) y ejecute el comando siguiente:

```powershell
Get-Module -ListAvailable AzureAD*
```

En función de la salida, realice una de las siguientes acciones:

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

Puede recibir un mensaje que indica que está instalando el módulo desde un repositorio de confianza. Esto se produce si el repositorio de PSGallery no se ha establecido previamente como un repositorio de confianza. Presione **Y** para instalar el módulo.

### <a name="get-test-email-accounts"></a>Obtención de cuentas de correo electrónico de prueba

Necesita dos o más cuentas de correo electrónico de prueba a las que poder enviar las invitaciones. Las cuentas deben estar fuera de su organización. Puede usar cualquier tipo de cuenta, incluidas las cuentas sociales como las direcciones de gmail.com o outlook.com.

## <a name="prepare-the-csv-file"></a>Preparación del archivo .csv

En Microsoft Excel, cree un archivo .csv con la lista de nombres de usuarios invitados y las direcciones de correo electrónico. Asegúrese de incluir los encabezados de columna **Nombre** e **InvitedUserEmailAddress**.

Por ejemplo, cree una hoja de cálculo con el formato siguiente:

![Salida de PowerShell que muestra la aceptación pendiente del usuario](media/tutorial-bulk-invite/AddUsersExcel.png)

Guarde el archivo como **C:\BulkInvite\Invitations.csv**. 

Si no tiene Excel, puede crear un archivo .csv en cualquier editor de texto como el Bloc de notas. Separe cada valor con una coma y cada fila con una línea nueva. 

## <a name="sign-in-to-your-tenant"></a>Inicio de sesión en su inquilino

Ejecute el siguiente comando para conectarse al dominio del inquilino:

```powershell
Connect-AzureAD -TenantDomain "<Tenant_Domain_Name>"
```

Por ejemplo, `Connect-AzureAD -TenantDomain "contoso.onmicrosoft.com"`.

Cuando se le solicite, escriba las credenciales.

## <a name="send-bulk-invitations"></a>Envío de invitaciones en masa

Para enviar las invitaciones, ejecute el siguiente script de PowerShell (donde **c:\bulkinvite\invitations.csv** es la ruta de acceso al archivo .csv):

```powershell
$invitations = import-csv c:\bulkinvite\invitations.csv

$messageInfo = New-Object Microsoft.Open.MSGraph.Model.InvitedUserMessageInfo

$messageInfo.customizedMessageBody = "Hello. You are invited to the Contoso organization."

foreach ($email in $invitations)
   {New-AzureADMSInvitation `
      -InvitedUserEmailAddress $email.InvitedUserEmailAddress `
      -InvitedUserDisplayName $email.Name `
      -InviteRedirectUrl https://myapps.microsoft.com `
      -InvitedUserMessageInfo $messageInfo `
      -SendInvitationMessage $true
   }
```

El script enviará una invitación a las direcciones de correo electrónico del archivo Invitations.csv. Debería ver una salida similar a la siguiente para cada usuario:

![Salida de PowerShell que muestra la aceptación pendiente del usuario](media/tutorial-bulk-invite/B2BBulkImport.png)

## <a name="verify-users-exist-in-the-directory"></a>Verificación de la existencia de los usuarios en el directorio

Para verificar que se agregaron los usuarios invitados a Azure AD, ejecute el siguiente comando:

```powershell
 Get-AzureADUser -Filter "UserType eq 'Guest'"
```

Debería ver los usuarios invitados en la lista, con un nombre principal de usuario (UPN) con el formato *direccióndecorreoelectrónico*#EXT#\@*dominio*. Por ejemplo, *lstokes_fabrikam.com#EXT#\@contoso.onmicrosoft.com*, donde contoso.onmicrosoft.com es la organización desde la que se enviaron las invitaciones.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no sean necesarias, puede eliminar las cuentas de usuario de prueba del directorio. Ejecute el siguiente comando para eliminar una cuenta de usuario:

```powershell
 Remove-AzureADUser -ObjectId "<UPN>"
```

Por ejemplo: `Remove-AzureADUser -ObjectId "lstokes_fabrikam.com#EXT#@contoso.onmicrosoft.com"`

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha enviado invitaciones en masa a usuarios invitados fuera de la organización. A continuación, obtenga información sobre cómo funciona el proceso de canje de la invitación.

> [!div class="nextstepaction"]
> [Obtenga información sobre el proceso de canje de la invitación de colaboración de Azure AD B2B](redemption-experience.md)

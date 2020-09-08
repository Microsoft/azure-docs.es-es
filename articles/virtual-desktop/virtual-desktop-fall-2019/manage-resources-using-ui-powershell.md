---
title: 'Implementación de una herramienta de administración para Windows Virtual Desktop (clásico) mediante la entidad de servicio: Azure'
description: Cómo implementar la herramienta de administración de Windows Virtual Desktop (clásico) mediante PowerShell.
author: Heidilohr
ms.topic: how-to
ms.date: 03/30/2020
ms.author: helohr
ms.custom: devx-track-azurepowershell
manager: lizross
ms.openlocfilehash: d7219751d584eb458cded9f4e30cccb1439dfa1b
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89069024"
---
# <a name="deploy-a-windows-virtual-desktop-classic-management-tool-with-powershell"></a>Implementación de una herramienta de administración de Windows Virtual Desktop (clásico) con PowerShell

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop (clásico), que no admite objetos de Windows Virtual Desktop para Azure Resource Manager.

En este artículo se describe cómo implementar la herramienta de administración mediante el uso de PowerShell.

## <a name="important-considerations"></a>Consideraciones importantes

La suscripción de cada inquilino de Azure Active Directory (Azure AD) necesita su propia implementación independiente de la herramienta de administración. Esta herramienta no admite escenarios de negocio a negocio (B2B) de Azure AD.

Esta herramienta de administración es un ejemplo. Microsoft proporcionará actualizaciones de calidad y seguridad importantes. [El código fuente está disponible en GitHub](https://github.com/Azure/RDS-Templates/tree/master/wvd-templates/wvd-management-ux/deploy). Tanto si es un cliente como un partner, le recomendamos que personalice la herramienta para satisfacer sus necesidades empresariales.

Los siguientes exploradores son compatibles con la herramienta de administración:

- Google Chrome 68 o una versión posterior
- Microsoft Edge 40.15063 o una versión posterior
- Mozilla Firefox 52.0 o una versión posterior
- Safari 10 o una versión posterior (solo macOS)

## <a name="what-you-need-to-deploy-the-management-tool"></a>Requisitos para implementar la herramienta de administración

Antes de implementar la herramienta de administración, necesitará que un usuario de Azure Active Directory (Azure AD) cree un registro de aplicaciones e implemente la interfaz de usuario de administración. Dicho usuario debe:

- Tener los permisos para crear recursos en la suscripción de Azure.
- Tener los permisos para crear una aplicación de Azure AD. Siga los pasos que se describen en la sección [Permisos necesarios](../../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app) para comprobar si el usuario tiene los permisos que necesita.

Después de implementar y configurar la herramienta de administración, se recomienda pedir a un usuario que inicie la interfaz de usuario de administración para asegurarse de que todo funciona correctamente. El usuario que inicie la interfaz de usuario de administración deberá tener una asignación de roles que le permita ver o editar el inquilino de Windows Virtual Desktop.

## <a name="set-up-powershell"></a>Configurar PowerShell

Para empezar, inicie sesión en los módulos Az y Azure AD para PowerShell. Siga estos pasos para iniciar sesión:

1. Abra PowerShell como administrador y navegue al directorio donde guardó los scripts de PowerShell.
2. Inicie sesión en Azure con una cuenta que tenga permisos de propietario o de colaborador en la suscripción de Azure que usará para crear la herramienta de administración, para ello, ejecute el siguiente cmdlet:

    ```powershell
    Login-AzAccount
    ```

3. Ejecute este otro cmdlet para iniciar sesión en Azure AD con la misma cuenta que usó para el módulo Az de PowerShell:

    ```powershell
    Connect-AzureAD
    ```

4. Después, vaya a la carpeta donde guardó los dos scripts de PowerShell desde el repositorio RDS-Templates de GitHub.

Mantenga abierta la ventana de PowerShell que usó para iniciar sesión a fin de ejecutar otros cmdlets de PowerShell mientras tiene la sesión iniciada.

## <a name="create-an-azure-active-directory-app-registration"></a>Creación de un registro de aplicaciones de Azure Active Directory

Para poder implementar y configurar correctamente la herramienta de administración, primero debe descargar los siguientes scripts de PowerShell desde el [repositorio RDS-templates de GitHub](https://github.com/Azure/RDS-Templates/tree/master/wvd-templates/wvd-management-ux/deploy/scripts).

```powershell
Set-Location -Path "c:\temp"
$uri = "https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-management-ux/deploy/scripts/createWvdMgmtUxAppRegistration.ps1"
Invoke-WebRequest -Uri $uri -OutFile ".\createWvdMgmtUxAppRegistration.ps1"
$uri = "https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-management-ux/deploy/scripts/updateWvdMgmtUxApiUrl.ps1"
Invoke-WebRequest -Uri $uri -OutFile ".\updateWvdMgmtUxApiUrl.ps1"
```

Ejecute los siguientes comandos para crear el registro de aplicaciones con los permisos de API necesarios:

```powershell
$appName = Read-Host -Prompt "Enter a unique name for the management tool's app registration. The name can't contain spaces or special characters."
$azureSubscription = Get-AzSubscription | Out-GridView -PassThru
$subscriptionId = $azureSubscription.Id
Get-AzSubscription -SubscriptionId $subscriptionId | Select-AzSubscription

.\createWvdMgmtUxAppRegistration.ps1 -AppName $appName -SubscriptionId $subscriptionId
```

Ahora que ha completado el registro de aplicaciones de Azure AD, puede implementar la herramienta de administración.

## <a name="deploy-the-management-tool"></a>Implementación de la herramienta de administración

Ejecute los siguientes comandos de PowerShell para implementar la herramienta de administración y para asociarla a la entidad de servicio que acaba de crear:

```powershell
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$templateParameters = @{}
$templateParameters.Add('isServicePrincipal', $true)
$templateParameters.Add('azureAdminUserPrincipalNameOrApplicationId', $ServicePrincipalCredentials.UserName)
$templateParameters.Add('azureAdminPassword', $servicePrincipalCredentials.Password)
$templateParameters.Add('applicationName', $appName)

Get-AzSubscription -SubscriptionId $subscriptionId | Select-AzSubscription
New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
    -TemplateUri "https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-management-ux/deploy/mainTemplate.json" `
    -TemplateParameterObject $templateParameters `
    -Verbose
```

Después de crear la aplicación web, debe agregar un URI de redirección a la aplicación de Azure AD para que los usuarios inicien sesión correctamente.

## <a name="set-the-redirect-uri"></a>Establecimiento del URI de redirección

Ejecute los siguientes comandos de PowerShell para recuperar la dirección URL de la aplicación web y establecerla como el URI de redirección de autenticación (también conocido como dirección URL de respuesta):

```powershell
$webApp = Get-AzWebApp -ResourceGroupName $resourceGroupName -Name $appName
$redirectUri = "https://" + $webApp.DefaultHostName + "/"
Get-AzureADApplication -All $true | where { $_.AppId -match $servicePrincipalCredentials.UserName } | Set-AzureADApplication -ReplyUrls $redirectUri
```

Ahora que ha agregado un URI de redirección, debe actualizar la dirección URL de la API para que la herramienta de administración pueda interactuar con el servicio back-end de la API.

## <a name="update-the-api-url-for-the-web-application"></a>Actualización de la dirección URL de la API de la aplicación web

Ejecute el siguiente script para actualizar la configuración de la dirección URL de la API en el front-end de la aplicación web:

```powershell
.\updateWvdMgmtUxApiUrl.ps1 -AppName $appName -SubscriptionId $subscriptionId
```

Ahora que ha configurado completamente la aplicación web de la herramienta de administración, es el momento de comprobar la configuración de la aplicación de Azure AD y dar su consentimiento.

## <a name="verify-the-azure-ad-application-and-provide-consent"></a>Comprobación de la aplicación de Azure AD y otorgación de consentimiento

Para comprobar la configuración de la aplicación de Azure AD y otorgar su consentimiento:

1. Abra el explorador de Internet e inicie sesión en [Azure Portal](https://portal.azure.com/) con la cuenta administrativa.
2. En la barra de búsqueda, situada en la parte superior de Azure Portal, busque **Registros de aplicaciones** y seleccione el elemento en **Servicios**.
3. Seleccione **Todas las aplicaciones** y busque el nombre de aplicación único que proporcionó en el script de PowerShell en el paso [Creación de un registro de aplicaciones de Azure Active Directory](#create-an-azure-active-directory-app-registration).
4. En el panel de la izquierda del explorador, seleccione **Autenticación** y asegúrese de que el URI de redirección es igual que la dirección URL de la aplicación web para la herramienta de administración, tal y como se muestra en la siguiente imagen.

   [ ![La página de autenticación con el URI de redirección especificado](../media/management-ui-redirect-uri-inline.png) ](../media/management-ui-redirect-uri-expanded.png#lightbox)

5. En el panel izquierdo, seleccione **Permisos de API** para comprobar que se agregaron los permisos. Si es un administrador global, seleccione el botón **Conceder consentimiento del administrador para `tenantname`** y siga las indicaciones del cuadro de diálogo para proporcionar consentimiento de administrador para su organización.

    [ ![La página de permisos de la API](../media/management-ui-permissions-inline.png) ](../media/management-ui-permissions-expanded.png#lightbox)

Ya puede empezar a usar la herramienta de administración.

## <a name="use-the-management-tool"></a>Uso de la herramienta de administración

Ahora que ha configurado la herramienta de administración, puede iniciarla en cualquier momento y en cualquier lugar. Siga estos pasos para iniciar la herramienta:

1. Abra la URL de la aplicación web en un explorador web. Si no recuerda la dirección URL, inicie sesión en Azure, busque el servicio de aplicaciones que ha implementado para la herramienta de administración y, después, seleccione la dirección URL.
2. Inicie sesión con sus credenciales de Windows Virtual Desktop.

   > [!NOTE]
   > Si no concedió su consentimiento de administrador al configurar la herramienta de administración, cada usuario que inicie sesión deberá proporcionar su propio consentimiento de usuario para poder usarla.

3. Cuando se le pida que elija un grupo de inquilinos, seleccione **Default Tenant Group** (grupo de inquilinos predeterminado) en la lista desplegable.
4. Al seleccionar **Default Tenant Group**, debería aparecer un menú en la parte izquierda de la ventana. En este menú, busque el nombre de su grupo de inquilinos y selecciónelo.

   > [!NOTE]
   > Si tiene un grupo de inquilinos personalizado, escriba el nombre manualmente en lugar de elegirlo en la lista desplegable.

## <a name="report-issues"></a>Informar de problemas

Si experimenta algún problema con la herramienta de administración o con cualquier otra herramienta de Windows Virtual Desktop, siga las instrucciones que encontrará en el artículo sobre [plantillas de Azure Resource Manager para Servicios de Escritorio remoto](https://github.com/Azure/RDS-Templates/blob/master/README.md) para notificarlo en GitHub.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a implementar y conectarse a la herramienta de administración, puede obtener más información sobre cómo usar Azure Service Health para supervisar problemas de servicio y avisos de estado. Para más información, vea nuestro tutorial [Configuración de alertas de servicio](set-up-service-alerts-2019.md).

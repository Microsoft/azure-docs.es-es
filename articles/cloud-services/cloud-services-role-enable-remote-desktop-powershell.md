---
title: Uso de PowerShell para habilitar Escritorio remoto para un rol
titleSuffix: Azure Cloud Services
description: Configuración de la aplicación de servicios en la nube de Azure con PowerShell para permitir conexiones a Escritorio remoto
services: cloud-services
documentationcenter: ''
author: tgore03
ms.service: cloud-services
ms.topic: article
ms.date: 07/18/2017
ms.author: tagore
ms.openlocfilehash: 13d5be5b1e0bcdf84001f21a45364296d67a5da4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87013904"
---
# <a name="enable-remote-desktop-connection-for-a-role-in-azure-cloud-services-using-powershell"></a>Habilitación de la conexión a Escritorio remoto para un rol de Azure Cloud Services mediante PowerShell

> [!div class="op_single_selector"]
> * [Azure Portal](cloud-services-role-enable-remote-desktop-new-portal.md)
> * [PowerShell](cloud-services-role-enable-remote-desktop-powershell.md)
> * [Visual Studio](cloud-services-role-enable-remote-desktop-visual-studio.md)

Escritorio remoto le permite tener acceso al escritorio de un rol que se ejecuta en Azure. Puede usar la conexión de Escritorio remoto para solucionar y diagnosticar problemas con su aplicación mientras se ejecuta.

En este artículo se describe cómo habilitar Escritorio remoto en los roles del servicio en la nube con PowerShell. Para conocer los requisitos previos necesarios para este artículo, consulte [Cómo instalar y configurar Azure PowerShell](/powershell/azure/) . PowerShell usa la extensión de Escritorio remoto, por lo que Escritorio remoto se puede habilitar después de la implementación de la aplicación.

## <a name="configure-remote-desktop-from-powershell"></a>Configuración de Escritorio remoto desde PowerShell
El cmdlet [Set-AzureServiceRemoteDesktopExtension](/powershell/module/servicemanagement/azure.service/set-azureserviceremotedesktopextension?view=azuresmps-3.7.0) permite habilitar Escritorio remoto en los roles especificados o en todos los roles de la implementación del servicio en la nube. Este cmdlet permite especificar el nombre de usuario y la contraseña del usuario de Escritorio remoto a través del parámetro *Credential* , que acepta un objeto PSCredential.

Si PowerShell se usa de forma interactiva, el objeto PSCredential se puede establecer fácilmente mediante una llamada al cmdlet [Get-Credentials](https://technet.microsoft.com/library/hh849815.aspx) .

```powershell
$remoteusercredentials = Get-Credential
```

Este comando muestra un cuadro de diálogo que permite especificar el nombre de usuario y la contraseña del usuario remoto de forma segura.

Dado que PowerShell sirve de ayuda en los escenarios de automatización, también es posible configurar el objeto **PSCredential** de manera que no requiera la interacción del usuario. En primer lugar es preciso configurar una contraseña segura. Para empezar, es preciso especificar una contraseña de texto sin formato y convertirla en una cadena segura con [ConvertTo-SecureString](https://technet.microsoft.com/library/hh849818.aspx). Seguidamente, hay que convertir esta cadena segura en una cadena estándar cifrada, para lo que se usa [ConvertFrom-SecureString](https://technet.microsoft.com/library/hh849814.aspx). Ya se puede guardar la cadena estándar cifrada en un archivo con [Set-Content](https://technet.microsoft.com/library/ee176959.aspx).

También puede crear un archivo de contraseña segura para que no tenga que escribir la contraseña cada vez. Además, un archivo de contraseña segura es mejor que un archivo de texto sin formato. Use la siguiente instrucción de PowerShell para crear un archivo de contraseña seguro:

```powershell
ConvertTo-SecureString -String "Password123" -AsPlainText -Force | ConvertFrom-SecureString | Set-Content "password.txt"
```

> [!IMPORTANT]
> Al establecer la contraseña asegúrese de cumplir los [requisitos de complejidad](https://technet.microsoft.com/library/cc786468.aspx).

Para crear el objeto de credencial desde el archivo de contraseña seguro debe leer el contenido del archivo y convertirlo en una cadena segura con [ConvertTo-SecureString](https://technet.microsoft.com/library/hh849818.aspx).

El cmdlet [Set-AzureServiceRemoteDesktopExtension](/powershell/module/servicemanagement/azure.service/set-azureserviceremotedesktopextension?view=azuresmps-3.7.0) también acepta un parámetro *Expiration* que especifica un valor **DateTime** en que la cuenta de usuario caducará. Por ejemplo, puede establecer que la cuenta caduque unos días después de la fecha y hora actuales.

En este ejemplo de PowerShell se muestra cómo establecer la extensión de Escritorio remoto en un servicio en la nube:

```powershell
$servicename = "cloudservice"
$username = "RemoteDesktopUser"
$securepassword = Get-Content -Path "password.txt" | ConvertTo-SecureString
$expiry = $(Get-Date).AddDays(1)
$credential = New-Object System.Management.Automation.PSCredential $username,$securepassword
Set-AzureServiceRemoteDesktopExtension -ServiceName $servicename -Credential $credential -Expiration $expiry
```
Opcionalmente, también puede especificar la ranura de implementación y los roles en que desea habilitar Escritorio remoto. Si no se especifican estos parámetros, el cmdlet habilita el Escritorio remoto en todos los roles de la ranura de implementación de **producción** .

La extensión de Escritorio remoto se asocia con una implementación. Si crea una nueva implementación para el servicio, tendrá que habilitar el Escritorio remoto en la nueva implementación. Si desea que Escritorio remoto esté siempre habilitado, debe considerar la integración de scripts de PowerShell en el flujo de trabajo de implementación.

## <a name="remote-desktop-into-a-role-instance"></a>Escritorio remoto en una instancia de rol

El cmdlet [Get-AzureRemoteDesktopFile](/powershell/module/servicemanagement/azure.service/get-azureremotedesktopfile?view=azuresmps-3.7.0) se usa para introducir Escritorio remoto en una instancia de rol específica del servicio en la nube. Puede usar el parámetro *LocalPath* para descargar el archivo RDP localmente. O bien puede usar el parámetro *Launch* para iniciar directamente el cuadro de diálogo Conexión a Escritorio remoto para tener acceso a la instancia de rol del servicio en la nube.

```powershell
Get-AzureRemoteDesktopFile -ServiceName $servicename -Name "WorkerRole1_IN_0" -Launch
```

## <a name="check-if-remote-desktop-extension-is-enabled-on-a-service"></a>Comprobación de si la extensión de Escritorio remoto está habilitada en un servicio

El cmdlet [Get-AzureServiceRemoteDesktopExtension](/powershell/module/servicemanagement/azure.service/get-azureremotedesktopfile?view=azuresmps-3.7.0) muestra si Escritorio remoto está habilitado o no en una implementación de servicio. El cmdlet devuelve el nombre de usuario del usuario de Escritorio remoto y los roles en la que está habilitada la extensión de Escritorio remoto. De forma predeterminada, esto se realiza en la ranura de implementación y se puede utilizar la ranura de ensayo en su lugar.

```powershell
Get-AzureServiceRemoteDesktopExtension -ServiceName $servicename
```

## <a name="remove-remote-desktop-extension-from-a-service"></a>Eliminación de la extensión de Escritorio remoto de un servicio

Si ya habilitó la extensión de Escritorio remoto en una implementación y necesita actualizar la configuración de Escritorio remota, primero debe quitar la extensión. Y, después, volver a habilitarla con la nueva configuración. Por ejemplo, si quiere establecer una nueva contraseña para la cuenta de usuario remoto o porque la cuenta ha caducado. Esto solo se requiere en las implementaciones existentes que tienen la extensión de Escritorio remoto habilitada. En las nuevas implementaciones, simplemente aplique la extensión directamente.

Para quitar la extensión de Escritorio remoto de la implementación, puede usar el cmdlet [Remove-AzureServiceRemoteDesktopExtension](/powershell/module/servicemanagement/azure.service/remove-azureserviceremotedesktopextension?view=azuresmps-3.7.0) . Opcionalmente, también puede especificar la ranura de implementación y los roles de los que desea habilitar la extensión de Escritorio remoto.

```powershell
Remove-AzureServiceRemoteDesktopExtension -ServiceName $servicename -UninstallConfiguration
```

> [!NOTE]
> Para quitar completamente la configuración de la extensión se debe llamar al cmdlet *remove* con el parámetro **UninstallConfiguration** .
>
> El parámetro **UninstallConfiguration** desinstala cualquier configuración de la extensión que se aplica al servicio. Cada configuración de la extensión se asocia a la configuración del servicio. Si se llama al cmdlet *remove* sin **UninstallConfiguration**, se desasocia la <mark>implementación</mark> de la configuración de la extensión, con lo que se quitará eficazmente la extensión. Sin embargo, la configuración de la extensión sigue estando asociada al servicio.

## <a name="additional-resources"></a>Recursos adicionales

[Configuración de Cloud Services](cloud-services-how-to-configure-portal.md)



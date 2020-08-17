---
title: Solución de problemas de Azure Cloud Shell | Microsoft Docs
description: Solución de problemas de Azure Cloud Shell
services: azure
documentationcenter: ''
author: maertendMSFT
manager: hemantm
tags: azure-resource-manager
ms.assetid: ''
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 07/24/2018
ms.author: damaerte
ms.openlocfilehash: eea64520dd5440467c911b6de42d8c8c31fc1bde
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543459"
---
# <a name="troubleshooting--limitations-of-azure-cloud-shell"></a>Solución de problemas y limitaciones de Azure Cloud Shell

Entre las resoluciones conocidas de problemas en Azure Cloud Shell se incluyen las siguientes:

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="general-troubleshooting"></a>Solución general de problemas

### <a name="error-running-azuread-cmdlets-in-powershell"></a>Error al ejecutar los cmdlets de AzureAD en PowerShell

- **Detalles**: Al ejecutar cmdlets de AzureAD como `Get-AzureADUser` en Cloud Shell, es posible que aparezca un error: `You must call the Connect-AzureAD cmdlet before calling any other cmdlets`. 
- **Solución:** Ejecute el cmdlet `Connect-AzureAD`. Anteriormente, Cloud Shell ejecutaba este cmdlet automáticamente durante el inicio de PowerShell. Para acelerar la hora de inicio, el cmdlet ya no se ejecuta automáticamente. Para restaurar el comportamiento anterior, puede optar por agregar `Connect-AzureAD` al archivo $PROFILE en PowerShell.

### <a name="early-timeouts-in-firefox"></a>Tiempos de expiración tempranos en Firefox

- **Detalles**: Cloud Shell usa una conexión WebSocket abierta para pasar la entrada/salida al explorador. Firefox tiene directivas preestablecidas que pueden cerrar la conexión WebSocket prematuramente, lo que genera tiempos de expiración tempranos en Cloud Shell.
- **Solución:** Abra FireFox y escriba "about:config" en el cuadro de dirección URL. Busque "network.websocket.timeout.ping.request" y cambie el valor de 0 a 10.

### <a name="disabling-cloud-shell-in-a-locked-down-network-environment"></a>Deshabilitación de Cloud Shell en un entorno de red bloqueada

- **Detalles**: Es posible que los administradores deseen deshabilitar el acceso a Cloud Shell para sus usuarios. Cloud Shell utiliza el acceso al dominio `ux.console.azure.com` (que se puede denegar), por lo que se detiene cualquier acceso a los puntos de entrada de Cloud Shell, incluido portal.azure.com, shell.azure.com, la extensión de la cuenta de Azure de Visual Studio Code y docs.microsoft.com. En la nube del gobierno de EE. UU., el punto de entrada es `ux.console.azure.us`; no hay ningún punto de entrada shell.azure.us correspondiente.
- **Solución:** Restrinja el acceso a `ux.console.azure.com` o `ux.console.azure.us` a través de la configuración de red para su entorno. El icono de Cloud Shell seguirá existiendo en Azure Portal, pero no se conectará correctamente al servicio.

### <a name="storage-dialog---error-403-requestdisallowedbypolicy"></a>Cuadro de diálogo de almacenamiento - Error: 403 RequestDisallowedByPolicy

- **Detalles**: Al crear una cuenta de almacenamiento a través de Cloud Shell, se produce un error debido a una asignación de Azure Policy creada por el administrador. El mensaje de error incluirá: `The resource action 'Microsoft.Storage/storageAccounts/write' is disallowed by one or more policies.`
- **Solución:** Póngase en contacto con el administrador de Azure para quitar o actualizar la asignar de Azure Policy que deniega la creación del almacenamiento.

### <a name="storage-dialog---error-400-disallowedoperation"></a>Cuadro de diálogo de almacenamiento - Error: 400 DisallowedOperation

- **Detalles**: Cuando se usa una suscripción de Azure Active Directory, no puede crear almacenamiento.
- **Solución:** Use una suscripción de Azure capaz de crear recursos de almacenamiento. Las suscripciones de Azure AD no pueden crear recursos de Azure.

### <a name="terminal-output---error-failed-to-connect-terminal-websocket-cannot-be-established-press-enter-to-reconnect"></a>Salida de terminal: Error: Failed to connect terminal: websocket cannot be established (Error al conectar con el terminal: no se puede establecer el websocket). Presione `Enter` para volver a conectarse.
- **Detalles**: Cloud Shell requiere la capacidad de establecer una conexión de WebSocket con la infraestructura de Cloud Shell.
- **Solución:** Compruebe que ha configurado las opciones de red para permitir el envío de solicitudes https y websocket para dominios en *.console.azure.com.

### <a name="set-your-cloud-shell-connection-to-support-using-tls-12"></a>Establezca la conexión de Cloud Shell para admitir el uso de TLS 1.2.
 - **Detalles**: Para definir la versión de TLS para la conexión con Cloud Shell, debe establecer la configuración específica del explorador.
 - **Solución:** Vaya a la configuración de seguridad del explorador y seleccione la casilla junto a "Usar TLS 1.2".

## <a name="bash-troubleshooting"></a>Solución de problemas de Bash

### <a name="cannot-run-the-docker-daemon"></a>No se puede ejecutar el demonio de Docker

- **Detalles**: Cloud Shell usa un contenedor para hospedar el entorno de shell, lo que hace que no se permita la ejecución del demonio.
- **Solución:** Use la [máquina de Docker](https://docs.docker.com/machine/overview/), que se instala de forma predeterminada, para administrar contenedores de Docker desde un host de Docker remoto.

## <a name="powershell-troubleshooting"></a>Solución de problemas de PowerShell

### <a name="gui-applications-are-not-supported"></a>No se admiten las aplicaciones con GUI

- **Detalles**: Si un usuario inicia una aplicación con GUI, no se devuelve el símbolo del sistema. Por ejemplo, cuando se clona un repositorio GitHub privado que tiene habilitada la autenticación en dos fases, aparece un cuadro de diálogo para completar esta autenticación.
- **Solución:** Cierre y vuelva a abrir el shell.

### <a name="troubleshooting-remote-management-of-azure-vms"></a>Solución de problemas de la administración remota de máquinas virtuales de Azure
> [!NOTE]
> Las máquinas virtuales de Azure deben tener direcciones IP públicas.

- **Detalles**: Debido a la configuración predeterminada del Firewall de Windows para WinRM, es posible que el usuario vea el error siguiente: `Ensure the WinRM service is running. Remote Desktop into the VM for the first time and ensure it can be discovered.`
- **Solución:**  Ejecute `Enable-AzVMPSRemoting` para habilitar todos los aspectos de la comunicación remota de PowerShell en la máquina de destino.

### <a name="dir-does-not-update-the-result-in-azure-drive"></a>`dir` no actualiza el resultado en la unidad de Azure.

- **Detalles**: De forma predeterminada, para optimizar la experiencia del usuario, los resultados de `dir` se almacenan en caché en la unidad de Azure.
- **Solución:** Después de crear, actualizar o quitar un recurso de Azure, ejecute `dir -force` para actualizar los resultados en la unidad de Azure.

## <a name="general-limitations"></a>Limitaciones generales

Azure Cloud Shell tiene las limitaciones conocidas siguientes:

### <a name="quota-limitations"></a>Limitaciones de cuota

Azure Cloud Shell tiene un límite de 20 usuarios simultáneos por inquilino y región. Si intenta abrir más sesiones simultáneas que el límite, verá el error "Tenant User Over Quota" (Usuario inquilino por encima de la cuota). Si tiene una necesidad legítima de tener un mayor número de sesiones abiertas que estas (por ejemplo, para sesiones de aprendizaje), póngase en contacto con el servicio de soporte técnico antes del uso previsto para solicitar un aumento de la cuota.

Cloud Shell se proporciona como servicio gratuito y está diseñado para configurar el entorno de Azure, no como una plataforma de computación de uso general. Un uso automatizado excesivo se puede considerar como incumplimiento de los términos de servicio de Azure y podría dar lugar a que se bloquee el acceso a Cloud Shell.

### <a name="system-state-and-persistence"></a>Persistencia y estado del sistema

La máquina que proporciona la sesión de Cloud Shell es temporal y se recicla después de que la sesión esté inactiva durante 20 minutos. Cloud Shell requiere montar un recurso compartido de archivos de Azure. Como resultado, la suscripción debe poder configurar los recursos de almacenamiento para tener acceso a Cloud Shell. Otras consideraciones:

- Con el almacenamiento montado, solo se conservan las modificaciones dentro del directorio `clouddrive`. En Bash, también se conserva el directorio `$HOME`.
- Solo se pueden montar recursos compartidos de archivos de Azure desde la [región asignada](persisting-shell-storage.md#mount-a-new-clouddrive).
  - En Bash, ejecute `env` para buscar la región establecida como `ACC_LOCATION`.
- Azure Files solo admite cuentas de almacenamiento con redundancia local o de almacenamiento con redundancia geográfica.

### <a name="browser-support"></a>Compatibilidad con exploradores

Cloud Shell admite las versiones más recientes de los siguientes exploradores:

- Microsoft Edge
- Microsoft Internet Explorer
- Google Chrome
- Mozilla Firefox
- Apple Safari
  - Safari en modo privado no es compatible.

### <a name="copy-and-paste"></a>Copiar y pegar

[!INCLUDE [copy-paste](../../includes/cloud-shell-copy-paste.md)]

### <a name="usage-limits"></a>Límites de uso

Cloud Shell está pensado para casos de uso interactivos. Por tanto, todas las sesiones que no sean de este tipo y que se prolonguen durante mucho tiempo se finalizarán sin previo aviso.

### <a name="user-permissions"></a>Permisos de usuario

Los permisos se establecen como usuarios normales sin acceso a sudo. No se conserva cualquier instalación fuera del directorio `$Home`.

## <a name="bash-limitations"></a>Limitaciones de Bash

### <a name="editing-bashrc"></a>Editar .bashrc

Tenga cuidado al editar .bashrc, ya que puede producir errores inesperados en Cloud Shell.

## <a name="powershell-limitations"></a>Limitaciones de PowerShell

### <a name="preview-version-of-azuread-module"></a>Versión preliminar del módulo de Azure AD

Actualmente, `AzureAD.Standard.Preview`, hay disponible una versión preliminar del módulo basado en .NET Standard. Este módulo proporciona la misma funcionalidad que `AzureAD`.

## <a name="personal-data-in-cloud-shell"></a>Datos personales en Cloud Shell

Azure Cloud Shell se toma en serio sus datos, los datos capturados y almacenados por el servicio Azure Cloud Shell se usan para proporcionar los valores predeterminados de su experiencia, tales como el shell que ha utilizado más recientemente, el tamaño y el tipo de fuente que prefiere y los detalles de recursos compartidos de archivos que respaldan la unidad en la nube. En caso de que quiera exportar o eliminar estos datos, use las siguientes instrucciones.

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-intro-sentence.md)]

### <a name="export"></a>Exportación
Para poder **exportar** la configuración de usuario, Cloud Shell guarda automáticamente datos tales como el shell, el tamaño y el tipo de fuente preferidos ejecutando los comandos siguientes.

1. [![Imagen que muestra un botón denominado Launch Azure Cloud Shell.](https://shell.azure.com/images/launchcloudshell.png)](https://shell.azure.com)

2. Ejecute los siguientes comandos en Bash o PowerShell:

Bash:

  ```
  token="Bearer $(curl http://localhost:50342/oauth2/token --data "resource=https://management.azure.com/" -H Metadata:true -s | jq -r ".access_token")"
  curl https://management.azure.com/providers/Microsoft.Portal/usersettings/cloudconsole?api-version=2017-12-01-preview -H Authorization:"$token" -s | jq
  ```

PowerShell:

  ```powershell
  $token= ((Invoke-WebRequest -Uri "$env:MSI_ENDPOINT`?resource=https://management.core.windows.net/" -Headers @{Metadata='true'}).content |  ConvertFrom-Json).access_token
  ((Invoke-WebRequest -Uri https://management.azure.com/providers/Microsoft.Portal/usersettings/cloudconsole?api-version=2017-12-01-preview -Headers @{Authorization = "Bearer $token"}).Content | ConvertFrom-Json).properties | Format-List
```

### <a name="delete"></a>Eliminar
Para poder **eliminar** la configuración de usuario, Cloud Shell guarda automáticamente datos tales como el shell, el tamaño y el tipo de fuente preferidos ejecutando los comandos siguientes. La próxima vez que inicie Cloud Shell se le pedirá que vuelva a incorporar un recurso compartido de archivos. 

>[!Note]
> Si elimina la configuración de usuario, el recurso compartido real de Azure Files no se eliminará. Vaya a Azure Files para completar esa acción.

1. [![Imagen que muestra un botón denominado Launch Azure Cloud Shell.](https://shell.azure.com/images/launchcloudshell.png)](https://shell.azure.com)

2. Ejecute los siguientes comandos en Bash o PowerShell:

Bash:

  ```
  token="Bearer $(curl http://localhost:50342/oauth2/token --data "resource=https://management.azure.com/" -H Metadata:true -s | jq -r ".access_token")"
  curl -X DELETE https://management.azure.com/providers/Microsoft.Portal/usersettings/cloudconsole?api-version=2017-12-01-preview -H Authorization:"$token"
  ```

PowerShell:

  ```powershell
  $token= ((Invoke-WebRequest -Uri "$env:MSI_ENDPOINT`?resource=https://management.core.windows.net/" -Headers @{Metadata='true'}).content |  ConvertFrom-Json).access_token
  Invoke-WebRequest -Method Delete -Uri https://management.azure.com/providers/Microsoft.Portal/usersettings/cloudconsole?api-version=2017-12-01-preview -Headers @{Authorization = "Bearer $token"}
  ```
## <a name="azure-government-limitations"></a>Limitaciones de Azure Government
Azure Cloud Shell en Azure Government solo es accesible mediante Azure Portal.

>[!Note]
> Actualmente no se admite la conexión a las nubes GCC-High o DoD de Government de Exchange Online.

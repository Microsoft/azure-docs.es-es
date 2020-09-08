---
title: Solución de problemas del servicio Azure Image Builder
description: Solución de errores y problemas comunes al usar el servicio Azure VM Image Builder
author: cynthn
ms.author: danis
ms.date: 08/07/2020
ms.topic: troubleshooting
ms.service: virtual-machines
ms.subservice: imaging
ms.openlocfilehash: 754d9324137632b928e67bbe4c67a3e6c72e452a
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88068027"
---
# <a name="troubleshoot-azure-image-builder-service"></a>Solución de problemas del servicio Azure Image Builder

Este artículo le ayuda a solucionar problemas conocidos que se pueden producir al usar el servicio Azure Image Builder.

Los errores de AIB se pueden producir en dos áreas:
- Envío de la plantilla de imagen
- Compilación de la imagen

## <a name="troubleshoot-image-template-submission-errors"></a>Solucionar errores de envío de la plantilla de imagen

Los errores de envío de la plantilla de imagen solo se devuelven en el envío. No hay ningún registro de errores para los errores de envío de la plantilla de imagen. Si se produjo un error durante el envío, puede devolver el error comprobando el estado de la plantilla, mediante la revisión específica de `ProvisioningState` y `ProvisioningErrorMessage`/`provisioningError`.

```azurecli
az image builder show --name $imageTemplateName  --resource-group $imageResourceGroup
```

```azurepowershell-interactive
Get-AzImageBuilderTemplate -ImageTemplateName  <imageTemplateName> -ResourceGroupName <imageTemplateResourceGroup> | Select-Object ProvisioningState, ProvisioningErrorMessage
```
> [!NOTE]
> En el caso de PowerShell, debe instalar los [módulos de PowerShell para Azure Image Builder](../windows/image-builder-powershell.md#prerequisites).

En las secciones siguientes se incluye la guía de resolución de problemas para errores comunes de envío de plantillas de imagen.

### <a name="updateupgrade-of-image-templates-is-currently-not-supported"></a>Actualmente no se admite la actualización de las plantillas de imagen

#### <a name="error"></a>Error

```text
'Conflict'. Details: Update/Upgrade of image templates is currently not supported
```

#### <a name="cause"></a>Causa

La plantilla ya existe.

#### <a name="solution"></a>Solución

Si envía una plantilla de configuración de imagen y se produce un error en el envío, significa que todavía existe un artefacto de plantilla con errores. Elimine la plantilla con errores.

### <a name="the-resource-operation-completed-with-terminal-provisioning-state-failed"></a>La operación del recurso finalizó con un estado de aprovisionamiento final "Error"

#### <a name="error"></a>Error

```text
Microsoft.VirtualMachineImages/imageTemplates 'helloImageTemplateforSIG01' failed with message '{
  "status": "Failed",
  "error": {
    "code": "ResourceDeploymentFailure",
    "message": "The resource operation completed with terminal provisioning state 'Failed'.",
    "details": [
      {
        "code": "InternalOperationError",
        "message": "Internal error occurred."
```
#### <a name="cause"></a>Causa

En la mayoría de los casos, el error de implementación de recursos se produce debido a la falta de permisos.

#### <a name="solution"></a>Solución

En función del escenario, Azure Image Builder puede necesitar permisos para:
- Imagen de origen o grupo de recursos de Shared Image Gallery.
- Imagen de distribución o recurso de Shared Image Gallery.
- La cuenta de almacenamiento, el contenedor o el blob a los que accede el personalizador de archivos. 

Para más información sobre la configuración de permisos, vea [Configuración de los permisos del Azure Image Builder la mediante CLI de Azure](image-builder-permissions-cli.md) o [Configuración de los permisos del servicio Azure Image Builder mediante PowerShell](image-builder-permissions-powershell.md).

### <a name="error-getting-managed-image"></a>Error al obtener la imagen administrada

#### <a name="error"></a>Error

```text
Build (Managed Image) step failed: Error getting Managed Image '/subscriptions/.../providers/Microsoft.Compute/images/mymanagedmg1': Error getting managed image (...): compute.
ImagesClient#Get: Failure responding to request: StatusCode=403 -- Original Error: autorest/azure: Service returned an error.
Status=403 Code="AuthorizationFailed" Message="The client '......' with object id '......' does not have authorization to perform action 'Microsoft.Compute/images/read' over scope 
```
#### <a name="cause"></a>Causa

Faltan permisos.

#### <a name="solution"></a>Solución

En función del escenario, Azure Image Builder puede necesitar permisos para:
* Imagen de origen o grupo de recursos de Shared Image Gallery.
* Imagen de distribución o recurso de Shared Image Gallery.
* La cuenta de almacenamiento, el contenedor o el blob a los que accede el personalizador de archivos. 

Para más información sobre la configuración de permisos, vea [Configuración de los permisos del Azure Image Builder la mediante CLI de Azure](image-builder-permissions-cli.md) o [Configuración de los permisos del servicio Azure Image Builder mediante PowerShell](image-builder-permissions-powershell.md).

### <a name="build--step-failed-for-image-version"></a>Error en el paso de compilación de la versión de la imagen

#### <a name="error"></a>Error
```text
Build (Shared Image Version) step failed for Image Version '/subscriptions/.../providers/Microsoft.Compute/galleries/.../images/... /versions/0.23768.4001': Error getting Image Version '/subscriptions/.../resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/.../images/.../versions/0.23768.4001': Error getting image version '... :0.23768.4001': compute.GalleryImageVersionsClient#Get: Failure responding to request: StatusCode=404 -- Original Error: autorest/azure: Service returned an error. 
Status=404 Code="ResourceNotFound" Message="The Resource 'Microsoft.Compute/galleries/.../images/.../versions/0.23768.4001' under resource group '<rgName>' was not found."
```
#### <a name="cause"></a>Causa

Azure Image Builder no encuentra la imagen de origen.

#### <a name="solution"></a>Solución

Asegúrese de que la imagen de origen es correcta y existe en la ubicación del servicio Azure Image Builder.

### <a name="downloading-external-file-to-local-file"></a>Descarga de un archivo externo en un archivo local

#### <a name="error"></a>Error

```text
Downloading external file (<myFile>) to local file (xxxxx.0.customizer.fp) [attempt 1 of 10] failed: Error downloading '<myFile>' to 'xxxxx.0.customizer.fp'..
```

#### <a name="cause"></a>Causa

El nombre o la ubicación del archivo no son correctos o no se puede acceder a la ubicación.

#### <a name="solution"></a>Solución

Asegúrese de que se puede acceder al archivo. Compruebe que el nombre y la ubicación son correctos.

## <a name="troubleshoot-build-failures"></a>Solución de errores de compilación

En el caso de los errores de compilación de la imagen, puede obtener el error de `lastrunstatus` y, después, revisar los detalles en el archivo customization.log.


```azurecli
az image builder show --name $imageTemplateName  --resource-group $imageResourceGroup
```

```azurepowershell-interactive
Get-AzImageBuilderTemplate -ImageTemplateName  <imageTemplateName> -ResourceGroupName <imageTemplateResourceGroup> | Select-Object LastRunStatus, LastRunStatusMessage
```

### <a name="customization-log"></a>Registro de personalización

Cuando se ejecuta la compilación de la imagen, los registros se crean y almacenan en una cuenta de almacenamiento. Azure Image Builder crea la cuenta de almacenamiento en el grupo de recursos temporal cuando se crea un artefacto de plantilla de imagen.

El nombre de la cuenta de almacenamiento usa el siguiente patrón: **IT_\<ImageResourceGroupName\> _\<TemplateName\>_ \<GUID\>**

Por ejemplo, *IT_aibmdi_helloImageTemplateLinux01*.

Puede ver el archivo customization.log en la cuenta de almacenamiento del grupo de recursos; para ello, seleccione **Cuenta de almacenamiento** > **Blobs** > `packerlogs`.  A continuación, seleccione **Directorio > customization.log**.


### <a name="understanding-the-customization-log"></a>Descripción del registro de personalización

El registro es detallado. Trata la compilación de la imagen, incluidos los problemas con la distribución de la imagen, como la replicación de Shared Image Gallery. Estos errores aparecen en el mensaje de error del estado de la plantilla de imagen.

El archivo customization.log incluye las siguientes fases:

1. Implemente la máquina virtual de compilación y las dependencias mediante plantillas de ARM en la fase del grupo de recursos de almacenamiento provisional IT_. Esta fase incluye varias PUBLICACIONES en el proveedor de recursos de Azure Image Builder:
    ```text
    Azure request method="POST" request="https://management.azure.com/subscriptions/<subID>/resourceGroups/IT_aibImageRG200_window2019VnetTemplate01_dec33089-1cc3-cccc-cccc-ccccccc/providers/Microsoft.Storage/storageAccounts
    ..
    PACKER OUT ==> azure-arm: Deploying deployment template ...
    ..
    ```

2. Estado de la fase de las implementaciones. Esta fase incluye el estado de cada implementación de recursos:
    ```text
    PACKER ERR 2020/04/30 23:28:50 packer: 2020/04/30 23:28:50 Azure request method="GET" request="https://management.azure.com/subscriptions/<subID>/resourcegroups/IT_aibImageRG200_window2019VnetTemplate01_dec33089-1cc3-4505-ae28-6661e43fac48/providers/Microsoft.Resources/deployments/pkrdp51lc0339jg/operationStatuses/08586133176207523519?[REDACTED]" body=""
    ```

3. Conéctese a la fase de máquinas virtuales de compilación.

    Si se trata de Windows, el servicio Azure Image Builder se conecta mediante WinRM:
    ```text
    PACKER ERR 2020/04/30 23:30:50 packer: 2020/04/30 23:30:50 Waiting for WinRM, up to timeout: 10m0s
    ..
    PACKER OUT     azure-arm: WinRM connected.
    ```

    Si se trata de Linux, el servicio Azure Image Builder se conectará mediante SSH:
    ```text
    PACKER OUT ==> azure-arm: Waiting for SSH to become available...
    PACKER ERR 2019/12/10 17:20:51 packer: 2020/04/10 17:20:51 [INFO] Waiting for SSH, up to timeout: 20m0s
    PACKER OUT ==> azure-arm: Connected to SSH!
    ```

4. Ejecute la fase de personalizaciones. Cuando se ejecutan las personalizaciones, puede identificarlas revisando el archivo customization.log. Busque *(telemetry)* .
    ```text
    (telemetry) Starting provisioner windows-update
    (telemetry) ending windows-update
    (telemetry) Starting provisioner powershell
    (telemetry) ending powershell
    (telemetry) Starting provisioner file
    (telemetry) ending file
    (telemetry) Starting provisioner windows-restart
    (telemetry) ending windows-restart
    
    (telemetry) Finalizing. - This means the build hasfinished
    ```
5. Fase de desaprovisionamiento. Azure Image Builder agrega un personalizador oculto. Este paso de desaprovisionamiento es responsable de preparar la máquina virtual para el desaprovisionamiento. Ejecuta Windows Sysprep (mediante c:\DeprovisioningScript.ps1) o en el desaprovisionamiento de waagent en Linux (mediante /tmp/DeprovisioningScript.sh). 

    Por ejemplo:
    ```text
    PACKER ERR 2020/03/04 23:05:04 [INFO] (telemetry) Starting provisioner powershell
    PACKER ERR 2020/03/04 23:05:04 packer: 2020/03/04 23:05:04 Found command: if( TEST-PATH c:\DeprovisioningScript.ps1 ){cat c:\DeprovisioningScript.ps1} else {echo "Deprovisioning script [c:\DeprovisioningScript.ps1] could not be found. Image build may fail or the VM created from the Image may not boot. Please make sure the deprovisioning script is not accidentally deleted by a Customizer in the Template."}
    ```

6. Fase de limpieza. Una vez completada la compilación, se eliminan los recursos de Azure Image Builder.
    ```text
    PACKER ERR 2020/02/04 02:04:23 packer: 2020/02/04 02:04:23 Azure request method="DELETE" request="https://management.azure.com/subscriptions/<subId>/resourceGroups/IT_aibDevOpsImg_t_vvvvvvv_yyyyyy-de5f-4f7c-92f2-xxxxxxxx/providers/Microsoft.Network/networkInterfaces/pkrnijamvpo08eo?[REDACTED]" body=""
    ```
## <a name="tips-for-troubleshooting-scriptinline-customization"></a>Sugerencias para la solución de problemas de la personalización de script o comandos insertados
- Prueba del código antes de proporcionarlo a Image Builder
- Asegúrese de que Azure Policy y los firewalls permiten la conectividad a los recursos remotos.
- Envíe comentarios a la consola, como el uso de `Write-Host` o `echo`, ya que esto permitirá buscar en el archivo customization.log.

## <a name="troubleshoot-common-build-errors"></a>Solución de errores de compilación comunes

### <a name="packer-build-command-failure"></a>Error de comando de compilación de Packer

#### <a name="error"></a>Error

```text
  "provisioningState": "Succeeded",
  "lastRunStatus": {
   "startTime": "2020-04-30T23:24:06.756985789Z",
   "endTime": "2020-04-30T23:39:14.268729811Z",
   "runState": "Failed",
   "message": "Failed while waiting for packerizer: Microservice has failed: Failed while processing request: Error when executing packerizer: Packer build command has failed: exit status 1. During the image build, a failure has occurred, please review the build log to identify which build/customization step failed. For more troubleshooting steps go to https://aka.ms/azvmimagebuilderts. Image Build log location: https://xxxxxxxxxx.blob.core.windows.net/packerlogs/xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx/customization.log. OperationId: xxxxxx-5a8c-4379-xxxx-8d85493bc791. Use this operationId to search packer logs."
```

#### <a name="cause"></a>Causa

Error de personalización

#### <a name="solution"></a>Solución

Revise el registro para buscar los errores de los personalizadores. Busque *(telemetry)* . 

Por ejemplo:
```text
(telemetry) Starting provisioner windows-update
(telemetry) ending windows-update
(telemetry) Starting provisioner powershell
(telemetry) ending powershell
(telemetry) Starting provisioner file
(telemetry) ending file
(telemetry) Starting provisioner windows-restart
(telemetry) ending windows-restart

(telemetry) Finalizing. - This means the build has finished
```

### <a name="timeout-exceeded"></a>Se superó el tiempo de espera.

#### <a name="error"></a>Error

```text
Deployment failed. Correlation ID: xxxxx-xxxx-xxxx-xxxx-xxxxxxxxx. Failed in building/customizing image: Failed while waiting for packerizer: Timeout waiting for microservice to complete: 'context deadline exceeded'
```

#### <a name="cause"></a>Causa

La compilación superó el tiempo de espera. Este error aparece en "lastrunstatus".

#### <a name="solution"></a>Solución

1. Revise el archivo customization.log. Identifique el último personalizador que se va a ejecutar. Busque `(telemetry)` a partir de la parte inferior del registro. 

2. Compruebe las personalizaciones del script. Puede que las personalizaciones no supriman la interacción del usuario con los comandos, como las opciones `quiet`. Por ejemplo, `apt-get install -y` da como resultado la ejecución del script en espera de la interacción del usuario.

3. Si usa el personalizador de `File` para descargar artefactos de más de 20 MB, consulte la sección de soluciones alternativas.

4. Revise los errores y las dependencias del script que pueden hacer que el script espere.

5. Si espera que las personalizaciones necesiten más tiempo, aumente [buildTimeoutInMinutes](image-builder-json.md). El valor predeterminado es cuatro horas.

6. Si tiene acciones que consumen muchos recursos, como la descarga de gigabytes de archivos, tenga en cuenta el tamaño de la máquina virtual de compilación subyacente. El servicio usa una máquina virtual Standard_D1_v2. La máquina virtual tiene 1 vCPU y 3,5 GB de memoria. Si va a descargar 50 GB, es probable que se agoten los recursos de la máquina virtual y que se produzcan errores de comunicación entre el servicio Azure Image Builder y la máquina virtual de compilación. Vuelva a intentar la compilación con una máquina virtual con más memoria, estableciendo el valor [VM_Size](image-builder-json.md#vmprofile).

### <a name="long-file-download-time"></a>Tiempo de descarga largo de archivos

#### <a name="error"></a>Error
```text
[086cf9c4-0457-4e8f-bfd4-908cfe3fe43c] PACKER OUT 
myBigFile.zip 826 B / 826000 B  1.00%
[086cf9c4-0457-4e8f-bfd4-908cfe3fe43c] PACKER OUT 
myBigFile.zip 1652 B / 826000 B  2.00%
[086cf9c4-0457-4e8f-bfd4-908cfe3fe43c] PACKER OUT 
..
hours later...
..
myBigFile.zip 826000 B / 826000 B  100.00%
[086cf9c4-0457-4e8f-bfd4-908cfe3fe43c] PACKER OUT 
```
#### <a name="cause"></a>Causa 

El personalizador de archivos está descargando un archivo grande.

#### <a name="solution"></a>Solución

El personalizador de archivos solo es adecuado para descargas de archivos pequeños, con menos de 20 MB. En el caso de descargas de archivos más grandes, use un script o un comando insertado. Por ejemplo, en Linux, puede usar `wget` o `curl`. En Windows, puede usar `Invoke-WebRequest`.

### <a name="error-waiting-on-shared-image-gallery"></a>Error al esperar en Shared Image Gallery

#### <a name="error"></a>Error

```text
Deployment failed. Correlation ID: XXXXXX-XXXX-XXXXXX-XXXX-XXXXXX. Failed in distributing 1 images out of total 1: {[Error 0] [Distribute 0] Error publishing MDI to shared image gallery:/subscriptions/<subId>/resourceGroups/xxxxxx/providers/Microsoft.Compute/galleries/xxxxx/images/xxxxxx, Location:eastus. Error: Error returned from SIG client while publishing MDI to shared image gallery for dstImageLocation: eastus, dstSubscription: <subId>, dstResourceGroupName: XXXXXX, dstGalleryName: XXXXXX, dstGalleryImageName: XXXXXX. Error: Error waiting on shared image gallery future for resource group: XXXXXX, gallery name: XXXXXX, gallery image name: XXXXXX.Error: Future#WaitForCompletion: context has been cancelled: StatusCode=200 -- Original Error: context deadline exceeded}
```

#### <a name="cause"></a>Causa

Image Builder agotó el tiempo de espera para que la imagen se agregara y replicara en Shared Image Gallery (SIG). Si la imagen se inserta en SIG, se puede suponer que la imagen se ha creado correctamente. Sin embargo, se produjo un error en el proceso general porque Image Builder estaba esperando en Shared Image Gallery a que se completara la replicación. Aunque se ha producido un error en la compilación, la replicación continúa. Puede obtener las propiedades de la versión de la imagen comprobando la distribución *runOutput*.

```bash
$runOutputName=<distributionRunOutput>
az resource show \
    --ids "/subscriptions/$subscriptionID/resourcegroups/$imageResourceGroup/providers/Microsoft.VirtualMachineImages/imageTemplates/$imageTemplateName/runOutputs/$runOutputName"  \
    --api-version=2019-05-01-preview
```

#### <a name="solution"></a>Solución

Aumente **buildTimeoutInMinutes**.
 
### <a name="low-windows-resource-information-events"></a>Pocos eventos de información de recursos de Windows

#### <a name="error"></a>Error

```text
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT     azure-arm: Waiting for operation to complete (system performance: 1% cpu; 37% memory)...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT     azure-arm: Waiting for operation to complete (system performance: 51% cpu; 35% memory)...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT     azure-arm: Waiting for operation to complete (system performance: 21% cpu; 37% memory)...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT     azure-arm: Waiting for operation to complete (system performance: 21% cpu; 36% memory)...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT     azure-arm: Waiting for operation to complete (system performance: 90% cpu; 32% memory)...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT     azure-arm: Waiting for the Windows Modules Installer to exit...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer: 2020/04/30 23:38:58 [INFO] command 'PowerShell -ExecutionPolicy Bypass -OutputFormat Text -File C:/Windows/Temp/packer-windows-update-elevated.ps1' exited with code: 101
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT ==> azure-arm: Restarting the machine...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer: 2020/04/30 23:38:58 [INFO] RPC endpoint: Communicator ended with: 101
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] 1672 bytes written for 'stdout'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] 0 bytes written for 'stderr'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] RPC client: Communicator ended with: 101
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] RPC endpoint: Communicator ended with: 101
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT ==> azure-arm: Waiting for machine to become available...
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer-provisioner-windows-update: 2020/04/30 23:38:58 [INFO] 1672 bytes written for 'stdout'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer-provisioner-windows-update: 2020/04/30 23:38:58 [INFO] 0 bytes written for 'stderr'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer-provisioner-windows-update: 2020/04/30 23:38:58 [INFO] RPC client: Communicator ended with: 101
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer: 2020/04/30 23:38:58 [INFO] starting remote command: shutdown.exe -f -r -t 0 -c "packer restart"
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer: 2020/04/30 23:38:58 [INFO] command 'shutdown.exe -f -r -t 0 -c "packer restart"' exited with code: 0
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer: 2020/04/30 23:38:58 [INFO] RPC endpoint: Communicator ended with: 0
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] 0 bytes written for 'stderr'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] 0 bytes written for 'stdout'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT ==> azure-arm: A system shutdown is in progress.(1115)
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] RPC client: Communicator ended with: 0
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 [INFO] RPC endpoint: Communicator ended with: 0
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer-provisioner-windows-update: 2020/04/30 23:38:58 [INFO] 0 bytes written for 'stdout'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer-provisioner-windows-update: 2020/04/30 23:38:58 [INFO] 0 bytes written for 'stderr'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:58 packer-provisioner-windows-update: 2020/04/30 23:38:58 [INFO] RPC client: Communicator ended with: 0
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer: 2020/04/30 23:38:59 [INFO] starting remote command: shutdown.exe -f -r -t 60 -c "packer restart test"
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer: 2020/04/30 23:38:59 [INFO] command 'shutdown.exe -f -r -t 60 -c "packer restart test"' exited with code: 1115
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer: 2020/04/30 23:38:59 [INFO] RPC endpoint: Communicator ended with: 1115
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 [INFO] 0 bytes written for 'stdout'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 [INFO] 40 bytes written for 'stderr'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 [INFO] RPC client: Communicator ended with: 1115
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 [INFO] RPC endpoint: Communicator ended with: 1115
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer-provisioner-windows-update: 2020/04/30 23:38:59 [INFO] 40 bytes written for 'stderr'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer-provisioner-windows-update: 2020/04/30 23:38:59 [INFO] 0 bytes written for 'stdout'
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer-provisioner-windows-update: 2020/04/30 23:38:59 [INFO] RPC client: Communicator ended with: 1115
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER ERR 2020/04/30 23:38:59 packer-provisioner-windows-update: 2020/04/30 23:38:59 Retryable error: Machine not yet available (exit status 1115)
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT Build 'azure-arm' errored: unexpected EOF
[45f485cf-5a8c-4379-9937-8d85493bc791] PACKER OUT 
```
#### <a name="cause"></a>Causa

Agotamiento de recursos. Este problema suele producirse con Windows Update en ejecución mediante el uso del tamaño de la máquina virtual de compilación predeterminado D1_V2.

#### <a name="solution"></a>Solución

Aumente el tamaño de la máquina virtual de compilación.

### <a name="builds-finished-but-no-artifacts-were-created"></a>Compilaciones finalizadas pero sin artefactos creados

#### <a name="error"></a>Error

```text
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER OUT Build 'azure-arm' errored: Future#WaitForCompletion: context has been cancelled: StatusCode=200 -- Original Error: context deadline exceeded
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER ERR ==> Some builds didn't complete successfully and had errors:
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER OUT 
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER ERR 2020/04/30 22:29:23 machine readable: azure-arm,error []string{"Future#WaitForCompletion: context has been cancelled: StatusCode=200 -- Original Error: context deadline exceeded"}
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER OUT ==> Some builds didn't complete successfully and had errors:
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER ERR ==> Builds finished but no artifacts were created.
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER OUT --> azure-arm: Future#WaitForCompletion: context has been cancelled: StatusCode=200 -- Original Error: context deadline exceeded
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER ERR 2020/04/30 22:29:23 Cancelling builder after context cancellation context canceled
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER OUT 
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER ERR 2020/04/30 22:29:23 [INFO] (telemetry) Finalizing.
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER OUT ==> Builds finished but no artifacts were created.
[a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER ERR 2020/04/30 22:29:24 waiting for all plugin processes to complete...
Done exporting Packer logs to Azure for Packer prefix: [a170b40d-2d77-4ac3-8719-72cdc35cf889] PACKER OUT
```
#### <a name="cause"></a>Causa

Tiempo de espera debido a la creación de los recursos de Azure necesarios.

#### <a name="solution"></a>Solución

Vuelva a ejecutar la compilación para intentarlo de nuevo.

### <a name="resource-not-found"></a>Recurso no encontrado

#### <a name="error"></a>Error

```text
  "provisioningState": "Succeeded",
  "lastRunStatus": {
   "startTime": "2020-05-01T00:13:52.599326198Z",
   "endTime": "2020-05-01T00:15:13.62366898Z",
   "runState": "Failed",
   "message": "network.InterfacesClient#UpdateTags: Failure responding to request: StatusCode=404 -- Original Error: autorest/azure: Service returned an error. Status=404 Code=\"ResourceNotFound\" Message=\"The Resource 'Microsoft.Network/networkInterfaces/aibpls7lz2e.nic.4609d697-be0a-4cb0-86af-49b6fe877fe1' under resource group 'IT_aibImageRG200_window2019VnetTemplate01_9988723b-af56-413a-9006-84130af0e9df' was not found.\""
  },
```
#### <a name="cause"></a>Causa

Faltan permisos.

#### <a name="solution"></a>Solución

Vuelva a comprobar si Azure Image Builder tiene todos los permisos que necesita. 

Para más información sobre la configuración de permisos, vea [Configuración de los permisos del Azure Image Builder la mediante CLI de Azure](image-builder-permissions-cli.md) o [Configuración de los permisos del servicio Azure Image Builder mediante PowerShell](image-builder-permissions-powershell.md).

### <a name="sysprep-timing"></a>Intervalos de Sysprep

#### <a name="error"></a>Error

```text
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: Write-Output '>>> Waiting for GA Service (RdAgent) to start ...'
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: while ((Get-Service RdAgent) -and ((Get-Service RdAgent).Status -ne 'Running')) { Start-Sleep -s 5 }
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: Write-Output '>>> Waiting for GA Service (WindowsAzureTelemetryService) to start ...'
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: while ((Get-Service WindowsAzureTelemetryService) -and ((Get-Service WindowsAzureTelemetryService).Status -ne 'Running')) { Start-Sleep -s 5 }
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: Write-Output '>>> Waiting for GA Service (WindowsAzureGuestAgent) to start ...'
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: while ((Get-Service WindowsAzureGuestAgent) -and ((Get-Service WindowsAzureGuestAgent).Status -ne 'Running')) { Start-Sleep -s 5 }
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: Write-Output '>>> Sysprepping VM ...'
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: if( Test-Path $Env:SystemRoot\system32\Sysprep\unattend.xml ) {
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm:   Remove-Item $Env:SystemRoot\system32\Sysprep\unattend.xml -Force
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: }
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: & $Env:SystemRoot\System32\Sysprep\Sysprep.exe /oobe /generalize /quiet /quit
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: while($true) {
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm:   $imageState = (Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Setup\State).ImageState
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm:   Write-Output $imageState
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm:   if ($imageState -eq 'IMAGE_STATE_GENERALIZE_RESEAL_TO_OOBE') { break }
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm:   Start-Sleep -s 5
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: }
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: Write-Output '>>> Sysprep complete ...'
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: >>> Waiting for GA Service (RdAgent) to start ...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: >>> Waiting for GA Service (WindowsAzureTelemetryService) to start ...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: >>> Waiting for GA Service (WindowsAzureGuestAgent) to start ...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: >>> Sysprepping VM ...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_COMPLETE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm: Get-Service : Cannot find any service with service name 'WindowsAzureGuestAgent'.
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm: At C:\DeprovisioningScript.ps1:6 char:9
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm: + while ((Get-Service WindowsAzureGuestAgent) -and ((Get-Service Window ...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm: +         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm:     + CategoryInfo          : ObjectNotFound: (WindowsAzureGuestAgent:String) [Get-Service], ServiceCommandException
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm:     + FullyQualifiedErrorId : NoServiceFoundForGivenName,Microsoft.PowerShell.Commands.GetServiceCommand
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm:
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT     azure-arm: IMAGE_STATE_UNDEPLOYABLE
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER ERR 2020/05/05 22:26:17 Cancelling builder after context cancellation context canceled
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT Cancelling build after receiving terminated
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER ERR 2020/05/05 22:26:17 packer: 2020/05/05 22:26:17 Cancelling provisioning due to context cancellation: context canceled
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm: 
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER ERR 2020/05/05 22:26:17 packer: 2020/05/05 22:26:17 Cancelling hook after context cancellation context canceled
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER OUT ==> azure-arm: The resource group was not created by Packer, deleting individual resources ...
[922bdf36-b53c-4e78-9cd8-6b70b9674685] PACKER ERR ==> azure-arm: The resource group was not created by Packer, deleting individual resources ...
```
#### <a name="cause"></a>Causa

La causa puede ser un problema de intervalos debido al tamaño de la máquina virtual D1_V2. Si las personalizaciones están limitadas y se ejecutan en menos de tres segundos, Azure Image Builder ejecuta los comandos Sysprep para el desaprovisionamiento. Cuando Azure Image Builder se desaprovisiona, el comando Sysprep comprueba *WindowsAzureGuestAgent*, que puede que no esté completamente instalado, lo que provoca un problema en los intervalos. 

#### <a name="solution"></a>Solución

Aumente el tamaño de la máquina virtual. O bien, puede agregar una personalización de suspensión de PowerShell de 60 segundos para evitar el problema en los intervalos.

## <a name="devops-task"></a>Tarea de DevOps 

### <a name="troubleshooting-the-task"></a>Solución de problemas con la tarea
Solo se plantearán problemas con la tarea si se produce un error durante la personalización; cuando esto sucede, la tarea informará de un error y dejará el grupo de recursos de almacenamiento provisional, con los registros, para que pueda identificar el problema. 

Para buscar el registro, debe conocer el nombre de la plantilla, ir a Canalización > Error al compilar y profundizar en la tarea de DevOps de AIB; después, verá el registro y el nombre de una plantilla:
```text
start reading task parameters...
found build at:  /home/vsts/work/r1/a/_ImageBuilding/webapp
end reading parameters
getting storage account details for aibstordot1556933914
created archive /home/vsts/work/_temp/temp_web_package_21475337782320203.zip
Source for image:  { type: 'SharedImageVersion',
  imageVersionId: '/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/<galleryName>/images/<imageDefName>/versions/<imgVersionNumber>' }
template name:  t_1556938436xxx
``` 

Vaya al portal, busque el nombre de la plantilla en el grupo de recursos y busque el grupo de recursos con IT_*.
Vaya a la cuenta de almacenamiento > blobs > contenedores > registros.

### <a name="troubleshooting-successful-builds"></a>Solución de problemas de compilaciones correctas
Puede haber algunos casos en los que necesite investigar las compilaciones correctas y desee revisar el registro. Como se mencionó, si la creación de la imagen se realiza correctamente, el grupo de recursos de almacenamiento provisional que contiene los registros se eliminará como parte de la limpieza. Sin embargo, lo que puede hacer es introducir una suspensión después del comando insertado y, a continuación, obtener los registros mientras la compilación está en pausa. Para ello, siga estos pasos:
 
1. Actualice el comando insertado y agregue: Write-Host / Echo “Sleep”: permite buscar en el registro.
2. Agregue una suspensión de al menos 10 minutos; puede usar [Start-Sleep](/powershell/module/microsoft.powershell.utility/start-sleep) o el comando `Sleep` de Linux.
3. Use el método anterior para identificar la ubicación del registro y, a continuación, siga descargando o comprobando el registro hasta que llegue el momento de la suspensión.


### <a name="operation-was-canceled"></a>Operación cancelada

#### <a name="error"></a>Error

```text
2020-05-05T18:28:24.9280196Z ##[section]Starting: Azure VM Image Builder Task
2020-05-05T18:28:24.9609966Z ==============================================================================
2020-05-05T18:28:24.9610739Z Task         : Azure VM Image Builder Test(Preview)
2020-05-05T18:28:24.9611277Z Description  : Build images using Azure Image Builder resource provider.
2020-05-05T18:28:24.9611608Z Version      : 1.0.18
2020-05-05T18:28:24.9612003Z Author       : Microsoft Corporation
2020-05-05T18:28:24.9612718Z Help         : For documentation, and end to end example, please visit: http://aka.ms/azvmimagebuilderdevops
2020-05-05T18:28:24.9613390Z ==============================================================================
2020-05-05T18:28:26.0651512Z start reading task parameters...
2020-05-05T18:28:26.0673377Z found build at:  d:\a\r1\a\_AppsAndImageBuilder\webApp
2020-05-05T18:28:26.0708785Z end reading parameters
2020-05-05T18:28:26.0745447Z getting storage account details for aibstagstor1565047758
2020-05-05T18:28:29.8812270Z created archive d:\a\_temp\temp_web_package_09737279437949953.zip
2020-05-05T18:28:33.1568013Z Source for image:  { type: 'PlatformImage',
2020-05-05T18:28:33.1584131Z   publisher: 'MicrosoftWindowsServer',
2020-05-05T18:28:33.1585965Z   offer: 'WindowsServer',
2020-05-05T18:28:33.1592768Z   sku: '2016-Datacenter',
2020-05-05T18:28:33.1594191Z   version: '14393.3630.2004101604' }
2020-05-05T18:28:33.1595387Z template name:  t_1588703313152
2020-05-05T18:28:33.1597453Z starting put template...
2020-05-05T18:28:52.9278603Z put template:  Succeeded
2020-05-05T18:28:52.9281282Z starting run template...
2020-05-05T19:33:14.3923479Z ##[error]The operation was canceled.
2020-05-05T19:33:14.3939721Z ##[section]Finishing: Azure VM Image Builder Task
```
#### <a name="cause"></a>Causa

Si un usuario no canceló la compilación, entonces lo hizo el agente de usuario de Azure DevOps. Lo más probable es que se haya agotado el tiempo de espera de una hora debido a las funcionalidades de Azure DevOps. Si usa un proyecto y un agente privados, dispondrá de 60 minutos de tiempo de compilación. Si la compilación supera el tiempo de espera, DevOps cancela la tarea en ejecución.

Para más información sobre las funcionalidades y limitaciones de Azure DevOps, vea [Agentes hospedados por Microsoft](https://docs.microsoft.com/azure/devops/pipelines/agents/hosted?view=azure-devops#capabilities-and-limitations).
 
#### <a name="solution"></a>Solución

Puede hospedar sus propios agentes de DevOps o tratar de reducir el tiempo de la compilación. Por ejemplo, si va a realizar la distribución a Shared Image Gallery, replique en una región. Si desea realizar la replicación de forma asincrónica. 
 
## <a name="vms-created-from-aib-images-do-not-create-successfully"></a>Las máquinas virtuales creadas a partir de imágenes de AIB no se crean correctamente

De forma predeterminada, Azure Image Builder ejecuta código de *desaprovisionamiento* al final de cada fase de personalización de la imagen con el fin de *generalizar* la imagen. La generalización es un proceso en el que la imagen se configura para poder reutilizarla a fin de crear varias máquinas virtuales y poder usarla en la configuración de la máquina virtual, como el nombre de host, el nombre de usuario, etc. Si se trata de Windows, Azure Image Builder ejecuta *Sysprep*, y en Linux, Azure Image Builder ejecuta `waagent -deprovision`. 

En Windows, Azure Image Builder usa un comando Sysprep genérico. Sin embargo, esto puede no ser conveniente para todas las generalizaciones de Windows correctas. Azure Image Builder permite personalizar el comando Sysprep. Tenga en cuenta que Azure Image Builder es una herramienta de automatización de imágenes. Es responsable de ejecutar el comando Sysprep correctamente. Sin embargo, puede que necesite comandos Sysprep diferentes para que la imagen se pueda reutilizar. En Linux, Azure Image Builder usa un comando `waagent -deprovision+user` genérico. Para más información, vea la [documentación del agente Linux de Microsoft Azure](https://github.com/Azure/WALinuxAgent#command-line-options).

Si migra una personalización existente y usa comandos Sysprep/waagent diferentes, puede probar los comandos genéricos de Image Builder. Si se produce un error en la creación de la máquina virtual, use los comandos Sysprep/waagent anteriores.

> [!NOTE]
> Si AIB crea una imagen personalizada de Windows correctamente y se crea una máquina virtual a partir de ella, la máquina virtual no se creará correctamente (por ejemplo, el comando de creación de la máquina virtual no se completa o se agota el tiempo de espera), deberá revisar la documentación de Sysprep de Windows Server. O bien, puede generar una solicitud de soporte técnico con el equipo de soporte técnico de los servicios de atención al cliente de Sysprep de Windows Server, que puede solucionar problemas y aconsejarle sobre el comando Sysprep correcto.

### <a name="command-locations-and-filenames"></a>Nombres de archivo y ubicaciones de comandos

Windows:

```
c:\DeprovisioningScript.ps1
```

Linux:
```bash
/tmp/DeprovisioningScript.sh
```

### <a name="sysprep-command-windows"></a>Comando Sysprep: Windows

```PowerShell
Write-Output '>>> Waiting for GA Service (RdAgent) to start ...'
while ((Get-Service RdAgent).Status -ne 'Running') { Start-Sleep -s 5 }
Write-Output '>>> Waiting for GA Service (WindowsAzureTelemetryService) to start ...'
while ((Get-Service WindowsAzureTelemetryService) -and ((Get-Service WindowsAzureTelemetryService).Status -ne 'Running')) { Start-Sleep -s 5 }
Write-Output '>>> Waiting for GA Service (WindowsAzureGuestAgent) to start ...'
while ((Get-Service WindowsAzureGuestAgent).Status -ne 'Running') { Start-Sleep -s 5 }
Write-Output '>>> Sysprepping VM ...'
if( Test-Path $Env:SystemRoot\system32\Sysprep\unattend.xml ) {
  Remove-Item $Env:SystemRoot\system32\Sysprep\unattend.xml -Force
}
& $Env:SystemRoot\System32\Sysprep\Sysprep.exe /oobe /generalize /quiet /quit
while($true) {
  $imageState = (Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Setup\State).ImageState
  Write-Output $imageState
  if ($imageState -eq 'IMAGE_STATE_GENERALIZE_RESEAL_TO_OOBE') { break }
  Start-Sleep -s 5
}
Write-Output '>>> Sysprep complete ...'
```

### <a name="deprovision-command-linux"></a>Comando de desaprovisionamiento: Linux

```bash
/usr/sbin/waagent -force -deprovision+user && export HISTSIZE=0 && sync
```

### <a name="overriding-the-commands"></a>Reemplazo de los comandos

Para reemplazar los comandos, use los aprovisionadores de scripts de PowerShell o Shell para crear los archivos de comandos con el nombre de archivo exacto y colóquelos en los directorios indicados anteriormente. Azure Image Builder lee estos comandos, y la salida se escribe en el archivo *customization.log*.

## <a name="getting-support"></a>Obtención de soporte técnico
Si aplicó las directrices y sigue sin poder solucionar el problema, puede abrir un caso de soporte técnico. Para ello, seleccione un tema de soporte y un producto adecuados, para poder interactuar con el equipo de soporte técnico de Azure VM Image Builder.

Selección del producto para el caso:
```bash
Product Family: Azure
Product: Virtual Machine Running Windows
Support Topic: Management
Support Subtopic: Issues with Azure Image Builder
```

## <a name="next-steps"></a>Pasos siguientes

Para más información, vea [Introducción a Azure Image Builder](image-builder-overview.md).

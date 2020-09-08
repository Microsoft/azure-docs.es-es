---
title: Scripts de PowerShell de ejemplo
description: Ejemplos que muestran cómo usar el front-end mediante los scripts de PowerShell
author: florianborn71
ms.author: flborn
ms.date: 02/12/2020
ms.topic: sample
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: fbac172952c9feea92341dbc028567235b9250bc
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89075280"
---
# <a name="example-powershell-scripts"></a>Scripts de PowerShell de ejemplo

Azure Remote Rendering proporciona las dos API REST siguientes:

- [API REST de conversión](../how-tos/conversion/conversion-rest-api.md)
- [API REST de sesión](../how-tos/session-rest-api.md)

El [repositorio de ejemplos de ARR](https://github.com/Azure/azure-remote-rendering) contiene scripts de ejemplo en la carpeta *Scripts* para interactuar con las API REST del servicio. En este artículo se describe su uso.

> [!CAUTION]
Llamar a las funciones de la API REST con demasiada frecuencia hará que el servidor se limite y devuelva el error finalmente. El identificador del código de error HTTP en este caso es 429 ("demasiadas solicitudes"). Como regla general, debería haber un retraso de entre **5 y 10 segundos entre las llamadas subsiguientes**.

## <a name="prerequisites"></a>Requisitos previos

Para ejecutar los scripts de ejemplo, necesita una configuración funcional de [Azure PowerShell](https://docs.microsoft.com/powershell/azure/).

1. Instale Azure PowerShell:
    1. Abra una ventana de PowerShell con derechos de administrador.
    1. Ejecute `Install-Module -Name Az -AllowClobber`.

1. Si recibe errores en la ejecución de scripts, asegúrese de que la [directiva de ejecución](https://docs.microsoft.com/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-6) está establecida correctamente:
    1. Abra una ventana de PowerShell con derechos de administrador.
    1. Ejecute `Set-ExecutionPolicy -ExecutionPolicy Unrestricted`.

1. [Prepare una cuenta de Azure Storage](../how-tos/conversion/blob-storage.md#prepare-azure-storage-accounts)

1. Inicie sesión en la suscripción que contiene la cuenta de Azure Remote Rendering:
    1. Abra una ventana de PowerShell.
    1. Ejecute: `Connect-AzAccount` y siga las instrucciones que aparecen en pantalla.

    > [!NOTE]
    > Si la organización tiene más de una suscripción, puede que tenga que especificar los argumentos de SubscriptionId y Tenant. Busque los detalles en la [documentación de Connect-AzAccount](https://docs.microsoft.com/powershell/module/az.accounts/connect-azaccount).

1. Descargue la carpeta *Scripts* desde el [repositorio de GithHub de Azure Remote Rendering](https://github.com/Azure/azure-remote-rendering).

## <a name="configuration-file"></a>Archivo de configuración

Junto a los archivos `.ps1` hay un `arrconfig.json` que tiene que rellenar:

```json
{
    "accountSettings": {
        "arrAccountId": "<fill in the account ID from the Azure Portal>",
        "arrAccountKey": "<fill in the account key from the Azure Portal>",
        "region": "<select from available regions>"
    },
    "renderingSessionSettings": {
        "vmSize": "<select standard or premium>",
        "maxLeaseTime": "<hh:mm:ss>"
    },
  "assetConversionSettings": {
    "resourceGroup": "<resource group which contains the storage account you created, only needed when uploading or generating SAS>",
    "storageAccountName": "<name of the storage account you created>",
    "blobInputContainerName": "<input container inside the storage container>",
    "blobOutputContainerName": "<output container inside the storage container>",
    "localAssetDirectoryPath": "<fill in a path to a local directory containing your asset (and files referenced from it like textures)>",
    "inputFolderPath": "<optional: base folderpath in the input container for asset upload. uses / as dir separator>",
    "inputAssetPath": "<the path to the asset under inputcontainer/inputfolderpath pointing to the input asset e.g. box.fbx>",
    "outputFolderPath": "<optional: base folderpath in the output container - the converted asset and log files will be placed here>",
    "outputAssetFileName": "<optional: filename for the converted asset, this will be placed in the output container under the outputpath>"
  }
}
```

> [!CAUTION]
> Asegúrese de realizar correctamente el escape de las barras diagonales inversas en la ruta de acceso LocalAssetDirectoryPath mediante el uso de barras diagonales inversas dobles: "\\\\" y use barras diagonales "/" en todas las demás rutas de acceso como inputFolderPath y inputAssetPath.

> [!CAUTION]
> Se deben rellenar los valores opcionales o deberá quitar la clave y el valor por completo. Por ejemplo, si no usa el parámetro `"outputAssetFileName"`, debe eliminar toda la línea dentro de `arrconfig.json`.

### <a name="accountsettings"></a>accountSettings

Para `arrAccountId` y `arrAccountKey`, consulte [Creación de una cuenta de Azure Remote Rendering](../how-tos/create-an-account.md).
Para obtener `region` consulte la [lista de regiones disponibles](../reference/regions.md).

### <a name="renderingsessionsettings"></a>renderingSessionSettings

Si quiere ejecutar **RenderingSession.ps1**, se debe rellenar esta estructura:

- **vmSize:** Selecciona el tamaño de la máquina virtual. Seleccione, [*estándar*](../reference/vm-sizes.md) o [*premium*](../reference/vm-sizes.md). Cierre las sesiones de representación cuando ya no las necesite.
- **maxLeaseTime:** La duración del tiempo de la concesión de la máquina virtual que desea realizar. Se cerrará cuando expire la concesión. El tiempo de la concesión se puede ampliar más tarde (consulte más adelante).

### <a name="assetconversionsettings"></a>assetConversionSettings

Si desea ejecutar **Conversion.ps1** tiene que rellenar esta estructura.

Para más información, consulte [Preparación de una cuenta de Azure Storage](../how-tos/conversion/blob-storage.md#prepare-azure-storage-accounts).

## <a name="script-renderingsessionps1"></a>Script: RenderingSession.ps1

Este script se usa para crear, consultar y detener sesiones de representación.

> [!IMPORTANT]
> Asegúrese de que ha rellenado las secciones *accountSettings* y *renderingSessionSettings* en arrconfig.json.

### <a name="create-a-rendering-session"></a>Creación de una sesión de representación

Uso normal con un arrconfig.json totalmente rellenado:

```PowerShell
.\RenderingSession.ps1
```

El script llamará a la [API REST de administración de sesión](../how-tos/session-rest-api.md) para poner en marcha una máquina virtual de representación con los valores especificados. Si se ejecuta correctamente, recuperará el *sessionId*. A continuación, sondeará las propiedades de la sesión hasta que esta esté preparada o se produzca un error.

Para usar un archivo de **configuración alternativo**:

```PowerShell
.\RenderingSession.ps1 -ConfigFile D:\arr\myotherconfigFile.json
```

Puede **invalidar valores individuales** del archivo de configuración:

```PowerShell
.\RenderingSession.ps1 -Region <region> -VmSize <vmsize> -MaxLeaseTime <hh:mm:ss>
```

Si desea solamente **iniciar una sesión sin sondeo**, puede usar:

```PowerShell
.\RenderingSession.ps1 -CreateSession
```

El *sessionId* que recupera el script tiene que pasarse a la mayoría de los demás comandos de sesión.

### <a name="retrieve-session-properties"></a>Recuperación de las propiedades de sesión

Para obtener las propiedades de una sesión, ejecute:

```PowerShell
.\RenderingSession.ps1 -GetSessionProperties -Id <sessionID> [-Poll]
```

Use `-Poll` para esperar hasta que la sesión esté *lista* o se produzca un error.

### <a name="list-active-sessions"></a>Enumeración de sesiones activas

```PowerShell
.\RenderingSession.ps1 -GetSessions
```

### <a name="stop-a-session"></a>Detención de una sesión

```PowerShell
.\RenderingSession.ps1 -StopSession -Id <sessionID>
```

### <a name="change-session-properties"></a>Cambio de las propiedades de una sesión

En este momento, solo se admite el cambio de maxLeaseTime de una sesión.

> [!NOTE]
> El tiempo de la concesión siempre se cuenta desde el momento en que se creó inicialmente la máquina virtual de sesión. Por lo tanto, para prorrogar el tiempo de concesión de sesión otra hora más, aumente *maxLeaseTime* en una hora.

```PowerShell
.\RenderingSession.ps1 -UpdateSession -Id <sessionID> -MaxLeaseTime <hh:mm:ss>
```

## <a name="script-conversionps1"></a>Script: Conversion.ps1

Este script se usa para convertir los modelos de entrada en el formato en tiempo de ejecución específico de Azure Remote Rendering.

> [!IMPORTANT]
> Asegúrese de que ha rellenado las secciones *accountSettings* y *assetConversionSettings* en arrconfig.json.

El script muestra las dos opciones para usar las cuentas de almacenamiento con el servicio:

- Cuenta de almacenamiento vinculada con la cuenta de Azure Remote Rendering
- Proporcionar acceso al almacenamiento mediante firmas de acceso compartido (SAS)

### <a name="linked-storage-account"></a>Cuenta de almacenamiento vinculada

Una vez que haya rellenado por completo arrconfig.json y vinculado una cuenta de almacenamiento, puede usar el siguiente comando. La vinculación de la cuenta de almacenamiento se describe en [Creación de una cuenta](../how-tos/create-an-account.md#link-storage-accounts).

La manera preferida de utilizar el servicio de conversión es usando una cuenta de almacenamiento vinculada, ya que de esta forma no es necesario generar firmas de acceso compartido.

```PowerShell
.\Conversion.ps1
```

1. Cargue todos los archivos contenidos en `assetConversionSettings.modelLocation` en el contenedor de blobs de entrada que se encuentra en el elemento `inputFolderPath` dado.
1. Llame a la [API REST de conversión de modelos](../how-tos/conversion/conversion-rest-api.md) para iniciar la [conversión de modelos](../how-tos/conversion/model-conversion.md)
1. Sondee el estado de conversión hasta que esta se realice correctamente o se produzca un error.
1. Genere los detalles de la ubicación del archivo convertido (cuenta de almacenamiento, contenedor de salida, ruta de acceso del archivo en el contenedor).

### <a name="access-to-storage-via-shared-access-signatures"></a>Acceso al almacenamiento mediante firmas de acceso compartido

```PowerShell
.\Conversion.ps1 -UseContainerSas
```

De este modo:

1. Cargue el archivo local desde `assetConversionSettings.localAssetDirectoryPath` en el contenedor de blobs de entrada.
1. Genere un URI de SAS para el contenedor de entrada.
1. Genere un URI de SAS para el contenedor de salida.
1. Llame a la [API REST de conversión de modelos](../how-tos/conversion/conversion-rest-api.md) para iniciar la [conversión de modelos](../how-tos/conversion/model-conversion.md).
1. Sondee el estado de conversión hasta que esta se realice correctamente o se produzca un error.
1. Genere los detalles de la ubicación del archivo convertido (cuenta de almacenamiento, contenedor de salida, ruta de acceso del archivo en el contenedor).
1. Genere un URI de SAS para el modelo convertido en el contenedor de blobs de salida.

### <a name="additional-command-line-options"></a>Opciones adicionales de la línea de comandos

Para usar un archivo de **configuración alternativo**:

```PowerShell
.\Conversion.ps1 -ConfigFile D:\arr\myotherconfigFile.json
```

Si desea solamente **iniciar la conversión del modelo sin sondeo**, puede usar:

```PowerShell
.\Conversion.ps1 -ConvertAsset
```

Puede **invalidar valores individuales** del archivo de configuración mediante los siguientes modificadores de línea de comandos:

* **Id:** ConversionId usado con GetConversionStatus
* **ArrAccountId:** arrAccountId de accountSettings
* **ArrAccountKey:** reemplazo de arrAccountKey de accountSettings
* **Region:** reemplazo de la región de accountSettings
* **ResourceGroup:** reemplazo de resourceGroup de assetConversionSettings
* **StorageAccountName:** reemplazo de storageAccountName de assetConversionSettings
* **BlobInputContainerName:** reemplazo de blobInputContainer de assetConversionSettings
* **LocalAssetDirectoryPath:** reemplazo de localAssetDirectoryPath de assetConversionSettings
* **InputAssetPath:** reemplazo de inputAssetPath de assetConversionSettings
* **BlobOutputContainerName:** reemplazo de blobOutputContainerName de assetConversionSettings
* **OutputFolderPath:** reemplazo de outputFolderPath de assetConversionSettings
* **OutputAssetFileName:** reemplazo de outputAssetFileName de assetConversionSettings

Por ejemplo, puede combinar varias opciones como se puede ver a continuación:

```PowerShell
.\Conversion.ps1 -LocalAssetDirectoryPath "C:\\models\\box" -InputAssetPath box.fbx -OutputFolderPath another/converted/box -OutputAssetFileName newConversionBox.arrAsset
```

### <a name="run-the-individual-conversion-stages"></a>Ejecución de las fases de conversión individuales

Si desea ejecutar pasos individuales del proceso, puede usar:

Cargue solo los datos del elemento LocalAssetDirectoryPath dado.

```PowerShell
.\Conversion.ps1 -Upload
```

Iniciar solo el proceso de conversión de un modelo ya cargado en el almacenamiento de blobs (no ejecutar la carga, no sondear el estado de conversión). El script devolverá un *conversionId*.

```PowerShell
.\Conversion.ps1 -ConvertAsset
```

Y puede recuperar el estado de conversión de esta conversión mediante:

```PowerShell
.\Conversion.ps1 -GetConversionStatus -Id <conversionId> [-Poll]
```

Use `-Poll` para esperar hasta que se realice la conversión o se produzca un error.

## <a name="next-steps"></a>Pasos siguientes

- [Inicio rápido: Representación de un modelo con Unity](../quickstarts/render-model.md)
- [Inicio rápido: Conversión de un modelo para su representación](../quickstarts/convert-model.md)
- [Conversión de modelos](../how-tos/conversion/model-conversion.md)

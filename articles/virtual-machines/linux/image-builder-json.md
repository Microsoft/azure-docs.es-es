---
title: Crear una plantilla de generador de imágenes de Azure (versión preliminar)
description: Obtenga información sobre cómo crear una plantilla para usarla con el generador de imágenes de Azure.
author: cynthn
ms.author: cynthn
ms.date: 05/10/2019
ms.topic: article
ms.service: virtual-machines-linux
manager: jeconnoc
ms.openlocfilehash: cf8264cbad3c5c88c58cff3b95cb5c68adf0686c
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65538291"
---
# <a name="preview-create-an-azure-image-builder-template"></a>Vista previa: Crear una plantilla de Azure Image Builder 

Generador de imágenes de Azure usa un archivo .json para pasar información en el servicio de generador de imágenes. En este artículo analizaremos las secciones del archivo json, por lo que puede crear las suyas propias. Para ver ejemplos de archivos .json completa, consulte el [Azure Image Builder GitHub](https://github.com/danielsollondon/azvmimagebuilder/tree/master/quickquickstarts).

Este es el formato de la plantilla básica:

```json
 { 
    "type": "Microsoft.VirtualMachineImages/imageTemplates", 
    "apiVersion": "2019-05-01-preview", 
    "location": "<region>", 
    "tags": {
        "<name": "<value>",
        "<name>": "<value>"
             }
    "identity":{},           
    "dependsOn": [], 
    "properties": { 
        "buildTimeoutInMinutes": <minutes>, 
        "build": {}, 
        "customize": {}, 
        "distribute": {} 
      } 
 } 
```



## <a name="type-and-api-version"></a>Versión de tipo y la API

El `type` es el tipo de recurso, que debe ser `"Microsoft.VirtualMachineImages/imageTemplates"`. El `apiVersion` cambiará con el tiempo como los cambios de API, pero debe ser `"2019-05-01-preview"` para vista previa.

```json
    "type": "Microsoft.VirtualMachineImages/imageTemplates",
    "apiVersion": "2019-05-01-preview",
```

## <a name="location"></a>Ubicación

La ubicación es la región donde se creará la imagen personalizada. Para la vista previa del generador de imágenes, se admiten las siguientes regiones:

- Este de EE. UU
- Este de EE. UU. 2
- Centro occidental de EE.UU.
- Oeste de EE. UU.
- Oeste de EE. UU. 2


```json
    "location": "<region>",
```
    
## <a name="depends-on-optional"></a>Depende (opcional)

En esta sección opcional se puede usar para asegurarse de que las dependencias se completan antes de continuar. 

```json
    "dependsOn": [],
```

Para obtener más información, consulte [definir dependencias de recursos](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-define-dependencies#dependson).

## <a name="identity"></a>Identidad
De forma predeterminada, admite el generador de imágenes mediante secuencias de comandos, o copiar archivos desde varias ubicaciones, como GitHub y Azure storage. Para utilizar estos métodos, debe estar accesibles públicamente.

También puede usar una identidad de administrada Azure User-Assigned, definida por el usuario, para permitir el acceso al generador de imágenes de Azure Storage, siempre que se concedió la identidad de un mínimo de 'Lector de datos de Blob de almacenamiento' en la cuenta de almacenamiento de Azure. Esto significa que no es necesario realizar los blobs de almacenamiento accesible desde el exterior, o Tokens de SAS del programa de instalación.


```json
    "identity": {
    "type": "UserAssigned",
          "userAssignedIdentities": {
            "<imgBuilderId>": {}
        }
        },
```

Para obtener un ejemplo completo, vea [ usar una identidad administrada de Azure User-Assigned acceso a los archivos de Azure Storage](https://github.com/danielsollondon/azvmimagebuilder/tree/master/quickquickstarts/7_Creating_Custom_Image_using_MSI_to_Access_Storage).

Asistencia del generador para una identidad asignada por el usuario de la imagen: • admite una sola identidad solo • no admite nombres de dominio personalizados

Para obtener más información, consulte [What ' s identidades administradas para los recursos de Azure?](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).
Para obtener más información sobre la implementación de esta característica, consulte [configurar identidades para los recursos de Azure en una máquina virtual de Azure mediante la CLI de Azure de administradas](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm#user-assigned-managed-identity).

## <a name="properties-source"></a>Propiedades: origen

El `source` sección contiene información acerca de la imagen de origen que se usará en el generador de imágenes.

La API requiere un "SourceType" que define el origen de la compilación de imagen, actualmente hay tres tipos:
- ISO - Utilice esta opción cuando el origen es un ISO de RHEL.
- PlatformImage - indica que la imagen de origen es una imagen de Marketplace.
- ManagedImage - Úselo cuando se inicia desde una imagen administrada normal.
- SharedImageVersion - se utiliza cuando se usa una versión de la imagen en una galería de imágenes compartidas como el origen.

### <a name="iso-source"></a>Origen ISO

Generador de imágenes de Azure solo admite el uso de publicado Red Hat Enterprise Linux 7.x binario DVD ISO, para la versión preliminar. Admite el generador de imágenes:
- RHEL 7.3 
- RHEL 7.4 
- RHEL 7.5 
 
```json
"source": {
       "type": "ISO",
       "sourceURI": "<sourceURI from the download center>",
       "sha256Checksum": "<checksum associated with ISO>"
}
```

Para obtener el `sourceURI` y `sha256Checksum` valores, vaya a `https://access.redhat.com/downloads` , a continuación, seleccione el producto **Red Hat Enterprise Linux**y una versión compatible. 

En la lista de **instaladores e imágenes para Red Hat Enterprise Linux Server**, también deberá copiar el vínculo de Red Hat Enterprise Linux 7.x binario DVD y la suma de comprobación.

> [!NOTE]
> Los tokens de acceso de los vínculos se actualizan a intervalos frecuentes, por lo que cada vez que desea enviar una plantilla, debe comprobar si vincula el RH ha cambiado la dirección.
 
### <a name="platformimage-source"></a>Origen PlatformImage 
Generador de imágenes de Azure admite las siguientes imágenes de Marketplace de Azure:
* Ubuntu 18.04
* Ubuntu 16.04
* RHEL 7.6
* CentOS 7.6
* Windows 2016
* Windows 2019

```json
        "source": {
            "type": "PlatformImage",
                "publisher": "Canonical",
                "offer": "UbuntuServer",
                "sku": "18.04-LTS",
                "version": "18.04.201903060"
        },
```


Las propiedades aquí son los mismos que se utilizan para crear la máquina virtual, con AZ CLI, ejecute el siguiente para obtener las propiedades: 
 
```azurecli-interactive
az vm image list -l westus -f UbuntuServer -p Canonical --output table –-all 
```

> [!NOTE]
> Versión no puede ser "latest", debe usar el comando anterior para obtener un número de versión. 

### <a name="managedimage-source"></a>Origen ManagedImage

Establece la imagen de origen como una imagen administrada existente de una máquina virtual o un disco duro virtual generalizado. La imagen administrada de origen debe ser de un sistema operativo compatible y encontrarse en la misma región que la plantilla generador de imágenes de Azure. 

```json
        "source": { 
            "type": "ManagedImage", 
                "imageId": "/subscriptions/<subscriptionId>/resourceGroups/{destinationResourceGroupName}/providers/Microsoft.Compute/images/<imageName>"
        }
```

El `imageId` debe ser el valor de ResourceId de la imagen administrada. Use `az image list` para enumerar imágenes disponibles.


### <a name="sharedimageversion-source"></a>Origen SharedImageVersion
Establece la imagen de origen en una versión de imagen existente en una galería de imágenes compartidas. La versión de la imagen debe ser de un sistema operativo compatible, y la imagen debe replicarse en la misma región que la plantilla generador de imágenes de Azure. 

```json
        "source": { 
            "type": "SharedImageVersion", 
            "imageVersionID": "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/p  roviders/Microsoft.Compute/galleries/<sharedImageGalleryName>/images/<imageDefinitionName/versions/<imageVersion>" 
   } 
```

El `imageVersionId` debe ser el valor de ResourceId de la versión de la imagen. Use [lista versión de la imagen de az sig](/cli/azure/sig/image-version#az-sig-image-version-list) a versiones de imágenes de la lista.

## <a name="properties-customize"></a>Propiedades: personalizar


Generador de imágenes admite varios 'personalizadores'. Personalizadores son funciones que se utilizan para personalizar la imagen, como ejecutar secuencias de comandos o reiniciar los servidores. 

Cuando se usa `customize`: 
- Puede usar varios personalizadores, pero deben tener un único `name`.
- Personalizadores ejecutarán en el orden especificado en la plantilla.
- Si se produce un error en uno personalizador, el componente de la personalización completa se producirá un error y notificar un error.
- Considere la posibilidad de cuánto tiempo se requieren la compilación de imagen y ajusta la propiedad 'buildTimeoutInMinutes' para permitir suficiente tiempo para completar el generador de imágenes.
- Es muy recomendable que probar exhaustivamente la secuencia de comandos antes de usar una plantilla. Depurar el script en su propia máquina virtual será más fácil.
- No coloque los datos confidenciales en las secuencias de comandos. 
- Las ubicaciones de la secuencia de comandos deben estar accesible públicamente, salvo que use [MSI](https://github.com/danielsollondon/azvmimagebuilder/tree/master/quickquickstarts/7_Creating_Custom_Image_using_MSI_to_Access_Storage).

```json
        "customize": [
            {
                "type": "Shell",
                "name": "<name>",
                "scriptUri": "<path to script>"
            },
            {
                "type": "Shell",
                "name": "<name>",
                "inline": [
                    "<command to run inline>",
                ]
            }

        ],
```     

 
La sección Personalizar es una matriz. Se ejecutará el generador de imágenes de Azure a través de los personalizadores en orden secuencial. Cualquier error en cualquier personalizador fallará el proceso de compilación. 
 
 
### <a name="shell-customizer"></a>Personalizador del shell

El personalizador de shell admite ejecutar secuencias de comandos de shell, estos deben ser accesibles públicamente para IB tener acceso a ellos.

```json
    "customize": [ 
        { 
            "type": "Shell", 
            "name": "<name>", 
            "scriptUri": "<link to script>"        
        }, 
    ], 
        "customize": [ 
        { 
            "type": "Shell", 
            "name": "<name>", 
            "inline": "<commands to run>"
        }, 
    ], 
```

Sistemas operativos admitidos: Linux 
 
Personalizar las propiedades:

- **tipo** : Shell 
- **nombre** : el nombre para el seguimiento de la personalización 
- **scriptUri** -URI a la ubicación del archivo 
- **inline** -matriz de comandos de shell, separados por comas.
 
> [!NOTE]
> Cuando se ejecuta el personalizador shell con código fuente de RHEL ISO, deberá asegurarse de los identificadores de shell personalización primer registro con un servidor de derechos de Red Hat antes de que se produce ninguna personalización. Una vez completada la personalización, debe anular el registro de la secuencia de comandos con el servidor de derechos.

### <a name="windows-restart-customizer"></a>Reinicio de Windows del personalizador 
El reinicio de personalizador permite reiniciar una máquina virtual de Windows y espere a que se vuelvan a conectarse, esto le permite instalar software que requiere un reinicio.  

```json 
     "customize": [ 
            "type{ ": "WindowsRestart", 
            "restartCommand": "shutdown /r /f /t 0 /c", 
            "restartCheckCommand": "echo Azure-Image-Builder-Restarted-the-VM  > buildArtifacts/azureImageBuilderRestart.txt",
            "restartTimeout": "5m"
         }],
```

Sistemas operativos admitidos:  Windows
 
Personalizar las propiedades:
- **Tipo**: WindowsRestart
- **restartCommand** : comando para ejecutar el reinicio (opcional). El valor predeterminado es `'shutdown /r /f /t 0 /c \"packer restart\"'`.
- **restartCheckCommand** : comando para comprobar si reinicio se realizó correctamente (opcional). 
- **restartTimeout** -reiniciar el tiempo de espera especificado como una cadena de magnitud y la unidad. Por ejemplo, `5m` (5 minutos) o `2h` (2 horas). El valor predeterminado es: '5m'


### <a name="powershell-customizer"></a>PowerShell personalizador 
El personalizador shell admite la ejecución de scripts de PowerShell y comandos en línea, las secuencias de comandos deben ser accesibles públicamente para IB tener acceso a ellos.

```json 
     "customize": [
        { 
             "type": "PowerShell",
             "name":   "<name>",  
             "scriptUri": "<path to script>" 
        },  
        { 
             "type": "PowerShell", 
             "name": "<name>", 
             "inline": "<PowerShell syntax to run>", 
             "valid_exit_codes": "<exit code>" 
         } 
    ], 
```

Sistemas operativos admitidos: Windows y Linux

Personalizar las propiedades:

- **tipo** : PowerShell.
- **scriptUri** -URI a la ubicación del archivo de script de PowerShell. 
- **inline** – en línea comandos para ejecutarse, separados por comas.
- **valid_exit_codes** : opcional, códigos válidos que pueden devolverse desde el script o en línea de comandos, esto evitará el error notificado del comando de script/en línea.

### <a name="file-customizer"></a>Personalizador del archivo

El personalizador del archivo permite que el generador de imágenes descargar un archivo desde un almacenamiento de Azure o GitHub. Si tiene una canalización de compilación de imagen que se basa en los artefactos de compilación, puede establecer el personalizador del archivo para descargar desde el recurso compartido de la compilación y mover los artefactos en la imagen.  

```json
     "customize": [ 
         { 
            "type": "File", 
             "name": "<name>", 
             "sourceUri": "<source location>",
             "destination": "<destination>" 
         }
     ]
```

Sistemas operativos admitidos: Linux y Windows 

Propiedades del archivo del personalizador:

- **sourceUri** -un punto de conexión de almacenamiento accesibles, puede tratarse de GitHub o Azure storage. Sólo se puede descargar un archivo, no un directorio completo. Si tiene que descargar un directorio, utilice un archivo comprimido, a continuación, mediante el Shell o PowerShell personalizadores descomprimir. 
- **destino** : éste es el nombre de ruta de acceso y de destino completa. Cualquier ruta de acceso que se hace referencia y sus subdirectorios deben existe, use los personalizadores Shell o PowerShell para establecerlos con antelación. Puede usar los personalizadores de secuencia de comandos para crear la ruta de acceso. 

Esto es compatible con los directorios de Windows y las rutas de acceso de Linux, pero hay algunas diferencias: 
- Sistema operativo Linux: la ruta de acceso única imagen generador puede escribir en es/tmp.
- Windows: ninguna restricción de ruta de acceso, pero la ruta de acceso deben existir.
 
 
Si se produce un error al intentar descargar el archivo o colocarla en un directorio especificado, se producirá un error en el paso de personalizar y será el customization.log.

Archivos en el personalizador del archivo se pueden descargar desde Azure Storage mediante [MSI](https://github.com/danielsollondon/azvmimagebuilder/tree/master/quickquickstarts/7_Creating_Custom_Image_using_MSI_to_Access_Storage).

### <a name="generalize"></a>Generalize 
De forma predeterminada, el generador de imágenes de Azure también se ejecutará código 'desaprovisionar' al final de cada fase de personalización de la imagen, para "generalizar" la imagen. Generalizar es un proceso donde se configura la imagen por lo que puede volver a usar para crear varias máquinas virtuales. Las máquinas virtuales de Windows Azure Image Builder utiliza Sysprep. Para Linux, se ejecuta el generador de imágenes de Azure ' waagent-deprovision'. 

Los comandos de los usuarios del generador de imágenes para generalizar pueden no ser adecuados para cada situación, por lo que el generador de imágenes de Azure, podrá personalizar este comando, si es necesario. 

Si va a migrar personalización existente y usa comandos de Sysprep/waagent diferentes, puede mediante los comandos de Image Builder genéricos y si se produce un error en la creación de máquinas virtuales, use sus propios comandos de Sysprep o de waagent.

Si el generador de imágenes de Azure crea una imagen personalizada de Windows correctamente y cree una máquina virtual desde él, a continuación, busque que la creación de máquinas virtuales, se produce un error o no se completa correctamente, deberá revisar la documentación de Sysprep de Windows Server o presente una solicitud de soporte técnico con el Equipo de soporte de servicios al cliente de Windows Server Sysprep, que puede solucionar problemas y avisar sobre el uso correcto de Sysprep.


#### <a name="default-sysprep-command"></a>Comando de Sysprep de forma predeterminada
```powershell
echo '>>> Waiting for GA to start ...'
while ((Get-Service RdAgent).Status -ne 'Running') { Start-Sleep -s 5 }
while ((Get-Service WindowsAzureTelemetryService).Status -ne 'Running') { Start-Sleep -s 5 }
while ((Get-Service WindowsAzureGuestAgent).Status -ne 'Running') { Start-Sleep -s 5 }
echo '>>> Sysprepping VM ...'
if( Test-Path $Env:SystemRoot\\windows\\system32\\Sysprep\\unattend.xml ){ rm $Env:SystemRoot\\windows\\system32\\Sysprep\\unattend.xml -Force} & $Env:SystemRoot\\System32\\Sysprep\\Sysprep.exe /oobe /generalize /quiet /quit
while($true) { $imageState = Get-ItemProperty HKLM:\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Setup\\State | Select ImageState; if($imageState.ImageState -ne 'IMAGE_STATE_GENERALIZE_RESEAL_TO_OOBE') { Write-Output $imageState.ImageState; Start-Sleep -s 5  } else { break } }
```
#### <a name="default-linux-deprovision-command"></a>Comando de desaprovisionamiento de Linux predeterminada

```bash
/usr/sbin/waagent -force -deprovision+user && export HISTSIZE=0 && sync
```

#### <a name="overriding-the-commands"></a>Reemplazar los comandos
Para reemplazar los comandos, use los aprovisionadores de script de PowerShell o el Shell para crear los archivos de comandos con el nombre de archivo exacto y colocarlos en los directorios correctos:

* Windows: c:\DeprovisioningScript.ps1
* Linux: /tmp/DeprovisioningScript.sh

Generador de imágenes leerá estos comandos, estos se escriben en los registros de AIB, 'customization.log'. Consulte [solución de problemas de](https://github.com/danielsollondon/azvmimagebuilder/blob/master/troubleshootingaib.md#collecting-and-reviewing-aib-logs) sobre cómo recopilar registros.
 
## <a name="properties-distribute"></a>Propiedades: distribuir

Generador de imágenes de Azure admite tres destinos de distribución: 

- **managedImage** : administrado imagen.
- **sharedImage** -Galería de imágenes compartidas.
- **Disco duro virtual** -VHD en una cuenta de almacenamiento.

Puede distribuir una imagen a ambos de los tipos de destino en la misma configuración, consulte [ejemplos](https://github.com/danielsollondon/azvmimagebuilder/blob/7f3d8c01eb3bf960d8b6df20ecd5c244988d13b6/armTemplates/azplatform_image_deploy_sigmdi.json#L80).

Dado que puede tener más de un destino para distribuir a, Image Builder mantiene un estado para cada destino de la distribución que puede obtenerse consultando la `runOutputName`.  El `runOutputName` es un objeto que puede consultar registrar distribución para obtener información acerca de esa distribución. Por ejemplo, puede consultar la ubicación del disco duro virtual o regiones que la versión de la imagen se replicaron a. Se trata de una propiedad de cada destino de la distribución. El `runOutputName` debe ser único para cada destino de la distribución.
 
### <a name="distribute-managedimage"></a>Distribute: managedImage

El resultado de la imagen será un recurso de imagen administrada.

```json
"distribute": [
        {
"type":"managedImage",
       "imageId": "<resource ID>",
       "location": "<region>",
       "runOutputName": "<name>",
       "artifactTags": {
            "<name": "<value>",
             "<name>": "<value>"
               }
         }]
```
 
Distribuir las propiedades:
- **type** – managedImage 
- **imageId** : Id. de recurso de la imagen de destino, se esperaba el formato: / Subscriptions /<subscriptionId>/ResourceGroups /<destinationResourceGroupName>/providers/Microsoft.Compute/images/<imageName>
- **ubicación** -ubicación de la imagen administrada.  
- **runOutputName** : único para identificar la distribución.  
- **artifactTags** -usuario opcional especifica las etiquetas de par clave-valor.
 
 
> [!NOTE]
> El grupo de recursos de destino debe existir.
> Si desea que la imagen que se distribuyen a una región distinta, aumentará el tiempo de implementación. 

### <a name="distribute-sharedimage"></a>Distribuir: sharedImage 
La Galería de imágenes de Azure compartido es un nuevo servicio de administración de imágenes que permite la administración de replicación de la región de imagen, compartir imágenes personalizadas y control de versiones. Generador de imágenes de Azure es compatible con la distribución con este servicio, por lo que puede distribuir las imágenes para las regiones compatibles con galerías de imágenes compartidas. 
 
Una galería de imágenes compartido se compone de: 
 
- Galería: contenedor para varias imágenes compartidas. Una galería se implementa en una región.
- Definiciones de imagen: una agrupación conceptual para las imágenes. 
- Versiones de imágenes: se trata de un tipo de imagen usado para implementar un conjunto de escala o la máquina virtual. Versiones de imágenes se pueden replicar en otras regiones donde las máquinas virtuales deben implementarse.
 
Antes de que se puede distribuir a la Galería de imágenes, debe crear una galería y una definición de la imagen, consulte [imágenes compartidas](shared-images.md). 

```json
{
     "type": "sharedImage",
     "galleryImageId": “<resource ID>”,
     "runOutputName": "<name>",
     "artifactTags": {
          "<name": "<value>",
           "<name>": "<value>"
             }
     "replicationRegions": [
        "<region where the gallery is deployed>",
        "<region>"
    ]}
``` 

Distribuir las propiedades de galerías de imágenes compartidas:

- **type** - sharedImage  
- **galleryImageId** : Id. de la Galería de imágenes compartidas. El formato es: / Subscriptions /<subscriptionId>/ResourceGroups /<resourceGroupName>/providers/Microsoft.Compute/galleries/<sharedImageGalleryName>/images /<imageGalleryName>.
- **runOutputName** : único para identificar la distribución.  
- **artifactTags** -usuario opcional especifica las etiquetas de par clave-valor.
- **replicationRegions** -matriz de las regiones para la replicación. Una de las regiones debe ser la región donde se implementa la galería.
 
> [!NOTE]
> Puede usar el generador de imágenes de Azure en una región distinta a la galería, pero el servicio de generador de imágenes de Azure tendrá que transferir la imagen entre los centros de datos y Esto tardará más tiempo. Generador de imágenes configurará automáticamente y de versión de la imagen, basándose en un entero que se monotónicas, no puede especificar actualmente. 

### <a name="distribute-vhd"></a>Distribuir: VHD  
Puede generar un disco duro virtual. A continuación, puede copiar el disco duro virtual y usarlo para publicar en Azure MarketPlace, o usar con Azure Stack.  

```json
 { 
     "type": "VHD",
     "runOutputName": "<VHD name>",
     "tags": {
          "<name": "<value>",
           "<name>": "<value>"
             }
 }
```
 
Sistemas operativos admitidos: Windows y Linux

Distribuir los parámetros de disco duro virtual:

- **tipo** -VHD.
- **runOutputName** : único para identificar la distribución.  
- **etiquetas** -usuario opcional especifica las etiquetas de par clave-valor.
 
Generador de imágenes de Azure no permite al usuario especificar una ubicación de la cuenta de almacenamiento, pero puede consultar el estado de la `runOutputs` para obtener la ubicación.  

```azurecli-interactive
az resource show \
   --ids "/subscriptions/$subscriptionId/resourcegroups/<imageResourceGroup>/providers/Microsoft.VirtualMachineImages/imageTemplates/<imageTemplateName>/runOutputs/<runOutputName>"  | grep artifactUri 
```

> [!NOTE]
> Una vez creado el disco duro virtual, cópielo en una ubicación distinta, tan pronto como sea posible. El disco duro virtual se almacena en una cuenta de almacenamiento en el grupo de recursos temporal creado cuando la plantilla de imagen se envía al servicio de generador de imágenes de Azure. Si elimina la plantilla de imagen, se perderá el disco duro virtual. 
 
## <a name="next-steps"></a>Pasos siguientes

Hay archivos .json de ejemplo para diferentes escenarios en los [Azure Image Builder GitHub](https://github.com/danielsollondon/azvmimagebuilder).
 
 
 
 
 
 
 
 
 
 

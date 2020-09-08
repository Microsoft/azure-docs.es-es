---
title: Creación de una plantilla de Azure Image Builder, versión preliminar
description: Obtenga información sobre cómo crear una plantilla para usarla con Azure Image Builder.
author: danielsollondon
ms.author: danis
ms.date: 08/13/2020
ms.topic: conceptual
ms.service: virtual-machines-linux
ms.subservice: imaging
ms.reviewer: cynthn
ms.openlocfilehash: 3c2dbf8c98901d5a4147939c42e289abf25f7d21
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89378379"
---
# <a name="preview-create-an-azure-image-builder-template"></a>Vista previa: Creación de una plantilla de Azure Image Builder 

Azure Image Builder utiliza un archivo .json para trasladar la información al servicio Image Builder. En este artículo analizaremos las secciones del archivo json, para que pueda compilar su propio archivo. Para ver ejemplos de archivos .json completos, consulte el [GitHub de Azure Image Builder](https://github.com/danielsollondon/azvmimagebuilder/tree/master/quickquickstarts).

Este es el formato de plantilla básico:

```json
 { 
    "type": "Microsoft.VirtualMachineImages/imageTemplates", 
    "apiVersion": "2020-02-14", 
    "location": "<region>", 
    "tags": {
        "<name": "<value>",
        "<name>": "<value>"
     },
    "identity":{},           
    "dependsOn": [], 
    "properties": { 
        "buildTimeoutInMinutes": <minutes>, 
        "vmProfile": 
            {
            "vmSize": "<vmSize>",
            "osDiskSizeGB": <sizeInGB>,
            "vnetConfig": {
                "subnetId": "/subscriptions/<subscriptionID>/resourceGroups/<vnetRgName>/providers/Microsoft.Network/virtualNetworks/<vnetName>/subnets/<subnetName>"
                }
            },
        "source": {}, 
        "customize": {}, 
        "distribute": {} 
      } 
 } 
```



## <a name="type-and-api-version"></a>Tipo y versión de API

`type` es el tipo de recurso, que debe ser `"Microsoft.VirtualMachineImages/imageTemplates"`. `apiVersion` cambiará con el tiempo a medida que cambie la API, pero debería ser `"2020-02-14"` para la versión preliminar.

```json
    "type": "Microsoft.VirtualMachineImages/imageTemplates",
    "apiVersion": "2020-02-14",
```

## <a name="location"></a>Location

La ubicación es la región donde se creará la imagen personalizada. Para la versión preliminar de Image Builder, se admiten las siguientes regiones:

- Este de EE. UU.
- Este de EE. UU. 2
- Centro-Oeste de EE. UU.
- Oeste de EE. UU.
- Oeste de EE. UU. 2
- Norte de Europa
- Oeste de Europa


```json
    "location": "<region>",
```
## <a name="vmprofile"></a>vmProfile
De forma predeterminada, el generador de imágenes usará una máquina virtual de compilación "Standard_D1_v2", que puede reemplazar. Por ejemplo, si desea personalizar una imagen para una máquina virtual de GPU, necesita un tamaño de máquina virtual de GPU. Esto es opcional.

```json
 {
    "vmSize": "Standard_D1_v2"
 },
```

## <a name="osdisksizegb"></a>osDiskSizeGB

De forma predeterminada, el generador de imágenes no cambiará el tamaño de la imagen, sino que usará el tamaño de la imagen de origen. **Solo** puede aumentar el tamaño del disco del sistema operativo (Win y Linux), opcionalmente, y un valor de 0 significa que se aplica el mismo tamaño de la imagen de origen. No se puede reducir el tamaño del disco del sistema operativo a un tamaño inferior al de la imagen de origen.

```json
 {
    "osDiskSizeGB": 100
 },
```

## <a name="vnetconfig"></a>vnetConfig
Si no especifica ninguna propiedad de la red virtual, Image Builder creará su propia red virtual, la dirección IP pública y el NSG. La dirección IP pública se usa para que el servicio se comunique con la máquina virtual de compilación; sin embargo, si no quiere que una dirección IP pública o desea que Image Builder tenga acceso a los recursos de red virtual existentes, por ejemplo, servidores de configuración (DSC, Chef, Puppet, Ansible...), recursos compartidos de archivos, etc., puede especificar una red virtual. Para obtener más información, consulte la [documentación de redes](https://github.com/danielsollondon/azvmimagebuilder/blob/master/aibNetworking.md#networking-with-azure-vm-image-builder), que es opcional.

```json
    "vnetConfig": {
        "subnetId": "/subscriptions/<subscriptionID>/resourceGroups/<vnetRgName>/providers/Microsoft.Network/virtualNetworks/<vnetName>/subnets/<subnetName>"
        }
    }
```
## <a name="tags"></a>Etiquetas

Estos son los pares clave-valor que puede especificar para la imagen que se genera.

## <a name="depends-on-optional"></a>Dependencia (opcional)

Esta sección opcional se puede usar para asegurarse de que las dependencias se completan antes de continuar. 

```json
    "dependsOn": [],
```

Para obtener más información, consulte [Definir dependencias de recursos](../../azure-resource-manager/templates/define-resource-dependency.md#dependson).

## <a name="identity"></a>Identidad

Obligatoria: para que Image Builder tenga permisos de lectura y escritura de imágenes, así como de lectura en los scripts de Azure Storage, debe crear una identidad asignada por el usuario de Azure que tenga permisos para los recursos individuales. Para más información sobre cómo funcionan los permisos de Image Builder y los pasos pertinentes, consulte la [documentación](https://github.com/danielsollondon/azvmimagebuilder/blob/master/aibPermissions.md#azure-vm-image-builder-permissions-explained-and-requirements).


```json
    "identity": {
    "type": "UserAssigned",
          "userAssignedIdentities": {
            "<imgBuilderId>": {}
        }
        },
```


Compatibilidad de Image Builder con una identidad asignada por el usuario:
* Solo admite una identidad
* No admite los nombres de dominio personalizados

Para obtener más información, consulte [¿Qué son las identidades administradas para los recursos de Azure?](../../active-directory/managed-identities-azure-resources/overview.md)
Para obtener más información sobre la implementación de esta característica, consulte [Configurar identidades administradas para los recursos de Azure en una máquina virtual de Azure mediante la CLI de Azure](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md#user-assigned-managed-identity).

## <a name="properties-source"></a>Propiedades: origen

La sección `source` contiene información acerca de la imagen de origen que usará Image Builder. Image Builder actualmente solo admite de forma nativa la creación de imágenes de Hyper-V de la generación 1 (Gen1) en el servicio Shared Image Gallery (SIG) de Azure o en una imagen administrada. Si quiere crear imágenes Gen2, debe usar una imagen de origen Gen2 y distribuirla a VHD. Después, tendrá que crear una imagen administrada a partir del disco duro virtual e insertarla en SIG como imagen Gen2.

La API requiere un "SourceType" que define el origen de la compilación de imagen. Actualmente hay tres tipos:
- PlatformImage: indicado para los casos en que la imagen de origen es una imagen de Marketplace.
- ManagedImage: use esta opción cuando empiece desde una imagen administrada normal.
- SharedImageVersion: se utiliza cuando se usa como origen una versión de la imagen de una galería de imágenes compartidas.


> [!NOTE]
> Al usar imágenes personalizadas de Windows existentes, puede ejecutar el comando Sysprep hasta 8 veces en una sola imagen de Windows. Para obtener más información, consulte la documentación de [sysprep](/windows-hardware/manufacture/desktop/sysprep--generalize--a-windows-installation#limits-on-how-many-times-you-can-run-sysprep).

### <a name="platformimage-source"></a>Origen de PlatformImage 
El generador de imágenes de Azure admite Windows Server y el cliente, así como las imágenes de Azure Marketplace de Linux. Consulte [aquí](../windows/image-builder-overview.md#os-support) para ver la lista completa. 

```json
        "source": {
            "type": "PlatformImage",
                "publisher": "Canonical",
                "offer": "UbuntuServer",
                "sku": "18.04-LTS",
                "version": "latest"
        },
```


Aquí, las propiedades son las mismas que se utilizan para crear máquinas virtuales. Mediante la CLI de Azure, ejecute lo siguiente para obtener las propiedades: 
 
```azurecli-interactive
az vm image list -l westus -f UbuntuServer -p Canonical --output table –-all 
```

Puede usar "latest" en la versión; la versión se evalúa cuando se produce la compilación de la imagen, no cuando se envía la plantilla. Si usa esta funcionalidad con el destino de Shared Image Gallery, puede evitar volver a enviar la plantilla y ejecutar de nuevo la compilación de la imagen a intervalos, por lo que las imágenes se volverán a crear a partir de las imágenes más recientes.

#### <a name="support-for-market-place-plan-information"></a>Compatibilidad con la información de un plan de Marketplace
También puede especificar la información del plan, por ejemplo:
```json
    "source": {
        "type": "PlatformImage",
        "publisher": "RedHat",
        "offer": "rhel-byos",
        "sku": "rhel-lvm75",
        "version": "latest",
        "planInfo": {
            "planName": "rhel-lvm75",
            "planProduct": "rhel-byos",
            "planPublisher": "redhat"
       }
```
### <a name="managedimage-source"></a>Origen de ManagedImage

Establece la imagen de origen como una imagen administrada existente de una máquina virtual o un disco duro virtual generalizados.

> [!NOTE]
> La imagen administrada de origen debe ser de un sistema operativo compatible y debe estar en la misma región que la plantilla de Azure Image Builder. 

```json
        "source": { 
            "type": "ManagedImage", 
                "imageId": "/subscriptions/<subscriptionId>/resourceGroups/{destinationResourceGroupName}/providers/Microsoft.Compute/images/<imageName>"
        }
```

`imageId` debe ser el valor de ResourceId de la imagen administrada. Use `az image list` para enumerar las imágenes disponibles.


### <a name="sharedimageversion-source"></a>Origen de SharedImageVersion
Establece la imagen de origen en una versión de la imagen existente de una galería de imágenes compartidas.

> [!NOTE]
> La imagen administrada de origen debe ser de un sistema operativo compatible y debe estar en la misma región que la plantilla de Azure Image Builder. Si no es así, replique la versión de la imagen en la región de la plantilla de Image Builder.


```json
        "source": { 
            "type": "SharedImageVersion", 
            "imageVersionID": "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/p  roviders/Microsoft.Compute/galleries/<sharedImageGalleryName>/images/<imageDefinitionName/versions/<imageVersion>" 
   } 
```

El `imageVersionId` debe ser el valor de ResourceId de la versión de la imagen. Use [lista de versiones de imagen con firma de Azure](/cli/azure/sig/image-version#az-sig-image-version-list) para obtener una lista de las versiones de imagen.


## <a name="properties-buildtimeoutinminutes"></a>Propiedades: buildTimeoutInMinutes

De manera predeterminada, Image Builder se ejecutará durante 240 minutos. Después de esto, se agotará el tiempo de espera y se detendrá, así se haya completado la compilación de la imagen como si no. Si se alcanza el tiempo de espera, verá un error similar al siguiente:

```text
[ERROR] Failed while waiting for packerizer: Timeout waiting for microservice to
[ERROR] complete: 'context deadline exceeded'
```

Si no se especifica un valor de buildTimeoutInMinutes o se establece en 0, se usará el valor predeterminado. Puede aumentar o disminuir el valor, hasta el máximo de 960 min (16 horas). En Windows, no se recomienda establecer este valor por debajo de 60 minutos. Si alcanza el tiempo de espera, revise los [registros](https://github.com/danielsollondon/azvmimagebuilder/blob/master/troubleshootingaib.md#collecting-and-reviewing-aib-image-build-logs) para ver si el paso de personalización está esperando algo, como la entrada del usuario. 

Si necesita más tiempo para que se completen las personalizaciones, establezca el valor en lo que crea que necesita, con una pequeña sobrecarga. Pero no lo establezca demasiado alto, ya que es posible que tenga que esperar a que se agote el tiempo de espera antes de ver un error. 


## <a name="properties-customize"></a>Propiedades: personalización

Image Builder admite varios "personalizadores". Los personalizadores son funciones que se utilizan para personalizar la imagen, como, por ejemplo, ejecutar scripts o reiniciar los servidores. 

Al usar `customize`: 
- Puede usar varios personalizadores, pero deben tener un único `name`.
- Los personalizadores se ejecutan en el orden especificado en la plantilla.
- Si se produce un error en un personalizador, todo el componente de personalización producirá un error e informará de un error.
- Es muy recomendable que pruebe exhaustivamente el script antes de usarlo en una plantilla. Será más fácil depurar el script en su propia máquina virtual.
- No incluya información confidencial en los scripts. 
- Las ubicaciones de los scripts deben estar accesibles públicamente, a menos que esté usando [MSI](https://github.com/danielsollondon/azvmimagebuilder/tree/master/quickquickstarts/7_Creating_Custom_Image_using_MSI_to_Access_Storage).

```json
        "customize": [
            {
                "type": "Shell",
                "name": "<name>",
                "scriptUri": "<path to script>",
                "sha256Checksum": "<sha256 checksum>"
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

 
La sección de personalización es una matriz. Azure Image Builder recorrerá los personalizadores en orden secuencial. Cualquier error en un personalizador producirá un error en el proceso de compilación. 

> [!NOTE]
> Los comandos alineados se pueden ver en la definición de la plantilla de imagen; además, el soporte técnico de Microsoft puede verlos al ayudarle con un caso de soporte técnico. Si tiene información confidencial, se debe pasar a scripts en Azure Storage, ya que el acceso requiere autenticación.
 
### <a name="shell-customizer"></a>Personalizador de shell

El personalizador de shell admite la ejecución de scripts de shell, que deben estar accesibles públicamente para que IB pueda acceder a ellos.

```json
    "customize": [ 
        { 
            "type": "Shell", 
            "name": "<name>", 
            "scriptUri": "<link to script>",
            "sha256Checksum": "<sha256 checksum>"       
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

SO compatible: Linux 
 
Propiedades de personalización:

- **tipo**: Shell 
- **nombre**: nombre para realizar el seguimiento de la personalización 
- **scriptUri**: URI a la ubicación del archivo 
- **inline**: matriz de comandos de shell, separados por comas.
- **sha256Checksum**: valor de la suma de comprobación sha256 del archivo, se genera de forma local y, a continuación, Image Builder realizará la suma de comprobación y la validación.
    * Para generar el valor de sha256Checksum, mediante un terminal en Mac o Linux ejecute `sha256sum <fileName>`


Para que los comandos se ejecuten con privilegios de superusuario, deben tener el prefijo `sudo`.

> [!NOTE]
> Los comandos alineados se almacenan como parte de la definición de la plantilla de imagen; puede verlos al volcar la definición de la imagen y también son visibles para el soporte técnico de Microsoft en el caso de una situación de soporte técnico para la solución de problemas. Si tiene comandos o valores confidenciales, se recomienda moverlos a scripts y usar una identidad de usuario para autenticarse en Azure Storage.

### <a name="windows-restart-customizer"></a>Personalizador de reinicio de Windows 
El personalizador de reinicio le permite reiniciar una máquina virtual de Windows y esperar a que vuelva a conectarse, lo que le permite instalar software que requiere un reinicio.  

```json 
     "customize": [ 

            {
                "type": "WindowsRestart",
                "restartCommand": "shutdown /r /f /t 0", 
                "restartCheckCommand": "echo Azure-Image-Builder-Restarted-the-VM  > c:\\buildArtifacts\\azureImageBuilderRestart.txt",
                "restartTimeout": "5m"
            }
  
        ],
```

SO compatible: Windows
 
Propiedades de personalización:
- **Tipo**: WindowsRestart
- **restartCommand**: comando para ejecutar el reinicio (opcional). El valor predeterminado es `'shutdown /r /f /t 0 /c \"packer restart\"'`.
- **restartCheckCommand**: comando para comprobar si el reinicio se realizó correctamente (opcional). 
- **restartTimeout**: tiempo de expiración del reinicio especificado como una cadena de magnitud y unidad. Por ejemplo, `5m` (5 minutos) o `2h` (2 horas). El valor predeterminado es: "5 m"

### <a name="linux-restart"></a>Reinicio de Linux  
No hay ningún personalizador de reinicio de Linux; sin embargo, si está instalando controladores o componentes que requieren un reinicio, puede instalarlos e invocar un reinicio mediante el personalizador de Shell. Para ello, hay un tiempo de espera de SSH de 20 min en la máquina virtual de compilación.

### <a name="powershell-customizer"></a>Personalizador de PowerShell 
El personalizador de shell admite la ejecución de scripts de PowerShell y comandos alineados; los scripts deben estar accesibles públicamente para que IB pueda acceder a ellos.

```json 
     "customize": [
        { 
             "type": "PowerShell",
             "name":   "<name>",  
             "scriptUri": "<path to script>",
             "runElevated": "<true false>",
             "sha256Checksum": "<sha256 checksum>" 
        },  
        { 
             "type": "PowerShell", 
             "name": "<name>", 
             "inline": "<PowerShell syntax to run>", 
             "validExitCodes": "<exit code>",
             "runElevated": "<true or false>" 
         } 
    ], 
```

SO compatible: Windows y Linux

Propiedades de personalización:

- **type**: PowerShell.
- **scriptUri**: URI a la ubicación del archivo de script de PowerShell. 
- **inline**: comandos alineados que se ejecutarán, separados por comas.
- **validExitCodes**: opcional, códigos válidos que pueden devolverse desde el comando de script o alineado; esto evitará que se informe de un error del comando de script o alineado.
- **runElevated**: opcional, booleano, admite la ejecución de comandos y scripts con permisos elevados.
- **sha256Checksum**: valor de la suma de comprobación sha256 del archivo, se genera de forma local y, a continuación, Image Builder realizará la suma de comprobación y la validación.
    * Para generar el valor de sha256Checksum, con PowerShell en Windows utilice [Get-hash](/powershell/module/microsoft.powershell.utility/get-filehash?view=powershell-6)


### <a name="file-customizer"></a>Personalizador de archivos

El personalizador de archivos permite que el generador de imágenes descargue un archivo desde un almacenamiento de GitHub o Azure Storage. Si tiene una canalización de compilación de imagen basada en artefactos de compilación, puede configurar el personalizador de archivos para que se descargue desde el recurso compartido de la compilación y mover los artefactos a la imagen.  

```json
     "customize": [ 
         { 
            "type": "File", 
             "name": "<name>", 
             "sourceUri": "<source location>",
             "destination": "<destination>",
             "sha256Checksum": "<sha256 checksum>"
         }
     ]
```

SO compatible: Linux y Windows 

Propiedades del personalizador de archivos:

- **sourceUri**: un punto de conexión de almacenamiento accesible, puede tratarse de GitHub o Azure Storage. Solo puede descargar un archivo, no un directorio completo. Si necesita descargar un directorio, utilice un archivo comprimido y, a continuación, descomprímalo mediante los personalizadores de Shell o PowerShell. 
- **destination**: es la ruta de acceso completa y el nombre de archivo de destino. Todas las rutas de acceso y los subdirectorios a los que se hace referencia deben existir; use los personalizadores de Shell o PowerShell para establecerlos con antelación. Puede usar los personalizadores de script para crear la ruta de acceso. 

Los directorios de Windows y las rutas de acceso de Linux lo admiten, aunque hay algunas diferencias: 
- Sistema operativo Linux: la única ruta de acceso en la que el generador de imágenes puede escribir es /tmp.
- Windows: no hay ninguna restricción en la ruta de acceso, pero esta debe existir.
 
 
Si se produce un error al intentar descargar el archivo o colocarlo en un directorio concreto, se producirá un error en el paso de personalización, que se registrará en customization.log.

> [!NOTE]
> el personalizador de archivos solo es adecuado para descargas de archivos pequeños, inferiores a 20 MB. Para descargas de archivos más grandes, use un script o un comando insertado, el código de uso para descargar archivos, como `wget` o `curl` de Linux, o `Invoke-WebRequest` de Windows.

Los archivos del personalizador de archivos se pueden descargar desde Azure Storage mediante [MSI](https://github.com/danielsollondon/azvmimagebuilder/tree/master/quickquickstarts/7_Creating_Custom_Image_using_MSI_to_Access_Storage).

### <a name="windows-update-customizer"></a>Personalizador de Windows Update
Este personalizador se basa en el [aprovisionador de Windows Update de la comunidad](https://packer.io/docs/provisioners/community-supported.html) para Packer, que es un proyecto de código abierto que mantiene la comunidad de Packer. Microsoft comprueba y valida el aprovisionamiento con el servicio Image Builder, y permite investigar problemas con él, así como trabajar para resolver problemas, pero Microsoft no admite oficialmente el proyecto de código abierto. Para obtener documentación detallada y ayuda con el aprovisionamiento de Windows Update, vea el repositorio del proyecto.

```json
     "customize": [
            {
                "type": "WindowsUpdate",
                "searchCriteria": "IsInstalled=0",
                "filters": [
                    "exclude:$_.Title -like '*Preview*'",
                    "include:$true"
                            ],
                "updateLimit": 20
            }
               ], 
OS support: Windows
```

Propiedades de personalización:
- **type**: WindowsUpdate.
- **searchCriteria**: (opcional) define qué tipo de actualizaciones se instalan (recomendadas, importantes, etc.); BrowseOnly=0 e IsInstalled=0 (recomendado) son los valores predeterminados.
- **filters**: (opcional) permite especificar un filtro para incluir o excluir actualizaciones.
- **updateLimit**: (opcional) define el número de actualizaciones que se pueden instalar, el valor predeterminado es 1000.
 
> [!NOTE]
> Se puede producir un error en el personalizador de Windows Update si hay algún reinicio de Windows pendiente, o bien si hay instalaciones de aplicaciones que todavía se están ejecutando. Normalmente, verá este error en el archivo customization.log, `System.Runtime.InteropServices.COMException (0x80240016): Exception from HRESULT: 0x80240016`. Le recomendamos encarecidamente que considere la posibilidad de agregar un reinicio de Windows o permitir el tiempo suficiente para completar las instalaciones mediante los comandos [sleep] o wait (https://docs.microsoft.com/powershell/module/microsoft.powershell.utility/start-sleep?view=powershell-7) en los comandos o scripts insertados antes de ejecutar Windows Update.

### <a name="generalize"></a>Generalize 
De forma predeterminada, Azure Image Builder también ejecutará código de "desaprovisionamiento" al final de cada fase de personalización de la imagen con el fin de "generalizar" la imagen. La generalización es un proceso en el que la imagen se configura para que pueda volver a usarse para crear varias máquinas virtuales. Para las máquinas virtuales de Windows, Azure Image Builder utiliza Sysprep. Para Linux, Azure Image Builder ejecuta "waagent-deprovision". 

Es posible que los comandos que utiliza Image Builder para la generalización no sean adecuados para todas las situaciones, por lo que Azure Image Builder le permitirá personalizar este comando si es necesario. 

Si está migrando una personalización existente y usa comandos Sysprep/waagent diferentes, puede usar los comandos genéricos de Image Builder y, si la creación de la máquina virtual produce un error, usar sus propios comandos Sysprep o waagent.

Si Azure Image Builder crea una imagen personalizada de Windows correctamente y usted crea una máquina virtual a partir de ella pero la creación produce un error o no se realiza correctamente, deberá revisar la documentación de Windows Server Sysprep o presentar una solicitud de soporte técnico al equipo de soporte técnico del Servicio al cliente de Windows Server Sysprep, quien podrá solucionar problemas y orientarlo en la utilización correcta de Sysprep.


#### <a name="default-sysprep-command"></a>Comando de Sysprep predeterminado
```powershell
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
#### <a name="default-linux-deprovision-command"></a>Comando de desaprovisionamiento de Linux predeterminado

```bash
/usr/sbin/waagent -force -deprovision+user && export HISTSIZE=0 && sync
```

#### <a name="overriding-the-commands"></a>Reemplazo de los comandos
Para reemplazar los comandos, use los aprovisionadores de scripts de PowerShell o Shell para crear los archivos de comandos con el nombre de archivo exacto y colóquelos en los directorios correctos:

* Windows: c:\DeprovisioningScript.ps1
* Linux: /tmp/DeprovisioningScript.sh

Image Builder leerá estos comandos, que se escriben en los registros de AIB, "customization.log". Consulte la [solución de problemas](https://github.com/danielsollondon/azvmimagebuilder/blob/master/troubleshootingaib.md#collecting-and-reviewing-aib-logs) sobre cómo recopilar registros.
 
## <a name="properties-distribute"></a>Propiedades: distribución

Azure Image Builder admite tres destinos de distribución: 

- **managedImage**: imagen administrada.
- **sharedImage**: galería de imágenes compartidas.
- **VHD**: disco duro virtual en una cuenta de almacenamiento.

Puede distribuir una imagen a ambos tipos de destino en la misma configuración.

> [!NOTE]
> El comando sysprep predeterminado de AIB no incluye "/mode:vm"; sin embargo, puede que sea necesario al crear imágenes que vayan a tener instalado el rol de HyperV. Si necesita agregar este argumento de comando, debe invalidar el comando sysprep.

Dado que puede tener más de un destino al que distribuir, Image Builder mantiene un estado para cada destino de distribución al que puede accederse consultando el `runOutputName`.  El `runOutputName` es un objeto que puede consultar después de la distribución para obtener información acerca de esa distribución. Por ejemplo, puede consultar la ubicación del disco duro virtual o las regiones a las que se ha replicado la versión de la imagen o donde se ha creado la versión de la imagen de SIG. Se trata de una propiedad de cada destino de distribución. El `runOutputName` debe ser único para cada destino de distribución. Este es un ejemplo, en el que se consulta una distribución de Shared Image Gallery:

```bash
subscriptionID=<subcriptionID>
imageResourceGroup=<resourceGroup of image template>
runOutputName=<runOutputName>

az resource show \
        --ids "/subscriptions/$subscriptionID/resourcegroups/$imageResourceGroup/providers/Microsoft.VirtualMachineImages/imageTemplates/ImageTemplateLinuxRHEL77/runOutputs/$runOutputName"  \
        --api-version=2020-02-14
```

Salida:
```json
{
  "id": "/subscriptions/xxxxxx/resourcegroups/rheltest/providers/Microsoft.VirtualMachineImages/imageTemplates/ImageTemplateLinuxRHEL77/runOutputs/rhel77",
  "identity": null,
  "kind": null,
  "location": null,
  "managedBy": null,
  "name": "rhel77",
  "plan": null,
  "properties": {
    "artifactId": "/subscriptions/xxxxxx/resourceGroups/aibDevOpsImg/providers/Microsoft.Compute/galleries/devOpsSIG/images/rhel/versions/0.24105.52755",
    "provisioningState": "Succeeded"
  },
  "resourceGroup": "rheltest",
  "sku": null,
  "tags": null,
  "type": "Microsoft.VirtualMachineImages/imageTemplates/runOutputs"
}
```

### <a name="distribute-managedimage"></a>Distribución: managedImage

El resultado de la imagen será un recurso de imagen administrada.

```json
{
       "type":"managedImage",
       "imageId": "<resource ID>",
       "location": "<region>",
       "runOutputName": "<name>",
       "artifactTags": {
            "<name": "<value>",
            "<name>": "<value>"
        }
}
```
 
Propiedades de la distribución:
- **type**: managedImage 
- **imageId**: id. de recurso de la imagen de destino; el formato previsto es: /subscriptions/\<subscriptionId>/resourceGroups/\<destinationResourceGroupName>/providers/Microsoft.Compute/images/\<imageName>
- **location**: ubicación de la imagen administrada.  
- **runOutputName**: nombre único para identificar la distribución.  
- **artifactTags**: etiquetas de par clave-valor opcionales especificadas por el usuario.
 
 
> [!NOTE]
> El grupo de recursos de destino debe existir.
> Si quiere que la imagen se distribuya a otra región, el tiempo de implementación será mayor. 

### <a name="distribute-sharedimage"></a>Distribución: sharedImage 
La Galería de imágenes compartidas de Azure es un nuevo servicio de administración de imágenes que permite administrar la replicación de las regiones de la imagen, el control de versiones y el uso compartido de imágenes personalizadas. Azure Image Builder admite la distribución con este servicio, por lo que puede distribuir imágenes a las regiones admitidas por las galerías de imágenes compartidas. 
 
Una galería de imágenes compartidas tiene los siguientes componentes: 
 
- Galería: contenedor para varias imágenes compartidas. Una galería se implementa en una región.
- Definiciones de imagen: una agrupación conceptual para las imágenes. 
- Versiones de las imágenes: se trata de un tipo de imagen usado para implementar una máquina virtual o un conjunto de escalado. Las versiones de las imágenes se pueden replicar a otras regiones en las que deban implementarse máquinas virtuales.
 
Antes de poder distribuir a la Galería de imágenes, debe crear una galería y una definición de imagen; para ello, consulte [Imágenes compartidas](../shared-images-cli.md). 

```json
{
    "type": "SharedImage",
    "galleryImageId": "<resource ID>",
    "runOutputName": "<name>",
    "artifactTags": {
        "<name>": "<value>",
        "<name>": "<value>"
    },
    "replicationRegions": [
        "<region where the gallery is deployed>",
        "<region>"
    ]
}
``` 

Propiedades de la distribución para las galerías de imágenes compartidas:

- **tipo**: sharedImage  
- **galleryImageId**: id. de la galería de imágenes compartidas, que puede especificarse en dos formatos:
    * Control de versiones automático: Image Builder generará un número de versión monotónico de forma automática, que resulta útil cuando quiere seguir compilando imágenes a partir de la misma plantilla: El formato es: `/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/galleries/<sharedImageGalleryName>/images/<imageGalleryName>`.
    * Control de versiones explícito: puede pasar el número de versión que quiera que Image Builder use. El formato es: `/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/<sharedImageGalName>/images/<imageDefName>/versions/<version e.g. 1.1.1>`.

- **runOutputName**: nombre único para identificar la distribución.  
- **artifactTags**: etiquetas de par clave-valor opcionales especificadas por el usuario.
- **replicationRegions**: matriz de regiones para la replicación. Una de las regiones debe ser la región en la que está implementada la galería. La adición de regiones implicará un aumento en el tiempo de compilación, ya que esta no se completa hasta que la replicación se haya completado.
- **excludeFromLatest** (opcional). Esto le permite marcar la versión de la imagen que haya creado para que no se use como última versión en la definición de SIG; el valor predeterminado es "false".
- **storageAccountType** (opcional). AIB admite la especificación de estos tipos de almacenamiento para la versión de la imagen que se va a crear:
    * "Standard_LRS"
    * "Standard_ZRS"


> [!NOTE]
> Si la plantilla de imagen y la definición `image definition` a la que se hace referencia no están en la misma ubicación, se necesitará tiempo adicional para crear las imágenes. En la actualidad, Image Builder no tiene un parámetro `location` para el recurso de versión de la imagen, este se toma de su `image definition` primaria. Por ejemplo, si una definición de imagen está en la región westus y quiere que la versión de la imagen se replique en eastus, se copia un blob en westus, se crea un recurso de versión de imagen a partir de él en westus y, a continuación, se replica en eastus. Para evitar el tiempo de replicación adicional, asegúrese de que la definición `image definition` y la plantilla de imagen se encuentren en la misma ubicación.


### <a name="distribute-vhd"></a>Distribución: VHD  
Puede generar un disco duro virtual. A continuación, puede copiar el disco duro virtual y usarlo para publicar en Azure MarketPlace o usarlo con Azure Stack.  

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
 
SO compatible: Windows y Linux

Parámetros de distribución de VHD:

- **tipo**: VHD.
- **runOutputName**: nombre único para identificar la distribución.  
- **etiquetas**: etiquetas de par clave-valor opcionales especificadas por el usuario.
 
Azure Image Builder no permite al usuario especificar una ubicación de la cuenta de almacenamiento, pero puede consultar el estado de `runOutputs` para obtener la ubicación.  

```azurecli-interactive
az resource show \
   --ids "/subscriptions/$subscriptionId/resourcegroups/<imageResourceGroup>/providers/Microsoft.VirtualMachineImages/imageTemplates/<imageTemplateName>/runOutputs/<runOutputName>"  | grep artifactUri 
```

> [!NOTE]
> Una vez creado el disco duro virtual, cópielo en una ubicación distinta tan pronto como sea posible. El disco duro virtual se almacena en una cuenta de almacenamiento del grupo de recursos temporal creado al enviar la plantilla de imagen al servicio Azure Image Builder. Si elimina la plantilla de imagen, perderá el disco duro virtual. 

## <a name="image-template-operations"></a>Operaciones de plantilla de imagen

### <a name="starting-an-image-build"></a>Inicio de una compilación de imagen
Para iniciar una compilación, debe invocar el comando "run" en el recurso de plantilla de imagen; estos son ejemplos de comandos `run`:

```PowerShell
Invoke-AzResourceAction -ResourceName $imageTemplateName -ResourceGroupName $imageResourceGroup -ResourceType Microsoft.VirtualMachineImages/imageTemplates -ApiVersion "2020-02-14" -Action Run -Force
```


```bash
az resource invoke-action \
     --resource-group $imageResourceGroup \
     --resource-type  Microsoft.VirtualMachineImages/imageTemplates \
     -n helloImageTemplateLinux01 \
     --action Run 
```

### <a name="cancelling-an-image-build"></a>Cancelación de una compilación de imagen
Si va a ejecutar una compilación de imagen que cree que es incorrecta y en espera de la entrada del usuario, o si cree que no se completará correctamente, puede cancelar la compilación.

La compilación se puede cancelar en cualquier momento. Si se ha iniciado la fase de distribución, aún puede cancelar la operación, pero deberá limpiar todas las imágenes que no se hayan completado. El comando de cancelación no espera a que se complete la cancelación, así que debe supervisar el valor de `lastrunstatus.runstate` para cancelar el progreso, mediante estos [comandos](https://github.com/danielsollondon/azvmimagebuilder/blob/master/troubleshootingaib.md#get-statuserror-of-the-template-submission-or-template-build-status) de estado.


Ejemplos de comandos `cancel`:

```powerShell
Invoke-AzResourceAction -ResourceName $imageTemplateName -ResourceGroupName $imageResourceGroup -ResourceType Microsoft.VirtualMachineImages/imageTemplates -ApiVersion "2020-02-14" -Action Cancel -Force
```

```bash
az resource invoke-action \
     --resource-group $imageResourceGroup \
     --resource-type  Microsoft.VirtualMachineImages/imageTemplates \
     -n helloImageTemplateLinux01 \
     --action Cancel 
```

## <a name="next-steps"></a>Pasos siguientes

Encontrará archivos .json de ejemplo para diferentes escenarios en el [GitHub de Azure Image Builder](https://github.com/danielsollondon/azvmimagebuilder).

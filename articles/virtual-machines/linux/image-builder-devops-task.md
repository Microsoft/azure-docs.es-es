---
title: Tarea de DevOps del servicio Azure Image Builder
description: Tarea de Azure DevOps para insertar artefactos de compilación en una imagen de máquina virtual, para que pueda instalar y configurar la aplicación y el sistema operativo
author: danielsollondon
ms.author: danis
ms.date: 08/10/2020
ms.topic: article
ms.service: virtual-machines
ms.subservice: imaging
ms.openlocfilehash: 9f948fcc8ad36f8bef8b1ab6a1b74131faea9bd3
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88068045"
---
# <a name="azure-image-builder-service-devops-task"></a>Tarea de DevOps del servicio Azure Image Builder

En este artículo se explica cómo usar una tarea de Azure DevOps para insertar artefactos de compilación en una imagen de máquina virtual, para que pueda instalar y configurar la aplicación y el sistema operativo.

## <a name="devops-task-versions"></a>Versiones de la tarea de DevOps
Hay dos tareas de DevOps de Azure VM Image Builder (AIB):

* [Tarea de AIB "estable"](https://marketplace.visualstudio.com/items?itemName=AzureImageBuilder.devOps-task-for-azure-image-builder): permite agregar las últimas actualizaciones y características, permitir a los clientes probarlas antes de promocionarlas a la tarea "estable", aproximadamente una semana después. 


* [Tarea de AIB "inestable"](https://marketplace.visualstudio.com/items?itemName=AzureImageBuilder.devOps-task-for-azure-image-builder-canary): nos permite agregar las últimas actualizaciones y características, permitir a los clientes probarlas antes de promocionarlas a la tarea "estable". Si no se ha notificado ningún problema y los datos de telemetría no muestran ningún problema, aproximadamente una semana después, promocionaremos el código de la tarea a "estable". 

## <a name="prerequisites"></a>Prerrequisitos

* Instale la [tarea de DevOps estable de Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=AzureImageBuilder.devOps-task-for-azure-image-builder).
* Debe tener una cuenta de VSTS DevOps y una canalización de compilación creada.
* Registre y habilite los requisitos de las características de Image Builder en la suscripción que usan las canalizaciones:
    * [Az PowerShell](https://docs.microsoft.com/azure/virtual-machines/windows/image-builder-powershell#register-features)
    * [Az CLI](https://docs.microsoft.com/azure/virtual-machines/windows/image-builder#register-the-features)
    
* Cree una cuenta de Azure Storage estándar en el grupo de recursos de la imagen de origen; puede usar otras cuentas de almacenamiento o grupo de recursos. La cuenta de almacenamiento se usa para transferir los artefactos de compilación de la tarea de DevOps a la imagen.

    ```powerShell
    # Az PowerShell
    $timeInt=$(get-date -UFormat "%s")
    $storageAccName="aibstorage"+$timeInt
    $location=westus
    # create storage account and blob in resource group
    New-AzStorageAccount -ResourceGroupName $strResourceGroup -Name $storageAccName -Location $location -SkuName Standard_LRS
    ```

    ```bash
    # Az CLI
    location=westus
    scriptStorageAcc=aibstordot$(date +'%s')
    # create storage account and blob in resource group
    az storage account create -n $scriptStorageAcc -g $strResourceGroup -l $location --sku Standard_LRS
    ```

## <a name="add-task-to-release-pipeline"></a>Adición de la tarea a la canalización de versión

Seleccione **Canalización de versión** > **Editar**.

En el agente de usuario, seleccione *+* para agregarlo y, a continuación, busque **Image Builder**. Seleccione **Agregar**.

Establezca las propiedades de la tarea siguientes:

### <a name="azure-subscription"></a>Suscripción de Azure

En el menú desplegable, seleccione la suscripción donde desea que se ejecute Image Builder. Use la misma suscripción en la que se encuentran las imágenes de origen y donde se distribuyen las imágenes. Debe autorizar el acceso de colaborador de Image Builder a la suscripción o al grupo de recursos.

### <a name="resource-group"></a>Grupo de recursos

Use el grupo de recursos donde se almacenará el artefacto de plantilla de imagen temporal. Al crear un artefacto de plantilla, se crea un grupo de recursos `IT_<DestinationResourceGroup>_<TemplateName>_guid` de Image Builder temporal adicional. El grupo de recursos temporal almacena los metadatos de la imagen, como scripts. Al final de la tarea, se eliminan el artefacto de la plantilla de imagen y el grupo de recursos de Image Builder temporal.
 
### <a name="location"></a>Location

La ubicación es la región donde se ejecutará Image Builder. Solo se admite un número establecido de [regiones](https://docs.microsoft.com/azure/virtual-machines/windows/image-builder-overview#regions). Las imágenes de origen deben estar presentes en esta ubicación. Por ejemplo, si usa Shared Image Gallery, debe existir una réplica en esa región.

### <a name="managed-identity-required"></a>Identidad administrada (requerida)
Image Builder requiere una identidad administrada, que se usa para leer imágenes personalizadas de origen, conectarse a Azure Storage y crear imágenes personalizadas. Obtenga más información [aquí](https://aka.ms/azvmimagebuilder#permissions).

### <a name="vnet-support"></a>Compatibilidad de redes virtuales

Actualmente, la tarea de DevOps no permite especificar una subred existente; está en la hoja de ruta, pero si desea utilizar una red virtual existente, puede usar una plantilla de ARM, con una plantilla de Image Builder anidada dentro de ella; vea los ejemplos de la plantilla de Image Builder con Windows sobre cómo se consigue, o bien use [AZ AIB PowerShell](https://docs.microsoft.com/azure/virtual-machines/windows/image-builder-powershell).

### <a name="source"></a>Source

Las imágenes de origen debe ser del sistema operativo de Image Builder compatible. Puede elegir imágenes personalizadas existentes en la misma región desde la que se ejecuta Image Builder:
* Imagen administrada: debe pasar el identificador de recurso, por ejemplo:
    ```json
    /subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/images/<imageName>
    ```
* Azure Shared Image Gallery: debe pasar el identificador de recurso de la versión de la imagen, por ejemplo:
    ```json
    /subscriptions/$subscriptionID/resourceGroups/$sigResourceGroup/providers/Microsoft.Compute/galleries/$sigName/images/$imageDefName/versions/<versionNumber>
    ```

    Si necesita obtener la última versión de Shared Image Gallery, puede tener una tarea AZ PowerShell o AZ CLI antes de que obtenga la última versión y establezca una variable de DevOps. Use la variable en la tarea AZ VM de DevOps de Image Builder. Para obtener más información, vea los [ejemplos](https://github.com/danielsollondon/azvmimagebuilder/tree/master/solutions/8_Getting_Latest_SIG_Version_ResID#getting-the-latest-image-version-resourceid-from-shared-image-gallery).

* Imagen base (Marketplace): hay una lista desplegable de imágenes populares; estas siempre usarán la "última" versión de los sistemas operativos compatibles. 

    Si la imagen base no está en la lista, puede especificar la imagen exacta mediante `Publisher:Offer:Sku`.

    Versión de la imagen base (opcional): puede proporcionar la versión de la imagen que desea usar; el valor predeterminado es `latest`.

### <a name="customize"></a>Personalizar

#### <a name="provisioner"></a>Aprovisionador

Inicialmente, se admiten dos personalizadores: **Shell** y **PowerShell**. Solo se admite la inserción. Si desea descargar scripts, puede pasar comandos insertados para hacerlo.

En el sistema operativo, seleccione PowerShell o Shell.

#### <a name="windows-update-task"></a>Tarea de Windows Update

Solo para Windows, la tarea ejecuta Windows Update al final de las personalizaciones. Controla los reinicios necesarios.

Se ejecuta la siguiente configuración de Windows Update:
```json
    "type": "WindowsUpdate",
    "searchCriteria": "IsInstalled=0",
    "filters": [
        "exclude:$_.Title -like '*Preview*'",
        "include:$true"
```
Instala las actualizaciones de Windows importantes y recomendadas que no están en versión preliminar.

#### <a name="handling-reboots"></a>Control de los reinicios
Actualmente, la tarea de DevOps no admite el reinicio de las compilaciones de Windows; si intenta reiniciar con el código de PowerShell, se producirá un error en la compilación. Sin embargo, puede usar código para reiniciar las compilaciones de Linux.

#### <a name="build-path"></a>Ruta de acceso de compilación

La tarea está diseñada para poder insertar artefactos de versión de compilación de DevOps en la imagen. Para realizar este trabajo, debe configurar una canalización de compilación. En la configuración de la canalización de versión, debe agregar el repositorio de los artefactos de compilación.

:::image type="content" source="./media/image-builder-devops-task/add-artifact.png" alt-text="Seleccionar Agregar un artefacto en la canalización de versión":::

Seleccione el botón **Ruta de acceso de compilación** para elegir la carpeta de compilación que desea colocar en la imagen. La tarea de Image Builder copia todos los archivos y directorios que contiene. Cuando se crea la imagen, Image Builder implementa los archivos y directorios en diferentes rutas de acceso, en función del sistema operativo.

> [!IMPORTANT]
> Al agregar un artefacto de repositorio, es posible que el directorio tenga como prefijo un guion bajo *_* . El guion bajo puede producir problemas con los comandos insertados. Use las comillas adecuadas en los comandos.
> 

En los ejemplos siguientes se explica cómo funciona:

:::image type="content" source="./media/image-builder-devops-task/build-artifacts.png" alt-text="Una estructura de directorio que muestra la jerarquía":::


* Windows: existen archivos en `C:\`. Se crea un directorio denominado `buildArtifacts` que incluye el directorio `webapp`.

* Linux: existen archivos en `/tmp`. Se crea el directorio `webapp` que incluye todos los archivos y directorios. Debe trasladar los archivos de este directorio. De lo contrario, se eliminarán, ya que se encuentra en el directorio temporal.

#### <a name="inline-customization-script"></a>Script de personalización insertado

* Windows: puede escribir comandos insertados de PowerShell separados por comas. Si desea ejecutar un script en el directorio de compilación, puede usar:

    ```PowerShell
    & 'c:\buildArtifacts\webapp\webconfig.ps1'
    ```

* Linux: en los sistemas Linux, los artefactos de compilación se colocan en el directorio `/tmp`. Sin embargo, en muchos sistemas operativos Linux, al reiniciar, se elimina el contenido del directorio /tmp. Si desea que los artefactos existan en la imagen, debe crear otro directorio y copiarlos.  Por ejemplo:

    ```bash
    sudo mkdir /lib/buildArtifacts
    sudo cp -r "/tmp/_ImageBuilding/webapp" /lib/buildArtifacts/.
    ```
    
    Si está de acuerdo con el directorio "/tmp", puede usar el código siguiente para ejecutar el script.
    
    ```bash
    # grant execute permissions to execute scripts
    sudo chmod +x "/tmp/_ImageBuilding/webapp/coreConfig.sh"
    echo "running script"
    sudo . "/tmp/AppsAndImageBuilderLinux/_WebApp/coreConfig.sh"
    ```
    
#### <a name="what-happens-to-the-build-artifacts-after-the-image-build"></a>¿Qué ocurre con los artefactos de compilación después de la compilación de la imagen?

> [!NOTE]
> Image Builder no quita automáticamente los artefactos de compilación; se recomienda encarecidamente que siempre tenga código para quitar los artefactos de compilación.
> 

* Windows: Image Builder implementa archivos en el directorio `c:\buildArtifacts`. El directorio se conserva y debe quitarlo. Puede quitarlo en el script que se ejecuta. Por ejemplo:

    ```PowerShell
    # Clean up buildArtifacts directory
    Remove-Item -Path "C:\buildArtifacts\*" -Force -Recurse
    
    # Delete the buildArtifacts directory
    Remove-Item -Path "C:\buildArtifacts" -Force 
    ```
    
* Linux: los artefactos de compilación se colocan en el directorio `/tmp`. Sin embargo, en muchos sistemas operativos Linux, al reiniciar, se elimina el contenido del directorio `/tmp`. Se sugiere que tenga código para quitar el contenido y no basarse para ello en el sistema operativo. Por ejemplo:

    ```bash
    sudo rm -R "/tmp/AppsAndImageBuilderLinux"
    ```
    
#### <a name="total-length-of-image-build"></a>Longitud total de la compilación de la imagen

La longitud total no se puede cambiar todavía en la tarea de canalización de DevOps. Utiliza el valor predeterminado de 240 minutos. Si desea aumentar [buildTimeoutInMinutes](https://docs.microsoft.com/azure/virtual-machines/linux/image-builder-json?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json&bc=%2Fazure%2Fvirtual-machines%2Fwindows%2Fbreadcrumb%2Ftoc.json#properties-buildtimeoutinminutes), puede usar una tarea AZ CLI en la canalización de versión. Configure la tarea para copiar una plantilla y enviarla. Para obtener un ejemplo, vea esta [solución](https://github.com/danielsollondon/azvmimagebuilder/tree/master/solutions/4_Using_ENV_Variables#using-environment-variables-and-parameters-with-image-builder) o use Az PowerShell.


#### <a name="storage-account"></a>Cuenta de almacenamiento

Seleccione la cuenta de almacenamiento que creó en los requisitos previos. Si no la ve en la lista, Image Builder no tiene permisos para ello.

Cuando se inicia la compilación, Image Builder creará un contenedor denominado `imagebuilder-vststask`. El contenedor es donde se almacenan los artefactos de compilación del repositorio.

> [!NOTE]
> Debe eliminar manualmente la cuenta de almacenamiento o el contenedor después de cada compilación.
>

### <a name="distribute"></a>Distribución

Se admiten tres tipos de distribución.

#### <a name="managed-image"></a>Imagen administrada

* ResourceID:
    ```bash
    /subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/images/<imageName>
    ```

* Ubicaciones

#### <a name="azure-shared-image-gallery"></a>Galería de imágenes compartidas de Azure

Shared Image Gallery ya **debe** existir.

* ResourceID: 
    ```bash
    /subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/<galleryName>/images/<imageDefName>
    ```

* Regiones: lista de regiones, separadas por comas. Por ejemplo, westus, eastus y centralus

#### <a name="vhd"></a>VHD

No se pueden pasar valores aquí; Image Builder emitirá el disco duro virtual al grupo de recursos de Image Builder temporal, `IT_<DestinationResourceGroup>_<TemplateName>`, en el contenedor *vhds*. Cuando se inicia la compilación de versión, Image Builder emite registros. Cuando haya terminado, emitirá la dirección URL del disco duro virtual.

### <a name="optional-settings"></a>Configuración opcional

* [Tamaño de máquina virtual](image-builder-json.md#vmprofile): puede reemplazar el tamaño de la máquina virtual, a partir del valor predeterminado *Standard_D1_v2*. Puede reemplazar para reducir el tiempo total de personalización o porque desea crear las imágenes que dependen de ciertos tamaños de máquina virtual, como GPU/HPC, etc.

## <a name="how-it-works"></a>Funcionamiento

Al crear la versión, la tarea crea un contenedor en la cuenta de almacenamiento, denominado *imagebuilder-vststask*. Comprime y carga los artefactos de compilación y crea un token de SAS para el archivo ZIP.

La tarea utiliza las propiedades que se pasan a la tarea para crear el artefacto de plantilla de Image Builder. La tarea realiza lo siguiente:
* Descarga el archivo ZIP del artefacto de compilación y cualquier otro script asociado. Los archivos se guardan en una cuenta de almacenamiento en el grupo de recursos `IT_<DestinationResourceGroup>_<TemplateName>` de Image Builder temporal.
* Crea una plantilla con el prefijo *t_* y un entero de 10 dígitos monotónico. La plantilla se guarda en el grupo de recursos seleccionado. La plantilla existe mientras dure la compilación en el grupo de recursos. 

Salida de ejemplo:

```text
start reading task parameters...
found build at:  /home/vsts/work/r1/a/_ImageBuilding/webapp
end reading parameters
getting storage account details for aibstordot1556933914
created archive /home/vsts/work/_temp/temp_web_package_21475337782320203.zip
Source for image:  { type: 'SharedImageVersion',
  imageVersionId: '/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/<galleryName>/images/<imageDefName>/versions/<imgVersionNumber>' }
template name:  t_1556938436xxx
starting put template...
```

Cuando se inicia la compilación de la imagen, el estado de ejecución se muestra en los registros de versión:

```text
starting run template...
```

Cuando se complete la compilación de la imagen, verá un texto similar al siguiente:

```text
2019-05-06T12:49:52.0558229Z starting run template...
2019-05-06T13:36:33.8863094Z run template:  Succeeded
2019-05-06T13:36:33.8867768Z getting runOutput for  SharedImage_distribute
2019-05-06T13:36:34.6652541Z ==============================================================================
2019-05-06T13:36:34.6652925Z ## task output variables ##
2019-05-06T13:36:34.6658728Z $(imageUri) =  /subscriptions/<subscriptionID>/resourceGroups/aibwinsig/providers/Microsoft.Compute/galleries/my22stSIG/images/winWAppimages/versions/0.23760.13763
2019-05-06T13:36:34.6659989Z ==============================================================================
2019-05-06T13:36:34.6663500Z deleting template t_1557146959485...
2019-05-06T13:36:34.6673713Z deleting storage blob imagebuilder-vststask\webapp/18-1/webapp_1557146958741.zip
2019-05-06T13:36:34.9786039Z blob imagebuilder-vststask\webapp/18-1/webapp_1557146958741.zip is deleted
2019-05-06T13:38:37.4884068Z delete template:  Succeeded
```

La plantilla de imagen y `IT_<DestinationResourceGroup>_<TemplateName>` se eliminan.

Puede adoptar la variable de VSTS "$(imageUri)" y usarla en la tarea siguiente o simplemente usar el valor y crear una máquina virtual.

## <a name="output-devops-variables"></a>Variables de DevOps de salida

Pub/oferta/SKU/Versión de la imagen de marketplace de origen:
* $(pirPublisher)
* $(pirOffer)
* $(pirSku)
* $(pirVersion)

URI de imagen: el identificador de recurso de la imagen distribuida:
* $(imageUri)

## <a name="faq"></a>Preguntas más frecuentes

### <a name="can-i-use-an-existing-image-template-i-have-already-created-outside-of-devops"></a>¿Se puede usar una plantilla de imagen existente que ya se he creado, fuera de DevOps?

De momento, no.

### <a name="can-i-specify-the-image-template-name"></a>¿Se puede especificar el nombre de la plantilla de imagen?

No. Se usa un nombre de plantilla único y luego se elimina.

### <a name="the-image-builder-failed-how-can-i-troubleshoot"></a>Image Builder genera un error. ¿Cómo se puede solucionar el problema?

Si se produce un error de compilación, la tarea de DevOps no elimina el grupo de recursos de almacenamiento provisional. Puede acceder al grupo de recursos de almacenamiento provisional que contiene el registro de personalización de compilación.

Verá un error en el registro de DevOps de la tarea de VM Image Builder y verá la ubicación del archivo customization.log. Por ejemplo:

:::image type="content" source="./media/image-builder-devops-task/devops-task-error.png" alt-text="Ejemplo de error de la tarea de DevOps":::

Para más información sobre la solución de problemas, vea [Solución de problemas del servicio Azure Image Builder](image-builder-troubleshoot.md). 

Después de investigar el error, puede eliminar el grupo de recursos de almacenamiento provisional. En primer lugar, elimine el artefacto de recurso de la plantilla de imagen. El artefacto tiene el prefijo *t_* y se puede encontrar en el registro de compilación de la tarea de DevOps:

```text
...
Source for image:  { type: 'SharedImageVersion',
  imageVersionId: '/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/<galleryName>/images/<imageDefName>/versions/<imgVersionNumber>' }
...
template name:  t_1556938436xxx
...

```

El artefacto de recurso de la plantilla de imagen está en el grupo de recursos especificado inicialmente en la tarea. Cuando haya terminado de solucionar problemas, elimine el artefacto. Si va a realizar la eliminación desde Azure Portal, dentro del grupo de recursos, seleccione **Mostrar tipos ocultos** para ver el artefacto.


## <a name="next-steps"></a>Pasos siguientes

Para más información, vea [Introducción a Azure Image Builder](image-builder-overview.md).

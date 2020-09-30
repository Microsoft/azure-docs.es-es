---
title: Implementación de máquinas virtuales en el dispositivo Azure Stack Edge Pro mediante plantillas
description: Describe cómo crear y administrar máquinas virtuales en un dispositivo Azure Stack Edge Pro mediante plantillas.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/04/2020
ms.author: alkohli
ms.openlocfilehash: eeefbcdc080620c60f7cd49b8f749375e23ddd02
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90899708"
---
# <a name="deploy-vms-on-your-azure-stack-edge-pro-gpu-device-via-templates"></a>Implementación de máquinas virtuales en el dispositivo Azure Stack Edge Pro con GPU mediante plantillas

En este tutorial se describe cómo crear y administrar una máquina virtual en el dispositivo Azure Stack Edge Pro mediante plantillas. Estas plantillas son archivos JSON (notación de objetos JavaScript) que definen tanto la infraestructura como la configuración de una máquina virtual. En las plantillas se especifican los recursos que se van a implementar y las propiedades de esos recursos.

Las plantillas son flexibles en entornos diferentes, ya que pueden tomar parámetros como entrada en tiempo de ejecución desde un archivo. La estructura de nomenclatura estándar es `TemplateName.json` para la plantilla y `TemplateName.parameters.json` para el archivo de parámetros. Para obtener más información sobre las plantillas de Resource Manager, vaya a [¿Qué son las plantillas de Resource Manager?](../azure-resource-manager/templates/overview.md)

En este tutorial, usaremos plantillas de ejemplo ya escritas para crear recursos. No necesitará editar el archivo de plantilla y solo podrá modificar los archivos `.parameters.json` para personalizar la implementación en su máquina. 

## <a name="vm-deployment-workflow"></a>Flujo de trabajo de implementación de una máquina virtual

Para implementar máquinas virtuales de Azure Stack Edge Pro en muchos dispositivos, puede usar un solo disco duro virtual preparado con sysprep para todos los dispositivos, la misma plantilla para la implementación y realizar únicamente pequeños cambios en los parámetros de la plantilla para cada ubicación de implementación (estos cambios podrían hacerse de forma manual, como explicamos aquí, o mediante programación). 

El resumen general del flujo de trabajo de implementación mediante plantillas es el siguiente:

1. **Configurar los requisitos previos**. Hay 3 tipos de requisitos previos: del dispositivo, del cliente y de la máquina virtual.

    1. **Requisitos previos del dispositivo**

        1. Conéctese a Azure Resource Manager en el dispositivo.
        2. Habilite el proceso a través de la interfaz de usuario local.

    2. **Requisitos previos del cliente**

        1. Descargue las plantillas de máquina virtual y los archivos asociados en el cliente.
        1. Opcionalmente, configure TLS 1.2 en el cliente.
        1. [Descargue e instale el Explorador de Microsoft Azure Storage](https://go.microsoft.com/fwlink/?LinkId=708343&clcid=0x409) en el cliente.

    3. **Requisitos previos de la máquina virtual**

        1. Cree un grupo de recursos en la ubicación del dispositivo que contendrá todos los recursos de la máquina virtual.
        1. Cree una cuenta de almacenamiento para cargar el disco duro virtual (VHD) que se usa para crear la imagen de la máquina virtual.
        1. Agregue el identificador URI de la cuenta de almacenamiento local al archivo DNS o hosts del cliente que tiene acceso al dispositivo.
        1. Instale el certificado de almacenamiento de blobs en el dispositivo y también en el cliente local que accede al dispositivo. Opcionalmente, instale el certificado de almacenamiento de blobs en el Explorador de Storage.
        1. Cree y cargue un disco duro virtual en la cuenta de almacenamiento creada anteriormente.

2. **Crear una máquina virtual a partir de plantillas**

    1. Cree una imagen de máquina virtual y una red virtual con el archivo de parámetros `CreateImageAndVnet.parameters.json` y la plantilla de implementación `CreateImageAndVnet.json`.
    1. Cree una máquina virtual con los recursos creados anteriormente mediante el archivo de parámetros `CreateVM.parameters.json` y la plantilla de implementación `CreateVM.json`.

## <a name="device-prerequisites"></a>Requisitos previos del dispositivo

Configure estos requisitos previos en el dispositivo Azure Stack Edge Pro.

[!INCLUDE [azure-stack-edge-gateway-deploy-virtual-machine-prerequisites](../../includes/azure-stack-edge-gateway-deploy-virtual-machine-prerequisites.md)]

## <a name="client-prerequisites"></a>Requisitos previos del cliente

Configure estos requisitos previos en el cliente que se usará para acceder al dispositivo Azure Stack Edge Pro.

1. [Descargue el Explorador de Storage](https://azure.microsoft.com/features/storage-explorer/) si lo usa para cargar un disco duro virtual. Si lo prefiere, puede descargar AzCopy para cargar el disco duro virtual. Es posible que tenga que configurar TLS 1.2 en la máquina cliente si ejecuta versiones anteriores de AzCopy. 
1. [Descargue las plantillas de máquina virtual y los archivos de parámetros](https://aka.ms/ase-vm-templates) en la máquina cliente. Descomprímalos en un directorio que usará como directorio de trabajo.


## <a name="vm-prerequisites"></a>Requisitos previos de la máquina virtual

Configure estos requisitos previos para crear los recursos que se necesitarán al crear la máquina virtual. 

    
### <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un grupo de recursos de Azure con [New-AzureRmResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup). Un grupo de recursos es un contenedor lógico en el que se implementan y administran los recursos de Azure, como la cuenta de almacenamiento, el disco y el disco administrado.

> [!IMPORTANT]
> Todos los recursos se crean en la misma ubicación que el dispositivo y la ubicación se establece en **DBELocal**.

```powershell
New-AzureRmResourceGroup -Name <Resource group name> -Location DBELocal
```

A continuación se muestra una salida de ejemplo.

```powershell
PS C:\windows\system32> New-AzureRmResourceGroup -Name myasegpurgvm -Location DBELocal

ResourceGroupName : myasegpurgvm
Location          : dbelocal
ProvisioningState : Succeeded
Tags              :
ResourceId        : /subscriptions/DDF9FC44-E990-42F8-9A91-5A6A5CC472DB/resourceGroups/myasegpurgvm

PS C:\windows\system32>
```

### <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

Cree una cuenta de almacenamiento en el grupo de recursos creado en el paso anterior. Se trata de una **cuenta de almacenamiento local** que se usará para cargar la imagen de disco virtual de la máquina virtual.

```powershell
New-AzureRmStorageAccount -Name <Storage account name> -ResourceGroupName <Resource group name> -Location DBELocal -SkuName Standard_LRS
```

> [!NOTE]
> Con Azure Resource Manager solo se pueden crear cuentas de almacenamiento local, como las cuentas de almacenamiento con redundancia local (Standard_LRS o Premium_LRS). Para crear cuentas de almacenamiento en capas, vea los pasos que se describen en el tema sobre [incorporación y conexión a cuentas de almacenamiento en el dispositivo Azure Stack Edge Pro](azure-stack-edge-j-series-deploy-add-storage-accounts.md).

A continuación se muestra una salida de ejemplo.

```powershell
PS C:\windows\system32> New-AzureRmStorageAccount -Name myasegpusavm -ResourceGroupName myasegpurgvm -Location DBELocal -SkuName Standard_LRS

StorageAccountName ResourceGroupName Location SkuName     Kind    AccessTier CreationTime
------------------ ----------------- -------- -------     ----    ---------- ------------
myasegpusavm       myasegpurgvm      DBELocal StandardLRS Storage            7/29/2020 10:11:16 PM

PS C:\windows\system32>
```

Para obtener la clave de la cuenta de almacenamiento, ejecute el comando `Get-AzureRmStorageAccountKey`. A continuación se muestra una salida de ejemplo del comando.

```powershell
PS C:\windows\system32> Get-AzureRmStorageAccountKey

cmdlet Get-AzureRmStorageAccountKey at command pipeline position 1
Supply values for the following parameters:
(Type !? for Help.)
ResourceGroupName: myasegpurgvm
Name: myasegpusavm

KeyName    Value                                                  Permissions   
-------     -----                                                   --
key1 GsCm7QriXurqfqx211oKdfQ1C9Hyu5ZutP6Xl0dqlNNhxLxDesDej591M8y7ykSPN4fY9vmVpgc4ftkwAO7KQ== 11 
key2 7vnVMJUwJXlxkXXOyVO4NfqbW5e/5hZ+VOs+C/h/ReeoszeV+qoyuBitgnWjiDPNdH4+lSm1/ZjvoBWsQ1klqQ== ll
```

### <a name="add-blob-uri-to-hosts-file"></a>Incorporación del identificador URI del blob al archivo hosts

Asegúrese de que ya ha agregado el identificador URI del blob en el archivo hosts del cliente que está usando para conectarse al almacenamiento de blobs. **Ejecute el Bloc de notas como administrador** y agregue la entrada siguiente para el identificador URI del blob en el archivo `C:\windows\system32\drivers\etc\hosts`:

`<Device IP> <storage account name>.blob.<Device name>.<DNS domain>`

En un entorno típico, tendría configurado el DNS para que todas las cuentas de almacenamiento señalasen al dispositivo Azure Stack Edge Pro con una entrada `*.blob.devicename.domainname.com`.

### <a name="optional-install-certificates"></a>(Opcional) Instalación de certificados

Omita este paso si va a conectarse a través del Explorador de Storage con *http*. Si usa *https*, deberá instalar los certificados adecuados en el Explorador de Storage. En este caso, instale el certificado del punto de conexión de blob. Para más información, consulte cómo crear y cargar certificados en [Administración de certificados](azure-stack-edge-j-series-manage-certificates.md). 

### <a name="create-and-upload-a-vhd"></a>Creación y carga de un disco duro virtual

Asegúrese de que tiene una imagen de disco virtual que puede usar para la carga en el paso posterior. Siga los pasos que se indican en el tema sobre la [creación de una imagen de máquina virtual](azure-stack-edge-j-series-create-virtual-machine-image.md). 

Copie las imágenes de disco que se van a usar en los blobs en páginas en la cuenta de almacenamiento local que creó en los pasos anteriores. Puede usar una herramienta como [Explorador de Storage](https://azure.microsoft.com/features/storage-explorer/) o [AzCopy para cargar el disco duro virtual en la cuenta de almacenamiento](azure-stack-edge-j-series-deploy-virtual-machine-powershell.md#upload-a-vhd) que creó en los pasos anteriores. 

### <a name="use-storage-explorer-for-upload"></a>Uso del Explorador de Storage para la carga

1. Abra el Explorador de Storage. Vaya a **Editar** y asegúrese de que la aplicación esté establecida en **API de Azure Stack de destino**.

    ![Establecimiento del destino en API de Azure Stack](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/set-target-apis-1.png)

2. Instale el certificado de cliente con formato PEM. Vaya a **Editar > Certificados SSL > Importar certificados**. Seleccione el certificado de cliente.

    ![Importación del certificado del punto de conexión de Blob Storage](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/import-blob-storage-endpoint-certificate-1.png)

    - Si usa certificados generados por el dispositivo, descargue y convierta el certificado `.cer` del punto de conexión de Blog Storage al formato `.pem`. Ejecute el siguiente comando: 
    
        ```powershell
        PS C:\windows\system32> Certutil -encode 'C:\myasegpu1_Blob storage (1).cer' .\blobstoragecert.pem
        Input Length = 1380
        Output Length = 1954
        CertUtil: -encode command completed successfully.
        ```
    - Si trae su propio certificado, use el certificado raíz de la cadena de firma con formato `.pem`.

3. Después de haber importado el certificado, reinicie el Explorador de Storage para que los cambios surtan efecto.

    ![Reinicio de Explorador de Storage](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/restart-storage-explorer-1.png)

4. En el panel izquierdo, haga clic con el botón derecho en **Cuentas de almacenamiento** y seleccione **Conectar a Azure Storage**. 

    ![Conectar a Azure Storage 1](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/connect-azure-storage-1.png)

5. Seleccione **Usar un nombre y clave de la cuenta de almacenamiento**. Seleccione **Next** (Siguiente).

    ![Conectar a Azure Storage 2](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/connect-azure-storage-2.png)

6. En **Conectarse con el nombre y la clave**, indique los valores de **Nombre para mostrar**, **Nombre de cuenta** de almacenamiento, **Clave de cuenta** de Azure Storage. Seleccione el valor **Otros** en Dominio de almacenamiento y especifique la cadena de conexión `<device name>.<DNS domain>`. Si no ha instalado un certificado en el Explorador de Storage, marque la opción **Usar HTTP**. Seleccione **Next** (Siguiente).

    ![Conectarse con el nombre y la clave](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/connect-name-key-1.png)

7. Revise el **Resumen de conexiones** y seleccione **Conectar**.

8. La cuenta de almacenamiento aparece en el panel izquierdo. Seleccione y expanda la cuenta de almacenamiento. Seleccione **Contenedores de blob**, haga clic con el botón derecho y seleccione **Crear contenedor de blobs**. Escriba el nombre del contenedor de blobs.

9. Seleccione el contenedor que acaba de crear y, en el panel de la derecha, seleccione **Cargar > Cargar archivos**. 

    ![Carga del archivo de disco duro virtual 1](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/upload-vhd-file-1.png)

10. Busque y seleccione el disco duro virtual que desea cargar en **Archivos seleccionados**. Seleccione **Blob en páginas** como **Tipo de blob** y elija **Cargar**.

    ![Carga de archivo de disco duro virtual 2](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/upload-vhd-file-2.png)

11. Una vez cargado el disco duro virtual en el contenedor de blobs, seleccione el disco duro virtual, haga clic con el botón derecho y, a continuación, seleccione **Propiedades**. 

    ![Carga de archivo de disco duro virtual 3](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/upload-vhd-file-3.png)

12. Copie y guarde el identificador **URI**, ya que lo usará en los pasos posteriores.

    ![Copia de URI](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/copy-uri-1.png)

<!--### Use AzCopy for upload

Before you use AzCopy, make sure that the [AzCopy is configured correctly](#configure-azcopy) for use with the blob storage REST API version that you are using with your Azure Stack Edge Pro device.


```powershell
AzCopy /Source:<sourceDirectoryForVHD> /Dest:<blobContainerUri> /DestKey:<storageAccountKey> /Y /S /V /NC:32  /BlobType:page /destType:blob 
```

> ![NOTE]
> Set `BlobType` to page for creating a managed disk out of VHD. Set `BlobType` to block when writing to tiered storage accounts using AzCopy.

You can download the disk images from the marketplace. For detailed steps, go to [Get the virtual disk image from Azure marketplace](azure-stack-edge-j-series-create-virtual-machine-image.md).

A sample output using AzCopy 7.3 is shown below. For more information on this command, go to [Upload VHD file to storage account using AzCopy](../devtest-labs/devtest-lab-upload-vhd-using-azcopy.md).


```powershell
AzCopy /Source:\\hcsfs\scratch\vm_vhds\linux\ /Dest:http://sa191113014333.blob.dbe-1dcmhq2.microsoftdatabox.com/vmimages /DestKey:gJKoyX2Amg0Zytd1ogA1kQ2xqudMHn7ljcDtkJRHwMZbMK== /Y /S /V /NC:32 /BlobType:page /destType:blob /z:2e7d7d27-c983-410c-b4aa-b0aa668af0c6
```-->

## <a name="create-image-and-vnet-for-your-vm"></a>Creación de una imagen y una red virtual para la máquina virtual

Para crear una imagen y una red virtual para la máquina virtual, tendrá que editar el archivo de parámetros `CreateImageAndVnet.parameters.json` y, a continuación, implementar la plantilla `CreateImageAndVnet.json` que usa este archivo de parámetros.


### <a name="edit-parameters-file"></a>Edición del archivo de parámetros

El archivo `CreateImageAndVnet.parameters.json` toma los parámetros siguientes: 

```json
"parameters": {
        "osType": {
              "value": "<Operating system corresponding to the VHD you upload can be Windows or Linux>"
        },
        "imageName": {
            "value": "<Name for the VM iamge>"
        },
        "imageUri": {
              "value": "<Path to the VHD that you uploaded in the Storage account>"
        },
        "vnetName": {
            "value": "<Name for the virtual network where you will deploy the VM>"
        },
        "subnetName": {
            "value": "<Name for the subnet for the VNet>"
        },
        "addressPrefix": {
            "value": "<Address prefix for the virtual network>"
        },
        "subnetPrefix": {
            "value": "<Subnet prefix for the subnet for the Vnet>"
        }
    }
```

Edite el archivo `CreateImageAndVnet.parameters.json` para que se incluya la siguiente información para el dispositivo Azure Stack Edge Pro:

1. Proporcione el tipo de sistema operativo correspondiente al disco duro virtual que va a cargar. El tipo de sistema operativo puede ser Windows o Linux.

    ```json
    "parameters": {
            "osType": {
              "value": "Windows"
            },
    ```

2. Cambie el identificador URI de la imagen al URI de la imagen que cargó en el paso anterior:

    ```json
    "imageUri": {
        "value": "https://myasegpusavm.blob.myasegpu1.wdshcsso.com/windows/WindowsServer2016Datacenter.vhd"
        },
    ```
    Si usa *http* con el Explorador de Storage, cámbielo a un identificador URI *https*.

3. Cambie los valores de `addressPrefix` y `subnetPrefix`. En la interfaz de usuario local del dispositivo, vaya a la página **Red**. Busque el puerto que habilitó para el proceso. Obtenga la dirección IP de la red base y agregue la máscara de subred para crear la notación CIDR. Si tiene una subred estándar 255.255.255.0, reemplace el último número de la dirección IP por 0 y agregue /24 al final. Así, la dirección IP 10.126.68.0 con una máscara de subred 255.255.255.0 se convierte en 10.126.68.0/24. 
    
    ```json
    "addressPrefix": {
                "value": "10.126.68.0/24"
            },
            "subnetPrefix": {
                "value": "10.126.68.0/24"
            }
    ```  

4. Indique el nombre de imagen, nombre de red virtual y nombre de subred únicos en los parámetros.

    Este es el ejemplo de JSON que se usa en este artículo.

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
      "parameters": {
        "osType": {
          "value": "Windows"
        },
        "imageName": {
          "value": "image1"
        },
        "imageUri": {
          "value": "https://myasegpusavm.blob.myasegpu1.wdshcsso.com/windows/WindowsServer2016Datacenter.vhd"
        },
        "vnetName": {
          "value": "vnet1"
        },
        "subnetName": {
          "value": "subnet1"
        },
        "addressPrefix": {
          "value": "10.126.68.0/24"
        },
        "subnetPrefix": {
          "value": "10.126.68.0/24"
        }
      }
    }
    ```
5. Guarde el archivo de parámetros.


### <a name="deploy-template"></a>Implementar plantilla 

Implemente la plantilla `CreateImageAndVnet.json`. Esta plantilla implementa los recursos de red virtual e imagen que se usarán para crear máquinas virtuales en el paso posterior.

> [!NOTE]
> Si se produce un error de autenticación al implementar la plantilla, es posible que las credenciales de Azure para esta sesión hayan expirado. Vuelva a ejecutar el comando `login-AzureRM` para conectarse de nuevo a Azure Resource Manager en el dispositivo Azure Stack Edge Pro.

1. Ejecute el siguiente comando: 
    
    ```powershell
    $templateFile = "Path to CreateImageAndVnet.json"
    $templateParameterFile = "Path to CreateImageAndVnet.parameters.json"
    $RGName = "<Name of your resource group>"
    New-AzureRmResourceGroupDeployment `
        -ResourceGroupName $RGName `
        -TemplateFile $templateFile `
        -TemplateParameterFile $templateParameterFile `
        -Name "<Name for your deployment>"
    ```

2. Compruebe si los recursos de imagen y red virtual se han aprovisionado correctamente. Este es un ejemplo de salida de una imagen y una red virtual que se han creado correctamente.
    
    ```powershell
    PS C:\07-30-2020> login-AzureRMAccount -EnvironmentName aztest1 -TenantId c0257de7-538f-415c-993a-1b87a031879d
    
    Account               SubscriptionName              TenantId                             Environment
    -------               ----------------              --------                             -----------
    EdgeArmUser@localhost Default Provider Subscription c0257de7-538f-415c-993a-1b87a031879d aztest1
    
    PS C:\07-30-2020> $templateFile = "C:\07-30-2020\CreateImageAndVnet.json"
    PS C:\07-30-2020> $templateParameterFile = "C:\07-30-2020\CreateImageAndVnet.parameters.json"
    PS C:\07-30-2020> $RGName = "myasegpurgvm"
    PS C:\07-30-2020> New-AzureRmResourceGroupDeployment `
    >>     -ResourceGroupName $RGName `
    >>     -TemplateFile $templateFile `
    >>     -TemplateParameterFile $templateParameterFile `
    >>     -Name "Deployment1"
    
    DeploymentName          : Deployment1
    ResourceGroupName       : myasegpurgvm
    ProvisioningState       : Succeeded
    Timestamp               : 7/30/2020 5:53:32 PM
    Mode                    : Incremental
    TemplateLink            :
    Parameters              :
                              Name             Type                       Value
                              ===============  =========================  ==========
                              osType           String                     Windows
                              imageName        String                     image1
                              imageUri         String
                              https://myasegpusavm.blob.myasegpu1.wdshcsso.com/windows/WindowsServer2016Datacenter.vhd
                              vnetName         String                     vnet1
                              subnetName       String                     subnet1
                              addressPrefix    String                     10.126.68.0/24
                              subnetPrefix     String                     10.126.68.0/24
    
    Outputs                 :
    DeploymentDebugLogLevel :
    
    PS C:\07-30-2020>
    ```
    
## <a name="create-vm"></a>Creación de una máquina virtual

### <a name="edit-parameters-file-to-create-vm"></a>Edición del archivo de parámetros para crear una máquina virtual
 
Para crear una máquina virtual, use el archivo de parámetros `CreateVM.parameters.json`. Toma los parámetros siguientes.
    
```json
"vmName": {
            "value": "<Name for your VM>"
        },
        "adminUsername": {
            "value": "<Username to log into the VM>"
        },
        "Password": {
            "value": "<Password to log into the VM>"
        },
        "imageName": {
            "value": "<Name for your image>"
        },
        "vmSize": {
            "value": "<A supported size for your VM>"
        },
        "vnetName": {
            "value": "<Name for the virtual network you created earlier>"
        },
        "subnetName": {
            "value": "<Name for the subnet you created earlier>"
        },
        "nicName": {
            "value": "<Name for the network interface>"
        },
        "privateIPAddress": {
            "value": "<Private IP address, enter a static IP in the subnet created earlier or leave empty to assign DHCP>"
        },
        "IPConfigName": {
            "value": "<Name for the ipconfig associated with the network interface>"
        }
```    

Asigne los parámetros adecuados en `CreateVM.parameters.json` para el dispositivo Azure Stack Edge Pro.

1. Proporcione un nombre único, un nombre de interfaz de red y un nombre de ipconfig. 
1. Escriba un nombre de usuario, una contraseña y un tamaño de máquina virtual admitido.
1. Asigne el mismo nombre en **VnetName**, **subnetName** e **ImageName** que se indica en los parámetros de `CreateImageAndVnet.parameters.json`. Por ejemplo, si ha especificado los valores de VnetName, subnetName e ImageName como **vnet1**, **subnet1** e **image1**, utilice también esos valores en los parámetros de esta plantilla.
1. Ahora necesitará una dirección IP estática para asignarla a la máquina virtual que se encuentra en la red de subred definida anteriormente. Reemplace **PrivateIPAddress** por esta dirección en el archivo de parámetros. Para que la máquina virtual obtenga una dirección IP del servidor DCHP local, deje en blanco el valor `privateIPAddress`.  
    
    ```json
    "privateIPAddress": {
                "value": "5.5.153.200"
            },
    ```
    
4. Guarde el archivo de parámetros.

    Este es el ejemplo de JSON que se usa en este artículo.
    
    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "vmName": {
                "value": "mywindowsvm"
            },
            "adminUsername": {
                "value": "Administrator"
            },
            "Password": {
                "value": "Password1"
            },
            "imageName": {
                "value": "image1"
            },
            "vmSize": {
                "value": "Standard_D1_v2"
            },
            "vnetName": {
                "value": "vnet1"
            },
            "subnetName": {
                "value": "subnet1"
            },
            "nicName": {
                "value": "nic1"
            },
            "privateIPAddress": {
                "value": "10.126.68.186"
            },
            "IPConfigName": {
                "value": "ipconfig1"
            }
        }
    }
    ```      

### <a name="deploy-template-to-create-vm"></a>Implementación de la plantilla para crear una máquina virtual

Implemente la plantilla de creación de máquinas virtuales `CreateVM.json`. Esta plantilla crea una interfaz de red a partir de la red virtual existente y la máquina virtual a partir de la imagen implementada.

1. Ejecute el siguiente comando: 
    
    ```powershell
    Command:
        
        $templateFile = "<Path to CreateVM.json>"
        $templateParameterFile = "<Path to CreateVM.parameters.json>"
        $RGName = "<Resource group name>"
             
        New-AzureRmResourceGroupDeployment `
            -ResourceGroupName $RGName `
            -TemplateFile $templateFile `
            -TemplateParameterFile $templateParameterFile `
            -Name "<DeploymentName>"
    ```   

    La creación de la máquina virtual tardará entre 15 y 20 minutos. Este es un ejemplo de salida de una máquina virtual creada correctamente.
    
    ```powershell
    PS C:\07-30-2020> $templateFile = "C:\07-30-2020\CreateWindowsVM.json"
        PS C:\07-30-2020> $templateParameterFile = "C:\07-30-2020\CreateWindowsVM.parameters.json"
        PS C:\07-30-2020> $RGName = "myasegpurgvm"
        PS C:\07-30-2020> New-AzureRmResourceGroupDeployment `
        >>     -ResourceGroupName $RGName `
        >>     -TemplateFile $templateFile `
        >>     -TemplateParameterFile $templateParameterFile `
        >>     -Name "Deployment2"    
        
        DeploymentName          : Deployment2
        ResourceGroupName       : myasegpurgvm
        ProvisioningState       : Succeeded
        Timestamp               : 7/30/2020 6:21:09 PM
        Mode                    : Incremental
        TemplateLink            :
        Parameters              :
                                  Name             Type                       Value
                                  ===============  =========================  ==========
                                  vmName           String                     MyWindowsVM
                                  adminUsername    String                     Administrator
                                  password         String                     Password1
                                  imageName        String                     image1
                                  vmSize           String                     Standard_D1_v2
                                  vnetName         String                     vnet1
                                  subnetName       String                     subnet1
                                  nicName          String                     Nic1
                                  ipConfigName     String                     ipconfig1
                                  privateIPAddress  String                    10.126.68.186
        
        Outputs                 :
        DeploymentDebugLogLevel :    
        
        PS C:\07-30-2020>
    ```   
También puede ejecutar el comando `New-AzureRmResourceGroupDeployment` de forma asincrónica con el parámetro `–AsJob`. Este es un ejemplo de salida cuando el cmdlet se ejecuta en segundo plano. Después, puede consultar el estado del trabajo que se crea mediante el cmdlet `Get-Job` .

    ```powershell   
    PS C:\WINDOWS\system32> New-AzureRmResourceGroupDeployment `
    >>     -ResourceGroupName $RGName `
    >>     -TemplateFile $templateFile `
    >>     -TemplateParameterFile $templateParameterFile `
    >>     -Name "Deployment2" `
    >>     -AsJob
     
    Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
    --     ----            -------------   -----         -----------     --------             -------
    2      Long Running... AzureLongRun... Running       True            localhost            New-AzureRmResourceGro...
     
    PS C:\WINDOWS\system32> Get-Job -Id 2
     
    Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
    --     ----            -------------   -----         -----------     --------             -------
    2      Long Running... AzureLongRun... Completed     True            localhost            New-AzureRmResourceGro...
    ```

7. Compruebe si la máquina virtual se ha aprovisionado correctamente. Ejecute el siguiente comando:

    `Get-AzureRmVm`


## <a name="connect-to-a-vm"></a>Conexión a una máquina virtual

Los pasos para conectarse pueden ser diferentes dependiendo de si se ha creado una máquina virtual Windows o Linux.

### <a name="connect-to-windows-vm"></a>Conexión a una máquina virtual Windows

Siga estos pasos para conectarse a una máquina virtual Windows.

[!INCLUDE [azure-stack-edge-gateway-connect-vm](../../includes/azure-stack-edge-gateway-connect-virtual-machine-windows.md)]

### <a name="connect-to-linux-vm"></a>Conexión a una máquina virtual Linux

Siga estos pasos para conectarse a una máquina virtual Linux.

[!INCLUDE [azure-stack-edge-gateway-connect-vm](../../includes/azure-stack-edge-gateway-connect-virtual-machine-linux.md)]

<!--## Manage VM

The following section describes some of the common operations around the VM that you will create on your Azure Stack Edge Pro device.

[!INCLUDE [azure-stack-edge-gateway-manage-vm](../../includes/azure-stack-edge-gateway-manage-vm.md)]-->


## <a name="supported-vm-sizes"></a>Tamaños de máquinas virtuales que se admiten

[!INCLUDE [azure-stack-edge-gateway-supported-vm-sizes](../../includes/azure-stack-edge-gateway-supported-vm-sizes.md)]

## <a name="unsupported-vm-operations-and-cmdlets"></a>Operaciones y cmdlets de máquina virtual que no se admiten

No se admiten las extensiones, los conjuntos de escalado, los conjuntos de disponibilidad ni las instantáneas.

<!--## Configure AzCopy

When you install the latest version of AzCopy, you will need to configure AzCopy to ensure that it matches the blob storage REST API version of your Azure Stack Edge Pro device.

On the client used to access your Azure Stack Edge Pro device, set up a global variable to match the blob storage REST API version.

### On Windows client 

`$Env:AZCOPY_DEFAULT_SERVICE_API_VERSION = "2017-11-09"`

### On Linux client

`export AZCOPY_DEFAULT_SERVICE_API_VERSION=2017-11-09`

To verify if the environment variable for AzCopy was set correctly, take the following steps:

1. Run "azcopy env".
2. Find `AZCOPY_DEFAULT_SERVICE_API_VERSION` parameter. This should have the value you set in the preceding steps.-->


## <a name="next-steps"></a>Pasos siguientes

[Cmdlets de Azure Resource Manager](https://docs.microsoft.com/powershell/module/azurerm.resources/?view=azurermps-6.13.0)

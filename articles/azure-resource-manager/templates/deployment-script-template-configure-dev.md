---
title: Configuración del entorno de desarrollo para scripts de implementación en plantillas | Microsoft Docs
description: Configure el entorno de desarrollo para los scripts de implementación en las plantillas de Azure Resource Manager.
services: azure-resource-manager
author: mumian
ms.service: azure-resource-manager
ms.topic: conceptual
ms.date: 07/27/2020
ms.author: jgao
ms.openlocfilehash: 232a1ae5d125a2ea1d5723e85073fb3dd02420cc
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87293829"
---
# <a name="configure-development-environment-for-deployment-scripts-in-templates-preview"></a>Configuración del entorno de desarrollo para scripts de implementación en plantillas (versión preliminar)

Aprenda a crear un entorno de desarrollo para desarrollar y probar scripts de implementación con una imagen de script de implementación. Puede crear una [instancia de contenedor de Azure](../../container-instances/container-instances-overview.md) o bien usar [Docker](https://docs.docker.com/get-docker/). Ambas opciones se describen en este artículo.

## <a name="prerequisites"></a>Requisitos previos

Si no tiene un script de implementación, puede crear un archivo **hello.ps1** con el siguiente contenido:

```powershell
param([string] $name)
$output = 'Hello {0}' -f $name
Write-Output $output
$DeploymentScriptOutputs = @{}
$DeploymentScriptOutputs['text'] = $output
```

## <a name="use-azure-container-instance"></a>Uso de una instancia de contenedor de Azure

Para crear los scripts en el equipo, debe crear una cuenta de almacenamiento y montarla en la instancia de contenedor. Así, podrá cargar el script en la cuenta de almacenamiento y ejecutar el script en la instancia de contenedor.

> [!NOTE]
> La cuenta de almacenamiento que cree para probar el script no es la misma que usa el servicio de script de implementación para ejecutarlo. El servicio de implementación del script crea un nombre único como un recurso compartido de archivos en cada ejecución.

### <a name="create-an-azure-container-instance"></a>Creación de una instancia de contenedor de Azure

La siguiente plantilla de ARM crea una instancia de contenedor y un recurso compartido de archivos, y, a continuación, monta el recurso compartido de archivos en la imagen de contenedor.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "projectName": {
      "type": "string",
      "metadata": {
        "description": "Specify a project name that is used for generating resource names."
      }
    },
    "containerImage": {
      "type": "string",
      "defaultValue": "mcr.microsoft.com/azuredeploymentscripts-powershell:az4.3",
      "metadata": {
        "description": "Specify the container image."
      }
    },
    "mountPath": {
      "type": "string",
      "defaultValue": "deploymentScript",
      "metadata": {
        "description": "Specify the mount path."
      }
    }
  },
  "variables": {
    "storageAccountName": "[concat(parameters('projectName'), 'store')]",
    "fileShareName": "[concat(parameters('projectName'), 'share')]",
    "containerGroupName": "[concat(parameters('projectName'), 'cg')]",
    "containerName": "[concat(parameters('projectName'), 'container')]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-06-01",
      "name": "[variables('storageAccountName')]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS",
        "tier": "Standard"
      },
      "kind": "StorageV2",
      "properties": {
        "accessTier": "Hot"
      }
    },
    {
      "type": "Microsoft.Storage/storageAccounts/fileServices/shares",
      "apiVersion": "2019-06-01",
      "name": "[concat(variables('storageAccountName'), '/default/', variables('fileShareName'))]",
      "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
      ]
    },
    {
      "type": "Microsoft.ContainerInstance/containerGroups",
      "apiVersion": "2019-12-01",
      "name": "[variables('containerGroupName')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
      ],
      "properties": {
        "containers": [
          {
            "name": "[variables('containerName')]",
            "properties": {
              "image": "[parameters('containerImage')]",
              "resources": {
                "requests": {
                  "cpu": 1,
                  "memoryInGb": 1.5
                }
              },
              "ports": [
                {
                  "protocol": "TCP",
                  "port": 80
                }
              ],
              "volumeMounts": [
                {
                  "name": "filesharevolume",
                  "mountPath": "[parameters('mountPath')]"
                }
              ],
              "command": [
                "/bin/sh",
                "-c",
                "pwsh -c 'Start-Sleep -Seconds 1800'"
              ]
            }
          }
        ],
        "osType": "Linux",
        "volumes": [
          {
            "name": "filesharevolume",
            "azureFile": {
              "readOnly": false,
              "shareName": "[variables('fileShareName')]",
              "storageAccountName": "[variables('storageAccountName')]",
              "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')), '2019-06-01').keys[0].value]"
            }
          }
        ]
      }
    }
  ]
}
```
El valor predeterminado de la ruta de montaje es **deploymentScript**.  Se trata de la ruta de acceso en la instancia de contenedor donde se monta en el recurso compartido de archivos.

La imagen de contenedor predeterminada especificada en la plantilla es **mcr.microsoft.com/azuredeploymentscripts-powershell:az4.3"** .  Para obtener una lista de las versiones de Azure PowerShell y las versiones de la CLI de Azure admitidas, consulte [Azure PowerShell o CLI de Azure](./deployment-script-template.md#prerequisites).

La plantilla suspende la instancia de contenedor 1800 segundos. Dispone de 30 minutos antes de que la instancia de contenedor pase al estado terminal y finalice la sesión.

Para implementar la plantilla:

```azurepowershell
$projectName = Read-Host -Prompt "Enter a project name that is used to generate resource names"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$templateFile = Read-Host -Prompt "Enter the template file path and file name"
$resourceGroupName = "${projectName}rg"

New-azResourceGroup -Location $location -name $resourceGroupName
New-AzResourceGroupDeployment -resourceGroupName $resourceGroupName -TemplateFile $templatefile -projectName $projectName
```

### <a name="upload-deployment-script"></a>Carga de un script de implementación

Cargue el script de implementación en la cuenta de almacenamiento. Este es un ejemplo de PowerShell:

```azurepowershell
$projectName = Read-Host -Prompt "Enter the same project name that you used earlier"
$fileName = Read-Host -Prompt "Enter the deployment script file name with the path"

$resourceGroupName = "${projectName}rg"
$storageAccountName = "${projectName}store"
$fileShareName = "${projectName}share"

$context = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName).Context
Set-AzStorageFileContent -Context $context -ShareName $fileShareName -Source $fileName -Force
```

También puede cargar el archivo mediante Azure Portal y la CLI de Azure.

### <a name="test-the-deployment-script"></a>Prueba del script de implementación

1. En Azure Portal, abra el grupo de recursos en el que implementó la instancia de contenedor y la cuenta de almacenamiento.
1. Abra el grupo de contenedores. El nombre del grupo de contenedores predeterminado es el nombre del proyecto con **cg** anexado. Verá que la instancia de contenedor está en el estado **En ejecución**.
1. Seleccione **Contenedores** en el menú de la izquierda. Verá una instancia de contenedor.  El nombre de la instancia de contenedor es el del proyecto con **container** (contenedor) anexado.

    ![instancia de contenedor de conexión de script de implementación](./media/deployment-script-template-configure-dev/deployment-script-container-instance-connect.png)

1. Seleccione **Conectar** y, después, **Conectar**. Si no puede conectarse a la instancia de contenedor, reinicie el grupo de contenedores e inténtelo de nuevo.
1. En el panel de la consola, ejecute el siguiente comando:

    ```
    cd deploymentScript
    ls
    pwsh ./hello.ps1 "John Dole"
    ```

    La salida es **Hello John Dole**.

    ![prueba de instancia de contenedor de script de implementación](./media/deployment-script-template-configure-dev/deployment-script-container-instance-test.png)

## <a name="use-docker"></a>Uso de Docker

Puede usar una imagen de contenedor de Docker preconfigurada como entorno de desarrollo del script de implementación. Para instalar Docker, consulte [Obtener Docker](https://docs.docker.com/get-docker/).
También debe configurar el uso compartido de archivos para montar el directorio que contiene los scripts de implementación en el contenedor de Docker.

1. Extraiga la imagen del contenedor de scripts de implementación en el equipo local:

    ```command
    docker pull mcr.microsoft.com/azuredeploymentscripts-powershell:az4.3
    ```

    En este ejemplo se usa PowerShell versión 4.3.0.

    Para extraer una imagen de la CLI de un Registro de contenedor de Microsoft (MCR):

    ```command
    docker pull mcr.microsoft.com/azure-cli:2.0.80
    ```

    En este ejemplo se usa la CLI versión 2.0.80. El script de implementación usa las imágenes predeterminadas de contenedores de la CLI que se encuentran [aquí](https://hub.docker.com/_/microsoft-azure-cli).

1. Ejecute la imagen de Docker localmente.

    ```command
    docker run -v <host drive letter>:/<host directory name>:/data -it mcr.microsoft.com/azuredeploymentscripts-powershell:az4.3
    ```

    Reemplace **&lt;letra de unidad del host >** y **&lt;nombre del directorio host >** por una carpeta existente en la unidad compartida.  La carpeta se asigna a la carpeta **/data** del contenedor. Por ejemplo, para asignar D:\docker:

    ```command
    docker run -v d:/docker:/data -it mcr.microsoft.com/azuredeploymentscripts-powershell:az4.3
    ```

    **-it** significa mantener activa la imagen del contenedor.

    Un ejemplo de la CLI:

    ```command
    docker run -v d:/docker:/data -it mcr.microsoft.com/azure-cli:2.0.80
    ```

1. En la siguiente captura de pantalla se muestra cómo ejecutar un script de PowerShell, dado que tiene un archivo helloworld.ps1 en la unidad compartida.

    ![CMD de Docker del script de implementación de plantilla de Resource Manager](./media/deployment-script-template/resource-manager-deployment-script-docker-cmd.png)

Después de que el script se pruebe correctamente, puede usarlo como script de implementación en sus plantillas.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, aprendió a usar scripts de implementación. Para seguir un tutorial sobre scripts de implementación:

> [!div class="nextstepaction"]
> [Tutorial: Uso de scripts de implementación en plantillas de Azure Resource Manager](./template-tutorial-deployment-script.md).

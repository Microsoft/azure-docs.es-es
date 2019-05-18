---
title: Utilice canalizaciones de Azure para compilar e implementar soluciones HPC - Azure Batch | Microsoft Docs
description: Obtenga información sobre cómo implementar una canalización de compilación y lanzamiento de una aplicación de HPC que se ejecuta en Azure Batch.
author: christianreddington
ms.author: chredd
ms.date: 03/28/2019
ms.topic: conceptual
ms.custom: fasttrack-new
services: batch
ms.openlocfilehash: a811a9cb1b124aff7c64d25cf71a1b84bff0c173
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65541747"
---
# <a name="use-azure-pipelines-to-build-and-deploy-hpc-solutions"></a>Utilice canalizaciones de Azure para compilar e implementar soluciones de HPC

Servicios de DevOps de Azure proporcionan una gama de herramientas que usan los equipos de desarrollo al compilar una aplicación personalizada. Las herramientas proporcionadas por Azure DevOps pueden traducir a la compilación y pruebas de soluciones de proceso de alto rendimiento automatizadas. En este artículo se muestra cómo configurar una integración continua (CI) y la implementación continua (CD) mediante que canalizaciones de Azure para un alto rendimiento de proceso solución implementada en Azure Batch.

Canalizaciones de Azure proporciona una gama de procesos modernas de CI/CD para compilar, implementar, probar y software de supervisión. Estos procesos aceleran la entrega de software, que le permite centrarse en el código de soporte técnico de infraestructura y operaciones.

## <a name="create-an-azure-pipeline"></a>Crear una canalización de Azure

En este ejemplo, se creará una compilación y versión de canalización para implementar una infraestructura de Azure Batch y liberar un paquete de aplicación. Suponiendo que el código se desarrolla localmente, este es el flujo general de implementación:

![Diagrama que muestra el flujo de implementación en nuestra canalización](media/batch-ci-cd/DeploymentFlow.png)

### <a name="setup"></a>Configuración

Para seguir los pasos descritos en este artículo, necesita una organización de DevOps de Azure y un proyecto de equipo.

* [Creación de una organización de Azure DevOps](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization?view=azure-devops)
* [Crear un proyecto de DevOps de Azure](https://docs.microsoft.com/azure/devops/organizations/projects/create-project?view=azure-devops)

### <a name="source-control-for-your-environment"></a>Control de código fuente para su entorno

Control de código fuente permite a los equipos realizar un seguimiento de los cambios realizados en la base de código e inspeccionar las versiones anteriores del código.

Normalmente, se considera el control de código fuente de mano en mano con el código de software. ¿Qué es la infraestructura subyacente? Esto nos lleva a infraestructura como código, donde vamos a utilizar otras alternativas de código abierto o plantillas de Azure Resource Manager para definir mediante declaración la infraestructura subyacente.

Este ejemplo se basa principalmente en un número de plantillas de Resource Manager (documentos JSON) y los archivos binarios existentes. Puede copiar estos ejemplos en el repositorio y transmitirlos a Azure DevOps.

La estructura del código base utilizada en este ejemplo se parece a lo siguiente:

* Un **plantillas de arm** carpeta, que contiene una serie de plantillas de Azure Resource Manager. Las plantillas se explican en este artículo.
* Un **aplicaciones cliente** carpeta, que es una copia de la [Azure .NET archivo de procesamiento por lotes con ffmpeg](https://github.com/Azure-Samples/batch-dotnet-ffmpeg-tutorial) ejemplo. Esto no es necesario para este artículo.
* Un **aplicación: hpc** carpeta, que es Windows 64 bits versión de [ffmpeg 3.4](https://ffmpeg.zeranoe.com/builds/win64/static/ffmpeg-3.4-win64-static.zip).
* Un **canalizaciones** carpeta. Contiene un archivo YAML esquematización nuestro proceso de compilación. Esto se explica en el artículo.

En esta sección se supone que está familiarizado con las plantillas de Resource Manager de control y el diseño de versión. Si no está familiarizado con estos conceptos, consulte las páginas siguientes para obtener más información.

* [¿Qué es el control de código fuente?](https://docs.microsoft.com/azure/devops/user-guide/source-control?view=azure-devops)
* [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](../azure-resource-manager/resource-group-authoring-templates.md)

#### <a name="azure-resource-manager-templates"></a>Plantillas del Administrador de recursos de Azure

En este ejemplo aprovecha varias plantillas de Resource Manager para implementar nuestra solución. Para ello, usamos una serie de plantillas de función (similares a las unidades o módulos) que implementan una porción de funcionalidad. También se usa una plantilla de solución de extremo a otro que es responsable de llevarlos subyacentes capacidades juntos. Hay un par de ventajas de este enfoque:

* Las plantillas de la funcionalidad subyacente individualmente puede realizar pruebas unitarias.
* Las plantillas de la funcionalidad subyacente pueden definirse como un estándar dentro de una organización y reutilizarse en varias soluciones.

En este ejemplo, hay una plantilla de solución de extremo a otro (deployment.json) que implementa tres plantillas. Las plantillas subyacentes son plantillas de funcionalidad, responsables de implementar un aspecto específico de la solución.

![Ejemplo de estructura de plantilla vinculada mediante plantillas de Azure Resource Manager](media/batch-ci-cd/ARMTemplateHierarchy.png)

La primera plantilla que veremos es para una cuenta de almacenamiento de Azure. Nuestra solución requiere una cuenta de almacenamiento para implementar la aplicación a nuestra cuenta de Batch. Merece la pena conocer el [Guía de referencia de plantilla de Resource Manager para los tipos de recursos Microsoft.Storage](https://docs.microsoft.com/azure/templates/microsoft.storage/allversions) al crear plantillas de Resource Manager para las cuentas de almacenamiento.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "accountName": {
            "type": "string",
            "metadata": {
                 "description": "Name of the Azure Storage Account"
             }
         }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[parameters('accountName')]",
            "sku": {
                "name": "Standard_LRS"
            },
            "apiVersion": "2018-02-01",
            "location": "[resourceGroup().location]",
            "properties": {}
        }
    ],
    "outputs": {
        "blobEndpoint": {
          "type": "string",
          "value": "[reference(resourceId('Microsoft.Storage/storageAccounts', parameters('accountName'))).primaryEndpoints.blob]"
        },
        "resourceId": {
          "type": "string",
          "value": "[resourceId('Microsoft.Storage/storageAccounts', parameters('accountName'))]"
        }
    }
}
```

A continuación, veremos la plantilla de la cuenta de Azure Batch. La cuenta de Batch de Azure actúa como una plataforma para ejecutar muchas aplicaciones en grupos (las agrupaciones de las máquinas). Merece la pena conocer el [Guía de referencia de plantilla de Resource Manager para tipos de recursos de Microsoft.Batch](https://docs.microsoft.com/azure/templates/microsoft.batch/allversions) al crear plantillas de Resource Manager para las cuentas de Batch.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "batchAccountName": {
           "type": "string",
           "metadata": {
                "description": "Name of the Azure Batch Account"
            }
        },
        "storageAccountId": {
           "type": "string",
           "metadata": {
                "description": "ID of the Azure Storage Account"
            }
        }
    },
    "variables": {},
    "resources": [
        {
            "name": "[parameters('batchAccountName')]",
            "type": "Microsoft.Batch/batchAccounts",
            "apiVersion": "2017-09-01",
            "location": "[resourceGroup().location]",
            "properties": {
              "poolAllocationMode": "BatchService",
              "autoStorage": {
                  "storageAccountId": "[parameters('storageAccountId')]"
              }
            }
          }
    ],
    "outputs": {}
}
```

La plantilla siguiente muestra un ejemplo de creación de un grupo de Batch de Azure (las máquinas back-end para procesar las aplicaciones). Merece la pena conocer el [Guía de referencia de plantilla de Resource Manager para tipos de recursos de Microsoft.Batch](https://docs.microsoft.com/azure/templates/microsoft.batch/allversions) al crear plantillas de Resource Manager para grupos de la cuenta de Batch.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "batchAccountName": {
           "type": "string",
           "metadata": {
                "description": "Name of the Azure Batch Account"
           }
        },
        "batchAccountPoolName": {
            "type": "string",
            "metadata": {
                 "description": "Name of the Azure Batch Account Pool"
             }
         }
    },
    "variables": {},
    "resources": [
        {
            "name": "[concat(parameters('batchAccountName'),'/', parameters('batchAccountPoolName'))]",
            "type": "Microsoft.Batch/batchAccounts/pools",
            "apiVersion": "2017-09-01",
            "properties": {
                "deploymentConfiguration": {
                    "virtualMachineConfiguration": {
                        "imageReference": {
                            "publisher": "Canonical",
                            "offer": "UbuntuServer",
                            "sku": "18.04-LTS",
                            "version": "latest"
                        },
                        "nodeAgentSkuId": "batch.node.ubuntu 18.04"
                    }
                },
                "vmSize": "Standard_D1_v2"
            }
          }
    ],
    "outputs": {}
}
```

Por último, tenemos una plantilla que actúa como un orquestador. Esta plantilla es responsable de implementar las plantillas de función.

También puede encontrar más información acerca de [crear plantillas de Azure Resource Manager vinculado](../azure-resource-manager/resource-manager-tutorial-create-linked-templates.md) en otro artículo.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "templateContainerUri": {
           "type": "string",
           "metadata": {
                "description": "URI of the Blob Storage Container containing the Azure Resouce Manager templates"
            }
        },
        "templateContainerSasToken": {
           "type": "string",
           "metadata": {
                "description": "The SAS token of the container containing the Azure Resouce Manager templates"
            }
        },
        "applicationStorageAccountName": {
            "type": "string",
            "metadata": {
                 "description": "Name of the Azure Storage Account"
            }
         },
        "batchAccountName": {
            "type": "string",
            "metadata": {
                 "description": "Name of the Azure Batch Account"
            }
         },
         "batchAccountPoolName": {
             "type": "string",
             "metadata": {
                  "description": "Name of the Azure Batch Account Pool"
              }
          }
    },
    "variables": {},
    "resources": [
        {
            "apiVersion": "2017-05-10",
            "name": "storageAccountDeployment",
            "type": "Microsoft.Resources/deployments",
            "properties": {
                "mode": "Incremental",
                "templateLink": {
                  "uri": "[concat(parameters('templateContainerUri'), '/storageAccount.json', parameters('templateContainerSasToken'))]",
                  "contentVersion": "1.0.0.0"
                },
                "parameters": {
                    "accountName": {"value": "[parameters('applicationStorageAccountName')]"}
                }
            }
        },  
        {
            "apiVersion": "2017-05-10",
            "name": "batchAccountDeployment",
            "type": "Microsoft.Resources/deployments",
            "dependsOn": [
                "storageAccountDeployment"
            ],
            "properties": {
                "mode": "Incremental",
                "templateLink": {
                  "uri": "[concat(parameters('templateContainerUri'), '/batchAccount.json', parameters('templateContainerSasToken'))]",
                  "contentVersion": "1.0.0.0"
                },
                "parameters": {
                    "batchAccountName": {"value": "[parameters('batchAccountName')]"},
                    "storageAccountId": {"value": "[reference('storageAccountDeployment').outputs.resourceId.value]"}
                }
            }
        },
        {
            "apiVersion": "2017-05-10",
            "name": "poolDeployment",
            "type": "Microsoft.Resources/deployments",
            "dependsOn": [
                "batchAccountDeployment"
            ],
            "properties": {
                "mode": "Incremental",
                "templateLink": {
                  "uri": "[concat(parameters('templateContainerUri'), '/batchAccountPool.json', parameters('templateContainerSasToken'))]",
                  "contentVersion": "1.0.0.0"
                },
                "parameters": {
                    "batchAccountName": {"value": "[parameters('batchAccountName')]"},
                    "batchAccountPoolName": {"value": "[parameters('batchAccountPoolName')]"}
                }
            }
        }
    ],
    "outputs": {}
}
```

#### <a name="the-hpc-solution"></a>La solución HPC

La infraestructura y el software pueden definirse como código y colocadas en el mismo repositorio.

Para esta solución, el ffmpeg se utiliza como el paquete de aplicación. Se puede descargar el paquete de ffmpeg [aquí](https://ffmpeg.zeranoe.com/builds/win64/static/ffmpeg-3.4-win64-static.zip).

![Ejemplo de estructura de repositorio de Git](media/batch-ci-cd/git-repository.jpg)

Hay cuatro secciones principales en este repositorio:

* El **plantillas de arm** carpeta que almacena nuestra infraestructura como código
* El **aplicación: hpc** carpeta que contiene los archivos binarios de ffmpeg
* El **canalizaciones** carpeta que contiene la definición para nuestra canalización de compilación.
* **Opcional**: El **aplicaciones cliente** carpeta que almacenaría el código de aplicación. NET. No usamos esto en el ejemplo, pero en su propio proyecto, puede que desee ejecutar las ejecuciones de la aplicación de Batch de HPC a través de una aplicación cliente.

> [!NOTE]
> Esto es sólo un ejemplo de una estructura a un código base. Este enfoque se usa con el fin de demostrar que la aplicación, infraestructura y código de la canalización se almacenan en el mismo repositorio.

Ahora que el código fuente se ha configurado, podemos empezar la primera compilación.

## <a name="continuous-integration"></a>Integración continua

[Las canalizaciones de Azure](https://docs.microsoft.com/azure/devops/pipelines/get-started/?view=azure-devops), dentro de los servicios de Azure DevOps, le ayuda a implementar una canalización de compilación, prueba e implementación para sus aplicaciones.

En esta fase de la canalización, normalmente se ejecutan pruebas para validar el código y compilar los fragmentos adecuados del software. El número y los tipos de pruebas y las tareas adicionales que ejecutan dependerá de la compilación más amplia y estrategia de lanzamiento.

## <a name="preparing-the-hpc-application"></a>Preparación de la aplicación HPC

En este ejemplo, nos centraremos en el **aplicación: hpc** carpeta. El **aplicación: hpc** carpeta es el software de ffmpeg que se ejecutará desde dentro de la cuenta de Azure Batch.

1. Vaya a la sección de las compilaciones de canalizaciones de Azure en su organización de DevOps de Azure. Crear un **nueva canalización**.

    ![Crear una nueva canalización de compilación](media/batch-ci-cd/new-build-pipeline.jpg)

1. Tiene dos opciones para crear una canalización de compilación:

     a. [Mediante el diseñador Visual](https://docs.microsoft.com/azure/devops/pipelines/get-started-designer?view=azure-devops&tabs=new-nav). Para ello, haga clic en "Usar el diseñador visual" en el **nueva canalización** página.

    b. [Crea mediante YAML](https://docs.microsoft.com/azure/devops/pipelines/get-started-yaml?view=azure-devops). Puede crear una nueva canalización YAML al hacer clic en la opción de GitHub en la página nueva de la canalización o repositorios de Azure. Como alternativa, puede almacenar el ejemplo siguiente en el control de código fuente y hacer referencia a un archivo YAML existente haciendo clic en el diseñador Visual y, a continuación, usar la plantilla YAML.

    ```yml
    # To publish an application into Azure Batch, we need to
    # first zip the file, and then publish an artifact, so that
    # we can take the necessary steps in our release pipeline.
    steps:
    # First, we Zip up the files required in the Batch Account
    # For this instance, those are the ffmpeg files
    - task: ArchiveFiles@2
      displayName: 'Archive applications'
      inputs:
        rootFolderOrFile: hpc-application
        includeRootFolder: false
        archiveFile: '$(Build.ArtifactStagingDirectory)/package/$(Build.BuildId).zip'
    # Publish that zip file, so that we can use it as part
    # of our Release Pipeline later
    - task: PublishPipelineArtifact@0
      inputs:
        artifactName: 'hpc-application'
        targetPath: '$(Build.ArtifactStagingDirectory)/package'
    ```

1. Una vez configurada la compilación según sea necesario, seleccione **guardar y poner en cola**. Si tiene habilitada la integración continua (en el **desencadenadores** sección), la compilación desencadenará automáticamente cuando se realiza una nueva confirmación en el repositorio, cumplen las condiciones establecido en la compilación.

    ![Ejemplo de una canalización de compilación existente](media/batch-ci-cd/existing-build-pipeline.jpg)

1. Ver las actualizaciones directas del progreso de la compilación en DevOps de Azure, vaya a la **compilar** sección de canalizaciones de Azure. Seleccione la versión adecuada de la definición de compilación.

    ![Ver salidas en vivo de la compilación](media/batch-ci-cd/Build-1.jpg)

> [!NOTE]
> Si usa una aplicación cliente para ejecutar la aplicación de Batch de HPC, deberá crear una definición de compilación independiente para esa aplicación. Puede encontrar una serie de guías de procedimientos en el [canalizaciones de Azure](https://docs.microsoft.com/azure/devops/pipelines/get-started/index?view=azure-devops) documentación.

## <a name="continuous-deployment"></a>Implementación continua

Las canalizaciones de Azure también se usan para implementar la aplicación y la infraestructura subyacente. [Las canalizaciones de versiones](https://docs.microsoft.com/azure/devops/pipelines/release) es el componente que permite la implementación continua y automatiza el proceso de publicación.

### <a name="deploying-your-application-and-underlying-infrastructure"></a>Implementación de la aplicación y la infraestructura subyacente

Hay una serie de pasos necesarios para implementar la infraestructura. Como hemos usado [plantillas vinculadas](../azure-resource-manager/resource-group-linked-templates.md), esas plantillas deberá ser accesible desde un punto de conexión público (HTTP o HTTPS). Podría tratarse de un repositorio de GitHub, o una cuenta de Azure Blob Storage o en otra ubicación de almacenamiento. Los artefactos de plantilla cargada pueden permanecer seguros, ya que pueden incluirse en un modo privado pero acceder mediante algún tipo de token de acceso compartido (SAS) de firma. El ejemplo siguiente muestra cómo implementar una infraestructura con plantillas de un blob de Azure Storage.

1. Crear un **nueva definición de versión**y seleccione una definición vacía. A continuación, necesitamos cambiar el nombre del entorno recién creado a la importante para nuestra canalización.

    ![Canalización de la versión inicial](media/batch-ci-cd/Release-0.jpg)

1. Cree una dependencia en la canalización de compilación para obtener el resultado de nuestra aplicación HPC.

    > [!NOTE]
    > Una vez más, tenga en cuenta la **Alias de origen**, ya que esto será necesario cuando se crean tareas dentro de la definición de versión.

    ![Cree un vínculo del artefacto a la HPCApplicationPackage en la canalización de compilación](media/batch-ci-cd/Release-1.jpg)

1. Crear un vínculo a otro artefacto, esta vez, un repositorio de Azure. Esto es necesario para tener acceso a las plantillas de Resource Manager almacenadas en el repositorio. Plantillas de Resource Manager no requiere la compilación, no es necesario inserción a ellos a través de una canalización de compilación.

    > [!NOTE]
    > Una vez más, tenga en cuenta la **Alias de origen**, ya que esto será necesario cuando se crean tareas dentro de la definición de versión.

    ![Cree un vínculo de artefacto para los repositorios de Azure](media/batch-ci-cd/Release-2.jpg)

1. Navegue hasta la **variables** sección. Se recomienda para crear una serie de variables en la canalización, por lo que no están introduciendo la misma información en varias tareas. Estas son las variables utilizadas en este ejemplo y cómo afectan la implementación.

    * **applicationStorageAccountName**: Nombre de la cuenta de almacenamiento para almacenar los archivos binarios de aplicación de HPC
    * **batchAccountApplicationName**: Nombre de la aplicación en la cuenta de Batch de Azure
    * **batchAccountName**: Nombre de la cuenta de Azure Batch
    * **batchAccountPoolName**: Nombre del grupo de máquinas virtuales que se encarga del procesamiento
    * **batchApplicationId**: Identificador único para la aplicación de Azure Batch
    * **batchApplicationVersion**: Versión semántica de la aplicación de batch (es decir, los archivos binarios de ffmpeg)
    * **location**: Ubicación para que se implementan los recursos de Azure
    * **resourceGroupName**: Nombre del grupo de recursos que se creará, y donde se implementarán los recursos
    * **storageAccountName**: Nombre de la cuenta de almacenamiento para almacenar las plantillas de Resource Manager vinculadas

    ![Ejemplo de las variables definidas para la versión de las canalizaciones de Azure](media/batch-ci-cd/Release-4.jpg)

1. Vaya a las tareas para el entorno de desarrollo. En la siguiente instantánea, puede ver las seis tareas. Estas tareas le: descargar los archivos zip de ffmpeg, implemente una cuenta de almacenamiento para hospedar las plantillas de Resource Manager anidadas, copie las plantillas de Resource Manager para la cuenta de almacenamiento, implementar la cuenta de batch y las dependencias necesarias, creará una aplicación en la cuenta de Batch de Azure y carga el paquete de aplicación a la cuenta de Batch de Azure.

    ![Ejemplo de las tareas que se utiliza para liberar la aplicación HPC a Azure Batch](media/batch-ci-cd/Release-3.jpg)

1. Agregar el **descargar el artefacto de canalización (versión preliminar)** de tareas y establezca las propiedades siguientes:
    * **Nombre para mostrar:** Descargar ApplicationPackage al agente
    * **El nombre del artefacto para descargar:** aplicación: hpc
    * **Ruta de acceso para descargar en**: $ (System.defaultworkingdirectory)

1. Cree una cuenta de almacenamiento para almacenar los artefactos. Se podría usar una cuenta de almacenamiento existente de la solución, pero para el aislamiento de contenido y autocontenida ejemplo, vamos a hacer que una cuenta de almacenamiento dedicado para nuestros artefactos (específicamente, las plantillas de Resource Manager).

    Agregar el **implementación del grupo de recursos de Azure** de tareas y establezca las propiedades siguientes:
    * **Nombre para mostrar:** Implemente la cuenta de almacenamiento para las plantillas de Resource Manager
    * **Suscripción de Azure:** Seleccione la suscripción de Azure adecuada
    * **Acción**: Creación o actualización del grupo de recursos
    * **Grupo de recursos**: $ (resourcegroupname)
    * **Ubicación**: $(location)
    * **Template**: $(System.ArtifactsDirectory)/**{YourAzureRepoArtifactSourceAlias}**/arm-templates/storageAccount.json
    * **Reemplazar parámetros de plantilla**: - accountName $(storageAccountName)

1. Cargue los artefactos desde el Control de código fuente en la cuenta de almacenamiento. Hay una tarea de canalización de Azure para realizar esto. Como parte de esta tarea, pueden enviar la URL de contenedor de cuenta de almacenamiento y el Token de SAS a una variable en las canalizaciones de Azure. Esto significa que se pueda reutilizar a lo largo de esta fase de agente.

    Agregar el **Azure File Copy** de tareas y establezca las propiedades siguientes:
    * **Source:** $(System.ArtifactsDirectory)/**{YourAzureRepoArtifactSourceAlias}**/arm-templates/
    * **Tipo de conexión Azure**: Azure Resource Manager
    * **Suscripción de Azure:** Seleccione la suscripción de Azure adecuada
    * **Tipo de destino**: Blob de Azure
    * **Cuenta de almacenamiento de RM**: $(storageAccountName)
    * **Nombre del contenedor**: plantillas
    * **URI del contenedor de almacenamiento**: templateContainerUri
    * **Token de SAS del contenedor de almacenamiento**: templateContainerSasToken

1. Implemente la plantilla de orchestrator. Recuerde la plantilla de orchestrator desde versiones anteriores, observará que hay parámetros para la URL de contenedor de cuenta de almacenamiento, además del token SAS. Tenga en cuenta que las variables necesarias en la plantilla de Resource Manager o se mantienen en la sección de variables de la definición de versión o se han definido en otra tarea de canalizaciones de Azure (por ejemplo, la parte de la tarea de copia de Blob de Azure).

    Agregar el **implementación del grupo de recursos de Azure** de tareas y establezca las propiedades siguientes:
    * **Nombre para mostrar:** Implementación de Azure Batch
    * **Suscripción de Azure:** Seleccione la suscripción de Azure adecuada
    * **Acción**: Creación o actualización del grupo de recursos
    * **Grupo de recursos**: $ (resourcegroupname)
    * **Ubicación**: $(location)
    * **Template**: $(System.ArtifactsDirectory)/**{YourAzureRepoArtifactSourceAlias}**/arm-templates/deployment.json
    * **Reemplazar parámetros de plantilla**: ```-templateContainerUri $(templateContainerUri) -templateContainerSasToken $(templateContainerSasToken) -batchAccountName $(batchAccountName) -batchAccountPoolName $(batchAccountPoolName) -applicationStorageAccountName $(applicationStorageAccountName)```

Una práctica común es usar las tareas de Azure Key Vault. Si la entidad de servicio (conexión a su suscripción de Azure) tiene una directivas de acceso adecuado establecido, puede descargar los secretos de Azure Key Vault y se usan como variables en la canalización. El nombre del secreto se establecerá con el valor asociado. Por ejemplo, podría hacer referencia a un secreto de sshPassword con $(sshPassword) en la definición de versión.

1. Los pasos siguientes, llame a la CLI de Azure. La primera se usa para crear una aplicación en Azure Batch. y cargue los paquetes asociados.

    Agregar el **CLI de Azure** de tareas y establezca las propiedades siguientes:
    * **Nombre para mostrar:** Crear aplicaciones en la cuenta de Azure Batch
    * **Suscripción de Azure:** Seleccione la suscripción de Azure adecuada
    * **Ubicación del script**: Script en línea
    * **El Script en línea**: ```az batch application create --application-id $(batchApplicationId) --name $(batchAccountName) --resource-group $(resourceGroupName)```

1. El segundo paso se usa para cargar los paquetes asociados a la aplicación. En nuestro caso, los archivos de ffmpeg.

    Agregar el **CLI de Azure** de tareas y establezca las propiedades siguientes:
    * **Nombre para mostrar:** Cargar el paquete a la cuenta de Azure Batch
    * **Suscripción de Azure:** Seleccione la suscripción de Azure adecuada
    * **Ubicación del script**: Script en línea
    * **El Script en línea**: ```az batch application package create --application-id $(batchApplicationId)  --name $(batchAccountName)  --resource-group $(resourceGroupName) --version $(batchApplicationVersion) --package-file=$(System.DefaultWorkingDirectory)/$(Release.Artifacts.{YourBuildArtifactSourceAlias}.BuildId).zip```

    > [!NOTE]
    > El número de versión del paquete de aplicación se establece en una variable. Esto resulta útil si sobrescribir las versiones anteriores del paquete funciona para usted, y si desea controlar manualmente el número de versión del paquete que se insertan en Azure Batch.

1. Crear una nueva versión seleccionando **versión > crear una nueva versión**. Una vez desencadenada, seleccione el vínculo a la nueva versión para ver el estado.

1. Puede ver la salida en vivo desde el agente seleccionando el **registros** botón debajo de su entorno.

    ![Ver el estado de la versión](media/batch-ci-cd/Release-5.jpg)

### <a name="testing-the-environment"></a>El entorno de pruebas

Una vez configurado el entorno, confirme que las siguientes pruebas se pueden completar correctamente.

Conectarse a la nueva cuenta de Batch de Azure, mediante la CLI de Azure desde un símbolo del sistema de PowerShell.

* Inicie sesión en su cuenta de Azure con `az login` y siga las instrucciones para autenticarse.
* Ahora autenticar la cuenta de Batch: `az batch account login -g <resourceGroup> -n <batchAccount>`

#### <a name="list-the-available-applications"></a>Enumerar las aplicaciones disponibles

```azurecli
az batch application list -g <resourcegroup> -n <batchaccountname>
```

#### <a name="check-the-pool-is-valid"></a>Compruebe que el grupo es válido

```azurecli
az batch pool list
```

Tenga en cuenta el valor de `currentDedicatedNodes` desde la salida de este comando. Este valor se ajusta en la siguiente prueba.

#### <a name="resize-the-pool"></a>El tamaño del grupo

El tamaño del grupo, por lo que hay nodos disponibles para su trabajo y las pruebas de la tarea de proceso, póngase en contacto con el comando de la lista de grupo para ver el estado actual hasta que se ha completado el cambio de tamaño y hay nodos disponibles

```azurecli
az batch pool resize --pool-id <poolname> --target-dedicated-nodes 4
```

## <a name="next-steps"></a>Pasos siguientes

Además de este artículo, hay dos tutoriales que usan ffmpeg, con .NET y Python. Consulte estos tutoriales para obtener más información sobre cómo interactuar con una cuenta de Batch a través de una aplicación sencilla.

* [Ejecutar una carga de trabajo paralela con Azure Batch con la API de Python](tutorial-parallel-python.md)
* [Ejecutar una carga de trabajo paralela con Azure Batch con la API de .NET](tutorial-parallel-dotnet.md)

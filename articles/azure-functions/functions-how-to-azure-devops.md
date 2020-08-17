---
title: Actualización continua del código de aplicación de funciones mediante Azure DevOps
description: Aprenda a configurar una canalización de Azure DevOps con Azure Functions como destino.
author: craigshoemaker
ms.topic: conceptual
ms.date: 04/18/2019
ms.author: cshoe
ms.custom: devx-track-python, devx-track-azurecli
ms.openlocfilehash: 15ac5bde6029b5608ff02c35451d4041bec29796
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87853354"
---
# <a name="continuous-delivery-by-using-azure-devops"></a>Entrega continua con Azure DevOps

Puede implementar automáticamente su función en una aplicación de Azure Functions con [Azure Pipelines](/azure/devops/pipelines/).

Tiene dos opciones para definir la canalización:

- **Archivo YAML**: archivo YAML que describe la canalización. Es posible que el archivo incluya una sección de pasos de compilación y una sección de versión. El archivo YAML debe estar en el mismo repositorio que la aplicación.
- **Plantilla**: las plantillas son tareas preparadas que compilan o implementan la aplicación.

## <a name="yaml-based-pipeline"></a>Canalización basada en YAML

Para crear una canalización basada en YAML, compile primero la aplicación y luego impleméntela.

### <a name="build-your-app"></a>Compilación de la aplicación

La compilación de la aplicación en Azure Pipelines depende del lenguaje de programación de la aplicación. Cada lenguaje tiene pasos de compilación específicos que crean un artefacto de implementación. Se usa un artefacto de implementación para implementar la aplicación de funciones en Azure.

# <a name="c"></a>[C\#](#tab/csharp)

Puede utilizar el ejemplo siguiente con el fin de crear el archivo YAML para compilar la aplicación de .NET:

```yaml
pool:
      vmImage: 'VS2017-Win2016'
steps:
- script: |
    dotnet restore
    dotnet build --configuration Release
- task: DotNetCoreCLI@2
  inputs:
    command: publish
    arguments: '--configuration Release --output publish_output'
    projects: '*.csproj'
    publishWebProjects: false
    modifyOutputPath: false
    zipAfterPublish: false
- task: ArchiveFiles@2
  displayName: "Archive files"
  inputs:
    rootFolderOrFile: "$(System.DefaultWorkingDirectory)/publish_output"
    includeRootFolder: false
    archiveFile: "$(System.DefaultWorkingDirectory)/build$(Build.BuildId).zip"
- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: '$(System.DefaultWorkingDirectory)/build$(Build.BuildId).zip'
    artifactName: 'drop'
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Puede utilizar el ejemplo siguiente con el fin de crear el archivo YAML para compilar la aplicación de JavaScript:

```yaml
pool:
      vmImage: ubuntu-16.04 # Use 'VS2017-Win2016' if you have Windows native +Node modules
steps:
- bash: |
    if [ -f extensions.csproj ]
    then
        dotnet build extensions.csproj --output ./bin
    fi
    npm install 
    npm run build --if-present
    npm prune --production
- task: ArchiveFiles@2
  displayName: "Archive files"
  inputs:
    rootFolderOrFile: "$(System.DefaultWorkingDirectory)"
    includeRootFolder: false
    archiveFile: "$(System.DefaultWorkingDirectory)/build$(Build.BuildId).zip"
- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: '$(System.DefaultWorkingDirectory)/build$(Build.BuildId).zip'
    artifactName: 'drop'
```

# <a name="python"></a>[Python](#tab/python)

Puede usar uno de los ejemplos siguientes con el fin de crear un archivo YAML para crear una aplicación para una versión específica de Python. Python solo es compatible con las aplicaciones de funciones que se ejecutan en Linux.

**Versión 3.7**

```yaml
pool:
  vmImage: ubuntu-16.04
steps:
- task: UsePythonVersion@0
  displayName: "Setting python version to 3.7 as required by functions"
  inputs:
    versionSpec: '3.7'
    architecture: 'x64'
- bash: |
    if [ -f extensions.csproj ]
    then
        dotnet build extensions.csproj --output ./bin
    fi
    pip install --target="./.python_packages/lib/site-packages" -r ./requirements.txt
- task: ArchiveFiles@2
  displayName: "Archive files"
  inputs:
    rootFolderOrFile: "$(System.DefaultWorkingDirectory)"
    includeRootFolder: false
    archiveFile: "$(System.DefaultWorkingDirectory)/build$(Build.BuildId).zip"
- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: '$(System.DefaultWorkingDirectory)/build$(Build.BuildId).zip'
    artifactName: 'drop'
```

**Versión 3.6**

```yaml
pool:
  vmImage: ubuntu-16.04
steps:
- task: UsePythonVersion@0
  displayName: "Setting python version to 3.6 as required by functions"
  inputs:
    versionSpec: '3.6'
    architecture: 'x64'
- bash: |
    if [ -f extensions.csproj ]
    then
        dotnet build extensions.csproj --output ./bin
    fi
    pip install --target="./.python_packages/lib/python3.6/site-packages" -r ./requirements.txt
- task: ArchiveFiles@2
  displayName: "Archive files"
  inputs:
    rootFolderOrFile: "$(System.DefaultWorkingDirectory)"
    includeRootFolder: false
    archiveFile: "$(System.DefaultWorkingDirectory)/build$(Build.BuildId).zip"
- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: '$(System.DefaultWorkingDirectory)/build$(Build.BuildId).zip'
    artifactName: 'drop'
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Puede utilizar el ejemplo siguiente con el fin de crear el archivo YAML para compilar la aplicación de PowerShell. PowerShell solo es compatible con Linux Azure Functions.

```yaml
pool:
      vmImage: 'VS2017-Win2016'
steps:
- task: ArchiveFiles@2
  displayName: "Archive files"
  inputs:
    rootFolderOrFile: "$(System.DefaultWorkingDirectory)"
    includeRootFolder: false
    archiveFile: "$(System.DefaultWorkingDirectory)/build$(Build.BuildId).zip"
- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: '$(System.DefaultWorkingDirectory)/build$(Build.BuildId).zip'
    artifactName: 'drop'
```

---

### <a name="deploy-your-app"></a>Implementación de la aplicación

En función del sistema operativo de hospedaje, deberá incluir uno de los siguientes ejemplos YAML en el archivo YAML.

#### <a name="windows-function-app"></a>Aplicación de funciones de Windows

Puede usar el siguiente fragmento de código para implementar una aplicación de funciones de Windows:

```yaml
steps:
- task: AzureFunctionApp@1
  inputs:
    azureSubscription: '<Azure service connection>'
    appType: functionApp
    appName: '<Name of function app>'
    #Uncomment the next lines to deploy to a deployment slot
    #deployToSlotOrASE: true
    #resourceGroupName: '<Resource Group Name>'
    #slotName: '<Slot name>'
```

#### <a name="linux-function-app"></a>Aplicación de funciones de Linux

Puede usar el siguiente fragmento de código para implementar una aplicación de funciones de Linux:

```yaml
steps:
- task: AzureFunctionApp@1
  inputs:
    azureSubscription: '<Azure service connection>'
    appType: functionAppLinux
    appName: '<Name of function app>'
    #Uncomment the next lines to deploy to a deployment slot
    #Note that deployment slots is not supported for Linux Dynamic SKU
    #deployToSlotOrASE: true
    #resourceGroupName: '<Resource Group Name>'
    #slotName: '<Slot name>'
```

## <a name="template-based-pipeline"></a>Canalización basada en plantilla

Las plantillas de Azure DevOps son un grupo predefinido de tareas que compilan o implementan una aplicación.

### <a name="build-your-app"></a>Compilación de la aplicación

La compilación de la aplicación en Azure Pipelines depende del lenguaje de programación de la aplicación. Cada lenguaje tiene pasos de compilación específicos que crean un artefacto de implementación. Se usa un artefacto de implementación para actualizar la aplicación de funciones en Azure.

Con el fin de usar las plantillas de compilación integradas, al crear una nueva canalización de compilación, seleccione **Usar el editor clásico** para crear una canalización mediante las plantillas de diseñador.

![Selección del editor clásico de Azure Pipelines](media/functions-how-to-azure-devops/classic-editor.png)

Después de configurar el origen del código, busque las plantillas de compilación de Azure Functions. Seleccione la plantilla que coincida con el idioma de la aplicación.

![Selección de una plantilla de compilación de Azure Functions](media/functions-how-to-azure-devops/build-templates.png)

En algunos casos, los artefactos de compilación tienen una estructura de carpetas específica. Es posible que tenga que activar la casilla **Anteponer el nombre de la carpeta raíz a las rutas de acceso de archivos**.

![Opción para anteponer el nombre de la carpeta raíz](media/functions-how-to-azure-devops/prepend-root-folder.png)

#### <a name="javascript-apps"></a>Aplicaciones de JavaScript

Si la aplicación de JavaScript tiene una dependencia en los módulos nativos de Windows, debe actualizar la versión del grupo de agentes a **Hosted VS2017**.

![Actualización de la versión del grupo de agentes](media/functions-how-to-azure-devops/change-agent.png)

### <a name="deploy-your-app"></a>Implementación de la aplicación

Cuando cree una nueva canalización de versión, busque la plantilla de versión de Azure Functions.

![Búsqueda de la plantilla de versión de Azure Functions](media/functions-how-to-azure-devops/release-template.png)

En la plantilla de versión no se admite la implementación en una ranura de implementación.

## <a name="create-a-build-pipeline-by-using-the-azure-cli"></a>Creación de una canalización de compilación mediante la CLI de Azure

Para crear una canalización de compilación en Azure, use el [comando](/cli/azure/functionapp/devops-pipeline#az-functionapp-devops-pipeline-create) `az functionapp devops-pipeline create`. La canalización de compilación se crea para compilar y liberar los cambios de código que se realicen en el repositorio. El comando generará un nuevo archivo YAML que define la canalización de versión y compilación y lo confirma en el repositorio. Los requisitos previos para este comando dependen de la ubicación del código.

- Si el código está en GitHub:

    - Debe contar con permisos de **escritura** en su suscripción.

    - Debe ser el administrador del proyecto en Azure DevOps.

    - Debe tener permisos para crear un token de acceso personal (PAT) en GitHub que tenga permisos suficientes. Para obtener más información, consulte [Requisitos de permiso de token de acceso personal en GitHub](https://aka.ms/azure-devops-source-repos).

    - Debe tener permisos para confirmar en la rama maestra del repositorio de GitHub con el fin de confirmar el archivo YAML generado automáticamente.

- Si el código está en Azure Repos:

    - Debe contar con permisos de **escritura** en su suscripción.

    - Debe ser el administrador del proyecto en Azure DevOps.

## <a name="next-steps"></a>Pasos siguientes

- Revise la [introducción a Azure Functions](functions-overview.md).
- Revise la [introducción a Azure DevOps](/azure/devops/pipelines/).

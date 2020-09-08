---
title: Adición de un repositorio de artefactos a un laboratorio en Azure DevTest Labs | Microsoft Docs
description: Obtenga información sobre cómo especificar su propio repositorio de artefactos para el laboratorio en Azure DevTest Labs, a fin de almacenar herramientas que no están disponibles en el repositorio de artefactos públicos.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 7553f6b1afa416a5428577a8313bdadb669e32c2
ms.sourcegitcommit: 2bab7c1cd1792ec389a488c6190e4d90f8ca503b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88270995"
---
# <a name="add-an-artifact-repository-to-your-lab-in-devtest-labs"></a>Agregar un repositorio de artefactos a un laboratorio en Azure DevTest Labs
DevTest Labs permite especificar que se agregue un artefacto a una máquina virtual en el mismo momento de crear la máquina virtual o después de que se haya creado. Este artefacto puede ser una herramienta o una aplicación que desee instalar en la máquina virtual. Los artefactos se definen en un archivo JSON que se carga desde un repositorio GitHub o un repositorio GIT de Azure DevOps.

El [repositorio de artefactos público](https://github.com/Azure/azure-devtestlab/tree/master/Artifacts), que mantiene DevTest Labs, proporciona muchas herramientas comunes tanto para Windows como para Linux. Automáticamente se agrega un vínculo a este repositorio al laboratorio. Puede crear su propio repositorio de artefactos con herramientas concretas que no estén disponibles en el repositorio de artefactos público. Para aprender a crear artefactos personalizados, consulte [Creación de artefactos personalizados](devtest-lab-artifact-author.md).

En este artículo se proporciona información acerca de cómo agregar un repositorio de artefactos personalizados desde Azure Portal, plantillas de Azure Resource Management y Azure PowerShell. La incorporación de un repositorio de artefactos a un laboratorio se puede automatizar mediante la escritura de scripts de PowerShell o CLI.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Requisitos previos
Para agregar un repositorio al laboratorio, obtenga cierta información clave del repositorio. En las secciones siguientes, se describe cómo obtener la información requerida para los repositorios hospedados en **GitHub** o **Azure DevOps**.

### <a name="get-the-github-repository-clone-url-and-personal-access-token"></a>Obtención de la dirección URL de clonación del repositorio de GitHub y el token de acceso personal

1. Vaya a la página principal del repositorio de GitHub que contiene las definiciones de artefacto o de la plantilla de Azure Resource Manager.
2. Seleccione **Clone or download**(Clonar o descargar).
3. Para copiar la dirección URL en el Portapapeles, seleccione el botón **Dirección URL de clonación de HTTPS**. Guarde la dirección URL para su uso posterior.
4. En la esquina superior derecha de GitHub, seleccione la imagen del perfil y, después, seleccione **Configuración**.
5. En el menú **Configuración personal** de la izquierda, seleccione **Configuración de desarrollador**.
6. Seleccione **Tokens de acceso personal** en el menú de la izquierda.
7. Seleccione **Generar nuevo token**.
8. En la página **New personal access token** (Nuevo token de acceso personal), bajo **Descripción de token** escriba una descripción. Acepte los elementos predeterminados en **Seleccionar ámbitos** y, a continuación, seleccione **Generar token**.
9. Guarde el token generado. Usará el token más adelante.
10. Cierre GitHub.   

### <a name="get-the-azure-repos-clone-url-and-personal-access-token"></a>Obtención de la dirección URL de clonación de Azure Repos y el token de acceso personal
1. Vaya a la página principal de la colección de equipo (por ejemplo, `https://contoso-web-team.visualstudio.com`) y, después, seleccione el proyecto.
2. En la página de inicio del proyecto, seleccione **Código**.
3. Para ver la dirección URL de clonación, en la página **Código** del proyecto, seleccione **Clonar**.
4. Guarde la dirección URL. Usará la dirección URL más adelante.
5. Para crear un token de acceso personal, en el menú desplegable de la cuenta de usuario, seleccione **Mi perfil**.
6. En la página de información de perfil, seleccione **Seguridad**.
7. En la pestaña **Seguridad > Tokens de acceso personal**, seleccione **+ Nuevo Token**.
8. En la página **Crear un token de acceso personal**:
   1. Rellene el campo **Nombre** con un nombre para el token.
   2. En la lista **Organización**, seleccione **Todas las organizaciones accesibles**.
   3. En la lista **Expiración (UTC)** , seleccione **90 días** o un período de expiración definido personalizado.
   4. Seleccione la opción **Acceso Completo** para los ámbitos.
   5. Seleccione **Crear**.
9. El nuevo token aparece en la lista **Tokens de acceso personal** . Seleccione **token**y luego guarde el valor del token para usarlo más adelante.
10. Vaya a la sección Conexión de un laboratorio con el repositorio.

## <a name="use-azure-portal"></a>Usar Azure Portal
En esta sección se proporcionan los pasos necesarios para agregar un repositorio de artefactos a un laboratorio en Azure Portal.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **Más servicios** y, luego, **DevTest Labs** en la lista de servicios.
3. En la lista de laboratorios, seleccione el suyo.
4. Seleccione **Configuración y directivas** en el menú de la izquierda.
5. Seleccione **Repositorios** en la sección **Recursos externos** del menú izquierdo.
6. Seleccione **Agregar** en la barra de herramientas.

    ![Botón Agregar repositorio](./media/devtest-lab-add-repo/devtestlab-add-repo.png)
5. En la página **Repositorios**, especifique la información siguiente:
   1. **Nombre**. Escriba un nombre para el repositorio.
   2. **URL de clonación de Git**. Escriba la dirección URL de clonación HTTPS de GIT que copió anteriormente de GitHub o de Azure DevOps Services.
   3. **Rama**. Escriba la rama para obtener las definiciones.
   4. **Token de acceso personal**. Especifique el token de acceso personal que obtuvo anteriormente de GitHub o de Azure DevOps Services.
   5. **Rutas de acceso de carpeta**. Escriba al menos una ruta de acceso de carpeta con respecto a la dirección URL de clonación que contenga las definiciones del artefacto o de la plantilla de Azure Resource Manager. Cuando especifique un subdirectorio, asegúrese de incluir la barra diagonal en la ruta de acceso de la carpeta.

        ![Área Repositorios](./media/devtest-lab-add-repo/devtestlab-repo-blade.png)
6. Seleccione **Guardar**.

## <a name="use-azure-resource-manager-template"></a>Usar plantillas de Azure Resource Manager
Las plantillas de Azure Resource Management (Azure Resource Manager) son archivos JSON que describen los recursos de Azure que se desean crear. Para más información acerca de dichas plantillas, consulte [Creación de plantillas de Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md).

En esta sección se proporcionan los pasos necesarios para agregar un repositorio de artefactos a un laboratorio mediante una plantilla de Azure Resource Manager.  La plantilla crea el laboratorio, en caso de que no exista.

### <a name="template"></a>Plantilla
La plantilla de ejemplo que se usa en este artículo, recopila la siguiente información a través de parámetros. La mayoría de los parámetros tienen valores predeterminados inteligentes, pero hay algunos valores que deben especificarse. Es preciso especificar el nombre del laboratorio, el identificador URI del repositorio de artefactos y el token de seguridad del repositorio.

- Nombre de laboratorio.
- Nombre para mostrar del repositorio de artefactos en la interfaz de usuario (UI) de DevTest Labs. El valor predeterminado es: `Team Repository`.
- Identificador URI del repositorio (por ejemplo: `https://github.com/<myteam>/<nameofrepo>.git` o `"https://MyProject1.visualstudio.com/DefaultCollection/_git/TeamArtifacts"`.
- Rama del repositorio que contiene artefactos. El valor predeterminado es: `master`.
- Asigne un nombre a la carpeta que contiene los artefactos. El valor predeterminado es: `/Artifacts`.
- Tipo del repositorio. Los valores permitidos son: `VsoGit` o `GitHub`.
- Token de acceso del repositorio.

    ```json
    {

        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "labName": {
                "type": "string"
            },
            "artifactRepositoryDisplayName": {
                "type": "string",
                "defaultValue": "Team Repository"
            },
            "artifactRepoUri": {
                "type": "string"
            },
            "artifactRepoBranch": {
                "type": "string",
                "defaultValue": "master"
            },
            "artifactRepoFolder": {
                "type": "string",
                "defaultValue": "/Artifacts"
            },
            "artifactRepoType": {
                "type": "string",
                "allowedValues": ["VsoGit", "GitHub"]
            },
            "artifactRepoSecurityToken": {
                "type": "securestring"
            }
        },
        "variables": {
            "artifactRepositoryName": "[concat('Repo-', uniqueString(subscription().subscriptionId))]"
        },
        "resources": [{
                "apiVersion": "2016-05-15",
                "type": "Microsoft.DevTestLab/labs",
                "name": "[parameters('labName')]",
                "location": "[resourceGroup().location]",
                "resources": [
                    {
                        "apiVersion": "2016-05-15",
                        "name": "[variables('artifactRepositoryName')]",
                        "type": "artifactSources",
                        "dependsOn": [
                            "[resourceId('Microsoft.DevTestLab/labs', parameters('labName'))]"
                        ],
                        "properties": {
                            "uri": "[parameters('artifactRepoUri')]",
                            "folderPath": "[parameters('artifactRepoFolder')]",
                            "branchRef": "[parameters('artifactRepoBranch')]",
                            "displayName": "[parameters('artifactRepositoryDisplayName')]",
                            "securityToken": "[parameters('artifactRepoSecurityToken')]",
                            "sourceType": "[parameters('artifactRepoType')]",
                            "status": "Enabled"
                        }
                    }
                ]
            }
        ]
    }
    ```


### <a name="deploy-the-template"></a>Implementación de la plantilla
Hay varias formas de implementar la plantilla en Azure y de que el recurso se cree, si no existe, o se actualice, si existe. Para obtener detalles, vea los siguientes artículos:

- [Implementación de recursos con las plantillas de Resource Manager y Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md)
- [Implementación de recursos con plantillas de Resource Manager y la CLI de Azure](../azure-resource-manager/templates/deploy-cli.md)
- [Implementación de recursos con las plantillas de Resource Manager y Azure Portal](../azure-resource-manager/templates/deploy-portal.md)
- [Implementación de recursos con las plantillas de Resource Manager y la API de REST de Resource Manager](../azure-resource-manager/templates/deploy-rest.md)

Sigamos y veamos cómo implementar la plantilla en PowerShell. Los cmdlets que se usan para implementar la plantilla son específicos del contexto, por lo que se usan el inquilino y la suscripción actuales. Use [Set-AzContext](/powershell/module/az.accounts/set-azcontext) antes de implementar la plantilla, si fuera necesario, para cambiar el contexto.

En primer lugar, cree un grupo de recursos mediante [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Si el grupo de recursos que desea utilizar ya existe, omita este paso.

```powershell
New-AzResourceGroup -Name MyLabResourceGroup1 -Location westus
```

Luego, cree una implementación para el grupo de recursos mediante [New AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment). Este cmdlet aplica los cambios de recursos en Azure. Se pueden realizar varias implementaciones de recursos en cualquier grupo de recursos determinado. Si va a realizar la implementación varias veces en el mismo grupo de recursos, asegúrese de que el nombre de cada implementación es único.

```powershell
New-AzResourceGroupDeployment `
    -Name MyLabResourceGroup-Deployment1 `
    -ResourceGroupName MyLabResourceGroup1 `
    -TemplateFile azuredeploy.json `
    -TemplateParameterFile azuredeploy.parameters.json
```

Después de que New-AzResourceGroupDeployment se ejecute correctamente, el comando genera información importante, como el estado de aprovisionamiento (debería ser correcto) y los resultados de la plantilla.

## <a name="use-azure-powershell"></a>Uso de Azure PowerShell
En esta sección se proporciona un script de PowerShell de ejemplo que se puede usar para agregar un repositorio de artefactos a un laboratorio. Si no tiene Azure PowerShell, consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/?view=azps-1.2.0) para obtener instrucciones detalladas para instalarlo.

### <a name="full-script"></a>Script completo
Este es el script completo, incluidos algunos mensajes detallados y comentarios:

**New-DevTestLabArtifactRepository.ps1**:

```powershell

<#

.SYNOPSIS
This script creates a new custom repository and adds it to an existing DevTest Lab.

.PARAMETER LabName
The name of the lab.

.PARAMETER LabResourceGroupName
The name of the resource group that contains the lab.

.PARAMETER ArtifactRepositoryName
Name for the new artifact repository.
Script creates a random name for the repository if it is not specified.

.PARAMETER ArtifactRepositoryDisplayName
Display name for the artifact repository.
This is the name that shows up in the Azure portal (https://portal.azure.com) when viewing all the artifact repositories for a lab.

.PARAMETER RepositoryUri
Uri to the repository.

.PARAMETER RepositoryBranch
Branch in which artifact files can be found. Defaults to 'master'.

.PARAMETER FolderPath
Folder under which artifacts can be found. Defaults to '/Artifacts'

.PARAMETER PersonalAccessToken
Security token for access to GitHub or VSOGit repository.
See https://azure.microsoft.com/documentation/articles/devtest-lab-add-artifact-repo/ for instructions to get personal access token

.PARAMETER SourceType
Whether artifact is VSOGit or GitHub repository.

.EXAMPLE
Set-AzContext -SubscriptionId 11111111-1111-1111-1111-111111111111
.\New-DevTestLabArtifactRepository.ps1 -LabName "mydevtestlab" -LabResourceGroupName "mydtlrg" -ArtifactRepositoryName "MyTeam Repository" -RepositoryUri "https://github.com/<myteam>/<nameofrepo>.git" -PersonalAccessToken "1111...." -SourceType "GitHub"

.NOTES
Script uses the current Az context. To set the context, use the Set-AzContext cmdlet

#>


[CmdletBinding()]
Param(

    [Parameter(Mandatory=$true)]
    $LabName,

    [Parameter(Mandatory=$true)]
    $LabResourceGroupName,
    $ArtifactRepositoryName,
    $ArtifactRepositoryDisplayName  = 'Team Artifact Repository',

    [Parameter(Mandatory=$true)]
    $RepositoryUri,
    $RepositoryBranch = 'master',
    $FolderPath = '/Artifacts',

    [Parameter(Mandatory=$true)]
    $PersonalAccessToken ,

    [Parameter(Mandatory=$true)]
    [ValidateSet('VsoGit', 'GitHub')]
    $SourceType
)


#Set artifact repository internal name,
# if not set by user.

if ($ArtifactRepositoryName -eq $null){
    $ArtifactRepositoryName = "PrivateRepo" + (Get-Random -Maximum 999)
}

# Sign in to Azure
Connect-AzAccount


#Get Lab Resource
$LabResource = Get-AzResource -ResourceType 'Microsoft.DevTestLab/labs' -ResourceName $LabName -ResourceGroupName $LabResourceGroupName

Write-Verbose "Lab Name: $($LabResource.Name)"
Write-Verbose "Lab Resource Group Name: $($LabResource.ResourceGroupName)"
Write-Verbose "Lab Resource Location: $($LabResource.Location)"

Write-Verbose "Artifact Repository Internal Name: $ArtifactRepositoryName"

#Prepare properties object for call to New-AzResource
$propertiesObject = @{
    uri = $RepositoryUri;
    folderPath = $FolderPath;
    branchRef = $RepositoryBranch;
    displayName = $ArtifactRepositoryDisplayName;
    securityToken = $PersonalAccessToken;
    sourceType = $SourceType;
    status = 'Enabled'
}

Write-Verbose @"Properties to be passed to New-AzResource:$($propertiesObject | Out-String)"

#Resource will be added to current subscription.
$resourcetype = 'Microsoft.DevTestLab/labs/artifactSources'
$resourceName = $LabName + '/' + $ArtifactRepositoryName
Write-Verbose "Az ResourceType: $resourcetype"
Write-Verbose "Az ResourceName: $resourceName"

Write-Verbose "Creating artifact repository '$ArtifactRepositoryDisplayName'..."
$result = New-AzResource -Location $LabResource.Location -ResourceGroupName $LabResource.ResourceGroupName -properties $propertiesObject -ResourceType $resourcetype -ResourceName $resourceName -ApiVersion 2016-05-15 -Force


#Alternate implementation:
# Use resourceId rather than resourcetype and resourcename parameters.
# Using resourceId allows you to specify the $SubscriptionId rather than using the
# subscription id of Get-AzContext.
#$resourceId = "/subscriptions/$SubscriptionId/resourceGroups/$($LabResource.ResourceGroupName)/providers/Microsoft.DevTestLab/labs/$LabName/artifactSources/$ArtifactRepositoryName"
#$result = New-AzResource -properties $propertiesObject -ResourceId $resourceId -ApiVersion 2016-05-15 -Force


# Check the result
if ($result.Properties.ProvisioningState -eq "Succeeded") {
    Write-Verbose ("Successfully added artifact repository source '$ArtifactRepositoryDisplayName'")
}
else {
    Write-Error ("Error adding artifact repository source '$ArtifactRepositoryDisplayName'")
}

#Return the newly created resource so it may be used in subsequent scripts
return $result
```

### <a name="run-the-powershell-script"></a>Ejecución del script de PowerShell
En el ejemplo siguiente se muestra cómo ejecutar el script:

```powershell
Set-AzContext -SubscriptionId <Your Azure subscription ID>

.\New-DevTestLabArtifactRepository.ps1 -LabName "mydevtestlab" -LabResourceGroupName "mydtlrg" -ArtifactRepositoryName "MyTeam Repository" -RepositoryUri "https://github.com/<myteam>/<nameofrepo>.git" -PersonalAccessToken "1111...." -SourceType "GitHub"
```


### <a name="parameters"></a>Parámetros
El script de PowerShell de ejemplo de este artículo usa los parámetros siguientes:

| Parámetro | Descripción |
| --------- | ----------- |
| LabName | El nombre del laboratorio. |
| ArtifactRepositoryName | Nombre del nuevo repositorio de artefactos. El script crea un nombre aleatorio para el repositorio si no se especifica. |
| ArtifactRepositoryDisplayName | Nombre para mostrar del nuevo repositorio de artefactos. Es el nombre que se muestra en Azure Portal (https://portal.azure.com) cuando se ven todos los repositorios de artefactos de un laboratorio. |
| RepositoryUri | Identificador URI del repositorio. Ejemplos: `https://github.com/<myteam>/<nameofrepo>.git` o `"https://MyProject1.visualstudio.com/DefaultCollection/_git/TeamArtifacts"`.|
| RepositoryBranch | Rama en que se pueden encontrar archivos de artefacto. El valor predeterminado es "master". |
| FolderPath | Carpeta en la que se pueden encontrar los artefactos. El valor predeterminado es "/Artifacts" |
| PersonalAccessToken | Token de seguridad para acceder al repositorio de GitHub o de VSOGit. Consulte la sección de requisitos previos para obtener las instrucciones necesarias para obtener un token de acceso personal |
| SourceType | Si el artefacto es repositorio de VSOGit o de GitHub. |

El repositorio en sí necesita un interno de identificación nombre, que no es el mismo que el nombre para mostrar que se ve en Azure Portal. Desde Azure Portal no se ve el nombre interno, pero se ve cuando se usan las API REST de Azure o Azure PowerShell. El script proporciona un nombre, siempre que no lo haya especificado el usuario del script.

```powershell
#Set artifact repository name, if not set by user
if ($ArtifactRepositoryName -eq $null){
    $ArtifactRepositoryName = "PrivateRepo" + (Get-Random -Maximum 999)
}
```

### <a name="powershell-commands-used-in-the-script"></a>Comandos de PowerShell que se usan en el script

| Comando de PowerShell | Notas |
| ------------------ | ----- |
| [Get-AzResource](/powershell/module/az.resources/get-azresource) | Este comando se usa para obtener detalles del laboratorio, como su ubicación. |
| [New-AzResource](/powershell/module/az.resources/new-azresource) | No hay ningún comando específico para agregar repositorios de artefactos. El cmdlet genérico [New AzResource](/powershell/module/az.resources/new-azresource) realiza el trabajo. Este cmdlet necesita **ResourceId** o el par **Nombre de recurso** y **Tipo de recurso** para saber el tipo de recurso que debe crear. Este script de ejemplo usa el par nombre de recurso y tipo de recurso. <br/><br/>Tenga en cuenta que va a crear el origen del repositorio de artefactos en la misma ubicación y en el mismo grupo de recursos que el laboratorio.|

El script agrega un nuevo recurso a la suscripción actual. Use [Get-AzContext](/powershell/module/az.accounts/get-azcontext) para ver esta información. Use [Set-AzContext](/powershell/module/az.accounts/set-azcontext) para establecer el inquilino y la suscripción actuales.

La mejor manera de detectar la información de nombre de recurso y tipo de recurso es usar el sitio web en que se [prueban las API REST de Azure](https://azure.github.io/projects/apis/) sitio Web. Extraiga del repositorio el proveedor de [DevTest Labs: 2016-05-15](https://aka.ms/dtlrestapis) para ver las API REST disponibles para el proveedor de DevTest Labs. El script usa el siguiente identificador de recurso.

```powershell
"/subscriptions/$SubscriptionId/resourceGroups/$($LabResource.ResourceGroupName)/providers/Microsoft.DevTestLab/labs/$LabName/artifactSources/$ArtifactRepositoryName"
```

El tipo de recurso es todo lo que aparece después de "providers" en el identificador URI, excepto los elementos que aparecen entre corchetes. El nombre del recurso es todo lo que ve entre corchetes. Si se espera que más de un elemento para el nombre de recurso, separe cada elemento con una barra diagonal, tal como hemos hecho.

```powershell
$resourcetype = 'Microsoft.DevTestLab/labs/artifactSources'
$resourceName = $LabName + '/' + $ArtifactRepositoryName
```


## <a name="next-steps"></a>Pasos siguientes
- [Especificación de artefactos obligatorios para su laboratorio de Azure DevTest Labs](devtest-lab-mandatory-artifacts.md)
- [Creación de artefactos personalizados para una máquina virtual de DevTest Labs](devtest-lab-artifact-author.md)
- [Diagnóstico de errores de artefactos en el laboratorio](devtest-lab-troubleshoot-artifact-failure.md)

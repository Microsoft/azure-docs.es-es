---
title: Creación de entornos de varias máquinas virtuales y recursos PaaS con plantillas
description: Obtenga información acerca de cómo crear entornos de varias máquinas virtuales y recursos de PaaS en Azure DevTest Labs a partir de una plantilla de Azure Resource Manager
ms.topic: article
ms.date: 08/12/2020
ms.openlocfilehash: 97659d4ab95fdbe75460161d0ceed71a1cb5cf82
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88182415"
---
# <a name="create-multi-vm-environments-and-paas-resources-with-azure-resource-manager-templates"></a>Creación de entornos de varias máquinas virtuales y recursos de PaaS con plantillas de Azure Resource Manager

Los entornos de Azure DevTest Labs permiten a los usuarios implementar fácilmente infraestructuras complejas de forma coherente dentro de los confines del laboratorio. Puede usar las [plantillas de Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md) para crear entornos con conjuntos de recursos en DevTest Labs. Estos entornos pueden contener cualquiera de los recursos de Azure que pueden crear las plantillas de Resource Manager.

Puede [agregar fácilmente una máquina virtual (VM) a la vez](devtest-lab-add-vm.md) a un laboratorio mediante [Azure Portal](https://portal.azure.com). Sin embargo, los escenarios como las aplicaciones web de niveles múltiples o las granjas de servidores de SharePoint necesitan un mecanismo para crear varias máquinas virtuales en un solo paso. Mediante el uso de plantillas de Azure Resource Manager, puede definir la infraestructura y la configuración de la solución de Azure e implementar repetidamente varias máquinas virtuales en un estado coherente.

Las plantillas de Azure Resource Manager también proporcionan las siguientes ventajas:

- Las plantillas de Azure Resource Manager se cargan directamente desde el repositorio de control de código fuente de GitHub o Azure Repos.
- Los usuarios pueden crear un entorno al seleccionar una plantilla de Azure Resource Manager configurada desde Azure Portal como lo hacen con otros tipos de [bases de máquinas virtuales](devtest-lab-comparing-vm-base-image-types.md).
- Puede aprovisionar recursos de PaaS de Azure así como máquinas virtuales de IaaS en un entorno a partir de una plantilla de Azure Resource Manager.
- Puede controlar el costo de los entornos desde el laboratorio además del de las máquinas virtuales individuales creadas por otros tipos de bases de datos. Se crean los recursos de PaaS y se muestran en el seguimiento de costos, aunque el cierre automático de las máquinas virtuales no se aplica a los recursos de PaaS.

Para obtener más información acerca de las ventajas del uso de plantillas de Resource Manager para implementar, actualizar o eliminar muchos recursos de laboratorio en una sola operación, consulte las [Ventajas de usar plantillas de Resource Manager](../azure-resource-manager/management/overview.md#the-benefits-of-using-resource-manager).

> [!NOTE]
> Cuando usa una plantilla de Resource Manager como punto de partida para crear más máquinas virtuales de laboratorio, existen algunas diferencias entre crear una o varias. Para obtener más información, consulte [Uso de una plantilla de Azure Resource Manager en una máquina virtual](devtest-lab-use-resource-manager-template.md).
>

## <a name="use-devtest-labs-public-environments"></a>Uso de los entornos públicos de DevTest Labs
Azure DevTest Labs tiene un [repositorio público de plantillas de Azure Resource Manager](https://github.com/Azure/azure-devtestlab/tree/master/Environments) que puede usar para crear entornos sin tener que conectarse a un origen externo de GitHub. Este repositorio público es similar al repositorio público de artefactos que está disponible en Azure Portal para todos los laboratorios que crea. El repositorio del entorno le permite empezar a trabajar rápidamente con plantillas de entorno previamente creadas que tienen pocos parámetros de entrada. Estas plantillas proporcionan una experiencia de introducción fluida para los recursos de PaaS dentro de los laboratorios.

En el repositorio público, el equipo de DevTest Labs y otros usuarios han creado y compartido plantillas usadas con frecuencia como aplicaciones web de Azure, clúster de Service Fabric y un entorno de granja de servidores de SharePoint para desarrollo. Puede usar estas plantillas directamente o personalizarlas para adaptarlas a sus necesidades. Para más información, consulte [Configuración y uso de entornos públicos en Azure DevTest Labs](devtest-lab-configure-use-public-environments.md). Después de crear sus propias plantillas, puede almacenarlas en este repositorio para compartirlas con otros usuarios o configurar su propio repositorio de Git.

<a name="configure-your-own-template-repositories"></a>
## <a name="create-your-own-template-repositories"></a>Creación de sus propios repositorios de plantilla

Como uno de los procedimientos recomendados para la infraestructura como código y la configuración como código, debe administrar las plantillas de entorno mediante control de código fuente. Azure DevTest Labs sigue este procedimiento y carga todas las plantillas de Azure Resource Manager directamente desde los repositorios de GitHub o Azure Repos. Como resultado, puede utilizar las plantillas de Resource Manager en todo el ciclo de lanzamiento, desde el entorno de prueba hasta el entorno de producción.

Hay varias reglas que debe seguir para organizar las plantillas de Azure Resource Manager en un repositorio:

- El archivo de la plantilla maestra debe denominarse *azuredeploy.json*.

- Si quiere usar valores de parámetro definidos en un archivo de parámetros, este debe llamarse *azuredeploy.parameters.json*.

  Puede usar los parámetros `_artifactsLocation` y `_artifactsLocationSasToken` para construir el valor de identificador URI parametersLink y permitir a DevTest Labs administrar automáticamente las plantillas anidadas. Para obtener más información, consulte [Implementar plantillas anidadas de Azure Resource Manager para probar entornos](deploy-nested-template-environments.md).

- Puede definir metadatos para especificar el nombre para mostrar y la descripción de la plantilla en un archivo denominado *metadata.json*, como se indica a continuación:

  ```json
  {
    "itemDisplayName": "<your template name>",
    "description": "<description of the template>"
  }
  ```

![Archivos de plantilla principal de Azure Resource Manager](./media/devtest-lab-create-environment-from-arm/master-template.png)

## <a name="add-template-repositories-to-the-lab"></a>Adición de repositorios de plantillas al laboratorio

Después de crear y configurar el repositorio, puede agregarlo al laboratorio mediante Azure Portal:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione **Todos los servicios** y, luego, **DevTest Labs** en la lista.
1. En la lista de laboratorios, seleccione el laboratorio que quiera.
1. En el panel **Información general** del laboratorio, seleccione **Configuración y directivas**.

   ![Directivas y configuración](./media/devtest-lab-create-environment-from-arm/configuration-and-policies-menu.png)

1. Desde la lista de opciones de **Configuración y directivas**, seleccione **Repositorios**. Se genera automáticamente un repositorio **Public Artifact Repo** para todos los laboratorios y se conecta al [repositorio público de GitHub de DevTest Labs](https://github.com/Azure/azure-devtestlab).

1. Para agregar el repositorio de plantillas de Resource Manager, seleccione **Agregar**.

   ![Repositorio público](./media/devtest-lab-create-environment-from-arm/public-repo.png)

1. En el panel **Repositorios**, escriba la siguiente información:

   - **Name**: escriba un nombre de repositorio para usarlo en el laboratorio.
   - **URL de clonación de Git**: escriba la URL de clonación HTTPS de Git de GitHub o Azure Repos.
   - **Branch (rama)** (opcional): nombre de la rama para acceder a las definiciones de plantilla de Azure Resource Manager.
   - **Token de acceso personal**: se usa para acceder de forma segura al repositorio.
     - Para obtener el token de Azure Repos, en su perfil, seleccione **Configuración de usuario** > **Seguridad** > **Tokens de acceso personal**.
     - Para obtener el token de GitHub, en su perfil, seleccione **Configuración** > **Configuración de desarrollador** > **Tokens de acceso personal**.
   - **Rutas de acceso de carpeta**: escriba la ruta de acceso de la carpeta relativa a su URI de clonación de Git para las definiciones de artefactos o para las definiciones de plantilla de Azure Resource Manager.

1. Seleccione **Guardar**.

   ![Adición de un nuevo repositorio](./media/devtest-lab-create-environment-from-arm/repo-values.png)

Una vez que agrega una plantilla de Azure Resource Manager al laboratorio, los usuarios del laboratorio pueden crear entornos mediante la plantilla.

## <a name="configure-access-rights-for-lab-users"></a>Configuración de los derechos de acceso para los usuarios del laboratorio

Los usuarios del laboratorio tienen un rol de **Lector** de forma predeterminada, por lo que no pueden cambiar los recursos de un grupo de recursos del entorno. Por ejemplo, no pueden detener ni iniciar sus recursos.

Para dar el rol de **Colaborador** a los usuarios de laboratorio a fin de que puedan editar los recursos en sus entornos, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com), desde el panel de **Información general** del laboratorio, seleccione **Configuración y directivas** y después seleccione **Configuración del laboratorio**.

1. En el panel **Configuración de laboratorio**, seleccione **Colaborador** y después **Guardar** para conceder permisos de escritura a los usuarios del laboratorio.

   ![Configuración de los derechos de acceso de los usuarios del laboratorio](./media/devtest-lab-create-environment-from-arm/config-access-rights.png)

La siguiente sección le guía a través de la creación de entornos a partir de una plantilla de Azure Resource Manager.

## <a name="create-environments-from-templates-in-the-azure-portal"></a>Creación de entornos a partir de plantillas en Azure Portal

Una vez que agrega una plantilla de Azure Resource Manager al laboratorio, los usuarios del laboratorio pueden crear entornos en Azure Portal mediante los pasos siguientes:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Seleccione **Todos los servicios** y, luego, **DevTest Labs** en la lista.

1. En la lista de laboratorios, seleccione el laboratorio que quiera.

1. En el panel del laboratorio, seleccione **Agregar**.

1. En el panel **Elegir una base** se muestran las imágenes base que se pueden usar con las plantillas de Azure Resource Manager que se muestran primero. Seleccione la plantilla de Azure Resource Manager que quiera.

   ![Elegir base de datos](./media/devtest-lab-create-environment-from-arm/choose-a-base.png)

1. En el panel **Agregar**, escriba el **Nombre del entorno** que se va a mostrar a los usuarios del entorno.

   La plantilla de Azure Resource Manager define el resto de los campos de entrada. Si el archivo de plantilla *azuredeploy.parameter.json* define valores predeterminados, los campos de entrada muestran esos valores.

   Para los parámetros de tipo *cadena segura* puede usar secretos de su instancia de Azure Key Vault. Para obtener información sobre cómo almacenar secretos en un almacén de claves y utilizarlos al crear recursos de laboratorio, consulte [Almacenamiento de secretos en Azure Key Vault](devtest-lab-store-secrets-in-key-vault.md).  

   ![Agregar panel](./media/devtest-lab-create-environment-from-arm/add.png)

   > [!NOTE]
   > Los siguientes valores de parámetro no aparecen en los campos de entrada, incluso si la plantilla los especifica. En su lugar, el formulario muestra campos de entrada en blanco donde los usuarios del laboratorio deben especificar un valor al crear el entorno.
   >
   > - GEN-UNIQUE
   > - GEN-UNIQUE-[N]
   > - GEN-SSH-PUB-KEY
   > - GEN-PASSWORD

1. Seleccione **Add** (Agregar) para crear el entorno.

   El entorno empieza el aprovisionamiento inmediatamente y el estado se muestra en la lista **Mis máquinas virtuales**. El laboratorio crea automáticamente un grupo de recursos nuevo para aprovisionar todos los recursos definidos en la plantilla de Azure Resource Manager.

1. Una vez creado el entorno, selecciónelo en la lista **Mis máquinas virtuales** para abrir el panel del grupo de recursos y examinar todos los recursos que aprovisionó el entorno.

   ![Recursos de entorno](./media/devtest-lab-create-environment-from-arm/all-environment-resources.png)

   También puede expandir el entorno para ver solo la lista de máquinas virtuales aprovisionadas por el entorno.

   ![Lista My virtual machines (Mis máquinas virtuales)](./media/devtest-lab-create-environment-from-arm/my-vm-list.png)

1. Seleccione cualquiera de los entornos para ver las acciones disponibles; por ejemplo, es posible aplicar artefactos, adjuntar discos de datos, cambiar el tiempo de apagado automático y mucho más.

   ![Acciones de entorno](./media/devtest-lab-create-environment-from-arm/environment-actions.png)

<a name="automate-deployment-of-environments"></a>
## <a name="automate-environment-creation-with-powershell"></a>Automatización de la creación de entornos con PowerShell

Resulta posible usar Azure Portal para agregar un solo entorno a un laboratorio, pero cuando un escenario de desarrollo o pruebas debe crear varios entornos, la implementación automatizada es una mejor experiencia.

Antes de continuar, asegúrese de tener una plantilla de Azure Resource Manager que defina los recursos que se van a crear. [Agregue y configure la plantilla en un repositorio de Git](#configure-your-own-template-repositories) y [agregue el repositorio al laboratorio](#add-template-repositories-to-the-lab).

En el siguiente script de ejemplo se crea un entorno en el laboratorio. Los comentarios le ayudan a comprender mejor el script.

1. Guarde el siguiente script de PowerShell de ejemplo en el disco duro con el nombre *deployenv.ps1*.

   [!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

   ```powershell
   #Requires -Module Az.Resources

   [CmdletBinding()]

   param (
   # ID of the Azure Subscription for the lab
   [string] [Parameter(Mandatory=$true)] $SubscriptionId,

   # Name of the existing lab in which to create the environment
   [string] [Parameter(Mandatory=$true)] $LabName,

   # Name of the connected repository in the lab
   [string] [Parameter(Mandatory=$true)] $RepositoryName,

   # Name of the template (folder name in the Git repository)
   [string] [Parameter(Mandatory=$true)] $TemplateName,

   # Name of the environment to be created in the lab
   [string] [Parameter(Mandatory=$true)] $EnvironmentName,

   # The parameters to be passed to the template. Each parameter is prefixed with "-param_".
   # For example, if the template has a parameter named "TestVMName" with a value of "MyVMName",
   # the string in $Params will have the form: -param_TestVMName MyVMName.
   # This convention allows the script to dynamically handle different templates.
   [Parameter(ValueFromRemainingArguments=$true)]
       $Params
   )

   # Sign in to Azure.
   # Comment out the following statement to completely automate the environment creation.
   Connect-AzAccount

   # Select the subscription that has the lab.  
   Set-AzContext -SubscriptionId $SubscriptionId | Out-Null

   # Get information about the user, specifically the user ID, which is used later in the script.  
   $UserId = $((Get-AzADUser -UserPrincipalName ((Get-AzContext).Account).Id).Id)

   # Get information about the lab, such as lab location.
   $lab = Get-AzResource -ResourceType "Microsoft.DevTestLab/labs" -Name $LabName
   if ($lab -eq $null) { throw "Unable to find lab $LabName in subscription $SubscriptionId." }

   # Get information about the repository in the lab.
   $repository = Get-AzResource -ResourceGroupName $lab.ResourceGroupName `
       -ResourceType 'Microsoft.DevTestLab/labs/artifactsources' `
       -ResourceName $LabName `
       -ApiVersion 2016-05-15 `
       | Where-Object { $RepositoryName -in ($_.Name, $_.Properties.displayName) } `
       | Select-Object -First 1
   if ($repository -eq $null) { throw "Unable to find repository $RepositoryName in lab $LabName." }

   # Get information about the Resource Manager template base for the environment.
   $template = Get-AzResource -ResourceGroupName $lab.ResourceGroupName `
       -ResourceType "Microsoft.DevTestLab/labs/artifactSources/armTemplates" `
       -ResourceName "$LabName/$($repository.Name)" `
       -ApiVersion 2016-05-15 `
       | Where-Object { $TemplateName -in ($_.Name, $_.Properties.displayName) } `
       | Select-Object -First 1
   if ($template -eq $null) { throw "Unable to find template $TemplateName in lab $LabName." }

   # Build the template parameters with parameter name and values.  
   $parameters = Get-Member -InputObject $template.Properties.contents.parameters -MemberType NoteProperty | Select-Object -ExpandProperty Name
   $templateParameters = @()

   # Extract the custom parameters from $Params and format as name/value pairs.
   $Params | ForEach-Object {
       if ($_ -match '^-param_(.*)' -and $Matches[1] -in $parameters) {
           $name = $Matches[1]                
       } elseif ( $name ) {
           $templateParameters += @{ "name" = "$name"; "value" = "$_" }
           $name = $null #reset name variable
       }
   }

   # Once name/value pairs are isolated, create an object to hold the necessary template properties.
   $templateProperties = @{ "deploymentProperties" = @{ "armTemplateId" = "$($template.ResourceId)"; "parameters" = $templateParameters }; }

   # Now, create or deploy the environment in the lab by using the New-AzResource command.
   New-AzResource -Location $Lab.Location `
       -ResourceGroupName $lab.ResourceGroupName `
       -Properties $templateProperties `
       -ResourceType 'Microsoft.DevTestLab/labs/users/environments' `
       -ResourceName "$LabName/$UserId/$EnvironmentName" `
       -ApiVersion '2016-05-15' -Force

   Write-Output "Environment $EnvironmentName completed."
   ```

1. Ejecute el script como se indica a continuación, con los valores específicos de SubscriptionId, LabName, ResourceGroupName, RepositoryName, TemplateName (carpeta en el repositorio de Git) y EnvironmentName.

   ```powershell
   ./deployenv.ps1 -SubscriptionId "000000000-0000-0000-0000-0000000000000" -LabName "mydevtestlab" -ResourceGroupName "mydevtestlabRG000000" -RepositoryName "myRepository" -TemplateName "My Environment template name" -EnvironmentName "myGroupEnv"
   ```

También puede usar la CLI de Azure para implementar recursos con plantillas de Resource Manager. Para más información, consulte [Implementación de recursos con plantillas de Resource Manager y la CLI de Azure](../azure-resource-manager/templates/deploy-cli.md).

> [!NOTE]
> Solo un usuario con permisos de propietario de laboratorio puede crear máquinas virtuales desde una plantilla de Resource Manager con Azure PowerShell. Si desea automatizar la creación de máquinas virtuales mediante una plantilla de Resource Manager y solo tiene permisos de usuario, puede usar el comando de la CLI [az lab vm create](/cli/azure/lab/vm#az-lab-vm-create).

## <a name="resource-manager-template-limitations-in-devtest-labs"></a>Limitaciones de las plantillas de Resource Manager en DevTest Labs

Tenga en cuenta estas limitaciones cuando use plantillas de Resource Manager en DevTest Labs:

- No se pueden crear fórmulas ni imágenes personalizadas desde las máquinas virtuales de laboratorio que se crearon a partir de una plantilla de Resource Manager.

- La mayoría de las directivas no se evalúan al implementar plantillas de Resource Manager.

Por ejemplo, es posible que tenga una directiva de laboratorio que establece que un usuario puede crear solo cinco máquinas virtuales. Sin embargo, un usuario puede implementar una plantilla de Resource Manager que cree docenas de máquinas virtuales. Entre las directivas que no se evalúan se incluyen:

  - Número de máquinas virtuales por usuario

  - Número de máquinas virtuales premium por usuario de laboratorio

  - Número de discos premium por usuario de laboratorio

## <a name="next-steps"></a>Pasos siguientes
- Una vez creada la máquina virtual, puede conectarse a esta al seleccionar **Conectar** en el panel de administración de la máquina virtual.
- Consulte y administre recursos en un entorno al seleccionarlo de la lista **My virtual machines** (Mis máquinas virtuales) del laboratorio.
- Explore las [plantillas de Azure Resource Manager de la galería de plantillas de inicio rápido de Azure](https://github.com/Azure/azure-quickstart-templates).

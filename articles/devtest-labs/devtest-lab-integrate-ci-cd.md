---
title: Integración de Azure DevTest Labs en Azure Pipelines
description: Obtenga información sobre cómo integrar Azure DevTest Labs en la canalización de integración y entrega continuas de Azure Pipelines
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 96f99d41d0a7ea07bf3854292f9c3bd6245414b3
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87288933"
---
# <a name="integrate-azure-devtest-labs-into-your-azure-pipelines-cicd-pipeline"></a>Integración de Azure DevTest Labs en la canalización de integración y entrega continuas de Azure Pipelines

Puede usar la extensión *Tareas de Azure DevTest Labs* para integrar sus canalizaciones de compilación y versión de integración continua y entrega continua (CI/CD) de Azure Pipelines con Azure DevTest Labs. La extensión instala varias tareas, entre las que se incluyen las siguientes: 

- Creación de una máquina virtual (VM)
- Crear una imagen personalizada a partir de una máquina virtual
- Eliminación de una máquina virtual 

Estas tareas facilitan, por ejemplo, la implementación rápida de una máquina virtual de *imagen maestra* para una tarea de prueba específica y, luego, la elimina cuando la prueba finaliza.

En este artículo se muestra cómo usar Tareas de Azure DevTest Labs para crear e implementar una máquina virtual, crear una imagen personalizada y, a continuación, eliminar la máquina virtual, todo ello como una única canalización de versión. Lo habitual sería realizar las tareas de forma individual en su propia canalización personalizada de compilación, prueba e implementación.

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="prerequisites"></a>Prerrequisitos

- Regístrese o inicie sesión en su organización de [Azure DevOps](https://dev.azure.com) y [cree un proyecto](/vsts/organizations/projects/create-project) en la organización. 
  
- Instale la extensión Tareas de Azure DevTest Labs desde Visual Studio Marketplace.
  
  1. Vaya a [Azure DevTest Labs Tasks](https://marketplace.visualstudio.com/items?itemName=ms-azuredevtestlabs.tasks).
  1. Seleccione **Obtener gratis**.
  1. Seleccione su organización de Azure DevOps en la lista desplegable y seleccione **Instalar**. 
  
## <a name="create-the-template-to-build-an-azure-vm"></a>Creación de la plantilla para compilar una máquina virtual de Azure 

En esta sección se describe cómo crear la plantilla de Azure Resource Manager que se usa para crear una máquina virtual de Azure a petición.

1. Para crear una plantilla de Resource Manager en su suscripción, siga el procedimiento que se describe en [Uso de una plantilla de Resource Manager](devtest-lab-use-resource-manager-template.md).
   
1. Antes de generar la plantilla de Resource Manager, agregue el [artefacto WinRM](https://github.com/Azure/azure-devtestlab/tree/master/Artifacts/windows-winrm) como parte de la creación de la máquina virtual. Las tareas de implementación, como *Azure File Copy* y *PowerShell en las máquinas de destino*, necesitan acceso a WinRM. El artefacto WinRM requiere un nombre de host como parámetro, que debe ser el nombre de dominio completo (FQDN) de la máquina virtual. 
   
   > [!NOTE]
   > Cuando se usa WinRM con una dirección IP compartida, debe agregar una regla NAT para asignar un puerto externo al puerto de WinRM. La regla NAT no es necesaria si crea la máquina virtual con una dirección IP pública.
   
   
1. Guarde la plantilla en el equipo como un archivo denominado *CreateVMTemplate.json*.
   
1. Proteja la plantilla en el sistema de control de código fuente.

## <a name="create-a-script-to-get-vm-properties"></a>Creación de un script para obtener las propiedades de la máquina virtual

Al ejecutar los pasos de la tarea, como *Azure File Copy* o *PowerShell en las máquinas de destino* en la canalización de versión, el siguiente script recopila los valores que necesita para implementar una aplicación en una máquina virtual. Estas tareas se usarían normalmente para implementar la aplicación en una máquina virtual de Azure. Las tareas requieren valores como el nombre del grupo de recursos de VM, la dirección IP y el nombre de dominio completo (FQDN).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Para crear el archivo de script:

1. Abra un editor de texto y pegue el código siguiente.
   
   ```powershell
   Param( [string] $labVmId)

   $labVmComputeId = (Get-AzResource -Id $labVmId).Properties.ComputeId

   # Get lab VM resource group name
   $labVmRgName = (Get-AzResource -Id $labVmComputeId).ResourceGroupName

   # Get the lab VM Name
   $labVmName = (Get-AzResource -Id $labVmId).Name

   # Get lab VM public IP address
   $labVMIpAddress = (Get-AzPublicIpAddress -ResourceGroupName $labVmRgName
                   -Name $labVmName).IpAddress

   # Get lab VM FQDN
   $labVMFqdn = (Get-AzPublicIpAddress -ResourceGroupName $labVmRgName
              -Name $labVmName).DnsSettings.Fqdn

   # Set a variable labVmRgName to store the lab VM resource group name
   Write-Host "##vso[task.setvariable variable=labVmRgName;]$labVmRgName"

   # Set a variable labVMIpAddress to store the lab VM Ip address
   Write-Host "##vso[task.setvariable variable=labVMIpAddress;]$labVMIpAddress"

   # Set a variable labVMFqdn to store the lab VM FQDN name
   Write-Host "##vso[task.setvariable variable=labVMFqdn;]$labVMFqdn"
   ```

1. Guarde el archivo con un nombre como *GetLabVMParams.ps1* y protéjalo en su sistema de control de código fuente. 

## <a name="create-a-release-pipeline-in-azure-pipelines"></a>Crear una canalización de versión en Azure Pipelines

Para crear una nueva canalización de versión:

1. En la página del proyecto de Azure DevOps, seleccione **Canalizaciones** > **Versiones** en el panel de navegación de la izquierda.
1. Seleccione **Nueva canalización**.
1. En **Seleccionar una plantilla**, desplácese hacia abajo y seleccione **Fase vacía** y, a continuación, seleccione **Aplicar**.

### <a name="add-and-set-variables"></a>Agregar y establecer variables

Las tareas de la canalización usan los valores que asignó a la máquina virtual cuando creó la plantilla de Resource Manager en Azure Portal. 

Para agregar variables para los valores: 

1. En la página de la canalización, seleccione la pestaña **Variables**.
   
1. Para cada variable, seleccione **Agregar** y escriba el nombre y el valor:
   
   |Nombre|Value|
   |---|---|
   |*vmName*|Nombre de la máquina virtual que asignó en la plantilla de Resource Manager.|
   |*userName*|Nombre de usuario para acceder a la máquina virtual.|
   |*password*|Contraseña para el nombre de usuario. Seleccione el icono de candado para ocultar y proteger la contraseña.

### <a name="create-a-devtest-labs-vm"></a>Creación de una máquina virtual de DevTest Labs

El siguiente paso consiste en crear la máquina virtual de imagen maestra que se usará para futuras implementaciones. La máquina virtual se creará en la instancia de Azure DevTest Labs mediante la tarea de *creación de una máquina virtual de Azure DevTest Labs*.

1. En la pestaña **Canalización** de la canalización de versión, seleccione el texto de hipervínculo de **Fase 1** para **Ver tareas de la fase** y, a continuación, seleccione el signo más **+** junto a **Trabajo de agente**. 
   
1. En **Agregar tareas**, seleccione **Azure DevTest Labs Create VM** (Crear VM de Azure DevTest Labs) y seleccione **Agregar**. 
   
1. Seleccione **Create Azure DevTest Labs VM** (Crear VM de Azure DevTest Labs) en el panel izquierdo. 

1. En el panel de la derecha, rellene el formulario como se indica a continuación:
   
   |Campo|Value|
   |---|---|
   |**Suscripción a Azure Resource Manager**|Seleccione una conexión de servicio o suscripción de **Conexiones disponibles del servicio de Azure** o **Suscripciones de Azure disponibles** en el menú desplegable y seleccione **Autorizar** si es necesario.<br /><br />**Nota:** Para más información sobre cómo crear una conexión de permisos más restringida a su suscripción de Azure, consulte [Punto de conexión de servicio de Azure Resource Manager](/azure/devops/pipelines/library/service-endpoints#sep-azure-resource-manager).|
   |**Nombre de laboratorio**|Seleccione el nombre de un laboratorio existente en el que se creará la máquina virtual de laboratorio.|
   |**Nombre de plantilla**|Escriba la ruta de acceso completa y el nombre del archivo de plantilla que guardó en el repositorio de código fuente. Puede usar las propiedades integradas para simplificar la ruta de acceso, por ejemplo:<br /><br />`$(System.DefaultWorkingDirectory)/Templates/CreateVMTemplate.json`|
   |**Parámetros de plantilla**|Escriba los parámetros de las variables que definió anteriormente:<br /><br />`-newVMName '$(vmName)' -userName '$(userName)' -password (ConvertTo-SecureString -String '$(password)' -AsPlainText -Force)`|
   |**Variables de salida** > **Identificador de máquina virtual de laboratorio**|Escriba la variable para el identificador de máquina virtual de laboratorio creado. Si usa el valor predeterminado **labVMId**, puede hacer referencia a la variable en tareas posteriores como *$(labVMId)* .<br /><br />Puede crear un nombre distinto del predeterminado, pero recuerde usar el nombre correcto en las tareas posteriores. El identificador de la máquina virtual de laboratorio se puede escribir en el formato siguiente:<br /><br />`/subscriptions/{subscription Id}/resourceGroups/{resource group Name}/providers/Microsoft.DevTestLab/labs/{lab name}/virtualMachines/{vmName}`|

### <a name="collect-the-details-of-the-devtest-labs-vm"></a>Recopilación de los detalles de la máquina virtual de DevTest Labs

Ejecute el script que creó anteriormente para recopilar los detalles de la máquina virtual de DevTest Labs. 

1. En la pestaña **Canalización** de la canalización de versión, seleccione el texto de hipervínculo de **Fase 1** para **Ver tareas de la fase** y, a continuación, seleccione el signo más **+** junto a **Trabajo de agente**. 
   
1. En **Agregar tareas**, seleccione **Azure PowerShell** y, después, seleccione **Agregar**. 
   
1. Seleccione **Script de Azure PowerShell: FilePath** en el panel izquierdo. 
   
1. En el panel de la derecha, rellene el formulario como se indica a continuación:
   
   |Campo|Value|
   |---|---|
   |**Tipo de conexión de Azure**|Seleccione **Azure Resource Manager**.|
   |**Suscripción de Azure**|Seleccione su suscripción o conexión de servicio.| 
   |**Tipo de script**|Seleccione **Ruta de acceso del archivo de script**.|
   |**Ruta de acceso del script**|Escriba la ruta de acceso completa y el nombre del script de PowerShell que guardó en el repositorio de código fuente. Puede usar las propiedades integradas para simplificar la ruta de acceso, por ejemplo:<br /><br />`$(System.DefaultWorkingDirectory/Scripts/GetLabVMParams.ps1`|
   |**Argumentos de script**|Escriba el nombre de la variable *labVmId* rellenada en la tarea anterior, por ejemplo:<br /><br />`-labVmId '$(labVMId)'`|

El script recopila los valores necesarios y los almacena en variables de entorno dentro de la canalización de versión, para que así pueda consultarlos fácilmente en pasos posteriores.

### <a name="create-a-vm-image-from-the-devtest-labs-vm"></a>Creación de una imagen de máquina virtual desde la máquina virtual de DevTest Labs

La siguiente tarea consiste en crear una imagen de la máquina virtual recién implementada en la instancia de Azure DevTest Labs. Luego, puede usar la imagen para crear copias de la máquina virtual a petición siempre que quiera ejecutar una tarea de desarrollo o ejecutar algunas pruebas. 

1. En la pestaña **Canalización** de la canalización de versión, seleccione el texto de hipervínculo de **Fase 1** para **Ver tareas de la fase** y, a continuación, seleccione el signo más **+** junto a **Trabajo de agente**. 
   
1. En **Agregar tareas**, seleccione **Azure DevTest Labs Create Custom Image** (Crear imagen personalizada de Azure DevTest Labs) y seleccione **Agregar**. 
   
1. Configure la tarea de la siguiente manera:
   
   |Campo|Value|
   |---|---|
   |**Suscripción a Azure Resource Manager**|Seleccione su suscripción o conexión de servicio.|
   |**Nombre de laboratorio**|Seleccione el nombre de un laboratorio existente en el que se creará la imagen.|
   |**Nombre de imagen personalizada**|Escriba un nombre para la imagen personalizada.|
   |**Descripción** (opcional)|Escriba una descripción para facilitar la selección de la imagen correcta más adelante.|
   |**Máquina virtual de laboratorio de origen** > **Identificador de máquina virtual de laboratorio de origen**|Si cambió el nombre predeterminado de la variable LabVMId, escríbalo aquí. El valor predeterminado es **$(labVMId)** .|
   |**Variables de salida** > **Identificador de imagen personalizada**|Puede modificar el nombre predeterminado de la variable si es necesario.|
   
### <a name="deploy-your-app-to-the-devtest-labs-vm-optional"></a>Implementación de la aplicación en la máquina de DevTest Labs (opcional)

Puede agregar tareas para implementar la aplicación en la nueva máquina virtual de DevTest Labs. Las tareas que se usan habitualmente para implementar la aplicación son *Azure File Copy* y *PowerShell en las máquinas de destino*.

La información de la máquina virtual que se necesita para los parámetros de estas tareas se almacena en tres variables de configuración denominadas **labVmRgName**, **labVMIpAddress** y **labVMFqdn** dentro de la canalización de versión. Si solo desea experimentar con la creación de una máquina virtual de DevTest Labs y una imagen personalizada, sin implementar una aplicación en ella, puede omitir este paso.

### <a name="delete-the-vm"></a>Eliminación de la máquina virtual

La tarea final consiste en eliminar la máquina virtual que se implementó en la instancia de Azure DevTest Labs. Lo habitual es eliminar la máquina virtual después de ejecutar las tareas de desarrollo o de ejecutar las pruebas que necesita en la máquina virtual implementada. 

1. En la pestaña **Canalización** de la canalización de versión, seleccione el texto de hipervínculo de **Fase 1** para **Ver tareas de la fase** y, a continuación, seleccione el signo más **+** junto a **Trabajo de agente**. 
   
1. En **Agregar tareas**, seleccione **Azure DevTest Labs Delete VM** (Eliminar VM de Azure DevTest Labs) y seleccione **Agregar**. 
   
1. Configure la tarea de la siguiente manera:
   
   - En **Azure RM Subscription** (Suscripción de Azure Resource Manager), seleccione su suscripción o conexión de servicio. 
   - En **Lab VM ID** (Id. de máquina virtual de laboratorio), si cambió el nombre predeterminado de la variable LabVMId, escríbalo aquí. El valor predeterminado es **$(labVMId)** .
   
### <a name="save-the-release-pipeline"></a>Guardar la canalización de versión

Para guardar la nueva canalización de versión:

1. Seleccione el nombre **Nueva canalización de versión**en la página de canalización de versión y escriba un nuevo nombre para la canalización. 
   
1. Seleccione el icono **Guardar** en la esquina superior derecha. 

## <a name="create-and-run-a-release"></a>Creación y ejecución de una versión

Para crear y ejecutar una versión con la nueva canalización:

1. En la esquina superior derecha de la página de canalización de versión, seleccione **Crear versión**. 
   
1. En **Artefactos**, seleccione la compilación más reciente y, a continuación, seleccione **Crear**.
   
1. En cada fase de la versión, actualice la vista de la instancia de DevTest Labs en Azure Portal para ver la creación de la máquina virtual, la creación de la imagen y la eliminación de la máquina virtual.

Ahora puede usar la imagen personalizada para crear máquinas virtuales cuando sea necesario.

## <a name="next-steps"></a>Pasos siguientes
- Aprenda a [crear entornos de varias máquinas virtuales con plantillas de Resource Manager](devtest-lab-create-environment-from-arm.md).
- Explore más plantillas de inicio rápido de Resource Manager para la automatización de DevTest Labs desde el [repositorio público de GitHub de DevTest Labs](https://github.com/Azure/azure-quickstart-templates).
- Si es necesario, consulte la página de [solución de problemas de Azure DevOps](/azure/devops/pipelines/troubleshooting).
 

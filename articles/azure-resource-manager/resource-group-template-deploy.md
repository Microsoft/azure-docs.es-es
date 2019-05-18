---
title: Implementación de recursos con la plantilla y PowerShell | Microsoft Docs
description: Use Azure Resource Manager y Azure PowerShell para implementar recursos en Azure. Los recursos se definen en una plantilla de Resource Manager.
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: conceptual
ms.date: 05/14/2019
ms.author: tomfitz
ms.openlocfilehash: 5203519b1553de54d4e3cd1fafe6fb3d1c18ebd6
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65779956"
---
# <a name="deploy-resources-with-resource-manager-templates-and-azure-powershell"></a>Implementación de recursos con las plantillas de Resource Manager y Azure PowerShell

Aprenda a utilizar Azure PowerShell con plantillas de Resource Manager para implementar recursos en Azure. Para más información sobre los conceptos asociados a la implementación y administración de las soluciones de Azure, consulte [Introducción a Azure Resource Manager](resource-group-overview.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="deployment-scope"></a>Ámbito de implementación

Puede tener como destino la implementación de una suscripción de Azure o un grupo de recursos dentro de una suscripción. En la mayoría de los casos, serán las objetivo implementación en un grupo de recursos. Use las implementaciones de suscripción para aplicar directivas y las asignaciones de roles a través de la suscripción. También se pueden usar implementaciones de la suscripción para crear un grupo de recursos e implementar recursos en ella. Según el ámbito de la implementación, use comandos diferentes.

Para implementar en un **grupo de recursos**, utilice [New AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment):

```azurepowershell
New-AzResourceGroupDeployment -ResourceGroupName <resource-group-name> -TemplateFile <path-to-template>
```

Para implementar en un **suscripción**, utilice [New AzDeployment](/powershell/module/az.resources/new-azdeployment):

```azurepowershell
New-AzDeployment -Location <location> -TemplateFile <path-to-template>
```

Los ejemplos en este artículo usan las implementaciones de grupo de recursos. Para obtener más información acerca de las implementaciones de suscripción, consulte [crear grupos de recursos y recursos en el nivel de suscripción](deploy-to-subscription.md).

## <a name="prerequisites"></a>Requisitos previos

Necesita una plantilla para implementar. Si aún no tiene uno, descargue y guarde un [plantilla de ejemplo](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json) desde el repositorio de plantillas de inicio rápido de Azure. El nombre de archivo local que se utiliza en este artículo es **c:\MyTemplates\azuredeploy.json**.

A menos que use Azure Cloud shell para implementar plantillas, es preciso instalar Azure PowerShell y conéctese a Azure:

- **Instale los cmdlets de Azure PowerShell en el equipo local.** Para más información, consulte el artículo de [introducción a Azure PowerShell](/powershell/azure/get-started-azureps).
- **Conéctese a Azure utilizando [Connect-AZAccount](/powershell/module/az.accounts/connect-azaccount)**. Si tiene varias suscripciones de Azure, es posible que también tenga que ejecutar [Set-AzContext](/powershell/module/Az.Accounts/Set-AzContext). Para más información, consulte [Use multiple Azure subscriptions](/powershell/azure/manage-subscriptions-azureps) (Uso de varias suscripciones de Azure).

## <a name="deploy-local-template"></a>Implementar una plantilla local

El ejemplo siguiente crea un grupo de recursos e implementa una plantilla desde el equipo local. El nombre del grupo de recursos solo puede incluir caracteres alfanuméricos, puntos, guiones bajos, guiones y paréntesis. Puede tener hasta 90 caracteres. No puede terminar en punto.

```azurepowershell
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"

New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
  -TemplateFile c:\MyTemplates\azuredeploy.json
```

La implementación puede demorar unos minutos en completarse.

## <a name="deploy-remote-template"></a>Implementar plantilla remoto

En lugar de almacenar las plantillas de Resource Manager en el equipo local, quizás prefiera almacenarlas en una ubicación externa. Puede almacenar plantillas en un repositorio de control de código fuente (por ejemplo, GitHub). O bien, puede almacenarlas en una cuenta de Azure Storage para el acceso compartido en su organización.

Para implementar una plantilla externa, use el parámetro **TemplateUri**. Use el identificador URI en el ejemplo para implementar la plantilla de ejemplo de GitHub.

```azurepowershell
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"

New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json
```

En el ejemplo anterior, se requiere un identificador URI accesible públicamente para la plantilla, que funciona con la mayoría de los escenarios porque la plantilla no debe incluir datos confidenciales. Si tiene que especificar datos confidenciales (por ejemplo, una contraseña de administrador), pase ese valor como un parámetro seguro. Sin embargo, si no desea que la plantilla sea accesible públicamente, puede almacenarla en un contenedor de almacenamiento privado para protegerla. Para más información sobre la implementación de una plantilla que requiere un token de Firma de acceso compartido (SAS), consulte [Implementación de una plantilla privada con el token de SAS](resource-manager-powershell-sas-token.md). Para realizar un tutorial, consulte [Tutorial: Integración de Azure Key Vault en Resource Manager Template Deployment](./resource-manager-tutorial-use-key-vault.md).

## <a name="deploy-from-azure-cloud-shell"></a>Implementar desde Azure Cloud shell

Puede usar [Azure Cloud Shell](https://shell.azure.com) para implementar la plantilla. Para implementar una plantilla externa, use el identificador URI de la plantilla. Para implementar una plantilla local, primero debe cargar la plantilla en la cuenta de almacenamiento de Cloud Shell. Para cargar archivos en el shell, seleccione el icono de menú **Cargar/Descargar archivos** en la ventana del shell.

Para abrir Cloud Shell, vaya a [https://shell.azure.com](https://shell.azure.com) o seleccione **Try-it** en la siguiente sección de código:

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"

New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json
```

Para pegar el código en el shell, haga clic dentro del shell y seleccione **Pegar**.

## <a name="redeploy-when-deployment-fails"></a>Nueva implementación cuando se produce un error en la implementación

Esta característica también es conocido como *reversión en caso de error*. Cuando se produce un error en una implementación, puede ejecutar automáticamente desde el historial de implementación una implementación anterior que sea correcta. Para especificar la nueva implementación, utilice el parámetro `-RollbackToLastDeployment` o `-RollBackDeploymentName` en el comando de implementación. Esta funcionalidad es útil si tiene un estado correcto conocido para la implementación de infraestructura y desea volver a este estado. Hay una serie de advertencias y restricciones:

- La nueva implementación se ejecuta exactamente como se ha ejecutado anteriormente con los mismos parámetros. No se puede cambiar los parámetros.
- La implementación anterior se ejecuta con la [modo completo](./deployment-modes.md#complete-mode). Se eliminan todos los recursos que no se incluyen en la implementación anterior y se establecen las configuraciones de recursos a su estado anterior. Asegúrese de que comprende perfectamente el [modos de implementación](./deployment-modes.md).
- La reimplementación solo afecta a los recursos, no se ven afectados los cambios de datos.
- Esta característica solo se admite en las implementaciones de grupo de recursos, no implementaciones de nivel de suscripción. Para obtener más información acerca de la implementación de nivel de suscripción, consulte [crear grupos de recursos y recursos en el nivel de suscripción](./deploy-to-subscription.md).

Para usar esta opción, las implementaciones deben tener nombres únicos para que se puedan identificar en el historial. Si no tienen nombres únicos, la implementación con error en cuestión podría sobrescribir la implementación anteriormente correcta en el historial. Solo se puede usar esta opción con las implementaciones de nivel de raíz. Las implementaciones de una plantilla anidada no están disponibles para volver a implementarse.

Para volver a implementar la última implementación correcta, agregue el parámetro `-RollbackToLastDeployment` como una marca.

```azurepowershell-interactive
New-AzResourceGroupDeployment -Name ExampleDeployment02 `
  -ResourceGroupName $resourceGroupName `
  -TemplateFile c:\MyTemplates\azuredeploy.json `
  -RollbackToLastDeployment
```

Para volver a implementar una implementación específica, use el parámetro `-RollBackDeploymentName` y proporcione el nombre de la implementación.

```azurepowershell-interactive
New-AzResourceGroupDeployment -Name ExampleDeployment02 `
  -ResourceGroupName $resourceGroupName `
  -TemplateFile c:\MyTemplates\azuredeploy.json `
  -RollBackDeploymentName ExampleDeployment01
```

La implementación especificada debe haberse realizado correctamente.

## <a name="pass-parameter-values"></a>Pase de valores de parámetros

Para pasar valores de parámetros, puede usar parámetros en línea o un archivo de parámetros. Los ejemplos anteriores de este artículo muestran parámetros en línea.

### <a name="inline-parameters"></a>Parámetros en línea

Para pasar parámetros en línea, proporcione los nombres de parámetro con el comando `New-AzResourceGroupDeployment`. Por ejemplo, para pasar una cadena y una matriz a una plantilla, use:

```powershell
$arrayParam = "value1", "value2"
New-AzResourceGroupDeployment -ResourceGroupName testgroup `
  -TemplateFile c:\MyTemplates\demotemplate.json `
  -exampleString "inline string" `
  -exampleArray $arrayParam
```

También puede obtener el contenido del archivo y proporcionar ese contenido como un parámetro en línea.

```powershell
$arrayParam = "value1", "value2"
New-AzResourceGroupDeployment -ResourceGroupName testgroup `
  -TemplateFile c:\MyTemplates\demotemplate.json `
  -exampleString $(Get-Content -Path c:\MyTemplates\stringcontent.txt -Raw) `
  -exampleArray $arrayParam
```

Obtener un valor de parámetro de un archivo es útil cuando se necesita proporcionar valores de configuración. Por ejemplo, puede proporcionar [valores de cloud-init para una máquina virtual Linux](../virtual-machines/linux/using-cloud-init.md).

Si necesita pasar una matriz de objetos, crear las tablas hash en PowerShell y agregarlos a una matriz. Pasar esa matriz como un parámetro durante la implementación.

```powershell
$hash1 = @{ Name = "firstSubnet"; AddressPrefix = "10.0.0.0/24"}
$hash2 = @{ Name = "secondSubnet"; AddressPrefix = "10.0.1.0/24"}
$subnetArray = $hash1, $hash2
New-AzResourceGroupDeployment -ResourceGroupName testgroup `
  -TemplateFile c:\MyTemplates\demotemplate.json `
  -exampleArray $subnetArray
```


### <a name="parameter-files"></a>Archivos de parámetros

En lugar de pasar parámetros como valores en línea en el script, quizá le resulte más fácil usar un archivo JSON que contiene los valores de parámetro. El archivo de parámetros puede ser un archivo local o un archivo externo con un identificador URI accesible.

El archivo de parámetros debe estar en el siguiente formato:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
     "storageAccountType": {
         "value": "Standard_GRS"
     }
  }
}
```

Tenga en cuenta que la sección de parámetros incluye un nombre de parámetro que coincide con el parámetro definido en la plantilla (storageAccountType). El archivo de parámetros contiene un valor para el parámetro. Este valor se pasa automáticamente a la plantilla durante la implementación. Puede crear varios archivos de parámetros y, después, pasar el archivo de parámetros adecuado para el escenario.

Copie el ejemplo anterior y guárdelo como un archivo denominado `storage.parameters.json`.

Para pasar un archivo de parámetros local, use el parámetro **TemplateParameterFile**:

```powershell
New-AzResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup `
  -TemplateFile c:\MyTemplates\azuredeploy.json `
  -TemplateParameterFile c:\MyTemplates\storage.parameters.json
```

Para pasar un archivo de parámetros externo, use el parámetro **TemplateParameterUri**:

```powershell
New-AzResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json `
  -TemplateParameterUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.parameters.json
```

### <a name="parameter-precedence"></a>Prioridad de parámetros

Puede usar parámetros en línea y un archivo de parámetros local en la misma operación de implementación. Por ejemplo, puede especificar algunos valores en el archivo de parámetros local y agregar otros valores en línea durante la implementación. Si proporciona valores para un parámetro en el archivo de parámetros local y en línea, el valor en línea tiene prioridad.

Sin embargo, cuando se utiliza un archivo de parámetros externo, no se pueden pasar otros valores ya sea en línea o desde un archivo local. Cuando se especifica un archivo de parámetros en el parámetro **TemplateParameterUri**, se omiten todos los parámetros en línea. Proporcione todos los valores de parámetro en el archivo externo. Si la plantilla incluye un valor confidencial que no puede incluir en el archivo de parámetros, agregue ese valor a un almacén de claves o proporcione dinámicamente todos los valores de parámetro en línea.

### <a name="parameter-name-conflicts"></a>Conflictos de nombres de parámetro

Si la plantilla incluye un parámetro con el mismo nombre que uno de los parámetros del comando de PowerShell, PowerShell presenta el parámetro de la plantilla con el postfijo **FromTemplate**. Por ejemplo, un parámetro denominado **ResourceGroupName** de su plantilla entra en conflicto con el parámetro **ResourceGroupName** del cmdlet [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment) nuevo. Se le pedirá que proporcione un valor para **ResourceGroupNameFromTemplate**. Por lo general, debe evitar esta confusión no nombrando los parámetros con el mismo nombre que los parámetros utilizados para operaciones de implementación.

## <a name="test-template-deployments"></a>Prueba de implementaciones de plantillas

Para probar los valores de parámetro y de plantilla sin implementar realmente ningún recurso, use [Test-AzureRmResourceGroupDeployment](/powershell/module/az.resources/test-azresourcegroupdeployment). 

```powershell
Test-AzResourceGroupDeployment -ResourceGroupName ExampleResourceGroup `
  -TemplateFile c:\MyTemplates\azuredeploy.json -storageAccountType Standard_GRS
```

Si no se detectan errores, el comando finaliza sin una respuesta. Si se detecta un error, el comando devuelve un mensaje de error. Por ejemplo, si se pasa un valor incorrecto para la SKU de la cuenta de almacenamiento, se devuelve el error siguiente:

```powershell
Test-AzResourceGroupDeployment -ResourceGroupName testgroup `
  -TemplateFile c:\MyTemplates\azuredeploy.json -storageAccountType badSku

Code    : InvalidTemplate
Message : Deployment template validation failed: 'The provided value 'badSku' for the template parameter 'storageAccountType'
          at line '15' and column '24' is not valid. The parameter value is not part of the allowed value(s):
          'Standard_LRS,Standard_ZRS,Standard_GRS,Standard_RAGRS,Premium_LRS'.'.
Details :
```

Si la plantilla tiene un error de sintaxis, el comando devuelve un error que indica que la plantilla no se pudo analizar. El mensaje indica el número de línea y la posición del error de análisis.

```powershell
Test-AzResourceGroupDeployment : After parsing a value an unexpected character was encountered: 
  ". Path 'variables', line 31, position 3.
```

## <a name="next-steps"></a>Pasos siguientes

- Para el lanzamiento seguro del servicio en más de una región, consulte [Azure Deployment Manager](deployment-manager-overview.md).
- Para especificar cómo controlar los recursos que existen en el grupo de recursos, pero que no están definidos en la plantilla, consulte [Modos de implementación de Azure Resource Manager](deployment-modes.md).
- Para entender cómo definir parámetros en la plantilla, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](resource-group-authoring-templates.md).
- Para más información sobre la implementación de una plantilla que requiere un token de SAS, vea [Implementación de una plantilla privada con el token de SAS](resource-manager-powershell-sas-token.md).

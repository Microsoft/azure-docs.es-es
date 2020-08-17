---
title: Implementación de recursos con una plantilla y la CLI de Azure
description: Use Azure Resource Manager y la CLI de Azure para implementar recursos en Azure. Los recursos se definen en una plantilla de Resource Manager.
ms.topic: conceptual
ms.date: 07/21/2020
ms.openlocfilehash: da865d3b425da6b5969e540a424b513d9a58bd9a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87040808"
---
# <a name="deploy-resources-with-arm-templates-and-azure-cli"></a>Implementación de recursos con plantillas de ARM y la CLI de Azure

En este artículo, se explica el uso de la CLI de Azure con plantillas de Azure Resource Manager (ARM) para implementar los recursos en Azure. Si no está familiarizado con los conceptos de implementación y administración de las soluciones de Azure, vea [Información general sobre plantillas](overview.md).

Los comandos de implementación cambiaron en la CLI de Azure, versión 2.2.0. Los ejemplos de este artículo requieren la CLI de Azure, versión 2.2.0 o posterior.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

Si no tiene instalada la CLI de Azure, puede usar [Cloud Shell](#deploy-template-from-cloud-shell).

## <a name="deployment-scope"></a>Ámbito de la implementación

La implementación puede tener como destino un grupo de recursos, una suscripción, un grupo de administración o un inquilino. En la mayoría de los casos, la implementación tendrá como destino un grupo de recursos. Para aplicar directivas y asignaciones de roles en un ámbito mayor, use implementaciones de suscripción, de grupo de administración o de inquilino. Al implementar en una suscripción, puede crear un grupo de recursos e implementar recursos en él.

Según el ámbito de la implementación, usará comandos diferentes.

* Para implementar en un **grupo de recursos**, use [az deployment group create](/cli/azure/deployment/group?view=azure-cli-latest#az-deployment-group-create):

  ```azurecli-interactive
  az deployment group create --resource-group <resource-group-name> --template-file <path-to-template>
  ```

* Para implementar en una **suscripción**, use [az deployment sub create](/cli/azure/deployment/sub?view=azure-cli-latest#az-deployment-sub-create):

  ```azurecli-interactive
  az deployment sub create --location <location> --template-file <path-to-template>
  ```

  Para más información sobre las implementaciones en el nivel de suscripción, consulte [Creación de grupos de recursos y otros recursos en el nivel de suscripción](deploy-to-subscription.md).

* Para implementar en un **grupo de administración**, use [az deployment mg create](/cli/azure/deployment/mg?view=azure-cli-latest#az-deployment-mg-create):

  ```azurecli-interactive
  az deployment mg create --location <location> --template-file <path-to-template>
  ```

  Para obtener más información sobre las implementaciones de nivel de grupo de administración, consulte [Creación de recursos en el nivel de grupo de administración](deploy-to-management-group.md).

* Para implementar en un **inquilino**, use [az deployment tenant create](/cli/azure/deployment/tenant?view=azure-cli-latest#az-deployment-tenant-create):

  ```azurecli-interactive
  az deployment tenant create --location <location> --template-file <path-to-template>
  ```

  Para obtener más información sobre las implementaciones a nivel de inquilino, consulte [Creación de recursos en el nivel de inquilino](deploy-to-tenant.md).

Los ejemplos de este artículo usan las implementaciones del grupo de recursos.

## <a name="deploy-local-template"></a>Implementar una plantilla local

Al implementar recursos en Azure, siga estos pasos:

1. Inicio de sesión en la cuenta de Azure.
2. Cree un grupo de recursos que actúe como contenedor para los recursos implementados. El nombre del grupo de recursos solo puede incluir caracteres alfanuméricos, puntos, guiones bajos, guiones y paréntesis. Puede tener hasta 90 caracteres. No puede terminar en punto.
3. Implemente en el grupo de recursos la plantilla que define los recursos que se van a crear.

Una plantilla puede incluir parámetros que le permiten personalizar la implementación. Por ejemplo, puede proporcionar valores que están diseñados para un entorno concreto (como desarrollo, prueba y producción). La plantilla de ejemplo define un parámetro para la SKU de la cuenta de almacenamiento.

En el ejemplo siguiente se crea un grupo de recursos y se implementa una plantilla desde la máquina local:

```azurecli-interactive
az group create --name ExampleGroup --location "Central US"
az deployment group create \
  --name ExampleDeployment \
  --resource-group ExampleGroup \
  --template-file storage.json \
  --parameters storageAccountType=Standard_GRS
```

La implementación puede demorar unos minutos en completarse. Cuando termine, verá un mensaje que incluye el resultado:

```output
"provisioningState": "Succeeded",
```

## <a name="deployment-name"></a>Nombre de implementación

En el ejemplo anterior, el nombre que asignó a la implementación fue `ExampleDeployment`. Si no especifica un nombre para la implementación, se utilizará el nombre del archivo de la plantilla. Por ejemplo, si implementa una plantilla llamada `azuredeploy.json` y no especifica ningún nombre para la implementación, el nombre que se asignará será `azuredeploy`.

Cada vez que se ejecuta una implementación, se agrega una entrada al historial de implementación del grupo de recursos con el nombre de la implementación. Si ejecuta otra implementación y le asigna el mismo nombre, la entrada anterior se reemplazará por la implementación actual. Si desea que todas las entradas del historial de implementaciones sean diferentes, asigne un nombre único a cada implementación.

Para crear un nombre único, puede asignar un número aleatorio.

```azurecli-interactive
deploymentName='ExampleDeployment'$RANDOM
```

También puede agregar un valor de fecha.

```azurecli-interactive
deploymentName='ExampleDeployment'$(date +"%d-%b-%Y")
```

Si ejecuta implementaciones simultáneas en el mismo grupo de recursos utilizando el mismo nombre de implementación, solo se completará la última implementación. Aquellas implementaciones que tengan el mismo nombre y no hayan finalizado se sustituirán por la última implementación. Por ejemplo, si ejecuta una implementación llamada `newStorage` que implementa la cuenta de almacenamiento `storage1` y, al mismo tiempo, ejecuta otra implementación llamada `newStorage` que implementa la cuenta de almacenamiento `storage2`, solo se implementará una única cuenta de almacenamiento. La cuenta de almacenamiento resultante será `storage2`.

Sin embargo, si ejecuta una implementación llamada `newStorage` que implementa la cuenta de almacenamiento `storage1` e inmediatamente después ejecuta otra implementación llamada `newStorage` que implementa la cuenta de almacenamiento `storage2`, tendrá dos cuentas de almacenamiento. Una se llamará `storage1` y la otra, `storage2`. Sin embargo, solo tendrá una entrada en el historial de implementaciones.

Si especifica un nombre único para cada implementación, podrá ejecutarlas simultáneamente sin conflictos. Si ejecuta una implementación llamada `newStorage1` que implementa la cuenta de almacenamiento `storage1` y, al mismo tiempo, ejecuta otra implementación llamada `newStorage2` que implementa la cuenta de almacenamiento `storage2`, tendrá dos cuentas de almacenamiento y dos entradas en el historial de implementación.

Para evitar conflictos con las implementaciones simultáneas y garantizar que las entradas del historial de implementaciones son únicas, asigne un nombre diferente a cada implementación.

## <a name="deploy-remote-template"></a>Implementación de una plantilla remota

En lugar de almacenar las plantillas de ARM en el equipo local, quizás prefiera almacenarlas en una ubicación externa. Puede almacenar plantillas en un repositorio de control de código fuente (por ejemplo, GitHub). O bien, puede almacenarlas en una cuenta de Azure Storage para el acceso compartido en su organización.

Para implementar una plantilla externa, use el parámetro **template-uri**. Use el identificador URI en el ejemplo para implementar la plantilla de ejemplo de GitHub.

```azurecli-interactive
az group create --name ExampleGroup --location "Central US"
az deployment group create \
  --name ExampleDeployment \
  --resource-group ExampleGroup \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json" \
  --parameters storageAccountType=Standard_GRS
```

En el ejemplo anterior, se requiere un identificador URI accesible públicamente para la plantilla, que funciona con la mayoría de los escenarios porque la plantilla no debe incluir datos confidenciales. Si tiene que especificar datos confidenciales (por ejemplo, una contraseña de administrador), pase ese valor como un parámetro seguro. Sin embargo, si no desea que la plantilla sea accesible públicamente, puede almacenarla en un contenedor de almacenamiento privado para protegerla. Para más información sobre la implementación de una plantilla que requiere un token de Firma de acceso compartido (SAS), consulte [Implementación de una plantilla privada con el token de SAS](secure-template-with-sas-token.md).

## <a name="preview-changes"></a>Vista previa de los cambios

Antes de implementar la plantilla, puede obtener una vista previa de los cambios que la plantilla realizará en su entorno. Use la [operación Y si](template-deploy-what-if.md) para comprobar que la plantilla realiza los cambios esperados. La operación y si también valida que la plantilla no tenga errores.

[!INCLUDE [resource-manager-cloud-shell-deploy.md](../../../includes/resource-manager-cloud-shell-deploy.md)]

En Cloud Shell, use los comandos siguientes:

```azurecli-interactive
az group create --name examplegroup --location "South Central US"
az deployment group create --resource-group examplegroup \
  --template-uri <copied URL> \
  --parameters storageAccountType=Standard_GRS
```

## <a name="parameters"></a>Parámetros

Para pasar valores de parámetros, puede usar parámetros en línea o un archivo de parámetros.

### <a name="inline-parameters"></a>Parámetros en línea

Para pasar parámetros en línea, proporcione los valores en `parameters`. Por ejemplo, para pasar una cadena y una matriz a una plantilla en un shell de Bash, use:

```azurecli-interactive
az deployment group create \
  --resource-group testgroup \
  --template-file demotemplate.json \
  --parameters exampleString='inline string' exampleArray='("value1", "value2")'
```

Si usa la CLI de Azure con el símbolo del sistema de Windows (CMD) o PowerShell, pase la matriz con el formato `exampleArray="['value1','value2']"`.

También puede obtener el contenido del archivo y proporcionar ese contenido como un parámetro en línea.

```azurecli-interactive
az deployment group create \
  --resource-group testgroup \
  --template-file demotemplate.json \
  --parameters exampleString=@stringContent.txt exampleArray=@arrayContent.json
```

Obtener un valor de parámetro de un archivo es útil cuando se necesita proporcionar valores de configuración. Por ejemplo, puede proporcionar [valores de cloud-init para una máquina virtual Linux](../../virtual-machines/linux/using-cloud-init.md).

El formato de arrayContent.json es:

```json
[
    "value1",
    "value2"
]
```

### <a name="parameter-files"></a>Archivos de parámetros

En lugar de pasar parámetros como valores en línea en el script, quizá le resulte más fácil usar un archivo JSON que contiene los valores de parámetro. El archivo de parámetros debe ser un archivo local. No se admiten los archivos de parámetros externos con la CLI de Azure.

Para más información sobre el archivo de parámetro, consulte [Creación de un archivo de parámetros de Resource Manager](parameter-files.md).

Para pasar un archivo de parámetros local, use `@` para especificar un archivo local denominado storage.parameters.json.

```azurecli-interactive
az deployment group create \
  --name ExampleDeployment \
  --resource-group ExampleGroup \
  --template-file storage.json \
  --parameters @storage.parameters.json
```

## <a name="handle-extended-json-format"></a>Control del formato JSON extendido

Para poder implementar una plantilla con cadenas o comentarios multilínea utilizando la versión 2.3.0 u otra anterior de la CLI de Azure, debe usar el conmutador `--handle-extended-json-format`.  Por ejemplo:

```json
{
  "type": "Microsoft.Compute/virtualMachines",
  "apiVersion": "2018-10-01",
  "name": "[variables('vmName')]", // to customize name, change it in variables
  "location": "[
    parameters('location')
    ]", //defaults to resource group location
  /*
    storage account and network interface
    must be deployed first
  */
  "dependsOn": [
    "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
    "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
  ],
```

## <a name="next-steps"></a>Pasos siguientes

- Para revertir a una implementación correcta cuando se produce un error, consulte [Revertir en caso de error a una implementación correcta](rollback-on-error.md).
- Para especificar cómo controlar los recursos que existen en el grupo de recursos, pero que no están definidos en la plantilla, consulte [Modos de implementación de Azure Resource Manager](deployment-modes.md).
- Para entender cómo definir parámetros en la plantilla, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](template-syntax.md).
- Para obtener sugerencias para resolver los errores de implementación más comunes, consulte [Solución de errores comunes de implementación de Azure con Azure Resource Manager](common-deployment-errors.md).
- Para más información sobre la implementación de una plantilla que requiere un token de SAS, vea [Implementación de una plantilla privada con el token de SAS](secure-template-with-sas-token.md).

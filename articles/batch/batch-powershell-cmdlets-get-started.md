---
title: Introducción a PowerShell
description: Una rápida introducción a los cmdlets de Azure PowerShell que puede usar para administrar recursos de Batch.
ms.topic: how-to
ms.date: 01/15/2019
ms.custom: seodec18
ms.openlocfilehash: 2c80da92c7acad5180c763d259357f5369f225f7
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87092786"
---
# <a name="manage-batch-resources-with-powershell-cmdlets"></a>Administración de recursos de Batch con cmdlets de PowerShell

Con los cmdlets de PowerShell de Azure Batch puede realizar directamente y mediante scripts muchas de las tareas que se llevan a cabo con las API de Batch, Azure Portal y la interfaz de la línea de comandos (CLI) de Azure. Esta es una breve introducción a los cmdlets que se pueden usar para administrar cuentas de Batch y trabajar con recursos de Batch tales como grupos, trabajos y tareas.

Para obtener una lista completa de los cmdlets de Batch y la sintaxis detallada de los cmdlets, consulte la [referencia de los cmdlets de Azure Batch](/powershell/module/az.batch).

En este artículo se usan los cmdlets de la versión 1.0.0 del módulo Az Batch. Se recomienda actualizar los módulos de Azure PowerShell con frecuencia para aprovechar las mejoras y actualizaciones del servicio.

## <a name="prerequisites"></a>Prerrequisitos

* [Instale y configure el módulo de Azure PowerShell](/powershell/azure/). Para instalar un módulo específico de Azure Batch, como una versión preliminar, consulte [PowerShell Gallery](https://www.powershellgallery.com/packages/Az.Batch/1.0.0).

* Ejecute el cmdlet **Connect-AzAccount** para conectarse a su suscripción (los cmdlets de Azure Batch se incluyen en el módulo de Azure Resource Manager):

  ```powershell
  Connect-AzAccount
  ```

* **Registro con el espacio de nombres del proveedor de Batch**. Solo tiene que realizar esta operación **una vez por cada suscripción**.
  
  ```powershell
  Register-AzResourceProvider -ProviderNamespace Microsoft.Batch
  ```

## <a name="manage-batch-accounts-and-keys"></a>Administrar claves y cuentas por Batch

### <a name="create-a-batch-account"></a>Crear una cuenta de Batch

**New-AzBatchAccount** crea una cuenta de Batch en un grupo de recursos especificado. Si aún no dispone de un grupo de recursos, para crearlo debe ejecutar el cmdlet [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Especifique una de las regiones de Azure en el parámetro **Ubicación**, como "centro de EE. UU.". Por ejemplo:

```powershell
New-AzResourceGroup –Name MyBatchResourceGroup –Location "Central US"
```

Luego, cree una cuenta de Batch en el nuevo grupo de recursos. Especifique su nombre en <*account_name*>, e indique también la ubicación y el nombre del grupo de recursos. La operación de crear la cuenta de Batch puede tardar un tiempo en completarse. Por ejemplo:

```powershell
New-AzBatchAccount –AccountName <account_name> –Location "Central US" –ResourceGroupName <res_group_name>
```

> [!NOTE]
> El nombre de la cuenta de Batch debe ser único para la región de Azure en el grupo de recursos, contener entre 3 y 24 caracteres y usar solo letras minúsculas y números.

### <a name="get-account-access-keys"></a>Obtener claves de acceso de cuenta

**Get-AzBatchAccountKeys** muestra las claves de acceso asociadas a una cuenta de Azure Batch. Por ejemplo, ejecute lo siguiente para obtener las claves principales y secundarias de la cuenta que creó.

 ```powershell
$Account = Get-AzBatchAccountKeys –AccountName <account_name>

$Account.PrimaryAccountKey

$Account.SecondaryAccountKey
```

### <a name="generate-a-new-access-key"></a>Generar una nueva clave de acceso

**New-AzBatchAccountKey** genera una nueva clave de cuenta principal o secundaria para una cuenta de Azure Batch. Por ejemplo, para generar una nueva clave principal para la cuenta de Batch, escriba:

```powershell
New-AzBatchAccountKey -AccountName <account_name> -KeyType Primary
```

> [!NOTE]
> Para generar una nueva clave secundaria, especifique "Secondary" para el parámetro **KeyType** . Deberá volver a generar las claves principales y secundarias por separado.

### <a name="delete-a-batch-account"></a>Eliminar una cuenta de Batch

**Remove-AzBatchAccount** elimina una cuenta de Batch. Por ejemplo:

```powershell
Remove-AzBatchAccount -AccountName <account_name>
```

Cuando se le pida, confirme que desea quitar la cuenta. La eliminación de cuenta puede tardar unos minutos en completarse.

## <a name="create-a-batchaccountcontext-object"></a>Creación de un objeto BatchAccountContext

Puede usar la autenticación de clave compartida o de Azure Active Directory para autenticarse y administrar los recursos de Batch. Para la autenticación mediante los cmdlets de PowerShell de Batch, cree primero un objeto BatchAccountContext para almacenar las credenciales de cuenta o la identidad. Pase el objeto BatchAccountContext a los cmdlets que usan el parámetro **BatchContext** .

### <a name="shared-key-authentication"></a>Autenticación de clave compartida

```powershell
$context = Get-AzBatchAccountKeys -AccountName <account_name>
```

> [!NOTE]
> De forma predeterminada, la clave que se utiliza para la autenticación es la clave principal de la cuenta. Sin embargo, puede seleccionar otra clave explícitamente modificando la propiedad **KeyInUse** del objeto BatchAccountContext: `$context.KeyInUse = "Secondary"`.

### <a name="azure-active-directory-authentication"></a>Autenticación con Azure Active Directory

```powershell
$context = Get-AzBatchAccount -AccountName <account_name>
```

## <a name="create-and-modify-batch-resources"></a>Creación y modificación de recursos de Batch

Utilice cmdlets, como **New-AzBatchPool**, **New-AzBatchJob** y **New-AzBatchTask**, para crear recursos en una cuenta de Batch. Tiene a su disposición cmdlets **Get-** y **Set-** equivalentes para actualizar las propiedades de los recursos existentes y cmdlets **Remove-** para quitar recursos de una cuenta de Batch.

Cuando se utilizan muchos de estos cmdlets, además de pasar un objeto BatchContext, es preciso crear o pasar objetos que contienen la configuración detallada de los recursos, como se muestra en el ejemplo siguiente. Para obtener ejemplos adicionales, consulte la ayuda detallada de cada cmdlet.

### <a name="create-a-batch-pool"></a>Crear un grupo de Batch

Al crear o actualizar un grupo de Batch, seleccione la configuración de servicios en la nube o de máquina virtual para el sistema operativo de los nodos de proceso (consulte [Nodos y grupos](nodes-and-pools.md#configurations)). Si se especifica la configuración de servicios en la nube, se crea la imagen de sus nodos de proceso con una de las [versiones del sistema operativo invitado de Azure](../cloud-services/cloud-services-guestos-update-matrix.md#releases). Si se especifica la configuración de la máquina virtual, puede especificar alguna de las imágenes de máquina virtual de Linux o Windows que aparecen en [Azure Virtual Machines Marketplace][vm_marketplace], o proporcionar una imagen personalizada que haya preparado.

Al ejecutar **New-AzBatchPool**, pase la configuración del sistema operativo en un objeto PSCloudServiceConfiguration o PSVirtualMachineConfiguration. Por ejemplo, el siguiente fragmento de código crea un grupo de Batch con nodos de proceso de tamaño Standard_A1 en la configuración de la máquina virtual con una imagen de Ubuntu Server 18.04-LTS. En este caso, el parámetro **VirtualMachineConfiguration** especifica la variable *$configuration* como objeto PSVirtualMachineConfiguration. El parámetro **BatchContext** especifica una variable definida anteriormente *$context* como objeto BatchAccountContext.

```powershell
$imageRef = New-Object -TypeName "Microsoft.Azure.Commands.Batch.Models.PSImageReference" -ArgumentList @("UbuntuServer","Canonical","18.04-LTS")

$configuration = New-Object -TypeName "Microsoft.Azure.Commands.Batch.Models.PSVirtualMachineConfiguration" -ArgumentList @($imageRef, "batch.node.ubuntu 18.04")

New-AzBatchPool -Id "mypspool" -VirtualMachineSize "Standard_a1" -VirtualMachineConfiguration $configuration -AutoScaleFormula '$TargetDedicated=4;' -BatchContext $context
```

El número destino de nodos de proceso en el nuevo grupo se calcula mediante una fórmula de escalado automático. En este caso, la fórmula es simplemente **$TargetDedicated=4**, que indica que el número máximo de nodos de proceso en el grupo es 4.

## <a name="query-for-pools-jobs-tasks-and-other-details"></a>Consulta de grupos, trabajos, tareas y otros detalles

Utilice cmdlets, como **Get-AzBatchPool**, **Get-AzBatchJob** y **Get-AzBatchTask**, para consultar las entidades creadas en una cuenta de Batch.

### <a name="query-for-data"></a>Consulta de datos

Por ejemplo, use **Get-AzBatchPools** para encontrar sus grupos. De forma predeterminada, esto consulta todos los grupos de su cuenta, siempre que ya haya almacenado el objeto BatchAccountContext en *$context*:

```powershell
Get-AzBatchPool -BatchContext $context
```

### <a name="use-an-odata-filter"></a>Uso de un filtro OData

Puede proporcionar un filtro de OData mediante el parámetro **Filter** para buscar únicamente los objetos que le interesen. Por ejemplo, puede encontrar todos los grupos cuyos identificadores empiecen por "myPool":

```powershell
$filter = "startswith(id,'myPool')"

Get-AzBatchPool -Filter $filter -BatchContext $context
```

Este método no es tan flexible como "Where-Object" en una canalización local. Sin embargo, la consulta se envía al servicio Batch directamente para que todos los filtros se apliquen en el servidor, lo cual ahorra ancho de banda de Internet.

### <a name="use-the-id-parameter"></a>Uso del parámetro Id

Una alternativa al filtro OData es el parámetro **Id**. Para consultar un grupo específico con el Id. "myPool":

```powershell
Get-AzBatchPool -Id "myPool" -BatchContext $context
```

El parámetro **Id** solo admite búsquedas de identificadores completos. No se pueden usar caracteres comodín ni filtros de tipo OData.

### <a name="use-the-maxcount-parameter"></a>Uso del parámetro MaxCount

De forma predeterminada, cada cmdlet devuelve un máximo de 1000 objetos. Si se alcanza este límite, puede refinar el filtro para que devuelva menos objetos, o establecer explícitamente un máximo mediante el parámetro **MaxCount** . Por ejemplo:

```powershell
Get-AzBatchTask -MaxCount 2500 -BatchContext $context
```

Para quitar el límite superior, establezca **MaxCount** en 0 o menos.

### <a name="use-the-powershell-pipeline"></a>Uso de la canalización de PowerShell

Los cmdlets de Batch usan la canalización de PowerShell para enviar datos entre los cmdlets. El resultado es el mismo que si se especifica un parámetro, pero facilita el trabajo cuando se utilizan varias entidades.

Por ejemplo, busque y muestre todas las tareas de su cuenta:

```powershell
Get-AzBatchJob -BatchContext $context | Get-AzBatchTask -BatchContext $context
```

Reinicie todos los nodos de proceso de un grupo:

```powershell
Get-AzBatchComputeNode -PoolId "myPool" -BatchContext $context | Restart-AzBatchComputeNode -BatchContext $context
```

## <a name="application-package-management"></a>Administración de paquetes de aplicación

Los paquetes de aplicación proporcionan una manera simplificada de implementar aplicaciones a los nodos de proceso de los grupos. Con los cmdlets de PowerShell incluidos en Batch, puede cargar y administrar paquetes de aplicación de la cuenta de Batch e implementar versiones del paquete en los nodos de proceso.

**Cree** una aplicación:

```powershell
New-AzBatchApplication -AccountName <account_name> -ResourceGroupName <res_group_name> -ApplicationId "MyBatchApplication"
```

**Agregue** un paquete de aplicación:

```powershell
New-AzBatchApplicationPackage -AccountName <account_name> -ResourceGroupName <res_group_name> -ApplicationId "MyBatchApplication" -ApplicationVersion "1.0" -Format zip -FilePath package001.zip
```

Especifique la **versión predeterminada** de la aplicación:

```powershell
Set-AzBatchApplication -AccountName <account_name> -ResourceGroupName <res_group_name> -ApplicationId "MyBatchApplication" -DefaultVersion "1.0"
```

**Muestre** los paquetes de la aplicación:

```powershell
$application = Get-AzBatchApplication -AccountName <account_name> -ResourceGroupName <res_group_name> -ApplicationId "MyBatchApplication"

$application.ApplicationPackages
```

**Elimine** un paquete de aplicación:

```powershell
Remove-AzBatchApplicationPackage -AccountName <account_name> -ResourceGroupName <res_group_name> -ApplicationId "MyBatchApplication" -ApplicationVersion "1.0"
```

**Elimine** una aplicación:

```powershell
Remove-AzBatchApplication -AccountName <account_name> -ResourceGroupName <res_group_name> -ApplicationId "MyBatchApplication"
```

> [!NOTE]
> Para poder eliminar una aplicación, primero debe eliminar todas las versiones de los paquetes. Si intenta eliminar una aplicación que contiene paquetes, aparecerá un error de tipo 'conflicto'.

### <a name="deploy-an-application-package"></a>Implementación de un paquete de aplicación

Al crear un grupo, puede especificar uno o varios paquetes de aplicación para implementarlos. Cuando se especifica un paquete en el momento de la creación de un grupo, se implementa en todos los nodos cuando cada nodo se incorpora al grupo. También se implementan paquetes cuando un nodo se reinicia o se restablece su imagen inicial.

Especifique la opción `-ApplicationPackageReference` al crear un grupo para implementar un paquete de aplicación en los nodos del grupo cuando se unen a este. En primer lugar, cree un objeto **PSApplicationPackageReference** y configúrelo con el identificador de la aplicación y la versión del paquete que desee implementar en los nodos de proceso del grupo:

```powershell
$appPackageReference = New-Object Microsoft.Azure.Commands.Batch.Models.PSApplicationPackageReference

$appPackageReference.ApplicationId = "MyBatchApplication"

$appPackageReference.Version = "1.0"
```

Ahora se crea la configuración y el grupo. En este ejemplo se usa el parámetro **CloudServiceConfiguration** con un objeto de tipo `PSCloudServiceConfiguration` inicializado en `$configuration`, que establece **OSFamily** en `6` para "Windows Server 2019" y **OSVersion** en `*`. Especifique el objeto de referencia del paquete como argumento para la opción `ApplicationPackageReferences`:

```powershell
$configuration = New-Object -TypeName "Microsoft.Azure.Commands.Batch.Models.PSCloudServiceConfiguration" -ArgumentList @(6,"*")  # 6 = OSFamily 'Windows Server 2019'
New-AzBatchPool -Id "PoolWithAppPackage" -VirtualMachineSize "Small" -CloudServiceConfiguration $configuration -BatchContext $context -ApplicationPackageReferences $appPackageReference
```

Puede encontrar más información sobre los paquetes de aplicación en [Implementación de aplicaciones en nodos de proceso con paquetes de aplicaciones de Batch](batch-application-packages.md).

> [!IMPORTANT]
> Para utilizar paquetes de aplicación, primero se debe vincular una cuenta de Azure Storage a su cuenta de Batch.

### <a name="update-a-pools-application-packages"></a>Actualización de los paquetes de aplicación de un grupo

Para actualizar las aplicaciones asignadas a un grupo existente, antes es preciso crear un objeto PSApplicationPackageReference con las propiedades que desee (identificador de aplicación y versión del paquete):

```powershell
$appPackageReference = New-Object Microsoft.Azure.Commands.Batch.Models.PSApplicationPackageReference

$appPackageReference.ApplicationId = "MyBatchApplication"

$appPackageReference.Version = "2.0"

```

A continuación, debe obtener el grupo de Batch, eliminar los paquetes existentes, agregar la nueva referencia de paquete y actualizar el servicio Batch con la nueva configuración del grupo:

```powershell
$pool = Get-AzBatchPool -BatchContext $context -Id "PoolWithAppPackage"

$pool.ApplicationPackageReferences.Clear()

$pool.ApplicationPackageReferences.Add($appPackageReference)

Set-AzBatchPool -BatchContext $context -Pool $pool
```

Ahora, las propiedades del grupo están actualizadas en el servicio Batch. Para que la implementación del nuevo paquete de aplicación en los nodos de proceso del grupo surta efecto, debe reiniciar estos nodos o restablecer su imagen inicial. Puede reiniciar todos los nodos de un grupo con este comando:

```powershell
Get-AzBatchComputeNode -PoolId "PoolWithAppPackage" -BatchContext $context | Restart-AzBatchComputeNode -BatchContext $context
```

> [!TIP]
> Puede implementar varios paquetes de aplicación en los nodos de proceso de un grupo. Si en lugar de reemplazar los paquetes que están instalados actualmente prefiere *agregar* un paquete de aplicación, omita la línea `$pool.ApplicationPackageReferences.Clear()` anterior.

## <a name="next-steps"></a>Pasos siguientes

* Para conocer la sintaxis detallada de cmdlets y ejemplos de los mismos, consulte [Azure Batch Cmdlets](/powershell/module/az.batch)(Cmdlets de Lote de Azure).
* Para más información sobre las aplicaciones y los paquetes de aplicación de Batch, consulte [Implementación de aplicaciones en nodos de proceso con paquetes de aplicaciones de Batch](batch-application-packages.md).

[vm_marketplace]: https://azuremarketplace.microsoft.com/marketplace/apps/category/compute?filters=virtual-machine-images&page=1

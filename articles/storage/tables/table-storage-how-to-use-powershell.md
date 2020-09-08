---
title: Operaciones en Azure Table Storage con PowerShell | Microsoft Docs
description: Aprenda a ejecutar tareas comunes como crear, consultar y eliminar datos de una cuenta de Azure Table Storage mediante PowerShell.
author: roygara
ms.service: storage
ms.topic: article
ms.date: 04/05/2019
ms.author: rogarana
ms.subservice: tables
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e643a7ce5ccf4aa5107df1e505d90a0767517350
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89070418"
---
# <a name="perform-azure-table-storage-operations-with-azure-powershell"></a>Operaciones en Azure Table Storage con Azure PowerShell 
[!INCLUDE [storage-table-cosmos-db-tip-include](../../../includes/storage-table-cosmos-db-langsoon-tip-include.md)]

Azure Table Storage es un almacén de datos NoSQL que puede utilizar para almacenar y consultar conjuntos grandes de datos estructurados y no relacionales. Los componentes principales del servicio son tablas, entidades y propiedades. Una tabla es una colección de entidades. Una entidad es un conjunto de propiedades. Cada entidad puede tener hasta 252 propiedades, las cuales son todas pares nombre-valor. En este artículo supondremos que ya está familiarizado con los conceptos del servicio Azure Table Storage. Para más información, consulte [Introducción al modelo de datos de Table Service](/rest/api/storageservices/Understanding-the-Table-Service-Data-Model) e [Introducción a Azure Table Storage mediante .NET](../../cosmos-db/table-storage-how-to-use-dotnet.md).

En este artículo de instrucciones se describen las operaciones de Azure Table Storage más habituales. Aprenderá a: 

> [!div class="checklist"]
> * Creación de una tabla
> * Recuperar una tabla
> * Agregar entidades de tabla
> * Consultar una tabla
> * Eliminar entidades de tabla
> * Eliminar una tabla

En este artículo de instrucciones se muestra cómo crear una nueva cuenta de Azure Storage en un nuevo grupo de recursos, de modo que pueda eliminarla fácilmente cuando haya terminado. Si prefiere utilizar una cuenta de almacenamiento existente, puede hacerlo.

Los ejemplos requieren los módulos de PowerShell de Az `Az.Storage (1.1.0 or greater)` y `Az.Resources (1.2.0 or greater)`. En una ventana de PowerShell, ejecute `Get-Module -ListAvailable Az*` para buscar la versión. Si no aparece nada o necesita una actualización, vea [Install and configure Azure PowerShell](/powershell/azure/install-az-ps) (Instalar y configurar Azure PowerShell).

> [!IMPORTANT]
> El uso de esta característica de Azure desde PowerShell requiere que tenga el módulo `Az` instalado. La versión actual de `AzTable` no es compatible con el módulo de AzureRM anterior.
> Siga las [instrucciones de instalación más recientes para el módulo de Az](/powershell/azure/install-az-ps) en caso necesario.

Después de instalar o actualizar Azure PowerShell, hay que instalar el módulo **AzTable**, que contiene los comandos para administrar las entidades. Para instalarlo, ejecute PowerShell como administrador y use el comando **Install-Module**.

> [!IMPORTANT]
> Para obtener información sobre los motivos de compatibilidad en el nombre del módulo, todavía publicamos en Galería de PowerShell este mismo módulo con el nombre antiguo `AzureRmStorageTables`. Este documento hará referencia únicamente al nuevo nombre.

```powershell
Install-Module AzTable
```

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en la suscripción a Azure con el comando `Add-AzAccount` y siga las instrucciones de la pantalla.

```powershell
Add-AzAccount
```

## <a name="retrieve-list-of-locations"></a>Recuperación de la lista de ubicaciones

Si no sabe qué ubicación desea usar, puede enumerar las ubicaciones disponibles. Cuando se muestre la lista, busque la que desee usar. En estos ejemplos se utiliza **eastus**. Guarde este valor en la variable **location** para usarlo más adelante.

```powershell
Get-AzLocation | select Location
$location = "eastus"
```

## <a name="create-resource-group"></a>Creación de un grupo de recursos

Cree un grupo de recursos con el comando [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). 

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Guarde el nombre del grupo de recursos en una variable para usarlo más adelante. En este ejemplo se crea un grupo de recursos denominado *pshtablesrg* en la región *eastus*.

```powershell
$resourceGroup = "pshtablesrg"
New-AzResourceGroup -ResourceGroupName $resourceGroup -Location $location
```

## <a name="create-storage-account"></a>Crear cuenta de almacenamiento

Cree una cuenta de almacenamiento genérica estándar con almacenamiento con redundancia local (LRS) mediante [New-AzStorageAccount](/powershell/module/az.storage/New-azStorageAccount). Asegúrese de especificar un nombre de cuenta de almacenamiento único. Luego, obtenga el contexto que representa la cuenta de almacenamiento. Cuando actúa en una cuenta de almacenamiento, puede hacer referencia al contexto en lugar de proporcionar varias veces las credenciales.

```powershell
$storageAccountName = "pshtablestorage"
$storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroup `
  -Name $storageAccountName `
  -Location $location `
  -SkuName Standard_LRS `
  -Kind Storage

$ctx = $storageAccount.Context
```

## <a name="create-a-new-table"></a>Creación de una nueva tabla

Para crear una tabla, utilice el cmdlet [New-AzStorageTable](/powershell/module/az.storage/New-AzStorageTable). En este ejemplo, la tabla se denomina `pshtesttable`.

```powershell
$tableName = "pshtesttable"
New-AzStorageTable –Name $tableName –Context $ctx
```

## <a name="retrieve-a-list-of-tables-in-the-storage-account"></a>Recuperación de una lista de tablas de la cuenta de almacenamiento

Recupere una lista de tablas de la cuenta de almacenamiento mediante [Get-AzStorageTable](/powershell/module/azure.storage/Get-AzureStorageTable).

```powershell
Get-AzStorageTable –Context $ctx | select Name
```

## <a name="retrieve-a-reference-to-a-specific-table"></a>Recuperación de una referencia a una tabla específica

Para llevar a cabo operaciones en una tabla, se necesita una referencia a la tabla concreta. Obtenga una referencia mediante [Get-AzStorageTable](/powershell/module/azure.storage/Get-AzureStorageTable).

```powershell
$storageTable = Get-AzStorageTable –Name $tableName –Context $ctx
```

## <a name="reference-cloudtable-property-of-a-specific-table"></a>Propiedad CloudTable de referencia de una tabla específica

> [!IMPORTANT]
> La utilización de CloudTable es obligatoria cuando se trabaja con el módulo de PowerShell **AzTable**. Llame al comando **Get-AzStorageTable** para obtener la referencia a este objeto. Este comando también crea la tabla si aún no existe.

Para llevar a cabo operaciones en una tabla con **AzTable**, se necesita una referencia a la propiedad de CloudTable de una tabla concreta.

```powershell
$cloudTable = (Get-AzStorageTable –Name $tableName –Context $ctx).CloudTable
```

[!INCLUDE [storage-table-entities-powershell-include](../../../includes/storage-table-entities-powershell-include.md)]

## <a name="delete-a-table"></a>Eliminar una tabla

Para eliminar una tabla, use [Remove-AzStorageTable](/powershell/module/az.storage/Remove-AzStorageTable). Este cmdlet elimina la tabla, incluidos todos sus datos.

```powershell
Remove-AzStorageTable –Name $tableName –Context $ctx

# Retrieve the list of tables to verify the table has been removed.
Get-AzStorageTable –Context $Ctx | select Name
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ha creado un nuevo grupo de recursos y una cuenta de almacenamiento al principio de este artículo de ayuda, puede quitar el grupo de recursos para eliminar todos los recursos creados en este ejercicio. Con este comando se eliminan todos los recursos incluidos en el grupo de recursos, además del grupo de recursos en sí.

```powershell
Remove-AzResourceGroup -Name $resourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo de instrucciones ha aprendido a realizar operaciones habituales de Azure Table Storage con PowerShell, por ejemplo: 

> [!div class="checklist"]
> * Creación de una tabla
> * Recuperar una tabla
> * Agregar entidades de tabla
> * Consultar una tabla
> * Eliminar entidades de tabla
> * Eliminar una tabla

Para obtener más información, consulte los siguientes artículos:

* [Cmdlets de PowerShell de almacenamiento](/powershell/module/az.storage#storage)

* [Trabajo con tablas de Azure de PowerShell: AzureRmStorageTable/AzTable Módulo PS v2.0](https://paulomarquesc.github.io/working-with-azure-storage-tables-from-powershell)

* El [Explorador de Microsoft Azure Storage](../../vs-azure-tools-storage-manage-with-storage-explorer.md) es una aplicación independiente y gratuita de Microsoft que permite trabajar visualmente con los datos de Azure Storage en Windows, macOS y Linux.

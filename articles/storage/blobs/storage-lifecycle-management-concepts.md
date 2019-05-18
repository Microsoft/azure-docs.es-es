---
title: Administrar el ciclo de vida de almacenamiento de Azure
description: Aprenda a crear reglas de directiva de ciclo de vida para realizar la transición de datos antiguos del nivel de almacenamiento de acceso frecuente al nivel de almacenamiento de acceso esporádico y al nivel de almacenamiento de archivo.
services: storage
author: mhopkins-msft
ms.service: storage
ms.topic: conceptual
ms.date: 05/09/2019
ms.author: mhopkins
ms.reviewer: yzheng
ms.subservice: common
ms.openlocfilehash: 26ff592ea0d0a57049ae11a981fe8d8e77ca876f
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65606951"
---
# <a name="manage-the-azure-blob-storage-lifecycle"></a>Administrar el ciclo de vida de almacenamiento de blobs de Azure

Los conjuntos de datos tienen ciclos de vida únicos. Al principio del ciclo de vida, las personas acceden con frecuencia a algunos datos. Pero la necesidad de acceso desciende drásticamente a medida que los datos se hacen más antiguos. Algunos datos permanecen inactivos en la nube y, una vez almacenados, no se suele acceder a ellos. Otros datos expiran en días o meses después de su creación, mientras que otros conjuntos de datos se leen y modifican de forma activa en el transcurso de sus ciclos de vida. Administración de ciclo de vida de Azure Blob storage ofrece una directiva enriquecida basado en reglas para las cuentas de almacenamiento de GPv2 y Blob. Use la directiva para realizar la transición de los datos a los niveles de acceso adecuados o hacer que expiren al final de su ciclo de vida.

La directiva de administración del ciclo de vida le permite:

- Realizar la transición de los blobs a un nivel de almacenamiento de acceso esporádico (frecuente a esporádico, frecuente a archivo o esporádico a archivo) para optimizar el rendimiento y el costo
- Eliminar los blobs al final de sus ciclos de vida
- Definir reglas que se ejecutarán una vez al día en el nivel de cuenta de almacenamiento
- Aplicar reglas a contenedores o a un subconjunto de blobs (mediante prefijos como filtros)

Considere un escenario donde datos obtiene acceso frecuente durante las primeras fases del ciclo de vida, pero sólo en ocasiones, después de dos semanas. Transcurrido el primer mes, rara vez se accede al conjunto de datos. En este escenario, es mejor el almacenamiento de acceso frecuente durante las primeras etapas. Almacenamiento de acceso esporádico es más adecuado para un acceso ocasional. Archive storage es la mejor opción de niveles después de que envejecen los datos a través de un mes. Con el ajuste de los niveles de almacenamiento en relación con la antigüedad de los datos, puede designar las opciones de almacenamiento menos caras para satisfacer sus necesidades. Para conseguir esta transición, las reglas de directivas de administración del ciclo de vida se encuentran disponibles para mover los datos antiguos a niveles de almacenamiento de acceso más esporádico.

## <a name="storage-account-support"></a>Compatibilidad con la cuenta de almacenamiento

La directiva de administración del ciclo de vida está disponible con cuentas de uso general v2 (GPv2) y de Blob Storage. En el portal de Azure, puede actualizar una cuenta existente de propósito General (GPv1) a una cuenta de GPv2. Para más información sobre las cuentas de almacenamiento, vea [Introducción a las cuentas de Azure Storage](../common/storage-account-overview.md).  

## <a name="pricing"></a>Precios

La característica de administración del ciclo de vida es gratuita. A los clientes se les cobra el costo operativo habitual para las llamadas API [Mostrar blobs](https://docs.microsoft.com/rest/api/storageservices/list-blobs) y [Establecer el nivel del blob](https://docs.microsoft.com/rest/api/storageservices/set-blob-tier). Operación de eliminación es gratuita. Para más información sobre los precios, consulte [Precios de los blobs en bloques](https://azure.microsoft.com/pricing/details/storage/blobs/).

## <a name="regional-availability"></a>Disponibilidad regional

La característica de administración del ciclo de vida está disponible en todas las regiones de Azure globales.

## <a name="add-or-remove-a-policy"></a>Incorporación o eliminación de una directiva

Puede agregar, editar o quitar una directiva mediante cualquiera de los métodos siguientes:

* [Azure Portal](https://portal.azure.com)
* [Azure PowerShell](https://github.com/Azure/azure-powershell/releases)
* [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli)
* [API de REST](https://docs.microsoft.com/rest/api/storagerp/managementpolicies)

En este artículo se muestra cómo administrar la directiva mediante el portal y los métodos de PowerShell.  

> [!NOTE]
> Si habilita reglas de firewall para la cuenta de almacenamiento, puede que se bloqueen las solicitudes de administración del ciclo de vida. Puede desbloquear estas solicitudes si proporciona excepciones. La omisión necesaria son: `Logging,  Metrics,  AzureServices`. Para más información, consulte la sección Excepciones en [Configuración de firewalls y redes virtuales](https://docs.microsoft.com/azure/storage/common/storage-network-security#exceptions).

### <a name="azure-portal"></a>Azure Portal

1. Inicie sesión en el [Azure Portal](https://portal.azure.com).

2. Seleccione **todos los recursos** y, a continuación, seleccione la cuenta de almacenamiento.

3. En **Blob Service**, seleccione **administración del ciclo de vida** para ver o cambiar la directiva.

4. El siguiente JSON es un ejemplo de una regla que se puede pegar en el **administración del ciclo de vida** página del portal.

   ```json
   {
     "rules": [
       {
         "name": "ruleFoo",
         "enabled": true,
         "type": "Lifecycle",
         "definition": {
           "filters": {
             "blobTypes": [ "blockBlob" ],
             "prefixMatch": [ "container1/foo" ]
           },
           "actions": {
             "baseBlob": {
               "tierToCool": { "daysAfterModificationGreaterThan": 30 },
               "tierToArchive": { "daysAfterModificationGreaterThan": 90 },
               "delete": { "daysAfterModificationGreaterThan": 2555 }
             },
             "snapshot": {
               "delete": { "daysAfterCreationGreaterThan": 90 }
             }
           }
         }
       }
     ]
   }
   ```

5. Para obtener más información acerca de este ejemplo JSON, consulte el [directiva](#policy) y [reglas](#rules) secciones.

### <a name="powershell"></a>PowerShell

El siguiente script de PowerShell puede usarse para agregar una directiva a la cuenta de almacenamiento. El `$rgname` variable se debe inicializar con el nombre del grupo de recursos. El `$accountName` variable se debe inicializar con el nombre de cuenta de almacenamiento.

```powershell
#Install the latest module
Install-Module -Name Az -Repository PSGallery

#Initialize the following with your resource group and storage account names
$rgname = ""
$accountName = ""

#Create a new action object
$action = Add-AzStorageAccountManagementPolicyAction -BaseBlobAction Delete -daysAfterModificationGreaterThan 2555
$action = Add-AzStorageAccountManagementPolicyAction -InputObject $action -BaseBlobAction TierToArchive -daysAfterModificationGreaterThan 90
$action = Add-AzStorageAccountManagementPolicyAction -InputObject $action -BaseBlobAction TierToCool -daysAfterModificationGreaterThan 30
$action = Add-AzStorageAccountManagementPolicyAction -InputObject $action -SnapshotAction Delete -daysAfterCreationGreaterThan 90

# Create a new filter object
# PowerShell automatically sets BlobType as “blockblob” because it is the only available option currently
$filter = New-AzStorageAccountManagementPolicyFilter -PrefixMatch ab,cd

#Create a new rule object
#PowerShell automatically sets Type as “Lifecycle” because it is the only available option currently
$rule1 = New-AzStorageAccountManagementPolicyRule -Name Test -Action $action -Filter $filter

#Set the policy
$policy = Set-AzStorageAccountManagementPolicy -ResourceGroupName $rgname -StorageAccountName $accountName -Rule $rule1
```

## <a name="azure-resource-manager-template-with-lifecycle-management-policy"></a>Plantilla de Azure Resource Manager con la directiva de administración del ciclo de vida

Puede definir la administración del ciclo de vida mediante el uso de plantillas de Azure Resource Manager. Esta es una plantilla de ejemplo para implementar una cuenta de almacenamiento de RA-GRS GPv2 con una directiva de administración del ciclo de vida.

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "variables": {
    "storageAccountName": "[uniqueString(resourceGroup().id)]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[variables('storageAccountName')]",
      "location": "[resourceGroup().location]",
      "apiVersion": "2019-04-01",
      "sku": {
        "name": "Standard_RAGRS"
      },
      "kind": "StorageV2",
      "properties": {
        "networkAcls": {}
      }
    },
    {
      "name": "[concat(variables('storageAccountName'), '/default')]",
      "type": "Microsoft.Storage/storageAccounts/managementPolicies",
      "apiVersion": "2019-04-01",
      "dependsOn": [
        "[variables('storageAccountName')]"
      ],
      "properties": {
        "policy": {...}
      }
    }
  ],
  "outputs": {}
}
```

## <a name="policy"></a>Directiva

Una directiva de administración del ciclo de vida es una colección de reglas en un documento JSON:

```json
{
  "rules": [
    {
      "name": "rule1",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {...}
    },
    {
      "name": "rule2",
      "type": "Lifecycle",
      "definition": {...}
    }
  ]
}
```

Una directiva es una colección de reglas:

| Nombre de parámetro | Tipo de parámetro | Notas |
|----------------|----------------|-------|
| `rules`        | Una matriz de objetos de regla | Se requiere al menos una regla en una directiva. Puede definir hasta 100 reglas en una directiva.|

Cada regla dentro de la directiva tiene varios parámetros:

| Nombre de parámetro | Tipo de parámetro | Notas | Obligatorio |
|----------------|----------------|-------|----------|
| `name`         | String |Nombre de una regla puede incluir hasta 256 caracteres alfanuméricos. El nombre de regla distingue mayúsculas de minúsculas.  Debe ser único dentro de una directiva. | True  |
| `enabled`      | Boolean | Un valor booleano opcional para permitir que una regla ser temporal deshabilitada. Valor predeterminado es true si no se establece. | False | 
| `type`         | Un valor de enumeración | El tipo actual válido es `Lifecycle`. | True  |
| `definition`   | Un objeto que define la regla del ciclo de vida | Cada definición se compone de un conjunto de filtros y un conjunto de acciones. | True  |

## <a name="rules"></a>Reglas

Cada definición de regla incluye un conjunto de filtros y un conjunto de acciones. El [conjunto de filtros](#rule-filters) limita las acciones de regla a un determinado conjunto de objetos dentro de un contenedor o nombres de objetos. El [conjunto de acciones ](#rule-actions) aplica las acciones de nivel o eliminación al conjunto filtrado de objetos.

### <a name="sample-rule"></a>Ejemplo de regla

La regla de ejemplo siguiente filtra la cuenta para ejecutar las acciones en los objetos que existen dentro de `container1` y comenzar con `foo`.  

- Establecer el nivel de blob en nivel esporádico 30 días después de la última modificación
- Establecer el nivel de blob en nivel de almacenamiento de archivo 90 días después de la última modificación
- Eliminar el blob 2555 días (siete años) después de la última modificación
- Eliminar instantáneas de blob 90 días después de la creación de las instantáneas

```json
{
  "rules": [
    {
      "name": "ruleFoo",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": [ "blockBlob" ],
          "prefixMatch": [ "container1/foo" ]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": { "daysAfterModificationGreaterThan": 30 },
            "tierToArchive": { "daysAfterModificationGreaterThan": 90 },
            "delete": { "daysAfterModificationGreaterThan": 2555 }
          },
          "snapshot": {
            "delete": { "daysAfterCreationGreaterThan": 90 }
          }
        }
      }
    }
  ]
}
```

### <a name="rule-filters"></a>Filtros de reglas

Los filtros limitan las acciones de regla a un subconjunto de blobs dentro de la cuenta de almacenamiento. Si se define más de un filtro, se ejecuta un valor lógico `AND` en todos los filtros.

Los filtros incluyen:

| Nombre de filtro | Tipo de filtro | Notas | Necesario |
|-------------|-------------|-------|-------------|
| blobTypes   | Una matriz de valores de enumeración predefinidos. | La versión actual admite `blockBlob`. | Sí |
| prefixMatch | Una matriz de cadenas de prefijos con los que debe hacer coincidencias. Cada regla puede definir hasta 10 prefijos. Una cadena de prefijos debe comenzar con el nombre de un contenedor. Por ejemplo, si desea hacer coincidir todos los blobs `https://myaccount.blob.core.windows.net/container1/foo/...` para una regla, es el prefixMatch `container1/foo`. | Si no define prefixMatch, la regla se aplica a todos los blobs dentro de la cuenta de almacenamiento.  | No |

### <a name="rule-actions"></a>Acciones de regla

Las acciones se aplican a los blobs filtrados cuando se cumple la condición de ejecución.

Administración del ciclo de vida es compatible con los niveles y eliminación de blobs y la eliminación de instantáneas de blob. Defina al menos una acción para cada regla en los blobs o las instantáneas de blob.

| .        | Blob de base                                   | Instantánea      |
|---------------|---------------------------------------------|---------------|
| tierToCool    | Admite blobs actualmente en el nivel de acceso frecuente.         | No compatible |
| tierToArchive | Admite blobs actualmente en el nivel de acceso frecuente o esporádico. | No compatible |
| eliminar        | Compatible                                   | Compatible     |

>[!NOTE]
>Si define más de una acción en el mismo blob, la administración del ciclo de vida aplica la acción menos cara al blob. Por ejemplo, la acción `delete` es más económica que la acción `tierToArchive`. La acción `tierToArchive` es más económica que la acción `tierToCool`.

Las condiciones de ejecución se basan en la edad. Para realizar el seguimiento de la antigüedad, los blobs de base usan la hora de la última modificación y las instantáneas de blob usan la hora de creación de la instantánea.

| Ejecutar la condición de acción             | Valor de la condición                          | DESCRIPCIÓN                             |
|----------------------------------|------------------------------------------|-----------------------------------------|
| daysAfterModificationGreaterThan | Valor entero que indica la antigüedad en días | La condición para las acciones de blob de base     |
| daysAfterCreationGreaterThan     | Valor entero que indica la antigüedad en días | La condición para las acciones de instantánea de blob |

## <a name="examples"></a>Ejemplos

Los ejemplos siguientes muestran cómo abordar escenarios comunes con las reglas de directivas del ciclo de vida.

### <a name="move-aging-data-to-a-cooler-tier"></a>Cambio de los datos antiguos a un nivel de acceso más esporádico

En este ejemplo se muestra cómo realizar la transición de blobs en bloques con el prefijo `container1/foo` o `container2/bar`. La directiva realiza la transición de los blobs que no se han modificado durante más de 30 días al almacenamiento de acceso esporádico, y los blobs no modificados en 90 días al nivel de acceso de archivo:

```json
{
  "rules": [
    {
      "name": "agingRule",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": [ "blockBlob" ],
          "prefixMatch": [ "container1/foo", "container2/bar" ]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": { "daysAfterModificationGreaterThan": 30 },
            "tierToArchive": { "daysAfterModificationGreaterThan": 90 }
          }
        }
      }
    }
  ]
}
```

### <a name="archive-data-at-ingest"></a>Archivado de datos en la ingesta

Algunos datos permanecen inactivos en la nube y, una vez almacenados, no se accede a ellos prácticamente nunca. La siguiente directiva de ciclo de vida está configurada para archivar los datos una vez que se ingiere. En este ejemplo los blobs en la cuenta de almacenamiento dentro del contenedor de bloques de transiciones `archivecontainer` en un nivel de archivo. Se realiza la transición al actuar en los blobs en 0 días después de la hora de última modificación:

```json
{
  "rules": [
    {
      "name": "archiveRule",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": [ "blockBlob" ],
          "prefixMatch": [ "archivecontainer" ]
        },
        "actions": {
          "baseBlob": {
              "tierToArchive": { "daysAfterModificationGreaterThan": 0 }
          }
        }
      }
    }
  ]
}

```

### <a name="expire-data-based-on-age"></a>Expiración de datos en función de la antigüedad

Algunos datos se esperan que caduque días o meses después de la creación. Puede configurar una directiva de administración del ciclo de vida para que los datos expiren mediante eliminación en función de su antigüedad. El ejemplo siguiente muestra una directiva que elimina todos los blobs en bloques anteriores a 365 días.

```json
{
  "rules": [
    {
      "name": "expirationRule",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": [ "blockBlob" ]
        },
        "actions": {
          "baseBlob": {
            "delete": { "daysAfterModificationGreaterThan": 365 }
          }
        }
      }
    }
  ]
}
```

### <a name="delete-old-snapshots"></a>Eliminación de instantáneas antiguas

En el caso de los datos que se modifican y a los que se accede con regularidad a lo largo de su ciclo de vida, las instantáneas suelen utilizarse para realizar un seguimiento de las versiones anteriores de los datos. Puede crear una directiva que elimine las instantáneas anteriores en función de la antigüedad de la instantánea. La antigüedad de la instantánea se determina mediante la evaluación de la hora de creación de la instantánea. Esta regla de directiva elimina las instantáneas de blobs en bloques en el contenedor `activedata` con una antigüedad de 90 días o más a contar desde el momento de la creación de la instantánea.

```json
{
  "rules": [
    {
      "name": "snapshotRule",
      "enabled": true,
      "type": "Lifecycle",
    "definition": {
        "filters": {
          "blobTypes": [ "blockBlob" ],
          "prefixMatch": [ "activedata" ]
        },
        "actions": {
          "snapshot": {
            "delete": { "daysAfterCreationGreaterThan": 90 }
          }
        }
      }
    }
  ]
}
```

## <a name="faq"></a>Preguntas más frecuentes

**He creado una nueva directiva, ¿por qué las acciones no se ejecutan inmediatamente?**  
La plataforma ejecuta la directiva del ciclo de vida una vez al día. Una vez que configure una directiva, puede tardar hasta 24 horas para algunas acciones ejecutar por primera vez.  

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo recuperar datos tras la eliminación accidental:

- [Eliminación temporalmente de blobs de Azure Storage](../blobs/storage-blob-soft-delete.md)

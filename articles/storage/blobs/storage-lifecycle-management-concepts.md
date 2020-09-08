---
title: Administración del ciclo de vida de Azure Storage
description: Aprenda a crear reglas de directiva de ciclo de vida para realizar la transición de datos antiguos del nivel de almacenamiento de acceso frecuente al nivel de almacenamiento de acceso esporádico y al nivel de almacenamiento de archivo.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 04/24/2020
ms.service: storage
ms.subservice: common
ms.topic: conceptual
ms.reviewer: yzheng
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: b1bf8fbfb6d2c141a2b18c3599631f6383883908
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89074430"
---
# <a name="manage-the-azure-blob-storage-lifecycle"></a>Administración del ciclo de vida de Azure Blob Storage

Los conjuntos de datos tienen ciclos de vida únicos. Al principio del ciclo de vida, las personas acceden con frecuencia a algunos datos. Pero la necesidad de acceso desciende drásticamente a medida que los datos se hacen más antiguos. Algunos datos permanecen inactivos en la nube y, una vez almacenados, no se suele acceder a ellos. Otros datos expiran en días o meses después de su creación, mientras que otros conjuntos de datos se leen y modifican de forma activa en el transcurso de sus ciclos de vida. La administración del ciclo de vida de Azure Blob Storage ofrece una directiva completa basada en reglas para las cuentas de GPv2 y Blob Storage. Use la directiva para realizar la transición de los datos a los niveles de acceso adecuados o hacer que expiren al final de su ciclo de vida.

La directiva de administración del ciclo de vida le permite:

- Realizar la transición de los blobs a un nivel de almacenamiento de acceso esporádico (frecuente a esporádico, frecuente a archivo o esporádico a archivo) para optimizar el rendimiento y el costo
- Eliminar los blobs al final de sus ciclos de vida
- Definir reglas que se ejecutarán una vez al día en el nivel de cuenta de almacenamiento
- Aplicar reglas a contenedores o a un subconjunto de blobs (mediante prefijos de nombre o [etiquetas de índice de blobs](storage-manage-find-blobs.md) como filtros)

Considere un escenario donde los datos tienen acceso frecuente durante las primeras fases del ciclo de vida, pero solo ocasionalmente al cabo de dos semanas. Transcurrido el primer mes, rara vez se accede al conjunto de datos. En este escenario, es mejor el almacenamiento de acceso frecuente durante las primeras etapas. El almacenamiento de acceso esporádico es más adecuado para un acceso ocasional. El almacenamiento de archivo es la mejor opción de nivel una vez que los datos tengan un mes. Con el ajuste de los niveles de almacenamiento en relación con la antigüedad de los datos, puede designar las opciones de almacenamiento menos caras para satisfacer sus necesidades. Para conseguir esta transición, las reglas de directivas de administración del ciclo de vida se encuentran disponibles para mover los datos antiguos a niveles de almacenamiento de acceso más esporádico.

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

## <a name="availability-and-pricing"></a>Disponibilidad y precios

La directiva de administración del ciclo de vida está disponible en todas las regiones de Azure para cuentas de almacenamiento de uso general v2 (GPv2), de Blob Storage y de Premium Block Blob. En Azure Portal, puede convertir una cuenta existente de uso general (GPv1) en una cuenta de GPv2. Para más información sobre las cuentas de almacenamiento, vea [Introducción a las cuentas de Azure Storage](../common/storage-account-overview.md).  

La característica de administración del ciclo de vida es gratuita. A los clientes se les cobra el costo operativo habitual para las llamadas API [Establecer el nivel del blob](https://docs.microsoft.com/rest/api/storageservices/set-blob-tier). La operación de eliminación es gratuita. Para más información sobre los precios, consulte [Precios de los blobs en bloques](https://azure.microsoft.com/pricing/details/storage/blobs/).

## <a name="add-or-remove-a-policy"></a>Incorporación o eliminación de una directiva

Puede agregar, editar o quitar una directiva mediante cualquiera de los métodos siguientes:

* [Azure Portal](https://portal.azure.com)
* [Azure PowerShell](https://github.com/Azure/azure-powershell/releases)
* [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli)
* [API de REST](https://docs.microsoft.com/rest/api/storagerp/managementpolicies)

Una directiva se puede leer o escribir en su totalidad. No se admiten las actualizaciones parciales. 

> [!NOTE]
> Si habilita reglas de firewall para la cuenta de almacenamiento, puede que se bloqueen las solicitudes de administración del ciclo de vida. Puede desbloquear estas solicitudes proporcionando excepciones para los servicios de confianza de Microsoft. Para más información, consulte la sección Excepciones en [Configuración de firewalls y redes virtuales](https://docs.microsoft.com/azure/storage/common/storage-network-security#exceptions).

En este artículo se muestra cómo administrar la directiva mediante el portal y los métodos de PowerShell.  

# <a name="portal"></a>[Portal](#tab/azure-portal)

Hay dos formas de agregar una directiva en Azure Portal. 

* [Vista de lista de Azure Portal](#azure-portal-list-view)
* [Vista de código de Azure Portal](#azure-portal-code-view)

#### <a name="azure-portal-list-view"></a>Vista de lista de Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. En Azure Portal, busque y seleccione su cuenta de almacenamiento. 

3. En **Blob service**, seleccione **Administración del ciclo de vida** para ver o cambiar las reglas.

4. Seleccione la pestaña **Vista de lista**.

5. Seleccione **Agregar regla** y después rellene los campos del formulario **Conjunto de acciones**. En el siguiente ejemplo, los blobs se mueven al almacenamiento esporádico si no se han modificado durante 30 días.

   ![Página del conjunto de acciones de administración del ciclo de vida en Azure Portal](media/storage-lifecycle-management-concepts/lifecycle-management-action-set.png)

6. Seleccione **Conjunto de filtros** para agregar un filtro opcional. A continuación, seleccione **Examinar** para especificar un contenedor y una carpeta por los que filtrar.

   ![Página del conjunto de filtros de administración del ciclo de vida en Azure Portal](media/storage-lifecycle-management-concepts/lifecycle-management-filter-set-browse.png)

8. Seleccione **Revisar + agregar** para revisar la configuración de la directiva.

9. Seleccione **Agregar** para agregar la nueva directiva.

#### <a name="azure-portal-code-view"></a>Vista de código de Azure Portal
1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. En Azure Portal, busque y seleccione su cuenta de almacenamiento.

3. En **Blob service**, seleccione **Administración del ciclo de vida** para ver o cambiar la directiva.

4. El siguiente JSON es un ejemplo de una directiva que se puede pegar en la pestaña **Vista Código**.

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

5. Seleccione **Guardar**.

6. Para obtener más información sobre este ejemplo JSON, consulte las secciones [Directiva](#policy) y [Reglas](#rules).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

El siguiente script de PowerShell puede usarse para agregar una directiva a la cuenta de almacenamiento. La variable `$rgname` se debe inicializar con el nombre del grupo de recursos. La variable `$accountName` se debe inicializar con el nombre de la cuenta de almacenamiento.

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

# <a name="template"></a>[Plantilla](#tab/template)

Puede definir la administración del ciclo de vida mediante plantillas de Azure Resource Manager. Aquí tiene una plantilla de ejemplo para implementar una cuenta de almacenamiento de RA-GRS GPv2 con una directiva de administración del ciclo de vida.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
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

---

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

Cada regla de la directiva tiene varios parámetros:

| Nombre de parámetro | Tipo de parámetro | Notas | Obligatorio |
|----------------|----------------|-------|----------|
| `name`         | String |Un nombre de regla puede incluir hasta 256 caracteres alfanuméricos. El nombre de regla distingue mayúsculas de minúsculas.  Debe ser único dentro de una directiva. | True |
| `enabled`      | Boolean | Un valor booleano opcional para permitir que una regla se deshabilite de forma temporal. El valor predeterminado es true si no se establece. | False | 
| `type`         | Un valor de enumeración | El tipo actual válido es `Lifecycle`. | True |
| `definition`   | Un objeto que define la regla del ciclo de vida | Cada definición se compone de un conjunto de filtros y un conjunto de acciones. | True |

## <a name="rules"></a>Reglas

Cada definición de regla incluye un conjunto de filtros y un conjunto de acciones. El [conjunto de filtros](#rule-filters) limita las acciones de regla a un determinado conjunto de objetos dentro de un contenedor o nombres de objetos. El [conjunto de acciones ](#rule-actions) aplica las acciones de nivel o eliminación al conjunto filtrado de objetos.

### <a name="sample-rule"></a>Ejemplo de regla

La siguiente regla de ejemplo filtra la cuenta para ejecutar las acciones en objetos que existen dentro de `container1` y empiezan por `foo`.  

>[!NOTE]
>- La administración del ciclo de vida solo admite el tipo de blob en bloques.<br>
>- La administración del ciclo de vida no afecta a los contenedores del sistema como $logs y $web.

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

Entre los filtros están los siguientes:

| Nombre de filtro | Tipo de filtro | Notas | Es obligatorio |
|-------------|-------------|-------|-------------|
| blobTypes   | Una matriz de valores de enumeración predefinidos. | La versión actual admite `blockBlob`. | Sí |
| prefixMatch | Una matriz de cadenas de prefijos con los que debe hacer coincidencias. Cada regla puede definir hasta 10 prefijos. Una cadena de prefijos debe comenzar con el nombre de un contenedor. Por ejemplo, si quiere que todos los blobs de `https://myaccount.blob.core.windows.net/container1/foo/...` coincidan en una regla, prefixMatch es `container1/foo`. | Si no define prefixMatch, la regla se aplica a todos los blobs de la cuenta de almacenamiento.  | No |
| blobIndexMatch | Una matriz de valores de diccionario que se compone de las condiciones de clave y valor de la etiqueta de índice de blobs con las que debe haber coincidencias. Cada regla puede definir hasta 10 condiciones de etiqueta de índice de blobs. Por ejemplo, si quiere que todos los blobs coincidan con `Project = Contoso` en `https://myaccount.blob.core.windows.net/` en relación a una regla, el valor de blobIndexMatch es `{"name": "Project","op": "==","value": "Contoso"}`. | Si no define blobIndexMatch, la regla se aplica a todos los blobs de la cuenta de almacenamiento. | No |

> [!NOTE]
> El índice de blobs está en versión preliminar pública y se encuentra disponible en las regiones **Centro de Canadá**, **Este de Canadá**, **Centro de Francia** y **Sur de Francia**. Para más información sobre esta característica junto con las limitaciones y los problemas conocidos, consulte [Administración y búsqueda de datos en Azure Blob Storage con el Índice de blobs (versión preliminar)](storage-manage-find-blobs.md).

### <a name="rule-actions"></a>Acciones de regla

Las acciones se aplican a los blobs filtrados cuando se cumple la condición de ejecución.

La administración del ciclo de vida admite el cambio de niveles y la eliminación de blobs e instantáneas de blob. Defina al menos una acción para cada regla en los blobs o las instantáneas de blob.

| Acción        | Blob de base                                   | Instantánea      |
|---------------|---------------------------------------------|---------------|
| tierToCool    | Admite blobs actualmente en el nivel de acceso frecuente.         | No compatible |
| tierToArchive | Admite blobs actualmente en el nivel de acceso frecuente o esporádico. | No compatible |
| delete        | Compatible                                   | Compatible     |

>[!NOTE]
>Si define más de una acción en el mismo blob, la administración del ciclo de vida aplica la acción menos cara al blob. Por ejemplo, la acción `delete` es más económica que la acción `tierToArchive`. La acción `tierToArchive` es más económica que la acción `tierToCool`.

Las condiciones de ejecución se basan en la antigüedad. Para realizar el seguimiento de la antigüedad, los blobs de base usan la hora de la última modificación y las instantáneas de blob usan la hora de creación de la instantánea.

| Condición de ejecución de acción             | Valor de la condición                          | Descripción                             |
|----------------------------------|------------------------------------------|-----------------------------------------|
| daysAfterModificationGreaterThan | Valor entero que indica la antigüedad en días | Condición de las acciones de blob de base     |
| daysAfterCreationGreaterThan     | Valor entero que indica la antigüedad en días | Condición de las acciones de instantánea de blob |

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

### <a name="archive-data-after-ingest"></a>Archivado de datos después de la ingesta

Algunos datos permanecen inactivos en la nube y, una vez almacenados, no se accede a ellos prácticamente nunca. La siguiente directiva del ciclo de vida está configurada para archivar los datos poco después de que se ingieran. En este ejemplo se realiza la transición de los blobs en bloques en la cuenta de almacenamiento en el contenedor `archivecontainer` a un nivel de archivo. La transición se realiza al actuar en los blobs 0 días después de la hora de la última modificación:

> [!NOTE] 
> Se recomienda cargar los blobs directamente en el nivel de archivo para que sea más eficaz. Puede usar el encabezado x-ms-access-tier para [PutBlob](https://docs.microsoft.com/rest/api/storageservices/put-blob) o [PutBlockList](https://docs.microsoft.com/rest/api/storageservices/put-block-list) con la versión de REST 2018-11-09 y versiones más recientes o las bibliotecas de cliente de Blob Storage más recientes. 

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

Se espera que algunos datos expiren días o meses después de la creación. Puede configurar una directiva de administración del ciclo de vida para que los datos expiren mediante eliminación en función de su antigüedad. En el ejemplo siguiente se muestra una directiva que elimina todos los blobs en bloques con una antigüedad superior a 365 días.

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

### <a name="delete-data-with-blob-index-tags"></a>Eliminación de datos con etiquetas de índice de blobs
Algunos datos solo deben expirar si se marcan explícitamente para su eliminación. Puede configurar una directiva de administración del ciclo de vida para que expiren los datos etiquetados con los atributos de clave/valor del índice de blobs. En el ejemplo siguiente se muestra una directiva que elimina todos los blobs en bloques con `Project = Contoso`. Para más información sobre el índice de blobs, consulte [Administración y búsqueda de datos en Azure Blob Storage con el Índice de blobs (versión preliminar)](storage-manage-find-blobs.md).

```json
{
    "rules": [
        {
            "enabled": true,
            "name": "DeleteContosoData",
            "type": "Lifecycle",
            "definition": {
                "actions": {
                    "baseBlob": {
                        "delete": {
                            "daysAfterModificationGreaterThan": 0
                        }
                    }
                },
                "filters": {
                    "blobIndexMatch": [
                        {
                            "name": "Project",
                            "op": "==",
                            "value": "Contoso"
                        }
                    ],
                    "blobTypes": [
                        "blockBlob"
                    ]
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

**He creado una directiva, ¿por qué las acciones no se ejecutan inmediatamente?**  
La plataforma ejecuta la directiva del ciclo de vida una vez al día. Una vez que configure una directiva, algunas acciones pueden tardar hasta 24 horas en ejecutarse por primera vez.  

**Si actualizo una directiva existente, ¿cuánto tiempo tardan en ejecutarse las acciones?**  
La directiva actualizada tarda hasta 24 horas en entrar en vigor. Una vez que la directiva está en vigor, las acciones pueden tardar hasta 24 horas en ejecutarse. Por lo tanto, las acciones de la directiva pueden tardar hasta 48 horas en completarse.   

**He rehidratado manualmente un blob archivado, ¿cómo evito que vuelva temporalmente al nivel de archivo?**  
Cuando se mueve un blob desde un nivel de acceso a otro, su hora de última modificación no cambia. Si rehidrata manualmente un blob archivado al nivel de acceso frecuente, el motor de administración del ciclo de vida podría devolverlo al nivel de archivo. Deshabilite la regla que afecte temporalmente a este blob para impedir que se vuelva a archivar. Vuelva a habilitar la regla cuando el blob se pueda volver a mover con seguridad al nivel de archivo. También puede copiar el blob en otra ubicación si necesita permanecer en el nivel de acceso frecuente o esporádico de forma permanente.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo recuperar datos tras la eliminación accidental:

- [Eliminación temporal de blobs de Azure Storage](../blobs/storage-blob-soft-delete.md)

Aprenda a administrar y buscar datos con el índice de blobs:

- [Administración y búsqueda de datos en Azure Blob Storage con el Índice de blobs](storage-manage-find-blobs.md)

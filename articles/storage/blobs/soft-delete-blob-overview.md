---
title: Eliminación temporal para blobs
titleSuffix: Azure Storage
description: La eliminación temporal para blobs protege los datos para que pueda recuperarlos más fácilmente si una aplicación u otro usuario de la cuenta de almacenamiento los modifican o eliminan por error.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 07/15/2020
ms.author: tamram
ms.subservice: blobs
ms.openlocfilehash: 2e390c9d5d2fa7c6551ed661c6c25096732eefd5
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88057098"
---
# <a name="soft-delete-for-blobs"></a>Eliminación temporal para blobs

La eliminación temporal para blobs evita que los datos se modifiquen o eliminen de forma accidental o errónea. Cuando la eliminación temporal para blobs está habilitada para una cuenta de almacenamiento, los blobs, las versiones de blobs (versión preliminar) y las instantáneas de esa cuenta de almacenamiento pueden recuperarse una vez eliminados, dentro de un período de retención que especifique.

Si existe la posibilidad de que una aplicación u otro usuario de la cuenta de almacenamiento modifiquen o borren sus datos accidentalmente, Microsoft aconseja activar la eliminación temporal. Para más información sobre cómo habilitar la eliminación temporal, consulte [Habilitación y administración de la eliminación temporal para blobs](soft-delete-enable.md).

[!INCLUDE [storage-data-lake-gen2-support](../../../includes/storage-data-lake-gen2-support.md)]

## <a name="about-soft-delete-for-blobs"></a>Acerca de la eliminación temporal para blobs

Si la eliminación temporal para blobs está habilitada en una cuenta de almacenamiento, puede recuperar los objetos después de eliminarlos dentro del período de retención de datos especificado. Esta protección se extiende a todos los blobs (blobs en bloques, blobs en anexos y blobs en páginas) que se borran como resultado de una sobrescritura.

Si se eliminan los datos de un blob o una instantánea existentes mientras la eliminación temporal de blobs está habilitada, pero el control de versiones de blobs (versión preliminar) no está habilitado, se genera una instantánea de eliminación temporal para guardar el estado de los datos sobrescritos. Una vez expirado el período de retención especificado, el objeto se elimina de forma permanente.

Si el control de versiones de blobs y la eliminación temporal de blobs están habilitadas en la cuenta de almacenamiento, al eliminar un blob se crea una nueva versión, en lugar de una instantánea eliminada temporalmente. La nueva versión no se elimina de forma temporal y no se quita cuando expira el período de retención de eliminación temporal. Las versiones eliminadas temporalmente de un blob se pueden restaurar en el período de retención mediante una llamada a la operación [Undelete Blob](/rest/api/storageservices/undelete-blob). Posteriormente, el blob se puede restaurar desde una de sus versiones llamando a la operación [Copy Blob](/rest/api/storageservices/copy-blob). Para obtener más información sobre el uso conjunto del control de versiones de blobs y la eliminación temporal, vea [Control de versiones de blobs y eliminación temporal](versioning-overview.md#blob-versioning-and-soft-delete).

Los objetos que se han eliminado temporalmente no se ven, salvo que se enumeren de forma explícita.

La eliminación temporal de blobs es compatible con las versiones anteriores por lo que no es preciso realizar ningún cambio en las aplicaciones para aprovechar las ventajas de los mecanismos de protección que se obtienen con esta característica. Sin embargo, la [recuperación de datos](#recovery) incorpora la nueva **Undelete Blob** API.

La eliminación temporal de blobs está disponible para las cuentas de almacenamiento de blobs y las de uso general v2 y v1 nuevas y existentes. Se admiten los tipos de cuenta Estándar y Premium. Sí, la eliminación temporal de blobs se puede realizar en todas las capas de almacenamiento, es decir, la de acceso frecuente, la de acceso esporádico y la de archivo. La eliminación temporal está disponible para discos no administrados, que son blobs en páginas en segundo plano, pero no está disponible para discos administrados.

### <a name="configuration-settings"></a>Parámetros de configuración

Cuando se crea una cuenta, la eliminación temporal está deshabilitada de forma predeterminada. La eliminación temporal también está deshabilitada de forma predeterminada en las cuentas de almacenamiento existentes. Puede habilitar o deshabilitar la eliminación temporal para una cuenta de almacenamiento en cualquier momento.

Al habilitar la eliminación temporal, debe configurar el período de retención. El período de retención indica la cantidad de tiempo durante el que los datos eliminados temporalmente se almacenan y están disponibles para su recuperación. En el caso de los objetos que se eliminan explícitamente, el reloj del período de retención se pone en marcha cuando se eliminan los datos. Sin embargo, en el caso de las instantáneas o versiones eliminadas temporalmente que genera la característica de eliminación temporal cuando se sobrescriben los datos, se pone en marcha cuando se genera la instantánea o la versión. El período de retención puede durar entre 1 y 365 días.

El período de retención de la eliminación temporal se puede cambiar en cualquier momento. El período de retención actualizado solo se aplica a los datos recién eliminados. Los datos eliminados con anterioridad expiran en función del periodo de retención que se configuró en el momento en que se eliminaron. Si intenta eliminar un objeto eliminado temporalmente, su hora de expiración no se verá afectada.

Si deshabilita la eliminación temporal, puede seguir accediendo a los datos eliminados temporalmente de la cuenta de almacenamiento que se guardaron mientras la característica estaba habilitada y recuperarlos.

### <a name="saving-deleted-data"></a>Guardado de los datos eliminados

La eliminación temporal conserva los datos en muchos casos en los que los objetos se eliminan o se sobrescriben.

Si se sobrescribe un blob mediante **Put Blob**, **Put Block List** o **Copy Blob**, se genera automáticamente una versión o instantánea del estado del blob antes de la operación de escritura. Este objeto es invisible, a menos que se muestren explícitamente los objetos eliminados temporalmente. Para aprender a enumerar objetos que se han eliminado temporalmente, consulte la sección [Recuperación](#recovery).

![Diagrama que muestra cómo se almacenan las instantáneas de blobs cuando se sobrescriben con Put Blob, Put Block List o Copy Blob.](media/soft-delete-blob-overview/storage-blob-soft-delete-overwrite.png)

*Los datos eliminados temporalmente son de color gris, mientras que los datos activos están son de color azul. Los últimos datos escritos aparecen debajo de los datos más antiguos. Cuando B0 se sobrescribe con B1, se genera una instantánea de eliminación temporal de B0. Cuando B1 se sobrescribe con B2, se genera una instantánea de eliminación temporal de B1.*

> [!NOTE]  
> La eliminación temporal solo ofrece protección contra las sobrescritura en las operaciones de copia cuando está activada en la cuenta del blob de destino.

> [!NOTE]  
> La eliminación temporal no permite la protección contra escritura de blobs en el nivel de archivo. Si un blob archivado se sobrescribe con blob nuevo en cualquier nivel, el primero expira de forma permanente.

Si se llama a **Delete Blob** en una instantánea, esta se marca como eliminada temporalmente. No se una nueva instantánea.

![Diagrama que muestra cómo las instantáneas de blobs se eliminan temporalmente al usar Delete Blob.](media/soft-delete-blob-overview/storage-blob-soft-delete-explicit-delete-snapshot.png)

*Los datos eliminados temporalmente son de color gris, mientras que los datos activos están son de color azul. Los últimos datos escritos aparecen debajo de los datos más antiguos. Si se llama a **Snapshot Blob**, B0 pasa a ser una instantánea y B1 es el estado activo del blob. Cuando se elimina la instantánea B0, se marca como eliminada temporalmente.*

Cuando se llama a **Delete Blob** en un blob base (cualquier blob que sea en sí mismo una instantánea), dicho blob se marca como eliminado temporalmente. Por coherencia con el comportamiento anterior, si se llama a **Delete Blob** en un blob que tiene instantáneas activas, se produce un error. Sin embargo, si se llama a **Delete Blob** en un blob con instantáneas eliminadas temporalmente, no se produce un error. Si la eliminación temporal está activada, se pueden eliminar un blob y todas sus instantáneas en una sola operación. Al hacerlo, tanto el blob base como las instantáneas de marcan como eliminados temporalmente.

![Diagrama que muestra lo que sucede cuando se llama a Delete blog en un blob de base.](media/soft-delete-blob-overview/storage-blob-soft-delete-explicit-include.png)

*Los datos eliminados temporalmente son de color gris, mientras que los datos activos están son de color azul. Los últimos datos escritos aparecen debajo de los datos más antiguos. En este caso, se realiza una llamada a **Delete Blob** para eliminar B2 y todas las instantáneas asociadas. El blob activo, B2, y todas las instantáneas asociadas se marcan como eliminados temporalmente.*

> [!NOTE]  
> Cuando se sobrescribe un blob eliminado temporalmente, se genera de forma automática una instantánea de eliminación temporal del estado del blob antes de la operación de escritura. El nuevo blob hereda el nivel del blob sobrescrito.

La eliminación temporal no guarda los datos si se eliminan el contenedor o la cuenta, ni tampoco cuando se sobrescriben los metadatos y las propiedades del blob. Para proteger una cuenta de almacenamiento contra cualquier eliminación por error, se puede configurar un bloqueo mediante Azure Resource Manager. Para más información, consulte el artículo de Azure Resource Manager [Bloqueo de recursos para impedir cambios inesperados](../../azure-resource-manager/management/lock-resources.md).

En la tabla siguiente se detalla el comportamiento esperado cuando se activa la eliminación temporal:

| Operación de API REST | Tipo de recurso | Descripción | Cambio en el comportamiento |
|--------------------|---------------|-------------|--------------------|
| [Eliminar](/rest/api/storagerp/StorageAccounts/Delete) | Cuenta | Elimina la cuenta de almacenamiento, incluidos todos los contenedores y blobs que contiene.                           | Sin cambios. Los contenedores y blobs de la cuenta eliminada no se pueden recuperar. |
| [Delete Container](/rest/api/storageservices/delete-container) | Contenedor | Elimina el contenedor, incluidos todos los blobs que contiene. | Sin cambios. Los blobs del contenedor eliminado no se pueden recuperar. |
| [Put Blob](/rest/api/storageservices/put-blob) | Blobs en bloques, de anexión y en páginas | Crea un nuevo blob o reemplaza uno en un contenedor | Si se usa para reemplazar un blob existente, se genera automáticamente una instantánea del estado del blob antes de la llamada. Esto también se aplica a cualquier blob que se haya eliminado temporalmente con anterioridad si, y solo si, se reemplaza por un blob del mismo tipo (en bloques, de anexión o de página). Si se reemplaza por un blob de cualquier otro tipo, todos los datos eliminados temporalmente existentes expirarán de forma permanente. |
| [Delete Blob](/rest/api/storageservices/delete-blob) | Blobs en bloques, de anexión y en páginas | Marca un blob o una instantánea de blob para su eliminación. El blob o la instantánea se eliminan posteriormente, durante la recolección de elementos no utilizados | Si se usa para eliminar una instantánea de blob, esta se marca como eliminada temporalmente. Si se usa para eliminar un blob, este se marca como eliminado temporalmente. |
| [Copy Blob](/rest/api/storageservices/copy-blob) | Blobs en bloques, de anexión y en páginas | Copia un blob de origen en un blob de destino de la misma cuenta de almacenamiento o de otra. | Si se usa para reemplazar un blob existente, se genera automáticamente una instantánea del estado del blob antes de la llamada. Esto también se aplica a cualquier blob que se haya eliminado temporalmente con anterioridad si, y solo si, se reemplaza por un blob del mismo tipo (en bloques, de anexión o de página). Si se reemplaza por un blob de cualquier otro tipo, todos los datos eliminados temporalmente existentes expirarán de forma permanente. |
| [Put Block](/rest/api/storageservices/put-block) | Blobs en bloques | Crea un nuevo bloque que se confirmará como parte de un blob en bloques. | Si se utiliza para confirmar un bloque en un blob que esté activo, no habrá cambio alguno. Si utiliza para confirmar un bloque en un blob que se ha eliminado temporalmente, se crea un nuevo blob y se genera automáticamente una instantánea para capturar el estado del blob eliminado temporalmente. |
| [Put Block List](/rest/api/storageservices/put-block-list) | Blobs en bloques | Confirma un blob mediante la especificación del conjunto de identificadores de bloque que componen el blob en bloques. | Si se usa para reemplazar un blob existente, se genera automáticamente una instantánea del estado del blob antes de la llamada. Esto también se aplica a un blob que se eliminó temporalmente si, y solo si, es un blob en bloques. Si se reemplaza por un blob de cualquier otro tipo, todos los datos eliminados temporalmente existentes expirarán de forma permanente. |
| [Put Page](/rest/api/storageservices/put-page) | Blobs en páginas | Escribe un rango de páginas en un blob en páginas. | Sin cambios. Los datos del blob en páginas que se sobrescriben o se borrar con esta operación no se guardan ni se pueden recuperar. |
| [Append Block](/rest/api/storageservices/append-block) | Blobs de anexión | Escribe un bloque de datos al final de un blob de anexión. | Sin cambios. |
| [Set Blob Properties](/rest/api/storageservices/set-blob-properties) | Blobs en bloques, de anexión y en páginas | Establece los valores de las propiedades del sistema definidas para un blob. | Sin cambios. Las propiedades del blob sobrescribe no se pueden recuperar. |
| [Set Blob Metadata](/rest/api/storageservices/set-blob-metadata) | Blobs en bloques, de anexión y en páginas | Establece los metadatos definidos por el usuario para el blob especificado como uno o varios pares de nombre y valor. | Sin cambios. Los metadatos del blob sobrescrito no se pueden recuperar. |

Es importante tener en cuenta que si se llama a **Put Page** para sobrescribir o borrar los rangos de un blob en páginas no se generarán instantáneas de forma automática. Los discos de máquina virtual tienen el respaldo de los blobs en páginas y usan **Put Page** para escribir los datos.

### <a name="recovery"></a>Recuperación

Si se llama a la operación [Undelete Blob](/rest/api/storageservices/undelete-blob) en un blob base eliminado temporalmente, se restauran tanto el blob como todas las instantáneas asociadas eliminadas temporalmente como activas. Si se llama a la operación **Undelete Blob** en un blob base activo, todas las instantáneas asociadas eliminadas temporalmente se restauran como activas. Cuando las instantáneas se restauran como activas, parecen generadas por el usuario; no sobrescriben el blob base.

Para restaurar un blob en una instantánea eliminada temporalmente, puede llamar a **Undelete Blob** en el blob base. Luego puede copiar la instantánea sobre el blob activo. También puede copiar la instantánea en un blob nuevo.

![Diagrama que muestra lo que sucede cuando se usa Undelete blob.](media/soft-delete-blob-overview/storage-blob-soft-delete-recover.png)

*Los datos eliminados temporalmente son de color gris, mientras que los datos activos están son de color azul. Los últimos datos escritos aparecen debajo de los datos más antiguos. En este caso, **Undelete Blob** se llama en el blob B, con lo que se restaura el blob base, B1, y todas las instantáneas asociadas, aquí simplemente B0, como activas. En el segundo paso, se copia B0 sobre el blob base. Esta operación de copia genera una instantánea de eliminación automática de B1.*

Para ver los blobs y las instantáneas de blob eliminados temporalmente, puede elegir incluir datos eliminados en **List Blobs**. Puede elegir ver solo los blobs base eliminados temporalmente o incluir también las instantáneas de blob eliminadas temporalmente. En todos los datos eliminados temporalmente se puede ver la hora en que se eliminaron, así como el número de días que faltan para que expiren de forma permanente.

### <a name="example"></a>Ejemplo

A continuación se muestra la salida de consola de un script de .NET que carga, sobrescribe, crea instantáneas, elimina y restaura un blob denominado *HelloWorld* cuando la eliminación temporal está activada:

```bash
Upload:
- HelloWorld (is soft deleted: False, is snapshot: False)

Overwrite:
- HelloWorld (is soft deleted: True, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: False)

Snapshot:
- HelloWorld (is soft deleted: True, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: False)

Delete (including snapshots):
- HelloWorld (is soft deleted: True, is snapshot: True)
- HelloWorld (is soft deleted: True, is snapshot: True)
- HelloWorld (is soft deleted: True, is snapshot: False)

Undelete:
- HelloWorld (is soft deleted: False, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: False)

Copy a snapshot over the base blob:
- HelloWorld (is soft deleted: False, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: True)
- HelloWorld (is soft deleted: True, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: False)
```

En la sección [Pasos siguientes](#next-steps) puede encontrar un puntero a la aplicación que generó esta salida.

## <a name="pricing-and-billing"></a>Precios y facturación

Todos los datos eliminados temporalmente se factura con la misma tasa que los datos activos. Sin embargo, los datos que se eliminen permanentemente después del periodo de retención configurado no se cobrarán. Para ver un análisis más profundo de las instantáneas y cómo generan cargos, consulte [Creación de una instantánea de un blob](storage-blob-snapshots.md).

No se le cobrarán las transacciones relacionadas con la generación automática de instantáneas. Las transacciones de **Undelete Blob** se le cobrarán con la tarifa de operaciones de escritura.

Para más información acerca de los precios de Azure Blob Storage en general, consulte la [página de precios de Azure Blob Storage](https://azure.microsoft.com/pricing/details/storage/blobs/).

Cuando activa inicialmente la eliminación temporal, Microsoft recomienda usar un período de retención pequeño para comprender mejor cómo afecta la característica a la facturación.

Habilitar eliminación temporal para datos sobrescritos con frecuencia puede generar mayores cargos por capacidad de almacenamiento y una mayor latencia al enumerar los blobs. Tanto este costo adicional como la latencia se pueden mitigar almacenando los datos que se sobrescriben con frecuencia en una cuenta de almacenamiento independiente en la que la eliminación temporal esté deshabilitada.

## <a name="faq"></a>Preguntas más frecuentes

### <a name="can-i-use-the-set-blob-tier-api-to-tier-blobs-with-soft-deleted-snapshots"></a>¿Se puede usar Set Blob Tier API para almacenar los blobs por niveles con instantáneas eliminadas temporalmente?

Sí. Las instantáneas eliminadas temporalmente permanecerán en el nivel original, pero el blob base se moverá al nivel nuevo.

### <a name="premium-storage-accounts-have-a-per-blob-snapshot-limit-of-100-do-soft-deleted-snapshots-count-toward-this-limit"></a>Las cuentas de almacenamiento Premium tienen un límite de instantáneas de blob de 100. ¿Cuentan las instantáneas eliminadas temporalmente para este límite?

No, no cuentan.

### <a name="if-i-delete-an-entire-account-or-container-with-soft-delete-turned-on-will-all-associated-blobs-be-saved"></a>Si elimino toda una cuenta o un contenedor con la eliminación temporal activada, ¿se guardarán todos los blobs asociados?

No, si elimina toda una cuenta o un contenedor, todos los blobs asociados se eliminarán permanentemente. Para más información sobre cómo proteger una cuenta de almacenamiento de eliminaciones accidentales, consulte [Bloqueo de recursos para impedir cambios inesperados](../../azure-resource-manager/management/lock-resources.md).

### <a name="can-i-view-capacity-metrics-for-deleted-data"></a>¿Puedo ver las métricas de capacidad de los datos eliminados?

Los datos eliminados temporalmente se incluyen como parte de la capacidad total de la cuenta de almacenamiento. Para más información sobre el seguimiento y la supervisión de la capacidad de almacenamiento, consulte [Storage Analytics](../common/storage-analytics.md).

### <a name="can-i-read-and-copy-out-soft-deleted-snapshots-of-my-blob"></a>¿Puedo leer y copiar instantáneas eliminadas temporalmente de mi blob?  

Sí, pero antes debe llamar a Undelete en el blob.

### <a name="is-soft-delete-available-for-virtual-machine-disks"></a>¿Está la eliminación temporal disponible para los discos de máquina virtual?  

La eliminación temporal está disponible para discos no administrados Premium y Estándar, que son blobs en páginas en segundo plano. La eliminación temporal solo le ayudará a recuperar los datos eliminados por las operaciones **Delete Blob**, **Put Blob**, **Put Block List** y **Copy Blob**. Los datos que se sobrescriben con una llamada a **Put Page** no se pueden recuperar.

Una máquina virtual de Azure escribe en un disco no administrado mediante llamadas a **Put Page**, por lo que el uso de la eliminación temporal para deshacer las escrituras en un disco no administrado desde una máquina virtual de Azure no es un escenario admitido.

### <a name="do-i-need-to-change-my-existing-applications-to-use-soft-delete"></a>¿Es preciso cambiar las aplicaciones existentes para usar la eliminación temporal?

Se puede sacar provecho de la eliminación temporal independientemente de la versión de API que se use. Sin embargo, para enumerar y recuperar blobs e instantáneas de blob eliminados temporalmente, será preciso usar la versión del 29-07-2017 de la [API REST de Azure Storage](/rest/api/storageservices/Versioning-for-the-Azure-Storage-Services) o una versión superior. Microsoft recomienda usar siempre la versión más reciente de la API de Azure Storage.

## <a name="next-steps"></a>Pasos siguientes

- [Habilitación de la eliminación temporal para blobs](soft-delete-enable.md)
- [Versiones de blobs (versión preliminar)](versioning-overview.md)

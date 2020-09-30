---
title: Instantáneas de blob
titleSuffix: Azure Storage
description: Comprenda cómo funcionan las instantáneas de blob y cómo se facturan.
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 08/27/2020
ms.author: tamram
ms.subservice: blobs
ms.openlocfilehash: ab4c152f30ab96fe5e221a605a2339c773e32547
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89295414"
---
# <a name="blob-snapshots"></a>Instantáneas de blob

Una instantánea es una versión de solo lectura de un blob que se ha realizado en un momento dado.

> [!NOTE]
> El control de versiones de blobs ofrece una manera mejor de mantener las versiones anteriores de un blob. Para más información, consulte [Control de versiones de blobs](versioning-overview.md).

## <a name="about-blob-snapshots"></a>Sobre las instantáneas de blob

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

Una instantánea de un blob es idéntica a su blob base, salvo que el identificador URI tiene un valor **DateTime** anexado al URI del blob para indicar el momento en que se tomó la instantánea. Por ejemplo, si el identificador URI de blob en páginas es `http://storagesample.core.blob.windows.net/mydrives/myvhd`, el identificador URI de instantánea es similar a `http://storagesample.core.blob.windows.net/mydrives/myvhd?snapshot=2011-03-09T01:42:34.9360000Z`.

> [!NOTE]
> Todas las instantáneas comparten el identificador URI del blob base. La única distinción entre el blob base y la instantánea es el valor **DateTime** anexado.

Un blob puede tener cualquier número de instantáneas. Las instantáneas se conservan hasta que se eliminan de forma explícita, ya sea de forma independiente o como parte de la operación [Eliminar blob](/rest/api/storageservices/delete-blob) del blob de base. Puede enumerar las instantáneas asociadas al blob base para llevar un seguimiento de las instantáneas actuales.

Cuando se crea una instantánea de un blob, las propiedades del sistema se copian en la instantánea con los mismos valores. Los metadatos del blob base también se copian en la instantánea, a menos que especifique metadatos independientes para la instantánea al crearla. Después de crear una instantánea, puede leerla, copiarla o eliminarla, pero no puede modificarla.

Las concesiones asociadas con el blob base no afectan la instantánea. No puede adquirir una concesión en una instantánea.

Se usa un archivo VHD para almacenar el estado y la información actual de un disco de máquina virtual. Puede desconectar un disco desde la máquina virtual o apagándola y, después, realizar una instantánea de su archivo VHD. Puede usar ese archivo de instantánea más adelante para recuperar el archivo VHD en ese momento y volver a crear la máquina virtual.

## <a name="understand-how-snapshots-accrue-charges"></a>Descripción de cómo las instantáneas pueden incrementar los costes

### <a name="important-billing-considerations"></a>Consideraciones importantes sobre facturación

La lista siguiente incluye los puntos clave que hay que tener en cuenta a la hora de crear una instantánea.

- La cuenta de almacenamiento genera cargos para páginas o bloques únicos, bien se encuentren en el blob o en la instantánea. La cuenta no generará gastos adicionales para instantáneas asociadas a un blob hasta que actualice el blob en el que se basan. Después de actualizar el blob base, discrepa de sus instantáneas. Cuando esto sucede, se le cobra por las páginas o bloques únicos en cada blob o instantánea.
- Al reemplazar un bloque en un blob en bloques, ese bloque será considerado como un bloque único a la hora de aplicar cargos. Esto es así incluso aunque el bloque tenga el mismo identificador de bloque y los mismos datos que tiene en la instantánea. Cuando se vuelva a confirmar el bloque, divergirá de su equivalente en cualquier instantánea y se aplicarán cargos por sus datos. Lo mismo sucede con una página de un blob en páginas que se haya actualizado con datos idénticos.
- Al actualizar un blob en bloques mediante la llamada a un método que sobrescribe todo el contenido del blob, se reemplazarán todos los bloques del blob. Si una instantánea está asociada a ese blob, todos los bloques del blob y la instantánea de base divergen ahora y se aplicarán cargos por todos los bloques en ambos blobs. Esto es así incluso si los datos del blob e instantánea de base siguen siendo idénticos.
- Azure Blob service no dispone de medios para determinar si dos bloques contienen datos idénticos. Cada bloque que se carga y confirma se trata como único, incluso si tiene los mismos datos y el mismo identificador de bloque. Dado que los cargos se acumulan en bloques únicos, es importante tener en cuenta que la actualización de un blob que tiene una instantánea genera bloques únicos y cargos adicionales.

### <a name="minimize-costs-with-snapshot-management"></a>Minimización de los costos con la administración de instantáneas

Se recomienda administrar las instantáneas con cuidado para evitar cargos adicionales. El seguimiento de estas prácticas recomendadas le ayudará a minimizar los costos que acarreará el almacenamiento de las instantáneas:

- Elimine y vuelva a crear las instantáneas asociadas a un blob siempre que lo actualice, incluso si lo hace con datos idénticos, a menos que el diseño de la aplicación requiera que se conserven las instantáneas. Si elimina y vuelve a crear las instantáneas del blob, puede estar seguro de que el blob y las instantáneas no van a divergir.
- Si va a mantener instantáneas de un blob, evite llamar a métodos que sobrescriban todo el blob cuando lo actualice. En cambio, actualice el menor número posible de bloques con el fin de reducir al mínimo los costos.

### <a name="snapshot-billing-scenarios"></a>Escenarios de facturación de instantáneas

En las siguientes situaciones, se muestra cómo se ven incrementados los cargos para un blob en bloques y sus instantáneas.

## <a name="pricing-and-billing"></a>Precios y facturación

Crear una instantánea, que es una copia de solo lectura de un blob, puede conllevar cargos adicionales de almacenamiento de datos en la cuenta. Al diseñar una aplicación, es importante tener en cuenta cómo se pueden acumular estos gastos para que pueda de minimizar los costos.

Las versiones de blobs, al igual que las instantáneas de blobs, se facturan con la misma tarifa que los datos activos. La forma en que se facturan las instantáneas depende de si ha establecido explícitamente el nivel del blob base o de cualquiera de sus instantáneas (o versiones). Para más información sobre los niveles de blobs, consulte [Azure Blob Storage: niveles de acceso frecuente, esporádico y de archivo](storage-blob-storage-tiers.md).

Si no ha cambiado el nivel de un blob o de una instantánea, se le facturarán los bloques únicos de datos en ese blob, sus instantáneas y las versiones que pueda tener. Para más información, consulte [Facturación cuando el nivel de blob no se ha establecido explícitamente](#billing-when-the-blob-tier-has-not-been-explicitly-set).

Si ha cambiado el nivel de un blob o de una instantánea, se le facturará todo el objeto, con independencia de que el blob y la instantánea estén en el mismo nivel de nuevo. Para más información, consulte [Facturación cuando el nivel de blob se ha establecido explícitamente](#billing-when-the-blob-tier-has-been-explicitly-set).

Para más información sobre los detalles de facturación de las versiones de blobs, consulte [Control de versiones de blobs](versioning-overview.md).

### <a name="billing-when-the-blob-tier-has-not-been-explicitly-set"></a>Facturación cuando el nivel de blob no se ha establecido explícitamente

Si no ha establecido explícitamente el nivel del blob para un blob base o cualquiera de sus instantáneas, se le cobrará por bloques o páginas únicos en el blob, sus instantáneas y las versiones que pueda tener. Los datos que se comparten entre un blob y sus instantáneas se cobran solo una vez. Cuando se actualiza un blob, los datos de un blob base difieren de los datos almacenados en sus instantáneas y se cobran los datos únicos por bloque o página.

Al reemplazar un bloque en un blob en bloques, ese bloque será considerado como un bloque único a la hora de aplicar cargos. Esto es así incluso aunque el bloque tenga el mismo identificador de bloque y los mismos datos que tiene en la instantánea. Cuando se vuelva a confirmar el bloque, diferirá de su equivalente en la instantánea y se aplicarán cargos por sus datos. Lo mismo sucede con una página de un blob en páginas que se haya actualizado con datos idénticos.

Blob Storage no dispone de medios para determinar si dos bloques contienen datos idénticos. Cada bloque que se carga y confirma se trata como único, incluso si tiene los mismos datos y el mismo identificador de bloque. Dado que los cargos se acumulan con los bloques únicos, es importante tener en cuenta que, cuando se actualiza un blob que tiene instantáneas o versiones, se generarán más bloques únicos y se aplicarán cargos adicionales.

Cuando un blob tiene instantáneas, diseñe las operaciones de actualización en los blobs en bloques para que actualicen el menor número posible de bloques. Las operaciones de escritura que permiten un control más exhaustivo de los bloques son [Put Block](/rest/api/storageservices/put-block) y [Put Block List](/rest/api/storageservices/put-block-list). Por otro lado, la operación [Put Blob](/rest/api/storageservices/put-blob) reemplaza todo el contenido de un blob, lo que puede dar lugar a cargos adicionales.

En los siguientes escenarios se muestra cómo se acumulan los cargos para un blob en bloques y sus instantáneas cuando el nivel del blob no se ha establecido explícitamente.

#### <a name="scenario-1"></a>Escenario 1

En el escenario 1, el blob de base no se ha actualizado después de realizarse la instantánea, así que se aplicarán cargos únicamente para los bloques 1, 2 y 3.

![Diagrama 1 que muestra la facturación de los bloques únicos en el blob base y la instantánea.](./media/snapshots-overview/storage-blob-snapshots-billing-scenario-1.png)

#### <a name="scenario-2"></a>Escenario 2

En el escenario 2, el blob de base está actualizado, pero no así la instantánea. Se ha actualizado el bloque 3 y, aunque contiene los mismos datos y el mismo identificador, no es igual que el bloque 3 de la instantánea. Como resultado, se aplicarán cargos a la cuenta por cuatro bloques.

![Diagrama 2 que muestra la facturación de los bloques únicos en el blob base y la instantánea.](./media/snapshots-overview/storage-blob-snapshots-billing-scenario-2.png)

#### <a name="scenario-3"></a>Escenario 3

En el escenario 3, el blob de base está actualizado, pero no así la instantánea. El bloque 3 se ha reemplazado por el bloque 4 en el blob de base, pero la instantánea sigue reflejando el bloque 3. Como resultado, se aplicarán cargos a la cuenta por cuatro bloques.

![Diagrama 3 que muestra la facturación de los bloques únicos en el blob base y la instantánea.](./media/snapshots-overview/storage-blob-snapshots-billing-scenario-3.png)

#### <a name="scenario-4"></a>Escenario 4

En la situación 4, el blob de base se ha actualizado totalmente y no contiene ninguno de los bloques originales. Como resultado, se aplicarán cargos a la cuenta por la totalidad de los ocho bloques únicos.

![Diagrama 4 que muestra la facturación de los bloques únicos en el blob base y la instantánea.](./media/snapshots-overview/storage-blob-snapshots-billing-scenario-4.png)

> [!TIP]
> Evite llamar a métodos que sobrescriban el blob entero y, en su lugar, actualice bloques individuales para mantener los costos al mínimo.

### <a name="billing-when-the-blob-tier-has-been-explicitly-set"></a>Facturación cuando el nivel del blob se ha establecido explícitamente

Si ha establecido explícitamente el nivel de blob de un blob o una instantánea (o versión), se le cobrará la longitud completa del contenido del objeto en el nuevo nivel, con independencia de si comparte bloques con un objeto del nivel original. También se le cobra la longitud de contenido completo de la versión más antigua del nivel original. En las versiones o instantáneas que permanecen en el nivel original se cobran los bloques únicos que pueden compartir, tal como se describe en [Facturación cuando el nivel de blob no se ha establecido explícitamente](#billing-when-the-blob-tier-has-not-been-explicitly-set).

#### <a name="moving-a-blob-to-a-new-tier"></a>Traslado de un blob a un nuevo nivel

En la tabla siguiente se describe el comportamiento de facturación de un blob o una instantánea cuando se mueven a un nuevo nivel.

| Cuando el nivel de blob se establece explícitamente en... | Se le factura entonces... |
|-|-|
| Un blob base con una instantánea | El blob base del nuevo nivel y la instantánea más antigua del nivel original, además de los bloques únicos de otras versiones.<sup>1</sup> |
| Un blob base con una versión anterior y una instantánea | El blob base del nuevo nivel, la versión más antigua del nivel original y la instantánea más antigua del nivel original, además de los bloques únicos de otras versiones o instantáneas<sup>1</sup>. |
| Una instantánea | La instantánea del nuevo nivel y el blob base del nivel original, además de los bloques únicos de otras instantáneas.<sup>1</sup> |

<sup>1</sup>Si hay otras versiones o instantáneas anteriores que no se han migrado desde su nivel original, esas versiones o instantáneas se cobran en función del número de bloques únicos que contienen, tal como se describe en [Facturación cuando el nivel del blob no se ha establecido explícitamente](#billing-when-the-blob-tier-has-not-been-explicitly-set).

En el diagrama siguiente se muestra cómo se facturan los objetos cuando un blob con instantánea se mueve a un nivel diferente.

:::image type="content" source="media/snapshots-overview/snapshot-billing-tiers.png" alt-text="Diagrama que muestra cómo se facturan los objetos cuando un blob con instantáneas está explícitamente organizado en niveles.":::

El establecimiento explícito del nivel para un blob, una versión o una instantánea no se puede deshacer. Si mueve un blob a un nuevo nivel y luego lo devuelve a su nivel original, se le cobrará la longitud completa del contenido del objeto incluso si comparte bloques con otros objetos del nivel original.

Las operaciones que establecen explícitamente el nivel de un blob, una versión o una instantánea incluyen:

- [Set Blob Tier](/rest/api/storageservices/set-blob-tier)
- [Put Blob](/rest/api/storageservices/put-blob) con el nivel especificado
- [Put Block List](/rest/api/storageservices/put-block-list) con el nivel especificado
- [Copy Blob](/rest/api/storageservices/copy-blob) con el nivel especificado

#### <a name="deleting-a-blob-when-soft-delete-is-enabled"></a>Eliminación de un blob cuando está habilitada la eliminación temporal

Cuando está habilitada la eliminación temporal del blob, si elimina o sobrescribe un blob base cuyo nivel se ha establecido explícitamente, las versiones o instantáneas anteriores del blob eliminado temporalmente se facturan según la longitud del contenido completo. Para más información sobre el uso conjunto del control de versiones de blobs y la eliminación temporal, consulte [Control de versiones de blobs y eliminación temporal](versioning-overview.md#blob-versioning-and-soft-delete).

En la tabla siguiente se describe el comportamiento de facturación de un blob que se elimina temporalmente, en función de si el control de versiones está habilitado o deshabilitado. Cuando el control de versiones está habilitado, se crea una versión cuando un blob se elimina temporalmente. Cuando el control de versiones está deshabilitado, la eliminación temporal de un blob crea una instantánea de eliminación temporal.

| Cuando se sobrescribe un blob base con su nivel establecido explícitamente... | Se le factura entonces... |
|-|-|
| Si la eliminación temporal del blob y el control de versiones están habilitados | Todas las versiones existentes con la longitud del contenido completo con independencia del nivel. |
| Si la eliminación temporal del blob está habilitada, pero el control de versiones está deshabilitado | Todas las instantáneas de eliminación temporal existentes con una longitud de contenido completo con independencia del nivel. |

## <a name="next-steps"></a>Pasos siguientes

- [Control de versiones de blobs](versioning-overview.md)
- [Creación y administración de una instantánea de blob en .NET](snapshots-manage-dotnet.md)
- [Copias de seguridad de discos de máquinas virtuales de Azure no administrados con instantáneas incrementales](../../virtual-machines/windows/incremental-snapshots.md)

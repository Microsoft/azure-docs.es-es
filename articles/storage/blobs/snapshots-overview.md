---
title: Instantáneas de blob
titleSuffix: Azure Storage
description: Aprenda a crear una instantánea de solo lectura de un blob para hacer una copia de seguridad de los datos de blob en un momento determinado.
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 08/19/2020
ms.author: tamram
ms.subservice: blobs
ms.openlocfilehash: 4c6c2774e0d71ec33449565efab797c040aa264f
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88640606"
---
# <a name="blob-snapshots"></a>Instantáneas de blob

Una instantánea es una versión de solo lectura de un blob que se ha realizado en un momento dado.

> [!NOTE]
> El control de versiones de blobs (versión preliminar) ofrece una forma alternativa de mantener versiones anteriores de un blob. Para más información, consulte [Control de versiones de blobs (versión preliminar)](versioning-overview.md).

## <a name="about-blob-snapshots"></a>Sobre las instantáneas de blob

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

Una instantánea de un blob es idéntica a su blob base, salvo que el identificador URI tiene un valor **DateTime** anexado al URI del blob para indicar el momento en que se tomó la instantánea. Por ejemplo, si el identificador URI de blob en páginas es `http://storagesample.core.blob.windows.net/mydrives/myvhd`, el identificador URI de instantánea es similar a `http://storagesample.core.blob.windows.net/mydrives/myvhd?snapshot=2011-03-09T01:42:34.9360000Z`.

> [!NOTE]
> Todas las instantáneas comparten el identificador URI del blob base. La única distinción entre el blob base y la instantánea es el valor **DateTime** anexado.
>

Un blob puede tener cualquier número de instantáneas. Las instantáneas se conservan hasta que se eliminan de forma explícita, ya sea de forma independiente o como parte de la operación [Eliminar blob](/rest/api/storageservices/delete-blob) del blob de base. Puede enumerar las instantáneas asociadas al blob base para llevar un seguimiento de las instantáneas actuales.

Cuando se crea una instantánea de un blob, las propiedades del sistema se copian en la instantánea con los mismos valores. Los metadatos del blob base también se copian en la instantánea, a menos que especifique metadatos independientes para la instantánea al crearla. Después de crear una instantánea, puede leerla, copiarla o eliminarla, pero no puede modificarla.

Las concesiones asociadas con el blob base no afectan la instantánea. No puede adquirir una concesión en una instantánea.

Se usa un archivo VHD para almacenar el estado y la información actual de un disco de máquina virtual. Puede desconectar un disco desde la máquina virtual o apagándola y, después, realizar una instantánea de su archivo VHD. Puede usar ese archivo de instantánea más adelante para recuperar el archivo VHD en ese momento y volver a crear la máquina virtual.

## <a name="understand-how-snapshots-accrue-charges"></a>Descripción de cómo las instantáneas pueden incrementar los costes

Crear una instantánea, que es una copia de solo lectura de un blob, puede conllevar cargos adicionales de almacenamiento de datos en la cuenta. Al diseñar una aplicación, es importante tener en cuenta cómo se pueden acumular estos gastos para que pueda de minimizar los costos.

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

#### <a name="scenario-1"></a>Escenario 1

En el escenario 1, el blob de base no se ha actualizado después de realizarse la instantánea, así que se aplicarán cargos únicamente para los bloques 1, 2 y 3.

![Recursos de Azure Storage](./media/snapshots-overview/storage-blob-snapshots-billing-scenario-1.png)

#### <a name="scenario-2"></a>Escenario 2

En el escenario 2, el blob de base está actualizado, pero no así la instantánea. Se ha actualizado el bloque 3 y, aunque contiene los mismos datos y el mismo identificador, no es igual que el bloque 3 de la instantánea. Como resultado, se aplicarán cargos a la cuenta por cuatro bloques.

![Recursos de Azure Storage](./media/snapshots-overview/storage-blob-snapshots-billing-scenario-2.png)

#### <a name="scenario-3"></a>Escenario 3

En el escenario 3, el blob de base está actualizado, pero no así la instantánea. El bloque 3 se ha reemplazado por el bloque 4 en el blob de base, pero la instantánea sigue reflejando el bloque 3. Como resultado, se aplicarán cargos a la cuenta por cuatro bloques.

![Recursos de Azure Storage](./media/snapshots-overview/storage-blob-snapshots-billing-scenario-3.png)

#### <a name="scenario-4"></a>Escenario 4

En la situación 4, el blob de base se ha actualizado totalmente y no contiene ninguno de los bloques originales. Como resultado, se aplicarán cargos a la cuenta por la totalidad de los ocho bloques únicos.

![Recursos de Azure Storage](./media/snapshots-overview/storage-blob-snapshots-billing-scenario-4.png)

> [!TIP]
> Evite llamar a métodos que sobrescriban el blob entero y, en su lugar, actualice bloques individuales para mantener los costos al mínimo.

## <a name="next-steps"></a>Pasos siguientes

- [Creación y administración de una instantánea de blob en .NET](snapshots-manage-dotnet.md)
- [Copias de seguridad de discos de máquinas virtuales de Azure no administrados con instantáneas incrementales](../../virtual-machines/windows/incremental-snapshots.md)

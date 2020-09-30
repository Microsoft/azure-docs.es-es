---
title: Redundancia de datos
titleSuffix: Azure Storage
description: Descripción de la redundancia de datos en Azure Storage. Los datos de su cuenta de Microsoft Azure Storage se replican para garantizar la durabilidad y la alta disponibilidad.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 08/24/2020
ms.author: tamram
ms.reviewer: artek
ms.subservice: common
ms.openlocfilehash: fbc24db21ee43e3c2aef3d0164e8510a79508fd2
ms.sourcegitcommit: f8d2ae6f91be1ab0bc91ee45c379811905185d07
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89658575"
---
# <a name="azure-storage-redundancy"></a>Redundancia de Azure Storage

Azure Storage siempre almacena varias copias de los datos, con el fin de protegerlos de eventos planeados y no planeados, como errores transitorios del hardware, interrupciones del suministro eléctrico o cortes de la red y desastres naturales masivos. La redundancia garantiza que la cuenta de almacenamiento cumpla el [contrato de nivel de servicio (SLA) para Azure Storage](https://azure.microsoft.com/support/legal/sla/storage/), incluso en caso de errores.

A la hora de decidir qué opción de redundancia es la más adecuada para su escenario, tenga en cuenta la solución intermedia entre los costos más bajos y disponibilidad y durabilidad más altas. Entre los factores que ayudan a determinar qué opción de redundancia debe elegir se incluye:  

- Cómo se replican los datos en la región primaria.
- Si los datos se replicarán en una segunda ubicación que está alejada geográficamente de la región primaria, para protegerse frente a desastres regionales.
- Si la aplicación necesita acceso de lectura a los datos replicados en la región secundaria en caso de que la región primaria deje de estar disponible por cualquier motivo.

## <a name="redundancy-in-the-primary-region"></a>Redundancia en la región primaria

Los datos de una cuenta de Azure Storage siempre se replican tres veces en la región primaria. Azure Storage ofrece dos métodos para replicar los datos en la región primaria:

- El **almacenamiento con redundancia local (LRS)** copia los datos de forma sincrónica tres veces dentro de una única ubicación física en la región primaria. LRS es la opción de replicación menos costosa, pero no se recomienda para las aplicaciones que requieren de alta disponibilidad.
- El **almacenamiento con redundancia de zona (ZRS)** copia los datos de forma sincrónica en tres zonas de disponibilidad de Azure en la región primaria. En el caso de las aplicaciones que requieren de alta disponibilidad, Microsoft recomienda usar ZRS en la región primaria, además de replicación en una región secundaria.

### <a name="locally-redundant-storage"></a>Almacenamiento con redundancia local

El almacenamiento con redundancia local (LRS) replica los datos tres veces dentro de una única ubicación física en la región primaria. LRS ofrece una durabilidad mínima del 99,999999999 % (once nueves) de los objetos en un año determinado.

LRS es la opción de redundancia de costo más bajo y ofrece la menor durabilidad en comparación con otras opciones. LRS protege los datos frente a errores en la estantería de servidores y en la unidad. No obstante, si se produce un desastre como un incendio o una inundación en el centro de datos, es posible que todas las réplicas de una cuenta de almacenamiento con LRS se pierdan o no se puedan recuperar. Para mitigar este riesgo, Microsoft recomienda el uso del [almacenamiento con redundancia de zona](#zone-redundant-storage) (ZRS), el [almacenamiento con redundancia geográfica](#geo-redundant-storage) (GRS) o el [almacenamiento con redundancia de zona geográfica](#geo-zone-redundant-storage) (GZRS).

Las solicitudes de escritura a una cuenta de almacenamiento que usa LRS se producen de forma sincrónica. Las operaciones de escritura se devuelven correctamente solo después de que los datos se escriben en las tres réplicas.

LRS es una buena opción para los siguientes escenarios:

- Si la aplicación almacena datos que se pueden reconstruir fácilmente si se produce una pérdida de datos, puede optar por LRS.
- Si la aplicación está restringida a la replicación de datos solo dentro de un país o región debido a requisitos de gobernanza de datos, puede optar por LRS. En algunos casos, las regiones emparejadas en las que los datos se replican geográficamente pueden estar en otro país o región. Para más información sobre las regiones emparejadas, consulte [Regiones de Azure](https://azure.microsoft.com/regions/).

### <a name="zone-redundant-storage"></a>Almacenamiento con redundancia de zona

El almacenamiento con redundancia de zona (ZRS) replica los datos de Azure Storage de forma sincrónica en tres zonas de disponibilidad de Azure en la región primaria. Cada zona de disponibilidad es una ubicación física individual con alimentación, refrigeración y redes independientes. ZRS proporciona a los objetos de datos de Azure Storage una durabilidad de al menos el 99,9999999999 % (doce nueves) durante un año determinado.

Con ZRS, los datos son accesibles para las operaciones de escritura y lectura incluso si una zona deja de estar disponible. Si alguna zona deja de estar disponible, Azure realiza las actualizaciones de la red, como el redireccionamiento de DNS. Estas actualizaciones pueden afectar a la aplicación si se accede a los datos antes de que se completen dichas actualizaciones. Al diseñar aplicaciones para ZRS, siga los procedimientos para el control de errores transitorios, incluida la implementación de directivas de reintentos con retroceso exponencial.

Las solicitudes de escritura a una cuenta de almacenamiento que usa ZRS se producen de forma sincrónica. Las operaciones de escritura se devuelven correctamente solo después de que los datos se escriben en todas las réplicas de las tres zonas de disponibilidad.

Microsoft recomienda el uso de ZRS en la región primaria para las situaciones que requieren coherencia, durabilidad y alta disponibilidad. También se recomienda usarlo si se desea restringir una aplicación para que replique los datos solo dentro de un país o región debido a los requisitos de la gobernanza de datos.

ZRS ofrece un rendimiento excelente, una latencia baja y resistencia para los datos si dicha región deja de estar disponible temporalmente. No obstante, ZRS por sí solo podría no proteger los datos frente a un desastre regional en el que varias zonas resulten afectadas permanentemente. Para protegerse frente a desastres regionales, Microsoft recomienda usar [almacenamiento con redundancia de zona geográfica](#geo-zone-redundant-storage) (GZRS), que usa ZRS en la región primaria y también replica geográficamente los datos en una región secundaria.

En la tabla siguiente se muestran los tipos de cuentas de almacenamiento que admiten ZRS en cada región:

| Tipo de cuenta de almacenamiento | Regiones admitidas | Servicios admitidos |
|--|--|--|
| Uso general v2<sup>1</sup> | Sudeste de Asia<br /> Este de Australia<br /> Norte de Europa<br />  Oeste de Europa<br /> Centro de Francia<br /> Japón Oriental<br /> Norte de Sudáfrica<br /> Sur de Reino Unido<br /> Centro de EE. UU.<br /> Este de EE. UU.<br /> Este de EE. UU. 2<br /> Oeste de EE. UU. 2 | Blobs en bloques<br /> Blobs en páginas<sup>2</sup><br /> Recursos compartidos de archivos (estándar)<br /> Tablas<br /> Colas<br /> |
| BlockBlobStorage<sup>1</sup> | Sudeste de Asia<br /> Este de Australia<br /> Norte de Europa<br /> Oeste de Europa<br /> Este de EE. UU. <br /> Este de EE. UU. 2 <br /> Oeste de EE. UU. 2| Solo blobs en bloques Premium |
| FileStorage | Sudeste de Asia<br /> Este de Australia<br /> Norte de Europa<br /> Oeste de Europa<br /> Este de EE. UU. <br /> Este de EE. UU. 2 <br /> Oeste de EE. UU. 2 | Solo recursos compartidos de archivos Premium |

<sup>1</sup> El nivel de archivo no se admite actualmente en las cuentas de ZRS.<br />
<sup>2</sup> Las cuentas de almacenamiento que contienen discos administrados de Azure para máquinas virtuales siempre usan almacenamiento con redundancia local. Los discos no administrados de Azure también deben usar almacenamiento con redundancia local. Es posible crear una cuenta de almacenamiento para discos no administrados de Azure que use almacenamiento con redundancia geográfica, pero no se recomienda debido a los posibles problemas de coherencia en la replicación geográfica asincrónica. Ni los discos administrados ni los no administrados admiten ZRS o GZRS. Para más información sobre los discos administrados, consulte [Precios de Azure Managed Disks](https://azure.microsoft.com/pricing/details/managed-disks/).

Para obtener información sobre qué regiones admiten ZRS, consulte **Soporte técnico de servicios por región** en [¿Qué son las zonas de disponibilidad en Azure?](../../availability-zones/az-overview.md).

## <a name="redundancy-in-a-secondary-region"></a>Redundancia en una región secundaria

En el caso de las aplicaciones que requieren de alta disponibilidad, puede optar por copiar los datos de la cuenta de almacenamiento en una región secundaria que esté a cientos de kilómetros de distancia de la región primaria. Si la cuenta de almacenamiento se copia a una región secundaria, sus datos se mantienen incluso ante un apagón regional completo o un desastre del cual la región primaria no se puede recuperar.

Al crear una cuenta de almacenamiento, seleccione la región principal de la cuenta. La región secundaria emparejada se determina según la región primaria y no es posible cambiarla. Para obtener más información sobre las regiones compatibles con Azure, consulte [Regiones de Azure](https://azure.microsoft.com/global-infrastructure/regions/).

Azure Storage ofrece dos opciones para copiar los datos a una región secundaria:

- El **almacenamiento con redundancia geográfica (GRS)** copia los datos de forma sincrónica tres veces dentro de una única ubicación física en la región primaria mediante LRS. Luego copia los datos de forma asincrónica en una única ubicación física en la región secundaria.
- El **almacenamiento con redundancia de zona geográfica (GZRS)** copia los datos de forma sincrónica en tres zonas de disponibilidad de Azure en la región primaria mediante ZRS. Luego copia los datos de forma asincrónica en una única ubicación física en la región secundaria.

La principal diferencia entre GRS y GZRS es la forma en que los datos se replican en la región primaria. Dentro de la región secundaria, los datos siempre se replican de forma sincrónica tres veces mediante LRS. LRS en la región secundaria protege los datos frente a errores de hardware.

Con GRS o GZRS, los datos de la región secundaria no están disponible para el acceso de lectura o escritura a menos que suceda una conmutación por error en la región secundaria. Para obtener acceso de lectura a la región secundaria, configure la cuenta de almacenamiento para usar el almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) o el almacenamiento con redundancia de zona geográfica con acceso de lectura (RA-GZRS). Para más información, consulte [Acceso de lectura a los datos de la región secundaria](#read-access-to-data-in-the-secondary-region).

Si la región primaria deja de estar disponible, puede conmutar por error a la región secundaria. Una vez completada la conmutación por error, la región secundaria se convierte en la región primaria y se pueden leer y escribir datos de nuevo. Para más información sobre la recuperación ante desastres y el método de conmutación por error a la región secundaria, consulte [Recuperación ante desastres y conmutación por error de la cuenta](storage-disaster-recovery-guidance.md).

> [!IMPORTANT]
> Dado que los datos se replican en la región secundaria de forma asincrónica, un error que afecte a la región primaria puede producir la pérdida de datos si no se puede recuperar dicha región. El intervalo entre las escrituras más recientes en la región primaria y la última escritura en la región secundaria se conoce como objetivo de punto de recuperación (RPO). El RPO indica momento concreto en que se pueden recuperar los datos. Normalmente, Azure Storage tiene un RPO inferior a 15 minutos, aunque actualmente no hay ningún contrato de nivel de servicio sobre el tiempo que se tarda en replicar los datos en la región secundaria.

### <a name="geo-redundant-storage"></a>Almacenamiento con redundancia geográfica

El almacenamiento con redundancia geográfica (GRS) copia los datos de forma sincrónica tres veces dentro de una única ubicación física en la región primaria mediante LRS. Después, copia los datos de forma asincrónica en una única ubicación física de una región secundaria que se encuentra a cientos de miles de kilómetros de distancia de la región primaria. GRS proporciona a los objetos de datos de Azure Storage una durabilidad de al menos el 99,99999999999999 % (dieciséis nueves) durante un año determinado.

Una operación se escritura se confirma primero en la ubicación principal y se replica mediante LRS. Después, la actualización se replica de manera asincrónica en la región secundaria. Cuando los datos se escriben en la ubicación secundaria, también se replican dentro de esa ubicación con LRS.

### <a name="geo-zone-redundant-storage"></a>Almacenamiento con redundancia de zona geográfica

El almacenamiento con redundancia de zona geográfica (GZRS) combina la alta disponibilidad que proporciona la redundancia entre zonas de disponibilidad con la protección frente a interrupciones regionales que proporciona la replicación geográfica. Los datos de una cuenta de almacenamiento de GZRS se almacenan en tres [zonas de disponibilidad de Azure](../../availability-zones/az-overview.md) en la región primaria y también se replican en una región geográfica secundaria para protegerlos frente a desastres regionales. Microsoft recomienda el uso de GZRS en aplicaciones que requieran de coherencia, durabilidad y disponibilidad máximas, además de rendimiento excelente y resistencia para la recuperación ante desastres.

Con una cuenta de almacenamiento de GZRS, puede seguir leyendo y escribiendo datos si una zona de disponibilidad deja de estar disponible o es irrecuperable. Además, los datos se mantienen en caso de un apagón completo de una región o de un desastre tras el que la región primaria no se puede recuperar. El almacenamiento con redundancia de zona geográfica (GZRS) está diseñado para proporcionar una durabilidad mínima del 99,99999999999999 % (dieciséis nueves) de los objetos en un año determinado.

Solo las cuentas de almacenamiento de uso general v2 son compatibles con GZRS y RA-GZRS. Para más información sobre los tipos de cuentas de almacenamiento, consulte [Introducción a las cuentas de Azure Storage](storage-account-overview.md). GZRS y RA-GZRS admiten blobs en bloques, blobs en páginas (salvo para discos VHD), archivos, tablas y colas.

GZRS y RA-GZRS se admiten en las siguientes regiones:

- Sudeste de Asia
- Norte de Europa
- Oeste de Europa
- Japón Oriental
- Sur de Reino Unido
- Centro de EE. UU.
- Este de EE. UU.
- Este de EE. UU. 2
- Oeste de EE. UU. 2

Para más información sobre los precios, consulte los detalles de precios de [blobs](https://azure.microsoft.com/pricing/details/storage/blobs), [archivos](https://azure.microsoft.com/pricing/details/storage/files/), [colas](https://azure.microsoft.com/pricing/details/storage/queues/)y [tablas](https://azure.microsoft.com/pricing/details/storage/tables/).

## <a name="read-access-to-data-in-the-secondary-region"></a>Acceso de lectura a los datos de la región secundaria

El almacenamiento con redundancia geográfica (con GRS o GZRS) replica los datos en otra ubicación física de la región secundaria para protegerlos frente a los apagones regionales. Sin embargo, los datos están disponibles para su lectura solo si el cliente o Microsoft inician una conmutación por error de la región primaria a la secundaria. Cuando habilita el acceso de lectura a la región secundaria, los datos están disponibles para leerse en todo momento, incluso en una situación en que la región primaria no esté disponible. Para obtener acceso de lectura a la región secundaria, habilite el almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) o el almacenamiento con redundancia de zona geográfica con acceso de lectura (RA-GZRS).

> [!NOTE]
> Azure Files no admite el almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) ni el almacenamiento con redundancia de zona geográfica con acceso de lectura (RA-GZRS).

### <a name="design-your-applications-for-read-access-to-the-secondary"></a>Diseño de aplicaciones para el acceso de lectura a la región secundaria

Si la cuenta de almacenamiento está configurada para usar el acceso de lectura a la región secundaria, puede diseñar sus aplicaciones para que fácilmente pasen a leer datos de la región secundaria si la región primaria deja de estar disponible por cualquier motivo. 

La región secundaria siempre está disponible para el acceso de lectura después de habilitar RA-GRS o RA-GRS, por lo que puede probar la aplicación de antemano para asegurarse de que leerá desde la región secundaria si se produce una interrupción. Para más información sobre cómo diseñar aplicaciones para alta disponibilidad, consulte [Uso de redundancia geográfica para diseñar aplicaciones de alta disponibilidad](geo-redundant-design.md).

Cuando está habilitado el acceso de lectura a la región secundaria, la aplicación se puede leer desde el punto de conexión secundario y desde el punto de conexión primario. El punto de conexión secundario anexa el sufijo *–secondary* al nombre de la cuenta. Por ejemplo, si el punto de conexión primario de Blob Storage es `myaccount.blob.core.windows.net`, el punto de conexión secundario es `myaccount-secondary.blob.core.windows.net`. Las claves de acceso de la cuenta son iguales para los puntos de conexión primario y secundario.

### <a name="check-the-last-sync-time-property"></a>Comprobación de la propiedad Hora de la última sincronización

Dado que los datos se replican en la región secundaria de forma asincrónica, la región secundaria suele ir por detrás de la región primaria. Si se produce un error en la región primaria, es probable que todas las operaciones de escritura en esta región no se hayan replicado todavía en la secundaria.

Para determinar qué operaciones de escritura se han replicado en la región secundaria, la aplicación puede comprobar la propiedad **Hora de la última sincronización** de la cuenta de almacenamiento. Todas las operaciones de escritura escritas en la región primaria antes de la hora de la última sincronización se han replicado correctamente en la región secundaria, lo que significa que están disponibles para leerse desde la región secundaria. Las operaciones de escritura escritas en la región primaria después de la hora de la última sincronización puede que se hayan replicado o no en la región secundaria, lo que significa que es posible que no estén disponibles para las operaciones de lectura.

Puede consultar el valor de la propiedad **Hora de la última sincronización** mediante Azure PowerShell, la CLI de Azure o una de las bibliotecas cliente de Azure Storage. La propiedad **Hora de la última sincronización** es un valor de fecha y hora GMT. Para obtener más información, consulte [Comprobación de la propiedad Hora de la última sincronización de una cuenta de almacenamiento](last-sync-time-get.md).

## <a name="summary-of-redundancy-options"></a>Resumen de las opciones de redundancia

Las tablas de las siguientes secciones resumen las opciones de redundancia disponibles para Azure Storage

### <a name="durability-and-availability-parameters"></a>Parámetros de durabilidad y disponibilidad

En la tabla siguiente se describen los parámetros clave de cada opción de redundancia:

| Parámetro | LRS | ZRS | GRS/RA-GRS | GZRS/RA-GZRS |
|:-|:-|:-|:-|:-|
| Porcentaje de durabilidad de los objetos a lo largo de un año determinado<sup>1</sup> | al menos 99,999999999 % (once nueves) | al menos 99,9999999999 % (doce nueves) | Como mínimo 99,99999999999999 % (dieciséis nueves) | Como mínimo 99,99999999999999 % (dieciséis nueves) |
| SLA de disponibilidad para las solicitudes de lectura<sup>1</sup> | Al menos un 99,9 % (99 % para el nivel de acceso esporádico) | Al menos un 99,9 % (99 % para el nivel de acceso esporádico) | Al menos un 99,9 % (99 % para el nivel de acceso esporádico) para GRS<br /><br />Al menos un 99,9 % (99,99 % para el nivel de acceso esporádico) para RA-GRS | Al menos un 99,9 % (99 % para el nivel de acceso esporádico) para GZRS<br /><br />Al menos un 99,9 % (99,99 % para el nivel de acceso esporádico) para RA-GZRS |
| SLA de disponibilidad para las solicitudes de escritura<sup>1</sup> | Al menos un 99,9 % (99 % para el nivel de acceso esporádico) | Al menos un 99,9 % (99 % para el nivel de acceso esporádico) | Al menos un 99,9 % (99 % para el nivel de acceso esporádico) | Al menos un 99,9 % (99 % para el nivel de acceso esporádico) |

<sup>1</sup> Para más información sobre las garantías de durabilidad y disponibilidad de Azure Storage, consulte el [Acuerdo de Nivel de Servicio de Azure Storage](https://azure.microsoft.com/support/legal/sla/storage/).

### <a name="durability-and-availability-by-outage-scenario"></a>Durabilidad y disponibilidad por escenario de interrupción

En la tabla siguiente se indica si los datos son duraderos y están disponibles en un escenario determinado, según el tipo de redundancia vigente para la cuenta de almacenamiento:

| Escenario de interrupción | LRS | ZRS | GRS/RA-GRS | GZRS/RA-GZRS |
|:-|:-|:-|:-|:-|
| Un nodo de un centro de datos deja de estar disponible | Sí | Sí | Sí | Sí |
| Un centro de datos completo (de zona o no de zona) deja de estar disponible | No | Sí | Sí<sup>1</sup> | Sí |
| Se produce una interrupción en toda la región en la región primaria | No | No | Sí<sup>1</sup> | Sí<sup>1</sup> |
| Hay disponible acceso de lectura a la región secundaria si la región primaria deja de estar disponible | No | No | Sí (con RA-GRS) | Sí (con RA-GZRS) |

<sup>1</sup> Se requiere la conmutación por error de cuenta para restaurar la disponibilidad de escritura si la región primaria deja de estar disponible. Para más información, consulte [Recuperación ante desastres y conmutación por error de la cuenta de almacenamiento](storage-disaster-recovery-guidance.md).

### <a name="supported-storage-account-types"></a>Tipos de cuenta de almacenamiento admitidos

En la tabla siguiente se muestran las opciones de redundancia que admite cada tipo de cuenta de almacenamiento. Para más información sobre los tipos de cuentas de almacenamiento, consulte [Introducción a las cuentas de Storage](storage-account-overview.md).

| LRS | ZRS | GRS/RA-GRS | GZRS/RA-GZRS |
|:-|:-|:-|:-|
| Uso general v2<br /> Uso general v1<br /> Almacenamiento de blobs en bloques<br /> Blob Storage<br /> File Storage | Uso general v2<br /> Almacenamiento de blobs en bloques<br /> File Storage | Uso general v2<br /> Uso general v1<br /> Blob Storage | Uso general v2 |

Todos los datos de todas las cuentas de almacenamiento se copian según la opción de redundancia de la cuenta de almacenamiento. Se copian los objetos, incluidos los blobs en bloques, blobs en anexos, blobs en páginas, colas, tablas y archivos. Se copian los datos de todos los niveles de servicio, incluido el nivel de archivo. Para más información sobre los niveles de blobs, consulte [Azure Blob Storage: niveles de acceso frecuente, esporádico y de archivo](../blobs/storage-blob-storage-tiers.md).

Para más información sobre los precios de las diferentes opciones de redundancia, consulte [Precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/).

> [!NOTE]
> Azure Premium Disk Storage solo admite actualmente almacenamiento con redundancia local (LRS). Las cuentas de almacenamiento de blobs en bloques admiten el almacenamiento con redundancia local (LRS) y el almacenamiento con redundancia de zona (ZRS) en determinadas regiones.

## <a name="data-integrity"></a>Integridad de datos

Azure Storage comprueba con regularidad la integridad de los datos almacenados mediante pruebas de redundancia cíclica (CRC). Si se detectan datos dañados, se reparan con datos redundantes. Azure Storage también calcula las sumas de comprobación de todo el tráfico de red para detectar daños en los paquetes de datos al almacenar o recuperar datos.

## <a name="see-also"></a>Consulte también

- [Comprobación de la propiedad Hora de la última sincronización de una cuenta de almacenamiento](last-sync-time-get.md)
- [Cambio de la opción de redundancia para una cuenta de almacenamiento](redundancy-migration.md)
- [Uso de redundancia geográfica para diseñar aplicaciones de alta disponibilidad](geo-redundant-design.md)
- [Recuperación ante desastres y conmutación por error de la cuenta de almacenamiento](storage-disaster-recovery-guidance.md)

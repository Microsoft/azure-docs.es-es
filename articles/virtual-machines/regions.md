---
title: Regiones de Azure
description: Obtención de información sobre las regiones para ejecutar máquinas virtuales en Azure.
author: cynthn
ms.author: cynthn
ms.service: virtual-machines
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 05/28/2019
ms.openlocfilehash: 094b78fa35da451101f4f5664cca46292d78d146
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88650560"
---
# <a name="regions-for-virtual-machines-in-azure"></a>Regiones para las máquinas virtuales de Azure

Es importante saber cómo y donde operan las máquinas virtuales (VM) en Azure, así como las opciones para maximizar el rendimiento, la disponibilidad y la redundancia. Este artículo proporciona una visión general de las características de disponibilidad y redundancia de Azure.


## <a name="what-are-azure-regions"></a>¿Cuáles son las regiones de Azure?
Azure funciona en varios centros de datos del mundo. Estos centros de datos se agrupan en regiones geográficas, proporcionándole flexibilidad al elegir dónde crear sus aplicaciones. 

Los recursos de Azure se crean en regiones geográficas definidas como 'Oeste de EE. UU.', 'Norte de Europa' o 'Sudeste de Asia'. Puede revisar la [lista de regiones y sus ubicaciones](https://azure.microsoft.com/regions/). En cada región existen varios centros de datos que proporcionan redundancia y disponibilidad. Este enfoque proporciona flexibilidad al diseñar aplicaciones para crear máquinas virtuales más cercanas a los usuarios y satisfacer cualquier propósito legal, de cumplimiento normativo o fiscal.

## <a name="special-azure-regions"></a>Regiones de Azure especiales
Azure tiene algunas regiones especiales que se pueden usar al crear las aplicaciones, en lo referente al cumplimiento normativo o a aspectos legales. Entre dichas regiones se incluyen:

* **US Gov Virginia** e **US Gov Iowa**
  * Una instancia física y lógica con aislamiento de red de Azure para asociados y agencias de la administración pública de EE. UU., operada por personal estadounidense seleccionado con rigor. Incluye certificaciones de cumplimiento adicionales como [FedRAMP](https://www.microsoft.com/en-us/TrustCenter/Compliance/FedRAMP) y [DISA](https://www.microsoft.com/en-us/TrustCenter/Compliance/DISA). Más información sobre [Azure Government](https://azure.microsoft.com/features/gov/).
* **Este de China** y **Norte de China**
  * Estas regiones están disponibles gracias a una exclusiva asociación entre Microsoft y 21Vianet, por el cual Microsoft no mantiene directamente los centros de datos. Obtenga más información sobre [Azure China 21Vianet](https://www.windowsazure.cn/).
* **Сentro de Alemania** y **Nordeste de Alemania**
  * Estas regiones están disponibles mediante un modelo de administrador de datos con el cual los datos del cliente permanecen en Alemania bajo el control de T-Systems, una empresa perteneciente a Deutsche Telekom que actúa como administrador de datos en Alemania.

## <a name="region-pairs"></a>Pares de región
Cada región de Azure se empareja con otra región de la misma zona geográfica (por ejemplo, EE. UU., Europa o Asia). Este enfoque permite la replicación de recursos, como el almacenamiento de máquina virtual, en una zona geográfica que debe reducir la probabilidad de desastres naturales, disturbios civiles, cortes del suministro eléctrico o interrupciones de la red física que afectan simultáneamente a ambas regiones. Entre las ventajas adicionales de los pares de región se incluyen:

* En caso de producirse una interrupción de Azure más amplia, una región tiene prioridad por cada pareja para ayudar a reducir el tiempo de restauración de las aplicaciones. 
* Las actualizaciones planeadas de Azure se implementan una a una en regiones emparejadas para minimizar el tiempo de inactividad y el riesgo de interrupción de la aplicación.
* Los datos siguen residiendo en la misma zona geográfica que su pareja (excepto Sur de Brasil) con fines de jurisdicción fiscal y de aplicación de la ley.

Entre los ejemplos de pares de región se incluyen:

| Principal | Secundario |
|:--- |:--- |
| Oeste de EE. UU. |Este de EE. UU. |
| Norte de Europa |Oeste de Europa |
| Sudeste de Asia |Este de Asia |

Puede consultar en su totalidad la [lista de pares regionales aquí](../best-practices-availability-paired-regions.md#what-are-paired-regions).

## <a name="feature-availability"></a>Disponibilidad de características
Algunos servicios o características de las máquinas virtuales solo están disponibles en determinadas regiones, como, por ejemplo, tipos de almacenamiento o tamaños de VM específicos. También hay algunos servicios globales de Azure que no requieren que seleccione una región concreta, como [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md), [Traffic Manager](../traffic-manager/traffic-manager-overview.md) o [DNS de Azure](../dns/dns-overview.md). Para obtener ayuda con el diseño del entorno de la aplicación, puede comprobar la [disponibilidad de servicios de Azure en cada región](https://azure.microsoft.com/regions/#services). También puede [consultar mediante programación los tamaños de máquina virtual admitidos y las restricciones de cada región](../azure-resource-manager/templates/error-sku-not-available.md).

## <a name="storage-availability"></a>Disponibilidad de almacenamiento
Es importante comprender las regiones y zonas geográficas de Azure a la hora de considerar las opciones de replicación de almacenamiento disponibles. Según el tipo de almacenamiento, tendrá opciones de replicación diferentes.

**Azure Managed Disks**
* Almacenamiento con redundancia local (LRS)
  * Replica los datos tres veces dentro de la región en la que creó su cuenta de almacenamiento.

**Discos basados en cuentas de almacenamiento**
* Almacenamiento con redundancia local (LRS)
  * Replica los datos tres veces dentro de la región en la que creó su cuenta de almacenamiento.
* Almacenamiento con redundancia de zona (ZRS)
  * Replica sus datos tres veces entre dos o tres instalaciones, ya sea dentro de una sola región o entre dos regiones.
* Almacenamiento con redundancia geográfica (GRS)
  * Replica sus datos en una región secundaria que se encuentra a cientos de kilómetros de distancia de la región primaria.
* Almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS).
  * Replica sus datos en una región secundaria, al igual que con GRS, pero también proporciona acceso de solo lectura a los datos en la ubicación secundaria.

La tabla siguiente proporciona una breve descripción de las diferencias entre los tipos de replicación de almacenamiento:

| Estrategia de replicación | LRS | ZRS | GRS | RA-GRS |
|:--- |:--- |:--- |:--- |:--- |
| Los datos se replican entre varias instalaciones |No |Sí |Sí |Sí |
| Los datos se pueden leer tanto desde la ubicación secundaria como desde la ubicación principal. |No |No |No |Sí |
| Cantidad de copias de datos mantenidas en nodos independientes |3 |3 |6 |6 |

Puede obtener más información sobre las [opciones de replicación de Azure Storage aquí](../storage/common/storage-redundancy.md). Para más información acerca de los discos administrados, consulte [Azure Managed Disks overview](./managed-disks-overview.md) (Introducción a los discos administrados de Azure).

### <a name="storage-costs"></a>Costos de almacenamiento
Los precios varían según el tipo de almacenamiento y la disponibilidad que seleccione.

**Azure Managed Disks**
* Las copias de seguridad de Managed Disks premium se realizan en unidades de estado sólido (SSD) y las de Managed Disks estándar en discos de rotación normales. Los discos administrados premium y estándar se cobran en función de la capacidad aprovisionada para el disco.

**Discos no administrados**
* Premium Storage está respaldado por unidades de estado sólido (SSD) y se cobra según la capacidad del disco.
* El almacenamiento estándar está respaldado por los discos giratorios habituales y se cobra según la capacidad en uso y la disponibilidad de almacenamiento deseada.
  * Para RA-GRS, existe un cargo adicional por la transferencia de datos de replicación geográfica por el ancho de banda de replicar dichos datos en otra región de Azure.

Consulte [Precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/) para más información sobre los diferentes tipos de almacenamiento y opciones de disponibilidad.

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte [Regiones de Azure](https://azure.microsoft.com/global-infrastructure/regions/).
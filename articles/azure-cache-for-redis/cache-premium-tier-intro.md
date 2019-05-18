---
title: Introducción a Azure Cache for Redis de nivel Prémium | Microsoft Docs
description: Aprenda a crear y a administrar la persistencia de Redis, la agrupación en clústeres de Redis y la compatibilidad de red virtual para las instancias de Azure Cache for Redis de nivel Prémium.
services: cache
documentationcenter: ''
author: yegu-ms
manager: jhubbard
editor: ''
ms.assetid: 30f46f9f-e6ec-4c38-a8cc-f9d4444856e5
ms.service: cache
ms.workload: tbd
ms.tgt_pltfrm: cache
ms.devlang: na
ms.topic: article
ms.date: 07/05/2017
ms.author: yegu
ms.openlocfilehash: 6960c21091e0bc01c198e713c0c276984566ac41
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65786078"
---
# <a name="introduction-to-the-azure-cache-for-redis-premium-tier"></a>Introducción a Azure Cache for Redis de nivel Prémium
Azure Cache for Redis es una memoria caché distribuida y administrada que ayuda a compilar aplicaciones muy útiles y altamente escalables mediante el acceso ultrarrápido a los datos. 

Premium es un nuevo nivel destinado a las empresas que incluye todas las características del nivel Estándar y otras adicionales, como un mejor rendimiento, cargas de trabajo más grandes, recuperación ante desastres, importación/exportación y seguridad mejorada. Siga leyendo para obtener más información acerca de las características adicionales de la memoria caché del nivel Premium.

## <a name="better-performance-compared-to-standard-or-basic-tier"></a>Mejor rendimiento en comparación con el nivel Estándar o Básico
**Mejor rendimiento respecto a los niveles Standard o Basic.**  Las memorias caché de nivel Premium se implementan en hardware con procesadores más rápidos que ofrece un mejor rendimiento en comparación con el nivel Standard o Basic. Además, el rendimiento de dichas memorias caché es superior y sus latencias son más bajas. 

**El rendimiento de una memoria caché de nivel Premium es superior al de una memoria caché de nivel Standard del mismo tamaño .**  Por ejemplo, el rendimiento de una memoria caché de 53 GB P4 (Premium) es de 250 000 solicitudes por segundo, en comparación con 150 000 para una memoria C6 (Standard).

Para más información sobre el tamaño, la transferencia y el ancho de banda de las memorias caché de nivel Prémium, consulte el artículo [Azure Cache for Redis FAQ](cache-faq.md#what-azure-cache-for-redis-offering-and-size-should-i-use) (Preguntas más frecuentes de Azure Cache for Redis).

## <a name="redis-data-persistence"></a>Persistencia de los datos en Redis
El nivel Premium permite conservar los datos de la memoria caché en una cuenta de Azure Storage. En las memorias caché de nivel Basic o Standard, todos los datos se almacenan únicamente en la memoria. En caso de problemas con la infraestructura subyacente, podría producirse una pérdida de los datos. Se recomienda usar la característica de persistencia de datos de Redis en el nivel Premium para aumentar la resistencia contra la pérdida de datos. Azure Cache for Redis ofrece las opciones RDB y AOF (próximamente) en la [persistencia de Redis](https://redis.io/topics/persistence). 

Para instrucciones sobre cómo configurar la persistencia, consulte [Configuración de la persistencia para Azure Cache for Redis de nivel Prémium](cache-how-to-premium-persistence.md).

## <a name="redis-cluster"></a>Clúster Redis
Si desea crear memorias caché de más de 53 GB o particionar los datos entre varios nodos de Redis, puede usar la agrupación en clústeres Redis, disponible en el nivel Premium. Cada nodo consta de un par de cachés principal-réplica administrado por Azure para ofrecer una alta disponibilidad. 

**La agrupación en clústeres de Redis proporciona una escalabilidad y un rendimiento máximos.** El rendimiento aumenta de manera lineal a medida que aumenta el número de particiones (nodos) del clúster. P. ej. Si se crea un clúster P4 de 10 particiones, el rendimiento posible es de 250 000 * 10 = 2,5 millones de solicitudes por segundo. Consulte el artículo [Azure Cache for Redis FAQ](cache-faq.md#what-azure-cache-for-redis-offering-and-size-should-i-use) (Preguntas más frecuentes de Azure Cache for Redis) para más detalles sobre el tamaño, rendimiento y ancho de banda de las memorias caché Prémium.

Para empezar a usar la agrupación en clústeres, consulte [How to configure clustering for a Premium Azure Cache for Redis](cache-how-to-premium-clustering.md) (Configuración de la agrupación en clústeres para Azure Cache for Redis de nivel Prémium).

## <a name="enhanced-security-and-isolation"></a>Aislamiento y seguridad mejorados
Las memorias caché creadas en los niveles Basic o Standard están disponibles en Internet. El acceso a las mismas se restringe mediante la clave de acceso. Con el nivel Premium puede asegurarse de que solo los clientes de una red determinada puedan tener acceso a la memoria caché. Puede implementar Azure Cache for Redis en [Azure Virtual Network (VNet)](https://azure.microsoft.com/services/virtual-network/). Además, puede usar todas las características de la red virtual, como las subredes y las directivas de control de acceso, entre otras, para restringir aún más el acceso a Redis.

Para más información, consulte [How to configure Virtual Network support for a Premium Azure Cache for Redis](cache-how-to-premium-vnet.md) (Configuración de la compatibilidad con Virtual Network para una instancia de Azure Cache for Redis de nivel Prémium).

## <a name="importexport"></a>Importar o exportar
Import/Export es una operación de administración de datos de Azure Cache for Redis que permite importar datos en Azure Cache for Redis o exporta datos de Azure Cache for Redis mediante la importación y exportación de una instantánea de base de datos de Azure Cache for Redis (RDB) desde una memoria caché prémium a un blob en páginas en una cuenta de Azure Storage. Permite migrar entre diferentes instancias de Azure Cache for Redis o rellenar la memoria caché de datos antes de su uso.

La importación se puede usar para traer los archivos RDB compatibles de Redis desde cualquier servidor de Redis que se ejecute en cualquier nube o entorno, incluidas las instancias de Redis que se ejecutan en Linux, Windows o cualquier proveedor en la nube como, por ejemplo, Amazon Web Services. La importación de datos supone una manera fácil de crear una caché con datos rellenados previamente. Durante el proceso de importación, Azure Cache for Redis carga los archivos RDB desde Azure Storage en la memoria y, luego, inserta las claves en la memoria caché.

La exportación permite exportar los datos almacenados en Azure Cache for Redis a archivos RDB compatibles. Puede utilizar esta característica para mover datos desde una instancia de Azure Cache for Redis a otra o a otro servidor de Redis. Durante el proceso de exportación, se crea un archivo temporal en la máquina virtual que hospeda la instancia del servidor de Azure Cache for Redis y el archivo se carga en la cuenta de almacenamiento designada. Una vez completada la operación de exportación (de manera correcta o incorrecta), se elimina el archivo temporal.

Para más información, consulte [How to import data into and export data from Azure Cache for Redis](cache-how-to-import-export-data.md)(Importación de datos en Azure Cache for Redis y exportación de datos desde este servicio).

## <a name="reboot"></a>Reiniciar
El nivel premium permite reiniciar uno o varios nodos de la memoria caché a petición. De este modo, podrá probar la resiliencia de la aplicación en caso de error. Puede reiniciar los siguientes nodos.

* Nodo maestro de la memoria caché
* Nodo secundario de la memoria caché
* Nodos principales y secundarios de la memoria caché
* Cuando se usa una caché premium con agrupación en clústeres, puede reiniciar el servidor principal, secundario o ambos nodos para particiones individuales en la memoria caché

Para más información, consulte [Reboot](cache-administration.md#reboot) y [Preguntas más frecuentes sobre el reinicio](cache-administration.md#reboot-faq).

>[!NOTE]
>La funcionalidad de inicio ahora está habilitada para todos los niveles de Azure Cache for Redis.
>
>

## <a name="schedule-updates"></a>Programar actualizaciones
La característica de actualizaciones programadas permite designar una ventana de mantenimiento para la memoria caché. Cuando se especifica la ventana de mantenimiento, las actualizaciones del servidor Redis se realizan en ese período. Para designar una ventana de mantenimiento, seleccione los días deseados y especifique la hora de inicio de la ventana de mantenimiento para cada día. Tenga en cuenta que la hora del período de mantenimiento está en formato UTC. 

Para más información, consulte [Programación de actualizaciones](cache-administration.md#schedule-updates) y [Preguntas más frecuentes sobre la programación de actualizaciones](cache-administration.md#schedule-updates-faq).

> [!NOTE]
> Las actualizaciones del servidor Redis solo se realizan durante el período programado de mantenimiento. La ventana de mantenimiento no se aplica a las actualizaciones de Azure o a las actualizaciones del sistema operativo de la máquina virtual.
> 
> 

## <a name="geo-replication"></a>Replicación geográfica

La **replicación geográfica** proporciona un mecanismo para vincular dos instancias de Azure Cache for Redis de nivel Prémium. Una memoria caché se designa como la caché vinculada principal y la otra, como la caché vinculada secundaria. La caché vinculada secundaria pasa a ser de solo lectura, por lo que los datos escritos en la caché principal se replican en la caché vinculada secundaria. Esta funcionalidad se puede usar para replicar una caché en varias regiones de Azure.

Para más información, consulte [Configuración de replicación geográfica para Azure Cache for Redis](cache-how-to-geo-replication.md).


## <a name="to-scale-to-the-premium-tier"></a>Para escalar al nivel premium
Para escalar al nivel premium, basta con elegir uno de los niveles premium en la hoja **Cambiar el plan de tarifa** . También puede escalar la memoria caché al nivel premium con PowerShell y la CLI. Para instrucciones detalladas acerca del escalado, consulte [How to Scale Azure Cache for Redis](cache-how-to-scale.md) (Escalado de Azure Cache for Redis) y [How to automate a scaling operation](cache-how-to-scale.md#how-to-automate-a-scaling-operation) (Automatización de una operación de escalado).

## <a name="next-steps"></a>Pasos siguientes
Cree una memoria caché y explore las nuevas características del nivel Premium.

* [Configuración de la persistencia en el nivel Prémium de Azure Cache for Redis](cache-how-to-premium-persistence.md)
* [Configuración de la compatibilidad de red virtual con el nivel Prémium de Azure Cache for Redis](cache-how-to-premium-vnet.md)
* [Configuración de la agrupación en clústeres en el nivel Prémium de Azure Cache for Redis](cache-how-to-premium-clustering.md)
* [Importación de datos en Azure Cache for Redis y exportación de datos desde este servicio](cache-how-to-import-export-data.md)
* [Administración de Azure Cache for Redis](cache-administration.md)


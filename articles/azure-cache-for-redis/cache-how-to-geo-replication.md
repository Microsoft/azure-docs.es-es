---
title: Configuración de replicación geográfica para Azure Cache for Redis | Microsoft Docs
description: Obtenga información sobre cómo replicar las instancias de Azure Cache for Redis en distintas regiones geográficas.
author: yegu-ms
ms.service: cache
ms.topic: conceptual
ms.date: 03/06/2019
ms.author: yegu
ms.openlocfilehash: 956e3e83686677f3eb9895354a008783df5f7dcd
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88003707"
---
# <a name="how-to-set-up-geo-replication-for-azure-cache-for-redis"></a>Configuración de replicación geográfica para Azure Cache for Redis

Replicación geográfica proporciona un mecanismo para vincular dos instancias de Azure Cache for Redis de nivel Premium. Una memoria caché se elige como la caché vinculada principal y la otra, como la caché vinculada secundaria. La caché vinculada secundaria pasa a ser de solo lectura, por lo que los datos escritos en la caché principal se replican en la caché vinculada secundaria. TLS protege la transferencia de datos entre las instancias de caché principal y secundaria. La replicación geográfica se puede usar para configurar una memoria caché que abarque dos regiones de Azure. En este artículo se proporciona una guía para configurar la replicación geográfica para las instancias de Azure Cache for Redis de nivel Premium.

> [!NOTE]
> La replicación geográfica está diseñada como una solución de recuperación ante desastres. De forma predeterminada, la aplicación escribirá en la región primaria y leerá desde ahí. Opcionalmente, se puede configurar para leer desde la región secundaria. La replicación geográfica no proporciona conmutación por error automática debido a problemas de latencia de red agregada entre regiones si el resto de la aplicación permanece en la región primaria. Deberá administrar e iniciar la conmutación por error desvinculando la caché secundaria. Esto la promoverá para ser la nueva instancia principal.

## <a name="geo-replication-prerequisites"></a>Requisitos previos de la replicación geográfica

Para configurar la replicación geográfica entre dos cachés, se deben cumplir los siguientes requisitos previos:

- Ambas deben ser cachés de [nivel Premium](cache-overview.md#service-tiers).
- Ambas cachés deben estar en la misma suscripción de Azure.
- La caché vinculada secundaria debe tener el mismo tamaño de caché o un tamaño de caché mayor que la caché vinculada principal.
- Ambas cachés deben estar creadas y en ejecución.

Algunas características no son compatibles con la replicación geográfica:

- La persistencia no es compatible con la replicación geográfica.
- Se admite la agrupación en clústeres si ambas cachés tienen la agrupación en clústeres habilitada y tienen el mismo número de particiones.
- Se admiten cachés en la misma red virtual.
- Las cachés de distintas redes virtuales son compatibles con advertencias. Consulte [¿Puedo usar la replicación geográfica con cachés en una VNET?](#can-i-use-geo-replication-with-my-caches-in-a-vnet) para obtener más información.

Una vez que se configura la replicación geográfica, se aplican las siguientes restricciones al par de cachés vinculadas:

- La caché vinculada secundaria es de solo lectura, porque puede leer de ella, pero no puede escribir datos. 
- Se quitarán los datos existentes en la caché vinculada secundaria antes de la vinculación. Sin embargo, si la replicación geográfica se quita posteriormente, los datos replicados permanecen en la caché vinculada secundaria.
- No puede [escalar](cache-how-to-scale.md) ninguna de las cachés mientras están vinculadas.
- No puede [cambiar el número de particiones](cache-how-to-premium-clustering.md) si la caché tiene la agrupación en clústeres habilitada.
- No puede habilitar la persistencia en ninguna de las cachés.
- Puede [exportar](cache-how-to-import-export-data.md#export) desde cualquier caché.
- No puede [importar](cache-how-to-import-export-data.md#import) en la caché vinculada secundaria.
- No puede eliminar la caché vinculada, ni el grupo de recursos que las contiene, hasta que se desvinculen las cachés. Para más información, vea [¿Por qué no se pudo realizar la operación al intentar eliminar la memoria caché vinculada?](#why-did-the-operation-fail-when-i-tried-to-delete-my-linked-cache)
- Si las memorias caché se encuentran en regiones diferentes, los costes de salida de red se aplican a los datos que se mueven entre regiones. Para más información, consulte el artículo sobre [cuánto cuesta replicar datos entre regiones de Azure](#how-much-does-it-cost-to-replicate-my-data-across-azure-regions).
- La conmutación automática por error no se produce entre la caché vinculada principal y secundaria. Para obtener más información y conocer cómo realizar la conmutación por error de una aplicación cliente, consulte [¿Cómo funciona la conmutación por error a la caché vinculada secundaria?](#how-does-failing-over-to-the-secondary-linked-cache-work)

## <a name="add-a-geo-replication-link"></a>Agregar un vínculo de replicación geográfica

1. Para vincular dos cachés para la replicación geográfica, primero haga clic en **Replicación geográfica** en el menú Recursos de la caché que vaya a ser la caché vinculada principal. A continuación, haga clic en **Agregar vínculo de replicación de caché** desde la hoja **Replicación geográfica**.

    ![Agregar vínculo](./media/cache-how-to-geo-replication/cache-geo-location-menu.png)

2. Haga clic en el nombre de la caché secundaria deseada en la lista **Cachés compatibles**. Si la caché secundaria no aparece en la lista, compruebe que se cumplen los [requisitos previos de replicación geográfica](#geo-replication-prerequisites) para la caché secundaria. Para filtrar las cachés por región, haga clic en la región en el mapa para mostrar solo esas cachés en la lista de **cachés disponibles**.

    ![Cachés compatibles de replicación geográfica](./media/cache-how-to-geo-replication/cache-geo-location-select-link.png)
    
    También puede iniciar el proceso de vinculación o ver detalles sobre la caché secundaria mediante el menú contextual.

    ![Menú contextual de replicación geográfica](./media/cache-how-to-geo-replication/cache-geo-location-select-link-context-menu.png)

3. Haga clic en **Vincular** para vincular las dos cachés y comenzar el proceso de replicación.

    ![Vincular cachés](./media/cache-how-to-geo-replication/cache-geo-location-confirm-link.png)

4. Puede ver el progreso del proceso de replicación en la hoja **Replicación geográfica**.

    ![Estado de la vinculación](./media/cache-how-to-geo-replication/cache-geo-location-linking.png)

    También puede consultar el estado de la vinculación en la hoja de **información general** para ambas cachés, la principal y la secundaria.

    ![Estado de cachés](./media/cache-how-to-geo-replication/cache-geo-location-link-status.png)

    Una vez que se completa el proceso de replicación, el **estado del vínculo** cambia a **Correcto**.

    ![Estado de cachés](./media/cache-how-to-geo-replication/cache-geo-location-link-successful.png)

    La caché vinculada principal sigue estando disponible para su uso durante el proceso de vinculación. La caché vinculada secundaria no está disponible hasta que se complete el proceso de vinculación.

## <a name="remove-a-geo-replication-link"></a>Quitar un vínculo de replicación geográfica

1. Para quitar el vínculo entre dos cachés y detener la replicación geográfica, haga clic en **Desvincular cachés** en la hoja **Replicación geográfica**.
    
    ![Desvincular cachés](./media/cache-how-to-geo-replication/cache-geo-location-unlink.png)

    Una vez que se completa el proceso de desvinculación, la caché secundaria queda disponible tanto para lecturas como para escrituras.

>[!NOTE]
>Cuando se quita el vínculo de replicación geográfica, los datos replicados de la caché vinculada principal se mantienen en la caché secundaria.
>
>

## <a name="geo-replication-faq"></a>Preguntas más frecuentes sobre la replicación geográfica

- [¿Puedo usar la replicación geográfica con una caché de nivel Estándar o Básico?](#can-i-use-geo-replication-with-a-standard-or-basic-tier-cache)
- [¿La caché está disponible para usarla durante los procesos de vinculación o desvinculación?](#is-my-cache-available-for-use-during-the-linking-or-unlinking-process)
- [¿Puedo vincular más de dos cachés?](#can-i-link-more-than-two-caches-together)
- [¿Puedo vincular dos cachés de distintas suscripciones de Azure?](#can-i-link-two-caches-from-different-azure-subscriptions)
- [¿Puedo vincular dos cachés de tamaños distintos?](#can-i-link-two-caches-with-different-sizes)
- [¿Puedo usar la replicación geográfica con la agrupación en clústeres habilitada?](#can-i-use-geo-replication-with-clustering-enabled)
- [¿Puedo usar la replicación geográfica con mis cachés en una VNET?](#can-i-use-geo-replication-with-my-caches-in-a-vnet)
- [¿Qué es la programación de replicación para la replicación geográfica de Redis?](#what-is-the-replication-schedule-for-redis-geo-replication)
- [¿Cuánto tiempo tarda la replicación geográfica?](#how-long-does-geo-replication-replication-take)
- [¿Se garantiza el punto de recuperación de replicación?](#is-the-replication-recovery-point-guaranteed)
- [¿Puedo usar PowerShell o la CLI de Azure para administrar la replicación geográfica?](#can-i-use-powershell-or-azure-cli-to-manage-geo-replication)
- [¿Cuánto cuesta replicar datos entre regiones de Azure?](#how-much-does-it-cost-to-replicate-my-data-across-azure-regions)
- [¿Por qué no se pudo realizar la operación al intentar eliminar la memoria caché vinculada?](#why-did-the-operation-fail-when-i-tried-to-delete-my-linked-cache)
- [¿Qué región se debe usar para la caché vinculada secundaria?](#what-region-should-i-use-for-my-secondary-linked-cache)
- [¿Cómo funciona la conmutación por error a la caché vinculada secundaria?](#how-does-failing-over-to-the-secondary-linked-cache-work)

### <a name="can-i-use-geo-replication-with-a-standard-or-basic-tier-cache"></a>¿Puedo usar la replicación geográfica con una caché de nivel Estándar o Básico?

No, la replicación geográfica solo está disponible para las cachés de nivel Premium.

### <a name="is-my-cache-available-for-use-during-the-linking-or-unlinking-process"></a>¿La caché está disponible para usarla durante los procesos de vinculación o desvinculación?

- Cuando se realiza el proceso de vinculación, la caché vinculada principal sigue estando disponible mientras se completa dicho proceso.
- En el proceso de vinculación, la caché vinculada secundaria no está disponible hasta que se complete dicho proceso.
- Cuando se realiza el proceso de desvinculación, ambas cachés siguen estando disponibles mientras se completa dicho proceso.

### <a name="can-i-link-more-than-two-caches-together"></a>¿Puedo vincular más de dos cachés?

No, solo puede vincular dos cachés juntas.

### <a name="can-i-link-two-caches-from-different-azure-subscriptions"></a>¿Puedo vincular dos cachés de distintas suscripciones de Azure?

No, ambas cachés deben estar en la misma suscripción de Azure.

### <a name="can-i-link-two-caches-with-different-sizes"></a>¿Puedo vincular dos cachés de tamaños distintos?

Sí, siempre que la caché vinculada secundaria sea mayor que la caché vinculada principal.

### <a name="can-i-use-geo-replication-with-clustering-enabled"></a>¿Puedo usar la replicación geográfica con la agrupación en clústeres habilitada?

Sí, siempre que ambas cachés tengan la misma cantidad de particiones.

### <a name="can-i-use-geo-replication-with-my-caches-in-a-vnet"></a>¿Puedo usar la replicación geográfica con mis cachés en una VNET?

Sí, la replicación geográfica de cachés en redes virtuales es compatible con advertencias:

- Se admite la replicación geográfica entre cachés de la misma VNET.
- También se admite la replicación geográfica entre cachés en distintas redes virtuales.
  - Si las redes virtuales están en la misma región, puede conectarlas mediante un [emparejamiento de la red virtual](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview) o una [conexión de red virtual a red virtual de VPN Gateway](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways#V2V).
  - Si las redes virtuales están en distintas regiones, la replicación geográfica que utiliza el emparejamiento de la red virtual no es compatible debido a una restricción con equilibradores de carga internos básicos. Para obtener más información sobre las restricciones de emparejamiento de redes virtuales, consulte [Red virtual - Emparejamiento - Requisitos y restricciones](https://docs.microsoft.com/azure/virtual-network/virtual-network-manage-peering#requirements-and-constraints). La solución recomendada es usar una conexión de red virtual a red virtual de VPN Gateway.

Puede usar [esta plantilla de Azure](https://azure.microsoft.com/resources/templates/201-redis-vnet-geo-replication/) para implementar rápidamente dos cachés con replicación geográfica en una red virtual conectada con una conexión puerta de enlace a puerta de enlace de VPN Gateway.

### <a name="what-is-the-replication-schedule-for-redis-geo-replication"></a>¿Qué es la programación de replicación para la replicación geográfica de Redis?

La replicación no se realiza en una programación específica, es continua y asincrónica. Todas las escrituras realizadas en el servidor principal se replican asincrónicamente y al instante en el servidor secundario.

### <a name="how-long-does-geo-replication-replication-take"></a>¿Cuánto tiempo tarda la replicación geográfica?

La replicación es incremental, asincrónica y continua, y el tiempo empleado no es muy diferente de la latencia entre regiones. En determinadas circunstancias, la caché secundaria puede requerir la realización de una sincronización completa de los datos del servidor principal. El tiempo de replicación en este caso depende del número de factores, como la carga en la caché principal, el ancho de banda de red disponible y la latencia entre regiones. Hemos detectado que el tiempo de replicación para un par de replicación geográfica de 53 GB completa puede estar entre 5 y 10 minutos.

### <a name="is-the-replication-recovery-point-guaranteed"></a>¿Se garantiza el punto de recuperación de replicación?

Para las cachés en un modo de replicación geográfica, la persistencia está deshabilitada. Si se ha desvinculado un par de replicación geográfica, como una conmutación por error iniciada por el cliente, la caché vinculada secundaria mantiene sus datos sincronizados hasta ese momento. En esas situaciones no se garantiza ningún punto de recuperación.

Para obtener un punto de recuperación, [exporte](cache-how-to-import-export-data.md#export) desde cualquier caché. Más adelante, puede [importar](cache-how-to-import-export-data.md#import) en la caché vinculada principal.

### <a name="can-i-use-powershell-or-azure-cli-to-manage-geo-replication"></a>¿Puedo usar PowerShell o la CLI de Azure para administrar la replicación geográfica?

Sí, se puede administrar la replicación geográfica mediante Azure Portal, PowerShell o la CLI de Azure. Para obtener más información, consulte los [documentos de PowerShell](https://docs.microsoft.com/powershell/module/az.rediscache/?view=azps-1.4.0#redis_cache) o [los documentos de la CLI de Azure](https://docs.microsoft.com/cli/azure/redis/server-link?view=azure-cli-latest).

### <a name="how-much-does-it-cost-to-replicate-my-data-across-azure-regions"></a>¿Cuánto cuesta replicar datos entre regiones de Azure?

Cuando usa la replicación geográfica, los datos de la caché vinculada principal se replican a la caché vinculada secundaria. Si las dos cachés vinculadas están en la misma región, no hay ningún cargo por la transferencia de datos. Si las dos cachés vinculadas se encuentran en regiones diferentes, el cargo por transferencia de datos es el coste de salida de red de los datos que se transfieren entre cualquier región. Para más información, consulte [Detalles de precios de ancho de banda](https://azure.microsoft.com/pricing/details/bandwidth/).

### <a name="why-did-the-operation-fail-when-i-tried-to-delete-my-linked-cache"></a>¿Por qué no se pudo realizar la operación al intentar eliminar la memoria caché vinculada?

Las cachés con replicación geográfica y sus grupos de recursos no se pueden eliminar si están vinculados hasta que quite el vínculo de replicación geográfica. Si intenta eliminar el grupo de recursos que contiene una o las dos cachés vinculadas, se eliminan los otros recursos del grupo de recursos, pero el grupo de recursos permanece en el estado `deleting` y las cachés en el grupo de recursos permanecen en el estado `running`. Para completar la eliminación del grupo de recursos y las cachés vinculadas dentro de él, desvincule las cachés como se describe en [Quitar un vínculo de replicación geográfica](#remove-a-geo-replication-link).

### <a name="what-region-should-i-use-for-my-secondary-linked-cache"></a>¿Qué región se debe usar para la caché vinculada secundaria?

En general, se recomienda que la caché esté en la misma región de Azure que la aplicación que accede a ella. Si las aplicaciones tienen regiones principales y de reserva, se recomiendan que las cachés principal y secundaria estén en esas mismas regiones. Para más información sobre regiones emparejadas, consulte el artículo sobre [procedimientos recomendados: regiones emparejadas de Azure](../best-practices-availability-paired-regions.md).

### <a name="how-does-failing-over-to-the-secondary-linked-cache-work"></a>¿Cómo funciona la conmutación por error a la caché vinculada secundaria?

No se admite la conmutación automática por error entre regiones de Azure para cachés de replicación geográfica. En un escenario de recuperación ante desastres, los clientes deben mostrar la pila de toda la aplicación de manera coordinada en su región de copia de seguridad. Permitir a los componentes de aplicación individuales cuándo cambiar a sus copias de seguridad por sí mismos puede afectar negativamente al rendimiento. Uno de los beneficios clave de Redis es que se trata de un almacén de latencia muy baja. Si la aplicación principal del cliente está en una región distinta a su caché, el tiempo de ida y vuelta agregado tendría un impacto apreciable en el rendimiento. Por este motivo, evitamos que se realice automáticamente la conmutación por error debido a problemas transitorios de disponibilidad.

Para ejecutar una conmutación por error iniciada por el cliente, desvincule primero las cachés. A continuación, cambie el cliente de Redis para utilizar el punto de conexión de la caché secundaria (anteriormente vinculada). Cuando se desvinculan las dos cachés, la réplica secundaria se vuelve a convertir en una caché normal de lectura y escritura, y acepta solicitudes directamente de los clientes de Redis.

## <a name="next-steps"></a>Pasos siguientes
Más información sobre las características de Azure Cache for Redis.

* [Niveles de servicio de Azure Cache for Redis](cache-overview.md#service-tiers)

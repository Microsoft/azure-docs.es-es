---
title: Alta disponibilidad en Azure Cosmos DB
description: En este artículo se describe cómo Azure Cosmos DB ofrece una alta disponibilidad
author: markjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 06/29/2020
ms.author: mjbrown
ms.reviewer: sngun
ms.openlocfilehash: 8bae89e68e5a016dbdc10c763f1ea2daedece3c8
ms.sourcegitcommit: d661149f8db075800242bef070ea30f82448981e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88605332"
---
# <a name="high-availability-with-azure-cosmos-db"></a>Alta disponibilidad con Azure Cosmos DB

Azure Cosmos DB replica con transparencia los datos en todas las regiones de Azure asociadas con la cuenta de Azure Cosmos. Azure Cosmos DB emplea diferentes capas de redundancia para los datos, tal como se muestra en la imagen siguiente:

:::image type="content" source="./media/high-availability/cosmosdb-data-redundancy.png" alt-text="Creación de particiones físicas" border="false":::

- Los datos de los contenedores de Azure Cosmos se [dividen horizontalmente en particiones](partitioning-overview.md).

- Un conjunto de particiones es una colección de varios conjuntos de réplicas. En cada una de las regiones, cada partición está protegida por un conjunto de réplicas con todas las escrituras replicadas y confirmadas de forma duradera por la mayoría de las réplicas. Las réplicas se distribuyen entre unos 10 o 20 dominios de error como máximo.

- Se replica cada partición de todas las regiones. Cada región contiene todas las particiones de datos de un contenedor de Azure Cosmos y puede aceptar operaciones de escritura y atender lecturas.  

Si la cuenta de Azure Cosmos se distribuye entre *N* regiones de Azure, habrá al menos *N*x 4 copias de todos los datos. Generalmente, tener una cuenta de Azure Cosmos en más de dos regiones mejora la disponibilidad de la aplicación y proporciona una latencia baja en las regiones asociadas. 

## <a name="slas-for-availability"></a>SLA para disponibilidad

Como base de datos distribuida globalmente, Azure Cosmos DB proporciona contratos de nivel de servicio completos que abarcan rendimiento, latencia en el percentil 99, coherencia y alta disponibilidad. En la tabla siguiente se muestran las garantías de alta disponibilidad proporcionadas por Azure Cosmos DB para cuentas de una y varias regiones. Para lograr alta disponibilidad, configure siempre las cuentas de Azure Cosmos para que tengan varias regiones de escritura (también denominado arquitectura multimaestro).

|Tipo de operación  | Región única |Varias regiones (escrituras de región única)|Varias regiones (escrituras de varias regiones) |
|---------|---------|---------|-------|
|Escrituras    | 99,99    |99,99   |99,999|
|Lecturas     | 99,99    |99,999  |99,999|

> [!NOTE]
> En la práctica, la disponibilidad de escritura real para modelos de posible coherencia, prefijo coherente, sesión y obsolescencia limitada es significativamente mayor que los SLA publicados. La disponibilidad de lectura real para todos los niveles de coherencia es significativamente mayor que los SLA publicados.

## <a name="high-availability-with-azure-cosmos-db-in-the-event-of-regional-outages"></a>Alta disponibilidad con Azure Cosmos DB en caso de interrupciones regionales

En los infrecuentes casos en que se produce una interrupción regional, Azure Cosmos DB garantiza que la base de datos siempre tenga una alta disponibilidad. Los siguientes detalles capturan el comportamiento de Azure Cosmos DB durante una interrupción, en función de la configuración de la cuenta de Azure Cosmos:

- Con Azure Cosmos DB, antes de que el cliente confirme una operación de escritura, un cuórum de réplicas dentro de la región que acepta las operaciones de escritura confirma los datos de forma permanente.

- Las cuentas de varias regiones configuradas con varias regiones de escritura o arquitectura multimaestro seguirán teniendo una alta disponibilidad tanto para las escrituras como para las lecturas. Las conmutaciones por error regionales son instantáneas y no requieren cambios de la aplicación.

- Las cuentas de una sola región pueden perder disponibilidad después de una interrupción regional. Se recomienda configurar siempre **al menos dos regiones** (si es posible, al menos dos regiones de escritura) con su cuenta de Azure Cosmos para garantizar una alta disponibilidad en todo momento.

### <a name="multi-region-accounts-with-a-single-write-region-write-region-outage"></a>Cuentas de varias regiones con una sola región de escritura (interrupción de la región de escritura)

- Durante una interrupción en la región de escritura, la cuenta de Azure Cosmos promoverá automáticamente una región secundaria para que sea la nueva región de escritura principal cuando la opción para **habilitar la conmutación por error automática** está configurada en la cuenta de Azure Cosmos. Cuando se habilita, la conmutación por error se producirá en otra región según el orden de prioridad de regiones que especificó.
- Cuando la región afectada previamente vuelva a estar en línea, los datos de escritura cuya replicación se anuló al producirse el error en la región se ponen a disposición a través de la [fuente de conflictos](how-to-manage-conflicts.md#read-from-conflict-feed). Las aplicaciones pueden leer la fuente de conflictos, resolver los conflictos de acuerdo con la lógica específica de la aplicación y escribir los datos actualizados de nuevo en el contenedor de Azure Cosmos según corresponda.
- Una vez que se recupera la región de escritura previamente afectada, se convierte en disponible automáticamente como una región de lectura. Puede volver a la región recuperada como la región de escritura. Puede cambiar las regiones con [PowerShell, la CLI de Azure o Azure Portal](how-to-manage-database-account.md#manual-failover). No se produce **ninguna pérdida de datos ni de disponibilidad** antes, durante ni después de cambiar la región de escritura, y la aplicación sigue siendo de alta disponibilidad.

> [!IMPORTANT]
> Se recomienda encarecidamente configurar las cuentas de Azure Cosmos usadas para que las cargas de trabajo de producción **habiliten la conmutación por error automática**. La conmutación por error manual requiere conectividad entre las regiones de escritura secundaria y primaria para completar una comprobación de coherencia a fin de garantizar que no haya pérdida de datos durante la conmutación por error. Si la región primaria no está disponible, no se puede completar esta comprobación de coherencia y la conmutación por error manual no se realizará correctamente, lo que provocará la pérdida de disponibilidad de escritura.

### <a name="multi-region-accounts-with-a-single-write-region-read-region-outage"></a>Cuentas de varias regiones con una sola región de escritura (interrupción de la región de lectura)

- Durante una interrupción de la región de lectura, las cuentas de Azure Cosmos mediante un nivel de coherencia o coherencia alta con tres o más regiones de lectura seguirán teniendo una alta disponibilidad para lecturas y escrituras.
- Las cuentas de Azure Cosmos que usen una coherencia alta con dos o menos regiones de lectura (que incluye la región de lectura y escritura) perderán la disponibilidad de escritura durante una interrupción de la región de lectura, pero mantendrán la disponibilidad de lectura para las demás regiones.
- La región afectada se desconecta automáticamente y se marcará como sin conexión. Los [SDK de Azure Cosmos DB](sql-api-sdk-dotnet.md) redirigirán las llamadas de lectura a la siguiente región disponible en la lista de regiones preferidas.
- Si ninguna de las regiones de la lista de las regiones preferidas está disponible, las llamadas se devuelven automáticamente a la región actual de escritura.
- No es necesario realizar ningún cambio en el código de su aplicación para gestionar la interrupción de la región de lectura. Cuando la región de lectura afectada se vuelve a conectar, se sincronizará automáticamente con la región de escritura actual y estará disponible de nuevo para atender las solicitudes de lectura.
- Las siguientes lecturas se redirigen a la región recuperada sin necesidad de realizar cambios en el código de la aplicación. Durante la conmutación por error y cuando se vuelva a unir una región previamente errónea, Azure Cosmos DB seguirá cumpliendo las garantías de coherencia de lectura.
- Incluso en un caso poco frecuente y desafortunado en que la región de Azure sea irrecuperable de forma permanente, no se produce pérdida de datos si se configura la cuenta de Azure Cosmos de varias regiones con coherencia *fuerte*. En el caso de una región de escritura irrecuperable de forma permanente, una cuenta de Azure Cosmos de varias regiones configurada con coherencia de obsolescencia limitada, la posible ventana de pérdida de datos se restringe a la ventana de obsolescencia (*K* o *T*) donde K = 100 000 actualizaciones y T = 5 minutos. En cuanto a los niveles de posible coherencia y prefijo coherente por sesión, el período de pérdida potencial de datos se restringe a un máximo de 15 minutos. Para obtener más información sobre los objetivos de RPO y RTO de Azure Cosmos DB, consulte [Durabilidad de datos y los niveles de coherencia](consistency-levels-tradeoffs.md#rto).

## <a name="availability-zone-support"></a>Compatibilidad de zonas de disponibilidad

Además de la resistencia entre regiones, ahora puede habilitar la **redundancia de zona** al seleccionar una región que asociar a su base de datos de Azure Cosmos.

Gracias a la compatibilidad de zonas de disponibilidad, Azure Cosmos DB garantizará que las réplicas estén colocadas en varias zonas de una región determinada para proporcionar alta disponibilidad y resistencia durante errores zonales. No hay ningún cambio en la latencia ni en otros SLA en esta configuración. En el caso de que haya un error de una sola zona, la redundancia de zona proporciona durabilidad total de los datos con el RPO=0 y la disponibilidad con RTO=0.

La redundancia de zona es una *funcionalidad complementaria* de la característica de [replicación en escrituras de varias regiones](how-to-multi-master.md). No se puede confiar en la redundancia de zona por sí sola para lograr la resistencia regional. Por ejemplo, si se producen interrupciones regionales o un acceso de baja latencia en las regiones, se recomienda tener varias regiones de escritura además de la redundancia de zona.

Al configurar las escrituras en varias regiones para la cuenta de Azure Cosmos, puede optar por recibir la redundancia de zona sin ningún costo adicional. En caso contrario, consulte la nota siguiente sobre los precios de la compatibilidad de la redundancia de zona. Puede habilitar la redundancia de zona en una región existente de su cuenta de Azure Cosmos si quita la región y la vuelve a agregar con la redundancia de zona habilitada.

Esta característica está disponible en: regiones del *Sur de Reino Unido, Sudeste Asiático, Este de EE. UU., Este de EE. UU. 2, Centro de EE. UU., Oeste de Europa, Oeste de EE. UU. 2, Este de Japón, Norte de Europa, Centro de Francia, Este de Australia*.

> [!NOTE]
> Si habilita Availability Zones en una cuenta de Azure Cosmos de una sola región, se ocasionarán cargos equivalentes a agregar una región adicional a su cuenta. Para obtener más información sobre los precios, consulte la [página de precios](https://azure.microsoft.com/pricing/details/cosmos-db/) y los artículos sobre [costos de varias regiones de Azure Cosmos DB](optimize-cost-regions.md).

En la tabla siguiente se resume la funcionalidad de alta disponibilidad de varias configuraciones de cuenta:

|KPI  |Región única sin Availability Zones (sin AZ)  |Región única con Availability Zones (AZ)  |Escrituras en varias regiones con Availability Zones (AZ, 2 regiones): configuración más recomendada |
|---------|---------|---------|---------|
|Contrato de Nivel de Servicio de disponibilidad de escritura | 99,99% | 99,99% | 99,999 % |
|Contrato de Nivel de Servicio de disponibilidad de lectura  | 99,99% | 99,99% | 99,999 % |
|Price | Tarifa de facturación de una sola región | Tarifa de facturación de zona de disponibilidad de una sola región | Tarifa de facturación de varias regiones |
|Errores de zona: pérdida de datos | Pérdida de datos | No se produce pérdida de datos | No se produce pérdida de datos |
|Errores de zona: disponibilidad | Pérdida de disponibilidad | Sin pérdida de disponibilidad | Sin pérdida de disponibilidad |
|Latencia de lectura | Entre regiones | Entre regiones | Bajo |
|Latencia de escritura | Entre regiones | Entre regiones | Bajo |
|Interrupción regional: pérdida de datos | Pérdida de datos |  Pérdida de datos | Pérdida de datos <br/><br/> Al usar la coherencia de obsolescencia limitada con varias regiones de escritura y más de una región, la pérdida de datos se limita a la obsolescencia limitada configurada en su cuenta. <br /><br />Puede evitar la pérdida de datos durante una interrupción regional configurando una coherencia alta con varias regiones. Esta opción conlleva ciertos inconvenientes que afectan al rendimiento y la disponibilidad. Solo se puede configurar en cuentas que estén configuradas para escrituras de una sola región. |
|Interrupción regional: disponibilidad | Pérdida de disponibilidad | Pérdida de disponibilidad | Sin pérdida de disponibilidad |
|Throughput | X RU/s de rendimiento aprovisionado | X RU/s de rendimiento aprovisionado | 2X RU/s de rendimiento aprovisionado <br/><br/> Este modo de configuración requiere dos veces la cantidad de rendimiento en comparación con una sola región con Availability Zones porque hay dos regiones. |

> [!NOTE]
> Para habilitar la compatibilidad de Availability Zones para una cuenta de Azure Cosmos DB de varias regiones, la cuenta debe tener habilitadas las escrituras de varias regiones.

Puede habilitar la redundancia de zona al agregar una región a cuentas de Azure Cosmos nuevas o existentes. Para habilitar la redundancia de zona en su cuenta de Azure Cosmos, debe establecer la marca `isZoneRedundant` en `true` para una ubicación específica. Puede establecer esta marca en la propiedad de ubicaciones. Por ejemplo, el siguiente fragmento de código de PowerShell habilita la redundancia de zona de la región "Sudeste Asiático":

```powershell
$locations = @(
    @{ "locationName"="Southeast Asia"; "failoverPriority"=0; "isZoneRedundant"= "true" },
    @{ "locationName"="East US"; "failoverPriority"=1; "isZoneRedundant"= "true" }
)
```

En el siguiente comando, se muestra cómo habilitar la redundancia de zona para las regiones "EastUS" y "WestUS2":

```azurecli-interactive
az cosmosdb create \
  --name mycosmosdbaccount \
  --resource-group myResourceGroup \
  --kind GlobalDocumentDB \
  --default-consistency-level Session \
  --locations regionName=EastUS failoverPriority=0 isZoneRedundant=True \
  --locations regionName=WestUS2 failoverPriority=1 isZoneRedundant=True
```

Puede habilitar Availability Zones mediante Azure Portal al crear una cuenta de Azure Cosmos. Cuando cree una cuenta, asegúrese de habilitar la **Redundancia geográfica** y las **Escrituras en varias regiones**, y elija una región en la que se admita Availability Zones:

:::image type="content" source="./media/high-availability/enable-availability-zones-using-portal.png" alt-text="Habilitación de Availability Zones mediante Azure Portal"::: 

## <a name="building-highly-available-applications"></a>Compilación de aplicaciones de alta disponibilidad

- Para garantizar una alta disponibilidad de escritura y lectura, configure la cuenta de Azure Cosmos para abarcar al menos dos regiones con varias regiones de escritura. Esta configuración proporcionará la disponibilidad más alta, la latencia más baja y la mejor escalabilidad para lecturas y escrituras respaldadas por los SLA. Para más información, vea cómo [configurar la cuenta de Azure Cosmos con varias regiones de escritura](tutorial-global-distribution-sql-api.md).

- Para cuentas de Azure Cosmos de varias regiones que estén configuradas con una región única de escritura, [habilite la conmutación por error automática mediante la CLI de Azure o Azure Portal](how-to-manage-database-account.md#automatic-failover). Después de habilitar la conmutación automática por error, siempre que se produzca un desastre regional, Cosmos DB conmutarán por error automáticamente su cuenta.  

- Incluso si su cuenta de Azure Cosmos es de alta disponibilidad, es posible que la aplicación no se haya diseñado correctamente para seguir teniendo alta disponibilidad. Para probar la alta disponibilidad de un extremo a otro de la aplicación, como parte de las pruebas de la aplicación o las exploraciones de recuperación ante desastres (DR), deshabilite temporalmente la conmutación automática por error de la cuenta, invoque la [conmutación por error manual mediante PowerShell, la CLI de Azure o Azure Portal](how-to-manage-database-account.md#manual-failover) y, luego, supervise la conmutación por error de la aplicación. Cuando haya terminado, puede realizar la conmutación por recuperación a la región primaria y restaurar la conmutación automática por error en la cuenta.

- En un entorno de base de datos distribuida de forma global, existe una relación directa entre el nivel de coherencia y la durabilidad de los datos si se produce una interrupción en toda la región. A medida que desarrolle el plan de continuidad empresarial, tendrá que saber el tiempo máximo aceptable para que la aplicación se recupere por completo tras un evento de interrupción. El tiempo necesario para que una aplicación se recupere totalmente se conoce como "objetivo de tiempo de recuperación (RTO)". También debe conocer el período máximo de actualizaciones de datos recientes que la aplicación puede tolerar perder al recuperarse después de un evento de interrupción. El período de tiempo de las actualizaciones que se puede permitir perder se conoce como objetivo de punto de recuperación (RPO). Para ver el RPO y el RTO de Azure Cosmos DB, consulte [Durabilidad de datos y los niveles de coherencia](consistency-levels-tradeoffs.md#rto).

## <a name="next-steps"></a>Pasos siguientes

Ahora puede pasar a la lectura de los artículos siguientes:

- [Availability and performance tradeoffs for various consistency levels](consistency-levels-tradeoffs.md) (Compromisos entre rendimiento y disponibilidad en los distintos niveles de coherencia)
- [Escalado del rendimiento aprovisionado globalmente](scaling-throughput.md)
- [Distribución global en segundo plano](global-dist-under-the-hood.md)
- [Niveles de coherencia en Azure Cosmos DB](consistency-levels.md)
- [Procedimiento para configurar la cuenta de Cosmos con varias regiones de escritura](how-to-multi-master.md)

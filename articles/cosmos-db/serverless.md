---
title: Oferta sin servidor basada en el consumo en Azure Cosmos DB
description: Más información sobre la oferta sin servidor basada en el consumo de Azure Cosmos DB.
author: ThomasWeiss
ms.author: thweiss
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 08/19/2020
ms.openlocfilehash: ef681f861a14fbbf86e7e350441d05f84fc95f58
ms.sourcegitcommit: afa1411c3fb2084cccc4262860aab4f0b5c994ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/23/2020
ms.locfileid: "88757904"
---
# <a name="azure-cosmos-db-serverless-preview"></a>Azure Cosmos DB sin servidor (versión preliminar)

> [!IMPORTANT]
> Azure Cosmos DB sin servidor está actualmente en versión preliminar. Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no es aconsejable usarla para cargas de trabajo de producción. Para obtener más información, consulte [Términos de uso complementarios de las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Azure Cosmos DB sin servidor le permite usar su cuenta de Azure Cosmos en un modo basado en el consumo, donde solo se cobran las unidades de solicitud que las operaciones de base de datos consumen y el almacenamiento consumido por los datos. No existe un cargo mínimo en el uso de Azure Cosmos DB en modo sin servidor.

> [!IMPORTANT] 
> ¿Tiene comentarios sobre el modo sin servidor? Queremos conocerlos. No dude en enviar un mensaje al equipo de Azure Cosmos DB sin servidor: [azurecosmosdbserverless@service.microsoft.com](mailto:azurecosmosdbserverless@service.microsoft.com).

Cuando se usa Azure Cosmos DB, cada operación de base de datos tiene un costo expresado en [unidades de solicitud](request-units.md). La forma en que se le cobra por este costo depende del tipo de cuenta de Azure Cosmos que use:

- En modo de [rendimiento aprovisionado](set-throughput.md), debe confirmar una cantidad determinada de rendimiento (expresado en unidades de solicitud por segundo) que se aprovisiona en las bases de datos y los contenedores. El costo de las operaciones de base de datos se deduce a partir del número de unidades de solicitud disponibles cada segundo. Al final del período de facturación, se le cobrará la cantidad de rendimiento aprovisionado.
- En el modo sin servidor, no tiene que aprovisionar ningún rendimiento al crear contenedores en su cuenta de Azure Cosmos. Al final del período de facturación, se le facturará el número de unidades de solicitud consumidas por las operaciones de base de datos.

## <a name="use-cases"></a>Casos de uso

Azure Cosmos DB sin servidor se adapta mejor a los escenarios en los que se espera:

- **Tráfico ligero**: dado que la capacidad de aprovisionamiento en estas situaciones no es necesaria y puede ser prohibitiva
- **Capacidad moderada de ráfaga**: dado que los contenedores sin servidor pueden proporcionar hasta 5000 unidades de solicitud por segundo
- **Rendimiento moderado**: dado que los contenedores sin servidor tienen [características de rendimiento específicas](#performance)

Por estos motivos, se debe tener en cuenta Azure Cosmos DB sin servidor para los siguientes tipos de carga de trabajo:

- Desarrollo
- Prueba
- Prototipos
- Prueba de concepto
- Aplicación no crítica con tráfico ligero

Consulte el artículo sobre la [elección entre el rendimiento aprovisionado y el modo sin servidor](throughput-serverless.md) para obtener más información sobre cómo elegir la oferta que mejor se adapte a su caso de uso.

## <a name="using-serverless-resources"></a>Uso de recursos sin servidor

Sin servidor es un nuevo tipo de cuenta de Azure Cosmos, lo que significa que tiene que elegir entre el **rendimiento aprovisionado** y el modo **sin servidor** al crear una nueva cuenta. Debe crear una nueva cuenta sin servidor para empezar a trabajar sin servidor. Durante la versión preliminar, la única manera admitida para crear una nueva cuenta sin servidor es mediante [Azure Portal](create-cosmosdb-resources-portal.md). Actualmente no se admite la migración de las cuentas existentes a o desde el modo sin servidor.

> [!NOTE]
> De momento, el modo sin servidor se admite solo en la API de Azure Cosmos DB Core (SQL).

Cualquier contenedor que se cree en una cuenta sin servidor será un contenedor sin servidor. Los contenedores sin servidor exponen las mismas funcionalidades que los contenedores creados en el modo de rendimiento aprovisionado, por lo que puede leer, escribir y consultar los datos de la misma manera. Sin embargo, las cuentas y los contenedores sin servidor también tienen características específicas:

> [!IMPORTANT]
> Es posible que algunas de estas limitaciones se reduzcan o eliminen cuando el servidor se comercialice de forma general y **sus comentarios** nos ayudarán a tomar decisiones. Póngase en contacto con nosotros y cuéntenos más sobre su experiencia en el modo sin servidor: [azurecosmosdbserverless@service.microsoft.com](mailto:azurecosmosdbserverless@service.microsoft.com).

- Una cuenta sin servidor solo puede ejecutarse en una única región de Azure. No es posible agregar más regiones de Azure a una cuenta sin servidor después de crearla.
- No es posible habilitar la [Característica en vista previa (GB) de Synapse Link](synapse-link.md) en una cuenta sin servidor.
- El aprovisionamiento de rendimiento no es necesario en los contenedores sin servidor, por lo que se aplican las siguientes instrucciones:
    - No se puede pasar ningún rendimiento al crear un contenedor sin servidor y, si se hace, se devuelve un error.
    - No se puede leer ni actualizar el rendimiento en un contenedor sin servidor y, si se hace, se devuelve un error.
    - No se puede crear una base de datos de rendimiento compartido en una cuenta sin servidor y, si se hace, se devuelve un error.
- Los contenedores sin servidor pueden ofrecer una capacidad de ráfaga de rendimiento máxima de 5000 unidades de solicitud por segundo.
- Los contenedores sin servidor pueden almacenar un máximo de 50 GB de datos e índices.

## <a name="monitoring-your-consumption"></a>Supervisión del consumo

Si ya ha usado Azure Cosmos DB en el modo de rendimiento aprovisionado, observará que sin servidor es más rentable cuando el tráfico no justifica la capacidad aprovisionada. La desventaja es que los costos serán menos predecibles porque se facturan en función del número de solicitudes procesadas por la base de datos. Por eso, es importante estar atento al consumo actual.

Al examinar el panel **Métricas** de su cuenta, encontrará un gráfico denominado **Request Units consumed** (Unidades de solicitud consumidas) en la pestaña **Información general**. En este gráfico se muestra el número de unidades de solicitud que ha consumido su cuenta:

:::image type="content" source="./media/serverless/request-units-consumed.png" alt-text="Gráfico que muestra las unidades de solicitud consumidas" border="false":::

Puede encontrar el mismo gráfico al usar Azure Monitor, tal y como se describe [aquí](monitor-request-unit-usage.md). Tenga en cuenta que Azure Monitor le permite configurar [alertas](../azure-monitor/platform/alerts-metric-overview.md), que se pueden usar para recibir una notificación cuando el consumo de unidades de solicitud ha superado un umbral determinado.

## <a name="performance"></a><a id="performance"></a>Rendimiento

Los recursos sin servidor brindan características de rendimiento específicas que son diferentes de las que ofrecen los recursos de rendimiento aprovisionados:

- **Disponibilidad**: una vez que la oferta sin servidor esté disponible con carácter general, la disponibilidad de los contenedores sin servidor se incluirá en un Acuerdo de Nivel de Servicio (SLA) del 99,9 % cuando no se use Availability Zones (redundancia de zona). El Acuerdo de Nivel de Servicio es del 99,99 % cuando se usa Availability Zones.
- **Latencia**: una vez que la oferta sin servidor esté disponible con carácter general, la latencia de los contenedores sin servidor se incluirá en un objetivo de nivel de servicio (SLO) de 10 milisegundos o menos para las lecturas de punto y de 30 milisegundos o menos para las escrituras. Una operación de lectura de punto consiste en capturar un solo elemento por su identificador y el valor de clave de partición.
- **Capacidad de ráfaga**: una vez que la oferta sin servidor esté disponible con carácter general, la capacidad de ráfaga de los contenedores sin servidor se incluirá en un objetivo de nivel de servicio (SLO) del 95 %. Esto significa que la capacidad máxima de ráfaga se puede alcanzar al menos el 95 % del tiempo.

> [!NOTE]
> Como cualquier versión preliminar de Azure, Azure Cosmos DB sin servidor se excluye de los Acuerdos de Nivel de Servicio (SLA). Las características de rendimiento mencionadas anteriormente se proporcionan como versión preliminar de lo que esta oferta proporcionará cuando esté disponible con carácter general.

## <a name="next-steps"></a>Pasos siguientes

Comience a trabajar con el modo sin servidor con los siguientes artículos:

- [Unidades de solicitud en Azure Cosmos DB](request-units.md)
- [Elija entre el rendimiento aprovisionado y sin servidor](throughput-serverless.md)
- [Modelo de precios de Azure Cosmos DB](how-pricing-works.md)

---
title: Modelo de precios de Azure Cosmos DB
description: En este artículo se explica el modelo de precios de Azure Cosmos DB y cómo simplifica la administración y el planeamiento de los costos.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 08/19/2020
ms.openlocfilehash: a992d240955f42ec030a84c887ba086ce92f9790
ms.sourcegitcommit: d661149f8db075800242bef070ea30f82448981e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88605253"
---
# <a name="pricing-model-in-azure-cosmos-db"></a>Modelo de precios de Azure Cosmos DB

El modelo de precios de Azure Cosmos DB simplifica la administración y el planeamiento de los costos. Con Azure Cosmos DB, paga por las operaciones que realiza en la base de datos y por el almacenamiento consumido por los datos.

- **Operaciones de la base de datos**: la forma en que se le cobran las operaciones de base de datos depende del tipo de cuenta de Azure Cosmos que se use.

  - **Rendimiento aprovisionado**: El [rendimiento aprovisionado](set-throughput.md) (también denominado "reservado") garantiza el alto rendimiento a cualquier escala. El rendimiento necesario se especifica en [Unidades de solicitud](request-units.md) por segundo (RU/s), y Azure Cosmos DB dedica los recursos requeridos para garantizar el rendimiento configurado. También puede [aprovisionar el rendimiento en una base de datos o un contenedor](set-throughput.md). En función de las necesidades de la carga de trabajo, puede escalar o reducir el rendimiento en cualquier momento o usar la [escalabilidad automática](provision-throughput-autoscale.md) (aunque se requiere un rendimiento mínimo en una base de datos o un contenedor para garantizar los Acuerdos de Nivel de Servicio). La facturación se realiza por hora al rendimiento máximo aprovisionado para una concreta.

   > [!NOTE]
   > Dado que el modelo de rendimiento aprovisionado dedica recursos al contenedor o la base de datos, se le cobrará por el rendimiento aprovisionado, aunque no ejecute ninguna carga de trabajo.

  - **Sin servidor**: en el modo [sin servidor](serverless.md), no tiene que aprovisionar ningún rendimiento al crear recursos en la cuenta de Azure Cosmos. Al final del período de facturación, se le factura el número de unidades de solicitud consumidas por las operaciones de base de datos.

- **Almacenamiento**: se le cobra una tarifa plana por la cantidad total de almacenamiento (GB) usada por los datos y los índices en una hora concreta. El almacenamiento se factura en función del consumo, por lo que no tiene que reservar ningún almacenamiento por adelantado. Se le cobrará solo por el almacenamiento que consuma.

El modelo de precios de Azure Cosmos DB es coherente en todas las API. Para más información, consulte la [página de precios de Azure Cosmos DB](https://azure.microsoft.com/pricing/details/cosmos-db/) y los artículos [Entienda la factura de Azure Cosmos DB](understand-your-bill.md) y [Rentabilidad del modelo de precios de Azure Cosmos DB para los clientes](total-cost-ownership.md).

Si implementa la cuenta de Azure Cosmos DB en una región no gubernamental de EE. UU., hay un precio mínimo en el modo de rendimiento aprovisionado para el rendimiento basado en contenedores y bases de datos. No hay ningún precio mínimo en el modo sin servidor. Los precios varían en función de la región que use; vea la página de precios de [Azure Cosmos DB](https://azure.microsoft.com/pricing/details/cosmos-db/) para obtener más información sobre los precios.

## <a name="try-azure-cosmos-db-for-free"></a>Pruebe gratis Azure Cosmos DB

Azure Cosmos DB ofrece muchas opciones para que los desarrolladores lo prueben de forma gratuita. Entre estas opciones se incluyen:

* **Nivel Gratis de Azure Cosmos DB**: El nivel Gratis de Azure Cosmos DB facilita la introducción, el desarrollo y la prueba de las aplicaciones, o incluso la ejecución de pequeñas cargas de trabajo de producción de forma gratuita. Cuando el nivel Gratis esté habilitado en una cuenta, obtendrá las primeras 400 RU/s y 5 GB de almacenamiento de la cuenta gratis, durante la duración de dicha cuenta. Puede tener una cuenta de nivel Gratis por cada suscripción de Azure y debe optar por tenerla al crear la cuenta. Para empezar, [cree una cuenta en Azure Portal con el nivel gratis habilitado](create-cosmosdb-resources-portal.md) o use una [plantilla de ARM](manage-sql-with-resource-manager.md#free-tier).

* **Cuenta gratuita de Azure**: Azure ofrece una [cuenta gratuita](https://azure.microsoft.com/free/) con 200 USD de crédito de Azure para los 30 primeros días y una cantidad limitada de servicios gratuitos durante 12 meses. Para más información consulte el sitio de [cuentas gratuitas de Azure](../cost-management-billing/manage/avoid-charges-free-account.md). Azure Cosmos DB forma parte de esta cuenta gratuita de Azure. Específicamente para Azure Cosmos DB, esta cuenta gratuita ofrece 5 GB de almacenamiento y 400 RU/s de rendimiento aprovisionado para todo el año.

* **Probar gratis Azure Cosmos DB**: Azure Cosmos DB se ofrece con las cuentas gratuitas por tiempo limitado. Puede crear una cuenta de Azure Cosmos DB, bases de datos y colecciones, y ejecutar una aplicación de ejemplo con la ayuda de artículos de inicio rápido y tutoriales. Puede ejecutar la aplicación de ejemplo sin tener que suscribirse a una cuenta de Azure o con la tarjeta de crédito. [Pruebe gratis Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) para disfrutar de ello durante un mes, con la posibilidad de renovar la cuenta las veces que desee.

* **Emulador de Azure Cosmos DB**: El emulador de Azure Cosmos DB proporciona un entorno local que emula el servicio Azure Cosmos DB con fines de desarrollo. El emulador se ofrece sin costo alguno y con alta fidelidad para el servicio en la nube. Mediante el emulador de Azure Cosmos DB, puede desarrollar y probar su aplicación localmente, sin crear una suscripción de Azure ni incurrir en costos. Puede desarrollar sus aplicaciones mediante el emulador localmente antes de pasar a producción. Cuando esté satisfecho con la funcionalidad de la aplicación en el emulador, puede cambiar a la cuenta de Azure Cosmos DB en la nube y ahorrará costos considerables. Para más información sobre el emulador, consulte el artículo sobre el [uso de Azure Cosmos DB para desarrollo y pruebas](local-emulator.md) para más detalles.

## <a name="pricing-with-reserved-capacity"></a>Precios con capacidad reservada

La [capacidad reservada](cosmos-db-reserved-capacity.md) de Azure Cosmos DB le ayuda a ahorrar dinero cuando se usa el rendimiento aprovisionado, ya que los recursos de Azure Cosmos DB se pagan por anticipado durante uno o tres años. Con los compromisos de un año o tres por adelantado, puede reducir los costos considerablemente y ahorrar de un 20 a un 65 % de descuento en comparación con los precios normales. La capacidad reservada de Azure Cosmos DB ayuda a reducir los costos al pagar previamente por la capacidad de proceso aprovisionada (RU/s) durante un período de uno o tres años y obtener un descuento en la capacidad de proceso aprovisionada. 

La capacidad reservada ofrece un descuento en la facturación y no afecta el estado del entorno de ejecución de sus recursos de Azure Cosmos DB. La capacidad reservada está disponible de manera coherente para todas las API (incluidos MongoDB, Cassandra, SQL, Gremlin y Azure Tables) y en todas las regiones del mundo. Puede encontrar más información sobre la capacidad reservada en el artículo sobre el [pago por adelantado de los recursos de Azure Cosmos DB con capacidad reservada](cosmos-db-reserved-capacity.md) y comprar capacidad reservada en [Azure Portal](https://portal.azure.com/).

## <a name="next-steps"></a>Pasos siguientes

Más información sobre la optimización de los costos de los recursos de Azure Cosmos DB en los siguientes artículos:

* Sobre la [optimización para el desarrollo y pruebas](optimize-dev-test.md)
* Obtenga más información sobre [la factura de Azure Cosmos DB](understand-your-bill.md).
* Obtenga más información sobre la [optimización del costo de la capacidad del rendimiento](optimize-cost-throughput.md).
* Obtenga más información sobre la [optimización del costo del almacenamiento](optimize-cost-storage.md).
* Obtenga más información sobre la [optimización del costo de la lectura y la escritura](optimize-cost-reads-writes.md).
* Obtenga más información sobre la [optimización del costo de las consultas](optimize-cost-queries.md).
* Más información sobre la [optimización del costo de las cuentas de Cosmos de varias regiones](optimize-cost-regions.md)
* Información sobre la [capacidad reservada de Azure Cosmos DB](cosmos-db-reserved-capacity.md)
* Información sobre el [emulador de Azure Cosmos DB](local-emulator.md)

---
title: Niveles de rendimiento retirados de Azure Cosmos DB
description: Obtenga información acerca de los niveles de rendimiento S1, S2 y S3 que estaban disponibles anteriormente en Azure Cosmos DB.
author: SnehaGunda
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 06/04/2018
ms.author: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: ce81ce9afa45c93010c457bc292bba037607f96f
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89020888"
---
# <a name="retiring-the-s1-s2-and-s3-performance-levels"></a>Retirada de los niveles de rendimiento S1, S2 y S3

> [!IMPORTANT] 
> Los niveles de rendimiento S1, S2 y S3 descritos en este artículo se han retirado y ya no están disponibles para las nuevas cuentas de Azure Cosmos DB.

En este artículo se proporciona información general de los niveles de rendimiento S1, S2 y S3, y se describe cómo se pueden migrar las colecciones que usan estos niveles de rendimiento a colecciones de partición única. Después de leer este artículo, podrá responder a las preguntas siguientes:

- [¿Por qué se han retirado los niveles de rendimiento S1, S2 y S3?](#why-retired)
- [¿Cómo se comparan las colecciones de partición única y con particiones con los niveles de rendimiento S1, S2 y S3?](#compare)
- [¿Qué tengo que hacer para garantizar el acceso ininterrumpido a mis datos?](#uninterrupted-access)
- [¿Cómo cambiará mi colección después de la migración?](#collection-change)
- [¿Cómo cambiará mi facturación después migrar a colecciones de partición única?](#billing-change)
- [¿Qué ocurre si necesito más de 20 GB de almacenamiento?](#more-storage-needed)
- [¿Puedo cambiar entre los niveles de rendimiento S1, S2 y S3 antes de la migración planeada?](#change-before)
- [¿Cómo migro de los niveles de rendimiento S1, S2 y S3 a las colecciones de partición única por mí mismo?](#migrate-diy)
- [¿Cómo me afecta si soy un cliente de EA?](#ea-customer)

<a name="why-retired"></a>

## <a name="why-are-the-s1-s2-and-s3-performance-levels-being-retired"></a>¿Por qué se han retirado los niveles de rendimiento S1, S2 y S3?

Los niveles de rendimiento S1, S2 y S3 no ofrecen la flexibilidad que proporciona la oferta estándar de Azure Cosmos DB. Con los niveles de rendimiento S1, S2 y S3, tanto la capacidad de rendimiento como la de almacenamiento están preestablecidas y no ofrecen elasticidad. Ahora Azure Cosmos DB ofrece la capacidad para personalizar su rendimiento y almacenamiento, lo que ofrece mucha más flexibilidad para el escalado a medida que cambien sus necesidades.

<a name="compare"></a>

## <a name="how-do-single-partition-collections-and-partitioned-collections-compare-to-the-s1-s2-s3-performance-levels"></a>¿Cómo se comparan las colecciones de partición única y con particiones con los niveles de rendimiento S1, S2 y S3?

En la tabla siguiente se comparan las opciones de rendimiento y almacenamiento disponibles en las colecciones de partición única, las colecciones con particiones y los niveles de rendimiento S1, S2 y S3. Este es un ejemplo para la región Este de EE. UU. 2:

| Nombre de cuota  |Colección con particiones|Colección de partición única|S1|S2|S3|
|---|---|---|---|---|---|
|Rendimiento máximo|Sin límite|10 000 RU/s|250 RU/s|1000 RU/s|2500 RU/s|
|Rendimiento mínimo|2500 RU/s|400 RU/s|250 RU/s|1000 RU/s|2500 RU/s|
|Almacenamiento máximo|Sin límite|20 GB|20 GB|20 GB|20 GB|
|Precio (mensual)|Rendimiento: 6 USD / 100 RU/s<br><br>Almacenamiento: 0,25 USD/GB|Rendimiento: 6 USD / 100 RU/s<br><br>Almacenamiento: 0,25 USD/GB|25 USD|50 USD|100 USD|

¿Es un cliente de EA? Si es así, consulte [¿Cómo me afecta si soy un cliente de EA?](#ea-customer)

<a name="uninterrupted-access"></a>

## <a name="what-do-i-need-to-do-to-ensure-uninterrupted-access-to-my-data"></a>¿Qué tengo que hacer para garantizar el acceso ininterrumpido a mis datos?

Si tiene una colección S1, S2 o S3, debe migrar la colección a una sola colección de particiones mediante programación [mediante el SDL de .NET](#migrate-diy). 

<a name="collection-change"></a>

## <a name="how-will-my-collection-change-after-the-migration"></a>¿Cómo cambiará mi colección después de la migración?

Si tiene una colección de S1, puede migrar a una colección de partición única con un rendimiento de 400 RU/s. 400 RU/s es el rendimiento más bajo disponible con colecciones de partición única. Sin embargo, el costo de 400 RU/s en la colección de partición única es aproximadamente el mismo que con la colección de S1 y 250 RU/s, por lo que no pagará las 150 RU/s extra disponibles.

Si tiene una colección de S2, puede migrar a una colección de partición única con 1 K RU/s. No verá ningún cambio en su nivel de rendimiento.

Si tiene una colección de S3, puede migrar a una colección de partición única con 2 2,5 K RU/s. No verá ningún cambio en su nivel de rendimiento.

En todos estos casos, después de migrar la colección, podrá personalizar el nivel de rendimiento o escalar y reducir verticalmente según sea necesario para proporcionar un acceso de baja latencia a los usuarios. 

<a name="billing-change"></a>

## <a name="how-will-my-billing-change-after-i-migrated-to-the-single-partition-collections"></a>¿Cómo cambiará mi facturación después migrar a colecciones de partición única?

Supongamos que tiene 10 colecciones S1 en la región Este de EE. UU. y 1 GB de almacenamiento para cada una, y las migra a 10 recopilaciones de partición única a 400 RU/seg. (el nivel mínimo). Su factura tendrá el siguiente aspecto si mantiene las 10 colecciones de partición única un mes completo:

:::image type="content" source="./media/performance-levels/s1-vs-standard-pricing.png" alt-text="Cómo se comparan los precios de S1 para 10 colecciones con los precios de 10 colecciones que usan una colección de partición única" border="false":::

<a name="more-storage-needed"></a>

## <a name="what-if-i-need-more-than-20-gb-of-storage"></a>¿Qué ocurre si necesito más de 20 GB de almacenamiento?

Independientemente de si tiene una colección con un nivel de rendimiento S1, S2 o S3, o si tiene una colección de partición única, todos con 20 GB de almacenamiento disponibles, puede utilizar la herramienta de migración de datos de Azure Cosmos DB para migrar los datos a una colección con particiones, con un almacenamiento prácticamente ilimitado. Para más información acerca de las ventajas de una colección con particiones, consulte el tema sobre [particiones y escalado en Azure Cosmos DB](sql-api-partition-data.md). 

<a name="change-before"></a>

## <a name="can-i-change-between-the-s1-s2-and-s3-performance-levels-before-the-planned-migration"></a>¿Puedo cambiar entre los niveles de rendimiento S1, S2 y S3 antes de la migración planeada?

Solo las cuentas existentes con los niveles de rendimiento S1, S2 y S3 pueden cambiar y modificar los niveles de rendimiento mediante programación [con el SDK de .NET](#migrate-diy). Si cambia de S1, S3 o S3 a una colección de partición única, no puede volver a los niveles de rendimiento S1, S2 o S3.

<a name="migrate-diy"></a>

## <a name="how-do-i-migrate-from-the-s1-s2-s3-performance-levels-to-single-partition-collections-on-my-own"></a>¿Cómo migro de los niveles de rendimiento S1, S2 y S3 a las colecciones de partición única por mí mismo?

Puede migrar de los niveles de rendimiento S1, S2 y S3 a colecciones de partición única mediante programación [con el SDK de .NET](#migrate-diy). Puede hacerlo por sí mismo antes de la migración planeada para beneficiarse de las opciones de rendimiento flexible disponibles con colecciones de partición única.

### <a name="migrate-to-single-partition-collections-by-using-the-net-sdk"></a>Migración a colecciones de partición única con el SDK de .NET

En esta sección solo se trata el cambio del nivel de rendimiento de una colección mediante la [API de .NET de SQL](sql-api-sdk-dotnet.md), pero el proceso es similar para los demás SDK.

A continuación se muestra un fragmento de código para cambiar el rendimiento de la colección a 5 000 unidades de solicitud por segundo:
    
```csharp
    //Fetch the resource to be updated
    Offer offer = client.CreateOfferQuery()
                      .Where(r => r.ResourceLink == collection.SelfLink)    
                      .AsEnumerable()
                      .SingleOrDefault();

    // Set the throughput to 5000 request units per second
    offer = new OfferV2(offer, 5000);

    //Now persist these changes to the database by replacing the original resource
    await client.ReplaceOfferAsync(offer);
```

Visite [MSDN](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.documentclient.aspx) para ver más ejemplos y obtener más información sobre nuestros métodos de oferta:

* [**ReadOfferAsync**](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.documentclient.readofferasync.aspx)
* [**ReadOffersFeedAsync**](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.documentclient.readoffersfeedasync.aspx)
* [**ReplaceOfferAsync**](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.documentclient.replaceofferasync.aspx)
* [**CreateOfferQuery**](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.linq.documentqueryable.createofferquery.aspx)

<a name="ea-customer"></a>

## <a name="how-am-i-impacted-if-im-an-ea-customer"></a>¿Cómo me afecta si soy un cliente de EA?

El precio para los clientes de EA estará protegido hasta el final de su contrato actual.

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información sobre los precios y la administración de datos con Azure Cosmos DB, consulte estos recursos:

1.  [Partición de datos en Cosmos DB](sql-api-partition-data.md). Información sobre la diferencia entre un contenedor de partición única y contenedores con particiones, así como sugerencias sobre cómo implementar una estrategia de particiones para escalar sin problemas.
2.  [Precios de Cosmos DB](https://azure.microsoft.com/pricing/details/cosmos-db/). Aprenda sobre los costes de rendimiento del aprovisionamiento y el consumo de almacenamiento.
3.  [Unidades de solicitud](request-units.md). Información sobre el consumo de rendimiento para diferentes tipos de operaciones; por ejemplo, lectura, escritura o consulta.

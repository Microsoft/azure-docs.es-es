---
title: 'Tutorial: Creación de aplicaciones móviles con Xamarin y Azure Cosmos DB'
description: 'Tutorial: Tutorial que permite crear una aplicación de Xamarin iOS, Android o Forms mediante Azure Cosmos DB. Azure Cosmos DB es una base de datos de nube, a escala mundial y rápida para aplicaciones móviles.'
author: SnehaGunda
ms.service: cosmos-db
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 11/05/2019
ms.author: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: 14a004864891aad768486c3a436b4936d693715e
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88999043"
---
# <a name="tutorial-build-mobile-applications-with-xamarin-and-azure-cosmos-db"></a>Tutorial: Creación de aplicaciones móviles con Xamarin y Azure Cosmos DB

> [!div class="op_single_selector"]
> * [.NET](sql-api-dotnet-application.md)
> * [Java](sql-api-java-application.md)
> * [Node.js](sql-api-nodejs-application.md)
> * [Python](sql-api-python-application.md)
> * [Xamarin](mobile-apps-with-xamarin.md)
> 

La mayoría de las aplicaciones móviles necesitan almacenar los datos en la nube, y Azure Cosmos DB es una base de datos de nube para aplicaciones móviles. Tiene todo lo que un desarrollador de aplicaciones móviles puede necesitar. Es una base de datos como servicio completamente administrada que se escala a petición. Puede llevar los datos a su aplicación de forma transparente, dondequiera que se encuentren los usuarios en cualquier parte del mundo. Mediante el [SDK de .NET Core para Azure Cosmos DB](sql-api-sdk-dotnet-core.md), puede permitir que las aplicaciones móviles de Xamarin interactúen directamente con Azure Cosmos DB, sin una capa intermedia.

En este artículo, se proporciona un tutorial para crear aplicaciones móviles con Xamarin y Azure Cosmos DB. Puede encontrar el código fuente completo del tutorial en [Xamarin y Azure Cosmos DB en Github](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/xamarin), además de información sobre cómo administrar usuarios y permisos.

## <a name="azure-cosmos-db-capabilities-for-mobile-apps"></a>Funcionalidades de Azure Cosmos DB para aplicaciones móviles
Azure Cosmos DB proporciona las siguientes funcionalidades clave para los desarrolladores de aplicaciones móviles:

:::image type="content" source="media/mobile-apps-with-xamarin/documentdb-for-mobile.png" alt-text="Funcionalidades de Azure Cosmos DB para aplicaciones móviles":::

* Consultas completas sobre datos sin esquema. Azure Cosmos DB almacena datos como documentos JSON sin esquema en colecciones heterogéneas. Ofrece [consultas completas y rápidas](how-to-sql-query.md) sin tener que preocuparse de esquemas o índices.
* Rendimiento rápido. Solo se tardan unas milésimas de segundos en leer y escribir documentos con Azure Cosmos DB. Los desarrolladores pueden especificar el rendimiento que necesitan y Azure Cosmos DB lo aplica mediante un Acuerdo de Nivel de Servicio con disponibilidad del 99,99 % para todas las cuentas de región individual y todas las cuentas de varias regiones con coherencia menos estricta, y disponibilidad de lectura del 99,999 % para todas las cuentas de base de datos de varias regiones.
* Escala ilimitada. Las colecciones de Azure Cosmos [crecen a medida que la aplicación crece](partition-data.md). Puede empezar con un tamaño de datos pequeño y el rendimiento de cientos de solicitudes por segundo. Las colecciones o bases de datos pueden crecer hasta petabytes de datos y un rendimiento arbitrariamente grande con centenares de millones de solicitudes por segundo.
* Distribución global. Los usuarios de aplicaciones móviles están moviéndose de un lado para otro, con frecuencia por todo el mundo. Azure Cosmos DB es una [base de datos distribuida globalmente](distribute-data-globally.md). Haga clic en el mapa para permitir que sus datos sean accesibles para sus usuarios.
* Autorización completa integrada. Con Azure Cosmos DB, puede implementar fácilmente patrones conocidos como [datos por usuario](https://github.com/kirillg/azure-documentdb-dotnet/tree/master/samples/xamarin/UserItems) o datos compartidos por varios usuarios sin un código de autorización personalizado complejo.
* Consultas geoespaciales. Muchas aplicaciones móviles ofrecen en la actualidad experiencias en contexto geográfico. Gracias a la compatibilidad de primera clase con [tipos geoespaciales](geospatial.md), con Azure Cosmos DB crear de estas experiencias es fácil de conseguir.
* Datos adjuntos binarios. Los datos de aplicaciones a menudo incluyen blobs binarios. La compatibilidad nativa con datos adjuntos facilita el uso de Azure Cosmos DB como tienda integral para los datos de las aplicaciones.

## <a name="azure-cosmos-db-and-xamarin-tutorial"></a>Tutorial sobre Xamarin y Azure Cosmos DB
En el siguiente tutorial se muestra cómo crear una aplicación móvil con Xamarin y Azure Cosmos DB. Puede encontrar el código fuente completo del tutorial en [Xamarin y Azure Cosmos DB en Github](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/xamarin).

### <a name="get-started"></a>Introducción
Es fácil comenzar a usar Azure Cosmos DB. Vaya a Azure Portal y cree una nueva cuenta de Azure Cosmos DB. Haga clic en la pestaña **Inicio rápido**. Descargue el ejemplo de lista de tareas pendientes de Xamarin Forms que ya está conectado a su cuenta de Azure Cosmos DB. 

:::image type="content" source="media/mobile-apps-with-xamarin/cosmos-db-quickstart.png" alt-text="Inicio rápido de Azure Cosmos DB para aplicaciones móviles":::

O bien, si ya tiene una aplicación de Xamarin, puede agregar el [paquete NuGet de Azure Cosmos DB](sql-api-sdk-dotnet-core.md). Azure Cosmos DB admite bibliotecas compartidas de Xamarin.IOS, Xamarin.Android y Xamarin Forms.

### <a name="work-with-data"></a>Trabajar con datos
Los registros de datos se almacenan en Azure Cosmos DB como documentos JSON sin esquema en colecciones heterogéneas. Puede almacenar documentos con diferentes estructuras en la misma colección:

```cs
    var result = await client.CreateDocumentAsync(collectionLink, todoItem);
```

En sus proyectos de Xamarin puede usar Language-Integrated Queries sobre datos sin esquema:

```cs
    var query = await client.CreateDocumentQuery<ToDoItem>(collectionLink)
                    .Where(todoItem => todoItem.Complete == false)
                    .AsDocumentQuery();

    Items = new List<TodoItem>();
    while (query.HasMoreResults) {
        Items.AddRange(await query.ExecuteNextAsync<TodoItem>());
    }
```
### <a name="add-users"></a>Agregar usuarios
Al igual que muchos otros ejemplos de introducción, el ejemplo de Azure Cosmos DB que ha descargado se autentica en el servicio mediante claves maestras codificadas en el código de la aplicación. Este valor predeterminado no es una buena práctica para una aplicación que pretende ejecutar en cualquier lugar excepto en el emulador local. Si un usuario no autorizado obtuviera la clave maestra, todos los datos que pasan por su cuenta de Azure Cosmos DB podrían verse comprometidos. Así que lo que quiere es que su aplicación acceda únicamente a los registros del usuario que inició sesión. Azure Cosmos DB les permite a los desarrolladores conceder permiso de lectura o de lectura y escritura para la aplicación a una colección, un conjunto de documentos agrupados por una clave de partición o un documento específico. 

Siga estos pasos para modificar la aplicación de lista de tareas pendientes y convertirla en una aplicación de lista de tareas pendientes para varios usuarios: 

  1. Agregue el inicio de sesión a la aplicación, mediante Facebook, Active Directory o cualquier otro proveedor.

  2. Cree una colección UserItems de Azure Cosmos DB con **/userId** como clave de partición. Al especificar la clave de partición de su colección, permite que Azure Cosmos DB escale hasta el infinito a medida que crece el número de usuarios de nuestra aplicación, al tiempo que sigue ofreciendo consultas rápidas.

  3. Agregue Resource Token Broker de Azure Cosmos DB. Esta API web sencilla autentica usuarios y emite tokens de corta duración a usuarios con sesión iniciada con acceso únicamente a los documentos dentro de su partición. En este ejemplo, Resource Token Broker está hospedado en App Service.

  4. Modifique la aplicación para autenticarse en Resource Token Broker con Facebook, y solicite los tokens de recursos para el usuario que ha iniciado sesión en Facebook. Luego, puede acceder a sus datos en la colección UserItems.  

Puede encontrar un código de ejemplo completo de este patrón en [Resource Token Broker en Github](https://github.com/kirillg/azure-documentdb-dotnet/tree/master/samples/xamarin/UserItems). Este diagrama ilustra la solución:

:::image type="content" source="media/mobile-apps-with-xamarin/documentdb-resource-token-broker.png" alt-text="Agente de permisos y usuarios de Azure Cosmos DB" border="false":::

Si quiere que dos usuarios accedan a la misma lista de tareas pendientes, puede agregar permisos adicionales al token de acceso en Resource Token Broker.

### <a name="scale-on-demand"></a>Escalado a petición
Azure Cosmos DB es una base de datos como servicio administrada. A medida que crece su base de usuarios, no es necesario preocuparse de aprovisionar máquinas virtuales o de aumentar los núcleos. Todo lo que tiene que hacer es indicar a Azure Cosmos DB cuántas aplicaciones por segundo (rendimiento) necesita su aplicación. Puede especificar el rendimiento mediante la pestaña **Escala** usando una medida del rendimiento llamada Unidades de solicitud (RU) por segundo. Por ejemplo, una operación de lectura en un documento de 1 KB requiere 1 RU. Puede agregar también alertas para la métrica **Rendimiento** con el fin de supervisar el crecimiento del tráfico y cambiar mediante programación el rendimiento cuando se disparen las alertas.

:::image type="content" source="media/mobile-apps-with-xamarin/cosmos-db-xamarin-scale.png" alt-text="Rendimiento a gran escala de Azure Cosmos DB a petición":::

### <a name="go-planet-scale"></a>Escala a nivel mundial
A medida que su aplicación gane popularidad, podría conseguir usuarios de todo el mundo. O bien, quizá desee estar preparado para imprevistos. Vaya a Azure Portal y abra su cuenta de Azure Cosmos DB. Haga clic en el mapa para hacer que los datos se replique continuamente en cualquier número de regiones en todo el mundo. Esta funcionalidad permite que los datos estén disponibles siempre que los usuarios lo estén. También puede agregar directivas de conmutación por error para estar preparado para contingencias.

:::image type="content" source="media/mobile-apps-with-xamarin/cosmos-db-xamarin-replicate.png" alt-text="Escala de Azure Cosmos DB entre regiones geográficas" border="false":::

¡Enhorabuena! Ha finalizado la solución y tiene una aplicación móvil con Xamarin y Azure Cosmos DB. Siga pasos similares para crear aplicaciones de Cordova mediante el SDK de JavaScript para Azure Cosmos DB, y aplicaciones nativas de iOS o Android mediante las API de REST de Azure Cosmos DB.

## <a name="next-steps"></a>Pasos siguientes
* Vea el código fuente de [Xamarin y Azure Cosmos DB en Github](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/xamarin).
* Descargue el [SDK de .NET Core para Azure Cosmos DB](sql-api-sdk-dotnet-core.md).
* Encuentre más ejemplos de código para [aplicaciones .NET](sql-api-dotnet-samples.md).
* Aprenda sobre las [funcionalidades de consultas completas de Azure Cosmos DB](how-to-sql-query.md).
* Aprenda sobre la [compatibilidad geoespacial en Azure Cosmos DB](geospatial.md).




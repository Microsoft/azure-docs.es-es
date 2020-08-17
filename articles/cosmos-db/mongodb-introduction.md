---
title: Introducción a API de Azure Cosmos DB para MongoDB
description: Vea cómo puede usar Azure Cosmos DB para almacenar y consultar grandes cantidades de datos mediante la API de Azure Cosmos DB para MongoDB.
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: overview
ms.date: 10/1/2019
author: sivethe
ms.author: sivethe
ms.openlocfilehash: 8066ba5c895ec5c3fdbf06ffc0a1f30117dcd4d1
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87076811"
---
# <a name="azure-cosmos-dbs-api-for-mongodb"></a>API de Azure Cosmos DB para MongoDB

[Azure Cosmos DB](introduction.md) es un servicio de base de datos con varios modelos y distribución global de Microsoft para aplicaciones críticas. Azure Cosmos DB ofrece una [distribución global inmediata](distribute-data-globally.md), [escalado elástico de rendimiento y almacenamiento](partition-data.md) en todo el mundo, latencias de menos de 10 ms en el percentil 99 y alta disponibilidad garantizada, todo ello respaldado por [acuerdos de nivel de servicio líderes del sector](https://azure.microsoft.com/support/legal/sla/cosmos-db/). Azure Cosmos DB [indexa datos automáticamente](https://www.vldb.org/pvldb/vol8/p1668-shukla.pdf) sin que haya que ocuparse de la administración de esquemas ni de índices. Admite varios modelos y admite documentos, pares clave-valor, grafos y modelos de datos en columnas. El servicio de Azure Cosmos DB implementa protocolos de conexión para las API no SQL más comunes, incluidas Cassandra, MongoDB, Gremlin y Azure Table Storage. Esto le permite usar los controladores de cliente de NoSQL y las herramientas que ya conoce para interactuar con la base de datos de Cosmos.

## <a name="wire-protocol-compatibility"></a>Compatibilidad de protocolo de conexión

Azure Cosmos DB implementa el protocolo de conexión para MongoDB. Esta implementación permite una compatibilidad transparente con las herramientas, los controladores y los SDK de cliente nativos de MongoDB. Azure Cosmos DB hospeda el motor de base de datos de MongoDB. Los detalles de las características admitidas por MongoDB se pueden encontrar aquí: 
- [API para la versión 3.6 del motor de Mongo DB de Azure Cosmos DB](mongodb-feature-support-36.md)
- [API para la versión 3.2 del motor de Mongo DB de Azure Cosmos DB](mongodb-feature-support.md)

De forma predeterminada, las nuevas cuentas creadas mediante la API de Azure Cosmos DB para MongoDB son compatibles con la versión 3.6 del protocolo de conexión de MongoDB. Cualquier controlador cliente de MongoDB que reconozca esta versión del protocolo debería poder conectarse de forma nativa a Cosmos DB.

:::image type="content" source="./media/mongodb-introduction/cosmosdb-mongodb.png" alt-text="API de Azure Cosmos DB para MongoDB" border="false":::

## <a name="key-benefits"></a>Ventajas principales

Las principales ventajas de Cosmos DB como base de datos completamente administrada y distribuida globalmente como servicio se describen [aquí](introduction.md). Además, al implementar de forma nativa los protocolos de conexión de API de NoSQL populares, Cosmos DB ofrece las siguientes ventajas:

* Migrar fácilmente la aplicación a Cosmos DB, al tiempo que conserva partes importantes de la lógica de aplicación.
* Mantener la aplicación portátil y permanecer independiente de proveedores de nube.
* Obtenga contratos de nivel de servicio con respaldo financiero líderes del sector para API de NoSQL comunes con tecnología de Cosmos DB.
* Escale de forma elástica el rendimiento y el almacenamiento aprovisionados para sus bases de datos de Cosmos según sus necesidades y pague solo el rendimiento y el almacenamiento que necesita. lo que se traduce en un importante ahorro de costos.
* Distribución global llave en mano con replicación de arquitectura multimaestro.

## <a name="cosmos-dbs-api-for-mongodb"></a>API de Cosmos DB para MongoDB

Siga los artículos de inicio rápido si desea crear una cuenta de Azure Cosmos y migrar la aplicación de MongoDB para usar Azure Cosmos DB, o bien compile una nueva:

* [Migración de una aplicación web Node.js de MongoDB existente](create-mongodb-nodejs.md)
* [Compilación de una aplicación web mediante la API de Azure Cosmos DB para MongoDB y el SDK de .NET](create-mongodb-dotnet.md)
* [Compilación de una aplicación de consola mediante la API de Azure Cosmos DB para MongoDB y el SDK de Java](create-mongodb-java.md)

## <a name="next-steps"></a>Pasos siguientes

Estas son algunas sugerencias para comenzar:

* Para aprender a obtener la información de cadena de conexión de la cuenta, siga el tutorial de [conexión de una aplicación de MongoDB a Azure Cosmos DB](connect-mongodb-account.md).
* Para aprender a crear una conexión entre la base de datos de Azure Cosmos DB y la aplicación de MongoDB en Studio 3T, siga el tutorial de [uso de Studio 3T con Azure Cosmos DB](mongodb-mongochef.md).
* Para importar los datos a una base de datos de Cosmos, siga el tutorial de [importación de datos de MongoDB en Azure Cosmos DB](mongodb-migrate.md).
* Conéctese a una cuenta de Cosmos con [Robo 3T](mongodb-robomongo.md).
* Aprenda a [configurar las preferencias de lectura para las aplicaciones distribuidas globalmente](../cosmos-db/tutorial-global-distribution-mongodb.md).
* Busque las soluciones a los errores más comunes en nuestra [Guía de solución de problemas](mongodb-troubleshoot.md).


<sup>Nota: En este artículo se describe una característica de Azure Cosmos DB que proporciona compatibilidad del protocolo de conexión con bases de datos de MongoDB. Microsoft no ejecuta bases de datos de MongoDB que ofrezcan este servicio. Azure Cosmos DB no está afiliado a MongoDB, Inc.</sup>

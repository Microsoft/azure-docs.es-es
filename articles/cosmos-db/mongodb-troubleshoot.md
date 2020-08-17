---
title: Solución de errores habituales de la API de Azure Cosmos DB para MongoDB
description: En este documento se describen las formas de solucionar los problemas habituales detectados en la API de Azure Cosmos DB para MongoDB.
author: LuisBosquez
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: troubleshooting
ms.date: 07/15/2020
ms.author: lbosq
ms.openlocfilehash: f75374fc88923a0f131d513bebf0ffe1feeca359
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87076763"
---
# <a name="troubleshoot-common-issues-in-azure-cosmos-dbs-api-for-mongodb"></a>Solución de errores habituales de la API de Azure Cosmos DB para MongoDB

En el siguiente artículo se describen los errores comunes y las soluciones para las bases de datos que usan la API de Azure Cosmos DB para MongoDB.

>[!Note]
> Azure Cosmos DB no hospeda el motor de MongoDB. Proporciona una implementación de la [versión 3.6 del protocolo de conexión](mongodb-feature-support-36.md) de MongoDB, y compatibilidad heredada con la [versión 3.2 del protocolo de conexión](mongodb-feature-support.md), por lo que algunos de estos errores solo se encuentran en la API de Azure Cosmos DB para MongoDB. 

## <a name="common-errors-and-solutions"></a>Errores habituales y soluciones

| Error               | Código  | Descripción  | Solución  |
|---------------------|-------|--------------|-----------|
| ExceededTimeLimit   | 50 | La solicitud ha superado el tiempo de espera de 60 segundos de ejecución. | Este error puede deberse a muchas causas. Una de las causas es cuando la capacidad de las unidades de solicitud asignada actual no es suficiente para completar la solicitud. Para solucionar esto, se pueden aumentar las unidades de solicitud de esa colección o base de datos. En otros casos, este error se puede solucionar dividiendo una solicitud grande en otras más pequeñas. |
| TooManyRequests     | 16500 | El número total de unidades de solicitud consumidas es mayor que la tasa de unidades de solicitud aprovisionadas para la colección y se ha limitado. | Considere la posibilidad de escalar el rendimiento asignado a un contenedor o un conjunto de contenedores desde Azure Portal o vuelva a intentarlo. |
| ExceededMemoryLimit | 16501 | Como se trata de un servicio de varios inquilinos, la operación ha superado la asignación de memoria del cliente. | Reduzca el ámbito de la operación a través de criterios de consulta más restrictivos o póngase en contacto con soporte técnico desde [Azure Portal](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade). Ejemplo: `db.getCollection('users').aggregate([{$match: {name: "Andy"}}, {$sort: {age: -1}}]))` |
| La ruta de acceso del índice correspondiente al elemento order-by especificado se excluye o la consulta order by no tiene un índice compuesto correspondiente desde el que se puede atender. | 2 | La consulta solicita que se ordene por un campo que no está indexado. | Cree un índice coincidente (o índice compuesto) para la consulta de ordenación que se intenta realizar. |
| Problemas de versión de la conexión de MongoDB | - | Las versiones anteriores de los controladores de MongoDB son no detectan el nombre de la cuenta de Azure Cosmos en las cadenas de conexión. | Anexe *appName = @**nombreDeLaCuenta**@* al final de la API de Cosmos DB para la cadena de conexión de MongoDB, donde ***nombreDeLaCuenta*** es el nombre de la cuenta de Cosmos DB. |

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [usar Studio 3T](mongodb-mongochef.md) con la API de Azure Cosmos DB para MongoDB.
- Aprenda a [usar Robo 3T](mongodb-robomongo.md) con la API de Azure Cosmos DB para MongoDB.
- Explore [ejemplos](mongodb-samples.md) de MongoDB con la API de Azure Cosmos DB para MongoDB.


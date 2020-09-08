---
title: Cuotas de servicio de Azure Cosmos DB
description: Límites predeterminados y cuotas de servicio de Azure Cosmos DB en diferentes tipos de recursos.
author: abhijitpai
ms.author: abpai
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 09/02/2020
ms.openlocfilehash: a6b1bd016248212967a4fe78274d5ba72c5df0fe
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89400320"
---
# <a name="azure-cosmos-db-service-quotas"></a>Cuotas de servicio de Azure Cosmos DB

En este artículo se proporciona una introducción a las cuotas predeterminadas que se ofrecen a distintos recursos en Azure Cosmos DB.

## <a name="storage-and-database-operations"></a>Operaciones de base de datos y almacenamiento

Después de crear una cuenta de Azure Cosmos DB en su suscripción a Azure, puede administrar los datos en su cuenta mediante la [creación de bases de datos, contenedores y elementos](databases-containers-items.md).

### <a name="provisioned-throughput"></a>Rendimiento aprovisionado

Puede aprovisionar el rendimiento en un nivel de contenedor o de base de datos en términos de [unidades de solicitud (RU/s o RU)](request-units.md). En la tabla siguiente se enumeran los límites de almacenamiento y rendimiento por contenedor y base de datos.

| Resource | Límite predeterminado |
| --- | --- |
| Número máximo de RU por contenedor ([modo aprovisionado de rendimiento dedicado](databases-containers-items.md#azure-cosmos-containers)) | 1 000 000 de forma predeterminada. Para aumentarlo, [puede rellenar una incidencia de soporte técnico de Azure](https://docs.microsoft.com/azure/azure-portal/supportability/how-to-create-azure-support-request). |
| Número máximo de RU por base de datos ([modo aprovisionado de rendimiento dedicado](databases-containers-items.md#azure-cosmos-containers)) | 1 000 000 de forma predeterminada. Para aumentarlo, [puede rellenar una incidencia de soporte técnico de Azure](https://docs.microsoft.com/azure/azure-portal/supportability/how-to-create-azure-support-request). |
| Número máximo de RU por partición (lógica) | 10 000 |
| Almacenamiento máximo en todos los elementos por partición (lógica) | 20 GB |
| Número máximo de claves de partición (lógicas) distintas | Sin límite |
| Almacenamiento máximo por contenedor | Sin límite |
| Almacenamiento máximo por base de datos | Sin límite |
| Tamaño máximo de datos adjuntos por cuenta (la característica de datos adjuntos está en desuso) | 2 GB |
| Mínimo de RU requeridas por 1 GB | 10 RU/s |

> [!NOTE]
> Para obtener información sobre el procedimiento recomendado para administrar las cargas de trabajo que tienen claves de partición que requieren límites más altos para el almacenamiento o el rendimiento, consulte [Crear una clave de partición sintética ](synthetic-partition-keys.md).

Un contenedor de Cosmos (o una base de datos de rendimiento compartido) debe tener un rendimiento mínimo de 400 RU/s. A medida que crece el contenedor, el rendimiento mínimo admitido también depende de los factores siguientes:

* Rendimiento máximo que se ha aprovisionado en algún momento en el contenedor. Por ejemplo, si el rendimiento aumentó a 50 000 RU/s, el rendimiento más bajo que se puede aprovisionar sería de 500 RU/s.
* Es decir, el almacenamiento actual en GB del contenedor. Por ejemplo, si el contenedor cuenta con 100 GB de almacenamiento, el rendimiento más bajo que se puede aprovisionar sería de 1000 RU/s.
* El rendimiento mínimo en una base de datos de rendimiento compartido depende del número total de contenedores que ha creado alguna vez en una base de datos de rendimiento compartido, medido en 100 RU/s por contenedor. Por ejemplo, si ha creado cinco contenedores dentro de una base de datos de rendimiento compartido, el rendimiento debe ser de 500 RU/s como mínimo.

El rendimiento actual y mínimo de un contenedor o una base de datos se puede recuperar desde Azure Portal o los SDK. Para obtener más información, consulte [Aprovisionar rendimiento en contenedores y bases de datos](set-throughput.md). 

> [!NOTE]
> En algunos casos, es posible que pueda reducir el rendimiento por debajo del 10 %. Use la API para obtener el valor mínimo exacto de RU por contenedor.

En resumen, los límites de RU de aprovisionamiento mínimos son los siguientes. 

| Resource | Límite predeterminado |
| --- | --- |
| Número mínimo de RU por contenedor ([modo aprovisionado de rendimiento dedicado](databases-containers-items.md#azure-cosmos-containers)) | 400 |
| Número mínimo de RU por base de datos ([modo aprovisionado de rendimiento compartido](databases-containers-items.md#azure-cosmos-containers)) | 400 |
| Número mínimo de RU por contenedor dentro de una base de datos de rendimiento compartido | 100 |

Cosmos DB admite el escalado elástico de rendimiento (RU) por contenedor o base de datos mediante los SDK o el portal. Cada contenedor puede escalar de forma sincrónica e inmediata dentro de un intervalo de escala de 10 a 100 veces, entre los valores mínimos y máximo. Si el valor de rendimiento solicitado está fuera del intervalo, el escalado se realiza asincrónicamente. El escalado asincrónico puede tardar de algunos minutos a varias horas en completarse según el rendimiento solicitado y el tamaño del almacenamiento de datos en el contenedor.  

### <a name="serverless"></a>Sin servidor

[Sin servidor](serverless.md) permite usar los recursos de Azure Cosmos DB de una forma basada en el consumo. En la tabla siguiente se enumeran los límites de capacidad de ráfaga de rendimiento y almacenamiento por contenedor y base de datos.

| Resource | Límite |
| --- | --- |
| Máximo de RU/s por contenedor | 5\.000 |
| Máximo de RU/s por partición (lógica) | 5\.000 |
| Almacenamiento máximo en todos los elementos por partición (lógica) | 20 GB |
| Número máximo de claves de partición (lógicas) distintas | Sin límite |
| Almacenamiento máximo por contenedor | 50 GB |

## <a name="control-plane-operations"></a>Operaciones del plano de control

Puede [aprovisionar y administrar su cuenta de Azure Cosmos](how-to-manage-database-account.md) mediante Azure Portal, Azure PowerShell, la CLI de Azure y plantillas de Azure Resource Manager. En la tabla siguiente se enumeran los límites por suscripción, cuenta y número de operaciones.

| Resource | Límite predeterminado |
| --- | --- |
| Número máximo de cuentas de base de datos por suscripción | 50 de forma predeterminada. Para aumentarlo, [puede rellenar una incidencia de soporte técnico de Azure](https://docs.microsoft.com/azure/azure-portal/supportability/how-to-create-azure-support-request).|
| Número máximo de conmutaciones por error regionales | 1/hora de forma predeterminada. Para aumentarlo, [puede rellenar una incidencia de soporte técnico de Azure](https://docs.microsoft.com/azure/azure-portal/supportability/how-to-create-azure-support-request).|

> [!NOTE]
> Las conmutaciones por error regionales solo se aplican a las cuentas de escritura de una sola región. Las cuentas de escritura de varias regiones no requieren ni tienen ningún límite para el cambio de la región de escritura.

Cosmos DB crea automáticamente copias de seguridad de los datos a intervalos regulares. Para obtener más información sobre los intervalos y las ventanas de retención de copias de seguridad, consulte [Copias de seguridad en línea y restauración de datos a petición en Azure Cosmos DB](online-backup-and-restore.md).

## <a name="per-account-limits"></a>Límites por cuenta

### <a name="provisioned-throughput"></a>Rendimiento aprovisionado

| Resource | Límite predeterminado |
| --- | --- |
| Número máximo de bases de datos | Sin límite |
| Número máximo de contenedores por base de datos con rendimiento compartido |25 |
| Número máximo de contenedores por base de datos o cuenta con rendimiento dedicado  |sin límite |
| Número máximo de regiones | Sin límite (todas las regiones de Azure) |

### <a name="serverless"></a>Sin servidor

| Resource | Límite |
| --- | --- |
| Número máximo de bases de datos | Sin límite |
| Número máximo de contenedores por cuenta  | 100 |
| Número máximo de regiones | 1 (cualquier región de Azure) |

## <a name="per-container-limits"></a>Límites de cada contenedor

En función de la API que utilice, un contenedor de Azure Cosmos puede representar una colección, una tabla o un grafo. Los contenedores admiten configuraciones para [las restricciones de clave única](unique-keys.md), [los procedimientos almacenados, los desencadenadores y las funciones definidas por el usuario (UDF)](stored-procedures-triggers-udfs.md) y [la directiva de indexación](how-to-manage-indexing-policy.md). En la tabla siguiente se enumeran los límites específicos de las configuraciones dentro de un contenedor. 

| Resource | Límite predeterminado |
| --- | --- |
| Longitud máxima del nombre de la base de datos o el contenedor | 255 |
| Procedimientos almacenados máximos por contenedor | 100 <sup>*</sup>|
| Número máximo de UDF por contenedor | 25 <sup>*</sup>|
| Número máximo de rutas de acceso en la directiva de indexación| 100 <sup>*</sup>|
| Número máximo de claves únicas por contenedor|10 <sup>*</sup>|
| Número máximo de rutas de acceso por restricción de clave única|16 <sup>*</sup>|
| Valor máximo de TTL |2147483647|

<sup>*</sup> Para aumentar cualquiera de estos límites por contenedor, puede ponerse en contacto con el equipo de soporte técnico de Azure.

## <a name="per-item-limits"></a>Límites por elemento

En función de la API que use, un elemento de Azure Cosmos puede representar un documento en una colección, una fila en una tabla o un nodo o un borde en un grafo. En la tabla siguiente, se muestran los límites por elemento de Cosmos DB. 

| Resource | Límite predeterminado |
| --- | --- |
| Tamaño máximo de un elemento | 2 MB (longitud en UTF-8 de la representación JSON) |
| Longitud máxima del valor de la clave de partición | 2048 bytes |
| Longitud máxima del valor de id. | 1023 bytes |
| Número máximo de propiedades por elemento | Ningún límite práctico |
| Máxima profundidad de anidamiento | Ningún límite práctico |
| Longitud máxima del nombre de la propiedad | Ningún límite práctico |
| Longitud máxima del nombre de la propiedad | Ningún límite práctico |
| Longitud máxima del valor de propiedad de la cadena | Ningún límite práctico |
| Longitud máxima del valor de propiedad numérico | IEEE754 de doble precisión de 64 bits |
| Valor máximo de TTL |2147483647|

No hay ninguna restricción en las cargas de elementos, como el número de propiedades o la profundidad de anidamiento, salvo las restricciones de longitud en los valores de identificador y clave de partición y la restricción de tamaño general de 2 MB. Es posible que deba configurar la directiva de indexación para contenedores con estructuras de elementos grandes o complejas a fin de reducir el consumo de RU. Consulte [Modelar elementos en Cosmos DB](how-to-model-partition-example.md) para obtener un ejemplo real y los patrones para administrar elementos de gran tamaño.

## <a name="per-request-limits"></a>Límites por solicitud

Azure Cosmos DB admite [operaciones CRUD y de consulta](/rest/api/cosmos-db/) con recursos como contenedores, elementos y bases de datos. También admite [solicitudes de lotes transaccionales](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch) con varios elementos con la misma clave de partición en un contenedor.

| Resource | Límite predeterminado |
| --- | --- |
| Tiempo máximo de ejecución para una sola operación (por ejemplo, la ejecución de un procedimiento almacenado o la recuperación de una página de consulta única)| 5 segundos |
| Tamaño máximo de la solicitud (por ejemplo, procedimiento almacenado, CRUD)| 2 MB |
| Tamaño máximo de respuesta (por ejemplo, consulta paginada) | 4 MB |
| Número máximo de operaciones en un lote transaccional | 100 |

Una vez que una operación como una consulta alcanza el límite del tamaño de respuesta o del tiempo de espera de ejecución, esta devuelve una página de resultados y un token de continuación al cliente para reanudar la ejecución. No hay ningún límite práctico en la duración de la ejecución de una sola consulta en páginas o continuaciones.

Cosmos DB utiliza HMAC para la autorización. Puede usar una clave maestra o [tokens de recursos](secure-access-to-data.md) para tener un control de acceso específico a recursos como contenedores, claves de partición o elementos. En la tabla siguiente se enumeran los límites de los tokens de autorización de Cosmos DB.

| Resource | Límite predeterminado |
| --- | --- |
| Tiempo de expiración máximo del token maestro | 15 minutos  |
| Tiempo de expiración mínimo del token maestro | 10 min  |
| Tiempo de expiración máximo del token de recursos | 24 horas de forma predeterminada. Para aumentarlo, [puede rellenar una incidencia de soporte técnico de Azure](https://docs.microsoft.com/azure/azure-portal/supportability/how-to-create-azure-support-request).|
| Distorsión máxima del reloj para la autorización del token| 15 minutos |

Cosmos DB admite la ejecución de desencadenadores durante las escrituras. El servicio admite un máximo de un desencadenador previo y un desencadenador posterior por operación de escritura. 

## <a name="limits-for-autoscale-provisioned-throughput"></a>Límites del rendimiento aprovisionado de escalabilidad automática

Consulte este artículo sobre [escalabilidad automática](provision-throughput-autoscale.md#autoscale-limits) y las [preguntas frecuentes](autoscale-faq.md#lowering-the-max-rus) para obtener una explicación más detallada de los límites de almacenamiento y rendimiento con escalabilidad automática.

| Resource | Límite predeterminado |
| --- | --- |
| Número máximo de RU/s a los que el sistema se puede escalar |  `Tmax`, el número máximo de RU/s de escalabilidad automática establecido por el usuario|
| Número mínimo de RU/s a los que el sistema se puede escalar | `0.1 * Tmax`|
| RU/s actuales a los que se escala el sistema  |  `0.1*Tmax <= T <= Tmax`, en función del uso|
| Número mínimo de RU/s facturables por hora| `0.1 * Tmax` <br></br>La facturación se realiza por hora, de modo que se le cobra el número máximo de RU/s a los que se escaló el sistema durante esa hora, o `0.1*Tmax`, lo que sea más alto. |
| Valor mínimo del número máximo de RU/s de escalabilidad automática para un contenedor  |  `MAX(4000, highest max RU/s ever provisioned / 10, current storage in GB * 100)` redondeado a los 1000 RU/s más cercanos. |
| Valor mínimo del número máximo de RU/s de escalabilidad automática para una base de datos  |  `MAX(4000, highest max RU/s ever provisioned / 10, current storage in GB * 100,  4000 + (MAX(Container count - 25, 0) * 1000))` redondeado a los 1000 RU/s más cercanos. <br></br>Nota: Si la base de datos tiene más de 25 contenedores, el sistema incrementa el valor mínimo de número máximo de RU/s de escalabilidad automática en 1000 RU/s por cada contenedor adicional. Por ejemplo, si tiene 30 contenedores, el valor más bajo de número máximo de RU/s de escalabilidad automática que puede establecer es 9000 RU/s (para un escalado entre 900 y 9000 RU/s).

## <a name="sql-query-limits"></a>Límites de la consulta SQL

Cosmos DB admite la consulta de elementos mediante [SQL](how-to-sql-query.md). En la tabla siguiente se describen las restricciones en las instrucciones de consulta, por ejemplo, en relación con el número de cláusulas o la longitud de la consulta.

| Resource | Límite predeterminado |
| --- | --- |
| Longitud máxima de la consulta SQL| 256 KB |
| Número máximo de cláusulas JOIN por consulta| 5 <sup>*</sup>|
| Número máximo de UDF por consulta| 10 <sup>*</sup>|
| Número máximo de puntos por polígono| 4096 |
| Número máximo de rutas de acceso incluidas por contenedor| 500 |
| Número máximo de rutas de acceso excluidas por contenedor| 500 |
| Propiedades máximas de un índice compuesto| 8 |

<sup>*</sup> Para aumentar estos límites de consulta SQL, puede ponerse en contacto con el equipo de soporte técnico de Azure.

## <a name="mongodb-api-specific-limits"></a>Límites específicos de la API de MongoDB

Cosmos DB admite el protocolo de conexión de MongoDB para las aplicaciones escritas para MongoDB. Puede encontrar los comandos admitidos y las versiones del protocolo en [Sintaxis y características que admite MongoDB](mongodb-feature-support.md).

En la tabla siguiente se enumeran los límites específicos a la compatibilidad con características de MongoDB. Los otros límites de servicio que se mencionan con la API de SQL (básica) también se aplican a la API de MongoDB.

| Resource | Límite predeterminado |
| --- | --- |
| Tamaño máximo de la memoria de consulta de MongoDB (esta limitación es solo para la versión de servidor 3.2) | 40 MB |
| Tiempo de ejecución máximo para las operaciones de MongoDB| 30 s |
| Tiempo de espera de conexión inactiva para el cierre de la conexión del lado servidor* | 30 minutos |

\* Se recomienda que las aplicaciones cliente establezcan el tiempo de espera de conexión inactiva en la configuración del controlador en 2-3 minutos, ya que el [tiempo de espera predeterminado de Azure LoadBalancer es de 4 minutos](../load-balancer/load-balancer-tcp-idle-timeout.md#tcp-idle-timeout).  Este tiempo de espera garantiza que un equilibrador de carga intermedio entre la máquina cliente y Azure Cosmos DB no cierre las conexiones inactivas.

## <a name="try-cosmos-db-free-limits"></a>Pruebe los límites gratuitos de Cosmos DB

En la tabla siguiente se enumeran los límites de la prueba de encontrará en [Pruebe gratis Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/).

| Resource | Límite predeterminado |
| --- | --- |
| Duración de la prueba | 30 días (se puede solicitar una nueva prueba después de su expiración) <br> Después de la expiración, se elimina la información almacenada. |
| Número máximo de contenedores por suscripción (API, Gremlin y Table API) | 1 |
| Número máximo de contenedores por suscripción (API de MongoDB) | 3 |
| Rendimiento máximo por contenedor | 5000 |
| Rendimiento máximo por base de datos de rendimiento compartido | 20000 |
| Almacenamiento total máximo por cuenta | 10 GB |

La Prueba de Cosmos DB admite la distribución global solo en las regiones de Centro de EE. UU., el Norte de Europa y el Sudeste de Asia. No se pueden crear incidencias de soporte técnico de Azure para las cuentas de prueba de Azure Cosmos DB. Sin embargo, se ofrece soporte técnico para aquellos suscriptores que cuenten con planes de soporte técnico existentes.

## <a name="free-tier-account-limits"></a>Límites de cuenta de nivel Gratis
En la tabla siguiente se enumeran los límites de las [cuentas de nivel Gratis de Azure Cosmos DB](optimize-dev-test.md#azure-cosmos-db-free-tier).

| Resource | Límite predeterminado |
| --- | --- |
| Número de cuentas de nivel Gratis por suscripción de Azure | 1 |
| Duración del descuento por nivel Gratis | Vigencia de la cuenta. Debe participar durante la creación de la cuenta. |
| Número máximo de RU/s gratis | 400 RU/s |
| Almacenamiento máximo gratis | 5 GB |
| Número máximo de bases de datos de rendimiento compartido | 5 |
| Número máximo de contenedores en una base de datos de rendimiento compartido | 25 <br>En las cuentas de nivel Gratis, el número mínimo de RU/s para una base de datos de rendimiento compartido con un máximo de 25 contenedores es 400 RU/s. |

  Además de lo anterior, los [límites por cuenta](#per-account-limits) también se aplican a las cuentas de nivel Gratis.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información acerca de los conceptos básicos de [distribución global](distribute-data-globally.md), [creación de particiones](partitioning-overview.md) y [rendimiento aprovisionado](request-units.md) de Cosmos DB.

Comience a usar Azure Cosmos DB con una de nuestras guías rápidas:

* [Introducción a SQL API de Azure Cosmos DB](create-sql-api-dotnet.md)
* [Introducción a la API de Azure Cosmos DB para MongoDB](create-mongodb-nodejs.md)
* [Introducción a la API Cassandra de Azure Cosmos DB](create-cassandra-dotnet.md)
* [Introducción a Gremlin API de Azure Cosmos DB](create-graph-dotnet.md)
* [Introducción a Table API de Azure Cosmos DB](create-table-dotnet.md)

> [!div class="nextstepaction"]
> [Pruebe gratis Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/)

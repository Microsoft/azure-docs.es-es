---
title: Preguntas más frecuentes sobre diferentes API en Azure Cosmos DB
description: Obtenga respuestas a las preguntas más frecuentes sobre Azure Cosmos DB, un servicio de base de datos con varios modelos distribuido globalmente. Obtenga más información sobre capacidad, niveles de rendimiento y escalado.
author: SnehaGunda
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 12/06/2018
ms.author: sngun
ms.custom: seodec18
ms.openlocfilehash: 8e4ae9b7c96677ce494bea31a49b8db83d6bcb3c
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65793803"
---
# <a name="frequently-asked-questions-about-different-apis-in-azure-cosmos-db"></a>Preguntas más frecuentes sobre diferentes API en Azure Cosmos DB

### <a name="what-happened-to-the-documentdb-api"></a>¿Qué ha ocurrido en la API de DocumentDB?

La API de DocumentDB o la API de SQL (DocumentDB) de Azure Cosmos DB ahora se conocen como API de SQL de Azure Cosmos DB. No es preciso cambiar nada para seguir ejecutando las aplicaciones compiladas con la API de DocumentDB. La funcionalidad no cambia.

Si ya tenía una cuenta de API de DocumentDB, ahora tiene una cuenta de SQL API, sin cambios en la facturación.

### <a name="what-happened-to-azure-documentdb-as-a-service"></a>¿Qué ha ocurrido en Azure DocumentDB como servicio?

El servicio Azure DocumentDB forma ahora parte del servicio Azure Cosmos DB y se manifiesta en forma de SQL API. Las aplicaciones compiladas en Azure DocumentDB se ejecutarán sin realizar ningún cambio en SQL API de Azure Cosmos DB. Cosmos DB también implementa los protocolos de transferencia de [Cassandra](cassandra-introduction.md), [MongoDB](mongodb-introduction.md), [Gremlin](graph-introduction.md) y [Azure Table Storage](table-introduction.md) directamente en el servicio. Esto le permite seleccionar controladores de cliente (y herramientas) para las API de NoSQL que se usan con frecuencia directamente en su base de datos de Cosmos.

### <a name="what-are-the-typical-use-cases-for-azure-cosmos-db"></a>¿Cuáles son los casos de uso típicos de Azure Cosmos DB?

Azure Cosmos DB es una buena elección para las nuevas aplicaciones web, móviles, de juegos y de IoT donde la escala automática, el rendimiento predecible, la orden rápida de tiempos de respuesta en milisegundos y la capacidad para consultar datos sin esquemas son importantes. Se presta a un desarrollo rápido y admite la iteración continua de modelos de datos de aplicaciones. Las aplicaciones que administran contenido y datos generados por el usuario son [casos de uso comunes de Azure Cosmos DB](use-cases.md).

### <a name="how-does-azure-cosmos-db-offer-predictable-performance"></a>¿Cómo ofrece Azure Cosmos DB un rendimiento predecible?

Una [unidad de solicitud](request-units.md) (RU) es la medida de procesamiento de Azure Cosmos DB. 1 RU corresponde al rendimiento de una solicitud GET de un documento de 1 KB. Todas las operaciones de Azure Cosmos DB, incluidas las lecturas, escrituras, consultas SQL y ejecuciones de procedimiento almacenado, tienen un valor de unidad de solicitud determinista en función del rendimiento necesario para completar la operación. En lugar de pensar en la CPU, las E/S y la memoria, y en cómo cada uno de estos aspectos afecta al rendimiento de la aplicación, es mejor que se centre en una unidad única de unidad de solicitud.

Cada contenedor de Azure Cosmos DB puede configurarse con rendimiento aprovisionado en términos de unidades de solicitud del rendimiento por segundo. Para las aplicaciones de cualquier escala, puede realizar pruebas comparativas de las solicitudes individuales para medir los valores de unidad de solicitud y aprovisionar un contenedor para administrar el total de unidades de solicitud en todas las solicitudes. También puede escalar o reducir verticalmente el rendimiento del contenedor a medida que evolucionen las necesidades de la aplicación. Para obtener más información acerca de las unidades de solicitud y ayuda a determinar sus necesidades de contenedor, pruebe el [Calculadora de rendimiento](https://www.documentdb.com/capacityplanner).

### <a name="how-does-azure-cosmos-db-support-various-data-models-such-as-keyvalue-columnar-document-and-graph"></a>¿Por qué Azure Cosmos DB admite varios modelos de datos, como pares clave-valor, columnas, documentos y gráficos?

Los modelos de datos de pares clave-valor (tabla), columnas, documentos y gráficos se admiten de forma nativa porque Azure Cosmos DB se basa en el diseño de ARS (átomos, registros y secuencias). Los átomos, los registros y las secuencias se pueden asignar y proyectar con facilidad en varios modelos de datos. Ya se encuentran disponibles las API para un subconjunto de modelos (SQL, MongoDB, Table y Gremlin), mientras que otras que son específicas para modelos de datos adicionales estarán disponibles en el futuro.

Azure Cosmos DB tiene un motor de indexación independiente del esquema capaz de indexar automáticamente todos los datos ingeridos sin necesidad de que el desarrollador proporcione índices de esquema o secundarios. El motor se basa en un conjunto de diseños de índices lógicos (invertidos, en columnas y en árbol) que desacopla el diseño de almacenamiento del índice y los subsistemas de procesamiento de consultas. Cosmos DB también tiene la capacidad de admitir un conjunto de API y protocolos de conexión de forma extensible y convertirlos de forma eficaz en el modelo de datos principal (1) y los diseños de índices lógicos (2), lo que le permite admitir con exclusividad más de un modelo de datos de forma nativa.

### <a name="is-azure-cosmos-db-hipaa-compliant"></a>¿Es compatible Azure Cosmos DB con HIPAA?

Sí, Azure Cosmos DB es compatible con HIPAA. HIPAA establece los requisitos para el uso, la divulgación y la protección de información de mantenimiento identificable individualmente. Para más información, consulte el [Centro de confianza de Microsoft](https://www.microsoft.com/en-us/TrustCenter/Compliance/HIPAA).

### <a name="what-are-the-storage-limits-of-azure-cosmos-db"></a>¿Cuáles son los límites de almacenamiento de Azure Cosmos DB?

No existe ningún límite en la cantidad total de datos que puede almacenar un contenedor en Azure Cosmos DB.

### <a name="what-are-the-throughput-limits-of-azure-cosmos-db"></a>¿Cuáles son los límites de rendimiento de Azure Cosmos DB?

No existe ningún límite en la cantidad total de rendimiento que admite un contenedor en Azure Cosmos DB. La idea clave es distribuir la carga de trabajo de manera más o menos uniforme entre una cantidad suficientemente grande de claves partición.

### <a name="are-direct-and-gateway-connectivity-modes-encrypted"></a>¿Se cifran los modos de conectividad directa y por puerta de enlace?

Sí, ambos modos se cifran siempre totalmente.

### <a name="how-much-does-azure-cosmos-db-cost"></a>¿Cuánto cuesta Azure Cosmos DB?

Para más información, vea la página de [precios de Azure Cosmos DB](https://azure.microsoft.com/pricing/details/cosmos-db/). Los cargos por uso de Azure Cosmos DB se determinan según la cantidad de contenedores aprovisionados, el número de horas que los contenedores estuvieron conectados y el rendimiento aprovisionado de cada contenedor.

### <a name="is-a-free-account-available"></a>¿Existe una cuenta gratuita disponible?

Sí, puede registrarse para una cuenta de tiempo limitado sin cargo, con ningún compromiso. Para registrarse, visite [Pruebe gratis Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) o consulte la sección de [preguntas más frecuentes sobre Azure Cosmos DB](#try-cosmos-db).

Si es la primera vez que usa Azure, regístrese para obtener una [cuenta gratuita de Azure](https://azure.microsoft.com/free/), que ofrece 30 días y un crédito para que pruebe todos los servicios de Azure. Si tiene una suscripción a Visual Studio, puede recibir [créditos gratis de Azure](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) para usarlos en cualquier servicio de Azure.

También puede usar el [Emulador de Azure Cosmos DB](local-emulator.md) para desarrollar y probar su aplicación localmente, de forma gratuita, sin necesidad de crear una suscripción de Azure. Cuando esté satisfecho con el funcionamiento de la aplicación en el Emulador, puede cambiar a una cuenta de Azure Cosmos DB en la nube.

### <a name="how-can-i-get-additional-help-with-azure-cosmos-db"></a>¿Cómo puedo obtener ayuda adicional con Azure Cosmos DB?

Para formular una pregunta técnica, puede realizar una publicación en alguno de estos dos foros de preguntas y respuestas:

* [Foro de MSDN](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurecosmosdb)
* [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-cosmosdb). Stack Overflow es la mejor opción para preguntas de programación. Asegúrese de que su pregunta es [relevante](https://stackoverflow.com/help/on-topic) y [proporcione tantos detalles como sea posible, formulando la pregunta de forma clara y responsable](https://stackoverflow.com/help/how-to-ask).

Para solicitar nuevas características, cree una nueva solicitud en [UserVoice](https://feedback.azure.com/forums/263030-azure-cosmos-db).

Para corregir un problema con la cuenta, envíe una [solicitud de soporte técnico](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) en Azure Portal.

Se pueden enviar otras consultas al equipo a [askcosmosdb@microsoft.com](mailto:askcosmosdb@microsoft.com); sin embargo, este no es un alias de soporte técnico.

## <a id="try-cosmos-db"></a>Prueba de las suscripciones de Azure Cosmos DB

Ya puede disfrutar de Azure Cosmos DB durante un tiempo limitado sin una suscripción, y sin que le suponga ningún gasto ni compromiso. Para registrarse en una suscripción de prueba de Azure Cosmos DB, vaya a [Pruebe gratis Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/). Esta suscripción es independiente de la [evaluación gratuita de Azure](https://azure.microsoft.com/free/) y puede utilizarse junto con una evaluación gratuita o una suscripción de pago de Azure.

Las suscripciones de prueba de Azure Cosmos DB aparecen en Azure Portal junto con otras suscripciones asociadas con el identificador de usuario.

Las condiciones siguientes se aplican a las suscripciones de prueba de Azure Cosmos DB:

* Un contenedor por cada suscripción para las cuentas de SQL, la Gremlin API y Table.
* Hasta 3 colecciones por suscripción para las cuentas de MongoDB.
* 10 GB de capacidad de almacenamiento.
* La replicación global está disponible en las [regiones de Azure](https://azure.microsoft.com/regions/) siguientes: Centro de EE. UU., Europa del Norte y Sudeste Asiático
* Rendimiento máximo de 5 K RU/s cuando se aprovisionan en el nivel de contenedor.
* Rendimiento máximo de 20 K RU/s cuando se aprovisionan en el nivel de base de datos.
* Las suscripciones expiran después de 30 días y pueden ampliarse hasta un máximo de 31 días en total.
* No se pueden crear incidencias de soporte técnico de Azure para las cuentas de prueba de Azure Cosmos DB; sin embargo, se proporciona soporte técnico para aquellos suscriptores con planes de soporte técnico ya existentes.

## <a name="set-up-azure-cosmos-db"></a>Configuración de Azure Cosmos DB

### <a name="how-do-i-sign-up-for-azure-cosmos-db"></a>¿Cómo me registro en Azure Cosmos DB?

Azure Cosmos DB está disponible en Azure Portal. En primer lugar, regístrese para obtener una suscripción de Azure. Una vez se haya registrado, puede agregar una cuenta de Azure Cosmos DB a su suscripción de Azure.

### <a name="what-is-a-master-key"></a>¿Qué es una clave maestra?

Una clave maestra es un token de seguridad para acceder a todos los recursos de una cuenta. Los usuarios con esta clave tienen acceso de lectura y escritura a todos los recursos de la cuenta de base de datos. Tenga cuidado cuando distribuya claves maestras. Las claves maestras principal y secundaria están disponibles en la hoja **Claves** de [Azure Portal][azure-portal]. Para más información sobre las claves, vea [Visualización, copia y regeneración de las claves de acceso](manage-with-cli.md#list-account-keys).

### <a name="what-are-the-regions-that-preferredlocations-can-be-set-to"></a>¿En qué regiones se puede establecer PreferredLocations?

El valor PreferredLocations se puede establecer en cualquiera de las regiones de Azure en las que Cosmos DB está disponible. Para obtener una lista de las regiones disponibles, vea [Regiones de Azure](https://azure.microsoft.com/regions/).

### <a name="is-there-anything-i-should-be-aware-of-when-distributing-data-across-the-world-via-the-azure-datacenters"></a>¿Hay algo que deba tener en cuenta al distribuir datos en todo el mundo a través de centros de datos de Azure?

Azure Cosmos DB se ofrece en todas las regiones de Azure, tal y como se especifica en la página [Regiones de Azure](https://azure.microsoft.com/regions/). Dado que se trata del servicio principal, cada nuevo centro de datos está presente en Azure Cosmos DB.

Al establecer una región, debe recordar que Cosmos DB respeta las nubes independientes y de administración pública. Es decir, si crea una cuenta en una [región soberana](https://azure.microsoft.com/global-infrastructure/), no puede replicarla fuera de dicha [región](https://azure.microsoft.com/global-infrastructure/). De manera similar, tampoco puede habilitar la replicación en otras ubicaciones soberanas desde una cuenta externa.

### <a name="is-it-possible-to-switch-from-container-level-throughput-provisioning-to-database-level-throughput-provisioning-or-vice-versa"></a>¿Es posible pasar del aprovisionamiento de rendimiento del nivel de contenedor al aprovisionamiento de rendimiento del nivel de base de datos? O viceversa

El aprovisionamiento del rendimiento de nivel de base de datos y de contenedor son ofertas diferentes y cambiar entre cualquiera de estos requiera migrar datos de origen a destino. Lo que significa que deberá crear una nueva base de datos o una colección nueva y luego migrar datos mediante el uso de [biblioteca de ejecutor masiva](bulk-executor-overview.md) o [Azure Data Factory](../data-factory/connector-azure-cosmos-db.md).

### <a name="does-azure-cosmosdb-support-time-series-analysis"></a>¿Azure Cosmos DB admite el análisis de series temporales?

Sí, Azure Cosmos DB admite el análisis de series temporales, este es un ejemplo de un [patrón de serie temporal](https://github.com/Azure/azure-cosmosdb-dotnet/tree/master/samples/Patterns). En este ejemplo se muestra cómo usar la fuente de cambios para generar vistas agregadas con los datos de las series temporales. Puede ampliar este enfoque mediante el uso de streaming de Spark u otro procesador de datos de streaming.

## <a name="sql-api"></a>API DE SQL

### <a name="how-do-i-start-developing-against-the-sql-api"></a>¿Cómo se empieza a desarrollar con SQL API?

En primer lugar, debe registrarse para obtener una suscripción de Azure. Una vez registrado para obtener una suscripción de Azure, puede agregar un contenedor de SQL API a la suscripción de Azure. Para obtener instrucciones sobre cómo agregar una cuenta de Azure Cosmos DB, vea [Creación de una cuenta de base de datos de Azure Cosmos DB](create-sql-api-dotnet.md#create-account).

[SDK](sql-api-sdk-dotnet.md) están disponibles para NET, Python, Node.js, JavaScript y Java. Los desarrolladores también pueden usar las [API de HTTP RESTful](/rest/api/cosmos-db/) para interactuar con recursos de Azure Cosmos DB en varias plataformas y lenguajes.

### <a name="can-i-access-some-ready-made-samples-to-get-a-head-start"></a>¿Se puede acceder a algunos ejemplos disponibles que me puedan ayudar a empezar?

En GitHub se encuentran disponibles ejemplos de los SDK de [.NET](sql-api-dotnet-samples.md), [Java](https://github.com/Azure/azure-documentdb-java), [Node.js](sql-api-nodejs-samples.md) y [Python](sql-api-python-samples.md) de SQL API.

### <a name="does-the-sql-api-database-support-schema-free-data"></a>¿Las bases de datos de SQL API admiten datos sin esquemas?

Sí, SQL API permite que las aplicaciones almacenen documentos JSON arbitrarios sin definiciones ni sugerencias de esquemas. Los datos están disponibles inmediatamente para consulta a través de la interfaz de consulta SQL de Azure Cosmos DB.

### <a name="does-the-sql-api-support-acid-transactions"></a>¿SQL API admite transacciones ACID?

Sí, SQL API admite transacciones entre documentos expresadas como procedimientos almacenados y desencadenadores JavaScript. Las transacciones se limitan a una única partición dentro de cada contenedor y se ejecutan con semántica ACID, como "todo o nada", aisladas de otras solicitudes de usuario y código que se ejecutan a la vez. Si se producen excepciones por la ejecución en el servidor de código de aplicación JavaScript, la transacción entera se revierte. 

### <a name="what-is-a-container"></a>¿Qué es un contenedor?

Un contenedor es un grupo de documentos y su lógica de aplicación de JavaScript asociada. Un contenedor es una entidad facturable, en la que el [costo](performance-levels.md) se determina en función del rendimiento y del almacenamiento utilizado. Los contenedores pueden abarcar una o varias particiones o uno o varios servidores y se pueden escalar para administrar volúmenes prácticamente ilimitados de almacenamiento o rendimiento.

* Para las API de SQL y de Cosmos DB de las cuentas de MongoDB, un contenedor se asigna a una colección.
* Para las cuentas de Cassandra y Table API, se asigna un contenedor a una tabla.
* Para las cuentas de API de Gremlin, se asigna un contenedor a un gráfico.

Los contenedores también son las entidades de facturación de Azure Cosmos DB. Cada contenedor se factura por hora según el rendimiento aprovisionado y el espacio de almacenamiento usado. Para más información, vea los [precios de Azure Cosmos DB](https://azure.microsoft.com/pricing/details/cosmos-db/).

### <a name="how-do-i-create-a-database"></a>¿Cómo se crea una base de datos?

Puede crear bases de datos mediante [Azure Portal](https://portal.azure.com), tal y como se describe en [Agregar una colección](create-sql-api-dotnet.md#create-collection-database), uno de los [SDK de Azure Cosmos DB](sql-api-sdk-dotnet.md) o las [API de REST](/rest/api/cosmos-db/).

### <a name="how-do-i-set-up-users-and-permissions"></a>¿Cómo se configuran los usuarios y los permisos?

Puede crear usuarios y permisos mediante uno de los [SDK de la API de Cosmos DB](sql-api-sdk-dotnet.md) o de las [API de REST](/rest/api/cosmos-db/).

### <a name="does-the-sql-api-support-sql"></a>¿SQL API admite SQL?

El lenguaje de consulta SQL que admiten las cuentas de SQL API es un subconjunto mejorado de la funcionalidad de consulta admitida por SQL. El lenguaje de consulta SQL de Azure Cosmos DB ofrece complejos operadores relacionales y jerárquicos y extensibilidad a través de funciones definidas por el usuario (UDF) basadas en JavaScript. La sintaxis JSON permite el modelado de documentos JSON como árboles con nodos etiquetados, algo que usan las técnicas de indexación automática de Azure Cosmos DB y el dialecto de consulta SQL de Azure Cosmos DB. Para obtener información sobre el uso de la gramática de SQL, vea el artículo [SQL Query][query].

### <a name="does-the-sql-api-support-sql-aggregation-functions"></a>¿SQL API admite las funciones de agregación de SQL?

SQL API admite la agregación de baja latencia a cualquier escala a través de las funciones de agregado `COUNT`, `MIN`, `MAX`, `AVG` y `SUM` a través de la gramática de SQL. Para más información, consulte [Funciones de agregado](how-to-sql-query.md#Aggregates).

### <a name="how-does-the-sql-api-provide-concurrency"></a>¿De qué modo proporciona simultaneidad SQL API?

SQL API admite el control de simultaneidad optimista (OCC) a través de etiquetas de entidad HTTP o ETag. Cada recurso de SQL API tiene una ETag, que se establece en el servidor cada vez que se actualiza un documento. El encabezado de la ETag y el valor actual se incluyen en todos los mensajes de respuesta. Las etiquetas ETag pueden utilizarse con el encabezado If-Match para permitir que el servidor decida si debe actualizarse un recurso. El valor de If-Match es el valor de ETag con el que se va a cotejar. Si el valor de ETag coincide con el valor de ETag del servidor, el recurso se actualiza. Si el valor de ETag ya no es el actual, el servidor rechaza la operación con un código de respuesta "HTTP 412 Precondition failure" (HTTP 412: error de condición previa). Después, el cliente recupera el recurso para obtener el valor de Etag actual del recurso. Además, las etiquetas ETag pueden emplearse con el encabezado If-None-Match para determinar si hay que volver a recuperar un recurso.

Para usar la simultaneidad optimista en .NET, utilice la clase [AccessCondition](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.accesscondition.aspx) . Para ver un ejemplo de .NET, vea la sección [Program.cs](https://github.com/Azure/azure-documentdb-dotnet/blob/master/samples/code-samples/DocumentManagement/Program.cs) en el ejemplo del artículo DocumentManagement de GitHub.

### <a name="how-do-i-perform-transactions-in-the-sql-api"></a>¿Cómo se realizan transacciones en SQL API?

SQL API admite transacciones integradas en el lenguaje a través de procedimientos almacenados y desencadenadores JavaScript. Todas las operaciones de base de datos contenidas en scripts se ejecutan con aislamiento de instantánea. Si se trata de una colección de partición única, el ámbito de ejecución es la colección. Si la colección se particiona, el ámbito de la ejecución son documentos con el mismo valor de clave de partición que hay dentro de la colección. Una instantánea de las versiones de documento (etiquetas ETag) se toma al comienzo de la transacción y solo se confirma si el script se ejecuta correctamente. Si el JavaScript produce un error, la transacción se revierte. Para más información, vea [Programación de JavaScript en el lado del servidor de Azure Cosmos DB](stored-procedures-triggers-udfs.md).

### <a name="how-can-i-bulk-insert-documents-into-cosmos-db"></a>¿Cómo se pueden insertar documentos de forma masiva en Cosmos DB?

Hay dos maneras de insertar documentos de forma masiva en Azure Cosmos DB:

* La herramienta de ejecutor en masa, como se describe en [Uso de la biblioteca de BulkExecutor en .NET](bulk-executor-dot-net.md) y [Uso de la biblioteca de BulkExecutor en Java](bulk-executor-java.md).
* La herramienta de migración de datos, como se describe en [Herramienta de migración de bases de datos de Azure Cosmos DB](import-data.md).
* Procedimientos almacenados, como se describe en [Programación de JavaScript en el lado del servidor de Azure Cosmos DB](stored-procedures-triggers-udfs.md).

### <a name="does-the-sql-api-support-resource-link-caching"></a>¿SQL API admite el almacenamiento en caché de vínculos de recursos?

Sí. Como Azure Cosmos DB es un servicio RESTful, los vínculos de recursos son inmutables y se pueden almacenar en caché. Los clientes de SQL API pueden especificar un encabezado "If-None-Match" para lecturas en cualquier documento o colección de tipo recurso y actualizar sus copias locales cuando la versión del servidor cambie.

### <a name="is-a-local-instance-of-sql-api-available"></a>¿Hay disponible una instancia local de SQL API?

Sí. El [Emulador de Azure Cosmos DB](local-emulator.md) proporciona una emulación de gran fidelidad del servicio Cosmos DB. Admite una funcionalidad que es idéntica a la de Azure Cosmos DB, incluida la compatibilidad con la creación y la consulta de documentos JSON, el aprovisionamiento y el escalado de colecciones y la ejecución de desencadenadores y procedimientos almacenados. Puede desarrollar y probar aplicaciones mediante el Emulador de Azure Cosmos DB e implementarlas en Azure a escala global con tan solo un cambio de configuración en el punto de conexión de la conexión de Azure Cosmos DB.

### <a name="why-are-long-floating-point-values-in-a-document-rounded-when-viewed-from-data-explorer-in-the-portal"></a>¿Por qué se redondean los valores de punto flotante largos cuando se ven desde el explorador de datos en el portal?

Se trata de una limitación de JavaScript. JavaScript usa números de formato de punto flotante de precisión doble que se especifica en IEEE 754 y con seguridad que puede contener números entre-(2<sup>53</sup> -1) y 2<sup>53</sup>-1 (es decir, 9007199254740991) solamente.

### <a name="where-are-permissions-allowed-in-the-object-hierarchy"></a>¿Donde se permiten los permisos en la jerarquía de objetos?

La creación de permisos mediante ResourceTokens se permite tanto en el nivel de contenedor como en sus descendientes (por ejemplo, documentos, datos adjuntos). Esto implica que la creación de un permiso en el nivel de base de datos o de cuenta no está permitido actualmente.

## <a name="azure-cosmos-dbs-api-for-mongodb"></a>API de Azure Cosmos DB para MongoDB

### <a name="what-is-the-azure-cosmos-dbs-api-for-mongodb"></a>¿Qué es la API de Azure Cosmos DB para MongoDB?

API de Azure Cosmos DB para MongoDB es una capa de compatibilidad de protocolo de conexión que permite a las aplicaciones comunicarse fácilmente y transparente con el motor de base de datos de Azure Cosmos DB nativo mediante existente, admitido por la Comunidad de SDK y de los controladores de MongoDB. Los desarrolladores ahora pueden usar cadenas de herramientas de MongoDB existentes y las habilidades para crear aplicaciones que aprovechan las ventajas de Azure Cosmos DB. Los desarrolladores aprovechar las funcionalidades únicas de Azure Cosmos DB, que incluyen la distribución global con la replicación de varios maestra, mantenimiento de copia de seguridad, la indexación automática, con respaldo financiero acuerdos de nivel de servicio (SLA) etcetera.

### <a name="how-do-i-connect-to-my-database"></a>¿Cómo me conecto a mi base de datos?

La forma más rápida de conectarse a una base de datos de Cosmos con la API de Azure Cosmos DB para MongoDB es a través de [Azure Portal](https://portal.azure.com). Vaya a la cuenta y luego, en el menú de navegación izquierdo, haga clic en **Inicio rápido**. Guía de inicio rápido es la mejor manera de obtener fragmentos de código para conectarse a la base de datos.

Azure Cosmos DB aplica estándares y requisitos de seguridad estrictos. Las cuentas de Azure Cosmos DB requieren autenticación y comunicación segura a través de SSL, por lo que debe usar TLS v1.2.

Para más información, consulte [Conectar una aplicación de MongoDB a Azure Cosmos DB](connect-mongodb-account.md).

### <a name="are-there-additional-error-codes-that-i-need-to-deal-with-while-using-azure-cosmos-dbs-api-for-mongodb"></a>¿Existen otros códigos de error adicionales que tenga que solucionar al usar la API de Azure Cosmos DB para MongoDB?

Además de los códigos de error comunes de MongoDB, la API de Azure Cosmos DB para MongoDB tiene sus propios códigos de error:

| Error               | Código  | DESCRIPCIÓN  | Solución  |
|---------------------|-------|--------------|-----------|
| TooManyRequests     | 16500 | El número total de unidades de solicitud consumidas es mayor que la tasa de unidades de solicitud aprovisionadas para la colección y se ha limitado. | Considere la posibilidad de escalar el rendimiento asignado a un contenedor o un conjunto de contenedores desde Azure Portal o reintentarlo de nuevo. |
| ExceededMemoryLimit | 16501 | Como se trata de un servicio de varios inquilinos, la operación ha superado la asignación de memoria del cliente. | Reduzca el ámbito de la operación a través de criterios de consulta más restrictivos o póngase en contacto con soporte técnico desde [Azure Portal](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade). <br><br>Example: <em>&nbsp;&nbsp;&nbsp;&nbsp;db.getCollection('users').aggregate([<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{$match: {nombre: "Andy"}}, <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{$sort: {age: -1}}<br>&nbsp;&nbsp;&nbsp;&nbsp;])</em>) |

### <a name="is-the-simba-driver-for-mongodb-supported-for-use-with-azure-cosmos-dbs-api-for-mongodb"></a>¿Se puede usar el controlador Simba para la API de Azure Cosmos DB para MongoDB?

Sí, puede usar el controlador ODBC para Mongo de Simba con la API de Azure Cosmos DB para MongoDB.

## <a id="table"></a>Table API

### <a name="how-can-i-use-the-table-api-offering"></a>¿Cómo se puede usar la oferta de Table API?

Table API de Azure Cosmos DB está disponible en [Azure Portal][azure-portal]. En primer lugar, debe registrarse para obtener una suscripción de Azure. Una vez registrado, puede agregar una cuenta de Table API de Azure Cosmos DB a la suscripción de Azure y luego agregar tablas a la cuenta.

Puede encontrar los lenguajes admitidos y las guías de inicio rápido asociadas en la [Introducción a Table API de Azure Cosmos](table-introduction.md).

### <a name="do-i-need-a-new-sdk-to-use-the-table-api"></a>¿Se necesita un nuevo SDK para usar Table API?

No, los SDK de almacenamiento existentes deberían seguir funcionando. Sin embargo, se recomienda obtener siempre los SDK más recientes para lograr la mejor compatibilidad y, en muchos casos, un rendimiento superior. Vea la lista de lenguajes disponibles en la [Introducción a Table API de Azure Cosmos DB](table-introduction.md).

### <a name="where-is-table-api-not-identical-with-azure-table-storage-behavior"></a>¿En qué se diferencia Table API del comportamiento de Azure Table Storage?

Hay algunas diferencias de comportamiento que los usuarios acostumbrados a Azure Table Storage que deseen crear tablas con Table API de Azure Cosmos DB deben tener en cuenta:

* Table API de Azure Cosmos DB utiliza un modelo de capacidad reservada con el fin de asegurar un rendimiento garantizado, pero esto significa que se paga por la capacidad en cuanto se crea la tabla, incluso aunque no se esté usando. Con Azure Table Storage, solo se paga por la capacidad que se utilice. Esto ayuda a explicar por qué Table API puede ofrecer un Acuerdo de Nivel de Servicio con lecturas de 10 ms y escrituras de 15 ms en el percentil 99 mientras que Azure Table Storage ofrece un Acuerdo de Nivel de Servicio de 10 segundos. Pero, como consecuencia, con las tablas de Table API, incluso las tablas vacías sin ninguna solicitud cuestan dinero, con el fin de garantizar que la capacidad está disponible para atender las solicitudes que les llegan en el Acuerdo de Nivel de Servicio ofrecido por Azure Cosmos DB.
* Resultados de la consulta devueltos por la API de tabla no se ordenan en orden de clave de fila/partición tal como están en Azure Table storage.
* Las claves de fila solo pueden ser de hasta 255 bytes
* Los lotes solo pueden contener hasta 2 MB.
* CORS no se admite actualmente.
* Los nombres de tabla en Azure Table storage no distingue mayúsculas de minúsculas, pero están en Azure Cosmos DB Table API
* Algunos de los formatos internos de Azure Cosmos DB para codificar la información, como los campos binarios, no son actualmente tan eficientes como se podría desear. Por lo tanto, esto puede causar limitaciones inesperadas en el tamaño de los datos. Por ejemplo, actualmente no se puede utilizar el Meg completo de una entidad de tabla para almacenar datos binarios porque la codificación aumenta el tamaño de los datos.
* Nombre de propiedad de entidad 'ID' no admitido actualmente
* TableQuery TakeCount no se limita a 1000.

En cuanto a la API REST, hay una serie de opciones de puntos de conexión o consulta que no son compatibles con Table API de Azure Cosmos DB:

| Métodos de REST | Opción de punto de conexión o consulta de REST | Direcciones URL de documento | Explicación |
| ------------| ------------- | ---------- | ----------- |
| GET, PUT | /?restype=service@comp=properties| [Establecer propiedades de Table service](https://docs.microsoft.com/rest/api/storageservices/set-table-service-properties) y [Obtener propiedades de Table service](https://docs.microsoft.com/rest/api/storageservices/get-table-service-properties) | Este punto de conexión se usa para establecer reglas de CORS, de configuración del análisis de almacenamiento y de configuración del registro. CORS no se admite actualmente y el análisis y el registro se tratan de forma diferente en Azure Cosmos DB y en Azure Storage Tables |
| OPCIONES | /\<table-resource-name> | [Solicitud de tabla preparatoria CORS](https://docs.microsoft.com/rest/api/storageservices/preflight-table-request) | Esto forma parte de CORS, el cual Azure Cosmos DB no admite actualmente. |
| GET | /?restype=service@comp=stats | [Obtener estadísticas de Table service](https://docs.microsoft.com/rest/api/storageservices/get-table-service-stats) | Proporciona información sobre la rapidez con que los datos se están replicando datos entre los elementos principales y los secundarios. Esto no es necesario en Cosmos DB ya que la replicación es parte de las escrituras. |
| GET, PUT | /mytable?comp=acl | [Obtener tabla ACL](https://docs.microsoft.com/rest/api/storageservices/get-table-acl) y [Definir ACL de tabla](https://docs.microsoft.com/rest/api/storageservices/set-table-acl) | Así se obtienen y establecen las directivas de acceso almacenadas que se usan para administrar las firmas de acceso compartido (SAS). Aunque SAS se admite, las firmas se establecen y administran de manera diferente. |

Además, Table API de Azure Cosmos DB solo es compatible con el formato JSON, no con ATOM.

Aunque Azure Cosmos DB admite firmas de acceso compartido (SAS), hay determinadas directivas que no admite, en concreto las relacionadas con las operaciones de administración como el derecho para crear nuevas tablas.

Para el SDK de .NET en concreto, hay algunas clases y métodos que Azure Cosmos DB no admite actualmente.

| Clase | Método no admitido |
|-------|-------- |
| CloudTableClient | \*ServiceProperties* |
|                  | \*ServiceStats* |
| CloudTable | SetPermissions* |
|            | GetPermissions* |
| TableServiceContext | * (esta clase está en desuso) |
| TableServiceEntity | " " |
| TableServiceExtensions | " " |
| TableServiceQuery | " " |

Si alguna de estas diferencias supone un problema para el proyecto, póngase en contacto con [askcosmosdb@microsoft.com](mailto:askcosmosdb@microsoft.com) y háganoslo saber.

### <a name="how-do-i-provide-feedback-about-the-sdk-or-bugs"></a>¿Cómo se pueden hacer comentarios sobre el SDK o los errores?

Puede proporcionar comentarios de alguna de las siguientes formas:

* [Voz del usuario](https://feedback.azure.com/forums/263030-azure-cosmos-db)
* [Foro de MSDN](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurecosmosdb)
* [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-cosmosdb). Stack Overflow es la mejor opción para preguntas de programación. Asegúrese de que su pregunta es [relevante](https://stackoverflow.com/help/on-topic) y [proporcione tantos detalles como sea posible, formulando la pregunta de forma clara y responsable](https://stackoverflow.com/help/how-to-ask).

### <a name="what-is-the-connection-string-that-i-need-to-use-to-connect-to-the-table-api"></a>¿Qué cadena de conexión hay que usar para conectarse a Table API?

La cadena de conexión es:

```
DefaultEndpointsProtocol=https;AccountName=<AccountNamefromCosmos DB;AccountKey=<FromKeysPaneofCosmosDB>;TableEndpoint=https://<AccountName>.table.cosmosdb.azure.com
```

Puede obtener la cadena de conexión en la página de cadenas de conexión de Azure Portal.

### <a name="how-do-i-override-the-config-settings-for-the-request-options-in-the-net-sdk-for-the-table-api"></a>¿Cómo se invalidan los valores de configuración de las opciones de solicitud del SDK para .NET para Table API?

Algunas opciones de configuración se tratan en el método CreateCloudTableClient y otros a través del archivo app.config en la sección appSettings en la aplicación cliente. Para información sobre las opciones de configuración, vea [Funcionalidades de Azure Cosmos DB](tutorial-develop-table-dotnet.md).

### <a name="are-there-any-changes-for-customers-who-are-using-the-existing-azure-table-storage-sdks"></a>¿Existen cambios para los clientes que usan los SDK existentes de Azure Table Storage?

Ninguno. No hay ningún cambio para los clientes nuevos o existentes que usan los SDK existentes de Azure Table Storage.

### <a name="how-do-i-view-table-data-thats-stored-in-azure-cosmos-db-for-use-with-the-table-api"></a>¿Cómo se ven los datos de tablas que se almacenan en Azure Cosmos DB para usarlos con Table API?

Puede usar Azure Portal para examinar los datos. También puede usar el código de Table API o las herramientas que se mencionan en la respuesta siguiente.

### <a name="which-tools-work-with-the-table-api"></a>¿Qué herramientas funcionarán con Table API?

Puede usar el [Explorador de Azure Storage](https://docs.microsoft.com/azure/vs-azure-tools-storage-manage-with-storage-explorer).

Las herramientas que tengan suficiente flexibilidad como para aceptar una cadena de conexión en el formato especificado antes son compatibles con la nueva Table API. En la página [Herramientas de cliente de Azure Storage](../storage/common/storage-explorers.md) se ofrece una lista de herramientas de tabla.

### <a name="is-the-concurrency-on-operations-controlled"></a>¿Se controla la simultaneidad en las operaciones?

Sí, se ofrece simultaneidad optimista a través del mecanismo de ETag.

### <a name="is-the-odata-query-model-supported-for-entities"></a>¿Es el modelo de consulta de OData compatible con entidades?

Sí. Table API admite la consulta OData y la consulta LINQ.

### <a name="can-i-connect-to-azure-table-storage-and-azure-cosmos-db-table-api-side-by-side-in-the-same-application"></a>¿Puedo conectarme en paralelo a Azure Table Storage y a Table API de Azure Cosmos DB en la misma aplicación?

Sí, puede conectarse mediante la creación de dos instancias independientes de CloudTableClient, donde cada una apunte a su propio URI a través de la cadena de conexión.

### <a name="how-do-i-migrate-an-existing-azure-table-storage-application-to-this-offering"></a>¿Cómo se puede migrar la aplicación de Azure Table Storage existente a esta nueva oferta?

Tanto [AzCopy](https://docs.microsoft.com/azure/storage/common/storage-use-azcopy) como la [Herramienta de migración de datos de Azure Cosmos DB](import-data.md) se admiten.

### <a name="how-is-expansion-of-the-storage-size-done-for-this-service-if-for-example-i-start-with-n-gb-of-data-and-my-data-will-grow-to-1-tb-over-time"></a>¿Cómo se realiza la expansión del tamaño de almacenamiento para este servicio si, por ejemplo, se empieza por *n* GB de datos y con el tiempo los datos crecen a 1 TB?

Azure Cosmos DB está diseñado para ofrecer almacenamiento ilimitado mediante el uso de escalado horizontal. Nuestro servicio puede supervisar y aumentar el almacenamiento de forma eficaz.

### <a name="how-do-i-monitor-the-table-api-offering"></a>¿Cómo se supervisa la oferta de Table API?

Puede usar el panel **Métricas** de Table API para supervisar las solicitudes y el uso del almacenamiento.

### <a name="how-do-i-calculate-the-throughput-i-require"></a>¿Cómo se puede calcular el rendimiento que se necesita?

Puede usar la calculadora de capacidad para calcular el valor de TableThroughput necesario para las operaciones. Para más información, vea [Estimate Request Units and Data Storage](https://www.documentdb.com/capacityplanner) (Cálculo de unidades de solicitud y almacenamiento de datos). Normalmente, puede mostrar la entidad como JSON e indicar las cantidades para las operaciones.

### <a name="can-i-use-the-table-api-sdk-locally-with-the-emulator"></a>¿Se puede usar el nuevo SDK de Table API localmente con el emulador?

De momento, no.

### <a name="can-my-existing-application-work-with-the-table-api"></a>¿Puede una aplicación existente funcionar con Table API?

Sí, se admiten las mismas API.

### <a name="do-i-need-to-migrate-my-existing-azure-table-storage-applications-to-the-sdk-if-i-dont-want-to-use-the-table-api-features"></a>¿Es necesario migrar las aplicaciones existentes de Azure Table Storage al nuevo SDK si no se quieren usar las características de Table API?

No, se pueden crear y usar recursos existentes de Azure Table Storage sin interrupciones de ningún tipo. Pero, si no usa la nueva Table API, no puede aprovechar el índice automático, la opción de coherencia adicional ni la distribución global.

### <a name="how-do-i-add-replication-of-the-data-in-the-table-api-across-more-than-one-region-of-azure"></a>¿Cómo se agrega la replicación de los datos de Table API en más de una región de Azure?

Puede usar la [configuración de replicación global](tutorial-global-distribution-sql-api.md#portal) del portal de Azure Cosmos DB para agregar regiones que sea adecuada para la aplicación. Para desarrollar una aplicación distribuida globalmente, debe agregar también la aplicación con la información de PreferredLocation establecida en la región local para proporcionar baja latencia de lectura.

### <a name="how-do-i-change-the-primary-write-region-for-the-account-in-the-table-api"></a>¿Cómo se puede cambiar la región principal de escritura para la cuenta de Table API?

Puede usar el panel del portal de replicación global de Azure Cosmos DB para agregar una región y luego conmutar por error a la región necesaria. Para obtener instrucciones, vea [Developing with multi-region Azure Cosmos DB accounts](high-availability.md) (Desarrollo con cuentas de Azure Cosmos DB de varias regiones).

### <a name="how-do-i-configure-my-preferred-read-regions-for-low-latency-when-i-distribute-my-data"></a>¿Cómo se configuran las regiones de lectura preferidas con una baja latencia al distribuir los datos?

Con el fin de leer desde la ubicación local, use la clave PreferredLocation en el archivo app.config. En aplicaciones existentes, Table API produce un error si se establece LocationMode. Quite el código, ya que Table API recoge esta información del archivo app.config. 

### <a name="how-should-i-think-about-consistency-levels-in-the-table-api"></a>¿Cómo debo plantearme los niveles de coherencia en Table API?

Azure Cosmos DB ofrece contrapartidas bien fundamentadas entre coherencia, disponibilidad y latencia. Azure Cosmos DB ofrece cinco niveles de coherencia para desarrolladores de Table API, por lo que puede elegir el modelo adecuado en el nivel de tabla y realizar solicitudes individuales al tiempo que se consultan datos. Al conectarse, el cliente puede especificar un nivel de coherencia. Puede cambiar el nivel a través del argumento consistencyLevel de CreateCloudTableClient.

Table API ofrece lecturas de baja latencia con la opción de lectura de sus propias escrituras con coherencia de obsolescencia limitada como valor predeterminado. Para más información, consulte [Niveles de coherencia](consistency-levels.md).

De forma predeterminada, Azure Table Storage ofrece una coherencia alta dentro de una región y una posible coherencia en las ubicaciones secundarias.

### <a name="does-azure-cosmos-db-table-api-offer-more-consistency-levels-than-azure-table-storage"></a>¿Table API de Azure Cosmos DB ofrece más niveles de coherencia que Azure Table Storage?

Sí, para obtener información sobre cómo beneficiarse de la naturaleza distribuida de Azure Cosmos DB, vea [Niveles de coherencia](consistency-levels.md). Dado que se ofrecen garantías para los niveles de coherencia, puede usarlos con toda confianza.

### <a name="when-global-distribution-is-enabled-how-long-does-it-take-to-replicate-the-data"></a>Cuando se habilita la distribución global, ¿cuánto se tarda en replicar los datos?

En cuestión de milisegundos, Azure Cosmos DB confirma los datos de manera duradera en la región local y los inserta inmediatamente en otras regiones. Esta replicación solo depende del tiempo de ida y vuelta (RTT) del centro de datos. Para obtener más información sobre la funcionalidad de distribución global de Azure Cosmos DB, vea [Azure Cosmos DB: Distribución de datos global con Azure Cosmos DB](distribute-data-globally.md).

### <a name="can-the-read-request-consistency-level-be-changed"></a>¿Se puede cambiar el nivel de coherencia de la solicitud de lectura?

Con Azure Cosmos DB, puede establecer el nivel de coherencia en el nivel de contenedor (en la tabla). Mediante el SDK para .NET, puede cambiar el nivel especificando el valor de la clave TableConsistencyLevel en el archivo app.config. Los valores posibles son: Alta, Obsolescencia limitada, Sesión, Prefijo coherente y Posible. Para más información, vea [Niveles de coherencia de datos optimizables en Azure Cosmos DB](consistency-levels.md). La idea clave es que no se puede establecer el nivel de coherencia de la solicitud por encima del establecido para la tabla. Por ejemplo, no puede establecer el nivel de coherencia de la tabla en Posible y el nivel de coherencia de la solicitud en Alta.

### <a name="how-does-the-table-api-handle-failover-if-a-region-goes-down"></a>¿Cómo trata Table API la conmutación por error en caso de que una región deje de funcionar?

Table API aprovecha la plataforma distribuida globalmente de Azure Cosmos DB. Para garantizar que la aplicación puede tolerar un tiempo de inactividad del centro de datos, habilite otra región adicional para la cuenta como mínimo en [Developing with multi-region Azure Cosmos DB accounts](high-availability.md) (Desarrollo con cuentas de Azure Cosmos DB de varias regiones) del portal de Cosmos DB. Puede establecer la prioridad de la región mediante [Developing with multi-region Azure Cosmos DB accounts](high-availability.md) (Desarrollo con cuentas de Azure Cosmos DB de varias regiones) del portal.

Puede agregar todas las regiones que quiera para la cuenta y controlar adónde puede conmutar por error especificando una prioridad. Para usar la base de datos, debe especificar también una aplicación allí. Al hacerlo, los clientes no experimentarán tiempo de inactividad. El [SDK cliente de .NET más reciente](table-sdk-dotnet.md) tiene hospedaje automático, a diferencia de los otros SDK. Es decir, puede detectar la región que no funciona y conmutar por error automáticamente a la nueva región.

### <a name="is-the-table-api-enabled-for-backups"></a>¿Está habilitada Table API para copias de seguridad?

Sí, Table API aprovecha la plataforma de Azure Cosmos DB para las copias de seguridad. Se realizan copias de seguridad automáticamente. Para más información, vea [Copias de seguridad y restauración automáticas en línea con Azure Cosmos DB](online-backup-and-restore.md).

### <a name="does-the-table-api-index-all-attributes-of-an-entity-by-default"></a>¿Table API indexa de forma predeterminada todos los atributos de una entidad?

Sí, todos los atributos de una entidad se indexan de forma predeterminada. Para obtener más información, vea [Directivas de indexación de Azure Cosmos DB](index-policy.md).

### <a name="does-this-mean-i-dont-have-to-create-more-than-one-index-to-satisfy-the-queries"></a>¿Significa esto que no hay que crear más de un índice para responder a las consultas?

Sí, Table API de Azure Cosmos DB ofrece indexación automática de todos los atributos sin ninguna definición de esquema. Esta automatización libera al desarrollador para que se centre en la aplicación y no en la creación y administración de índices. Para obtener más información, vea [Directivas de indexación de Azure Cosmos DB](index-policy.md).

### <a name="can-i-change-the-indexing-policy"></a>¿Se puede cambiar la directiva de indexación?

Sí, puede cambiar la directiva de indexación especificando la definición del índice. Esta configuración se tiene que codificar adecuadamente e incluir secuencias de escape.

Para los SDK que no son de .NET, la directiva de indexación solo se puede establecer en el portal en el **Explorador de datos**; vaya a la tabla concreta que quiera cambiar y, después, vaya a **Scale & Settings** (Escala y configuración)-> Directiva de indexación, realice el cambio deseado y, a continuación, haga clic en **Guardar**.

Desde el SDK para .NET pueden enviarse en el archivo app.config:

```JSON
{
  "indexingMode": "consistent",
  "automatic": true,
  "includedPaths": [
    {
      "path": "/somepath",
      "indexes": [
        {
          "kind": "Range",
          "dataType": "Number",
          "precision": -1
        },
        {
          "kind": "Range",
          "dataType": "String",
          "precision": -1
        }
      ]
    }
  ],
  "excludedPaths":
[
 {
      "path": "/anotherpath"
 }
]
}
```

### <a name="azure-cosmos-db-as-a-platform-seems-to-have-lot-of-capabilities-such-as-sorting-aggregates-hierarchy-and-other-functionality-will-you-be-adding-these-capabilities-to-the-table-api"></a>Azure Cosmos DB como plataforma parece tener gran cantidad de funcionalidades, como ordenación, agregados, jerarquía y otras funciones. ¿Se van a agregar estas funcionalidades a Table API?

Table API proporciona la misma funcionalidad de consulta que Azure Table Storage. Azure Cosmos DB también admite ordenación, agregados, consulta geoespacial, jerarquía y una amplia variedad de funciones integradas. Para más información, vea [Consultas de SQL](how-to-sql-query.md).

### <a name="when-should-i-change-tablethroughput-for-the-table-api"></a>¿Cuándo se debe cambiar TableThroughput por Table API?

Debe cambiar el valor de TableThroughput cuando se cumpla alguna de las condiciones siguientes:

* Va a realizar una extracción, transformación y carga (ETL) de datos o quiere cargar una gran cantidad de datos en un período corto de tiempo.
* Necesita más rendimiento desde el contenedor o desde un conjunto de contenedores en el back-end. Por ejemplo, verá que el rendimiento de uso es mayor que el rendimiento aprovisionado, lo que se traduce en limitaciones. Para más información, vea [Configuración del rendimiento para contenedores de Azure Cosmos DB](set-throughput.md).

### <a name="can-i-scale-up-or-scale-down-the-throughput-of-my-table-api-table"></a>¿Se puede escalar o reducir verticalmente el rendimiento de mi tabla de Table API?

Sí, puede usar el panel de escala del portal de Azure Cosmos DB para ello. Para más información, vea [Establecimiento del rendimiento](set-throughput.md).

### <a name="is-a-default-tablethroughput-set-for-newly-provisioned-tables"></a>¿Existe un TableThroughput predeterminado que se establezca para tablas recién aprovisionadas?

Sí, si no invalida TableThroughput a través de app.config y no usa un contenedor creado previamente en Azure Cosmos DB, el servicio crea una tabla con un rendimiento de 400.

### <a name="is-there-any-change-of-pricing-for-existing-customers-of-the-azure-table-storage-service"></a>¿Hay algún cambio de precio para los clientes actuales del servicio Azure Table Storage?

Ninguno. No hay ningún cambio de precio para los clientes existentes de Azure Table Storage.

### <a name="how-is-the-price-calculated-for-the-table-api"></a>¿Cómo se calcula el precio de Table API?

El precio depende del TableThroughput asignado.

### <a name="how-do-i-handle-any-rate-limiting-on-the-tables-in-table-api-offering"></a>¿Cómo se tratan las limitaciones de tasa en las tablas de la oferta de Table API?

Si la tasa de solicitudes es mayor que la capacidad del rendimiento aprovisionado para el contenedor subyacente o un conjunto de contenedores, obtiene un error y el SDK vuelve a intentar la llamada aplicando la directiva de reintentos.

### <a name="why-do-i-need-to-choose-a-throughput-apart-from-partitionkey-and-rowkey-to-take-advantage-of-the-table-api-offering-of-azure-cosmos-db"></a>¿Por qué hay que elegir un rendimiento además de PartitionKey y RowKey para aprovechar las ventajas de la oferta de Table API de Azure Cosmos DB?

Si no se especifica un rendimiento predeterminado para el contenedor en el archivo app.config o a través del portal, Azure Cosmos DB establecerá uno.

Azure Cosmos DB ofrece garantías de rendimiento y latencia con límites superiores por operación. Esta garantía es posible cuando un motor puede aplicar la gobernanza de las operaciones del inquilino. Al establecer TableThroughput se asegura de obtener el rendimiento y la latencia garantizados, ya que la plataforma reserva esta capacidad y garantiza el éxito operativo.

Si usa la especificación de rendimiento, puede cambiarla de forma elástica para aprovechar la estacionalidad de la aplicación, satisfacer las necesidades de rendimiento y ahorrar costos.

### <a name="azure-table-storage-has-been-inexpensive-for-me-because-i-pay-only-to-store-the-data-and-i-rarely-query-the-azure-cosmos-db-table-api-offering-seems-to-be-charging-me-even-though-i-havent-performed-a-single-transaction-or-stored-anything-can-you-explain"></a>Azure Table Storage me ha resultado económico ya que solo pago por almacenar los datos y rara vez hago consultas. Parece que la oferta de Table API de Azure Cosmos DB me cobra aunque no haya realizado ni una sola transacción ni haya almacenado nada. ¿Cómo se explica esto?

Azure Cosmos DB se ha diseñado para ser un sistema basado en contratos de nivel de servicio distribuido globalmente con garantías de disponibilidad, latencia y rendimiento. A diferencia de otros sistemas, cuando se reserva rendimiento en Azure Cosmos DB, este está garantizado. Azure Cosmos DB ofrece funcionalidades adicionales que los clientes han solicitado, como índices secundarios y distribución global.

### <a name="i-never-get-a-quota-full-notification-indicating-that-a-partition-is-full-when-i-ingest-data-into-azure-table-storage-with-the-table-api-i-do-get-this-message-is-this-offering-limiting-me-and-forcing-me-to-change-my-existing-application"></a>Nunca aparece una notificación de "cuota completa" (que indica que una partición está completa) cuando se ingieren datos en Azure Table Storage. Con Table API, recibo este mensaje. ¿Esta oferta me limita y me obliga a cambiar mi aplicación actual?

Azure Cosmos DB es un sistema basado en contratos de nivel de servicio que ofrece escalado ilimitado con garantías de latencia, rendimiento, disponibilidad y coherencia. Para obtener el rendimiento premium garantizado, asegúrese de que el tamaño de los datos y el índice son administrables y escalables. El límite de 10 GB en el número de entidades o elementos por clave de partición tiene por objeto asegurar un excelente rendimiento de consultas y búsquedas. Para asegurarse de que la aplicación se escala correctamente, incluso con Azure Storage, es recomendable que *no* cree una partición muy activa almacenando toda la información en una partición y realizando consultas en ella.

### <a name="so-partitionkey-and-rowkey-are-still-required-with-the-table-api"></a>Entonces, ¿PartitionKey y RowKey siguen siendo necesarios con Table API?

Sí. Dado que el área expuesta de Table API es similar al del SDK de Azure Table Storage, la clave de partición supone una forma excelente de distribuir los datos. La clave de fila es única en esa partición. La clave de fila tiene que estar presente y no puede ser nula como en el SDK estándar. La longitud de RowKey es 255 bytes y la de PartitionKey es 1 KB.

### <a name="what-are-the-error-messages-for-the-table-api"></a>¿Cuáles son los mensajes de error de Table API?

Azure Table Storage y Table API de Azure Cosmos DB usan el mismo SDK, por lo que la mayoría de los errores coincidirán.

### <a name="why-do-i-get-throttled-when-i-try-to-create-lot-of-tables-one-after-another-in-the-table-api"></a>¿Por qué me veo limitado cuando trato de crear muchas tablas seguidas en Table API?

Azure Cosmos DB es un sistema basado en Acuerdos de Nivel de Servicio que ofrece garantías de latencia, rendimiento, disponibilidad y coherencia. Puesto que es un sistema aprovisionado, reserva recursos para garantizar estos requisitos. Se detecta y limita la frecuencia rápida de creación de tablas. Se recomienda que examine la tasa de creación de tablas y la reduzca a menos de cinco por minuto. Recuerde que Table API es un sistema aprovisionado. Se empieza a pagar por él desde el momento en que se aprovisiona.

## <a name="gremlin-api"></a>API de Gremlin

### <a name="for-cnet-development-should-i-use-the-microsoftazuregraphs-package-or-gremlinnet"></a>¿Para desarrollar con C# o .NET, debo usar el paquete de Microsoft.Azure.Graphs o Gremlin.NET?

Gremlin API de Azure Cosmos DB aprovecha los controladores de código abierto como conectores principales para el servicio. Por lo tanto, la opción recomendada es usar [controladores que sean compatibles con Apache Tinkerpop](https://tinkerpop.apache.org/).

### <a name="how-are-rus-charged-when-running-queries-on-a-graph-database"></a>¿Cómo se cobran las RU/s cuando se ejecutan consultas en una base de datos de grafos?

Todos los objetos de gráficos, vértices y bordes se muestran como documentos JSON en el servidor back-end. Como una consulta de Gremlin puede modificar uno o varios objetos de grafo a la vez, el costo asociado con él está directamente relacionado con los objetos, los bordes que la consulta procesa. Este es el mismo proceso que Azure Cosmos DB usa para las demás API. Para más información, vea [Unidades de solicitud en Azure Cosmos DB](request-units.md).

El cargo por las RU se basa en el conjunto de datos de trabajo del recorrido, y no en el conjunto de resultados. Por ejemplo, si una consulta pretende obtener como resultado un solo vértice pero, para ello, debe atravesar más de un objeto en el camino, el costo se basará en todos los objetos de gráfico que se necesiten para procesar el resultado de un vértice.

### <a name="whats-the-maximum-scale-that-a-graph-database-can-have-in-azure-cosmos-db-gremlin-api"></a>¿Cuál es la escala máxima que puede tener una base de datos de grafos de Gremlin API de Azure Cosmos DB?

Azure Cosmos DB usa [particiones horizontales](partition-data.md) para abordar automáticamente el aumento en los requisitos de almacenamiento y capacidad de proceso. La capacidad de proceso y la capacidad de almacenamiento máximas de una carga de trabajo viene determinada por el número de particiones que están asociadas con una colección determinada. Sin embargo, una colección de Gremlin API tiene un conjunto específico de directrices para garantizar una experiencia de rendimiento adecuada a escala. Para más información sobre la creación de particiones y procedimientos recomendados, vea el artículo sobre [creación de particiones en Azure Cosmos DB](partition-data.md).

### <a name="how-can-i-protect-against-injection-attacks-using-gremlin-drivers"></a>¿Cómo me puedo proteger frente a ataques de inyección de código con los controladores Gremlin?

La mayoría de los controladores Gremlin de Tinkerpop nativos ofrecen la opción de proporcionar un diccionario de parámetros para la ejecución de consultas. Este es un ejemplo de cómo hacerlo en [Gremlin.Net](https://tinkerpop.apache.org/docs/3.2.7/reference/#gremlin-DotNet) y en [Gremlin-Javascript](https://github.com/Azure-Samples/azure-cosmos-db-graph-nodejs-getting-started/blob/master/app.js).

### <a name="why-am-i-getting-the-gremlin-query-compilation-error-unable-to-find-any-method-error"></a>¿Por qué recibo el mensaje "Gremlin Query Compilation Error: Unable to find any method" (Error de compilación de consultas de Gremlin: no se encuentra ningún método)?

Gremlin API de Azure Cosmos DB implementa un subconjunto de la funcionalidad definida en el área expuesta de Gremlin. Para ver pasos admitidos y obtener más información, consulte el artículo sobre [compatibilidad con Gremlin](gremlin-support.md).

La mejor solución es volver a escribir los pasos necesarios de Gremlin con la funcionalidad admitida, porque todos los pasos de Gremlin esenciales son compatibles con Azure Cosmos DB.

### <a name="why-am-i-getting-the-websocketexception-the-server-returned-status-code-200-when-status-code-101-was-expected-error"></a>¿Por qué recibo el mensaje "WebSocketException: The server returned status code "200" when status code "101" was expected" (WebSocketException: el servidor devolvió el código de estado "200" cuando el código de estado esperado era "101")?

Este error probablemente se produce cuando se usa el punto de conexión incorrecto. El punto de conexión que genera este error tiene el siguiente patrón:

`https:// YOUR_DATABASE_ACCOUNT.documents.azure.com:443/`

Este es el punto de conexión de documentos de la base de datos de grafos.  El punto de conexión correcto que hay que usar es el punto de conexión de Gremlin, que tiene el formato siguiente:

`https://YOUR_DATABASE_ACCOUNT.gremlin.cosmosdb.azure.com:443/`

### <a name="why-am-i-getting-the-requestrateistoolarge-error"></a>¿Por qué recibo el error "RequestRateIsTooLarge"?

Este error significa que las unidades de solicitud por segundo asignadas no son suficientes para atender la consulta. Este error suele aparecer cuando se ejecuta una consulta que obtiene todos los vértices:

```
// Query example:
g.V()
```

Esta consulta intentará recuperar todos los vértices del grafo. Por lo tanto, el costo de esta consulta será igual que el número de vértices en términos de RU, como mínimo. El valor de RU/s se debería ajustar para poder atender esta consulta.

### <a name="why-do-my-gremlin-driver-connections-get-dropped-eventually"></a>¿Por qué las conexiones del controlador de Gremlin terminan por desconectarse?

Las conexiones de Gremlin se realizan a través de una conexión de WebSocket. Aunque las conexiones WebSocket no tienen un tiempo de vida específico, Gremlin API de Azure Cosmos DB terminará las conexiones inactivas después de 30 minutos de inactividad.

### <a name="why-cant-i-use-fluent-api-calls-in-the-native-gremlin-drivers"></a>¿Por qué no puedo usar llamadas a API fluida en los controladores nativos de Gremlin?

Gremlin API de Azure Cosmos DB aún no admite llamadas API fluidas. Las llamadas API fluidas requieren una característica de formato interna conocida como compatibilidad con código de bytes, que actualmente no se admite en Gremlin API de Azure Cosmos DB. Por el mismo motivo, el controlador más reciente de Gremlin-JavaScript tampoco es compatible.

### <a name="how-can-i-evaluate-the-efficiency-of-my-gremlin-queries"></a>¿Cómo puedo evaluar la eficiencia de mis consultas de Gremlin?

Se puede usar el paso de vista previa **executionProfile()** para realizar un análisis del plan de ejecución de la consulta. Este paso debe agregarse al final de cualquier consulta de Gremlin, tal y como se muestra en el ejemplo siguiente:

**Ejemplo de consulta**

```
g.V('mary').out('knows').executionProfile()
```

**Salida del ejemplo**

```json
[
  {
    "gremlin": "g.V('mary').out('knows').executionProfile()",
    "totalTime": 8,
    "metrics": [
      {
        "name": "GetVertices",
        "time": 3,
        "annotations": {
          "percentTime": 37.5
        },
        "counts": {
          "resultCount": 1
        }
      },
      {
        "name": "GetEdges",
        "time": 5,
        "annotations": {
          "percentTime": 62.5
        },
        "counts": {
          "resultCount": 0
        },
        "storeOps": [
          {
            "count": 0,
            "size": 0,
            "time": 0.6
          }
        ]
      },
      {
        "name": "GetNeighborVertices",
        "time": 0,
        "annotations": {
          "percentTime": 0
        },
        "counts": {
          "resultCount": 0
        }
      },
      {
        "name": "ProjectOperator",
        "time": 0,
        "annotations": {
          "percentTime": 0
        },
        "counts": {
          "resultCount": 0
        }
      }
    ]
  }
]
```

El resultado del perfil anterior muestra cuánto tiempo se dedica a obtener los objetos de vértice, los objetos de borde y el tamaño del conjunto de datos de trabajo. Esto tiene que ver con las mediciones de los costos estándar para las consultas de Azure Cosmos DB.

## <a id="cassandra"></a> Cassandra API

### <a name="what-is-the-protocol-version-supported-by-azure-cosmso-db-cassandra-api-is-there-a-plan-to-support-other-protocols"></a>¿Qué es la versión del protocolo compatible con la API de Cassandra de Azure Cosmso DB? ¿Hay un plan para admitir otros protocolos?

La API de Apache Cassandra para Azure Cosmos DB admite ahora CQL versión 4. Si tiene comentarios sobre la compatibilidad con otros protocolos, háganoslo saber a través de [comentarios de UserVoice](https://feedback.azure.com/forums/263030-azure-cosmos-db) o envíe un correo electrónico a [askcosmosdbcassandra@microsoft.com](mailto:askcosmosdbcassandra@microsoft.com).

### <a name="why-is-choosing-a-throughput-for-a-table-a-requirement"></a>¿Por qué es un requisito elegir un rendimiento para una tabla?

Azure Cosmos DB establece el rendimiento predeterminado para el contenedor en función de a partir de dónde se crea la tabla: desde el portal o desde CQL.
Azure Cosmos DB ofrece garantías de rendimiento y latencia con límites superiores por operación. Esta garantía es posible cuando un motor puede aplicar la gobernanza de las operaciones del inquilino. Al establecer el rendimiento, se asegura de obtener el rendimiento y la latencia garantizados, ya que la plataforma reserva esta capacidad y asegura el éxito operativo.
Puede cambiar con total flexibilidad el rendimiento para beneficiarse de la estacionalidad de su aplicación y ahorrar costos.

El concepto de rendimiento se explica en el artículo [Unidades de solicitud en Azure Cosmos DB](request-units.md). El rendimiento de una tabla se distribuye equitativamente entre las particiones físicas subyacentes.

### <a name="what-is-the-default-rus-of-table-when-created-through-cql-what-if-i-need-to-change-it"></a>¿Cuál es la cantidad predeterminada de RU/s de una tabla cuando se crea mediante CQL? ¿Qué ocurre si deseo cambiarlo?

Azure Cosmos DB utiliza unidades de solicitud por segundo (RU/s) como la medida para proporcionar el rendimiento. Las tablas creadas a través de CQL tienen 400 RU. Puede cambiar la RU en el portal.

CQL

```
CREATE TABLE keyspaceName.tablename (user_id int PRIMARY KEY, lastname text) WITH cosmosdb_provisioned_throughput=1200
```

.NET

```csharp
int provisionedThroughput = 400;
var simpleStatement = new SimpleStatement($"CREATE TABLE {keyspaceName}.{tableName} (user_id int PRIMARY KEY, lastname text)");
var outgoingPayload = new Dictionary<string, byte[]>();
outgoingPayload["cosmosdb_provisioned_throughput"] = Encoding.UTF8.GetBytes(provisionedThroughput.ToString());
simpleStatement.SetOutgoingPayload(outgoingPayload);
```

### <a name="what-happens-when-throughput-is-used-up"></a>¿Qué ocurre cuando se usa el rendimiento?

Azure Cosmos DB ofrece garantías de rendimiento y latencia con límites superiores por operación. Esta garantía es posible cuando un motor puede aplicar la gobernanza de las operaciones del inquilino. Esto es posible por la configuración del rendimiento, que asegura que se obtiene el rendimiento y la latencia garantizados, ya que la plataforma reserva esta capacidad y asegura el éxito operativo.
Al superar esta capacidad, obtiene mensajes de error de sobrecarga que indican que se usó la capacidad.
0 x 1001 elementos sobrecargados: no se puede procesar la solicitud porque la "Tasa de solicitudes es grande". En este momento, es esencial ver qué operaciones ocasionan este problema y su volumen. Puede hacerse una idea sobre el exceso de capacidad consumida que supera la aprovisionada con las métricas del portal. A continuación, debe asegurarse de que se consume la capacidad prácticamente igual en todas las particiones subyacentes. Si ve que una partición consume la mayor parte del rendimiento, tiene un sesgo de la carga de trabajo.

Hay disponibles métricas que muestran cómo se usa el rendimiento en horas, días y cada siete días, entre particiones o en total. Para más información, consulte [Supervisión y depuración con métricas de Azure Cosmos DB](use-metrics.md).

Los registros de diagnóstico se explican en el artículo [Registro de diagnóstico de Azure Cosmos DB](logging.md).

### <a name="does-the-primary-key-map-to-the-partition-key-concept-of-azure-cosmos-db"></a>¿Se corresponde la clave principal con el concepto de clave de partición de Azure Cosmos DB?

Sí, la clave de partición se utiliza para colocar la entidad en la ubicación correcta. En Azure Cosmos DB, se usa para encontrar una partición lógica apropiada que esté almacenada en una partición física. El concepto de creación de particiones también se explica en el artículo [Partición y escalado en Azure Cosmos DB](partition-data.md). La lección esencial de esto es que una partición lógica no debe superar el límite de 10 GB en la actualidad.

### <a name="what-happens-when-i-get-a-quota-full-notification-indicating-that-a-partition-is-full"></a>¿Qué ocurre cuando recibo una notificación de "cuota completa" que indica que una partición está llena?

Azure Cosmos DB es un sistema basado en Acuerdos de Nivel de Servicio que ofrece un escalado ilimitado con garantías de latencia, rendimiento, disponibilidad y coherencia. Este almacenamiento ilimitado se basa en la ampliación horizontal de los datos mediante la creación de particiones como concepto clave. El concepto de creación de particiones también se explica en el artículo [Partición y escalado en Azure Cosmos DB](partition-data.md).

El límite de 10 GB en el número de entidades o elementos por partición lógica que debe cumplir. Para asegurarse de que la aplicación se escala correctamente incluso con Azure Storage, le pedimos que *no* cree una partición muy activa almacenando toda la información en una partición y realizando consultas en ella. Este error solo puede producirse si los datos están sesgados; es decir, tiene una gran cantidad de datos para una clave de partición (más de 10&nbsp;GB). Puede encontrar la distribución de datos mediante el portal de almacenamiento. La manera de corregir este error consiste en volver a crear la tabla y elegir un principal granular (una clave de partición), lo que permite una mejor distribución de datos.

### <a name="is-it-possible-to-use-cassandra-api-as-key-value-store-with-millions-or-billions-of-individual-partition-keys"></a>¿Es posible usar la API de Cassandra como almacén de valor es de clave con millones o miles de millones de claves de partición individuales?

Azure Cosmos DB puede almacenar datos ilimitados mediante el escalado horizontal del almacenamiento. Esto es independiente del rendimiento. Sí, puede usar siempre la API de Cassandra para almacenar y recuperar la clave y los valores especificando la clave de partición y principal correcta. Estas claves individuales obtienen su propia partición lógica y se colocan encima de la partición física sin problemas.

### <a name="is-it-possible-to-create-more-than-one-table-with-apache-cassandra-api-of-azure-cosmos-db"></a>¿Es posible crear más de una tabla con Apache Cassandra API de Azure Cosmos DB?

Sí, es posible crear más de una tabla con Apache Cassandra API. Cada una de esas tablas se trata como unidad de rendimiento y almacenamiento.

### <a name="is-it-possible-to-create-more-than-one-table-in-succession"></a>¿Es posible crear más de una tabla en sucesión?

Azure Cosmos DB es un sistema regulado por recursos tanto para las actividades de planeamiento de control como de datos. Los contenedores como las colecciones y las tablas son entidades del entorno de tiempo de ejecución que se han aprovisionado para una capacidad de rendimiento determinado. La creación de estos contenedores en sucesión rápida no es una actividad esperada y puede estar limitada. Si tiene pruebas que eliminan o crean tablas inmediatamente, intente espaciarlas.

### <a name="what-is-maximum-number-of-tables-that-can-be-created"></a>¿Cuál es el número máximo de tablas que se pueden crear?

No hay ningún límite físico en el número de tablas; envíe un correo electrónico a [askcosmosdbcassandra@microsoft.com](mailto:askcosmosdbcassandra@microsoft.com) si debe crear un gran número de tablas (donde el tamaño constante total supere los 10 TB de datos), en lugar de las decenas o cientos habituales.

### <a name="what-is-the-maximum--of-keyspace-that-we-can-create"></a>¿Cuál es el número máximo de espacios de claves que podemos crear?

No hay ningún límite físico en el número de espacios de claves, ya que son contenedores de metadatos; envíe un correo electrónico a [askcosmosdbcassandra@microsoft.com](mailto:askcosmosdbcassandra@microsoft.com) si, por algún motivo, tiene un gran número de espacios de claves.

### <a name="is-it-possible-to-bring-in-lot-of-data-after-starting-from-normal-table"></a>¿Es posible tener una gran cantidad de datos tras comenzar con una tabla normal?

La capacidad de almacenamiento se administra automáticamente y aumenta a medida que inserta más datos en ella. Así, puede importar con confianza tantos datos como sea necesario sin administrar ni aprovisionar los nodos y mucho más.

### <a name="is-it-possible-to-supply-yaml-file-settings-to-configure-apache-casssandra-api-of-azure-cosmos-db-behavior"></a>¿Es posible proporcionar la configuración del archivo yaml para configurar el comportamiento de Apache Casssandra API de Azure Cosmos DB?

La API de Apache Cassandra de Azure Cosmos DB es un servicio de plataforma. Proporciona compatibilidad de nivel de protocolo para la ejecución de operaciones. Oculta la complejidad de la administración, supervisión y configuración. Como desarrollador y usuario, no tiene que preocuparse sobre la disponibilidad, los marcadores de exclusión, la memoria caché de claves, la memoria caché de filas, los filtros de Bloom y muchas otras opciones. La API de Apache Cassandra de Azure Cosmos DB se centra en proporcionar el rendimiento de lectura y escritura que necesite sin la sobrecarga de la configuración y la administración.

### <a name="will-apache-cassandra-api-for-azure-cosmos-db-support-node-additioncluster-statusnode-status-commands"></a>¿Admitirá la API de Apache Cassandra de Azure Cosmos DB comandos de estado de nodo, estado de clústeres o adición de nodos?

Apache Cassandra API es un servicio de plataforma que facilita el planeamiento de la capacidad, respondiendo a las exigencias de flexibilidad en el rendimiento y el almacenamiento. Con Azure Cosmos DB, se aprovisiona el rendimiento que se necesita. A continuación, puede escalar verticalmente aumentando o disminuyendo lo que necesite a lo largo del día sin preocuparse de agregar o eliminar nodos ni de administrarlos. Esto implica que no es necesario usar el nodo, ni tampoco la herramienta de administración del clúster.

### <a name="what-happens-with-respect-to-various-config-settings-for-keyspace-creation-like-simplenetwork"></a>¿Qué ocurre con respecto a los distintos valores de configuración para la creación de espacios de claves como una red o un elemento sencillo?

Azure Cosmos DB proporciona una distribución global directamente, debido a la baja latencia y la disponibilidad. No es necesario configurar réplicas ni nada más. Todas las escrituras son siempre forma duradera quórum confirmado en cualquier región donde escriba, mientras proporciona garantías de rendimiento.

### <a name="what-happens-with-respect-to-various-settings-for-table-metadata-like-bloom-filter-caching-read-repair-change-gcgrace-compression-memtableflushperiod-and-more"></a>¿Qué ocurre con respecto a los diversos valores de configuración para los metadatos de tablas como los filtros de Bloom, el almacenamiento en caché, los cambios de reparación de las lecturas, gc_grace y el valor memtable_flush_period de la compresión, entre otros?

Azure Cosmos DB proporciona el rendimiento para las lecturas y escrituras, y la capacidad de proceso sin necesidad de variar ninguno de los valores de configuración y correr el riesgo de manipularlos accidentalmente.

### <a name="is-time-to-live-ttl-supported-for-cassandra-tables"></a>¿Se admite el período de vida (TTL) en las tablas de Cassandra?

Sí, TTL se admite.

### <a name="is-it-possible-to-monitor-node-status-replica-status-gc-and-os-parameters-earlier-with-various-tools-what-needs-to-be-monitored-now"></a>¿Es posible supervisar los parámetros anteriores del sistema operativo, gc y el estado de las réplicas y de los nodos con varias herramientas? ¿Qué debe supervisarse ahora?

Azure Cosmos DB es un servicio de plataforma que le ayuda a aumentar la productividad y no preocuparse de la administración y supervisión de la infraestructura. Solo debe vigilar el rendimiento que está disponible en las métricas del portal para averiguar si está siendo limitado y aumentarlo o disminuirlo.
Supervise los [Acuerdos de Nivel de Servicio](monitor-accounts.md).
Use [Métricas](use-metrics.md). Use [registros de diagnóstico](logging.md).

### <a name="which-client-sdks-can-work-with-apache-cassandra-api-of-azure-cosmos-db"></a>¿Los SDK de cliente pueden funcionar con la API de Apache Cassandra de Azure Cosmos DB?

Los controladores de cliente del SDK de Apache Cassandra que usan CQLv3 se utilizaban para programas cliente. Si usa otros controladores o si está teniendo problemas, envíe un correo electrónico a [askcosmosdbcassandra@microsoft.com](mailto:askcosmosdbcassandra@microsoft.com).

### <a name="is-composite-partition-key-supported"></a>¿Se admite la clave de partición compuesta?

Sí, puede usar la sintaxis normal para crear una clave de partición compuesta.

### <a name="can-i-use-stable-loader-for-data-loading"></a>¿Puedo usar el cargador sstable para cargar los datos?

No, en la versión preliminar, no se admite el cargador sstable.

### <a name="can-an-on-premises-cassandra-cluster-be-paired-with-azure-cosmos-dbs-apache-cassandra-api"></a>¿Un clúster de Cassandra local se puede emparejar con la API de Apache Cassandra de Azure Cosmos DB?

En la actualidad, Azure Cosmos DB tiene una experiencia optimizada para el entorno en la nube sin sobrecarga de operaciones. Si requiere del emparejamiento, envíe un correo electrónico a [askcosmosdbcassandra@microsoft.com](mailto:askcosmosdbcassandra@microsoft.com) con una descripción de su escenario.

### <a name="does-cassandra-api-provide-full-backups"></a>¿Proporciona la API de Cassandra copias de seguridad completas?

Azure Cosmos DB ofrece dos copias de seguridad completas gratis realizadas en el día, en intervalos de cuatro horas, entre todas las API. Así se garantiza que no es necesario configurar una programación de copia de seguridad u otras acciones.
Si desea modificar la retención y la frecuencia, envíe un correo electrónico a [askcosmosdbcassandra@microsoft.com ](mailto:askcosmosdbcassandra@microsoft.com) o genere una incidencia de soporte técnico. En el artículo [Copias de seguridad y restauración automáticas en línea con Azure Cosmos DB](online-backup-and-restore.md), se proporciona información sobre la funcionalidad de copia de seguridad.

### <a name="how-does-the-cassandra-api-account-handle-failover-if-a-region-goes-down"></a>¿Cómo trata la cuenta de la API de Cassandra la conmutación por error en caso de que una región deje de funcionar?

La API de Cassandra de Azure Cosmos DB adopta la plataforma distribuida globalmente de Azure Cosmos DB. Para garantizar que la aplicación puede tolerar un tiempo de inactividad del centro de datos, habilite otra región adicional para la cuenta como mínimo en [Developing with multi-region Azure Cosmos DB accounts](high-availability.md) (Desarrollo con cuentas de Azure Cosmos DB de varias regiones) del portal de Cosmos DB. Puede establecer la prioridad de la región mediante [Developing with multi-region Azure Cosmos DB accounts](high-availability.md) (Desarrollo con cuentas de Azure Cosmos DB de varias regiones) del portal.

Puede agregar todas las regiones que quiera para la cuenta y controlar adónde puede conmutar por error especificando una prioridad. Para usar la base de datos, debe especificar también una aplicación allí. Al hacerlo, los clientes no experimentarán tiempo de inactividad.

### <a name="does-the-apache-cassandra-api-index-all-attributes-of-an-entity-by-default"></a>¿Indexa la API de Apache Cassandra todos los atributos de una entidad de forma predeterminada?

Sí, Azure Cosmos DB indexa todos los atributos de una entidad de forma predeterminada. Para obtener más información, vea [Directivas de indexación de Azure Cosmos DB](index-policy.md). Se obtienen ventajas de rendimiento garantizado con indexación coherente y escrituras confirmadas con cuórum perdurable siempre.

### <a name="does-this-mean-i-dont-have-to-create-more-than-one-index-to-satisfy-the-queries"></a>¿Significa esto que no hay que crear más de un índice para responder a las consultas?

Sí, Azure Cosmos DB ofrece indexación automática de todos los atributos sin ninguna definición de esquema. Esta automatización libera al desarrollador para que se centre en la aplicación y no en la creación y administración de índices. Para obtener más información, vea [Directivas de indexación de Azure Cosmos DB](index-policy.md).

### <a name="can-i-use-the-new-cassandra-api-sdk-locally-with-the-emulator"></a>¿Se puede usar el nuevo SDK de la API de Cassandra localmente con el emulador?

Tenemos previsto admitir esta funcionalidad en el futuro.

### <a name="azure-cosmos-db-as-a-platform-seems-to-have-lot-of-capabilities-such-as-change-feed-and-other-functionality-will-these-capabilities-be-added-to-the-cassandra-api"></a>Azure Cosmos DB como plataforma parece tener gran cantidad de funcionalidades, como fuente de cambios y otras funciones. ¿Se agregarán estas funcionalidades a la API de Cassandra?

La API de Cassandra Apache proporciona la misma funcionalidad CQL que Apache Cassandra. Tenemos previsto estudiar la viabilidad de admitir varias funcionalidades en el futuro.

### <a name="feature-x-of-regular-cassandra-api-isnt-working-as-today-where-can-the-feedback-be-provided"></a>Una característica de Cassandra API normal no funciona en este momento, ¿dónde se pueden proporcionar comentarios al respecto?

Proporcione sus comentarios a través de [comentarios de UserVoice](https://feedback.azure.com/forums/263030-azure-cosmos-db).

[azure-portal]: https://portal.azure.com
[query]: sql-api-sql-query.md

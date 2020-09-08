---
title: Preguntas más frecuentes acerca de Table API en Azure Cosmos DB
description: Obtenga respuestas a las preguntas más frecuentes sobre Table API para Azure Cosmos DB
author: SnehaGunda
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 08/12/2020
ms.author: sngun
ms.openlocfilehash: 65f276662ac4837003c7a7078b6197ba155eadc9
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88167596"
---
# <a name="frequently-asked-questions-about-the-table-api-in-azure-cosmos-db"></a>Preguntas más frecuentes acerca de Table API en Azure Cosmos DB

Table API de Azure Cosmos DB está disponible en [Azure Portal](https://portal.azure.com). Antes, debe registrarse para obtener una suscripción de Azure. Una vez registrado, puede agregar una cuenta de Table API de Azure Cosmos DB a la suscripción de Azure y luego agregar tablas a la cuenta. Puede encontrar los lenguajes admitidos y las guías de inicio rápido asociadas en la [Introducción a Table API de Azure Cosmos](table-introduction.md).

## <a name="table-api-in-azure-cosmos-db-vs-azure-table-storage"></a><a id="table-api-vs-table-storage"></a>Comparación de Table API en Azure Cosmos DB y Azure Table Storage

### <a name="where-is-table-api-not-identical-with-azure-table-storage-behavior"></a>¿En qué se diferencia Table API del comportamiento de Azure Table Storage?

Hay algunas diferencias de comportamiento que los usuarios acostumbrados a Azure Table Storage que deseen crear tablas con Table API de Azure Cosmos DB deben tener en cuenta:

* Table API de Azure Cosmos DB utiliza un modelo de capacidad reservada con el fin de asegurar un rendimiento garantizado, pero esto significa que se paga por la capacidad en cuanto se crea la tabla, incluso aunque no se esté usando. Con Azure Table Storage, solo se paga por la capacidad que se utilice. Esto ayuda a explicar por qué Table API puede ofrecer un Acuerdo de Nivel de Servicio con lecturas de 10 ms y escrituras de 15 ms en el percentil 99 mientras que Azure Table Storage ofrece un Acuerdo de Nivel de Servicio de 10 segundos. Pero, como consecuencia, con las tablas de Table API, incluso las tablas vacías sin ninguna solicitud cuestan dinero, con el fin de garantizar que la capacidad está disponible para atender las solicitudes que les llegan en el Acuerdo de Nivel de Servicio ofrecido por Azure Cosmos DB.

* Los resultados de las consultas que devuelve Table API no se ordenan por la clave de fila ni por la clave de partición como sucede en Azure Table Storage.

* Las claves de fila solo pueden ser de hasta 255 bytes.

* Los lotes solo pueden contener hasta 2 MB.

* CORS no se admite actualmente.

* En los nombres de tabla de Azure Table Storage no se distinguen mayúsculas de minúsculas, a diferencia de lo que ocurre en Table API de Azure Cosmos DB.

* Algunos de los formatos internos de Azure Cosmos DB para codificar la información, como los campos binarios, no son actualmente tan eficientes como se podría desear. Por lo tanto, esto puede causar limitaciones inesperadas en el tamaño de los datos. Por ejemplo, actualmente no se puede utilizar el Meg completo de una entidad de tabla para almacenar datos binarios porque la codificación aumenta el tamaño de los datos.

* El "identificador" del nombre de la propiedad de la entidad no se admite actualmente.

* TableQuery TakeCount no se limita a 1000.

* En cuanto a la API REST, hay una serie de opciones de puntos de conexión o consulta que no son compatibles con Table API de Azure Cosmos DB:

  | Métodos de REST | Opción de punto de conexión o consulta de REST | Direcciones URL de documento | Explicación |
  | ------------| ------------- | ---------- | ----------- |
  | GET, PUT | `/?restype=service@comp=properties`| [Establecer propiedades de Table service](/rest/api/storageservices/set-table-service-properties) y [Obtener propiedades de Table service](/rest/api/storageservices/get-table-service-properties) | Este punto de conexión se usa para establecer reglas de CORS, de configuración del análisis de almacenamiento y de configuración del registro. CORS no se admite actualmente y el análisis y el registro se tratan de forma diferente en Azure Cosmos DB y en Azure Storage Tables |
  | OPCIONES | `/<table-resource-name>` | [Solicitud de tabla preparatoria CORS](/rest/api/storageservices/preflight-table-request) | Esto forma parte de CORS, el cual Azure Cosmos DB no admite actualmente. |
  | GET | `/?restype=service@comp=stats` | [Obtener estadísticas de Table service](/rest/api/storageservices/get-table-service-stats) | Proporciona información sobre la rapidez con que los datos se están replicando datos entre los elementos principales y los secundarios. Esto no es necesario en Cosmos DB ya que la replicación es parte de las escrituras. |
  | GET, PUT | `/mytable?comp=acl` | [Obtener tabla ACL](/rest/api/storageservices/get-table-acl) y [Definir ACL de tabla](/rest/api/storageservices/set-table-acl) | Así se obtienen y establecen las directivas de acceso almacenadas que se usan para administrar las firmas de acceso compartido (SAS). Aunque SAS se admite, las firmas se establecen y administran de manera diferente. |

* Table API de Azure Cosmos DB solo es compatible con el formato JSON, no con ATOM.

* Aunque Azure Cosmos DB admite firmas de acceso compartido (SAS), hay determinadas directivas que no admite, en concreto las relacionadas con las operaciones de administración como el derecho para crear nuevas tablas.

* Para el SDK de .NET en concreto, hay algunas clases y métodos que Azure Cosmos DB no admite actualmente.

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

## <a name="other-frequently-asked-questions"></a>Otras preguntas frecuentes

### <a name="do-i-need-a-new-sdk-to-use-the-table-api"></a>¿Se necesita un nuevo SDK para usar Table API?

No, los SDK de almacenamiento existentes deberían seguir funcionando. Sin embargo, se recomienda obtener siempre los SDK más recientes para lograr la mejor compatibilidad y, en muchos casos, un rendimiento superior. Vea la lista de lenguajes disponibles en la [Introducción a Table API de Azure Cosmos DB](table-introduction.md).

### <a name="what-is-the-connection-string-that-i-need-to-use-to-connect-to-the-table-api"></a>¿Qué cadena de conexión hay que usar para conectarse a Table API?

La cadena de conexión es:

```
DefaultEndpointsProtocol=https;AccountName=<AccountNamefromCosmosDB;AccountKey=<FromKeysPaneofCosmosDB>;TableEndpoint=https://<AccountName>.table.cosmosdb.azure.com
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
* Necesita más rendimiento desde el contenedor o desde un conjunto de contenedores en el back-end. Por ejemplo, verá que el rendimiento de uso es mayor que el rendimiento aprovisionado, lo que se traduce en limitaciones. Para más información, vea [Set throughput for Azure Cosmos containers](set-throughput.md) (Configuración del rendimiento para contenedores de Azure Cosmos).

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

Azure Cosmos DB es un sistema basado en contratos de nivel de servicio que ofrece escalado ilimitado con garantías de latencia, rendimiento, disponibilidad y coherencia. Para obtener el rendimiento premium garantizado, asegúrese de que el tamaño de los datos y el índice son administrables y escalables. El límite de 20 GB en el número de entidades o elementos por clave de partición tiene por objeto asegurar un excelente rendimiento de consultas y búsquedas. Para asegurarse de que la aplicación se escala correctamente, incluso con Azure Storage, es recomendable que *no* cree una partición muy activa almacenando toda la información en una partición y realizando consultas en ella.

### <a name="so-partitionkey-and-rowkey-are-still-required-with-the-table-api"></a>Entonces, ¿PartitionKey y RowKey siguen siendo necesarios con Table API?

Sí. Dado que el área expuesta de Table API es similar al del SDK de Azure Table Storage, la clave de partición supone una forma excelente de distribuir los datos. La clave de fila es única en esa partición. La clave de fila tiene que estar presente y no puede ser nula como en el SDK estándar. La longitud de RowKey es 255 bytes y la de PartitionKey es 1 KB.

### <a name="what-are-the-error-messages-for-the-table-api"></a>¿Cuáles son los mensajes de error de Table API?

Azure Table Storage y Table API de Azure Cosmos DB usan el mismo SDK, por lo que la mayoría de los errores coincidirán.

### <a name="why-do-i-get-throttled-when-i-try-to-create-lot-of-tables-one-after-another-in-the-table-api"></a>¿Por qué me veo limitado cuando trato de crear muchas tablas seguidas en Table API?

Azure Cosmos DB es un sistema basado en Acuerdos de Nivel de Servicio que ofrece garantías de latencia, rendimiento, disponibilidad y coherencia. Puesto que es un sistema aprovisionado, reserva recursos para garantizar estos requisitos. Se detecta y limita la frecuencia rápida de creación de tablas. Se recomienda que examine la tasa de creación de tablas y la reduzca a menos de cinco por minuto. Recuerde que Table API es un sistema aprovisionado. Se empieza a pagar por él desde el momento en que se aprovisiona.

### <a name="how-do-i-provide-feedback-about-the-sdk-or-bugs"></a>¿Cómo se pueden hacer comentarios sobre el SDK o los errores?

Puede proporcionar comentarios de alguna de las siguientes formas:

* [Voz del usuario](https://feedback.azure.com/forums/263030-azure-cosmos-db)
* [Página de preguntas y respuestas de Microsoft](https://docs.microsoft.com/answers/topics/azure-cosmos-db.html)
* [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-cosmosdb). Stack Overflow es la mejor opción para preguntas de programación. Asegúrese de que su pregunta es [relevante](https://stackoverflow.com/help/on-topic) y [proporcione tantos detalles como sea posible, formulando la pregunta de forma clara y responsable](https://stackoverflow.com/help/how-to-ask).

## <a name="next-steps"></a>Pasos siguientes

* [Creación de una aplicación de Table API con el SDK de .NET y Azure Cosmos DB](create-table-dotnet.md)
* [Creación de una aplicación Java para administrar los datos de Table API de Azure Cosmos DB](create-table-java.md)
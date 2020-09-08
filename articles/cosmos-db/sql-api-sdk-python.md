---
title: API, SDK y recursos de Python para SQL de Azure Cosmos DB
description: Obtenga toda la información sobre la API y el SDK de Python para SQL, incluidas la fechas de lanzamiento, fechas de retirada y cambios realizados entre las versiones del SDK de Python para Azure Cosmos DB.
author: Rodrigossz
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: python
ms.topic: reference
ms.date: 08/12/2020
ms.author: anfeldma
ms.custom: devx-track-python
ms.openlocfilehash: e9f9daea2c0d570efb81603784ee730b11668426
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88585991"
---
# <a name="azure-cosmos-db-python-sdk-for-sql-api-release-notes-and-resources"></a>SDK de Python para Azure Cosmos DB para SQL API: Notas de la versión y recursos

> [!div class="op_single_selector"]
> * [.NET SDK v3](sql-api-sdk-dotnet-standard.md)
> * [.NET SDK v2](sql-api-sdk-dotnet.md)
> * [SDK de .NET Core v2](sql-api-sdk-dotnet-core.md)
> * [SDK de fuente de cambios de .NET, versión 2](sql-api-sdk-dotnet-changefeed.md)
> * [Node.js](sql-api-sdk-node.md)
> * [SDK de Java v4](sql-api-sdk-java-v4.md)
> * [Versión 2 del SDK de Java asincrónico](sql-api-sdk-async-java.md)
> * [SDK de Java v2 sincrónico](sql-api-sdk-java.md)
> * [Spring Data v2](sql-api-sdk-java-spring-v2.md)
> * [Spring Data v3](sql-api-sdk-java-spring-v3.md)
> * [Conector de Spark](sql-api-sdk-java-spark.md)
> * [Python](sql-api-sdk-python.md)
> * [REST](/rest/api/cosmos-db/)
> * [Proveedor de recursos de REST](/rest/api/cosmos-db-resource-provider/)
> * [SQL](sql-api-query-reference.md)
> * [Bulk Executor: .NET v2](sql-api-sdk-bulk-executor-dot-net.md)
> * [Bulk Executor: Java](sql-api-sdk-bulk-executor-java.md)

| |  |
|---|---|
|**Descargar SDK**|[PyPI](https://pypi.org/project/azure-cosmos)|
|**Documentación de la API**|[Documentación de referencia de la API de Python](https://docs.microsoft.com/python/api/azure-cosmos/?view=azure-python)|
|**Instrucciones de instalación del SDK**|[Instrucciones de instalación del SDK de Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cosmos/azure-cosmos)|
|**Introducción**|[Introducción al SDK de Python](create-sql-api-python.md)|
|**Plataforma admitida actualmente**|[Python 2.7](https://www.python.org/downloads/) y [Python 3.5.3+](https://www.python.org/downloads/)|

## <a name="release-history"></a>Historial de versiones

### <a name="410-2020-08-10"></a>4.1.0 (10-08-2020)

- Se ha agregado la advertencia de desuso para el modo de indexación "diferida". El back-end ya no permite la creación de contenedores con este modo y los establecerá en coherentes en su lugar.

**Nuevas características:**
- Se ha agregado la capacidad de establecer el TTL de almacenamiento analítico al crear un nuevo contenedor.

**Correcciones de errores**
- Se ha corregido la compatibilidad con elementos dict como entradas para las API get_client.
- Se ha corregido la compatibilidad con Python 2/3 en los iteradores de consulta.
- Se ha corregido el error de sugerencia de tipo (problema n.º 12570).
- Se ha corregido un error que provocaba que los encabezados de opciones no se agregaran a la función upsert_item. Problema n.º 11791: gracias @aalapatirvbd.
- Se ha corregido un error generado al usarse un identificador que no es de cadena en un elemento. Ahora genera TypeError en lugar de AttributeError (problema n.º 11793).

### <a name="400"></a>4.0.0

* Versión estable.
* Se agregó HttpLoggingPolicy a la canalización para permitir el paso de un registrador personalizado para los encabezados de solicitud y respuesta.

### <a name="400b6"></a>4.0.0b6

* Se corrigió el error en synchronized_request para las API de multimedia.
* Se quitaron MediaReadMode y MediaRequestTimeout de ConnectionPolicy, ya que no se admiten las solicitudes multimedia.

### <a name="400b5"></a>4.0.0b5

* Módulo azure.cosmos.errors desusado y reemplazado por azure.cosmos.exceptions.
* Los parámetros de la condición de acceso (`access_condition`, `if_match`, `if_none_match`) han quedado en desuso en favor de los parámetros independientes `match_condition` y `etag`.
* Se corrigió el error en el proveedor de mapas de enrutamiento.
* Se agregó compatibilidad con Distinct, Offset y Limit de consulta.
* El contexto de ejecución de consulta de documento predeterminado se usa ahora para:

  * Consultas ChangeFeed
  * Consultas de una sola partición (partitionkey, partitionKeyRangeId está presente en las opciones)
  * Consultas de no documentos

* Se producen errores en los agregados en varias particiones, con la habilitación de consultas entre particiones establecida en true, pero no hay ninguna palabra clave "value" presente
* Alcanza el punto de conexión final del plan de consulta para otros escenarios para recuperar cambios de dicho plan
* Se agregó compatibilidad con `__repr__` para objetos entidad de Cosmos.
* Documentación actualizada.

### <a name="400b4"></a>4.0.0b4

* Se agregó compatibilidad para un argumento de palabra clave `timeout` a todas las operaciones con el fin de especificar un tiempo de espera absoluto en segundos en el que se debe completar la operación. Si se supera el valor de tiempo de espera, se generará `azure.cosmos.errors.CosmosClientTimeoutError`.

* Se agregó un nuevo `ConnectionRetryPolicy` para administrar el comportamiento de reintentos durante los errores de conexión HTTP.

* Se agregaron nuevos argumentos de la palabra clave de configuración de constructor y por operación:

  * `retry_total`: número máximo de reintentos.
  * `retry_backoff_max`: tiempo de espera de reintentos máximo en segundos.
  * `retry_fixed_interval`: intervalo de reintentos fijado en milisegundos.
  * `retry_read`: número máximo de reintentos de lectura de sockets.
  * `retry_connect`: número máximo de reintentos de error de conexión.
  * `retry_status`: número máximo de reintentos con códigos de estado de error.
  * `retry_on_status_codes`: lista de códigos de estado específicos en los que se va a reintentar.
  * `retry_backoff_factor`: factor para calcular el tiempo de espera entre reintentos.

### <a name="400b3"></a>4.0.0b3

* Se han agregado las funcionalidades `create_database_if_not_exists()` y `create_container_if_not_exists` a CosmosClient y Database, respectivamente.

### <a name="400b2"></a>4.0.0b2

* La versión 4.0.0b2 es la segunda iteración en nuestros esfuerzos por crear una biblioteca cliente que se adapte a los procedimientos recomendados del lenguaje Python.

**Cambios importantes**

* La conexión del cliente se ha adaptado para consumir la canalización HTTP definida en `azure.core.pipeline`.

* Ahora se ha cambiado el nombre de los objetos interactivos como servidores proxy. Esto incluye:

  * `Database` -> `DatabaseProxy`
  * `User` -> `UserProxy`
  * `Container` -> `ContainerProxy`
  * `Scripts` -> `ScriptsProxy`

* Se ha actualizado el constructor de `CosmosClient`:

  * Se ha cambiado el nombre del parámetro `auth` a `credential` y ahora tomará un tipo de autenticación directamente. Esto significa que se puede pasar el valor de la clave maestra, un diccionario de tokens de recursos o una lista de permisos. Sin embargo, todavía se admite el formato de diccionario antiguo.

  * El parámetro `connection_policy` se ha convertido en un parámetro solo de palabra clave y, aunque todavía se admite, cada uno de los atributos individuales de la directiva ahora se puede pasar como argumentos de palabra clave explícitos:

    * `request_timeout`
    * `media_request_timeout`
    * `connection_mode`
    * `media_read_mode`
    * `proxy_config`
    * `enable_endpoint_discovery`
    * `preferred_locations`
    * `multiple_write_locations`

* Se ha agregado un nuevo constructor a `CosmosClient` para habilitar la creación a través de una cadena de conexión recuperada de Azure Portal.

* Se ha cambiado el nombre de algunas operaciones `read_all` a operaciones `list`:

  * `CosmosClient.read_all_databases` -> `CosmosClient.list_databases`
  * `Container.read_all_conflicts` -> `ContainerProxy.list_conflicts`
  * `Database.read_all_containers` -> `DatabaseProxy.list_containers`
  * `Database.read_all_users` -> `DatabaseProxy.list_users`
  * `User.read_all_permissions` -> `UserProxy.list_permissions`

* Todas las operaciones que toman parámetros `request_options` o `feed_options`, se han migrado a parámetros solo de palabra clave. Además, aunque todavía se admiten estos diccionarios de opciones, cada una de las opciones individuales del diccionario ahora se admiten como argumentos de palabra clave explícitos.

* La jerarquía de errores ahora se hereda de `azure.core.AzureError`:

  * El nombre de `HTTPFailure` ha cambiado a `CosmosHttpResponseError`
  * `JSONParseFailure` se ha quitado y se ha reemplazado por `azure.core.DecodeError`
  * Se han agregado errores adicionales para códigos de respuesta específicos:
    * `CosmosResourceNotFoundError` para el estado 404
    * `CosmosResourceExistsError` para el estado 409
    * `CosmosAccessConditionFailedError` para el estado 412

* `CosmosClient` ahora se puede ejecutar en un administrador de contexto para controlar el cierre de la conexión de cliente.

* Las respuestas iterables (por ejemplo, las respuestas de consulta y las respuestas de lista) son ahora de tipo `azure.core.paging.ItemPaged`. El método `fetch_next_block` se ha reemplazado con un iterador secundario, al que el método `by_page` accede.

### <a name="400b1"></a>4.0.0b1

La versión 4.0.0b1 es la primera versión preliminar de nuestros esfuerzos por crear una biblioteca cliente sencilla que se adapte a los procedimientos recomendados del lenguaje Python. Para obtener más información sobre esto y las versiones preliminares de otras bibliotecas de Azure SDK, visite https://aka.ms/azure-sdk-preview1-python.

**Últimos cambios: Nuevo diseño de API**

* Ahora, las operaciones se limitan a un cliente determinado:

  * `CosmosClient`: este cliente controla las operaciones de nivel de cuenta. Esto incluye la administración de las propiedades del servicio y el listado de las bases de datos de una cuenta.
  * `Database`: este cliente controla las operaciones de nivel de base de datos. Esto incluye la creación y eliminación de contenedores, usuarios y procedimientos almacenados. Se puede acceder a él desde una instancia de cosmosClient por nombre.
  * `Container`: este cliente controla las operaciones para un contenedor determinado. Esto incluye consultar e insertar elementos y administrar propiedades.
  * `User`: este cliente controla las operaciones para un usuario determinado. Esto incluye agregar y eliminar permisos y administrar propiedades de usuario.

    Se puede acceder a estos clientes desplazándose hacia abajo por la jerarquía de clientes mediante el método `get_<child>_client`. Para obtener información completa sobre la nueva API, consulte la [documentación de referencia](https://aka.ms/azsdk-python-cosmos-ref).

* Se tiene acceso a los clientes por nombre en lugar de por identificador. No es necesario concatenar cadenas para crear vínculos.

* No es necesario importar tipos y métodos desde módulos individuales. El área de la superficie de la API pública está disponible directamente en el paquete `azure.cosmos`.

* Se pueden proporcionar propiedades de solicitud individuales como argumentos de palabra clave en lugar de construir una instancia `RequestOptions` independiente.

### <a name="302"></a>3.0.2

* Se agregó compatibilidad para el tipo de datos MultiPolygon
* Corrección de errores en la directiva de reintentos de lectura de la sesión
* Corrección de errores para los problemas de relleno incorrecto al descodificar cadenas en base 64

### <a name="301"></a>3.0.1

* Corrección de errores en LocationCache
* Corrección de errores de la lógica de reintentos de punto de conexión
* Documentación corregida

### <a name="300"></a>3.0.0

* Compatibilidad con escrituras de varias regiones agregada
* Cambios de nomenclatura
  * DocumentClient a CosmosClient
  * Colección a contenedor
  * Documento a elemento
  * Nombre del paquete actualizado a "azure-cosmos"
  * Espacio de nombres actualizado a "azure-cosmos"

### <a name="233"></a>2.3.3

* Se agregó compatibilidad con proxy.
* Se ha agregado compatibilidad para leer la fuente de cambios
* Se ha agregado compatibilidad con los encabezados de cuota de colección
* Corrección para el problema con los tokens de sesión grandes
* Corrección para ReadMedia API
* Corrección de errores en la memoria caché de intervalo de claves de partición

### <a name="232"></a>2.3.2

* Compatibilidad agregada para reintentos predeterminados en los problemas de conexión.

### <a name="231"></a>2.3.1

* Documentación actualizada para la referencia de Microsoft Azure Cosmos DB en lugar de Azure DocumentDB.

### <a name="230"></a>2.3.0

* Esta versión del SDK requiere la versión más reciente del emulador de Azure Cosmos DB disponible para su descarga desde https://aka.ms/cosmosdb-emulator.

### <a name="221"></a>2.2.1

* Corrección de errores para el diccionario de agregados
* Corrección de errores para recortar las barras diagonales en el vínculo de recursos
* Pruebas para la codificación unicode

### <a name="220"></a>2.2.0

* Se agregó compatibilidad con la característica Unidad de solicitud por minuto (RU/m).
* Se agregó compatibilidad con un nuevo nivel de coherencia denominado ConsistentPrefix.

### <a name="210"></a>2.1.0

* Se agregó compatibilidad con consultas de agregación (COUNT, MIN, MAX, SUM y AVG).
* Se agregó una opción para deshabilitar la comprobación de SSL cuando se ejecuta en el emulador de DocumentDB.
* Se ha quitado la restricción del módulo de solicitudes dependientes para que sea exactamente 2.10.0.
* Reducción del procesamiento mínimo en las colecciones particionadas de 10 100 RU/s a 2500 RU/s.
* Se ha agregado compatibilidad para habilitar el registro de scripts durante la ejecución de procedimientos almacenados.
* La versión de API de REST se incrementó a "2017-01-19" con esta versión.

### <a name="201"></a>2.0.1

* Se realizan los cambios editoriales de los comentarios de documentación.

### <a name="200"></a>2.0.0

* Compatibilidad agregada para Python 3.5.
* Compatibilidad agregada para agrupaciones de conexiones con el módulo de solicitudes.
* Compatibilidad agregada para la coherencia de la sesión.
* Compatibilidad agregada para consultas TOP/ORDERBY para colecciones particionadas.

### <a name="190"></a>1.9.0

* Se ha agregado compatibilidad de la directiva de reintentos con las solicitudes de limitación. (Las solicitudes limitadas reciben una excepción demasiado grande de la tasa de solicitudes, código de error 429). De manera predeterminada, DocumentDB realiza nueve reintentos para cada solicitud cuando aparece el código de error 429, cumpliendo el tiempo de retryAfter en el encabezado de respuesta.
  Ahora puede establecerse un tiempo del intervalo de reintento fijo como parte de la propiedad RetryOptions del objeto ConnectionPolicy si quiere ignorar el tiempo de retryAfter que ha devuelto el servidor entre los reintentos.
  DocumentDB espera ahora un máximo de 30 segundos para cada solicitud que se está limitando (independientemente del recuento de reintentos) y devuelve la respuesta con el código de error 429.
  Este tiempo también puede reemplazarse en la propiedad RetryOptions del objeto ConnectionPolicy.

* DocumentDB ahora devuelve x-ms-throttle-retry-count y x-ms-throttle-retry-wait-time-ms como los encabezados de respuesta de cada solicitud para denotar el recuento de reintentos de limitación y el tiempo acumulativo que ha esperado la solicitud entre los reintentos.

* Se ha quitado la clase RetryPolicy y la propiedad correspondiente (retry_policy) que estaba expuesta en la clase document_client y, en su lugar, se ha introducido una clase RetryOptions que expone la propiedad RetryOptions en la clase ConnectionPolicy que puede usarse para reemplazar algunas de las opciones de reintentos predeterminadas.

### <a name="180"></a>1.8.0

* Se ha agregado compatibilidad con cuentas de base de datos con replicación geográfica.
* Pruebe las correcciones para trasladar el host global y masterKey a las clases de prueba individuales.

### <a name="170"></a>1.7.0

* Se ha agregado compatibilidad con la característica de período de vida (TTL) para los documentos.

### <a name="161"></a>1.6.1

* Correcciones de errores relacionados con la creación de particiones del lado servidor para permitir caracteres especiales en la ruta de acceso de la clave de partición.

### <a name="160"></a>1.6.0

* Se agregó compatibilidad con la característica de colecciones con particiones del lado servidor.

### <a name="150"></a>1.5.0

* Se agregó el marco de particionamiento del lado cliente al SDK. Se implementaron las clases HashPartionResolver y RangePartitionResolver.

### <a name="142"></a>1.4.2

* Implementación de Upsert. Se han agregado nuevos métodos upsertXXX para admitir la característica Upsert.
* Se implementa el enrutamiento por identificador. Sin cambios en la API pública, todos los cambios son internos.

### <a name="130"></a>1.3.0

* Versión omitida para alinear el número de versión con otros SDK

### <a name="120"></a>1.2.0

* Compatible con índice geoespacial.
* Valida la propiedad ID para todos los recursos. Los identificadores de recursos no pueden contener los caracteres `?, /, #, \\` ni terminar con un espacio.
* Agrega el nuevo encabezado "progreso de transformación de índices" a ResourceResponse.

### <a name="110"></a>1.1.0

* Implementación de la directiva de indexación V2

### <a name="101"></a>1.0.1

* Compatibilidad con conexión de proxy

## <a name="release--retirement-dates"></a>Fechas de lanzamiento y retirada

Microsoft notifica la retirada de un SDK con al menos **12 meses** de antelación para facilitar la transición a una versión compatible o más reciente. Solo se agregan nuevas características, funcionalidad y optimizaciones al SDK actual, por lo que se recomienda actualizar siempre a la última versión del SDK tan pronto como sea posible.

> [!WARNING]
> Después del 31 de agosto de 2022, Azure Cosmos DB ya no hará correcciones de errores, no agregará nuevas características ni proporcionará soporte técnico para las versiones 1.x o 2.x del SDK de Python de Azure Cosmos DB para la API de SQL. Si prefiere no realizar la actualización, el servicio Azure Cosmos DB seguirá atendiendo las solicitudes enviadas desde la versión 1.x y 2.x del SDK.

| Versión | Fecha de la versión | Fecha de retirada |
| --- | --- | --- |
| [4.0.0](#400) |20 de mayo de 2020 |--- |
| [3.0.2](#302) |15 de noviembre de 2018 |--- |
| [3.0.1](#301) |04 de octubre de 2018 |--- |
| [2.3.3](#233) |08 de septiembre de 2018 |30 de agosto de 2020 |
| [2.3.2](#232) |8 de mayo de 2018 |30 de agosto de 2020 |
| [2.3.1](#231) |21 de diciembre de 2017 |30 de agosto de 2020 |
| [2.3.0](#230) |10 de noviembre de 2017 |30 de agosto de 2020 |
| [2.2.1](#221) |29 de septiembre de 2017 |30 de agosto de 2020 |
| [2.2.0](#220) |10 de mayo de 2017 |30 de agosto de 2020 |
| [2.1.0](#210) |1 de mayo de 2017 |30 de agosto de 2020 |
| [2.0.1](#201) |30 de octubre de 2016 |30 de agosto de 2020 |
| [2.0.0](#200) |29 de septiembre de 2016 |30 de agosto de 2020 |
| [1.9.0](#190) |7 de julio de 2016 |30 de agosto de 2020 |
| [1.8.0](#180) |14 de junio de 2016 |30 de agosto de 2020 |
| [1.7.0](#170) |26 de abril de 2016 |30 de agosto de 2020 |
| [1.6.1](#161) |08 de abril de 2016 |30 de agosto de 2020 |
| [1.6.0](#160) |29 de marzo de 2016 |30 de agosto de 2020 |
| [1.5.0](#150) |3 de enero de 2016 |30 de agosto de 2020 |
| [1.4.2](#142) |6 de octubre de 2015 |30 de agosto de 2020 |
| 1.4.1 |6 de octubre de 2015 |30 de agosto de 2020 |
| [1.2.0](#120) |6 de agosto de 2015 |30 de agosto de 2020 |
| [1.1.0](#110) |09 de julio de 2015 |30 de agosto de 2020 |
| [1.0.1](#101) |25 de mayo de 2015 |30 de agosto de 2020 |
| 1.0.0 |07 de abril de 2015 |30 de agosto de 2020 |
| 0.9.4-prelease |14 de enero de 2015 |29 de febrero de 2016 |
| 0.9.3-prelease |9 de diciembre de 2014 |29 de febrero de 2016 |
| 0.9.2-prelease |25 de noviembre de 2014 |29 de febrero de 2016 |
| 0.9.1-prelease |23 de septiembre de 2014 |29 de febrero de 2016 |
| 0.9.0-prelease |21 de agosto de 2014 |29 de febrero de 2016 |

## <a name="faq"></a>Preguntas más frecuentes

[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Cosmos DB, consulte la página del servicio [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/). 

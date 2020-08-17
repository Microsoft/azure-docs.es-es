---
title: 'Azure Cosmos DB: API de SQL para Node.js, SDK y recursos'
description: Obtenga toda la información sobre la API y el SDK de Node.js para SQL incluidas la fechas de lanzamiento, fechas de retirada y cambios realizados entre las versiones del SDK de Node.js para Azure Cosmos DB.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: nodejs
ms.topic: reference
ms.date: 05/11/2020
ms.author: anfeldma
ms.custom: devx-track-javascript
ms.openlocfilehash: ddbbb9f39dd17daf5c500bf582c100b4e1754a15
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87461364"
---
# <a name="azure-cosmos-db-nodejs-sdk-for-sql-api-release-notes-and-resources"></a>SDK de Azure Cosmos DB para .NET para SQL API: Notas de la versión y recursos
> [!div class="op_single_selector"]
> * [.NET SDK v3](sql-api-sdk-dotnet-standard.md)
> * [.NET SDK v2](sql-api-sdk-dotnet.md)
> * [SDK de .NET Core v2](sql-api-sdk-dotnet-core.md)
> * [SDK de fuente de cambios de .NET, versión 2](sql-api-sdk-dotnet-changefeed.md)
> * [Node.js](sql-api-sdk-node.md)
> * [SDK de Java v4](sql-api-sdk-java-v4.md)
> * [Versión 2 del SDK de Java asincrónico](sql-api-sdk-async-java.md)
> * [SDK de Java v2 sincrónico](sql-api-sdk-java.md)
> * [Python](sql-api-sdk-python.md)
> * [REST](/rest/api/cosmos-db/)
> * [Proveedor de recursos de REST](/rest/api/cosmos-db-resource-provider/)
> * [SQL](sql-api-query-reference.md)
> * [Bulk Executor: .NET v2](sql-api-sdk-bulk-executor-dot-net.md)
> * [Bulk Executor: Java](sql-api-sdk-bulk-executor-java.md)

|Recurso  |Vínculo  |
|---------|---------|
|Descargar SDK  |   [NPM](https://www.npmjs.com/package/@azure/cosmos) 
|Documentación de la API  |  [Documentación de referencia del SDK de JavaScript](https://docs.microsoft.com/javascript/api/%40azure/cosmos/?view=azure-node-latest)
|Instrucciones de instalación del SDK  |  [Instrucciones de instalación](https://github.com/Azure/azure-sdk-for-js)
|Contribuya al SDK | [GitHub](https://github.com/Azure/azure-cosmos-js/tree/master)
| Ejemplos | [Ejemplos de código Node.js](sql-api-nodejs-samples.md)
| Tutorial de inicio | [Introducción al SDK de JavaScript](sql-api-nodejs-get-started.md)
| Tutorial de la aplicación web | [Creación de una aplicación web de Node.js con Azure Cosmos DB](sql-api-nodejs-application.md)
| Plataforma admitida actualmente | [Node.js v12.x](https://nodejs.org/en/blog/release/v12.7.0/), versión 3.x.x del SDK<br/>[Node.js v10.x](https://nodejs.org/en/blog/release/v10.6.0/), versión 3.x.x del SDK<br/>[Node.js v8.x](https://nodejs.org/en/blog/release/v8.16.0/), versión 3.x.x del SDK<br/>[Node.js v6.x](https://nodejs.org/en/blog/release/v6.10.3/), versión 2.x.x del SDK<br/>[Node.js v4.2.0](https://nodejs.org/en/blog/release/v4.2.0/), versión 1.x.x del SDK<br/> [Node.js v0.12](https://nodejs.org/en/blog/release/v0.12.0/), versión 1.x.x del SDK<br/> [Node.js v0.10](https://nodejs.org/en/blog/release/v0.10.0/), versión 1.x.x del SDK

## <a name="release-notes"></a>Notas de la versión

### <a name="310"></a><a name="3.1.0"></a>3.1.0
* Establecer el valor predeterminado de ResponseContinuationTokenLimitInKB en 1 KB. El límite es 1 KB de forma predeterminada para evitar encabezados largos (Node.js tiene un límite de tamaño de encabezado global). Un usuario puede establecer este campo para que permita encabezados más largos, lo que puede ayudar al back-end a optimizar la ejecución de las consultas.
* Quitar disableSSLVerification. Esta opción tiene nuevas alternativas que se describen en [#388](https://github.com/Azure/azure-cosmos-js/pull/388).

### <a name="304"></a><a name="3.0.4"></a>3.0.4
* Permitir que initialHeaders establezca explícitamente el encabezado de clave de partición.
* Usar package.json#files para evitar que se publiquen archivos extraños.
* Corregir el error de ordenación del mapa de enrutamiento en una versión anterior de node+v8.
* Corrige el error cuando el usuario proporciona opciones de reintento parciales.

### <a name="303"></a><a name="3.0.3"></a>3.0.3
* Impedir que Webpack resuelva los módulos llamados con require.

### <a name="302"></a><a name="3.0.2"></a>3.0.2
* Corrige un error de mucho tiempo en el que siempre se notificaban 0 RU para las consultas de agregado.

### <a name="300"></a><a name="3.0.0"></a>3.0.0

🎉 Versión v3. 🎉 Muchas características nuevas, correcciones de errores y algunos cambios con un impacto importante. Objetivos principales de esta versión:

* Implementar nuevas características principales.
  * Consultas DISTINCT
  * Consultas LIMIT/OFFSET
  * Solicitudes que puede cancelar el usuario
* Actualizar a la última versión de la API REST de Cosmos, donde todos los contenedores tienen una escala ilimitada.
* Facilitar el uso de Cosmos desde el explorador.
* Mejorar la alineación con las nuevas instrucciones del SDK de Azure para JavaScript.

#### <a name="migration-guide-for-breaking-changes"></a>Guía de migración para cambios con un impacto importante
##### <a name="improved-client-constructor-options"></a>Opciones de constructor de cliente mejoradas

Las opciones de constructor se han simplificado:

* Se ha cambiado el nombre de la clave masterKey y se ha movido al nivel superior.
* Las propiedades que antes estaban en options.auth se han movido al nivel superior.

``` js
// v2
const client = new CosmosClient({
    endpoint: "https://your-database.cosmos.azure.com",
    auth: {
        masterKey: "your-primary-key"
    }
})

// v3
const client = new CosmosClient({
    endpoint: "https://your-database.cosmos.azure.com",
    key: "your-primary-key"
})
```

##### <a name="simplified-queryiterator-api"></a>API QueryIterator simplificada
En la versión 2 había muchas formas diferentes de iterar o recuperar los resultados de una consulta. Hemos intentado simplificar la API v3 y quitar API similares o duplicadas:

* Quitar iterator.next() e iterator.current(). Usar fetchNext() para obtener páginas de resultados.
* Quitar iterator.forEach(). Usar iteradores asincrónicos en su lugar.
* Se ha cambiado el nombre de iterator.executeNext() a iterator.fetchNext().
* Se ha cambiado el nombre de iterator.toArray() a iterator.fetchAll().
* Ahora las páginas son objetos de respuesta adecuados en lugar de objetos de JS estándar.
* const container = client.database(dbId).container(containerId).

``` js
// v2
container.items.query('SELECT * from c').toArray()
container.items.query('SELECT * from c').executeNext()
container.items.query('SELECT * from c').forEach(({ body: item }) => { console.log(item.id) })

// v3
container.items.query('SELECT * from c').fetchAll()
container.items.query('SELECT * from c').fetchNext()
for await(const { result: item } in client.databases.readAll().getAsyncIterator()) {
    console.log(item.id)
}
```

##### <a name="fixed-containers-are-now-partitioned"></a>Los contenedores fijos ahora tienen particiones
El servicio Cosmos ahora admite claves de partición en todos los contenedores, incluidos los que se habían creado como contenedores fijos. El SDK v3 actualiza a la última versión de la API, que implementa este cambio, pero no tiene un impacto importante. Si no proporciona una clave de partición para las operaciones, se usará de forma predeterminada una clave del sistema que funcione con todos los contenedores y documentos actuales.

##### <a name="upsert-removed-for-stored-procedures"></a>Se ha quitado upsert para procedimientos almacenados
Antes se permitía upsert para colecciones sin particiones, pero, con la actualización de la versión de la API, todas las colecciones se particionan, así que se ha quitado por completo.

##### <a name="item-reads-will-not-throw-on-404"></a>Las lecturas de elementos no producirán el error 404
const container = client.database(dbId).container(containerId).

``` js
// v2
try {
    container.items.read(id, undefined)
} catch (e) {
    if (e.code === 404) { console.log('item not found') }
}

// v3
const { result: item }  = container.items.read(id, undefined)
if (item === undefined) { console.log('item not found') }
```

##### <a name="default-multi-region-write"></a>Escritura en varias regiones predeterminada
Ahora el SDK escribirá en varias regiones de forma predeterminada si la configuración de Cosmos lo admite. Antes este era un comportamiento opcional.

##### <a name="proper-error-objects"></a>Objetos de error correctos
Ahora las solicitudes que no se pueden ejecutar devuelven un error o subclases de error correctos. Antes devolvían objetos de JS estándar.

#### <a name="new-features"></a>Nuevas características
##### <a name="user-cancelable-requests"></a>Solicitudes que puede cancelar el usuario
El cambio a la recuperación de cambios interna nos permite usar la API AbortController del explorador para admitir operaciones que puede cancelar el usuario. En el caso de las operaciones en las que puede haber varias solicitudes en curso (como las consultas entre particiones), se cancelarán todas las solicitudes de la operación. Los usuarios de los nuevos exploradores ya tendrán AbortController. Los usuarios de Node.js deberán usar una biblioteca de polyfill.

``` js
 const controller = new AbortController()
 const {result: item} = await items.query('SELECT * from c', { abortSignal: controller.signal});
 controller.abort()
```

##### <a name="set-throughput-as-part-of-dbcontainer-create-operation"></a>Establecer el procesamiento como parte de la operación de creación de una base de datos o un contenedor
``` js
const { database }  = client.databases.create({ id: 'my-database', throughput: 10000 })
database.containers.create({ id: 'my-container', throughput: 10000 })
```

##### <a name="azurecosmos-sign"></a>@azure/cosmos-sign
La generación de tokens de encabezado se ha separado en una biblioteca nueva, @azure/cosmos-sign. Cualquiera que llame directamente a la API REST de Cosmos puede utilizarla para firmar encabezados con el mismo código que se llama en @azure/cosmos.

##### <a name="uuid-for-generated-ids"></a>UUID para identificadores generados
La versión 2 tenía código personalizado para generar id. de elemento. Hemos cambiado al conocido y mantenido UUID de biblioteca de la comunidad.

##### <a name="connection-strings"></a>Cadenas de conexión
Ahora se puede pasar una cadena de conexión copiada de Azure Portal:

``` js
const client = new CosmosClient("AccountEndpoint=https://test-account.documents.azure.com:443/;AccountKey=c213asdasdefgdfgrtweaYPpgoeCsHbpRTHhxuMsTaw==;")
Add DISTINCT and LIMIT/OFFSET queries (#306)
 const { results } = await items.query('SELECT DISTINCT VALUE r.name FROM ROOT').fetchAll()
 const { results } = await items.query('SELECT * FROM root r OFFSET 1 LIMIT 2').fetchAll()
```

#### <a name="improved-browser-experience"></a>Experiencia de explorador mejorada
Aunque antes se podía usar el SDK v2 en el explorador, no era una experiencia idónea. Había que usar polyfill en varias bibliotecas integradas de Node.js y un empaquetador, como Webpack o Parcel. El SDK v3 mejora la experiencia de configuración rápida para los usuarios de explorador.

* Reemplazar los componentes internos de las solicitudes con fetch (#245).
* Quitar el uso de búfer (#330).
* Quitar el uso integrado de nodos en favor de paquetes universales o API (#328).
* Cambiar a node-abort-controller (#294).

#### <a name="bug-fixes"></a>Corrección de errores
* Corregir la lectura de las ofertas y restablecer las pruebas de ofertas (#224).
* Corregir EnableEndpointDiscovery (#207).
* Corregir las RU que faltan en los resultados paginados (#360).
* Expandir el tipo de parámetro de consulta SQL (#346).
* Agregar TTL a ItemDefinition (#341).
* Corregir las métricas de las consultas de CP (#311).
* Agregar activityId a FeedResponse (#293).
* Cambiar el tipo de _ts de cadena a número (#252)(#295).
* Corregir la agregación de cargos de solicitudes (#289).
* Permitir claves de partición de cadenas en blanco (#277).
* Agregar cadena al tipo de consulta de conflicto (#237).
* Agregar uniqueKeyPolicy al contenedor (#234).

#### <a name="engineering-systems"></a>Sistemas de ingeniería
No son siempre los cambios más visibles, pero ayudan a nuestro equipo a distribuir un código de más calidad en menos tiempo.

* Usar rollup para compilaciones de producción (#104).
* Actualizar a Typescript 3.5 (#327).
* Convertir en referencias de proyecto de TS. Extraer la carpeta de prueba (#270).
* Habilitar noUnusedLocals y noUnusedParameters (#275).
* Azure Pipelines YAML para las compilaciones de CI (#298).

### <a name="215"></a><a name="2.1.5"></a>2.1.5
* No hay cambios de código. Corrige un problema por el que se incluían algunos archivos adicionales en el paquete 2.1.4.

### <a name="214"></a><a name="2.1.4"></a>2.1.4
* Corregir la conmutación por error regional en la directiva de reintentos.
* Corregir la propiedad hasMoreResults de ChangeFeed.
* Actualizaciones de dependencias de desarrollo.
* Agregar PolicheckExclusions.txt.

### <a name="213"></a><a name="2.1.3"></a>2.1.3
* Cambiar el tipo de _ts de cadena a número.
* Corregir las pruebas de indexación predeterminadas.
* Retroimportar (backport) uniqueKeyPolicy en la versión 2.
* Correcciones de demostraciones y en la depuración de demostraciones.

### <a name="212"></a><a name="2.1.2"></a>2.1.2
* Retroimportar (backport) en la oferta correcciones de la rama v3.
* Corregir un error en la signatura de tipos de executeNext().
* Correcciones de errores de escritura.

### <a name="211"></a><a name="2.1.1"></a>2.1.1
* Reestructuración de la compilación. Permite extraer la versión del SDK en el momento de la compilación.

### <a name="210"></a><a name="2.1.0"></a>2.1.0
#### <a name="new-features"></a>Nuevas características
* Se ha agregado compatibilidad con ChangeFeed (#196).
* Se ha agregado el tipo de datos MultiPolygon para la indexación (#191)
* Agregar la propiedad "key" al constructor como alias para masterKey (#202).

#### <a name="fixes"></a>Correcciones
* Corregir el error en el que next() devuelve un valor incorrecto en el iterador.

#### <a name="engineering-improvements"></a>Mejoras de ingeniería
* Agregar una prueba de integración para el consumo de TypeScript (#199).
* Habilitar la instalación directamente desde GitHub (#194).

### <a name="205"></a><a name="2.0.5"></a>2.0.5
* Agrega la interfaz para el tipo de agente de nodo. Los usuarios de Typescript ya no tendrán que instalar @types/node como dependencia.
* Ahora se respetan debidamente las ubicaciones preferidas.
* Mejoras en la documentación para desarrolladores colaboradores.
* Varias correcciones de errores de escritura.

### <a name="204"></a><a name="2.0.4"></a>2.0.4
* Corrige el problema de definición de tipo detectado en 2.0.3.

### <a name="203"></a><a name="2.0.3"></a>2.0.3
* Se quita la dependencia `big-integer`.
* Se cambia a directivas de referencia para el tipo AsyncIterable. Los usuarios de Typescript ya no tienen que personalizar su configuración de "lib".
* Correcciones de errores de escritura.

### <a name="202"></a><a name="2.0.2"></a>2.0.2
* Se corrigen vínculos de archivo Léame.

### <a name="201"></a><a name="2.0.1"></a>2.0.1
* Se corrige la implementación de la interfaz de reintento.

### <a name="200"></a><a name="2.0.0"></a>2.0.0
* La disponibilidad general de la versión 2.0.0 del SDK de JavaScript
* Se agregó compatibilidad para escrituras en varias regiones.

### <a name="200-3"></a><a name="2.0.0-3"></a>2.0.0-3
* RC1 de versión 2.0.0 del SDK de JavaScript de la versión preliminar pública.
* Nuevo modelo de objetos, con CosmosClient de nivel superior y métodos divididos entre las clases pertinentes Database, Container e Item. 
* Compatibilidad con [promises](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Using_promises). 
* SDK se convierte en TypeScript.

### <a name="1144"></a><a name="1.14.4"></a>1.14.4
* Documentación de NPM corregida.

### <a name="1143"></a><a name="1.14.3"></a>1.14.3
* Compatibilidad agregada para reintentos predeterminados en los problemas de conexión.
* Compatibilidad agregada para leer la fuente de cambios de la colección.
* Error de coherencia con la sesión corregido que de vez en cuando generó el mensaje "sesión de lectura no disponible".
* Compatibilidad agregada para métricas de consulta.
* Número máximo de conexiones modificado del Agente de HTTP.

### <a name="1142"></a><a name="1.14.2"></a>1.14.2
* Documentación actualizada para la referencia de Microsoft Azure Cosmos DB en lugar de Azure DocumentDB.
* Se agregó compatibilidad para la configuración de proxyUrl en ConnectionPolicy.

### <a name="1141"></a><a name="1.14.1"></a>1.14.1
* Revisión secundaria para sistemas de archivos con distinción de mayúsculas y minúsculas.

### <a name="1140"></a><a name="1.14.0"></a>1.14.0
* Agrega compatibilidad para la coherencia de la sesión.
* Esta versión del SDK requiere la versión más reciente del [emulador de Azure Cosmos DB](https://aka.ms/cosmosdb-emulator).

### <a name="1130"></a><a name="1.13.0"></a>1.13.0
* Consultas entre particiones a prueba de divisiones.
* Agrega compatibilidad para vínculos de recursos con barras diagonales iniciales y finales (y sus correspondientes pruebas).

### <a name="1122"></a><a name="1.12.2"></a>1.12.2
*    Documentación de NPM corregida.

### <a name="1121"></a><a name="1.12.1"></a>1.12.1
* Se ha corregido un error en executeStoredProcedure en el que los documentos implicados tenían caracteres especiales de Unicode (LS, PS).
* Se ha corregido un error a la hora de administrar los documentos con caracteres Unicode en la clave de partición.
* Se ha corregido el problema de compatibilidad con la creación de colecciones con los medios de nombre. Problema de GitHub n.º 114.
* Se ha corregido el problema de compatibilidad con el token de autorización de permiso. Problema de GitHub n.º 178.

### <a name="1120"></a><a name="1.12.0"></a>1.12.0
* Se agregó compatibilidad con un nuevo [nivel de coherencia](consistency-levels.md) denominado ConsistentPrefix.
* Se agregó compatibilidad con UriFactory.
* Se ha corregido un error de compatibilidad con Unicode. Problema de GitHub n.º 171.

### <a name="1110"></a><a name="1.11.0"></a>1.11.0
* Se agregó compatibilidad con consultas de agregación (COUNT, MIN, MAX, SUM y AVG).
* Se agregó la opción para controlar el grado de paralelismo de las consultas de partición cruzada.
* Se agregó la opción para deshabilitar la comprobación de TLS cuando se ejecuta en el emulador de Azure Cosmos DB.
* Reducción del procesamiento mínimo en las colecciones particionadas de 10 100 RU/s a 2500 RU/s.
* Se ha corregido el error de token de continuación de la colección de una sola partición. Problema de GitHub n.º 107.
* Se ha corregido el error executeStoredProcedure en el control 0 como parámetro único. Problema de GitHub n.º 155.

### <a name="1102"></a><a name="1.10.2"></a>1.10.2
* Encabezado de agente de usuario fijo para incluir la versión del SDK.
* Limpieza menor de código.

### <a name="1101"></a><a name="1.10.1"></a>1.10.1
* Se ha deshabilitado la comprobación de TLS cuando se usa el SDK para alcanzar el emulador (nombre de host = host local).
* Se ha agregado compatibilidad para habilitar el registro de scripts durante la ejecución de procedimientos almacenados.

### <a name="1100"></a><a name="1.10.0"></a>1.10.0
* Compatibilidad agregada con las consultas paralelas entre particiones.
* Se ha agregado compatibilidad con las consultas TOP y ORDER BY en las colecciones particionadas.

### <a name="190"></a><a name="1.9.0"></a>1.9.0
* Se ha agregado compatibilidad de la directiva de reintentos con las solicitudes de limitación. (Las solicitudes limitadas reciben una excepción demasiado grande de la tasa de solicitudes, código de error 429). De manera predeterminada, Azure Cosmos DB realiza nueve reintentos para cada solicitud cuando aparece el código de error 429, respetando el tiempo de retryAfter en el encabezado de respuesta. Ahora puede establecerse un tiempo del intervalo de reintento fijo como parte de la propiedad RetryOptions del objeto ConnectionPolicy si quiere ignorar el tiempo de retryAfter que ha devuelto el servidor entre los reintentos. Azure Cosmos DB espera ahora un máximo de 30 segundos para cada solicitud que se está limitando (independientemente del recuento de reintentos) y devuelve la respuesta con el código de error 429. Este tiempo también puede reemplazarse en la propiedad RetryOptions del objeto ConnectionPolicy.
* Cosmos DB ahora devuelve x-ms-throttle-retry-count y x-ms-throttle-retry-wait-time-ms como los encabezados de respuesta de cada solicitud para denotar el recuento de reintentos de limitación y el tiempo acumulativo que ha esperado la solicitud entre los reintentos.
* Se ha agregado la clase RetryOptions que expone la propiedad RetryOptions en la clase ConnectionPolicy que puede usarse para reemplazar algunas de las opciones de reintentos predeterminadas.

### <a name="180"></a><a name="1.8.0"></a>1.8.0
* Se ha agregado compatibilidad con cuentas de base de datos de varias regiones.

### <a name="170"></a><a name="1.7.0"></a>1.7.0
* Se ha agregado compatibilidad con la característica de período de vida (TTL) para los documentos.

### <a name="160"></a><a name="1.6.0"></a>1.6.0
* Se han implementado [colecciones particionadas](partition-data.md) y [niveles de rendimiento definidos por el usuario](performance-levels.md).

### <a name="156"></a><a name="1.5.6"></a>1.5.6
* Solucionado el error RangePartitionResolver.resolveForRead por el que no se devolvían vínculos debido a una mala concatenación de resultados.

### <a name="155"></a><a name="1.5.5"></a>1.5.5
* Valor hashParitionResolver resolveForRead() corregido: cuando ninguna clave de partición proporcionada ha lanzado una excepción, en lugar de devolver una lista de todos los vínculos registrados.

### <a name="154"></a><a name="1.5.4"></a>1.5.4
* Corrige el problema [100](https://github.com/Azure/azure-documentdb-node/issues/100) -agente específico de HTTPS: evite la modificación del agente global para fines de Azure Cosmos DB. Use un agente dedicado para todas las solicitudes de LIB.

### <a name="153"></a><a name="1.5.3"></a>1.5.3
* Corrige el problema [81](https://github.com/Azure/azure-documentdb-node/issues/81): controla correctamente guiones en identificadores de medios.

### <a name="152"></a><a name="1.5.2"></a>1.5.2
* Corrige el problema [95](https://github.com/Azure/azure-documentdb-node/issues/95) : advertencia de pérdida de escucha de EventEmitter.

### <a name="151"></a><a name="1.5.1"></a>1.5.1
* Corrige el problema [92](https://github.com/Azure/azure-documentdb-node/issues/90): cambio de nombre de la carpeta Hash a hash para sistemas que distinguen mayúsculas de minúsculas.

### <a name="150"></a><a name="1.5.0"></a>1.5.0
* Se implementa la compatibilidad con el particionamiento, para lo que se agregan resolvedores de hash y de particiones de intervalo.

### <a name="140"></a><a name="1.4.0"></a>1.4.0
* Implementación de Upsert. Se han agregado nuevos métodos upsertXXX en documentClient.

### <a name="130"></a><a name="1.3.0"></a>1.3.0
* Omitida para alinear el número de versión con otros SDK.

### <a name="122"></a><a name="1.2.2"></a>1.2.2
* Divide el contenedor de objetos Q promise en nuevo repositorio.
* Actualización del archivo de paquete para registro npm.

### <a name="121"></a><a name="1.2.1"></a>1.2.1
* Se implementa el enrutamiento por identificador.
* Corrige el problema [49](https://github.com/Azure/azure-documentdb-node/issues/49) : la propiedad current entra en conflicto con el método current().

### <a name="120"></a><a name="1.2.0"></a>1.2.0
* Se agregó compatibilidad con índice geoespacial.
* Valida la propiedad ID para todos los recursos. Los identificadores de recursos no pueden contener los caracteres ?, /, #, &#47;&#47;, ni terminar con un espacio.
* Agrega el nuevo encabezado "progreso de transformación de índices" a ResourceResponse.

### <a name="110"></a><a name="1.1.0"></a>1.1.0
* Implementación de la directiva de indexación V2.

### <a name="103"></a><a name="1.0.3"></a>1.0.3
* Problema [40](https://github.com/Azure/azure-documentdb-node/issues/40) : se implementaron configuraciones eslint y grunt en el núcleo y el SDK de promise.

### <a name="102"></a><a name="1.0.2"></a>1.0.2
* Problema [#45](https://github.com/Azure/azure-documentdb-node/issues/45) : el contenedor de objetos promise no incluye el encabezado con el error.

### <a name="101"></a><a name="1.0.1"></a>1.0.1
* Implementación de capacidad para consultar conflictos agregando readConflicts, readConflictAsync y queryConflicts.
* Documentación de la API actualizada.
* Problema [41](https://github.com/Azure/azure-documentdb-node/issues/41) : error client.createDocumentAsync.

### <a name="100"></a><a name="1.0.0"></a>1.0.0
* SDK de GA.

## <a name="release--retirement-dates"></a>Fechas de lanzamiento y de retirada

Microsoft notifica la retirada de un SDK con al menos **12 meses** de antelación para facilitar la transición a una versión compatible o más reciente. Solo se agregan nuevas características, funcionalidad y optimizaciones al SDK actual, por lo que se recomienda actualizar siempre a la última versión del SDK tan pronto como sea posible.

| Versión | Fecha de la versión | Fecha de retirada |
| --- | --- | --- |
| [3.1.0](#3.1.0) |26 de julio de 2019 |--- |
| [3.0.4](#3.0.4) |22 de julio de 2019 |--- |
| [3.0.3](#3.0.3) |17 de julio de 2019 |--- |
| [3.0.2](#3.0.2) |9 de julio de 2019 |--- |
| [3.0.0](#3.0.0) |28 de junio de 2019 |--- |
| [2.1.5](#2.1.5) |20 de marzo de 2019 |--- |
| [2.1.4](#2.1.4) |15 de marzo de 2019 |--- |
| [2.1.3](#2.1.3) |8 de marzo de 2019 |--- |
| [2.1.2](#2.1.2) |28 de enero de 2019 |--- |
| [2.1.1](#2.1.1) |5 de diciembre de 2018 |--- |
| [2.1.0](#2.1.0) |4 de diciembre de 2018 |--- |
| [2.0.5](#2.0.5) |7 de noviembre de 2018 |--- |
| [2.0.4](#2.0.4) |30 de octubre de 2018 |--- |
| [2.0.3](#2.0.3) |30 de octubre de 2018 |--- |
| [2.0.2](#2.0.2) |10 de octubre de 2018 |--- |
| [2.0.1](#2.0.1) |25 de septiembre de 2018 |--- |
| [2.0.0](#2.0.0) |24 de septiembre de 2018 |--- |
| [2.0.0-3 (RC)](#2.0.0-3) |2 de agosto de 2018 |--- |
| [1.14.4](#1.14.4) |3 de mayo de 2018 |30 de agosto de 2020 |
| [1.14.3](#1.14.3) |3 de mayo de 2018 |30 de agosto de 2020 |
| [1.14.2](#1.14.2) |21 de diciembre de 2017 |30 de agosto de 2020 |
| [1.14.1](#1.14.1) |10 de noviembre de 2017 |30 de agosto de 2020 |
| [1.14.0](#1.14.0) |9 de noviembre de 2017 |30 de agosto de 2020 |
| [1.13.0](#1.13.0) |11 de octubre de 2017 |30 de agosto de 2020 |
| [1.12.2](#1.12.2) |10 de agosto de 2017 |30 de agosto de 2020 |
| [1.12.1](#1.12.1) |10 de agosto de 2017 |30 de agosto de 2020 |
| [1.12.0](#1.12.0) |10 de mayo de 2017 |30 de agosto de 2020 |
| [1.11.0](#1.11.0) |16 de marzo de 2017 |30 de agosto de 2020 |
| [1.10.2](#1.10.2) |27 de enero de 2017 |30 de agosto de 2020 |
| [1.10.1](#1.10.1) |22 de diciembre de 2016 |30 de agosto de 2020 |
| [1.10.0](#1.10.0) |03 de octubre de 2016 |30 de agosto de 2020 |
| [1.9.0](#1.9.0) |7 de julio de 2016 |30 de agosto de 2020 |
| [1.8.0](#1.8.0) |14 de junio de 2016 |30 de agosto de 2020 |
| [1.7.0](#1.7.0) |26 de abril de 2016 |30 de agosto de 2020 |
| [1.6.0](#1.6.0) |29 de marzo de 2016 |30 de agosto de 2020 |
| [1.5.6](#1.5.6) |8 de marzo de 2016 |30 de agosto de 2020 |
| [1.5.5](#1.5.5) |2 de febrero de 2016 |30 de agosto de 2020 |
| [1.5.4](#1.5.4) |1 de febrero de 2016 |30 de agosto de 2020 |
| [1.5.2](#1.5.2) |26 de enero de 2016 |30 de agosto de 2020 |
| [1.5.2](#1.5.2) |22 de enero de 2016 |30 de agosto de 2020 |
| [1.5.1](#1.5.1) |4 de enero de 2016 |30 de agosto de 2020 |
| [1.5.0](#1.5.0) |31 de diciembre de 2015 |30 de agosto de 2020 |
| [1.4.0](#1.4.0) |6 de octubre de 2015 |30 de agosto de 2020 |
| [1.3.0](#1.3.0) |6 de octubre de 2015 |30 de agosto de 2020 |
| [1.2.2](#1.2.2) |10 de septiembre de 2015 |30 de agosto de 2020 |
| [1.2.1](#1.2.1) |15 de agosto de 2015 |30 de agosto de 2020 |
| [1.2.0](#1.2.0) |05 de agosto de 2015 |30 de agosto de 2020 |
| [1.1.0](#1.1.0) |09 de julio de 2015 |30 de agosto de 2020 |
| [1.0.3](#1.0.3) |4 de junio de 2015 |30 de agosto de 2020 |
| [1.0.2](#1.0.2) |23 de mayo de 2015 |30 de agosto de 2020 |
| [1.0.1](#1.0.1) |15 de mayo de 2015 |30 de agosto de 2020 |
| [1.0.0](#1.0.0) |08 de abril de 2015 |30 de agosto de 2020 |

## <a name="faq"></a>Preguntas más frecuentes
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="see-also"></a>Consulte también
Para más información sobre Cosmos DB, consulte la página del servicio [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).


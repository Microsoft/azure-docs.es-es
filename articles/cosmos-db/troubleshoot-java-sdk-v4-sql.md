---
title: Diagnóstico y solución de problemas del SDK de Azure Cosmos DB para Java v4
description: Use características como el registro del lado cliente y otras herramientas de terceros para identificar, diagnosticar y solucionar problemas de Azure Cosmos DB en el SDK para Java v4.
author: anfeldma-ms
ms.service: cosmos-db
ms.date: 06/11/2020
ms.author: anfeldma
ms.devlang: java
ms.subservice: cosmosdb-sql
ms.topic: troubleshooting
ms.custom: devx-track-java
ms.openlocfilehash: 67813aa36b0e0824db3ed89c7b7dbc06c3fd46d8
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87321044"
---
# <a name="troubleshoot-issues-when-you-use-azure-cosmos-db-java-sdk-v4-with-sql-api-accounts"></a>Solución de problemas con el uso del SDK de Azure Cosmos DB para Java v4 con cuentas de SQL API

> [!div class="op_single_selector"]
> * [SDK para Java v4](troubleshoot-java-sdk-v4-sql.md)
> * [SDK sincrónico para Java v2](troubleshoot-java-async-sdk.md)
> * [.NET](troubleshoot-dot-net-sdk.md)
> 

> [!IMPORTANT]
> Este artículo trata solo sobre la solución de problemas con el SDK de Azure Cosmos DB para Java v4. Consulte las [notas de la versión](sql-api-sdk-java-v4.md) del SDK de Azure Cosmos DB para Java v4, el [repositorio de Maven](https://mvnrepository.com/artifact/com.azure/azure-cosmos) y las [sugerencias de rendimiento](performance-tips-java-sdk-v4-sql.md) para más información. Si en la actualidad usa una versión anterior a la v4, consulte la guía [Migración al SDK de Azure Cosmos DB para Java v4](migrate-java-v4-sdk.md) para obtener ayuda para actualizar a la versión v4.
>

En este artículo se tratan problemas comunes, soluciones alternativas, pasos de diagnóstico y herramientas al usar el SDK de Azure Cosmos DB para Java v4 con cuentas de SQL API de Azure Cosmos DB.
El SDK de Azure Cosmos DB para Java v4 proporciona la representación lógica del lado cliente para acceder a SQL API de Azure Cosmos DB. En este artículo se describen herramientas y enfoques para ayudarle si surge algún problema.

Comience con esta lista:

* Eche un vistazo a la sección [Problemas comunes y soluciones alternativas] de este artículo.
* Consulte el SDK para Java en el repositorio central de Azure Cosmos DB, que está disponible como [código abierto en GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cosmos/azure-cosmos). Tiene una [sección de problemas](https://github.com/Azure/azure-sdk-for-java/issues) que se supervisa activamente. Compruebe si encuentra algún problema similar con una solución alternativa ya registrada. Una sugerencia útil es filtrar los problemas por la etiqueta *cosmos:v4-item*.
* Revise las [sugerencias de rendimiento](performance-tips-java-sdk-v4-sql.md) del SDK de Azure Cosmos DB para Java v4 y siga los procedimientos sugeridos.
* Lea el resto de este artículo y, si no encuentra una solución, registre un [problema de GitHub](https://github.com/Azure/azure-sdk-for-java/issues). Si hay una opción para agregar etiquetas al problema de GitHub, agregue la etiqueta *cosmos:v4-item*.

## <a name="common-issues-and-workarounds"></a><a name="common-issues-workarounds"></a>Problemas comunes y soluciones alternativas

### <a name="network-issues-netty-read-timeout-failure-low-throughput-high-latency"></a>Problemas de red, error de tiempo de espera de lectura de Netty, rendimiento bajo, latencia alta

#### <a name="general-suggestions"></a>Sugerencias generales
Para obtener el mejor rendimiento:
* Asegúrese de que la aplicación se está ejecutando en la misma región que la cuenta de Azure Cosmos DB. 
* Compruebe el uso de la CPU en el host donde se ejecuta la aplicación. Si el uso de CPU es de un 50 por ciento o más, ejecute la aplicación en un host con una configuración mayor. O bien, distribuya la carga en más máquinas.
    * Si ejecuta la aplicación en Azure Kubernetes Service, puede [usar Azure Monitor para supervisar el uso de CPU](https://docs.microsoft.com/azure/azure-monitor/insights/container-insights-analyze).

#### <a name="connection-throttling"></a>Limitación de la conexión
La limitación de la conexión puede deberse a un [Límite de conexiones en una máquina host] o al [agotamiento de puertos SNAT (PAT) de Azure].

##### <a name="connection-limit-on-a-host-machine"></a><a name="connection-limit-on-host"></a>Límite de conexiones en una máquina host
Algunos sistemas Linux, como Red Hat, tienen un límite máximo para el número total de archivos abiertos. En Linux, los sockets se implementan como archivos, por lo que este número también limita el número total de conexiones.
Ejecute el siguiente comando:

```bash
ulimit -a
```
El número máximo permitido de archivos abiertos, que se identifican como "nofile", debe al menos doblar el tamaño del grupo de conexiones. Para más información, consulte [Sugerencias de rendimiento](performance-tips-java-sdk-v4-sql.md) del SDK de Azure Cosmos DB para Java v4.

##### <a name="azure-snat-pat-port-exhaustion"></a><a name="snat"></a>Agotamiento de puertos SNAT (PAT) de Azure

Si la aplicación está implementada en Azure Virtual Machines sin una dirección IP pública, los [puertos SNAT de Azure](https://docs.microsoft.com/azure/load-balancer/load-balancer-outbound-connections#preallocatedports) se usan de manera predeterminada para establecer conexiones con cualquier punto de conexión fuera de la máquina virtual. El número de conexiones permitidas desde la máquina virtual hasta el punto de conexión de Azure Cosmos DB está limitado por la [configuración de Azure SNAT](https://docs.microsoft.com/azure/load-balancer/load-balancer-outbound-connections#preallocatedports).

 Los puertos SNAT de Azure se usan solo cuando la máquina virtual tiene una dirección IP privada y un proceso de la máquina virtual intenta conectarse con una dirección IP pública. Hay dos soluciones alternativas para evitar la limitación de Azure SNAT:

* Agregue el punto de conexión de servicio de Azure Cosmos DB a la subred de la red virtual de Azure Virtual Machines. Para obtener más información, consulte [puntos de conexión de servicio de red virtual de Azure](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview). 

    Cuando se habilita el punto de conexión de servicio, las solicitudes ya no se envían desde una dirección IP pública a Azure Cosmos DB. En su lugar, se envían la red virtual y la identidad de la subred. Este cambio puede producir caídas de firewall si solo se permiten direcciones IP públicas. Si usa un firewall, cuando se habilite el punto de conexión de servicio, agregue una subred al firewall mediante las [ACL de Virtual Network](https://docs.microsoft.com/azure/virtual-network/virtual-networks-acl).
* Asignar una dirección IP pública a la máquina virtual de Azure.

##### <a name="cant-reach-the-service---firewall"></a><a name="cant-connect"></a>No se puede conectar con el servicio: firewall
``ConnectTimeoutException`` indica que el SDK no puede conectar con el servicio.
Es posible que obtenga un error similar al siguiente cuando use el modo directo:
```
GoneException{error=null, resourceAddress='https://cdb-ms-prod-westus-fd4.documents.azure.com:14940/apps/e41242a5-2d71-5acb-2e00-5e5f744b12de/services/d8aa21a5-340b-21d4-b1a2-4a5333e7ed8a/partitions/ed028254-b613-4c2a-bf3c-14bd5eb64500/replicas/131298754052060051p//', statusCode=410, message=Message: The requested resource is no longer available at the server., getCauseInfo=[class: class io.netty.channel.ConnectTimeoutException, message: connection timed out: cdb-ms-prod-westus-fd4.documents.azure.com/101.13.12.5:14940]
```

Si tiene un firewall que se ejecuta en el equipo de la aplicación, abra el intervalo de puertos de 10 000 a 20 000, que son los que usa el modo directo.
Consulte también las indicaciones de [Límite de conexiones en una máquina host](#connection-limit-on-host).

#### <a name="http-proxy"></a>Proxy HTTP

Si usa un Proxy HTTP, asegúrese de que pueda admitir el número de conexiones configuradas en el SDK de `ConnectionPolicy`.
En caso contrario, se encontrará con problemas de conexión.

#### <a name="invalid-coding-pattern-blocking-netty-io-thread"></a>Patrón de codificación no válido: bloqueo del subproceso de E/S de Netty

El SDK usa la biblioteca de E/S de [Netty](https://netty.io/) para comunicarse con Azure Cosmos DB. El SDK tiene una API asincrónica y usa las API de E/S de Netty sin bloqueo. El trabajo de E/S del SDK se realiza en subprocesos de E/S de Netty. El número de subprocesos de E/S de Netty está configurado para ser el mismo que el número de núcleos de CPU de la máquina de la aplicación. 

Los subprocesos de E/S de Netty solo están diseñados para usarse con trabajos de E/S de Netty sin bloqueos. El SDK devuelve el resultado de la invocación de la API en uno de los subprocesos de E/S de Netty al código de la aplicación. Si la aplicación realiza una operación de larga duración después de recibir los resultados en el subproceso de Netty, puede que el SDK no tenga suficientes subprocesos de E/S para realizar su trabajo de E/S interno. Este tipo de codificación de aplicación puede producir un rendimiento bajo, alta latencia y errores de `io.netty.handler.timeout.ReadTimeoutException`. La solución consiste en cambiar el subproceso cuando se sabe que la operación tarda tiempo.

Por ejemplo, eche un vistazo al siguiente fragmento de código, que agrega elementos a un contenedor (mire [aquí](create-sql-api-java.md) para obtener instrucciones sobre cómo configurar la base de datos y el contenedor). Puede realizar un trabajo de larga duración que tarda más de unos milisegundos en el subproceso de Netty. En ese caso, podrá entrar en un estado donde ningún subproceso de E/S de Netty está presente para procesar el trabajo de E/S. Como resultado, recibirá un error ReadTimeoutException.

### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-readtimeout"></a>API asincrónica del SDK para Java v4 (Maven com.azure::azure-cosmos)

[!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=TroubleshootNeedsSchedulerAsync)]

La solución consiste en cambiar el subproceso en el que realiza el trabajo que lleva tiempo. Defina una instancia singleton del programador para la aplicación.

### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-scheduler"></a>API asincrónica del SDK para Java v4 (Maven com.azure::azure-cosmos)

[!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=TroubleshootCustomSchedulerAsync)]

Es posible que necesite realizar trabajo que lleva tiempo, por ejemplo, trabajo computacionalmente intensivo o E/S de bloqueo. En este caso, cambie el subproceso a un trabajador proporcionado por el objeto `customScheduler` con la API `.publishOn(customScheduler)`.

### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-apply-custom-scheduler"></a>API asincrónica del SDK para Java v4 (Maven com.azure::azure-cosmos)

[!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=TroubleshootPublishOnSchedulerAsync)]

Mediante el uso de `publishOn(customScheduler)`, se libera el subproceso de E/S de Netty y se cambia a su propio subproceso personalizado proporcionado por el programador personalizado. Esta modificación resuelve el problema. No volverá a recibir un error `io.netty.handler.timeout.ReadTimeoutException`.

### <a name="request-rate-too-large"></a>Tasa de solicitudes demasiado grande
Se trata de un error de servidor. Indica que consumió el rendimiento aprovisionado. Vuelva a intentarlo más tarde. Si recibe este error con frecuencia, considere la posibilidad de aumentar el rendimiento de la colección.

* **Implementación del retroceso según intervalos de getRetryAfterInMilliseconds**

    Durante las pruebas de rendimiento, debe aumentar la carga hasta que se limite una tasa de solicitudes pequeña. Si se limita, la aplicación cliente debe retroceder de acuerdo con el intervalo de reintento que el servidor especificó. Respetar el retroceso garantiza que dedica una cantidad de tiempo mínima de espera entre reintentos.

### <a name="failure-connecting-to-azure-cosmos-db-emulator"></a>Error al conectarse al emulador de Azure Cosmos DB

El certificado HTTPS del emulador de Azure Cosmos DB es un certificado autofirmado. Para que SDK funcione con el emulador, importe el certificado del emulador a Java TrustStore. Para obtener más información, consulte [Exportación de los certificados del emulador de Azure Cosmos DB](local-emulator-export-ssl-certificates.md).

### <a name="dependency-conflict-issues"></a>Problemas de conflictos de dependencias

El SDK de Azure Cosmos DB para Java extrae varias dependencias; en general, si el árbol de dependencias del proyecto incluye una versión anterior de un artefacto del que depende el SDK de Azure Cosmos DB para Java, esto puede provocar errores inesperados al ejecutar la aplicación. Si va a depurar el motivo por el que la aplicación inicia inesperadamente una excepción, es una buena idea comprobar que el árbol de dependencias no extrae accidentalmente una versión anterior de una o varias de las dependencias del SDK de Azure Cosmos DB para Java.

La solución alternativa para este problema es identificar qué dependencias del proyecto vienen de la versión anterior, excluir la dependencia transitiva de esa versión anterior y permitir que el SDK de Azure Cosmos DB para Java incorpore la versión más reciente.

Para identificar qué dependencias del proyecto vienen de una versión anterior de algo de lo que depende el SDK de Azure Cosmos DB para Java, ejecute el siguiente comando en el archivo pom.xml del proyecto:
```bash
mvn dependency:tree
```
Para obtener más información, vea la [guía sobre el árbol de dependencia de Maven](https://maven.apache.org/plugins/maven-dependency-plugin/examples/resolving-conflicts-using-the-dependency-tree.html).

Una vez que sepa qué dependencia del proyecto depende de una versión anterior, puede modificar la dependencia en esa biblioteca en el archivo pom y excluir la dependencia transitiva, según el ejemplo siguiente (que presupone que *reactor-core* es la dependencia obsoleta):

```xml
<dependency>
  <groupId>${groupid-of-lib-which-brings-in-reactor}</groupId>
  <artifactId>${artifactId-of-lib-which-brings-in-reactor}</artifactId>
  <version>${version-of-lib-which-brings-in-reactor}</version>
  <exclusions>
    <exclusion>
      <groupId>io.projectreactor</groupId>
      <artifactId>reactor-core</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```

Para obtener más información, vea la [guía sobre cómo excluir dependencias transitivas](https://maven.apache.org/guides/introduction/introduction-to-optional-and-excludes-dependencies.html).


## <a name="enable-client-sdk-logging"></a><a name="enable-client-sice-logging"></a>Habilitar el registro del SDK de cliente

El SDK de Azure Cosmos DB para Java v4 usa SLF4j como fachada de registro, que admite el registro en plataformas de registro populares como log4j y logback.

Por ejemplo, si quiere usar log4j como plataforma de registro, agregue las siguientes bibliotecas en la classpath de Java.

```xml
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
  <version>${slf4j.version}</version>
</dependency>
<dependency>
  <groupId>log4j</groupId>
  <artifactId>log4j</artifactId>
  <version>${log4j.version}</version>
</dependency>
```

Agregue también una configuración de log4j.
```
# this is a sample log4j configuration

# Set root logger level to DEBUG and its only appender to A1.
log4j.rootLogger=INFO, A1

log4j.category.com.microsoft.azure.cosmosdb=DEBUG
#log4j.category.io.netty=INFO
#log4j.category.io.reactivex=INFO
# A1 is set to be a ConsoleAppender.
log4j.appender.A1=org.apache.log4j.ConsoleAppender

# A1 uses PatternLayout.
log4j.appender.A1.layout=org.apache.log4j.PatternLayout
log4j.appender.A1.layout.ConversionPattern=%d %5X{pid} [%t] %-5p %c - %m%n
```

Para obtener más información, revise el [registro manual de sfl4j](https://www.slf4j.org/manual.html).

## <a name="os-network-statistics"></a><a name="netstats"></a>Estadísticas de red de SO
Ejecute el comando netstat para hacerse una idea de cuántas conexiones se encuentran en el estado `ESTABLISHED` y en el estado `CLOSE_WAIT`.

En Linux, puede ejecutar el comando siguiente.
```bash
netstat -nap
```

En Windows, puede ejecutar el mismo comando con diferentes marcas de argumento:
```bash
netstat -abn
```

Filtrar el resultado para obtener solo las conexiones al punto de conexión de Azure Cosmos DB.

El número de conexiones al punto de conexión de Azure Cosmos DB en el estado `ESTABLISHED` no puede ser mayor que el tamaño del grupo de conexiones configurado.

Muchas de las conexiones al punto de conexión de Azure Cosmos DB podrían estar en el estado `CLOSE_WAIT`. Podría haber más de 1000. Un número tan alto como este indica que las conexiones se establecen y se cancelan rápidamente. Esta situación puede causar problemas. Para obtener más información, revise la sección [Problemas comunes y soluciones alternativas].

 <!--Anchors-->
[Problemas comunes y soluciones alternativas]: #common-issues-workarounds
[Enable client SDK logging]: #enable-client-sice-logging
[Límite de conexiones en una máquina host]: #connection-limit-on-host
[Agotamiento de puertos SNAT (PAT) de Azure]: #snat



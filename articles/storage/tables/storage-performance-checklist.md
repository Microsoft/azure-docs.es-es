---
title: Lista de comprobación de rendimiento y de escalabilidad para Table Storage (Azure Storage)
description: Lista de comprobación de procedimientos de eficacia probada que se usan en Table Storage al desarrollar aplicaciones de alto rendimiento.
services: storage
author: tamram
ms.author: tamram
ms.service: storage
ms.topic: overview
ms.date: 10/10/2019
ms.subservice: tables
ms.custom: devx-track-csharp
ms.openlocfilehash: 9d3f7d5f496634f10b48e7509c21cd634fd92d3c
ms.sourcegitcommit: 4a7a4af09f881f38fcb4875d89881e4b808b369b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89458339"
---
# <a name="performance-and-scalability-checklist-for-table-storage"></a>Lista de comprobación de rendimiento y de escalabilidad para Table Storage

Microsoft ha desarrollado varios procedimientos de eficacia probada para desarrollar aplicaciones de alto rendimiento con Table Storage. Esta lista de comprobación identifica los procedimientos clave que pueden seguir los desarrolladores para optimizar su rendimiento. Tenga presente estos procedimientos tanto para diseñar su aplicación como a lo largo de todo el proceso.

Azure Storage tiene objetivos de escalabilidad y rendimiento en lo que se refiere a capacidad, tasa de transacciones y ancho de banda. Para más información sobre los objetivos de escalabilidad de Azure Storage, consulte [Objetivos de escalabilidad y rendimiento para cuentas de almacenamiento estándar](../common/scalability-targets-standard-account.md?toc=%2fazure%2fstorage%2ftables%2ftoc.json) y [Objetivos de escalabilidad y rendimiento de Table Storage](scalability-targets.md).

## <a name="checklist"></a>Lista de comprobación

En este artículo se organizan los procedimientos de eficacia probada a la hora de mejorar el rendimiento en una lista de comprobación que puede seguir al desarrollar su aplicación en Table Storage.

| ¡Listo! | Category | Consideraciones acerca del diseño |
| --- | --- | --- |
| &nbsp; |Objetivos de escalabilidad |[¿Puede diseñar la aplicación para que no supere el número máximo de cuentas de almacenamiento?](#maximum-number-of-storage-accounts) |
| &nbsp; |Objetivos de escalabilidad |[¿Evita acercarse a los límites de transacción y capacidad?](#capacity-and-transaction-targets) |
| &nbsp; |Objetivos de escalabilidad |[¿Se aproxima a los objetivos de escalabilidad para entidades por segundo?](#targets-for-data-operations) |
| &nbsp; |Redes |[¿Disponen los dispositivos del cliente de un ancho de banda suficientemente grande y una latencia suficientemente baja como para lograr el rendimiento necesario?](#throughput) |
| &nbsp; |Redes |[¿Tienen los dispositivos del cliente un enlace de red de alta calidad?](#link-quality) |
| &nbsp; |Redes |[¿Está la aplicación cliente en la misma región que la cuenta de almacenamiento?](#location) |
| &nbsp; |Acceso directo del cliente |[¿Utiliza firmas de acceso compartido (SAS) y el uso compartido de recursos entre orígenes (CORS) para habilitar el acceso directo a Azure Storage?](#sas-and-cors) |
| &nbsp; |Lotes |[¿La aplicación procesa actualizaciones por lotes mediante transacciones del grupo de entidades?](#batch-transactions) |
| &nbsp; |Configuración de .NET |[¿Usa .NET Core 2.1, o cualquier versión posterior, para lograr un rendimiento óptimo?](#use-net-core) |
| &nbsp; |Configuración de .NET |[¿Ha configurado el cliente para usar un número suficiente de conexiones simultáneas?](#increase-default-connection-limit) |
| &nbsp; |Configuración de .NET |[En el caso de las aplicaciones .NET, ¿ha configurado .NET para que use un número suficiente de subprocesos?](#increase-minimum-number-of-threads) |
| &nbsp; |Paralelismo |[¿Se ha asegurado de que el paralelismo está enlazado correctamente para no sobrecargar las funcionalidades del cliente ni acercarse a los objetivos de escalabilidad?](#unbounded-parallelism) |
| &nbsp; |Herramientas |[¿Usa las versiones más recientes de las herramientas y bibliotecas de cliente que proporciona Microsoft?](#client-libraries-and-tools) |
| &nbsp; |Reintentos |[¿Usa una directiva de reintentos con un retroceso exponencial para los errores de limitación y tiempos de espera?](#timeout-and-server-busy-errors) |
| &nbsp; |Reintentos |[¿Evita su aplicación reintentos para errores que no se pueden reintentar?](#non-retryable-errors) |
| &nbsp; |Configuración |[¿Usa JSON para sus solicitudes de tabla?](#use-json) |
| &nbsp; |Configuración |[¿Ha desactivado el algoritmo de Nagle para mejorar el rendimiento de las solicitudes pequeñas?](#disable-nagle) |
| &nbsp; |Tablas y particiones |[¿Ha particionado sus datos correctamente?](#schema) |
| &nbsp; |Particiones calientes |[¿Evita los patrones Solo anexar y Solo anteponer?](#append-only-and-prepend-only-patterns) |
| &nbsp; |Particiones calientes |[¿Se despliegan sus inserciones y actualizaciones por muchas particiones?](#high-traffic-data) |
| &nbsp; |Ámbito de la consulta |[¿Ha diseñado su esquema para permitir el uso de consultas puntuales en la mayoría de los casos y el uso de consultas de tabla con moderación?](#query-scope) |
| &nbsp; |Densidad de la consulta |[¿Normalmente sus consultas solamente examinan y devuelven filas que usará su aplicación?](#query-density) |
| &nbsp; |Limitación de los datos devueltos |[¿Usa filtros para evitar la devolución de entidades que no se necesitan?](#limiting-the-amount-of-data-returned) |
| &nbsp; |Limitación de los datos devueltos |[¿Usa proyección para evitar la devolución de propiedades que no se necesitan?](#limiting-the-amount-of-data-returned) |
| &nbsp; |Desnormalización |[¿Ha desnormalizado los datos de forma que evite consultas ineficientes o varias solicitudes de lectura cuando intenta obtener datos?](#denormalization) |
| &nbsp; |Inserción, actualización y eliminación |[¿Procesa por lotes solicitudes que necesitan ser transaccionales o se pueden realizar al mismo tiempo para reducir recorridos de ida y vuelta?](#batching) |
| &nbsp; |Inserción, actualización y eliminación |[¿Evita la recuperación de una entidad simplemente para determinar si llama a la inserción o a la actualización?](#upsert) |
| &nbsp; |Inserción, actualización y eliminación |[¿Ha pensado en almacenar series de datos que se recuperarán frecuentemente de forma conjunta en una sola entidad como propiedades en lugar de varias entidades?](#storing-data-series-in-a-single-entity) |
| &nbsp; |Inserción, actualización y eliminación |[Para entidades que siempre se recuperarán conjuntamente y que se pueden escribir en lotes (por ejemplo, datos de serie de temporales), ¿ha pensado en usar blobs en lugar de tablas?](#storing-structured-data-in-blobs) |

## <a name="scalability-targets"></a>Objetivos de escalabilidad

Si su aplicación se aproxima o supera cualquiera de estos objetivos de escalabilidad, puede encontrar un aumento en la limitación o latencias de transacción. Cuando Azure Storage limita su aplicación, el servicio comienza a devolver códigos de error 503 (Servidor ocupado) o 500 (Tiempo de espera de la operación). Evitar estos errores, mediante la permanencia en los límites de los objetivos de escalabilidad, es una parte importante de la mejora del rendimiento de la aplicación.

Para más información acerca de los objetivos de escalabilidad de Table service, consulte [Objetivos de escalabilidad y rendimiento para Table Storage](scalability-targets.md).

### <a name="maximum-number-of-storage-accounts"></a>Número máximo de cuentas de almacenamiento

Si se aproxima al número máximo de cuentas de almacenamiento permitido para una combinación de suscripción/región concreta, ¿usa varias cuentas de almacenamiento para la partición para aumentar la entrada, la salida, las operaciones de E/S por segundo o la capacidad? En este escenario, Microsoft recomienda aprovechar las ventajas del aumento de los límites de las cuentas de almacenamiento para reducir el número de cuentas de almacenamiento necesarias para la carga de trabajo, siempre que sea posible. Póngase en contacto con el [soporte técnico de Azure](https://azure.microsoft.com/support/options/) para solicitar el aumento de los límites de la cuenta de almacenamiento. Para más información, consulte [Announcing larger, higher scale storage accounts](https://azure.microsoft.com/blog/announcing-larger-higher-scale-storage-accounts/) (Anuncio de cuentas de almacenamiento más grandes y con mayor escala).

### <a name="capacity-and-transaction-targets"></a>Objetivos de capacidad y transacción

Si su aplicación se aproxima a los objetivos de escalabilidad para una sola cuenta de almacenamiento, plantéese la adopción de uno de los siguientes enfoques:  

- Reconsidere la carga de trabajo que hace que su aplicación se aproxime al objetivo de escalabilidad o lo supere. ¿Puede designarla de forma diferente para que use menos ancho de banda o capacidad, o menos transacciones?
- Si su aplicación debe superar uno de los objetivos de escalabilidad, cree varias cuentas de almacenamiento y realice particiones de los datos de su aplicación entre esas cuentas. Si usa este patrón, entonces debe asegurarse de designar la aplicación de forma que pueda agregar más cuentas de almacenamiento en el futuro para equilibrio de carga. Las propias cuentas de almacenamiento no tienen ningún costo aparte del de su uso en términos de datos almacenados, transacciones realizadas o datos transferidos.
- Si la aplicación se aproxima a los objetivos de ancho de banda, considere la posibilidad de comprimir los datos en el cliente para reducir el ancho de banda necesario para enviar los datos a Azure Storage.
    Aunque comprimir datos puede ahorrar ancho de banda y mejorar el rendimiento de la red, también puede tener efectos negativos en el rendimiento. Evalúe el impacto en el rendimiento de los requisitos de procesamiento adicionales para la compresión y descompresión de datos en el cliente. Tenga en cuenta que el almacenamiento de los datos comprimidos puede dificultar la solución de problemas, ya que es muy probable que sea más complicado ver los datos con herramientas estándar.
- Si la aplicación se acerca a los objetivos de escalabilidad, asegúrese de que usa un retroceso exponencial para los reintentos. Es recomendable que intente evitar llegar a los objetivos de escalabilidad mediante la implementación de las recomendaciones que se describen en este artículo. Sin embargo, el uso de un retroceso exponencial para los reintentos impedirá que la aplicación vuelva a intentarlo rápidamente, lo que podría empeorar la limitación. Para más información, consulte la sección [Errores de tiempo de expiración y servidor ocupado](#timeout-and-server-busy-errors).

### <a name="targets-for-data-operations"></a>Destinos para operaciones de datos

Azure Storage equilibra la carga a medida que aumenta el tráfico hacia la cuenta de almacenamiento, pero si el tráfico exhibe ráfagas repentinas, es posible que no pueda obtener este volumen de rendimiento de inmediato. Espere ver el límite o los tiempos de espera durante la ráfaga ya que Azure Storage equilibra automáticamente la carga de la tabla. El aumento lento generalmente proporciona mejores resultados, ya que el sistema tiene tiempo para equilibrar la carga apropiadamente.

#### <a name="entities-per-second-storage-account"></a>Entidades por segundo (cuenta de almacenamiento)

El límite de escalabilidad para el acceso a tablas es de hasta 20 000 unidades (1 KB cada una) por segundo para una cuenta. En general, cada entidad que se inserta, actualiza, elimina o examina, cuenta para este objetivo. Por tanto, una inserción por lotes que contiene 100 entidades contaría como 100 entidades. Una consulta que examina 1.000 y devuelve 5, contaría como 1.000 entidades.

#### <a name="entities-per-second-partition"></a>Entidades por segundo (partición)

Dentro de una sola partición, el objetivo de escalabilidad para obtener acceso a las tablas es de 2000 entidades (1 KB cada una) por segundo, usando el mismo recuento descrito en la sección anterior.

## <a name="networking"></a>Redes

Las restricciones de red físicas de la aplicación pueden tener un impacto considerable en el rendimiento. En las siguientes secciones se describen algunas de las limitaciones que pueden encontrar los usuarios.  

### <a name="client-network-capability"></a>Capacidad de red del cliente

El ancho de banda y la calidad del vínculo de red juegan roles importantes en el rendimiento de la aplicación, como se describe en las secciones siguientes.

#### <a name="throughput"></a>Throughput

Para el ancho de banda, el problema suele residir en las capacidades del cliente. Las instancias de Azure mayores tienen NIC con mayor capacidad, por lo que debe plantearse la posibilidad de usar una instancia mayor o más máquinas virtuales si necesita aumentar los límites de red de una sola máquina. Si accede a Azure Storage desde una aplicación local, se aplica la misma regla: conozca las funcionalidades de red del dispositivo cliente y la conectividad de red con la ubicación de Azure Storage y mejórelas según sea necesario o diseñe la aplicación para que funcione dentro de sus funcionalidades.

#### <a name="link-quality"></a>Calidad del vínculo

Como siempre que se usa la red, tenga en cuenta que las condiciones de la red generan errores y la pérdida de paquetes reducirá el rendimiento efectivo.  El uso de WireShark o NetMon puede ayudar a diagnosticar este problema.  

### <a name="location"></a>Location

En cualquier entorno distribuido, la ubicación del cliente cerca del servidor ofrece el mejor rendimiento. Para acceder a Azure Storage con la mínima latencia, la mejor ubicación para el cliente es dentro de la misma región de Azure. Por ejemplo, si tiene una aplicación web de Azure que usa Azure Storage, coloque ambas en una sola región, como Oeste de EE. UU. o Sudeste de Asia. La colocalización reduce la latencia y el costo, ya que el uso del ancho de banda dentro de una sola región es gratuito.  

Si las aplicaciones cliente acceden a Azure Storage, pero no están hospedadas en Azure, como por ejemplo las aplicaciones de dispositivos móviles o los servicios empresariales locales, la colocación de la cuenta de almacenamiento en una región próxima a los clientes puede reducir la latencia. Si los clientes están distribuidos en sitios distantes (por ejemplo, algunos en Norteamérica y otros en Europa), considere la posibilidad de usar una cuenta de almacenamiento por región. Este enfoque es más sencillo de implementar si los datos que almacena la aplicación son específicos de usuarios individuales y no se requiere la replicación de datos entre cuentas de almacenamiento.

## <a name="sas-and-cors"></a>SAS y CORS

Supongamos que necesita autorizar código, como JavaScript, que se ejecuta en el explorador web de un usuario o en una aplicación de teléfono móvil para acceder a los datos de Azure Storage. Un enfoque consiste en crear una aplicación de servicio que actúe como proxy. El dispositivo del usuario se autentica con el servicio, que a su vez autoriza el acceso a los recursos de Azure Storage. De esta forma, puede evitar la exposición de sus claves de cuenta de almacenamiento en dispositivos no seguros. Sin embargo, este enfoque supone una importante sobrecarga para la aplicación de servicio, ya que todos los datos transferidos entre el dispositivo del usuario y Azure Storage deben pasar por la aplicación de servicio.

El uso de firmas de acceso compartido (SAS) le permite evitar el uso de una aplicación de servicio como proxy para Azure Storage. Mediante SAS se puede permitir que el dispositivo del usuario realice solicitudes directamente en Azure Storage mediante un token de acceso limitado. Por ejemplo, si un usuario desea cargar una foto en la aplicación, la aplicación de servicio puede generar una SAS y enviarla al dispositivo del usuario. El token de SAS puede conceder permiso para escribir en un recurso de Azure Storage durante un intervalo de tiempo especificado, después del cual expirará el token. Para obtener más información sobre las firmas de acceso compartido, consulte [Otorgar acceso limitado a recursos de Azure Storage con firmas de acceso compartido (SAS)](../common/storage-sas-overview.md).  

Por lo general, los exploradores web no permiten que haya código JavaScript en una página hospedada por un sitio web de un dominio que realiza operaciones concretas, como operaciones de escritura, en otro dominio. Esta directiva, que se conoce como directiva de mismo origen, impide que un script malintencionado de una página acceda a los datos de otra página web. Sin embargo, la directiva de mismo origen puede ser una limitación al compilar una solución en la nube. El uso compartido de recursos entre orígenes (CORS) es una característica del explorador que permite al dominio de destino comunicarle al explorador en el que confía solicitudes que se originan en el dominio de origen.

Por ejemplo, supongamos que una aplicación web que se ejecuta en Azure realiza una solicitud de un recurso en una cuenta de Azure Storage. La aplicación web es el dominio de origen y la cuenta de almacenamiento es el dominio de destino. Puede configurar CORS para que cualquiera de los servicios de Azure Storage comunique al explorador web que Azure Storage confía en las solicitudes del dominio de origen. Para más información sobre CORS, consulte [Soporte técnico del uso compartido de recursos entre orígenes (CORS) para Azure Storage](/rest/api/storageservices/Cross-Origin-Resource-Sharing--CORS--Support-for-the-Azure-Storage-Services).  
  
Tanto SAS como CORS pueden evitar una carga innecesaria en la aplicación web.  

## <a name="batch-transactions"></a>Transacciones por lotes

Table service admite transacciones por lotes en entidades que están en la misma tabla y pertenecen al mismo grupo de particiones. Para más información, consulte [Realización de transacciones de grupos de entidades](/rest/api/storageservices/performing-entity-group-transactions).

## <a name="net-configuration"></a>Configuración de .NET

Si usa .NET Framework, esta sección enumera varias configuraciones rápidas que puede usar para realizar mejoras de rendimiento significativas.  Si usa otros lenguajes, compruebe si se aplican conceptos similares en el lenguaje elegido.  

### <a name="use-net-core"></a>Uso de .NET Core

Desarrolle las aplicaciones de Azure Storage con .NET Core 2.1, o cualquier versión posterior, para aprovechar las mejoras en el rendimiento. Se recomienda usar .NET Core 3.x siempre que sea posible.

Para más información sobre las mejoras en el rendimiento en .NET Core, consulte las siguientes entradas de blog:

- [Mejoras de rendimiento en .NET Core 3.0](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-core-3-0/)
- [Mejoras de rendimiento en .NET Core 2.1](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-core-2-1/)

### <a name="increase-default-connection-limit"></a>Aumento del límite de conexiones predeterminado

En .NET, el siguiente código aumenta el límite de conexiones predeterminado (que normalmente es 2 en un entorno cliente o 10 en un entorno servidor) a 100. Normalmente, debe establecer el valor en aproximadamente el número de subprocesos usados por su aplicación.  

```csharp
ServicePointManager.DefaultConnectionLimit = 100; //(Or More)  
```

Establezca el límite de conexiones antes de abrir cualquier conexión.  

En el caso de otros lenguajes de programación, consulte la documentación del lenguaje en cuestión para determinar cómo establecer el límite de conexiones.  

Para obtener más información, vea la entrada de blog [Servicios web: conexiones simultáneas](https://blogs.msdn.microsoft.com/darrenj/2005/03/07/web-services-concurrent-connections/).  

### <a name="increase-minimum-number-of-threads"></a>Aumento del número mínimo de subprocesos

Si utiliza llamadas sincrónicas junto con tareas asincrónicas, puede que desee aumentar el número de subprocesos en el grupo de subprocesos:

```csharp
ThreadPool.SetMinThreads(100,100); //(Determine the right number for your application)  
```

Para más información, consulte el método [ThreadPool.SetMinThreads](/dotnet/api/system.threading.threadpool.setminthreads).  

## <a name="unbounded-parallelism"></a>Paralelismo sin enlazar

Aunque el paralelismo puede ser excelente para el rendimiento, tenga cuidado al usar el paralelismo sin enlazar, lo que significa que no se aplica ningún límite en cuanto al número de subprocesos o solicitudes paralelas. Asegúrese de limitar las solicitudes paralelas para cargar o descargar datos, para acceder a varias particiones de la misma cuenta de almacenamiento o para acceder a varios elementos de la misma partición. Si el paralelismo está sin enlazar, la aplicación puede superar las funcionalidades del dispositivo cliente o los objetivos de escalabilidad de la cuenta de almacenamiento, lo que puede provocar latencias y limitaciones mayores.  

## <a name="client-libraries-and-tools"></a>Herramientas y bibliotecas cliente

Para obtener el mejor rendimiento posible, utilice siempre las herramientas y bibliotecas de cliente más recientes que proporciona Microsoft. Las bibliotecas de cliente de Azure Storage están disponibles para varios lenguajes. Azure Storage también admite PowerShell y la CLI de Azure. Microsoft desarrolla de forma activa estas herramientas y bibliotecas de cliente pensando en el rendimiento, las mantiene actualizadas con las versiones de servicio más recientes y se asegura de que controlan internamente muchos de los procedimientos de rendimiento de eficacia probada. Para más información, consulte la [documentación de referencia de Azure Storage](/azure/storage/#reference).

## <a name="handle-service-errors"></a>Control de errores del servicio

Azure Storage devuelve un error cuando el servicio no puede procesar una solicitud. Conocer los errores que puede devolver Azure Storage en un escenario dado resulta útil para optimizar el rendimiento.

### <a name="timeout-and-server-busy-errors"></a>Errores debidos al tiempo de expiración y a que el servidor está ocupado

Azure Storage puede limitar la aplicación si se aproxima a los límites de escalabilidad. En algunos casos, es posible que Azure Storage no pueda controlar una solicitud debido a alguna condición transitoria. En ambos casos, el servicio puede devolver un error 503 (servidor ocupado) o 500 (tiempo de expiración). Estos errores también pueden producirse si el servicio reequilibra las particiones de datos para permitir un mayor rendimiento. Normalmente, la aplicación cliente debería reintentar la operación que provoca cualquiera de estos errores. Sin embargo, si Azure Storage está limitando la aplicación porque está superando los objetivos de escalabilidad, o incluso si el servicio no ha podido atender a la solicitud por alguna otra razón, los intentos agresivos pueden empeorar el problema. Se recomienda usar una directiva de reintentos de retroceso exponencial y las bibliotecas cliente se comportan así de forma predeterminada. Por ejemplo, su aplicación puede llevar a cabo los reintentos después de 2 segundos, luego 4 segundos, después 10 segundos, luego 30 segundos y, después, renunciar. De esta forma, la aplicación reduce considerablemente su carga en el servicio, en lugar de exagerar el comportamiento que puede provocar la limitación.  

Los errores de conectividad se pueden reintentar inmediatamente porque no se derivan de la limitación y se espera que sean transitorios.  

### <a name="non-retryable-errors"></a>Errores que no se pueden reintentar

Las bibliotecas cliente controlan los reintentos y saben qué errores se pueden reintentar y cuáles no. Sin embargo, si va a llamar directamente a la API REST de Azure Storage, hay algunos errores que no debe reintentar. Por ejemplo, un error 400 (solicitud incorrecta) indica que la aplicación cliente ha enviado una solicitud que no se pudo procesar porque no tenía el formato esperado. El reenvío de esta solicitud genera la misma respuesta siempre, por lo que no tiene sentido reintentarla. Si llama directamente a la API REST de Azure Storage, tenga en cuenta los posibles errores y si deben reintentarse.

Para más información sobre los códigos de error de Azure Storage, consulte [Estado y códigos de error](/rest/api/storageservices/status-and-error-codes2).

## <a name="configuration"></a>Configuración

En esta sección se enumeran varias configuraciones rápidas que puede usar para realizar mejoras de rendimiento significativas en Table service:

### <a name="use-json"></a>Uso de JSON

A partir de la versión del 15 de agosto de 2013 del servicio Storage, Table service admite el uso de JSON en lugar del formato AtomPub basado en XML para transferir datos de las tablas. El uso de JSON puede reducir los tamaños de carga hasta en un 75 % y puede mejorar significativamente el rendimiento de la aplicación.

Para más información, consulte la publicación [Tablas de Microsoft Azure: introducción a JSON](https://docs.microsoft.com/archive/blogs/windowsazurestorage/windows-azure-tables-introducing-json) y [Payload Format for Table Service Operations](https://msdn.microsoft.com/library/azure/dn535600.aspx) (Formato de carga para las operaciones de Table service).

### <a name="disable-nagle"></a>Deshabilitación de Nagle

El algoritmo de Nagle está ampliamente implementado en redes TCP/IP como medio de mejorar el rendimiento de la red. Sin embargo, no es óptimo en todas las situaciones (como por ejemplo en entornos altamente interactivos). El algoritmo de Nagle tiene un efecto negativo en el rendimiento de las solicitudes en Azure Table service y debe deshabilitarlo si es posible.

## <a name="schema"></a>Schema

La forma de representar los datos y realizar consultas es el factor más importante que afecta al rendimiento de Table service. Aunque cada aplicación es diferente, en esta sección se describen algunas prácticas probadas generales relacionadas con:

- Diseño de tablas
- Consultas eficientes
- Actualizaciones de datos eficientes

### <a name="tables-and-partitions"></a>Tablas y particiones

Las tablas se dividen en dos particiones. Cada entidad almacenada en una partición comparte la misma clave de partición y tiene una clave de fila única para identificarla dentro de esa partición. Las particiones proporcionan ventajas pero también presentan limitaciones de escalabilidad.

- Ventajas: Puede actualizar entidades de la misma partición en una sola transacción por lotes atómica que contenga hasta 100 operaciones de almacenamiento independientes (límite de tamaño total de 4 MB). Suponiendo que se recupera el mismo número de entidades, también puede consultar datos dentro de una sola partición más eficientemente que los datos que se extienden por particiones (siga leyendo para conocer más recomendaciones sobre la consulta de datos de tabla).
- Limite de escalabilidad: no se puede realizar el equilibrio de carga en el acceso a entidades almacenadas en una sola partición porque las particiones admiten transacciones por lotes atómicas. Por esta razón, el objetivo de escalabilidad de una partición de tabla individual es inferior al de Table service en conjunto.

Debido a estas características de tablas y particiones, debe adoptar los siguientes principios de diseño:

- Busque los datos que su aplicación cliente actualiza o consulta con frecuencia en la misma unidad lógica de trabajo de la misma partición. Por ejemplo, busque los datos en la misma partición si la aplicación agrega escrituras o realiza operaciones por lotes atómicas. Asimismo, los datos de una sola partición pueden ser consultados de forma más eficiente en una sola consulta que los datos que se encuentran repartidos por particiones.
- Busque los datos que la aplicación cliente no inserta, actualiza o consulta en la misma unidad lógica de trabajo (es decir, en una consulta o una actualización por lotes) en particiones independientes. Tenga en cuenta que no hay límite en el número de claves de partición que contiene una sola tabla, por lo que tener millones de claves de partición no es un problema y no afectará al rendimiento. Por ejemplo, si la aplicación es un sitio web conocido con inicio de sesión de usuario, el uso del identificador de usuario como clave de partición podría ser una buena elección.

#### <a name="hot-partitions"></a>Particiones calientes

Una partición caliente es aquella que recibe un porcentaje desproporcionado de tráfico en una cuenta y no se puede realizar un equilibrio de carga en ella porque es una sola partición. En general, las particiones calientes se crean de una de dos formas:

#### <a name="append-only-and-prepend-only-patterns"></a>Patrones Solo anexar y Solo anteponer

El patrón "Solo anexar" es aquel en el que todo (o casi todo) el tráfico que llega a una clave de partición dada aumenta y disminuye en función de la hora. Por ejemplo, suponga que su aplicación usa la fecha actual como clave de partición para los datos de registro. El resultado de este diseño es que todas las inserciones van a la última partición de la tabla y el sistema no puede equilibrar la carga correctamente. Si el volumen del tráfico en esa partición supera el objetivo de escalabilidad de nivel de partición, dará lugar a una limitación. Es mejor garantizar que el tráfico se envía a varias particiones para habilitar el equilibrio de carga de las solicitudes en la tabla.

#### <a name="high-traffic-data"></a>Datos con mucho tráfico

Si el esquema de participaciones da lugar a una sola partición que solamente tiene datos y que se utiliza mucho más que otras particiones, también puede ver limitaciones cuando esa partición se aproxime al objetivo de escalabilidad para una sola partición. Conviene asegurarse de que el esquema de partición no provoca que las particiones individuales se aproximen a los objetivos de escalabilidad.

### <a name="querying"></a>Consultas

En esta sección se describen procedimientos de eficacia probada para realizar consultas en Table service.

#### <a name="query-scope"></a>Ámbito de la consulta

Hay varias formas de especificar el intervalo de entidades para consultar. En la siguiente lista se describe cada una de las opciones del ámbito de la consulta.

- **Consultas de punto:** las consultas de punto recuperan exactamente una entidad mediante la especificación tanto de la clave de la partición como de la clave de la fila de la entidad que se va a recuperar. Estas consultas son eficientes y debe usarlas siempre que sea posible.
- **Consultas de partición:** Una consulta de partición es una consulta que recupera un conjunto de datos que comparte una clave de partición común. Normalmente, la consulta especifica un intervalo de valores de clave de fila o un intervalo de valores para alguna propiedad de entidad además de una clave de partición. Estas consultas son menos eficientes que las consultas de punto y se deben usar con moderación.
- **Consultas de tabla:** Una consulta de tabla es una consulta que recupera un conjunto de entidades que no comparte una clave de partición común. Estas consultas no son diferentes y debe evitarlas siempre que sea posible.

En general, evite las exploraciones (consultas mayores que una sola entidad), pero si debe realizarlas, intente organizar los datos de forma que dichas exploraciones recuperen los datos necesarios sin explorar o devolver grandes cantidades de entidades innecesarias.

#### <a name="query-density"></a>Densidad de la consulta

Otro factor clave en la eficiencia de las consultas es el número de entidades devueltas comparado con el número de entidades examinadas para encontrar el conjunto devuelto. Si la aplicación realiza una consulta de tabla con un filtro para un valor de propiedad que solamente comparte el 1 % de los datos, la consulta examinará 100 entidades por cada entidad que devuelva. Todos los objetivos de escalabilidad de tabla tratados anteriormente están relacionados con el número de entidades que se analizan, no con el número de entidades que se devuelven: una densidad de consulta baja fácilmente puede provocar que Table service limite la aplicación porque debe analizar muchas entidades para recuperar la entidad que está buscando. Para más información acerca de cómo evitar la limitación, consulte la sección [Desnormalización](#denormalization).

#### <a name="limiting-the-amount-of-data-returned"></a>Limitación de la cantidad de datos devueltos

Cuando sepa que una consulta va a devolver entidades que no necesita en la aplicación cliente, plantéese el uso de un filtro para reducir el tamaño del conjunto devuelto. Aunque las entidades no devueltas al cliente siguen contando para los límites de escalabilidad, el rendimiento de la aplicación mejorará debido a la reducción del tamaño de la carga de red y a la reducción del número de entidades que la aplicación cliente debe procesar. Tenga en cuenta que los objetivos de escalabilidad están relacionados con el número de entidades examinadas, por lo que una consulta que filtre muchas entidades puede seguir dando lugar a limitaciones aunque se devuelvan pocas entidades. Para más información sobre cómo mejorar la eficacia de las consultas, consulte la sección [Densidad de consulta](#query-density).

Si la aplicación cliente solamente necesita un conjunto limitado de propiedades de las entidades de la tabla, puede usar proyección para limitar el tamaño del conjunto de datos devuelto. Como en el caso de los filtros, una proyección ayuda a reducir la carga de la red y el procesamiento del cliente.

#### <a name="denormalization"></a>Desnormalización

A diferencia del trabajo con bases de datos relacionales, las prácticas probadas para consultar datos de tabla de forma eficiente conducen a la desnormalización de los datos. Es decir, duplicar los mismos datos en varias entidades (una por cada clave que puede usar para encontrar los datos) para minimizar el número de entidades que una consulta debe examinar para encontrar los datos que el cliente necesita, en lugar de tener que examinar grandes números de entidades para encontrar los datos que la aplicación necesita. Por ejemplo, en un sitio web de comercio electrónico, puede que le interese encontrar un pedido tanto por el identificador del cliente (ver los pedidos de este cliente) como por la fecha (ver los pedidos con esa fecha). En Table Storage, es mejor almacenar la entidad (o una referencia a ella) dos veces, una vez con el nombre de tabla, la clave de partición y la clave de fila para facilitar la búsqueda por identificador de cliente y una vez para facilitar su búsqueda por la fecha.  

### <a name="insert-update-and-delete"></a>Inserción, actualización y eliminación

En esta sección se describen prácticas de eficacia probada para modificar entidades almacenadas en Table service.  

#### <a name="batching"></a>Lotes

Las transacciones por lotes se conocen como transacciones de grupos de entidades en Azure Storage. Todas las operaciones de una transacción de grupos de entidades deben estar en una sola partición de una sola tabla. Siempre que sea posible, use las transacciones de grupos de entidades para realizar inserciones, actualizaciones y eliminaciones por lotes. El uso de transacciones de grupos de entidades reduce el número de recorridos de ida y vuelta de la aplicación cliente al servidor, disminuye el número de transacciones facturables (una transacción de grupos de entidades cuenta como una sola transacción desde el punto de vista de facturación y puede contener hasta 100 operaciones de almacenamiento) y permite actualizaciones atómicas (todas las operaciones se realizan correctamente o en todas ellas hay errores dentro de una transacción de grupos de entidades). Los entornos con altas latencias, como por ejemplo dispositivos móviles, sacarán un enorme provecho al uso de transacciones de grupos de entidades.  

#### <a name="upsert"></a>Upsert

Use operaciones **Upsert** de tabla siempre que sea posible. Hay dos tipos de operaciones **Upsert**; ambos pueden ser más eficientes que las operaciones **Insert** y **Update** tradicionales:  

- **InsertOrMerge**: Use esta operación cuando quiera cargar un subconjunto de propiedades de la entidad, pero no está seguro de si la entidad ya existe. Si la entidad ya existe, esta llamada actualiza las propiedades incluidas en la operación **Upsert** y deja todas las propiedades existentes tal y como están; si la entidad no existe, inserta la nueva entidad. Esto es similar a usar proyección en una consulta, donde solamente necesita actualizar las propiedades que cambian.
- **InsertOrReplace**: Use esta operación cuando quiera cargar una entidad completamente nueva, pero no está seguro de si ya existe. Use esta operación cuando sepa que la entidad recién cargada es totalmente correcta porque sobrescribe completamente la entidad antigua. Por ejemplo, desea actualizar la entidad que almacena la ubicación actual de un usuario independientemente de si la aplicación ha almacenado los datos de ubicación del usuario anteriormente; la entidad de la nueva ubicación está completa y no necesita información de ninguna otra entidad anterior.

#### <a name="storing-data-series-in-a-single-entity"></a>Almacenamiento de series de datos en una sola entidad

A veces, una aplicación almacena una serie de datos que necesita recuperar de una sola vez con frecuencia: por ejemplo, un aplicación podría hacer un seguimiento del uso de CPU a lo largo del tiempo para representar un gráfico dinámico de los datos de las últimas 24 horas. Un enfoque es tener una entidad de tabla por hora, de forma que cada entidad represente una hora específica y almacene el uso de CPU para esa hora. Para representar estos datos, la aplicación necesita recuperar las entidades que contienen los datos de las 24 horas más recientes.  

Como alternativa, la aplicación podría almacenar el uso de CPU por cada hora como propiedad independiente de una sola entidad: para actualizar cada hora, la aplicación puede usar una sola llamada **InsertOrMerge Upsert** para actualizar el valor de la hora más reciente. Para representar los datos, la aplicación solamente necesita recuperar una entidad, no 24, lo que hace que la consulta sea eficiente. Para más información sobre la eficacia de las consultas, consulte la sección [Ámbito de la consulta](#query-scope)).

#### <a name="storing-structured-data-in-blobs"></a>Almacenamiento de datos estructurados en blobs

Si va a realizar inserciones por lotes y, después, a recuperar intervalos de entidades de forma conjunta, considere la posibilidad de usar blobs en lugar de tablas. Un buen ejemplo es un archivo de registro. Puede procesar por lotes varios minutos de registros, insertarlos y, después, recuperar varios minutos de registros simultáneamente. En ese caso, el rendimiento mejora si se usan blobs en lugar de tablas, ya que se puede reducir considerablemente el número de objetos en los que se escribe o que se leen, y posiblemente el número de solicitudes que hay que realizar.  

## <a name="next-steps"></a>Pasos siguientes

- [Objetivos de escalabilidad y rendimiento para Table Storage](scalability-targets.md)
- [Destinos de escalabilidad y rendimiento para cuentas de almacenamiento estándar](../common/scalability-targets-standard-account.md?toc=%2fazure%2fstorage%2ftables%2ftoc.json)
- [Estado y códigos de error](/rest/api/storageservices/Status-and-Error-Codes2)

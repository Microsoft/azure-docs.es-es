---
title: Lista de comprobación de escalabilidad y rendimiento para Blob Storage (Azure Storage)
description: Lista de comprobación de procedimientos de eficacia probada que se usan con Blob Storage al desarrollar aplicaciones de alto rendimiento.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 10/10/2019
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-csharp
ms.openlocfilehash: 85701f3d073b8f743ddf48910822f152420c4fc0
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89001389"
---
# <a name="performance-and-scalability-checklist-for-blob-storage"></a>Lista de comprobación de escalabilidad y rendimiento para Blob Storage

Microsoft ha desarrollado varios procedimientos de eficacia probada para desarrollar aplicaciones de alto rendimiento con Blob Storage. Esta lista de comprobación identifica los procedimientos clave que pueden seguir los desarrolladores para optimizar su rendimiento. Tenga presente estos procedimientos tanto para diseñar su aplicación como a lo largo de todo el proceso.

Azure Storage tiene objetivos de escalabilidad y rendimiento en lo que se refiere a capacidad, tasa de transacciones y ancho de banda. Para más información sobre los objetivos de escalabilidad de Azure Storage, consulte [Objetivos de escalabilidad y rendimiento para cuentas de almacenamiento estándar](../common/scalability-targets-standard-account.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) y [Objetivos de escalabilidad y rendimiento de Blob Storage](scalability-targets.md).

## <a name="checklist"></a>Lista de comprobación

En este artículo se organizan los procedimientos de eficacia probada para mejorar el rendimiento en una lista de comprobación que puede seguir al desarrollar su aplicación en Blob Storage.

| ¡Listo! | Category | Consideraciones acerca del diseño |
| --- | --- | --- |
| &nbsp; |Objetivos de escalabilidad |[¿Puede diseñar la aplicación para que no supere el número máximo de cuentas de almacenamiento?](#maximum-number-of-storage-accounts) |
| &nbsp; |Objetivos de escalabilidad |[¿Evita acercarse a los límites de transacción y capacidad?](#capacity-and-transaction-targets) |
| &nbsp; |Objetivos de escalabilidad |[¿Tiene un gran número de clientes que acceden a la vez a un único blob?](#multiple-clients-accessing-a-single-blob-concurrently) |
| &nbsp; |Objetivos de escalabilidad |[¿Permanece su aplicación dentro de los objetivos de escalabilidad de un solo blob?](#bandwidth-and-operations-per-blob) |
| &nbsp; |Creación de particiones |[Is your naming convention designed to enable better load-balancing? (¿Está su convención de nomenclatura diseñada para habilitar un mejor equilibrio de carga?)](#partitioning) |
| &nbsp; |Redes |[¿Disponen los dispositivos del cliente de un ancho de banda suficientemente grande y una latencia suficientemente baja como para lograr el rendimiento necesario?](#throughput) |
| &nbsp; |Redes |[¿Tienen los dispositivos del cliente un enlace de red de alta calidad?](#link-quality) |
| &nbsp; |Redes |[¿Está la aplicación cliente en la misma región que la cuenta de almacenamiento?](#location) |
| &nbsp; |Acceso directo del cliente |[¿Utiliza firmas de acceso compartido (SAS) y el uso compartido de recursos entre orígenes (CORS) para habilitar el acceso directo a Azure Storage?](#sas-and-cors) |
| &nbsp; |Almacenamiento en memoria caché |[¿La aplicación almacena en caché datos a los que se accede con frecuencia y que rara vez cambian?](#reading-data) |
| &nbsp; |Almacenamiento en memoria caché |[¿Su aplicación procesa por lotes las actualizaciones (las almacena en caché en el cliente y, luego, las carga en conjuntos más grandes)?](#uploading-data-in-batches) |
| &nbsp; |Configuración de .NET |[¿Usa .NET Core 2.1, o cualquier versión posterior, para lograr un rendimiento óptimo?](#use-net-core) |
| &nbsp; |Configuración de .NET |[¿Ha configurado el cliente para usar un número suficiente de conexiones simultáneas?](#increase-default-connection-limit) |
| &nbsp; |Configuración de .NET |[En el caso de las aplicaciones .NET, ¿ha configurado .NET para que use un número suficiente de subprocesos?](#increase-minimum-number-of-threads) |
| &nbsp; |Paralelismo |[¿Se ha asegurado de que el paralelismo está enlazado correctamente para no sobrecargar las funcionalidades del cliente ni acercarse a los objetivos de escalabilidad?](#unbounded-parallelism) |
| &nbsp; |Herramientas |[¿Usa las versiones más recientes de las herramientas y bibliotecas de cliente que proporciona Microsoft?](#client-libraries-and-tools) |
| &nbsp; |Reintentos |[¿Usa una directiva de reintentos con un retroceso exponencial para los errores de limitación y tiempos de espera?](#timeout-and-server-busy-errors) |
| &nbsp; |Reintentos |[¿Evita su aplicación reintentos para errores que no se pueden reintentar?](#non-retryable-errors) |
| &nbsp; |Copia de blobs |[¿Copia blobs de la manera más eficiente?](#blob-copy-apis) |
| &nbsp; |Copia de blobs |[¿Usa la versión más reciente de AzCopy para las operaciones de copia masiva?](#use-azcopy) |
| &nbsp; |Copia de blobs |[¿Usa la familia de Azure Data Box para importar grandes volúmenes de datos?](#use-azure-data-box) |
| &nbsp; |Distribución de contenido |[¿Usa una red CDN para distribución de contenido?](#content-distribution) |
| &nbsp; |Uso de metadatos |[¿Almacena metadatos usados frecuentemente acerca de blobs en los metadatos de estos?](#use-metadata) |
| &nbsp; |Carga rápida |[Cuando intenta cargar un blob rápidamente, ¿carga blobs en paralelo?](#upload-one-large-blob-quickly) |
| &nbsp; |Carga rápida |[Cuando intenta cargar muchos blobs rápidamente, ¿carga blobs en paralelo?](#upload-many-blobs-quickly) |
| &nbsp; |Tipo de blob |[¿Usa blobs en páginas o blobs en bloques cuando es apropiado?](#choose-the-correct-type-of-blob) |

## <a name="scalability-targets"></a>Objetivos de escalabilidad

Si su aplicación se aproxima o supera cualquiera de estos objetivos de escalabilidad, puede encontrar un aumento en la limitación o latencias de transacción. Cuando Azure Storage limita su aplicación, el servicio comienza a devolver códigos de error 503 (Servidor ocupado) o 500 (Tiempo de espera de la operación). Evitar estos errores, mediante la permanencia en los límites de los objetivos de escalabilidad, es una parte importante de la mejora del rendimiento de la aplicación.

Para más información acerca de los objetivos de escalabilidad de Queue service, consulte [Objetivos de escalabilidad y rendimiento de Azure Storage en cuentas de almacenamiento](/azure/storage/queues/scalability-targets#scale-targets-for-queue-storage).

### <a name="maximum-number-of-storage-accounts"></a>Número máximo de cuentas de almacenamiento

Si se está aproximando al número máximo de cuentas de almacenamiento permitidas para una combinación determinada de región y suscripción, evalúe su caso y determine si se aplica alguna de las condiciones siguientes:

- ¿Usa cuentas de almacenamiento para almacenar discos no administrados y agregar esos discos a las máquinas virtuales (VM)? En este caso, Microsoft recomienda usar discos administrados. Los discos administrados se escalan automáticamente y sin necesidad de crear y administrar cuentas de almacenamiento individuales. Para más información, consulte [Introducción a los discos administrados de Azure](../../virtual-machines/managed-disks-overview.md).
- ¿Usa una cuenta de almacenamiento por cliente, con el fin de aislar los datos? En este caso, Microsoft recomienda usar un contenedor de blobs para cada cliente, en lugar de una cuenta de almacenamiento completa. Azure Storage ahora le permite asignar roles de Azure para cada contenedor. Para más información, consulte [Conceder acceso a datos de blob y cola de Azure con RBAC en Azure Portal](../common/storage-auth-aad-rbac-portal.md).
- ¿Usa varias cuentas de almacenamiento como particiones para aumentar las operaciones de entrada, salida e IOPS (operaciones de E/S por segundo) o la capacidad? En este escenario, Microsoft recomienda aprovechar las ventajas del aumento de los límites de las cuentas de almacenamiento para reducir el número de cuentas de almacenamiento necesarias para la carga de trabajo, siempre que sea posible. Póngase en contacto con el [soporte técnico de Azure](https://azure.microsoft.com/support/options/) para solicitar el aumento de los límites de la cuenta de almacenamiento. Para más información, consulte [Announcing larger, higher scale storage accounts](https://azure.microsoft.com/blog/announcing-larger-higher-scale-storage-accounts/) (Anuncio de cuentas de almacenamiento más grandes y con mayor escala).

### <a name="capacity-and-transaction-targets"></a>Objetivos de capacidad y transacción

Si su aplicación se aproxima a los objetivos de escalabilidad para una sola cuenta de almacenamiento, plantéese la adopción de uno de los siguientes enfoques:  

- Si la aplicación alcanza el destino de la transacción, podría usar cuentas de almacenamiento de blobs en bloques, que están optimizadas para tasas elevadas de transacciones y una latencia baja y uniforme. Para más información, vea [Introducción a las cuentas de Azure Storage](../common/storage-account-overview.md).
- Reconsidere la carga de trabajo que hace que su aplicación se aproxime al objetivo de escalabilidad o lo supere. ¿Puede designarla de forma diferente para que use menos ancho de banda o capacidad, o menos transacciones?
- Si su aplicación debe superar uno de los objetivos de escalabilidad, cree varias cuentas de almacenamiento y realice particiones de los datos de su aplicación entre esas cuentas. Si usa este patrón, entonces debe asegurarse de designar la aplicación de forma que pueda agregar más cuentas de almacenamiento en el futuro para equilibrio de carga. Las propias cuentas de almacenamiento no tienen ningún costo aparte del de su uso en términos de datos almacenados, transacciones realizadas o datos transferidos.
- Si la aplicación se aproxima a los objetivos de ancho de banda, considere la posibilidad de comprimir los datos en el cliente para reducir el ancho de banda necesario para enviar los datos a Azure Storage.
    Aunque comprimir datos puede ahorrar ancho de banda y mejorar el rendimiento de la red, también puede tener efectos negativos en el rendimiento. Evalúe el impacto en el rendimiento de los requisitos de procesamiento adicionales para la compresión y descompresión de datos en el cliente. Tenga en cuenta que el almacenamiento de los datos comprimidos puede dificultar la solución de problemas, ya que es muy probable que sea más complicado ver los datos con herramientas estándar.
- Si la aplicación se acerca a los objetivos de escalabilidad, asegúrese de que usa un retroceso exponencial para los reintentos. Es recomendable que intente evitar llegar a los objetivos de escalabilidad mediante la implementación de las recomendaciones que se describen en este artículo. Sin embargo, el uso de un retroceso exponencial para los reintentos impedirá que la aplicación vuelva a intentarlo rápidamente, lo que podría empeorar la limitación. Para más información, consulte la sección [Errores de tiempo de expiración y servidor ocupado](#timeout-and-server-busy-errors).

### <a name="multiple-clients-accessing-a-single-blob-concurrently"></a>Acceso simultáneo a un único blob por parte de varios clientes

Si tiene un gran número de clientes que acceden de forma simultánea a un único blob, debe tener en cuenta los objetivos de escalabilidad por objeto y por cuenta de almacenamiento. El número exacto de clientes que pueden tener acceso a un único blob variará en función de factores como el número de clientes que soliciten el blob a la vez, el tamaño del blob y las condiciones de la red.

Si el blob se puede distribuir a través de una red CDN, como las imágenes o los vídeos procedentes de un sitio web, puede usar una de estas redes. Para más información, consulte la sección [Distribución de contenido](#content-distribution).

En otros casos, como las simulaciones científicas, donde los datos son confidenciales, tiene dos opciones. La primera es escalonar el acceso de la carga de trabajo para que se acceda al blob a lo largo de un período, en lugar de hacerlo de manera simultánea. Como alternativa, puede copiar el blob temporalmente en varias cuentas de almacenamiento a fin de aumentar el número total de IOPS por objeto y entre cuentas de almacenamiento. Los resultados variarán en función del comportamiento de la aplicación, así que asegúrese de probar los patrones de simultaneidad durante el diseño.

### <a name="bandwidth-and-operations-per-blob"></a>Ancho de banda y operaciones por blob

Un único blob admite hasta 500 solicitudes por segundo. Si tiene varios clientes que necesitan leer el mismo blob y podría superar este límite, tiene la posibilidad de usar una cuenta de almacenamiento de blob en bloques. Una cuenta de almacenamiento de blob en bloques proporciona una tasa de solicitudes más elevada o un número mayor de operaciones de E/S por segundo (IOPS).

También puede usar una red de entrega de contenido (CDN), como Azure CDN, para distribuir las operaciones en el blob. Para más información sobre Azure CDN, consulte [Introducción a Azure CDN](../../cdn/cdn-overview.md).  

## <a name="partitioning"></a>Creación de particiones

Para mejorar el rendimiento, es útil comprender cómo Azure Storage realiza el particionamiento de los datos del blob. Azure Storage puede suministrar datos de una sola partición con mayor rapidez que los datos que abarcan varias particiones. Con la asignación de un nombre apropiado para los blobs, puede mejorar la eficacia de las solicitudes de lectura.

Blob Storage emplea un esquema de particiones basado en intervalos para el escalado y el equilibrio de carga. Cada blob tiene una clave de partición compuesta por el nombre completo del blob (cuenta+contenedor+blob). La clave de partición se usa para particionar los datos de blob en intervalos. Luego, se equilibra la carga de los intervalos en Blob Storage.

La creación de particiones basada en intervalos significa que las convenciones de nomenclatura que usan el orden léxico (por ejemplo, *mypayroll*, *myperformance*, *myemployees*, etc.) o timestamps (*log20160101*, *log20160102*, *log20160102*, etc.) pueden dar lugar con una mayor probabilidad a que las particiones se ubiquen conjuntamente en el mismo servidor de particiones, hasta que el aumento de la carga requiera que se dividan en intervalos más pequeños. La ubicación conjunta de los blobs en el mismo servidor de particiones mejora el rendimiento, por lo que una parte importante de la mejora del rendimiento implica asignar a los blobs nombres que los organicen de manera más eficaz.

Por ejemplo, un servidor único puede atender a todos los blobs de un contenedor hasta que la carga de estos requiera un reequilibrado adicional de los intervalos de partición. Igualmente, un servidor único puede atender a un grupo de cuentas ligeramente cargadas con sus nombres organizados en orden léxico hasta que la carga de una o de todas ellas requiera su división en varios servidores de particiones.

Cada una de las operaciones de equilibrio de carga puede afectar a la latencia de las llamadas de almacenamiento realizadas durante la operación. La capacidad del sistema para administrar un aumento súbito del tráfico en una partición está limitada por la escalabilidad de un servidor de una única partición hasta que la operación de equilibrio de carga se inicia y se vuelve a equilibrar el intervalo de claves de la partición.

Puede seguir algunas prácticas recomendadas para reducir la frecuencia de dichas operaciones.  

- Si es posible, use tamaños de blob o bloque de más de 4 MiB para las cuentas de almacenamiento estándar y de más de 256 KiB para las cuentas de almacenamiento premium. Los tamaños grandes de blob o bloque activan automáticamente los blobs en bloques de alto rendimiento. Los blobs en bloques de alto rendimiento proporcionan un alto rendimiento en la ingesta que no se ve afectado por la nomenclatura de las particiones.
- Examine la convención de nomenclatura que usa para las cuentas, los contenedores, los blobs, las tablas y las colas. Podría colocar un prefijo en los nombres de cuenta, contenedor o blob con un hash de tres dígitos mediante la función hash que mejor se adapte a sus necesidades.
- Si organiza sus datos mediante marcas de tiempo o identificadores numéricos, asegúrese de no usar patrones de tráfico de solo anexar o solo anteponer. Estos patrones no son adecuados para un sistema de particionamiento basado en intervalos, ya que pueden dar lugar a que todo el tráfico se dirija a una partición e impedir el equilibrio de la carga del sistema de forma efectiva.

    Por ejemplo, si tiene operaciones diarias que usan un blob con una marca de tiempo, como *aaaammdd*, todo el tráfico de esa operación diaria se dirige a un solo objeto atendido por un servidor de una única partición. Compruebe si los límites por blob y por partición cubren sus necesidades y vea si puede dividir esta operación en varios blobs si es necesario. Igualmente, si almacena datos de series temporales en las tablas, todo el tráfico podría dirigirse a la última parte del espacio de nombres clave. Si usa identificadores numéricos, anteponga al identificador un hash de tres dígitos. Si usa marcas de tiempo, anteponga a la marca de tiempo el valor de segundos, por ejemplo, *ssddmmaaaa*. Si con la aplicación se realizan operaciones de enumeración y consulta de manera habitual, elija una función hash que limite el número de consultas. En algunos casos, un prefijo aleatorio puede ser suficiente.
  
- Para más información sobre el esquema de partición usado en Azure Storage, consulte [Azure Storage: A Highly Available Cloud Storage Service with Strong Consistency](https://sigops.org/sosp/sosp11/current/2011-Cascais/printable/11-calder.pdf) (Microsoft Azure Storage: Un servicio de almacenamiento en nube altamente disponible de gran coherencia).

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

Para una distribución amplia del contenido de los blobs, use una red de entrega de contenido, como Azure CDN. Para obtener más información acerca de Azure CDN, consulte [Azure CDN](../../cdn/cdn-overview.md).  

## <a name="sas-and-cors"></a>SAS y CORS

Supongamos que necesita autorizar código, como JavaScript, que se ejecuta en el explorador web de un usuario o en una aplicación de teléfono móvil para acceder a los datos de Azure Storage. Un enfoque consiste en crear una aplicación de servicio que actúe como proxy. El dispositivo del usuario se autentica con el servicio, que a su vez autoriza el acceso a los recursos de Azure Storage. De esta forma, puede evitar la exposición de sus claves de cuenta de almacenamiento en dispositivos no seguros. Sin embargo, este enfoque supone una importante sobrecarga para la aplicación de servicio, ya que todos los datos transferidos entre el dispositivo del usuario y Azure Storage deben pasar por la aplicación de servicio.

El uso de firmas de acceso compartido (SAS) le permite evitar el uso de una aplicación de servicio como proxy para Azure Storage. Mediante SAS se puede permitir que el dispositivo del usuario realice solicitudes directamente en Azure Storage mediante un token de acceso limitado. Por ejemplo, si un usuario desea cargar una foto en la aplicación, la aplicación de servicio puede generar una SAS y enviarla al dispositivo del usuario. El token de SAS puede conceder permiso para escribir en un recurso de Azure Storage durante un intervalo de tiempo especificado, después del cual expirará el token. Para obtener más información sobre las firmas de acceso compartido, consulte [Otorgar acceso limitado a recursos de Azure Storage con firmas de acceso compartido (SAS)](../common/storage-sas-overview.md).  

Por lo general, los exploradores web no permiten que haya código JavaScript en una página hospedada por un sitio web de un dominio que realiza operaciones concretas, como operaciones de escritura, en otro dominio. Esta directiva, que se conoce como directiva de mismo origen, impide que un script malintencionado de una página acceda a los datos de otra página web. Sin embargo, la directiva de mismo origen puede ser una limitación al compilar una solución en la nube. El uso compartido de recursos entre orígenes (CORS) es una característica del explorador que permite al dominio de destino comunicarle al explorador en el que confía solicitudes que se originan en el dominio de origen.

Por ejemplo, supongamos que una aplicación web que se ejecuta en Azure realiza una solicitud de un recurso en una cuenta de Azure Storage. La aplicación web es el dominio de origen y la cuenta de almacenamiento es el dominio de destino. Puede configurar CORS para que cualquiera de los servicios de Azure Storage comunique al explorador web que Azure Storage confía en las solicitudes del dominio de origen. Para más información sobre CORS, consulte [Soporte técnico del uso compartido de recursos entre orígenes (CORS) para Azure Storage](/rest/api/storageservices/Cross-Origin-Resource-Sharing--CORS--Support-for-the-Azure-Storage-Services).  
  
Tanto SAS como CORS pueden evitar una carga innecesaria en la aplicación web.  

## <a name="caching"></a>Almacenamiento en memoria caché

El almacenamiento en caché desempeña un papel importante en el rendimiento. En las secciones siguientes se describen los procedimientos recomendados de almacenamiento en caché.

### <a name="reading-data"></a>Lectura de datos

En general, es preferible leer los datos una vez que leerlos dos veces. Tome el ejemplo de una aplicación web que ha recuperado un blob 50 MiB de Azure Storage para servirlo como contenido a un usuario. Lo ideal es que la aplicación almacene en caché el blob localmente en el disco y que luego recupere la versión almacenada en caché en las solicitudes de usuario posteriores.

Una manera de evitar la recuperación de un blob si no se ha modificado desde que se almacenó en caché es calificar la operación GET con un encabezado condicional para la hora de modificación. Si la hora de la última modificación es posterior a la hora en que el blob se almacenó en caché, el blob se recupera y se vuelve a almacenar en caché. De lo contrario, el blob almacenado en caché se recupera para obtener un rendimiento óptimo.

También puede decidir diseñar la aplicación para asumir que el blob permanece sin cambios durante un breve período después de recuperarlo. En este caso, la aplicación no necesita comprobar si el blob se modificó durante ese intervalo.

Los datos de configuración, búsqueda y de otro tipo que con frecuencia se usan en la aplicación, son magníficos candidatos para el almacenamiento en caché.  

Para más información sobre los encabezados condicionales, consulte [Especificación de encabezados condicionales para las operaciones de Blob service](/rest/api/storageservices/specifying-conditional-headers-for-blob-service-operations).  

### <a name="uploading-data-in-batches"></a>Carga de datos en lotes

En algunos casos, puede agregar datos localmente para después cargarlos periódicamente en un lote en lugar de cargar cada fragmento de datos inmediatamente. Por ejemplo, supongamos que una aplicación web mantiene un archivo de registro de actividades. La aplicación puede cargar los detalles de cada actividad que sucede en una tabla (lo que requiere muchas operaciones de almacenamiento), o puede guardarlos en un archivo de registro local y, luego, cargarlos todos periódicamente como un archivo delimitado en un blob. Si cada entrada de registro tiene un tamaño de 1 KB, puede cargar miles de entradas en una sola transacción. Una sola transacción admite la carga de un blob de hasta 64 MiB. El desarrollador de la aplicación debe pensar en la posibilidad de que se produzcan errores de carga o del dispositivo del cliente. Si es necesario descargar los datos de actividad durante un intervalo en lugar de una sola actividad, se recomienda usar el almacenamiento de blobs por encima del almacenamiento de tablas.

## <a name="net-configuration"></a>Configuración de .NET

Si usa .NET Framework, esta sección enumera varias configuraciones rápidas que puede usar para realizar mejoras de rendimiento significativas.  Si usa otros lenguajes, compruebe si se aplican conceptos similares en el lenguaje elegido.  

### <a name="use-net-core"></a>Uso de .NET Core

Desarrolle las aplicaciones de Azure Storage con .NET Core 2.1, o cualquier versión posterior, para aprovechar las mejoras en el rendimiento. Se recomienda usar .NET Core 3.x siempre que sea posible.

Para más información sobre las mejoras en el rendimiento en .NET Core, consulte las siguientes entradas de blog:

- [Mejoras de rendimiento en .NET Core 3.0](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-core-3-0/)
- [Mejoras de rendimiento en .NET Core 2.1](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-core-2-1/)

### <a name="increase-default-connection-limit"></a>Aumento del límite de conexiones predeterminado

En .NET, el siguiente código aumenta el límite de conexiones predeterminado (que normalmente es dos en un entorno cliente o diez en un entorno servidor) a 100. Normalmente, debe establecer el valor en aproximadamente el número de subprocesos usados por su aplicación. Establezca el límite de conexiones antes de abrir cualquier conexión.

```csharp
ServicePointManager.DefaultConnectionLimit = 100; //(Or More)  
```

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

## <a name="copying-and-moving-blobs"></a>Copia y movimiento de blobs

Azure Storage ofrece varias soluciones para copiar y mover blobs dentro de una cuenta de almacenamiento, entre cuentas de almacenamiento y entre sistemas locales y la nube. En esta sección se describen algunas de estas opciones en términos de sus efectos sobre el rendimiento. Para información sobre cómo transferir datos de forma efectiva en Blob Storage, consulte [Elección de la solución de Azure para la transferencia de datos](../common/storage-choose-data-transfer-solution.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

### <a name="blob-copy-apis"></a>API de copia de blobs

Para copiar blobs entre cuentas de almacenamiento, use la operación [Put Block From URL](/rest/api/storageservices/put-block-from-url). Esta operación copia los datos sincrónicamente desde cualquier origen de dirección URL hasta un blob en bloques. El uso de la operación `Put Block from URL` puede reducir considerablemente el ancho de banda necesario al migrar datos entre cuentas de almacenamiento. Dado que la operación de copia se realiza en el lado del servicio, no es necesario descargar y volver a cargar los datos.

Para copiar los datos dentro de la misma cuenta de almacenamiento, use la operación [Copy Blob](/rest/api/storageservices/Copy-Blob). Las copias dentro de la misma cuenta de almacenamiento se suelen realizar rápidamente.  

### <a name="use-azcopy"></a>Uso de AzCopy

La utilidad de línea de comandos AzCopy es una opción sencilla y eficaz para la transferencia masiva de blobs entre cuentas de almacenamiento. Esta herramienta está optimizada para este escenario y con ella se pueden conseguir elevadas tasas de transferencia. La versión 10 de AzCopy usa la operación `Put Block From URL` para copiar datos de blobs entre cuentas de almacenamiento. Para más información, consulte [Copia o movimiento de datos a Azure Storage mediante AzCopy v10](/azure/storage/common/storage-use-azcopy-v10).  

### <a name="use-azure-data-box"></a>Uso de Azure Data Box

Para importar grandes volúmenes de datos en Blob Storage, tiene la opción de usar la familia Azure Data Box para las transferencias sin conexión. Los dispositivos Data Box que proporciona Microsoft son una buena opción para mover grandes cantidades de datos a Azure cuando existan limitaciones de tiempo, disponibilidad de la red o costos. Para más información, consulte la [documentación de Azure Data Box](/azure/databox/).

## <a name="content-distribution"></a>Distribución de contenido

Algunas veces, una aplicación necesita servir el mismo contenido a muchos usuarios (por ejemplo, un vídeo de demostración de producto usado en la página principal de un sitio web) ubicados bien en la misma región o en otras diferentes. En este caso, también puede usar una red de entrega de contenido (CDN), como Azure CDN, para distribuir el contenido de blobs entre las distintas ubicaciones geográficas. A diferencia de la cuenta de Azure Storage que existe en una sola región y que no puede entregar contenido con baja latencia a otras regiones, Azure CDN usa servidores en varios centros de datos alrededor del mundo. Además, una red CDN normalmente puede admitir límites de salida mucho más altos que una sola cuenta de almacenamiento.  

Para obtener más información acerca de Azure CDN, consulte [Azure CDN](../../cdn/cdn-overview.md).

## <a name="use-metadata"></a>Uso de metadatos

Blob service admite solicitudes HEAD que pueden incluir propiedades o metadatos del blob. Por ejemplo, si la aplicación necesita datos Exif (formato de archivo intercambiable) de una foto, podría recuperar la foto y extraerlos. Para ahorrar ancho de banda y mejorar el rendimiento, la aplicación puede almacenar los datos Exif en los metadatos del blob cuando cargue la foto. Luego, puede recuperar los datos Exif de los metadatos con solo una solicitud HEAD. La recuperación de solo los metadatos y no del contenido completo del blob ahorra un ancho de banda considerable y reduce el tiempo de procesamiento necesario para extraer los datos Exif. Tenga en cuenta que se pueden almacenar 8 KiB de metadatos por blob.  

## <a name="upload-blobs-quickly"></a>Carga rápida de blobs

Para cargar blobs rápidamente, determine primero si va a cargar un blob o varios. Utilice las instrucciones siguientes para determinar el método correcto que va a utilizar dependiendo del escenario.  

### <a name="upload-one-large-blob-quickly"></a>Carga rápida de un blob grande

Para cargar un solo blob grande rápidamente, la aplicación cliente puede cargar sus bloques o páginas en paralelo, teniendo en cuenta los objetivos de escalabilidad de blobs individuales y la cuenta de almacenamiento como un todo. Las bibliotecas cliente de Azure Storage admiten la carga en paralelo. Por ejemplo, puede usar las siguientes propiedades para especificar el número de solicitudes simultáneas permitidas en .NET o Java. Las bibliotecas cliente de otros idiomas admitidos proporcionan opciones similares.

- En .NET, establezca la propiedad [BlobRequestOptions.ParallelOperationThreadCount](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions.paralleloperationthreadcount).
- En Java/Android, llame al método [BlobRequestOptions.setConcurrentRequestCount(final Integer concurrentRequestCount)](/java/api/com.microsoft.azure.storage.blob.blobrequestoptions.setconcurrentrequestcount).

### <a name="upload-many-blobs-quickly"></a>Carga rápida de muchos blobs

Para cargar muchos blobs rápidamente, cárguelos en paralelo. Este método es más rápido que cargar blobs de uno en uno con cargas de bloque paralelas porque distribuye la carga entre varias particiones del servicio de almacenamiento. AzCopy realiza cargas en paralelo de forma predeterminada y es la herramienta recomendada en este caso. Para más información, consulte [Introducción a AzCopy](../common/storage-use-azcopy-v10.md).  

## <a name="choose-the-correct-type-of-blob"></a>Elección del tipo correcto de blob

Azure Storage admite blobs en bloques, blobs en anexos y blobs en páginas. Para un escenario de uso dado, el tipo de blob que elija afectará al rendimiento y escalabilidad de la solución.

Los blobs en bloques son adecuados cuando se quieren cargar grandes cantidades de datos de forma eficaz. Por ejemplo, una aplicación cliente que carga fotos o vídeo en Blob Storage irá destinada a los blobs en bloques.

Los blobs en anexos son similares a los blobs en bloques, ya que se componen de bloques. Cuando se modifica un blob en anexos, se agregan bloques solo al final del blob. Los blobs en anexos son útiles en casos como el registro, cuando una aplicación necesita agregar datos a un blob existente.

Los blobs en páginas son adecuados si la aplicación necesita realizar operaciones de escritura aleatorias en los datos. Por ejemplo, los discos de las máquinas virtuales de Azure se almacenan como blobs en páginas. Para más información, consulte [Uso de los blobs en bloques, los blobs en anexión y los blobs en páginas](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs).  

## <a name="next-steps"></a>Pasos siguientes

- [Objetivos de escalabilidad y rendimiento de Blob Storage](scalability-targets.md)
- [Destinos de escalabilidad y rendimiento para cuentas de almacenamiento estándar](../common/scalability-targets-standard-account.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
- [Estado y códigos de error](/rest/api/storageservices/Status-and-Error-Codes2)

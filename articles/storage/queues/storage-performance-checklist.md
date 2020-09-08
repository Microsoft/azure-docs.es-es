---
title: Lista de comprobación de rendimiento y escalabilidad para Queue Storage (Azure Storage)
description: Lista de comprobación de procedimientos de eficacia probada que se usan en Queue Storage al desarrollar aplicaciones de alto rendimiento.
services: storage
author: tamram
ms.service: storage
ms.topic: overview
ms.date: 10/10/2019
ms.author: tamram
ms.subservice: queues
ms.custom: devx-track-csharp
ms.openlocfilehash: 3b9aadf7d9cd27763cafb878d0b35d13a140a304
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89008410"
---
# <a name="performance-and-scalability-checklist-for-queue-storage"></a>Lista de comprobación de rendimiento y escalabilidad para Queue Storage

Microsoft ha desarrollado varios procedimientos de eficacia probada para desarrollar aplicaciones de alto rendimiento con Queue Storage. Esta lista de comprobación identifica los procedimientos clave que pueden seguir los desarrolladores para optimizar su rendimiento. Tenga presente estos procedimientos tanto para diseñar su aplicación como a lo largo de todo el proceso.

Azure Storage tiene objetivos de escalabilidad y rendimiento en lo que se refiere a capacidad, tasa de transacciones y ancho de banda. Para más información sobre los objetivos de escalabilidad de Azure Storage, consulte [Objetivos de escalabilidad y rendimiento para cuentas de almacenamiento estándar](../common/scalability-targets-standard-account.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json) y [Objetivos de escalabilidad y rendimiento de Queue Storage](scalability-targets.md).

## <a name="checklist"></a>Lista de comprobación

En este artículo se organizan los procedimientos de eficacia probada a la hora de mejorar el rendimiento en una lista de comprobación que puede seguir al desarrollar su aplicación en Queue Storage.

| ¡Listo! | Category | Consideraciones acerca del diseño |
| --- | --- | --- |
| &nbsp; |Objetivos de escalabilidad |[¿Puede diseñar la aplicación para que no supere el número máximo de cuentas de almacenamiento?](#maximum-number-of-storage-accounts) |
| &nbsp; |Objetivos de escalabilidad |[¿Evita acercarse a los límites de transacción y capacidad?](#capacity-and-transaction-targets) |
| &nbsp; |Redes |[¿Disponen los dispositivos del cliente de un ancho de banda suficientemente grande y una latencia suficientemente baja como para lograr el rendimiento necesario?](#throughput) |
| &nbsp; |Redes |[¿Tienen los dispositivos del cliente un enlace de red de alta calidad?](#link-quality) |
| &nbsp; |Redes |[¿Está la aplicación cliente en la misma región que la cuenta de almacenamiento?](#location) |
| &nbsp; |Acceso directo del cliente |[¿Utiliza firmas de acceso compartido (SAS) y el uso compartido de recursos entre orígenes (CORS) para habilitar el acceso directo a Azure Storage?](#sas-and-cors) |
| &nbsp; |Configuración de .NET |[¿Usa .NET Core 2.1, o cualquier versión posterior, para lograr un rendimiento óptimo?](#use-net-core) |
| &nbsp; |Configuración de .NET |[¿Ha configurado el cliente para usar un número suficiente de conexiones simultáneas?](#increase-default-connection-limit) |
| &nbsp; |Configuración de .NET |[En el caso de las aplicaciones .NET, ¿ha configurado .NET para que use un número suficiente de subprocesos?](#increase-minimum-number-of-threads) |
| &nbsp; |Paralelismo |[¿Se ha asegurado de que el paralelismo está enlazado correctamente para no sobrecargar las funcionalidades del cliente ni acercarse a los objetivos de escalabilidad?](#unbounded-parallelism) |
| &nbsp; |Herramientas |[¿Usa las versiones más recientes de las herramientas y bibliotecas de cliente que proporciona Microsoft?](#client-libraries-and-tools) |
| &nbsp; |Reintentos |[¿Usa una directiva de reintentos con un retroceso exponencial para los errores de limitación y tiempos de espera?](#timeout-and-server-busy-errors) |
| &nbsp; |Reintentos |[¿Evita su aplicación reintentos para errores que no se pueden reintentar?](#non-retryable-errors) |
| &nbsp; |Configuración |[¿Ha desactivado el algoritmo de Nagle para mejorar el rendimiento de las solicitudes pequeñas?](#disable-nagle) |
| &nbsp; |Tamaño del mensaje |[¿Son sus mensajes compactos para mejorar el rendimiento de la cola?](#message-size) |
| &nbsp; |Recuperación en masa |[¿Recupera varios mensajes en una sola operación GET?](#batch-retrieval) |
| &nbsp; |Frecuencia de sondeo |[¿Realiza sondeos con la suficiente frecuencia para reducir la latencia percibida de su aplicación?](#queue-polling-interval) |
| &nbsp; |Actualizar mensaje |[¿Usa la operación Actualizar mensaje para almacenar el progreso en los mensajes de procesamiento con el fin de evitar volver a procesar todo el mensaje si se produce un error?](#use-update-message) |
| &nbsp; |Architecture |[¿Usa colas para hacer que toda su aplicación sea más escalable manteniendo cargas de trabajo de ejecución prolongada fuera de la ruta de acceso crítica y escala después de forma independiente?](#application-architecture) |

## <a name="scalability-targets"></a>Objetivos de escalabilidad

Si su aplicación se aproxima o supera cualquiera de estos objetivos de escalabilidad, puede encontrar un aumento en la limitación o latencias de transacción. Cuando Azure Storage limita su aplicación, el servicio comienza a devolver códigos de error 503 (Servidor ocupado) o 500 (Tiempo de espera de la operación). Evitar estos errores, mediante la permanencia en los límites de los objetivos de escalabilidad, es una parte importante de la mejora del rendimiento de la aplicación.

Para más información acerca de los objetivos de escalabilidad de Queue service, consulte [Objetivos de escalabilidad y rendimiento de Azure Storage en cuentas de almacenamiento](/azure/storage/queues/scalability-targets#scale-targets-for-queue-storage).

### <a name="maximum-number-of-storage-accounts"></a>Número máximo de cuentas de almacenamiento

Si se aproxima al número máximo de cuentas de almacenamiento permitido para una combinación de suscripción/región concreta, ¿usa varias cuentas de almacenamiento para la partición para aumentar la entrada, la salida, las operaciones de E/S por segundo o la capacidad? En este escenario, Microsoft recomienda aprovechar las ventajas del aumento de los límites de las cuentas de almacenamiento para reducir el número de cuentas de almacenamiento necesarias para la carga de trabajo, siempre que sea posible. Póngase en contacto con el [soporte técnico de Azure](https://azure.microsoft.com/support/options/) para solicitar el aumento de los límites de la cuenta de almacenamiento. Para más información, consulte [Announcing larger, higher scale storage accounts](https://azure.microsoft.com/blog/announcing-larger-higher-scale-storage-accounts/) (Anuncio de cuentas de almacenamiento más grandes y con mayor escala).

### <a name="capacity-and-transaction-targets"></a>Objetivos de capacidad y transacción

Si su aplicación se aproxima a los objetivos de escalabilidad para una sola cuenta de almacenamiento, plantéese la adopción de uno de los siguientes enfoques:  

- Si los objetivos de escalabilidad de las colas no son suficientes para su aplicación, use varias colas y distribuya los mensajes entre ellas.
- Reconsidere la carga de trabajo que hace que su aplicación se aproxime al objetivo de escalabilidad o lo supere. ¿Puede designarla de forma diferente para que use menos ancho de banda o capacidad, o menos transacciones?
- Si su aplicación debe superar uno de los objetivos de escalabilidad, cree varias cuentas de almacenamiento y realice particiones de los datos de su aplicación entre esas cuentas. Si usa este patrón, entonces debe asegurarse de designar la aplicación de forma que pueda agregar más cuentas de almacenamiento en el futuro para equilibrio de carga. Las propias cuentas de almacenamiento no tienen ningún costo aparte del de su uso en términos de datos almacenados, transacciones realizadas o datos transferidos.
- Si la aplicación se aproxima a los objetivos de ancho de banda, considere la posibilidad de comprimir los datos en el cliente para reducir el ancho de banda necesario para enviar los datos a Azure Storage.
    Aunque comprimir datos puede ahorrar ancho de banda y mejorar el rendimiento de la red, también puede tener efectos negativos en el rendimiento. Evalúe el impacto en el rendimiento de los requisitos de procesamiento adicionales para la compresión y descompresión de datos en el cliente. Tenga en cuenta que el almacenamiento de los datos comprimidos puede dificultar la solución de problemas, ya que es muy probable que sea más complicado ver los datos con herramientas estándar.
- Si la aplicación se acerca a los objetivos de escalabilidad, asegúrese de que usa un retroceso exponencial para los reintentos. Es recomendable que intente evitar llegar a los objetivos de escalabilidad mediante la implementación de las recomendaciones que se describen en este artículo. Sin embargo, el uso de un retroceso exponencial para los reintentos impedirá que la aplicación vuelva a intentarlo rápidamente, lo que podría empeorar la limitación. Para más información, consulte la sección [Errores de tiempo de expiración y servidor ocupado](#timeout-and-server-busy-errors).

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

## <a name="disable-nagle"></a>Deshabilitación de Nagle

El algoritmo de Nagle está ampliamente implementado en redes TCP/IP como medio de mejorar el rendimiento de la red. Sin embargo, no es óptimo en todas las situaciones (como por ejemplo en entornos altamente interactivos). El algoritmo de Nagle tiene un efecto negativo en el rendimiento de las solicitudes en Azure Table service y debe deshabilitarlo si es posible.

## <a name="message-size"></a>Tamaño del mensaje

El rendimiento y la escalabilidad de las colas se reducen a medida que aumenta el tamaño de los mensajes. Coloque solo la información que necesita el receptor en un mensaje.  

## <a name="batch-retrieval"></a>Recuperación por lotes

Puede recuperar hasta 32 mensajes de una cola en una sola operación. La recuperación por lotes puede reducir el número de recorridos de ida y vuelta de la aplicación cliente, lo que es especialmente útil para entornos con alta latencia como, por ejemplo, dispositivos móviles.  

## <a name="queue-polling-interval"></a>Intervalo de sondeo de la cola

La mayoría de aplicaciones sondean los mensajes de una cola, que puede ser uno de los principales orígenes de las transacciones de la aplicación. Seleccione el intervalo de sondeo con cuidado: un sondeo demasiado frecuente puede provocar que la aplicación alcance los objetivos de escalabilidad para la cola. Sin embargo, a 200.000 transacciones por 0,01 USD (en el momento de redactar), un solo procesador que sondeara una vez cada pocos segundos durante un mes costaría menos de 15 centavos, así que el coste de sondeo no suele ser un factor que afecte a la elección del intervalo de sondeo.  

Para obtener información de costo actualizada, consulte [Precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/).  

## <a name="use-update-message"></a>Uso de Actualizar mensaje

Puede usar la operación **Actualizar mensaje** para aumentar el tiempo de expiración de la invisibilidad o para actualizar la información de estado de un mensaje. El uso de **Actualizar mensaje** puede ser un enfoque mucho más eficaz que tener un flujo de trabajo que pasa un trabajo de una cola a la siguiente a medida que se completa cada uno de los pasos del trabajo. La aplicación puede guardar el estado del trabajo en el mensaje y, después, continuar trabajando, en lugar de volver a poner en cola el mensaje para el próximo paso del trabajo cada vez que se completa un paso. Tenga en cuenta que cada operación **Actualizar mensaje** cuenta para el objetivo de escalabilidad.

## <a name="application-architecture"></a>Arquitectura de la aplicación

Use colas para que la arquitectura de la aplicación sea escalable. A continuación se enumeran algunas formas de usar colas para que la aplicación sea más escalable:  

- Puede usar colas para crear trabajos pendientes para el procesamiento y aliviar las cargas de la aplicación. Por ejemplo, podría poner en cola solicitudes de usuarios para realizar un trabajo que requiere muchos recursos de procesador, como, por ejemplo, cambiar el tamaño de las imágenes cargadas.
- Puede usar colas para desacoplar partes de la aplicación para poder escalarlas por separado. Por ejemplo, un front-end web podría poner resultados de encuesta de usuarios en una cola para analizarlos y almacenarlos posteriormente. Podría agregar más instancias de rol de trabajo para procesar los datos de cola según sea necesario.  

## <a name="next-steps"></a>Pasos siguientes

- [Objetivos de escalabilidad y rendimiento para Queue Storage](scalability-targets.md)
- [Destinos de escalabilidad y rendimiento para cuentas de almacenamiento estándar](../common/scalability-targets-standard-account.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json)
- [Estado y códigos de error](/rest/api/storageservices/Status-and-Error-Codes2)

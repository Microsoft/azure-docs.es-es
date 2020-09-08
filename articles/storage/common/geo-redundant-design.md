---
title: Uso de redundancia geográfica para diseñar aplicaciones de alta disponibilidad
titleSuffix: Azure Storage
description: Aprenda a usar el almacenamiento con redundancia geográfica para diseñar una aplicación de alta disponibilidad lo bastante flexible como para enfrentarse a interrupciones.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 05/05/2020
ms.author: tamram
ms.reviewer: artek
ms.subservice: common
ms.custom: devx-track-csharp
ms.openlocfilehash: a6aed0630acf6ee6624c72831a2cdc88e6c0a91d
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89013068"
---
# <a name="use-geo-redundancy-to-design-highly-available-applications"></a>Uso de redundancia geográfica para diseñar aplicaciones de alta disponibilidad

Una característica común de las infraestructuras basadas en la nube como Azure Storage es que proporcionan una plataforma de alta disponibilidad y duración para hospedar datos y aplicaciones. Los desarrolladores de aplicaciones basadas en la nube deben considerar atentamente cómo aprovechar esta plataforma para maximizar estas ventajas para sus usuarios. Azure Storage ofrece almacenamiento con redundancia geográfica para garantizar la alta disponibilidad incluso en caso de interrupción regional. Las cuentas de almacenamiento configuradas para la replicación con redundancia geográfica se replican de forma sincrónica en la región primaria y luego se replican de forma asincrónica en una región secundaria que se encuentra a cientos de kilómetros de distancia.

Azure Storage ofrece dos tipos de replicación con redundancia geográfica. La única diferencia entre estas dos opciones es la forma en que los datos se replican en la región principal:

* [Almacenamiento con redundancia de zona geográfica (GZRS)](storage-redundancy.md): Los datos se replican de forma sincrónica en tres zonas de disponibilidad de Azure en la región principal mediante el *almacenamiento con redundancia de zona (ZRS)* y, luego, se replican de forma asincrónica en la región secundaria. Para obtener acceso de lectura a los datos de la región secundaria, habilite el almacenamiento con redundancia de zona geográfica con acceso de lectura (RA-GZRS).

    Microsoft recomienda el uso de GZRS/RA-GZRS para escenarios que requieren la máxima disponibilidad y durabilidad.

* [Almacenamiento con redundancia geográfica (GRS)](storage-redundancy.md): Los datos se replican de forma sincrónica tres veces en la región principal mediante el *almacenamiento con redundancia local (LRS)* y, luego, se replican de forma asincrónica en la región secundaria. Para obtener acceso de lectura a los datos de la región secundaria, habilite el almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS).

En este artículo se muestra cómo diseñar la aplicación para controlar una interrupción en la región primaria. Si la región primaria deja de estar disponible, la aplicación se puede adaptar para realizar operaciones de lectura en la región secundaria en su lugar. Antes de comenzar, asegúrese de que la cuenta de almacenamiento está configurada para RA-GRS o RA-GZRS.

## <a name="application-design-considerations-when-reading-from-the-secondary"></a>Consideraciones sobre el diseño de aplicaciones al leer desde la región secundaria

El propósito de este artículo es mostrar cómo diseñar una aplicación que siga funcionando (aunque con limitaciones) incluso si se produce un desastre importante en el centro de datos principal. Puede diseñar la aplicación para que se enfrente a problemas transitorios o prolongados pasando a leer desde la región secundaria cuando se produzca un problema que interfiera con la lectura desde la región primaria. Cuando la región primaria vuelva a estar disponible, la aplicación podrá volver a leer desde ella.

Cuando diseñe su aplicación para RA-GRS o RA-GZRS debe tener en cuenta estos puntos clave:

* Azure Storage mantiene una copia de solo lectura de los datos que almacena en la región primaria en una región secundaria. Como se mencionó anteriormente, el servicio de almacenamiento determina la ubicación de la región secundaria.

* La copia de solo lectura es de [coherencia final](https://en.wikipedia.org/wiki/Eventual_consistency) con los datos en la región primaria.

* Para blobs, tablas y colas, puede consultar en la región secundaria un valor *Hora de última sincronización* que indica cuándo se produjo la última replicación desde la región primaria a la secundaria (Esto no se admite para Azure Files, que no tiene redundancia RA-GRS en este momento).

* Puede usar la biblioteca cliente de Storage para leer y escribir los datos en la región primaria o la secundaria. También puede redirigir solicitudes de lectura automáticamente a la región secundaria si se agota el tiempo de espera de una solicitud de lectura a la región principal.

* Si la región primaria deja de estar disponible, puede iniciar una conmutación por error de la cuenta. Si la conmutación por error tiene como destino la región secundaria, las entradas DNS que apunten a la región primaria se modificarán para que apunten a la región secundaria. Una vez completada la conmutación por error, se restaurará el acceso de escritura en las cuentas GRS y RA-GRS. Para obtener más información, consulte [Recuperación ante desastres y conmutación por error de la cuenta de almacenamiento](storage-disaster-recovery-guidance.md).

### <a name="using-eventually-consistent-data"></a>Uso de datos con coherencia final

En la solución propuesta se da por supuesto que es aceptable devolver datos potencialmente obsoletos a la aplicación que realiza la llamada. Debido a que, en última instancia, los datos de la región secundaria son coherentes, puede que no sea posible acceder a la región primaria antes de que una actualización en la región secundaria haya terminado de replicarse.

Por ejemplo, imagine que el cliente envía correctamente una actualización, pero se produce un error en la región primaria antes de que la actualización se propague a la región secundaria. Cuando el cliente pide leer de nuevo los datos, se reciben los datos obsoletos de la región secundaria en lugar de los datos actualizados. Cuando diseña la aplicación, debe decidir si esto es aceptable y, en tal caso, cómo enviará mensajes al cliente. 

Más adelante en este artículo, le mostraremos cómo comprobar la hora de la última sincronización de los datos secundarios para ver si la región secundaria está actualizada.

### <a name="handling-services-separately-or-all-together"></a>Control de servicios por separado o conjuntamente

Aunque no es probable, es posible que un servicio deje de estar disponible mientras que los demás sigan siendo totalmente funcionales. Puede controlar los reintentos y el modo de solo lectura para cada servicio por separado (blobs, colas, tablas) o puede controlar los reintentos de forma genérica para todos los servicios de almacenamiento en conjunto.

Por ejemplo, si usa colas y blobs en la aplicación, puede decidir colocarlos de forma independiente en el código para poder controlar los errores con posibilidad de reintento para cada uno de ellos. Entonces, si obtiene un reintento desde Blob service, pero Queue service sigue funcionando, solo se verá afectada la parte de la aplicación que controla blobs. Si decide controlar todos los reintentos de servicios de almacenamiento de forma genérica y una llamada a Blob service devuelve un error con posibilidad de reintento, se verán afectadas las solicitudes tanto a Blob service como a Queue service.

En última instancia, esto depende de la complejidad de la aplicación. Es posible que decida no controlar los errores para cada servicio, sino redirigir las solicitudes de lectura para todos los servicios de almacenamiento a la región secundaria y ejecutar la aplicación en modo de solo lectura cuando se detecte un problema con cualquier servicio de almacenamiento en la región primaria.

### <a name="other-considerations"></a>Otras consideraciones

En el resto del artículo, también se tratarán estas consideraciones.

* Control de los reintentos de solicitudes de lectura mediante el patrón de disyuntor

* Datos de coherencia final y la hora de la última sincronización

* Prueba

## <a name="running-your-application-in-read-only-mode"></a>Ejecución de la aplicación en modo de solo lectura

Para prepararse de forma eficaz para una interrupción en la región primaria, debe ser capaz de controlar tanto solicitudes de lectura con errores como solicitudes de actualización con errores (aquí actualización se refiere a inserciones, actualizaciones y eliminaciones). Si se produce un error en la región primaria, se pueden redirigir las solicitudes de lectura a la región secundaria. Sin embargo, las solicitudes de actualización no se puede redirigir a la región secundaria porque es de solo lectura. Por este motivo, debe diseñar la aplicación para que se ejecute en modo de solo lectura.

Por ejemplo, puede establecer una marca que se comprueba antes de que se envíe cualquier solicitud de actualización a Azure Storage. Cuando llegue una de las solicitudes de actualización, puede pasarla por alto y devolver una respuesta adecuada al cliente. Incluso es posible que desee deshabilitar totalmente determinadas características hasta que se resuelva el problema y notificar a los usuarios que esas características no están disponibles temporalmente.

Si decide controlar errores para cada servicio por separado, también necesitará controlar la capacidad de ejecutar la aplicación en modo de solo lectura para cada servicio. Por ejemplo, es posible que tenga marcas de solo lectura en cada servicio que se puede habilitar o deshabilitar. Luego puede manipular la marca en los lugares correspondientes del código.

Poder ejecutar la aplicación en modo de solo lectura ofrece otra ventaja: la capacidad de garantizar una funcionalidad limitada durante una actualización importante de la aplicación. Puede desencadenar la aplicación para que se ejecute en modo de solo lectura y apunte al centro de datos secundario, de forma que nadie acceda a los datos de la región primaria mientras se estén llevando a cabo actualizaciones.

## <a name="handling-updates-when-running-in-read-only-mode"></a>Control de las actualizaciones durante la ejecución en modo de solo lectura

Existen muchas formas de controlar solicitudes de actualización durante la ejecución en modo de solo lectura. No se va a tratar esto de forma exhaustiva, pero, por lo general, hay un par de patrones que tener en cuenta.

1. Puede responder al usuario y indicarle que actualmente no se aceptan actualizaciones. Por ejemplo, un sistema de administración de contactos podría permitir que los clientes accedan a información de contacto, pero no que la actualicen.

2. Puede poner las actualizaciones en cola en otra región. En este caso, podría escribir las solicitudes de actualización pendientes en una cola de una región distinta y, después, contar con una forma de procesar esas solicitudes una vez que el centro de datos principal vuelva a estar en línea. En este escenario, debería hacer saber al cliente que la actualización solicitada está en cola para procesarse más adelante.

3. Puede escribir las actualizaciones en una cuenta de almacenamiento de otra región. Después, cuando el centro de datos principal vuelva a estar en línea, puede disponer de una forma de combinar esas actualizaciones con los datos principales, en función de la estructura de estos. Por ejemplo, si va a crear archivos distintos con una marca de fecha y hora en el nombre, puede copiar esos archivos de nuevo a la región primaria. Esto funciona para algunas cargas de trabajo como los datos de registro e IoT.

## <a name="handling-retries"></a>Control de los reintentos

La biblioteca cliente de Azure Storage le ayuda a determinar qué errores se pueden reintentar. Por ejemplo, un error 404 (recurso no encontrado) no se intentaría de nuevo porque no es probable que el nuevo intento se complete de forma correcta. Por otro lado, un error 500 se puede intentar de nuevo porque es un error del servidor y puede tratarse simplemente de un problema transitorio. Para más detalles, visite el [código fuente abierto de la clase ExponentialRetry](https://github.com/Azure/azure-storage-net/blob/87b84b3d5ee884c7adc10e494e2c7060956515d0/Lib/Common/RetryPolicies/ExponentialRetry.cs) en la biblioteca del cliente de almacenamiento de .NET. (Busque el método ShouldRetry).

### <a name="read-requests"></a>Solicitudes de lectura

Las solicitudes de lectura se pueden redirigir a almacenamiento secundario si hay un problema con el principal. Como se indicó antes en [Uso de datos con coherencia final](#using-eventually-consistent-data), debe ser aceptable que su aplicación lea datos potencialmente obsoletos. Si usa la biblioteca del cliente de almacenamiento para acceder a datos desde la región secundaria, puede especificar el comportamiento de reintento de una solicitud de lectura estableciendo un valor para la propiedad **LocationMode** en una de las siguientes opciones:

* **PrimaryOnly** (valor predeterminado)

* **PrimaryThenSecondary**

* **SecondaryOnly**

* **SecondaryThenPrimary**

Cuando se establece **LocationMode** en **PrimaryThenSecondary**, si la solicitud de lectura inicial para el punto de conexión principal genera un error que se puede reintentar, el cliente presenta automáticamente otra solicitud de lectura al punto de conexión secundario. Si el error es por agotamiento del tiempo de espera del servidor, el cliente tendrá que esperar a que el tiempo de espera expire antes de recibir un error con posibilidad de reintento del servicio.

Básicamente, se deben tener en cuenta dos escenarios al decidir cómo responder a un error con posibilidad de reintento:

* Se trata de un problema aislado y las solicitudes posteriores al punto de conexión principal no devolverán un error con posibilidad de reintento. Un ejemplo de cuándo puede ocurrir esto es un error de red transitorio.

    En este escenario, no hay ninguna reducción notable del rendimiento por tener **LocationMode** establecido en **PrimaryThenSecondary**, ya que esto solo sucede en raras ocasiones.

* Se trata de un problema con al menos uno de los servicios de almacenamiento en la región principal y es probable que todas las solicitudes posteriores a ese servicio en la región principal devuelvan errores con posibilidad de reintento durante un tiempo. Un ejemplo se da si la región principal es totalmente inaccesible.

    En este escenario, el rendimiento se ve afectado porque todas las solicitudes de lectura intentarán en primer lugar el punto de conexión principal, esperarán a que el tiempo de espera expire y después cambiarán al punto de conexión secundario.

Para estos escenarios, debe reconocer que existe un problema continuado con el punto de conexión principal y enviar todas las solicitudes de lectura directamente al punto de conexión secundario; para ello, establezca la propiedad **LocationMode** en **SecondaryOnly**. En este momento, también debería cambiar la aplicación para que se ejecute en modo de solo lectura. Este enfoque se conoce como el [patrón de disyuntor](/azure/architecture/patterns/circuit-breaker).

### <a name="update-requests"></a>Solicitudes de actualización

El patrón de disyuntor también se puede aplicar a las solicitudes de actualización. Sin embargo, estas no se pueden redirigir al almacenamiento secundario, que es de solo lectura. Para estas solicitudes, debe dejar la propiedad **LocationMode** establecida en **PrimaryOnly** (valor predeterminado). Para controlar estos errores, puede aplicar una métrica a estas solicitudes (por ejemplo, 10 errores consecutivos) y, cuando se alcance el umbral, cambiar la aplicación al modo de solo lectura. Puede usar los mismos métodos para volver al modo de actualización que los descritos en la siguiente sección sobre el patrón de disyuntor.

## <a name="circuit-breaker-pattern"></a>Patrón de disyuntor

Si usa el patrón de disyuntor en la aplicación, puede impedir que reintente una operación que es probable que produzca errores una y otra vez. Permite que la aplicación siga ejecutándose en lugar de ocupar tiempo mientras se reintenta la operación de forma exponencial. Además, detecta cuándo se ha corregido el error, momento en el que la aplicación puede intentar la operación de nuevo.

### <a name="how-to-implement-the-circuit-breaker-pattern"></a>Cómo implementar el patrón de disyuntor

Para identificar la existencia de un problema continuado con un punto de conexión principal, puede supervisar la frecuencia con que el cliente encuentra errores con posibilidad de reintento. Como cada caso es diferente, tendrá que decidir el umbral que desea usar para la decisión de cambiar al punto de conexión secundario y ejecutar la aplicación en modo de solo lectura. Por ejemplo, podría decidir cambiar si hay diez errores consecutivos sin ninguna ejecución correcta. Otro ejemplo consiste en cambiar si se produce un error en el 90 % de las solicitudes en un periodo de dos minutos.

Para el primer escenario, basta con que cuente los errores y, si se produce un resultado correcto antes de alcanzar el máximo, restablezca el recuento a cero. Para el segundo escenario, una posibilidad de implementación es usar el objeto MemoryCache (en. NET). Para cada solicitud, agregue un elemento CacheItem a la memoria caché, establezca el valor en correcto (1) o error (0), y establezca la hora de expiración en 2 minutos a partir del momento actual (o la restricción temporal que sea). Cuando se alcanza la hora de expiración de una entrada, esta se quita automáticamente. Esto le dará un intervalo gradual de 2 minutos. Cada vez que se envíe una solicitud al servicio de almacenamiento, use primero una consulta Linq en el objeto MemoryCache para calcular el porcentaje de corrección, para lo cual se suman los valores y se dividen por el recuento. Cuando el porcentaje de corrección sea inferior a un umbral (por ejemplo, 10 %), establezca la propiedad **LocationMode** para solicitudes de lectura en **SecondaryOnly** y cambie la aplicación al modo de solo lectura antes de continuar.

El umbral de errores que se usa para determinar cuándo se debe realizar el cambio puede variar para cada servicio en la aplicación, por lo que debería considerar convertir los umbrales en parámetros configurables. Aquí también se decide controlar los errores con posibilidad de reintento de cada servicio por separado o en conjunto, según lo descrito antes.

Otra consideración es cómo controlar varias instancias de una aplicación y qué hacer cuando se detecten errores con posibilidad de reintento en cada instancia. Por ejemplo, puede que tenga 20 máquinas virtuales que se ejecutan con la misma aplicación cargada. ¿Va a controlar cada instancia por separado? Si una instancia empieza a tener problemas, ¿desea limitar la respuesta solo a esa instancia o intentar que todas las instancias respondan de la misma manera cuando una de ellas tenga un problema? Controlar las instancias por separado es mucho más sencillo que intentar coordinar la respuesta en todas ellas, pero la forma de hacerlo depende de la arquitectura de la aplicación.

### <a name="options-for-monitoring-the-error-frequency"></a>Opciones para supervisar la frecuencia de los errores

Dispone de tres opciones principales para supervisar la frecuencia de reintentos en la región primaria y determinar cuándo se debe cambiar a la región secundaria y hacer que la aplicación se ejecute en modo de solo lectura.

* Agregue un controlador para el evento [**Retrying**](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.table.operationcontext.retrying) en el objeto [**OperationContext**](https://docs.microsoft.com/java/api/com.microsoft.applicationinsights.extensibility.context.operationcontext) que se pasa a las solicitudes de almacenamiento: este método se muestra en este artículo y se usa en el ejemplo que lo acompaña. Estos eventos se activan cada vez que el cliente reintenta una solicitud, lo que le permite realizar un seguimiento de la frecuencia con que el cliente encuentra errores con posibilidad de reintento en un punto de conexión principal.

    ```csharp
    operationContext.Retrying += (sender, arguments) =>
    {
        // Retrying in the primary region
        if (arguments.Request.Host == primaryhostname)
            ...
    };
    ```

* En el método [**Evaluate**](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.table.iextendedretrypolicy.evaluate) de una directiva de reintentos personalizada, puede ejecutar código personalizado siempre que se lleve a cabo un reintento. Además de registrar cuándo sucede un reintento, esto también ofrece la oportunidad de modificar el comportamiento de reintento.

    ```csharp
    public RetryInfo Evaluate(RetryContext retryContext,
    OperationContext operationContext)
    {
        var statusCode = retryContext.LastRequestResult.HttpStatusCode;
        if (retryContext.CurrentRetryCount >= this.maximumAttempts
            || ((statusCode >= 300 && statusCode < 500 && statusCode != 408)
            || statusCode == 501 // Not Implemented
            || statusCode == 505 // Version Not Supported
            ))
        {
            // Do not retry
            return null;
        }

        // Monitor retries in the primary location
        ...

        // Determine RetryInterval and TargetLocation
        RetryInfo info =
            CreateRetryInfo(retryContext.CurrentRetryCount);

        return info;
    }
    ```

* El tercer enfoque consiste en implementar un componente de supervisión personalizado en la aplicación que haga ping continuamente al punto de conexión de almacenamiento principal con solicitudes de lectura ficticias (por ejemplo, la lectura de un blob pequeño) para determinar su estado. Esto consumiría recursos, pero no en cantidades notables. Cuando se detecte un problema que alcance el umbral, se llevar a cabo el cambio a **SecondaryOnly** y al modo de solo lectura.

En algún momento, querrá que se vuelva a usar el punto de conexión principal y se permitan las actualizaciones. Si usa uno de los dos primeros métodos mencionados antes, podría simplemente cambiar al punto de conexión principal y habilitar el modo de actualización después de un intervalo de tiempo arbitrario o de que se haya llevado a cabo un número de operaciones. A continuación, puede dejar que se repita la lógica de reintento. Si se ha corregido el problema, procederá a usar el punto de conexión principal y a permitir las actualizaciones. Si el problema persiste, volverá una vez más a cambiar al punto de conexión secundario y al modo de solo lectura después de que se incumplan los criterios que haya establecido.

Para el tercer escenario, cuando vuelva a hacer ping correctamente al punto de conexión de almacenamiento principal, puede desencadenar el cambio a **PrimaryOnly** y continuar permitiendo las actualizaciones.

## <a name="handling-eventually-consistent-data"></a>Control de datos con coherencia final

El almacenamiento con redundancia geográfica funciona mediante la replicación de transacciones de la región primaria en la región secundaria. Este proceso de replicación garantiza que los datos de la región secundaria tengan *coherencia final*. Esto significa que todas las transacciones en la región primaria terminarán por aparecer en la región secundaria, aunque puede darse un retraso antes de que aparezcan y no hay ninguna garantía de que las transacciones lleguen a la región secundaria en el mismo orden en que se aplicaron originalmente en la región primaria. Si las transacciones llegan desordenadas a la región secundaria, *podría* considerar que los datos en la región secundaria se encuentran en un estado incoherente hasta que el servicio se ponga al día.

En la tabla siguiente, se muestra un ejemplo de lo que podría suceder al actualizar los detalles de un empleado para convertirlo en miembro del rol de *administrador*. Para este ejemplo, esto requiere actualizar la entidad **empleado** y actualizar una entidad **rol de administrador** con un recuento del número total de administradores. Observe cómo se aplican las actualizaciones desordenadas en la región secundaria.

| **Time** | **Transacción**                                            | **Replicación**                       | **Hora de última sincronización** | **Resultado** |
|----------|------------------------------------------------------------|---------------------------------------|--------------------|------------| 
| T0       | Transacción A: <br> Insertar entidad <br> empleado en región primaria |                                   |                    | Transacción A insertada en región primaria,<br> aún sin replicar. |
| T1       |                                                            | Transacción A <br> replicada en<br> región secundaria | T1 | Transacción A replicada en región secundaria. <br>Hora de última sincronización actualizada.    |
| T2       | Transacción B:<br>Actualizar<br> entidad empleado<br> en región primaria  |                                | T1                 | Transacción B escrita en región primaria,<br> aún sin replicar.  |
| T3       | Transacción C:<br> Actualizar <br>administrator<br>entidad rol administrador en región<br>primary |                    | T1                 | Transacción C escrita en región primaria,<br> aún sin replicar.  |
| *T4*     |                                                       | Transacción C <br>replicada en<br> región secundaria | T1         | Transacción C replicada en región secundaria.<br>LastSyncTime no actualizado porque <br>la transacción B está aún sin replicar.|
| *T5*     | Leer entidades <br>desde región secundaria                           |                                  | T1                 | Obtiene el valor obsoleto de la entidad <br> empleado porque la transacción B aún <br> no se ha replicado. Obtiene el valor nuevo para la<br> entidad rol de administrador porque C<br> se ha replicado. La hora de la última sincronización aún no se ha<br> actualizado porque la transacción B<br> no se ha replicado. Puede ver que la<br>entidad rol de administrador es incoherente <br>porque la fecha y hora de la entidad son posteriores a <br>Hora de última sincronización. |
| *T6*     |                                                      | Transacción B<br> replicada en<br> región secundaria | T6                 | *T6*: todas las transacciones hasta la C <br>se han replicado, Hora de última sincronización<br> se ha actualizado. |

En este ejemplo, suponga que el cliente pasa a leer desde la región secundaria en T5. Puede leer correctamente la entidad **rol administrador** en este momento, pero la entidad contiene un valor para el recuento de administradores que no es coherente con el número de entidades **empleado** que están marcadas como administradores en la región secundaria en este momento. El cliente podría mostrar simplemente este valor y correr el riesgo de que sea información incoherente. O bien, el cliente podría intentar determinar que el **rol administrador** se encuentra en un estado potencialmente incoherente porque las actualizaciones se han producido desordenadas y después informar al usuario sobre este hecho.

Para reconocer que tiene datos potencialmente incoherentes, el cliente puede usar el valor de *Hora de última sincronización*, que puede obtener en cualquier momento al consultar un servicio de almacenamiento. Esto indica la hora en que los datos de la región secundaria fueron coherentes por última vez y cuándo aplicó el servicio todas las transacciones hasta ese momento. En el ejemplo mostrado antes, una vez que el servicio inserta la entidad **empleado** en la región secundaria, la hora de la última sincronización se establece en *T1*. Permanece en *T1* hasta que el servicio actualiza la entidad **empleado** en la región secundaria cuando se establece en *T6*. Si el cliente recupera la última hora de sincronización cuando lee la entidad en *T5*, puede compararla con la marca de tiempo en la entidad. Si la marca de tiempo en la entidad es posterior a la hora de la última sincronización, la entidad se encuentra en un estado potencialmente incoherente, por lo que podrá llevar a cabo la acción adecuada para su aplicación. Para usar este campo, es necesario saber cuándo se completó la última actualización en la región primaria.

Para aprender a consultar la hora de la última sincronización, vea [Comprobación de la propiedad Hora de la última sincronización de una cuenta de almacenamiento](last-sync-time-get.md).

## <a name="testing"></a>Prueba

Es importante comprobar que la aplicación se comporte según lo esperado cuando encuentre errores con posibilidad de reintento. Por ejemplo, debe probar que la aplicación cambie a la región secundaria y al modo de solo lectura cuando se detecte un problema y que cambie de vuelta a la región primaria cuando vuelva a estar disponible. Para ello, necesita una manera de simular errores con posibilidad de reintento y controlar la frecuencia con que se producen.

Puede usar [Fiddler](https://www.telerik.com/fiddler) para interceptar y modificar respuestas HTTP en un script. Este script puede identificar las respuestas que proceden de su punto de conexión principal y cambiar el código de estado HTTP para que la biblioteca del cliente de almacenamiento lo reconozca como error con posibilidad de reintento. Este fragmento de código muestra un ejemplo sencillo de un script de Fiddler que intercepta respuestas a solicitudes de lectura para la tabla **employeedata** y devuelve un estado 502:

```java
static function OnBeforeResponse(oSession: Session) {
    ...
    if ((oSession.hostname == "\[yourstorageaccount\].table.core.windows.net")
      && (oSession.PathAndQuery.StartsWith("/employeedata?$filter"))) {
        oSession.responseCode = 502;
    }
}
```

Puede ampliar este ejemplo para interceptar una gama más amplia de solicitudes y solamente cambiar el elemento **responseCode** en algunas de ellas para simular mejor un escenario real. Para más información sobre cómo personalizar scripts de Fiddler, consulte [Modifying a Request or Response](https://docs.telerik.com/fiddler/KnowledgeBase/FiddlerScript/ModifyRequestOrResponse) (Modificación de una solicitud o respuesta) en la documentación de Fiddler.

Si ha hecho configurables los umbrales para cambiar la aplicación al modo de solo lectura, será más fácil probar el comportamiento con volúmenes de transacciones que no sean de producción.

## <a name="next-steps"></a>Pasos siguientes

Para ver un ejemplo completo de cómo cambiar entre el punto de conexión principal y el secundario, consulte [Ejemplos de Azure: uso del patrón de interruptor con almacenamiento de RA-GRS](https://github.com/Azure-Samples/storage-dotnet-circuit-breaker-pattern-ha-apps-using-ra-grs).

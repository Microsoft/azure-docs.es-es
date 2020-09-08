---
title: Alta disponibilidad y recuperación ante desastres de Azure IoT Hub | Microsoft Docs
description: Describe las características de Azure e IoT Hub que lo ayudarán a crear soluciones de IoT de Azure de alta disponibilidad con funcionalidades de recuperación ante desastres.
author: jlian
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 03/17/2020
ms.author: philmea
ms.openlocfilehash: d4a5ad36e9d6d71ad88d0b5c56b6079f34483347
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89021439"
---
# <a name="iot-hub-high-availability-and-disaster-recovery"></a>Alta disponibilidad y recuperación ante desastres de IoT Hub

Como primer paso para implementar una solución IoT resistente, los arquitectos, desarrolladores y propietarios de empresas deben definir los objetivos de tiempo de actividad para las soluciones que vayan a crear. Estos objetivos se pueden definir principalmente en función de los objetivos de negocio específicos para cada escenario. En este contexto, en el artículo de [orientación técnica sobre la continuidad empresarial de Azure](https://docs.microsoft.com/azure/architecture/resiliency/) se describe un marco general para ayudar a reflexionar sobre la continuidad empresarial y recuperación ante desastres. El documento [Recuperación ante desastres y alta disponibilidad para aplicaciones de Azure](https://docs.microsoft.com/azure/architecture/reliability/disaster-recovery) proporciona una guía de arquitectura enfocada a estrategias para que las aplicaciones de Azure logren alta disponibilidad y recuperación ante desastres.

En este artículo se describen las características de alta disponibilidad y recuperación ante desastres que ofrece específicamente el servicio IoT Hub. Las amplias áreas tratadas en este artículo son:

- Alta disponibilidad dentro de una región
- Recuperación ante desastres entre regiones
- Lograr alta disponibilidad entre regiones

Según los objetivos de tiempo de actividad que defina para las soluciones de IoT, debe determinar cuál de las opciones que se describen a continuación se adapta mejor a sus objetivos de negocio. La incorporación de cualquiera de estas alternativas de alta disponibilidad y recuperación ante desastres en la solución de IoT requiere una evaluación cuidadosa de las ventajas y desventajas entre:

- El nivel de resistencia que necesita 
- La complejidad de implementación y mantenimiento
- El impacto de COGS

## <a name="intra-region-ha"></a>Alta disponibilidad dentro de una región

El servicio IoT Hub proporciona alta disponibilidad dentro de una región mediante la implementación de redundancias en casi todos los niveles del servicio. El [Acuerdo de Nivel de Servicio publicado por el servicio IoT Hub](https://azure.microsoft.com/support/legal/sla/iot-hub) se consigue con el uso de estas redundancias. Los desarrolladores de una solución de IoT no deben realizar ningún trabajo adicional para poder beneficiarse de estas características de alta disponibilidad. Aunque IoT Hub ofrece una garantía de tiempo de actividad razonablemente elevada, los errores transitorios todavía se pueden esperar al igual que con cualquier plataforma de informática distribuida. Si acaba de empezar con la migración de sus soluciones a la nube desde una solución local, su enfoque debe cambiar de la optimización de "tiempo medio entre errores" a "tiempo medio para recuperación". En otras palabras, los errores transitorios se deben considerar normales al trabajar con la nube en la combinación. Se deben integrar [directivas de reintento](iot-hub-reliability-features-in-sdks.md) apropiadas en los componentes que interactúan con una aplicación en la nube para solucionar errores transitorios.

> [!NOTE]
> Algunos servicios de Azure también proporcionan niveles adicionales de disponibilidad dentro de una región mediante la integración con [zonas de disponibilidad](../availability-zones/az-overview.md). El servicio IoT Hub no admite actualmente las zonas de disponibilidad.

## <a name="cross-region-dr"></a>Recuperación ante desastres entre regiones

Podría haber algunas situaciones excepcionales cuando un centro de datos sufre interrupciones prolongadas debido a errores de alimentación u otros errores que afectan a recursos físicos. Estos eventos son poco frecuentes, durante los cuales la funcionalidad de alta disponibilidad dentro de la región que se ha descrito anteriormente no siempre resulta útil. IoT Hub ofrece varias soluciones para recuperarse de tales interrupciones prolongadas. 

Las opciones de recuperación disponibles para los clientes en tal situación son [conmutación por error iniciada por Microsoft](#microsoft-initiated-failover) y [conmutación por error manual](#manual-failover). La diferencia principal entre ambas opciones es que Microsoft inicia la primera y el usuario inicia la última. Además, la conmutación por error manual ofrece un objetivo de tiempo de recuperación (RTO) inferior en comparación con la opción de conmutación por error iniciada por Microsoft. Los objetivos de tiempo de recuperación específicos ofrecidos con cada opción se tratan en las secciones siguientes. Si se utiliza alguna de estas opciones para realizar la conmutación por error de una instancia de IoT Hub desde su región primaria, dicha instancia queda totalmente operativa en la [región de Azure emparejada geográficamente](../best-practices-availability-paired-regions.md) correspondiente.

Ambas opciones de conmutación por error ofrecen los siguientes objetivos de punto de recuperación (RPO):

| Tipo de datos | Objetivos de punto de recuperación (RPO) |
| --- | --- |
| Registro de identidad |Pérdida de datos de 0-5 minutos |
| Datos del dispositivo gemelo |Pérdida de datos de 0-5 minutos |
| Mensajes de la nube al dispositivo<sup>1</sup> |Pérdida de datos de 0-5 minutos |
| Trabajos de dispositivo y primarios<sup>1</sup> |Pérdida de datos de 0-5 minutos |
| Mensajes de dispositivo a nube |Se pierden todos los mensajes no leídos |
| Mensajes de supervisión de operaciones |Se pierden todos los mensajes no leídos |
| Mensajes de comentarios de nube a dispositivo |Se pierden todos los mensajes no leídos |

<sup>1</sup>Los mensajes de la nube al dispositivo y los trabajos primarios no se recuperan como parte de la conmutación por error manual.

Una vez completada la operación de conmutación por error para la instancia de IoT Hub, se espera que todas las operaciones de las aplicaciones de dispositivo y back-end sigan funcionando sin necesidad de una intervención manual. Esto significa que los mensajes del dispositivo a la nube deben seguir funcionando y todo el registro del dispositivo está intacto. Los eventos emitidos mediante Event Grid pueden consumirse con las mismas suscripciones configuradas anteriormente, siempre y cuando esas suscripciones de Event Grid sigan estando disponibles. No se requiere manipulación adicional para los puntos de conexión personalizados.

> [!CAUTION]
> - El nombre compatible con Event Hub y el punto de conexión de eventos integrados en IoT Hub cambian tras la conmutación por error. Cuando se reciben mensajes de telemetría desde el punto de conexión integrado mediante el cliente de Event Hub o el host del procesador de eventos, se debe [usar la cadena de conexión de IoT Hub](iot-hub-devguide-messages-read-builtin.md#read-from-the-built-in-endpoint) para establecer la conexión. Esto garantiza que las aplicaciones de back-end sigan funcionando sin necesidad de intervención manual después de la conmutación por error. Si usa el nombre y el punto de conexión compatibles con Event Hub directamente en la aplicación, tendrá que [capturar el nuevo punto de conexión compatible con Event Hub](iot-hub-devguide-messages-read-builtin.md#read-from-the-built-in-endpoint) después de la conmutación por error para continuar con las operaciones. 
>
> - Si usa Azure Functions o Azure Stream Analytics para conectar el punto de conexión de eventos integrado, puede que tenga que realizar un **reinicio**. Esto se debe a que, durante la conmutación por error, los desplazamientos anteriores ya no son válidos.
>
> - Al enrutar al almacenamiento, se recomienda enumerar los blobs o los archivos e iterar sobre ellos para garantizar que se leen todos sin pasar por alto ninguna partición. El intervalo de partición podría cambiar durante una conmutación por error iniciada por Microsoft o una conmutación por error manual. Puede usar la [API List Blobs](https://docs.microsoft.com/rest/api/storageservices/list-blobs) para la lista de blobs o la [API List ADLS Gen2](https://docs.microsoft.com/rest/api/storageservices/datalakestoragegen2/path/list) para la lista de archivos. Para obtener más información, vea [Azure Storage como punto de conexión de enrutamiento](iot-hub-devguide-messages-d2c.md#azure-storage-as-a-routing-endpoint).

## <a name="microsoft-initiated-failover"></a>Conmutación por error iniciada por Microsoft

La conmutación por error iniciada por Microsoft la ejecuta Microsoft en situaciones concretas para conmutar por error todos los centros de IoT desde una región afectada en la región emparejada geográficamente correspondiente. Este proceso es una opción predeterminada (los usuarios no pueden omitirla) y no requiere ninguna intervención por parte del usuario. Microsoft se reserva el derecho a tomar una determinación sobre cuándo ejecutar esta opción. Este mecanismo no precisa de consentimiento del usuario antes de realizar la conmutación por error del centro del usuario. La conmutación por error iniciada por Microsoft tiene un objetivo de tiempo de recuperación (RTO) de 2 a 26 horas. 

El RTO prolongado se debe a que Microsoft debe ejecutar la operación de conmutación por error en nombre de todos los clientes afectados en dicha región. Si ejecuta una solución de IoT menos crítica que pueda soportar un tiempo de inactividad de un día más o menos, entonces puede depender de esta opción para satisfacer todos los objetivos de recuperación ante desastres de la solución de IoT. El tiempo total de las operaciones de tiempo de ejecución para conseguir la operatividad total una vez desencadenado este proceso se describe en la sección "Tiempo de recuperación".

## <a name="manual-failover"></a>Conmutación por error manual

Si no se cumplen los objetivos de tiempo de actividad del negocio con el RTO que ofrece la conmutación por error iniciada por Microsoft, considere el uso de la conmutación por error manual para desencadenar el proceso de conmutación por error por sí mismo. El RTO con el uso de esta opción podría ser cualquier valor entre 10 minutos y un par de horas. Actualmente, el RTO es una función del número de dispositivos registrados en la instancia de IoT Hub en la que se realiza la conmutar por error. Puede esperar que el RTO de un centro que hospeda aproximadamente 100 000 dispositivos sea aproximadamente de 15 minutos. El tiempo total de las operaciones de tiempo de ejecución para conseguir la operatividad total una vez desencadenado este proceso se describe en la sección "Tiempo de recuperación".

La opción de conmutación por error manual siempre está disponible para su uso con independencia de si la región primaria está experimentando tiempos de inactividad o no. Por lo tanto, esta opción puede emplearse para realizar conmutaciones por error planeadas. Un ejemplo de uso de las conmutaciones por error planeadas consiste en realizar operaciones periódicas de conmutación por error. Sin embargo hay que tener la precaución de que una operación de conmutación por error planeada genera un tiempo de inactividad del centro durante el período definido por el RTO para esta opción, lo que genera también una pérdida de datos, según se define en la tabla del RPO anterior. Puede considerar la posibilidad de configurar una prueba de instancia de IoT Hub para ejecutar la opción de conmutación por error planeada periódicamente para incrementar la confiabilidad en su capacidad de ejecutar las soluciones de un extremo a otro cuando se produce un desastre real.

La conmutación por error manual está disponible sin costo adicional para los centros de IoT creados después del 18 de mayo de 2017.

Para obtener instrucciones paso a paso, consulte [Tutorial: Realización de una conmutación por error manual de una instancia de IoT Hub](tutorial-manual-failover.md)

### <a name="running-test-drills"></a>Ejecución de simulacros de prueba

Las operaciones de prueba no deben realizarse en instancias de IoT Hub que se usan en los entornos de producción.

### <a name="dont-use-manual-failover-to-migrate-iot-hub-to-a-different-region"></a>No use la conmutación por error manual para migrar IoT Hub a otra región

La conmutación por error manual *no* debe usarse como un mecanismo para migrar de forma permanente su centro entre las regiones de Azure emparejadas geográficamente. Hacerlo podría incrementar la latencia para las operaciones que se realizan en el centro de IoT desde dispositivos alojados en la región primaria anterior.

## <a name="failback"></a>Conmutación por recuperación

La conmutación por recuperación a la región primaria anterior puede conseguirse si se vuelve a desencadenar una acción de conmutación por error. Si se realizó la operación de conmutación por error original para recuperarse de una interrupción prolongada en la región primaria original, se recomienda realizar la conmutación por error del centro a la ubicación original una vez que se ha recuperado esa ubicación de la situación de interrupción.

> [!IMPORTANT]
> - A los usuarios solo se les permite realizar dos operaciones correctas de conmutación por error y dos operaciones correctas de conmutación por recuperación al día.
>
> - No se permite realizar operaciones consecutivas de conmutación por error o conmutación por recuperación. Tendrá que esperar una hora entre estas operaciones.

## <a name="time-to-recover"></a>Tiempo de recuperación

Si bien el FQDN (y, por tanto, la cadena de conexión) de la instancia de IoT Hub sigue siendo el mismo tras la conmutación por error, cambia la dirección IP subyacente. Por tanto, el tiempo total para que las operaciones de tiempo de ejecución ejecutadas en la instancia de IoT Hub estén totalmente operativas tras el inicio del proceso de conmutación por error se puede expresar con la siguiente función.

Tiempo de recuperación = RTO [10 m - 2 horas de conmutación por error manual | 2 a 26 horas de conmutación por error iniciada por Microsoft] + Retraso de propagación de DNS + Tiempo que emplea la aplicación cliente para actualizar la dirección IP de IoT Hub almacenada en caché.

> [!IMPORTANT]
> Los SDK de IoT no almacenan en caché la dirección IP de IoT Hub. Se recomienda que el código de usuario que interactúa con los SDK no almacenen en caché la dirección IP de IoT Hub.

## <a name="achieve-cross-region-ha"></a>Lograr alta disponibilidad entre regiones

Si no se cumplen los objetivos de tiempo de actividad del negocio con el RTO que ofrecen la conmutación por error iniciada por Microsoft o la conmutación por error manual, debe considerar la opción de implementar un mecanismo de conmutación por error automático entre regiones por cada dispositivo.
El objetivo de este artículo no es proporcionar una descripción completa de las topologías de implementación de soluciones de IoT. Este artículo explica el modelo de implementación de *conmutación por error regional* para la alta disponibilidad y la recuperación ante desastres.

En un modelo de conmutación por error regional, el back-end de la solución se ejecuta principalmente en una ubicación de centro de datos. En otra ubicación de centro de datos se implementan un centro de IoT y un back-end secundarios. Si el centro de IoT de la región primaria sufre una interrupción o se interrumpe la conectividad de red del dispositivo a la región primaria, los dispositivos usan un punto de conexión de servicio secundario. Puede mejorar la disponibilidad de la solución mediante la implementación de un modelo de conmutación por error entre regiones en lugar de permanecer dentro de una única región. 

A grandes rasgos, para implementar un modelo de conmutación por error regional con IoT Hub, necesitará seguir estos pasos:

* **Un centro de IoT secundario y lógica de enrutamiento de dispositivo**: si se produce una interrupción del servicio en la región primaria, los dispositivos deben empezar a conectarse a la región secundaria. Como se conoce el estado de la mayoría de los servicios implicados, es habitual que los administradores de la solución desencadenen el proceso de conmutación por error entre regiones. La mejor forma de comunicar el nuevo punto de conexión con los dispositivos, sin perder el control del proceso, es hacer que comprueben periódicamente un servicio de *conserje* para el punto de conexión activo actual. El servicio de conserje puede ser una aplicación web simple que se replica y se mantiene accesible mediante técnicas de redirección de DNS (por ejemplo, con [Azure Traffic Manager](../traffic-manager/traffic-manager-overview.md)).

   > [!NOTE]
   > El servicio IoT Hub no es un tipo de punto de conexión admitido en Azure Traffic Manager. Se recomienda integrar el servicio de soporte propuesto con Azure Traffic Manager haciendo que implemente la API de sondeo de estado del punto de conexión.

* **Replicación del registro de identidad**: para que el centro de IoT secundario pueda usarse, debe contener todas las identidades de dispositivo que se pueden conectar a la solución. La solución debe conservar copias de seguridad de replicación geográfica de las identidades de dispositivo y cargarlas en el Centro de IoT secundario antes de cambiar el punto de conexión activo de los dispositivos. La funcionalidad de exportación de identidades de dispositivo de IoT Hub resulta muy útil en este contexto. Para obtener más información, consulte la [Guía para desarrolladores de IoT Hub: Registro de identidades](iot-hub-devguide-identity-registry.md).

* **Lógica de combinación**: cuando la región primaria vuelve a estar disponible, todos los estados y datos que se crearon en el sitio secundario deben volver a migrarse a la región primaria. Este estado y estos datos tienen que ver principalmente con las identidades de dispositivo y los metadatos de aplicación, que deben combinarse con el centro de IoT principal y con otros almacenes específicos de la aplicación en la región primaria. 

Para simplificar este paso, debe usar las operaciones de idempotente. Estas minimizan los efectos secundarios de la posible distribución uniforme de eventos y de los duplicados o la entrega desordenada de eventos. Además, la lógica de aplicación debe diseñarse para que tolere posibles incoherencias o un estado ligeramente desactualizado. Esta situación se puede producir debido al tiempo adicional que tarda el sistema en repararse según los objetivos de punto de recuperación (RPO).

## <a name="choose-the-right-hadr-option"></a>Elección de la opción correcta de alta disponibilidad/recuperación ante desastres

A continuación le mostramos un resumen de las opciones de alta disponibilidad y recuperación ante desastres presentadas en este artículo, que puede utilizarse como un marco de referencia para elegir la opción correcta para su solución.

| Opción de alta disponibilidad/recuperación ante desastres | RTO | RPO | ¿Requiere intervención manual? | Complejidad de la implementación | Implicación de costos adicionales|
| --- | --- | --- | --- | --- | --- |
| Conmutación por error iniciada por Microsoft |2 - 26 horas|Consulte la tabla de RPO anterior|No|None|None|
| Conmutación por error manual |10 m - 2 horas|Consulte la tabla de RPO anterior|Sí|Muy baja. Solo necesita desencadenar esta operación desde el portal.|None|
| Alta disponibilidad entre regiones |< 1 m|Depende de la frecuencia de replicación de la solución de alta disponibilidad personalizada|No|Alto|> 1 x el costo de 1 instancia de IoT Hub|

## <a name="next-steps"></a>Pasos siguientes

* [¿Qué es Azure IoT Hub?](about-iot-hub.md)
* [Introducción a los centros de IoT (inicio rápido)](quickstart-send-telemetry-dotnet.md)
* [Tutorial: Realización de una conmutación por error manual de una instancia de IoT Hub](tutorial-manual-failover.md)

---
title: Información de la limitación y las cuotas de IoT de Azure | Microsoft Docs
description: 'Guía del desarrollador: descripción de las cuotas que se aplican a IoT Hub y comportamiento esperado de limitación'
author: robinsh
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/08/2019
ms.custom:
- 'Role: Cloud Development'
- 'Role: Operations'
- 'Role: Technical Support'
ms.openlocfilehash: ec8d277de177942386d9f3becdf329cdff1ca812
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88797809"
---
# <a name="reference---iot-hub-quotas-and-throttling"></a>Referencia: Cuotas y limitación de IoT Hub

En este artículo se explican las cuotas para una instancia de IoT Hub y se proporciona información para ayudarle a entender cómo funciona la limitación.

## <a name="quotas-and-throttling"></a>Cuotas y limitación

Cada suscripción de Azure puede tener como máximo cincuenta centros de IoT y al menos un centro gratuito.

Cada instancia de IoT Hub se aprovisiona con un determinado número de unidades en un nivel determinado. El nivel y el número de unidades determinan la cuota diaria máxima de mensajes que puede enviar. El tamaño de mensaje que se usa para calcular la cuota diaria es 0,5 KB para un centro de nivel gratuito y de 4 KB para todos los otros niveles. Para más información, consulte [Precios de Azure IoT Hub](https://azure.microsoft.com/pricing/details/iot-hub/).

El nivel también determina los valores de limitación que aplica Azure IoT Hub a las operaciones.

## <a name="iot-plug-and-play"></a>IoT Plug and Play

Los dispositivos IoT Plug and Play envían al menos un mensaje de telemetría para cada interfaz, incluida la raíz, lo que puede aumentar el número de mensajes considerados en la cuota de mensajes.

## <a name="operation-throttles"></a>Limitaciones de operación

Las limitaciones de operación son las limitaciones de velocidad que se aplican en los intervalos de minutos y están diseñadas para evitar abusos. También están sujetas a [modelado del tráfico](#traffic-shaping).

En la tabla siguiente se muestran las limitaciones exigidas. Los valores hacen referencia a un centro individual.

| Limitación | Gratis, S1 y B1 | B2 y S2 | B3 y S3 | 
| -------- | ------- | ------- | ------- |
| [Operaciones de registro de identidad](#identity-registry-operations-throttle) (crear, recuperar, enumerar, actualizar y eliminar) | 1,67/s/unidad (100/min/unidad) | 1,67/s/unidad (100/min/unidad) | 83,33/s/unidad (5000/m/unidad) |
| [Nuevas conexiones de dispositivo](#device-connections-throttle) (este límite se aplica a la velocidad de las _nuevas conexiones_, no al número total de conexiones) | Mayor de 100/s o 12/s/unidad <br/> Por ejemplo, dos unidades S1 son 2\*12 = 24 nuevas conexiones por segundo, pero dispone de al menos 100 nuevas conexiones por segundo entre todas las unidades. Con nueve unidades S1 tiene 108 nuevas conexiones por segundo (9\*12) entre todas las unidades. | 120 conexiones nuevas por segundo por unidad | 6000 conexiones nuevas por segundo por unidad |
| Envíos de dispositivo a nube | Más de 100 operaciones de envío por segundo o 12 operaciones de envío por segundo por unidad <br/> Por ejemplo, dos unidades S1 equivalen a 2\*12 = 24/s, pero tendrá al menos 100 operaciones de envío por segundo en todas las unidades. Con nueve unidades S1, tiene 108 nuevas operaciones de envío por segundo (9\*12) entre todas las unidades. | 120 operaciones de envío por segundo por unidad | 6000 operaciones de envío por segundo por unidad |
| Envíos de nube a dispositivo<sup>1</sup> | 1,67 operaciones de envío por segundo por unidad (100 mensajes por minuto por unidad) | 1,67 operaciones de envío por segundo por unidad (100 operaciones de envío por minuto por unidad) | 83,33 operaciones de envío por segundo por unidad (5000 operaciones de envío por minuto por unidad) |
| Recepciones de nube a dispositivo<sup>1</sup> <br/> (solo cuando el dispositivo usa HTTPS)| 16,67 operaciones de envío por segundo por unidad (1000 operaciones de recepción por minuto por unidad) | 16,67 operaciones de envío por segundo por unidad (1000 operaciones de recepción por minuto por unidad) | 833,33 operaciones de envío por segundo por unidad (50 000 operaciones de recepción por minuto por unidad) |
| Carga de archivos | 1,67 inicios de carga de archivos por segundo por unidad (100 por minuto por unidad) | 1,67 inicios de carga de archivos por segundo por unidad (100 por minuto por unidad) | 83,33 inicios de carga de archivos por segundo por unidad (5000 por minuto por unidad) |
| Métodos directos<sup>1</sup> | 160 MB/s/unidad<sup>2</sup> | 480 MB/s/unidad<sup>2</sup> | 24 MB/s/unidad<sup>2</sup> | 
| Consultas | 20/m/unidad | 20/m/unidad | 1000/m/unidad |
| Lecturas de (dispositivos y módulos) gemelos<sup>1</sup> | 100/s | Mayor de 100/s o 10/s/unidad | 500/s/unidad |
| Actualizaciones de (dispositivos y módulos) gemelos<sup>1</sup> | 50/s | Mayor de 50/s o 5/s/unidad | 250/s/unidad |
| Operaciones de trabajos<sup>1</sup> <br/> (crear, actualizar, enumerar, eliminar) | 1,67/s/unidad (100/min/unidad) | 1,67/s/unidad (100/min/unidad) | 83,33/s/unidad (5000/m/unidad) |
| Operaciones de dispositivo de trabajos<sup>1</sup> <br/> (actualizar gemelos, invocar método directo) | 10/s | Mayor de 10/s o 1/s/unidad | 50/s/unidad |
| Configuraciones e implementaciones perimetrales<sup>1</sup> <br/> (crear, actualizar, enumerar, eliminar) | 0,33/s/unidad (20/m/unidad) | 0,33/s/unidad (20/m/unidad) | 0,33/s/unidad (20/m/unidad) |
| Velocidad de iniciación de flujo de dispositivos<sup>1</sup> | 5 nuevas secuencias/s | 5 nuevas secuencias/s | 5 nuevas secuencias/s |
| Número máximo de flujos de dispositivos conectados simultáneamente<sup>1</sup> | 50 | 50 | 50 |
| Nivel máximo de transferencia de datos de flujo de dispositivos<sup>1</sup> (volumen agregado por día) | 300 MB | 300 MB | 300 MB |

<sup>1</sup>Esta característica no está disponible en el nivel básico de IoT Hub. Para más información, consulte [Elección de la instancia de IoT Hub correcta](iot-hub-scaling.md). <br/><sup>2</sup>El tamaño del medidor de limitación es de 4 KB.

### <a name="throttling-details"></a>Detalles de la limitación

* El tamaño del medidor determina en qué incrementos se consume el límite de la limitación. Si la carga de la llamada directa está entre 0 y 4 KB, se cuentan como 4 KB. Puede realizar hasta 40 llamadas por segundo por unidad antes de alcanzar el límite de 160 KB/s/unidad.

   Del mismo modo, si la carga se encuentra entre 4 y 8 KB, cada llamada tiene 8 KB y puede realizar hasta 20 llamadas por segundo por unidad antes de alcanzar el límite máximo.

   Por último, si el tamaño de la carga se encuentra entre 156 y 160 KB, solo podrá realizar una llamada por segundo por unidad en el centro antes de alcanzar el límite de 160 KB/s/unidad.

*  Para *operaciones de dispositivos de trabajo (actualizar dispositivo, invocar método directo)* para el nivel S2, el límite de 50/s/unidad solo se aplica cuando invoca métodos mediante trabajos. Si invoca directamente métodos directos, se aplica el límite original de 24 MB/s/unidad (para S2).

*  **Cuota** es el número agregado de mensajes que se pueden enviar en el centro *por día*. Puede encontrar el límite de cuota del centro en la columna **número total de mensajes/día** en la [página de precios de IoT Hub](https://azure.microsoft.com/pricing/details/iot-hub/).

*  Sus limitaciones de la nube al dispositivo y del dispositivo a la nube determinan la *velocidad máxima* a la que puede enviar mensajes: número de mensajes con independencia de los fragmentos de 4 KB. Cada mensaje puede tener hasta 256 KB, que es el [tamaño máximo del mensaje](iot-hub-devguide-quotas-throttling.md#other-limits).

*  Se recomienda limitar las llamadas para que no alcancen o superen los límites. Si alcanza el límite, IoT Hub responde con el código de error 429 y el cliente debería interrumpirse y volver a iniciarse. Estos límites son por centro (o en algunos casos por centro por unidad). Para más información, consulte [Administración de la conectividad y de los patrones de mensajería confiable y de reintento](iot-hub-reliability-features-in-sdks.md#retry-patterns).

### <a name="traffic-shaping"></a>Modelado del tráfico

Para dar cabida al tráfico por ráfagas, IoT Hub acepta solicitudes por encima de la limitación durante un tiempo limitado. Las primeras solicitudes se procesan inmediatamente. Pero si el número de solicitudes sigue infringiendo la limitación, IoT Hub empieza a colocar las solicitudes en una cola y las procesa según la tarifa límite de IoT Hub. Este efecto se conoce como *modelado del tráfico*. Además, el tamaño de esta cola es limitado. Si se sigue produciendo una infracción de limitación, la cola acaba llenándose y IoT Hub empieza a rechazar solicitudes con `429 ThrottlingException`.

Por ejemplo, imagine que usa un dispositivo simulado para enviar 200 mensajes de dispositivo a nube por segundo a S1 IoT Hub (que tiene un límite 100/s envíos D2C.). Durante el primer minuto o dos, los mensajes se procesan inmediatamente. Pero como el dispositivo sigue enviando más mensajes que el límite, IoT Hub empieza a procesar solo 100 mensajes por segundo y pone el resto en cola. Se empieza a notar una mayor latencia. Por último, empieza a recibir `429 ThrottlingException` a medida que se llena la cola y el "número de errores de limitación" en [métricas de IoT Hub](iot-hub-metrics.md) empieza a aumentar.

### <a name="identity-registry-operations-throttle"></a>Limitación de operaciones de registro de identidad

Las operaciones de registro de identidad de dispositivo están diseñadas para usarse en tiempo de ejecución en escenarios de administración y aprovisionamiento de dispositivos. La lectura o actualización de un gran número de identidades de dispositivo se realiza mediante [trabajos de importación y exportación](iot-hub-devguide-identity-registry.md#import-and-export-device-identities).

### <a name="device-connections-throttle"></a>Limitación de conexiones de dispositivo

La limitación de las *conexiones de dispositivo* determina la velocidad a la que se pueden establecer nuevas conexiones de dispositivo con una instancia de IoT Hub. La limitación de las *conexiones de dispositivo* no controla el número máximo de dispositivos conectados a la vez. La limitación de *conexiones de dispositivo* depende del número de unidades aprovisionadas para el centro de IoT.

Por ejemplo, si compra una sola unidad S1, tendrá una limitación de 100 conexiones por segundo. Así que para conectar 100 000 dispositivos, se tarda al menos 1000 segundos (aproximadamente 16 minutos). Sin embargo, puede tener el mismo número de dispositivos conectados al mismo tiempo que de dispositivos registrados en el registro de identidad.

## <a name="other-limits"></a>Otros límites

IoT Hub exige otros límites operativos:

| Operación | Límite |
| --------- | ----- |
| Dispositivos | El número total de dispositivos más módulos que se pueden registrar en un único centro de IoT se limita a 1 millón. La única manera de aumentar este límite es ponerse en contacto con el [soporte técnico de Microsoft](https://azure.microsoft.com/support/options/).|
| Cargas de archivos | 10 cargas de archivos simultáneas por dispositivo. |
| Trabajos<sup>1</sup> | El número máximo de trabajos simultáneos es 1 para gratuitos y S1, 5 para S2 y 10 para S3. No obstante, el número máximo de [trabajos de importación/exportación de dispositivo](iot-hub-bulk-identity-mgmt.md) simultáneos es 1 para todos los niveles. <br/>El historial de trabajos se conserva durante 30 días como máximo. |
| Puntos de conexión adicionales | Los centros de SKU de pago pueden tener 10 puntos de conexión adicionales. Los centros de SKU gratis pueden tener un punto de conexión adicional. |
| Consultas de enrutamiento de mensajes | Los centros de SKU de pago pueden tener 100 consultas de enrutamiento. Los centros de SKU gratis pueden tener cinco consultas de enrutamiento. |
| Enriquecimientos de mensajes | Los centros de SKU de pago pueden tener hasta 10 enriquecimientos de mensajes. Los centros de SKU gratis pueden tener hasta dos enriquecimientos de mensajes.|
| Mensajería de un dispositivo a la nube | Tamaño máximo del mensaje 256 KB |
| Mensajería de la nube a un dispositivo<sup>1</sup> | Tamaño máximo del mensaje 64 KB. El número máximo de mensajes pendientes para la entrega es 50 por dispositivo. |
| Método directo<sup>1</sup> | El tamaño de carga máximo del método directo es de 128 KB. |
| Configuraciones automáticas de dispositivos y módulos<sup>1</sup> | 100 configuraciones por centro de SKU de pago. 20 configuraciones por centro de SKU gratis. |
| Implementaciones automáticas de IoT Edge<sup>1</sup> | 50 módulos por implementación. 100 implementaciones (incluidas las implementaciones superpuestas) por centro de SKU de pago. 10 implementaciones por centro de SKU gratis. |
| Gemelos<sup>1</sup> | Tamaño máximo de las secciones de propiedades deseadas y propiedades notificadas son de 32 KB cada una. El tamaño máximo de la sección de etiquetas es de 8 KB. |
| Directivas de acceso compartido | El número máximo de directivas de acceso compartido es 16. |
| Certificados de entidad de certificación x509 | El número máximo de certificados de entidad de certificación x509 que se pueden registrar en una instancia de IoT Hub es 25. |

<sup>1</sup>Esta característica no está disponible en el nivel básico de IoT Hub. Para más información, consulte [Elección de la instancia de IoT Hub correcta](iot-hub-scaling.md).

## <a name="increasing-the-quota-or-throttle-limit"></a>Aumentar el límite de cuota o limitación

En cualquier momento, puede aumentar las cuotas o las limitaciones si [aumenta el número de unidades aprovisionadas en un centro de IoT](iot-hub-upgrade.md).

## <a name="latency"></a>Latencia

IoT Hub se esfuerza por proporcionar una latencia baja para todas las operaciones. Pero debido a las condiciones de la red y otros factores impredecibles no puede garantizar una determinada latencia. Cuando diseñe la solución, debería:

* Evitar realizar suposiciones sobre la latencia máxima de cualquier operación de IoT Hub.
* Aprovisionar el IoT Hub en la región de Azure más cercana a los dispositivos.
* Considere la posibilidad de usar Azure IoT Edge para realizar operaciones sensibles a la latencia en el dispositivo o en una puerta de enlace próxima al dispositivo.

Muchas unidades de IoT Hub afectan a la limitación, tal como se ha descrito anteriormente, pero no proporcionan ventajas ni garantías de latencia adicionales.

Si ve aumentosinesperados de la latencia de operación, póngase en contacto con el [Soporte técnico de Microsoft](https://azure.microsoft.com/support/options/).

## <a name="next-steps"></a>Pasos siguientes

Consulte la entrada de blog [IoT Hub throttling and you](https://azure.microsoft.com/blog/iot-hub-throttling-and-you/) (Limitación de IoT Hub) para ver una explicación detallada del comportamiento de limitación de IoT Hub.

Otros temas de referencia en la Guía del desarrollador de IoT Hub son:

* [Puntos de conexión de IoT Hub](iot-hub-devguide-endpoints.md)

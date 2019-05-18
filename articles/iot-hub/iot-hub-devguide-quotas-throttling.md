---
title: Información de la limitación y las cuotas de IoT de Azure | Microsoft Docs
description: 'Guía del desarrollador: descripción de las cuotas que se aplican a IoT Hub y comportamiento esperado de limitación'
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 05/11/2019
ms.openlocfilehash: ea9bea8b314d00db87ad7addacc49a976e0da08e
ms.sourcegitcommit: f013c433b18de2788bf09b98926c7136b15d36f1
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65550467"
---
# <a name="reference---iot-hub-quotas-and-throttling"></a>Referencia: Cuotas y limitación de IoT Hub

## <a name="quotas-and-throttling"></a>Cuotas y limitación

Cada suscripción de Azure puede tener como máximo cincuenta centros de IoT y al menos un centro gratuito.

Cada instancia de IoT Hub se aprovisiona con un determinado número de unidades en un nivel determinado. El nivel y el número de unidades determinan la cuota diaria máxima de mensajes que puede enviar. El tamaño de mensaje que se usa para calcular la cuota diaria es 0,5 KB para un centro de nivel gratuito y de 4 KB para todos los otros niveles. Para más información, consulte [Precios de Azure IoT Hub](https://azure.microsoft.com/pricing/details/iot-hub/).

El nivel también determina los valores de limitación que aplica Azure IoT Hub a las operaciones.

## <a name="operation-throttles"></a>Limitaciones de operación

Limitaciones de operación son las limitaciones de velocidad que se aplican en los intervalos de minutos y están diseñadas para evitar abusos. También están sujetos a [catalogación del tráfico](#traffic-shaping).

En la tabla siguiente se muestran las limitaciones exigidas. Los valores hacen referencia a un centro individual.

| Limitación | Gratis, S1 y B1 | B2 y S2 | B3 y S3 | 
| -------- | ------- | ------- | ------- |
| [Las operaciones de registro de identidad](#identity-registry-operations-throttle) (crear, recuperar, enumerar, actualizar y eliminar) | 1,67/s/unidad (100/min/unidad) | 1,67/s/unidad (100/min/unidad) | 83.33/SEC/Unit (5000/min/unidad) |
| [Nuevas conexiones de dispositivo](#device-connections-throttle) (este límite se aplica a la tasa de _nuevas conexiones_, no el número total de conexiones) | Mayor de 100/s o 12/s/unidad <br/> Por ejemplo, dos unidades S1 son 2\*12 = 24 nuevas conexiones por segundo, pero dispone de al menos 100 nuevas conexiones por segundo entre todas las unidades. Con nueve unidades S1 tiene 108 nuevas conexiones por segundo (9\*12) entre todas las unidades. | 120 conexiones nuevas por segundo por unidad | 6000 conexiones/s/unidad nueva |
| Envíos de dispositivo a nube | Mayor de 100/s o 12/s/unidad <br/> Por ejemplo, dos unidades S1 equivalen a 2\*12 = 24/s, pero tendrá al menos 100/s en todas las unidades. Con nueve unidades S1 tiene 108/s (9\*12) en las unidades. | 120/s/unidad | 6000/s/unidad |
| Envíos de nube a dispositivo<sup>1</sup> | 1,67/s/unidad (100/min/unidad) | 1,67/s/unidad (100/min/unidad) | 83.33/SEC/Unit (5000/min/unidad) |
| Recepciones de nube a dispositivo<sup>1</sup> <br/> (solo cuando el dispositivo usa HTTPS)| 16.67/SEC/Unit (1000/min/unidad) | 16.67/SEC/Unit (1000/min/unidad) | 833.33/SEC/Unit (50000/min/unidad) |
| Carga de archivo | 1,67 notificaciones de cargas de archivos/s/unidad (100/min/unidad) | 1,67 notificaciones de cargas de archivos/s/unidad (100/min/unidad) | 83,33 archivos carga notificaciones/s/unidad (5000/min/unidad) |
| Métodos directos<sup>1</sup> | 160 MB/s/unidad<sup>2</sup> | 480 MB/s/unidad<sup>2</sup> | 24 MB/s/unidad<sup>2</sup> | 
| Consultas | 20/min/unidad | 20/min/unidad | 1000/min/unidad |
| Lecturas de (dispositivos y módulos) gemelos<sup>1</sup> | 100/seg. | Mayor de 100/s o 10/s/unidad | 500/s/unidad |
| Actualizaciones de (dispositivos y módulos) gemelos<sup>1</sup> | 50/seg. | Mayor de 50/s o 5/s/unidad | 250/s/unidad |
| Operaciones de trabajos<sup>1</sup> <br/> (crear, actualizar, enumerar, eliminar) | 1,67/s/unidad (100/min/unidad) | 1,67/s/unidad (100/min/unidad) | 83.33/SEC/Unit (5000/min/unidad) |
| Operaciones de dispositivo de trabajos<sup>1</sup> <br/> (actualizar gemelos, invocar método directo) | 10/s | Mayor de 10/s o 1/s/unidad | 50/s/unidad |
| Configuraciones e implementaciones perimetrales<sup>1</sup> <br/> (crear, actualizar, enumerar, eliminar) | 0,33/s/unidad (20/m/unidad) | 0,33/s/unidad (20/m/unidad) | 0,33/s/unidad (20/m/unidad) |
| Velocidad de iniciación de flujo de dispositivo<sup>1</sup> | 5 nuevas secuencias/s | 5 nuevas secuencias/s | 5 nuevas secuencias/s |
| Número máximo de dispositivos conectados simultáneamente secuencias<sup>1</sup> | 50 | 50 | 50 |
| Transferencia de datos de secuencia máxima del dispositivo<sup>1</sup> (Agregar volumen por día) | 300 MB | 300 MB | 300 MB |

<sup>1</sup>Esta característica no está disponible en el nivel básico de IoT Hub. Para más información, consulte [Elección de la instancia de IoT Hub correcta](iot-hub-scaling.md). <br/><sup>2</sup>limitación de tamaño del medidor es de 4 KB.

### <a name="traffic-shaping"></a>Modelado de tráfico

Para dar cabida a tráfico por ráfagas, IoT Hub acepta las solicitudes por encima de la limitación durante un tiempo limitado. Las primeras de estas solicitudes se procesan inmediatamente. Sin embargo, si se mantiene el número de solicitudes infringe la limitación, se inicia colocando las solicitudes en una cola y procesa según la tarifa de límite de IoT Hub. Este efecto se denomina *catalogación del tráfico*. Además, el tamaño de esta cola es limitado. Si la infracción de limitación persiste, finalmente, la cola se llena y IoT Hub se inicia rechazando las solicitudes con `429 ThrottlingException`. 

Por ejemplo, usa un dispositivo simulado para enviar mensajes de dispositivo a nube 200 por segundo a S1 IoT Hub (que tiene un límite de envíos de D2C 100/seg.). Durante el primer minuto o dos, los mensajes se procesan inmediatamente. Sin embargo, puesto que el dispositivo sigue enviar más mensajes que el límite del acelerador, IoT Hub comienza a solo proceso 100 mensajes por segundo y coloca el resto de una cola. Empezar a observar una mayor latencia. Finalmente, empezará a recibir `429 ThrottlingException` como la cola se llena rápidamente y el "número de errores de limitación" en [métricas del centro de IoT](iot-hub-metrics.md) comienza a aumentar.

### <a name="identity-registry-operations-throttle"></a>Limitación las operaciones de registro de identidad

Las operaciones de registro de identidad de dispositivo están diseñadas para su uso de tiempo de ejecución en escenarios de aprovisionamiento y administración de dispositivos. La lectura o actualización de un gran número de identidades de dispositivo se realiza mediante [trabajos de importación y exportación](iot-hub-devguide-identity-registry.md#import-and-export-device-identities).

### <a name="device-connections-throttle"></a>Limitación las conexiones del dispositivo

La limitación de las *conexiones de dispositivo* determina la velocidad a la que se pueden establecer nuevas conexiones de dispositivo con una instancia de IoT Hub. La limitación de las *conexiones de dispositivo* no controla el número máximo de dispositivos conectados a la vez. La limitación de *conexiones de dispositivo* depende del número de unidades aprovisionadas para el centro de IoT.

Por ejemplo, si compra una sola unidad S1, tendrá una limitación de 100 conexiones por segundo. Por lo tanto, para conectar 100,, 000 dispositivos, se tarda al menos 1000 segundos (aproximadamente 16 minutos). Sin embargo, puede tener el mismo número de dispositivos conectados al mismo tiempo que de dispositivos registrados en el registro de identidad.


## <a name="other-limits"></a>Otros límites

IoT Hub exige otros límites operativos:

| Operación | Límite |
| --------- | ----- |
| Dispositivos | El número máximo de dispositivos que pueden conectarse a un único centro de IoT es 1.000.000. La única manera de aumentar este límite es ponerse en contacto con [Microsoft Support](https://azure.microsoft.com/support/options/).| 
| URI de carga de archivos | 10000 URI de SAS puede estar fuera de una cuenta de almacenamiento al mismo tiempo. <br/>  10 URI/dispositivo de SAS puede estar fuera al mismo tiempo. |
| Trabajos<sup>1</sup> | Máximo de trabajos simultáneos es 1 (para gratuitos y S1), 5 (para S2) y 10 (para S3). Sin embargo, simultáneos max [trabajos de import/export de dispositivo](iot-hub-bulk-identity-mgmt.md) es 1 para todos los niveles. <br/>Historial de trabajos se conserva hasta 30 días. |
| Puntos de conexión adicionales | Los centros de SKU de pago pueden tener 10 puntos de conexión adicionales. Los centros de SKU gratis pueden tener un punto de conexión adicional. |
| Reglas de enrutamiento de mensajes | Los centros de SKU de pago pueden tener 100 reglas de enrutamiento. Los centros de SKU gratis pueden tener cinco reglas de enrutamiento. |
| Mensajería de un dispositivo a la nube | Tamaño máximo del mensaje 256 KB |
| Mensajería de la nube a un dispositivo<sup>1</sup> | Tamaño máximo del mensaje 64 KB. El número máximo de mensajes pendientes para la entrega es 50. |
| Método directo<sup>1</sup> | El tamaño de carga máximo del método directo es de 128 KB. |
| Configuraciones automáticas de dispositivos<sup>1</sup> | 100 configuraciones por centro de SKU de pago. 20 configuraciones por centro de SKU gratis. |
| Implementaciones automáticas de Edge<sup>1</sup> | 20 módulos por implementación. 100 implementaciones por centro de SKU de pago. 20 implementaciones por centro de SKU gratis. |
| Gemelos<sup>1</sup> | El tamaño máximo por sección de gemelos (etiquetas, propiedades deseadas y propiedades notificadas) es de 8 KB. |

<sup>1</sup>Esta característica no está disponible en el nivel básico de IoT Hub. Para más información, consulte [Elección de la instancia de IoT Hub correcta](iot-hub-scaling.md).

## <a name="increasing-the-quota-or-throttle-limit"></a>Aumentar el límite de cuota o limitación

En un momento dado, puede aumentar las cuotas o limitar los límites por [aumentar el número de unidades aprovisionadas en un centro de IoT](iot-hub-upgrade.md).

## <a name="latency"></a>Latencia

IoT Hub se esfuerza por proporcionar una latencia baja para todas las operaciones. Sin embargo, debido a condiciones de red y otros factores impredecibles no puede garantizar una latencia determinada. Cuando diseñe la solución, debería:

* Evitar realizar suposiciones sobre la latencia máxima de cualquier operación de IoT Hub.
* Aprovisionar el IoT Hub en la región de Azure más cercana a los dispositivos.
* Considere la posibilidad de usar Azure IoT Edge para realizar operaciones sensibles a la latencia en el dispositivo o en una puerta de enlace próxima al dispositivo.

Muchas unidades de IoT Hub afectan a la limitación, tal como se ha descrito anteriormente, pero no proporcionan ventajas ni garantías de latencia adicionales.

Si ve aumentos inesperados de la latencia de operación, póngase en contacto con el [Soporte técnico de Microsoft](https://azure.microsoft.com/support/options/).

## <a name="next-steps"></a>Pasos siguientes

Consulte la entrada de blog [IoT Hub throttling and you](https://azure.microsoft.com/blog/iot-hub-throttling-and-you/) (Limitación de IoT Hub) para ver una explicación detallada del comportamiento de limitación de IoT Hub.

Otros temas de referencia en la Guía del desarrollador de IoT Hub son:

* [Puntos de conexión de IoT Hub](iot-hub-devguide-endpoints.md)

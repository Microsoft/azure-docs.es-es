---
title: Escalado de Azure IoT Hub | Microsoft Docs
description: Cómo escalar el centro de IoT Hub para respaldar el rendimiento de mensajes previsto y características deseadas. Incluye un resumen del rendimiento admitido para cada nivel y opciones de particionamiento.
author: wesmc7777
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 06/28/2019
ms.author: wesmc
ms.custom:
- amqp
- mqtt
- 'Role: Cloud Development'
- 'Role: Operations'
ms.openlocfilehash: 4ded10d9475c363c3f44032c7aa8cc7db03ab37f
ms.sourcegitcommit: 3fc3457b5a6d5773323237f6a06ccfb6955bfb2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90033649"
---
# <a name="choose-the-right-iot-hub-tier-for-your-solution"></a>Elección del nivel adecuado de IoT Hub para la solución

Cada solución de IoT es diferente, por tanto Azure IoT Hub ofrece varias opciones en función del precio y la escala. El objetivo de este artículo es ayudarle a evaluar sus necesidades de IoT Hub. Para más información sobre los niveles de IoT Hub, consulte [Precios de IoT Hub](https://azure.microsoft.com/pricing/details/iot-hub).

Para decidir qué nivel de IoT Hub es el adecuado para la solución, hágase dos preguntas:

**¿Qué características voy a utilizar?**

Azure IoT Hub ofrece dos niveles, Basic y Estándar, que se diferencian en el número de características que admiten. Si la solución de IoT se basa en la recopilación de datos de los dispositivos y en su análisis de forma centralizada, el nivel Basic es probablemente el más adecuado. Si desea utilizar configuraciones más avanzadas para controlar dispositivos IoT de forma remota o distribuir algunas de las cargas de trabajo en los mismos dispositivos, debería considerar el nivel Estándar. Para obtener un análisis detallado de las características que se incluyen en cada nivel, continúe a la sección sobre los [niveles Basic y Estándar](#basic-and-standard-tiers).

**¿Qué cantidad de datos tiene previsto mover cada día?**

Cada nivel del IoT Hub está disponible en tres tamaños, basados en la cantidad de datos que pueden tratar en un día determinado. Estos tamaños se identifican numéricamente como 1, 2 y 3. Por ejemplo, cada unidad de un centro de IoT de nivel 1 puede controlar 400 000 mensajes al día, mientras que una unidad de nivel 3 puede controlar 300 millones. Para más información acerca de las directrices de datos, vaya a la sección [Rendimiento de mensajes](#message-throughput).

## <a name="basic-and-standard-tiers"></a>Niveles Basic y Estándar

El nivel estándar de IoT Hub permite todas las características y es necesario para cualquier solución de IoT que desee hacer uso de las funcionalidades de comunicación bidireccional. El nivel Basic permite un subconjunto de las características y está pensado para las soluciones de IoT que solo necesitan comunicación unidireccional de los dispositivos a la nube. Ambos niveles ofrecen las mismas características de seguridad y autenticación.

Solo se puede elegir un tipo de [edición](https://azure.microsoft.com/pricing/details/iot-hub/) dentro de un nivel por cada instancia de IoT Hub. Por ejemplo, puede crear una instancia de IoT Hub con varias unidades de S1, pero no con una combinación de unidades de versiones distintas como, por ejemplo, S1 y S2.

| Capacidad | Nivel Basic | Nivel Estándar o Gratis |
| ---------- | ---------- | ------------- |
| [Telemetría del dispositivo a la nube](iot-hub-devguide-messaging.md) | Sí | Sí |
| [Identidad por dispositivo](iot-hub-devguide-identity-registry.md) | Sí | Sí |
| [Enrutamiento de mensajes](iot-hub-devguide-messages-read-custom.md), [enriquecimientos de mensajes](iot-hub-message-enrichments-overview.md) e [integración con Event Grid](iot-hub-event-grid.md) | Sí | Sí |
| [Protocolos HTTP, AMQP y MQTT](iot-hub-devguide-protocols.md) | Sí | Sí |
| [Servicio Device Provisioning](../iot-dps/about-iot-dps.md) | Sí | Sí |
| [Supervisión y diagnóstico](iot-hub-monitor-resource-health.md) | Sí | Sí |
| [Mensajería de la nube a un dispositivo](iot-hub-devguide-c2d-guidance.md) |   | Sí |
| [Dispositivos gemelos](iot-hub-devguide-device-twins.md), [Módulos gemelos](iot-hub-devguide-module-twins.md) y [Administración de dispositivos](iot-hub-device-management-overview.md) |   | Sí |
| [Flujos de dispositivos (versión preliminar)](iot-hub-device-streams-overview.md) |   | Sí |
| [Azure IoT Edge](../iot-edge/about-iot-edge.md) |   | Sí |
| [Versión preliminar de IoT Plug and Play](../iot-pnp/overview-iot-plug-and-play.md) |   | Sí |

IoT Hub también ofrece un nivel gratis que está diseñado para pruebas y evaluación. Tiene todas las capacidades del nivel estándar, pero las concesiones de mensajería son limitadas. No puede actualizar desde el nivel gratis al plan Básico o Estándar.

## <a name="partitions"></a>Particiones

Azure IoT Hub contiene muchos componentes principales de [Azure Event Hubs](../event-hubs/event-hubs-features.md), incluidas las [particiones](../event-hubs/event-hubs-features.md#partitions). Los flujos de eventos de IoT Hub generalmente se rellenan con datos de telemetría entrantes que se notifican mediante varios dispositivos de IoT. La creación de particiones del flujo de eventos se usa para reducir las contenciones que se producen al leer y escribir simultáneamente en flujos de eventos.

El límite de particiones se elige cuando se crea la instancia de IoT Hub y no se puede cambiar. El límite máximo de particiones para los niveles básico y estándar de IoT Hub es 32. La mayoría de instancias de IoT Hub solo necesitan 4 particiones. Para más información para determinar el número de particiones, consulte esta pregunta frecuente sobre Event Hubs [¿Cuántas particiones necesito?](../event-hubs/event-hubs-faq.md#how-many-partitions-do-i-need)

## <a name="tier-upgrade"></a>Actualización del plan

Una vez creado el centro de IoT, puede actualizarlo desde el nivel Basic al nivel Estándar sin interrumpir las operaciones existentes. Para más información, consulte [How to upgrade your IoT hub](iot-hub-upgrade.md) (Actualización de IoT Hub).

La configuración de la partición permanecerá invariable cuando migre de un nivel básico a un nivel estándar.

> [!NOTE]
> El nivel Gratis no admite la actualización al nivel Básico o Estándar.

## <a name="iot-hub-rest-apis"></a>API REST de IoT Hub

La diferencia de funcionalidades admitidas entre los niveles Basic y Estándar de IoT Hub significa que algunas llamadas a la API no funcionan con centros de nivel Basic. En la tabla siguiente se muestran las API que están disponibles:

| API | Nivel Basic | Nivel Estándar o Gratis |
| --- | ---------- | ------------- |
| [Eliminar un dispositivo](https://docs.microsoft.com/javascript/api/azure-iot-digitaltwins-service/registrymanager?view=azure-node-latest#deletedevice-string--models-registrymanagerdeletedeviceoptionalparams-) | Sí | Sí |
| [Obtener dispositivo](https://docs.microsoft.com/azure/iot-hub/iot-c-sdk-ref/iothub-registrymanager-h/iothubregistrymanager-getdevice) | Sí | Sí |
| [Eliminar módulo](https://docs.microsoft.com/azure/iot-hub/iot-c-sdk-ref/iothub-registrymanager-h/iothubregistrymanager-deletemodule) | Sí | Sí |
| [Obtener módulo](https://docs.microsoft.com/java/api/com.microsoft.azure.sdk.iot.service.registrymanager.getmodule?view=azure-java-stable) | Sí | Sí |
| [Obtener estadísticas del registro](https://docs.microsoft.com/javascript/api/azure-iot-digitaltwins-service/registrymanager?view=azure-node-latest#getdevicestatistics-msrest-requestoptionsbase-) | Sí | Sí |
| [Obtener estadísticas de servicios](https://docs.microsoft.com/javascript/api/azure-iot-digitaltwins-service/registrymanager?view=azure-node-latest#getservicestatistics-msrest-requestoptionsbase-) | Sí | Sí |
| [Crear o actualizar el dispositivo](https://docs.microsoft.com/javascript/api/azure-iot-digitaltwins-service/registrymanager?view=azure-node-latest#createorupdatedevice-string--device--servicecallback-device--) | Sí | Sí |
| [Crear o actualizar el módulo](https://docs.microsoft.com/javascript/api/azure-iot-digitaltwins-service/registrymanager?view=azure-node-latest#createorupdatemodule-string--string--module--models-registrymanagercreateorupdatemoduleoptionalparams-) | Sí | Sí |
| [Consultar IoT Hub](https://docs.microsoft.com/dotnet/api/microsoft.azure.devices.registrymanager?view=azure-dotnet) | Sí | Sí |
| [Crear el URI de SAS de carga de archivos](https://docs.microsoft.com/rest/api/iothub/device/createfileuploadsasuri) | Sí | Sí |
| [Recibir notificación de dispositivo enlazado](https://docs.microsoft.com/rest/api/iothub/device/receivedeviceboundnotification) | Sí | Sí |
| [Enviar evento de dispositivo](https://docs.microsoft.com/rest/api/iothub/device/senddeviceevent) | Sí | Sí |
| Enviar eventos de módulo | Solo AMQP y MQTT | Solo AMQP y MQTT |
| [Actualizar estado de la carga de archivo](https://docs.microsoft.com/rest/api/iothub/device/updatefileuploadstatus) | Sí | Sí |
| [Operación de dispositivos en bloque](https://docs.microsoft.com/javascript/api/azure-iot-digitaltwins-service/registrymanager?view=azure-node-latest#bulkdevicecrud-exportimportdevice----msrest-requestoptionsbase-) | Sí, excepto las funcionalidades de IoT Edge | Sí |
| [Cancelar trabajo de importación y exportación](https://docs.microsoft.com/rest/api/iothub/service/jobs/cancelimportexportjob) | Sí | Sí |
| [Crear trabajo de importación y exportación](https://docs.microsoft.com/rest/api/iothub/service/jobs/createimportexportjob) | Sí | Sí |
| [Obtener trabajo de importación y exportación](https://docs.microsoft.com/rest/api/iothub/service/jobs/getimportexportjob) | Sí | Sí |
| [Obtener trabajos de importación y exportación](https://docs.microsoft.com/rest/api/iothub/service/jobs/getimportexportjobs) | Sí | Sí |
| [Purgar cola de comandos](https://docs.microsoft.com/javascript/api/azure-iot-digitaltwins-service/registrymanager?view=azure-node-latest#purgecommandqueue-string--msrest-requestoptionsbase-) |   | Sí |
| [Obtener dispositivo gemelo](https://docs.microsoft.com/java/api/com.microsoft.azure.sdk.iot.device.deviceclient.getdevicetwin?view=azure-java-stable) |   | Sí |
| [Obtener módulo gemelo](https://docs.microsoft.com/azure/iot-hub/iot-c-sdk-ref/iothub-devicetwin-h/iothubdevicetwin-getmoduletwin) |   | Sí |
| [Invocar método de dispositivo](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-direct-methods) |   | Sí |
| [Actualizar dispositivo gemelo](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-device-twins) |   | Sí |
| [Actualizar módulo gemelo](https://docs.microsoft.com/azure/iot-hub/iot-c-sdk-ref/iothub-devicetwin-h/iothubdevicetwin-updatemoduletwin) |   | Sí |
| [Abandonar notificación de dispositivo enlazado](https://docs.microsoft.com/rest/api/iothub/device/abandondeviceboundnotification) |   | Sí |
| [Completar notificación de dispositivo enlazado](https://docs.microsoft.com/rest/api/iothub/device/completedeviceboundnotification) |   | Sí |
| [Cancelar trabajo](https://docs.microsoft.com/rest/api/media/jobs/canceljob) |   | Sí |
| [Crear trabajo](https://docs.microsoft.com/rest/api/media/jobs/create) |   | Sí |
| [Obtener trabajo](https://docs.microsoft.com/java/api/com.microsoft.azure.sdk.iot.service.jobs.jobclient.getjob?view=azure-java-stable) |   | Sí |
| [Consultar trabajos](https://docs.microsoft.com/javascript/api/azure-iot-digitaltwins-service/jobclient?view=azure-node-latest#queryjobs-jobclientqueryjobsoptionalparams--servicecallback-queryresult--) |   | Sí |

## <a name="message-throughput"></a>Rendimiento de mensajes

La mejor forma de dimensionar una solución de IoT Hub es evaluar el tráfico en cada dispositivo. En concreto, tenga en cuenta la capacidad de procesamiento máxima requerida para las siguientes categorías de operaciones:

* Mensajes de dispositivo a nube
* Mensajes de nube a dispositivo
* Operaciones de registro de identidad

El tráfico en la instancia de IoT Hub se mide por unidad. Al crear una instancia de IoT Hub, elija el nivel y la edición, y establezca el número de unidades disponibles. Puede adquirir hasta 200 unidades para las ediciones B1, B2, S1 o S2, o hasta 10 unidades para B3 o S3. Después de crear la instancia de IoT Hub, puede cambiar el número de unidades disponibles dentro de su edición, actualizar o cambiar a una versión anterior entre las de su mismo nivel (de B1 a B2) o actualizar del nivel básico al estándar (de B1 a S1) sin interrumpir las operaciones existentes. Para más información, consulte [How to upgrade your IoT hub](iot-hub-upgrade.md) (Actualización de IoT Hub).  

Como ejemplo de las funcionalidades de tráfico de cada nivel, los mensajes del dispositivo a la nube siguen estas directrices de rendimiento sostenidas:

| Versiones de nivel | Capacidad de procesamiento sostenida | Velocidad de envío sostenida |
| --- | --- | --- |
| B1, S1 |Hasta 1111 KB/minuto por unidad<br/>(1,5 GB/día/unidad) |Promedio de 278 mensajes/minuto por unidad<br/>(400 000 mensajes/día por unidad) |
| B2, S2 |Hasta 16 MB/minuto por unidad<br/>(22,8 GB/día/unidad) |Promedio de 4167 mensajes/minuto por unidad<br/>(6 millones de mensajes/día por unidad) |
| B3, S3 |Hasta 814 MB/minuto por unidad<br/>(1144,4 GB/día/unidad) |Promedio de 208.333 mensajes/minuto por unidad<br/>(300 millones de mensajes/día por unidad) |

El rendimiento del dispositivo a la nube es solo una de las métricas que se deben tener en cuenta al diseñar una solución de IoT. Para información más completa, consulte las [cuotas y limitaciones de IoT Hub](iot-hub-devguide-quotas-throttling.md).

### <a name="identity-registry-operation-throughput"></a>Capacidad de procesamiento para las operaciones de registro de identidad

Las operaciones de registro de identidad de IoT Hub no deberían ser operaciones en tiempo de ejecución porque tienen que ver principalmente con el aprovisionamiento de dispositivos.

Vea las cifras de rendimiento de ráfaga específicas en [Cuotas y limitaciones de IoT Hub](iot-hub-devguide-quotas-throttling.md).

## <a name="auto-scale"></a>Escalado automático

Si se está aproximando al límite de mensajes permitido por IoT Hub, puede usar estos [pasos para realizar un escalado automático](https://azure.microsoft.com/resources/samples/iot-hub-dotnet-autoscale/) que aumente una unidad de IoT Hub en el mismo nivel.

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre las funcionalidades de IoT Hub y detalles de rendimiento, consulte [Precios de IoT Hub](https://azure.microsoft.com/pricing/details/iot-hub) o [Cuotas y limitación de IoT Hub](iot-hub-devguide-quotas-throttling.md).

* Para cambiar el nivel de IoT Hub, siga los pasos descritos en el artículo sobre cómo [actualizar IoT Hub](iot-hub-upgrade.md).

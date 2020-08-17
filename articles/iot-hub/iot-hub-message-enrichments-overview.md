---
title: Introducción al enriquecimiento de mensajes de Azure IoT Hub
description: En este artículo se muestran los enriquecimientos de mensajes, que proporcionan al centro de IoT la posibilidad de marcar los mensajes con información adicional antes de enviarlos al punto de conexión designado.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 05/10/2019
ms.author: robinsh
ms.openlocfilehash: 08eb5fd4d28421105988d64b99afb441a0b824f4
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87290309"
---
# <a name="message-enrichments-for-device-to-cloud-iot-hub-messages"></a>Enriquecimientos de mensajes para mensajes de IoT Hub del dispositivo a la nube

*El enriquecimiento de mensajes* es la capacidad de IoT Hub para *marcar* los mensajes con información adicional antes de que los mensajes se envíen al punto de conexión designado. Una razón para utilizar el enriquecimiento de mensajes es incluir datos que se pueden usar para simplificar el procesamiento descendente. Por ejemplo, el enriquecimiento de mensajes de telemetría de dispositivos con una etiqueta de dispositivo gemelo puede reducir la carga de los clientes para realizar llamadas API de dispositivos gemelos para obtener esta información.

![Flujo del enriquecimiento de mensajes](./media/iot-hub-message-enrichments-overview/message-enrichments-flow.png)

El enriquecimiento de mensajes tiene tres elementos clave:

* Nombre o clave de enriquecimiento

* Valor A

* Uno o varios [puntos de conexión](iot-hub-devguide-endpoints.md) a los que debe aplicarse el enriquecimiento.

La **clave** es una cadena. Una clave solo puede incluir caracteres alfanuméricos o estos caracteres especiales: guion (`-`), guion bajo (`_`) y punto (`.`).

El **valor** puede ser cualquiera de los ejemplos siguientes:

* Cualquier cadena estática. No se permiten valores dinámicos como condiciones, lógica, operaciones y funciones. Por ejemplo, si desarrolla una aplicación SaaS que la utilizan varios clientes, puede asignar un identificador a cada cliente y hacer que ese identificador esté disponible en la aplicación. Cuando se ejecuta la aplicación, IoT Hub marcará los mensajes de telemetría de los dispositivos con el identificador del cliente, haciendo posible procesar los mensajes de forma diferente para cada cliente.

* El nombre del centro de IoT que envía el mensaje. Este valor es *$iothubname*.

* Información del dispositivo gemelo, como la ruta de acceso. Algunos ejemplos serían *$twin.tags.field* y *$twin.tags.latitude*.

   > [!NOTE]
   > En este momento, solo se admiten las variables $iothubname, $twin.tags, $twin.properties.desired y $twin.properties.reported para el enriquecimiento de mensajes.

Los enriquecimientos de mensajes se agregan como propiedades de la aplicación a los mensajes enviados a los puntos de conexión elegidos.  

## <a name="applying-enrichments"></a>Aplicación de enriquecimientos

Los mensajes pueden proceder de cualquier origen de datos admitido por el [enrutamiento de mensajes de IoT Hub](iot-hub-devguide-messages-d2c.md), incluidos los ejemplos siguientes:

* telemetría del dispositivo, como la temperatura o la presión
* notificaciones de cambio de dispositivo gemelo: cambios en el dispositivo gemelo
* eventos de ciclo de vida del dispositivo, como cuando se crea o elimina el dispositivo

Puede agregar enriquecimientos a los mensajes que van al punto de conexión incorporado de una instancia de IoT Hub, o mensajes que se enrutan a puntos de conexión personalizados, como Azure Blob Storage, una cola de Service Bus o un tema de Service Bus.

Puede agregar enriquecimientos a los mensajes que se publican en Event Grid seleccionando el punto de conexión como Event Grid. Creamos una ruta predeterminada en IoT Hub para la telemetría del dispositivo, en función de su suscripción de Event Grid. Esta ruta única puede controlar todas las suscripciones de Event Grid. Puede configurar enriquecimientos para el punto de conexión de Event Grid después de crear la suscripción de Event Grid para la telemetría del dispositivo. Para más información, consulte [Iot Hub y Event Grid](iot-hub-event-grid.md).

Se aplican los enriquecimientos de punto de conexión. Si especifica cinco enriquecimientos que se van a marcar para un punto de conexión específico, todos los mensajes que lleguen a ese punto de conexión se marcan con los mismos cinco enriquecimientos.

Los enriquecimientos se pueden configurar mediante los métodos siguientes:

| **Método** | **Comando** |
| ----- | -----| 
| Portal | [Azure Portal](https://portal.azure.com) | Consulte el [tutorial sobre enriquecimientos de mensajes](tutorial-message-enrichments.md). | 
| Azure CLI   | [az iot hub message-enrichment](https://docs.microsoft.com/cli/azure/iot/hub/message-enrichment?view=azure-cli-latest) |
| Azure PowerShell | [Add-AzIotHubMessageEnrichment](https://docs.microsoft.com/powershell/module/az.iothub/add-aziothubmessageenrichment) |

La adición de enriquecimientos de mensajes no agrega latencia al enrutamiento de mensajes.

Para probar enriquecimientos de mensajes, consulte el [tutorial sobre enriquecimientos de mensajes](tutorial-message-enrichments.md).

## <a name="limitations"></a>Limitaciones

* Puede agregar hasta 10 enriquecimientos por instancia de IoT Hub para esos centros en el nivel Estándar o Básico. Para las instancias de IoT Hub de nivel Gratis, puede agregar hasta dos enriquecimientos.

* En algunos casos, si está aplicando un enriquecimiento con un valor establecido en una etiqueta o propiedad en el dispositivo gemelo, el valor se marcará como un valor de cadena. Por ejemplo, si se establece un valor de enriquecimiento en $twin.tags.field, los mensajes se marcarán con la cadena "$twin.tags.field" en lugar del valor de ese campo del gemelo. Esto sucede en los casos siguientes:

   * El centro de IoT está en el nivel Básico. Los centros de IoT de nivel Básico no admiten dispositivos gemelos.

   * El centro de IoT está en el nivel Estándar, pero el dispositivo que envía el mensaje no tiene ningún dispositivo gemelo.

   * El centro de IoT está en el nivel Estándar, pero no existe la ruta de acceso que usa el dispositivo gemelo para el valor del enriquecimiento. Por ejemplo, si el valor de enriquecimiento se establece en $twin.tags.location y el dispositivo gemelo no tiene una propiedad de ubicación bajo las etiquetas, el mensaje se marca con la cadena "$twin.tags.location". 

* Las actualizaciones de un dispositivo gemelo pueden tardar hasta cinco minutos en quedar reflejadas en el valor de enriquecimiento correspondiente.

* El tamaño total del mensaje, incluidos los enriquecimientos, no puede superar los 256 KB. Si el tamaño del mensaje supera los 256 KB, el centro de IoT eliminará el mensaje. Puede usar [métricas de IoT Hub](iot-hub-metrics.md) para identificar y depurar errores cuando se eliminan los mensajes. Por ejemplo, puede supervisar d2c.telemetry.egress.invalid.

* El enriquecimiento de mensajes no se aplica a los eventos de cambio Digital Twins (parte de la [versión preliminar pública de IoT Plug and Play](../iot-pnp/overview-iot-plug-and-play.md)).

## <a name="pricing"></a>Precios

El enriquecimiento de mensajes está disponible sin ningún cargo adicional. Actualmente, se le cobra cuando envía un mensaje a un centro de IoT. Solo se le cobrará una vez por ese mensaje, incluso si el mensaje va a varios puntos de conexión.

## <a name="next-steps"></a>Pasos siguientes

Consulte estos artículos para obtener más información sobre el enrutamiento de mensajes en una instancia de IoT Hub:

* [Tutorial sobre enriquecimientos de mensajes](tutorial-message-enrichments.md)

* [Uso del enrutamiento de mensajes de IoT Hub para enviar mensajes del dispositivo a la nube a distintos puntos de conexión](iot-hub-devguide-messages-d2c.md)

* [Tutorial: Enrutamiento de IoT Hub](tutorial-routing.md)

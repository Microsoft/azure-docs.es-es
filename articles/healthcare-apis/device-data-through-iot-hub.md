---
title: 'Tutorial: Recepción de datos del dispositivo mediante Azure IoT Hub'
description: En este tutorial, aprenderá a habilitar el enrutamiento de datos de dispositivos desde IoT Hub a Azure API for FHIR mediante el conector de IoT.
services: healthcare-apis
author: ms-puneet-nagpal
ms.service: healthcare-apis
ms.subservice: iomt
ms.topic: tutorial
ms.date: 06/27/2020
ms.author: punagpal
ms.openlocfilehash: d606cd4f5c4b901c060c97d73524997b94eaf225
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87446296"
---
# <a name="tutorial-receive-device-data-through-azure-iot-hub"></a>Tutorial: Recepción de datos del dispositivo mediante Azure IoT Hub

El conector de IoT proporciona la funcionalidad de ingesta de datos de dispositivos de Internet de las cosas médicas (IoMT) en Azure API for FHIR. La guía de inicio rápido [Implementación del conector de IoT (versión preliminar) mediante Azure Portal](iot-fhir-portal-quickstart.md) muestra un ejemplo de un dispositivo administrado por Azure IoT Central que [envía datos de telemetría](iot-fhir-portal-quickstart.md#connect-your-devices-to-iot) al conector de IoT. El conector de IoT también puede trabajar con dispositivos aprovisionados y administrados mediante Azure IoT Hub. En este tutorial se proporciona el procedimiento para conectar y enrutar los datos del dispositivo desde Azure IoT Hub al conector de IoT.

## <a name="prerequisites"></a>Requisitos previos

- Una suscripción de Azure activa: [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
- Un recurso de Azure API for FHIR con al menos un conector de IoT: [Implementación del conector de IoT (versión preliminar) mediante Azure Portal](iot-fhir-portal-quickstart.md)
- Un recurso de Azure IoT Hub conectado a dispositivos reales o simulados: [Creación de un centro de IoT con Azure Portal](https://docs.microsoft.com/azure/iot-hub/quickstart-send-telemetry-dotnet)

> [!TIP]
> Si usa una aplicación de dispositivo simulado de Azure IoT Hub, no dude en elegir la aplicación que prefiera entre los distintos lenguajes y sistemas compatibles.

## <a name="get-connection-string-for-iot-connector-preview"></a>Obtención de la cadena de conexión para el conector de IoT (versión preliminar)

Azure IoT Hub requiere una cadena de conexión para conectarse de forma segura al conector de IoT. Cree una nueva cadena de conexión para el conector de IoT como se describe en [Generación de una cadena de conexión](iot-fhir-portal-quickstart.md#generate-a-connection-string). Conserve esta cadena de conexión para usarla en el paso siguiente.

El conector de IoT usa una instancia de Azure Event Hubs en segundo plano para recibir los mensajes del dispositivo. La cadena de conexión creada anteriormente es básicamente la cadena de conexión a este centro de eventos subyacente.

## <a name="connect-azure-iot-hub-with-the-iot-connector-preview"></a>Conexión de Azure IoT Hub al conector de IoT (versión preliminar)

Azure IoT Hub admite una característica llamada [enrutamiento de mensajes](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-messages-d2c) que proporciona la funcionalidad de enviar datos del dispositivo a varios servicios de Azure, como Event Hubs, una cuenta de Azure Storage y Service Bus. El conector de IoT aprovecha esta característica para conectarse y enviar datos del dispositivo desde Azure IoT Hub al punto de conexión del centro de eventos.

> [!NOTE] 
> En este momento, solo se pueden usar el comando de PowerShell o de la CLI para [crear el enrutamiento de mensajes](https://docs.microsoft.com/azure/iot-hub/tutorial-routing), porque el centro de eventos del conector de IoT no está hospedado en la suscripción del cliente, por lo que no será visible mediante Azure Portal. Sin embargo, una vez que se agregan los objetos de ruta de mensaje mediante PowerShell o la CLI, están visibles en Azure Portal y se pueden administrar desde allí.

La configuración de un enrutamiento de mensajes consta de dos pasos.

### <a name="add-an-endpoint"></a>Agregación de un extremo
En este paso se define el punto de conexión al que IoT Hub enrutará los datos. Puede crear este punto de conexión mediante el comando [Add-AzIotHubRoutingEndpoint](https://docs.microsoft.com/powershell/module/az.iothub/Add-AzIotHubRoutingEndpoint) de PowerShell o el comando [az iot hub routing-endpoint create](https://docs.microsoft.com/cli/azure/iot/hub/routing-endpoint?#az-iot-hub-routing-endpoint-create) de la CLI, según sus preferencias.

Esta es la lista de parámetros que se usan con el comando para crear un punto de conexión:

|Parámetro de PowerShell|Parámetro de la CLI|Descripción|
|---|---|---|
|ResourceGroupName|resource-group|Nombre del grupo de recursos de su recurso de IoT Hub.|
|Nombre|hub-name|Nombre de su recurso de IoT Hub.|
|EndpointName|endpoint-name|Nombre que le gustaría asignar al punto de conexión que se va a crear.|
|EndpointType|endpoint-type|Tipo de punto de conexión al que necesita conectarse IoT Hub. Use el valor literal "EventHub" para PowerShell y "eventhub" para la CLI.|
|EndpointResourceGroup|endpoint-resource-group|Nombre del grupo de recursos de su recurso de Azure API for FHIR del conector de IoT. Puede obtener este valor en la página de información general de Azure API for FHIR.|
|EndpointSubscriptionId|endpoint-subscription-id|Identificador de la suscripción del recurso de Azure API for FHIR del conector de IoT. Puede obtener este valor en la página de información general de Azure API for FHIR.|
|ConnectionString|connection-string|Cadena de conexión al conector de IoT. Use el valor que obtuvo en el paso anterior.|

### <a name="add-a-message-route"></a>Adición de una ruta de mensajes
En este paso se define una ruta de mensajes mediante el punto de conexión creado anteriormente. Puede crear una ruta mediante el comando [Add-AzIotHubRoute](https://docs.microsoft.com/powershell/module/az.iothub/Add-AzIoTHubRoute) de PowerShell o el comando [az iot hub route create](https://docs.microsoft.com/cli/azure/iot/hub/route#az-iot-hub-route-create) de la CLI, según sus preferencias.

Esta es la lista de parámetros que se usan con el comando para crear un punto de conexión:

|Parámetro de PowerShell|Parámetro de la CLI|Descripción|
|---|---|---|
|ResourceGroupName|g|Nombre del grupo de recursos de su recurso de IoT Hub.|
|Nombre|hub-name|Nombre de su recurso de IoT Hub.|
|EndpointName|endpoint-name|Nombre del punto de conexión que ha creado anteriormente.|
|RouteName|route-name|Nombre que desea asignar a la ruta de mensajes que se va a crear.|
|Source|source-type|Tipo de datos que se van a enviar al punto de conexión. Use el valor literal "DeviceMessages" para PowerShell y "devicemessages" para la CLI.|

## <a name="send-device-message-to-iot-hub"></a>Envío del mensaje de dispositivo a IoT Hub

Use el dispositivo (real o simulado) para enviar el mensaje de frecuencia cardíaca de ejemplo que se muestra a continuación a Azure IoT Hub. Este mensaje se enrutará al conector de IoT, donde el mensaje se transformará en un recurso de observación de FHIR y se almacenará en Azure API for FHIR.

```json
{
  "HeartRate": 80,
  "RespiratoryRate": 12,
  "HeartRateVariability": 64,
  "BodyTemperature": 99.08839032397609,
  "BloodPressure": {
    "Systolic": 23,
    "Diastolic": 34
  },
  "Activity": "walking"
}
```
> [!IMPORTANT]
> Asegúrese de enviar un mensaje de dispositivo que se ajuste a las [plantillas de asignación](iot-mapping-templates.md) configuradas con el conector de IoT.

## <a name="view-device-data-in-azure-api-for-fhir"></a>Visualización de los datos del dispositivo en Azure API for FHIR

Puede ver los recursos de observación de FHIR creados por el conector de IoT en Azure API for FHIR mediante Postman. Configure [Postman para acceder a Azure API for FHIR](access-fhir-postman-tutorial.md) y realice una solicitud `GET` a `https://your-fhir-server-url/Observation?code=http://loinc.org|8867-4` para ver los recursos de observación de FHIR con el valor de la frecuencia cardíaca enviado en el mensaje del ejemplo anterior.

> [!TIP]
> Asegúrese de que el usuario tiene acceso adecuado al plano de datos de Azure API for FHIR. Use el [control de acceso basado en roles de Azure (Azure RBAC)](configure-azure-rbac.md) para asignar los roles del plano de datos requeridos.


## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha configurado Azure IoT Hub para enrutar los datos del dispositivo al conector de IoT. Seleccione entre los pasos siguientes para más información sobre el conector de IoT:

Comprenda las distintas fases del flujo de datos en el conector de IoT.

>[!div class="nextstepaction"]
>[Flujo de datos del conector de IoT](iot-data-flow.md)

Aprenda a configurar el conector de IoT mediante plantillas de asignación de dispositivos y FHIR.

>[!div class="nextstepaction"]
>[Plantillas de asignación del conector de IoT](iot-mapping-templates.md)

FHIR es la marca registrada de HL7 y se usa con su permiso.

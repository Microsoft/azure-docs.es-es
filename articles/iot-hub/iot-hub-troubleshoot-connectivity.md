---
title: Supervisión y solución de problemas de desconexión con Azure IoT Hub
description: Aprenda a supervisar y solucionar errores comunes de conectividad de los dispositivos para Azure IoT Hub
author: jlian
manager: briz
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 01/30/2020
ms.author: jlian
ms.custom:
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
- 'Role: Technical Support'
ms.openlocfilehash: b22573fc98d4647c2b05763d9b67c37f6f023f98
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87322727"
---
# <a name="monitor-diagnose-and-troubleshoot-disconnects-with-azure-iot-hub"></a>Supervisión, diagnóstico y solución de problemas de desconexión con Azure IoT Hub

Los problemas de conectividad de los dispositivos IoT pueden ser difíciles de solucionar porque existen varios puntos de error posibles. La lógica de aplicación, las redes físicas, los protocolos, el hardware, IoT Hub y otros servicio en la nube pueden causar problemas. La capacidad de detectar e identificar el origen de un problema es fundamental. Sin embargo, una solución de IoT a gran escala puede tener miles de dispositivos, por lo que no es práctico comprobar manualmente cada dispositivo. Para que le sea más fácil detectar, diagnosticar y solucionar estos problemas a gran escala, use las funcionalidades de supervisión que IoT Hub proporciona mediante Azure Monitor. Estas funcionalidades se limitan a lo que observa IoT Hub, por lo que también se recomienda seguir los procedimientos recomendados de supervisión para los dispositivos y otros servicios de Azure.

## <a name="get-alerts-and-error-logs"></a>Recepción de alertas y registros de errores

Use Azure Monitor para recibir alertas y escribir registros cuando se produzca la desconexión de dispositivos.

### <a name="turn-on-diagnostic-logs"></a>Activación de los registros de diagnósticos

Para registrar errores y eventos de conexión de dispositivos, active los diagnósticos de IoT Hub. Se recomienda activar estos registros lo antes posible, ya que si los registros de diagnóstico no están habilitados, cuando se produzca la desconexión del dispositivo, no tendrá información con la que solucionar el problema.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. Vaya a su instancia de IoT Hub.

3. Seleccione **Configuración de diagnóstico**.

4. Seleccione **Activar diagnósticos**.

5. Habilite los registros de **Conexiones** para que se recopilen.

6. Para facilitar el análisis, debe activar **Enviar a Log Analytics** ([consulte los precios](https://azure.microsoft.com/pricing/details/log-analytics/)). Consulte el ejemplo en la sección dedicada a [resolver los errores de conectividad](#resolve-connectivity-errors).

   ![Configuración recomendada](./media/iot-hub-troubleshoot-connectivity/diagnostic-settings-recommendation.png)

Para obtener más información, consulte [Supervisión del mantenimiento de Azure IoT Hub y diagnóstico de problemas rápidamente](iot-hub-monitor-resource-health.md).

### <a name="set-up-alerts-for-device-disconnect-at-scale"></a>Configuración de alertas para la desconexión de dispositivos a gran escala

Para recibir alertas cuando se desconecten los dispositivos, configúrelas en la métrica **Connected devices (preview)** .

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. Vaya a su instancia de IoT Hub.

3. Seleccione **Alertas**.

4. Seleccione **Nueva regla de alertas**.

5. Seleccione **Agregar condición** y, a continuación, seleccione "Dispositivos conectados (versión preliminar)".

6. Configuración del umbral y las alertas según indicaciones.

Para más información, consulte [¿Qué son las alertas en Microsoft Azure?](../azure-monitor/platform/alerts-overview.md)

#### <a name="detecting-individual-device-disconnects"></a>Detección de la desconexión de dispositivos individuales

Para detectar las desconexiones *por dispositivo*, por ejemplo, cuando necesite saber que una factoría se ha desconectado, [configure eventos de desconexión de dispositivo con Event Grid](iot-hub-event-grid.md).

## <a name="resolve-connectivity-errors"></a>Resolución de errores de conectividad

Cuando active las alertas y los registros de diagnóstico de los dispositivos conectados, recibirá alertas cuando se produzca algún error. En esta sección se describe cómo buscar problemas comunes al recibir una alerta. En los pasos siguientes se da por sentado que ha configurado los registros de Azure Monitor para los registros de diagnóstico.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Vaya a su instancia de IoT Hub.

1. Seleccione **Registros**.

1. Para aislar los registros de errores de conectividad de IoT Hub, escriba esta consulta y pulse **Ejecutar**:

    ```kusto
    AzureDiagnostics
    | where ( ResourceType == "IOTHUBS" and Category == "Connections" and Level == "Error")
    ```

1. Si hay resultados, busque `OperationName`, `ResultType` (código de error) y `ResultDescription` (mensaje de error) para obtener más detalles sobre el error.

   ![Ejemplo de registro de errores](./media/iot-hub-troubleshoot-connectivity/diag-logs.png)

1. Siga las guías de resolución de problemas para los errores más comunes:

    - **[404104 DeviceConnectionClosedRemotely](iot-hub-troubleshoot-error-404104-deviceconnectionclosedremotely.md)**
    - **[401003 IoTHubUnauthorized](iot-hub-troubleshoot-error-401003-iothubunauthorized.md)**
    - **[409002 LinkCreationConflict](iot-hub-troubleshoot-error-409002-linkcreationconflict.md)**
    - **[500001 ServerError](iot-hub-troubleshoot-error-500xxx-internal-errors.md)**
    - **[500008 GenericTimeout](iot-hub-troubleshoot-error-500xxx-internal-errors.md)**

## <a name="i-tried-the-steps-but-they-didnt-work"></a>He intentado estos pasos, pero no funcionan

Si los pasos anteriores no le son de utilidad, intente lo siguiente:

* Si tiene acceso a los dispositivos problemáticos, tanto físicamente como en remoto (por ejemplo, SSH), siga la [Guía de solución de problemas de dispositivos](https://github.com/Azure/azure-iot-sdk-node/wiki/Troubleshooting-Guide-Devices) para continuar solucionando los problemas.

* Compruebe que los dispositivos están **habilitados** en Azure Portal > su instancia de IoT Hub > Dispositivos IoT.

* Si el dispositivo usa el protocolo MQTT, compruebe que el puerto 8883 está abierto. Para más información, consulte [Conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

* Obtenga ayuda en la [Página de preguntas y respuestas de Microsoft sobre Azure IoT Hub](https://docs.microsoft.com/answers/topics/azure-iot-hub.html), [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-iot-hub) o el [soporte técnico de Azure](https://azure.microsoft.com/support/options/).

Con el fin de mejorar la documentación para los usuarios, deje un comentario a continuación si esta guía no le ha ayudado.

## <a name="next-steps"></a>Pasos siguientes

* Para más información acerca de cómo resolver problemas transitorios, consulte [Control de errores transitorios](/azure/architecture/best-practices/transient-faults).

* Para más información sobre el SDK de Azure IoT y la administración de reintentos, consulte [Administración de la conectividad y mensajería confiable mediante los SDK de dispositivo de Azure IoT Hub](iot-hub-reliability-features-in-sdks.md#connection-and-retry).

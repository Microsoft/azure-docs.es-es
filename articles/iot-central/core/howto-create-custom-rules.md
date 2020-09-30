---
title: Ampliación de Azure IoT Central con notificaciones y reglas personalizadas | Microsoft Docs
description: Como desarrollador de soluciones, configure una aplicación de IoT Central para enviar notificaciones por correo electrónico cuando un dispositivo deja de enviar datos de telemetría. Esta solución utiliza Azure Stream Analytics, Azure Functions y SendGrid.
author: dominicbetts
ms.author: dobett
ms.date: 12/02/2019
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.custom: mvc, devx-track-csharp
manager: philmea
ms.openlocfilehash: 288fb5b552eab2029ea72f73a835fc73d97244b9
ms.sourcegitcommit: 43558caf1f3917f0c535ae0bf7ce7fe4723391f9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90018206"
---
# <a name="extend-azure-iot-central-with-custom-rules-using-stream-analytics-azure-functions-and-sendgrid"></a>Extensión de Azure IoT Central con reglas personalizadas mediante Stream Analytics, Azure Functions y SendGrid

Esta guía paso a paso le muestra, como desarrollador de soluciones, cómo ampliar la aplicación de IoT Central con notificaciones y reglas personalizadas. El ejemplo muestra el envío de una notificación a un operador cuando un dispositivo deja de enviar datos de telemetría. La solución utiliza una consulta de [Azure Stream Analytics](https://docs.microsoft.com/azure/stream-analytics/) para detectar cuándo un dispositivo ha dejado de enviar datos de telemetría. El trabajo de Stream Analytics utiliza [Azure Functions](https://docs.microsoft.com/azure/azure-functions/) para enviar correos electrónicos de notificación mediante [SendGrid](https://sendgrid.com/docs/for-developers/partners/microsoft-azure/).

Esta guía paso a paso le muestra cómo ampliar IoT Central todavía más con las reglas y acciones integradas.

En esta guía paso a paso, aprenderá a:

* Hacer streaming de datos de telemetría desde una aplicación de IoT Central mediante la *exportación de datos continua*.
* Crear una consulta de Stream Analytics que detecta cuándo un dispositivo ha dejado de enviar datos.
* Enviar una notificación por correo electrónico con los servicios de SendGrid y Azure Functions.

## <a name="prerequisites"></a>Prerequisites

Recuerde que para completar los pasos de esta guía paso a paso, necesita una suscripción activa a Azure.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

### <a name="iot-central-application"></a>Aplicación de IoT Central

Cree una aplicación de IoT Central desde el sitio web del [administrador de aplicaciones de Azure IoT Central](https://aka.ms/iotcentral) con la configuración siguiente:

| Configuración | Value |
| ------- | ----- |
| Plan de precios | Estándar |
| Plantilla de la aplicación | Análisis en la tienda: supervisión del estado |
| Nombre de la aplicación | Acepte el valor predeterminado o elija su propio nombre. |
| URL | Acepte el valor predeterminado o elija un prefijo de dirección URL único. |
| Directorio | El inquilino de Azure Active Directory |
| Suscripción de Azure | Su suscripción de Azure |
| Region | La región más cercana |

En los ejemplos y las capturas de pantalla de este artículo se usa la región **Estados Unidos**. Elija una ubicación cercana a usted y asegúrese de que crea todos los recursos en la misma región.

Esta plantilla de aplicación incluye dos dispositivos de termostato simulados que envían telemetría.

### <a name="resource-group"></a>Resource group

Use [Azure Portal para crear un grupo de recursos](https://portal.azure.com/#create/Microsoft.ResourceGroup) llamado **DetectStoppedDevices** que contenga los otros recursos que cree. Cree los recursos de Azure en la misma ubicación que la aplicación de IoT Central.

### <a name="event-hubs-namespace"></a>Espacio de nombres de Event Hubs

Use [Azure Portal para crear un espacio de nombres de Event Hubs](https://portal.azure.com/#create/Microsoft.EventHub) con la siguiente configuración:

| Configuración | Value |
| ------- | ----- |
| Nombre    | Elija el nombre del espacio de nombres |
| Plan de tarifa | Básica |
| Subscription | Su suscripción |
| Resource group | DetectStoppedDevices |
| Location | Este de EE. UU. |
| Unidades de procesamiento | 1 |

### <a name="stream-analytics-job"></a>Trabajo de Stream Analytics

Use [Azure Portal para crear un trabajo de Stream Analytics](https://portal.azure.com/#create/Microsoft.StreamAnalyticsJob) con la siguiente configuración:

| Configuración | Value |
| ------- | ----- |
| Nombre    | Elija el nombre del trabajo |
| Subscription | Su suscripción |
| Resource group | DetectStoppedDevices |
| Location | Este de EE. UU. |
| Entorno de hospedaje | Nube |
| Unidades de streaming | 3 |

### <a name="function-app"></a>Aplicación de función

Use [Azure Portal para crear una aplicación de función](https://portal.azure.com/#create/Microsoft.FunctionApp) con la siguiente configuración:

| Configuración | Value |
| ------- | ----- |
| Nombre de la aplicación    | Elija el nombre de la aplicación de función |
| Subscription | Su suscripción |
| Resource group | DetectStoppedDevices |
| SO | Windows |
| Plan de hospedaje | Plan de consumo |
| Location | Este de EE. UU. |
| Pila de tiempo de ejecución | .NET |
| Storage | Crear nuevo |

### <a name="sendgrid-account"></a>Cuenta de SendGrid

Use [Azure Portal para crear una cuenta de SendGrid](https://portal.azure.com/#create/Sendgrid.sendgrid) con la siguiente configuración:

| Configuración | Value |
| ------- | ----- |
| Nombre    | Elija el nombre de cuenta de SendGrid |
| Contraseña | Cree una contraseña |
| Subscription | Su suscripción |
| Resource group | DetectStoppedDevices |
| Plan de tarifa | F1 Gratis |
| Información de contacto | Rellene la información necesaria |

Cuando haya creado los recursos necesarios, el grupo de recursos **DetectStoppedDevices** se parecerá al de la captura de pantalla siguiente:

![Grupo de recursos de detección de dispositivos detenidos](media/howto-create-custom-rules/resource-group.png)

## <a name="create-an-event-hub"></a>Creación de un centro de eventos

Puede configurar una aplicación de IoT Central para la exportación continua de datos de telemetría a un centro de eventos. En esta sección, creará un centro de eventos para recibir datos de telemetría de la aplicación de IoT Central. El centro de eventos entrega los datos de telemetría al trabajo de Stream Analytics para su procesamiento.

1. En Azure Portal, vaya al espacio de nombres de Event Hubs y seleccione **+ Centro de eventos**.
1. Asigne al centro de eventos el nombre **centralexport** y seleccione **Crear**.

El espacio de nombres de Event Hubs se parece a la captura de pantalla siguiente:

![Espacio de nombres de Event Hubs](media/howto-create-custom-rules/event-hubs-namespace.png)

## <a name="get-sendgrid-api-key"></a>Obtención de la clave de API de SendGrid

La aplicación de función necesita una clave de API de SendGrid para enviar mensajes de correo electrónico. Para crear una clave de API de SendGrid:

1. En Azure Portal, vaya a la cuenta de SendGrid. A continuación, elija **Administrar** para acceder a la cuenta de SendGrid.
1. En la cuenta de SendGrid elija **Configuración** y, a continuación, **Claves de API**. Elija **Crear clave de API**:

    ![Creación de una clave de API de SendGrid](media/howto-create-custom-rules/sendgrid-api-keys.png)

1. En la página **Crear clave de API**, cree una clave denominada **AzureFunctionAccess** con permisos de **Acceso completo**.
1. Tome nota de la clave de API, la necesitará al configurar la aplicación de función.

## <a name="define-the-function"></a>Definición de la función

Esta solución usa una aplicación de Azure Functions para enviar una notificación por correo electrónico cuando el trabajo de Stream Analytics detecta un dispositivo detenido. Para crear la aplicación de función:

1. En Azure Portal, vaya a la instancia de **App Service** en el grupo de recursos **DetectStoppedDevices**.
1. Seleccione **+** para crear una nueva función.
1. En la página **ELIJA UN ENTORNO DE DESARROLLO**, seleccione **En el portal** y después **Continuar**.
1. En la página **CREAR FUNCIÓN**, elija **Webhook y API** y, a continuación, seleccione **Crear**.

El portal crea una función predeterminada denominada **HttpTrigger1**:

![Función de desencadenador HTTP predeterminada](media/howto-create-custom-rules/default-function.png)

### <a name="configure-function-bindings"></a>Configuración de enlaces de función

Para enviar correos electrónicos con SendGrid, tiene que configurar los enlaces de la función como sigue:

1. Seleccione **Integrar**, elija la salida **HTTP ($return)** y, a continuación, seleccione **Eliminar**.
1. Elija **+ Nueva salida**, después **SendGrid**y, a continuación, elija **Seleccionar**. Elija **Instalar** para instalar la extensión de SendGrid.
1. Cuando se complete la instalación, seleccione **Usar el valor devuelto de la función**. Agregar una **Dirección de destino** válida para recibir notificaciones por correo electrónico.  Agregar una **Dirección de origen** que se usará como remitente de correo electrónico.
1. Seleccione **nueva** junto a **Configuración de aplicación de la clave de API de SendGrid**. Escriba **SendGridAPIKey** como la clave y la clave de API de SendGrid que anotó anteriormente como el valor. Seleccione **Crear**.
1. Elija **Guardar** para guardar los enlaces de SendGrid para la función.

La configuración integrada será similar a la siguiente captura de pantalla:

![Integraciones de aplicación de función](media/howto-create-custom-rules/function-integrate.png)

### <a name="add-the-function-code"></a>Incorporación del código de función

Para implementar la función, agregue el código C# código para analizar la solicitud HTTP entrante y enviar los correos electrónicos de la manera siguiente:

1. Elija la función **HttpTrigger1** en la aplicación de función y reemplazar el código C# con el código siguiente:

    ```csharp
    #r "Newtonsoft.Json"
    #r "..\bin\SendGrid.dll"

    using System;
    using SendGrid.Helpers.Mail;
    using Microsoft.Azure.WebJobs.Host;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Primitives;
    using Newtonsoft.Json;

    public static SendGridMessage Run(HttpRequest req, ILogger log)
    {
        string requestBody = new StreamReader(req.Body).ReadToEnd();
        log.LogInformation(requestBody);
        var notifications = JsonConvert.DeserializeObject<IList<Notification>>(requestBody);

        SendGridMessage message = new SendGridMessage();
        message.Subject = "Contoso device notification";

        var content = "The following device(s) have stopped sending telemetry:<br/><br/><table><tr><th>Device ID</th><th>Time</th></tr>";
        foreach(var notification in notifications) {
            log.LogInformation($"No message received - Device: {notification.deviceid}, Time: {notification.time}");
            content += $"<tr><td>{notification.deviceid}</td><td>{notification.time}</td></tr>";
        }
        content += "</table>";
        message.AddContent("text/html", content);

        return message;
    }

    public class Notification
    {
        public string deviceid { get; set; }
        public string time { get; set; }
    }
    ```

    Es posible que vea un mensaje de error hasta que guarde el nuevo código.

1. Seleccione **Guardar** para guardar la función.

### <a name="test-the-function-works"></a>Prueba del funcionamiento de la función

Para probar la función en el portal, elija **Registros** en la parte inferior del editor de código. A continuación, elija **Probar** a la derecha del editor de código. Use el siguiente archivo JSON como **Cuerpo de la solicitud**:

```json
[{"deviceid":"test-device-1","time":"2019-05-02T14:23:39.527Z"},{"deviceid":"test-device-2","time":"2019-05-02T14:23:50.717Z"},{"deviceid":"test-device-3","time":"2019-05-02T14:24:28.919Z"}]
```

Los mensajes del registro de la función aparecen en el panel **Registros**:

![Salida del registro de función](media/howto-create-custom-rules/function-app-logs.png)

Después de unos minutos, la dirección de correo electrónico de **destino** recibe un correo electrónico con el siguiente contenido:

```txt
The following device(s) have stopped sending telemetry:

Device ID    Time
test-device-1    2019-05-02T14:23:39.527Z
test-device-2    2019-05-02T14:23:50.717Z
test-device-3    2019-05-02T14:24:28.919Z
```

## <a name="add-stream-analytics-query"></a>Incorporación de una consulta de Stream Analytics

Esta solución utiliza una consulta de Stream Analytics para detectar cuándo un dispositivo ha dejado de enviar datos de telemetría durante más de 120 segundos. La consulta usa la telemetría del centro de eventos como entrada. El trabajo envía los resultados de consulta a la aplicación de función. En esta sección configurará un trabajo de Stream Analytics:

1. En Azure Portal, vaya a su trabajo de Stream Analytics, en **Topología de trabajo** seleccione **Entradas**, elija **+ Agregar entrada de flujo** y, luego, **Centro de eventos**.
1. Use la información de la tabla siguiente para configurar la entrada usando el centro de eventos que creó anteriormente, y luego elija **Guardar**:

    | Configuración | Value |
    | ------- | ----- |
    | Alias de entrada | centraltelemetry |
    | Subscription | Su suscripción |
    | Espacio de nombres del Centro de eventos | Espacio de nombres del centro de eventos |
    | Nombre del centro de eventos | Usar existente: **centralexport** |

1. En **Topología de trabajo**, seleccione **Salidas**, elija **+ Agregar**y, a continuación, elija **función de Azure**.
1. Use la información de la tabla siguiente para configurar la salida, y luego elija **Guardar**:

    | Configuración | Value |
    | ------- | ----- |
    | Alias de salida | emailnotification |
    | Subscription | Su suscripción |
    | Aplicación de función | Su aplicación de función |
    | Función  | HttpTrigger1 |

1. En **Topología de trabajo**, seleccione **Consulta** y reemplace la consulta existente con el siguiente SQL:

    ```sql
    with
    LeftSide as
    (
        SELECT
        -- Get the device ID from the message metadata and create a column
        GetMetadataPropertyValue([centraltelemetry], '[EventHub].[IoTConnectionDeviceId]') as deviceid1, 
        EventEnqueuedUtcTime AS time1
        FROM
        -- Use the event enqueued time for time-based operations
        [centraltelemetry] TIMESTAMP BY EventEnqueuedUtcTime
    ),
    RightSide as
    (
        SELECT
        -- Get the device ID from the message metadata and create a column
        GetMetadataPropertyValue([centraltelemetry], '[EventHub].[IoTConnectionDeviceId]') as deviceid2, 
        EventEnqueuedUtcTime AS time2
        FROM
        -- Use the event enqueued time for time-based operations
        [centraltelemetry] TIMESTAMP BY EventEnqueuedUtcTime
    )

    SELECT
        LeftSide.deviceid1 as deviceid,
        LeftSide.time1 as time
    INTO
        [emailnotification]
    FROM
        LeftSide
        LEFT OUTER JOIN
        RightSide 
        ON
        LeftSide.deviceid1=RightSide.deviceid2 AND DATEDIFF(second,LeftSide,RightSide) BETWEEN 1 AND 120
        where
        -- Find records where a device didn't send a message 120 seconds
        RightSide.deviceid2 is NULL
    ```

1. Seleccione **Guardar**.
1. Para iniciar el trabajo de Stream Analytics, elija **Información general**, después **Inicio** y **Ahora** y, a continuación, **Inicio**:

    ![Stream Analytics](media/howto-create-custom-rules/stream-analytics.png)

## <a name="configure-export-in-iot-central"></a>Configuración de la exportación en IoT Central

En el sitio web del [administrador de aplicaciones de Azure IoT Central](https://aka.ms/iotcentral), vaya a la aplicación IOT Central que creó a partir de la plantilla de Contoso. En esta sección va a configurar la aplicación para que haga streaming de los datos de telemetría desde sus dispositivos simulados al centro de eventos. Para configurar la exportación:

1. Vaya a la página **Exportación de datos**, seleccione **+ Nuevo** y, después, **Azure Event Hubs**.
1. Utilice los siguientes valores para configurar la exportación y, luego, seleccione **Guardar**:

    | Configuración | Value |
    | ------- | ----- |
    | Display Name (Nombre para mostrar) | Exportar a Event Hubs |
    | habilitado | Por |
    | Espacio de nombres de Event Hubs | El nombre de su espacio de nombres de Event Hubs |
    | Centro de eventos | centralexport |
    | Medidas | Por |
    | Dispositivos | Off |
    | Plantillas de dispositivo | Off |

![Configuración de la exportación de datos continua](media/howto-create-custom-rules/cde-configuration.png)

Espere hasta que el estado de la exportación sea **En ejecución** antes de continuar.

## <a name="test"></a>Prueba

Para probar la solución, puede deshabilitar la exportación continua de datos de IoT Central para los dispositivos simulados detenidos:

1. En la aplicación IoT Central, vaya hasta la página **Exportación de datos** y seleccione la configuración de exportación **Exportar a Event Hubs**.
1. Establezca **Habilitado** en **Desactivado** y elija **Guardar**.
1. Después de un mínimo de dos minutos la dirección de correo electrónico de **destino** recibe uno o más correos electrónicos que tienen un aspecto similar al ejemplo siguiente:

    ```txt
    The following device(s) have stopped sending telemetry:

    Device ID         Time
    Thermostat-Zone1  2019-11-01T12:45:14.686Z
    ```

## <a name="tidy-up"></a>Limpieza

Para hacer limpieza después de este artículo de procedimientos y evitar costos innecesarios, elimine el grupo de recursos **DetectStoppedDevices** de Azure Portal.

Puede eliminar la aplicación de IoT Central desde la página **Administración** de la aplicación.

## <a name="next-steps"></a>Pasos siguientes

En esta guía paso a paso, ha aprendido lo siguiente:

* Hacer streaming de datos de telemetría desde una aplicación de IoT Central mediante la *exportación de datos continua*.
* Crear una consulta de Stream Analytics que detecta cuándo un dispositivo ha dejado de enviar datos.
* Enviar una notificación por correo electrónico con los servicios de SendGrid y Azure Functions.

Ahora que sabe cómo crear notificaciones y reglas personalizadas, el siguiente paso que le sugerimos es que aprenda sobre la [Ampliación de Azure IoT Central con análisis personalizados](howto-create-custom-analytics.md).

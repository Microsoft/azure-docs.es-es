---
title: Ampliar Azure IoT Central con notificaciones y reglas personalizadas | Microsoft Docs
description: Como desarrollador de soluciones, configurar una aplicación IoT Central para enviar notificaciones por correo electrónico cuando un dispositivo deja de enviar datos de telemetría. Esta solución utiliza Azure Stream Analytics y Azure Functions.
author: dominicbetts
ms.author: dobett
ms.date: 05/14/2019
ms.topic: conceptual
ms.service: iot-central
services: iot-central
ms.custom: mvc
manager: philmea
ms.openlocfilehash: 6140a8aea3fe0fe0a8f1c01cd1c97404c41f7a69
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65805987"
---
# <a name="extend-azure-iot-central-with-custom-rules-that-send-notifications"></a>Ampliar Azure IoT Central con reglas personalizadas que envían notificaciones

Esta guía muestra, como desarrollador de soluciones, cómo ampliar la aplicación IoT Central con reglas personalizadas y notificaciones. El ejemplo muestra envía una notificación a un operador cuando un dispositivo deja de enviar datos de telemetría. La solución utiliza un [Azure Stream Analytics](https://docs.microsoft.com/azure/stream-analytics/) consulta para detectar cuándo ha detenido un dispositivo envía datos de telemetría. Usa el trabajo de los análisis de Stream [Azure Functions](https://docs.microsoft.com/azure/azure-functions/) enviar una notificación envía un correo electrónico mediante [SendGrid](https://sendgrid.com/docs/for-developers/partners/microsoft-azure/).

Esta guía muestra cómo ampliar IoT Central más allá de lo que puede hacer ya con las reglas integradas y las acciones.

En esta guía, obtendrá información sobre cómo:

* Stream telemetría desde una aplicación IoT Central mediante *exportación continua de datos*.
* Crear una consulta de Stream Analytics que detecta cuándo un dispositivo ha detenido el envío de datos.
* Enviar una notificación por correo electrónico con los servicios de SendGrid y Azure Functions.

## <a name="prerequisites"></a>Requisitos previos

Recuerde que para completar los pasos de esta guía paso a paso, necesita una suscripción activa a Azure.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

### <a name="iot-central-application"></a>Aplicación de IoT Central

Crear una aplicación IoT Central desde la [Azure IoT Central - mis aplicaciones](https://aka.ms/iotcentral) página con la configuración siguiente:

| Configuración | Valor |
| ------- | ----- |
| Plan de pago | Pago por uso |
| Plantilla de la aplicación | Contoso de ejemplo |
| Nombre de la aplicación | Acepte el valor predeterminado o elija su propio nombre |
| Dirección URL | Acepte el valor predeterminado o elija su propio prefijo de dirección URL única |
| Directorio | El inquilino de Azure Active Directory |
| Suscripción de Azure | Su suscripción de Azure |
| Área | Este de EE. UU |

Los ejemplos y capturas de pantalla en este artículo utilizan la **East US** región. Elija una ubicación cercana a usted y asegúrese de que crear todos los recursos en la misma región.

### <a name="resource-group"></a>Grupos de recursos

Use la [Azure portal para crear un grupo de recursos](https://portal.azure.com/#create/Microsoft.ResourceGroup) llamado **DetectStoppedDevices** para contener los otros recursos que cree. Cree los recursos de Azure en la misma ubicación que la aplicación IoT Central.

### <a name="event-hubs-namespace"></a>Espacio de nombres de los Event Hubs

Use la [Azure portal para crear un espacio de nombres de Event Hubs](https://portal.azure.com/#create/Microsoft.EventHub) con las siguientes opciones:

| Configuración | Valor |
| ------- | ----- |
| NOMBRE    | Elija el nombre de espacio de nombres |
| Plan de tarifa | Básica |
| Subscription | Su suscripción |
| Grupos de recursos | DetectStoppedDevices |
| Ubicación | Este de EE. UU |
| Unidades de procesamiento | 1 |

### <a name="stream-analytics-job"></a>Trabajo de Stream Analytics

Use la [Azure portal para crear un trabajo de Stream Analytics](https://portal.azure.com/#create/Microsoft.StreamAnalyticsJob) con las siguientes opciones:

| Configuración | Valor |
| ------- | ----- |
| NOMBRE    | Elija el nombre del trabajo |
| Subscription | Su suscripción |
| Grupos de recursos | DetectStoppedDevices |
| Ubicación | Este de EE. UU |
| Entorno de hospedaje | Nube |
| Unidades de streaming | 3 |

### <a name="function-app"></a>Aplicación de función

Use la [Azure portal para crear una aplicación de función](https://portal.azure.com/#create/Microsoft.FunctionApp) con las siguientes opciones:

| Configuración | Valor |
| ------- | ----- |
| Nombre de la aplicación    | Elija el nombre de la aplicación de función |
| Subscription | Su suscripción |
| Grupos de recursos | DetectStoppedDevices |
| SO |  Windows |
| Plan de hospedaje | Plan de consumo |
| Ubicación | Este de EE. UU |
| Pila en tiempo de ejecución | .NET |
| Almacenamiento | Crear nuevo |

### <a name="sendgrid-account"></a>Cuenta de SendGrid

Use la [Azure portal para crear una cuenta de SendGrid](https://portal.azure.com/#create/Sendgrid.sendgrid) con las siguientes opciones:

| Configuración | Valor |
| ------- | ----- |
| NOMBRE    | Elija el nombre de cuenta de SendGrid |
| Contraseña | Creación de una contraseña |
| Subscription | Su suscripción |
| Grupos de recursos | DetectStoppedDevices |
| Plan de tarifa | F1 Gratis |
| Información de contacto | Rellene la información necesaria |

Cuando haya creado todos los recursos necesarios, su **DetectStoppedDevices** grupo de recursos se parece a la captura de pantalla siguiente:

![Detectar grupo de recursos de los dispositivos detenido](media/howto-create-custom-rules/resource-group.png)

## <a name="create-an-event-hub"></a>Creación de un centro de eventos

Puede configurar una aplicación IoT Central para exportar continuamente datos de telemetría a un centro de eventos. En esta sección, creará un centro de eventos para recibir datos de telemetría desde su aplicación IoT Central. El centro de eventos entrega los datos de telemetría a su trabajo de Stream Analytics para su procesamiento.

1. En el portal de Azure, vaya al espacio de nombres de Event Hubs y seleccione **+ centro de eventos**.
1. Nombre de su centro de eventos **centralexport**y seleccione **crear**.

El espacio de nombres de Event Hubs se parece a la captura de pantalla siguiente:

![Espacio de nombres de los Event Hubs](media/howto-create-custom-rules/event-hubs-namespace.png)

## <a name="get-sendgrid-api-key"></a>Obtener clave de API de SendGrid

La aplicación de función necesita una clave de API de SendGrid para enviar mensajes de correo electrónico. Para crear una clave de API de SendGrid:

1. En el portal de Azure, vaya a la cuenta de SendGrid. A continuación, elija **administrar** para tener acceso a la cuenta de SendGrid.
1. En la cuenta de SendGrid elija **configuración**, a continuación, **claves de API**. Elija **crear clave de API**:

    ![Crear clave de API de SendGrid](media/howto-create-custom-rules/sendgrid-api-keys.png)

1. En el **Create API Key** página, cree una clave denominada **AzureFunctionAccess** con **acceso completo** permisos.
1. Tome nota de la clave de API, necesita al configurar la aplicación de función.

## <a name="define-the-function"></a>Definir la función

Esta solución usa una aplicación de Azure Functions para enviar una notificación por correo electrónico cuando el trabajo de Stream Analytics detecta un dispositivo detenido. Para crear la aplicación de función:

1. En el portal de Azure, vaya a la **App Service** de instancia en el **DetectStoppedDevices** grupo de recursos.
1. Seleccione **+** para crear una nueva función.
1. En el **elegir un entorno de desarrollo** página, elija **en el portal** y, a continuación, seleccione **continuar**.
1. En el **CREATE A FUNCTION** página, elija **Webhook y API** y, a continuación, seleccione **crear**.

El portal crea una función predeterminada denominada **HttpTrigger1**:

![Función de desencadenador HTTP predeterminada](media/howto-create-custom-rules/default-function.png)

### <a name="configure-function-bindings"></a>Configurar los enlaces de función

Para enviar correos electrónicos con SendGrid, deberá configurar los enlaces de la función como sigue:

1. Seleccione **integrar**, elija la salida **HTTP ($return)** y, a continuación, seleccione **eliminar**.
1. Elija **+ nueva salida**, a continuación, elija **SendGrid**y, a continuación, elija **seleccione**. Elija **instalar** para instalar la extensión de SendGrid.
1. Cuando se complete la instalación, seleccione **usar el valor devuelto de función**. Agregar válido **a dirección** para recibir notificaciones por correo electrónico.  Agregar válido **desde dirección** que se usará como el remitente de correo electrónico.
1. Seleccione **nueva** junto a **configuración de aplicación de clave de API de SendGrid**. Escriba **SendGridAPIKey** como clave y la clave de API de SendGrid se ha indicado anteriormente, como el valor. Seleccione **Crear**.
1. Elija **guardar** para guardar los enlaces de SendGrid para la función.

La configuración de integración de aspecto de la captura de pantalla siguiente:

![Integraciones de aplicaciones de función](media/howto-create-custom-rules/function-integrate.png)

### <a name="add-the-function-code"></a>Agregue el código de función

Para implementar la función, agregue el C# código para analizar la solicitud HTTP entrante y enviar los correos electrónicos de la manera siguiente:

1. Elija la **HttpTrigger1** funcionar en la aplicación de función y reemplazar el C# código con el código siguiente:

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

    Es posible que vea un mensaje de error hasta que se guarda el nuevo código.

1. Seleccione **guardar** para guardar la función.

### <a name="test-the-function-works"></a>Probar la función funciona

Para probar la función en el portal, elija **registros** en la parte inferior del editor de código. A continuación, elija **prueba** a la derecha del editor de código. Use el siguiente JSON como el **cuerpo de la solicitud**:

```json
[{"deviceid":"test-device-1","time":"2019-05-02T14:23:39.527Z"},{"deviceid":"test-device-2","time":"2019-05-02T14:23:50.717Z"},{"deviceid":"test-device-3","time":"2019-05-02T14:24:28.919Z"}]
```

Los mensajes de registro de la función aparecen en la **registros** panel:

![Salida del registro (función)](media/howto-create-custom-rules/function-app-logs.png)

Después de unos minutos, el **a** dirección de correo electrónico recibe un correo electrónico con el siguiente contenido:

```txt
The following device(s) have stopped sending telemetry:

Device ID   Time
test-device-1   2019-05-02T14:23:39.527Z
test-device-2   2019-05-02T14:23:50.717Z
test-device-3   2019-05-02T14:24:28.919Z
```

## <a name="add-stream-analytics-query"></a>Agregar consulta de Stream Analytics

Esta solución usa una consulta de Stream Analytics para detectar cuándo un dispositivo deja de enviar datos de telemetría durante más de 120 segundos. La consulta usa la telemetría desde el centro de eventos como entrada. El trabajo envía los resultados de consulta a la aplicación de función. En esta sección, configurará el trabajo de Stream Analytics:

1. En el portal de Azure, vaya a su trabajo de Stream analytics, en **topología trabajos** seleccione **entradas**, elija **+ Agregar entrada de flujo**y, a continuación, elija **eventos Concentrador**.
1. Use la información de la tabla siguiente para configurar la entrada usando el centro de eventos que creó anteriormente y luego elija **guardar**:

    | Configuración | Valor |
    | ------- | ----- |
    | Alias de entrada | centraltelemetry |
    | Subscription | Su suscripción |
    | Espacio de nombres del Centro de eventos | El espacio de nombres del centro de eventos |
    | Nombre del centro de eventos | Usar existente: **centralexport** |

1. En **topología trabajos**, seleccione **salidas**, elija **+ agregar**y, a continuación, elija **función Azure**.
1. Use la información en la tabla siguiente para configurar la salida, a continuación, elija **guardar**:

    | Configuración | Valor |
    | ------- | ----- |
    | Alias de salida | EmailNotification |
    | Subscription | Su suscripción |
    | Aplicación de función | La aplicación de función |
    | Función  | HttpTrigger1 |

1. En **topología trabajos**, seleccione **consulta** y reemplace la consulta existente por el siguiente código SQL:

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
1. Para iniciar el trabajo de Stream Analytics, elija **información general sobre**, a continuación, **iniciar**, a continuación, **ahora**y, a continuación, **iniciar**:

    ![Análisis de transmisiones](media/howto-create-custom-rules/stream-analytics.png)

## <a name="configure-export-in-iot-central"></a>Configurar la exportación en IoT Central

Navegue hasta la [aplicación IoT Central](https://aka.ms/iotcentral) creó a partir de la plantilla de Contoso. En esta sección, configurará la aplicación para transmitir los datos de telemetría desde sus dispositivos simulados al centro de eventos. Para configurar la exportación:

1. Navegue hasta la **exportación continua de datos** página, seleccione **+ nuevo**y, a continuación, **Azure Event Hubs**.
1. Utilice la siguiente configuración para configurar la exportación y luego seleccione **guardar**:

    | Configuración | Valor |
    | ------- | ----- |
    | Nombre para mostrar | Exportar a Centros de eventos |
    | Enabled | En |
    | Espacio de nombres de los Event Hubs | El nombre del espacio de nombres de Event Hubs |
    | Centro de eventos | centralexport |
    | Medidas | En |
    | Dispositivos | Desactivada |
    | Plantillas de dispositivo | Desactivada |

![Configuración de exportación continua de datos](media/howto-create-custom-rules/cde-configuration.png)

Espere hasta que el estado de exportación **ejecutando** antes de continuar.

## <a name="test"></a>Probar

Para probar la solución, puede deshabilitar la exportación continua de datos de IoT Central para los dispositivos simulados de detenido:

1. En la aplicación IoT Central, desplácese hasta la **exportación continua de datos** página y seleccione el **exportar a Event Hubs** exportar la configuración.
1. Establecer **habilitado** a **desactivar** y elija **guardar**.
1. Después de dos minutos como mínimo, el **a** dirección de correo electrónico recibe uno o más correos electrónicos que tienen un aspecto similar al ejemplo siguiente contenidos:

    ```txt
    The following device(s) have stopped sending telemetry:

    Device ID   Time
    7b169aee-c843-4d41-9f25-7a02671ee659    2019-05-09T14:28:59.954Z
    ```

## <a name="tidy-up"></a>Limpieza

Para limpiar después de este tema de procedimientos y evitar costos innecesarios, eliminar el **DetectStoppedDevices** grupo de recursos en el portal de Azure.

Puede eliminar la aplicación IoT Central desde la **administración** página dentro de la aplicación.

## <a name="next-steps"></a>Pasos siguientes

En esta guía paso a paso, ha aprendido lo siguiente:

* Stream telemetría desde una aplicación IoT Central mediante *exportación continua de datos*.
* Crear una consulta de Stream Analytics que detecta cuándo un dispositivo ha detenido el envío de datos.
* Enviar una notificación por correo electrónico con los servicios de SendGrid y Azure Functions.

Ahora que sabe cómo crear reglas personalizadas y notificaciones, el siguiente paso sugerido es obtener información sobre cómo [visualizar y analizar los datos de Azure IoT Central en un panel de Power BI](howto-connect-powerbi.md).

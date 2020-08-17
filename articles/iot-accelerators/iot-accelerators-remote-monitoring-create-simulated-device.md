---
title: Simulación de dispositivos con la supervisión remota de IoT - Azure | Microsoft Docs
description: En esta guía paso a paso, se muestra cómo usar el simulador de dispositivos con el acelerador de soluciones de supervisión remota.
author: dominicbetts
manager: timlt
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 03/08/2019
ms.topic: conceptual
ms.custom: mqtt, devx-track-javascript
ms.openlocfilehash: c0aac897d85cd96d537238bc81cbc89d3626a07c
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87422766"
---
# <a name="create-and-test-a-new-simulated-device"></a>Crear y probar un nuevo dispositivo simulado

El acelerador de soluciones de supervisión remota le permite definir sus propios dispositivos simulados. En este artículo muestra cómo definir un nuevo dispositivo simulado de Bombilla y, a continuación, probarlo localmente. El acelerador de soluciones incluye dispositivos simulados, como refrigeradores y camiones. Sin embargo, puede definir sus propios dispositivos simulados para probar sus soluciones de IoT antes de implementar dispositivos reales.

> [!NOTE]
> En este artículo se describe cómo usar dispositivos simulados hospedados en el servicio de simulación del dispositivo. Si desea crear un dispositivo real, consulte [Conexión del dispositivo al acelerador de soluciones de supervisión remota](iot-accelerators-connecting-devices.md).

En esta guía se muestra cómo personalizar el microservicio de simulación de dispositivos. Este microservicio es parte del acelerador de soluciones de supervisión remota. Para mostrar las funcionalidades del simulador de dispositivos, en esta guía paso a paso se usan dos escenarios en la aplicación IoT de Contoso:

En el primer escenario, debe agregar un tipo de telemetría nuevo al dispositivo **Refrigerador** existente de Contoso.

En el segundo escenario, Contoso quiere probar un nuevo dispositivo de bombilla inteligente. Para realizar las pruebas, cree un dispositivo simulado nuevo con las características siguientes:

*Propiedades*

| Nombre                     | Valores                      |
| ------------------------ | --------------------------- |
| Color                    | Blanco, rojo, azul            |
| Brillo               | De 0 a 100                    |
| Vida restante estimada | Cuenta atrás desde 10 000 horas |

*Telemetría*

En la tabla siguiente se muestran los datos de los que la bombilla informa a la nube como flujo de datos:

| Nombre   | Valores      |
| ------ | ----------- |
| Status | "on", "off" |
| Temperatura | Grados F |
| online (en línea) | true, false |

> [!NOTE]
> El valor de telemetría **online** (en línea) es obligatorio para todos los tipos simulados.

*Métodos*

La tabla siguiente muestra las acciones que admite el dispositivo nuevo:

| Nombre        |
| ----------- |
| Encender   |
| Apagar  |

*Estado inicial*

La tabla siguiente muestra el estado inicial del dispositivo:

| Nombre                     | Valores |
| ------------------------ | -------|
| Color inicial            | Blanco  |
| Brillo inicial       | 75     |
| Vida restante inicial   | 10 000 |
| Estado de telemetría inicial | "on"   |
| Temperatura de telemetría inicial | 200   |

Recuerde que para completar los pasos de esta guía paso a paso, necesita una suscripción activa a Azure.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="prerequisites"></a>Prerrequisitos

Para completar esta guía paso a paso, necesita:

* Código de Visual Studio. También puede [descargar Visual Studio Code para Mac, Linux y Windows](https://code.visualstudio.com/download).
* .NET Core. También puede descargar [.NET Core para Mac, Linux y Windows](https://www.microsoft.com/net/download).
* [C# para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
* Postman También puede descargar [Postman para Mac, Linux y Windows](https://www.getpostman.com/apps).
* Una instancia de [IoT Hub implementada en la suscripción de Azure](../../articles/iot-hub/iot-hub-create-through-portal.md). Necesita la cadena de conexión de IoT Hub para completar los pasos descritos en esta guía. Puede obtener la cadena de conexión de Azure Portal.
* Una base de datos de Cosmos DB que usa SQL API y que está configurada para tener una [coherencia fuerte](../../articles/cosmos-db/how-to-manage-database-account.md). Necesita la cadena de conexión de la base de datos de Cosmos DB para completar los pasos descritos en esta guía. Puede obtener la cadena de conexión de Azure Portal.

## <a name="prepare-your-development-environment"></a>Preparación del entorno de desarrollo

Complete las tareas siguientes para preparar el entorno de desarrollo:

* Descargue el código fuente para el microservicio de simulación de dispositivo.
* Descargue el código fuente para el microservicio del adaptador de almacenamiento.
* Ejecute el microservicio del adaptador de almacenamiento de manera local.

En las instrucciones de este artículo se asume que está usando Windows. Si usa otro sistema operativo, deberá ajustar algunas de las rutas de acceso de archivos y los comandos, para que se adapten a su entorno.

### <a name="download-the-microservices"></a>Descargar los microservicios

Descargue y descomprima los [microservicios de supervisión remota](https://github.com/Azure/remote-monitoring-services-dotnet/archive/master.zip) desde GitHub en una ubicación adecuada del equipo local. En este artículo, se presupone que el nombre de esta carpeta es **remote-monitoring-services-dotnet-master**.

Descargue y descomprima el [microservicio de simulación de dispositivo](https://github.com/Azure/device-simulation-dotnet/archive/master.zip) desde GitHub en una ubicación adecuada del equipo local. En este artículo, se presupone que el nombre de esta carpeta es **device-simulation-dotnet-master**.

### <a name="run-the-storage-adapter-microservice"></a>Ejecute el microservicio del adaptador de almacenamiento.

Abra la carpeta **remote-monitoring-services-dotnet-master\storage-adapter** en Visual Studio Code. Haga clic en cualquiera de los botones **Restaurar** para corregir cualquier dependencia sin resolver.

Abra el archivo **storage-adapter/WebService/appsettings.ini** y asigne la cadena de conexión de Cosmos DB a la variable **documentDBConnectionString**.

Para ejecutar el microservicio localmente, haga clic en **Depurar > Iniciar depuración**.

La ventana **Terminal** en Visual Studio Code muestra el resultado del microservicio en ejecución, que incluye una dirección URL para comprobar el estado del servicio web: [http://127.0.0.1:9022/v1/status](http://127.0.0.1:9022/v1/status). Cuando vaya a esta dirección, el estado debería ser "OK: activo y correcto".

Deje el microservicio del adaptador de almacenamiento ejecutándose en esta instancia de Visual Studio Code mientras completa los siguientes pasos.

## <a name="modify-the-chiller"></a>Modificar el refrigerador

En esta sección, agregará un nuevo tipo de telemetría de **temperatura interna** al tipo de dispositivo **Refrigerador** existente:

1. Cree una nueva carpeta  **C:\temp\devicemodels** en la máquina local.

1. Copie los siguientes archivos en su nueva carpeta desde la copia descargada del microservicio de simulación de dispositivo:

    | Source | Destination |
    | ------ | ----------- |
    | Services\data\devicemodels\chiller-01.json | C:\temp\devicemodels\chiller-01.json |
    | Services\data\devicemodels\scripts\chiller-01-state.js | C:\temp\devicemodels\scripts\chiller-01-state.js |
    | Services\data\devicemodels\scripts\Reboot-method.js | C:\temp\devicemodels\scripts\Reboot-method.js |
    | Services\data\devicemodels\scripts\FirmwareUpdate-method.js | C:\temp\devicemodels\scripts\FirmwareUpdate-method.js |
    | Services\data\devicemodels\scripts\EmergencyValveRelease-method.js | C:\temp\devicemodels\scripts\EmergencyValveRelease-method.js |
    | Services\data\devicemodels\scripts\IncreasePressure-method.js | C:\temp\devicemodels\scripts\IncreasePressure-method.js |

1. Abra el archivo **C:\temp\devicemodels\chiller-01.json**.

1. En la sección **InitialState**, agregue las dos definiciones siguientes:

    ```json
    "internal_temperature": 65.0,
    "internal_temperature_unit": "F",
    ```

1. En la matriz **Telemetría**, agregue la definición siguiente:

    ```json
    {
      "Interval": "00:00:05",
      "MessageTemplate": "{\"internal_temperature\":${internal_temperature},\"internal_temperature_unit\":\"${internal_temperature_unit}\"}",
      "MessageSchema": {
        "Name": "chiller-internal-temperature;v1",
        "Format": "JSON",
        "Fields": {
          "temperature": "double",
          "temperature_unit": "text"
        }
      }
    },
    ```

1. Guarde el archivo **C:\temp\devicemodels\chiller-01.json**.

1. Abra el archivo **C:\temp\devicemodels\scripts\chiller-01-state.js**.

1. Agregue los campos siguientes a la variable **state**:

    ```js
    internal_temperature: 65.0,
    internal_temperature_unit: "F",
    ```

1. Actualice la función **principal** tal como se indica a continuación:

    ```js
    function main(context, previousState, previousProperties) {

        // Restore the global state before generating the new telemetry, so that
        // the telemetry can apply changes using the previous function state.
        restoreSimulation(previousState, previousProperties);

        // 75F +/- 5%,  Min 25F, Max 100F
        state.temperature = vary(75, 5, 25, 100);

        // 70% +/- 5%,  Min 2%, Max 99%
        state.humidity = vary(70, 5, 2, 99);

        // 65F +/- 2%,  Min 15F, Max 125F
        state.internal_temperature = vary(65, 2, 15, 125);

        log("Simulation state: " + state.simulation_state);
        if (state.simulation_state === "high_pressure") {
            // 250 psig +/- 25%,  Min 50 psig, Max 300 psig
            state.pressure = vary(250, 25, 50, 300);
        } else {
            // 150 psig +/- 10%,  Min 50 psig, Max 300 psig
            state.pressure = vary(150, 10, 50, 300);
        }

        updateState(state);
        return state;
    }
    ```

1. Guarde el archivo **C:\temp\devicemodels\scripts\chiller-01-state.js**.

## <a name="create-the-lightbulb"></a>Crear la bombilla

En esta sección, se define un nuevo tipo de dispositivo **Bombilla**:

1. Cree un archivo **C:\temp\devicemodels\lightbulb-01.json** y agregue el siguiente contenido:

    ```json
    {
      "SchemaVersion": "1.0.0",
      "Id": "lightbulb-01",
      "Version": "0.0.1",
      "Name": "Lightbulb",
      "Description": "Smart lightbulb device.",
      "Protocol": "MQTT",
      "Simulation": {
        "InitialState": {
          "online": true,
          "temperature": 200.0,
          "temperature_unit": "F",
          "status": "on"
        },
        "Interval": "00:00:20",
        "Scripts": [
          {
            "Type": "javascript",
            "Path": "lightbulb-01-state.js"
          }
        ]
      },
      "Properties": {
        "Type": "Lightbulb",
        "Color": "White",
        "Brightness": 75,
        "EstimatedRemainingLife": 10000
      },
      "Tags": {
        "Location": "Building 2",
        "Floor": "2",
        "Campus": "Redmond"
      },
      "Telemetry": [
        {
          "Interval": "00:00:20",
          "MessageTemplate": "{\"temperature\":${temperature},\"temperature_unit\":\"${temperature_unit}\",\"status\":\"${status}\"}",
          "MessageSchema": {
            "Name": "lightbulb-status;v1",
            "Format": "JSON",
            "Fields": {
              "temperature": "double",
              "temperature_unit": "text",
              "status": "text"
            }
          }
        }
      ],
      "CloudToDeviceMethods": {
        "SwitchOn": {
          "Type": "javascript",
          "Path": "SwitchOn-method.js"
        },
        "SwitchOff": {
          "Type": "javascript",
          "Path": "SwitchOff-method.js"
        }
      }
    }
    ```

    Guarde los cambios en **C:\temp\devicemodels\lightbulb-01.json**.

1. Cree un archivo **C:\temp\devicemodels\scripts\lightbulb-01-state.js** y agregue el siguiente contenido:

    ```javascript
    "use strict";

    // Default state
    var state = {
      online: true,
      temperature: 200.0,
      temperature_unit: "F",
      status: "on"
    };

    // Default device properties
    var properties = {};

    /**
     * Restore the global state using data from the previous iteration.
     *
     * @param previousState device state from the previous iteration
     * @param previousProperties device properties from the previous iteration
     */
    function restoreSimulation(previousState, previousProperties) {
      // If the previous state is null, force a default state
      if (previousState) {
        state = previousState;
      } else {
        log("Using default state");
      }

      if (previousProperties) {
        properties = previousProperties;
      } else {
        log("Using default properties");
      }
    }

    /**
     * Simple formula generating a random value around the average
     * in between min and max
     *
     * @returns random value with given parameters
     */
    function vary(avg, percentage, min, max) {
      var value = avg * (1 + ((percentage / 100) * (2 * Math.random() - 1)));
      value = Math.max(value, min);
      value = Math.min(value, max);
      return value;
    }

    /**
     * Simple formula that sometimes flips the status of the lightbulb
     */
    function flip(value) {
      if (Math.random() < 0.2) {
        return (value == "on") ? "off" : "on"
      }
      return value;
    }

    /**
     * Entry point function called by the simulation engine.
     * Returns updated simulation state.
     * Device property updates must call updateProperties() to persist.
     *
     * @param context             The context contains current time, device model and id
     * @param previousState       The device state since the last iteration
     * @param previousProperties  The device properties since the last iteration
     */
    function main(context, previousState, previousProperties) {

      // Restore the global device properties and the global state before
      // generating the new telemetry, so that the telemetry can apply changes
      // using the previous function state.
      restoreSimulation(previousState, previousProperties);

      state.temperature = vary(200, 5, 150, 250);

      // Make this flip every so often
      state.status = flip(state.status);

      updateState(state);

      return state;
    }
    ```

    Guarde los cambios en **C:\temp\devicemodels\scripts\lightbulb-01-state.js**.

1. Cree un archivo **C:\temp\devicemodels\scripts\lightbulb-01-state.js** y agregue el siguiente contenido:

    ```javascript
    "use strict";

    // Default state
    var state = {
      status: "on"
    };

    /**
     * Entry point function called by the method.
     *
     * @param context        The context contains current time, device model and id
     * @param previousState  The device state since the last iteration
     * @param previousProperties  The device properties since the last iteration
     */
    function main(context, previousState) {
      log("Executing lightbulb Switch On method.");
      state.status = "on";
      updateState(state);
    }
    ```

    Guarde los cambios en **C:\temp\devicemodels\scripts\SwitchOn-method.js**.

1. Cree un archivo **C:\temp\devicemodels\scripts\SwitchOff-method.js** y agregue el siguiente contenido:

    ```javascript
    "use strict";

    // Default state
    var state = {
      status: "on"
    };

    /**
     * Entry point function called by the method.
     *
     * @param context        The context contains current time, device model and id
     * @param previousState  The device state since the last iteration
     * @param previousProperties  The device properties since the last iteration
     */
    function main(context, previousState) {
      log("Executing lightbulb Switch Off method.");
      state.status = "off";
      updateState(state);
    }
    ```

    Guarde los cambios en **C:\temp\devicemodels\scripts\SwitchOff-method.js**.

Ya ha creado una versión personalizada del tipo de dispositivo **Refrigerador** y creó un nuevo tipo de dispositivo **Bombilla**.

## <a name="test-the-devices"></a>Probar los dispositivos

En esta sección, probará los tipos de dispositivo que creó en las secciones anteriores de forma local.

### <a name="run-the-device-simulation-microservice"></a>Ejecutar el microservicio de simulación de dispositivo

Abra la carpeta **device-simulation-dotnet-master** que descargó de GitHub en una nueva instancia de Visual Studio Code. Haga clic en cualquiera de los botones **Restaurar** para corregir cualquier dependencia sin resolver.

Abra el archivo **WebService/appsettings.ini**, asigne la cadena de conexión de Cosmos DB a la variable **documentdb_connstring** y modifique la configuración como se indica a continuación:

```ini
device_models_folder = C:\temp\devicemodels\

device_models_scripts_folder = C:\temp\devicemodels\scripts\
```

Para ejecutar el microservicio localmente, haga clic en **Depurar > Iniciar depuración**.

La ventana **Terminal** en Visual Studio Code muestra el resultado del microservicio en ejecución.

Deje el microservicio de simulación de dispositivo ejecutándose en esta instancia de Visual Studio Code mientras completa los siguientes pasos.

### <a name="set-up-a-monitor-for-device-events"></a>Configurar un monitor de eventos de dispositivo

En esta sección, se utiliza la CLI de Azure para configurar un monitor de eventos para ver los datos de telemetría enviados desde los dispositivos conectados a IoT Hub.

En el siguiente script se da por supuesto que el nombre de su IoT Hub es**device-simulation-test**.

```azurecli-interactive
# Install the IoT extension if it's not already installed
az extension add --name azure-iot

# Monitor telemetry sent to your hub
az iot hub monitor-events --hub-name device-simulation-test
```

Deje el monitor de eventos en ejecución mientras prueba los dispositivos simulados.

### <a name="create-a-simulation-with-the-updated-chiller-device-type"></a>Crear una simulación con el tipo de dispositivo Refrigerador actualizado

En esta sección, use la herramienta Postman para solicitar el microservicio de simulación de dispositivo para ejecutar una simulación mediante el tipo de dispositivo Refrigerador actualizado. Postman es una herramienta que le permite enviar solicitudes de REST a un servicio web. Los archivos de configuración de Postman que necesita están en la copia local del repositorio **device-simulation-dotnet**.

Para configurar Postman:

1. Abra Postman en el equipo local.

1. Haga clic en **File > Import** (Archivo > Importar). A continuación, haga clic en **Choose Files** (Elegir archivos).

1. Vaya a la carpeta **device-simulation-dotnet-master/docs/postman**. Seleccione **Azure IoT Device Simulation solution accelerator.postman_collection** (accelerator.postman_collection de la solución Simulación de dispositivo de Azure IoT) y **Azure IoT Device Simulation solution accelerator.postman_environment** (accelerator.postman_environment de la solución Simulación de dispositivo de Azure IoT) y haga clic en **Open** (Abrir).

1. Expanda el **Acelerador de soluciones de simulación de dispositivos de Azure IoT** a las solicitudes que se puede enviar.

1. Haga clic en **No Environment** (Ningún entorno) y seleccione **Azure IoT Device Simulation solution accelerator** (Acelerador de soluciones de simulación de dispositivos de Azure IoT).

Ahora tiene una colección y un entorno cargados en el espacio de trabajo de Postman que puede usar para interactuar con el microservicio de simulación del dispositivo.

Para configurar y ejecutar la simulación:

1. En la colección de Postman, seleccione **Create modified chiller simulation** (Crear simulación de Refrigerador modificada) y haga clic en **Send** (Enviar). Esta solicitud crea cuatro instancias del tipo de dispositivo Refrigerador simulado.

1. La salida del monitor de eventos en la ventana de la CLI de Azure muestra la telemetría de los dispositivos simulados, incluidos los nuevos valores de **internal_temperature**.

Para detener la simulación, seleccione **Stop simulation** (Detener simulación)en Postman y haga clic en **Send** (Enviar).

### <a name="create-a-simulation-with-the-lightbulb-device-type"></a>Crear una simulación con el tipo de dispositivo Bombilla

En esta sección, use la herramienta Postman para solicitar el microservicio de simulación de dispositivo para ejecutar una simulación mediante el tipo de dispositivo Bombilla. Postman es una herramienta que le permite enviar solicitudes de REST a un servicio web.

Para configurar y ejecutar la simulación:

1. En la colección de Postman, seleccione **Create lightbulb simulation** (Crear simulación de Bombilla) y haga clic en **Send** (Enviar). Esta solicitud crea dos instancias del tipo de dispositivo Bombilla simulado.

1. La salida del monitor de eventos en la ventana de la CLI de Azure muestra la telemetría de los dispositivos de tipo Bombilla simulados.

Para detener la simulación, seleccione **Stop simulation** (Detener simulación)en Postman y haga clic en **Send** (Enviar).

## <a name="clean-up-resources"></a>Limpieza de recursos

Puede detener los dos microservicios que se ejecutan localmente en sus instancias de Visual Studio Code (**Depurar > Detener depuración**).

Si ya no necesita las instancias de IoT Hub y Cosmos DB, elimínelas de su suscripción de Azure para evitar cualquier cargo innecesario.

## <a name="next-steps"></a>Pasos siguientes

En esta guía se ha explicado cómo crear tipos de dispositivos simulados y personalizados y cómo probarlos mediante la ejecución local del microservicio de simulación de dispositivos.

El siguiente paso sugerido es obtener información sobre cómo implementar los tipos de dispositivos simulados y personalizados en el [Acelerador de soluciones de supervisión remota](iot-accelerators-remote-monitoring-deploy-simulated-device.md).

---
title: 'Utilización de búferes de protocolo con la simulación de dispositivo: Azure | Microsoft Docs'
description: En esta guía paso a paso, obtendrá información sobre cómo utilizar los búferes de protocolo para serializar los datos de telemetría enviados desde el acelerador de soluciones de simulación de dispositivo.
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.custom: mvc, amqp, devx-track-csharp
ms.date: 11/06/2018
ms.author: dobett
ms.openlocfilehash: c9c8aa86aa8a374a33750e306529ef212c9a8bfc
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89012337"
---
# <a name="serialize-telemetry-using-protocol-buffers"></a>Serialización de datos de telemetría con búferes de protocolo

Los búferes de protocolo (Protobuf) son un formato de serialización binaria para datos estructurados. Protobuf está diseñado para enfatizar la simplicidad y el rendimiento con el objetivo de ser más pequeño y más rápido que XML.

La simulación de dispositivo es compatible con la versión **proto3** del lenguaje de búferes de protocolo.

Dado que Protobuf requiere código compilado para serializar los datos, debe compilar una versión personalizada de simulación de dispositivo.

Los pasos de esta guía le muestran cómo:

1. Preparar un entorno de desarrollo
1. Especificar el formato Protobuf en un modelo de dispositivo
1. Definir el formato Protobuf
1. Generar clases Protobuf
1. Prueba local

## <a name="prerequisites"></a>Prerrequisitos

Para seguir los pasos de esta guía, necesita:

* Código de Visual Studio. También puede descargar [Visual Studio Code para Mac, Linux y Windows](https://code.visualstudio.com/download).
* .NET Core. También puede descargar [.NET Core para Mac, Linux y Windows](https://www.microsoft.com/net/download).
* Postman También puede descargar [Postman para Mac, Linux y Windows](https://www.getpostman.com/apps).
* Una instancia de [IoT Hub implementada en la suscripción de Azure](../iot-hub/iot-hub-create-through-portal.md). Necesita la cadena de conexión de IoT Hub para completar los pasos descritos en esta guía. Puede obtener la cadena de conexión de Azure Portal.
* Una [base de datos de Cosmos DB implementada en la suscripción de Azure](../cosmos-db/create-sql-api-dotnet.md#create-account) que use SQL API y que esté configurada para tener una [coherencia fuerte](../cosmos-db/manage-account.md). Necesita la cadena de conexión de la base de datos de Cosmos DB para completar los pasos descritos en esta guía. Puede obtener la cadena de conexión de Azure Portal.
* Una [cuenta de Azure Storage implementada en la suscripción a Azure](../storage/common/storage-account-create.md). Necesita la cadena de conexión de la cuenta de almacenamiento para completar los pasos descritos en esta guía. Puede obtener la cadena de conexión de Azure Portal.

## <a name="prepare-your-development-environment"></a>Preparación del entorno de desarrollo

Complete las tareas siguientes para preparar el entorno de desarrollo:

* Descargue el código fuente para el microservicio de simulación de dispositivo.
* Descargue el código fuente para el microservicio del adaptador de almacenamiento.
* Ejecute el microservicio del adaptador de almacenamiento de manera local.

En las instrucciones de este artículo se asume que está usando Windows. Si usa otro sistema operativo, deberá ajustar algunas de las rutas de acceso de archivos y los comandos, para que se adapten a su entorno.

### <a name="download-the-microservices"></a>Descargar los microservicios

Descargue y descomprima los [microservicios de supervisión remota](https://github.com/Azure/remote-monitoring-services-dotnet/archive/master.zip) desde GitHub en una ubicación adecuada del equipo local. Este repositorio incluye el microservicio de adaptador de almacenamiento que necesita para estas instrucciones.

Descargue y descomprima el [microservicio de simulación de dispositivo](https://github.com/Azure/device-simulation-dotnet/archive/master.zip) desde GitHub en una ubicación adecuada del equipo local.

### <a name="run-the-storage-adapter-microservice"></a>Ejecute el microservicio del adaptador de almacenamiento.

En Visual Studio Code, abra la carpeta **remote-monitoring-services-dotnet-master\storage-adapter**. Haga clic en cualquiera de los botones **Restaurar** para corregir las dependencias sin resolver.

Abra el archivo **.vscode/launch.json** y asigne su cadena de conexión de Cosmos DB a la variable de entorno **PCS\_STORAGEADAPTER\_DOCUMENTDB\_CONNSTRING**.

> [!NOTE]
> Al ejecutar el microservicio localmente en su equipo, requiere además una instancia de Cosmos DB en Azure para que funcione correctamente.

Para ejecutar el microservicio de adaptador de almacenamiento localmente, haga clic en **Depurar \> Iniciar depuración**.

La ventana **Terminal** en Visual Studio Code muestra el resultado del microservicio en ejecución, que incluye una dirección URL para comprobar el estado del servicio web: <http://127.0.0.1:9022/v1/status>. Cuando vaya a esta dirección, el estado debería ser "OK: activo y correcto".

Deje el microservicio de adaptador de almacenamiento en ejecución en esta instancia de Visual Studio Code mientras completa los pasos siguientes.

## <a name="define-your-device-model"></a>Definir el modelo de dispositivo

Abra la carpeta **device-simulation-dotnet-master** que descargó de GitHub en una nueva instancia de Visual Studio Code. Haga clic en cualquiera de los botones **Restaurar** para corregir cualquier dependencia sin resolver.

En esta guía paso a paso, crea un nuevo modelo de dispositivo para un rastreador de recursos:

1. Cree un nuevo archivo de modelo de dispositivo denominado **assettracker-01.json** en la carpeta **Services\data\devicemodels**.

1. Defina la funcionalidad del dispositivo en el archivo **assettracker-01.json** del modelo de dispositivo. La sección de telemetría de un modelo de dispositivo Protobuf debe:

   * Incluir el nombre de la clase Protobuf que genera para el dispositivo. En la sección siguiente se muestra cómo generar esta clase.
   * Especificar Protobuf como formato del mensaje.

     ```json
     {
     "SchemaVersion": "1.0.0",
     "Id": "assettracker-01",
     "Version": "0.0.1",
     "Name": "Asset Tracker",
     "Description": "An asset tracker with location, temperature, and humidity",
     "Protocol": "AMQP",
     "Simulation": {
       "InitialState": {
         "online": true,
         "latitude": 47.445301,
         "longitude": -122.296307,
         "temperature": 38.0,
         "humidity": 62.0
       },
       "Interval": "00:01:00",
       "Scripts": [
         {
           "Type": "javascript",
           "Path": "assettracker-01-state.js"
         }
       ]
     },
     "Properties": {
       "Type": "AssetTracker",
       "Location": "Field",
       "Latitude": 47.445301,
       "Longitude": -122.296307
     },
     "Telemetry": [
       {
         "Interval": "00:00:10",
         "MessageTemplate": "{\"latitude\":${latitude},\"longitude\":${longitude},\"temperature\":${temperature},\"humidity\":${humidity}}",
         "MessageSchema": {
           "Name": "assettracker-sensors;v1",
           "ClassName": "Microsoft.Azure.IoTSolutions.DeviceSimulation.Services.Models.Protobuf.AssetTracker",
           "Format": "Protobuf",
           "Fields": {
             "latitude": "double",
             "longitude": "double",
             "temperature": "double",
             "humidity": "double"
           }
         }
       }
     ]
     }
     ```

### <a name="create-device-behaviors-script"></a>Crear un script de comportamientos del dispositivo

Escriba el script de comportamientos que defina cómo se comporta el dispositivo. Para obtener más información, consulte [Create an advanced simulated device](iot-accelerators-device-simulation-advanced-device.md) (Crear un dispositivo simulado avanzado).

## <a name="define-your-protobuf-format"></a>Definir el formato Protobuf

Cuando tenga un modelo de dispositivo y haya determinado el formato del mensaje, puede crear un archivo **proto**. En el archivo **proto**, agregue:

* Un objeto `csharp_namespace` que coincida con la propiedad **ClassName** en el modelo de dispositivo.
* Un mensaje para cada estructura de datos que se va a serializar.
* Un nombre y tipo para cada campo del mensaje.

1. Cree un nuevo archivo denominado **assettracker.proto** en la carpeta **Services\Models\Protobuf\proto**.

1. Defina la sintaxis, el espacio de nombres y el esquema de mensaje en el archivo **proto**, de la manera siguiente:

    ```proto
    syntax = "proto3";

    option csharp_namespace = "Microsoft.Azure.IoTSolutions.DeviceSimulation.Services.Models.Protobuf";

    message AssetTracker {
      double latitude=1;
      double longitude=2;
      double temperature=5;
      double humidity=6;
    }
    ```

Los marcadores `=1` y `=2` de cada elemento especifican una etiqueta única que el campo usa en la codificación binaria. Los números del 1 al 15 requieren un byte menos para la codificación que los números más altos.

## <a name="generate-the-protobuf-class"></a>Generar la clase Protobuf

Cuando tiene un archivo **proto**, el siguiente paso es generar las clases necesarias para leer y escribir mensajes. Para completar este paso, necesita el compilador **Protoc** de Protobuf.

1. [Descargar el compilador de Protobuf desde GitHub](https://github.com/protocolbuffers/protobuf/releases/download/v3.4.0/protoc-3.4.0-win32.zip)

1. Ejecute el compilador, especificando el directorio de origen, el directorio de destino y el nombre de su archivo **proto**. Por ejemplo:

    ```cmd
    protoc -I c:\temp\device-simulation-dotnet-master\Services\Models\Protobuf\proto --csharp_out=C:\temp\device-simulation-dotnet-master\Services\Models\Protobuf assettracker.proto
    ```

    Este comando genera un archivo **Assettracker.cs** en la carpeta **Services\Models\Protobuf**.

## <a name="test-protobuf-locally"></a>Probar Protobuf localmente

En esta sección, probará el dispositivo de rastreador de recursos que creó en las secciones anteriores de forma local.

### <a name="run-the-device-simulation-microservice"></a>Ejecutar el microservicio de simulación de dispositivo

Abra el archivo **.vscode/launch.json** y asigne su:

* Cadena de conexión de IoT Hub a la variable de entorno **PCS\_IOTHUB\_CONNSTRING**.
* Cadena de conexión de la cuenta de almacenamiento a la variable de entorno **PCS\_AZURE\_STORAGE\_ACCOUNT**.
* Cadena de conexión de Cosmos DB a la variable de entorno **PCS\_STORAGEADAPTER\_DOCUMENTDB\_CONNSTRING**.

Abra el archivo **WebService/Properties/launchSettings.json** y asigne su:

* Cadena de conexión de IoT Hub a la variable de entorno **PCS\_IOTHUB\_CONNSTRING**.
* Cadena de conexión de la cuenta de almacenamiento a la variable de entorno **PCS\_AZURE\_STORAGE\_ACCOUNT**.
* Cadena de conexión de Cosmos DB a la variable de entorno **PCS\_STORAGEADAPTER\_DOCUMENTDB\_CONNSTRING**.

Abra el archivo **WebService\appsettings.ini** y modifique la configuración del siguiente modo:

#### <a name="configure-the-solution-to-include-your-new-device-model-files"></a>Configurar la solución para incluir los nuevos archivos de modelo de dispositivo

De forma predeterminada, los archivos JSON y JS del nuevo modelo de dispositivo no se copiarán en la solución compilada. Debe incluirlos explícitamente.

Agregue una entrada al archivo **services/services.csproj** para cada archivo que quiera incluir. Por ejemplo:

```xml
<None Update="data\devicemodels\assettracker-01.json">
    <CopyToOutputDirectory>Always</CopyToOutputDirectory>
</None>
<None Update="data\devicemodels\scripts\assettracker-01-state.js">
    <CopyToOutputDirectory>Always</CopyToOutputDirectory>
</None>
```

Para ejecutar el microservicio localmente, haga clic en **Depurar \> Iniciar depuración**.

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

### <a name="create-a-simulation-with-the-asset-tracker-device-type"></a>Crear una simulación con el tipo de dispositivo de rastreador de recursos

En esta sección, use la herramienta Postman para solicitar el microservicio de simulación de dispositivo para ejecutar una simulación con el tipo de dispositivo de rastreador de recursos. Postman es una herramienta que le permite enviar solicitudes de REST a un servicio web.

Para configurar Postman:

1. Abra Postman en el equipo local.

1. Haga clic en **File \> Import** (Archivo > Importar). A continuación, haga clic en **Choose Files** (Elegir archivos).

1. Seleccione **Azure IoT Device Simulation solution accelerator.postman\_collection** (accelerator.postman_collection de la solución de simulación de dispositivo de Azure IoT) y **Azure IoT Device Simulation solution accelerator.postman\_environment** (accelerator.postman_environment de la solución de simulación de dispositivo de Azure IoT) y haga clic en **Open (Abrir)** .

1. Expanda **Azure IoT Device Simulation solution accelerator** (Acelerador de soluciones de simulación de dispositivos de Azure IoT) para ver las solicitudes que se pueden enviar.

1. Haga clic en **No Environment** (Ningún entorno) y seleccione **Azure IoT Device Simulation solution accelerator** (Acelerador de soluciones de simulación de dispositivo de Azure IoT).

Ahora tiene una colección y un entorno cargados en el espacio de trabajo de Postman que puede usar para interactuar con el microservicio de simulación del dispositivo.

Para configurar y ejecutar la simulación:

1. En la colección de Postman, seleccione **Create asset tracker simulation** (Crear simulación de rastreador de recursos) y haga clic en **Send** (Enviar). Esta solicitud crea cuatro instancias del tipo de dispositivo de rastreador de recursos.

1. La salida del monitor de eventos en la ventana de la CLI de Azure muestra la telemetría de los dispositivos simulados.

Para detener la simulación, seleccione **Stop simulation** (Detener simulación)en Postman y haga clic en **Send** (Enviar).

### <a name="clean-up-resources"></a>Limpieza de recursos

Puede detener los dos microservicios que se ejecutan localmente en sus instancias de Visual Studio Code (**Depurar \> Detener depuración**).

Si ya no necesita las instancias de IoT Hub y Cosmos DB, elimínelas de su suscripción de Azure para evitar cualquier cargo innecesario.

## <a name="iot-hub-support"></a>Soporte técnico de IoT Hub

Muchas características de IoT Hub no admiten de forma nativa Protobuf u otros formatos binarios. Por ejemplo, no puede enrutar según la carga del mensaje porque IoT Hub no podrá procesar la carga del mensaje. Sin embargo, puede enrutar en función de los encabezados del mensaje.

## <a name="next-steps"></a>Pasos siguientes

Ha aprendido a personalizar la simulación de dispositivo para usar Protobuf a fin de enviar datos de telemetría; el siguiente paso es aprender a [implementar una imagen personalizada en la nube](iot-accelerators-device-simulation-deploy-image.md).

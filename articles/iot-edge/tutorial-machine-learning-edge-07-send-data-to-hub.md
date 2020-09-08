---
title: 'Tutorial: Envío de datos de dispositivo mediante una puerta de enlace transparente: Machine Learning en Azure IoT Edge'
description: Este tutorial le muestra cómo usar la máquina de desarrollo como dispositivo de IoT Edge simulado para enviar datos al centro de IoT a través de un dispositivo configurado como puerta de enlace transparente.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 6/30/2020
ms.topic: tutorial
ms.service: iot-edge
services: iot-edge
ms.custom: devx-track-csharp
ms.openlocfilehash: 78d3cbac28a85452f23e4a7c7ba3904fe3d1f488
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88997887"
---
# <a name="tutorial-send-data-via-transparent-gateway"></a>Tutorial: Envío de datos mediante una puerta de enlace transparente

> [!NOTE]
> Este artículo forma parte de un tutorial sobre el uso de Azure Machine Learning en IoT Edge. Si ha llegado directamente a este artículo, le recomendamos que comience con el [primer el artículo](tutorial-machine-learning-edge-01-intro.md) de la serie para obtener los mejores resultados.

En este artículo usaremos de nuevo la máquina virtual de desarrollo como un dispositivo simulado. Sin embargo, en lugar de enviar datos directamente a IoT Hub, el dispositivo envía los datos al dispositivo de IoT Edge configurado como puerta de enlace transparente.

Supervisamos el funcionamiento del dispositivo de IoT Edge mientras el dispositivo simulado envía datos. Una vez que el dispositivo termina de ejecutarse, nos centramos en los datos en nuestra cuenta de almacenamiento para validar que todo haya funcionado según lo previsto.

Normalmente, este paso está a cargo de un desarrollador de nube o de dispositivo.

## <a name="review-device-harness"></a>Revisar DeviceHarness

Reutiliza el [proyecto DeviceHarness](tutorial-machine-learning-edge-03-generate-data.md) para simular dispositivo de nivel inferior (u hoja). La conexión a la puerta de enlace transparente requiere dos cosas adicionales:

* Registrar el certificado para que el dispositivo IoT de nivel inferior confíe en la entidad de certificación que usa el entorno de ejecución de IoT Edge. En nuestro caso, el dispositivo de nivel inferior es la máquina virtual de desarrollo.
* Agrega el nombre de dominio completo (FQDN) de la puerta de enlace de perímetro a la cadena de conexión del dispositivo.

Examina el código para ver cómo se implementan estos dos elementos.

1. Abre Visual Studio Code en la máquina de desarrollo.

1. Use **archivo** > **Abrir carpeta…** para abrir C:\\origen\\IoTEdgeAndMlSample\\DeviceHarness.

1. Examina el método InstallCertificate() en Program.cs.

1. Ten en cuenta que, si el código encuentra la ruta de acceso del certificado, llama al método CertificateManager.InstallCACert para instalar el certificado en la máquina.

1. Ahora examina el método GetIotHubDevice en la clase TurbofanDevice.

1. Cuando el usuario especifica el FQDN de la puerta de enlace con la opción "-g", ese valor se pasa a este método como la variable `gatewayFqdn`, que se anexa a la cadena de conexión del dispositivo.

   ```csharp
   connectionString = $"{connectionString};GatewayHostName={gatewayFqdn.ToLower()}";
   ```

## <a name="build-and-run-leaf-device"></a>Compilar y ejecutar el dispositivo hoja

1. Con el proyecto DeviceHarness todavía abierto en Visual Studio Code, compile el proyecto. En el menú **Terminal**, seleccione **Ejecutar tarea de compilación** y después **Compilación**.

1. Para buscar el nombre de dominio completo (FQDN) para la puerta de enlace de Edge, vaya al dispositivo IoT Edge (Máquina virtual Linux) en Azure Portal y copie el valor de **Nombre de DNS** desde la página de información general.

1. Si aún no se está ejecutando, inicie el dispositivo IoT (máquina virtual Linux).

1. Abra el terminal Visual Studio Code. En el menú **Terminal**, seleccione **Nuevo terminal** y ejecute el siguiente comando, reemplazando `<edge_device_fqdn>` por el nombre DNS que copió del dispositivo IoT Edge (máquina virtual Linux):

   ```cmd
   dotnet run -- --gateway-host-name "<edge_device_fqdn>" --certificate C:\edgecertificates\certs\azure-iot-test-only.root.ca.cert.pem --max-devices 1
   ```

1. La aplicación intenta instalar el certificado en la máquina de desarrollo. Al hacerlo, acepta la advertencia de seguridad.

1. Cuando se le pida la cadena de conexión de IoT Hub, haga clic en los puntos suspensivos ( **…** ) en el panel de dispositivos de Azure IoT Hub y seleccione **Copy IoT Hub Connection String** (Copia cadena de conexión de IoT Hub). Pega el valor en el terminal.

1. Verás un resultado similar a:

   ```output
   Found existing device: Client_001
   Using device connection string: HostName=<your hub>.azure-devices.net;DeviceId=Client_001;SharedAccessKey=xxxxxxx; GatewayHostName=iotedge-xxxxxx.<region>.cloudapp.azure.com
   Device: 1 Message count: 50
   Device: 1 Message count: 100
   Device: 1 Message count: 150
   Device: 1 Message count: 200
   Device: 1 Message count: 250
   ```

   Ten en cuenta la adición de "GatewayHostName" a la cadena de conexión del dispositivo, lo que hace que el dispositivo se comunique con IoT Hub a través de la puerta de enlace transparente de IoT Edge.

## <a name="check-output"></a>Consulta la salida

### <a name="iot-edge-device-output"></a>Salida del dispositivo de IoT Edge

La salida del módulo avroFileWriter puede observarse fácilmente echando un vistazo al dispositivo de IoT Edge.

1. Usa SSH en la máquina virtual de IoT Edge.

1. Busca los archivos escritos en el disco.

   ```bash
   find /data/avrofiles -type f
   ```

1. La salida del comando tendrá un aspecto similar al ejemplo siguiente:

   ```output
   /data/avrofiles/2019/4/18/22/10.avro
   ```

   Es posible que haya más de un único archivo, según el tiempo de la ejecución.

1. Presta atención a las marcas de tiempo. El módulo avroFileWriter carga los archivos en la nube una vez que han transcurrido más de 10 minutos desde la hora de la última modificación (consulta MODIFIED\_FILE\_TIMEOUT en uploader.py en el módulo avroFileWriter).

1. Una vez que hayan transcurrido los 10 minutos, el módulo debería cargar los archivos. Si la carga es correcta, los archivos se eliminan del disco.

### <a name="azure-storage"></a>Azure Storage

Podemos observar los resultados de nuestro dispositivo hoja que envía datos si echamos un vistazo a las cuentas de almacenamiento adonde esperamos que se enruten los datos.

1. Abre Visual Studio Code en la máquina de desarrollo.

1. En el panel "AZURE STORAGE" en la ventana del explorador, vea al árbol para encontrar la cuenta de almacenamiento.

1. Expanda el nodo **Contenedores de blobs**.

1. A partir del trabajo que hicimos en la parte anterior del tutorial, esperamos que el contenedor **ruldata** contenga los mensajes con la vida útil restante. Expande el nodo **ruldata**.

1. Verás uno o varios archivos de blob denominados como: `<IoT Hub Name>/<partition>/<year>/<month>/<day>/<hour>/<minute>`.

1. Haz clic con el botón derecho en uno de los archivos y elige **Descargar blob** para guardar el archivo en la máquina de desarrollo.

1. Luego, expande el nodo **uploadturbofanfiles**. En el artículo anterior, establecimos esta ubicación como destino para los archivos cargados por el módulo avroFileWriter.

1. Haz clic con el botón derecho en los archivos y elige **Descargar blob** para guardarlo en la máquina de desarrollo.

### <a name="read-avro-file-contents"></a>Leer el contenido del archivo Avro

Incluimos una utilidad de línea de comandos sencilla para leer un archivo Avro y devolver una cadena JSON de los mensajes en el archivo. En esta sección, la instalaremos y ejecutaremos.

1. Abre un terminal en Visual Studio Code (**Terminal** > **Nuevo terminal**).

1. Instala hubavroreader:

   ```cmd
   pip install c:\source\IoTEdgeAndMlSample\HubAvroReader
   ```

1. Usa hubavroreader para leer el archivo Avro que descargaste de **ruldata**.

   ```cmd
   hubavroreader <avro file with ath> | more
   ```

1. Ten en cuenta que el cuerpo del mensaje se ve como lo esperábamos con el id. de dispositivo y la vida útil restante predicha.

   ```json
   {
       "Body": {
           "ConnectionDeviceId": "Client_001",
           "CorrelationId": "3d0bc256-b996-455c-8930-99d89d351987",
           "CycleTime": 1.0,
           "PredictedRul": 170.1723693909444
       },
       "EnqueuedTimeUtc": "<time>",
       "Properties": {
           "ConnectionDeviceId": "Client_001",
           "CorrelationId": "3d0bc256-b996-455c-8930-99d89d351987",
           "CreationTimeUtc": "01/01/0001 00:00:00",
           "EnqueuedTimeUtc": "01/01/0001 00:00:00"
       },
       "SystemProperties": {
           "connectionAuthMethod": "{\"scope\":\"module\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
           "connectionDeviceGenerationId": "636857841798304970",
           "connectionDeviceId": "aaTurbofanEdgeDevice",
           "connectionModuleId": "turbofanRouter",
           "contentEncoding": "utf-8",
           "contentType": "application/json",
           "correlationId": "3d0bc256-b996-455c-8930-99d89d351987",
           "enqueuedTime": "<time>",
           "iotHubName": "mledgeiotwalkthroughhub"
       }
   }
   ```

1. Ejecuta el mismo comando pasando el archivo Avro que descargaste de **uploadturbofanfiles**.

1. Según lo previsto, estos mensajes contienen todos los datos de los sensores y la configuración operativa del mensaje original. Estos datos se podrían usar para mejorar el modelo RUL en el dispositivo de borde.

   ```json
   {
       "Body": {
           "CycleTime": 1.0,
           "OperationalSetting1": -0.0005000000237487257,
           "OperationalSetting2": 0.00039999998989515007,
           "OperationalSetting3": 100.0,
           "PredictedRul": 170.17236328125,
           "Sensor1": 518.6699829101562,
           "Sensor10": 1.2999999523162842,
           "Sensor11": 47.29999923706055,
           "Sensor12": 522.3099975585938,
           "Sensor13": 2388.010009765625,
           "Sensor14": 8145.31982421875,
           "Sensor15": 8.424599647521973,
           "Sensor16": 0.029999999329447746,
           "Sensor17": 391.0,
           "Sensor18": 2388.0,
           "Sensor19": 100.0,
           "Sensor2": 642.3599853515625,
           "Sensor20": 39.11000061035156,
           "Sensor21": 23.353700637817383,
           "Sensor3": 1583.22998046875,
           "Sensor4": 1396.8399658203125,
           "Sensor5": 14.619999885559082,
           "Sensor6": 21.610000610351562,
           "Sensor7": 553.969970703125,
           "Sensor8": 2387.9599609375,
           "Sensor9": 9062.169921875
       },
           "ConnectionDeviceId": "Client_001",
           "CorrelationId": "70df0c98-0958-4c8f-a422-77c2a599594f",
           "CreationTimeUtc": "0001-01-01T00:00:00+00:00",
           "EnqueuedTimeUtc": "<time>"
   }
   ```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si tienes previsto explorar los recursos usados por este tutorial completo, espera hasta terminar de limpiar los recursos que has creado. De lo contrario, siga estos pasos para eliminar los recursos:

1. Elimine los grupos de recursos creados para contener la máquina virtual de desarrollo, la máquina virtual de IoT Edge, IoT Hub, la cuenta de almacenamiento, el servicio de área de trabajo de aprendizaje automático (y los recursos creados: registro de contenedor, Application Insights, almacén de claves, cuenta de almacenamiento).

1. Elimine el proyecto de aprendizaje automático en [Azure Notebooks](https://notebooks.azure.com).

1. Si has clonado el repositorio localmente, cierra las ventanas de PowerShell o VS Code que hagan referencia al repositorio local y, luego, elimina el directorio del repositorio.

1. Si has creado los certificados de forma local, elimina la carpeta c:\\edgeCertificates.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, hemos usado la máquina virtual de desarrollo para simular un dispositivo hoja que envía dados operativos y de sensores al dispositivo IoT Edge. Hemos validado que los módulos en el dispositivo enrutaran, clasificaran, persistieran y cargaran los datos mediante el examen de la operación en tiempo real del dispositivo perimetral y echando un vistazo a los archivos cargados en la cuenta de almacenamiento.

Puedes encontrar más información en las páginas siguientes:

* [Conexión de un dispositivo de bajada a una puerta de enlace Azure IoT Edge](how-to-connect-downstream-device.md)
* [Almacene datos en el perímetro con Azure Blob Storage en IoT Edge (versión preliminar)](how-to-store-data-blob.md)

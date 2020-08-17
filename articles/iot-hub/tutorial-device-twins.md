---
title: Sincronización del estado del dispositivo desde Azure IoT Hub | Microsoft Docs
description: Aprenda a usar dispositivos gemelos para configurar los dispositivos desde la nube y recibir datos de estado y cumplimiento de los dispositivos.
services: iot-hub
author: wesmc7777
ms.author: wesmc
ms.service: iot-hub
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 06/21/2019
ms.custom:
- mvc
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
- devx-track-javascript
ms.openlocfilehash: f3dad81a5cba9dd817e0d4e75590d374fe059358
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87424110"
---
<!-- **TODO** Update publish config with repo paths before publishing! -->

# <a name="tutorial-configure-your-devices-from-a-back-end-service"></a>Tutorial: Configuración de dispositivos desde un servicio back-end

Además de la recepción de telemetría de los dispositivos, puede que necesite configurar los dispositivos desde su servicio back-end. Cuando envíe una configuración deseada a los dispositivos, también puede recibir actualizaciones de estado y cumplimiento de esos mismos dispositivos. Por ejemplo, puede establecer un intervalo de temperatura operativa que desee como objetivo para un dispositivo, o puede recopilar información de versión de firmware de los dispositivos.

Para sincronizar la información de estado entre un dispositivo y un centro de IoT, use _dispositivos gemelos_. Un [dispositivo gemelo](iot-hub-devguide-device-twins.md) es un documento JSON, asociado con un dispositivo específico y almacenado por IoT Hub en la nube en donde puede [consultarlo](iot-hub-devguide-query-language.md). Un dispositivo gemelo contiene _propiedades deseadas_, _propiedades notificadas_ y _etiquetas_. Una propiedad deseada la establece una aplicación back-end y la lee un dispositivo. Una propiedad notificada la establece un dispositivo y la lee una aplicación back-end. Una etiqueta la establece una aplicación back-end y nunca se envía a un dispositivo. Las etiquetas se usan para organizar los dispositivos. Este tutorial muestra cómo utilizar propiedades deseadas y notificadas para sincronizar la información de estado:

![Resumen de dispositivos gemelos](media/tutorial-device-twins/DeviceTwins.png)

En este tutorial se realizan las siguientes tareas:

> [!div class="checklist"]
> * Creación de un centro de IoT e incorporación de un dispositivo de prueba en el registro de identidades.
> * Uso de propiedades deseadas para enviar la información de estado para el dispositivo simulado.
> * Uso de propiedades notificadas para recibir la información de estado del dispositivo simulado.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Las dos aplicaciones de ejemplo que se ejecutan en este inicio rápido se escriben con Node.js. Necesitará Node.js v10.x.x o una versión posterior en la máquina de desarrollo.

Puede descargar Node.js para varias plataformas desde [nodejs.org](https://nodejs.org).

Puede verificar la versión actual de Node.js en el equipo de desarrollo con el comando siguiente:

```cmd/sh
node --version
```

Descargue el proyecto de muestra de Node.js desde https://github.com/Azure-Samples/azure-iot-samples-node/archive/master.zip y extraiga el archivo ZIP.

Asegúrese de que está abierto el puerto 8883 del firewall. En el dispositivos de ejemplo de este tutorial se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y para saber cómo solucionar este problema, consulte el artículo sobre la [conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="set-up-azure-resources"></a>Configuración de los recursos de Azure

Para completar este tutorial, su suscripción de Azure tiene que contener un centro de IoT con un dispositivo agregado en el registro de identidad del dispositivo. La entrada en el registro de identidad del dispositivo permite que el dispositivo simulado que se ejecuta en este tutorial se conecte con el centro.

Si aún no tiene un centro de IoT configurado en su suscripción, puede configurar uno con el siguiente script de la CLI. Este script utiliza el nombre **tutorial-iot-hub** para el centro de IoT, cuando lo ejecute, tiene que reemplazar este nombre por otro que sea exclusivo. El script crea el grupo de recursos y el centro en la región **Centro de EE. UU.** , que usted puede cambiar a otra región más cercana. El script recupera la cadena de conexión de servicio del centro de IoT, que se utiliza en el ejemplo de back-end para conectar con el centro de IoT:

```azurecli-interactive
hubname=tutorial-iot-hub
location=centralus

# Install the IoT extension if it's not already installed:
az extension add --name azure-iot

# Create a resource group:
az group create --name tutorial-iot-hub-rg --location $location

# Create your free-tier IoT Hub. You can only have one free IoT Hub per subscription:
az iot hub create --name $hubname --location $location --resource-group tutorial-iot-hub-rg --sku F1

# Make a note of the service connection string, you need it later:
az iot hub show-connection-string --name $hubname --policy-name service -o table

```

Este tutorial usa un dispositivo simulado llamado **MyTwinDevice**. El siguiente script agrega este dispositivo al registro de identidades y recupera su cadena de conexión:

```azurecli-interactive
# Set the name of your IoT hub:
hubname=tutorial-iot-hub

# Create the device in the identity registry:
az iot hub device-identity create --device-id MyTwinDevice --hub-name $hubname --resource-group tutorial-iot-hub-rg

# Retrieve the device connection string, you need this later:
az iot hub device-identity show-connection-string --device-id MyTwinDevice --hub-name $hubname --resource-group tutorial-iot-hub-rg -o table

```

## <a name="send-state-information"></a>Envío de la información de estado

Para enviar la información de estado de una aplicación back-end a un dispositivo se usan las propiedades deseadas. En esta sección verá como:

* Recibir y procesar las propiedades deseadas en un dispositivo.
* Enviar propiedades deseadas desde una aplicación back-end.

Para ver el código de ejemplo del dispositivo simulado que recibe las propiedades deseadas, vaya a la carpeta **iot-hub/Tutorials/DeviceTwins** del proyecto de Node.js de ejemplo que descargó. Luego abra el archivo SimulatedDevice.js. con un editor de texto.

Las secciones siguientes describen el código que se ejecuta en el dispositivo simulado que responde a los cambios de propiedad deseada enviados desde la aplicación back-end:

### <a name="retrieve-the-device-twin-object"></a>Recuperación del objeto de dispositivo gemelo

El código siguiente se conecta con el centro de IoT mediante una cadena de conexión de dispositivo:

[!code-javascript[Create IoT Hub client](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/SimulatedDevice.js?name=createhubclient&highlight=2 "Create IoT Hub client")]

El código siguiente obtiene un gemelo desde el objeto de cliente:

[!code-javascript[Get twin](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/SimulatedDevice.js?name=gettwin&highlight=2 "Get twin")]

### <a name="sample-desired-properties"></a>Propiedades deseadas de ejemplo

Puede estructurar las propiedades deseadas de la forma que más le convenga a la aplicación. Este ejemplo utiliza una propiedad de nivel superior denominada **fanOn** y agrupa las propiedades restantes en diferentes **componentes**. El siguiente fragmento JSON muestra la estructura de las propiedades deseadas que utiliza este tutorial:

[!code[Sample desired properties](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/desired.json "Sample desired properties")]

### <a name="create-handlers"></a>Creación de controladores

Puede crear controladores para las actualizaciones de la propiedad deseada que responden a las actualizaciones a diferentes niveles en la jerarquía JSON. Por ejemplo, este controlador ve todos los cambios de propiedad deseada enviados al dispositivo desde una aplicación back-end. La variable **delta** contiene las propiedades deseadas enviadas desde el back-end de soluciones:

[!code-javascript[Handle all properties](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/SimulatedDevice.js?name=allproperties&highlight=2 "Handle all properties")]

El siguiente controlador solo responde a los cambios realizados en la propiedad deseada **fanOn**:

[!code-javascript[Handle fan property](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/SimulatedDevice.js?name=fanproperty&highlight=2 "Handle fan property")]

### <a name="handlers-for-multiple-properties"></a>Controladores para varias propiedades

En el fragmento JSON de las propiedades deseadas de ejemplo que se ha mostrado anteriormente, el nodo **climate** de **components** contiene dos propiedades, **minTemperature** y  **maxTemperature**.

Un objeto **gemelo** local de un dispositivo almacena un conjunto completo de propiedades deseadas y notificadas. El **delta** enviado desde el back-end podría actualizar solo un subconjunto de las propiedades deseadas. En el siguiente fragmento de código, si el dispositivo simulado recibe una actualización de una sola de las propiedades **minTemperature** y **maxTemperature**, usa el valor en el gemelo local para el otro valor para poder configurar el dispositivo:

[!code-javascript[Handle climate component](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/SimulatedDevice.js?name=climatecomponent&highlight=2 "Handle climate component")]

El objeto **gemelo** local almacena un conjunto completo de propiedades deseadas y notificadas. El **delta** enviado desde el back-end podría actualizar solo un subconjunto de las propiedades deseadas.

### <a name="handle-insert-update-and-delete-operations"></a>Tratamiento de las operaciones de inserción, actualización y eliminación

Las propiedades deseadas enviadas desde el back-end no indican qué operación se realiza en una propiedad deseada específica. El código tiene que deducir la operación desde el conjunto actual de propiedades deseadas almacenadas localmente y los cambios que se enviaron desde el centro.

El fragmento de código siguiente muestra cómo el dispositivo simulado controla las operaciones de inserción, actualización y eliminación en la lista de **componentes** en las propiedades deseadas. Puede ver cómo usar valores **null** para indicar que un componente debe eliminarse:

[!code-javascript[Handle components](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/SimulatedDevice.js?name=components&highlight=2,6,13 "Handle components")]

### <a name="send-desired-properties-to-a-device-from-the-back-end"></a>Envío de propiedades deseadas a un dispositivo desde el back-end

Ha visto cómo un dispositivo implementa controladores para recibir actualizaciones de propiedad deseada. En esta sección se muestra cómo enviar los cambios de propiedad deseada a un dispositivo desde una aplicación back-end.

Para ver el código de ejemplo del dispositivo simulado que recibe las propiedades deseadas, vaya a la carpeta **iot-hub/Tutorials/DeviceTwins** del proyecto de Node.js de ejemplo que descargó. Luego abra el archivo ServiceClient.js. con un editor de texto.

El fragmento de código siguiente muestra cómo conectar con el registro de identidad del dispositivo y acceder al gemelo para un dispositivo específico:

[!code-javascript[Create registry and get twin](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/ServiceClient.js?name=getregistrytwin&highlight=2,6 "Create registry and get twin")]

El siguiente fragmento muestra otras *revisiones* de propiedad deseada que la aplicación back-end envía al dispositivo:

[!code-javascript[Patches sent to device](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/ServiceClient.js?name=patches&highlight=2,12,26,41,56 "Patches sent to device")]

El siguiente fragmento muestra cómo la aplicación back-end envía una actualización de propiedad deseada a un dispositivo:

[!code-javascript[Send desired properties](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/ServiceClient.js?name=senddesiredproperties&highlight=2 "Send desired properties")]

### <a name="run-the-applications"></a>Ejecución de las aplicaciones

En esta sección, se ejecutan dos aplicaciones de ejemplo para observar como una aplicación back-end envía las actualizaciones de propiedad deseada a una aplicación de dispositivo simulado.

Para ejecutar el dispositivo simulado y las aplicaciones back-end, necesita las cadenas de conexión de dispositivo y servicio. Cuando creó los recursos al principio de este tutorial, tomó nota de las cadenas de conexión.

Para ejecutar la aplicación de dispositivo simulado, abra una ventana del shell o del símbolo del sistema y vaya a la carpeta **iot-hub/Tutorials/DeviceTwins** en el proyecto Node.js que descargó. Luego, ejecute los siguientes comandos:

```cmd/sh
npm install
node SimulatedDevice.js "{your device connection string}"
```

Para ejecutar la aplicación back-end, abra otra ventana del shell o del símbolo del sistema. A continuación, vaya a la carpeta **iot-hub/Tutorials/DeviceTwins** del proyecto de Node.js que ha descargado. Luego, ejecute los siguientes comandos:

```cmd/sh
npm install
node ServiceClient.js "{your service connection string}"
```

La siguiente captura de pantalla muestra el resultado de la aplicación del dispositivo simulado y resalta cómo trata una actualización de la propiedad deseada **maxTemperature**. Puede ver cómo se ejecutan el controlador de nivel superior y los controladores del componente climate:

![Dispositivo simulado](./media/tutorial-device-twins/SimulatedDevice1.png)

La siguiente captura de pantalla muestra el resultado de la aplicación back-end y resalta cómo envía una actualización a la propiedad deseada **maxTemperature**:

![Aplicación back-end](./media/tutorial-device-twins/BackEnd1.png)

## <a name="receive-state-information"></a>Recepción de la información de estado

La aplicación back-end recibe información de estado de un dispositivo como propiedades notificadas. Un dispositivo establece las propiedades notificadas y las envía al centro. Una aplicación back-end puede leer los valores actuales de las propiedades notificadas desde el dispositivo gemelo almacenado en el centro.

### <a name="send-reported-properties-from-a-device"></a>Envío de propiedades notificadas desde un dispositivo

Puede enviar actualizaciones a valores de propiedad notificada tales como una revisión. El siguiente fragmento muestra una plantilla para la revisión que envía el dispositivo simulado. El dispositivo simulado actualiza los campos de la revisión antes de enviarla al centro:

[!code-javascript[Reported properties patches](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/SimulatedDevice.js?name=reportedpatch&highlight=2 "Reported properties patches")]

El dispositivo simulado utiliza la siguiente función para enviar la revisión que contiene las propiedades notificadas al centro:

[!code-javascript[Send reported properties](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/SimulatedDevice.js?name=sendreportedproperties&highlight=2 "Send reported properties")]

### <a name="process-reported-properties"></a>Procesamiento de propiedades notificadas

Una aplicación back-end tiene acceso a los valores de propiedad notificada actuales para un dispositivo a través del dispositivo gemelo. El siguiente fragmento muestra cómo la aplicación back-end lee los valores de propiedad notificada para el dispositivo simulado:

[!code-javascript[Display reported properties](~/iot-samples-node/iot-hub/Tutorials/DeviceTwins/ServiceClient.js?name=displayreportedproperties&highlight=2 "Display reported properties")]

### <a name="run-the-applications"></a>Ejecución de las aplicaciones

En esta sección, se ejecutan dos aplicaciones de ejemplo para observar como una aplicación de dispositivo simulado envía las actualizaciones de propiedad notificada a una aplicación back-end.

Se ejecutan las mismas dos aplicaciones de ejemplo que se ejecutaron para ver cómo las propiedades deseadas se envían a un dispositivo.

Para ejecutar el dispositivo simulado y las aplicaciones back-end, necesita las cadenas de conexión de dispositivo y servicio. Cuando creó los recursos al principio de este tutorial, tomó nota de las cadenas de conexión.

Para ejecutar la aplicación de dispositivo simulado, abra una ventana del shell o del símbolo del sistema y vaya a la carpeta **iot-hub/Tutorials/DeviceTwins** en el proyecto Node.js que descargó. Luego, ejecute los siguientes comandos:

```cmd/sh
npm install
node SimulatedDevice.js "{your device connection string}"
```

Para ejecutar la aplicación back-end, abra otra ventana del shell o del símbolo del sistema. A continuación, vaya a la carpeta **iot-hub/Tutorials/DeviceTwins** del proyecto de Node.js que ha descargado. Luego, ejecute los siguientes comandos:

```cmd/sh
npm install
node ServiceClient.js "{your service connection string}"
```

La siguientes captura de pantalla muestra el resultado de la aplicación de dispositivo simulado y resalta cómo envía una actualización de la propiedad notificada al centro:

![Dispositivo simulado](./media/tutorial-device-twins/SimulatedDevice2.png)

La siguiente captura de pantalla muestra la salida de la aplicación back-end y resalta cómo recibe y procesa una actualización de la propiedad notificada desde un dispositivo:

![Aplicación back-end](./media/tutorial-device-twins/BackEnd2.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si tiene pensado completar el siguiente tutorial, deje el centro de IoT y el grupo de recursos y vuelva a usarlos más adelante.

Si ya no los va a necesitar más, elimínelos en el portal. Para ello, seleccione el grupo de recursos **tutorial-iot-hub-rg** que contiene el centro de IoT y haga clic en **Eliminar**.

Como alternativa, use la CLI:

```azurecli-interactive
# Delete your resource group and its contents
az group delete --name tutorial-iot-hub-rg
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a sincronizar la información de estado entre los dispositivos y el centro de IoT. Avance hasta el siguiente tutorial para aprender a usar dispositivos gemelos para implementar un proceso de actualización de firmware.

> [!div class="nextstepaction"]
> [Implementación de un proceso de actualización de firmware de dispositivos](tutorial-firmware-update.md)

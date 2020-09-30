---
title: Uso de una plantilla de ARM para publicar en IoT Hub, crear una cuenta de almacenamiento y enrutar mensajes
description: Uso de una plantilla de ARM para publicar en IoT Hub, crear una cuenta de almacenamiento y enrutar mensajes
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: quickstart
ms.date: 08/24/2020
ms.author: robinsh
ms.custom: mvc, subject-armqs
ms.openlocfilehash: 4112e8aae485e229beb16d21e90280750e1465e1
ms.sourcegitcommit: 4a7a4af09f881f38fcb4875d89881e4b808b369b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89462673"
---
# <a name="quickstart-deploy-an-azure-iot-hub-and-a-storage-account-using-an-arm-template"></a>Inicio rápido: Implementación de un centro de Azure IoT Hub y una cuenta de almacenamiento mediante una plantilla de ARM

En esta guía de inicio rápido, usará una plantilla de Azure Resource Manager (plantilla de ARM) para crear un centro de IoT que enrute los mensajes a Azure Storage y una cuenta de almacenamiento que contenga los mensajes. Después de agregar manualmente un dispositivo IoT virtual al centro para enviar los mensajes, configure esa información de conexión en una aplicación llamada *arm-read-write* para enviar mensajes desde el dispositivo al centro. El centro se configura de modo que los mensajes enviados al centro se enruten automáticamente a la cuenta de almacenamiento. Al final de esta guía de inicio rápido, podrá abrir la cuenta de almacenamiento y ver los mensajes enviados.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en esta guía de inicio rápido se llama `101-iothub-auto-route-messages` y forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-iothub-auto-route-messages).

:::code language="json" source="~/quickstart-templates/101-iothub-auto-route-messages/azuredeploy.json":::

En la plantilla se definen dos recursos de Azure: 
* [Microsoft.Devices/Iothubs](/azure/templates/microsoft.devices/iothubs)
* [Microsoft.Storage/](/azure/templates/microsoft.storage/allversions)

## <a name="deploy-the-template-and-run-the-sample-app"></a>Implementación de la plantilla y ejecución de la aplicación de ejemplo

En esta sección se proporcionan los pasos para implementar la plantilla, crear un dispositivo virtual y ejecutar la aplicación arm-read-write para enviar los mensajes.

1. Cree los recursos mediante la implementación de la plantilla de ARM.

    > [!TIP]
    > Seleccione el siguiente botón para iniciar la implementación de la plantilla. Mientras está en ejecución, configure la aplicación arm-read-write para que se ejecute.

    [![Implementar en Azure](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.svg?sanitize=true)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-iothub-auto-route-messages%2Fazuredeploy.json)

1. Descargue y descomprima los [ejemplos de IoT en C#](https://docs.microsoft.com/samples/azure-samples/azure-iot-samples-csharp/azure-iot-samples-for-csharp-net/).

1. Abra una ventana de comandos y vaya a la carpeta en la que descomprimió los ejemplos de IoT en C#. Busque la carpeta con el archivo arm-read-write.csproj. En esta ventana de comandos se crean las variables de entorno. Inicie sesión en Azure Portal (https://portal.azure.com ) para obtener las claves. Seleccione **Grupos de recursos** y, a continuación, el grupo de recursos que se usa para esta guía de inicio rápido.

   ![Selección del grupo de recursos](./media/horizontal-arm-route-messages/01-select-resource-group.png)

1. Verá el centro de IoT y la cuenta de almacenamiento que se crearon al implementar la plantilla de ARM. Espere hasta que la plantilla se implemente por completo antes de continuar. A continuación, seleccione el grupo de recursos para ver los recursos.

   ![Ver los recursos del grupo de recursos](./media/horizontal-arm-route-messages/02-view-resources-in-group.png)

1. Necesitará el **nombre del centro**. Seleccione el centro en la lista de recursos. Copie el nombre del centro de la parte superior de la sección IoT Hub en el Portapapeles de Windows. 
 
   ![Copiar el nombre del centro](./media/horizontal-arm-route-messages/03-copy-hub-name.png)

    Sustituya el nombre del centro en este comando, en el lugar que se indica y ejecute este comando en la ventana comandos:
   
    ```cmd
    SET IOT_HUB_URI=<hub name goes here>.azure-devices-net;
    ```

   que será similar a este ejemplo:

   ```cmd
   SET IOT_HUB_URI=ContosoTestHubdlxlud5h.azure-devices-net;
   ```

1. La siguiente variable de entorno es la clave del dispositivo IoT. Agregue un nuevo dispositivo al centro; para ello, seleccione **Dispositivos IoT** en el menú IoT Hub del centro. 

   ![Selección de dispositivos IoT](./media/horizontal-arm-route-messages/04-select-iot-devices.png)

1. En el lado derecho de la pantalla, seleccione **+ Nuevo** para agregar un nuevo dispositivo. 

   Rellene el nombre del nuevo dispositivo. En esta guía de inicio rápido se usa un nombre que empieza por **Contoso-Test-Device**. Guarde el dispositivo y, a continuación, vuelva a abrir la pantalla para recuperar la clave del dispositivo. (La clave se genera automáticamente cuando se cierra el panel). Seleccione la clave principal o la secundaria y cópiela en el Portapapeles de Windows. En la ventana de comandos, establezca el comando que se va a ejecutar y, a continuación, presione **Entrar**. El comando debe ser similar al siguiente, pero con la clave del dispositivo pegada:

   ```cmd
   SET IOT_DEVICE_KEY=<device-key-goes-here>
   ```

1. La última variable de entorno es el **identificador de dispositivo**. En la ventana de comandos, configure el comando y ejecútelo. 
   
   ```cms
   SET IOT_DEVICE_ID=<device-id-goes-here> 
   ```

   será similar a este ejemplo:

   ```cmd
   SET IOT_DEVICE_ID=Contoso-Test-Device
   ```

1. Para ver las variables de entorno que ha definido, escriba SET en la línea de comandos y presione **Entrar**; a continuación, busque las que comienzan por **IoT**.

   ![Ver las variables de entorno](./media/horizontal-arm-route-messages/06-environment-variables.png)

Ahora se han establecido las variables de entorno, ejecute la aplicación desde la misma ventana de comandos. Dado que usa la misma ventana, las variables estarán accesibles en memoria al ejecutar la aplicación.

1. Para ejecutar la aplicación, escriba el siguiente comando en la ventana de comandos y presione **Entrar**.

    `dotnet run arm-read-write`

   La aplicación genera y muestra mensajes en la consola a medida que envía cada mensaje al centro de IoT. El centro se configuró en la plantilla de ARM para que tenga enrutamiento automatizado. Los mensajes que contienen el texto "level = storage" se enrutan automáticamente a la cuenta de almacenamiento. Permita que la aplicación se ejecute entre 10 y 15 minutos y, a continuación, presione **Entrar** una o dos veces hasta que deje de ejecutarse.

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

1. Inicie sesión en [Azure Portal](https://portal.azure.com), seleccione el grupo de recursos y, a continuación, seleccione la cuenta de almacenamiento.

1. Explore en profundidad la cuenta de almacenamiento hasta que encuentre los archivos.

   ![Examinar los archivos de la cuenta de almacenamiento](./media/horizontal-arm-route-messages/07-see-storage.png)

1. Seleccione uno de los archivos, seleccione **Descargar** y descargue el archivo en una ubicación que pueda encontrar más adelante. Tendrá un nombre numérico, como 47. Agregue ".txt" al final y, a continuación, haga doble clic en el archivo para abrirlo.

1. Al abrir el archivo, cada fila corresponde a un mensaje diferente; también se cifra el cuerpo de cada mensaje. Debe estar en orden para que pueda realizar consultas sobre el cuerpo del mensaje.

   ![Ver los mensajes enviados](./media/horizontal-arm-route-messages/08-messages.png)

   > [!NOTE]
   > Estos mensajes se codifican en UTF-32 y Base 64. Si va a leer el mensaje, tendrá que descodificarlo de Base 64 y UTF-32 para poder leerlo como ASCII. Si le interesa, puede usar el método ReadOneRowFromFile del tutorial de enrutamiento para leer estos archivos de mensajes de uno en uno y descodificarlos en ASCII. ReadOneRowFromFile se encuentra en el repositorio de ejemplos de IOT en C# que descomprimió para esta guía de inicio rápido. Esta es la ruta de acceso de la parte superior de la carpeta: *./iot-hub/Tutorials/Routing/SimulatedDevice/Program.cs*. Establezca el valor booleano `readTheFile` en True, incluya en el código la ruta de acceso al archivo en el disco y se abrirá y traducirá la primera fila del archivo.

Ha implementado una plantilla de ARM para crear un centro de IoT y una cuenta de almacenamiento y ha ejecutado un programa para enviar mensajes al centro. Los mensajes se almacenan automáticamente en la cuenta de almacenamiento, donde se pueden ver.

## <a name="clean-up-resources"></a>Limpieza de recursos

Para eliminar los recursos agregados durante esta guía de inicio rápido, inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Grupos de recursos** y, a continuación, busque el grupo de recursos que ha utilizado para esta guía de inicio rápido. Seleccione el grupo de recursos y, a continuación, seleccione *Eliminar*. Se eliminarán todos los recursos del grupo.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Creación e implementación de la primera plantilla de ARM](/azure/azure-resource-manager/templates/template-tutorial-create-first-template)

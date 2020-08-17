---
title: Instalación y uso Azure IoT Explorer | Microsoft Docs
description: Instale la herramienta Azure IoT Explorer y úsela para interactuar con los dispositivos IoT Plug and Play (versión preliminar) conectados a IoT Hub.
author: rido-min
ms.author: rmpablos
ms.date: 05/06/2020
ms.topic: how-to
ms.service: iot-pnp
services: iot-pnp
ms.custom: mvc
ms.openlocfilehash: 083dcde44e56af34f17d952c46e554b234818f27
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87352025"
---
# <a name="install-and-use-azure-iot-explorer"></a>Instalación y uso de Azure IoT Explorer

Azure IoT Explorer es una herramienta gráfica para interactuar con dispositivos IoT Plug and Play (versión preliminar) y probarlos. Después de instalar la herramienta en la máquina local, puede usarla para conectarse a un centro. Puede usar la herramienta para ver la telemetría que envían los dispositivos, trabajar con propiedades de los dispositivos e invocar comandos.

En este artículo aprenderá a:

- Instale y configure la herramienta Azure IoT Explorer.
- Use la herramienta para interactuar con los dispositivos y probarlos.

## <a name="prerequisites"></a>Requisitos previos

Para usar la herramienta Azure IoT Explorer, necesitará lo siguiente:

- Un centro de Azure IoT. Hay muchas maneras de agregar una instancia de IoT Hub a la suscripción de Azure, como la [creación de un centro de IoT mediante la CLI de Azure](../iot-hub/iot-hub-create-using-cli.md). Necesitará la cadena de conexión de IoT Hub para ejecutar la herramienta Azure IoT Explorer. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.
- Un dispositivo registrado en su centro de IoT. Puede usar comando de la CLI de Azure siguiente para registrar un dispositivo. Asegúrese de reemplazar los marcadores de posición `{YourIoTHubName}` y `{YourDeviceID}` por sus valores:

    ```azurecli-interactive
    az iot hub device-identity create --hub-name {YourIoTHubName} --device-id {YourDeviceID}
    ```

## <a name="install-azure-iot-explorer"></a>Instalación de Azure IoT Explorer

Vaya a las [versiones de Azure IoT Explorer](https://github.com/Azure/azure-iot-explorer/releases) y expanda la lista de recursos de la versión más reciente. Descargue e instale la versión más reciente de la aplicación.

>[!Important]
>A partir de la versión 0.11.0, IoT Explorer solo admite la versión preliminar de mayo de 2020 de IoT Plug and Play. Para usar las características disponibles en la versión preliminar anterior de agosto de 2019, debe instalar una versión 0.10.x.

## <a name="use-azure-iot-explorer"></a>Uso de Azure IoT Explorer

Para un dispositivo, puede conectar su propio dispositivo o usar uno de nuestros dispositivos simulados de ejemplo. Siga [estas instrucciones](https://github.com/Azure/azure-iot-sdk-c/tree/public-preview/iothub_client/samples) para ejecutar el ejemplo de dispositivo simulado.

### <a name="connect-to-your-hub"></a>Conexión con el centro

La primera vez que ejecute Azure IoT Explorer, se le solicitará la cadena de conexión de su centro de IoT. Después de agregar la cadena de conexión, seleccione **Connect** (Conectar). Puede usar la configuración de la herramienta para cambiar a otro centro de IoT mediante la actualización de la cadena de conexión.

La definición del modelo de un dispositivo IoT Plug and Play se almacena en el repositorio público, en el repositorio conectado o en una carpeta local. De forma predeterminada, la herramienta busca la definición del modelo en el repositorio público y en el dispositivo conectado. Puede agregar y quitar orígenes, o configurar la prioridad de los orígenes en **Settings** (Configuración):

Para agregar un origen:

1. Vaya a **Home/IoT Plug and Play Settings** (Inicio/IoT Plug and Play).
2. Seleccione **Add** (Agregar) y elija el origen: repositorio o carpeta local.

Para quitar un origen:

1. Vaya a **Home/IoT Plug and Play Settings** (Inicio/IoT Plug and Play).
2. Busque el origen que desea quitar.
3. Seleccione **X** para quitarlo.

Cambie las prioridades de origen:

Puede arrastrar y colocar uno de los orígenes de la definición de modelo en otra clasificación de la lista. 

### <a name="view-devices"></a>Vista de dispositivos

Cuando la herramienta se conecta al centro de IoT, muestra la página de lista **Dispositivos** que contiene todas las identidades de dispositivo registradas en ese centro de IoT. Puede seleccionar cualquier entrada de la lista para ver más información.

En la página de lista **Dispositivos** puede:

- Seleccione **New** (Nuevo) para registrar un nuevo dispositivo en el centro. A continuación, especifique un identificador de dispositivo. Use la configuración predeterminada para la generación automática de claves de autenticación y permita la conexión con el centro.
- Seleccionar un dispositivo y, a continuación, seleccionar **Eliminar** para eliminar una identidad de dispositivo. Revise los detalles del dispositivo antes de completar esta acción para asegurarse de que está eliminando la identidad de dispositivo correcta.

## <a name="interact-with-a-device"></a>Interactuación con un dispositivo

En la página de lista **Dispositivos**, seleccione un valor en la columna **ID de dispositivo** para ver la página de detalles del dispositivo registrado. Para cada dispositivo, hay dos secciones: **Dispositivo** y **gemelo digital**.

### <a name="device"></a>Dispositivo

En esta sección se incluyen las pestañas **Device Identity** (Identidad de dispositivo), **Device Twin** (Dispositivo gemelo), **Telemetry** (Telemetría), **Direct method** (Método directo), **Cloud-to-device message** (Mensaje de nube a dispositivo) y **Module Identity** (Identidad de módulo).

- Puede ver y actualizar la información de la [identidad del dispositivo](../iot-hub/iot-hub-devguide-identity-registry.md) en la pestaña **Id. de dispositivo**.
- Puede acceder a la información del [dispositivo gemelo](../iot-hub/iot-hub-devguide-device-twins.md) en la pestaña **Dispositivo gemelo**.
- Si un dispositivo está conectado y envía datos de forma activa, puede ver la [telemetría](../iot-hub/iot-hub-devguide-messages-read-builtin.md) en la pestaña **Telemetría**.
- Puede llamar a un [método directo](../iot-hub/iot-hub-devguide-direct-methods.md) en el dispositivo en la pestaña **Método directo**.
- Puede enviar un [mensaje de nube a dispositivo](../iot-hub/iot-hub-devguide-messages-c2d.md) en la pestaña **Cloud-to-device messages** (Mensajes de nube a dispositivo).
- Puede acceder a la información del [módulo gemelo](../iot-hub/iot-hub-devguide-module-twins.md).

### <a name="iot-plug-and-play-components"></a>Componentes de IoT Plug and Play

Si el dispositivo está conectado al centro mediante un **identificador de modelo**, la herramienta muestra la pestaña **IoT Plug and Play components** (Componentes de IoT Plug and Play) donde puede ver este identificador.

Si el **identificador de modelo** está disponible en uno de los orígenes configurados: repositorio público o carpeta local, se muestra la lista de componentes. Al seleccionar un componente, se muestran las propiedades, los comandos y la telemetría disponibles.

En la página **Component** (Componente), puede ver las propiedades de solo lectura, actualizar las propiedades editables, invocar comandos y ver los mensajes de telemetría generados por este componente.

:::image type="content" source="media/howto-use-iot-explorer/components-iot-explorer.png" alt-text="Visualización de los componentes de Azure IoT Explorer":::

#### <a name="properties"></a>Propiedades

:::image type="content" source="media/howto-use-iot-explorer/properties-iot-explorer.png" alt-text="Visualización de propiedades de Azure IoT Explorer":::

Puede ver las propiedades de solo lectura definidas en una interfaz en la pestaña **Properties (read-only)** (Propiedades [solo lectura]). Puede actualizar las propiedades editables definidas en una interfaz en la pestaña **Properties (writable)** (Propiedades [editables]):

1. Vaya a la pestaña **Properties (writable)** (Propiedades [editables]).
1. Haga clic en la propiedad que desea actualizar.
1. Introduzca el nuevo valor de la propiedad.
1. Obtenga una vista previa de la carga que se va a enviar al dispositivo.
1. Envíe el cambio.

Después de enviar un cambio, puede realizar un seguimiento del estado de actualización: **sincronizando**, **correcto** o **error**. Una vez completada la sincronización, verá el nuevo valor de la propiedad en la columna **Reported Property** (Propiedad notificada). Si se desplaza a otras páginas antes de que se complete la sincronización, la herramienta le notificará cuando se complete la actualización. También puede usar el centro de notificaciones de la herramienta para ver el historial de notificaciones.

#### <a name="commands"></a>Comandos:

Para enviar un comando a un dispositivo, vaya a la pestaña **Commands** (Comandos):

1. En la lista de comandos, expanda el comando que desea desencadenar.
1. Escriba los valores requeridos para el comando.
1. Obtenga una vista previa de la carga que se va a enviar al dispositivo.
1. Envíe el comando.

#### <a name="telemetry"></a>Telemetría

Para ver la telemetría de la interfaz seleccionada, vaya a su pestaña **Telemetry** (Telemetría).

## <a name="next-steps"></a>Pasos siguientes

En este artículo de procedimientos, ha aprendido a instalar y usar Azure IoT Explorer para interactuar con los dispositivos IoT Plug and Play. El siguiente paso sugerido es aprender a usar los [comandos de IoT Plug and Play de la CLI de Azure](./howto-use-iot-pnp-cli.md).

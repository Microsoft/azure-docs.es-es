---
title: archivo de inclusión
description: archivo de inclusión
services: iot-central
author: dominicbetts
ms.service: iot-central
ms.topic: include
ms.date: 07/07/2020
ms.author: dobett
ms.custom: include file
ms.openlocfilehash: b32465091f82fec0aeae288ee9bfd5540bfe8b9d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87002053"
---
## <a name="create-a-device-template"></a>Creación de una plantilla de dispositivo

Cree una carpeta denominada `environmental-sensor` en la máquina local.

Descargue el archivo JSON del [modelo de funcionalidad del sensor ambiental](https://raw.githubusercontent.com/Azure/IoTPlugandPlay/9004219bff1e958b7cd6ff2a52209f4b7ae19396/samples/EnvironmentalSensorInline.capabilitymodel.json) y guárdelo en la carpeta `environmental-sensor`.

Use un editor de texto para reemplazar las dos instancias de `{YOUR_COMPANY_NAME_HERE}` por el nombre de la empresa en el archivo `EnvironmentalSensorInline.capabilitymodel.json` que ha descargado. Use solo los caracteres a-z, A-Z, 0-9 y el carácter de subrayado.

En la aplicación de Azure IoT Central, cree una plantilla de dispositivo llamada *Environmental sensor* (Sensor ambiental); para ello, importe el archivo del modelo de funcionalidad del dispositivo `EnvironmentalSensorInline.capabilitymodel.json`:

![Plantilla de dispositivo con el modelo de funcionalidad del dispositivo importado](./media/iot-central-add-environmental-sensor/device-template.png)

El modelo de funcionalidad del dispositivo incluye dos interfaces: la **Device Information** (Información de dispositivo) estándar y la **Environmenal Sensor** (Sensor ambiental) personalizada. La interfaz **Environmental Sensor** (Sensor ambiental) define las siguientes funcionalidades:

| Tipo | Display Name (Nombre para mostrar) | Descripción |
| ---- | ------------ | ----------- |
| Propiedad | Device State (Estado del dispositivo)     | Estado del dispositivo. Hay dos estados disponibles: en línea/sin conexión. |
| Propiedad (grabable) | Nombre del cliente    | Nombre del cliente que opera actualmente el dispositivo. |
| Propiedad (grabable) | Nivel de brillo | Nivel de brillo de la iluminación en el dispositivo. Se puede especificar como 1 (alto), 2 (intermedio), 3 (bajo). |
| Telemetría | Temperatura | Temperatura actual que detecta el dispositivo. |
| Telemetría | Humedad    | Humedad actual que detecta el dispositivo. |
| Get-Help | blink          | Comienza a parpadear el LED en el dispositivo durante un intervalo de tiempo determinado. |
| Get-Help | turnon         | Enciende el indicador LED en el dispositivo. |
| Get-Help | turnoff        | Apaga el indicador LED en el dispositivo. |
| Get-Help | rundiagnostics | Este comando asincrónico inicia una ejecución de diagnóstico en el dispositivo. |

Para personalizar la forma en que se muestra la propiedad **Device State** (Estado del dispositivo) en la aplicación de IoT Central, seleccione **Customize** (Personalizar) en la plantilla de dispositivo. Expanda la entrada **Device State** (Estado del dispositivo), escriba _Online_ (En línea) para **True name** (Nombre real) y _Offline_ (Sin conexión) para **False name** (Nombre falso). Guarde los cambios:

![Personalización de la plantilla de dispositivo](./media/iot-central-add-environmental-sensor/customize-template.png)

## <a name="create-views"></a>Creación de vistas

Las vistas permiten interactuar con los dispositivos conectados a la aplicación de IoT Central. Por ejemplo, puede tener vistas que muestren telemetría, vistas que muestren propiedades y vistas que permitan editar las propiedades de escritura y de la nube. Las vistas forman parte de una plantilla de dispositivo.

Para agregar vistas predeterminadas a la plantilla de dispositivo **Environmental sensor** (Sensor ambiental), vaya a la plantilla de dispositivo, seleccione **Views** (Vistas) y seleccione el icono **Generate Default views** (Generar vistas predeterminadas). Asegúrese de que **Información general** y **Acerca de** se encuentran en estado **Activo** y seleccione **Generar vistas de panel predeterminadas**. Ahora tiene dos vistas predeterminadas definidas en la plantilla.

La interfaz **Environmental Sensor** (Sensor ambiental) incluye dos propiedades con posibilidad de escritura: **Customer Name** (Nombre de cliente) y **Brightness Level** (Nivel de brillo). Para crear una vista, puede editar estas propiedades:

1. Seleccione **Views** (Vistas) y el icono **Editing device and cloud data** (Edición de los datos del dispositivo y de la nube).

1. Escriba _Properties_ (Propiedades) como nombre del formulario.

1. Seleccione las propiedades **Brightness Level** (Nivel de brillo) y **Customer Name** (Nombre de cliente). Después, seleccione **Add section** (Agregar sección).

1. Guarde los cambios.

![Incorporación de una vista para habilitar la edición de propiedades](./media/iot-central-add-environmental-sensor/properties-view.png)

## <a name="publish-the-template"></a>Publicación de la plantilla

Antes de poder agregar un dispositivo que use la plantilla de dispositivo **Environmental Sensor**, debe publicar esta.

En la plantilla de dispositivo, seleccione **Publish** (Publicar). En el panel **Publicar esta plantilla de dispositivo en la aplicación**, seleccione **Publicar**.

Para comprobar que la plantilla está lista para usar, vaya a la página **Devices** (Dispositivos) en la aplicación de IoT Central. En la sección **Devices** (Dispositivos) se muestra una lista de los dispositivos publicados en la aplicación:

![Plantillas publicadas en la página de dispositivos](./media/iot-central-add-environmental-sensor/published-templates.png)

## <a name="add-a-real-device"></a>Adición de un dispositivo real

En la aplicación de Azure IoT Central, agregue un dispositivo real a la plantilla de dispositivo que creó en la sección anterior:

1. En la página **Devices** (Dispositivos), seleccione la plantilla de dispositivo **Environmental Sensor** (Sensor ambiental).

1. Seleccione **+ Nuevo**.

1. En el cuadro de diálogo **Create a new device** (Crear un nuevo dispositivo), asegúrese de que el tipo de plantilla es **Environmental Sensor** (Sensor ambiental) y de que la opción **Simulate this device?** (¿Simular este dispositivo?) está establecida en **No**.

1. Seleccione **Crear**.

Haga clic en el nombre de dispositivo y seleccione **Connect** (Conectar). Anote la información de conexión del dispositivo en la página **Device Connection** (Conexión de dispositivos): **ID scope** (Ámbito de Id.), **Device ID** (Id. de dispositivo) y **Primary key** (Clave principal). Necesitará estos valores al crear el código del dispositivo:

![Información de conexión del dispositivo](./media/iot-central-add-environmental-sensor/device-connection.png)

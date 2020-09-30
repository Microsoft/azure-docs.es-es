---
title: Descripción del control de versiones de las plantillas de dispositivo para las aplicaciones de Azure IoT Central | Microsoft Docs
description: Iteración sobre las plantillas de dispositivo mediante la creación de nuevas versiones y sin afectar a los dispositivos conectados en vivo
author: sarahhubbard
ms.author: sahubbar
ms.date: 04/24/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
manager: peterpr
ms.custom: device-developer
ms.openlocfilehash: 2025b127a428afa478cfe839c7619df2d7d688d3
ms.sourcegitcommit: 43558caf1f3917f0c535ae0bf7ce7fe4723391f9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90015966"
---
# <a name="create-a-new-device-template-version"></a>Creación de una nueva versión de plantilla de dispositivo

*Este artículo se aplica a generadores de soluciones y desarrolladores de dispositivos.*

Una plantilla de dispositivo incluye un esquema que describe el modo en que un dispositivo interactúa con IoT Central. Estas interacciones incluyen telemetría, propiedades y comandos. Tanto el dispositivo como la aplicación de IoT Central se basan en una comprensión compartida de este esquema para intercambiar información. Solo puede realizar cambios limitados en el esquema sin romper el contrato. Por eso, la mayoría de los cambios de esquema requieren una nueva versión de la plantilla de dispositivo. El control de versiones de la plantilla de dispositivo permite que los dispositivos antiguos continúen con la versión de esquema que comprenden, mientras que los más recientes o actualizados usan una versión posterior.

El esquema de una plantilla de dispositivo se define en el modelo de funcionalidad del dispositivo (DCM) y sus interfaces. Las plantillas de dispositivo incluyen otra información, como las propiedades de la nube, las personalizaciones de presentación y las vistas. Si realiza cambios en aquellas partes de la plantilla de dispositivo que no definen el modo en que el dispositivo intercambia datos con IoT Central, no es necesario que cree una versión de la plantilla.

Debe publicar cualquier cambio de plantilla de dispositivo, independientemente de si requiere o no una actualización de versión, para que un operador pueda usarla. IoT Central impide la publicación de cambios importantes en una plantilla de dispositivo sin cambiar primero la versión de la plantilla.

> [!NOTE]
> Para más información sobre cómo crear una plantilla de dispositivo, consulte [Configuración de una plantilla de dispositivo](howto-set-up-template.md).

## <a name="versioning-rules"></a>Reglas de control de versiones

En esta sección se resumen las reglas de control de versiones que se aplican a las plantillas de dispositivo. Tanto las interfaces como las DCM tienen números de versión. En el fragmento de código siguiente se muestra la DCM para un dispositivo de sensor de entorno. La DCM tiene dos interfaces: **DeviceInformation** y **EnvironmentalSensor**. Puede ver los números de versión al final de los campos `@id`. Para ver esta información en la interfaz de usuario de IoT Central, seleccione **Ver identidad** en el editor de plantillas de dispositivos.

```json
{
  "@id": "urn:contoso:sample_device:1",
  "@type": "CapabilityModel",
  "implements": [
    {
      "@id": "urn:contoso:sample_device:deviceinfo:1",
      "@type": "InterfaceInstance",
      "name": "deviceinfo",
      "schema": {
        "@id": "urn:azureiot:DeviceManagement:DeviceInformation:1",
        "@type": "Interface",
        "displayName": {
          "en": "Device Information"
        },
        "contents": [...
        ]
      }
    },
    {
      "@id": "urn:contoso:sample_device:sensor:1",
      "@type": "InterfaceInstance",
      "name": "sensor",
      "schema": {
        "@id": "urn:contoso:EnvironmentalSensor:2",
        "@type": "Interface",
        "displayName": {
          "en": "Environmental Sensor"
        },
        "contents": [...
        ]
      }
    }
  ],
  "displayName": {
    "en": "Environment Sensor Capability Model"
  },
  "@context": [
    "http://azureiot.com/v1/contexts/IoTModel.json"
  ]
}
```

* Después de publicar una DCM, no puede quitar ninguna interfaz, ni siquiera en una nueva versión de la plantilla de dispositivo.
* Después de publicar una DCM, puede agregar una interfaz si crea una nueva versión de la plantilla de dispositivo.
* Después de publicar una DCM, puede reemplazar una interfaz con una versión más reciente si crea una versión nueva de la plantilla de dispositivo. Por ejemplo, si la plantilla de dispositivo de sensor v1 usa la interfaz EnvironmentalSensor v1, puede crear una plantilla de dispositivo de sensor v2 que use la interfaz EnvironmentalSensor v2.
* Una vez publicada una interfaz, no puede quitar ningún contenido de ella, ni siquiera en una versión nueva de la plantilla del dispositivo.
* Una vez publicada una interfaz, puede agregar elementos a su contenido si crea una versión nueva de la plantilla de dispositivo e interfaz. Entre los elementos que se pueden agregar a la interfaz se incluyen telemetría, propiedades y comandos.
* Una vez publicada una interfaz, puede realizar cambios que no sean de esquema en los elementos existentes en la interfaz si crea una versión nueva de la plantilla de dispositivo e interfaz. Entre las partes que no son de esquema de un elemento de interfaz se incluyen el nombre para mostrar y el tipo semántico. Las partes del esquema de un elemento de interfaz que no se pueden cambiar son el nombre, el tipo de funcionalidad y el esquema.

En las secciones siguientes se explican algunos ejemplos de cómo modificar plantillas de dispositivo en IoT Central.

## <a name="customize-the-device-template-without-versioning"></a>Personalización de la plantilla de dispositivo sin control de versiones

Algunos elementos de las funcionalidades del dispositivo se pueden editar sin necesidad de crear una versión de la plantilla de dispositivo ni de las interfaces. Por ejemplo, algunos de estos campos incluyen el nombre para mostrar, el tipo semántico, el valor mínimo, el valor máximo, las posiciones decimales, el color, la unidad, la unidad de visualización, el comentario y la descripción. Para agregar una de estas personalizaciones:

1. Vaya a la página **Plantillas de dispositivo**.
1. Seleccione la plantilla de dispositivo que desea personalizar.
1. Elija la pestaña **Personalizar**.
1. Todas las funcionalidades definidas en el modelo de funcionalidad del dispositivo se mostrarán aquí. Puede editar, guardar y usar todos estos campos sin necesidad de tener una versión de la plantilla de dispositivo. Si hay campos que quiere editar y son de solo lectura, tendrá que hacer una versión de la plantilla de dispositivo para cambiarlos. Seleccione el campo que desea editar y escriba los nuevos valores.
1. Haga clic en **Save**(Guardar). Ahora, estos valores reemplazarán todo lo que se guardó inicialmente en la plantilla de dispositivo y se usarán en la aplicación.

## <a name="version-a-device-template"></a>Versión de una plantilla de dispositivo

Al crear una versión nueva de la plantilla de dispositivo, se crea una versión de borrador de la plantilla en la que se puede editar el modelo de funcionalidad del dispositivo. Todas las interfaces publicadas permanecerán publicadas hasta que tengan versiones individuales. Para modificar una interfaz publicada, primero debe crear una nueva plantilla de dispositivo.

Solo debe controlar la versión de la plantilla de dispositivo cuando esté intentando editar una parte del modelo de funcionalidad del dispositivo que no se pueda editar en la sección de personalizaciones.

Para controlar la versión de una plantilla de dispositivo:

1. Vaya a la página **Plantillas de dispositivo**.
1. Seleccione la plantilla de dispositivo cuya versión intenta controlar.
1. Haga clic en el botón **Versión** en la parte superior de la página y asigne un nuevo nombre a la plantilla. IoT Central sugiere un nombre nuevo, que se puede editar.
1. Haga clic en **Crear**.
1. Ahora la plantilla de dispositivo está en modo borrador. Puede ver que las interfaces todavía están bloqueadas. Controle las versiones de cualquier interfaz que desee modificar.

## <a name="version-an-interface"></a>Control de las versiones de una interfaz

El control de versiones de una interfaz permite agregar, actualizar y quitar las funcionalidades dentro de la interfaz que ya se ha creado.

Para controlar las versiones de una interfaz:

1. Vaya a la página **Plantillas de dispositivo**.
1. Seleccione la plantilla de dispositivo que tiene en modo borrador.
1. Seleccione la interfaz que está en modo publicado y que desea modificar y controlar la versión.
1. Haga clic en el botón **Versión** en la parte superior de la página de la interfaz.
1. Haga clic en **Crear**.
1. Ahora la interfaz está en modo borrador. Podrá agregar funcionalidades a la interfaz, o modificarlas, sin interrumpir las personalizaciones y vistas existentes.

## <a name="migrate-a-device-across-versions"></a>Migración de un dispositivo entre versiones

Puede crear varias versiones de la plantilla de dispositivo. Con el tiempo, tendrá varios dispositivos conectados mediante estas plantillas de dispositivo. Puede migrar los dispositivos de una versión de la plantilla de dispositivo a otra. Los pasos siguientes describen cómo migrar un dispositivo:

1. Vaya a la página del **Explorador de dispositivos**.
1. Seleccione el dispositivo que necesita migrar a otra versión.
1. Elija **Migrate** (Migrar).
1. Seleccione la plantilla de dispositivo con el número de versión a la que desea migrar el dispositivo y elija **Migrar**.

![Migración de un dispositivo](media/howto-version-device-template/pick-version.png)

## <a name="next-steps"></a>Pasos siguientes

Si es operador o generador de soluciones, el siguiente paso sugerido es aprender [cómo administrar los dispositivos](./howto-manage-devices.md).

Si es desarrollador de dispositivos, el siguiente paso sugerido es leer acerca de los [dispositivos de Azure IoT Edge y Azure IoT Central](./concepts-iot-edge.md).

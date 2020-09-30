---
title: Azure IoT Edge y Azure IoT Central | Microsoft Docs
description: Aprenda a usar Azure IoT Edge con una aplicación de IoT Central.
author: dominicbetts
ms.author: dobett
ms.date: 12/12/2019
ms.topic: conceptual
ms.service: iot-central
services: iot-central
ms.custom:
- device-developer
- iot-edge
ms.openlocfilehash: 5c51649e742f6e314e0e324ae19c38f1a83b02ee
ms.sourcegitcommit: 43558caf1f3917f0c535ae0bf7ce7fe4723391f9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90016867"
---
# <a name="connect-azure-iot-edge-devices-to-an-azure-iot-central-application"></a>Conexión de dispositivos de Azure IoT Edge a una aplicación de Azure IoT Central

*Este artículo se aplica a generadores de soluciones y desarrolladores de dispositivos.*

IoT Edge está formado por tres componentes:

* Los **módulos de IoT Edge** son contenedores que ejecutan servicios de Azure, de asociados o código propio del usuario. Los módulos se implementan en los dispositivos de IoT Edge y se ejecutan de forma local en ellos.
* El **entorno en tiempo de ejecución de IoT Edge** se ejecuta en todos los dispositivos de IoT Edge y administra los módulos que se implementan en cada dispositivo.
* Una **interfaz basada en la nube** permite supervisar y administrar los dispositivos de IoT Edge de forma remota. IoT Central es la interfaz de la nube.

Un dispositivo **Azure IoT Edge** puede ser un dispositivo de puerta de enlace con dispositivos de nivel inferior que se conectan al dispositivo de IoT Edge. En este artículo se comparte más información sobre los patrones de conectividad de los dispositivos de nivel inferior.

Una **plantilla de dispositivo** define las funcionalidades del dispositivo y de los módulos de IoT Edge. Entre estas se incluyen la telemetría que envía el módulo, las propiedades del módulo y los comandos a los que un módulo responde.

## <a name="downstream-device-relationships-with-a-gateway-and-modules"></a>Relaciones de dispositivo de nivel inferior con una puerta de enlace y módulos

Los dispositivos de nivel inferior pueden conectarse a un dispositivo de puerta de enlace IoT Edge mediante el módulo `$edgeHub`. Este dispositivo de IoT Edge se convierte en una puerta de enlace transparente en este escenario.

![Diagrama de una puerta de enlace transparente](./media/concepts-iot-edge/gateway-transparent.png)

Los dispositivos de nivel inferior también pueden conectarse a un dispositivo de puerta de enlace IoT Edge mediante un módulo personalizado. En el escenario siguiente, los dispositivos de nivel inferior se conectan mediante un módulo personalizado de Modbus.

![Diagrama de conexión de módulo personalizado](./media/concepts-iot-edge/gateway-module.png)

En el diagrama siguiente se muestra la conexión a un dispositivo de puerta de enlace IoT Edge mediante ambos tipos de módulos (personalizado y `$edgeHub`).  

![Diagrama de conexión mediante ambos módulos de conexión](./media/concepts-iot-edge/gateway-module-transparent.png)

Finalmente, los dispositivos de nivel inferior se pueden conectar a un dispositivo de puerta de enlace IoT Edge mediante varios módulos personalizados. En el diagrama siguiente se muestran los dispositivos de nivel inferior que se conectan mediante un módulo personalizado de Modbus, un módulo personalizado de BLE y el módulo `$edgeHub`. 

![Diagrama de conexión mediante varios módulos personalizados](./media/concepts-iot-edge/gateway-module2-transparent.png)

## <a name="deployment-manifests-and-device-templates"></a>Manifiestos de implementación y plantillas de dispositivo

En IoT Edge, puede implementar y administrar la lógica de negocios en forma de módulos. Los módulos de IoT Edge son la unidad más pequeña de cálculo que administra IoT Edge y pueden contener servicios de Azure (por ejemplo, Azure Stream Analytics) o su propio código específico de la solución. Para entender cómo se desarrollan, implementan y mantienen los módulos, consulte [Módulos de IoT Edge](../../iot-edge/iot-edge-modules.md).

A nivel general, un manifiesto de implementación es una lista de módulos gemelos que se configuran con sus propiedades deseadas. Un manifiesto de implementación indica a un dispositivo IoT Edge (o a un grupo de dispositivos) qué módulos debe instalar y cómo configurarlos. Los manifiestos de implementación incluyen las propiedades deseadas de cada módulo gemelo. Los dispositivos IoT Edge informan sobre las propiedades notificadas de cada módulo.

Use Visual Studio Code para crear un manifiesto de implementación. Para más información, consulte [Azure IoT Edge para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-edge).

En Azure IoT Central, puede importar un manifiesto de implementación para crear una plantilla de dispositivo. El siguiente diagrama de flujo muestra el ciclo de vida de un manifiesto de implementación en IoT Central.

![Diagrama de flujo del ciclo de vida de un manifiesto de implementación](./media/concepts-iot-edge/dmflow.png)

IoT Central modela un dispositivo IoT Edge de la siguiente manera:

* Cada plantilla de dispositivo de IoT Edge tiene un modelo de funcionalidad del dispositivo.
* Para cada módulo personalizado incluido en el manifiesto de implementación, se genera un modelo de funcionalidad del módulo.
* Se establece una relación entre cada modelo de funcionalidad del módulo y un modelo de funcionalidad del dispositivo.
* Un modelo de funcionalidad del módulo implementa interfaces de módulo.
* Cada interfaz del módulo contiene datos de telemetría, propiedades y comandos.

![Diagrama de modelado de IoT Edge](./media/concepts-iot-edge/edgemodelling.png)

## <a name="iot-edge-gateway-devices"></a>Dispositivos de puerta de enlace de IoT Edge

Si seleccionó un dispositivo de IoT Edge como dispositivo de puerta de enlace, puede agregar relaciones de nivel inferior a los modelos de funcionalidad de los dispositivos que desea conectar a él.

## <a name="next-steps"></a>Pasos siguientes

Si es desarrollador de dispositivos, el siguiente paso sugerido es conocer los [tipos de dispositivo de puerta de enlace de IoT Central](./tutorial-define-gateway-device-type.md).

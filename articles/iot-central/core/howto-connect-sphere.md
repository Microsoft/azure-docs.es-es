---
title: Conexión de un dispositivo de Azure Sphere en Azure IoT Central | Microsoft Docs
description: Aprenda a conectar un dispositivo de Azure Sphere (DevKit) a una aplicación de Azure IoT Central.
services: iot-central
ms.service: iot-central
ms.topic: how-to
ms.author: sandeepu
author: sandeeppujar
ms.date: 04/30/2020
ms.custom: device-developer
ms.openlocfilehash: 34b767a2cc48c94fdb5c2db032321b9254bf4ce2
ms.sourcegitcommit: 43558caf1f3917f0c535ae0bf7ce7fe4723391f9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90017700"
---
# <a name="connect-an-azure-sphere-device-to-your-azure-iot-central-application"></a>Conexión de un dispositivo de Azure Sphere a la aplicación de Azure IoT Central

*Este artículo se aplica a los desarrolladores de dispositivos.*

En este artículo se muestra cómo conectar un dispositivo de Azure Sphere (DevKit) a una aplicación de Azure IoT Central.

Azure Sphere es una plataforma de aplicaciones segura y de alto nivel con características integradas de comunicación y seguridad para dispositivos conectados a Internet. Incluye una unidad de microcontroladores (MCU) cruzados, conectados y seguros, un sistema operativo basado en Linux de alto nivel y personalizado y un servicio de seguridad basado en la nube que ofrece una seguridad renovable y continua. Para más información, vea [¿Qué es Azure Sphere?](https://docs.microsoft.com/azure-sphere/product-overview/what-is-azure-sphere)

Los [kits de desarrollo de Azure Sphere](https://azure.microsoft.com/services/azure-sphere/get-started/) proporcionan todo lo que necesita para empezar a crear prototipos y desarrollar aplicaciones de Azure Sphere. Azure IoT Central con Azure Sphere habilita una pila de un extremo a otro para una solución de IoT. Azure Sphere proporciona compatibilidad con dispositivos e IoT Central como plataforma de aplicaciones de IoT administradas de código cero.

En este artículo paso a paso, puede:

- Cree un dispositivo de Azure Sphere en IoT Central mediante la plantilla de dispositivo de Azure Sphere DevKit de la biblioteca.
- Prepare el dispositivo de Azure Sphere DevKit para Azure IoT.
- Conecte Azure Sphere DevKit a Azure IoT Central.
- Consulte la telemetría del dispositivo en IoT Central.

## <a name="prerequisites"></a>Prerrequisitos

Para completar los pasos de este artículo, necesitará los siguientes recursos:

- Una aplicación de Azure IoT Central.
- Visual Studio 2019, versión 16.4 o posterior.
- Un [kit de desarrollo MT3620 de Azure Sphere de Seeed Studios](https://docs.microsoft.com/azure-sphere/hardware/mt3620-reference-board-design).

> [!NOTE]
> Si no tiene un dispositivo físico, después del primer paso, vaya a la última sección para probar un dispositivo simulado.

## <a name="create-the-device-in-iot-central"></a>Creación del dispositivo en IoT Central

Para crear un dispositivo de Azure Sphere en IoT Central:

1. Desde la aplicación de Azure IoT Central, seleccione la pestaña **Plantillas de dispositivo** y, después, **+ Nueva**. En la sección **Uso de una plantilla de dispositivo destacada**, seleccione **Dispositivo de ejemplo de Azure Sphere**.

    :::image type="content" source="media/howto-connect-sphere/sphere-create-template.png" alt-text="Plantilla de dispositivo para Azure Sphere DevKit":::

1. En la plantilla de dispositivo, edite la vista denominada **Información general** para mostrar la **temperatura** y el **botón Presionar**.

1. Seleccione el tipo de vista **Edición del dispositivo y los datos de la nube** para agregar otra vista que muestre la propiedad de lectura o escritura **LED de estado**. Arrastre la propiedad **LED de estado** al rectángulo con puntos vacío situado en el lado derecho del formulario. Seleccione **Guardar**.

## <a name="prepare-the-device"></a>Preparar el dispositivo

Antes de conectar el dispositivo de Azure Sphere DevKit a IoT Central, debe [configurar el dispositivo y el entorno de desarrollo](https://github.com/Azure/azure-sphere-samples/tree/master/Samples/AzureIoT).

## <a name="connect-the-device"></a>Conexión del dispositivo

Para habilitar el ejemplo para que se conecte a IoT Central, debe [configurar una aplicación de Azure IoT Central y, a continuación, modificar el manifiesto de aplicación del ejemplo](https://aka.ms/iotcentral-sphere-git-readme).

## <a name="view-the-telemetry-from-the-device"></a>Visualización de la telemetría desde el dispositivo

Cuando el dispositivo está conectado a IoT Central, puede ver la telemetría en el panel.

:::image type="content" source="media/howto-connect-sphere/sphere-view.png" alt-text="Panel de Azure Sphere DevKit":::

## <a name="create-a-simulated-device"></a>Cree un dispositivo simulado:

Si no tiene un dispositivo físico de Azure Sphere DevKit, puede crear un dispositivo simulado para probar la aplicación de Azure IoT Central.

Para crear un dispositivo simulado:

- Seleccione **Dispositivos > Azure IoT Sphere**
- Seleccione **+ Nuevo**.
- Especifique un **id. de dispositivo** único y un **nombre de dispositivo** descriptivo.
- Habilite la opción **Simulated** (Simulado).
- Seleccione **Crear**.

## <a name="next-steps"></a>Pasos siguientes

Si es un desarrollador de dispositivos, algunos de los pasos sugeridos son los siguientes:

- Consultar sobre la [Conectividad de dispositivos en Azure IoT Central](./concepts-get-connected.md)
- Obtener información sobre la [Supervisión de la conectividad de dispositivos mediante la CLI de Azure](./howto-monitor-devices-azure-cli.md)

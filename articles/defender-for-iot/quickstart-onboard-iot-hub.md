---
title: 'Inicio rápido: Habilitar el servicio'
description: Aprenda a incorporar y habilitar el servicio Defender para seguridad de IoT en Azure IoT Hub.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/06/2020
ms.author: mlottner
ms.openlocfilehash: 3f84f3121d9982205ecf51ec64cfe332b6a5ad42
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945923"
---
# <a name="quickstart-onboard-azure-defender-for-iot-service-in-iot-hub"></a>Inicio rápido: Incorporación del servicio Azure Defender para IoT en IoT Hub

En este artículo se explica cómo habilitar el servicio Defender para IoT en la instancia existente de IoT Hub. Si actualmente no tiene una instancia de IoT Hub, consulte [Creación de una instancia de IoT Hub mediante Azure Portal](https://docs.microsoft.com/azure/iot-hub/iot-hub-create-through-portal) para comenzar.

> [!NOTE]
> Actualmente, Defender para IoT solo admite el nivel Estándar de las instancias de IoT Hub.

## <a name="prerequisites-for-enabling-the-service"></a>Requisitos previos para habilitar el servicio

- Área de trabajo de Log Analytics
  - De forma predeterminada, Defender para IoT almacena dos tipos de información en el área de trabajo de Log Analytics: **alertas de seguridad** y **recomendaciones**.
  - Puede elegir agregar almacenamiento de un tipo de información adicional, **eventos sin procesar**. Tenga en cuenta que almacenar **eventos sin procesar** en Log Analytics conlleva costos de almacenamiento adicionales.
- IoT Hub (nivel estándar)
- Cumplimiento de todos [requisitos previos de servicio](service-prerequisites.md)

## <a name="enable-defender-for-iot-on-your-iot-hub"></a>Habilitación de Defender para IoT en IoT Hub

Para habilitar la seguridad en IoT Hub:

1. Abra **IoT Hub** en Azure Portal.
1. En el menú **Seguridad**, haga clic en **Secure your IoT solution** (Proteger la solución de IoT).

Felicidades. Ha completado la habilitación de Defender para IoT en IoT Hub.

### <a name="geolocation-and-ip-address-handling"></a>Administración de la ubicación geográfica y la dirección IP

Para proteger la solución de IoT, se recopilan las direcciones IP de las conexiones entrantes y salientes de los dispositivos IoT, IoT Edge e IoT Hub y se almacenan de forma predeterminada. Esta información es fundamental para detectar conexiones anómalas de direcciones IP de origen sospechosas. Por ejemplo, cuando se realizan intentos para establecer conexiones desde una dirección IP de origen de una red de robots (botnet) conocida o desde una dirección IP de origen fuera de su geolocalización. El servicio Defender para IoT ofrece la flexibilidad necesaria para habilitar y deshabilitar la recopilación de datos de direcciones IP en cualquier momento.

Para habilitar o deshabilitar la recopilación de datos de direcciones IP:

1. Abra IoT Hub y seleccione **Configuración** en el menú **Seguridad**.
1. Elija la pantalla **Recopilación de datos** y modifique la geolocalización y las opciones de control de IP como desee.

### <a name="log-analytics-creation"></a>Creación de Log Analytics

Cuando se activa Defender para IoT, se crea un área de trabajo de Azure Log Analytics predeterminada para almacenar eventos de seguridad, alertas y recomendaciones sin procesar de los dispositivos IoT, IoT Edge e IoT Hub. Cada mes, los 5 primeros GB de datos ingeridos por cliente en el servicio Azure Log Analytics se ofrecen de forma gratuita. Cada GB de datos ingeridos en su área de trabajo de Azure Log Analytics se retiene sin cargo adicional los 31 primeros días. Obtenga más información sobre los [precios de Log Analytics](https://azure.microsoft.com/pricing/details/monitor/).

Para cambiar la configuración del área de trabajo de Log Analytics:

1. Abra IoT Hub y seleccione **Configuración** en el menú **Seguridad**.
1. Elija la pantalla **Recopilación de datos** y modifique la configuración del área de trabajo de Log Analytics que desee.

### <a name="customize-your-iot-security-solution"></a>Personalización de la solución de seguridad de IoT

De forma predeterminada, al activar la solución Defender para IoT, se protegen automáticamente todas las instancias de IoT Hub de la suscripción a Azure.

Para activar o desactivar el servicio Defender para IoT en una instancia de IoT Hub específica, haga lo siguiente:

1. Abra IoT Hub y seleccione **Configuración** en el menú **Seguridad**.
1. Elija la pantalla **Recopilación de datos** y modifique la configuración de seguridad de cualquier centro de IoT de la suscripción de Azure como desee.

## <a name="next-steps"></a>Pasos siguientes

Pase al siguiente artículo para configurar una solución...

> [!div class="nextstepaction"]
> [Configuración de una solución](quickstart-configure-your-solution.md)

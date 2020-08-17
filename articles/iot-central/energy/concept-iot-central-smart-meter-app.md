---
title: 'Conceptos de arquitectura en Azure IoT Central: Medidores inteligentes | Microsoft Docs'
description: En este artículo se presentan conceptos clave relacionados con la arquitectura de la plantilla de la aplicación de energía de Azure IoT Central.
author: op-ravi
ms.author: omravi
ms.date: 10/22/2019
ms.topic: overview
ms.service: iot-central
services: iot-central
manager: abjork
ms.openlocfilehash: aac5e35f3d0c42e97069ecb38fedaadcdc7794dc
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87001849"
---
# <a name="azure-iot-central---smart-meter-app-architecture"></a>Azure IoT Central: arquitectura de aplicaciones de medidores inteligentes



En este artículo se proporciona información general sobre la arquitectura de la plantilla de la aplicación de supervisión de medidores inteligentes. En el diagrama siguiente se muestra una arquitectura de uso común de la aplicación de medidores inteligentes en Azure que usa la plataforma de IoT Central.

> [!div class="mx-imgBorder"]
> ![arquitectura de medidores inteligentes](media/concept-iot-central-smart-meter/smart-meter-app-architecture.png)

Esta arquitectura consta de los siguientes componentes. Puede que algunas soluciones no requieran todos los componentes que se mencionan aquí.

## <a name="smart-meters-and-connectivity"></a>Medidores inteligentes y conectividad 

Un medidor inteligente es uno de los dispositivos más importantes entre todos los recursos energéticos. Registra datos de consumo energético y los comunica a servicios públicos con fines de supervisión y otros casos de uso, como la respuesta a la demanda y la facturación. Según el tipo de medidor, puede conectarse a IoT Central mediante puertas de enlace u otros sistemas o dispositivos intermedios, tales como dispositivos perimetrales y sistemas de cabeceras. Cree un dispositivo de puente de IoT Central para conectar los dispositivos que no pueden hacerlo directamente. El puente de dispositivo de IoT Central es una solución de código abierto y puede encontrar los detalles completos [aquí](https://docs.microsoft.com/azure/iot-central/core/howto-build-iotc-device-bridge). 


## <a name="iot-central-platform"></a>Plataforma de IoT Central

Azure IoT Central es una plataforma que simplifica la compilación de su solución de IoT y permite reducir la carga y los costos de la administración, las operaciones y el desarrollo de IoT. Con IoT Central, puede conectar, supervisar y administrar fácilmente sus recursos de Internet de las cosas (IoT) a escala. Después de conectar los medidores inteligentes a IoT Central, la plantilla de aplicación usa características integradas, como modelos de dispositivos, comandos y paneles. La plantilla de la aplicación también usa el almacenamiento de IoT Central para escenarios de ruta de acceso activa, como la supervisión, el análisis, las reglas y la visualización de datos de medición casi en tiempo real. 


## <a name="extensibility-options-to-build-with-iot-central"></a>Opciones de extensibilidad para la compilación con IoT Central
La plataforma de IoT Central proporciona dos opciones de extensibilidad: Exportación de datos continua (CDE) y API. Los clientes y asociados pueden elegir entre estas opciones en función de la personalización de sus soluciones para necesidades específicas. Por ejemplo, uno de nuestros asociados configuró CDE con Azure Data Lake Storage (ADLS). Usa ADLS para la retención de datos a largo plazo y otros escenarios de almacenamiento de la ruta de acceso pasiva, como el procesamiento por lotes, la auditoría e informes. 

## <a name="next-steps"></a>Pasos siguientes

* Ahora que ha aprendido acerca de la arquitectura, [cree una aplicación de medidores inteligentes de forma gratuita](https://apps.azureiotcentral.com/build/new/smart-meter-monitoring).
* Para más información acerca de IoT Central, consulte [Introducción a IoT Central](https://docs.microsoft.com/azure/iot-central/).

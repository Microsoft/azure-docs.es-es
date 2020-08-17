---
title: Diferencias con respecto a la versión anterior
titleSuffix: Azure Digital Twins
description: Explicación de los cambios de la nueva versión de Azure Digital Twins
author: baanders
ms.author: baanders
ms.date: 3/12/2020
ms.topic: overview
ms.service: digital-twins
ms.openlocfilehash: 9f95c3d03fed1950d6151ad1ed910bf2a8cd14f5
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87125265"
---
# <a name="how-is-the-new-azure-digital-twins-different-from-the-previous-version-2018"></a>¿En qué se diferencia la nueva versión de Azure Digital Twins de la anterior (2018)?

La primera versión preliminar pública de Azure Digital Twins se publicó en octubre de 2018. Aunque ahora los conceptos básicos de la versión anterior ya se han pasado al nuevo servicio en la versión preliminar pública, buena parte de las interfaces y los detalles de implementación han cambiado para que el servicio sea más flexible y accesible. Estos cambios están motivados por los comentarios de los clientes.

> [!IMPORTANT]
> En función de las funcionalidades ampliadas del nuevo servicio, el servicio Azure Digital Twins anterior se retirará a finales del año 2020.

Si usaba la primera versión de Azure Digital Twins durante la versión preliminar pública anterior, use la información y los procedimientos recomendados de este artículo para aprender a trabajar con el nuevo servicio y aprovechar sus características.

## <a name="differences-by-topic"></a>Diferencias por tema

El siguiente gráfico proporciona una vista en paralelo de los conceptos que han cambiado entre la versión anterior del servicio y la nueva (actual).

| Tema | Versión anterior | Versión nueva |
| --- | --- | --- | --- |
| **Modelado**<br>*Más flexible* | La versión anterior se diseñó en torno a los espacios inteligentes, por lo que incluía un vocabulario integrado para edificios. | La nueva versión de Azure Digital Twins es independiente del dominio. Puede definir su propio vocabulario personalizado y modelos personalizados para su solución, a fin de representar más tipos de entornos de maneras más flexibles.<br><br>Más información en [*Conceptos: Modelos personalizados*](concepts-models.md). |
| **Topología**<br>*Más flexible*| La versión anterior admitía una estructura de datos de árbol adaptada a espacios inteligentes. Los gemelos digitales se conectaban con relaciones jerárquicas. | Con la nueva versión, los gemelos digitales se pueden conectar en topologías de grafos arbitrarias, organizadas según sea necesario. Esto le proporciona más flexibilidad para expresar las complejas relaciones del mundo real.<br><br>Más información en [*Conceptos: Gemelos digitales y grafo de gemelos*](concepts-twins-graph.md). |
| **Proceso**<br>*Más rico y flexible* | En la versión anterior, la lógica de procesamiento de eventos y telemetría se definía en las funciones definidas por el usuario (UDF) de JavaScript. La depuración con UDF era limitada. | La nueva versión tiene un modelo de proceso abierto: para proporcionar lógica personalizada, debe conectar recursos de proceso externos como [Azure Functions](../azure-functions/functions-overview.md). Esto le permite usar un lenguaje de programación de su elección, acceder a las bibliotecas de código personalizado sin restricciones, y aprovechar los recursos de desarrollo y depuración que el servicio externo pueda tener.<br><br>Más información en [*Procedimientos: Configuración de una función de Azure para procesar datos*](how-to-create-azure-function.md). |
| **Administración de dispositivos con IoT Hub**<br>*Más accesible* | Los dispositivos administrados de la versión anterior con una instancia de [IoT Hub](../iot-hub/about-iot-hub.md) interna del servicio Azure Digital Twins. Este centro de conectividad integrado no era totalmente accesible para los desarrolladores. | En la nueva versión, usted "trae su propio" IoT Hub, para lo cual conecta una instancia de IoT Hub creada de forma independiente (junto con los dispositivos que ya administra). Esto le proporciona acceso completo a las funcionalidades de IoT Hub y le permite controlar la administración de dispositivos.<br><br>Más información en [*Procedimientos: Ingesta de telemetría desde IoT Hub*](how-to-ingest-iot-hub-data.md). |
| **Seguridad**<br>*Más estándar* | La versión anterior tenía roles predefinidos que podía usar para administrar el acceso a la instancia. | La nueva versión se integra con el mismo servicio de back-end de [control de acceso basado en roles (RBAC)](../role-based-access-control/overview.md) de Azure que usan otros servicios de Azure. Esto puede facilitar la autenticación entre otros servicios de Azure en la solución, como IoT Hub, Azure Functions, Event Grid, etc.<br>Con RBAC, puede seguir usando roles predefinidos o puede compilar y configurar roles personalizados.<br><br>Más información en [*Conceptos: Seguridad para las soluciones de Azure Digital Twins*](concepts-security.md). |
| **Escalabilidad**<br>*Superior* | La versión anterior tenía limitaciones de escala para dispositivos, mensajes, gráficos y unidades de escalado. Solo se admitía una instancia de Azure Digital Twins por suscripción.  | La nueva versión se basa en una nueva arquitectura con una escalabilidad mejorada y tiene una mayor capacidad de proceso. También admite 10 instancias por región y por suscripción.<br><br>Consulte [*Referencia: Límites de servicio de la versión preliminar pública*](reference-service-limits.md) para obtener detalles de los límites actuales de la versión preliminar pública. |

## <a name="service-limits-in-public-preview"></a>Límites de servicio de la versión preliminar pública

Para obtener una lista de los límites de Azure Digital Twins durante esta versión preliminar pública, consulte [*Referencia: Límites de servicio de la versión preliminar pública*](reference-service-limits.md).

## <a name="next-steps"></a>Pasos siguientes

A continuación, obtenga más información sobre el trabajo con Azure Digital Twins con el primer tutorial:

> [!div class="nextstepaction"]
> [*Tutorial: Programación de una aplicación cliente*](tutorial-code.md)
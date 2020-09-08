---
title: Uso de Azure Portal para invocar métodos directos
description: Este artículo es una introducción al uso del Azure Portal para invocar métodos directos.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.subservice: ''
ms.workload: ''
ms.topic: how-to
ms.custom: ''
ms.date: 07/24/2020
ms.author: inhenkel
ms.openlocfilehash: 9d784e1697dfbcbfec509c1a51c9b832b533c97b
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87830757"
---
# <a name="how-to-use-azure-portal-to-invoke-direct-methods"></a>Uso de Azure Portal para invocar métodos directos

IoT Hub permite invocar [métodos directos](../../iot-hub/iot-hub-devguide-direct-methods.md#method-invocation-for-iot-edge-modules) en dispositivos perimetrales desde la nube. El módulo Live Video Analytics on IoT Edge (LVA) expone varios [métodos directos](./direct-methods.md) que están disponibles para definir, implementar y crear instancias de diferentes flujos de trabajo con fines de análisis de vídeo en directo.

En este artículo, obtendrá información sobre cómo invocar llamadas a métodos directos en Live Video Analytics para un módulo de IoT Edge a través de Azure Portal.

## <a name="prerequisites"></a>Requisitos previos

* El módulo Live Video Analytics on IoT Edge tiene que estar ejecutándose en su dispositivo perimetral mediante uno de los métodos descritos en [Inicio rápido: Live Video Analytics on IoT Edge](./get-started-detect-motion-emit-events-quickstart.md), o bien a través del [portal](./deploy-iot-edge-device.md).

* Comprender [Live Video Analytics](./overview.md) y el [concepto de grafo multimedia](./media-graph-concept.md).

## <a name="invoking-direct-methods-via-azure-portal"></a>Invocación de métodos directos a través de Azure Portal

Los [métodos directos](./direct-methods.md) expuestos por el módulo LVA se puede invocar a través de Azure Portal. Los pasos siguientes incluyen los detalles de un método directo. Es posible invocar otros métodos directos por medio de pasos similares. Sin embargo, cada método directo requiere un cuerpo JSON específico.

Utilice la llamada de método `GraphTopologyList` para obtener una lista de todas las topologías de grafos implementadas actualmente en el módulo Live Video Analytics on IoT Edge. Siga estos pasos para invocar el método directo:

1. Inicie sesión en Azure Portal.
1. Busque el grupo de recursos correspondiente en la página principal del portal para buscar el módulo IoT Hub. Si ya conoce el módulo IoT Hub, selecciónelo.
    ![grupo de recursos en la página principal del portal](media/use-azure-portal-to-invoke-directs-methods/portal-rg-home.png)
1. Una vez en la página IoT Hub, seleccione IoT Edge en Administración automática de dispositivos para enumerar los distintos identificadores de dispositivo. Seleccione el identificador de dispositivo correspondiente para enumerar los módulos que están ejecutándose en el dispositivo.
    ![página iot hub](media/use-azure-portal-to-invoke-directs-methods/iot-hub-page.png)
1. Seleccione el módulo Live Video Analytics on IoT Edge para abrir su página de configuración.<br><br>
    ![Selección del módulo Live Video Analytics on IoT Edge para abrir su página de configuración](media/use-azure-portal-to-invoke-directs-methods/modules.png)
1. Seleccione la opción de menú Método directo. <br><br>
    ![Selección de la opción de menú Método directo](media/use-azure-portal-to-invoke-directs-methods/module-details.png)
    > [!NOTE]
    > Puede que tenga que añadir un valor en las secciones de cadena de conexión, tal como se muestra en la página actual. No es necesario ocultar ni cambiar nada en la sección Nombre del valor. Es correcto dejar que sea pública.

1. Escriba *GraphTopologyList* en el campo **Nombre del valor**.
1. Copie y pegue el siguiente contenido JSON en el campo **Carga**.
    ```json
    {
    "@apiVersion":
    }
    ```
1. Seleccione el botón **Invocar método** en la parte superior de la página.<br><br>
    ![botón de invocar método](media/use-azure-portal-to-invoke-directs-methods/direct-method.png)
1. Debería ver un mensaje de estado 200 en el área de **resultados**.<br><br>
    ![tiempo de espera de conexión agotado](media/use-azure-portal-to-invoke-directs-methods/connection-timeout.png)

## <a name="responses"></a>Respuestas

| Condición             | Código de estado | Código de error detallado |
|-----------------------|-------------|---------------------|
| Correcto               | 200         | N/D                 |
| Errores de usuario generales   | Intervalo 400   |                     |
| Errores generales del servidor | Intervalo 500   |                     |

## <a name="next-steps"></a>Pasos siguientes

Hay otros métodos directos disponibles en la página [Métodos directos](./direct-methods.md).

> [!NOTE]
> Una instancia de grafo crea instancias de una topología específica. Por lo tanto, asegúrese de haber establecido la topología correcta antes de crear una instancia de grafo.

[Inicio rápido: Detecte eventos Motion Emit](./get-started-detect-motion-emit-events-quickstart.md) es una buena fuente de referencia para comprender la secuencia exacta de llamadas a métodos directos que se van a realizar.
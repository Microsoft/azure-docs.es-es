---
title: 'Inicio rápido: Configuración de la solución'
description: En esta guía de inicio rápido, aprenda a configurar la solución de IoT de un extremo a otro mediante Azure Defender para IoT.
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
ms.openlocfilehash: 3ecb2e2022cab80abf6a8692ac8cc3ba54eff1e4
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90946250"
---
# <a name="quickstart-configure-your-azure-defender-for-iot-solution"></a>Inicio rápido: Configuración de la solución de Azure Defender para IoT

En este artículo se explica cómo realizar la configuración inicial de una solución de seguridad de IoT mediante Defender para IoT.

## <a name="what-is-defender-for-iot"></a>¿Qué es Defender para IoT?

Defender para IoT proporciona una seguridad completa, de un extremo a otro, para las soluciones de IoT que usen Azure.

Con Defender para IoT se puede supervisar toda una solución de IoT desde un solo panel, así como presentar todos los dispositivos de IoT, las plataformas de IoT y los recursos back-end de Azure.

Una vez que se habilita en IoT Hub, Defender para IoT identifica automáticamente otros servicios de Azure, que también estén conectados a IoT Hub y que estén relacionados con la solución de IoT.

Además de la detección automática de relaciones, se pueden elegir los otros grupos de recursos de Azure que se etiquetan como parte de la solución de IoT.

Las opciones que se seleccionen permiten agregar suscripciones completas, grupos de recursos o recursos individuales.

Después de definir todas las relaciones de los recursos, Defender para IoT aprovecha Defender para generar alertas y recomendaciones de seguridad relativas a estos recursos.

## <a name="add-azure-resources-to-your-iot-solution"></a>Adición de recursos de Azure a una solución de IoT

Para agregar un recurso nuevo a la solución de IoT, siga estos pasos:

1. Abra **IoT Hub** en Azure Portal.
1. Seleccione y abra **Configuración** en la sección **Seguridad** del menú de la izquierda y, a continuación, seleccione **Recursos supervisados**.
1. Seleccione **Editar** y los recursos supervisados que pertenezcan a la solución de IoT.
1. Haga clic en **Agregar**.

Felicidades. Ha agregado un nuevo grupo de recursos a la solución de IoT.

Defender para IoT ahora supervisa los grupos de recursos que acaba de agregar y muestra alertas y recomendaciones de seguridad pertinentes como parte de la solución de IoT.

## <a name="next-steps"></a>Pasos siguientes

Pase al siguiente artículo para aprender a crear módulos de seguridad...

> [!div class="nextstepaction"]
> [Creación de módulos de seguridad](quickstart-create-security-twin.md)

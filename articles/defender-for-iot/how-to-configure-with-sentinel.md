---
title: Configuración de Azure Sentinel para Defender para IoT (versión preliminar)
description: Explica cómo configurar Azure Sentinel para recibir datos de la solución de Defender para IoT.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/09/2020
ms.author: mlottner
ms.openlocfilehash: 71147352c5b75195ed0dff2b05acc5315f3183cc
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932399"
---
# <a name="connect-your-data-from-defender-for-iot-to-azure-sentinel-preview"></a>Conexión de los datos de Defender para IoT con Azure Sentinel (versión preliminar)

El conector de datos de Azure Security Center para IoT en Azure Sentinel se encuentra actualmente en versión preliminar pública. Esta característica se ofrece sin contrato de nivel de servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/en-us/support/legal/preview-supplemental-terms/).

En esta guía, aprenderá a conectar datos de Defender para IoT a Azure Sentinel.

> [!div class="checklist"]
> * Prerrequisitos
> * Configuración de conexión
> * Vista de alerta de Log Analytics

Conecte alertas de Defender para IoT y transmítalas directamente a Azure Sentinel.

## <a name="prerequisites"></a>Prerrequisitos

- Debe tener permisos de **lectura** y **escritura** de área de trabajo.
- **Defender para IoT** debe estar **habilitado** en las instancias pertinentes de IoT Hub.
- Debe tener permisos de **lectura** y **escritura** en la instancia de **Azure IoT Hub** que quiera conectar.
- También debe tener permisos de **lectura** y **escritura** en el **grupo de recursos de Azure IoT Hub**.

> [!NOTE]
> Debe tener la licencia de nivel Estándar de Azure Security Center en ejecución en la suscripción para enviar alertas de recursos de Azure generales. Con la licencia de nivel Gratis necesaria para Defender para IoT, solo se reenviarán a Azure Sentinel las alertas relacionadas con Defender para IoT.

## <a name="connect-to-defender-for-iot"></a>Conexión a Defender para IoT

1. En Azure Sentinel, seleccione **Conectores de datos** y, luego, haga clic en el mosaico **Azure Defender para IoT**.
1. Desde la parte inferior del panel derecho, haga clic en **Abrir la página del conector**.
1. Haga clic en **Conectar** junto a cada suscripción de IoT Hub cuyas alertas y alertas de dispositivo quiera transmitir a Azure Sentinel.
    - Si Defender para IoT no está habilitado en ese centro, verá el mensaje de advertencia Habilitar. Haga clic en el vínculo **Habilitar** para iniciar y habilitar el servicio.
1. Puede decidir si quiere que las alertas de Defender para IoT generen incidentes automáticamente en Azure Sentinel. En **Create incidents** (Crear incidentes), seleccione **Habilitar** para que la regla pueda crear incidentes de forma automática a partir de las alertas generadas.  Esta regla puede cambiarse o editarse en **Análisis** > **Reglas activas**.

> [!NOTE]
>Puede tardar 10 segundos o más en actualizar la lista de centros de conectividad tras realizar cambios en la conexión.

## <a name="log-analytics-alert-display"></a>Visualización de alertas de Log Analytics

Para usar el esquema correspondiente en Log Analytics a fin de mostrar las alertas de Defender para IoT, haga lo siguiente:

1. Abra **Registros** > **SecurityInsights** > **SecurityAlert** o busque **SecurityAlert**.
1. Use el siguiente filtro kql a fin de ver solo las alertas generadas por Defender para IoT:

```kusto
SecurityAlert | where ProductName == "Defender for IoT"
```

### <a name="service-notes"></a>Notas del servicio

Tras conectar una instancia de IoT Hub, los datos del centro de conectividad están disponibles en Azure Sentinel aproximadamente 15 minutos más tarde.

## <a name="next-steps"></a>Pasos siguientes

En este documento, ha obtenido información sobre cómo conectar Defender para IoT a Azure Sentinel. Para obtener más información sobre la detección de amenazas y el acceso a datos de seguridad, consulte los siguientes artículos:

- Aprenda a usar Azure Sentinel para [obtener visibilidad de los datos y de posibles amenazas](https://docs.microsoft.com/azure/sentinel/quickstart-get-visibility).

- Aprenda a [acceder a los datos de seguridad de IoT](how-to-security-data-access.md).

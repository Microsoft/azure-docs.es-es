---
title: Conexión de las alertas de Orca Security a Azure Sentinel | Microsoft Docs
description: Aprenda a conectar los datos de las alertas de Orca Security a Azure Sentinel para ver los paneles, crear alertas personalizadas y mejorar la investigación.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/17/2020
ms.author: yelevin
ms.openlocfilehash: 735f4b447d7258a9b444f3b75a6537dec20b1307
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87076277"
---
# <a name="connect-your-orca-security-alerts-to-azure-sentinel"></a>Conexión de las alertas de Orca Security a Azure Sentinel 

> [!IMPORTANT]
> El conector de alertas de Orca Security en Azure Sentinel se encuentra actualmente en versión preliminar pública.
> Esta característica se ofrece sin contrato de nivel de servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

El conector de las alertas de seguridad de Orca permite incorporar fácilmente las alertas de la solución de seguridad de [alertas de Orca](https://orca.security/) a Azure Sentinel, de modo que pueda verlas en los libros, usarlas para crear alertas personalizadas e incorporarlas para mejorar la investigación. La integración entre las alertas de Orca Security y Azure Sentinel usa la API REST.

> [!NOTE]
> Los datos se almacenarán en la ubicación geográfica del área de trabajo en la que Azure Sentinel se ejecute.

## <a name="configure-and-connect-orca-security-alerts"></a>Configuración y conexión de las alertas de Orca Security

Las alertas de Orca Security pueden integrar y exportar los registros directamente a Azure Sentinel.

1. En el portal de Azure Sentinel, haga clic en **Conectores de datos** y seleccione **Orca Security alerts** (alertas de Orca Security) y, luego, **Open connector page** (Abrir página de conectores).

2. Consulte https://orcasecurity.zendesk.com/hc/en-us/articles/360043941992-Azure-Sentinel-integration para completar la integración desde la plataforma de Orca.

## <a name="find-your-data"></a>Búsqueda de sus datos

Una vez establecida correctamente una conexión, los datos aparecen en Log Analytics en la sección **CustomLogs** de la tabla **OrcaAlerts_CL**.
Para usar el esquema correspondiente en Log Analytics para las alertas de Orca, busque `OrcaAlerts_CL`.

## <a name="validate-connectivity"></a>Validar conectividad
Hasta que los registros empiecen a aparecer en Log Analytics, pueden transcurrir más de 20 minutos. 


## <a name="next-steps"></a>Pasos siguientes
En este documento, ha aprendido a conectar alertas de Orca Security a Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:
- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a [detectar amenazas con Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Use libros](tutorial-monitor-your-data.md) para supervisar los datos.


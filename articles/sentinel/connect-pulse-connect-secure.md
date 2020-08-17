---
title: Conexión de datos de Pulse Connect Secure a Azure Sentinel | Microsoft Docs
description: Aprenda a conectar datos de Pulse Connect Secure a Azure Sentinel.
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
ms.openlocfilehash: f7f6de717f0ed9a2e27e15199b6f0ea566e0fdcd
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87021637"
---
# <a name="connect-your-pulse-connect-secure-to-azure-sentinel"></a>Conexión de Pulse Connect Secure a Azure Sentinel

> [!IMPORTANT]
> El conector de datos Pulse Connect Secure de Azure Sentinel se encuentra actualmente en versión preliminar pública.
> Esta característica se ofrece sin contrato de nivel de servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

En este artículo se explica cómo conectar el dispositivo de [Pulse Connect Secure](https://www.pulsesecure.net/products/pulse-connect-secure/) a Azure Sentinel. El conector de datos de Pulse Connect Secure permite conectar fácilmente los registros de Pulse Connect Secure con Azure Sentinel para ver paneles, crear alertas personalizadas y mejorar la investigación. La integración entre Pulse Connect Secure y Azure Sentinel usa Syslog.

> [!NOTE]
> Los datos se almacenarán en la ubicación geográfica del área de trabajo en la que Azure Sentinel se ejecute.

## <a name="forward-pulse-connect-secure-logs-to-the-syslog-agent"></a>Reenvío de registros de Pulse Connect Secure al agente de Syslog  

Configure Pulse Connect Secure para reenviar mensajes de Syslog a su área de trabajo de Azure a través del agente de Syslog.

1. En el portal de Azure Sentinel, haga clic en **Data connectors** (Conectores de datos) y seleccione el conector **Pulse Connect Secure**.

1. Seleccione **Open connector page** (Abrir página del conector).

1. Siga las instrucciones que aparecen en la página **Pulse Connect Secure**.

## <a name="find-your-data"></a>Búsqueda de sus datos

Una vez establecida una conexión correcta, los datos aparecen en Log Analytics bajo Syslog.

## <a name="validate-connectivity"></a>Validar conectividad

Hasta que los registros empiecen a aparecer en Log Analytics, pueden transcurrir más de 20 minutos. 

## <a name="next-steps"></a>Pasos siguientes

En este documento, ha aprendido a conectar Pulse Connect Secure a Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:

- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a [detectar amenazas con Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Use libros](tutorial-monitor-your-data.md) para supervisar los datos.
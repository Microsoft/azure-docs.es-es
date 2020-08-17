---
title: Conexión de datos de Infoblox Network Identity Operating System (NIOS) a Azure Sentinel | Microsoft Docs
description: Aprenda a conectar datos de Infoblox Network Identity Operating System (NIOS) a Azure Sentinel.
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
ms.openlocfilehash: c8af1c32607e5fc7bf548b40069e18923c666303
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87089658"
---
# <a name="connect-your-infoblox-nios-to-azure-sentinel"></a>Conexión de Infoblox NIOS a Azure Sentinel

> [!IMPORTANT]
> El conector de datos de Infoblox NIOS en Azure Sentinel se encuentra actualmente en versión preliminar pública.
> Esta característica se ofrece sin contrato de nivel de servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

En este artículo se explica cómo conectar su [dispositivo de Infoblox Network Identity Operating System (NIOS)](https://www.infoblox.com/glossary/network-identity-operating-system-nios/) a Azure Sentinel. El conector de datos de Infoblox NIOS permite conectar fácilmente los registros de Infoblox con Azure Sentinel para ver paneles, crear alertas personalizadas y mejorar la investigación. La integración entre Infoblox NIOS y Azure Sentinel usa Syslog.

> [!NOTE]
> Los datos se almacenarán en la ubicación geográfica del área de trabajo en la que Azure Sentinel se ejecute.

## <a name="forward-infoblox-logs-to-the-syslog-agent"></a>Reenvío de los registros de Infoblox al agente de Syslog  

Configure Infoblox para reenviar mensajes de Syslog a su área de trabajo de Azure a través del agente de Syslog.

1. En el portal de Azure Sentinel, haga clic en **Data connectors** (Conectores de datos) y seleccione el conector **Infoblox NIOS**.

1. Seleccione **Open connector page** (Abrir página del conector).

1. Siga las instrucciones de la página **Infoblox NIOS**.

## <a name="find-your-data"></a>Búsqueda de sus datos

Una vez establecida una conexión correcta, los datos aparecen en Log Analytics bajo Syslog.

## <a name="validate-connectivity"></a>Validar conectividad

Hasta que los registros empiecen a aparecer en Log Analytics, pueden transcurrir más de 20 minutos. 

## <a name="next-steps"></a>Pasos siguientes

En este documento, ha aprendido a conectar Infoblox NIOS a Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:

- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a [detectar amenazas con Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Use libros](tutorial-monitor-your-data.md) para supervisar los datos.
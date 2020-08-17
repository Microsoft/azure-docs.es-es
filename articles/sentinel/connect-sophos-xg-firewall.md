---
title: Conexión datos de Sophos XG Firewall a Azure Sentinel| Microsoft Docs
description: Aprenda a conectar datos de Sophos XG Firewals a Azure Sentinel.
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
ms.openlocfilehash: b48773272e050b47af73b197d6f1c8156318fd71
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095473"
---
# <a name="connect-your-sophos-xg-firewall-to-azure-sentinel"></a>Conexión de Sophos XG Firewal a Azure Sentinel

> [!IMPORTANT]
> El conector de datos de Sophos XG Firewal en Azure Sentinel se encuentra actualmente en versión preliminar pública.
> Esta característica se ofrece sin contrato de nivel de servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

En este artículo se explica cómo conectar el dispositivo de [Sophos XG Firewal](https://www.sophos.com/products/next-gen-firewall.aspx) con Azure Sentinel. El conector de datos de Sophos XG Firewal permite conectar fácilmente los registros de Sophos XG Firewal con Azure Sentinel para ver paneles, crear alertas personalizadas y mejorar la investigación. La integración entre Sophos XG Firewal y Azure Sentinel usa Syslog.

> [!NOTE]
> Los datos se almacenarán en la ubicación geográfica del área de trabajo en la que Azure Sentinel se ejecute.

## <a name="forward-sophos-xg-firewall-logs-to-the-syslog-agent"></a>Reenvío de registros de Sophos XG Firewal al agente de Syslog  

Configure Sophos XG Firewal para reenviar mensajes de Syslog a su área de trabajo de Azure a través del agente de Syslog.

1. En el portal de Azure Sentinel, haga clic en **Conectores de datos** y seleccione el conector **Sophos XG Firewal**.

1. Seleccione **Open connector page** (Abrir página del conector).

1. Siga las instrucciones que aparecen en la página de **Sophos XG Firewal**.

## <a name="find-your-data"></a>Búsqueda de sus datos

Una vez establecida una conexión correcta, los datos aparecen en Log Analytics bajo Syslog.

## <a name="validate-connectivity"></a>Validar conectividad

Hasta que los registros empiecen a aparecer en Log Analytics, pueden transcurrir más de 20 minutos.

## <a name="next-steps"></a>Pasos siguientes

En este documento, ha aprendido a conectar Sophos XG Firewal a Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:

- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a [detectar amenazas con Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Use libros](tutorial-monitor-your-data.md) para supervisar los datos.
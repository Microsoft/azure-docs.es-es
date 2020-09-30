---
title: Conexión de los datos de Microsoft Defender para Endpoint a Azure Sentinel | Microsoft Docs
description: Obtenga información sobre cómo conectar los datos de Microsoft Defender para Endpoint (anteriormente Microsoft Defender ATP) a Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/16/2020
ms.author: yelevin
ms.openlocfilehash: d1de9270534a3550d8db043fdd6bcfe56ec31bda
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90895601"
---
# <a name="connect-alerts-from-microsoft-defender-for-endpoint-formerly-microsoft-defender-atp"></a>Conexión de alertas de Microsoft Defender para Endpoint (anteriormente Microsoft Defender ATP)

> [!IMPORTANT]
>
> - **Microsoft Defender para Endpoint** se conocía anteriormente como **Protección contra amenazas avanzada de Microsoft Defender** o **MDATP**.
>
>     Puede ver que el nombre anterior todavía está en uso en el producto (incluido su conector de datos en Azure Sentinel) durante un período de tiempo.
>
> - La ingesta de alertas de Microsoft Defender para Endpoint se encuentra actualmente en versión preliminar pública.
> Esta característica se ofrece sin contrato de nivel de servicio y no se recomienda para cargas de trabajo de producción.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

El conector de [Microsoft Defender para Endpoint](https://docs.microsoft.com/windows/security/threat-protection/microsoft-defender-atp/microsoft-defender-advanced-threat-protection) permite transmitir alertas de Microsoft Defender para Endpoint a Azure Sentinel. Esto le permitirá analizar de manera más exhaustiva los eventos de seguridad en su organización y crear cuadernos de estrategias para obtener una respuesta efectiva e inmediata.

> [!NOTE]
>
> Para ingerir los registros nuevos de datos sin procesar de la [búsqueda avanzada](https://docs.microsoft.com/windows/security/threat-protection/microsoft-defender-atp/advanced-hunting-overview) de Microsoft Defender para Endpoint, use el conector nuevo para Microsoft 365 Defender (anteriormente Protección contra amenazas de Microsoft; [vea la documentación](./connect-microsoft-365-defender.md)).

## <a name="prerequisites"></a>Requisitos previos

- Debe tener una licencia válida de Microsoft Defender para Endpoint, tal como se describe en [Configuración de la implementación de Microsoft Defender para Endpoint](https://docs.microsoft.com/windows/security/threat-protection/microsoft-defender-atp/licensing). 

- Debe ser administrador global o de seguridad en el inquilino de Azure Sentinel.

## <a name="connect-to-microsoft-defender-for-endpoint"></a>Conexión a Microsoft Defender para Endpoint

Si Microsoft Defender para Endpoint ya está implementado e ingiriendo datos, las alertas se pueden transmitir a Azure Sentinel muy fácilmente.

1. En Azure Sentinel, seleccione **Data Connectors** (Conectores de datos), **Microsoft Defender para Endpoint** (es posible que todavía se llame *Protección contra amenazas avanzada de Microsoft Defender*) desde la galería y seleccione **Open connector page** (Abrir la página del conector).

1. Haga clic en **Conectar**. 

1. Para consultar las alertas de Microsoft Defender para Endpoint en **Registros**, escriba **SecurityAlert** en la ventana de consulta y agregue un filtro en el que el **nombre de proveedor** sea **MDATP**.

## <a name="next-steps"></a>Pasos siguientes
En este documento, ha descubierto cómo conectar Microsoft Defender para Endpoint a Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:
- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a [detectar amenazas con Azure Sentinel](tutorial-detect-threats.md).

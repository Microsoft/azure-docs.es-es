---
title: Conexión de datos de Azure Defender a Azure Sentinel
description: Descubra cómo conectar alertas de Azure Defender desde Azure Security Center y transmitirlas a Azure Sentinel.
author: yelevin
manager: rkarlin
ms.assetid: d28c2264-2dce-42e1-b096-b5a234ff858a
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.topic: how-to
ms.date: 09/07/2020
ms.author: yelevin
ms.openlocfilehash: b1188e533039b0137cebb22652d9921418c41deb
ms.sourcegitcommit: f8d2ae6f91be1ab0bc91ee45c379811905185d07
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89659643"
---
# <a name="connect-azure-defender-alert-data-from-azure-security-center"></a>Conexión de datos de alertas de Azure Defender desde Azure Security Center

Use el conector de alertas de Azure Defender para ingerir alertas de Azure Defender desde [Azure Security Center](../security-center/security-center-intro.md) y transmitirlas a Azure Sentinel. 

## <a name="prerequisites"></a>Requisitos previos

- El usuario debe tener el rol de lector de seguridad en la suscripción de los registros que transmita.

- Tendrá que habilitar Azure Defender en Azure Security Center (el nivel Estándar ya no existe, por lo que ya no es un requisito de licencia).

## <a name="connect-to-azure-defender"></a>Conexión a Azure Defender

1. En el menú de navegación de Azure Sentinel, seleccione **Conectores de datos**.

1. En la galería de conectores de datos, seleccione **Azure Defender alerts from ASC** (Alertas de Azure Defender desde ASC) (es posible que todavía se le llame Azure Security Center) y haga clic en el botón **Open connector page** (Abrir la página del conector).

1. En **Configuration** (Configuración), haga clic en **Connect** (Conectar) junto a cada suscripción cuyas alertas quiera transmitir a Azure Sentinel. El botón Conectar solo estará disponible si tiene los permisos necesarios.

1. Puede seleccionar si quiere que las alertas de Azure Defender generen incidentes automáticamente en Azure Sentinel. En **Create incidents** (Crear incidentes), seleccione **Enabled** (Habilitado) para activar la regla de análisis predeterminada que crea automáticamente incidentes a partir de alertas. Luego, puede editar esta regla en **Analytics** (Análisis), en la pestaña **Active rules** (Reglas activas).

1. Para usar el esquema pertinente en Log Analytics a fin de encontrar alertas de Azure Defender, busque **SecurityAlert**.

## <a name="next-steps"></a>Pasos siguientes
En este documento, ha obtenido información sobre cómo conectar Azure Defender a Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:
- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a [detectar amenazas con Azure Sentinel](tutorial-detect-threats-built-in.md).

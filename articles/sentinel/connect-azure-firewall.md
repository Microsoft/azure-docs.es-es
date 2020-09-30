---
title: Conexión de datos de Azure Firewall a Azure Sentinel
description: Aprenda a conectar datos de Azure Firewall a Azure Sentinel.
author: yelevin
manager: rkarlin
ms.assetid: bfa2eca4-abdc-49ce-b11a-0ee229770cdd
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.topic: how-to
ms.date: 08/30/2020
ms.author: yelevin
ms.openlocfilehash: 05a2869152def0fc9a44defaa21f0643d9fdbb08
ms.sourcegitcommit: 59ea8436d7f23bee75e04a84ee6ec24702fb2e61
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/07/2020
ms.locfileid: "89504883"
---
# <a name="connect-data-from-azure-firewall"></a>Conexión de datos de Azure Firewall

> [!IMPORTANT]
> El conector de datos de Azure Firewall en Azure Sentinel se encuentra actualmente en versión preliminar pública.
> Esta característica se ofrece sin contrato de nivel de servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Azure Firewall es un servicio de seguridad de red administrado y basado en la nube que protege los recursos de Azure Virtual Network. Se trata de un firewall como servicio con estado completo que incorpora alta disponibilidad y escalabilidad a la nube sin restricciones. 

Los registros de Azure Firewall se pueden conectar a Azure Sentinel, de forma que es posible ver los datos de registro en los libros, usarlos para crear alertas personalizadas e incorporarlos para mejorar su investigación.

Más información sobre la [supervisión de los registros de Azure Firewall](https://docs.microsoft.com/azure/firewall/tutorial-diagnostics).

## <a name="prerequisites"></a>Requisitos previos

- Debe tener permisos de lectura y escritura en el área de trabajo de Azure Sentinel.

## <a name="connect-to-azure-firewall"></a>Conexión a Azure Firewall
    
1. En el menú de navegación de Azure Sentinel, seleccione **Data connectors** (Conectores de datos).

1. Seleccione **Azure Firewall** en la galería de conectores de datos y, luego, seleccione **Open Connector Page** (Abrir página del conector) en el panel de vista previa.

1. Habilite **Registros de diagnóstico** en todos los firewalls cuyos registros quiere conectar:

    1. Seleccione el vínculo **Open Azure Firewall resource >** (Abrir recurso de Azure Firewall).

    1. En el menú de navegación **Firewalls**, seleccione **Configuración de diagnóstico**.

    1. Seleccione **+ Agregar configuración de diagnóstico**.

    1. En la pantalla **Configuración de diagnóstico**, escriba un nombre en el campo **Nombre de la configuración de diagnóstico**.
    
    1. Marque la casilla **Enviar a Log Analytics**. Se mostrarán dos nuevos campos debajo. Elija la **suscripción** y el **área de trabajo de Log Analytics** pertinentes (donde reside Azure Sentinel).

    1. Marque las casillas de los tipos de regla cuyos registros quiere ingerir. Se recomiendan **AzureFirewallApplicationRule** y **AzureFirewallNetworkRule**.

    1. En la parte superior de la pantalla, seleccione **Guardar**.

1. Para usar el esquema correspondiente de Log Analytics para las alertas de Azure Firewall, busque **AzureDiagnostics**.

> [!NOTE]
>
> Con este conector de datos concreto, los indicadores de estado de conectividad (una franja de color en la galería de conectores de datos y los iconos de conexión situados junto a los nombres de tipo de datos) se mostrarán como *Conectado* (verde) solo si los datos se han ingerido en algún momento en las dos últimas semanas. Transcurridas dos semanas sin ingesta de datos, el conector se mostrará como desconectado. En el momento en que circulen más datos, se devolverá el estado *conectado*.

## <a name="next-steps"></a>Pasos siguientes
En este documento, ha aprendido a conectar registros de Azure Firewall a Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:
- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a [detectar amenazas con Azure Sentinel](tutorial-detect-threats-built-in.md).

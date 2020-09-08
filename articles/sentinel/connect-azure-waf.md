---
title: Conexión de datos del firewall de aplicaciones web (WAF) de Azure a Azure Sentinel
description: Aprenda a conectar datos de WAF de Azure a Azure Sentinel.
author: yelevin
manager: rkarlin
ms.assetid: bfa2eca4-abdc-49ce-b11a-0ee229770cdd
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.topic: how-to
ms.date: 05/07/2020
ms.author: yelevin
ms.openlocfilehash: 8f21b415ef36442d6ac1aac518cd1327f70b8927
ms.sourcegitcommit: 64ad2c8effa70506591b88abaa8836d64621e166
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88263863"
---
# <a name="connect-data-from-azure-web-application-firewall-waf"></a>Conexión de datos del firewall de aplicaciones web (WAF) de Azure

Las aplicaciones web son cada vez más el objetivo de ataques malintencionados que aprovechan vulnerabilidades habitualmente conocidas. El firewall de aplicaciones web (WAF) de Azure proporciona protección centralizada de las aplicaciones web frente a vulnerabilidades de seguridad y amenazas comunes, como la inyección de código y el scripting entre sitios. WAF de Azure se puede implementar en el servicio [Azure Application Gateway](https://docs.microsoft.com/azure/web-application-firewall/ag/ag-overview), el servicio [Azure Front Door](https://docs.microsoft.com/azure/web-application-firewall/afds/afds-overview) y mediante una directiva WAF de [Azure Content Delivery Network (CDN)](https://docs.microsoft.com/azure/web-application-firewall/cdn/cdn-overview) (esta última actualmente en versión preliminar pública).
Los registros de WAF de Azure se pueden conectar a Azure Sentinel, de forma que es posible ver los datos de registro en los libros, usarlos para crear alertas personalizadas e incorporarlos para mejorar su investigación.

## <a name="prerequisites"></a>Requisitos previos

- Debe tener permisos de lectura y escritura en el área de trabajo de Azure Sentinel.

## <a name="connect-to-azure-waf"></a>Conexión a WAF de Azure

### <a name="instructions-tab"></a>Pestaña Instrucciones

1. En el menú de navegación de Azure Sentinel, seleccione **Data connectors** (Conectores de datos).

1. Seleccione **Azure web application firewall (WAF)** (Firewall de aplicaciones web [WAF]) en la galería de conectores de datos y, luego, seleccione **Open Connector Page** (Abrir página del conector) en el panel de vista previa.

1. Seleccione el vínculo correspondiente al tipo de recurso WAF cuyos registros quiere conectar: **Open Application Gateway resource >** , **Open Front Door resource >** o **Open Content Delivery Network (CDN) WAF policy >** (Abrir recurso de Application Gateway > Abrir recurso de Front Door > Abrir directiva WAF de Content Delivery Network [CDN]) y, una vez en la pantalla de la lista de recursos, elija un recurso WAF de la lista.

    1. En el menú de navegación del recurso WAF, seleccione **Configuración de diagnóstico**.

    1. Seleccione **+ Agregar configuración de diagnóstico**.

    1. En la pantalla **Configuración de diagnóstico**, escriba un nombre en el campo **Nombre de la configuración de diagnóstico**.

    1. Haga clic en la casilla **Enviar a Log Analytics**. Se mostrarán dos nuevos campos debajo. Elija la **suscripción** y el **área de trabajo de Log Analytics** pertinentes (donde reside Azure Sentinel).

    1. Haga clic en las casillas de los tipos de regla cuyos registros quiere ingerir. Nuestras recomendaciones para cada tipo de recurso:

        | Resource | Registros para seleccionar para la ingesta |
        |----------|------------------------------|
        | Application Gateway | ApplicationGatewayAccessLog<br>ApplicationGatewayFirewallLog |
        | Front Door          | FrontdoorAccessLog<br>FrontdoorWebApplicationFirewallLog |
        | Directiva WAF de CDN      | WebApplicationFirewallLogs |
        |

    1. Seleccione **Guardar**.

### <a name="next-steps-tab"></a>Pestaña Pasos siguientes

- Consulte los libros, los ejemplos de consulta y las plantillas de regla de análisis recomendados que se incluyen con el conector de datos **Firewall de aplicaciones web de Azure** para obtener información sobre los datos de registro de WAF de Azure.

- Para consultar los datos de WAF de Azure en **Registros**, escriba **AzureDiagnostics** en la ventana de consulta.

> [!NOTE]
>
> Con este conector de datos concreto, los indicadores de estado de conectividad (una franja de color en la galería de conectores de datos y los iconos de conexión situados junto a los nombres de tipo de datos) se mostrarán como *Conectado* (verde) solo si los datos se han ingerido en algún momento en las dos últimas semanas. Transcurridas dos semanas sin ingesta de datos, el conector se mostrará como desconectado. En el momento en que circulen más datos, se devolverá el estado *conectado*.

## <a name="next-steps"></a>Pasos siguientes
En este documento, ha aprendido a conectar registros de WAF de Azure a Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:
- Aprenda a [obtener visibilidad de los datos y de posibles amenazas](quickstart-get-visibility.md).
- Empiece a [detectar amenazas con Azure Sentinel](tutorial-detect-threats-built-in.md).

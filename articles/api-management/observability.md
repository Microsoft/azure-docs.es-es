---
title: Observabilidad en Azure API Management | Microsoft Docs
description: Información general de todas las opciones de observabilidad en Azure API Management.
services: api-management
documentationcenter: ''
author: begim
manager: alberts
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 06/01/2020
ms.author: apimpm
ms.openlocfilehash: 1ebebed465952bbb5d3e8f82ae1c7776c441c6b0
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095139"
---
# <a name="observability-in-azure-api-management"></a>Observabilidad en Azure API Management

La observabilidad es la capacidad de entender el estado interno de un sistema a partir de los datos que genera y la capacidad de explorar esos datos para responder a las preguntas sobre qué sucedió y por qué. 

Azure API Management ayuda a las organizaciones a centralizar la administración de todas las API. Dado que actúa como un único punto de entrada de todo el tráfico de las API, es un lugar ideal para observar estas. 

## <a name="observability-tools"></a>Herramientas de observabilidad

En la tabla siguiente se resumen todas las herramientas que administra API Management para observar las API, cada una de ellas es útil en uno o varios escenarios:

| Herramienta        | Útil para    | Retraso de datos | Retención | muestreo | Tipo de datos | habilitado|
|:------------- |:-------------|:---- |:----|:---- |:--- |:---- 
| **[API Inspector](api-management-howto-api-inspector.md)** | Prueba y depuración | Instantánea | Últimos 100 seguimientos | Activado por solicitud | Seguimientos de solicitud | Siempre
| **Análisis integrado** | Creación de informes y supervisión | Minutos | Vigencia | 100% | Informes y registros | Siempre |
| **[Métricas de Azure Monitor](api-management-howto-use-azure-monitor.md)** | Creación de informes y supervisión | Minutos | 93 días (actualizar para extender) | 100% | Métricas | Siempre |
| **[Registros de Azure Monitor](api-management-howto-use-azure-monitor.md)** | Informes, supervisión y depuración | Minutos | 31 días/5 GB (actualizar para extender) | 100 % (ajustable) | Registros | Opcional |
| **[Azure Application Insights](api-management-howto-app-insights.md)** | Informes, supervisión y depuración | Segundos | 90 días/5 GB (actualizar para extender) | Personalizado | Registros o métricas | Opcional |
| **[Registro a través del centro de eventos de Azure](api-management-howto-log-event-hubs.md)** | Escenarios personalizados | Segundos | Usuario administrado | Personalizado | Personalizado | Opcional |

### <a name="self-hosted-gateway"></a>Puerta de enlace autohospedada

Todas las herramientas mencionadas anteriormente son compatibles con la puerta de enlace administrada en la nube. Actualmente, la [puerta de enlace autohospedada](self-hosted-gateway-overview.md) no envía registros de diagnóstico a Azure Monitor. Sin embargo, es posible configurar y conservar los registros localmente donde se implementa la puerta de enlace autohospedada. Para obtener más información, consulte [Configuración de los registros y las métricas en la nube para la puerta de enlace autohospedada de Azure API Management](how-to-configure-cloud-metrics-logs.md) y [Configuración de los registros y las métricas locales para la puerta de enlace autohospedada de Azure API Management](how-to-configure-local-metrics-logs.md).

## <a name="next-steps"></a>Pasos siguientes

* [Siga nuestros tutoriales para más información sobre API Management](import-and-publish.md)

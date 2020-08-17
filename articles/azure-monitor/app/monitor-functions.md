---
title: 'Supervisión de aplicaciones que se ejecutan en Azure Functions con Application Insights: Azure Monitor | Microsoft Docs'
description: Azure Monitor se integra sin problemas en las aplicaciones que se ejecutan en Azure Functions, y permite supervisar el rendimiento y detectar problemas con las aplicaciones al instante.
ms.topic: conceptual
author: MS-jgol
ms.author: jgol
ms.date: 06/26/2020
ms.openlocfilehash: 6d790a6f02fdc07609e374639c6e452b9088262e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87024578"
---
# <a name="monitoring-azure-functions-with-azure-monitor-application-insights"></a>Supervisión de Azure Functions con Application Insights de Azure Monitor

[Azure Functions](../../azure-functions/functions-overview.md) ofrece integración incorporada en Azure Application Insights para supervisar las funciones. 

Application Insights recopila datos de registros, de rendimiento y de errores, y detecta automáticamente las anomalías en el rendimiento. Application Insights incluye herramientas de análisis eficaces que ayudan a diagnosticar problemas y comprender cómo se usan las funciones. Cuando tenga la visibilidad de los datos de su aplicación, puede mejorar continuamente el rendimiento y la facilidad de uso. Incluso puede usar Application Insights durante el desarrollo local de proyectos de aplicación de función. 

La instrumentación de Application Insights necesaria está integrada en Azure Functions. Lo único que necesita es una clave de instrumentación válida para conectar la aplicación de funciones a un recurso de Application Insights. La clave de instrumentación debe agregarse a la configuración de la aplicación cuando se cree el recurso de la aplicación de funciones en Azure. Si la aplicación de funciones aún no tiene esta clave, puede establecerla de forma manual. Para obtener más información, lea sobre la [supervisión de Azure Functions](../../azure-functions/functions-monitoring.md?tabs=cmd).

## <a name="distributed-tracing-for-java-applications-on-windows-public-preview"></a>Seguimiento distribuido para aplicaciones Java en Windows (versión preliminar pública)

> [!IMPORTANT]
> Esta característica se encuentra actualmente en versión preliminar pública para Java Azure Functions en Windows; no se admite el seguimiento distribuido para Java Azure Functions en Linux. En el plan de consumo, tiene un arranque en frío de 8 a 9 segundos.

Si las aplicaciones están escritas en Java, puede ver los datos enriquecidos de las aplicaciones de funciones, incluidas las solicitudes, las dependencias, los registros y las métricas. Los datos adicionales también le permiten ver y diagnosticar transacciones de un extremo a otro y ver el mapa de aplicación, que agrega muchas transacciones para mostrar una vista topológica de cómo interactúan los sistemas y cuáles son las tasas promedio de rendimiento y de errores.

Los diagnósticos de un extremo a otro y el mapa de aplicación proporcionan visibilidad en una única transacción/solicitud. Juntas, estas dos características son muy útiles para encontrar la causa principal de los problemas de confiabilidad y los cuellos de botella de rendimiento por solicitud.

### <a name="how-to-enable-distributed-tracing-for-java-function-apps"></a>¿Cómo se habilita el seguimiento distribuido para las aplicaciones de funciones de Java?

Vaya a la hoja de información general de la aplicación de funciones, vaya a Configuraciones. En Configuración de aplicación, haga clic en "+ Nueva configuración de la aplicación". Agregue las siguientes dos configuraciones de la aplicación con los valores siguientes y haga clic en Guardar en la parte superior izquierda. ¡Listo!

```
XDT_MicrosoftApplicationInsights_Java -> 1
ApplicationInsightsAgent_EXTENSION_VERSION -> ~2
```

## <a name="next-steps"></a>Pasos siguientes

* Lea más instrucciones e información sobre la [supervisión de Azure Functions](../../azure-functions/functions-monitoring.md).
* Obtenga información general sobre el [seguimiento distribuido](./distributed-tracing.md).
* Vea qué [mapa de aplicación](./app-map.md?tabs=net) puede servir a su empresa.
* Lea sobre las [solicitudes y dependencias para aplicaciones de Java](./java-in-process-agent.md).
* Obtenga más información sobre [Azure Monitor](../overview.md) y [Application Insights](./app-insights-overview.md).

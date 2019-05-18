---
title: Impacto del uso de Azure Application Insights | Microsoft Docs
description: Analice cómo las diferentes propiedades pueden afectar a las tasas de conversión de partes de sus aplicaciones.
services: application-insights
documentationcenter: ''
author: NumberByColors
manager: carmonm
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 01/08/2019
ms.reviewer: mbullwin
ms.pm_owner: daviste;NumberByColors
ms.author: daviste
ms.openlocfilehash: a1ff700bece2d64451294e72ebdf3c771ee644f8
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65604196"
---
# <a name="impact-analysis-with-application-insights"></a>Análisis de impacto con Application Insights

La herramienta Impacto analiza cómo influyen los tiempos de carga y otras propiedades en las tasas de conversión para las distintas partes de su aplicación. Para ser más exactos, detecta cómo **cualquier dimensión** de una **vista de página**, **evento personalizado** o **solicitud** afecta el uso de una diferente **vista de página** o **evento personalizado**. 

![Herramienta Impacto](./media/usage-impact/0001-impact.png)

## <a name="still-not-sure-what-impact-does"></a>¿Sigue sin tener claro la función de la herramienta Impacto?

La herramienta Impacto puede considerarse la herramienta por excelencia para establecer argumentos con alguien de su equipo acerca de cómo la lentitud de algún aspecto de su sitio está afectando a que los usuarios no se queden. Aunque los usuarios pueden tolerar una cierta lentitud, la herramienta Impacto proporciona una visión general de la mejor manera de equilibrar la optimización y el rendimiento para maximizar la conversión de usuario.

Pero analizar el rendimiento es solo un subconjunto de las capacidades de Impacto. Dado que la herramienta Impacto es compatible con dimensiones y eventos personalizados, la respuesta a preguntas acerca de cómo afecta la elección de explorador del usuario con las distintas tasas de conversión está a tan solo unos clics.

![Captura de pantalla de la conversión por exploradores](./media/usage-impact/0004-browsers.png)

> [!NOTE]
> El recurso de Application Insights debe contener vistas de páginas o eventos personalizados para poder utilizar la herramienta Impacto. [Aprenda a configurar la aplicación para que recopile vistas de página automáticamente con el SDK de JavaScript de Application Insights](../../azure-monitor/app/javascript.md). También es importante tener en cuenta que, puesto que va a analizar correlaciones, el tamaño del ejemplo importa.
>
>

## <a name="is-page-load-time-impacting-how-many-people-convert-on-my-page"></a>¿El tiempo de carga de página está afectando a cuánta gente se convierte en mi página?

Para empezar a responder a preguntas con la herramienta Impacto, elija una vista de página inicial, un evento personalizado o una solicitud.

![Herramienta Impacto](./media/usage-impact/0002-dropdown.png)

1. Seleccione una vista de página en la lista desplegable **For the page view** (Para la vista de página).
2. Deje la lista desplegable **analyze how its** (Analizar su) en la selección predeterminada de **Duración** (en este contexto **Duración** es un alias para **Tiempo de carga de la página**.)
3. Para la lista desplegable **afecta al uso de**, seleccione un evento personalizado. Este evento debe corresponder a un elemento de la interfaz de usuario en la vista de página que seleccionó en el paso 1.

![Captura de resultados](./media/usage-impact/0003-results.png)

En esta instancia, a medida que aumenta el tiempo de carga de **Página del producto** disminuye las veces que se hizo clic en **Adquirir producto**. Según la distribución anterior, puede conseguirse que una duración de carga de página óptima de 3,5 segundos alcance una tasa de conversión de un 55 %. Otras mejoras de rendimiento para reducir el tiempo de carga por debajo de 3,5 segundos no se correlacionan actualmente con ventajas de conversión adicionales.

## <a name="what-if-im-tracking-page-views-or-load-times-in-custom-ways"></a>¿Qué ocurre si sigo las vistas de página y los tiempos de carga de manera personalizada?

Impacto admite propiedades y medidas personalizadas y estándar, así que puede usar lo que prefiera. En lugar de duración, use filtros en los eventos principales y secundarios para obtener datos más específicos.

## <a name="do-users-from-different-countries-or-regions-convert-at-different-rates"></a>¿Las tasas de conversión de los usuarios varían en función de los países?

1. Seleccione una vista de página en la lista desplegable **For the page view** (Para la vista de página).
2. Elija "País o región" en la lista desplegable **analyze how its** (Analizar su).
3. En la lista desplegable **afecta el uso de**, seleccione un evento personalizado que corresponda a un elemento de la interfaz de usuario en la vista de página que eligió en el paso 1.

En este caso, los resultados ya no se ajustan en un modelo de eje x continuo como lo hacían en el primer ejemplo. En su lugar, se presenta una visualización similar a un embudo segmentado. Ordenar por **uso** para ver la variación de conversión para el evento personalizado según el país o región.


## <a name="how-does-the-impact-tool-calculate-these-conversion-rates"></a>¿Cómo calcula la herramienta Impacto la tasa de conversión?

En segundo plano, la herramienta Impacto se basa en el [coeficiente de correlación de Pearson](https://en.wikipedia.org/wiki/Pearson_correlation_coefficient). Los resultados se calculan entre -1 y 1, donde -1 representa una correlación lineal negativa y 1 representa una correlación lineal positiva.

El análisis básico de cómo funciona Análisis de impacto es el siguiente:

_A_ = la vista de página/evento personalizado/solicitud principal que seleccionó en la primera lista desplegable. (**Para la vista de página**).

_B_ = la vista de página/evento personalizado secundario que seleccionó (**afecta al uso de**).

Impacto examina una muestra de todas las sesiones de los usuarios en el intervalo de tiempo seleccionado. Para cada sesión, busca cada aparición de _A_.

A continuación las sesiones se dividen en dos tipos diferentes de _subsesiones_ basándose en una de estas dos condiciones:

- Una subsesión convertida se compone de una sesión que termina con un evento _B_ y abarca todos los eventos _A_ que se producen antes _B_.
- Se produce un subsesión no convertida cuando todos los eventos _A_ ocurren sin un terminal _B_.

La manera en la que se calcula el impacto varía en función de si estamos realizando el análisis por métricas o dimensiones. En el caso de las métricas, se calcula un promedio de todos los eventos _A_en una subsesión se promedian. Mientras que para las dimensiones, el valor de cada _A_ contribuye _1/N_ al valor asignado a _B_ donde _N_ es el número de eventos _A_en la subsesión.

## <a name="next-steps"></a>Pasos siguientes

- Para habilitar las experiencias de uso, empiece por enviar [eventos personalizados](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics#trackevent) o [vistas de páginas](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics#page-views).
- Si ya ha enviado eventos personalizados o vistas de página, explore las herramientas de uso para obtener información sobre cómo los usuarios utilizan el servicio.
    - [Embudos](usage-funnels.md)
    - [Retención](usage-retention.md)
    - [Flujos de usuario](usage-flows.md)
    - [Libros](../../azure-monitor/app/usage-workbooks.md)
    - [Adición de contexto de usuario](usage-send-user-context.md)
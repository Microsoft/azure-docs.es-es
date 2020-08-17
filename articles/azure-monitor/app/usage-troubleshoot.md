---
title: Solución de problemas de las herramientas de análisis de usuario - Azure Application Insights
description: 'Guía de solución de problemas: análisis de uso del sitio y las aplicaciones con Application Insights'
ms.topic: conceptual
author: NumberByColors
ms.author: daviste
ms.date: 07/11/2018
ms.reviewer: mbullwin
ms.openlocfilehash: 02d1bd9d204d88ba90218b1254c66ac0da80be85
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87323509"
---
# <a name="troubleshoot-user-behavior-analytics-tools-in-application-insights"></a>Solución de problemas de las herramientas de análisis de comportamiento de usuario en Application Insights
¿Tiene alguna pregunta sobre las [herramientas de análisis de comportamiento de usuario en Application Insights](usage-overview.md): [usuarios, sesiones, eventos](usage-segmentation.md), [embudos](usage-funnels.md), [flujos de usuario](usage-flows.md), [retención](usage-retention.md) o cohortes? Estas son algunas respuestas.

## <a name="counting-users"></a>Recuento de usuarios
**Las herramientas de análisis de comportamiento de usuario muestran que mi aplicación tiene una sesión y un usuario, pero sé que tiene varios usuarios y sesiones. ¿Cómo se solucionan estos recuentos incorrectos?**

Todos los eventos de telemetría de Application Insights tienen un [identificador de usuario anónimo](./data-model-context.md) y un [identificador de sesión](./data-model-context.md) como parte de las propiedades estándares. De forma predeterminada, todas las herramientas de análisis de uso de recuento de usuarios y sesiones se basan en estos identificadores. Si estas propiedades estándares no se rellenan con identificadores únicos para cada usuario y sesión de la aplicación, verá un recuento incorrecto de usuarios y sesiones en las herramientas de análisis de uso.

Si supervisa una aplicación web, la solución más sencilla consiste en agregar el [SDK de JavaScript de Application Insights](./javascript.md) a la aplicación y asegurarse de que el fragmento de código de script se carga en todas las páginas que desee supervisar. El SDK de JavaScript genera identificadores de usuario y sesión automáticamente y rellena los eventos de telemetría con ellos según se van enviando desde la aplicación.

Si supervisa un servicio web (sin interfaz de usuario), [cree un inicializador de telemetría que rellene las propiedades de los identificadores de usuario anónimo y sesión](usage-send-user-context.md) según las nociones de su servicio sobre los usuarios únicos y las sesiones.

Si la aplicación envía [identificadores de usuarios autenticados](./api-custom-events-metrics.md#authenticated-users), con la herramienta Usuarios puede realizar el recuento en función de estos. En la lista desplegable "Mostrar", seleccione "Usuarios autenticados".

Actualmente, las herramientas de análisis de comportamiento de usuario no admiten el recuento de usuarios o sesiones en función de propiedades distintas del identificador de usuario anónimo, el identificador de usuario autenticado o el identificador de sesión.

## <a name="naming-events"></a>Nomenclatura de los eventos
**Mi aplicación tiene miles de vistas de página y nombres de eventos personalizados distintos. Resulta difícil distinguirlos y las herramientas de análisis de comportamiento de usuario dejan de responder con frecuencia. ¿Cómo se solucionan estos problemas de nomenclatura?**

Las herramientas de análisis de comportamiento de usuario utilizan las vistas de página y los nombres de los eventos personalizados. La correcta nomenclatura de los eventos es fundamental para sacarles partido a estas herramientas. El objetivo es un equilibrio entre un número reducido de nombres demasiado genéricos ("Se hizo clic en el botón") y un número excesivo de nombres demasiado específicos ("Se hizo clic en el botón Editar de http:\//www.contoso.com/index").

Para realizar cambios en las vistas de página y los nombres de evento personalizados que envía la aplicación, debe cambiar el código fuente de la aplicación y volver a realizar la implementación. **Todos los datos de telemetría de Application Insights se almacenan durante 90 días y no se pueden eliminar**, por lo que los cambios realizados en los nombres de evento tardarán 90 días en manifestarse totalmente. Durante los 90 días después de los cambios en el nombre, en la telemetría aparecen tanto los nuevos nombres como los antiguos, por lo que es necesario ajustar las consultas y comunicarse con los equipos en consecuencia.

Si la aplicación envía demasiados nombres de vista de página, compruebe si estos se especifican manualmente en el código o si los envía automáticamente el SDK de JavaScript de Application Insights:

* Si se especifican manualmente en el código mediante [`trackPageView` API](https://github.com/Microsoft/ApplicationInsights-JS/blob/master/API-reference.md), cambie el nombre a uno menos específico. Evite los errores comunes, como colocar la dirección URL en el nombre de la vista de página. En su lugar, use el parámetro de dirección URL de `trackPageView` API. Mueva los demás detalles del nombre de la vista de página a propiedades personalizadas.

* Si el SDK de JavaScript de Application Insights envía los nombres de vista de página automáticamente, puede cambiar los títulos de las páginas o enviar los nombres de vista de página manualmente. De forma predeterminada, el SDK envía el [título](https://developer.mozilla.org/docs/Web/HTML/Element/title) de cada página como nombre de la vista de página. Puede cambiar los títulos a otros más generales, pero tenga en cuenta la optimización del motor de búsqueda y otros efectos que este cambio podría tener. Especificar manualmente los nombres de vista de página con `trackPageView` API invalida los nombres recopilados automáticamente, por lo que podría enviar nombres más generales a la telemetría sin cambiar los títulos de página.   

Si la aplicación envía demasiados nombres de evento personalizados, cambie los nombres en el código a otros menos específicos. Una vez más, evite colocar direcciones URL y otros datos de página o dinámicos directamente en los nombres de evento personalizados. En su lugar, mueva estos detalles a las propiedades personalizadas del evento personalizado con `trackEvent` API. Por ejemplo, en lugar de `appInsights.trackEvent("Edit button clicked on http://www.contoso.com/index")`, sugerimos algo parecido a `appInsights.trackEvent("Edit button clicked", { "Source URL": "http://www.contoso.com/index" })`.

## <a name="next-steps"></a>Pasos siguientes

* [Introducción a las herramientas de análisis de comportamiento de usuario](usage-overview.md)

## <a name="get-help"></a>Obtener ayuda
* [Stack Overflow](https://stackoverflow.com/questions/tagged/ms-application-insights)


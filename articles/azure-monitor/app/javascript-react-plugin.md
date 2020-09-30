---
title: Complemento React para el SDK de JavaScript de Application Insights
description: Instalación y uso del complemento React para el SDK de JavaScript de Application Insights
services: azure-monitor
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 07/28/2020
ms.openlocfilehash: 3a11f77384c520bed9824841269be4ad998adba4
ms.sourcegitcommit: 814778c54b59169c5899199aeaa59158ab67cf44
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/13/2020
ms.locfileid: "90056207"
---
# <a name="react-plugin-for-application-insights-javascript-sdk"></a>Complemento React para el SDK de JavaScript de Application Insights

El complemento React para el SDK de JavaScript de Application Insights habilita lo siguiente:

- Seguimiento de cambios de ruta
- Estadísticas de uso de componentes de React

## <a name="getting-started"></a>Introducción

Instalación del paquete de npm:

```bash

npm install @microsoft/applicationinsights-react-js

```

## <a name="basic-usage"></a>Uso básico

```javascript
import React from 'react';
import { ApplicationInsights } from '@microsoft/applicationinsights-web';
import { ReactPlugin, withAITracking } from '@microsoft/applicationinsights-react-js';
import { createBrowserHistory } from "history";

const browserHistory = createBrowserHistory({ basename: '' });
var reactPlugin = new ReactPlugin();
var appInsights = new ApplicationInsights({
    config: {
        instrumentationKey: 'YOUR_INSTRUMENTATION_KEY_GOES_HERE',
        extensions: [reactPlugin],
        extensionConfig: {
          [reactPlugin.identifier]: { history: browserHistory }
        }
    }
});
appInsights.loadAppInsights();

// To instrument various React components usage tracking, apply the `withAITracking` higher-order
// component function.

class MyComponent extends React.Component {
    ...
}

export default withAITracking(reactPlugin,appInsights, MyComponent);

```

## <a name="configuration"></a>Configuración

| Nombre    | Valor predeterminado | Descripción                                                                                                    |
|---------|---------|----------------------------------------------------------------------------------------------------------------|
| history | null    | Historial de react-router. Para obtener más información, consulte la [documentación del paquete de react-router](https://reactrouter.com/web/api/history). Para obtener información sobre cómo tener acceso al objeto de historial fuera de los componentes, consulte las [preguntas frecuentes de react-router](https://github.com/ReactTraining/react-router/blob/master/FAQ.md#how-do-i-access-the-history-object-outside-of-components).    |

### <a name="react-components-usage-tracking"></a>Seguimiento de uso de componentes de React

Para instrumentar el seguimiento del uso de varios componentes de React, aplique la función de componente de orden superior `withAITracking`.

Medirá el tiempo desde el evento de `ComponentDidMount` a través del evento `ComponentWillUnmount`. Sin embargo, para que esto sea más preciso, restará el tiempo en el que el usuario estuvo inactivo `React Component Engaged Time = ComponentWillUnmount timestamp - ComponentDidMount timestamp - idle time`.

Para ver esta métrica en Azure Portal, tiene que ir al recurso de Application Insights, seleccionar la pestaña "Métricas" y configurar los gráficos vacíos para mostrar el nombre de la métrica personalizada "React Component Engaged Time (seconds)" (Tiempo de interactividad del componente de React [segundos]), seleccionar la agregación (suma, promedio, etc.) de la métrica y aplicar la división "Component Name" (Nombre del componente).

![Captura de pantalla del gráfico que muestra la métrica personalizada"React Component Engaged Time (seconds)" (Tiempo de interactividad del componente de React [segundos]) dividida por "Component Name" (Nombre del componente).](./media/javascript-react-plugin/chart.png)

También puede ejecutar consultas personalizadas para dividir los datos de Application Insights con el fin de generar informes y visualizaciones según sus requisitos. En Azure Portal navegue hasta el recurso de Application Insights, seleccione "Análisis" en el menú superior de la pestaña de información general y ejecute la consulta.

![Captura de pantalla de los resultados de la consulta de métricas personalizadas.](./media/javascript-react-plugin/query.png)

> [!NOTE]
> Las nuevas métricas personalizadas pueden tardar hasta 10 minutos en aparecer en Azure Portal.

## <a name="sample-app"></a>Aplicación de ejemplo

Consulte la [demostración de React en Application Insights](https://github.com/Azure-Samples/application-insights-react-demo).

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre el SDK de JavaScript, consulte la [documentación del SDK de JavaScript de Application Insights](javascript.md).
- Para obtener información sobre el lenguaje de consulta de Kusto y la consulta de datos en Log Analytics, consulte la [información general sobre la consulta de registro](../../azure-monitor/log-query/log-query-overview.md).

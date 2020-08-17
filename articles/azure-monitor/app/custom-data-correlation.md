---
title: Azure Application Insights | Microsoft Docs
description: Correlacione los datos de Application Insights con otros conjuntos de datos, como el enriquecimiento de datos o las tablas de búsqueda, los orígenes de datos que no son de Application Insights y los datos personalizados.
ms.topic: conceptual
author: eternovsky
ms.author: evternov
ms.date: 08/08/2018
ms.reviewer: mbullwin
ms.openlocfilehash: 17de697686485d695586ffa798196efb4a34c251
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87092922"
---
# <a name="correlating-application-insights-data-with-custom-data-sources"></a>Correlación de los datos de Application Insights con orígenes de datos personalizados

Application Insights recopila diferentes tipos de datos: excepciones, seguimientos, vistas de página y otros. Si bien esto suele ser suficiente para investigar el rendimiento, la confiabilidad y el uso de la aplicación, hay casos en los que resulta útil correlacionar los datos almacenados en Application Insights con otros conjuntos de datos completamente personalizados.

Algunas situaciones en las que es posible que desee usar datos personalizados incluyen:

- Enriquecimiento de datos o tablas de búsqueda: complementar, por ejemplo, un nombre de servidor con el propietario del servidor y la ubicación del laboratorio en el que se puede encontrar. 
- Correlación con orígenes de datos que no son de Application Insights: por ejemplo, correlacionar datos sobre una compra en una tienda web con la información del servicio de entrega de compras para determinar el grado de precisión de las estimaciones de la hora de envío. 
- Datos completamente personalizados: a muchos de nuestros clientes les encantan el lenguaje de consulta y el rendimiento de la plataforma de registro de Azure Monitor que hay detrás de Application Insights, y quieren aprovecharlos para consultar datos que no están relacionados con Application Insights. Por ejemplo, para supervisar el rendimiento de un panel solar como parte de una instalación doméstica inteligente como se describe [aquí](https://www.catapultsystems.com/blogs/using-log-analytics-and-a-special-guest-to-forecast-electricity-generation/).

## <a name="how-to-correlate-custom-data-with-application-insights-data"></a>Correlación de datos personalizados con datos de Application Insights 

Puesto que Application Insights está respaldado por la eficaz plataforma de registro de Azure Monitor, se puede usar toda la potencia de Azure Monitor para la ingesta de datos. De este modo, se escriben consultas con el operador "join" que establecen una correlación de estos datos personalizados con los datos disponibles en los registros de Azure Monitor. 

## <a name="ingesting-data"></a>Ingesta de datos

En esta sección, revisamos cómo ingerir los datos en registros de Azure Monitor.

Si aún no tiene una, aprovisione una nueva área de trabajo de Log Analytics según [estas instrucciones](../learn/quick-collect-azurevm.md), incluido el paso "Creación de un área de trabajo".

Para empezar a enviar datos de registro a Azure Monitor. Existen varias opciones:

- Para un mecanismo sincrónico, puede llamar directamente a [Data Collector API](../platform/data-collector-api.md) o usar el conector de Logic Apps; simplemente busque "Azure Log Analytics" y elija la opción "Enviar datos":

  ![Captura de pantalla de la selección y la acción](./media/custom-data-correlation/01-logic-app-connector.png)  

- Para una opción asincrónica, use Data Collector API para crear una canalización de procesamiento. Consulte [este artículo](../platform/create-pipeline-datacollector-api.md) para más información.

## <a name="correlating-data"></a>Correlación de datos

Application Insights se basa en la plataforma de registro de Azure Monitor. Por lo tanto, podemos usar [combinaciones entre recursos](../log-query/cross-workspace-query.md) para correlacionar cualquier dato que se ingiera en Azure Monitor con los datos de Application Insights.

Por ejemplo, podemos ingerir el inventario y las ubicaciones del laboratorio en una tabla llamada "LabLocations_CL" en un área de trabajo de Log Analytics llamada "myLA". A continuación, si quisiéramos revisar las solicitudes de la aplicación de Application Insights llamada "myAI" y correlacionar los nombres de equipo que atienden las solicitudes con las ubicaciones de estos equipos almacenados en la tabla personalizada mencionada anteriormente, podemos ejecutar la siguiente consulta en el contexto de Application Insights o de Azure Monitor:

```
app('myAI').requests
| join kind= leftouter (
    workspace('myLA').LabLocations_CL
    | project Computer_S, Owner_S, Lab_S
) on $left.cloud_RoleInstance == $right.Computer
```

## <a name="next-steps"></a>Pasos siguientes

- Consulte la referencia de [Data Collector API](../platform/data-collector-api.md).
- Más información sobre [combinaciones entre recursos](../log-query/cross-workspace-query.md).

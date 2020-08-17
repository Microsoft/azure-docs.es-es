---
title: Modelo de datos de telemetría de Azure Application Insights | Microsoft Docs
description: Información general del modelo de datos de Application Insights
services: application-insights
documentationcenter: .net
manager: carmonm
ms.workload: TBD
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 10/14/2019
ms.reviewer: sergkanz
ms.openlocfilehash: 69ed934e97d2726995a7a5cd122fadbd4a791942
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87320534"
---
# <a name="application-insights-telemetry-data-model"></a>Modelo de datos de telemetría de Application Insights

[Azure Application Insights](./app-insights-overview.md) envía telemetría de la aplicación web en Azure Portal, por lo que puede analizar el rendimiento y el uso de la aplicación. El modelo de telemetría está normalizado para que sea posible crear una supervisión de independiente del lenguaje y de la plataforma. 

Los datos recopilados por Application Insights modelan este patrón de ejecución de aplicaciones típico:

![Modelo de aplicación de Application Insights](./media/data-model/application-insights-data-model.png)

Los siguientes tipos de telemetría se utilizan para supervisar la ejecución de la aplicación. Los tres tipos siguientes normalmente se recopilan automáticamente mediante el SDK de Application Insights desde la plataforma de aplicación web:

* [**Solicitud**](data-model-request-telemetry.md): generada para registrar una solicitud recibida por la aplicación. Por ejemplo, el SDK web de Application Insights genera automáticamente un elemento de telemetría de solicitud para cada solicitud HTTP que recibe la aplicación web. 

    Una **operación** es el subproceso de ejecución que procesa una solicitud. También puede [escribir código](./api-custom-events-metrics.md#trackrequest) para supervisar otros tipos de operación, como un "reactivo" de un trabajo web de trabajo o una función que periódicamente procesa los datos.  Cada operación tiene un identificador. Este identificador puede usarse para [agrupar](./correlation.md) toda la telemetría generada mientras la aplicación procesa la solicitud. Cada operación se realiza correctamente o produce un error y tiene una duración de tiempo.
* [**Excepción**](data-model-exception-telemetry.md): normalmente representa una excepción que produce un error en una operación.
* [**Dependencia**](data-model-dependency-telemetry.md): representa una llamada de la aplicación a un servicio o almacenamiento externo como una API de REST o SQL. En ASP.NET las llamadas de dependencia a SQL se definen mediante `System.Data`. Las llamadas a los puntos de conexión HTTP se definen mediante `System.Net`. 

Application Insights proporciona tres tipos de datos adicionales para telemetría personalizada:

* [Seguimiento](data-model-trace-telemetry.md): se usa directamente o a través de un adaptador que implementa el registro de diagnóstico mediante una plataforma de instrumentación con la que se está familiarizado, como `Log4Net` o `System.Diagnostics`.
* [Evento](data-model-event-telemetry.md): se usa habitualmente para capturar la interacción del usuario con el servicio para analizar los patrones de uso.
* [Métrica](data-model-metric-telemetry.md): se usa para notificar las medidas escalares periódicas.

Cada elemento de telemetría puede definir la [información de contexto](data-model-context.md) en forma de versión de aplicación o de identificador de sesión de usuario. El contexto es un conjunto de campos fuertemente tipados que desbloquean ciertos escenarios. Cuando la versión de la aplicación se haya inicializado correctamente, Application Insights puede detectar nuevos patrones de comportamiento de la aplicación correlacionados con la reimplementación. El identificador de sesión puede usarse para calcular el tiempo de interrupción o el impacto de un problema en los usuarios. Calcular recuentos distintivos de valores de identificador de sesión para ciertos errores de dependencia, seguimientos de errores o excepciones críticas proporciona información valiosa sobre impactos.

El modelo de telemetría de Application Insights define una forma de [correlacionar](./correlation.md) la telemetría con la operación de la que es una parte. Por ejemplo, una solicitud puede realizar llamadas de SQL Database y registrar información de diagnóstico. Puede establecer el contexto de correlación de los elementos de telemetría que lo vinculan a la telemetría de solicitudes.

## <a name="schema-improvements"></a>Mejoras de esquema

El modelo de datos de Application Insights es una manera sencilla y básica pero eficaz de modelar la telemetría de la aplicación. Nos esforzamos para que el modelo sea simple y compacto para admitir escenarios esenciales y permitir extender el esquema para el uso avanzado.

Para informar de problemas del esquema o del modelo de datos y proporcionar sugerencias, use el repositorio [ApplicationInsights-Home](https://github.com/Microsoft/ApplicationInsights-Home/issues) de GitHub.

## <a name="next-steps"></a>Pasos siguientes

- [Escritura de telemetría personalizada](./api-custom-events-metrics.md)
- Obtenga información sobre cómo [ampliar y filtrar la telemetría](./api-filtering-sampling.md).
- Use el [muestreo](./sampling.md) para minimizar la cantidad de telemetría en función del modelo de datos.
- Consulte las [plataformas](./platforms.md) compatibles con Application Insights.


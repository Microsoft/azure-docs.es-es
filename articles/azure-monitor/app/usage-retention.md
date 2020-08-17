---
title: Análisis de la retención de usuarios de aplicaciones web con Azure Application Insights
description: ¿Cuántos usuarios regresan a la aplicación?
ms.topic: conceptual
author: NumberByColors
ms.author: daviste
ms.date: 05/03/2017
ms.reviewer: mbullwin
ms.openlocfilehash: f7f6155d3230d234e1163ee12cd7685817331b93
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87323543"
---
# <a name="user-retention-analysis-for-web-applications-with-application-insights"></a>Análisis de retención de usuarios para aplicaciones web con Application Insights

La característica Retención de [Azure Application Insights](./app-insights-overview.md) le ayuda a analizar cuántos usuarios regresan a la aplicación, y con qué frecuencia realizan determinadas tareas o logran objetivos. Por ejemplo, si ejecuta un sitio de juegos, puede comparar el número de usuarios que regresan al sitio después de perder una partida con la cantidad que vuelven tras ganar. Esta información puede ayudarlo a mejorar la experiencia de los usuarios y la estrategia empresarial.

## <a name="get-started"></a>Introducción

Si aún no ve los datos en la herramienta Retención del portal de Application Insights, [obtenga información sobre cómo empezar a trabajar con las herramientas de uso](usage-overview.md).

## <a name="the-retention-tool"></a>La herramienta Retención

![Herramienta Retención](./media/usage-retention/retention.png)

1. La barra de herramientas permite a los usuarios crear nuevos informes de retención, abrir los informes ya existentes, guardar el informe de retención actual (o guardarlo mediante la opción Guardar como), revertir los cambios efectuados en los informes guardados, actualizar los datos del informe, compartir un informe por correo electrónico o vínculo directo y acceder a la página de documentación. 
2. De forma predeterminada, la herramienta de retención muestra todos los usuarios que hicieron algo y que , posteriormente, regresaron e hicieron algo más a lo largo de un período. Puede seleccionar una combinación diferente de eventos para centrar el enfoque en las actividades específicas del usuario.
3. Agregue uno o varios filtros en las propiedades. Por ejemplo, puede centrarse en los usuarios de un país o una región determinados. Haga clic en **Actualizar** después de establecer los filtros. 
4. El gráfico de retención general muestra un resumen de la retención del usuario durante un período de tiempo seleccionado. 
5. La cuadrícula muestra el número de usuarios retenidos según el generador de consultas en 2. Cada fila representa una cohorte de usuarios que llevó a cabo cualquier evento en el periodo mostrado. Cada celda de la fila muestra cuántos de esa cohorte regresaron al menos una vez en un periodo posterior. Algunos usuarios pueden regresar en más de un periodo. 
6. Las tarjetas de información muestran los cinco eventos de inicio más importantes y los cinco eventos devueltos principales para proporcionar a los usuarios una mejor comprensión de su informe de retención. 

![Mantener el puntero sobre la retención](./media/usage-retention/hover.png)

Los usuarios pueden mantener el puntero sobre las celdas de la herramienta Retención para acceder al botón de análisis y a la información que explica lo que significa cada celda. El botón Análisis conduce al usuario a la herramienta Análisis con una consulta rellenada previamente para generar usuarios desde la celda. 

## <a name="use-business-events-to-track-retention"></a>Uso de eventos empresariales para realizar un seguimiento de la retención

Para obtener el análisis de retención más útil, mida los eventos que representan actividades empresariales importantes. 

Por ejemplo, muchos usuarios podrían abrir una página de la aplicación sin jugar al juego que se muestra. Al realizar solo el seguimiento de las vistas de página, se proporcionaría una estimación inexacta de cuántas personas vuelven a reproducir el juego después de haberlo jugado anteriormente. Para obtener una visión clara de los jugadores que regresan, la aplicación debe enviar un evento personalizado cuando un usuario juega realmente.  

Se recomienda codificar los eventos personalizados que representan las acciones clave del negocio y usarlos para realizar el análisis de retención. Para obtener el resultado del juego, tiene que escribir una línea de código con el fin enviar un evento personalizado a Application Insights. Si se escribe en el código de la página web o en Node.JS, se vería similar a esto:

```JavaScript
    appinsights.trackEvent("won game");
```

O en el código del servidor ASP.NET:

```csharp
   telemetry.TrackEvent("won game");
```

[Obtenga más información sobre la creación de eventos personalizados](./api-custom-events-metrics.md#trackevent).


## <a name="next-steps"></a>Pasos siguientes
- Para habilitar las experiencias de uso, empiece por enviar [eventos personalizados](./api-custom-events-metrics.md#trackevent) o [vistas de páginas](./api-custom-events-metrics.md#page-views).
- Si ya ha enviado eventos personalizados o vistas de página, explore las herramientas de uso para obtener información sobre cómo los usuarios utilizan el servicio.
    - [Usuarios, sesiones, eventos](usage-segmentation.md)
    - [Embudos](usage-funnels.md)
    - [Flujos de usuario](usage-flows.md)
    - [Libros](../platform/workbooks-overview.md)
    - [Adición de contexto de usuario](usage-send-user-context.md)


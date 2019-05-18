---
title: Introducción a los diagnósticos de Azure App Service | Microsoft Docs
description: Obtenga información sobre cómo solucionar problemas con la aplicación con diagnósticos de App Service.
keywords: app service, azure app service, diagnósticos, compatibilidad, aplicación web, solución de problemas, autoayuda
services: app-service
documentationcenter: ''
author: jen7714
manager: cfowler
editor: ''
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/10/2017
ms.author: jennile
ms.custom: seodec18
ms.openlocfilehash: 3e304df51133d53adad50e672249bde6c9960712
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65539742"
---
# <a name="azure-app-service-diagnostics-overview"></a>Introducción a los diagnósticos de Azure App Service

Cuando ejecute una aplicación web, querrá estar preparado para los posibles problemas que puedan surgir, desde errores 500 hasta que sus usuarios le comuniquen que su sitio está fuera de servicio. Diagnósticos de App Service es una experiencia inteligente e interactiva para ayudarle a solucionar problemas de la aplicación sin necesidad de configuración. Cuando surgen problemas con la aplicación, diagnósticos de App Service señala cuál es el problema que le guiarán a la información correcta para con más facilidad y rapidez localizar y solucionar el problema.

Aunque esta experiencia es principalmente útil si tiene problemas con la aplicación en las últimas 24 horas, todos los gráficos de diagnóstico están siempre disponibles para su análisis.

El diagnóstico de App Service funciona no solo para la aplicación en Windows, sino también para las aplicaciones en [Linux/contenedores](https://docs.microsoft.com/azure/app-service/containers/app-service-linux-intro), el [entono de App Service](https://docs.microsoft.com/azure/app-service/environment/intro) y [Azure functions](https://docs.microsoft.com/azure/azure-functions/functions-overview).

## <a name="open-app-service-diagnostics"></a>Apertura de los diagnósticos de App Service

Para obtener acceso a diagnósticos de App Service, vaya a la aplicación web de app Service o App Service Environment en la [portal Azure](https://portal.azure.com). En el panel izquierdo, haga clic en **Diagnosticar y solucionar problemas**.

Para las funciones de Azure, vaya a la aplicación de función y, en el panel de navegación superior, haga clic en **características de la plataforma**y seleccione **diagnosticar y resolver problemas** desde el **deadministraciónderecursos** sección.

En la página de inicio de diagnóstico de App Service, puede elegir la categoría que mejor describa el problema con la aplicación mediante el uso de las palabras clave en cada icono página principal. Además, esta página es donde puede encontrar **herramientas de diagnóstico** para aplicaciones de Windows. Consulte [herramientas de diagnóstico (solo para la aplicación de Windows)](#diagnostic-tools-only-for-windows-app).

![Página de inicio](./media/app-service-diagnostics/app-service-diagnostics-homepage-1.png)

## <a name="interactive-interface"></a>Interfaz interactiva

Una vez que seleccione una categoría de la página principal que mejor se alinee con el problema de la aplicación, la interfaz interactiva de diagnósticos de App Service, genio, puede guiarle a través de diagnosticar y resolver el problema con la aplicación. Puede usar los métodos abreviados de mosaico proporcionados por el genio para ver el informe de diagnóstico completo de la categoría de problema que está interesado. Los accesos directos de iconos proporcionan una manera directa de obtener acceso a las métricas de diagnóstico.

![Accesos directos de iconos](./media/app-service-diagnostics/tile-shortcuts-2.png)

Después de hacer clic en estos iconos, puede ver una lista de temas relacionados con el problema descrito en el icono. Estos temas proporcionan fragmentos de información importante desde el informe completo. Puede hacer clic en cualquiera de estos temas para investigar los problemas aún más. Además, puede hacer clic en **ver el informe completo** para explorar todos los temas en una sola página.

![Temas](./media/app-service-diagnostics/application-logs-insights-3.png)

![Ver el informe completo](./media/app-service-diagnostics/view-full-report-4.png)

## <a name="diagnostic-report"></a>Informe de diagnóstico

Después de elegir investigar aún más el problema haciendo clic en un tema, puede ver más detalles sobre el tema que a menudo se complementa con gráficos y descuentos. Informe de diagnóstico puede ser una herramienta eficaz para identificar el problema con la aplicación.

![Informe de diagnóstico](./media/app-service-diagnostics/full-diagnostic-report-5.png)

## <a name="health-checkup"></a>Comprobación de mantenimiento

Si no sabe cuál es el problema con la aplicación o no sabe por dónde empezar a solucionar los problemas, la comprobación de mantenimiento es un buen lugar para comenzar. La comprobación de mantenimiento analiza sus aplicaciones para ofrecerle una breve introducción interactiva que destaca lo que es correcto y cuál es el problema, que le indica dónde mirar para investigar el problema. Su interfaz inteligente e interactiva le proporciona una guía por el proceso de solución de problemas. Comprobación de mantenimiento se integra con la experiencia de genio para aplicaciones de Windows y web app el detalle de diagnóstico informes para las aplicaciones de Linux.

### <a name="health-checkup-graphs"></a>Gráficos de resultados de la comprobación de mantenimiento

Hay cuatro gráficos diferentes en la comprobación de mantenimiento.

- **solicitudes y errores:** Un gráfico que muestra el número de solicitudes realizadas durante las últimas 24 horas, junto con los errores del servidor HTTP.
- **rendimiento de la aplicación:** Un gráfico que muestra el tiempo de respuesta a lo largo de las últimas 24 horas para diversos grupos de percentil.
- **Uso de CPU:** Un gráfico que muestra el porcentaje general uso de CPU por cada instancia a lo largo de las últimas 24 horas.  
- **uso de memoria:** Un gráfico que muestra el uso de memoria física total en porcentaje por cada instancia a lo largo de las últimas 24 horas.

![Comprobación de mantenimiento](./media/app-service-diagnostics/health-checkup-6.png)

### <a name="investigate-application-code-issues-only-for-windows-app"></a>Investigar los problemas de código de aplicación (solo para la aplicación de Windows)

Dado que muchos problemas de la aplicación están relacionados con el código de esta, el diagnóstico de App Service se integra con [Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview) para resaltar las excepciones y los problemas de dependencia para ponerlos en correlación con el tiempo de inactividad seleccionado. Application Insights tiene que estar habilitado por separado.

![Application Insights](./media/app-service-diagnostics/application-insights-7.png)

Para ver las dependencias y excepciones de Application Insights, seleccione el **aplicación web hacia abajo** o **aplicación web lenta** accesos directos de iconos.

### <a name="troubleshooting-steps-only-for-windows-app"></a>Pasos para solucionar problemas (solo para la aplicación de Windows)

Si se detecta un problema con una categoría determinada en las últimas 24 horas, puede ver el informe de diagnóstico completo y diagnósticos de App Service pueden pedirle que vea más consejos de solución de los pasos siguientes para una experiencia más interactiva.

![Application Insights y solución de problemas y próximos pasos](./media/app-service-diagnostics/troubleshooting-and-next-steps-8.png)

## <a name="diagnostic-tools-only-for-windows-app"></a>Herramientas de diagnóstico (solo para la aplicación de Windows)

Herramientas de diagnóstico incluyen herramientas de diagnóstico más avanzadas que ayuda al investigar aplicación código problemas, lentitud, las cadenas de conexión y mucho más. y herramientas proactivas que le ayudan a mitigar los problemas con el uso de CPU, las solicitudes y memoria.

### <a name="proactive-cpu-monitoring"></a>Supervisión proactiva de la CPU

Supervisión proactiva de la CPU proporciona una manera fácil y proactiva para realizar una acción cuando el proceso de aplicación o elemento secundario de la aplicación está consumiendo recursos de CPU elevados. Puede establecer sus propias reglas de umbral de CPU para mitigar temporalmente un problema de CPU elevado hasta que se encuentra la causa real del problema inesperado.

![Supervisión proactiva de la CPU](./media/app-service-diagnostics/proactive-cpu-monitoring-9.png)

### <a name="proactive-auto-healing"></a>Proactiva la recuperación automática

Al igual que la supervisión proactiva de la CPU, automático se recupere por ofrece un enfoque proactivo fácil de mitigar un comportamiento inesperado de la aplicación. Puede establecer sus propias reglas basadas en código de estado HTTP para desencadenar acciones de mitigación, una solicitud lenta, límite de memoria y número de solicitudes. Esta herramienta puede utilizarse para mitigar temporalmente un comportamiento inesperado hasta encontrar la causa del problema real. Para obtener más información sobre proactiva la recuperación automática, visite [anuncio automática nueva experiencia de diagnósticos de app service de recuperación](https://azure.github.io/AppService/2018/09/10/Announcing-the-New-Auto-Healing-Experience-in-App-Service-Diagnostics.html).

![Proactiva la recuperación automática](./media/app-service-diagnostics/proactive-auto-healing-10.png)

## <a name="change-analysis"></a>Análisis de cambios

En un entorno de desarrollo rápido, a veces puede ser difícil realizar un seguimiento de todos los cambios realizados a la aplicación y dejar que pinpoint por sí solo en un cambio que produjo un comportamiento incorrecto. Análisis de cambios pueden ayudar a reducir en los cambios realizados en la aplicación para facilitar la experiencia de solución de problemas. Análisis de cambio está incrustado en informe de diagnóstico como **aplicación se bloquea** para que pueda usar simultáneamente con otras métricas.

![Página predeterminada de análisis de cambio](./media/app-service-diagnostics/change-analysis-default-page-11.png)

![Vista de diferencias](./media/app-service-diagnostics/diff-view-12.png)

Análisis de cambios debe habilitarse antes de usar la característica. Para obtener más información sobre análisis de cambios, visite [anuncio de la nueva experiencia de análisis de cambios en diagnósticos de App Service](https://azure.github.io/AppService/2019/05/07/Announcing-the-new-change-analysis-experience-in-App-Service-Diagnostics-Analysis.html).
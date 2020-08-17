---
title: 'Detección inteligente: anomalías de rendimiento de Smart | Microsoft Docs'
description: Application Insights realiza un análisis inteligente de la telemetría de su aplicación y le advierte de los posibles problemas. Esta característica no necesita ninguna configuración.
ms.topic: conceptual
ms.date: 05/04/2017
ms.reviewer: antonfr
ms.openlocfilehash: c817e2c7f609bbbec52eff1b898a8d7c53209a28
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87321316"
---
# <a name="smart-detection---performance-anomalies"></a>Detección inteligente: anomalías de rendimiento

[Application Insights](./app-insights-overview.md) analiza automáticamente el rendimiento de su aplicación web y puede advertirle de los posibles problemas. Podría estar leyendo este artículo porque ha recibido una de nuestras notificaciones de detección inteligente.

Esta característica no requiere ninguna configuración especial, tan solo ajustar los parámetros de la aplicación para Application Insights para el [idioma admitido](./platforms.md). Se activará cuando la aplicación genera suficientes datos de telemetría.

## <a name="when-would-i-get-a-smart-detection-notification"></a>¿Cuándo recibiría una notificación de detección inteligente?

Application Insights ha detectado que el rendimiento de la aplicación se ha degradado en una de estas maneras:

* **Degradación del tiempo de respuesta**: la aplicación ha comenzado a responder a las solicitudes de una forma más lenta de lo habitual. El cambio podría haber sido rápido, por ejemplo, porque hubo una regresión en la implementación más reciente. También podría haber sido gradual, probablemente debido a una fuga de memoria. 
* **Degradación de la duración de la dependencia**: la aplicación realiza llamadas a una API de REST, una base de datos u otra dependencia. La dependencia responde de una forma más lenta de lo habitual.
* **Patrón de rendimiento lento**: la aplicación parece tener un problema de rendimiento que está afectando solo a algunas solicitudes. Por ejemplo, las páginas se cargan mucho más lentamente en un tipo de explorador que en otros, o las solicitudes se atienden de una forma más lenta desde un servidor concreto. En la actualidad, nuestros algoritmos examinan tiempos de carga de página, tiempos de respuesta de solicitud y tiempos de respuesta de dependencia.  

Detección inteligente requiere, al menos, 8 días de datos de telemetría en un volumen de trabajo con el fin de establecer una línea de base de rendimiento normal. Por lo tanto, después de que la aplicación se haya estado ejecutando durante ese periodo, se generará una notificación cuando se produzca un problema significativo.


## <a name="does-my-app-definitely-have-a-problem"></a>Entonces, ¿mi aplicación tiene un problema?

No, una notificación no significa que la aplicación tenga un problema. Simplemente es una sugerencia sobre algo que puede que desee examinar con más detenimiento.

## <a name="how-do-i-fix-it"></a>¿Cómo puedo corregirlo?

Las notificaciones incluyen información de diagnóstico. Este es un ejemplo:


![Este es un ejemplo de detección de degradación del tiempo de respuesta del servidor](media/proactive-performance-diagnostics/server_response_time_degradation.png)

1. **Evaluación de errores**. La notificación muestra el número de usuarios u operaciones afectado. Esto puede ayudarlo a asignar una prioridad al problema.
2. **Ámbito**. ¿Está el problema afectando a todo el tráfico o solo a algunas páginas? ¿Está limitado a ubicaciones o exploradores determinados? Esta información puede obtenerse de la notificación.
3. **Diagnóstico**. A menudo, la información de diagnóstico de la notificación sugiere cuál es la naturaleza del problema. Por ejemplo, si el tiempo de respuesta se ralentiza cuando la velocidad de solicitudes es alta, el problema podría estar en que su servidor o dependencias están sobrecargadas. 

    Si no sugiere la causa, abra la hoja Rendimiento de Application Insights. Allí encontrará los datos de [Profiler](profiler.md). Si se producen excepciones, también puede probar el [Depurador de instantáneas](./snapshot-debugger.md).



## <a name="configure-email-notifications"></a>Configuración de notificaciones de correo electrónico

Las notificaciones de Detección inteligente se habilitan de forma predeterminada y se envían a aquellos que tienen el acceso [Lector de supervisión](../../role-based-access-control/built-in-roles.md#monitoring-reader) y [Colaborador de supervisión](../../role-based-access-control/built-in-roles.md#monitoring-contributor) a la suscripción en la que reside el recurso de Application Insights. Para cambiar esta configuración, haga clic en la opción **Configurar** de la notificación por correo electrónico, o bien abra la configuración de Detección inteligente en Application Insights. 
  
  ![Configuración de Detección inteligente](media/proactive-performance-diagnostics/smart_detection_configuration.png)
  
  * Para dejar de recibir notificaciones por correo electrónico, puede usar el vínculo de **cancelación de suscripción** del correo electrónico de Detección inteligente .

Los mensajes de correo electrónico de las anomalías de rendimiento de Detección inteligente tienen una limitación de un correo electrónico al día por recurso de Application Insights. El mensaje de correo electrónico solo se enviará si se ha detectado, como mínimo, un problema nuevo ese día. No obtendrá repeticiones de ningún mensaje. 

## <a name="faq"></a>Preguntas más frecuentes

* *¿El personal de Microsoft mira mis datos?*
  * No. El servicio es completamente automático. Solo obtendrá las notificaciones. Sus datos son [privados](./data-retention-privacy.md).
* *¿Analiza todos los datos recopilados por Application Insights?*
  * No en este momento. Actualmente, analizamos el tiempo de respuesta de la solicitud, el tiempo de respuesta de dependencia y el tiempo de carga de la página. En un futuro analizaremos más métricas.

* ¿En qué tipos de aplicación funciona?
  * Estas degradaciones se detectan en cualquier aplicación que generan los datos de telemetría. Si instaló Application Insights en su aplicación web, el seguimiento de las solicitudes y las dependencias se realiza automáticamente. Sin embargo, en los servicios back-end u otras aplicaciones, si insertó llamadas a [TrackRequest()](./api-custom-events-metrics.md#trackrequest) o [TrackDependency](./api-custom-events-metrics.md#trackdependency), la Detección inteligente funcionará de la misma manera.

* *¿Puedo crear mis propias reglas de detección de anomalías o personalizar las existentes?*

  * Aún no, pero puede realizar lo siguiente:
    * [Configurar alertas](../platform/alerts-log.md) que le indiquen cuándo una métrica cruza un umbral.
    * [Exportar telemetría](./export-telemetry.md) a una [base de datos](./code-sample-export-sql-stream-analytics.md) o a [PowerBI](./export-power-bi.md) para poder realizar un análisis usted mismo.
* *¿Con qué frecuencia se lleva a cabo el análisis?*

  * Ejecutamos el análisis diariamente en la telemetría del día anterior (día completo en la zona horaria UTC).
* *¿Sustituye esto a las [alertas de métricas](../platform/alerts-log.md)?*
  * No.  No nos comprometemos a detectar cada comportamiento que el usuario podría anómalo.


* *Si no tomo ninguna medida como respuesta a una notificación, ¿recibiré un aviso?*
  * No, obtendrá un mensaje sobre cada problema solo una vez. Si el problema persiste, se actualizará en la hoja de fuente de Detección inteligente.
* *Perdí el mensaje de correo electrónico. ¿Dónde puedo encontrar las notificaciones en el portal?*
  * En la información general de Application Insights de su aplicación, haga clic en el icono **Detección inteligente**. Allí encontrará todas las notificaciones de los 90 últimos días.

## <a name="how-can-i-improve-performance"></a>¿Cómo puedo mejorar el rendimiento?
Las respuestas lentas y los errores se encuentran entre los principales motivos de frustración entre los usuarios de los sitios web, como bien sabrá por su propia experiencia. Por lo tanto, es importante que trate los problemas.

### <a name="triage"></a>Evaluación de errores
En primer lugar, ¿es realmente importante? Si una página siempre tarda en cargarse, pero solo un 1 % de los usuarios del sitio accede a ella, es posible que tenga cosas más importantes de las que ocuparse. Por otro lado, si solo un 1 % de los usuarios la abren, pero aún así produce excepciones cada vez, puede que merezca la pena investigar el problema.

Use la instrucción de impacto (usuarios afectados o porcentaje de tráfico) como guía general, pero tenga en cuenta que esta no le va a explicar todo lo que pasa. Recopile otras pruebas para confirmar.

Tenga en cuenta los parámetros del problema. Si es dependiente de la ubicación, configure [pruebas de disponibilidad](./monitor-web-app-availability.md) que incluyan la región: simplemente puede que haya problemas de red en esa área.

### <a name="diagnose-slow-page-loads"></a>Diagnostico de cargas de página lentas
¿Dónde está el problema? ¿Es el servidor lento para responder, es la página muy larga o requiere mucho trabajo del explorador para mostrarla?

Abra la hoja de métricas del navegador. La visualización segmentada del tiempo de carga de página del explorador muestra en qué se va el tiempo. 

* Si el **tiempo de solicitud de envío** es alto, o bien el servidor responde con lentitud o la solicitud es un envío con una gran cantidad de datos. Mire las [métricas de rendimiento](./web-monitor-performance.md#metrics) para investigar los tiempos de respuesta.
* Configure el [seguimiento de dependencias](./asp-net-dependencies.md) para ver si la lentitud se debe a los servicios externos o a su base de datos.
* Si predomina la **recepción de respuesta** , la página y sus elementos dependientes (JavaScript, CSS, imágenes y demás, excluyendo los datos cargados de forma asincrónica) son largos. Configure una [prueba de disponibilidad](./monitor-web-app-availability.md), y asegúrese de establecer la opción de cargar los elementos dependientes. Cuando obtenga algunos resultados, abra el detalle de un resultado y expándalo para ver los tiempos de carga de los distintos archivos.
* Un **Tiempo de procesamiento del cliente** alto sugiere que los scripts se ejecutan con lentitud. Si el motivo no es obvio, considere la posibilidad de agregar algún código de control de tiempo y enviar las horas en llamadas de trackMetric.

### <a name="improve-slow-pages"></a>Mejora de páginas lentas
Hay un sitio web completo de consejos sobre cómo mejorar las respuestas del servidor y los tiempos de carga de página, por lo que no intentaremos repetirlo aquí todo otra vez. Estas son algunas sugerencias que probablemente ya conozca, pero que pueden ayudarle a pensar en soluciones:

* Ralentización de la carga debido a archivos grandes: cargue los scripts y otras partes de forma asincrónica. Use la agrupación de scripts. Divida la página principal en widgets que cargan sus datos por separado. No envíe simples HTML antiguos para tablas de gran tamaño: use un script para solicitar los datos como JSON u otro formato compacto y luego rellene la tabla en su lugar. Existen excelentes marcos para ayudarle con todo esto. (Lo que también implican grandes scripts, por supuesto.)
* Ralentización de las dependencias de servidor: tenga en cuenta las ubicaciones geográficas de los componentes. Por ejemplo, si usa Azure, asegúrese de que el servidor web y la base de datos se encuentran en la misma región. ¿Es posible que consultas las consultas recuperen más información de la que necesitan? ¿Podría ayudar el almacenamiento en caché o el procesamiento por lotes?
* Problemas de capacidad: eche una ojeada a las métricas de servidor de tiempos de respuesta y recuentos de solicitudes. Si los tiempos de respuesta presentan picos desproporcionados en recuentos de solicitud, es probable que se está tirando en exceso de los servidores.


## <a name="server-response-time-degradation"></a>Degradación del tiempo de respuesta del servidor

La notificación de degradación del tiempo de respuesta indica lo siguiente:

* El tiempo de respuesta en comparación con el tiempo de respuesta normal de esta operación.
* El número de usuarios afectados.
* El tiempo promedio de respuesta y el tiempo de respuesta del percentil 90 de esta operación en el día de la detección y 7 días antes. 
* El número de solicitudes de esta operación en el día de la detección y 7 días antes.
* La correlación entre la degradación en esta operación y las degradaciones en las dependencias relacionadas. 
* Los vínculos para ayudarlo a diagnosticar el problema:
  * Los seguimientos del generador de perfiles para ayudarlo a ver en qué se ha invertido el tiempo de operación (el vínculo está disponible si se han recopilado ejemplos de seguimientos del generador de perfiles para esta operación durante el periodo de detección). 
  * Los informes de rendimiento del Explorador de métrica, donde puede segmentar y desglosar los filtros o intervalos de tiempo de esta operación.
  * Busque esta llamada ver las propiedades de llamadas específicas.
  * Informes de error: si el número es mayor que 1, significa que hubo errores en esta operación que pueden haber contribuido a la degradación del rendimiento.

## <a name="dependency-duration-degradation"></a>Degradación de la duración de la dependencia

Las aplicaciones modernas adoptan cada vez más enfoques de diseño de microservicios, lo que en muchos casos conduce a que el nivel de confiabilidad en los servicios externos sea muy elevado. Por ejemplo, si la aplicación se basa en alguna plataforma de datos o incluso si genera su propio servicio de bots, probablemente confiará en algún proveedor de servicios cognitivos para habilitar sus bots con el fin de que interactúen de una forma más humana, así como en algún servicio de almacenamiento de datos que utilicen dichos bots para responder.  

Notificación de degradación de dependencia de ejemplo:

![Este es un ejemplo de detección de degradación de la duración de la dependencia](media/proactive-performance-diagnostics/dependency_duration_degradation.png)

Observe que le indica:

* La duración en comparación con el tiempo de respuesta normal de esta operación
* El número de usuarios afectados.
* Duración promedio y duración del percentil 90 de esta dependencia en el día de la detección y 7 días antes.
* Número de llamadas de dependencia en el día de la detección y 7 días antes.
* Los vínculos para ayudarlo a diagnosticar el problema:
  * Informes de rendimiento del Explorador de métrica de esta dependencia.
  * Busque estas llamadas de dependencia para ver las propiedades de llamadas.
  * Informes de error: si el número es mayor que 1, significa que hubo errores de llamadas de dependencia durante el período de detección que podrían haber contribuido a la degradación de la duración. 
  * Abra Análisis con las consultas que calculan el número y la duración de esta dependencia.  

## <a name="smart-detection-of-slow-performing-patterns"></a>Detección inteligente de los patrones de rendimiento lento 

Application Insights busca los problemas de rendimiento que podrían afectar solo a algunos de sus usuarios, o bien exclusivamente a sus usuarios en algunos casos. Por ejemplo, una notificación de si las páginas de su aplicación se cargan más lentamente en un tipo de explorador que en otros, o de si las solicitudes se atienden de una forma más lenta desde un servidor determinado. También puede detectar problemas asociados con combinaciones de propiedades, como los clientes que usan un determinado sistema operativo y experimentan cargas lentas de página.  

Anomalías como estas son muy difíciles de detectar con tan solo inspeccionar los datos, pero son más comunes de lo que puede creer. A menudo, solo se muestran cuando se quejan los clientes. Para entonces, ya es demasiado tarde: los usuarios afectados ya se han pasado a sus competidores.

En la actualidad, nuestros algoritmos examinan tiempos de carga de página, tiempos de respuesta de solicitud en el servidor y tiempos de respuesta de dependencia.  

No tiene que establecer ningún umbral o regla de configuración. El aprendizaje automático y los algoritmos de minería de datos se usan para detectar patrones anormales.

![En la alerta de correo electrónico, haga clic en el vínculo para abrir el informe de diagnóstico en Azure.](./media/proactive-performance-diagnostics/03.png)

* **Cuándo** muestra la hora en que se detectó el problema.
* **Qué** describe:

  * el problema detectado;
  * las características del conjunto de eventos que encontramos mostraron el comportamiento del problema.
* La tabla compara el conjunto de bajo rendimiento con el comportamiento medio de todos los demás eventos.

Haga clic en los vínculos para abrir el Explorador de métricas y Búsqueda en los informes relevantes, filtrados en la hora y las propiedades del conjunto de rendimiento lento.

Modifique los filtros y el intervalo de tiempo para explorar la telemetría.

## <a name="next-steps"></a>Pasos siguientes
Estas herramientas de diagnóstico lo ayudarán a inspeccionar los datos de telemetría de su aplicación:

* [Generador de perfiles](profiler.md) 
* [depurador de instantáneas](./snapshot-debugger.md)
* [Analytics](../log-query/get-started-portal.md)
* [Diagnóstico de análisis inteligente](../log-query/log-query-overview.md)

Las detecciones inteligentes son completamente automáticas. Pero ¿quizás le gustaría configurar algunas alertas más?

* [Alertas de métricas configuradas manualmente](../platform/alerts-log.md)
* [Pruebas web de disponibilidad](./monitor-web-app-availability.md)


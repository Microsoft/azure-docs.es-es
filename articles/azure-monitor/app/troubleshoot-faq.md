---
title: Preguntas más frecuentes sobre Azure Application Insights | Microsoft Docs
description: Preguntas más frecuentes sobre Application Insights.
services: application-insights
documentationcenter: .net
author: mrbullwinkle
manager: carmonm
ms.assetid: 0e3b103c-6e2a-4634-9e8c-8b85cf5e9c84
ms.service: application-insights
ms.workload: mobile
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 12/17/2018
ms.author: mbullwin
ms.openlocfilehash: ab1327b42a76a6e76183d84cb1750cce8b85228f
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65604284"
---
# <a name="application-insights-frequently-asked-questions"></a>Application Insights: Preguntas más frecuentes

## <a name="configuration-problems"></a>Problemas de configuración
*Tengo problemas con la configuración de:*

* [Aplicación .NET](asp-net-troubleshoot-no-data.md)
* [Supervisión de una aplicación que ya se está ejecutando](monitor-performance-live-website-now.md#troubleshoot)
* [Diagnóstico de Azure](../../azure-monitor/platform/diagnostics-extension-to-application-insights.md)
* [Aplicaciones web de Java](java-troubleshoot.md)

*No recibo datos de mi servidor*

* [Establecer excepciones del firewall](ip-addresses.md)
* [Configurar un servidor ASP.NET](monitor-performance-live-website-now.md)
* [Configurar un servidor de Java](java-agent.md)

## <a name="can-i-use-application-insights-with-"></a>¿Se puede usar Application Insights con...?

* [Aplicaciones web en un servidor de IIS: local o en una máquina virtual](asp-net.md)
* [Aplicaciones web de Java](java-get-started.md)
* [Aplicaciones de Node.js](nodejs.md)
* [Aplicaciones web de Azure](azure-web-apps.md)
* [Cloud Services en Azure](cloudservices.md)
* [Servidores de aplicaciones que se ejecutan en Docker](docker.md)
* [Aplicaciones web de una sola página](javascript.md)
* [SharePoint](sharepoint.md)
* [Aplicación de escritorio de Windows](windows-desktop.md)
* [Otras plataformas](platforms.md)

## <a name="is-it-free"></a>¿Es gratis?

Sí, para su uso experimental. En el plan de precios básico, la aplicación puede enviar una determinada asignación de datos cada mes de forma gratuita. La asignación disponible es lo suficientemente amplia como para abarcar el desarrollo y la publicación de una aplicación para un número reducido de usuarios. Puede establecer un límite para evitar que se procese más de una determinada cantidad de datos.

Los volúmenes más grandes de telemetría se cobran por GB. Se proporcionan algunas sugerencias sobre cómo [limitar los gastos](pricing.md).

El plan Enterprise cobra por cada día que cada nodo de servidor web envía telemetría. Resulta adecuado si desea usar exportación continua a gran escala.

[Lea el plan de precios](https://azure.microsoft.com/pricing/details/application-insights/).

## <a name="how-much-is-it-costing"></a>¿Cuánto costará?

* Abra la página **Uso y costos estimados** en un recurso de Application Insights. Hay un gráfico de uso reciente. Puede establecer un límite de volumen de datos, si quiere.
* Abra la [hoja de facturación de Azure](https://portal.azure.com/#blade/Microsoft_Azure_Billing/BillingBlade/Overview) para ver las facturas de todos los recursos.

## <a name="q14"></a>¿Qué modifica Application Insights en mi proyecto?
Los detalles dependen del tipo de proyecto. Para una aplicación web:

* Agregue estos archivos al proyecto:

  * ApplicationInsights.config.
  * ai.js
* Instale estos paquetes de NuGet:

  * *API de Application Insights* : la API central.
  * *API de Application Insights para aplicaciones web* : se usa para enviar la telemetría del servidor.
  * *API de Application Insights para aplicaciones JavaScript* : se usa para enviar la telemetría del cliente.

    El paquete incluye estos ensamblados:
  * Microsoft.ApplicationInsights
  * Microsoft.ApplicationInsights.Platform
* Inserta elementos en:

  * Web.config
  * packages.config
* (Solo nuevos proyectos: si [agrega Application Insights a un proyecto existente][start], tiene que hacerlo manualmente). Inserte fragmentos de código en el código de cliente y servidor para inicializarlos con el identificador de recursos de Application Insights. Por ejemplo, en una aplicación MVC, el código se inserta en la página maestra Views/Shared/_Layout.cshtml.

## <a name="how-do-i-upgrade-from-older-sdk-versions"></a>¿Cómo se puede actualizar desde versiones anteriores de SDK?
Consulte las [notas de la versión](release-notes.md) del SDK adecuado para el tipo de aplicación.

## <a name="update"></a>¿Cómo puedo cambiar el recurso de Azure al que mi proyecto envía datos?
En el Explorador de soluciones, haga clic con el botón derecho en `ApplicationInsights.config` y elija **Actualizar Application Insights**. Puede enviar los datos a un recurso nuevo o existente en Azure. El Asistente para actualización cambia la clave de instrumentación en ApplicationInsights.config, que determina dónde el SDK del servidor envía los datos. A menos que desactive la opción "Actualizar todo", también cambiará la clave donde aparece en las páginas web.

## <a name="what-is-status-monitor"></a>¿Qué es el Monitor de estado?

Una aplicación de escritorio que puede usar en el servidor web de IIS para ayudar a configurar Application Insights en aplicaciones web. No recopila telemetría: puede detenerla cuando no vaya a configurar una aplicación. 

[Más información](monitor-performance-live-website-now.md#questions).

## <a name="what-telemetry-is-collected-by-application-insights"></a>¿Qué telemetría recopila Application Insights?

De las aplicaciones web de servidor:

* Solicitudes HTTP
* [Dependencias](asp-net-dependencies.md). Llamadas a: instancias de SQL Database; llamadas HTTP a servicios externos; Azure Cosmos DB, tabla, almacenamiento de blobs y colas. 
* [Excepciones](asp-net-exceptions.md) y seguimientos de pila.
* [Contadores de rendimiento](performance-counters.md): si usa el [Monitor de estado](monitor-performance-live-website-now.md), la [supervisión de Azure](azure-web-apps.md) o el [escritor de collectd de Application Insights](java-collectd.md).
* [Eventos y métricas personalizados](api-custom-events-metrics.md) que puede crear mediante código.
* [Registros de seguimiento](asp-net-trace-logs.md) si configura el recopilador adecuado.

De las [páginas web de cliente](javascript.md):

* [Recuentos de vistas de página](usage-overview.md)
* [Llamadas AJAX](asp-net-dependencies.md): solicitudes realizadas desde un script en funcionamiento
* Datos de carga de vista de página
* Recuentos de usuarios y sesiones
* [Id. de usuario autenticados](api-custom-events-metrics.md#authenticated-users).

De otros orígenes, si los configura:

* [Diagnóstico de Azure](../platform/diagnostics-extension-to-application-insights.md)
* [Importación a Analytics](../platform/data-collector-api.md)
* [Log Analytics](../platform/data-collector-api.md)
* [Logstash](../platform/data-collector-api.md)

## <a name="can-i-filter-out-or-modify-some-telemetry"></a>¿Puedo filtrar o modificar algunos datos de telemetría?

Sí, en el servidor puede escribir:

* Procesador de telemetría para filtrar o agregar propiedades a los elementos de telemetría seleccionados antes de que se envíen desde su aplicación.
* Inicializador de telemetría para agregar propiedades a todos los elementos de telemetría.

Más información sobre [ASP.NET](api-filtering-sampling.md) o [Java](java-filter-telemetry.md).

## <a name="how-are-city-countryregion-and-other-geo-location-data-calculated"></a>¿Cómo se calculan ciudad, país o región y otros datos de ubicación geográfica?

Buscamos la dirección IP (IPv4 o IPv6) del cliente web mediante [GeoLite2](https://dev.maxmind.com/geoip/geoip2/geolite2/).

* Telemetría del explorador: recopilamos la dirección IP del remitente.
* Telemetría del servidor: el módulo de Application Insights recopila la dirección IP del cliente. No se recopila si `X-Forwarded-For` está establecido.

Puede configurar `ClientIpHeaderTelemetryInitializer` para tomar la dirección IP de un encabezado distinto. En algunos sistemas, por ejemplo, se mueve mediante un servidor proxy, un equilibrador de carga o la red CDN `X-Originating-IP`. [Más información](https://apmtips.com/blog/2016/07/05/client-ip-address/).

También puede [usar Power BI](export-power-bi.md ) para mostrar la telemetría de solicitudes en un mapa.


## <a name="data"></a>¿Cuánto tiempo se retienen los datos en el portal? ¿Es seguro?
Eche un vistazo a [Privacidad y retención de los datos][data].

## <a name="could-personal-data-be-sent-in-the-telemetry"></a>¿Se podrían enviar datos personales en la telemetría?

Es posible si el código envía tales datos. También puede ocurrir si las variables de los seguimientos de pila incluyen datos personales. El equipo de desarrollo debe llevar a cabo las evaluaciones de riesgo para asegurarse de que los datos personales se tratan correctamente. [Más información sobre la retención y la privacidad de los datos](data-retention-privacy.md).

**Todos** los octetos de la dirección web del cliente siempre se establecen en 0 después de que se buscan los atributos de ubicación geográfica.

## <a name="my-instrumentation-key-is-visible-in-my-web-page-source"></a>Mi clave de instrumentación es visible en mi página web de origen. 

* Esta es una práctica común en las soluciones de supervisión.
* No se puede usar para robar sus datos.
* Se podría usar para sesgar los datos o desencadenar alertas.
* No hemos tenido noticias de que algún cliente haya tenido esos problemas.

Podría:

* Use dos diferentes claves de instrumentación (separe los recursos de Application Insights), para los datos de cliente y servidor. O
* Escribir un servidor proxy que se ejecute en el servidor y hacer que el cliente web envíe datos a través de dicho servidor proxy.

## <a name="post"></a>¿Cómo puedo ver datos POST en Búsqueda de diagnóstico?
Los datos POST no se registran automáticamente, pero se puede usar una llamada TrackTrace: incluya los datos en el parámetro de mensaje. Este tiene un límite de tamaño superior al de las propiedades de cadena, aunque no se puede filtrar por él.

## <a name="should-i-use-single-or-multiple-application-insights-resources"></a>¿Debo usar uno o varios recursos de Application Insights?

Utilice un único recurso para todos los componentes o funciones en un sistema de negocio individual. Use recursos independientes para las versiones de desarrollo, prueba y lanzamiento, y para aplicaciones independientes.

* [Consulte la explicación aquí](separate-resources.md).
* [Ejemplo: servicio en la nube con roles web y de trabajo](cloudservices.md)

## <a name="how-do-i-dynamically-change-the-instrumentation-key"></a>¿Cómo se cambia de forma dinámica la clave de instrumentación?

* [La explicación aquí](separate-resources.md)
* [Ejemplo: servicio en la nube con roles web y de trabajo](cloudservices.md)

## <a name="what-are-the-user-and-session-counts"></a>¿Qué son los recuentos de usuarios y sesiones?

* El SDK de JavaScript establece una cookie de usuario en el cliente web para identificar los usuarios que vuelven y una cookie de sesión para agrupar las actividades.
* Si no hay ningún script de cliente, puede [establecer cookies en el servidor](https://apmtips.com/blog/2016/07/09/tracking-users-in-api-apps/).
* Si un usuario real usa su sitio en exploradores diferentes o usa la exploración InPrivate o de incógnito, o distintas máquinas, entonces se contabilizará más de una vez.
* Para identificar un usuario que ha iniciado sesión en todas las máquinas y exploradores, agregue una llamada a [setAuthenticatedUserContext()](api-custom-events-metrics.md#authenticated-users).

## <a name="q17"></a> ¿He habilitado todo en Application Insights?
| Qué debería ver | Cómo conseguirlo | Razones para quererlo |
| --- | --- | --- |
| Gráficos de disponibilidad |[Pruebas web](monitor-web-app-availability.md) |Saber que la aplicación web funciona |
| Rendimiento de la aplicación de servidor: tiempos de respuesta, etc. |[Agregue Application Insights a su proyecto](asp-net.md) o [instale el Monitor de estado de Application Insights en un servidor](monitor-performance-live-website-now.md) (o escriba su propio código para [realizar un seguimiento de las dependencias](api-custom-events-metrics.md#trackdependency)). |Detectar problemas de rendimiento |
| Telemetría de dependencia |[Instalar el Monitor de estado de Application Insights en el servidor](monitor-performance-live-website-now.md) |Diagnosticar problemas con las bases de datos u otros componentes externos |
| Obtener seguimientos de pila de las excepciones |[Insertar llamadas TrackException en el código](asp-net-exceptions.md) (aunque algunas se notifican automáticamente) |Detectar y diagnosticar excepciones |
| Buscar seguimientos del registro |[Agregar un adaptador de registro](asp-net-trace-logs.md) |Diagnosticar excepciones, problemas de rendimiento |
| Aspectos básicos del uso de cliente: vistas de página, sesiones,... |[Inicializador de JavaScript en páginas web](javascript.md) |Análisis de uso |
| Métricas personalizadas de cliente |[Seguimiento de llamadas en páginas web](api-custom-events-metrics.md) |Mejorar la experiencia del usuario |
| Métricas personalizadas de servidor |[Seguimiento de llamadas en el código de servidor](api-custom-events-metrics.md) |Business intelligence |

## <a name="why-are-the-counts-in-search-and-metrics-charts-unequal"></a>¿Por qué los recuentos de los gráficos de búsqueda y métrica no son iguales?

El [muestreo](sampling.md) reduce el número de elementos de telemetría (solicitudes, eventos personalizados, etc.) que se envían realmente desde su aplicación al portal. En el gráfico de búsqueda, ve el número de elementos realmente recibidos. En los gráficos de métrica que muestran un recuento de eventos, ve el número de eventos originales que se han producido. 

Cada elemento que se transmite lleva una propiedad `itemCount` que muestra cuántos eventos originales representa ese elemento. Para ver el muestreo en funcionamiento, puede ejecutar esta consulta en Analytics:

```
    requests | summarize original_events = sum(itemCount), transmitted_events = count()
```


## <a name="automation"></a>Automation

### <a name="configuring-application-insights"></a>Configuración de Application Insights

También puede [escribir scripts de PowerShell](powershell.md) mediante el Monitor de recursos de Azure para:

* Crear y actualizar recursos de Application Insights
* Consultar el plan de precios
* Obtener la clave de instrumentación
* Agregar una alerta de métrica
* Agregar una prueba de disponibilidad.

No puede configurar un informe del Explorador de métricas ni configurar la exportación continua.

### <a name="querying-the-telemetry"></a>Consulta de la telemetría

Use la [API de REST](https://dev.applicationinsights.io/) para ejecutar consultas de [Analytics](analytics.md).

## <a name="how-can-i-set-an-alert-on-an-event"></a>¿Cómo puedo establecer una alerta sobre un evento?

Las alertas de Azure solo se establecen sobre métricas. Cree una métrica personalizada que cruce un umbral de valor cada vez que se produzca el evento. A continuación, establezca una alerta sobre la métrica. Tenga en cuenta que: cada vez que la métrica cruce el umbral en cualquier dirección, recibirá una notificación; no recibirá una notificación hasta que se cruza por primera vez, sin importar si el valor inicial es alto o bajo; siempre hay una latencia de algunos minutos.

## <a name="are-there-data-transfer-charges-between-an-azure-web-app-and-application-insights"></a>¿Existen cargos por transferencia de datos entre una aplicación web de Azure y Application Insights?

* Si la aplicación web de Azure se hospeda en un centro de datos donde hay un punto de conexión de recopilación de Application Insights, no hay ningún cargo. 
* Si no hay ningún punto de conexión de recopilación en el centro de datos host, se le cobrarán los [cargos salientes de Azure](https://azure.microsoft.com/pricing/details/bandwidth/) de la telemetría de la aplicación.

Esto no depende de dónde se encuentre hospedado su recurso de Application Insights. Simplemente depende de la distribución de nuestros puntos de conexión.

## <a name="can-i-send-telemetry-to-the-application-insights-portal"></a>¿Puedo enviar telemetría al portal de Application Insights?

Se recomienda usar nuestros SDK y la [API de SDK](api-custom-events-metrics.md). Hay variantes del SDK para distintas [plataformas](platforms.md). Estos SDK controlan el almacenamiento en búfer, la compresión, la limitación, los reintentos, etc. Sin embargo, el [esquema de ingesta](https://github.com/Microsoft/ApplicationInsights-dotnet/tree/develop/Schema/PublicSchema) y el [protocolo de punto de conexión](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/EndpointSpecs/ENDPOINT-PROTOCOL.md) son públicos.

## <a name="can-i-monitor-an-intranet-web-server"></a>¿Puedo supervisar un servidor web de una intranet?

Sí, pero será necesario permitir el tráfico en nuestros servicios mediante excepciones de firewall o redirecciones del proxy.
- QuickPulse `https://rt.services.visualstudio.com:443` 
- ApplicationIdProvider `https://dc.services.visualstudio.com:443` 
- TelemetryChannel `https://dc.services.visualstudio.com:443` 


Consulte nuestra lista completa de servicios y direcciones IP [aquí](../../azure-monitor/app/ip-addresses.md).

### <a name="firewall-exception"></a>Excepción de firewall

Permita que el servidor web envíe telemetría a nuestros puntos de conexión. 

### <a name="gateway-redirect"></a>Redirección de la puerta de enlace

Redirija el tráfico desde su servidor a una puerta de enlace de su intranet mediante la sobrescritura de los puntos de conexión en la configuración.
Si estas propiedades de "punto de conexión" no están presentes en la configuración, estas clases usarán los valores predeterminados que se muestran en el ejemplo ApplicationInsights.config siguiente. 

La puerta de enlace debe redirigir el tráfico a la dirección base del punto de conexión. En la configuración, reemplace los valores predeterminados por `http://<your.gateway.address>/<relative path>`.


#### <a name="example-applicationinsightsconfig-with-default-endpoints"></a>Ejemplo de ApplicationInsights.config con puntos de conexión predeterminados:
```xml
<ApplicationInsights>
  ...
  <TelemetryModules>
    <Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse.QuickPulseTelemetryModule, Microsoft.AI.PerfCounterCollector">
      <QuickPulseServiceEndpoint>https://rt.services.visualstudio.com/QuickPulseService.svc</QuickPulseServiceEndpoint>
    </Add>
  </TelemetryModules>
    ...
  <TelemetryChannel>
     <EndpointAddress>https://dc.services.visualstudio.com/v2/track</EndpointAddress>
  </TelemetryChannel>
  ...
  <ApplicationIdProvider Type="Microsoft.ApplicationInsights.Extensibility.Implementation.ApplicationId.ApplicationInsightsApplicationIdProvider, Microsoft.ApplicationInsights">
    <ProfileQueryEndpoint>https://dc.services.visualstudio.com/api/profiles/{0}/appId</ProfileQueryEndpoint>
  </ApplicationIdProvider>
  ...
</ApplicationInsights>
```

_Tenga en cuenta que ApplicationIdProvider está disponible a partir de la versión 2.6.0_

### <a name="proxy-passthrough"></a>Paso a través del proxy

Paso a través del proxy se puede lograr mediante la configuración de un nivel de equipo o en el nivel de aplicación proxy.
Para obtener más información consulte el artículo de dotnet en [DefaultProxy](https://docs.microsoft.com/dotnet/framework/configure-apps/file-schema/network/defaultproxy-element-network-settings).
 
 Ejemplo Web.config:
 ```xml
<system.net>
    <defaultProxy>
      <proxy proxyaddress="http://xx.xx.xx.xx:yyyy" bypassonlocal="true"/>
    </defaultProxy>
</system.net>
```
 

## <a name="can-i-run-availability-web-tests-on-an-intranet-server"></a>¿Puedo ejecutar pruebas web de disponibilidad en un servidor de intranet?

Nuestras [pruebas web](monitor-web-app-availability.md) se ejecutan en puntos de presencia que están repartidos por todo el mundo. Hay dos soluciones:

* Puerta de firewall: permitir solicitudes al servidor desde [la lista larga y modificable de agentes de prueba web](ip-addresses.md).
* Escribir su propio código para enviar solicitudes periódicas a su servidor desde dentro de la intranet. Con este fin, puede ejecutar pruebas web de Visual Studio. El evaluador puede enviar los resultados a Application Insights mediante la API TrackAvailability().

## <a name="how-long-does-it-take-for-telemetry-to-be-collected"></a>¿Cuánto tiempo tarda la recopilación de telemetría?

La mayoría de los datos de Application Insights tienen una latencia de menos de 5 minutos. Algunos datos pueden tardar más tiempo, sobre todo, los archivos de registro más grandes. Para más información, consulte [Acuerdo de Nivel de Servicio de Application Insights](https://azure.microsoft.com/support/legal/sla/application-insights/v1_2/).

## <a name="more-answers"></a>Más respuestas
* [Foro de Application Insights](https://social.msdn.microsoft.com/Forums/vstudio/en-US/home?forum=ApplicationInsights)

<!--Link references-->

[data]: data-retention-privacy.md
[platforms]: platforms.md
[start]: app-insights-overview.md
[windows]: app-insights-windows-get-started.md

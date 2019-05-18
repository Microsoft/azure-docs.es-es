---
title: Visualización de datos de aplicación de Azure Application Insights | Microsoft Docs
description: Puede usar la solución Application Insights Connector para diagnosticar problemas de rendimiento y comprender lo que los usuarios hacen con su aplicación cuando se supervisa mediante Application Insights.
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: 49280cad-3526-43e1-a365-c6a3bf66db52
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 02/13/2019
ms.author: magoedte
ms.openlocfilehash: c7c0d2e3fb818f74a65502674188c523d23729e8
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65606743"
---
# <a name="application-insights-connector-management-solution-deprecated"></a>Solución de administración de Application Insights Connector (en desuso)

![Símbolo de Application Insights](./media/app-insights-connector/app-insights-connector-symbol.png)

>[!NOTE]
> Con el soporte técnico de [consultas entre recursos](../../azure-monitor/log-query/cross-workspace-query.md), la solución de administración Application Insights Connector ya no es necesaria. Está en desuso y se ha eliminado de Azure Marketplace junto con el portal de OMS que dejó de usarse oficialmente el 15 de enero de 2019 para la nube comercial de Azure. Se retirará el 30 de marzo de 2019 para la nube de US Government de Azure.
>
>Las conexiones existentes continuarán funcionando hasta el 30 de junio de 2019.  Con el desuso del portal de OMS, no hay manera de configurar y quitar las conexiones existentes del portal. Consulte [Eliminación del conector con PowerShell](#removing-the-connector-with-powershell) que aparece a continuación para obtener un script que usa PowerShell para eliminar las conexiones existentes.
>
>Para obtener instrucciones sobre la realización de consultas en datos de registro de Application Insights, consulte [Unificación de varios recursos de Application Insights de Azure Monitor](../log-query/unify-app-resource-data.md). Para más información sobre el abandono del portal de OMS, consulte [Traslado del portal de OMS a Azure](../../azure-monitor/platform/oms-portal-transition.md).
>
> 

La solución Applications Insights Connector le permite diagnosticar problemas de rendimiento y comprender lo que los usuarios hacen con su aplicación cuando se supervisa mediante [Application Insights](../../azure-monitor/app/app-insights-overview.md). En Log Analytics, hay disponibles vistas de la misma telemetría de aplicación que los desarrolladores ven en Application Insights. Sin embargo, cuando se integran las aplicaciones de Application Insights con Log Analytics, la visibilidad de las aplicaciones aumenta debido a que los datos de operación y de aplicación están en el mismo lugar. Tener las mismas vistas le permite colaborar con los desarrolladores de aplicaciones. Las vistas comunes pueden ayudar a disminuir el tiempo para detectar y resolver problemas tanto de aplicaciones como de plataformas.

Cuando usa la solución, puede:

- Ver todas las aplicaciones de Application Insights en un solo lugar, incluso si se encuentran en suscripciones de Azure distintas.
- Correlacionar datos de infraestructura con datos de aplicación.
- Visualizar datos de aplicación con perspectivas en la búsqueda de registros.
- Pasar de los datos de Log Analytics a la aplicación de Application Insights en Azure Portal.


[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="connected-sources"></a>Orígenes conectados

A diferencia de la mayoría de las demás soluciones de Log Analytics, los agentes no recopilan datos para Application Insights Connector. Todos los datos usados por la solución proceden directamente de Azure.

| Origen conectado | Compatible | DESCRIPCIÓN |
| --- | --- | --- |
| [Agentes de Windows](../../azure-monitor/platform/agent-windows.md) | No | La solución no recopila información de los agentes de Windows. |
| [Agentes de Linux](../../azure-monitor/learn/quick-collect-linux-computer.md) | No | La solución no recopila información de los agentes de Linux. |
| [Grupo de administración de SCOM](../../azure-monitor/platform/om-agents.md) | No | La solución no recopila información de los agentes de un grupo de administración SCOM conectado. |
| [Cuenta de Almacenamiento de Azure](collect-azure-metrics-logs.md) | No | La solución no recopila información de Azure Storage. |

## <a name="prerequisites"></a>Requisitos previos

- Para obtener acceso a la información sobre Application Insights Connector, debe tener una suscripción de Azure
- Debe tener como mínimo un recurso de Application Insights configurado.
- Debe ser propietario o colaborador del recurso de Application Insights.

## <a name="configuration"></a>Configuración

1. Habilite la solución Azure Web Apps Analytics desde [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.AppInsights?tab=Overview) o mediante el proceso descrito en [Incorporación de soluciones de Log Analytics desde la galería de soluciones](../../azure-monitor/insights/solutions.md).
2. Vaya a [Azure Portal](https://portal.azure.com). Seleccione **Todos los servicios** para abrir Application Insights. Después, busque “Application Insights”. 
3. En **Seleccionar una suscripción**, seleccione una suscripción que tenga recursos de Application Insights y, después, en **Nombre**, seleccione una o más aplicaciones.
4. Haga clic en **Save**(Guardar).

Aproximadamente en 30 minutos, los datos quedan disponibles y el icono de Application Insights se actualiza con los datos, como se ve en la imagen siguiente:

![Icono de Application Insights](./media/app-insights-connector/app-insights-tile.png)

Otros puntos que se deben considerar:

- Solo puede vincular aplicaciones de Application Insights a un área de trabajo de Log Analytics.
- Solo puede vincular [recursos del nivel Básico o Enterprise de Application Insights ](https://azure.microsoft.com/pricing/details/application-insights) a Log Analytics. Sin embargo, puede usar el nivel Gratis de Log Analytics.

## <a name="management-packs"></a>Módulos de administración

Esta solución no instala ningún módulo de administración en grupos de administración conectados.

## <a name="use-the-solution"></a>Uso de la solución

Las secciones siguientes describen cómo puede usar las hojas que aparecen en el panel de Application Insights para ver los datos de las aplicaciones e interactuar con ellos.

### <a name="view-application-insights-connector-information"></a>Visualización de la información de Application Insights Connector

Haga clic en el icono **Application Insights** para abrir el panel **Application Insights** y ver las hojas siguientes.

![Panel Application Insights](./media/app-insights-connector/app-insights-dash01.png)

![Panel Application Insights](./media/app-insights-connector/app-insights-dash02.png)

El panel incluye las hojas que aparecen en la tabla. Cada hoja muestra hasta diez elementos que coinciden con los criterios de esa hoja para el ámbito e intervalo de tiempo especificados. Puede ejecutar una búsqueda de registros que devuelva todos los registros cuando haga clic en **Ver todo** en la parte inferior de la hoja o cuando haga clic en el encabezado de esta.


| **Columna** | **Descripción** |
| --- | --- |
| Aplicaciones: número de aplicaciones | Muestra el número de aplicaciones en los recursos de aplicación. También muestra los nombres de aplicación y la cantidad de registros de aplicación para cada uno. Haga clic en el número para ejecutar una búsqueda de registros de <code>ApplicationInsights &#124; summarize AggregatedValue = sum(SampledCount) by ApplicationName</code> <br><br>  Haga clic en un nombre de aplicación para ejecutar una búsqueda de registros de la aplicación que muestre registros de aplicación por host, registros por tipo de telemetría y todos los datos por tipo (según el último día). |
| Volumen de datos: hosts que envían datos | Muestra el número de hosts de equipos que envían datos. También muestra los hosts de equipos y números de registros para cada host. Haga clic en el número para ejecutar una búsqueda de registros de <code>ApplicationInsights &#124; summarize AggregatedValue = sum(SampledCount) by Host</code> <br><br> Haga clic en un nombre de equipo para ejecutar una búsqueda de registros del host que muestre registros de aplicación por host, registros por tipo de telemetría y todos los datos por tipo (según el último día). |
| Disponibilidad: resultados de WebTest | Muestra un gráfico de anillos para los resultados de pruebas web, que indican una prueba correcta o errónea. Haga clic en el gráfico para ejecutar una búsqueda de registros de <code>ApplicationInsights &#124; where TelemetryType == "Availability" &#124; summarize AggregatedValue = sum(SampledCount) by AvailabilityResult</code> <br><br> Los resultados muestran el número de pruebas correctas y erróneas en todas las pruebas. Muestra todas las instancias de Web Apps con tráfico durante el último minuto. Haga clic en un nombre de aplicación para ver una búsqueda de registros con los detalles de las pruebas web erróneas. |
| Solicitudes de servidor: solicitudes por hora | Muestra un gráfico de líneas de las solicitudes de servidor por hora para diversas aplicaciones. Mantenga el mouse sobre una línea del gráfico para ver las tres aplicaciones principales que reciben solicitudes en un momento específico. También muestra una lista de las aplicaciones que reciben solicitudes y el número de solicitudes para el período seleccionado. <br><br>Haga clic en el gráfico para ejecutar una búsqueda de registros de <code>ApplicationInsights &#124; where TelemetryType == "Request" &#124; summarize AggregatedValue = sum(SampledCount) by ApplicationName, bin(TimeGenerated, 1h)</code> que muestre un gráfico de líneas más detallado de las solicitudes de servidor por hora para las diversas aplicaciones. <br><br> Haga clic en una aplicación de la lista para ejecutar una búsqueda de registros de <code>ApplicationInsights &#124; where ApplicationName == "yourapplicationname" and TelemetryType == "Request" and iff(isnotnull(toint(RequestSuccess)), RequestSuccess == false, RequestSuccess == "false") == true</code> que muestre una lista de solicitudes, gráficos de las solicitudes en el tiempo y la duración de las solicitudes, y una lista de los códigos de respuestas de las solicitudes.   |
| Errores: solicitudes erróneas por hora | Muestra un gráfico de líneas de las solicitudes de aplicación erróneas por hora. Mantenga el mouse sobre el gráfico para ver las tres aplicaciones principales con solicitudes erróneas en un momento específico. También muestra una lista de aplicaciones con el número de solicitudes erróneas para cada una de ellas. Haga clic en el gráfico para ejecutar una búsqueda de registros de <code>ApplicationInsights &#124; where TelemetryType == "Request" and iff(isnotnull(toint(RequestSuccess)), RequestSuccess == false, RequestSuccess == "false") == true &#124; summarize AggregatedValue = sum(SampledCount) by ApplicationName, bin(TimeGenerated, 1h)</code> que muestre un gráfico de líneas más detallado con las solicitudes de aplicación erróneas. <br><br>Haga clic en un elemento de la lista para ejecutar una búsqueda de registros de <code>ApplicationInsights &#124; where ApplicationName == "yourapplicationname" and TelemetryType == "Request" and iff(isnotnull(toint(RequestSuccess)), RequestSuccess == false, RequestSuccess == "false") == true</code> que muestre solicitudes erróneas, gráficos de las solicitudes erróneas en el tiempo y la duración de las solicitudes, y una lista de los códigos de respuestas de las solicitudes erróneas. |
| Excepciones: excepciones por hora | Muestra un gráfico de líneas de las excepciones por hora. Mantenga el mouse sobre el gráfico para ver las tres aplicaciones principales con excepciones en un momento específico. También muestra una lista de aplicaciones con el número de excepciones para cada una de ellas. Haga clic en el gráfico para ejecutar una búsqueda de registros de <code>ApplicationInsights &#124; where TelemetryType == "Exception" &#124; summarize AggregatedValue = sum(SampledCount) by ApplicationName, bin(TimeGenerated, 1h)</code> que muestre un gráfico de líneas más detallado de excepciones. <br><br>Haga clic en un elemento de la lista para ejecutar una búsqueda de registros de <code>ApplicationInsights &#124; where ApplicationName == "yourapplicationname" and TelemetryType == "Exception"</code> que muestre una lista de excepciones, gráficos de las excepciones en el tiempo y las solicitudes erróneas, y una lista de los tipos de excepción.  |

### <a name="view-the-application-insights-perspective-with-log-search"></a>Visualización de la perspectiva de Application Insights con la búsqueda de registros

Cuando haga clic en cualquier elemento del panel, verá que aparece una perspectiva de Application Insights en la búsqueda. La perspectiva proporciona una visualización extendida, según el tipo de telemetría que seleccione. Por lo tanto, el contenido de la visualización cambia según los distintos tipos de telemetría.

Cuando hace clic en cualquier parte de la hoja Aplicaciones, ve la perspectiva **Aplicaciones** predeterminada.

![Perspectiva Aplicaciones de Application Insights](./media/app-insights-connector/applications-blade-drill-search.png)

La perspectiva muestra información general de la aplicación que seleccionó.

La hoja **Disponibilidad** muestra una vista de perspectiva distinta en la que puede ver los resultados de las pruebas web y las solicitudes erróneas relacionadas.

![Perspectiva Disponibilidad de Application Insights](./media/app-insights-connector/availability-blade-drill-search.png)

Cuando hace clic en cualquier parte de las hojas **Solicitudes de servidor** o **Errores**, los componentes de la perspectiva cambian para ofrecer una visualización relacionada con las solicitudes.

![Hoja Errores de Application Insights](./media/app-insights-connector/server-requests-failures-drill-search.png)

Cuando hace clic en cualquier parte de la hoja **Excepciones**, ve una visualización adaptada a las excepciones.

![Hoja Excepciones de Application Insights](./media/app-insights-connector/exceptions-blade-drill-search.png)

Independientemente de si hace clic en algún elemento del panel **Application Insights Connector**, en la página **Búsqueda** misma, cualquier consulta que devuelva datos de Application Insights muestra la perspectiva de Application Insights. Por ejemplo, si ve los datos de Application Insights, una consulta **&#42;** también muestra la pestaña de perspectivas, como se muestra en la imagen siguiente:

![Application Insights](./media/app-insights-connector/app-insights-search.png)

Los componentes de perspectiva se actualizan según la consulta de búsqueda. Esto significa que puede filtrar los resultados mediante cualquier campo de búsqueda que le permita ver los datos desde:

- Todas las aplicaciones
- Una sola aplicación seleccionada
- Un grupo de aplicaciones

### <a name="pivot-to-an-app-in-the-azure-portal"></a>Paso a una aplicación en Azure Portal

Las hojas de Application Insights Connector se han diseñado para permitirle cambiar a la aplicación seleccionada de Application Insights *cuando usa Azure Portal*. Puede usar la solución como una plataforma de supervisión de alto nivel que le ayuda a solucionar los problemas de una aplicación. Cuando vea un posible problema en cualquiera de las aplicaciones conectadas, puede profundizar en él en la búsqueda de Log Analytics o puede cambiar directamente a la aplicación de Application Insights.

Para hacer lo último, haga clic en los tres puntos suspensivos (**…**) que aparecen al final de cada línea y seleccione **Abrir en Application Insights**.

>[!NOTE]
>**Abrir en Application Insights** no está disponible en Azure Portal.

![Abrir en Application Insights](./media/app-insights-connector/open-in-app-insights.png)

### <a name="sample-corrected-data"></a>Datos de ejemplo corregidos

Application Insights proporciona *[corrección de muestreo](../../azure-monitor/app/sampling.md)* para ayudar a disminuir el tráfico de telemetría. Cuando se habilita el muestreo en la aplicación de Application Insights, obtiene un número reducido de entradas almacenadas en Application Insights y en Log Analytics. Si bien la coherencia de los datos se conserva en las perspectivas y en la página **Application Insights Connector**, debe corregir manualmente los datos muestreados para las consultas personalizadas.

A continuación, se muestra un ejemplo de una corrección de muestreo en una consulta de búsqueda de registros:

```
ApplicationInsights | summarize AggregatedValue = sum(SampledCount) by TelemetryType
```

El campo **Sampled Count** aparece en todas las entradas y muestra el número de puntos de datos que representa la entrada. Si activa el muestreo para la aplicación de Application Insights, el campo **Sampled Count** es mayor que 1. Para registrar el número real de entradas que genera la aplicación, sume los campos **Sampled Count**.

El muestreo afecta solo el número total de entradas que la aplicación genera. No necesita corregir el muestreo para campos de métrica como **RequestDuration** o **AvailabilityDuration**, porque esos campos muestran el promedio de las entradas representadas.

## <a name="input-data"></a>Datos de entrada

La solución recibe los siguientes tipos de telemetría de datos desde las aplicaciones conectadas de Application Insights:

- Disponibilidad
- Excepciones
- Solicitudes
- Vistas de página: para que el área de trabajo reciba vistas de página, se deben configurar las aplicaciones para que recopilen esa información. Para más información, consulte [PageViews](../../azure-monitor/app/api-custom-events-metrics.md#page-views).
- Eventos personalizados: para que el área de trabajo reciba eventos personalizados, se deben configurar las aplicaciones para que recopilen esa información. Para más información, consulte [TrackEvent](../../azure-monitor/app/api-custom-events-metrics.md#trackevent).

Log Analytics recibe los datos desde Application Insights en cuanto están disponibles.

## <a name="output-data"></a>Datos de salida

Se crea un registro con un *tipo* de *ApplicationInsights* para cada tipo de datos de entrada. Los registros de ApplicationInsights tienen propiedades que se muestran en las secciones siguientes:

### <a name="generic-fields"></a>Campos genéricos

| Propiedad | Descripción |
| --- | --- |
| Type | ApplicationInsights |
| ClientIP |   |
| TimeGenerated | Hora del registro |
| ApplicationId | Clave de instrumentación de la aplicación de Application Insights |
| ApplicationName | Nombre de la aplicación de Application Insights |
| RoleInstance | Identificador del host del servidor |
| DeviceType | Dispositivo de cliente |
| ScreenResolution |   |
| Continent | El continente donde se originó la solicitud |
| País | País o región donde se originó la solicitud |
| Provincia | La provincia, el estado o la localidad donde se originó la solicitud |
| Ciudad | La ciudad o el pueblo donde se originó la solicitud |
| isSynthetic | Indica si la solicitud la creó un usuario o un método automatizado. True = generada por el usuario o false = método automatizado |
| SamplingRate | Porcentaje de telemetría generado por el SDK que se envía al portal. Intervalo 0,0 a 100,0. |
| SampledCount | 100/(SamplingRate). Por ejemplo, 4 =&gt; 25 % |
| IsAuthenticated | Verdadero o falso |
| OperationID | Los elementos que tienen el mismo identificador de operación se muestran como elementos relacionados en el portal. Normalmente, el identificador de solicitud |
| ParentOperationID | El identificador de la operación principal |
| nombreOperación |   |
| SessionId | GUID para identificar de forma única la sesión en que se creó la solicitud |
| SourceSystem | ApplicationInsights |

### <a name="availability-specific-fields"></a>Campos específicos de disponibilidad

| Propiedad | DESCRIPCIÓN |
| --- | --- |
| TelemetryType | Disponibilidad |
| AvailabilityTestName | Nombre de la prueba web |
| AvailabilityRunLocation | Origen geográfico de la solicitud HTTP |
| AvailabilityResult | Indica el resultado correcto de la prueba web |
| AvailabilityMessage | El mensaje adjunto a la prueba web |
| AvailabilityCount | 100/(velocidad de muestreo). Por ejemplo, 4 =&gt; 25 % |
| DataSizeMetricValue | 1,0 o 0,0 |
| DataSizeMetricCount | 100/(velocidad de muestreo). Por ejemplo, 4 =&gt; 25 % |
| AvailabilityDuration | Tiempo, en milisegundos, de la duración de la prueba web |
| AvailabilityDurationCount | 100/(velocidad de muestreo). Por ejemplo, 4 =&gt; 25 % |
| AvailabilityValue |   |
| AvailabilityMetricCount |   |
| AvailabilityTestId | GUID único de la prueba web |
| AvailabilityTimestamp | Marca de tiempo precisa de la prueba de disponibilidad |
| AvailabilityDurationMin | En el caso de los registros muestreados, este campo muestra la duración mínima de la prueba web (en milisegundos) para los puntos de datos representados |
| AvailabilityDurationMax | En el caso de los registros muestreados, este campo muestra la duración máxima de la prueba web (en milisegundos) para los puntos de datos representados |
| AvailabilityDurationStdDev | En el caso de los registros muestreados, este campo muestra la desviación estándar entre las duraciones de todas las pruebas web (en milisegundos) para los puntos de datos representados |
| AvailabilityMin |   |
| AvailabilityMax |   |
| AvailabilityStdDev | &nbsp;  |

### <a name="exception-specific-fields"></a>Campos específicos de excepción

| Type | ApplicationInsights |
| --- | --- |
| TelemetryType | Excepción |
| ExceptionType | Tipo de la excepción |
| ExceptionMethod | El método que crea la excepción |
| ExceptionAssembly | El ensamblado incluye el marco y la versión, además del token de clave pública |
| ExceptionGroup | Tipo de la excepción |
| ExceptionHandledAt | Indica el nivel que controló la excepción |
| ExceptionCount | 100/(velocidad de muestreo). Por ejemplo, 4 =&gt; 25 % |
| ExceptionMessage | Mensaje de la excepción |
| ExceptionStack | Pila completa de la excepción |
| ExceptionHasStack | Verdadero, si la excepción tiene una pila |



### <a name="request-specific-fields"></a>Campos específicos de solicitud

| Propiedad | Descripción |
| --- | --- |
| Type | ApplicationInsights |
| TelemetryType | Solicitar |
| ResponseCode | Respuesta HTTP enviada al cliente |
| RequestSuccess | Indica una solicitud correcta o errónea. True o false. |
| RequestID | Identificador para identificar de forma única la solicitud |
| RequestName | GET/POST + dirección URL base |
| RequestDuration | Tiempo, en segundos, de la duración de la solicitud |
| Dirección URL | Dirección URL de la solicitud, sin incluir el host |
| Host | Host del servidor web |
| URLBase | Dirección URL completa de la solicitud |
| ApplicationProtocol | Tipo de protocolo que la aplicación usa |
| RequestCount | 100/(velocidad de muestreo). Por ejemplo, 4 =&gt; 25 % |
| RequestDurationCount | 100/(velocidad de muestreo). Por ejemplo, 4 =&gt; 25 % |
| RequestDurationMin | En el caso de los registros muestreados, este campo muestra la duración mínima de solicitud (en milisegundos) para los puntos de datos representados. |
| RequestDurationMax | En el caso de los registros muestreados, este campo muestra la duración máxima de solicitud (en milisegundos) para los puntos de datos representados. |
| RequestDurationStdDev | En el caso de los registros muestreados, este campo muestra la desviación estándar entre las duraciones de todas las solicitudes (en milisegundos) para los puntos de datos representados |

## <a name="sample-log-searches"></a>Búsquedas de registros de ejemplo

Esta solución no tiene un conjunto de búsquedas de registros de ejemplo que aparezca en el panel. Sin embargo, las consultas de búsquedas de registros de ejemplo con descripciones sí se muestran en la sección [Visualización de la información de Application Insights Connector](#view-application-insights-connector-information).

## <a name="removing-the-connector-with-powershell"></a>Eliminación del conector con PowerShell
Con el desuso del portal de OMS, no hay manera de configurar y quitar las conexiones existentes del portal. Puede quitar las conexiones existentes con el siguiente script de PowerShell. Debe poseer el rol de propietario o colaborador del área de trabajo y el de lector del recurso de Application Insights para llevar a cabo esta acción.

```powershell
$Subscription_app = "App Subscription Name"
$ResourceGroup_app = "App ResourceGroup"
$Application = "Application Name"
$Subscription_workspace = "Workspace Subscription Name"
$ResourceGroup_workspace = "Workspace ResourceGroup"
$Workspace = "Workspace Name"

Connect-AzAccount
Set-AzContext -SubscriptionId $Subscription_app
$AIApp = Get-AzApplicationInsights -ResourceGroupName $ResourceGroup_app -Name $Application 
Set-AzContext -SubscriptionId $Subscription_workspace
Remove-AzOperationalInsightsDataSource -WorkspaceName $Workspace -ResourceGroupName $ResourceGroup_workspace -Name $AIApp.Id
```

Puede recuperar una lista de aplicaciones mediante el siguiente script de PowerShell que invoca una llamada API REST. 

```powershell
Connect-AzAccount
$Tenant = "TenantId"
$Subscription_workspace = "Workspace Subscription Name"
$ResourceGroup_workspace = "Workspace ResourceGroup"
$Workspace = "Workspace Name"
$AccessToken = "AAD Authentication Token" 

Set-AzContext -SubscriptionId $Subscription_workspace
$LAWorkspace = Get-AzOperationalInsightsWorkspace -ResourceGroupName $ResourceGroup_workspace -Name $Workspace

$Headers = @{
    "Authorization" = "Bearer $($AccessToken)"
    "x-ms-client-tenant-id" = $Tenant
}

$Connections = Invoke-RestMethod -Method "GET" -Uri "https://management.azure.com$($LAWorkspace.ResourceId)/dataSources/?%24filter=kind%20eq%20'ApplicationInsights'&api-version=2015-11-01-preview" -Headers $Headers
$ConnectionsJson = $Connections | ConvertTo-Json
```
Este script requiere un token de autenticación de portador para la autenticación con Azure Active Directory. Una manera de recuperar este token es mediante un artículo que se encuentra en el [sitio de documentación de API REST](https://docs.microsoft.com/rest/api/loganalytics/datasources/createorupdate). Haga clic en **Probar** e inicie sesión en su suscripción de Azure. Puede copiar el token de portador de la **vista previa de la solicitud** tal como se muestra en la siguiente imagen.


![Token de portador](media/app-insights-connector/bearer-token.png)


También puede recuperar una lista de aplicaciones mediante una consulta de registro:

```Kusto
ApplicationInsights | summarize by ApplicationName
```

## <a name="next-steps"></a>Pasos siguientes

- Use [Búsqueda de registros](../../azure-monitor/log-query/log-query-overview.md) para ver información detallada de las aplicaciones de Application Insights.

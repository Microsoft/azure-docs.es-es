---
title: Mapa de aplicación en Azure Application Insights | Microsoft Docs
description: Supervisión de topologías de aplicaciones complejas con el mapa de aplicación
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 3bf37fe9-70d7-4229-98d6-4f624d256c36
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 03/15/2019
ms.reviewer: sdash
ms.author: mbullwin
ms.openlocfilehash: ba4643118c5d90b91c3e51d569e9a628c84159fc
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65780030"
---
# <a name="application-map-triage-distributed-applications"></a>Mapa de aplicación: Evaluación de prioridades de las aplicaciones distribuidas

El mapa de aplicación le ayuda a detectar los cuellos de botella en el rendimiento o las zonas activas con error en todos los componentes de la aplicación distribuida. Cada nodo del mapa representa un componente de aplicación o sus dependencias, e incluye el KPI de mantenimiento y el estado de alerta. Puede hacer clic a través de cualquier componente para ver un diagnóstico más detallado, como los eventos de Application Insights. Si su aplicación usa los servicios de Azure, también puede hacer clic por los diagnósticos de Azure, como las recomendaciones SQL Database Advisor.

## <a name="what-is-a-component"></a>¿Qué es un componente?

Los componentes son partes que se pueden implementar independientemente de su aplicación de microservicios o distribuida. Los equipos de operaciones y los desarrolladores pueden ver el código o acceder a la telemetría que generan estos componentes de la aplicación. 

* Los componentes son diferentes de las dependencias externas "observadas", como SQL, EventHub, etc. a las que su equipo u organización no pueden acceder (código o telemetría).
* Los componentes se ejecutan en cualquier número de instancias de rol, servidor o contenedor.
* Los componentes pueden ser claves de instrumentación de Application Insights independientes (incluso aunque las suscripciones sean diferentes) o diferentes roles que informan a una única clave de instrumentación de Application Insights. La experiencia de mapa de versión preliminar muestra los componentes con independencia de cómo están configurados.

## <a name="composite-application-map"></a>Mapa de aplicación compuesta

Puede ver la topología de aplicación completa a lo largo de varios niveles de componentes de aplicación relacionados. Los componentes podrían ser diferentes recursos de Application Insights o distintos roles de un único recurso. Para encontrar componentes, el mapa de aplicación sigue las llamadas de dependencia HTTP entre los servidores con el SDK de Application Insights instalado. 

Esta experiencia comienza con la detección progresiva de los componentes. Cuando se carga primero el mapa de aplicación, se desencadena un conjunto de consultas para detectar los componentes relacionados con este componente. Un botón en la esquina superior izquierda se actualiza con el número de componentes de la aplicación a medida que se detectan. 

Al hacer clic en "Update map components" (Actualizar componentes del mapa), el mapa se actualiza con todos los componentes detectados hasta ese momento. Según la complejidad de la aplicación, esta operación puede tardar un minuto en cargarse.

Si todos los componentes son roles dentro de un único recurso de Application Insights, este paso de detección no es necesario. La carga inicial para este tipo de aplicación tendrá todos sus componentes.

![Captura de pantalla de mapa de aplicación](media/app-map/app-map-001.png)

Uno de los objetivos principales de la experiencia es poder visualizar topologías complejas con cientos de componentes.

Haga clic en cada componente para ver información detallada relacionada e ir a la experiencia de evaluación de prioridades de rendimiento y errores de ese componente.

![Control flotante](media/app-map/application-map-002.png)

### <a name="investigate-failures"></a>Investigar los errores

Seleccione **Investigar los errores** para iniciar el panel de errores.

![Captura de pantalla del botón Investigar los errores](media/app-map/investigate-failures.png)

![Captura de pantalla de la experiencia de errores](media/app-map/failures.png)

### <a name="investigate-performance"></a>Investigar el rendimiento

Para solucionar problemas de rendimiento haga clic en **Investigar el rendimiento**.

![Captura de pantalla del botón Investigar el rendimiento](media/app-map/investigate-performance.png)

![Captura de pantalla de la experiencia de rendimiento](media/app-map/performance.png)

### <a name="go-to-details"></a>Ir a los detalles

Seleccione **vaya a detalles** para explorar la experiencia de transacción to-end, que puede ofrecer vistas hasta el nivel de la pila de llamadas.

![Captura de pantalla del botón Ir a los detalles](media/app-map/go-to-details.png)

![Captura de pantalla de los detalles de una transacción completa](media/app-map/end-to-end-transaction.png)

### <a name="view-in-analytics"></a>Ver en Analytics

Para consultar e investigar aún más los datos de aplicaciones, haga clic en **Ver en Analytics**.

![Captura de pantalla del botón Ver en Analytics](media/app-map/view-in-analytics.png)

![Captura de pantalla de la experiencia con Analytics](media/app-map/analytics.png)

### <a name="alerts"></a>Alertas

Para ver las alertas activas y las reglas subyacentes que provocan el desencadenamiento de estas, haga clic en **Alertas**.

![Captura de pantalla del botón de alertas](media/app-map/alerts.png)

![Captura de pantalla de la experiencia con Analytics](media/app-map/alerts-view.png)

## <a name="set-cloud-role-name"></a>Nombre del conjunto de roles en la nube

Mapa de aplicación usa el **nombre de rol en la nube** propiedad para identificar los componentes en el mapa. El SDK de Application Insights agrega automáticamente la propiedad de nombre de rol en la nube a la telemetría que se generen los componentes. Por ejemplo, el SDK agregará un nombre de rol de servicio o sitio web a la propiedad de nombre de rol en la nube. Pero hay casos en los que le interesará reemplazar el valor predeterminado. Para invalidar el nombre de rol en la nube y cambiar lo que se muestra en el mapa de aplicación:

### <a name="net"></a>.NET

```csharp
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.Extensibility;

namespace CustomInitializer.Telemetry
{
    public class MyTelemetryInitializer : ITelemetryInitializer
    {
        public void Initialize(ITelemetry telemetry)
        {
            if (string.IsNullOrEmpty(telemetry.Context.Cloud.RoleName))
            {
                //set custom role name here
                telemetry.Context.Cloud.RoleName = "Custom RoleName";
                telemetry.Context.Cloud.RoleInstance = "Custom RoleInstance";
            }
        }
    }
}
```

**Cargue su inicializador**

En ApplicationInsights.config:

```xml
    <ApplicationInsights>
      <TelemetryInitializers>
        <!-- Fully qualified type name, assembly name: -->
        <Add Type="CustomInitializer.Telemetry.MyTelemetryInitializer, CustomInitializer"/>
        ...
      </TelemetryInitializers>
    </ApplicationInsights>
```

Un método alternativo consiste en crear una instancia del inicializador en el código, por ejemplo en Global.aspx.cs:

```csharp
 using Microsoft.ApplicationInsights.Extensibility;
 using CustomInitializer.Telemetry;

    protected void Application_Start()
    {
        // ...
        TelemetryConfiguration.Active.TelemetryInitializers.Add(new MyTelemetryInitializer());
    }
```

### <a name="nodejs"></a>Node.js

```javascript
var appInsights = require("applicationinsights");
appInsights.setup('INSTRUMENTATION_KEY').start();
appInsights.defaultClient.context.tags["ai.cloud.role"] = "your role name";
appInsights.defaultClient.context.tags["ai.cloud.roleInstance"] = "your role instance";
```

### <a name="alternate-method-for-nodejs"></a>Método alternativo para Node.js

```javascript
var appInsights = require("applicationinsights");
appInsights.setup('INSTRUMENTATION_KEY').start();

appInsights.defaultClient.addTelemetryProcessor(envelope => {
    envelope.tags["ai.cloud.role"] = "your role name";
    envelope.tags["ai.cloud.roleInstance"] = "your role instance"
});
```

### <a name="java"></a>Java

Si usa Spring Boot con el iniciador de Spring Boot de Application Insights, el único cambio necesario es establecer el nombre personalizado para la aplicación en el archivo application.properties.

`spring.application.name=<name-of-app>`

El iniciador de Spring Boot asignará automáticamente el nombre de rol en la nube para el valor proporcionado para la propiedad spring.application.name.

Para obtener más información sobre Java correlación y cómo configurar el rol en la nube nombre para la extracción de las aplicaciones que no sean SpringBoot este [sección](https://docs.microsoft.com/azure/application-insights/application-insights-correlation#role-name) sobre la correlación.

### <a name="clientbrowser-side-javascript"></a>JavaScript del lado cliente o explorador

```javascript
appInsights.queue.push(() => {
appInsights.context.addTelemetryInitializer((envelope) => {
  envelope.tags["ai.cloud.role"] = "your role name";
  envelope.tags["ai.cloud.roleInstance"] = "your role instance";
});
});
```

### <a name="understanding-cloud-role-name-within-the-context-of-the-application-map"></a>Nombre de rol en la nube de comprensión dentro del contexto de la asignación de aplicaciones

En cuanto a cómo pensar sobre **nombre de rol en la nube**, puede ser útil ver un mapa de aplicación que tiene varios nombres de rol en la nube que presente:

![Captura de pantalla de mapa de aplicación](media/app-map/cloud-rolename.png)

En el mapa de aplicación encima de cada uno de los nombres de los cuadros verdes es nube valores de nombre de rol para los distintos aspectos de esta aplicación distribuida. Por lo que para esta aplicación, sus roles constan de: `Authentication`, `acmefrontend`, `Inventory Management`, un `Payment Processing Worker Role`. 

En el caso de esta aplicación de cada uno de esos nombres de rol en la nube también representa un único recurso de Application Insights diferente con sus propias claves de instrumentación. Puesto que el propietario de esta aplicación tiene acceso a cada uno de los cuatro recursos de Application Insights dispares, mapa de aplicación es capaz de unir un mapa de las relaciones subyacentes.

Para el [definiciones oficiales](https://github.com/Microsoft/ApplicationInsights-dotnet/blob/39a5ef23d834777eefdd72149de705a016eb06b0/Schema/PublicSchema/ContextTagKeys.bond#L93):

```
   [Description("Name of the role the application is a part of. Maps directly to the role name in azure.")]
    [MaxStringLength("256")]
    705: string      CloudRole = "ai.cloud.role";
    
    [Description("Name of the instance where the application is running. Computer name for on-premises, instance name for Azure.")]
    [MaxStringLength("256")]
    715: string      CloudRoleInstance = "ai.cloud.roleInstance";
```

Como alternativa, **instancia de rol en la nube** puede ser muy útil en escenarios donde **nombre de rol en la nube** indica el problema en algún lugar en el front-end web, pero podría estar ejecutándose el front-end a través de web varios con equilibrio de carga servidores tan permitirle profundizar en un nivel más profundo a través de las consultas de Kusto y saber si el problema afecta a todas las web front-end servidores o instancias, o solo uno puede ser muy importante.

Un escenario donde desea invalidar el valor de la instancia de rol en la nube podría ser si la aplicación se ejecuta en un entorno en contenedor que simplemente saber el servidor individual podría no ser suficiente información para localizar un problema determinado.

Para obtener más información acerca de cómo invalidar la propiedad de nombre de rol en la nube con los inicializadores de telemetría, consulte [agregar propiedades: ITelemetryInitializer](api-filtering-sampling.md#add-properties-itelemetryinitializer).

## <a name="troubleshooting"></a>solución de problemas

Si tiene dificultades para conseguir que el mapa de aplicación funcione según lo esperado, siga estos pasos:

### <a name="general"></a>General

1. Asegúrese de que usa un SDK oficialmente compatible. Es posible que los SDK no compatibles o de la comunidad no admitan la correlación.

    Consulte este [artículo](https://docs.microsoft.com/azure/application-insights/app-insights-platforms) para obtener una lista de los SDK compatibles.

2. Actualice todos los componentes a la versión más reciente del SDK.

3. Si usa Azure Functions con C#, actualice a [Functions V2](https://docs.microsoft.com/azure/azure-functions/functions-versions).

4. Confirmar [nombre de rol en la nube](#set-cloud-role-name) está configurado correctamente.

5. Si falta una dependencia, asegúrese de que se encuentra en la lista de [dependencias recopiladas automáticamente](https://docs.microsoft.com/azure/application-insights/auto-collect-dependencies). De lo contrario, todavía puede realizar su seguimiento de forma manual con una [llamada de seguimiento de dependencia](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics#trackdependency).

### <a name="too-many-nodes-on-the-map"></a>Hay demasiados nodos del mapa

Mapa de aplicación construye un nodo de aplicación para cada nombre de rol en la nube único presente en los datos de telemetría de solicitud y un nodo de dependencia para cada combinación única de tipo de destino y nombre de rol en la nube en la telemetría de dependencia. Si hay más de 10.000 nodos en los datos de telemetría, el mapa de aplicación no podrá capturar todos los nodos y vínculos, por lo que la asignación sea incompleta. Si esto ocurre, aparecerá un mensaje de advertencia al ver el mapa.

Además, el mapa de aplicación solo admite hasta 1000 nodos sin agrupar independientes representados a la vez. Mapa de aplicación reduce la complejidad visual agrupando las dependencias que tengan el mismo tipo y los autores de llamadas, pero si los datos de telemetría tiene demasiados nombres de rol en la nube único o demasiados tipos de dependencia, esa agrupación será suficiente y la asignación no se puede representar.

Para solucionar este problema, deberá cambiar la instrumentación para establecer correctamente el nombre de rol en la nube, tipo de dependencia y los campos de destino de dependencia.

* Destino de dependencia debe representar el nombre lógico de una dependencia. En muchos casos, es equivalente al servidor o el nombre de recurso de la dependencia. Por ejemplo, en el caso de las dependencias HTTP se establece en el nombre de host. No debe contener los identificadores únicos o parámetros que cambian de una solicitud a otro.

* Tipo de dependencia debe representar el tipo lógico de una dependencia. Por ejemplo, HTTP, SQL o Azure Blob son tipos de dependencia típica. No debe contener los identificadores únicos.

* El propósito de nombre de rol en la nube se describe en el [por encima de la sección](https://docs.microsoft.com/azure/azure-monitor/app/app-map#set-cloud-role-name).

## <a name="portal-feedback"></a>Comentarios del portal

Para proporcionar comentarios, use la opción de comentarios.

![Imagen MapLink-1](./media/app-map/14-updated.png)

## <a name="next-steps"></a>Pasos siguientes

* [Descripción de la correlación](https://docs.microsoft.com/azure/application-insights/application-insights-correlation)

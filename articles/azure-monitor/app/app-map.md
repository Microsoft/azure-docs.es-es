---
title: Mapa de aplicación en Azure Application Insights | Microsoft Docs
description: Supervisión de topologías de aplicaciones complejas con el mapa de aplicación
ms.topic: conceptual
ms.date: 03/15/2019
ms.reviewer: sdash
ms.openlocfilehash: b99998a7b1bcb2348a1a73696661de7cf8b44b85
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87421304"
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

Esta experiencia comienza con la detección progresiva de los componentes. La primera vez que carga el mapa de aplicación, se desencadena un conjunto de consultas para detectar los componentes relacionados con este componente. Un botón en la esquina superior izquierda se actualiza con el número de componentes de la aplicación a medida que se detectan. 

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

Haga clic en **Ir a los detalles** para explorar una experiencia de transacción completa, que puede ofrecer las vistas hasta el nivel de la pila de llamadas.

![Captura de pantalla del botón Ir a los detalles](media/app-map/go-to-details.png)

![Captura de pantalla de los detalles de una transacción completa](media/app-map/end-to-end-transaction.png)

### <a name="view-logs-analytics"></a>Visualización de Registros (Analytics)

Para consultar e investigar aún más los datos de aplicaciones, haga clic en **Ver en Registros (Analytics)** .

![Captura de pantalla del botón Ver en Analytics](media/app-map/view-logs.png)

![Captura de pantalla de la experiencia con Analytics. Gráfico de líneas que resume la duración media de respuesta de una solicitud en las últimas 12 horas.](media/app-map/log-analytics.png)

### <a name="alerts"></a>Alertas

Para ver las alertas activas y las reglas subyacentes que provocan el desencadenamiento de estas, haga clic en **Alertas**.

![Captura de pantalla del botón de alertas](media/app-map/alerts.png)

![Captura de pantalla de la experiencia con Analytics](media/app-map/alerts-view.png)

## <a name="set-or-override-cloud-role-name"></a>Establecimiento o reemplazo del nombre del rol en la nube

El mapa de aplicación usa la propiedad **nombre de rol en la nube** para identificar los componentes en el mapa. Para establecer o reemplazar manualmente el nombre de rol en la nube y cambiar lo que se muestra en el mapa de aplicación:

> [!NOTE]
> El Agente o SDK de Application Insights agregan de forma automática la propiedad de nombre de rol en la nube a la telemetría emitida por los componentes en un entorno de Azure App Service.

# <a name="netnetcore"></a>[.NET/.NetCore](#tab/net)

**Escriba un elemento TelemetryInitializer personalizado como el siguiente.**

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

**Aplicaciones ASP.NET: cargue el inicializador en el elemento TelemetryConfiguration activo.**

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

Un método alternativo para las aplicaciones web de ASP.NET consiste en crear una instancia del inicializador en el código, por ejemplo, en Global.aspx.cs:

```csharp
 using Microsoft.ApplicationInsights.Extensibility;
 using CustomInitializer.Telemetry;

    protected void Application_Start()
    {
        // ...
        TelemetryConfiguration.Active.TelemetryInitializers.Add(new MyTelemetryInitializer());
    }
```

> [!NOTE]
> La adición del inicializador mediante `ApplicationInsights.config` o `TelemetryConfiguration.Active` no es válida para las aplicaciones de ASP.NET Core. 

**Aplicaciones de ASP.NET Core: cargue el inicializador en el elemento TelemetryConfiguration.**

Para aplicaciones de [ASP.NET Core](asp-net-core.md#adding-telemetryinitializers), la adición de un nuevo elemento `TelemetryInitializer` se realiza agregándolo al contenedor de inserción de dependencias, como se muestra a continuación. Esto se hace en el método `ConfigureServices` de la clase `Startup.cs`.

```csharp
 using Microsoft.ApplicationInsights.Extensibility;
 using CustomInitializer.Telemetry;
 public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<ITelemetryInitializer, MyTelemetryInitializer>();
}
```

# <a name="java"></a>[Java](#tab/java)

**Agente de Java**

Para [Agente de Java 3.0](./java-in-process-agent.md) el nombre del rol en la nube se establece de la manera siguiente:

```json
{
  "instrumentationSettings": {
    "preview": {
      "roleName": "my cloud role name"
    }
  }
}
```

También puede establecer el nombre de rol en la nube mediante la variable de entorno ```APPLICATIONINSIGHTS_ROLE_NAME```.

**SDK de Java**

Si utiliza el SDK, a partir del SDK de Java 2.5.0 para Application Insights, puede especificar el nombre del rol en la nube si agrega `<RoleName>` al archivo `ApplicationInsights.xml`, por ejemplo,

```XML
<?xml version="1.0" encoding="utf-8"?>
<ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings" schemaVersion="2014-05-30">
   <InstrumentationKey>** Your instrumentation key **</InstrumentationKey>
   <RoleName>** Your role name **</RoleName>
   ...
</ApplicationInsights>
```

Si usa Spring Boot con el iniciador de Spring Boot de Application Insights, el único cambio necesario es establecer el nombre personalizado para la aplicación en el archivo application.properties.

`spring.application.name=<name-of-app>`

El iniciador de Spring Boot asignará de forma automática el nombre de rol en la nube al valor proporcionado para la propiedad spring.application.name.

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

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

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
appInsights.queue.push(() => {
appInsights.addTelemetryInitializer((envelope) => {
  envelope.tags["ai.cloud.role"] = "your role name";
  envelope.tags["ai.cloud.roleInstance"] = "your role instance";
});
});
```
---

### <a name="understanding-cloud-role-name-within-the-context-of-the-application-map"></a>Comprensión del nombre de rol en la nube dentro del contexto del mapa de aplicación

Para comprender el **nombre de rol en la nube**, puede ser útil consultar un mapa de aplicación que tenga presentes varios nombres de rol en la nube:

![Captura de pantalla de mapa de aplicación](media/app-map/cloud-rolename.png)

En el mapa de aplicación de arriba, cada uno de los nombres de los cuadros verdes son valores de nombre de rol en la nube para distintos aspectos de esta aplicación distribuida. Por lo que, para esta aplicación, sus roles constan de: `Authentication`, `acmefrontend`, `Inventory Management` y `Payment Processing Worker Role`. 

En el caso de esta aplicación, cada uno de esos nombres de rol en la nube también representa un único recurso de Application Insights diferente con sus propias claves de instrumentación. Puesto que el propietario de esta aplicación tiene acceso a cada uno de estos cuatro recursos de Application Insights, el mapa de aplicación es capaz de unir un mapa de las relaciones subyacentes.

Para las [definiciones oficiales](https://github.com/Microsoft/ApplicationInsights-dotnet/blob/39a5ef23d834777eefdd72149de705a016eb06b0/Schema/PublicSchema/ContextTagKeys.bond#L93):

```
   [Description("Name of the role the application is a part of. Maps directly to the role name in azure.")]
    [MaxStringLength("256")]
    705: string      CloudRole = "ai.cloud.role";
    
    [Description("Name of the instance where the application is running. Computer name for on-premises, instance name for Azure.")]
    [MaxStringLength("256")]
    715: string      CloudRoleInstance = "ai.cloud.roleInstance";
```

Como alternativa, la **instancia de rol en la nube** puede ser muy útil para escenarios donde el **nombre de rol en la nube** indica que el problema se encuentra en algún lugar en el front-end web, pero puede que ejecute el front-end web en varios servidores de carga equilibrada, por lo que poder profundizar a un nivel más profundo a través de las consultas de Kusto y saber si el problema afecta a todos los servidores o instancias de front-end web o solo a uno puede ser muy importante.

Un escenario en el que podría querer reemplazar el valor de la instancia de rol en la nube podría ser si la aplicación se ejecutara en un entorno en contenedor donde simplemente conocer el servidor individual no sería suficiente información para localizar un problema determinado.

Para obtener más información sobre cómo reemplazar la propiedad de nombre de rol en la nube con inicializadores de telemetría, vea [Agregar propiedades: ITelemetryInitializer](api-filtering-sampling.md#addmodify-properties-itelemetryinitializer).

## <a name="troubleshooting"></a>Solución de problemas

Si tiene dificultades para conseguir que el mapa de aplicación funcione según lo esperado, siga estos pasos:

### <a name="general"></a>General

1. Asegúrese de que usa un SDK oficialmente compatible. Es posible que los SDK no compatibles o de la comunidad no admitan la correlación.

    Consulte este [artículo](./platforms.md) para obtener una lista de los SDK compatibles.

2. Actualice todos los componentes a la versión más reciente del SDK.

3. Si usa Azure Functions con C#, actualice a [Functions V2](../../azure-functions/functions-versions.md).

4. Confirme que el [nombre de rol en la nube](#set-or-override-cloud-role-name) está configurado correctamente.

5. Si falta una dependencia, asegúrese de que se encuentra en la lista de [dependencias recopiladas automáticamente](./auto-collect-dependencies.md). De lo contrario, todavía puede realizar su seguimiento de forma manual con una [llamada de seguimiento de dependencia](./api-custom-events-metrics.md#trackdependency).

### <a name="too-many-nodes-on-the-map"></a>Hay demasiados nodos en el mapa

El mapa de aplicación construye un nodo de aplicación para cada nombre de rol en la nube único presente en la telemetría de solicitud y un nodo de dependencia para cada combinación única de tipo, destino y nombre de rol en la nube en la telemetría de dependencia. Si hay más de 10 000 nodos en la telemetría, el mapa de aplicación no podrá capturar todos los nodos y vínculos, por lo que la asignación será incompleta. Si ocurre esto, aparecerá un mensaje de advertencia al visualizar el mapa.

Además, el mapa de aplicación solo admite hasta 1000 nodos sin agrupar independientes representados a la vez. El mapa de aplicación reduce la complejidad visual agrupando las dependencias que tengan el mismo tipo y llamadores, pero si la telemetría tiene demasiados nombres de rol en la nube únicos o demasiados tipos de dependencia, esa agrupación será insuficiente y el mapa no se podrá representar.

Para solucionar este problema, deberá cambiar la instrumentación para establecer correctamente el nombre de rol en la nube, el tipo de dependencia y los campos de destino de dependencia.

* El destino de dependencia debe representar el nombre lógico de una dependencia. En muchos casos, es equivalente al nombre del servidor o del recurso de la dependencia. Por ejemplo, en el caso de las dependencias HTTP se establece en el nombre de host. No debe contener id. exclusivos o parámetros que cambian de una solicitud a otra.

* El tipo de dependencia debe representar el tipo lógico de una dependencia. Por ejemplo, HTTP, SQL o Azure Blob son tipos de dependencia típicos. No debe contener id. exclusivos.

* El propósito de nombre de rol en la nube se describe en la [sección anterior](#set-or-override-cloud-role-name).

## <a name="portal-feedback"></a>Comentarios del portal

Para proporcionar comentarios, use la opción de comentarios.

![Imagen MapLink-1](./media/app-map/14-updated.png)

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre cómo funciona la correlación en Application Insights, consulte el [artículo sobre la correlación de telemetría](correlation.md).
* La [experiencia de diagnóstico de transacción de un extremo a otro](transaction-diagnostics.md) correlaciona la telemetría del lado servidor con todos los componentes supervisados de Application Insights en una única vista.
* En el caso de escenarios avanzados de correlación en ASP.NET Core y ASP.NET, consulte el artículo sobre [seguimiento de operaciones personalizadas](custom-operations-tracking.md).
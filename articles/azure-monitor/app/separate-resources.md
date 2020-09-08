---
title: 'Diseño de la implementación de Application Insights: uno frente a muchos recursos'
description: Este artículo trata sobre el envío directo de la telemetría a los diferentes recursos para los sellos de desarrollo, prueba y producción.
ms.topic: conceptual
ms.date: 05/11/2020
ms.openlocfilehash: 264cbe35e7af50577b345d686b639e47760f428d
ms.sourcegitcommit: ef055468d1cb0de4433e1403d6617fede7f5d00e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/16/2020
ms.locfileid: "88258714"
---
# <a name="how-many-application-insights-resources-should-i-deploy"></a>¿Cuántos recursos de Application Insights se deben implementar?

Cuando esté desarrollando la próxima versión de una aplicación web, no querrá mezclar la telemetría de [Application Insights](../../azure-monitor/app/app-insights-overview.md) de la nueva versión con la que ya se ha publicado. Para evitar confusiones, envíe la telemetría de las distintas fases de desarrollo para separar los recursos de Application Insights con claves de instrumentación independientes (iKey). Para que sea más fácil cambiar la clave de instrumentación cuando pase una versión de una fase a otro, puede ser útil establecer la clave iKey en el código en lugar de en el archivo de configuración.

(Si el sistema es un servicio en la nube de Azure, hay [otra forma de configurar claves separadas](../../azure-monitor/app/cloudservices.md)).

## <a name="about-resources-and-instrumentation-keys"></a>Acerca de los recursos y las claves de instrumentación

Al configurar la supervisión de Application Insights para su aplicación web, se crea un *recurso* de Application Insights en Microsoft Azure. Abra este recurso en Azure Portal con el fin de ver y analizar la telemetría recopilada de la aplicación. Cada recurso se identifica con una *clave de instrumentación* (iKey). Cuando se instala el paquete de Application Insights para supervisar la aplicación, configúrelo con la clave de instrumentación para que sepa dónde enviar la telemetría.

Cada recurso de Application Insights incluye métricas disponibles de serie. En caso de que componentes totalmente independientes notifican al mismo recurso de Application Insights, puede que estas métricas no tengan sentido para el panel o la alerta.

### <a name="when-to-use-a-single-application-insights-resource"></a>Cuándo usar un único recurso de Application Insights

-   En componentes de aplicaciones que se han implementado juntos. Normalmente han sido desarrollados por un solo equipo, administrados por el mismo conjunto de usuarios de DevOps/ITOps.
-   Si tiene sentido agregar indicadores clave de rendimiento (KPI), como las duraciones de las respuestas, las tasas de error en el panel, etc., en todos ellos de forma predeterminada (puede optar por segmentar por nombre de rol en la experiencia del Explorador de métricas).
-   Si no hay necesidad de administrar el control de acceso basado en rol (RBAC) de manera diferente entre los componentes de aplicaciones.
-   Si no necesita criterios de alerta de métricas diferentes entre los componentes.
-   Si no necesita administrar exportaciones continuas de forma distinta entre los componentes.
-   Si no necesita administrar facturaciones o cuotas de forma distinta entre los componentes.
-   Si es correcto tener una clave de API con el mismo acceso a los datos de todos los componentes. Diez claves de API son suficientes para las necesidades en todos ellos.
-   Si es correcto tener la misma detección inteligente y la misma configuración de integración de elementos de trabajo en todos los roles.

### <a name="other-things-to-keep-in-mind"></a>Otros aspectos que se deben tener en cuenta

-   Es posible que tenga que agregar código personalizado para asegurarse de que se establezcan valores significativos en el atributo [Cloud_RoleName](./app-map.md?tabs=net#set-or-override-cloud-role-name). Sin valores significativos establecidos para este atributo, *NINGUNA* de las experiencias del portal va a funcionar.
- En el caso de las aplicaciones de Service Fabric y los servicios en la nube clásicos, el SDK lee automáticamente del entorno de roles de Azure y los establece. En todos los demás tipos de aplicaciones, es probable que tenga que establecer esto de forma explícita.
-   La experiencia de Live Metrics no admite la división por nombre de rol.

## <a name="dynamic-instrumentation-key"></a><a name="dynamic-ikey"></a> Copia de la clave de instrumentación

Para que sea más fácil cambiar el valor de iKey cuando el código pasa por las fases de producción, referencie la clave de forma dinámica en el código, en lugar de utilizar un valor estático o que se haya codificado de forma rígida.

Establezca la clave en un método de inicialización, como global.aspx.cs en un servicio de ASP.NET:

```csharp
protected void Application_Start()
{
  Microsoft.ApplicationInsights.Extensibility.
    TelemetryConfiguration.Active.InstrumentationKey = 
      // - for example -
      WebConfigurationManager.AppSettings["ikey"];
  ...
```

En este ejemplo, las ikeys para los distintos recursos se colocan en diferentes versiones del archivo de configuración web. Si cambia el archivo de configuración web (algo que se puede hacer como parte del script de lanzamiento), se intercambiará el recurso de destino.

### <a name="web-pages"></a>Páginas web
La clave de instrumentación también se usa en las páginas web de la aplicación, en el [script obtenido del panel de inicio rápido](../../azure-monitor/app/javascript.md). En vez de codificarla literalmente en el script, genérela desde el estado del servidor. Por ejemplo, en una aplicación ASP.NET:

```javascript
<script type="text/javascript">
// Standard Application Insights web page script:
var appInsights = window.appInsights || function(config){ ...
// Modify this part:
}({instrumentationKey:  
  // Generate from server property:
  "@Microsoft.ApplicationInsights.Extensibility.
     TelemetryConfiguration.Active.InstrumentationKey"
  }
 )
//...
```

## <a name="create-additional-application-insights-resources"></a>Creación de recurso de Application Insights adicionales

Para crear un recurso de Application Insights, siga la [guía de creación de recursos](./create-new-resource.md).

### <a name="getting-the-instrumentation-key"></a>Obtención de la clave de instrumentación
La clave de instrumentación identifica al recurso que ha creado.

Necesita las claves de instrumentación de todos los recursos a los que la aplicación enviará datos.

## <a name="filter-on-build-number"></a>Filtrado por número de compilación
Cuando se publica una nueva versión de la aplicación, querrá poder separar la telemetría en las diferentes versiones.

Puede establecer la propiedad de versión de la aplicación para filtrar los resultados de la [búsqueda](../../azure-monitor/app/diagnostic-search.md) y del [explorador de métricas](../../azure-monitor/platform/metrics-charts.md).

Hay diferentes métodos de establecer la propiedad de versión de la aplicación.

* Establezca directamente:

    `telemetryClient.Context.Component.Version = typeof(MyProject.MyClass).Assembly.GetName().Version;`
* Ajuste esa línea en un [inicializador de telemetría](../../azure-monitor/app/api-custom-events-metrics.md#defaults) para asegurarse de que todas las instancias de TelemetryClient se establecen de forma coherente.
* [ASP.NET] Establezca la versión en `BuildInfo.config`. El módulo web recogerá la versión del nodo BuildLabel. Incluya este archivo en el proyecto y recuerde que establecer la propiedad Copiar siempre en el Explorador de soluciones.

    ```XML
    <?xml version="1.0" encoding="utf-8"?>
    <DeploymentEvent xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns="http://schemas.microsoft.com/VisualStudio/DeploymentEvent/2013/06">
      <ProjectName>AppVersionExpt</ProjectName>
      <Build type="MSBuild">
        <MSBuild>
          <BuildLabel kind="label">1.0.0.2</BuildLabel>
        </MSBuild>
      </Build>
    </DeploymentEvent>

    ```
* [ASP.NET] Genere BuildInfo.config automáticamente en MSBuild. Para ello, agregue unas líneas a su archivo `.csproj`:

    ```XML
    <PropertyGroup>
      <GenerateBuildInfoConfigFile>true</GenerateBuildInfoConfigFile>    <IncludeServerNameInBuildInfo>true</IncludeServerNameInBuildInfo>
    </PropertyGroup>
    ```

    Esto genera un archivo denominado *suNombreProyecto*.BuildInfo.config. El proceso de publicación cambia su nombre a BuildInfo.config.

    La etiqueta de compilación contiene un marcador de posición (AutoGen_...) al compilar con Visual Studio. Pero cuando se crea con MSBuild, se rellena con el número de versión correcto.

    Para permitir que MSBuild genere números de versión, establezca la versión como `1.0.*` en AssemblyReference.cs.

## <a name="version-and-release-tracking"></a>Versión y seguimiento de versiones
Para realizar el seguimiento de la versión de la aplicación, asegúrese de que `buildinfo.config` lo genera el proceso de Microsoft Build Engine. En el archivo `.csproj`, agregue lo siguiente:  

```XML
<PropertyGroup>
  <GenerateBuildInfoConfigFile>true</GenerateBuildInfoConfigFile>
  <IncludeServerNameInBuildInfo>true</IncludeServerNameInBuildInfo>
</PropertyGroup>
```

Cuando tenga la información de la compilación, el módulo web de Application Insights agregará automáticamente la **versión de la aplicación** como una propiedad a cada elemento de telemetría. Esto le permite filtrar por versión al realizar [búsquedas de diagnósticos](../../azure-monitor/app/diagnostic-search.md) o al [explorar métricas](../../azure-monitor/platform/metrics-charts.md).

Tenga en cuenta, sin embargo, que el número de versión de la compilación solo lo genera Microsoft Build Engine, no la compilación de desarrollador de Visual Studio.

### <a name="release-annotations"></a>Anotaciones de la versión
Si usa Azure DevOps, puede [obtener un marcador de anotación](../../azure-monitor/app/annotations.md) agregado a los gráficos, siempre que publique una nueva versión. 

## <a name="next-steps"></a>Pasos siguientes

* [Recursos compartidos para varios roles](../../azure-monitor/app/app-map.md)
* [Creación de un inicializador de telemetría para distinguir variantes A/B](../../azure-monitor/app/api-filtering-sampling.md#add-properties)
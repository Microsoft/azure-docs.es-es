---
title: Supervisión de un sitio de SharePoint con Application Insights
description: Inicio de la supervisión de una nueva aplicación con una nueva clave de instrumentación
ms.topic: conceptual
ms.date: 07/11/2018
ms.openlocfilehash: 392c0e0ee46e8acd540d498cbda1d240611d182e
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87326535"
---
# <a name="monitor-a-sharepoint-site-with-application-insights"></a>Supervisión de un sitio de SharePoint con Application Insights

Azure Application Insights le permite supervisar la disponibilidad, el rendimiento y el uso de sus aplicaciones. Aquí aprenderá a configurarlo para un sitio de SharePoint.

> [!NOTE]
> Debido a algunos problemas de seguridad, no puede agregar directamente el script que se describe en este artículo a sus páginas web de la experiencia de usuario moderna de SharePoint. También puede usar [el marco de SharePoint (SPFx)](/sharepoint/dev/spfx/extensions/overview-extensions) para crear una extensión personalizada que puede usar para instalar Application Insights en los sitios de SharePoint. Para más información, consulte [Creación de una solución de extensión de SPFx con AppInsights instalado desde cero](https://github.com/microsoft/ApplicationInsights-JS/tree/master/SPO#how-to-create-a-spfx-extension-solution-with-appinsights-installed-from-scratch) o [consulte el ejemplo](https://github.com/microsoft/ApplicationInsights-JS/tree/master/SPO/AppInsightsExtensionSolutionSample). 

## <a name="create-an-application-insights-resource"></a>Creación de recursos en Application Insights
En el [portal de Azure](https://portal.azure.com), cree un nuevo recurso de Application Insights. Elija ASP.NET como el tipo de aplicación.

![Haga clic en Propiedades, seleccione la clave y presione ctrl + C.](./media/sharepoint/001.png)

En la ventana que se abre podrá ver los datos de uso y rendimiento acerca de la aplicación. Para volver a ella la próxima vez que inicie sesión en Azure, encontrará un icono correspondiente en la pantalla de inicio. También puede hacer clic en Examinar para buscarla.

## <a name="add-the-script-to-your-web-pages"></a>Incorporación del script a sus páginas web

```HTML
<!-- 
To collect user behavior analytics tools about your application, 
insert the following script into each page you want to track.
Place this code immediately before the closing </head> tag,
and before any other scripts. Your first data will appear 
automatically in just a few seconds.
-->
<script type="text/javascript">
var sdkInstance="appInsightsSDK";window[sdkInstance]="appInsights";var aiName=window[sdkInstance],aisdk=window[aiName]||function(n){var o={config:n,initialize:!0},t=document,e=window,i="script";setTimeout(function(){var e=t.createElement(i);e.src=n.url||"https://az416426.vo.msecnd.net/scripts/b/ai.2.min.js",t.getElementsByTagName(i)[0].parentNode.appendChild(e)});try{o.cookie=t.cookie}catch(e){}function a(n){o[n]=function(){var e=arguments;o.queue.push(function(){o[n].apply(o,e)})}}o.queue=[],o.version=2;for(var s=["Event","PageView","Exception","Trace","DependencyData","Metric","PageViewPerformance"];s.length;)a("track"+s.pop());var r="Track",c=r+"Page";a("start"+c),a("stop"+c);var u=r+"Event";if(a("start"+u),a("stop"+u),a("addTelemetryInitializer"),a("setAuthenticatedUserContext"),a("clearAuthenticatedUserContext"),a("flush"),o.SeverityLevel={Verbose:0,Information:1,Warning:2,Error:3,Critical:4},!(!0===n.disableExceptionTracking||n.extensionConfig&&n.extensionConfig.ApplicationInsightsAnalytics&&!0===n.extensionConfig.ApplicationInsightsAnalytics.disableExceptionTracking)){a("_"+(s="onerror"));var p=e[s];e[s]=function(e,n,t,i,a){var r=p&&p(e,n,t,i,a);return!0!==r&&o["_"+s]({message:e,url:n,lineNumber:t,columnNumber:i,error:a}),r},n.autoExceptionInstrumented=!0}return o}(
{
  instrumentationKey:"INSTRUMENTATION_KEY"
}
);(window[aiName]=aisdk).queue&&0===aisdk.queue.length&&aisdk.trackPageView({});
</script>
```

Inserte el script justo antes de la etiqueta &lt;/head&gt; de cada página que desea seguir. Si su sitio web tiene una página maestra, puede colocar el script allí. Por ejemplo, en un proyecto de ASP.NET MVC, lo colocaría en View\Shared\\_Layout.cshtml.

El script contiene la clave de instrumentación que dirige los datos de telemetría al recurso de Application Insights.

### <a name="add-the-code-to-your-site-pages"></a>Agregar el código a las páginas del sitio
#### <a name="on-the-master-page"></a>En la página maestra
Si se puede editar la página maestra del sitio, dispondrá de supervisión para cada página del sitio.

Desproteja y edite la página maestra mediante SharePoint Designer o cualquier otro editor.

![Captura de pantalla que muestra cómo editar la página maestra mediante SharePoint Designer u otro editor.](./media/sharepoint/03-master.png)

Agregue el código justo antes de la etiqueta </head> . 

![Captura de pantalla que muestra dónde agregar el código a la página del sitio.](./media/sharepoint/04-code.png)

#### <a name="or-on-individual-pages"></a>O, en páginas individuales
Para supervisar un conjunto limitado de páginas, agregue el script por separado a cada página. 

Inserte un elemento web e incruste el fragmento de código en él.

![Captura de pantalla que muestra cómo agregar el script para supervisar un conjunto limitado de páginas.](./media/sharepoint/05-page.png)

## <a name="view-data-about-your-app"></a>Visualización de los datos de la aplicación
Vuelva a implementar la aplicación.

Vuelva a la hoja de la aplicación en el [Portal de Azure](https://portal.azure.com).

Los primeros eventos aparecerán en la búsqueda. 

![Captura de pantalla que muestra los nuevos datos que se pueden ver en la aplicación.](./media/sharepoint/09-search.png)

Si espera más datos, haga clic en Actualizar después de unos segundos.

## <a name="capturing-user-id"></a>Captura del identificador de usuario
El fragmento de código de una página web estándar no captura el identificador de usuario de SharePoint, pero una pequeña modificación le permitirá hacerlo.

1. Copie la clave de instrumentación de la aplicación de la lista desplegable Essentials en Application Insights . 

    ![Captura de pantalla que muestra la copia de la instrumentación de la aplicación desde la lista desplegable de Essentials en Application Insights.](./media/sharepoint/02-props.png)

1. En el siguiente fragmento, sustituya la clave de instrumentación por "XXXX". 
2. Inserte el script en la aplicación de SharePoint en lugar del fragmento de código que obtenga desde el portal.

```


<SharePoint:ScriptLink ID="ScriptLink1" name="SP.js" runat="server" localizable="false" loadafterui="true" /> 
<SharePoint:ScriptLink ID="ScriptLink2" name="SP.UserProfiles.js" runat="server" localizable="false" loadafterui="true" /> 

<script type="text/javascript"> 
var personProperties; 

// Ensure that the SP.UserProfiles.js file is loaded before the custom code runs. 
SP.SOD.executeOrDelayUntilScriptLoaded(getUserProperties, 'SP.UserProfiles.js'); 

function getUserProperties() { 
    // Get the current client context and PeopleManager instance. 
    var clientContext = new SP.ClientContext.get_current(); 
    var peopleManager = new SP.UserProfiles.PeopleManager(clientContext); 

    // Get user properties for the target user. 
    // To get the PersonProperties object for the current user, use the 
    // getMyProperties method. 

    personProperties = peopleManager.getMyProperties(); 

    // Load the PersonProperties object and send the request. 
    clientContext.load(personProperties); 
    clientContext.executeQueryAsync(onRequestSuccess, onRequestFail); 
} 

// This function runs if the executeQueryAsync call succeeds. 
function onRequestSuccess() { 
var appInsights=window.appInsights||function(config){
function s(config){t[config]=function(){var i=arguments;t.queue.push(function(){t[config].apply(t,i)})}}var t={config:config},r=document,f=window,e="script",o=r.createElement(e),i,u;for(o.src=config.url||"//az416426.vo.msecnd.net/scripts/a/ai.0.js",r.getElementsByTagName(e)[0].parentNode.appendChild(o),t.cookie=r.cookie,t.queue=[],i=["Event","Exception","Metric","PageView","Trace"];i.length;)s("track"+i.pop());return config.disableExceptionTracking||(i="onerror",s("_"+i),u=f[i],f[i]=function(config,r,f,e,o){var s=u&&u(config,r,f,e,o);return s!==!0&&t["_"+i](config,r,f,e,o),s}),t
    }({
        instrumentationKey:"XXXX"
    });
    window.appInsights=appInsights;
    appInsights.trackPageView(document.title,window.location.href, {User: personProperties.get_displayName()});
} 

// This function runs if the executeQueryAsync call fails. 
function onRequestFail(sender, args) { 
} 
</script> 


```



## <a name="next-steps"></a>Pasos siguientes
* [Pruebas web](./monitor-web-app-availability.md) para supervisar la disponibilidad de su sitio.
* [Application Insights](./app-insights-overview.md) para otros tipos de aplicación.

<!--Link references-->


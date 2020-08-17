---
title: Colección de direcciones IP de Azure Application Insights | Microsoft Docs
description: Información acerca de cómo controlar las direcciones IP y la ubicación geográfica con Azure Application Insights
ms.topic: conceptual
ms.date: 09/11/2019
ms.custom: devx-track-javascript
ms.openlocfilehash: 28a7fa50a06dc8b80c7d8dd284cd88ebe4645da6
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87371658"
---
# <a name="geolocation-and-ip-address-handling"></a>Administración de la ubicación geográfica y la dirección IP

En este artículo se explica cómo realizar la búsqueda de geolocalización y la administración de direcciones IP en Application Insights, además de cómo modificar el comportamiento predeterminado.

## <a name="default-behavior"></a>Comportamiento predeterminado

De forma predeterminada, las direcciones IP se recopilan temporalmente, pero no se almacenan en Application Insights. El proceso básico es el siguiente:

Las direcciones IP se envían a Application Insights como parte de los datos de telemetría. Al llegar al punto de conexión de ingesta en Azure, la dirección IP se usa para realizar una búsqueda de geolocalización mediante [GeoLite2 de MaxMind](https://dev.maxmind.com/geoip/geoip2/geolite2/). Los resultados de esta búsqueda se usan para completar los campos `client_City`, `client_StateOrProvince`, `client_CountryOrRegion`. En este punto, la dirección IP se descarta y `0.0.0.0` se escribe en el campo `client_IP`.

* Telemetría del explorador: recopilamos temporalmente la dirección IP del remitente. La dirección IP se calcula mediante el punto de conexión de ingesta.
* Telemetría del servidor: el módulo de Application Insights recopila temporalmente la dirección IP del cliente. No se recopila si `X-Forwarded-For` está establecido.

Este comportamiento es así de forma predeterminada, ya que le ayudará a evitar la recopilación innecesaria de datos personales. Siempre que sea posible, se recomienda evitar la recopilación de datos personales. 

## <a name="overriding-default-behavior"></a>Invalidar el comportamiento predeterminado

Aunque el comportamiento predeterminado debe minimizar la recopilación de datos personales, todavía ofrecemos la opción para recopilar y almacenar datos de direcciones IP. Antes de decidir si quiere almacenar datos personales como direcciones IP, le recomendamos que compruebe que esta acción no infrinja los requisitos de cumplimiento o la normativa local a los que pueda estar sujeto. Para obtener más información sobre la administración de datos personales en Application Insights, consulte la [guía de datos personales](../platform/personal-data-mgmt.md).

## <a name="storing-ip-address-data"></a>Almacenamiento de datos de direcciones IP

Para habilitar la recopilación y el almacenamiento de direcciones IP, la propiedad `DisableIpMasking` del componente de Application Insights debe establecerse en `true`. Esta propiedad se puede establecer a través de las plantillas de Azure Resource Manager o llamando a la API de REST. 

### <a name="azure-resource-manager-template"></a>Plantilla de Azure Resource Manager

```json
{
       "id": "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/microsoft.insights/components/<resource-name>",
       "name": "<resource-name>",
       "type": "microsoft.insights/components",
       "location": "westcentralus",
       "tags": {
              
       },
       "kind": "web",
       "properties": {
              "Application_Type": "web",
              "Flow_Type": "Redfield",
              "Request_Source": "IbizaAIExtension",
              // ...
              "DisableIpMasking": true
       }
}
```

### <a name="portal"></a>Portal 

Si solo necesita modificar el comportamiento de un solo recurso de Application Insights, la forma más fácil de lograrlo es a través de Azure Portal.  

1. Vaya al recurso de Application Insights > **Configuración** > **Exportar plantilla**. 

    ![Exportación de la plantilla](media/ip-collection/export-template.png)

2. Seleccione **Implementar**.

    ![Botón de implementación resaltado en rojo](media/ip-collection/deploy.png)

3. Seleccione **Editar plantilla**. (Si su plantilla tiene propiedades o recursos adicionales que no aparecen en esta plantilla de ejemplo, proceda con precaución para asegurarse de que todos los recursos acepten la implementación de la plantilla como un cambio o actualización incremental).

    ![Editar plantilla](media/ip-collection/edit-template.png)

4. Realice los siguientes cambios en el archivo JSON del recurso y haga clic en **Guardar**:

    ![La captura de pantalla agrega una coma después de "IbizaAIExtension" y agrega una nueva línea a continuación con "DisableIpMasking": true.](media/ip-collection/save.png)

    > [!WARNING]
    > Si experimenta un error que dice: **_El grupo de recursos está en una ubicación que no es compatible con uno o varios recursos de la plantilla. Elija otro grupo de recursos._** Seleccione temporalmente otro grupo de recursos en la lista desplegable y, a continuación, vuelva a seleccionar el grupo de recursos original para resolver el error.

5. Seleccione **Acepto** > **Comprar**. 

    ![Editar plantilla](media/ip-collection/purchase.png)

    En este caso no se compra nada nuevo, simplemente estamos actualizando la configuración del recurso de Application Insights existente.

6. Una vez completada la implementación, se registrarán los nuevos datos de telemetría.

    Si tuviera que seleccionar y editar la plantilla nuevamente, solo vería la plantilla predeterminada y no la propiedad recién agregada y su valor asociado. Si no ve los datos de la dirección IP y quiere confirmar que `"DisableIpMasking": true` está configurado. Ejecute el siguiente código de PowerShell: (Reemplace `Fabrikam-dev` con el recurso apropiado y el nombre del grupo de recursos).
    
    ```powershell
    # If you aren't using the cloud shell you will need to connect to your Azure account
    # Connect-AzAccount 
    $AppInsights = Get-AzResource -Name 'Fabrikam-dev' -ResourceType 'microsoft.insights/components' -ResourceGroupName 'Fabrikam-dev'
    $AppInsights.Properties
    ```
    
    Como resultado, se devolverá una lista de propiedades. Una de las propiedades debe ser `DisableIpMasking: true`. Si ejecuta PowerShell antes de implementar la nueva propiedad con Azure Resource Manager, la propiedad no existirá.

### <a name="rest-api"></a>API de REST

La carga útil de la [API de Rest](/rest/api/azure/) para realizar las mismas modificaciones es la siguiente:

```
PATCH https://management.azure.com/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/microsoft.insights/components/<resource-name>?api-version=2018-05-01-preview HTTP/1.1
Host: management.azure.com
Authorization: AUTH_TOKEN
Content-Type: application/json
Content-Length: 54

{
       "location": "<resource location>",
       "kind": "web",
       "properties": {
              "Application_Type": "web",
              "DisableIpMasking": true
       }
}
```

## <a name="telemetry-initializer"></a>Inicializador de telemetría

Si necesita una alternativa más flexible que `DisableIpMasking` para registrar todo o parte de las direcciones IP, puede usar un [inicializador de telemetría](./api-filtering-sampling.md#addmodify-properties-itelemetryinitializer) para copiar todo o parte de la dirección IP en un campo personalizado. 

### <a name="aspnet--aspnet-core"></a>ASP.NET/ASP.NET Core

```csharp
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.DataContracts;
using Microsoft.ApplicationInsights.Extensibility;

namespace MyWebApp
{
    public class CloneIPAddress : ITelemetryInitializer
    {
        public void Initialize(ITelemetry telemetry)
        {
            ISupportProperties propTelemetry = telemetry as ISupportProperties;

            if (propTelemetry !=null && !propTelemetry.Properties.ContainsKey("client-ip"))
            {
                string clientIPValue = telemetry.Context.Location.Ip;
                propTelemetry.Properties.Add("client-ip", clientIPValue);
            }
        }
    } 
}
```

> [!NOTE]
> Si no puede acceder a `ISupportProperties`, asegúrese de que ejecuta la versión estable más reciente del SDK de Application Insights. `ISupportProperties` están pensadas para valores de cardinalidad alta, mientras que `GlobalProperties` son más adecuadas para valores de cardinalidad baja, como el nombre de la región, el nombre del entorno, etc. 

### <a name="enable-telemetry-initializer-for-aspnet"></a>Habilitar el inicializador de telemetría para .ASP.NET

```csharp
using Microsoft.ApplicationInsights.Extensibility;


namespace MyWebApp
{
    public class MvcApplication : System.Web.HttpApplication
    {
        protected void Application_Start()
        {
              //Enable your telemetry initializer:
              TelemetryConfiguration.Active.TelemetryInitializers.Add(new CloneIPAddress());
        }
    }
}

```

### <a name="enable-telemetry-initializer-for-aspnet-core"></a>Habilitar el inicializador de telemetría para ASP.NET Core

Puede crear su inicializador de telemetría de la misma manera en ASP.NET Core y en ASP.NET, pero para habilitar el inicializador, use el siguiente ejemplo como referencia:

```csharp
 using Microsoft.ApplicationInsights.Extensibility;
 using CustomInitializer.Telemetry;
 public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<ITelemetryInitializer, CloneIPAddress>();
}
```

### <a name="nodejs"></a>Node.js

```javascript
appInsights.defaultClient.addTelemetryProcessor((envelope) => {
    const baseData = envelope.data.baseData;
    if (appInsights.Contracts.domainSupportsProperties(baseData)) {
        const ipAddress = envelope.tags[appInsights.defaultClient.context.keys.locationIp];
        if (ipAddress) {
            baseData.properties["client-ip"] = ipAddress;
        }
    }
});
```

### <a name="client-side-javascript"></a>JavaScript del lado cliente

A diferencia de los SDK del lado del servidor, el SDK de Javascript del lado cliente no calcula la dirección IP. De forma predeterminada, el cálculo de la dirección IP para la telemetría del lado cliente se realiza en el punto de conexión de ingesta de Azure al llegar la telemetría. Esto significa que si envía datos del lado cliente a un proxy y luego los reenvía al punto de conexión de ingesta, el cálculo de la dirección IP puede mostrar la dirección IP del proxy y no del cliente. Si no se usa un proxy, esto no debería ser un problema.

Si quiere calcular la dirección IP directamente en el lado cliente, necesitará agregar su propia lógica personalizada para realizar este cálculo y usar el resultado para establecer la etiqueta `ai.location.ip`. Cuando se establece `ai.location.ip`, el punto de conexión de ingesta no realiza el cálculo de la dirección IP, y la dirección IP proporcionada se respeta y se usa para realizar la búsqueda geográfica. En este caso, la dirección IP seguirá estableciéndose en cero de forma predeterminada. 

Para retener la dirección IP completa calculada a partir de su lógica personalizada, puede usar un inicializador de telemetría que copie los datos de la dirección IP que proporcionó en `ai.location.ip` en un campo personalizado separado. Pero, de nuevo, a diferencia de los SDK del lado servidor, sin depender de bibliotecas de terceros o de su propia lógica de recopilación de IP del lado cliente, el SDK del lado cliente no calculará la IP.    


```javascript
appInsights.addTelemetryInitializer((item) => {
    const ipAddress = item.tags && item.tags["ai.location.ip"];
    if (ipAddress) {
        item.baseData.properties = {
            ...item.baseData.properties,
            "client-ip": ipAddress
        };
    }
});

```  

### <a name="view-the-results-of-your-telemetry-initializer"></a>Ver los resultados del inicializador de telemetría

Si a continuación desencadena un nuevo proceso de tráfico en su sitio y espera aproximadamente de 2 a 5 minutos para asegurarse de que haya tiempo para la ingesta, puede ejecutar una consulta de Kusto para ver si la recopilación de direcciones IP funciona:

```kusto
requests
| where timestamp > ago(1h) 
| project appName, operation_Name, url, resultCode, client_IP, customDimensions.["client-ip"]
```

Las direcciones IP recién recopiladas deben aparecer en la columna `customDimensions_client-ip`. La columna predeterminada `client-ip` todavía tendrá los 4 octetos establecidos en cero o solo mostrará los primeros tres octetos, dependiendo de cómo haya configurado la recopilación de direcciones IP en el nivel de componente. Si está realizando pruebas de forma local después de implementar el inicializador de telemetría y el valor que ve para `customDimensions_client-ip` es `::1`, este es el comportamiento esperado. `::1` representa la dirección de bucle invertido en IPv6. Es equivalente a `127.0.01` en IPv4 y es el resultado que verá cuando realice pruebas desde localhost.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre la [recopilación de datos personales](../platform/personal-data-mgmt.md) en Application Insights.

* Obtenga más información sobre cómo funciona la [recopilación de direcciones IP](https://apmtips.com/posts/2016-07-05-client-ip-address/) en Application Insights. (Esta es una publicación de blog externa anterior que escribió uno de nuestros ingenieros. Es anterior al comportamiento predeterminado actual donde la dirección IP se registra como `0.0.0.0`, pero se profundiza en la mecánica del elemento `ClientIpHeaderTelemetryInitializer` integrado).

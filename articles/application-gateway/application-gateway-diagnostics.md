---
title: Registros de mantenimiento y diagnóstico de back-end
titleSuffix: Azure Application Gateway
description: Obtenga información acerca de cómo habilitar y administrar registros de acceso y registros de rendimiento para Azure Application Gateway.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: article
ms.date: 11/22/2019
ms.author: victorh
ms.openlocfilehash: f752604b86634948954dd670d0b7f4edb5b3e2be
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86517882"
---
# <a name="back-end-health-and-diagnostic-logs-for-application-gateway"></a>Mantenimiento del back-end y registros de diagnóstico para Application Gateway

Puede supervisar los recursos de Azure Application Gateway de las maneras siguientes:

* [Mantenimiento del back-end](#back-end-health): Application Gateway permite supervisar el mantenimiento de los servidores en los grupos de back-end a través de Azure Portal y de PowerShell. El mantenimiento de los grupos de back-end también se puede encontrar en los registros de diagnóstico de rendimiento.

* [Registros](#diagnostic-logging): los registros permiten que un recurso guarde o consuma datos de rendimiento, acceso o de otro tipo con fines de supervisión.

* [Métricas](application-gateway-metrics.md): Application Gateway tiene varias métricas que le ayudan a comprobar que el sistema funciona según lo previsto.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="back-end-health"></a>Mantenimiento del back-end

Application Gateway permite supervisar el mantenimiento de cada uno de los miembros de los grupos de back-end mediante el portal, PowerShell y la interfaz de línea de comandos (CLI). El resumen acumulado del mantenimiento de los grupos de back-end también se puede encontrar en los registros de diagnóstico de rendimiento.

El informe de mantenimiento del back-end refleja la salida del sondeo de mantenimiento de Application Gateway en las instancias de back-end. Cuando el sondeo es correcto y el back-end puede recibir tráfico, se considera que su mantenimiento es correcto. En caso contrario, se considera incorrecto.

> [!IMPORTANT]
> Si hay un grupo de seguridad de red (NSG) en una subred Application Gateway, abra los intervalos de puertos 65503-65534 para las SKU v1 y 65200-65535 para las SKU de v2 en la subred Application Gateway para el tráfico entrante. Este intervalo de puertos es necesario para la comunicación de la infraestructura de Azure. Están protegidos (bloqueados) mediante certificados de Azure. Sin los certificados apropiados, las entidades externas, incluidos los clientes de esas puertas de enlace, no podrán iniciar ningún cambio en esos puntos de conexión.


### <a name="view-back-end-health-through-the-portal"></a>Visualización del mantenimiento del back-end mediante el portal

En el portal, el mantenimiento del back-end se proporciona automáticamente. En una puerta de enlace de aplicaciones existente, vaya a **Supervisión** > **Estado del back-end**.

Cada miembro en el grupo de back-end se muestra en esta página (ya sea una NIC, dirección IP o FQDN). Se muestran el nombre del grupo de back-end, el puerto, la configuración HTTP del back-end y el estado de mantenimiento. Los valores válidos para el estado de mantenimiento son **Correcto**, **Incorrecto** y **Desconocido**.

> [!NOTE]
> Si ve un estado de mantenimiento de back-end **Desconocido**, asegúrese de que el acceso al back-end no está bloqueado por una regla de NSG, una ruta definida por el usuario (UDR) o un DNS personalizado en la red virtual.

![Mantenimiento del back-end][10]

### <a name="view-back-end-health-through-powershell"></a>Visualización del mantenimiento del back-end mediante PowerShell

El código de PowerShell siguiente muestra cómo extraer el mantenimiento del back-end con el cmdlet `Get-AzApplicationGatewayBackendHealth`:

```powershell
Get-AzApplicationGatewayBackendHealth -Name ApplicationGateway1 -ResourceGroupName Contoso
```

### <a name="view-back-end-health-through-azure-cli"></a>Visualización del mantenimiento del back-end mediante la CLI de Azure

```azurecli
az network application-gateway show-backend-health --resource-group AdatumAppGatewayRG --name AdatumAppGateway
```

### <a name="results"></a>Results

El siguiente fragmento de código muestra un ejemplo de la respuesta:

```json
{
"BackendAddressPool": {
    "Id": "/subscriptions/00000000-0000-0000-000000000000/resourceGroups/ContosoRG/providers/Microsoft.Network/applicationGateways/applicationGateway1/backendAddressPools/appGatewayBackendPool"
},
"BackendHttpSettingsCollection": [
    {
    "BackendHttpSettings": {
        "Id": "/00000000-0000-0000-000000000000/resourceGroups/ContosoRG/providers/Microsoft.Network/applicationGateways/applicationGateway1/backendHttpSettingsCollection/appGatewayBackendHttpSettings"
    },
    "Servers": [
        {
        "Address": "hostname.westus.cloudapp.azure.com",
        "Health": "Healthy"
        },
        {
        "Address": "hostname.westus.cloudapp.azure.com",
        "Health": "Healthy"
        }
    ]
    }
]
}
```

## <a name="diagnostic-logs"></a><a name="diagnostic-logging"></a>Registros de diagnóstico

Puede usar diferentes tipos de registros en Azure para administrar y solucionar problemas de Application Gateway. Se puede acceder a algunos de estos registros mediante el portal. Se pueden extraer todos los registros de Azure Blob Storage y visualizarse en distintas herramientas, como los [registros de Azure Monitor](../azure-monitor/insights/azure-networking-analytics.md), Excel y PowerBI. Puede obtener más información sobre los diferentes tipos de registros en la lista siguiente:

* **Registro de actividades**: se puede usar el [registro de actividades de Azure](../monitoring-and-diagnostics/insights-debugging-with-events.md) (anteriormente conocido como registros operativos y registros de auditoría) para ver todas las operaciones enviadas a la suscripción de Azure, así como su estado. Las entradas del registro de actividades se recopilan de forma predeterminada y se pueden ver en Azure Portal.
* **Registro de acceso**: Puede usar este registro para ver los patrones de acceso de Application Gateway y analizar información importante. Esto incluye la dirección IP del autor de la llamada, la dirección URL solicitada, la latencia de la respuesta, el código de devolución y los bytes de entrada y salida. El registro de acceso se recopila cada 60 segundos. Este registro contiene un registro por cada instancia de Application Gateway. La instancia de Application Gateway se identifica por la propiedad instanceId.
* **Registro de rendimiento**: este registro se puede usar para ver el rendimiento de las instancias de Application Gateway. Este registro captura la información de rendimiento de cada instancia, incluida la cantidad total de solicitudes atendidas, el rendimiento en bytes, la cantidad de solicitudes con error y el número de instancias de back-end con un mantenimiento correcto o incorrecto. El registro de rendimiento se recopila cada 60 segundos. El registro de rendimiento solo está disponible para la SKU v1. En la SKU v2, use [Métricas](application-gateway-metrics.md) para los datos de rendimiento.
* **Registro de firewall**: este registro se puede usar para ver las solicitudes que se registran con el modo de detección o prevención de una puerta de enlace de aplicaciones que está configurada con el firewall de aplicaciones web. Los registros de firewall se recopilan cada 60 segundos. 

> [!NOTE]
> Los registros solo están disponibles para los recursos implementados en el modelo de implementación de Azure Resource Manager. No puede usar los registros de recursos del modelo de implementación clásica. Para entender mejor los dos modelos, consulte el artículo [Descripción de la implementación de Resource Manager y la implementación clásica](../azure-resource-manager/management/deployment-models.md) .

Tiene tres opciones para almacenar los archivos de registro:

* **Cuenta de almacenamiento**: cuentas que resultan especialmente útiles para registros cuando estos se almacenan durante mucho tiempo y se revisan cuando es necesario.
* **Centros de eventos**: Centro de eventos es una buena opción para la integración con otras herramientas de administración de eventos e información de seguridad (SIEM) para obtener alertas sobre los recursos.
* **Registros de Azure Monitor**: se usan para la supervisión general en tiempo real de la aplicación o para examinar las tendencias.

### <a name="enable-logging-through-powershell"></a>Habilitación del registro con PowerShell

El registro de actividades se habilita automáticamente para todos los recursos de Resource Manager. Debe habilitar el registro de acceso y rendimiento para iniciar la recopilación de los datos disponibles a través de esos registros. Para habilitar el registro, realice los siguientes pasos:

1. Anote el identificador de recurso de la cuenta de almacenamiento donde se almacenan los datos de registro. Este valor tiene el formato: /subscriptions/\<subscriptionId\>/resourceGroups/\<resource group name\>/providers/Microsoft.Storage/storageAccounts/\<storage account name\>. Puede usar cualquier cuenta de almacenamiento de la suscripción. Para buscar esta información, se puede usar Azure Portal.

    ![Portal: identificador de recurso de la cuenta de almacenamiento](./media/application-gateway-diagnostics/diagnostics1.png)

2. Observe el identificador de recurso de la puerta de enlace de aplicaciones para la que se está habilitando el registro. Este valor tiene el formato: /subscriptions/\<subscriptionId\>/resourceGroups/\<resource group name\>/providers/Microsoft.Network/applicationGateways/\<application gateway name\>. Para buscar esta información, use Azure Portal.

    ![Portal: identificador de recurso de la puerta de enlace de aplicaciones](./media/application-gateway-diagnostics/diagnostics2.png)

3. Habilite el registro de diagnósticos mediante el siguiente cmdlet de PowerShell:

    ```powershell
    Set-AzDiagnosticSetting  -ResourceId /subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/Microsoft.Network/applicationGateways/<application gateway name> -StorageAccountId /subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/Microsoft.Storage/storageAccounts/<storage account name> -Enabled $true     
    ```

> [!TIP]
>Los registros de actividades no requieren una cuenta de almacenamiento separada. El uso del almacenamiento para el registro de acceso y rendimiento supondrá un costo adicional de servicio.

### <a name="enable-logging-through-the-azure-portal"></a>Habilitación del registro mediante Azure Portal

1. En Azure Portal, busque el recurso y seleccione **Configuración de diagnóstico**.

   Hay tres registros de auditoría disponibles para Application Gateway:

   * Registro de acceso
   * Registro de rendimiento
   * Registro de firewall

2. Para empezar a recopilar los datos, haga clic en **Activar diagnóstico**.

   ![Activación de los diagnósticos][1]

3. En la página **Configuración de diagnóstico**, se encuentran las opciones de configuración de los registros de diagnóstico. En este ejemplo, se utiliza Log Analytics para almacenar los registros. Se pueden utilizar también Events Hubs y la cuenta de almacenamiento para guardar los registros de diagnóstico.

   ![Inicio del proceso de configuración][2]

5. Escriba un nombre para la configuración, confírmelo y seleccione **Guardar**.

### <a name="activity-log"></a>Registro de actividades

Azure genera el registro de actividad de forma predeterminada. Los registros se conservan durante 90 días en el almacén de registros de eventos de Azure. Para más información sobre estos registros, consulte el artículo [Visualización de eventos y registros de actividades](../monitoring-and-diagnostics/insights-debugging-with-events.md).

### <a name="access-log"></a>Registro de acceso

El registro de acceso solo se genera si lo habilitó para cada instancia de Application Gateway, tal y como se indicó en los pasos anteriores. Los datos se almacenan en la cuenta de almacenamiento que especificó cuando habilitó el registro. Cada acceso de Application Gateway se registra en formato JSON, tal y como se muestra a continuación. 

#### <a name="for-application-gateway-standard-and-waf-sku-v1"></a>Para la SKU Estándar de Application Gateway y de WAF (v1)

|Value  |Descripción  |
|---------|---------|
|instanceId     | Instancia de Application Gateway que atendió la solicitud.        |
|clientIP     | IP de origen de la solicitud.        |
|clientPort     | Puerto de origen de la solicitud.       |
|httpMethod     | Método HTTP utilizado por la solicitud.       |
|requestUri     | URI de la solicitud recibida.        |
|RequestQuery     | **Server-Routed**: instancia del grupo de back-end a la que se ha enviado la solicitud.</br>**X-AzureApplicationGateway-LOG-ID**: identificador de correlación que se ha usado para la solicitud. Se puede utilizar para solucionar problemas de tráfico en los servidores back-end. </br>**SERVER-STATUS**: código de respuesta HTTP que Application Gateway ha recibido del back-end.       |
|UserAgent     | Agente de usuario del encabezado de solicitud HTTP.        |
|httpStatus     | Código de estado HTTP que se devuelve al cliente desde Application Gateway.       |
|HttpVersion     | Versión HTTP de la solicitud.        |
|receivedBytes     | Tamaño de paquete recibido, en bytes.        |
|sentBytes| Tamaño de paquete enviado, en bytes.|
|timeTaken| Período de tiempo (en milisegundos) que se tarda en procesar una solicitud y en enviar la respuesta. Esto se calcula como el intervalo desde el momento en que Application Gateway recibe el primer byte de una solicitud HTTP hasta el momento en que termina la operación de envío de la respuesta. Es importante tener en cuenta que el campo Time-Taken normalmente incluye la hora a la que los paquetes de solicitud y respuesta se desplazan a través de la red. |
|sslEnabled| Indica si la comunicación con los grupos de back-end utilizaron TLS/SSL. Los valores válidos son on y off.|
|host| El nombre de host con el que se envió la solicitud al servidor back-end. Si se está reemplazando el nombre de host de back-end, se reflejará en este nombre.|
|originalHost| Nombre de host con el que Application Gateway recibió la solicitud del cliente.|
```json
{
    "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/PEERINGTEST/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/{applicationGatewayName}",
    "operationName": "ApplicationGatewayAccess",
    "time": "2017-04-26T19:27:38Z",
    "category": "ApplicationGatewayAccessLog",
    "properties": {
        "instanceId": "ApplicationGatewayRole_IN_0",
        "clientIP": "191.96.249.97",
        "clientPort": 46886,
        "httpMethod": "GET",
        "requestUri": "/phpmyadmin/scripts/setup.php",
        "requestQuery": "X-AzureApplicationGateway-CACHE-HIT=0&SERVER-ROUTED=10.4.0.4&X-AzureApplicationGateway-LOG-ID=874f1f0f-6807-41c9-b7bc-f3cfa74aa0b1&SERVER-STATUS=404",
        "userAgent": "-",
        "httpStatus": 404,
        "httpVersion": "HTTP/1.0",
        "receivedBytes": 65,
        "sentBytes": 553,
        "timeTaken": 205,
        "sslEnabled": "off",
        "host": "www.contoso.com",
        "originalHost": "www.contoso.com"
    }
}
```
#### <a name="for-application-gateway-and-waf-v2-sku"></a>Para la SKU de Application Gateway y WAF v2

|Value  |Descripción  |
|---------|---------|
|instanceId     | Instancia de Application Gateway que atendió la solicitud.        |
|clientIP     | IP de origen de la solicitud.        |
|clientPort     | Puerto de origen de la solicitud.       |
|httpMethod     | Método HTTP utilizado por la solicitud.       |
|requestUri     | URI de la solicitud recibida.        |
|UserAgent     | Agente de usuario del encabezado de solicitud HTTP.        |
|httpStatus     | Código de estado HTTP que se devuelve al cliente desde Application Gateway.       |
|HttpVersion     | Versión HTTP de la solicitud.        |
|receivedBytes     | Tamaño de paquete recibido, en bytes.        |
|sentBytes| Tamaño de paquete enviado, en bytes.|
|timeTaken| Período de tiempo (en **segundos**) que se tarda en procesar una solicitud y en enviar la respuesta. Esto se calcula como el intervalo desde el momento en que Application Gateway recibe el primer byte de una solicitud HTTP hasta el momento en que termina la operación de envío de la respuesta. Es importante tener en cuenta que el campo Time-Taken normalmente incluye la hora a la que los paquetes de solicitud y respuesta se desplazan a través de la red. |
|sslEnabled| Indica si la comunicación con los grupos de back-end utilizaron TLS. Los valores válidos son on y off.|
|sslCipher| Conjunto de cifrado que se usa para la comunicación TLS (si TLS está habilitado).|
|sslProtocol| Protocolo SSL/TLS que se usa (si se ha habilitado TLS).|
|serverRouted| Servidor back-end al que Application Gateway redirige la solicitud.|
|serverStatus| Código de estado HTTP del servidor back-end.|
|serverResponseLatency| Latencia de la respuesta del servidor back-end.|
|host| Dirección que aparece en el encabezado de host de la solicitud. Si se reescribe, este campo contiene el nombre de host actualizado|
|originalRequestUriWithArgs| Este campo contiene la dirección URL de la solicitud original |
|requestUri| Este campo contiene la dirección URL posterior a la operación de reescritura en Application Gateway |
|originalHost| Este campo contiene el nombre de host de la solicitud original
```json
{
    "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/PEERINGTEST/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/{applicationGatewayName}",
    "operationName": "ApplicationGatewayAccess",
    "time": "2017-04-26T19:27:38Z",
    "category": "ApplicationGatewayAccessLog",
    "properties": {
        "instanceId": "appgw_1",
        "clientIP": "191.96.249.97",
        "clientPort": 46886,
        "httpMethod": "GET",
        "requestUri": "/phpmyadmin/scripts/setup.php",
        "userAgent": "-",
        "httpStatus": 404,
        "httpVersion": "HTTP/1.0",
        "receivedBytes": 65,
        "sentBytes": 553,
        "timeTaken": 205,
        "sslEnabled": "off",
        "sslCipher": "",
        "sslProtocol": "",
        "serverRouted": "104.41.114.59:80",
        "serverStatus": "200",
        "serverResponseLatency": "0.023",
        "host": "www.contoso.com",
    }
}
```

### <a name="performance-log"></a>Registro de rendimiento

El registro de rendimiento solo se genera si lo habilitó para cada instancia de Application Gateway, tal y como se indicó en los pasos anteriores. Los datos se almacenan en la cuenta de almacenamiento que especificó cuando habilitó el registro. Los datos de registro de rendimiento se generan en intervalos de 1 minuto. Solo está disponible para la SKU v1. En la SKU v2, use [Métricas](application-gateway-metrics.md) para los datos de rendimiento. Se registran los datos siguientes:


|Value  |Descripción  |
|---------|---------|
|instanceId     |  Instancia de Application Gateway para la que se van a generar los datos. Si hay varias instancias de Application Gateway, hay una fila por cada instancia.        |
|healthyHostCount     | Número de hosts con un mantenimiento correcto en el grupo de back-end.        |
|unHealthyHostCount     | Número de hosts con un mantenimiento incorrecto en el grupo de back-end.        |
|requestCount     | Número de solicitudes atendidas.        |
|latency | Latencia media (en milisegundos) de las solicitudes desde la instancia hasta el back-end que atiende las solicitudes. |
|failedRequestCount| Número de solicitudes con error.|
|throughput| Rendimiento medio desde el último registro, medido en bytes por segundo.|

```json
{
    "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/{applicationGatewayName}",
    "operationName": "ApplicationGatewayPerformance",
    "time": "2016-04-09T00:00:00Z",
    "category": "ApplicationGatewayPerformanceLog",
    "properties":
    {
        "instanceId":"ApplicationGatewayRole_IN_1",
        "healthyHostCount":"4",
        "unHealthyHostCount":"0",
        "requestCount":"185",
        "latency":"0",
        "failedRequestCount":"0",
        "throughput":"119427"
    }
}
```

> [!NOTE]
> La latencia se calcula desde el momento en que se recibe el primer byte de la solicitud HTTP hasta el momento en que se envía el último byte de la respuesta HTTP. Es la suma del tiempo de procesamiento de Application Gateway más el costo de la red hasta el back-end, más el tiempo que el back-end tarda en procesar la solicitud.

### <a name="firewall-log"></a>Registro de firewall

El registro de firewall solo se genera si lo habilitó para cada instancia de Application Gateway, tal y como se indicó en los pasos anteriores. Este registro también requiere que el firewall de aplicaciones web esté configurado en una instancia de Application Gateway. Los datos se almacenan en la cuenta de almacenamiento que especificó cuando habilitó el registro. Se registran los datos siguientes:


|Value  |Descripción  |
|---------|---------|
|instanceId     | Instancia de Application Gateway para la que se van a generar los datos de firewall. Si hay varias instancias de Application Gateway, hay una fila por cada instancia.         |
|clientIp     |   IP de origen de la solicitud.      |
|clientPort     |  Puerto de origen de la solicitud.       |
|requestUri     | URI de la solicitud recibida.       |
|ruleSetType     | Tipo de conjunto de reglas. El valor disponible es OWASP.        |
|ruleSetVersion     | Versión utilizada del conjunto de reglas. Los valores disponibles son 2.2.9 y 3.0.     |
|ruleId     | Identificador de regla del evento desencadenador.        |
|message     | Mensaje descriptivo para el evento desencadenador. En la sección de detalles se proporciona más información.        |
|action     |  Acción realizada en la solicitud. Los valores disponibles Coincidente y Bloqueado.      |
|site     | Sitio para el que se generó el registro. Actualmente, solo se incluye Global porque las reglas son globales.|
|detalles     | Detalles del evento desencadenador.        |
|details.message     | Descripción de la regla.        |
|details.data     | Datos específicos que se encuentran en la solicitud y que corresponden a la regla.         |
|details.file     | Archivo de configuración que contiene la regla.        |
|details.line     | Número de línea del archivo de configuración que desencadenó el evento.       |
|hostname   | Nombre de host o dirección IP de la puerta de enlace de aplicaciones.    |
|transactionId  | Identificador único de una transacción determinada que ayuda a agrupar varias infracciones de reglas que se produjeron dentro de la misma solicitud.   |

```json
{
  "resourceId": "/SUBSCRIPTIONS/{subscriptionId}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/APPLICATIONGATEWAYS/{applicationGatewayName}",
  "operationName": "ApplicationGatewayFirewall",
  "time": "2017-03-20T15:52:09.1494499Z",
  "category": "ApplicationGatewayFirewallLog",
  "properties": {
    "instanceId": "ApplicationGatewayRole_IN_0",
    "clientIp": "104.210.252.3",
    "clientPort": "4835",
    "requestUri": "/?a=%3Cscript%3Ealert(%22Hello%22);%3C/script%3E",
    "ruleSetType": "OWASP",
    "ruleSetVersion": "3.0",
    "ruleId": "941320",
    "message": "Possible XSS Attack Detected - HTML Tag Handler",
    "action": "Blocked",
    "site": "Global",
    "details": {
      "message": "Warning. Pattern match \"<(a|abbr|acronym|address|applet|area|audioscope|b|base|basefront|bdo|bgsound|big|blackface|blink|blockquote|body|bq|br|button|caption|center|cite|code|col|colgroup|comment|dd|del|dfn|dir|div|dl|dt|em|embed|fieldset|fn|font|form|frame|frameset|h1|head|h ...\" at ARGS:a.",
      "data": "Matched Data: <script> found within ARGS:a: <script>alert(\\x22hello\\x22);</script>",
      "file": "rules/REQUEST-941-APPLICATION-ATTACK-XSS.conf",
      "line": "865"
    }
    "hostname": "40.90.218.100",
    "transactionId": "AYAcUqAcAcAcAcAcASAcAcAc"
  }
}

```

### <a name="view-and-analyze-the-activity-log"></a>Visualización y análisis del registro de actividades

Puede ver y analizar los datos del registro de actividades con cualquiera de los métodos siguientes:

* **Herramientas de Azure**: permiten recuperar información de los registros de actividades mediante Azure PowerShell, la CLI de Azure, la API REST de Azure o Azure Portal. En el artículo [Operaciones de actividades con Resource Manager](../azure-resource-manager/management/view-activity-logs.md) se detallan instrucciones paso a paso de cada método.
* **Power BI**: si todavía no tiene una cuenta de [Power BI](https://powerbi.microsoft.com/pricing), puede probarlo gratis. Las [aplicaciones de plantilla de Power BI](https://docs.microsoft.com/power-bi/service-template-apps-overview) permiten analizar los datos.

### <a name="view-and-analyze-the-access-performance-and-firewall-logs"></a>Visualización y análisis de los registros de acceso, rendimiento y firewall

Los [registros de Azure Monitor](../azure-monitor/insights/azure-networking-analytics.md) pueden recopilar los archivos de registro de eventos y contadores de la cuenta de Blob Storage. Incluye visualizaciones y eficaces funciones de búsqueda para analizar los registros.

También puede conectarse a la cuenta de almacenamiento y recuperar las entradas del registro de JSON de los registros de acceso y rendimiento. Después de descargar los archivos JSON, se pueden convertir a CSV y consultarlos en Excel, PowerBI o cualquier otra herramienta de visualización de datos.

> [!TIP]
> Si está familiarizado con Visual Studio y con los conceptos básicos de cambio de los valores de constantes y variables de C#, puede usar las [herramientas convertidoras de registros](https://github.com/Azure-Samples/networking-dotnet-log-converter) que encontrará en GitHub.
>
>

#### <a name="analyzing-access-logs-through-goaccess"></a>Analizar los registros de acceso mediante GoAccess

Hemos publicado una plantilla de Resource Manager que se instala y ejecuta el popular analizador de registros [GoAccess](https://goaccess.io/) para los registros de acceso de Application Gateway. GoAccess proporciona valiosas estadísticas de tráfico HTTP como visitantes únicos, archivos solicitados, hosts, sistemas operativos, exploradores, códigos de estado HTTP y mucho más. Para obtener más información, consulte el [archivo Léame en la carpeta de plantillas de Resource Manager en GitHub](https://aka.ms/appgwgoaccessreadme).

## <a name="next-steps"></a>Pasos siguientes

* Visualice el contador y los registros de eventos mediante los [registros de Azure Monitor](../azure-monitor/insights/azure-networking-analytics.md).
* Consulte la entrada de blog [Visualize your Azure Activity Logs with Power BI](https://powerbi.microsoft.com/blog/monitor-azure-audit-logs-with-power-bi/) (Visualizar los registros de actividades de Azure con Power BI).
* Consulte la entrada de blog [View and analyze Azure Audit Logs in Power BI and more](https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/) (Consulta y análisis de registros de auditoría de Azure en Power BI y más).

[1]: ./media/application-gateway-diagnostics/figure1.png
[2]: ./media/application-gateway-diagnostics/figure2.png
[3]: ./media/application-gateway-diagnostics/figure3.png
[4]: ./media/application-gateway-diagnostics/figure4.png
[5]: ./media/application-gateway-diagnostics/figure5.png
[6]: ./media/application-gateway-diagnostics/figure6.png
[7]: ./media/application-gateway-diagnostics/figure7.png
[8]: ./media/application-gateway-diagnostics/figure8.png
[9]: ./media/application-gateway-diagnostics/figure9.png
[10]: ./media/application-gateway-diagnostics/figure10.png

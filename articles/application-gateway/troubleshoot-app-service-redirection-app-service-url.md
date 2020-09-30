---
title: Solución de problemas de redireccionamiento a la dirección URL de App Service
titleSuffix: Azure Application Gateway
description: En este artículo se proporciona información sobre cómo solucionar el problema de redireccionamiento cuando se utiliza Azure Application Gateway con Azure App Service
services: application-gateway
author: abshamsft
ms.service: application-gateway
ms.topic: troubleshooting
ms.date: 11/14/2019
ms.author: absha
ms.openlocfilehash: f3a3ba3ee908204668ad9d7201ddfddec0a26f28
ms.sourcegitcommit: 1b320bc7863707a07e98644fbaed9faa0108da97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89595951"
---
# <a name="troubleshoot-app-service-issues-in-application-gateway"></a>Solución de problemas de App Service en Application Gateway

Obtenga información acerca de cómo diagnosticar y resolver los problemas que pueden surgir cuando se usa Azure App Service como destino de back-end con Azure Application Gateway.

## <a name="overview"></a>Información general

En este artículo aprenderá a resolver los siguientes problemas:

* La dirección URL de App Service se expone en el explorador cuando hay un redireccionamiento.
* Dominio de la cookie ARRAffinity de App Service establecido en el nombre de host de App Service (example.azurewebsites.net) en lugar del host original.

Cuando una aplicación back-end envía una respuesta de redireccionamiento, es posible que le interese redirigir el cliente a una dirección URL diferente de la especificada por la aplicación back-end. Es posible que quiera hacerlo cuando un servicio de aplicaciones se hospeda detrás de una puerta de enlace de aplicaciones y requiere que el cliente realice un redireccionamiento a su ruta de acceso relativa. Un ejemplo es un redireccionamiento de contoso.azurewebsites.net/path1 a contoso.azurewebsites.net/path2. 

Cuando el servicio de aplicaciones envía una respuesta de redireccionamiento, usa el mismo nombre de host en el encabezado de ubicación de su respuesta que el que aparece en la solicitud que recibe de la puerta de enlace de aplicaciones. Por ejemplo, el cliente hace la solicitud directamente a contoso.azurewebsites.net/path2, en lugar de pasar por la puerta de enlace de aplicaciones contoso.com/path2. No quiere omitir la puerta de enlace de aplicaciones.

Este problema puede ocurrir debido a los siguientes motivos principales:

- Ha configurado el redireccionamiento en su instancia de App Service. El redireccionamiento puede ser tan sencillo como agregar una barra diagonal final a la solicitud.
- Tiene autenticación de Azure Active Directory, lo que provoca el redireccionamiento.

Además, cuando se usa App Service detrás de una puerta de enlace de aplicaciones, el nombre de dominio asociado a la puerta de enlace de aplicaciones (example.com) es diferente del nombre de dominio de App Service (por ejemplo, example.azurewebsites.net). El valor de dominio para la cookie ARRAffinity establecida por App Service incluye el nombre de dominio example.azurewebsites.net, lo cual no es deseable. El nombre de host original (example.com) debe ser el valor del nombre de dominio de la cookie.

## <a name="sample-configuration"></a>Configuración de ejemplo

- Agente de escucha HTTP: básico o multisitio
- Grupo de direcciones de back-end: App Service
- Configuración de HTTP: **Seleccionar el nombre de host de la dirección de back-end** habilitado
- Sondeo: **Seleccionar el nombre de host de la configuración de HTTP de back-end** habilitado

## <a name="cause"></a>Causa

App Service es un servicio multiinquilino, por lo que usa el encabezado de host en la solicitud para enrutar la solicitud al punto de conexión correcto. El nombre de dominio predeterminado de App Service, *.azurewebsites.net (por ejemplo, contoso.azurewebsites.net), es diferente del nombre de dominio de la puerta de enlace de aplicaciones (por ejemplo, contoso.com). 

La solicitud original del cliente tiene el nombre de dominio de la puerta de enlace de aplicaciones (contoso.com) como nombre de host. Debe configurar la puerta de enlace de aplicaciones para cambiar el nombre de host de la solicitud original al nombre de host de App Service cuando enruta la solicitud al back-end de App Service. Use el conmutador **Seleccionar el nombre de host de la dirección de back-end** en la configuración HTTP de la puerta de enlace de aplicaciones. Use el conmutador **Seleccionar el nombre de host de la configuración de HTTP de back-end** en la configuración del sondeo de mantenimiento.



![La puerta de enlace de aplicaciones cambia el nombre de host.](./media/troubleshoot-app-service-redirection-app-service-url/appservice-1.png)

Cuando App Service realiza un redireccionamiento, usa el nombre de host reemplazado contoso.azurewebsites.net en el encabezado de ubicación, en lugar del nombre de host original contoso.com, a menos que esté configurado de otro modo. Compruebe los encabezados de solicitud y respuesta de ejemplo siguientes.
```
## Request headers to Application Gateway:

Request URL: http://www.contoso.com/path

Request Method: GET

Host: www.contoso.com

## Response headers:

Status Code: 301 Moved Permanently

Location: http://contoso.azurewebsites.net/path/

Server: Microsoft-IIS/10.0

Set-Cookie: ARRAffinity=b5b1b14066f35b3e4533a1974cacfbbd969bf1960b6518aa2c2e2619700e4010;Path=/;HttpOnly;Domain=contoso.azurewebsites.net

X-Powered-By: ASP.NET
```
En el ejemplo anterior, observe que el encabezado de respuesta tiene un código de estado 301 para el redireccionamiento. El encabezado de ubicación tiene el nombre de host de la instancia de App Service en lugar del nombre de host original, `www.contoso.com`.

## <a name="solution-rewrite-the-location-header"></a>Solución: Reescribir el encabezado de ubicación

Establezca el nombre de host del encabezado de ubicación en el nombre de dominio de la puerta de enlace de aplicaciones. Para ello, cree una [regla de reescritura](https://docs.microsoft.com/azure/application-gateway/rewrite-http-headers) con una condición que evalúe si el encabezado de ubicación en la respuesta contiene azurewebsites.net. También debe realizar una acción para volver a escribir el encabezado de ubicación de modo que tenga el nombre de host de la puerta de enlace de aplicaciones. Para obtener más información, vea las instrucciones para [volver a escribir el encabezado de ubicación](https://docs.microsoft.com/azure/application-gateway/rewrite-http-headers#modify-a-redirection-url).

> [!NOTE]
> La compatibilidad con la reescritura de encabezados HTTP solo está disponible para [Standard_v2 y WAF_v2 SKU](https://docs.microsoft.com/azure/application-gateway/application-gateway-autoscaling-zone-redundant) de Application Gateway. Si usa la SKU v1, se recomienda [migrar de v1 a V2](https://docs.microsoft.com/azure/application-gateway/migrate-v1-v2). Quiere usar la reescritura y otras [funcionalidades avanzadas](https://docs.microsoft.com/azure/application-gateway/application-gateway-autoscaling-zone-redundant#feature-comparison-between-v1-sku-and-v2-sku) que están disponibles con la SKU v2.

## <a name="alternate-solution-use-a-custom-domain-name"></a>Solución alternativa: Usar un dominio personalizado

Si usa la SKU v1, no puede volver a escribir el encabezado de la ubicación. Esta funcionalidad solo está disponible para la SKU v2. Para resolver el problema de redireccionamiento, pase el mismo nombre de host que recibe Application Gateway a App Service, en lugar de realizar un reemplazo del host.

App Service realizará el redireccionamiento (si existe) en el mismo encabezado de host original que apunta a Application Gateway y no en el suyo propio.

Debe tener un dominio personalizado y seguir este proceso:

- Registre el dominio en la lista de dominios personalizados de App Service. Debe tener un registro CNAME en el dominio personalizado que apunte al FQDN de App Service. Para más información, consulte [Asignación de un nombre DNS personalizado existente a Azure App Service](https://docs.microsoft.com//azure/app-service/app-service-web-tutorial-custom-domain).

    ![Lista de dominios personalizados de App Service](./media/troubleshoot-app-service-redirection-app-service-url/appservice-2.png)

- La instancia de App Service está lista para aceptar el nombre de host `www.contoso.com`. Cambie la entrada CNAME en DNS para que apunte al FQDN de la puerta de enlace de aplicaciones; por ejemplo, `appgw.eastus.cloudapp.azure.com`.

- Asegúrese de que el dominio `www.contoso.com` se resuelve en el FQDN de la puerta de enlace de aplicaciones al realizar una consulta de DNS.

- Establezca el sondeo personalizado para deshabilitar **Seleccionar el nombre de host de la configuración de HTTP de back-end**. En Azure Portal, desactive la casilla de la configuración del sondeo. En PowerShell, no use el conmutador **-PickHostNameFromBackendHttpSettings** en el comando **Set-AzApplicationGatewayProbeConfig**. En el campo de nombre de host del sondeo, escriba el FQDN de App Service, como example.azurewebsites.net. Las solicitudes de sondeo enviadas desde la puerta de enlace de aplicaciones llevan este FQDN en el encabezado de host.

  > [!NOTE]
  > En el paso siguiente, asegúrese de que el sondeo personalizado no está asociado a la configuración de HTTP de back-end. La configuración de HTTP aún tiene habilitada la opción **Seleccionar el nombre de host de la dirección de back-end** en este punto.

- Establezca la configuración de HTTP de la puerta de enlace de aplicaciones para deshabilitar **Seleccionar el nombre de host de la dirección de back-end**. En Azure Portal, desactive la casilla. En PowerShell, no use el conmutador **-PickHostNameFromBackendAddress** en el comando **Set-AzApplicationGatewayBackendHttpSettings**.

- Asocie el sondeo personalizado de nuevo a la configuración de HTTP de back-end y compruebe que el back-end está en buen estado.

- Ahora la puerta de enlace de aplicaciones debe reenviar el mismo nombre de host, `www.contoso.com`, a la instancia de App Service. El redireccionamiento se produce en el mismo nombre de host. Compruebe los encabezados de solicitud y respuesta de ejemplo siguientes.

Para implementar los pasos mencionados anteriormente con PowerShell en una instalación existente, use el siguiente script de Powershell de ejemplo. Tenga en cuenta que no hemos usado los modificadores **-PickHostname** en la configuración de sondeo y configuración de HTTP.

```azurepowershell-interactive
$gw=Get-AzApplicationGateway -Name AppGw1 -ResourceGroupName AppGwRG
Set-AzApplicationGatewayProbeConfig -ApplicationGateway $gw -Name AppServiceProbe -Protocol Http -HostName "example.azurewebsites.net" -Path "/" -Interval 30 -Timeout 30 -UnhealthyThreshold 3
$probe=Get-AzApplicationGatewayProbeConfig -Name AppServiceProbe -ApplicationGateway $gw
Set-AzApplicationGatewayBackendHttpSettings -Name appgwhttpsettings -ApplicationGateway $gw -Port 80 -Protocol Http -CookieBasedAffinity Disabled -Probe $probe -RequestTimeout 30
Set-AzApplicationGateway -ApplicationGateway $gw
```
  ```
  ## Request headers to Application Gateway:

  Request URL: http://www.contoso.com/path

  Request Method: GET

  Host: www.contoso.com

  ## Response headers:

  Status Code: 301 Moved Permanently

  Location: http://www.contoso.com/path/

  Server: Microsoft-IIS/10.0

  Set-Cookie: ARRAffinity=b5b1b14066f35b3e4533a1974cacfbbd969bf1960b6518aa2c2e2619700e4010;Path=/;HttpOnly;Domain=www.contoso.com

  X-Powered-By: ASP.NET
  ```
  ## <a name="next-steps"></a>Pasos siguientes

Si los pasos anteriores no resuelven el problema, abra una [incidencia de soporte técnico](https://azure.microsoft.com/support/options/).

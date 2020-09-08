---
title: Solución de problemas de estado degradado en Administrador de tráfico de Azure
description: Cómo solucionar problemas de perfiles de Administrador de tráfico cuando se muestra como muestra un estado degradado.
services: traffic-manager
documentationcenter: ''
author: duongau
manager: kumudD
ms.service: traffic-manager
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/03/2017
ms.author: duau
ms.openlocfilehash: 83dc432a1f88b443d500bf9a977abfed69211156
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89401561"
---
# <a name="troubleshooting-degraded-state-on-azure-traffic-manager"></a>Solución de problemas de estado degradado en el Administrador de tráfico de Azure

En este artículo se describe cómo solucionar problemas de un perfil de Azure Traffic Manager que muestra un estado degradado. El primer paso para solucionar un estado degradado de Azure Traffic Manager es habilitar el registro.  Para más información, consulte [Habilitación de los registros de recursos](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-diagnostic-logs). Para este escenario, considere que ha configurado un perfil de Traffic Manager orientado a algunos de sus servicios hospedados cloudapp.net. Si el estado de su instancia de Traffic Manager muestra **Degradado**, entonces el estado de uno o varios puntos de conexión puede ser **Degradado**:

![estado de punto de conexión degradado](./media/traffic-manager-troubleshooting-degraded/traffic-manager-degradedifonedegraded.png)

Si el estado de su instancia de Traffic Manager muestra **Inactivo**, ambos puntos de conexión pueden estar **deshabilitados**:

![Estado inactivo de Traffic Manager](./media/traffic-manager-troubleshooting-degraded/traffic-manager-inactive.png)

## <a name="understanding-traffic-manager-probes"></a>Descripción de los sondeos de Traffic Manager

* Traffic Manager considera que un punto de conexión está EN LÍNEA solo cuando el sondeo recibe una respuesta HTTP 200 desde la ruta de acceso del sondeo. Si la aplicación devuelve cualquier otro código de respuesta HTTP, debe agregar ese código de respuesta a los [intervalos de código de estado esperados](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-monitoring#configure-endpoint-monitoring) del perfil de Traffic Manager.
* Una respuesta de redireccionamiento de 30x se trata como un error, a menos que se haya especificado como un código de respuesta válido en los [intervalos de código de estado esperados](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-monitoring#configure-endpoint-monitoring) del perfil de Traffic Manager. Traffic Manager no sondea el destino de la redirección.
* En los sondeos HTTPs, se omiten los errores de certificado.
* No importa el contenido real de la ruta de acceso del sondeo, siempre y cuando se devuelva una respuesta 200. Realizar un sondeo de una dirección URL en algún contenido estático, como "/favicon.ico" es una técnica habitual. Los contenidos dinámicos, como las páginas ASP, puede que no siempre devuelvan una respuesta 200, incluso si la aplicación está en buen estado.
* Un procedimiento recomendado consiste en establecer la ruta de acceso del sondeo en algo que tenga una lógica suficiente para determinar si el sitio está activo o inactivo. En el ejemplo anterior, al establecer la ruta de acceso en "/favicon.ico", solo prueba que w3wp.exe está respondiendo. Puede que este sondeo no indique que la aplicación web está funcionando correctamente. Sería una mejor opción establecer una ruta de acceso a algo como "/Probe.aspx" que tiene lógica para determinar el estado del sitio. Por ejemplo, podría usar contadores de rendimiento para medir el uso de la CPU o el número de solicitudes con error. O bien, podría intentar acceder a los recursos de la base de datos o al estado de la sesión para asegurarse de que la aplicación web funciona.
* Si se degradan todos los puntos de conexión de un perfil, Traffic Manager los tratará a todos como correctos y enrutará el tráfico a todos ellos. Este comportamiento garantiza que los problemas con el mecanismo de sondeo no darán lugar a una interrupción completa del servicio.

## <a name="troubleshooting"></a>Solución de problemas

Para solucionar un error de sondeo, necesita una herramienta que muestre el código de estado HTTP devuelto desde la dirección URL de sondeo. Hay muchas herramientas disponibles que le muestran la respuesta HTTP sin formato.

* [Fiddler](https://www.telerik.com/fiddler)
* [curl](https://curl.haxx.se/)
* [wget](http://gnuwin32.sourceforge.net/packages/wget.htm)

Además, puede usar la pestaña Red de las herramientas de depuración de F12 de Internet Explorer para ver las respuestas HTTP.

En este ejemplo queremos ver la respuesta de la dirección URL de sondeo: http:\//watestsdp2008r2.cloudapp.net:80/Probe. El siguiente ejemplo de PowerShell ilustra el problema.

```powershell
Invoke-WebRequest 'http://watestsdp2008r2.cloudapp.net/Probe' -MaximumRedirection 0 -ErrorAction SilentlyContinue | Select-Object StatusCode,StatusDescription
```

Salida de ejemplo:

```output
StatusCode StatusDescription
---------- -----------------
        301 Moved Permanently
```

Observe que hemos recibido una respuesta de redirección. Como se indicó anteriormente, cualquier StatusCode distinto de 200 se considera un error. Traffic Manager cambia el estado de punto de conexión a Sin conexión. Para resolver el problema, compruebe la configuración del sitio web para asegurarse de que se puede devolver el StatusCode apropiado desde la ruta de acceso del sondeo. Vuelva a configurar el sondeo de Traffic Manager para que señale a una ruta de acceso que devuelva una respuesta 200.

Si el sondeo usa el protocolo HTTPS, es posible que tenga que deshabilitar la comprobación de certificados para evitar errores SSL/TLS durante la prueba. Las siguientes instrucciones de PowerShell deshabilitan la validación de certificados para la sesión actual de PowerShell:

```powershell
add-type @"
using System.Net;
using System.Security.Cryptography.X509Certificates;
public class TrustAllCertsPolicy : ICertificatePolicy {
    public bool CheckValidationResult(
    ServicePoint srvPoint, X509Certificate certificate,
    WebRequest request, int certificateProblem) {
    return true;
    }
}
"@
[System.Net.ServicePointManager]::CertificatePolicy = New-Object TrustAllCertsPolicy
```

## <a name="next-steps"></a>Pasos siguientes

[Información acerca de los métodos de enrutamiento del tráfico del Administrador de tráfico](traffic-manager-routing-methods.md)

[¿Qué es el Administrador de tráfico?](traffic-manager-overview.md)

[Cloud Services](https://go.microsoft.com/fwlink/?LinkId=314074)

[Azure App Service](https://azure.microsoft.com/documentation/services/app-service/web/)

[Operaciones del Administrador de tráfico (referencia de la API de REST)](https://go.microsoft.com/fwlink/?LinkId=313584)

[Cmdlets del Administrador de tráfico de Azure][1]

[1]: https://docs.microsoft.com/powershell/module/az.trafficmanager

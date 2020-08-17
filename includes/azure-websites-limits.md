---
author: rothja
ms.service: app-service
ms.topic: include
ms.date: 03/04/2020
ms.author: jroth
ms.openlocfilehash: 1811590dcf9077a503f89a900f661c52aa442c96
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87425039"
---
| Resource | Gratuito | Compartido | Básica | Estándar | Premium (v2) | Aislado </th> |
| --- | --- | --- | --- | --- | --- | --- |
| [Aplicaciones web, móviles o de API](https://azure.microsoft.com/services/app-service/) por [plan de Azure App Service](../articles/app-service/overview-hosting-plans.md)<sup>1</sup> |10 |100 |Ilimitado<sup>2</sup> |Ilimitado<sup>2</sup> |Ilimitado<sup>2</sup> |Ilimitado<sup>2</sup>|
| [plan de App Service](../articles/app-service/overview-hosting-plans.md) |10 por región |10 por grupo de recursos |100 por grupo de recursos |100 por grupo de recursos |100 por grupo de recursos |100 por grupo de recursos|
| Tipo de instancia de proceso |Compartido |Compartido |Dedicado<sup>3</sup> |Dedicado<sup>3</sup> |Dedicado<sup>3</sup></p> |Dedicado<sup>3</sup>|
| [Escalar horizontalmente](../articles/app-service/manage-scale-up.md) (número máximo de instancias) |1 compartido |1 compartido |3 dedicados<sup>3</sup> |10 dedicados<sup>3</sup> |30 dedicados<sup>3</sup>|100 dedicados<sup>4</sup>|
| Almacenamiento<sup>5</sup> |1 GB<sup>5</sup> |1 GB<sup>5</sup> |10 GB<sup>5</sup> |50 GB<sup>5</sup> |250 GB<sup>5</sup> <br/><br/> Para más de 250 GB, envíe una solicitud de soporte técnico. |1 TB<sup>5</sup> <br/><br/> La cuota de almacenamiento disponible es de 999 GB. |
| Tiempo de CPU (5 minutos)<sup>6</sup> |3 minutos |3 minutos |Sin límite, se paga según las [tarifas](https://azure.microsoft.com/pricing/details/app-service/)</a> estándar. |Sin límite, se paga según las [tarifas](https://azure.microsoft.com/pricing/details/app-service/)</a> estándar. |Sin límite, se paga según las [tarifas](https://azure.microsoft.com/pricing/details/app-service/)</a> estándar. |Sin límite, se paga según las [tarifas](https://azure.microsoft.com/pricing/details/app-service/)</a> estándar.|
| Tiempo de CPU (día)<sup>6</sup> |60 minutos |240 minutos |Sin límite, se paga según las [tarifas](https://azure.microsoft.com/pricing/details/app-service/)</a> estándar. |Sin límite, se paga según las [tarifas](https://azure.microsoft.com/pricing/details/app-service/)</a> estándar. |Sin límite, se paga según las [tarifas](https://azure.microsoft.com/pricing/details/app-service/)</a> estándar. |Sin límite, se paga según las [tarifas](https://azure.microsoft.com/pricing/details/app-service/)</a> estándar. |
| Memoria (1 hora) |1024 MB por plan de App Service |1024 MB por aplicación |N/D |N/D |N/D |N/D |
| Ancho de banda |165 MB |Ilimitado; se aplican [tasas por transferencia de datos](https://azure.microsoft.com/pricing/details/data-transfers/) |Ilimitado; se aplican [tasas por transferencia de datos](https://azure.microsoft.com/pricing/details/data-transfers/) |Ilimitado; se aplican [tasas por transferencia de datos](https://azure.microsoft.com/pricing/details/data-transfers/) |Ilimitado; se aplican [tasas por transferencia de datos](https://azure.microsoft.com/pricing/details/data-transfers/) |Ilimitado; se aplican [tasas por transferencia de datos](https://azure.microsoft.com/pricing/details/data-transfers/) |
| Arquitectura de la aplicación |32 bits |32 bits |32 bits/64 bits |32 bits/64 bits |32 bits/64 bits |32 bits/64 bits |
| Web Sockets por instancia<sup>7</sup> |5 |35 |350 |Sin límite |Sin límite |Sin límite |
| Conexiones IP | 600 | 600 | Depende del tamaño de la instancia<sup>8</sup> | Depende del tamaño de la instancia<sup>8</sup> | Depende del tamaño de la instancia<sup>8</sup> | 16 000 |
| Conexiones de depurador [simultáneas](../articles/app-service/troubleshoot-dotnet-visual-studio.md) por aplicación |1 |1 |1 |5 |5 |5 |
| Instancias de App Service Certificate por suscripción<sup>9</sup>| No compatible | No compatible |10 |10 |10 |10 |
| Dominios personalizados por aplicación</a> |0 (solo subdominio de azurewebsites.net)|500 |500 |500 |500 |500 |
| Compatibilidad con dominio [Compatibilidad con SSL](../articles/app-service/configure-ssl-certificate.md) |No admitido, certificado comodín para \*.azurewebsites.net disponible de forma predeterminada.|No admitido, certificado comodín para \*.azurewebsites.net disponible de forma predeterminada.|Conexiones SSL SNI ilimitadas |Se incluyen conexiones SNI SSL ilimitadas y 1 conexión SSL de IP |Se incluyen conexiones SNI SSL ilimitadas y 1 conexión SSL de IP | Se incluyen conexiones SNI SSL ilimitadas y 1 conexión SSL de IP|
| Conexiones híbridas | | | 5 por plan | 25 por plan | 200 por aplicación | 200 por aplicación |
| [Integración de Virtual Network](../articles/app-service/web-sites-integrate-with-vnet.md) | | |   |  X |  X  |  X  |
| Equilibrador de carga integrado | |X |X |X |X |X<sup>10</sup> |
| [Always On](../articles/app-service/configure-common.md) | | |X |X |X |X |
| [Copias de seguridad programadas](../articles/app-service/manage-backup.md) | | | | Copias de seguridad programadas cada 2 horas, con un máximo de 12 copias de seguridad al día (manuales y programadas). | Copias de seguridad programadas cada hora, con un máximo de 50 copias de seguridad al día (manuales y programadas). | Copias de seguridad programadas cada hora, con un máximo de 50 copias de seguridad al día (manuales y programadas). |
| [Autoscale](../articles/app-service/manage-scale-up.md) | | | |X |X |X |
| [WebJobs](../articles/app-service/webjobs-create.md)<sup>11</sup> |X |X |X |X |X |X |
| [Supervisión de extremos](../articles/app-service/web-sites-monitor.md) | | |X |X |X |X |
| [Espacios de ensayo](../articles/app-service/deploy-staging-slots.md) por aplicación| | | |5 |20 |20 |
| [Testing in Production](../articles/app-service/deploy-staging-slots.md#route-traffic)| | | |X |X |X |
| [Registros de diagnóstico](../articles/app-service/troubleshoot-diagnostic-logs.md) | X | X | X | X | X | X |
| Kudu | X | X | X | X | X | X |
| [Autenticación y autorización](../articles/app-service/overview-authentication-authorization.md) | X | X | X | X | X | X |
| [Certificados administrados de App Service (versión preliminar pública)](https://azure.microsoft.com/updates/secure-your-custom-domains-at-no-cost-with-app-service-managed-certificates-preview/)<sup>12</sup> | |  | X | X | X | X |
| Contrato de nivel de servicio | |  |99,95 %|99,95 %|99,95 %|99,95 %|  

<sup>1</sup>Las aplicaciones y las cuotas de almacenamiento son por plan de App Service, a menos que se indique lo contrario.  
<sup>2</sup>El número real de aplicaciones que puedes hospedar en estas máquinas depende de la actividad de las aplicaciones, el tamaño de las instancias de máquina y el correspondiente uso de los recursos.  
<sup>3</sup>Las instancias dedicadas pueden tener diferentes tamaños. Para más información, consulte [Precios de App Service](https://azure.microsoft.com/pricing/details/app-service/).  
<sup>4</sup>Se permiten más a petición.  
<sup>5</sup>El límite de almacenamiento es el tamaño total del contenido en todas las aplicaciones en el mismo plan de App Service. El tamaño total del contenido de todas las aplicaciones en todos los planes de App Service en un solo grupo de recursos y región no puede superar los 500 GB.  
<sup>6</sup>Estos recursos están limitados por los recursos físicos en las instancias dedicadas (el tamaño de la instancia y el número de instancias).  
<sup>7</sup>Si se escala una aplicación en el nivel Básico a dos instancias, dispones de 350 conexiones simultáneas para cada una de las dos. En el nivel Estándar u otro superior, no hay ningún límite teórico para los sockets web, pero otros factores pueden limitar el número de estos. Por ejemplo, el número máximo de solicitudes simultáneas permitidas (que define `maxConcurrentRequestsPerCpu`) es: 7500 por máquina virtual pequeña, 15 000 por máquina virtual mediana (7500 x 2 núcleos) y 75 000 por máquina virtual de gran tamaño (18 750 x 4 núcleos).  
<sup>8</sup> Las conexiones IP máximas se realizan por instancia y dependen del tamaño de la instancia: 1920 por instancia B1/S1/P1V2, 3968 por instancia B2/S2/P2V2, 8064 por instancia B3/S3/P3V2.  
<sup>9</sup> El límite de cuota de App Service Certificate por suscripción se puede aumentar a través de una solicitud de soporte técnico hasta un límite máximo de 200.  
<sup>10</sup>Las SKU de App Service aislado pueden tener equilibrio de carga interno (ILB) con Azure Load Balancer, lo que significa que no hay conectividad pública desde Internet. Como resultado, algunas características de un App Service aislado con ILB deben usarse desde máquinas que tienen acceso directo al punto de conexión de red del ILB.  
<sup>11</sup> Ejecute scripts o archivos ejecutables personalizados bajo demanda, según una programación o de manera continua como tarea en segundo plano dentro de su instancia de App Service. Siempre disponible se requiere para la ejecución continua de Trabajos web. No hay ningún límite predefinido en el número de trabajos web que se pueden ejecutar en una instancia de App Service. Hay límites prácticos que dependen de lo que el código de aplicación intente hacer.

<sup>12</sup>No se admiten los dominios desnudos. Solo se emiten certificados estándar (los certificados comodín no están disponibles). Está limitado a un solo certificado gratuito por dominio personalizado.

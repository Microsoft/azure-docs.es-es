---
title: 'Ejemplos de plantillas de Azure Resource Manager: Azure Front Door'
description: Ejemplos de plantillas de Azure Resource Manager para Azure Front Door
services: frontdoor
documentationcenter: ''
author: sharad4u
ms.service: frontdoor
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/04/2020
ms.author: sharadag
ms.openlocfilehash: 47db565288d1bfe1f98c58536e726eb7ec55d0a9
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87084643"
---
# <a name="azure-resource-manager-deployment-model-templates-for-front-door"></a>Plantillas del modelo de implementación de Azure Resource Manager para Front Door

En la tabla siguiente se incluyen vínculos a las plantillas del modelo de implementación de Azure Resource Manager para Azure Front Door. 

| Plantilla | Descripción |
| ---| ---|
| [Create a basic Front Door](https://github.com/Azure/azure-quickstart-templates/tree/master/101-front-door-create-basic)| Crea una configuración básica de Front Door con un único back-end. |
| [Create a Front Door with multiple backends and backend pools and URL based routing](https://github.com/Azure/azure-quickstart-templates/tree/master/101-front-door-create-multiple-backends)| Crea una instancia de Front Door con equilibrio de carga configurado para varios back-ends en el grupo de back-ends y también a través de grupos de back-ends en función de la ruta de acceso de la URL. |
| [Incorporación de un dominio personalizado con Front Door](https://github.com/Azure/azure-quickstart-templates/tree/master/101-front-door-custom-domain)| Agregue un dominio personalizado a Front Door. |
| [Create Front Door with geo filtering](https://github.com/Azure/azure-quickstart-templates/tree/master/101-front-door-geo-filtering)| Crea una instancia de Front Door que permite o bloquea tráfico desde determinados países y regiones. |
| [Control Health Probes for your backends on Front Door](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-health-probes)| Actualiza su instancia de Front Door para cambiar la configuración de sondeo de estado mediante la actualización de la ruta de acceso de sondeo y también los intervalos en los que se enviarán los sondeos. |
| [Create Front Door with Active/Standby backend configuration](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-priority-lb)| Crea una instancia de Front Door que demuestra el enrutamiento basado en la prioridad para la topología de aplicación activo/en espera; es decir, mediante el envío de forma predeterminada de todo el tráfico al back-end principal (prioridad más alta) hasta que deja de estar disponible. |
| [Create Front Door with caching enabled for certain routes](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-create-caching)| Crea una instancia de Front Door con el almacenamiento en caché habilitado para la configuración de enrutamiento definida, lo que almacena en caché los recursos estáticos para la carga de trabajo. |
| [Configure Session Affinity for your Front Door host names](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-session-affinity) | Actualiza una instancia de Front Door para habilitar la afinidad de sesión para el host de front-end; por lo tanto, envía el tráfico posterior desde la misma sesión de usuario para el mismo back-end. |
| [Configure Front Door for client IP whitelisting or blacklisting](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-waf-clientip)| Configura una instancia de Front Door para restringir el tráfico de determinadas direcciones IP cliente mediante el control de acceso personalizado con direcciones IP de cliente. |
| [Configure Front Door to take action with specific http parameters](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-waf-http-params)| Configura una instancia de Front Door para permitir o bloquear cierto tráfico según los parámetros de http en la solicitud entrante mediante el uso de reglas personalizadas para el control de acceso con parámetros http. |
| [Configure Front Door rate limiting](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-rate-limiting)| Configura una instancia de Front Door para limitar la velocidad del tráfico entrante para un host de front-end determinado. |
| | |

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [crear una instancia de Front Door](quickstart-create-front-door.md).
- Más información acerca de cómo [funciona Front Door](front-door-routing-architecture.md).

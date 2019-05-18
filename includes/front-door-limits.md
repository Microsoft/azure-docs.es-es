---
title: archivo de inclusión
description: archivo de inclusión
services: frontdoor
author: sharad4u
ms.service: frontdoor
ms.topic: include
ms.date: 05/09/2019
ms.author: sharadag
ms.custom: include file
ms.openlocfilehash: e1f5a1c8229544d97d9ff64748390f0d5237ab97
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/09/2019
ms.locfileid: "65754003"
---
| Recurso | Límite predeterminado y máximo |
| --- | --- |
| Recursos de servicio de puerta de Azure por suscripción | 100 |
| Hosts de front-end, que incluye los dominios personalizados por recurso | 100 |
| Reglas de enrutamiento por recurso | 100 |
| Grupos de back-end por recurso | 50 |
| Back-end por grupo de back-end | 100 |
| Patrones de ruta de acceso para coincidir con una regla de enrutamiento | 25 |
| Reglas de firewall de aplicación web personalizadas por directiva | 10 |
| Directiva del firewall de aplicaciones web por recurso | 100 |

### <a name="timeout-values"></a>Valores de tiempo de espera
#### <a name="client-to-front-door"></a>Cliente para Front Door
- Front Door tiene un tiempo de espera de la conexión TCP de 61 segundos.

#### <a name="front-door-to-application-back-end"></a>Puerta de entrada al back-end de aplicación
- Si la respuesta es una respuesta fragmentada, se devuelve una respuesta 200 si o cuando se recibe el primer fragmento.
- Una vez que se reenvía la solicitud HTTP para el back-end, puerta delantera espera 30 segundos para el primer paquete desde el back-end. A continuación, se devuelve un error 503 al cliente.
- Una vez recibido el primer paquete desde el back-end, puerta delantera espera 30 segundos en un tiempo de inactividad. A continuación, se devuelve un error 503 al cliente.
- Puerta de entrada para el tiempo de espera de sesión TCP de back-end es de 30 minutos.

### <a name="upload-and-download-data-limit"></a>Cargar y descargar el límite de datos

|  | Con (CTE) de codificación de transferencia fragmentada | Sin fragmentación de HTTP |
| ---- | ------- | ------- |
| **Descargar** | No hay ningún límite en el tamaño de descarga. | No hay ningún límite en el tamaño de descarga. |
| **Cargar** |  No hay ningún límite siempre y cuando cada carga de la CTE es inferior a 2 GB. | El tamaño no puede ser mayor que 2 GB. |

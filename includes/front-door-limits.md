---
title: archivo de inclusión
description: archivo de inclusión
services: frontdoor
author: duongau
ms.service: frontdoor
ms.topic: include
ms.date: 05/09/2019
ms.author: sharadag
ms.custom: include file
ms.openlocfilehash: 361fb18574e90fc46e45bff8914c51ee1afa2f96
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89411759"
---
| Resource | Límite |
| --- | --- |
| Recursos de Azure Front Door por suscripción | 100 |
| Hosts de front-end, que incluye los dominios personalizados por recurso | 500 |
| Reglas de enrutamiento por recurso | 500 |
| Grupos de servidores back-end por recurso | 50 |
| Servidores back-end por grupo de servidores back-end | 100 |
| Patrones de ruta de acceso para coincidir con una regla de enrutamiento | 25 |
| Direcciones URL en una sola llamada de purga de la caché | 100 |
| Reglas de firewall de aplicación web personalizadas por directiva | 100 |
| Directiva del firewall de aplicaciones web por suscripción | 100 |
| Condiciones de coincidencia del firewall de aplicaciones web por regla personalizada | 10 |
| Intervalos de direcciones IP del firewall de aplicaciones web por condición de coincidencia | 600 |
| Valores de coincidencia de la cadena del firewall de aplicaciones web por condición de coincidencia | 10 |
| Longitud del valor de coincidencia de la cadena del firewall de aplicaciones web | 256 |
| Longitud del nombre del parámetro del cuerpo POST del firewall de aplicaciones web | 256 |
| Longitud del nombre del encabezado HTTP del firewall de aplicaciones web | 256 |
| Longitud del nombre de la cookie del firewall de aplicaciones web | 256 |
| Tamaño del cuerpo de la solicitud HTTP del firewall de aplicaciones web inspeccionado | 128 KB |
| Longitud del cuerpo de respuesta personalizada del firewall de aplicaciones web | 2 KB |

### <a name="timeout-values"></a>Valores de tiempo de expiración
#### <a name="client-to-front-door"></a>Cliente para Front Door
* Front Door tiene un tiempo de espera de la conexión TCP de 61 segundos.

#### <a name="front-door-to-application-back-end"></a>Front Door al back-end de la aplicación
* Si la respuesta es una respuesta fragmentada, se devuelve una respuesta 200 si se recibe el primer fragmento, o cuando se recibe.
* Después de que la solicitud HTTP se reenvía al back-end, Front Door espera 30 segundos al primer paquete del back-end. A continuación, se devuelve un error 503 al cliente. Este valor se puede configurar desde el campo sendRecvTimeoutSeconds de la API.
    * En el caso de los escenarios de almacenamiento en caché, este tiempo de expiración no se puede configurar, por lo que si una solicitud se almacena en caché y el primer paquete tarda más de 30 segundos desde Front Door o desde el back-end, se devuelve un error 504 al cliente. 
* Una vez recibido el primer paquete del back-end, Front Door espera 30 segundos en un tiempo de expiración de inactividad. A continuación, se devuelve un error 503 al cliente. Este valor de tiempo de expiración no se puede configurar.
* El tiempo de expiración de la sesión TCP desde Front Door hasta el back-end es de 90 segundos.

### <a name="upload-and-download-data-limit"></a>Límite de carga y descarga de datos

|  | Con codificación de transferencia fragmentada (CTE) | Sin fragmentación de HTTP |
| ---- | ------- | ------- |
| **Descargar** | No hay ningún límite para el tamaño de descarga. | No hay ningún límite para el tamaño de descarga. |
| **Cargar** |    No hay ningún límite siempre y cuando cada carga de CTE sea inferior a 2 GB. | El tamaño no puede ser superior a 2 GB. |

### <a name="other-limits"></a>Otros límites
* Tamaño máximo de la dirección URL: 8192 bytes: especifica la longitud máxima de la dirección URL sin procesar (esquema + nombre de host + puerto + ruta de acceso + cadena de consulta de la dirección URL)
* Tamaño máximo de la cadena de consulta: 4096 bytes: especifica la longitud máxima de la cadena de consulta, en bytes.
* Tamaño máximo del encabezado de respuesta HTTP desde la dirección URL de sondeo de estado: 4096 bytes (se ha especificado la longitud máxima de todos los encabezados de respuesta de los sondeos de estado). 

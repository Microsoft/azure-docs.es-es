---
author: vhorne
ms.service: application-gateway
ms.topic: include
ms.date: 3/26/2019
ms.author: victorh
ms.openlocfilehash: e35f01cc5479a5d27d92c9ed74a12bc44db673cd
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/09/2019
ms.locfileid: "65754009"
---
| Recurso | Límite predeterminado y máximo | NOTA |
| --- | --- | --- |
| Azure Application Gateway |1.000 por suscripción | |
| Configuraciones de IP front-end |2 |1 pública y 1 privada |
| Puertos de front-end |100<sup>1</sup> | |
| Grupos de direcciones de back-end |100<sup>1</sup> | |
| Servidores back-end por grupo |1,200 | |
| Agentes de escucha HTTP |100<sup>1</sup> | |
| Reglas de equilibrio de carga HTTP |100<sup>1</sup> | |
| Configuración de HTTP de back-end |100<sup>1</sup> | |
| Instancias por puerta de enlace |32 | |
| Certificados SSL |100<sup>1</sup> |1 por agentes de escucha HTTP |
| Certificados de autenticación |100 | |
| Certificados raíz de confianza |100 | |
| Tiempo de espera mínimo |1 segundo | |
| Tiempo de espera de solicitud máximo |24 horas | |
| Número de sitios |100<sup>1</sup> |1 por agentes de escucha HTTP |
| Asignaciones de URL por agente de escucha |1 | |
| Número máximo de reglas basadas en rutas por mapa de direcciones URL|100||
| Configuraciones de redirección |100<sup>1</sup>| |
| Conexiones simultáneas de WebSocket |Puertas de enlace medianas 20k<br> Puertas de enlace grandes 50k| |
| Longitud máxima de dirección URL|8.000||
| Tamaño de carga máximo del archivo, estándar |2 GB | |
| Tamaño máximo de carga de archivos WAF |Puertas de enlace de WAF medianos, 100 MB<br>Puertas de enlace grandes WAF, 500 MB| |
| Límite de tamaño de cuerpo de WAF, sin los archivos|128 KB||

<sup>1</sup> en el caso de las SKU de WAF habilitado, se recomienda que limite el número de recursos a 40 para un rendimiento óptimo.

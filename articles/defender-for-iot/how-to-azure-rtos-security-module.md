---
title: Configuración y personalización de un módulo de seguridad para Azure RTOS
description: Obtenga información sobre cómo configurar y personalizar el módulo de seguridad para Azure RTOS.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/09/2020
ms.author: mlottner
ms.openlocfilehash: 9106e00fe2146978f97b480e3afd3b7ed58c9130
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931978"
---
# <a name="configure-and-customize-security-module-for-azure-rtos-preview"></a>Configuración y personalización de un módulo de seguridad para Azure RTOS (versión preliminar)

Use el siguiente archivo para configurar el comportamiento del dispositivo.

## <a name="azure_iot_security_moduleincasc_porth"></a>azure_iot_security_module/inc/asc_port.h

 El comportamiento predeterminado de cada configuración se proporciona en las tablas siguientes: 

### <a name="general"></a>General

| Nombre | Tipo | Valor predeterminado | Detalles |
| - | - | - | - |
| ASC_SECURITY_MODULE_ID | String | --- | Identificador único del dispositivo.  |
| ASC_SECURITY_MODULE_PENDING_TIME  | Number | 300 | Tiempo pendiente del módulo de seguridad en segundos. Si el tiempo supera el cambio de estado que se va a suspender. |

#### <a name="collection"></a>Colección

| Nombre | Tipo | Valor predeterminado | Detalles |
| - | - | - | - |
| ASC_HIGH_PRIORITY_INTERVAL | Number | 10 | Intervalo en segundos de grupo de prioridad alta de recopiladores. |
| ASC_MEDIUM_PRIORITY_INTERVAL | Number | 30 | Intervalo en segundos de grupo de prioridad media de recopiladores. |
| ASC_LOW_PRIORITY_INTERVAL | Number | 145,440  | Intervalo en segundos de grupo de prioridad baja de recopiladores. |

#### <a name="collector-network-activity"></a>Actividad de red del recopilador

Para personalizar la configuración de la actividad de red del recopilador, utilice lo siguiente:

| Nombre | Tipo | Valor predeterminado | Detalles |
| - | - | - | - |
| ASC_COLLECTOR_NETWORK_ACTIVITY_TCP_DISABLED | Boolean | false | Filtrado de la actividad de red `TCP` |
| ASC_COLLECTOR_NETWORK_ACTIVITY_UDP_DISABLED | Boolean | false | Filtrado de eventos de actividad de red `UDP` |
| ASC_COLLECTOR_NETWORK_ACTIVITY_ICMP_DISABLED | Boolean | false | Filtrado de eventos de actividad de red `ICMP` |
| ASC_COLLECTOR_NETWORK_ACTIVITY_CAPTURE_UNICAST_ONLY | Boolean | true | Capture solo paquetes entrantes de unidifusión. Cuando se establezca en false, capture también difusión y multidifusión. |
| ASC_COLLECTOR_NETWORK_ACTIVITY_MAX_IPV4_OBJECTS_IN_CACHE | Number | 64 | Número máximo de eventos de red IPv4 para almacenar en memoria |
| ASC_COLLECTOR_NETWORK_ACTIVITY_MAX_IPV6_OBJECTS_IN_CACHE | Number | 64  | Número máximo de eventos de red IPv6 para almacenar en memoria |


## <a name="compile-flags"></a>Marcas de compilación
Las marcas de compilación permiten invalidar las configuraciones predefinidas.

### <a name="collectors"></a>Recopiladores
| Nombre | Tipo | Valor predeterminado | Detalles |
| - | - | - | - |
| collector_heartbeat_enabled | Boolean | ACTIVAR | Habilitación del recopilador de latidos |
| collector_network_activity_enabled | Boolean | ACTIVAR | Habilitación del recopilador de actividades de red |
| collector_system_information_enabled | Boolean | ACTIVAR | Habilitación del recopilador de información del sistema |

## <a name="supported-security-alerts-and-recommendations"></a>Alertas y recomendaciones de seguridad admitidas

El módulo de seguridad para Azure RTOS admite alertas y recomendaciones de seguridad específicas. Asegúrese de [revisar y personalizar los valores de alerta y recomendación pertinentes](concept-rtos-security-alerts-recommendations.md) para el servicio.

## <a name="log-analytics-optional"></a>Log Analytics (opcional)

Aunque es opcional y no se requiere, la habilitación y configuración de Log Analytics puede resultar de utilidad cuando quiere investigar aún más las actividades y los eventos de los dispositivos. Lea sobre cómo configurar y usar [Log Analytics con el servicio Defender para IoT](how-to-security-data-access.md#log-analytics) para obtener más información. 

## <a name="next-steps"></a>Pasos siguientes

- Revisión y personalización de [alertas y recomendaciones de seguridad](concept-rtos-security-alerts-recommendations.md) del módulo de seguridad para Azure RTOS
- Consulte la [API del módulo de seguridad para Azure RTOS](azure-rtos-security-module-api.md) según sea necesario.


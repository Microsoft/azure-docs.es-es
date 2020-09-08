---
title: 'Azure Front Door: seguridad en el nivel de aplicación | Microsoft Docs'
description: Este artículo le ayudará a entender cómo Azure Front Door le permite proteger los servidores back-end de aplicaciones.
services: frontdoor
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/10/2018
ms.author: duau
ms.openlocfilehash: 4ee50b4c7da27df3630c1b4d263f076da44189bc
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89399946"
---
# <a name="application-layer-security-with-front-door"></a>Seguridad en el nivel de aplicación con Front Door
Azure Front Door proporciona funcionalidad de protección de aplicaciones web para proteger las aplicaciones web contra ataques de red y vulnerabilidades web habituales, tales como la inyección de código SQL o scripts entre sitios (XSS). La seguridad de nivel de aplicación de Front Door está habilitada en los servidores front-end http(s), está distribuida globalmente y tiene disponibilidad total para detener los ataques malintencionados en el perímetro de la red de Azure, muy lejos de los servidores back-end. Gracias a su mayor seguridad y a la optimización de su rendimiento, Front Door ofrece experiencias web rápidas y seguras a los usuarios finales.

## <a name="application-protection"></a>Protección de las aplicaciones
La protección de las aplicaciones de Front Door se configura en cada entorno perimetral del mundo en consonancia con las aplicaciones e impide de forma automática que el tráfico que no sea http(s) llegue a sus aplicaciones web. Nuestra arquitectura multiinquilino distribuida permite aplicar una protección global a cualquier escala sin sacrificar el rendimiento. En el caso de las cargas de trabajo http(s), el servicio de protección de aplicaciones de Front Door dispone de un avanzado motor de reglas personalizadas, un conjunto de reglas preconfiguradas contra ataques comunes y registros detallados de todas las solicitudes que coinciden con una regla. Se pueden utilizar acciones flexibles, como Permitir, Bloquear o Registrar solo.

## <a name="custom-access-control-rules"></a>Reglas de control de acceso personalizadas
- **Listas de direcciones IP permitidas y denegadas:** puede configurar reglas personalizadas para controlar el acceso a las aplicaciones web en función de una lista de direcciones IP de cliente. Se admiten direcciones IP v4 e IP v6.
- **Control de acceso basado en la ubicación geográfica:** puede configurar reglas personalizadas para controlar el acceso a las aplicaciones web en función del código de país del que procede una dirección IP de cliente.
- **Filtrado de los parámetros HTTP:** puede configurar reglas de acceso personalizadas basadas en la coincidencia de los parámetros de la solicitud http(s), incluidos los encabezados, las direcciones URL y las cadenas de consulta.

## <a name="azure-managed-rules"></a>Reglas administradas por Azure
- De forma predeterminada, hay habilitado un conjunto preconfigurado de reglas para las vulnerabilidades OWASP más habituales. En su versión preliminar, el conjunto de reglas incluye la comprobación de solicitudes sqli y xss. En el futuro, se agregarán otras reglas. Si lo desea, puede comenzar con la acción de solo registro para comprobar si las reglas preconfiguradas funcionan según lo previsto con sus aplicaciones. 

## <a name="rate-limiting"></a>Limitación de frecuencia
- Las reglas de control de frecuencia están diseñadas para limitar el tráfico anormalmente alto procedente de cualquier dirección IP de cliente.  Puede establecer un umbral en función del número de solicitudes web que se permiten en una dirección IP de cliente durante un minuto.

## <a name="centralized-protection-policy"></a>Directiva de protección centralizada
- Puede definir varias reglas de protección y agregarlas a una directiva por orden de prioridad. Las reglas personalizadas tienen una prioridad mayor que el conjunto de reglas administrado a la hora de permitir excepciones. Hay una única directiva asociada a la aplicación web.  La misma directiva de protección de aplicaciones web se replica en todos los servidores perimetrales de todas las ubicaciones, lo que garantiza que se aplica una directiva de seguridad coherente en todas las regiones.

## <a name="configuration"></a>Configuración
- Durante la versión preliminar, puede usar las API de REST, PowerShell o la CLI para crear e implementar las directivas y las reglas de protección de aplicaciones de Front Door. Se podrá acceder al portal antes de que el servicio esté disponible con carácter general. 


## <a name="monitoring"></a>Supervisión
Front Door permite supervisar las aplicaciones web frente a ataques utilizando métricas en tiempo real que se integran con Azure Monitor para realizar un seguimiento de las alertas y supervisar con facilidad las tendencias.

## <a name="pricing"></a>Precios
La seguridad en el nivel de aplicación de Front Door es gratis durante la versión preliminar.


## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [crear una instancia de Front Door](quickstart-create-front-door.md).
- Más información acerca de cómo [funciona Front Door](front-door-routing-architecture.md).

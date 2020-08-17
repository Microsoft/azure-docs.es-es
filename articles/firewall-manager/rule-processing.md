---
title: Lógica de procesamiento de reglas de Azure Firewall Manager
description: Información acerca de la lógica de procesamiento de reglas de Azure Firewall
services: firewall-manager
author: vhorne
ms.service: firewall-manager
ms.topic: conceptual
ms.date: 06/30/2020
ms.author: victorh
ms.openlocfilehash: 9184bf7baa85420e067edb4c0aafccb7e6711225
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86512187"
---
# <a name="azure-firewall-rule-processing-logic"></a>Lógica de procesamiento de reglas de Azure Firewall

Azure Firewall tiene reglas NAT, reglas de red y reglas de aplicaciones. Las reglas se procesan en función del tipo de regla.

## <a name="network-rules-and-applications-rules"></a>Reglas de red y reglas de aplicaciones

En primer lugar se aplican las reglas de red y después las reglas de aplicaciones. Las reglas están terminando. Por tanto, si se encuentra alguna coincidencia en las reglas de red, las reglas de aplicación no se procesan.  Si no coincide ninguna regla de red, y si el protocolo del paquete es HTTP/HTTPS, las reglas de aplicación lo evalúan posteriormente. Si sigue sin haber coincidencias, el paquete se evalúa con la colección de reglas de infraestructura. Si sigue sin haber coincidencias, el paquete se deniega de forma predeterminada.

## <a name="nat-rules"></a>Reglas NAT

Para habilitar la conectividad entrante, configure Traducción de direcciones de red de destino (DNAT) como se describe en [Tutorial: Filtro del tráfico entrante con la DNAT de Azure Firewall mediante Azure Portal](../firewall/tutorial-firewall-dnat.md). Las reglas de DNAT son las primeras que se aplican. Si se encuentra alguna coincidencia, se agrega una regla de red correspondiente implícita para permitir el tráfico traducido. Para invalidar este comportamiento, agregue explícitamente una colección de reglas de red con reglas de denegación que coinciden con el tráfico traducido. No se aplican reglas de aplicación a estas conexiones.

## <a name="inherited-rules"></a>Reglas heredadas

Las colecciones de reglas de red heredadas de una directiva primaria siempre se priorizan por encima de las colecciones de reglas de red definidas como parte de la nueva directiva. La misma lógica también se aplica a las colecciones de reglas de aplicación. Las colecciones de reglas de red siempre se procesan antes que las colecciones de reglas de aplicación independientemente de la herencia.

De forma predeterminada, la directiva hereda su modo de inteligencia de amenazas de directivas primaria. Puede invalidar esto al establecer el modo de inteligencia de amenazas en un valor diferente en la página de configuración de la directiva. Solo es posible invalidarlo con un valor más estricto. Por ejemplo, si la directiva primaria está establecida en *solo Alerta*, puede configurar esta directiva local para *Alertar y denegar*, pero no puede desactivarla.

## <a name="next-steps"></a>Pasos siguientes

- [Más información sobre Azure Firewall Manager](overview.md)
---
title: Novedades de Azure Load Balancer
description: Conozca las novedades de Azure Load Balancer, como, por ejemplo, las notas de la versión más recientes, los problemas conocidos, las correcciones de errores, las funcionalidades en desuso y los próximos cambios.
services: load-balancer
author: anavinahar
ms.service: load-balancer
ms.topic: conceptual
ms.date: 07/07/2020
ms.author: anavin
ms.openlocfilehash: ffea6cdd1c8558a07559829b025cb5338cc59ee3
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88586722"
---
# <a name="whats-new-in-azure-load-balancer"></a>Novedades de Azure Load Balancer

Azure Load Balancer se actualiza con regularidad. Manténgase al día con los anuncios más recientes. En este artículo se proporciona información acerca de lo siguiente:

- Versiones más recientes
- Problemas conocidos
- Corrección de errores
- Funcionalidad en desuso (si procede)

También puede encontrar las últimas actualizaciones de Azure Load Balancer y suscribirse a la fuente RSS [aquí](https://azure.microsoft.com/updates/?category=networking&query=load%20balancer).

## <a name="recent-releases"></a>Versiones recientes

| Tipo |Nombre |Descripción  |Fecha de adición  |
| ------ |---------|---------|---------|
| Característica | Compatibilidad con la administración de grupos de back-end basada en IP (versión preliminar) | Azure Load Balancer permite agregar y quitar recursos de un grupo de back-end a través de direcciones IPv4 o IPv6. Esto permite la administración sencilla de contenedores, máquinas virtuales y conjuntos de escalado de máquinas virtuales asociados a Load Balancer. También permitirá reservar direcciones IP como parte de un grupo de back-end antes de que se creen los recursos asociados. Obtenga más información [aquí](backend-pool-management.md).|Julio de 2020 |
| Característica| Información sobre Azure Load Balancer con Azure Monitor | Integrados como parte de Azure Monitor para redes, ahora los clientes cuentan con mapas topológicos de todas sus configuraciones de Load Balancer y paneles de mantenimiento de sus instancias de Standard Load Balancer preconfiguradas con métricas en Azure Portal. [Introducción y más información](https://azure.microsoft.com/blog/introducing-azure-load-balancer-insights-using-azure-monitor-for-networks/) | Junio de 2020 |
| Validación | Incorporación de la validación para puertos de alta disponibilidad | Se ha agregado una validación para garantizar que las reglas de puerto de alta disponibilidad y las reglas que no son de este tipo solo se puedan configurar cuando se habilite la IP flotante. Anteriormente, esta configuración se procesaba, pero no funcionaba según lo previsto. No se realizó ningún cambio en la funcionalidad. Puede obtener más información [aquí](load-balancer-ha-ports-overview.md#limitations)| Junio de 2020 |
| Característica| Compatibilidad de IPv6 con Azure Load Balancer (disponible con carácter general) | Puede tener direcciones IPv6 como front-end para las instancias de Azure Load Balancer. Aprenda a [crear una aplicación de pila doble aquí](../virtual-network/virtual-network-ipv4-ipv6-dual-stack-standard-load-balancer-powershell.md). |Abril de 2020|
| Característica| Restablecimientos TCP en tiempo de espera de inactividad (disponible con carácter general)| Use los restablecimientos TCP para crear un comportamiento más predecible de las aplicaciones. [Más información](load-balancer-tcp-reset.md)| Febrero de 2020 |

## <a name="known-issues"></a>Problemas conocidos

El grupo de productos funciona activamente en la resolución de los siguientes problemas conocidos:

|Incidencia |Descripción  |Mitigación  |
| ---------- |---------|---------|
| Exportación en Log Analytics | Log Analytics no pueden exportar las métricas de los equilibradores de carga estándar ni los registros de estado de los sondeos de estado de un equilibrador de carga básico  | [Utilice Azure Monitor para métricas multidimensionales para cualquier equilibrador de carga estándar](load-balancer-standard-diagnostics.md). Aunque Log Analytics no se puede usar para la supervisión, Azure Monitor permite ver un amplio conjunto de métricas multidimensionales. Puede aprovechar el panel de métricas preconfiguradas a través de la subhoja Información de su equilibrador de carga. Si usa la [actualización a la versión Estándar](upgrade-basic-standard.md) de un equilibrador de carga básico para la supervisión de métricas de nivel de producción.

  

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure Load Balancer, consulte [¿Qué e Azure Load Balancer?](load-balancer-overview.md) y las [preguntas frecuentes](load-balancer-faqs.md).

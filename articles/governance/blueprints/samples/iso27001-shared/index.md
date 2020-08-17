---
title: Ejemplo de plano técnico de la norma ISO 27001 sobre servicios compartidos, introducción
description: Introducción y arquitectura del ejemplo de plano técnico de la norma ISO 27001 sobre servicios compartidos. Este ejemplo de plano técnico de ejemplo ayuda a los clientes a evaluar determinados controles concretos de la norma ISO 27001.
ms.date: 07/13/2020
ms.topic: sample
ms.openlocfilehash: 09309fe05200cf8c7a958324f3412967296a8dc9
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87927358"
---
# <a name="overview-of-the-iso-27001-shared-services-blueprint-sample"></a>Introducción al ejemplo de plano técnico de la norma ISO 27001 sobre servicios compartidos

El ejemplo de plano técnico de la norma ISO 27001 sobre servicios compartidos proporciona un conjunto de patrones de infraestructura compatibles y una directiva de protección que ayuda a la obtención de la certificación ISO 27001. Este plano técnico ayuda a los clientes a implementar arquitecturas basadas en la nube que ofrecen soluciones para escenarios que tienen requisitos de acreditación o de cumplimiento.

El ejemplo de plano técnico para [cargas de trabajo de App Service Environment y SQL Database compatibles con ISO 27001](../iso27001-ase-sql-workload/index.md) amplía este ejemplo.

## <a name="architecture"></a>Architecture

El ejemplo de plano técnico de la norma ISO 27001 sobre servicios compartidos implementa una infraestructura básica en Azure que diferentes organizaciones pueden usar para hospedar varias cargas de trabajo según el enfoque de Virtual Datacenter (VDC).
VDC es un conjunto probado de arquitecturas de referencia, herramientas de automatización y modelos de involucración que emplea Microsoft con sus clientes empresariales de mayor tamaño. El ejemplo de plano técnico de los servicios compartidos se basa en un entorno totalmente nativo de Azure VDC que se muestra a continuación.

:::image type="content" source="../../media/sample-iso27001-shared/iso27001-shared-services-blueprint-sample-design.png" alt-text="Diseño de ejemplo de plano técnico de la norma ISO 27001 sobre servicios compartidos" border="false":::

Este entorno se compone de varios servicios de Azure que se usan para proporcionar una infraestructura de servicios compartidos segura, totalmente supervisada y preparada para la empresa, que se basa en los estándares de la norma ISO 27001. Este entorno se compone de:

- [Roles de Azure](../../../../role-based-access-control/overview.md) que se usan para la segregación de controles desde una perspectiva del plano de control. Antes de la implementación de cualquier infraestructura se definen tres roles:
  - El rol NetOps tiene permisos para administrar el entorno de red, incluida la configuración del firewall, la del grupo de seguridad de red, el enrutamiento y otras funcionalidades de la red.
  - El rol SecOps incluye los permisos necesarios para implementar y administrar [Azure Security Center](../../../../security-center/security-center-intro.md), definir [directivas de Azure](../../../policy/overview.md) y otros permisos relacionados con la seguridad.
  - El rol SysOps incluye los permisos necesarios para definir [directivas de Azure](../../../policy/overview.md) dentro de la suscripción, administrar [Log Analytics](../../../../azure-monitor/overview.md) para todo el entorno, entre otros permisos operativos.
- [Log Analytics](../../../../azure-monitor/overview.md) se implementa como el primer servicio de Azure para garantizar que todas las acciones y servicios se registran en una ubicación central desde el momento en que empieza la implementación segura.
- Una red virtual que admite subredes para la conectividad con un centro de datos local, una pila de entrada y de salida para la conectividad con Internet y una subred de servicios compartidos que usa grupos de seguridad de red y ASG para una microsegmentación completa que contiene:
  - Un jumpbox o un host de tipo bastión que se usa con fines de administración y al que solo se puede acceder a través de una instancia de [Azure Firewall](../../../../firewall/overview.md) implementada en la subred de la pila de entrada.
  - Dos máquinas virtuales que ejecutan Active Directory Domain Services (ADDS) y un sistema de nombres de dominio al que solo se puede acceder mediante el jumpbox, y que únicamente se puede configurar para que replique AD en una conexión VPN o de [ExpressRoute](../../../../expressroute/expressroute-introduction.md) (que el plano técnico no implementa).
  - Uso de [Azure Net Watcher](../../../../network-watcher/network-watcher-monitoring-overview.md) y protección contra DDoS estándar
- Una instancia de [Azure Key Vault](../../../../key-vault/general/overview.md) utilizada para hospedar los secretos empleados por las máquinas virtuales implementadas en el entorno de servicios compartidos

Todos estos elementos se atienen a los procedimientos probados que se publicaron en [Centro de arquitectura de Azure: Arquitecturas de referencia](/azure/architecture/reference-architectures/).

> [!NOTE]
> La infraestructura de servicios compartidos de la norma ISO 27001 presenta una arquitectura básica para cargas de trabajo.
> Deberá implementar cargas de trabajo adicionales más allá de esta arquitectura básica.

Para más información, consulte la [documentación sobre Virtual Datacenter](/azure/architecture/vdc/).

## <a name="next-steps"></a>Pasos siguientes

Ya ha revisado la introducción y arquitectura del ejemplo de plano técnico de la norma ISO 27001 sobre servicios compartidos.
Ahora, visite los siguientes artículos para más información sobre la asignación de control y la implementación de este ejemplo:

> [!div class="nextstepaction"]
> [Plano técnico de servicios compartidos de la norma ISO 27001: Asignación de controles](./control-mapping.md)
> [Plano técnico de servicios compartidos de la norma ISO 27001: Pasos de implementación](./deploy.md)

Artículos adicionales sobre planos técnicos y cómo utilizarlos:

- Información acerca del [ciclo de vida del plano técnico](../../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../../concepts/resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../../how-to/update-existing-assignments.md).

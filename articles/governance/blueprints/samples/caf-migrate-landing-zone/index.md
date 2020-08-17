---
title: Ejemplo de plano técnico de la zona de aterrizaje de la migración de CAF, introducción
description: Introducción y arquitectura del ejemplo de plano técnico de la zona de aterrizaje de la migración de Cloud Adoption Framework (CAF) para Azure.
ms.date: 04/15/2020
ms.topic: sample
ms.openlocfilehash: 6deb671070a742448560309b1b1a3572f5aa082d
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87852317"
---
# <a name="overview-of-the-microsoft-cloud-adoption-framework-for-azure-migration-landing-zone-blueprint-sample"></a>Introducción al ejemplo de plano técnico de la zona de aterrizaje de migración de Microsoft Cloud Adoption Framework para Azure

El plano técnico de la zona de aterrizaje de la migración de Microsoft Cloud Adoption Framework para Azure (CAF) consiste en un conjunto de infraestructuras que le ayudan a configurar la migración de la primera carga de trabajo y a administrar los recursos en la nube de acuerdo con CAF.

El plano técnico de [fundamentos de CAF](../caf-foundation/index.md) amplía este ejemplo.

## <a name="architecture"></a>Architecture

El ejemplo de plano técnico de la zona de aterrizaje de migración de CAF implementa los recursos de infraestructura básicos en Azure que pueden usar las organizaciones para preparar su suscripción a la migración de máquinas virtuales. También ayuda a poner en marcha los controles de gobernanza necesarios para administrar su propia nube. En este ejemplo se implementarán y aplicarán los recursos, las directivas y las plantillas que permitirán a una organización comenzar con confianza en Azure.

:::image type="content" source="../../media/caf-blueprints/caf-migration-landing-zone-architecture.png" alt-text="Zona de aterrizaje de la migración de CAF, en la imagen se describe lo que se instala como parte de la guía de CAF para la zona de aterrizaje inicial" border="false":::

Este entorno se compone de varios servicios de Azure que se usan para proporcionar una gobernanza segura, totalmente supervisada y preparada para la empresa. Este entorno se compone de:

- Una instancia de [Azure Key Vault](../../../../key-vault/general/overview.md) utilizada para hospedar los secretos empleados por los certificados, las claves y los secretos implementados en el entorno de servicios compartidos
- La implementación de [Log Analytics](../../../../azure-monitor/overview.md), que se realiza para garantizar que todas las acciones y servicios se registran en una ubicación central desde el momento en que empieza la migración
- La implementación de [Azure Virtual Network](../../../../virtual-network/virtual-networks-overview.md), que proporciona tanto redes como subredes aisladas para la máquina virtual
- La implementación de [Azure Migrate Project](../../../../migrate/migrate-services-overview.md), para la detección y la evaluación Estamos agregando las herramientas para la evaluación del servidor, la migración del servidor, la evaluación de la base de datos y la migración de bases de datos.  


Todos estos elementos se atienen a los procedimientos probados que se publicaron en [Centro de arquitectura de Azure: Arquitecturas de referencia](/azure/architecture/reference-architectures/).

> [!NOTE]
> El plano técnico de migración de CAF diseña una zona de aterrizaje para las cargas de trabajo. Todavía tiene que realizar la evaluación y la migración de las bases de datos o máquinas virtuales de esta arquitectura fundamental.

Para más información, consulte el artículo sobre la [migración de Microsoft Cloud Adoption Framework para Azure](/azure/architecture/cloud-adoption/migrate/).

## <a name="next-steps"></a>Pasos siguientes

Ya ha revisado la introducción y arquitectura del ejemplo de plano técnico de la zona de aterrizaje de migración de CAF.

> [!div class="nextstepaction"]
> [Plano técnico de la zona de aterrizaje de migración de CAF: pasos para la implementación](./deploy.md)

Artículos adicionales sobre planos técnicos y cómo utilizarlos:

- Información acerca del [ciclo de vida del plano técnico](../../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../../concepts/resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../../how-to/update-existing-assignments.md).

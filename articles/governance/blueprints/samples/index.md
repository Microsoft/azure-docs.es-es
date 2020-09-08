---
title: Índice de ejemplos de planos técnicos
description: Índice de ejemplos de cumplimiento y estándar para implementar entornos, directivas y las bases de Cloud Adoption Framework con Azure Blueprints.
ms.date: 07/10/2020
ms.topic: sample
ms.openlocfilehash: cf8109dc8446fe1e3625e4f7c6fd0b5493c352d6
ms.sourcegitcommit: de2750163a601aae0c28506ba32be067e0068c0c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89488361"
---
# <a name="azure-blueprints-samples"></a>Ejemplos de Azure Blueprints

En la tabla siguiente se incluyen vínculos a ejemplos de Azure Blueprints. Cada ejemplo tiene calidad de producción y está listo para su implementación inmediata para ayudarle a satisfacer sus requisitos de cumplimiento normativo.

## <a name="standards-based-blueprint-samples"></a>Ejemplos de planos técnicos basados en estándares

| Muestra | Descripción |
|---------|---------|
| [Australian Government ISM PROTECTED](./ism-protected/control-mapping.md) | Proporciona límites de protección para el cumplimiento de Australian Government ISM PROTECTED. |
| [Azure Security Benchmark](./azure-security-benchmark.md) | Proporciona protección para garantizar el cumplimiento de [Azure Security Benchmark](../../../security/benchmarks/overview.md). |
| [Canada Federal PBMM](./canada-federal-pbmm/index.md) | Proporciona barreras de seguridad para garantizar el cumplimiento con la norma Canada Federal Protected B, Medium Integrity, Medium Availability (PBMM). |
| [CIS Microsoft Azure Foundations Benchmark](./cis-azure-1-1-0.md)| Proporciona un conjunto de directivas para ayudar a cumplir con las recomendaciones de CIS Microsoft Azure Foundations Benchmark. |
| [Nivel 4 de impacto de DoD](./dod-impact-level-4/index.md) | Proporciona un conjunto de directivas que ayudan a cumplir los requisitos del nivel 4 de impacto del DoD. |
| [Nivel de impacto 5 de DoD5](./dod-impact-level-5/index.md) | Proporciona un conjunto de directivas que ayudan a cumplir los requisitos del nivel de impacto 5 de DoD. |
| [FedRAMP Moderate](./fedramp-m/index.md) | Proporciona un conjunto de directivas para ayudar a cumplir con FedRAMP Moderate. |
| [Nivel de impacto alto de FedRAMP](./fedramp-h/index.md) | Proporciona un conjunto de directivas que ayudar a cumplir los requisitos del nivel de impacto alto de FedRAMP. |
| [HIPAA/HITRUST 9.2](./hipaa-hitrust-9-2.md) | Proporciona un conjunto de directivas para ayudar a cumplir con HIPAA/HITRUST. |
| [IRS 1075](./irs-1075/index.md) | Proporciona barreras de seguridad para garantizar el cumplimiento con la norma IRS 1075.|
| [ISO 27001](./iso27001/index.md) | Proporciona barreras de seguridad para garantizar el cumplimiento con la norma ISO 27001. |
| [ISO 27001: servicios compartidos](./iso27001-shared/index.md) | Proporciona un conjunto de patrones de infraestructura compatibles y una directiva de protección que ayuda a la obtención de la certificación ISO 27001. |
| [Cargas de trabajo de App Service Environment y SQL Database compatibles con ISO 27001](./iso27001-ase-sql-workload/index.md) | Proporciona una infraestructura adicional para el ejemplo de plano técnico de los [servicios compartidos de la norma ISO 27001](./iso27001-shared/index.md). |
| [Elementos multimedia](./media/index.md) | Proporciona un conjunto de directivas que ayudan a cumplir los requisitos del nivel de impacto alto de Media MPAA. |
| [NIST SP 800-53 R4](./nist-sp-800-53-r4.md) | Proporciona barreras de seguridad para garantizar el cumplimiento con la norma NIST SP 800-53 R4. |
| [NIST SP 800-171 R2](./nist-sp-800-171-r2.md) | Proporciona límites de protección para garantizar el cumplimiento de la norma NIST SP 800-171 R2. |
| [PCI-DSS v3.2.1](./pci-dss-3.2.1/index.md) | Proporciona un conjunto de directivas que ayuda a lograr la conformidad con PCI DSS v3.2.1. |
| [SWIFT CSP-CSCF v2020](./swift-2020/index.md) | Ayuda en el cumplimiento de la norma SWIFT CSP-CSCF v2020. |
| [Gobernanza de UK OFFICIAL y UK NHS](./ukofficial/index.md) | Proporciona un conjunto de patrones de infraestructura compatibles y una directiva de protección que ayuda a la obtención de la certificación UK OFFICIAL y UK NHS. |
| [Fundamentos de CAF](./caf-foundation/index.md) | Proporciona un conjunto de controles para ayudarle a administrar su propia nube en consonancia con [Microsoft Cloud Adoption Framework para Azure (CAF)](/azure/architecture/cloud-adoption/governance/journeys/index). |
| [Zona de aterrizaje de migración de CAF](./caf-migrate-landing-zone/index.md) | Proporciona un conjunto de controles para ayudarle a configurar la migración de la primera carga de trabajo y para administrar su propia nube en consonancia con [Microsoft Cloud Adoption Framework para Azure (CAF)](/azure/architecture/cloud-adoption/migrate/index). |

## <a name="samples-strategy"></a>Estrategia de ejemplos

:::image type="content" source="../media/blueprint-samples-strategy.png" alt-text="Estrategia de ejemplos de planos técnicos" border="false":::

Los planos técnicos de la zona de aterrizaje de migración de CAF y de la fundación CAF asumen que el cliente prepara una suscripción individual limpia existente para migrar los recursos y cargas de trabajo locales a Azure
(regiones A y B en la ilustración).  

Hay una oportunidad de iterar en los planos técnicos de ejemplo y buscar patrones de las personalizaciones que solicita el cliente. También existe la oportunidad de abordar de forma proactiva los planos técnicos específicos del sector, como los servicios financieros y el comercio electrónico (extremo superior de la región B). Del mismo modo, se prevé la creación de planos técnicos para consideraciones de arquitectura complejas, tales como suscripciones múltiples, alta disponibilidad, recursos entre regiones y clientes que implementan controles sobre las suscripciones y recursos existentes (regiones C y D).

Hay planos técnicos de ejemplo que abordan un escenario de cliente en el que los requisitos de cumplimiento son elevados y las complejidades de la arquitectura son altas (región E en la ilustración). La región F de la ilustración es la que abordarán los clientes y asociados que van a aprovechar los planos técnicos de ejemplo y a personalizarlos en función de sus necesidades únicas.

## <a name="next-steps"></a>Pasos siguientes

- Información acerca del [ciclo de vida del plano técnico](../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../concepts/resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../how-to/update-existing-assignments.md).
- Puede consultar la información de [solución de problemas generales](../troubleshoot/general.md) para resolver los problemas durante la asignación de un plano técnico.

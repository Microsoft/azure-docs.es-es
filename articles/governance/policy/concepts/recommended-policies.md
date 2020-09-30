---
title: Directivas recomendadas para los servicios de Azure
description: Describe cómo buscar y aplicar las directivas recomendadas para los servicios de Azure, como Azure Virtual Machines.
ms.date: 09/02/2020
ms.topic: conceptual
ms.openlocfilehash: 919d09a569fd950ab2061ba3452a4a940d25cb5c
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89447669"
---
# <a name="recommended-policies-for-azure-services"></a>Directivas recomendadas para los servicios de Azure

Los clientes que no están familiarizados con Azure Policy suelen buscar definiciones de directivas comunes para administrar y controlar sus recursos. Las **directivas recomendadas** de Azure Policy proporcionan una lista detallada de definiciones de directivas comunes con las que empezar. Las experiencia de **directivas recomendadas** para los recursos admitidos se inserta en la experiencia del portal para ese recurso.

Puede encontrar elementos integrados adicionales de Azure Policy en [Definiciones integradas de Azure Policy](../samples/built-in-policies.md).

## <a name="azure-virtual-machines"></a>Azure Virtual Machines

Las **directivas recomendadas** para [Azure Virtual Machines](../../../virtual-machines/index.yml) se encuentran en la página **Información general** de las máquinas virtuales y en la pestaña **Capacidades**. En la tarjeta _Azure Policy_, seleccione el texto "No configurado" o "# asignadas" para abrir un panel lateral con las directivas recomendadas. Se atenuarán las definiciones de directiva que ya estén asignada a un ámbito al que pertenezca la máquina virtual. Seleccione las directivas recomendadas que se aplicarán a esta máquina virtual y seleccione **Asignar directivas** para crear una asignación para cada una.

A medida que una organización alcanza la madurez en términos de [organizar sus recursos y la jerarquía de recursos](/azure/cloud-adoption-framework/ready/azure-best-practices/organize-subscriptions), se recomienda realizar la transición de estas asignaciones de directivas de una por cada recurso a la suscripción o el [grupo de administración](../../management-groups/index.yml).

### <a name="azure-virtual-machines-recommended-policies"></a>Directivas recomendadas de Azure Virtual Machines

|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Auditoría de máquinas virtuales sin la recuperación ante desastres configurada](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0015ea4d-51ff-4ce3-8d8c-f3f8f0179a56) |Audita las máquinas virtuales que no tienen configurada la recuperación ante desastres. Para más información sobre la recuperación ante desastres, visite [https://aka.ms/asr-doc](https://aka.ms/asr-doc). |auditIfNotExists |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/RecoveryServices_DisasterRecovery_Audit.json) |
|[Auditar las máquinas virtuales que no utilizan discos administrados](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F06a78e20-9358-41c9-923c-fb736d382a4d) |Esta directiva audita las máquinas virtuales que no utilizan discos administrados. |auditoría |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Compute/VMRequireManagedDisk_Audit.json) |
|[Azure Backup debe estar habilitado para Virtual Machines.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F013e242c-8828-4970-87b3-ab247555486d) |Esta directiva ayuda a auditar si el servicio Azure Backup está habilitado para todas las máquinas virtuales. Azure Backup es una solución rentable de copia de seguridad con un clic que simplifica la recuperación de datos y es más fácil de habilitar que otros servicios de copia de seguridad en la nube. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Backup/VirtualMachines_EnableAzureBackup_Audit.json) |

## <a name="next-steps"></a>Pasos siguientes

- Puede consultar ejemplos en [Ejemplos de Azure Policy](../samples/index.md).
- Vea la [Descripción de los efectos de directivas](./effects.md).
- Obtenga información sobre cómo [corregir recursos no compatibles](../how-to/remediate-resources.md).
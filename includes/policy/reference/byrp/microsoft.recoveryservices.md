---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 07/10/2020
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: e5a725fc6e6bebbc58f25d1f4a1893fe0313e230
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89301206"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Azure Backup debe estar habilitado para Virtual Machines.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F013e242c-8828-4970-87b3-ab247555486d) |Esta directiva ayuda a auditar si el servicio Azure Backup está habilitado para todas las máquinas virtuales. Azure Backup es una solución rentable de copia de seguridad con un clic que simplifica la recuperación de datos y es más fácil de habilitar que otros servicios de copia de seguridad en la nube. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Backup/VirtualMachines_EnableAzureBackup_Audit.json) |
|[Configure copias de seguridad de las VM de una ubicación en un almacén central existente en la misma ubicación.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F09ce66bc-1220-4153-8104-e3f51c936913) |Esta directiva configura la protección de Azure Backup en máquinas virtuales de una ubicación determinada en un almacén central existente en la misma ubicación. Solo se aplica a las máquinas virtuales que aún no están configuradas para la copia de seguridad. Se recomienda que esta directiva no se asigne a más de 200 máquinas virtuales. Si la directiva está asignada a más de 200 máquinas virtuales, puede ocurrir que la copia de seguridad se desencadene unas horas después de la programación definida. Esta directiva se mejorará para admitir más imágenes de máquina virtual. |deployIfNotExists, auditIfNotExists, disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Backup/VirtualMachineBackup_Backup_DeployIfNotExists.json) |
|[Implementar la Configuración de diagnóstico para el almacén de Recovery Services para el área de trabajo de Log Analytics con categorías específicas de recursos](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc717fb0c-d118-4c43-ab3d-ece30ac81fb3). |Permite implementar la Configuración de diagnóstico para el almacén de Recovery Services a fin de transmitir en secuencias al área de trabajo de Log Analytics para categorías específicas de recursos. Si alguna de las categorías específicas de recursos no está habilitada, se crea una nueva opción de configuración de diagnóstico. |deployIfNotExists |[1.0.0-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/EnableRecoveryServiceVaultDiagnosticSetting_Backup_DeployIfNotExist.json) |

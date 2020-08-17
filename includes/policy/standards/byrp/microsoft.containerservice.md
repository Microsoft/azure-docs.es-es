---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 07/22/2020
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: e7750049ea7c2e504aec325c39b2760e93e3c9dd
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87351923"
---
## <a name="azure-security-benchmark"></a>Prueba comparativa de la seguridad de Azure

[Azure Security Benchmark](../../../../articles/security/benchmarks/overview.md) proporciona recomendaciones sobre cómo puede proteger sus soluciones de nube en Azure. Para ver cómo este servicio se asigna por completo a Azure Security Benchmark, consulte los [archivos de asignación de Az Security Benchmark](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

Para revisar el modo en que las integraciones de Azure Policy disponibles para todos los servicios de Azure se asignan a este estándar de cumplimiento, consulte [Cumplimiento normativo de Azure Policy: Azure Security Benchmark](../../../../articles/governance/policy/samples/azure-security-benchmark.md).

|Domain |Id. de control |Título de control |Directiva<br /><sub>(Azure Portal)</sub> |Versión de la directiva<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Seguridad de redes |1.1 |Proteja los recursos mediante grupos de seguridad de red o Azure Firewall en su red virtual |[Los intervalos IP autorizados deben definirse en los servicios de Kubernetes](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0e246bcf-5f6f-4f87-bc6f-775d4712c7ea) |[1.0.1-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableIpRanges_KubernetesService_Audit.json) |
|Protección de datos |4.6 |Uso de RBAC de Azure para controlar el acceso a los recursos |[Se debe usar el control de acceso basado en rol (RBAC) en los servicios de Kubernetes](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fac4a19c2-fa67-49b4-8ae5-0b2e78c49457) |[1.0.1-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableRBAC_KubernetesService_Audit.json) |
|Administración de vulnerabilidades |5.3 |Implemente una solución automatizada de administración de revisiones de software de terceros |[Kubernetes Services se debe actualizar a una versión de Kubernetes no vulnerable](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffb893a29-21bb-418c-a157-e99480ec364c) |[1.0.1-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_UpgradeVersion_KubernetesService_Audit.json) |
|Configuración segura |7.3 |Mantenga configuraciones de recursos de Azure seguras |[[Versión preliminar]: Las directivas de seguridad de pod deben definirse en los servicios de Kubernetes](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3abeb944-26af-43ee-b83d-32aaf060fb94). |[1.0.0-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnablePSP_KubernetesService_Audit.json) |
|Configuración segura |7.9 |Implemente la supervisión de configuración automatizada para servicios de Azure |[[Versión preliminar]: Las directivas de seguridad de pod deben definirse en los servicios de Kubernetes](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3abeb944-26af-43ee-b83d-32aaf060fb94). |[1.0.0-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnablePSP_KubernetesService_Audit.json) |

## <a name="cis-microsoft-azure-foundations-benchmark"></a>CIS Microsoft Azure Foundations Benchmark

Para revisar el modo en que las integraciones de Azure Policy disponibles para todos los servicios de Azure se asignan a este estándar de cumplimiento, consulte [Cumplimiento normativo de Azure Policy: CIS Microsoft Azure Foundations Benchmark 1.1.0](../../../../articles/governance/policy/samples/cis-azure-1-1-0.md).
Para más información sobre este estándar de cumplimiento, consulte [CIS Microsoft Azure Foundations Benchmark](https://www.cisecurity.org/benchmark/azure/).

|Domain |Id. de control |Título de control |Directiva<br /><sub>(Azure Portal)</sub> |Versión de la directiva<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Otras consideraciones de seguridad |8.5 |Habilite el control de acceso basado en rol (RBAC) en Azure Kubernetes Services. |[Se debe usar el control de acceso basado en rol (RBAC) en los servicios de Kubernetes](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fac4a19c2-fa67-49b4-8ae5-0b2e78c49457) |[1.0.1-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_EnableRBAC_KubernetesService_Audit.json) |

## <a name="nist-sp-800-171-r2"></a>NIST SP 800-171 R2

Para revisar el modo en que las integraciones de Azure Policy disponibles para todos los servicios de Azure se asignan a este estándar de cumplimiento, consulte [Cumplimiento normativo de Azure Policy: NIST SP 800-171 R2](../../../../articles/governance/policy/samples/nist-sp-800-171-r2.md).
Para más información acerca de este estándar normativo, consulte [NIST SP 800-171 R2](https://csrc.nist.gov/publications/detail/sp/800-171/rev-2/final).

|Domain |Id. de control |Título de control |Directiva<br /><sub>(Azure Portal)</sub> |Versión de la directiva<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Integridad del sistema y de la información |3.14.1 |Identifica, informa y corrige los errores del sistema de manera puntual. |[Kubernetes Services se debe actualizar a una versión de Kubernetes no vulnerable](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ffb893a29-21bb-418c-a157-e99480ec364c) |[1.0.1-preview](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Security%20Center/ASC_UpgradeVersion_KubernetesService_Audit.json) |


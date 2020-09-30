---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 09/16/2020
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 404a3a41519d329d0c6f317b876e8edc42062071
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90980441"
---
## <a name="azure-security-benchmark"></a>Prueba comparativa de la seguridad de Azure

[Azure Security Benchmark](../../../../articles/security/benchmarks/overview.md) proporciona recomendaciones sobre cómo proteger las soluciones de nube en Azure. Para ver cómo se asigna completamente este servicio a Azure Security Benchmark, consulte los [archivos de asignación de Azure Security Benchmark](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

Para revisar el modo en que las integraciones de Azure Policy disponibles para todos los servicios de Azure se asignan a este estándar de cumplimiento, consulte [Cumplimiento normativo de Azure Policy: Azure Security Benchmark](../../../../articles/governance/policy/samples/azure-security-benchmark.md).

|Domain |Id. de control |Título de control |Directiva<br /><sub>(Azure Portal)</sub> |Versión de la directiva<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Seguridad de redes |1.1 |Proteja los recursos mediante grupos de seguridad de red o Azure Firewall en su red virtual |[Service Bus debe usar un punto de conexión del servicio de red virtual](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F235359c5-7c52-4b82-9055-01c75cf9f60e) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_ServiceBus_AuditIfNotExists.json) |
|Registro y supervisión |2.3 |Habilitación del registro de auditoría para recursos de Azure |[Los registros de diagnóstico en Service Bus deben estar habilitados](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff8d36e2f-389b-4ee4-898d-21aeb69a0f45) |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Service%20Bus/ServiceBus_AuditDiagnosticLog_Audit.json) |

## <a name="hipaa-hitrust-92"></a>HIPAA/HITRUST 9.2

Con el fin de revisar el modo en que las integraciones de Azure Policy disponibles para los servicios de Azure siguen este estándar de cumplimiento, consulte [Cumplimiento normativo de Azure Policy: HIPAA/HITRUST 9.2](../../../../articles/governance/policy/samples/hipaa-hitrust-9-2.md).
Para más información acerca de este estándar de cumplimiento, consulte [HIPAA/HITRUST 9.2](https://www.hhs.gov/hipaa/for-professionals/security/laws-regulations/index.html).

|Domain |Id. de control |Título de control |Directiva<br /><sub>(Azure Portal)</sub> |Versión de la directiva<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Segregación en redes |0805.01m1Organizational.12 - 01.m |Las puertas de enlace de seguridad de la organización (por ejemplo, los firewalls) aplican las directivas de seguridad y están configuradas para filtrar el tráfico entre dominios, bloquean el acceso no autorizado y se usan para mantener la segregación entre segmentos de red cableados internos, inalámbricos internos y externos (por ejemplo, Internet), incluidas las redes perimetrales y aplican directivas de control de acceso para cada uno de los dominios. |[Service Bus debe usar un punto de conexión del servicio de red virtual](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F235359c5-7c52-4b82-9055-01c75cf9f60e) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_ServiceBus_AuditIfNotExists.json) |
|Segregación en redes |0806.01m2Organizational.12356 - 01.m |La red de la organización se segmenta de manera lógica y física con un perímetro de seguridad definido y un conjunto escalonado de controles, que incluye subredes para los componentes del sistema de acceso público que están separados lógicamente de la red interna, según los requisitos de la organización. El tráfico se controla con arreglo a la funcionalidad requerida y la clasificación de los datos o sistemas, de acuerdo con una evaluación de los riesgos y los correspondientes requisitos de seguridad. |[Service Bus debe usar un punto de conexión del servicio de red virtual](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F235359c5-7c52-4b82-9055-01c75cf9f60e) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_ServiceBus_AuditIfNotExists.json) |
|Segregación en redes |0894.01m2Organizational.7 - 01.m |Las redes se separan de las redes de nivel de producción al migrar servidores físicos, aplicaciones o datos a servidores virtualizados. |[Service Bus debe usar un punto de conexión del servicio de red virtual](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F235359c5-7c52-4b82-9055-01c75cf9f60e) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_ServiceBus_AuditIfNotExists.json) |
|Registro de auditoría |1208.09aa3System.1 - 09.aa |Los registros de auditoría se mantienen para las actividades de administración, inicio, apagado o errores del sistema y de aplicaciones, cambios en archivos y cambios en la directiva de seguridad. |[Los registros de diagnóstico en Service Bus deben estar habilitados](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff8d36e2f-389b-4ee4-898d-21aeb69a0f45) |[3.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Service%20Bus/ServiceBus_AuditDiagnosticLog_Audit.json) |
|Controles de red |0860.09m1Organizational.9 - 09.m |La organización administra formalmente los equipos de la red, incluidos los equipos de las áreas de usuario. |[Service Bus debe usar un punto de conexión del servicio de red virtual](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F235359c5-7c52-4b82-9055-01c75cf9f60e) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Network/VirtualNetworkServiceEndpoint_ServiceBus_AuditIfNotExists.json) |


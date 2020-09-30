---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 09/16/2020
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 8263d010927e3e627876fee21195feaeb6f87318
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90985974"
---
## <a name="azure-security-benchmark"></a>Prueba comparativa de la seguridad de Azure

[Azure Security Benchmark](../../../../articles/security/benchmarks/overview.md) proporciona recomendaciones sobre cómo proteger las soluciones de nube en Azure. Para ver cómo se asigna completamente este servicio a Azure Security Benchmark, consulte los [archivos de asignación de Azure Security Benchmark](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

Para revisar el modo en que las integraciones de Azure Policy disponibles para todos los servicios de Azure se asignan a este estándar de cumplimiento, consulte [Cumplimiento normativo de Azure Policy: Azure Security Benchmark](../../../../articles/governance/policy/samples/azure-security-benchmark.md).

|Domain |Id. de control |Título de control |Directiva<br /><sub>(Azure Portal)</sub> |Versión de la directiva<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Protección de datos |4.4. |Cifrado de toda la información confidencial en tránsito |[Solo se deben habilitar las conexiones seguras a la instancia de Azure Cache for Redis](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb). |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |

## <a name="hipaa-hitrust-92"></a>HIPAA/HITRUST 9.2

Con el fin de revisar el modo en que las integraciones de Azure Policy disponibles para los servicios de Azure siguen este estándar de cumplimiento, consulte [Cumplimiento normativo de Azure Policy: HIPAA/HITRUST 9.2](../../../../articles/governance/policy/samples/hipaa-hitrust-9-2.md).
Para más información acerca de este estándar de cumplimiento, consulte [HIPAA/HITRUST 9.2](https://www.hhs.gov/hipaa/for-professionals/security/laws-regulations/index.html).

|Domain |Id. de control |Título de control |Directiva<br /><sub>(Azure Portal)</sub> |Versión de la directiva<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Control de conexión de red |0809.01n2Organizational.1234 - 01.n |El tráfico de red se regula de acuerdo con la directiva de control de acceso de la organización mediante el firewall y otras restricciones relativas a la red para cada punto de acceso de red o interfaz administrada del servicio de telecomunicaciones externo. |[Solo se deben habilitar las conexiones seguras a la instancia de Azure Cache for Redis](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb). |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |
|Control de conexión de red |0810.01n2Organizational.5 - 01.n |La información transmitida se protege y, como mínimo, se cifra para las redes públicas y abiertas. |[Solo se deben habilitar las conexiones seguras a la instancia de Azure Cache for Redis](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb). |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |
|Control de conexión de red |0811.01n2Organizational.6 - 01.n |Las excepciones a la directiva de flujo de tráfico se documentan con fines de respaldo o por necesidad de la empresa, especifican la duración de la excepción y se revisan al menos una vez al año. Estas excepciones se eliminan cuando ya no son necesarias para una finalidad específica o un requisito empresarial. |[Solo se deben habilitar las conexiones seguras a la instancia de Azure Cache for Redis](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb). |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |
|Control de conexión de red |0812.01n2Organizational.8 - 01.n |Los dispositivos remotos que establecen conexiones no remotas no pueden comunicarse con recursos externos (remotos). |[Solo se deben habilitar las conexiones seguras a la instancia de Azure Cache for Redis](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb). |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |
|Control de conexión de red |0814.01n1Organizational.12 - 01.n |La capacidad de los usuarios para conectarse a la red interna está restringida mediante una directiva "denegar de forma predeterminada" y "permitir por excepción" en las interfaces administradas, de acuerdo con la directiva de control de acceso y los requisitos de las aplicaciones empresariales y de uso clínico. |[Solo se deben habilitar las conexiones seguras a la instancia de Azure Cache for Redis](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb). |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |
|Identificación de riesgos relacionados con entidades externas |1451.05iCSPOrganizational.2 - 05.i |Los proveedores de servicios en la nube diseñan e implementan controles para mitigar y contener los riesgos de seguridad de los datos mediante la separación adecuada de las obligaciones, el acceso basado en roles y el acceso con privilegios mínimos para todo el personal de la cadena de suministro. |[Solo se deben habilitar las conexiones seguras a la instancia de Azure Cache for Redis](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb). |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |
|Transacciones en línea |0946.09y2Organizational.14 - 09.y |La organización requiere el uso de cifrado entre las partes implicadas en la transacción y el uso de firmas electrónicas por parte de cada una de ellas. |[Solo se deben habilitar las conexiones seguras a la instancia de Azure Cache for Redis](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb). |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |

## <a name="nist-sp-800-171-r2"></a>NIST SP 800-171 R2

Para revisar el modo en que las integraciones de Azure Policy disponibles para todos los servicios de Azure se corresponden a este estándar de cumplimiento, consulte [Cumplimiento normativo de Azure Policy: NIST SP 800-171 R2](../../../../articles/governance/policy/samples/nist-sp-800-171-r2.md).
Para más información acerca de este estándar normativo, consulte [NIST SP 800-171 R2](https://csrc.nist.gov/publications/detail/sp/800-171/rev-2/final).

|Domain |Id. de control |Título de control |Directiva<br /><sub>(Azure Portal)</sub> |Versión de la directiva<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Protección del sistema y de las comunicaciones |3.13.1 |Supervisa, controla y protege las comunicaciones (es decir, la información transmitida o recibida por los sistemas de la organización) en los límites externos y los límites internos clave de los sistemas de la organización. |[Solo se deben habilitar las conexiones seguras a la instancia de Azure Cache for Redis](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb). |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |
|Protección del sistema y de las comunicaciones |3.13.8 |Implementa mecanismos criptográficos para evitar la divulgación no autorizada de CUI durante la transmisión, a menos que se proteja de otro modo con medidas de seguridad físicas alternativas. |[Solo se deben habilitar las conexiones seguras a la instancia de Azure Cache for Redis](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb). |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |

## <a name="nist-sp-800-53-r4"></a>NIST SP 800-53 R4

Para revisar el modo en que las integraciones de Azure Policy disponibles para todos los servicios de Azure se corresponden a este estándar de cumplimiento, consulte [Cumplimiento normativo de Azure Policy: NIST SP 800-53 R4](../../../../articles/governance/policy/samples/nist-sp-800-53-r4.md).
Para más información acerca de este estándar normativo, consulte [NIST SP 800-53 R4](https://nvd.nist.gov/800-53).

|Domain |Id. de control |Título de control |Directiva<br /><sub>(Azure Portal)</sub> |Versión de la directiva<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Protección del sistema y de las comunicaciones |SC-8 (1) |Integridad y confidencialidad de la transmisión \| Protección física alternativa o criptográfica |[Solo se deben habilitar las conexiones seguras a la instancia de Azure Cache for Redis](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb). |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |


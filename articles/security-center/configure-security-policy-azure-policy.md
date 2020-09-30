---
title: Creación y edición de directivas de seguridad de Azure Policy mediante la API REST
description: Obtenga información sobre la administración de directivas de Azure Policy a través de una API de REST.
services: security-center
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: conceptual
ms.date: 11/04/2019
ms.author: memildin
ms.openlocfilehash: 7a81b3016f86c6fd7de646ccbdbf533685471ac3
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90894803"
---
# <a name="configure-a-security-policy-in-azure-policy-using-the-rest-api"></a>Configuración de una directiva de seguridad mediante la API de REST

Como parte de la integración nativa con Azure Policy, Azure Security Center le permite aprovechar la API REST de Azure Policy para crear asignaciones de directivas. Las siguientes instrucciones le guiarán en la creación de asignaciones de directivas, así como en la personalización de asignaciones existentes. 

Conceptos importantes de Azure Policy: 

- Una **definición de directiva** es una regla. 

- Una **iniciativa** es una colección de definiciones de directivas (reglas). 

- Una **asignación** es una aplicación de una iniciativa o una directiva para un ámbito concreto (grupo de administración o suscripción, entre otros). 

Security Center tiene una iniciativa incorporada que incluye todas las directivas de seguridad. Para evaluar las directivas de Security Center sobre los recursos de Azure, debe crear una asignación en el grupo de administración o en la suscripción que quiera evaluar.

De forma predeterminada, lºa iniciativa integrada tiene todas las directivas de Security Center habilitadas. Puede optar por deshabilitar determinadas directivas de la iniciativa integrada. Por ejemplo, para aplicar todas las directivas de Security Center excepto el **firewall de aplicaciones web**, cambie el valor del parámetro de efecto de la directiva a **Deshabilitado**. 

## <a name="api-examples"></a>Ejemplos de API

En los siguientes ejemplos, reemplace estas variables:

- **{scope}** escriba el nombre del grupo de administración o de suscripción al que va a aplicar la directiva.
- **{policyAssignmentName}** escriba el [nombre de la asignación de directiva pertinente](#policy-names).
- **{name}** escriba su nombre o el nombre del administrador que ha aprobado el cambio de directiva.

En este ejemplo se muestra cómo asignar la iniciativa integrada de Security Center a una suscripción o grupo de administración.
 
 ```
    PUT  
    https://management.azure.com/{scope}/providers/Microsoft.Authorization/policyAssignments/{policyAssignmentName}?api-version=2018-05-01 

    Request Body (JSON) 

    { 

      "properties":{ 

    "displayName":"Enable Monitoring in Azure Security Center", 

    "metadata":{ 

    "assignedBy":"{Name}" 

    }, 

    "policyDefinitionId":"/providers/Microsoft.Authorization/policySetDefinitions/1f3afdf9-d0c9-4c3d-847f-89da613e70a8", 

    "parameters":{}, 

    } 

    } 
 ```

En este ejemplo se muestra cómo asignar la iniciativa integrada de Security Center a una suscripción, con las siguientes directivas deshabilitadas: 

- Actualizaciones del sistema ("systemUpdatesMonitoringEffect") 

- Configuraciones de seguridad ("systemConfigurationsMonitoringEffect") 

- Protección de extremos ("endpointProtectionMonitoringEffect") 

 ```
    PUT https://management.azure.com/{scope}/providers/Microsoft.Authorization/policyAssignments/{policyAssignmentName}?api-version=2018-05-01 
    
    Request Body (JSON) 
    
    { 
    
      "properties":{ 
    
    "displayName":"Enable Monitoring in Azure Security Center", 
    
    "metadata":{ 
    
    "assignedBy":"{Name}" 
    
    }, 
    
    "policyDefinitionId":"/providers/Microsoft.Authorization/policySetDefinitions/1f3afdf9-d0c9-4c3d-847f-89da613e70a8", 
    
    "parameters":{ 
    
    "systemUpdatesMonitoringEffect":{"value":"Disabled"}, 
    
    "systemConfigurationsMonitoringEffect":{"value":"Disabled"}, 
    
    "endpointProtectionMonitoringEffect":{"value":"Disabled"}, 
    
    }, 
    
     } 
    
    } 
 ```
En este ejemplo se muestra cómo quitar una asignación:
 ```
    DELETE   
    https://management.azure.com/{scope}/providers/Microsoft.Authorization/policyAssignments/{policyAssignmentName}?api-version=2018-05-01 
 ```

## <a name="policy-names-reference"></a>Referencia de los nombres de directiva <a name="policy-names"></a>

|Nombre de directiva de Security Center|El nombre de la directiva aparece en Azure Policy |Nombre de parámetro del efecto de directiva|
|----|----|----|
|Cifrado de SQL |Supervisión de la base de datos SQL sin cifrar en Azure Security Center |sqlEncryptionMonitoringEffect| 
|Auditoría de SQL |Supervisión de la base de datos SQL no auditada en Azure Security Center |sqlAuditingMonitoringEffect|
|Actualizaciones del sistema |Supervisión de las actualizaciones del sistema que faltan en Azure Security Center |systemUpdatesMonitoringEffect|
|Cifrado de almacenamiento |Auditoría de la falta del cifrado de blob de las cuentas de almacenamiento |storageEncryptionMonitoringEffect|
|Acceso de red JIT |Supervisión del posible acceso de red Just-In-Time (JIT) en Azure Security Center |jitNetworkAccessMonitoringEffect |
|Controles de aplicación adaptables |Supervisión de posibles listas de aplicaciones permitidas en Azure Security Center |adaptiveApplicationControlsMonitoringEffect|
|Grupos de seguridad de red |Supervisión del acceso de red permisivo en Azure Security Center |networkSecurityGroupsMonitoringEffect| 
|Configuraciones de seguridad |Supervisión de los puntos vulnerables del sistema operativo en Azure Security Center |systemConfigurationsMonitoringEffect| 
|Endpoint Protection |Supervisión de la falta de Endpoint Protection en Azure Security Center |endpointProtectionMonitoringEffect |
|Cifrado de discos |Supervisión de discos de máquinas virtuales sin cifrar en Azure Security Center |diskEncryptionMonitoringEffect|
|Evaluación de vulnerabilidades |Supervisión de los puntos vulnerables de máquinas virtuales en Azure Security Center |vulnerabilityAssessmentMonitoringEffect|
|Firewall de aplicaciones web |Supervisión de la aplicación web desprotegida en Azure Security Center |webApplicationFirewallMonitoringEffect |
|Firewall de próxima generación |Supervisión de puntos de conexión de red desprotegidos en Azure Security Center| |


## <a name="next-steps"></a>Pasos siguientes

Para obtener material relacionado, consulte los siguientes artículos: 

- [Directivas de seguridad personalizadas](custom-security-policies.md)
- [Introducción a las directivas de seguridad](tutorial-security-policy.md)
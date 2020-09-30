---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 09/16/2020
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 9cb570572fb055da40e9b0a47797593410983ccd
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90982441"
---
|Nombre<br /><sub>(Azure Portal)</sub> |Descripción |Efectos |Versión<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Las instancias de Container Registry se deben cifrar con una clave administrada por el cliente (CMK)](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F5b9159ae-1701-4a6f-9a7a-aa9c8ddd0580) |Audite las instancias de Container Registry que no tienen habilitado el cifrado con claves administradas por el cliente (CMK). Azure cifra automáticamente el contenido del registro en reposo con claves administradas por el servicio. Puede complementar el cifrado predeterminado con una capa de cifrado adicional mediante una clave que se crea y administra en Azure Key Vault. Para más información sobre el cifrado de CMK, visite [https://aka.ms/acr/CMK](https://aka.ms/acr/CMK). |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Container%20Registry/ACR_CMKEncryptionEnabled_Audit.json) |
|[Las instancias de Container Registry no deben permitir el acceso de red sin restricciones](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd0793b48-0edc-4296-a390-4c75d1bdfd71) |Audite las instancias de Container Registry que no tienen ninguna regla de red o de firewall (IP) configurada y permiten todo el acceso de red de forma predeterminada. La restricción del acceso de red protege las instancias de Container Registry de posibles amenazas. Las instancias de Container Registry con al menos una regla de IP o firewall o una red virtual configurada se considerarán compatibles. Para más información acerca de las reglas de red de Container Registry, consulte: [https://aka.ms/acr/portal/public-network](https://aka.ms/acr/portal/public-network) y [https://aka.ms/acr/vnet](https://aka.ms/acr/vnet). |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Container%20Registry/ACR_NetworkRulesExist_Audit.json) |
|[Las instancias de Container Registry deben usar vínculos privados](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe8eef0a8-67cf-4eb4-9386-14b0e78733d4) |Audite los registros de contenedor que no tengan al menos una conexión de punto de conexión privado aprobada. Los clientes de una red virtual pueden acceder de forma segura a los recursos que tengan conexiones de punto de conexión privadas mediante vínculos privados. Después, se puede deshabilitar el acceso público para asegurarse de que solo se pueden usar vínculos privados para conectarse al registro. Para más información, visite [https://aka.ms/acr/private-link](https://aka.ms/acr/private-link). |Audit, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Container%20Registry/ACR_PrivateEndpointEnabled_Audit.json) |

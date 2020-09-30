---
title: 'Tutorial sobre controles de aplicaciones y acceso: Azure Security Center'
description: En este tutorial se muestra cómo configurar una directiva de acceso a las máquinas virtuales Just-In-Time y una directiva de control de aplicaciones.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 61e95a87-39c5-48f5-aee6-6f90ddcd336e
ms.service: security-center
ms.devlang: na
ms.topic: tutorial
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/03/2018
ms.author: memildin
ms.openlocfilehash: 56dd74fba46aa8b79c94b1460996bb6edb1ff93f
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90904604"
---
# <a name="tutorial-protect-your-resources-with-azure-security-center"></a>Tutorial: Protección de los recursos con Azure Security Center
Security Center limita la exposición a amenazas mediante controles de acceso y aplicación para bloquear actividades malintencionadas. El acceso a las máquinas virtuales Just-In-Time (JIT) reduce la exposición a ataques mediante la posibilidad de denegar el acceso persistente a las máquinas virtuales. En su lugar, se proporciona acceso controlado y auditado a VM solo cuando se necesita. Los controles de aplicación adaptables ayudan a proteger las VM frente a malware controlando qué aplicaciones se pueden ejecutar en dichas VM. Security Center usa el aprendizaje automático para analizar los procesos que se ejecutan en la máquina virtual y le ayuda a aplicar reglas de inclusión en listas de permitidos con esta inteligencia.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configuración de una directiva de acceso a las máquinas virtuales Just-In-Time
> * Configuración de una directiva de control de aplicación

## <a name="prerequisites"></a>Prerrequisitos
Para recorrer las características descritas en este tutorial, debe tener habilitado Azure Defender. Puede probar Azure Defender sin costo alguno. Para más información, consulte [Prueba de Azure Defender](security-center-pricing.md).

## <a name="manage-vm-access"></a>Administración de acceso a VM
El acceso a VM JIT se puede usar para bloquear el tráfico entrante a las VM de Azure. Para ello, se reduce la exposición a ataques al mismo tiempo que se proporciona un acceso sencillo para conectarse a las VM cuando sea necesario.

No es necesario que los puertos de administración estén abiertos en todo momento. Solo deben estar abiertos mientras se está conectado a la máquina virtual, por ejemplo, para realizar tareas de administración o mantenimiento. Cuando se habilita Just-In-Time, Security Center usa las reglas del grupo de seguridad de red (NSG), que restringen el acceso a los puertos de administración para que no puedan ser objeto de ataques.

Siga las instrucciones de [Protección de los puertos de administración con el acceso Just-in-Time](security-center-just-in-time.md).

## <a name="harden-vms-against-malware"></a>Proteger VM frente a malware
Los controles de aplicación adaptables ayudan a definir un conjunto de aplicaciones que se pueden ejecutar en grupos de recursos configurados que, entre otras ventajas, le ayuda a proteger las VM frente a malware. Security Center usa el aprendizaje automático para analizar los procesos que se ejecutan en la máquina virtual y le ayuda a aplicar reglas de inclusión en listas de permitidos con esta inteligencia.

Siga las instrucciones de [Uso de controles de aplicaciones adaptables para reducir las superficies de ataque de las máquinas](security-center-adaptive-application.md).

## <a name="next-steps"></a>Pasos siguientes
En este tutorial, aprendió a limitar la exposición a amenazas mediante:

> [!div class="checklist"]
> * La configuración de un directiva de acceso a las máquinas virtuales Just-In-Time para proporcionar acceso controlado y auditado a las máquinas virtuales solo cuando sea necesario
> * La configuración de una directiva de controles de aplicación adaptables para controlar qué aplicaciones se pueden ejecutar en las VM

Pase al siguiente tutorial para aprender a responder a incidentes relacionados con la seguridad.

> [!div class="nextstepaction"]
> [Tutorial: Respuesta a incidentes de seguridad](tutorial-security-incident.md)

<!--Image references-->
[1]: ./media/tutorial-protect-resources/just-in-time-vm-access.png
[2]: ./media/tutorial-protect-resources/add-port.png
[3]: ./media/tutorial-protect-resources/adaptive-application-control-options.png
[4]: ./media/tutorial-protect-resources/recommended-resource-groups.png

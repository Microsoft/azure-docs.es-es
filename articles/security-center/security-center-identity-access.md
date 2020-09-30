---
title: Supervisión de identidad y acceso en Azure Security Center | Microsoft Docs
description: Aprenda a usar la funcionalidad de identidad y acceso de Azure Security Center para supervisar los problemas relacionados con la actividad de acceso e identidad de los usuarios.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 9f04e730-4cfa-4078-8eec-905a443133da
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/16/2020
ms.author: memildin
ms.openlocfilehash: 042780c313c444062fd512ab0d9f38aaeb6cf170
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90894565"
---
# <a name="monitor-identity-and-access"></a>Supervisión de la identidad y el acceso

> [!TIP]
> A partir de marzo de 2020, la identidad y las recomendaciones de acceso de Azure Security Center se incluyen en todas las suscripciones del plan de tarifa gratuito. Si tiene suscripciones en el plan de tarifa gratuito, su puntuación de seguridad se verá afectada ya que no se evaluó anteriormente su identidad y seguridad de acceso. 

Cuando Security Center identifica posibles vulnerabilidades de seguridad, crea recomendaciones que lo guiarán por el proceso de configuración de los controles necesarios para reforzar y proteger sus recursos.

El perímetro de seguridad ha evolucionado de un perímetro de red a uno de identidad. La seguridad se centra menos en proteger la red y más en proteger los datos, así como en administrar la seguridad de las aplicaciones y los usuarios. Hoy en día, donde cada vez más datos y aplicaciones se mueven a la nube, la identidad se convierte en el nuevo perímetro.

Con la supervisión de las actividades de identidad, se pueden realizar acciones proactivas antes de que se produzca un incidente o acciones reactivas para detener un intento de ataque. Por ejemplo, Security Center puede marcar las cuentas desusadas (aquellas que ya no son necesarias y que no pueden iniciar sesión en Azure Active Directory) para que se eliminen. 

Estos son algunos ejemplos de recomendaciones que podría ver en la sección de seguridad del recurso **Identity and Access** (Identidad y acceso) de Azure Security Center:

- MFA debe estar habilitada en las cuentas con permisos de propietario en la suscripción
- Debe designar un máximo de tres propietarios para la suscripción
- Las cuentas externas con permisos de lectura deben quitarse de la suscripción
- Las cuentas en desuso deben quitarse de la suscripción

Para más información sobre estas recomendaciones y obtener una lista completa de las recomendaciones que se pueden consultar aquí, vea [Recomendaciones de identidad y acceso](recommendations-reference.md#recs-identity).

> [!NOTE]
> Si su suscripción tiene más de 600 cuentas, Security Center no puede ejecutar las recomendaciones de identidades en su suscripción. Las recomendaciones que no se ejecutan se muestran bajo "evaluaciones no disponibles" a continuación.
Security Center no puede ejecutar las recomendaciones de identidad en los agentes de administración del asociado de un proveedor de soluciones en la nube (CSP).
>


Todas las recomendaciones de identidad y acceso están disponibles en dos controles de seguridad de la página **Recomendaciones**:

- Administración de acceso y permisos 
- Habilitar MFA

![Los dos controles de seguridad con las recomendaciones relacionadas con la identidad y el acceso](media/security-center-identity-access/two-security-controls-for-identity-and-access.png)


## <a name="enable-multi-factor-authentication-mfa"></a>Habilitar Multi-Factor Authentication (MFA)

Habilitar MFA requiere [permisos de inquilino de Azure Active Directory (AD)](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles). 

- Si tiene una edición Premium de AD, habilite MFA mediante el [acceso condicional](../active-directory/conditional-access/concept-conditional-access-policy-common.md).

- Si usa la edición gratuita de AD, habilite **valores predeterminados de seguridad** en Azure Active Directory tal y como se describe en la [documentación de AD](https://docs.microsoft.com/azure/active-directory/fundamentals/concept-fundamentals-security-defaults).


## <a name="next-steps"></a>Pasos siguientes
Para más información sobre las recomendaciones que se aplican a otros tipos de recursos de Azure, consulte los siguientes artículos:

- [Protección de las redes en Azure Security Center](security-center-network-recommendations.md)
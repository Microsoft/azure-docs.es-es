---
title: Administración de grupos de Azure AD con privilegios en Privileged Identity Management (PIM) | Microsoft Docs
description: Cómo administrar miembros y propietarios de grupos de acceso con privilegios en Privileged Identity Management (PIM)
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
ms.assetid: ''
ms.service: active-directory
ms.subservice: pim
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/27/2020
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1b6709d7589ebf4c82ec25c6b7a79fa31a305839
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87506446"
---
# <a name="management-capabilities-for-privileged-access-azure-ad-groups-preview"></a>Funcionalidades de administración de grupos de Azure AD de acceso con privilegios (versión preliminar)

En Privileged Identity Management (PIM), ya se pueden asignar la idoneidad para pertenecer a grupos de acceso con privilegios o poseerlos. A partir de esta versión preliminar, es posible asignar roles integrados de Azure Active Directory (Azure AD) a grupos en la nube y usar PIM para administrar la idoneidad y la activación tanto de los propietarios como de los miembros de los grupos. Para más información sobre los grupos a los que se puede asignar roles en Azure AD, consulte el artículo sobre el [uso de grupos en la nube para administrar las asignaciones de roles en Azure Active Directory (versión preliminar)](../users-groups-roles/roles-groups-concept.md).

## <a name="require-different-policies-for-each-role-assignable-group"></a>Requisitos de directivas diferentes en cada grupo al que se puedan asignar roles

Algunas organizaciones usan herramientas, como la colaboración B2B de Azure AD, para invitar a sus asociados a su organización de Azure AD. En lugar de crear una única directiva cuando es necesario para todas las asignaciones a un rol con privilegios, se pueden crear dos grupos de acceso con privilegios diferentes con sus propias directivas. Puede exigir requisitos menos estrictos para los empleados de confianza y más estrictos para los asociados, como el flujo de trabajo de aprobación, cuando solicitan la activación en su grupo asignado.

## <a name="activate-multiple-role-assignments-in-a-single-request"></a>Activación de varias asignaciones de roles en una única solicitud

Con la versión preliminar de los grupos de acceso con privilegios, puede proporcionar a los administradores específicos de las cargas de trabajo un acceso rápido a varios roles con una única solicitud cuando es necesario. Por ejemplo, es posible que unos administradores de Office de nivel 3 necesiten acceso cuando es necesario a los roles Administrador de Exchange, Administrador de aplicaciones de Office, Administrador de Teams y Administrador de Search para investigar los incidentes que se producen a diario. Antes se necesitarían cuatro solicitudes consecutivas, lo que conlleva un proceso que tarda un tiempo en realizarse. En su lugar, se puede crear un grupo, al que se pueden asignar roles, llamado “Administradores de Office de nivel 3”, asignarlo a los cuatro roles que hemos mencionado (o a cualquier rol integrado de Azure AD) y habilitarlo para un acceso con privilegios en la sesión Actividad del grupo. Una vez habilitado para el acceso con privilegios, puede configurar el acceso cuando es necesario de los miembros del grupo y asignar los administradores y propietarios como aptos. Cuando se eleven los privilegios de los administradores en el grupo, se convertirán en miembros de los cuatro roles de Azure AD.

## <a name="extend-and-renew-group-assignments"></a>Extensión y renovación de asignaciones de grupos

Después de configurar las asignaciones de miembros o propietarios con límite de tiempo, la primera pregunta que podría surgir es qué sucede si una asignación expira. En esta nueva versión, se proporcionan dos opciones para este escenario:

- Extensión: cuando una asignación de roles está a punto de expirar, el usuario puede usar Privileged Identity Management para solicitar una extensión.
- Renovación: cuando una asignación de roles ya ha expirado, el usuario puede usar Privileged Identity Management para solicitar una renovación.

Ambas acciones iniciadas por el usuario requieren una aprobación de un administrador global o un administrador de roles con privilegios. Los administradores ya no tendrán que encargarse de administrar estas expiraciones. Solo tendrán que esperar las solicitudes de extensión o renovación y aprobarlas si son válidas.

## <a name="next-steps"></a>Pasos siguientes

- [Asignación de un propietario o miembro de un grupo de acceso con privilegios](groups-assign-member-owner.md)
- [Aprobación o denegación de solicitudes de activación de propietarios y miembros de grupos de acceso con privilegios](groups-approval-workflow.md)

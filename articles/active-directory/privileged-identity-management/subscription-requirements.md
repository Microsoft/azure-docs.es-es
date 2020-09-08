---
title: 'Requisitos de licencia para usar Privileged Identity Management: Azure Active Directory | Microsoft Docs'
description: Describe los requisitos de licencia necesarios para usar Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: markwahl-msft
ms.assetid: 34367721-8b42-4fab-a443-a2e55cdbf33d
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: pim
ms.date: 08/06/2020
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 74c9cd1c55f1b0dde173a7ffbeac92e5518db81e
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88005812"
---
# <a name="license-requirements-to-use-privileged-identity-management"></a>Requisitos de licencia para usar Privileged Identity Management

Para Azure Active Directory (Azure AD) Privileged Identity Management (PIM), un directorio debe tener una licencia válida. Además, deben asignarse licencias a los administradores y usuarios pertinentes. Este artículo describe los requisitos de licencia para usar Privileged Identity Management.

## <a name="valid-licenses"></a>Licencias válidas

[!INCLUDE [Azure AD Premium P2 license](../../../includes/active-directory-p2-license.md)]

## <a name="licenses-you-must-have"></a>Licencias que debe tener

Asegúrese de que el directorio tiene al menos tantas licencias de Azure AD Premium P2 como empleados que van a realizar las tareas siguientes:

- Usuarios asignados como aptos para roles de Azure AD o Azure administrados mediante PIM
- Usuarios que están asignados como miembros válidos o propietarios de grupos de acceso con privilegios
- Los usuarios que pueden aprobar o rechazar solicitudes de activación en PIM
- Los usuarios asignados a una revisión de acceso
- Los usuarios que realizan revisiones de acceso

Las licencias de Azure AD Premium P2 **no** son necesarias para las tareas siguientes:

- No se requieren licencias para los usuarios que reciben alertas y configuran PIM, directivas y revisiones de acceso.

Para más información sobre las licencias, vea [Asignación o eliminación de licencias mediante el portal de Azure Active Directory](../fundamentals/license-users-groups.md).

## <a name="example-license-scenarios"></a>Escenarios de licencia de ejemplo

Estos son algunos escenarios de licencia de ejemplo que le ayudarán a determinar el número de licencias que debe tener.

| Escenario | Cálculo | Número de licencias |
| --- | --- | --- |
| Woodgrove Bank tiene 10 administradores para diferentes departamentos y 2 administradores globales que configuran y administran PIM. Eso hace cinco administradores válidos. | Cinco licencias para los administradores que sean válidos | 5 |
| Graphic Design Institute tiene 25 administradores, de los cuales 14 se administran a través de PIM. La activación de roles necesita aprobación y hay tres usuarios diferentes en la organización que pueden aprobar las activaciones. | 14 licencias para los roles válidos + 3 aprobadores | 17 |
| Contoso tiene 50 administradores, de los cuales 42 se administran a través de PIM. La activación de roles necesita aprobación y hay cinco usuarios diferentes en la organización que pueden aprobar las activaciones. Contoso también realiza revisiones mensuales de los usuarios asignados a los roles de administrador y los revisores son los administradores de los usuarios de los cuales seis no están en roles de administrador administrados por PIM. | 42 licencias para los roles válidos + 5 aprobadores + 6 revisores | 53 |

## <a name="when-a-license-expires"></a>Cuando expira una licencia

Si expira una licencia de Azure AD Premium P2, EMS E5 o una licencia de evaluación, las características de Privileged Identity Management ya no estarán disponibles en el directorio:

- Las asignaciones de roles permanentes para roles de Azure AD no se verán afectadas.
- El servicio de Privileged Identity Management en Azure Portal, así como los cmdlets de Graph API y las interfaces de PowerShell de Privileged Identity Management, ya no estarán disponibles para que los usuarios activen roles con privilegios, administren el acceso con privilegios o realicen revisiones de acceso de roles con privilegios.
- Se quitarán las asignaciones de roles elegibles de roles de Azure AD, puesto que los usuarios ya no podrán activar roles con privilegios.
- Las revisiones de acceso en curso de roles de Azure AD finalizarán y las opciones de configuración de Privileged Identity Management se quitarán.
- Privileged Identity Management no volverá a enviar correos electrónicos cuando se produzcan cambios de asignación de rol.

## <a name="next-steps"></a>Pasos siguientes

- [Implementación de Privileged Identity Management](pim-deployment-plan.md)
- [Empiece a usar Privileged Identity Management](pim-getting-started.md)
- [Roles que no se pueden administrar en Privileged Identity Management](pim-roles.md)

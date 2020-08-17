---
title: 'En qué consisten las licencias basadas en grupos: Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre las licencias basadas en grupo de Azure Active Directory, cómo funcionan y sus procedimientos recomendados.
services: active-directory
keywords: Licencias de Azure AD
author: ajburnle
manager: daveba
ms.service: active-directory
ms.subservice: fundamentals
ms.topic: conceptual
ms.workload: identity
ms.date: 10/29/2018
ms.author: ajburnle
ms.reviewer: krbain
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1ce0d543f11a1dcc61616d8a386176272ab44aee
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87797277"
---
# <a name="what-is-group-based-licensing-in-azure-active-directory"></a>¿En qué consisten las licencias basadas en grupos de Azure Active Directory?

Los servicios en la nube de pago de Microsoft, como Office 365, Enterprise Mobility + Security, Dynamics 365 y otros productos similares, requieren licencias. Estas licencias se asignan a cada usuario que necesita acceso a estos servicios. Para administrar las licencias, los administradores usan uno de los portales de administración (ya sea Office o Azure) y los cmdlets de PowerShell. Azure Active Directory (Azure AD) es la infraestructura subyacente que admite la administración de identidades para todos los servicios en la nube de Microsoft. Azure AD almacena información sobre los estados de asignación de licencias para los usuarios.

Hasta ahora, las licencias solo podían asignarse a nivel de cada usuario, lo que puede dificultar la administración a gran escala. Por ejemplo, para agregar o quitar licencias de usuario en función de los cambios que se producen en la organización, por ejemplo, la incorporación o la baja de un usuario en la organización o en un departamento, un administrador a menudo debe escribir un script de PowerShell complejo. Este script hace llamadas individuales al servicio en la nube.

Para abordar esos desafíos, Azure AD incluye ahora las licencias basadas en grupo. Puede asignar una o varias licencias de producto a un grupo. Azure AD garantiza que las licencias se asignen a todos los miembros del grupo. A todos los miembros nuevos que se unan al grupo se les asignarán las licencias correspondientes. Cuando salen del grupo, se quitan esas licencias. La administración de licencias elimina la necesidad de automatizar la administración de licencias a través de PowerShell para reflejar los cambios que se producen en la organización y en la estructura de departamento por cada usuario.

## <a name="licensing-requirements"></a>Requisitos de concesión de licencia
Para usar licencias basadas en grupos, debe tener una de las siguientes licencias:

- Suscripción de pago o de prueba de Azure AD Premium P1 y versiones posteriores

- Edición de pago o de prueba de Office 365 Enterprise E3, Office 365 A3, Office 365 GCC G3, Office 365 E3 para GCCH u Office 365 E3 para DOD y superior

### <a name="required-number-of-licenses"></a>Número necesario de licencias
Para cualquier grupo al que se le asigne una licencia, también debe tener una licencia para cada miembro exclusivo. Si bien no tiene que asignar una licencia a cada miembro del grupo, debe tener al menos suficientes licencias para incluir a todos los miembros. Por ejemplo, si tiene 1000 miembros exclusivos que forman parte de grupos con licencia en su inquilino, debe tener al menos 1000 licencias para cumplir el contrato de licencia.

## <a name="features"></a>Características

A continuación se indican las características principales de las licencias basadas en grupos:

- Se pueden asignar licencias a todos los grupos de seguridad en Azure AD. Los grupos de seguridad se pueden sincronizar desde el entorno local mediante Azure AD Connect. También puede crear grupos de seguridad directamente en Azure AD (también denominados grupos solo de nube) o de forma automática, a través de la característica de grupo dinámico de Azure AD.

- Cuando se asigna una licencia de producto a un grupo, el administrador puede deshabilitar uno o varios planes de servicio del producto. Habitualmente, esta asignación se hace cuando la organización todavía no está preparada para comenzar a usar un servicio incluido en un producto. Por ejemplo, el administrador podría asignar Office 365 a un departamento y deshabilitar temporalmente el servicio Yammer.

- Se admiten todos los Servicios en la nube de Microsoft que requieren licencias a nivel de usuario. Esta compatibilidad incluye todos los productos de Office 365, Enterprise Mobility + Security y Dynamics 365.

- Las licencias basadas en grupos actualmente solo están disponibles mediante [Azure Portal](https://portal.azure.com). Si usa principalmente otros portales de administración para la administración de usuarios y grupos, como [Centro de administración de Microsoft 365](https://admin.microsoft.com), puede seguir haciéndolo. Sin embargo, debe usar Azure Portal para administrar las licencias en el nivel de grupo.

- Azure AD administra automáticamente las modificaciones de licencia resultantes de los cambios de pertenencia a grupos. Habitualmente, las modificaciones de licencia entran en vigor minutos después de un cambio en la pertenencia.

- Un usuario puede ser miembro de varios grupos con directivas de licencia especificadas. Un usuario también puede tener algunas licencias que se asignaron directamente, fuera de cualquier grupo. El estado de usuario resultante es una combinación de todas las licencias de producto y servicio asignadas. Si se le asigna a un usuario la misma licencia desde varios orígenes, la licencia solo se usará una vez.

- En algunos casos, las licencias no se pueden asignar a un usuario. Por ejemplo, es posible que no haya licencias disponibles suficientes en el inquilino o puede que se hayan asignado servicios en conflicto al mismo tiempo. Los administradores tienen acceso a información sobre usuarios para los que Azure AD no pudo procesar íntegramente las licencias de grupo. Pueden realizar acciones correctivas según esa información.

## <a name="your-feedback-is-welcome"></a>Agradecemos sus comentarios.

Si tiene comentarios o solicitudes de características, compártalos con nosotros a través del [foro de administradores de Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=162510).

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre otros escenarios de administración de licencias basadas en grupos, consulte:

* [Asignación de licencias a un grupo en Azure Active Directory](../users-groups-roles/licensing-groups-assign.md)
* [Identificación y resolución de problemas de licencias de un grupo en Azure Active Directory](../users-groups-roles/licensing-groups-resolve-problems.md)
* [Migración de usuarios individuales con licencia a licencias basadas en grupos en Azure Active Directory](../users-groups-roles/licensing-groups-migrate-users.md)
* [Cómo migrar usuarios entre diferentes licencias de productos con licencias basadas en grupos de Azure Active Directory](../users-groups-roles/licensing-groups-change-licenses.md)
* [Azure Active Directory group-based licensing additional scenarios](../users-groups-roles/licensing-group-advanced.md) (Escenarios adicionales de licencias basadas en grupos de Azure Active Directory)
* [Ejemplos de PowerShell para licencias basadas en grupos de Azure AD](../users-groups-roles/licensing-ps-examples.md)

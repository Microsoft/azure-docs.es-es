---
title: Características de roles de Azure AD en Privileged Identity Management | Microsoft Docs
description: Procedimiento para administrar roles de Azure AD para la asignación de Privileged Identity Management (PIM)
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.assetid: ''
ms.service: active-directory
ms.subservice: pim
ms.topic: conceptual
ms.workload: identity
ms.date: 07/10/2020
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: c039842a04923bc02aa288576570d51c39156c40
ms.sourcegitcommit: 9c3cfbe2bee467d0e6966c2bfdeddbe039cad029
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88784018"
---
# <a name="management-capabilities-for-azure-ad-roles-in-privileged-identity-management"></a>Funcionalidades de administración de los roles de Azure AD en Privileged Identity Management

La experiencia de administración de los roles de Azure AD en Privileged Identity Management se ha actualizado para unificar cómo se administran los roles de Azure AD y los roles de recursos de Azure. Anteriormente, Privileged Identity Management ofrecía un par de características clave para los roles de recursos de Azure que no estaban disponibles para los roles de Azure AD.

Con la actualización que se está implementando actualmente, combinamos las dos en una sola experiencia de administración y, en ella, se obtiene la misma funcionalidad para los roles de Azure AD que para los roles de recursos de Azure. En este artículo se detallan las características actualizadas y los requisitos.

## <a name="time-bound-assignments"></a>Asignaciones con límites temporales

Anteriormente, había dos estados posibles para las asignaciones de roles: *válido* y *permanente*. Ahora también puede establecer una hora de inicio y finalización para cada tipo de asignación. Esta adición le proporciona cuatro estados posibles en los que puede colocar una asignación:

- Válido de manera permanente
- Activo de manera permanente
- Válido, con las fechas de inicio y finalización especificadas para la asignación
- Activo, con las fechas de inicio y finalización especificadas para la asignación

En muchos casos, aunque no quiera que los usuarios tengan la asignación válida y activen roles cada vez, puede proteger la organización de Azure AD mediante el establecimiento de una hora de expiración para las asignaciones. Por ejemplo, si tiene algunos usuarios temporales que son válidos, considere la posibilidad de establecer una expiración para quitarlos automáticamente de la asignación de roles cuando se complete su trabajo.

## <a name="new-role-settings"></a>Nueva configuración de rol

También hemos agregado nuevas opciones de configuración para los roles de Azure AD.

- **Anteriormente**, solo podía establecer la configuración de activación según el rol. Es decir, la configuración de activación, como los requisitos de autenticación multifactor y los de incidencias de incidentes y solicitudes, se aplica a todos los usuarios que pueden usar un rol específico.
- **Ahora** puede configurar si un usuario individual debe realizar la autenticación multifactor antes de poder activar un rol. Además, puede disponer de un control avanzado sobre los correos electrónicos de Privileged Identity Management relacionados con roles específicos.

## <a name="extend-and-renew-assignments"></a>Extensión y renovación de asignaciones

Tan pronto como comprenda la asignación con límites temporales, lo primero que probablemente se preguntará es qué sucede si expira un rol. En esta nueva versión, se proporcionan dos opciones para este escenario:

- **Extensión**: cuando una asignación de roles está a punto de expirar, el usuario puede usar Privileged Identity Management para solicitar una extensión para esa asignación de roles.
- **Renovación**: cuando una asignación de roles ha expirado, el usuario puede usar Privileged Identity Management para solicitar una renovación para esa asignación de roles.

Ambas acciones iniciadas por el usuario requieren una aprobación de un administrador global o un administrador de roles con privilegios. Los administradores ya no tendrán que encargarse de administrar estas expiraciones. Únicamente recibirán las solicitudes de extensión o renovación y deberán aprobarlas si son válidas.

## <a name="api-changes"></a>Cambios de API

Cuando los clientes tengan la versión actualizada implementada en su organización de Azure AD, los servicios de Graph API existentes dejarán de funcionar. Debe realizar una transición para usar [Graph API para los roles de recursos de Azure](/graph/api/resources/privilegedidentitymanagement-resources?view=graph-rest-beta). Para administrar roles de Azure AD con esa API, cambie `/azureResources` por `/aadroles` en la firma y use el identificador del directorio para el valor de `resourceId`.

Hemos hecho todo lo posible para ponernos en contacto con todos los clientes que usan la API anterior a fin de informales sobre este cambio de manera anticipada. Si su organización de Azure AD se migró a la nueva versión y aún depende de la API anterior, póngase en contacto con el equipo en pim_preview@microsoft.com.

## <a name="powershell-change"></a>Cambio de PowerShell

En el caso de los clientes que usan el módulo de PowerShell de Privileged Identity Management para los roles de Azure AD, PowerShell dejará de funcionar con la actualización. En lugar de los cmdlets anteriores, debe usar los cmdlets de Privileged Identity Management dentro del módulo de PowerShell de la versión preliminar de Azure AD. Instale el módulo de Azure AD PowerShell desde la [Galería de PowerShell](https://www.powershellgallery.com/packages/AzureADPreview/2.0.0.17). Ahora puede [leer la documentación y los ejemplos de las operaciones de PIM en este módulo de PowerShell](powershell-for-azure-ad-roles.md).

## <a name="next-steps"></a>Pasos siguientes

- [Asignación de un rol personalizado de Azure AD](azure-ad-custom-roles-assign.md)
- [Eliminación o actualización de una asignación de roles personalizados de Azure AD](azure-ad-custom-roles-update-remove.md)
- [Configuración de una asignación de roles personalizados de Azure AD](azure-ad-custom-roles-configure.md)
- [Definiciones de roles en Azure AD](../users-groups-roles/directory-assign-admin-roles.md)
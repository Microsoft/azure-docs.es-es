---
title: Escenarios comunes de administración de derechos - Azure AD
description: Conozca los pasos de alto nivel que debe seguir en escenarios comunes de administración de derechos de Azure Active Directory.
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 06/18/2020
ms.author: ajburnle
ms.reviewer: mwahl
ms.collection: M365-identity-device-management
ms.openlocfilehash: 59b66511720966ff978c8460b9ac69ad133b7501
ms.sourcegitcommit: 7374b41bb1469f2e3ef119ffaf735f03f5fad484
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/16/2020
ms.locfileid: "90707227"
---
# <a name="common-scenarios-in-azure-ad-entitlement-management"></a>Escenarios comunes de administración de derechos de Azure AD

Hay varias formas para configurar la administración de derechos para su organización. Sin embargo, si acaba de empezar, es útil comprender los escenarios comunes para los administradores, los propietarios de catálogos, los administradores de paquetes de acceso, los aprobadores y los solicitantes.

## <a name="delegate"></a>Delegar

### <a name="administrator-delegate-management-of-resources"></a>Administrador: Delegar la administración de los recursos

1. [Ver el vídeo: Delegación de TI al director de departamento](https://www.microsoft.com/videoplayer/embed/RE3Lq00)
1. [Delegación de usuarios al rol de creador de catálogos](entitlement-management-delegate-catalog.md)

### <a name="catalog-creator-delegate-management-of-resources"></a>Creador de catálogos: Delegar la administración de los recursos

- [Creación de un catálogo](entitlement-management-catalog-create.md#create-a-catalog)

### <a name="catalog-owner-delegate-management-of-resources"></a>Propietario del catálogo: Delegar la administración de los recursos

1. [Adición de copropietarios al catálogo](entitlement-management-catalog-create.md#add-additional-catalog-owners)
1. [Adición de recursos al catálogo](entitlement-management-catalog-create.md#add-resources-to-a-catalog)

### <a name="catalog-owner-delegate-management-of-access-packages"></a>Propietario del catálogo: delegar la administración de paquetes de acceso

1. [Ver el vídeo: Delegación del propietario del catálogo al administrador de paquetes de acceso](https://www.microsoft.com/videoplayer/embed/RE3Lq08)
1. [Delegación de usuarios al rol del administrador de paquetes de acceso](entitlement-management-delegate-managers.md)

## <a name="govern-access-for-users-in-your-organization"></a>Regulación del acceso de los usuarios de la organización

### <a name="access-package-manager-allow-employees-in-your-organization-to-request-access-to-resources"></a>Administrador de paquetes de acceso: permitir que los empleados de la organización soliciten acceso a los recursos

1. [Creación de un paquete de acceso](entitlement-management-access-package-create.md#start-new-access-package)
1. [Adición de grupos, equipos, aplicaciones o sitios de SharePoint al paquete de acceso](entitlement-management-access-package-create.md#resource-roles)
1. [Adición de una directiva de solicitud para permitir que los usuarios de su directorio soliciten acceso](entitlement-management-access-package-create.md#for-users-in-your-directory)
1. [Especificación de la configuración de expiración](entitlement-management-access-package-create.md#lifecycle)

### <a name="requestor-request-access-to-resources"></a>Solicitante: solicitar acceso a los recursos

1. [Iniciar sesión en el portal Mi acceso](entitlement-management-request-access.md#sign-in-to-the-my-access-portal)
1. Buscar el paquete de acceso
1. [Solicitar acceso](entitlement-management-request-access.md#request-an-access-package)

### <a name="approver-approve-requests-to-resources"></a>Aprobador: aprobar las solicitudes de acceso a los recursos

1. [Abrir la solicitud en el portal Mi acceso](entitlement-management-request-approve.md#open-request)
1. [Aprobar o denegar la solicitud de acceso](entitlement-management-request-approve.md#approve-or-deny-request)

### <a name="requestor-view-the-resources-you-already-have-access-to"></a>Solicitante: ver los recursos a los que ya tiene acceso

1. [Iniciar sesión en el portal Mi acceso](entitlement-management-request-access.md#sign-in-to-the-my-access-portal)
1. Ver los paquetes de acceso activos

## <a name="govern-access-for-users-outside-your-organization"></a>Regular el acceso de los usuarios de fuera de la organización

### <a name="administrator-collaborate-with-an-external-partner-organization"></a>Administrador: colaborar con una organización asociada externa

1. [Leer cómo funciona el acceso para los usuarios externos](entitlement-management-external-users.md#how-access-works-for-external-users)
1. [Revisar la configuración de los usuarios externos](entitlement-management-external-users.md#settings-for-external-users)
1. [Agregar una conexión a la organización externa](entitlement-management-organization.md)

### <a name="access-package-manager-collaborate-with-an-external-partner-organization"></a>Administrador de paquetes de acceso: colaborar con una organización asociada externa

1. [Creación de un paquete de acceso](entitlement-management-access-package-create.md#start-new-access-package)
1. [Adición de grupos, equipos, aplicaciones o sitios de SharePoint al paquete de acceso](entitlement-management-access-package-resources.md#add-resource-roles)
1. [Adición de una directiva de solicitud para permitir que los usuarios de su directorio soliciten acceso](entitlement-management-access-package-request-policy.md#for-users-not-in-your-directory)
1. [Especificación de la configuración de expiración](entitlement-management-access-package-create.md#lifecycle)
1. [Copia del vínculo para solicitar el paquete de acceso](entitlement-management-access-package-settings.md)
1. Envío del vínculo al asociado de contacto del asociado externo para compartirlo con sus usuarios

### <a name="requestor-request-access-to-resources-as-an-external-user"></a>Solicitante: solicitar acceso a los recursos como usuario externo

1. Buscar el vínculo del paquete de acceso que recibió de su contacto
1. [Iniciar sesión en el portal Mi acceso](entitlement-management-request-access.md#sign-in-to-the-my-access-portal)
1. [Solicitar acceso](entitlement-management-request-access.md#request-an-access-package)

### <a name="approver-approve-requests-to-resources"></a>Aprobador: aprobar las solicitudes de acceso a los recursos

1. [Abrir la solicitud en el portal Mi acceso](entitlement-management-request-approve.md#open-request)
1. [Aprobar o denegar la solicitud de acceso](entitlement-management-request-approve.md#approve-or-deny-request)

### <a name="requestor-view-the-resources-your-already-have-access-to"></a>Solicitante: ver los recursos a los que ya tiene acceso

1. [Iniciar sesión en el portal Mi acceso](entitlement-management-request-access.md#sign-in-to-the-my-access-portal)
1. Ver los paquetes de acceso activos

## <a name="day-to-day-management"></a>Administración diaria

### <a name="access-package-manager-update-the-resources-for-a-project"></a>Administrador de paquetes de acceso: actualizar los recursos de un proyecto

1. [Ver el vídeo: Administración diaria: las cosas han cambiado](https://www.microsoft.com/videoplayer/embed/RE3LD4Z)
1. Abrir el paquete de acceso
1. [Agregar o quitar grupos, equipos, aplicaciones o sitios de SharePoint](entitlement-management-access-package-resources.md#add-resource-roles)

### <a name="access-package-manager-update-the-duration-for-a-project"></a>Administrador de paquetes de acceso: actualizar la duración de un proyecto

1. [Ver el vídeo: Administración diaria: las cosas han cambiado](https://www.microsoft.com/videoplayer/embed/RE3LD4Z)
1. Abrir el paquete de acceso
1. [Abrir la configuración del ciclo de vida](entitlement-management-access-package-lifecycle-policy.md#open-lifecycle-settings)
1. [Actualizar la configuración de expiración](entitlement-management-access-package-lifecycle-policy.md#lifecycle)

### <a name="access-package-manager-update-how-access-is-approved-for-a-project"></a>Administrador de paquetes de acceso: actualizar cómo se aprueba el acceso para un proyecto

1. [Ver el vídeo: Administración diaria: las cosas han cambiado](https://www.microsoft.com/videoplayer/embed/RE3LD4Z)
1. [Abrir una directiva existente de configuración de solicitudes y aprobación](entitlement-management-access-package-request-policy.md#open-an-existing-policy-of-request-and-approval-settings)
1. [Actualizar la configuración de aprobación](entitlement-management-access-package-request-policy.md#approval)

### <a name="access-package-manager-update-the-people-for-a-project"></a>Administrador de paquetes de acceso: actualizar las personas de un proyecto

1. [Ver el vídeo: Administración diaria: las cosas han cambiado](https://www.microsoft.com/videoplayer/embed/RE3LD4Z)
1. [Quitar usuarios que ya no necesitan acceso](entitlement-management-access-package-assignments.md)
1. [Abrir una directiva existente de configuración de solicitudes y aprobación](entitlement-management-access-package-request-policy.md#open-an-existing-policy-of-request-and-approval-settings)
1. [Agregar usuarios que necesitan acceso](entitlement-management-access-package-request-policy.md#for-users-in-your-directory)

### <a name="access-package-manager-directly-assign-specific-users-to-an-access-package"></a>Administrador de paquetes de acceso: asignar directamente usuarios específicos a un paquete de acceso

1. [Si los usuarios necesitan una configuración de ciclo de vida diferente, agregue una nueva directiva al paquete de acceso](entitlement-management-access-package-request-policy.md#add-a-new-policy-of-request-and-approval-settings)
1. [Asignar directamente usuarios específicos al paquete de acceso](entitlement-management-access-package-assignments.md#directly-assign-a-user)

## <a name="assignments-and-reports"></a>Asignaciones e informes

### <a name="administrator-view-who-has-assignments-to-an-access-package"></a>Administrador: ver quién tiene asignaciones a un paquete de acceso

1. Abrir un paquete de acceso
1. [Ver asignaciones](entitlement-management-access-package-assignments.md#view-who-has-an-assignment)
1. [Archivar informes y registros](entitlement-management-logs-and-reporting.md)

### <a name="administrator-view-resources-assigned-to-users"></a>Administrador: ver los recursos asignados a los usuarios

1. [Ver los paquetes de acceso de un usuario](entitlement-management-reports.md#view-access-packages-for-a-user)
1. [Ver las asignaciones de recursos de un usuario](entitlement-management-reports.md#view-resource-assignments-for-a-user)

## <a name="programmatic-administration"></a>Administración mediante programación

También puede administrar paquetes de acceso, catálogos, directivas, solicitudes y asignaciones mediante Microsoft Graph.  Un usuario de un rol adecuado con una aplicación con el permiso `EntitlementManagement.ReadWrite.All` delegado puede llamar a la [API de administración de derechos](https://docs.microsoft.com/graph/tutorial-access-package-api?view=graph-rest-beta ).

## <a name="next-steps"></a>Pasos siguientes

- [Delegación y roles](entitlement-management-delegate.md)
- [Solicitud de notificaciones de proceso y correo electrónico](entitlement-management-process.md)
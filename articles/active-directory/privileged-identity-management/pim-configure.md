---
title: Qué es Privileged Identity Management -Azure AD | Microsoft Docs
description: Ofrece una introducción a Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.subservice: pim
ms.topic: overview
ms.date: 08/08/2020
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: a4a081bc5a8040fa3ccd0599341eb2671bbe2d4d
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89047826"
---
# <a name="what-is-azure-ad-privileged-identity-management"></a>¿Qué es Azure AD Privileged Identity Management?

 Privileged Identity Management (PIM) es un servicio de Azure Active Directory (Azure AD) que permite administrar, controlar y supervisar el acceso a recursos importantes de la organización. Estos recursos incluyen los recursos de Azure AD, los recursos de Azure y los de otros servicios de Microsoft Online Services, como Office 365 o Microsoft Intune.

## <a name="reasons-to-use"></a>Motivos para su uso

Las organizaciones buscan reducir el número de personas que tienen acceso a información segura o a recursos, ya que de esta manera se reduce la posibilidad de que personas malintencionadas obtengan acceso a ellos o de que algún usuario autorizado haga algo involuntariamente que pueda afectar a recursos sensibles. Sin embargo, los usuarios siguen teniendo la necesidad de llevar a cabo operaciones con privilegios en aplicaciones de Azure AD, Azure, Office 365 o SaaS. Las organizaciones pueden conceder a los usuarios privilegios de acceso Just-In-Time a los recursos de Azure y a Azure AD. Es necesario supervisar lo que los usuarios hacen con los privilegios de administrador.

## <a name="what-does-it-do"></a>¿Para qué sirve?

Privileged Identity Management proporciona activación de rol basada en tiempo y en aprobación para mitigar los riesgos de tener unos permisos de acceso excesivos, innecesarios o mal utilizados en los recursos que le interesan. Estas son algunas de las características clave de Privileged Identity Management:

- Conceda privilegios de acceso **Just-In-Time** a Azure AD y a los recursos de Azure
- Asigne acceso **limitado** a los recursos con fechas de inicio y fin
- Requiera **aprobación** para activar los roles con privilegios
- Aplique la **autenticación multifactor** para activar cualquier rol
- Use la **justificación** para comprender por qué se activan los usuarios
- Obtenga **notificaciones** cuando se activan los roles con privilegios
- Lleve a cabo **revisiones de acceso** para asegurarse de que los usuarios siguen necesitando roles
- Descargue el **historial de auditoría** para la auditoría interna o externa

## <a name="what-can-i-do-with-it"></a>¿Qué se puede hacer con Privileged Identity Management (PIM)?

Una vez que se haya configurado Privileged Identity Management, verá las opciones **Tareas**, **Administrar**, y **Actividad** en el menú de navegación izquierdo. Como administrador, tendrá que elegir entre distintas opciones, como administrar **roles de Azure AD**, administrar roles de **recursos de Azure** o grupos de acceso con privilegios. Cuando elija lo que desea administrar, verá el conjunto de opciones adecuado para esa opción.

![Captura de pantalla de Privileged Identity Management en Azure Portal](./media/pim-configure/pim-quickstart.png)

## <a name="who-can-do-what"></a>¿Quién puede realizar cada tarea en PIM?

En el caso de los roles de Azure AD en Privileged Identity Management, los usuarios que tengan el rol Administrador de roles con privilegios o el rol Administrador global son los únicos que pueden administrar las asignaciones de otros administradores. Puede [conceder acceso a otros administradores para administrar Privileged Identity Management](pim-how-to-give-access-to-pim.md). Los administradores globales, administradores de seguridad, lectores globales y lectores de seguridad pueden ver también las asignaciones de roles de Azure AD en Privileged Identity Management.

En el caso de los roles de los recursos de Azure en Privileged Identity Management, los únicos que pueden administrar asignaciones para los demás administradores son los administradores de suscripciones, los propietarios de los recursos o los administradores de acceso de usuario de los recursos. De forma predeterminada, los usuarios que son administradores de roles con privilegios, administradores de seguridad o lectores de seguridad no tienen acceso para ver asignaciones a roles de recursos de Azure en Privileged Identity Management.

## <a name="scenarios"></a>Escenarios

Privileged Identity Management admite los siguientes escenarios:

### <a name="privileged-role-administrator-permissions"></a>Permisos de administrador de roles con privilegios

- Habilitar la aprobación de roles específicos
- Especificar usuarios y grupos de aprobadores para la aprobación de solicitudes
- Ver el historial de solicitudes y aprobaciones de todos los roles con privilegios

### <a name="approver-permissions"></a>Permisos de aprobador

- Ver aprobaciones o solicitudes pendientes
- Aprobar o rechazar solicitudes de elevación de rol (de forma individual o masiva)
- Proporcionar una justificación de la aprobación o el rechazo

### <a name="eligible-role-user-permissions"></a>Permisos de usuario de rol válidos

- Solicitar la activación de un rol que requiere aprobación
- Ver el estado de la solicitud de activación
- Completar la tarea en Azure AD si la activación se ha aprobado

## <a name="terminology"></a>Terminología

Para conocer mejor Privileged Identity Management y su documentación, debe revisar los términos siguientes.

| Término o concepto | Categoría de asignación de roles | Descripción |
| --- | --- | --- |
| Apto | Tipo | Asignación de roles que requiere que un usuario realice una o varias acciones para usar el rol. Si un usuario es apto para un rol, eso significa que puede activarlo cuando necesite para realizar tareas con privilegios. No hay ninguna diferencia en el acceso proporcionado de forma permanente a una persona o una asignación de roles aptos. La única diferencia es que algunas personas no necesitan ese acceso todo el tiempo. |
| active | Tipo | Asignación de roles que no requiere que el usuario realice ninguna acción para usar el rol. Los usuarios asignados como activos tienen privilegios asignados al rol. |
| activar |  | Proceso de realizar una o varias acciones para usar un rol para el que es apto un usuario. Entre las acciones se puede incluir realizar una comprobación de autenticación multifactor (MFA), proporcionar una justificación de negocios o solicitar la aprobación de los aprobadores designados. |
| asignado | State | Usuario que tiene una asignación de roles activa. |
| activado | State | Usuario con una asignación de roles apta, que ha realizado las acciones necesarias para activar el rol y ahora está activo.  Una vez activado, el usuario puede utilizar el rol durante un período preconfigurado de tiempo antes de tener que volver a activarlo. |
| apto permanente | Duration | Asignación de roles en la que un usuario siempre es apto para activar el rol. |
| activo permanente | Duration | Asignación de roles en la que un usuario siempre puede usar el rol, sin realizar ninguna acción. |
| apto con expiración | Duration | Asignación de roles en la que un usuario es apto para activar el rol en un plazo específico de tiempo, con principio y fin. |
| activo con expiración | Duration | Asignación de roles en la que un usuario puede usar el rol sin realizar ninguna acción en un plazo determinado con principio y fin. |
| Acceso Just-In-Time (JIT) |  | Modelo en el que los usuarios reciben permisos temporales para realizar tareas con privilegios, lo que impide que usuarios malintencionados o sin autorización obtengan acceso después de que el permiso haya expirado. El acceso se concede solo cuando los usuarios lo necesitan. |
| Principio de acceso con privilegios mínimos |  | Una práctica de seguridad recomendada en la que se proporcionan a todos los usuarios únicamente los privilegios mínimos necesarios para realizar las tareas que están autorizados a realizar. Esta práctica minimiza el número de administradores globales, ya que en su lugar utiliza roles de administrador específicos para determinados escenarios. |

## <a name="license-requirements"></a>Requisitos de licencia

[!INCLUDE [Azure AD Premium P2 license](../../../includes/active-directory-p2-license.md)]

Para más información sobre las licencias para usuarios, consulte [License requirements to use Privileged Identity Management](subscription-requirements.md) (Requisitos de licencia para usar Privileged Identity Management).

## <a name="next-steps"></a>Pasos siguientes

- [Requisitos de licencia para usar Privileged Identity Management](subscription-requirements.md)
- [Protección del acceso con privilegios para las implementaciones híbridas y en la nube en Azure AD](../users-groups-roles/directory-admin-roles-secure.md?toc=%2fazure%2factive-directory%2fprivileged-identity-management%2ftoc.json)
- [Implementación de Privileged Identity Management](pim-deployment-plan.md)

---
title: 'Consulta, incorporación y eliminación de asignaciones para un paquete de acceso en la administración de derechos de Azure AD: Azure Active Directory'
description: Aprenda a consultar, agregar y eliminar asignaciones de un paquete de acceso en la administración de derechos de Azure Active Directory.
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 06/18/2020
ms.author: ajburnle
ms.reviewer: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: 355f0127829b709d82127fb340381830793c93ff
ms.sourcegitcommit: 9c3cfbe2bee467d0e6966c2bfdeddbe039cad029
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88783559"
---
# <a name="view-add-and-remove-assignments-for-an-access-package-in-azure-ad-entitlement-management"></a>Consulta, incorporación y eliminación de asignaciones para un paquete de acceso en la administración de derechos de Azure AD

En la administración de derechos de Azure AD, puede ver a quién se le han asignado los paquetes de acceso, su directiva y su estado. Si un paquete de acceso tiene una directiva adecuada, también puede asignar directamente al usuario un paquete de acceso. En este artículo se explica cómo consultar, agregar y eliminar asignaciones en paquetes de acceso.

## <a name="prerequisites"></a>Prerrequisitos

Para usar la administración de derechos de Azure AD y asignar usuario a los paquetes de acceso, debe tener una de las licencias siguientes:

- Azure AD Premium P2
- Licencia de Enterprise Mobility + Security (EMS) E5

## <a name="view-who-has-an-assignment"></a>Ver quién tiene una asignación

**Rol necesario:** Administrador global, administrador de usuarios, propietario del catálogo o administrador de paquetes de acceso.

1. En Azure Portal, haga clic en **Azure Active Directory** y, luego, en **Gobernanza de identidades**.

1. En el menú de la izquierda, haga clic en **Paquetes de acceso** y luego abra el paquete de acceso.

1. Haga clic en **Asignaciones** para ver una lista de las asignaciones activas.

    ![Lista de asignaciones a un paquete de acceso](./media/entitlement-management-access-package-assignments/assignments-list.png)

1. Haga clic en una asignación específica para ver detalles adicionales.

1. Para ver una lista de asignaciones que no tenían aprovisionados correctamente todos los roles de recursos, haga clic en el filtro de estado y seleccione **Entrega**.

    Puede ver detalles adicionales sobre los errores de entrega localizando la solicitud del usuario correspondiente en la página **Solicitudes**.

1. Para ver las asignaciones expiradas, haga clic en el filtro de estado y seleccione **Expiradas**.

1. Para descargar un archivo CSV de la lista filtrada, haga clic en **Descargar**.

### <a name="viewing-assignments-programmatically"></a>Visualización de asignaciones mediante programación

También puede recuperar las asignaciones de un paquete de acceso mediante Microsoft Graph.  Un usuario de un rol adecuado con una aplicación con el permiso `EntitlementManagement.ReadWrite.All` delegado puede llamar a la API para [enumerar elementos accessPackageAssignments](/graph/api/accesspackageassignment-list?view=graph-rest-beta).

## <a name="directly-assign-a-user"></a>Asignar directamente un usuario

En algunos casos, es posible que quiera asignar directamente usuarios específicos a un paquete de acceso para que los usuarios no tengan que pasar por el proceso de solicitar el paquete de acceso. Para asignar directamente usuarios, el paquete de acceso debe tener una directiva que permita las asignaciones directas de administrador.

**Rol necesario:** Administrador global, administrador de usuarios, propietario del catálogo o administrador de paquetes de acceso.

1. En Azure Portal, haga clic en **Azure Active Directory** y, luego, en **Gobernanza de identidades**.

1. En el menú de la izquierda, haga clic en **Paquetes de acceso** y luego abra el paquete de acceso.

1. En el menú de la izquierda, haga clic en **Asignaciones**.

1. Haga clic en **Nueva asignación** para abrir Add user to access package (Agregar usuario al paquete de acceso).

    ![Asignaciones - Add user to access package (Agregar usuario al paquete de acceso)](./media/entitlement-management-access-package-assignments/assignments-add-user.png)

1. Haga clic en **Agregar usuarios** para seleccionar los usuarios a los que quiere asignar este paquete de acceso.

1. En la lista **Seleccionar directiva**, seleccione una directiva por la que se regirán y se realizará un seguimiento de las solicitudes y el ciclo de vida futuros de los usuarios. Si desea que los usuarios seleccionados tengan una configuración de la directiva diferente, puede hacer clic en **Crear nueva directiva** para agregar una nueva directiva.

1. Establezca la fecha y hora en la que quiere que comience y finalice la asignación de los usuarios seleccionados. Si no se proporciona una fecha de finalización, se utilizará la configuración del ciclo de vida de la directiva.

1. Opcionalmente, proporcione una justificación para la asignación directa para mantener un registro.

1. Haga clic en **Agregar** para asignar directamente los usuarios seleccionados al paquete acceso.

    Transcurridos unos instantes, haga clic en **Actualizar** para ver los usuarios en la lista Asignaciones.

### <a name="directly-assigning-users-programmatically"></a>Asignación de usuarios directamente mediante programación

También puede asignar un usuario directamente a un paquete de acceso mediante Microsoft Graph.  Un usuario de un rol adecuado con una aplicación con el permiso `EntitlementManagement.ReadWrite.All` delegado puede llamar a la API para [crear un elemento accessPackageAssignmentRequest](/graph/api/accesspackageassignmentrequest-post?view=graph-rest-beta).

## <a name="remove-an-assignment"></a>Eliminación de una asignación

**Rol necesario:** Administrador global, administrador de usuarios, propietario del catálogo o administrador de paquetes de acceso.

1. En Azure Portal, haga clic en **Azure Active Directory** y, luego, en **Gobernanza de identidades**.

1. En el menú de la izquierda, haga clic en **Paquetes de acceso** y luego abra el paquete de acceso.

1. En el menú de la izquierda, haga clic en **Asignaciones**.
 
1. Haga clic en la casilla que hay al lado del usuario cuya asignación desea quitar del paquete de acceso. 

1. Haga clic en el botón **Quitar** situado cerca de la parte superior del panel izquierdo. 
 
    ![Asignaciones: quitar usuario del paquete de acceso](./media/entitlement-management-access-package-assignments/remove-assignment-select-remove-assignment.png)

    Aparecerá una notificación que le informará de que se ha quitado la asignación. 

## <a name="next-steps"></a>Pasos siguientes

- [Modificar la configuración de solicitud de un paquete de acceso](entitlement-management-access-package-request-policy.md)
- [Visualización de informes y registros](entitlement-management-reports.md)
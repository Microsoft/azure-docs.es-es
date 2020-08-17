---
title: Uso compartido de paneles de Azure Portal mediante el control de acceso basado en rol
description: En este artículo se explica cómo compartir un panel en Azure Portal mediante el control de acceso basado en rol.
services: azure-portal
documentationcenter: ''
author: mgblythe
manager: mtillman
ms.assetid: 8908a6ce-ae0c-4f60-a0c9-b3acfe823365
ms.service: azure-portal
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: na
ms.date: 03/23/2020
ms.author: mblythe
ms.openlocfilehash: 9ead7eb19e49574073f038648ca1d247b2dab98f
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87131708"
---
# <a name="share-azure-dashboards-by-using-role-based-access-control"></a>Uso compartido de paneles de Azure mediante el control de acceso basado en rol

Después de configurar un panel, puede publicarlo y compartirlo con otros usuarios de su organización. Mediante el [control de acceso basado en rol](../role-based-access-control/role-assignments-portal.md) (RBAC) de Azure puede permitir que otros usuarios vean su panel. Asigne un rol a un usuario o a un grupo de usuarios. Ese rol define si esos usuarios pueden ver o modificar el panel publicado.

Todos los paneles publicados se implementan como recursos de Azure. Existen como elementos administrables dentro de su suscripción y están contenidos en un grupo de recursos. Desde una perspectiva de control de acceso, los paneles no son distintos de otros recursos como una máquina virtual o una cuenta de almacenamiento.

> [!TIP]
> Los iconos individuales del panel tendrán sus propios requisitos de control de acceso en función de los recursos que muestran. Puede compartir un panel de manera generalizada y al mismo tiempo proteger los datos de iconos individuales.
> 
> 

## <a name="understanding-access-control-for-dashboards"></a>Descripción del control de acceso para los paneles

Mediante el control de acceso basado en rol (RBAC), puede asignar a usuarios a los roles en tres niveles distintos de ámbito:

* subscription
* resource group
* resource

Los permisos que asigne se heredan desde la suscripción hasta el recurso. El panel publicado es un recurso. Es posible que ya tenga usuarios asignados a roles de la suscripción que se apliquen al panel publicado.

Supongamos que tiene una suscripción de Azure y que a varios miembros del equipo se les han asignado los roles de *propietario*, *colaborador* o *lector* en la suscripción. Los usuarios que son propietarios o colaboradores pueden enumerar, ver, crear, modificar o eliminar paneles dentro de la suscripción. Los usuarios que son lectores pueden enumerar y ver los paneles, pero no pueden modificarlos ni eliminarlos. Los usuarios con acceso de lector pueden realizar modificaciones locales en un panel publicado (por ejemplo, para solucionar un problema), pero no pueden publicar esos cambios en el servidor. Pueden realizar una copia privada del panel para sí mismos.

También podría asignar permisos al grupo de recursos que contenga varios paneles o a un solo panel. Por ejemplo, puede decidir que un grupo de usuarios tenga permisos limitados a través de la suscripción pero un mayor acceso a un panel determinado. Asigne a esos usuarios un rol de ese panel.

## <a name="publish-dashboard"></a>Publicación del panel

Supongamos que configura un panel que quiere compartir con un grupo de usuarios de la suscripción. En los pasos siguientes se muestra cómo compartir un panel con un grupo llamado Administradores de almacenamiento. Puede asignar al grupo el nombre que desee. Para más información, consulte [Administración de grupos en Azure Active Directory](../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

Antes de asignar el acceso, debe publicar el panel.

1. En el panel, seleccione **Compartir**.

    ![seleccionar compartir para el panel](./media/azure-portal-dashboard-share-access/share-dashboard-for-access-control.png)

1. En **Uso compartido y control de acceso**, seleccione **Publicar**.

    ![publicar el panel](./media/azure-portal-dashboard-share-access/publish-dashboard-for-access-control.png)

     De forma predeterminada, se publica el panel en un grupo de recursos denominado **paneles**. Para seleccionar otro grupo de recursos, borre la casilla.

Tras esto, el panel ya se ha publicado. Si son adecuados los permisos heredados de la suscripción, ya no es necesario hacer nada más. Otros usuarios de la organización pueden acceder al panel y modificarlo según su rol de nivel de suscripción.

## <a name="assign-access-to-a-dashboard"></a>Asignación de acceso a un panel

Puede asignar un rol a un grupo de usuarios de ese panel.

1. Una vez publicado el panel, seleccione la opción **Compartir** o **Dejar de compartir** para acceder a **Uso compartido y control de acceso**.

1. En **Uso compartido y control de acceso**, seleccione **Administrar usuarios**.

    ![administrar usuarios de un panel](./media/azure-portal-dashboard-share-access/manage-users-for-access-control.png)

1. Seleccione **Asignaciones de roles** para ver los usuarios existentes que ya tienen asignado un rol en este panel.

1. Para agregar un nuevo usuario o grupo, seleccione **Agregar** y, a continuación, **Agregar asignación de roles**.

    ![agregar un usuario para el acceso al panel](./media/azure-portal-dashboard-share-access/manage-users-existing-users.png)

1. Seleccione el rol que representa los permisos que se van a conceder. En este ejemplo, seleccione **Colaborador**.

1. Seleccione el usuario o grupo al que quiere asignar el rol. Si no ve el usuario o grupo que busca en la lista, use el cuadro de búsqueda. La lista de grupos disponibles depende de los grupos que haya creado en Active Directory.

1. Cuando haya terminado de agregar usuarios o grupos, seleccione **Guardar**.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener una lista de roles, vea [Roles integrados de Azure](../role-based-access-control/built-in-roles.md).
* Para más información sobre cómo administrar recursos, consulte [Administración de recursos de Azure con Azure Portal](resource-group-portal.md).

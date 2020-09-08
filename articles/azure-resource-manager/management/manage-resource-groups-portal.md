---
title: 'Administración de grupos de recursos: Azure Portal'
description: Use Azure Portal para administrar grupos de recursos con Azure Resource Manager. Muestra cómo crear, enumerar y eliminar grupos de recursos.
author: mumian
ms.topic: conceptual
ms.date: 03/26/2019
ms.author: jgao
ms.openlocfilehash: feee3a0d6de97ac6aaa1f9a92f20860f3f461ae3
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87827238"
---
# <a name="manage-azure-resource-manager-resource-groups-by-using-the-azure-portal"></a>Administración de grupos de recursos de Azure Resource Manager mediante Azure Portal

Aprenda a utilizar [Azure Portal](https://portal.azure.com) con [Azure Resource Manager](overview.md) para administrar grupos de recursos de Azure. Para administrar recursos de Azure, vea [Administración de recursos de Azure mediante Azure Portal](manage-resources-portal.md).

Otros artículos sobre cómo administrar grupos de recursos:

- [Administrar grupos de recursos de Azure Resource Manager mediante la CLI de Azure](manage-resources-cli.md)
- [Administración de grupos de recursos de Azure mediante Azure PowerShell](manage-resources-powershell.md)

[!INCLUDE [Handle personal data](../../../includes/gdpr-intro-sentence.md)]

## <a name="what-is-a-resource-group"></a>¿Qué es un grupo de recursos?

Un grupo de recursos es un contenedor que almacena los recursos relacionados con una solución de Azure. El grupo de recursos puede incluir todos los recursos de la solución o solo aquellos que se desean administrar como grupo. Para decidir cómo asignar los recursos a los grupos de recursos, tenga en cuenta lo que más conviene a su organización. Por lo general, se recomienda agregar recursos que compartan el mismo ciclo de vida al mismo grupo de recursos para que los pueda implementar, actualizar y eliminar con facilidad como un grupo.

Los grupos de recursos almacenan metadatos acerca de los recursos. Por consiguiente, al especificar la ubicación del grupo de recursos, se especifica el lugar en que se almacenan dichos metadatos. Por motivos de compatibilidad, es posible que sea preciso asegurarse de que los datos se almacenan en una región concreta.


## <a name="create-resource-groups"></a>Crear grupos de recursos

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **Grupos de recursos**.

    ![agregar grupo de recursos](./media/manage-resource-groups-portal/manage-resource-groups-add-group.png)
3. Seleccione **Agregar**.
4. Escriba los siguientes valores:

   - **Suscripción**: Seleccione su suscripción a Azure. 
   - **Grupo de recursos**: Escriba un nuevo nombre para el grupo de recursos. 
   - **Región**: Seleccione una ubicación de Azure, como **Centro de EE. UU.** .

     ![Creación de un grupo de recursos](./media/manage-resource-groups-portal/manage-resource-groups-create-group.png)
5. Seleccione **Revisar + crear**.
6. Seleccione **Crear**. El grupo de recursos tarda unos segundos en crearse.
7. Seleccione **Actualizar** en el menú superior para actualizar la lista de grupos de recursos y, después, seleccione el grupo de recursos recién creado para abrirlo. También puede seleccionar **Notificación**(el icono de campana) en la parte superior y, después, seleccionar **Ir al grupo de recursos** para abrir el grupo de recursos recién creado

    ![Ir al grupo de recursos](./media/manage-resource-groups-portal/manage-resource-groups-add-group-go-to-resource-group.png)

## <a name="list-resource-groups"></a>Enumeración de grupos de recursos

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Para ver una lista de los grupos de recursos, seleccione **Grupos de recursos**.

    ![buscar grupos de recursos](./media/manage-resource-groups-portal/manage-resource-groups-list-groups.png)

3. Para personalizar la información que se muestra sobre los grupos de recursos, seleccione **Editar columnas**. La siguiente captura de pantalla muestra las columnas de adición que se pueden agregar a la visualización:

## <a name="open-resource-groups"></a>Abrir grupos de recursos

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **Grupos de recursos**.
3. Seleccione el grupo de recursos que quiera abrir.

## <a name="delete-resource-groups"></a>Eliminación de grupos de recursos

1. Abra el grupo de recursos que quiera eliminar.  Vea [Abrir grupos de recursos](#open-resource-groups).
2. Seleccione **Eliminar grupo de recursos**.

    ![Eliminación de un grupo de recursos de Azure](./media/manage-resource-groups-portal/delete-group.png)

Para obtener más información sobre cómo ordena Azure Resource Manager la eliminación de recursos, consulte [Eliminación del grupo de recursos en Azure Resource Manager](delete-resource-group.md).

## <a name="deploy-resources-to-a-resource-group"></a>Implementación de recursos en un grupo de recursos

Después de haber creado una plantilla de Resource Manager, puede usar Azure Portal para implementar los recursos de Azure. Parea crear una plantilla, consulte [Inicio rápido: Creación e implementación de plantillas de Azure Resource Manager mediante Azure Portal](../templates/quickstart-create-templates-use-the-portal.md). Para saber cómo implementar una plantilla con el portal, consulte el artículo sobre [implementación de recursos con las plantillas de Resource Manager y Azure Portal](../templates/deploy-portal.md).

## <a name="move-to-another-resource-group-or-subscription"></a>Traslado a otro grupo de recursos o suscripción

Puede mover los recursos de un grupo a otro grupo de recursos. Para obtener más información, consulte [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](move-resource-group-and-subscription.md).

## <a name="lock-resource-groups"></a>Bloqueo de grupos de recursos

Los bloqueos impiden que otros usuarios de la organización eliminen o modifiquen por error recursos esenciales, como una suscripción de Azure, un grupo de recursos o un recurso. 

1. Abra el grupo de recursos que quiera bloquear.  Vea [Abrir grupos de recursos](#open-resource-groups).
2. En el panel izquierdo, seleccione **Bloqueos**.
3. Seleccione **Agregar** para agregar un bloqueo al grupo de recursos.
4. Especifique un **Nombre del bloqueo**, un **Tipo de bloqueo** y **Notas**. Los tipos de bloqueo son **Solo lectura** y **Eliminar**.

    ![Bloqueo de grupo de recursos de Azure](./media/manage-resource-groups-portal/manage-resource-groups-add-lock.png)

Para obtener más información, vea [Bloqueo de recursos para impedir cambios inesperados](lock-resources.md).

## <a name="tag-resource-groups"></a>Etiquetado de grupos de recursos

Puede aplicar etiquetas a los recursos y grupos de recursos para organizar de manera lógica los recursos. Para obtener información, vea [Uso de etiquetas para organizar los recursos de Azure](tag-resources.md#portal).

## <a name="export-resource-groups-to-templates"></a>Exportación de grupos de recursos a plantillas

Para obtener más información sobre cómo exportar plantillas, vea [Exportar uno o varios recursos a una plantilla: Portal](../templates/export-template-portal.md).

## <a name="manage-access-to-resource-groups"></a>Administración del acceso a los grupos de recursos

El [control de acceso basado en rol (RBAC) de Azure](../../role-based-access-control/overview.md) es la forma en la que se administra el acceso a los recursos de Azure. Para saber más, vea [Administración del acceso mediante RBAC y Azure Portal](../../role-based-access-control/role-assignments-portal.md).

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información sobre Azure Resource Manager, consulte [Información general de Azure Resource Manager](overview.md).
- Para obtener información sobre la sintaxis de las plantillas de Resource Manager, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](../templates/template-syntax.md).
- Para obtener información sobre cómo desarrollar plantillas, consulte los [tutoriales paso a paso](../index.yml).
- Para ver los esquemas de plantilla de Azure Resource Manager, vea la [referencia de plantilla](/azure/templates/).

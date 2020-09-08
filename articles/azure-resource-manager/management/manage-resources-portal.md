---
title: 'Administración de recursos: Azure Portal'
description: Use Azure Portal y Azure Resource Manager para administrar los recursos. Muestra cómo implementar y eliminar los recursos.
author: mumian
ms.topic: conceptual
ms.date: 02/11/2019
ms.author: jgao
ms.openlocfilehash: 051361860a6a2d9ad70aa81e313857ec86cacfb4
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87827053"
---
# <a name="manage-azure-resources-by-using-the-azure-portal"></a>Administración de recursos de Azure con Azure Portal

Aprenda a utilizar [Azure Portal](https://portal.azure.com) con [Azure Resource Manager](overview.md) para administrar los recursos de Azure. Para la administración de grupos de recursos, consulte el artículo sobre [administración de grupos de recursos de Azure con Azure Portal](manage-resource-groups-portal.md).

Otros artículos sobre la administración de recursos:

- [Administración de grupos de recursos de Azure mediante la CLI de Azure](manage-resources-cli.md)
- [Administración de recursos de Azure con Azure PowerShell](manage-resources-powershell.md)

[!INCLUDE [Handle personal data](../../../includes/gdpr-intro-sentence.md)]

## <a name="deploy-resources-to-a-resource-group"></a>Implementación de recursos en un grupo de recursos

Después de haber creado una plantilla de Resource Manager, puede usar Azure Portal para implementar los recursos de Azure. Parea crear una plantilla, consulte [Inicio rápido: Creación e implementación de plantillas de Azure Resource Manager mediante Azure Portal](../templates/quickstart-create-templates-use-the-portal.md). Para saber cómo implementar una plantilla con el portal, consulte el artículo sobre [implementación de recursos con las plantillas de Resource Manager y Azure Portal](../templates/deploy-portal.md).

## <a name="open-resources"></a>Apertura de recursos

Los recursos de Azure se organizan por servicios de Azure y por grupos de recursos. Los procedimientos siguientes muestran cómo abrir una cuenta de almacenamiento denominada **mystorage0207**. La máquina virtual reside en un grupo de recursos denominado **mystorage0207rg**.

Para abrir un recurso de tipo de servicio:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. En el panel izquierdo, seleccione el servicio de Azure. En este caso, **Cuentas de almacenamiento**.  Si no ve el servicio en la lista, seleccione **Todos los servicios** y luego, el tipo de servicio.

    ![apertura del recurso de azure en el portal](./media/manage-resources-portal/manage-azure-resources-portal-open-service.png)

3. Seleccione el recurso que quiera abrir.

    ![apertura del recurso de azure en el portal](./media/manage-resources-portal/manage-azure-resources-portal-open-resource.png)

    Una cuenta de almacenamiento tiene el siguiente aspecto:

    ![apertura del recurso de azure en el portal](./media/manage-resources-portal/manage-azure-resources-portal-open-resource-storage.png)

Para abrir un recurso por grupo de recursos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. En el panel izquierdo, seleccione **Grupos de recursos** para mostrar el recurso en el grupo.
3. Seleccione el recurso que quiera abrir. 

## <a name="manage-resources"></a>Administrar recursos

Cuando vea un recurso en el portal, verá las opciones de administración de ese recurso concreto.

![administración de recursos de azure](./media/manage-resources-portal/manage-azure-resources-portal-manage-resource.png)

La captura de pantalla muestra las opciones de administración de una máquina virtual de Azure. Puede realizar operaciones tales como iniciar, reiniciar y detener una máquina virtual.

## <a name="delete-resources"></a>Eliminar recursos

1. Abra el recurso en el portal. Para conocer los pasos, consulte [Apertura de recursos](#open-resources).
2. Seleccione **Eliminar**. La captura de pantalla siguiente muestra las opciones de administración de una máquina virtual.

    ![eliminación de un recurso de Azure](./media/manage-resources-portal/manage-azure-resources-portal-delete-resource.png)
3. Escriba el nombre del recurso para confirmar la eliminación y luego seleccione **Eliminar**.

Para obtener más información sobre cómo ordena Azure Resource Manager la eliminación de recursos, consulte [Eliminación del grupo de recursos en Azure Resource Manager](delete-resource-group.md).

## <a name="move-resources"></a>Traslado de recursos

1. Abra el recurso en el portal. Para conocer los pasos, consulte [Apertura de recursos](#open-resources).
2. Seleccione **Mover**. La captura de pantalla siguiente muestra las opciones de administración para una cuenta de almacenamiento.

    ![traslado de recursos de Azure](./media/manage-resources-portal/manage-azure-resources-portal-move-resource.png)
3. Según sus necesidades, seleccione **Mover a otro grupo de recursos** o **Mover a otra suscripción**.

Para obtener más información, consulte [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](move-resource-group-and-subscription.md).

## <a name="lock-resources"></a>Bloqueo de recursos

Los bloqueos impiden que otros usuarios de la organización eliminen o modifiquen por error recursos esenciales, como una suscripción de Azure, un grupo de recursos o un recurso. 

1. Abra el recurso en el portal. Para conocer los pasos, consulte [Apertura de recursos](#open-resources).
2. Seleccione **Bloqueos**. La captura de pantalla siguiente muestra las opciones de administración para una cuenta de almacenamiento.

    ![bloqueo de recurso de Azure](./media/manage-resources-portal/manage-azure-resources-portal-lock-resource.png)
3. Seleccione **Agregar** y especifique las propiedades de bloqueo.

Para obtener más información, consulte [Bloqueo de recursos con el Administrador de recursos de Azure](lock-resources.md).

## <a name="tag-resources"></a>Etiquetado de recursos

El etiquetado ayuda a organizar lógicamente los grupos de recursos y los recursos. 

1. Abra el recurso en el portal. Para conocer los pasos, consulte [Apertura de recursos](#open-resources).
2. Seleccione **Etiquetas**. La captura de pantalla siguiente muestra las opciones de administración para una cuenta de almacenamiento.

    ![etiquetado del recurso de azure](./media/manage-resources-portal/manage-azure-resources-portal-tag-resource.png)
3. Especifique las propiedades de etiqueta y luego seleccione **Guardar**.

Para obtener información, vea [Uso de etiquetas para organizar los recursos de Azure](tag-resources.md#portal).

## <a name="monitor-resources"></a>Supervisión de recursos

Cuando se abre un recurso, el portal presenta los gráficos y las tablas predeterminados para supervisar ese tipo de recurso. La captura de pantalla siguiente muestra los gráficos de una máquina virtual:

![supervisión de recurso de Azure](./media/manage-resources-portal/manage-azure-resources-portal-monitor-resource.png)

Puede seleccionar el icono de anclaje en la esquina superior derecha de los gráficos para anclar el gráfico al panel. Para aprender a trabajar con paneles, vea el vídeo [Creación y uso compartido de paneles en Azure Portal](../../azure-portal/azure-portal-dashboards.md).

## <a name="manage-access-to-resources"></a>Administración del acceso a los recursos

El [control de acceso basado en rol (RBAC) de Azure](../../role-based-access-control/overview.md) es la forma en la que se administra el acceso a los recursos de Azure. Para saber más, vea [Administración del acceso mediante RBAC y Azure Portal](../../role-based-access-control/role-assignments-portal.md).

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información sobre Azure Resource Manager, consulte [Información general de Azure Resource Manager](overview.md).
- Para obtener información sobre la sintaxis de las plantillas de Resource Manager, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](../templates/template-syntax.md).
- Para obtener información sobre cómo desarrollar plantillas, consulte los [tutoriales paso a paso](../index.yml).
- Para ver los esquemas de plantilla de Azure Resource Manager, vea la [referencia de plantilla](/azure/templates/).

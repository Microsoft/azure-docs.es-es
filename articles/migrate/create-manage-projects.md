---
title: Creación y administración de proyectos de Azure Migrate
description: Busque, cree, administre y elimine proyectos en Azure Migrate.
ms.topic: how-to
ms.date: 07/23/2020
ms.openlocfilehash: d60868f9d0d4c60291cfd92a9e8d11fd3f9a42b9
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87071793"
---
# <a name="create-and-manage-azure-migrate-projects"></a>Creación y administración de proyectos de Azure Migrate

En este artículo se describe cómo crear, administrar y eliminar proyectos de [Azure Migrate](migrate-services-overview.md).


## <a name="create-a-project-for-the-first-time"></a>Creación de un proyecto por primera vez

La primera vez que se configura Azure Migrate, se crea un proyecto y se agrega una herramienta de evaluación o migración. [Siga estas instrucciones](how-to-add-tool-first-time.md) para realizar la configuración por primera vez.

## <a name="create-additional-projects"></a>Creación de proyectos adicionales

Si ya tiene un proyecto de Azure Migrate y quiere crear otro, haga lo siguiente:  

1. En el [portal público de Azure](https://portal.azure.com) o [Azure Government](https://portal.azure.us), busque **Azure Migrate**.
2. En el panel de Azure Migrate, vaya a **Servidores** y seleccione **Cambiar** en la esquina superior derecha.

   ![Cambio de un proyecto de Azure Migrate](./media/create-manage-projects/switch-project.png)

3. Para crear un nuevo proyecto, seleccione **Haga clic aquí**.

   ![Creación de un segundo proyecto de Azure Migrate](./media/create-manage-projects/create-new-project.png)


## <a name="find-a-project"></a>Búsqueda de un programa

Busque un proyecto como se indica a continuación:

1. En [Azure Portal](https://portal.azure.com), busque **Azure Migrate**.
2. En el panel de Azure Migrate, vaya a **Servidores** y seleccione **Cambiar** en la esquina superior derecha.

    ![Cambio a un proyecto de Azure Migrate existente](./media/create-manage-projects/switch-project.png)

3. Seleccione la suscripción y el proyecto de Azure Migrate adecuados.


Si creó el proyecto en la [versión anterior](migrate-services-overview.md#azure-migrate-versions) de Azure Migrate, haga lo siguiente para buscarlo:

1. En [Azure Portal](https://portal.azure.com), busque **Azure Migrate**.
2. En el panel de Azure Migrate, si ha creado un proyecto en la versión anterior, aparece un banner que hace referencia a los proyectos anteriores. Seleccione el banner.

    ![Acceso a proyectos existentes](./media/create-manage-projects/access-existing-projects.png)

3. Revise la lista de proyectos anteriores.


## <a name="delete-a-project"></a>Eliminación de un proyecto

Realice la eliminación de la siguiente manera:

1. Abra el grupo de recursos de Azure en el que se ha creado el proyecto.
2. En la página del grupo de recursos, seleccione **Mostrar tipos ocultos**.
3. Seleccione el proyecto de migración que quiere eliminar y los recursos asociados.
    - El tipo de recurso es **Microsoft.Migrate/migrateprojects**.
    - Si el grupo de recursos se utiliza de forma exclusiva en el proyecto de Azure Migrate, puede eliminar todo el grupo de recursos.

Observe lo siguiente:

- Al eliminar un proyecto, se eliminan tanto el proyecto como los metadatos sobre las máquinas detectadas.
- Si usa la versión anterior de Azure Migrate, abra el grupo de recursos de Azure en el que se creó el proyecto. Seleccione el proyecto de migración que quiere eliminar (el tipo de recurso es **Proyecto de migración**).
- Si usa el análisis de dependencias con una área de trabajo de Azure Log Analytics:
    - Si ha vinculado un área de trabajo de Log Analytics a la herramienta Server Assessment, el área de trabajo no se elimina automáticamente. La misma área de trabajo de Log Analytics se puede usar para varios escenarios.
    - Si desea eliminar el área de trabajo de Log Analytics, hágalo manualmente.
- La eliminación de un proyecto es irreversible. Los objetos eliminados no se pueden recuperar.

### <a name="delete-a-workspace-manually"></a>Eliminación manual de un área de trabajo

1. Vaya al área de trabajo de Log Analytics vinculada al proyecto.

    - Si no ha eliminado el proyecto de Azure Migrate, puede encontrar el vínculo al área de trabajo en **Essentials** > **Server Assessment**.
       ![Área de trabajo de LA](./media/create-manage-projects/loganalytics-workspace.png).
       
    - Si ya ha eliminado el proyecto de migración, seleccione **Grupos de recursos** en el panel izquierdo de Azure Portal y busque el área de trabajo.
       
2. [Siga las instrucciones](../azure-monitor/platform/delete-workspace.md) para eliminar el área de trabajo.

## <a name="next-steps"></a>Pasos siguientes

Agregue herramientas de [evaluación](how-to-assess.md) o [migración](how-to-migrate.md) a los proyectos de Azure Migrate.

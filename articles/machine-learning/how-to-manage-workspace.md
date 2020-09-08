---
title: Creación de áreas de trabajo en el portal
titleSuffix: Azure Machine Learning
description: Aprenda a crear, ver y eliminar áreas de trabajo de Azure Machine Learning en Azure Portal.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: sgilley
author: sdgilley
ms.date: 12/27/2019
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: e1f97fddb07e56946e37c04d9b9685412782c560
ms.sourcegitcommit: d18a59b2efff67934650f6ad3a2e1fe9f8269f21
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88659762"
---
# <a name="create-and-manage-azure-machine-learning-workspaces-in-the-azure-portal"></a>Creación y administración de áreas de trabajo de Azure Machine Learning en Azure Portal
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

En este artículo creará, verá y eliminará [**áreas de trabajo de Azure Machine Learning**](concept-workspace.md) en Azure Portal para [Azure Machine Learning](overview-what-is-azure-ml.md).  El portal es la forma más sencilla de empezar a trabajar con áreas de trabajo, pero a medida que cambian las necesidades o aumentan los requisitos de automatización, también puede crear y eliminar áreas de trabajo [mediante la CLI](reference-azure-machine-learning-cli.md), [con código de Python](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py) o [a través de la extensión de VS Code](tutorial-setup-vscode-extension.md).

## <a name="create-a-workspace"></a>Crear un área de trabajo

Para crear un área de trabajo, necesita una suscripción de Azure. Si no tiene una suscripción de Azure, cree una cuenta gratuita antes de empezar. Pruebe hoy mismo la [versión gratuita o de pago de Azure Machine Learning](https://aka.ms/AMLFree).

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con las credenciales de la suscripción de Azure. 

1. En la esquina superior izquierda de Azure Portal, seleccione **+ Crear un recurso**.

      ![Crear un nuevo recurso](./media/how-to-manage-workspace/create-workspace.gif)

1. Use la barra de búsqueda para encontrar **Machine Learning**.

1. Seleccione **Machine Learning**.

1. En el panel **Machine Learning**, seleccione **Crear** para comenzar.

1. Especifique la siguiente información para configurar la nueva área de trabajo:

   Campo|Descripción 
   ---|---
   Nombre del área de trabajo |Escriba un nombre único que identifique el área de trabajo. En este ejemplo, se usa **docs-ws**. Los nombres deben ser únicos en el grupo de recursos. Utilice un nombre que sea fácil de recordar y que se diferencie del de las áreas de trabajo creadas por otros. El nombre del área de trabajo no distingue mayúsculas de minúsculas.
   Suscripción |Seleccione la suscripción de Azure que quiera usar.
   Resource group | Use un grupo de recursos existente en su suscripción o escriba un nombre para crear un nuevo grupo de recursos. Un grupo de recursos almacena los recursos relacionados con una solución de Azure. En este ejemplo, se usa **docs-aml**. Necesita el rol *colaborador* o *propietario* para usar un grupo de recursos existente.  Para obtener más información sobre el acceso, consulte [Administración del acceso a un área de trabajo de Azure Machine Learning](how-to-assign-roles.md).
   Location | Seleccione la ubicación más cercana a los usuarios y los recursos de datos para crear el área de trabajo.
   Location | Seleccione la ubicación más cercana a los usuarios y los recursos de datos para crear el área de trabajo.
   Edición del área de trabajo | Seleccione **Basic** o **Enterprise**.  Esta edición del área de trabajo determina las características a las que tendrá acceso y los precios. Más información sobre las [ofertas de las ediciones Basic y Enterprise](overview-what-is-azure-ml.md#sku). 

    ![Configuración de un área de trabajo](./media/how-to-manage-workspace/select-edition.png)

1. Cuando haya terminado de configurar el área de trabajo, seleccione **Revisar y crear**.
2. Revise la configuración y realice cualquier cambio o corrección adicional. Cuando esté satisfecho con la configuración, seleccione **Crear**.

   > [!Warning] 
   > La creación del área de trabajo en la nube puede tardar varios minutos.

   Cuando finalice el proceso, aparecerá un mensaje de implementación correcta. 
 
 1. Para ver la nueva área de trabajo, seleccione **Ir al recurso**.

### <a name="download-a-configuration-file"></a>Descarga de un archivo de configuración

1. Si va a crear una [instancia de proceso](tutorial-1st-experiment-sdk-setup.md#azure), omita este paso.

1. Si tiene previsto usar código en el entorno local que haga referencia a esta área de trabajo, seleccione **Descargar config.json** en la sección **Información general** del área de trabajo.  

   ![Descargar config.json](./media/how-to-manage-workspace/configure.png)
   
   Coloque el archivo en la estructura de directorios que contiene los scripts de Python o las instancias de Jupyter Notebook. Puede estar en el mismo directorio, en un subdirectorio denominado *.azureml* o en un directorio principal. Al crear una instancia de proceso, este archivo se agrega automáticamente al directorio correcto de la máquina virtual.

## <a name="upgrade-to-enterprise-edition"></a><a name="upgrade"></a>Actualización a la edición Enterprise

Puede actualizar el área de trabajo de la edición Basic a la edición Enterprise para aprovechar las ventajas de las características mejoradas, como experiencias de poco código y características de seguridad mejoradas.

1. Inicie sesión en [Azure Machine Learning Studio](https://ml.azure.com).

1. Seleccione el área de trabajo que desea actualizar.

1. Seleccione **Más información** en la parte superior derecha de la página.

   [ ![Actualización de un área de trabajo](./media/how-to-manage-workspace/upgrade.png) ](./media/how-to-manage-workspace/upgrade.png#lightbox)

1. Seleccione **Actualizar** en la ventana que aparece.


> [!IMPORTANT]
> No se puede rebajar un área de trabajo de la edición Enterprise a un área de trabajo de la edición Basic. 

## <a name="find-a-workspace"></a><a name="view"></a>Buscar un área de trabajo

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. En el campo de búsqueda superior, escriba **Machine Learning**.  

1. Seleccione **Machine Learning**.

   ![Búsqueda del área de trabajo de Azure Machine Learning](./media/how-to-manage-workspace/find-workspaces.png)

1. Examine la lista de áreas de trabajo encontradas. Puede filtrar en función de suscripción, grupos de recursos y ubicaciones.  

1. Seleccione un área de trabajo para mostrar sus propiedades.

## <a name="delete-a-workspace"></a>Eliminar un área de trabajo

En [Azure Portal](https://portal.azure.com/), seleccione **Eliminar** en la parte superior del área de trabajo que desea eliminar.

:::image type="content" source="./media/how-to-manage-workspace/delete-workspace.png" alt-text="Eliminación del área de trabajo":::

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [aml-delete-resource-group](../../includes/aml-delete-resource-group.md)]

## <a name="troubleshooting"></a>Solución de problemas

### <a name="resource-provider-errors"></a>Errores del proveedor de recursos

[!INCLUDE [machine-learning-resource-provider](../../includes/machine-learning-resource-provider.md)]

### <a name="moving-the-workspace"></a>Movimiento del área de trabajo

> [!WARNING]
> No se admite mover el área de trabajo de Azure Machine Learning a otra suscripción ni mover la suscripción propietaria a un nuevo inquilino. Si lo hace, pueden producirse errores.

### <a name="deleting-the-azure-container-registry"></a>Eliminación de la instancia de Azure Container Registry

El área de trabajo de Azure Machine Learning usa Azure Container Registry (ACR) para algunas operaciones. La primera vez que se necesite una instancia de ACR, se creará automáticamente.

[!INCLUDE [machine-learning-delete-acr](../../includes/machine-learning-delete-acr.md)]

## <a name="next-steps"></a>Pasos siguientes

Siga el tutorial completo para obtener información sobre cómo usar un área de trabajo para compilar, entrenar e implementar modelos con Azure Machine Learning.

> [!div class="nextstepaction"]
> [Tutorial: Entrenamiento de modelos](tutorial-train-models-with-aml.md)

---
title: Creación de un laboratorio mediante Azure DevTest Labs y una plantilla de Azure Resource Manager
description: En este inicio rápido, creará un laboratorio en Azure DevTest Labs mediante una plantilla de Azure Resource Manager (plantilla de ARM). Los administradores de laboratorio configuran laboratorios, crean máquinas virtuales en los laboratorios y configuran las directivas.
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 06/26/2020
ms.openlocfilehash: 93bc4cf91b228de72c217c61ec03d61e87046dd3
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88653960"
---
# <a name="quickstart-set-up-a-lab-by-using-azure-devtest-labs-arm-template"></a>Inicio rápido: Configuración de un laboratorio mediante una plantilla de ARM de Azure DevTest Labs
En este inicio rápido, creará un laboratorio con una máquina virtual Windows Server 2019 Datacenter mediante el uso de una plantilla de Azure Resource Manager (ARM). 

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

En este inicio rápido, lleve a cabo las acciones siguientes:

> [!div class="checklist"]
> * Revisión de la plantilla 
> * Implementación de la plantilla
> * Comprobación de la plantilla
> * Limpieza de recursos

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-dtl-create-lab-windows-vm%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-dtl-create-lab-windows-vm/).

:::code language="json" source="~/quickstart-templates/101-dtl-create-lab-windows-vm/azuredeploy.json":::

Los recursos definidos en la plantilla incluyen:

- [**Microsoft.DevTestLab/labs**](/azure/templates/microsoft.devtestlab/labs)
- [**Microsoft.DevTestLab labs/virtualnetworks**](/azure/templates/microsoft.devtestlab/labs/virtualnetworks)
- [**Microsoft.DevTestLab labs/virtualmachines**](/azure/templates/microsoft.devtestlab/labs/virtualmachines)

Para buscar más ejemplos de plantillas, consulte [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Devtestlab).

## <a name="deploy-the-template"></a>Implementación de la plantilla
Para ejecutar automáticamente la implementación, haga clic en el botón siguiente. 

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-dtl-create-lab-windows-vm%2Fazuredeploy.json)

1. Cree un **nuevo grupo de recursos** para que facilitar la limpieza más adelante.
1. Seleccione una **ubicación** para el grupo de recursos. 
1. Escriba el **nombre del laboratorio**. 
1. Escriba el **nombre de la máquina virtual**. 
1. Escriba un **nombre de usuario** que pueda acceder a la máquina virtual. 
1. Escriba la **contraseña** del usuario. 
1. Seleccione **Acepto los términos y condiciones indicados anteriormente**. 
1. Después, seleccione **Adquirir**.

    :::image type="content" source="./media/create-lab-windows-vm-template/deploy-template-page.png" alt-text="Página de implementación de la plantilla":::

## <a name="validate-the-deployment"></a>Validación de la implementación
1. Seleccione **Notificaciones** en la parte superior para ver el estado de la implementación y haga clic en el vínculo **Implementación en curso**.

    :::image type="content" source="./media/create-lab-windows-vm-template/deployment-notification.png" alt-text="Notificación de implementación":::
2. En la página **Información general de la implementación**, espere hasta que esta se complete. Esta operación (especialmente la creación de una máquina virtual) tarda algún tiempo en completarse. A continuación, seleccione **Ir al grupo de recursos** o el **nombre del grupo de recursos** como se muestra en la siguiente imagen: 

    :::image type="content" source="./media/create-lab-windows-vm-template/navigate-resource-group.png" alt-text="Navegación al grupo de recursos":::
3. En la página **Grupo de recursos**, verá la lista de recursos del grupo de recursos. Confirme que entre los recursos se encuentra el laboratorio de tipo: `DevTest Lab`. Dentro del grupo de recursos también verá los recursos dependientes, como la red virtual y la máquina virtual. 

    :::image type="content" source="./media/create-lab-windows-vm-template/resource-group-home-page.png" alt-text="Página principal del grupo de recursos":::
4. Seleccione el laboratorio en la lista de recursos para ver la página principal del mismo. Confirme que la máquina virtual Windows Server 2019 Datacenter se encuentra en la lista **Mis máquinas virtuales**. En la imagen siguiente, la sección **Información esencial** aparece minimizada. 

    :::image type="content" source="./media/create-lab-windows-vm-template/lab-home-page.png" alt-text="Página principal del laboratorio":::

    > [!IMPORTANT] 
    > Mantenga esta página abierta y aplique las instrucciones de la sección siguiente para limpiar los recursos y evitar así los costos de ejecución del laboratorio y la máquina virtual en Azure. Si desea consultar el siguiente tutorial para probar el acceso a la máquina virtual en el laboratorio, limpie los recursos después de realizar el tutorial. 

## <a name="clean-up-resources"></a>Limpieza de recursos

1. En primer lugar, elimine el laboratorio para poder eliminar el grupo de recursos. No podrá eliminar el grupo de recursos si este contiene un laboratorio. Para eliminar el laboratorio, seleccione **Eliminar** en la barra de herramientas. 

    :::image type="content" source="./media/create-lab-windows-vm-template/delete-lab-button.png" alt-text="Botón Eliminar laboratorio":::
 2. En la página Confirmación, escriba el **nombre del laboratorio** y seleccione **Eliminar**. 
 3. Espere hasta que se elimine el laboratorio. Seleccione el icono de la **campana** para ver la notificación de la operación de eliminación. Este proceso tarda algún tiempo. Confirme la eliminación del laboratorio y, a continuación, seleccione el **grupo de recursos** en el menú de la ruta de navegación. 
 
    :::image type="content" source="./media/create-lab-windows-vm-template/confirm-lab-deletion.png" alt-text="Confirmación de la eliminación de la máquina virtual en las notificaciones":::
 1. En la página **Grupo de recursos**, seleccione **Eliminar grupo de recursos**. En la página de confirmación, escriba el **nombre del grupo de recursos** y seleccione **Eliminar**. Compruebe las notificaciones para confirmar que se ha eliminado el grupo de recursos.
 
    :::image type="content" source="./media/create-lab-windows-vm-template/delete-resource-group-button.png" alt-text="Botón Eliminar grupo de recursos":::

## <a name="next-steps"></a>Pasos siguientes
En este inicio rápido, ha creado un laboratorio con una máquina virtual. Para información acerca de cómo acceder al laboratorio, avance hasta el tutorial siguiente:

> [!div class="nextstepaction"]
> [Tutorial: Acceso al laboratorio](tutorial-use-custom-lab.md)

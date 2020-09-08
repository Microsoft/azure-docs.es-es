---
title: Traslado de un espacio de nombres de Service Bus a otra región | Microsoft Docs
description: En este artículo se muestra cómo trasladar un espacio de nombres de Azure Service Bus desde la región actual a otra región.
ms.topic: how-to
ms.date: 06/23/2020
ms.custom: subject-moving-resources
ms.openlocfilehash: 573ea96437cf6cb76854ffa417fd3ad3fb86138b
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88860486"
---
# <a name="move-an-azure-service-bus-namespace-to-another-region"></a>Traslado de un espacio de nombres de Azure Service Bus a otra región
Existen varios escenarios en los que puede que deba trasladar su espacio de nombres de Service Bus existente de una región a otra. Por ejemplo, si desea crear un espacio de nombres con la misma configuración para realizar pruebas. También es posible que desee mover un espacio de nombres secundario a otra región como parte del [planeamiento de recuperación ante desastres](service-bus-geo-dr.md).

Los pasos generales son los siguientes:

1. Exporte el espacio de nombres de Service Bus de la región actual a una plantilla de Azure Resource Manager. 
1. Actualice la ubicación de los recursos en la plantilla. Además, quite el filtro de suscripción predeterminado de la plantilla porque no se puede crear una regla predeterminada, ya que se crea automáticamente. 
1. Use la plantilla para implementar el espacio de nombres de Service Bus en la región de destino. 
1. Compruebe la implementación para asegurarse de que el espacio de nombres, las colas, los temas y las suscripciones de los temas se han creado en la región de destino. 
1. Complete el traslado eliminando el espacio de nombres de la región de origen después de procesar todos los mensajes. 

## <a name="prerequisites"></a>Requisitos previos
Asegúrese de que Azure Service Bus y las características que usa su cuenta se admitan en la región de destino.
 
## <a name="prepare"></a>Preparación
Para empezar, exporte una plantilla de Resource Manager. Esta plantilla contiene la configuración que describe el espacio de nombres de Service Bus.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **Todos los recursos** y, después, seleccione su espacio de nombres de Service Bus.
3. Seleccione > **Configuración** > **Exportar plantilla**.
4. Elija **Descargar** en la página **Exportar plantilla**.

    :::image type="content" source="./media/move-across-regions/download-template.png" alt-text="Descarga de la plantilla de Resource Manager":::
5. Busque el archivo ZIP que descargó desde el portal y descomprímalo en la carpeta que prefiera. Este archivo ZIP contiene los archivos JSON de plantilla y parámetros. 
1. Abra el archivo template.json en la carpeta extraída. 
1. Busque `location` y reemplace el valor de la propiedad por el nuevo nombre de la región o ubicación. Para obtener los códigos de ubicación, consulte [Ubicaciones de Azure](https://azure.microsoft.com/global-infrastructure/locations/). El código de una región es el nombre de la región sin espacios, por ejemplo, `West US` es igual a `westus`.
1. Quite definiciones de recursos de tipo: `Microsoft.ServiceBus/namespaces/topics/subscriptions/rules`. No olvide quitar el carácter de coma (`,`) que precede a esta sección para conservar la validez de JSON.  

    > [!NOTE]
    > No se puede crear una regla predeterminada para una suscripción mediante el uso de una plantilla de Resource Manager. La regla predeterminada se crea automáticamente cuando se crea la suscripción en la región de destino. 

## <a name="move"></a>Mover
Implemente la plantilla para crear un espacio de nombres de Service Bus en la región de destino. 

1. En Azure Portal, haga clic en **Crear un recurso**.
2. En **Buscar en Marketplace**, escriba **implementación de plantillas** para el texto de búsqueda, seleccione **Template Deployment (implementar mediante plantillas personalizadas)** y, a continuación, presione **Entrar**.

    :::image type="content" source="./media/move-across-regions/new-template-deployment.png" alt-text="Nueva implementación de plantilla":::    
1. En la página **Implementación de plantillas**, seleccione **Crear**.

    :::image type="content" source="./media/move-across-regions/template-deployment-create-button.png" alt-text="Nueva implementación de plantilla, botón Crear":::        
1. En la página **Implementación personalizada**, seleccione **Cree su propia plantilla en el editor**.

    :::image type="content" source="./media/move-across-regions/build-template-link.png" alt-text="Cree su propia plantilla en el editor, vínculo":::            
1. En la página **Editar plantilla**, seleccione **Cargar archivo** en la barra de herramientas y, después, siga las instrucciones para cargar el archivo **template.json** que descargó en la última sección.

    :::image type="content" source="./media/move-across-regions/select-template.png" alt-text="Seleccionar plantilla":::                
1. Seleccione **Guardar** para guardar la plantilla. 

    :::image type="content" source="./media/move-across-regions/save-template.png" alt-text="Guardar plantilla":::                    
1. En la página **Implementación personalizada**, siga estos pasos: 
    1. Seleccione una **suscripción** de Azure. 
    2. Seleccione un **grupo de recursos** existente o cree uno. 
    3. Seleccione la **ubicación** o región de destino. Si ha seleccionado un grupo de recursos existente, esta configuración es de solo lectura. 
    4. Escriba un nuevo **nombre para el espacio de nombres**.
    1. Seleccione **Revisar + crear**. 

        :::image type="content" source="./media/move-across-regions/deploy-template.png" alt-text="Implementación de la plantilla de Resource Manager":::
    1. En la página **Revisar y crear**, seleccione **Crear** en la parte inferior de la página. 
    
## <a name="verify"></a>Comprobar
1. Cuando la implementación sea correcta, seleccione **Ir al grupo de recursos**.

    :::image type="content" source="./media/move-across-regions/resource-group-navigation-link.png" alt-text="Vínculo Ir al grupo de recursos":::    
1. En la página **Grupo de recursos**, seleccione el espacio de nombres de Service Bus. 

    :::image type="content" source="./media/move-across-regions/select-namespace.png" alt-text="Selección del espacio de nombres de Service Bus":::    
1. En la página **Espacio de nombres de Service Bus**, compruebe que ve las colas, los temas y las suscripciones de la región de origen. 
    1. Verá **colas** en el espacio de nombres, en la parte inferior del panel derecho.         
    
        :::image type="content" source="./media/move-across-regions/queue-namespace.png" alt-text="Colas en el espacio de nombres":::
    2. Cambie a la pestaña **Temas** para ver los temas en el espacio de nombres.
    
        :::image type="content" source="./media/move-across-regions/topics-namespace.png" alt-text="Temas en el espacio de nombres":::
    3. Seleccione el tema para comprobar que se han creado las suscripciones. 

        :::image type="content" source="./media/move-across-regions/topic-subscriptions.png" alt-text="Suscripciones de tema":::      
    
    

## <a name="discard-or-clean-up"></a>Descarte o limpieza
Después de la implementación, si quiere empezar de nuevo, puede eliminar el **espacio de nombres de Service Bus de destino** y repetir los pasos descritos en las secciones [Preparación](#prepare) y [Traslado](#move) de este artículo.

Para confirmar los cambios y completar el traslado de un espacio de nombres de Service Bus, elimine el **espacio de nombres de Service Bus de origen**. Asegúrese de procesar todos los mensajes antes de eliminar el espacio de nombres. 

Para eliminar un espacio de nombres de Service Bus (de origen o destino) mediante Azure Portal:

1. En la ventana de búsqueda de la parte superior de Azure Portal, escriba **Service Bus** y seleccione **Service Bus** en los resultados de la búsqueda. Verá los espacios de nombres de Service Bus en una lista.
2. Seleccione el espacio de nombres de destino que desea eliminar y, a continuación, seleccione **Eliminar** en la barra de herramientas. 

    ![Eliminar espacio de nombres - botón](./media/move-across-regions/delete-namespace-button.png)
3. En la página **Eliminar recursos**, compruebe los recursos seleccionados y confirme la eliminación escribiendo **yes** y, a continuación, seleccione **Eliminar**. 

    Otra opción es eliminar el grupo de recursos que contiene el espacio de nombres de Service Bus. En la página **Grupo de recursos**, seleccione **Eliminar grupo de recursos** en la barra de herramientas y, a continuación, confirme la eliminación. 

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha trasladado un espacio de nombres de Azure Service Bus de una región a otra y ha limpiado los recursos de origen.  Para obtener más información sobre cómo trasladar recursos entre regiones y la recuperación ante desastres en Azure, consulte:

- [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../azure-resource-manager/management/move-resource-group-and-subscription.md)

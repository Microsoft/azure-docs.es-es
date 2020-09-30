---
title: Traslado de temas del sistema de Azure Event Grid a otra región
description: En este artículo se muestra cómo trasladar temas personalizados del sistema de Azure Event Grid de una región a otra.
ms.topic: how-to
ms.custom: subject-moving-resources
ms.date: 08/28/2020
ms.openlocfilehash: d0656a4f6ec1c7431cf7111f786b0f1d779166e3
ms.sourcegitcommit: d7352c07708180a9293e8a0e7020b9dd3dd153ce
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/30/2020
ms.locfileid: "89145353"
---
# <a name="move-azure-event-grid-custom-topics-to-another-region"></a>Traslado de temas del sistema de Azure Event Grid a otra región
Puede mover sus recursos a otra región por varios motivos. Por ejemplo, para aprovechar una nueva región de Azure, para cumplir los requisitos de gobernanza y las directivas internas o como respuesta a los requisitos de planeamiento de capacidad. 

Estos son los pasos resumidos que se describen en este artículo: 

- **Exporte el recurso del tema personalizado** a una plantilla de Azure Resource Manager. 

    > [!IMPORTANT]
    > Solo se exporta el tema personalizado a la plantilla. No se exportan las suscripciones del tema.
- **Use la plantilla para implementar el tema personalizado** en la región de destino. 
- **Cree suscripciones de forma manual** en la región de destino. Cuando exportó el tema personalizado a una plantilla en la región actual, solo se exporta el tema. Las suscripciones no se incluyen en la plantilla, por lo que debe crearlas manualmente una vez creado el tema personalizado en la región de destino. 
- **Comprobar la implementación**. Compruebe que el tema personalizado se crea en la región de destino. 
- Para **completar el traslado**, elimine el tema personalizado de la región de origen. 

## <a name="prerequisites"></a>Requisitos previos
- Complete la guía [Inicio rápido: Enrutamiento de eventos personalizados al punto de conexión web](custom-event-quickstart-portal.md) en la región de origen. Realice este paso para poder probar los pasos de este artículo. 
- Asegúrese de que el servicio Event Grid esté disponible en la región de destino. Consulte los [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=event-grid&regions=all).

## <a name="prepare"></a>Preparación
Para empezar, exporte una plantilla de Resource Manager para el tema personalizado. 

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. En la barra de búsqueda del tema, escriba **Temas de Event Grid** y, a continuación, seleccione **Temas de Event Grid** en la lista desplegable. 

    :::image type="content" source="./media/move-custom-topics-across-regions/search-topics.png" alt-text="Búsqueda y selección de temas de Event Grid":::
3. Seleccione el **tema** que desea exportar a una plantilla de Resource Manager. 

    :::image type="content" source="./media/move-custom-topics-across-regions/select-custom-topic.png" alt-text="Selección del tema personalizado":::   
4. En la página **Tema de Event Grid**, seleccione **Exportar plantilla** en **Configuración** en el menú de la izquierda y, a continuación, seleccione **Descargar** en la barra de herramientas. 

    :::image type="content" source="./media/move-custom-topics-across-regions/export-template-download.png" alt-text="Exportar plantilla -> Descargar":::   

    > [!IMPORTANT]
    > Solo se exporta el tema a la plantilla. No se exportan las suscripciones del tema. Por lo tanto, debe crear suscripciones para el tema después de trasladar el tema a la región de destino. 
5. Busque el archivo **.zip** que descargó desde el portal y descomprímalo en la carpeta que prefiera. Este archivo .zip contiene los archivos JSON de plantilla y parámetros. 
1. Abra **template.json** en el editor que prefiera. 
8. Actualice `location` para el recurso de **tema** en la región o ubicación de destino. Para obtener los códigos de ubicación, consulte [Ubicaciones de Azure](https://azure.microsoft.com/global-infrastructure/locations/). El código de una región es el nombre de la región sin espacios, por ejemplo, `West US` es igual a `westus`.

    ```json
    "type": "Microsoft.EventGrid/topics",
    "apiVersion": "2020-06-01",
    "name": "[parameters('topics_mytopic0130_name')]",
    "location": "westus"
    ```
1. **Guarde** la plantilla. 

## <a name="recreate"></a>Volver a crear 
Implemente la plantilla para crear un tema personalizado en la región de destino. 

1. En Azure Portal, haga clic en **Crear un recurso**.
2. En **Buscar en Marketplace**, escriba **implementación de plantillas** y, después, presione **ENTRAR**.
3. Seleccione **Implementación de plantillas**.
4. Seleccione **Crear**.
5. Seleccione **Cree su propia plantilla en el editor**.
6. Seleccione **Cargar archivo** y, después, siga las instrucciones para cargar el archivo **template.json** que descargó en la última sección.
7. Seleccione **Guardar** para guardar la plantilla. 
8. En la página **Implementación personalizada**, siga estos pasos: 
    1. Seleccione una **suscripción** de Azure. 
    1. Seleccione un **grupo de recursos** en la región de destino o cree uno. 
    1. En **Región**, seleccione la región de destino. Si ha seleccionado un grupo de recursos existente, esta configuración es de solo lectura. 
    1. En el **nombre de tema**, escriba un nuevo nombre para el tema. 
    1. En la parte inferior de la página, seleccione **Revisar y crear**. 
    
        :::image type="content" source="./media/move-custom-topics-across-regions/deploy-template.png" alt-text="Implementación personalizada":::
    1. En la página **Revisar y crear**, examine la configuración y seleccione **Crear**. 

## <a name="verify"></a>Comprobar

1. Una vez que la implementación se haya realizado correctamente, seleccione **Ir al recurso**. 

    :::image type="content" source="./media/move-custom-topics-across-regions/navigate-custom-topic.png" alt-text="Ir al recurso":::
1. Confirme que ve la página **Tema de Event Grid**  del tema personalizado.   
1. Siga los pasos de la [Enrutamiento de eventos personalizados a un punto de conexión web](custom-event-quickstart-portal.md#send-an-event-to-your-topic) para enviar eventos al tema. Compruebe que se invoca el controlador de eventos de webhook. 

## <a name="discard-or-clean-up"></a>Descarte o limpieza
Para completar el traslado, elimine el tema personalizado de la región de origen.  

Si desea volver a empezar, elimine el tema en la región de destino y repita los pasos de las secciones [Preparar](#prepare) y [Volver a crear](#recreate) de este artículo.

Para crear un tema mediante Azure Portal, siga estos pasos:

1. En la ventana de búsqueda de la parte superior de Azure Portal, escriba **Temas de Event Grid** y seleccione **Temas de Event Grid** en los resultados de la búsqueda. 
2. Seleccione el tema que desea eliminar y, a continuación, seleccione **Eliminar** en la barra de herramientas. 
3. En la página de confirmación, escriba el nombre del grupo de recursos y seleccione **Eliminar**.  

Para eliminar el grupo de recursos que contiene el tema personalizado mediante Azure Portal, siga estos pasos:

1. En la ventana de búsqueda de la parte superior de Azure Portal, escriba **Grupos de recursos** y seleccione **Grupos de recursos** de los resultados de la búsqueda. 
2. Seleccione el grupo de recursos que desea eliminar y, a continuación, seleccione **Eliminar** en la barra de herramientas. 
3. En la página de confirmación, escriba el nombre del grupo de recursos y seleccione **Eliminar**.  

## <a name="next-steps"></a>Pasos siguientes
Ha aprendido a trasladar un tema de Event Grid personalizado de una región a otra. Consulte los artículos siguientes para trasladar temas del sistema, dominios y espacios de nombres del asociado entre regiones.

- [Traslado de temas del sistema entre regiones](move-system-topics-across-regions.md) 
- [Traslado de dominios entre regiones](move-domains-across-regions.md) 
- [Traslado de espacios de nombres del asociado entre regiones](move-partner-namespaces-across-regions.md)

Para obtener más información sobre el traslado de recursos entre regiones y la recuperación ante desastres en Azure, consulte el siguiente artículo: [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../azure-resource-manager/management/move-resource-group-and-subscription.md)
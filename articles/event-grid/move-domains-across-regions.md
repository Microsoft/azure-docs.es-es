---
title: Traslado de dominios de Azure Event Grid a otra región
description: En este artículo se muestra cómo trasladar dominios de Azure Event Grid de una región a otra.
ms.topic: how-to
ms.custom: subject-moving-resources
ms.date: 08/20/2020
ms.openlocfilehash: fff8638a819511f84f15c52ad0695cdd5759f971
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89082643"
---
# <a name="move-azure-event-grid-domains-to-another-region"></a>Traslado de dominios de Azure Event Grid a otra región
Puede mover sus recursos a otra región por varios motivos. Por ejemplo, para aprovechar una nueva región de Azure, para cumplir los requisitos de gobernanza y las directivas internas o como respuesta a los requisitos de planeamiento de capacidad. 

Estos son los pasos resumidos que se describen en este artículo: 

- **Exporte el recurso de dominio** a una plantilla de Azure Resource Manager. 

    > [!IMPORTANT]
    > El recurso de dominio y los temas del dominio se exportan a la plantilla. No se exportan las suscripciones a los temas del dominio. 
- **Use la plantilla para implementar el dominio** en la región de destino. 
- **Cree suscripciones para temas del dominio de forma manual** en la región de destino. Cuando exportó el dominio a una plantilla en la región actual, no se exportan las suscripciones para los temas del dominio. Por lo tanto, créelos después de que los temas del dominio y el dominio se creen en la región de destino. 
- **Comprobar la implementación**. Envíe un evento a un tema del dominio y compruebe que se invoca el controlador de eventos asociado a la suscripción. 
- Para **completar el traslado**, elimine el dominio de la región de origen. 

## <a name="prerequisites"></a>Requisitos previos
- Asegúrese de que el servicio Event Grid esté disponible en la región de destino. Consulte los [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=event-grid&regions=all).

## <a name="prepare"></a>Preparación
Para empezar, exporte una plantilla de Resource Manager para el dominio. 

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. En la barra de búsqueda del tema, escriba **Dominios de Event Grid** y, a continuación, seleccione **Dominios de Event Grid** en la lista desplegable. 

    :::image type="content" source="./media/move-domains-across-regions/search-domains.png" alt-text="Búsqueda y selección de dominios de Event Grid":::
3. Seleccione el **dominio** que desea exportar a una plantilla de Resource Manager. 

    :::image type="content" source="./media/move-domains-across-regions/select-domain.png" alt-text="Selección del dominio":::   
4. En la página **Dominio de Event Grid**, seleccione **Exportar plantilla** en **Configuración** en el menú de la izquierda y, a continuación, seleccione **Descargar** en la barra de herramientas. 

    :::image type="content" source="./media/move-domains-across-regions/export-template-download.png" alt-text="Exportar plantilla -> Descargar" lightbox="./media/move-domains-across-regions/export-template-download.png":::   

    > [!IMPORTANT]
    > El dominio y los temas del dominio se exportan. No se exportan las suscripciones a los temas del dominio. Por lo tanto, debes crear suscripciones para los temas del dominio después de trasladar los temas del dominio. 
5. Busque el archivo **.zip** que descargó desde el portal y descomprímalo en la carpeta que prefiera. Este archivo .zip contiene los archivos JSON de plantilla y parámetros. 
1. Abra **template.json** en el editor que prefiera. 
8. Actualice `location` para el recurso de **dominio** en la región o ubicación de destino. Para obtener los códigos de ubicación, consulte [Ubicaciones de Azure](https://azure.microsoft.com/global-infrastructure/locations/). El código de una región es el nombre de la región sin espacios, por ejemplo, `West US` es igual a `westus`.

    ```json
    "type": "Microsoft.EventGrid/domains",
    "apiVersion": "2020-06-01",
    "name": "[parameters('domains_spegriddomain_name')]",
    "location": "westus",
    ```
1. **Guarde** la plantilla. 

## <a name="recreate"></a>Volver a crear 
Implemente la plantilla para crear los temas de dominio y dominio en la región de destino. 

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
    1. En el **nombre de dominio**, escriba un nuevo nombre para el dominio. 
    1. Seleccione **Revisar + crear**. 
    
        :::image type="content" source="./media/move-domains-across-regions/deploy-template.png" alt-text="Implementar plantilla":::        
    1. Una vez que la validación de la plantilla sea correcta, seleccione **Crear** en la parte inferior de la página para implementar el recurso. 
    1. Una vez finalizada la implementación, seleccione **Ir al grupo de recursos** para ir a la página del grupo de recursos. Confirme que hay un dominio en el grupo de recursos. Seleccione el dominio. Confirme que hay temas del dominio en el dominio. 

## <a name="discard-or-clean-up"></a>Descarte o limpieza
Para completar el traslado, elimine el dominio de la región de origen.  

Si desea volver a empezar, elimine el dominio en la región de destino y repita los pasos de las secciones [Preparar](#prepare) y [Volver a crear](#recreate) de este artículo.

Para eliminar un dominio mediante Azure Portal, siga estos pasos:

1. En la ventana de búsqueda de la parte superior de Azure Portal, escriba **Dominios de Event Hubs** y seleccione **Dominios de Event Hubs** en los resultados de la búsqueda. 
2. Seleccione el dominio que desea eliminar y, a continuación, seleccione **Eliminar** en la barra de herramientas. 
3. En la página de confirmación, escriba el nombre del grupo de recursos y seleccione **Eliminar**.  

Para eliminar el grupo de recursos que contiene el dominio mediante Azure Portal, siga estos pasos:

1. En la ventana de búsqueda de la parte superior de Azure Portal, escriba **Grupos de recursos** y seleccione **Grupos de recursos** de los resultados de la búsqueda. 
2. Seleccione el grupo de recursos que desea eliminar y, a continuación, seleccione **Eliminar** en la barra de herramientas. 
3. En la página de confirmación, escriba el nombre del grupo de recursos y seleccione **Eliminar**.  

## <a name="next-steps"></a>Pasos siguientes
Ha aprendido a trasladar un dominio de Event Grid de una región a otra. Consulte los artículos siguientes para trasladar temas del sistema, temas personalizados, dominios y espacios de nombres del asociado entre regiones.

- [Traslado de temas del sistema entre regiones](move-system-topics-across-regions.md) 
- [Traslado de temas personalizados entre regiones](move-custom-topics-across-regions.md) 
- [Traslado de espacios de nombres del asociado entre regiones](move-partner-namespaces-across-regions.md)

Para obtener más información sobre el traslado de recursos entre regiones y la recuperación ante desastres en Azure, consulte el siguiente artículo: [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../azure-resource-manager/management/move-resource-group-and-subscription.md)
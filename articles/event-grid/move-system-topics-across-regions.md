---
title: Traslado de temas del sistema de Azure Event Grid a otra región
description: En este artículo se muestra cómo trasladar temas del sistema de Azure Event Grid de una región a otra.
ms.topic: how-to
ms.custom: subject-moving-resources
ms.date: 08/28/2020
ms.openlocfilehash: eb6029b206e7d47789371ee81e75c4e05c69ee65
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89082637"
---
# <a name="move-azure-event-grid-system-topics-to-another-region"></a>Traslado de temas del sistema de Azure Event Grid a otra región
Puede mover sus recursos a otra región por varios motivos. Por ejemplo, para aprovechar una nueva región de Azure, para cumplir los requisitos de gobernanza y las directivas internas o como respuesta a los requisitos de planeamiento de capacidad. 

Estos son los pasos resumidos que se describen en este artículo: 

- **Exporte el grupo de recursos** que contiene la cuenta de Azure Storage y su tema del sistema asociado a una plantilla de Resource Manager. También puede exportar una plantilla solo para el tema del sistema. Si va a hacer esto último, no olvide mover el origen de eventos de Azure (en este ejemplo, una cuenta de Azure Storage) a la otra región antes de mover el tema del sistema. A continuación, en la plantilla exportada para el tema del sistema, actualice el identificador externo de la cuenta de almacenamiento en la región de destino. 
- **Modifique la plantilla** para agregar la propiedad `endpointUrl` con el fin de que apunte a un webhook que se suscribe al tema del sistema. Cuando se exporta el tema del sistema, su suscripción (en este caso, un webhook) también se exporta a la plantilla, pero no se incluye la propiedad `endpointUrl`. Por lo tanto, debe actualizarla para que apunte al punto de conexión que se suscribe al tema. Además, actualice el valor de la propiedad `location` a la nueva ubicación o región. En otros tipos de controladores de eventos, solo necesita actualizar la ubicación. 
- **Use la plantilla para implementar recursos** en la región de destino. Especificará los nombres de la cuenta de almacenamiento y el tema del sistema que se va a crear en la región de destino. 
- **Comprobar la implementación**. Compruebe que el webhook se invoca cuando carga un archivo en el almacenamiento de blobs de la región de destino. 
- Para **completar el traslado**, elimine los recursos (el origen del evento y el tema del sistema) de la región de origen. 

## <a name="prerequisites"></a>Requisitos previos
- Complete la guía [Inicio rápido: Enrutamiento de eventos de Blob Storage a un punto de conexión web personalizado con Azure Portal](blob-event-quickstart-portal.md) en la región de origen. Este paso es **opcional**. Sígalo para probar los pasos de este artículo. Mantenga la cuenta de almacenamiento en un grupo de recursos independiente del plan de App Service y App Service. 
- Asegúrese de que el servicio Event Grid esté disponible en la región de destino. Consulte los [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=event-grid&regions=all).

## <a name="prepare"></a>Preparación
Para empezar, exporte una plantilla de Resource Manager para el grupo de recursos que contiene el origen de eventos del sistema (la cuenta de Azure Storage) y su tema del sistema asociado. 

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione **Grupos de recursos** en el menú de la izquierda. A continuación, seleccione el grupo de recursos que contiene el origen del evento para el que se creó el tema del sistema. En el ejemplo siguiente, es la cuenta de **Azure Storage**. El grupo de recursos contiene la cuenta de almacenamiento y su tema del sistema asociado. 

    :::image type="content" source="./media/move-system-topics-across-regions/resource-group-page.png" alt-text="Página Grupo de recursos":::        
3. En el menú de la izquierda, seleccione **Exportar plantilla** en **Configuración** y, a continuación, seleccione **Descargar** en la barra de herramientas. 

    :::image type="content" source="./media/move-system-topics-across-regions/export-template-menu.png" alt-text="Cuenta de almacenamiento: página Exportar plantilla":::        
5. Busque el archivo **.zip** que descargó desde el portal y descomprímalo en la carpeta que prefiera. Este archivo .zip contiene los archivos JSON de plantilla y parámetros. 
1. Abra **template.json** en el editor que prefiera. 
1. La dirección URL del webhook no se exporta a la plantilla. Por tanto, siga estos pasos:
    1. En el archivo de plantilla, busque **Webhook**. 
    1. En la sección **Propiedades**, agregue un carácter de coma (`,`) al final de la última línea. En este ejemplo, es `"preferredBatchSizeInKilobytes": 64`. 
    1. Agregue la propiedad `endpointUrl` con el valor establecido en la dirección URL del webhook, tal como se muestra en el ejemplo siguiente. 

        ```json
        "destination": {
            "properties": {
                "maxEventsPerBatch": 1,
                "preferredBatchSizeInKilobytes": 64,
                "endpointUrl": "https://mysite.azurewebsites.net/api/updates"
            },
            "endpointType": "WebHook"
        }
        ```

        > [!NOTE]
        > Para otros tipos de controladores de eventos, todas las propiedades se exportan a la plantilla. Solo tiene que actualizar la propiedad `location` en la región de destino, tal como se muestra en el paso siguiente. 
7. Actualice `location` para el recurso de **cuenta de almacenamiento** en la región o ubicación de destino. Para obtener los códigos de ubicación, consulte [Ubicaciones de Azure](https://azure.microsoft.com/global-infrastructure/locations/). El código de una región es el nombre de la región sin espacios, por ejemplo, `West US` es igual a `westus`.

    ```json
    "type": "Microsoft.Storage/storageAccounts",
    "apiVersion": "2019-06-01",
    "name": "[parameters('storageAccounts_spegridstorage080420_name')]",
    "location": "westus",
    ```
8. Repita el paso con el fin de actualizar `location` para el recurso de **tema del sistema** en la plantilla. 

    ```json
    "type": "Microsoft.EventGrid/systemTopics",
    "apiVersion": "2020-04-01-preview",
    "name": "[parameters('systemTopics_spegridsystopic080420_name')]",
    "location": "westus",
    ```
1. **Guarde** la plantilla. 

## <a name="recreate"></a>Volver a crear 
Implemente la plantilla para crear una cuenta de almacenamiento y un tema del sistema para la cuenta de almacenamiento en la región de destino. 

1. En Azure Portal, haga clic en **Crear un recurso**.
2. En **Buscar en Marketplace**, escriba **implementación de plantillas** y, después, presione **ENTRAR**.
3. Seleccione **Implementación de plantillas**.
4. Seleccione **Crear**.
5. Seleccione **Cree su propia plantilla en el editor**.
6. Seleccione **Cargar archivo** y, después, siga las instrucciones para cargar el archivo **template.json** que descargó en la última sección.
7. Seleccione **Guardar** para guardar la plantilla. 
8. En la página **Implementación personalizada**, siga estos pasos. 
    1. Seleccione una **suscripción** de Azure. 
    1. Seleccione un **grupo de recursos** en la región de destino o cree uno. 
    1. En **Región**, seleccione la región de destino. Si ha seleccionado un grupo de recursos existente, esta configuración es de solo lectura.
    1. En **Nombre del tema del sistema**, escriba un nombre para el tema del sistema que se asociará a la cuenta de almacenamiento.  
    1. En **Nombre de la cuenta de almacenamiento**, escriba un nombre para la cuenta de almacenamiento que se va a crear en la región de destino. 

        :::image type="content" source="./media/move-system-topics-across-regions/deploy-template.png" alt-text="Implementación de la plantilla de Resource Manager":::
    5. En la parte inferior de la página, seleccione **Revisar y crear**. 
    1. En la página **Revisar y crear**, examine la configuración y seleccione **Crear**. 

## <a name="verify"></a>Comprobar
1. Una vez que la implementación se haya realizado correctamente, seleccione **Ir al grupo de recursos**. 
1. En la página **Grupo de recursos**, compruebe que se han creado el origen del evento (en este ejemplo, la cuenta de Azure Storage) y el tema del sistema. 
1. Cargue un archivo en un contenedor de Azure Blob Storage y compruebe que el webhook ha recibido el evento. Para obtener más información, consulte [Envío de un evento al punto de conexión](blob-event-quickstart-portal.md#send-an-event-to-your-endpoint).

## <a name="discard-or-clean-up"></a>Descarte o limpieza
Para completar el traslado, elimine el grupo de recursos que contiene la cuenta de almacenamiento y su tema del sistema asociado en la región de origen.  

Si desea volver a empezar, elimine el grupo de recursos de la región de destino y repita los pasos de las secciones [Preparar](#prepare) y [Volver a crear](#recreate) de este artículo.

Para eliminar un grupo de recursos (origen o destino) mediante Azure Portal:

1. En la ventana de búsqueda de la parte superior de Azure Portal, escriba **Grupos de recursos** y seleccione **Grupos de recursos** en los resultados de la búsqueda. 
2. Seleccione el grupo de recursos que desea eliminar y, a continuación, seleccione **Eliminar** en la barra de herramientas. 

    :::image type="content" source="./media/move-system-topics-across-regions/delete-resource-group-button.png" alt-text="Eliminación de un grupo de recursos":::
3. En la página de confirmación, escriba el nombre del grupo de recursos y seleccione **Eliminar**.  

## <a name="next-steps"></a>Pasos siguientes
Ha aprendido a trasladar un origen de eventos de Azure y su tema del sistema asociado de una región a otra. Consulte los artículos siguientes para trasladar temas personalizados, dominios y espacios de nombres del asociado entre regiones.

- [Traslado de temas personalizados entre regiones](move-custom-topics-across-regions.md) 
- [Traslado de dominios entre regiones](move-domains-across-regions.md) 
- [Traslado de espacios de nombres del asociado entre regiones](move-partner-namespaces-across-regions.md) 

Para obtener más información sobre el traslado de recursos entre regiones y la recuperación ante desastres en Azure, consulte el siguiente artículo: [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../azure-resource-manager/management/move-resource-group-and-subscription.md)

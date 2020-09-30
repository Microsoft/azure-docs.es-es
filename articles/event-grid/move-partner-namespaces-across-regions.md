---
title: Traslado de espacios de nombres del asociado de Azure Event Grid a otra región
description: En este artículo se muestra cómo trasladar espacios de nombres del asociado de Azure Event Grid de una región a otra.
ms.topic: how-to
ms.custom: subject-moving-resources
ms.date: 08/20/2020
ms.openlocfilehash: 6783db6b9bb1c7d48b308234a179925d6f30e281
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89082642"
---
# <a name="move-azure-event-grid-partner-namespaces-to-another-region"></a>Traslado de espacios de nombres del asociado de Azure Event Grid a otra región
Puede mover sus recursos a otra región por varios motivos. Por ejemplo, para aprovechar una nueva región de Azure, para cumplir los requisitos de gobernanza y las directivas internas o como respuesta a los requisitos de planeamiento de capacidad. 

Estos son los pasos resumidos que se describen en este artículo: 

- **Exporte el recurso de espacio de nombres del asociado** a una plantilla de Azure Resource Manager. Elimine las definiciones de los recursos del canal de eventos en la plantilla. Un canal de eventos puede tener una referencia al identificador de Azure Resource Manager del tema del asociado, que es propiedad de un cliente. Por lo tanto, no se pueden crear usando la plantilla en la región de destino.  
- **Use la plantilla para implementar el espacio de nombres del asociado** en la región de destino. A continuación, cree canales de eventos en el espacio de nombres del asociado en la región de destino. 
- Para **completar el traslado**, elimine el espacio de nombres del asociado de la región de origen. 

    > [!NOTE]
    > - No se pueden exportar **temas del asociado** a una plantilla de Azure Resource Manager porque los clientes no pueden crear directamente un tema de asociado. 
    > - Los **registros de asociados** son recursos globales (no están asociados a ninguna región específica), por lo que no se pueden mover de una región a otra. 

## <a name="prerequisites"></a>Requisitos previos
- Asegúrese de que el servicio Event Grid esté disponible en la región de destino. Consulte los [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=event-grid&regions=all).

## <a name="prepare"></a>Preparación
Para empezar, exporte una plantilla de Resource Manager para el espacio de nombres del asociado. 

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. En la barra de búsqueda de la parte superior, escriba **Espacio de nombres del asociado de Event Grid** y seleccione **Espacio de nombres del asociado de Event Grid** en la lista de resultados. 
3. Seleccione el **espacio de nombres del asociado** que desea exportar a una plantilla de Resource Manager. 
4. En la página **Espacio de nombres del asociado de Event Grid**, seleccione **Exportar plantilla** en **Configuración** en el menú de la izquierda y, a continuación, seleccione **Descargar** en la barra de herramientas. 

    :::image type="content" source="./media/move-partner-namespaces-across-regions/download-template.png" alt-text="Exportar plantilla -> Descargar" lightbox="./media/move-partner-namespaces-across-regions/download-template.png":::   
5. Busque el archivo **.zip** que descargó desde el portal y descomprímalo en la carpeta que prefiera. Este archivo .zip contiene los archivos JSON de plantilla y parámetros. 
1. Abra **template.json** en el editor que prefiera. 
8. Actualice `location` para el recurso de **tema** en la región o ubicación de destino. Para obtener los códigos de ubicación, consulte [Ubicaciones de Azure](https://azure.microsoft.com/global-infrastructure/locations/). El código de una región es el nombre de la región sin espacios, por ejemplo, `West US` es igual a `westus`.

    ```json
    {
        "type": "Microsoft.EventGrid/partnerNamespaces",
        "apiVersion": "2020-04-01-preview",
        "name": "[parameters('partnerNamespace_name')]",
        "location": "westus",
        "properties": {
            "partnerRegistrationFullyQualifiedId": "[parameters('partnerRegistrations_ContosoCorpAccount1_externalid')]"
        }
    }
    ``` 
1. **Guarde** la plantilla. 

## <a name="recreate"></a>Volver a crear 
Implemente la plantilla para crear un espacio de nombres del asociado en la región de destino. 

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
    1. En **Ubicación**, seleccione la región de destino. Si ha seleccionado un grupo de recursos existente, esta configuración es de solo lectura. 
    1. En el **nombre del espacio de nombres del asociado**, escriba un nombre para el nuevo espacio de nombres del asociado. 
    1. En el caso del identificador externo del registro del asociado, escriba el identificador de recurso del registro del asociado con el siguiente formato: `/subscriptions/<Azure subscription ID>/resourceGroups/<resource group name>/providers/Microsoft.EventGrid/partnerRegistrations/<Partner registration name>`.
    1. Seleccione la casilla **Acepto los términos y condiciones indicados anteriormente**.     
    1. Haga clic en **Revisar y crear** para iniciar el proceso de implementación. 
    1. En la página **Revisar y crear**, examine la configuración y seleccione **Crear**. 

## <a name="discard-or-clean-up"></a>Descarte o limpieza
Para completar el traslado, elimine el espacio de nombres del asociado en la región de origen.  

Si desea volver a empezar, elimine el espacio de nombres del asociado en la región de destino y repita los pasos de las secciones [Preparar](#prepare) y [Volver a crear](#recreate) de este artículo.

Para eliminar un espacio de nombres del asociado mediante Azure Portal, siga estos pasos:

1. En la ventana de búsqueda de la parte superior de Azure Portal, escriba **Espacio de nombres del asociado de Event Grid** y seleccione **Espacio de nombres del asociado de Event Grid** en los resultados de la búsqueda. 
2. Seleccione el espacio de nombres del asociado que desea eliminar y, a continuación, seleccione **Eliminar** en la barra de herramientas. 
3. **Confirme** la eliminación para eliminar el espacio de nombres del asociado. 

## <a name="next-steps"></a>Pasos siguientes
Ha aprendido a trasladar un espacio de nombres del asociado de Event Grid de una región a otra. Consulte los artículos siguientes para trasladar temas del sistema, temas personalizados y dominios entre regiones.

- [Traslado de temas del sistema entre regiones](move-system-topics-across-regions.md) 
- [Traslado de temas personalizados entre regiones](move-custom-topics-across-regions.md) 
- [Traslado de dominios entre regiones](move-domains-across-regions.md)

Para obtener más información sobre el traslado de recursos entre regiones y la recuperación ante desastres en Azure, consulte el siguiente artículo: [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../azure-resource-manager/management/move-resource-group-and-subscription.md)
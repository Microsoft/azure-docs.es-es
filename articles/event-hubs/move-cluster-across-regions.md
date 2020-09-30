---
title: Traslado de un clúster dedicado de Azure Event Hubs a otra región | Microsoft Docs
description: En este artículo se muestra cómo trasladar un clúster dedicado de Azure Event Hubs desde la región actual a otra región.
ms.topic: how-to
ms.date: 09/01/2020
ms.openlocfilehash: 94be44ee8f9442a3a65e899d7a58524b2570f194
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89380488"
---
# <a name="move-an-azure-event-hubs-dedicated-cluster-to-another-region"></a>Traslado de un clúster dedicado de Azure Event Hubs a otra región
En este artículo se muestra cómo exportar una plantilla de Azure Resource Manager para un clúster dedicado de Event Hubs existente y, a continuación, usar la plantilla para crear un clúster con la misma configuración en otra región. 

Si tiene otros recursos como espacios de nombres y centros de eventos en el grupo de recursos de Azure que contiene el clúster de Event Hubs, puede exportar la plantilla en el nivel del grupo de recursos para que todos los recursos relacionados se puedan trasladar a la nueva región en un solo paso. Los pasos que aparecen en este artículo le indican cómo exportar un **clúster de Event Hubs** a la plantilla. Los pasos para exportar un **grupo de recursos** a la plantilla son parecidos. 

## <a name="prerequisites"></a>Requisitos previos
Asegúrese de que se puede crear el clúster dedicado en la región de destino. La forma más sencilla de averiguarlo es usar Azure Portal para intentar [crear un clúster dedicado de Event Hubs](event-hubs-dedicated-cluster-create-portal.md). Puede ver la lista de regiones que se admiten en ese momento para crear el clúster. 

## <a name="prepare"></a>Preparación
Para empezar, exporte una plantilla de Resource Manager. Esta plantilla contiene la configuración que describe el clúster dedicado de Event Hubs.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **Todos los recursos** y, después, seleccione su clúster dedicado de Event Hubs.
3. Seleccione > **Configuración** > **Exportar plantilla**.
4. Elija **Descargar** en la página **Exportar plantilla**.

    :::image type="content" source="./media/move-cluster-across-regions/download-template.png" alt-text="Descarga de la plantilla de Resource Manager" lightbox="./media/move-cluster-across-regions/download-template.png":::
5. Busque el archivo ZIP que descargó desde el portal y descomprímalo en la carpeta que prefiera.

   Este archivo ZIP contiene los archivos .json que incluyen la plantilla y los scripts para implementar la plantilla.


## <a name="move"></a>Move

Implemente la plantilla para crear un clúster dedicado de Event Hubs en la región de destino. 


1. En Azure Portal, haga clic en **Crear un recurso**.
2. En **Buscar en Marketplace**, escriba **implementación de plantillas** y seleccione **Template Deployment (implementar mediante plantillas personalizadas)** .
5. Seleccione **Cree su propia plantilla en el editor**.
6. Seleccione **Cargar archivo** y, después, siga las instrucciones para cargar el archivo **template.json** que descargó en la última sección.
1. Actualice el valor de la propiedad `location` para que apunte a la nueva región. Para obtener los códigos de ubicación, consulte [Ubicaciones de Azure](https://azure.microsoft.com/global-infrastructure/locations/). El código de una región es el nombre de la región sin espacios, por ejemplo, `West US` es igual a `westus`.
1. Seleccione **Guardar** para guardar la plantilla. 
1. En la página **Implementación personalizada**, siga estos pasos: 
    1. Seleccione una **suscripción** de Azure. 
    2. Seleccione un **grupo de recursos** existente o cree uno. 
    3. Seleccione la **ubicación** o región de destino. Si ha seleccionado un grupo de recursos existente, esta configuración es de solo lectura. 
    4. En la sección **CONFIGURACIÓN**, lleve a cabo estos pasos:    
        1. Escriba el nuevo **nombre del clúster**. 

            :::image type="content" source="./media/move-cluster-across-regions/deploy-template.png" alt-text="Implementación de la plantilla de Resource Manager":::
    5. En la parte inferior de la página, seleccione **Revisar y crear**. 
    1. En la página **Revisar y crear**, examine la configuración y, después, seleccione **Crear**.  

## <a name="discard-or-clean-up"></a>Descarte o limpieza
Después de la implementación, si quiere empezar de nuevo, puede eliminar el **clúster dedicado de Event Hubs de destino** y repetir los pasos descritos en las secciones [Preparación](#prepare) y [Traslado](#move) de este artículo.

Para confirmar los cambios y completar el traslado de un clúster de Event Hubs, elimine el **clúster de Event Hubs** de la región original. 

Para eliminar un clúster de Event Hubs (de origen o destino) mediante Azure Portal:

1. En la ventana de búsqueda de la parte superior de Azure Portal, escriba **Clústeres de Event Hubs** y seleccione **Clústeres de Event Hubs** en los resultados de la búsqueda. Podrá ver el clúster de Event Hubs en una lista.
2. Seleccione el clúster que desea eliminar y, a continuación, seleccione **Eliminar** en la barra de herramientas. 
3. En la página **Eliminar clúster**, para confirmar la eliminación, escriba el **nombre del clúster** y, a continuación, seleccione **Eliminar**. 

## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha aprendido a trasladar un clúster dedicado de Event Hubs de una región a otra. 

Consulte el artículo [Traslado de espacios de nombres de Event Hubs entre regiones](move-across-regions.md) para obtener instrucciones sobre cómo trasladar un espacio de nombres de una región a otra. 

Para obtener más información sobre cómo trasladar recursos entre regiones y la recuperación ante desastres en Azure, consulte:

- [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../azure-resource-manager/management/move-resource-group-and-subscription.md)
- [Traslado de máquinas virtuales de Azure a otra región](../site-recovery/azure-to-azure-tutorial-migrate.md)

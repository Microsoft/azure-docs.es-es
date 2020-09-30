---
title: Traslado de un espacio de nombres de Azure Event Hubs a otra región | Microsoft Docs
description: En este artículo se muestra cómo trasladar un espacio de nombres de Azure Event Hubs desde la región actual a otra región.
ms.topic: how-to
ms.date: 09/01/2020
ms.openlocfilehash: b177c3916919e3d97325f9d8c6b6027c00cb476f
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89375200"
---
# <a name="move-an-azure-event-hubs-namespace-to-another-region"></a>Traslado de un espacio de nombres de Azure Event Hubs a otra región
En este artículo se muestra cómo exportar una plantilla de Azure Resource Manager para un espacio de nombres de Event Hubs existente y, a continuación, usar la plantilla para crear un espacio de nombres con la misma configuración en otra región. Sin embargo, este proceso no traslada los eventos que todavía no se han procesado. Debe procesar los eventos del espacio de nombres original antes de eliminarlo.
 
Si tiene otros recursos en el grupo de recursos de Azure que contiene el espacio de nombres de Event Hubs, puede exportar la plantilla en el nivel del grupo de recursos para que todos los recursos relacionados se puedan trasladar a la nueva región en un solo paso. Los pasos que aparecen en este artículo le indican cómo exportar un **espacio de nombres** a la plantilla. Los pasos para exportar un **grupo de recursos** a la plantilla son parecidos. 

## <a name="prerequisites"></a>Prerrequisitos

- Asegúrese de que los servicios y las características que usa su cuenta se admitan en la región de destino.
- Si tiene **Característica de captura** habilitada para Event Hubs en el espacio de nombres, mueva las cuentas de [Azure Storage o Azure Data Lake Store Gen 2](../storage/common/storage-account-move.md) o [Azure Data Lake Store Gen 1](../data-lake-store/data-lake-store-migration-cross-region.md) antes de mover el espacio de nombres Event Hubs. También puede trasladar el grupo de recursos que contiene los espacios de nombres de Storage y de Event Hubs a la otra región siguiendo unos pasos similares a los que se describen en este artículo. 
- Si el espacio de nombres de Event Hubs está en un **clúster de Event Hubs**, [mueva el clúster dedicado](move-cluster-across-regions.md) a la **región de destino** antes de seguir los pasos de este artículo. También puede usar la [plantilla de inicio rápido en GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/201-eventhubs-create-cluster-namespace-eventhub/) para crear un clúster de Event Hubs. En la plantilla, quite la parte del espacio de nombres de JSON para crear solo el clúster. 

## <a name="prepare"></a>Preparación
Para empezar, exporte una plantilla de Resource Manager. Esta plantilla contiene la configuración que describe el espacio de nombres de Event Hubs.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **Todos los recursos** y, después, seleccione su espacio de nombres de Event Hubs.
3. Seleccione > **Configuración** > **Exportar plantilla**.
4. Elija **Descargar** en la página **Exportar plantilla**.

    ![Descarga de la plantilla de Resource Manager](./media/move-across-regions/download-template.png)
5. Busque el archivo ZIP que descargó desde el portal y descomprímalo en la carpeta que prefiera.

   Este archivo ZIP contiene los archivos .json que incluyen la plantilla y los scripts para implementar la plantilla.


## <a name="move"></a>Move

Implemente la plantilla para crear un espacio de nombres de Event Hubs en la región de destino. 


1. En Azure Portal, haga clic en **Crear un recurso**.
2. En **Buscar en Marketplace**, escriba **implementación de plantillas** y seleccione **Template Deployment (implementar mediante plantillas personalizadas)** .
5. Seleccione **Cree su propia plantilla en el editor**.
6. Seleccione **Cargar archivo** y, después, siga las instrucciones para cargar el archivo **template.json** que descargó en la última sección.
1. Actualice el valor de la propiedad `location` para que apunte a la nueva región. Para obtener los códigos de ubicación, consulte [Ubicaciones de Azure](https://azure.microsoft.com/global-infrastructure/locations/). El código de una región es el nombre de la región sin espacios, por ejemplo, `West US` es igual a `westus`.
1. Seleccione **Guardar** para guardar la plantilla. 
1. En la página **Implementación personalizada**, siga estos pasos: 
    1. Seleccione una **suscripción** de Azure. 
    2. Seleccione un **grupo de recursos** existente o cree uno. Si el espacio de nombres de origen se encontraba en un clúster de Event Hubs, seleccione el grupo de recursos que contiene el clúster en la región de destino. 
    3. Seleccione la **ubicación** o región de destino. Si ha seleccionado un grupo de recursos existente, esta configuración es de solo lectura. 
    4. En la sección **CONFIGURACIÓN**, lleve a cabo estos pasos:    
        1. Escriba el **nombre del nuevo espacio de nombres**. 

            ![Implementación de la plantilla de Resource Manager](./media/move-across-regions/deploy-template.png)
        2. Si el espacio de nombres de origen se encontraba en un **clúster de Event Hubs**, escriba los nombres del **grupo de recursos** y del **clúster de Event Hubs** como parte del **identificador externo**. 

              ```
              /subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<CLUSTER'S RESOURCE GROUP>/providers/Microsoft.EventHub/clusters/<CLUSTER NAME>
              ```   
        3. Si el centro de eventos del espacio de nombres usa una cuenta de Storage para capturar eventos, especifique el nombre del grupo de recursos y la cuenta de almacenamiento para el campo `StorageAccounts_<original storage account name>_external`. 
            
            ```
            /subscriptions/0000000000-0000-0000-0000-0000000000000/resourceGroups/<STORAGE'S RESOURCE GROUP>/providers/Microsoft.Storage/storageAccounts/<STORAGE ACCOUNT NAME>
            ```    
    5. En la parte inferior de la página, seleccione **Revisar y crear**. 
    1. En la página **Revisar y crear**, examine la configuración y, después, seleccione **Crear**.   

## <a name="discard-or-clean-up"></a>Descarte o limpieza
Después de la implementación, si quiere empezar de nuevo, puede eliminar el **espacio de nombres de Event Hubs de destino** y repetir los pasos descritos en las secciones [Preparación](#prepare) y [Traslado](#move) de este artículo.

Para confirmar los cambios y completar el traslado de un espacio de nombres de Event Hubs, elimine el **espacio de nombres de Event Hubs** de la región original. Asegúrese de que ha procesado todos los eventos en el espacio de nombres antes de eliminarlo. 

Para eliminar un espacio de nombres de Event Hubs (de origen o destino) mediante Azure Portal:

1. En la ventana de búsqueda de la parte superior de Azure Portal, escriba **Event Hubs** y seleccione **Event Hubs** en los resultados de la búsqueda. Verá los espacios de nombres de Event Hubs en una lista.
2. Seleccione el espacio de nombres de destino que desea eliminar y, a continuación, seleccione **Eliminar** en la barra de herramientas. 

    ![Eliminar espacio de nombres - botón](./media/move-across-regions/delete-namespace-button.png)
3. En la página **Eliminar espacio de nombres**, para confirmar la eliminación, escriba el **nombre del espacio de nombres** y, a continuación, seleccione **Eliminar**. 

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha trasladado un espacio de nombres de Azure Event Hubs de una región a otra y ha limpiado los recursos de origen.  Para obtener más información sobre cómo trasladar recursos entre regiones y la recuperación ante desastres en Azure, consulte:


- [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../azure-resource-manager/management/move-resource-group-and-subscription.md)
- [Traslado de máquinas virtuales de Azure a otra región](../site-recovery/azure-to-azure-tutorial-migrate.md)

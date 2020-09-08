---
title: Procedimiento para trasladar el recurso de servicio entre regiones
titleSuffix: Azure Cognitive Search
description: En este artículo se mostrará cómo trasladar los recursos de Azure Cognitive Search de una región a otra en la nube de Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: how-to
ms.custom: subject-moving-resources
ms.date: 03/24/2020
ms.openlocfilehash: 71846b8e26efb3853705fabff78831e746727191
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88926956"
---
# <a name="move-your-azure-cognitive-search-service-to-another-azure-region"></a>Traslado del servicio Azure Cognitive Search a otra región de Azure

En ocasiones, los clientes preguntan sobre el traslado de un servicio de búsqueda a otra región. Actualmente, no hay ningún mecanismo ni herramienta integrado que facilite esta tarea, pero este artículo puede ayudarle a comprender los pasos manuales para lograr el mismo resultado.

> [!NOTE]
> En Azure Portal, todos los servicios tienen un comando **Export template**. En el caso de Azure Cognitive Search, este comando genera una definición básica de un servicio (nombre, ubicación, nivel, réplica y recuento de particiones), pero no reconoce el contenido del servicio, ni tampoco las claves, roles o registros. Aunque el comando existe, no se recomienda utilizarlo para trasladar un servicio de búsqueda.

## <a name="guidance-for-moving-a-service"></a>Instrucciones para trasladar un servicio

1. Identifique las dependencias y los servicios relacionados para comprender el impacto total de la reubicación de un servicio, en caso de que necesite trasladar algo más que solo Azure Cognitive Search.

   Azure Storage se usa para el registro y la creación de un almacén de conocimiento y, además, es un origen de datos externo que se usa habitualmente para el enriquecimiento con IA y la indexación. Cognitive Services es una dependencia del enriquecimiento con IA. Es necesario que Cognitive Services y el servicio de búsqueda estén en la misma región si va a utilizar el enriquecimiento con IA.

1. Cree un inventario de todos los objetos del servicio para que sepa qué debe mover: índices, mapas de sinónimos, indexadores, orígenes de datos, conjuntos de aptitudes. Si ha habilitado el registro, cree y archive los informes que pueda necesitar para llevar un registro histórico.

1. Compruebe los precios y la disponibilidad en la nueva región para garantizar la disponibilidad de Azure Cognitive Search, así como la de cualquier servicio de la nueva región. La mayoría de las características están disponibles en todas las regiones, pero algunas características de la versión preliminar tienen disponibilidad restringida.

1. Cree un servicio en la nueva región y vuelva a publicar a partir del código fuente todos los índices, mapas de sinónimos, indexadores, orígenes de datos y conjuntos de aptitudes existentes. Recuerde que los nombres de servicios deben ser exclusivos, de modo que no puede volver a utilizar un nombre ya existente. Compruebe cada conjunto de aptitudes para ver si las conexiones a Cognitive Services siguen siendo válidas en términos del requisito de la misma región. Además, si se crean almacenes de conocimiento, compruebe las cadenas de conexión de Azure Storage si usa otro servicio.

1. Recargue los índices y almacenes de conocimiento si procede. Tendrá que usar código de la aplicación para insertar datos JSON en un índice o volver a ejecutar los indexadores para extraer los documentos a partir de fuentes externas. 

1. Habilite los registros, y si va a usarlos, vuelva a crear los roles de seguridad.

1. Actualice las aplicaciones cliente y los conjuntos de pruebas para que usen el nombre y claves de API del nuevo servicio, y pruebe todas las aplicaciones.

1. Elimine el antiguo servicio una vez que haya probado por completo el nuevo y vea que está operativo.

## <a name="next-steps"></a>Pasos siguientes

Los vínculos siguientes pueden ayudarle a encontrar más información al completar los pasos descritos anteriormente.

+ [Precios y regiones de Azure Cognitive Search](https://azure.microsoft.com/pricing/details/search/)
+ [Elección de un nivel](search-sku-tier.md)
+ [Creación de una instancia de Search Service](search-create-service-portal.md)
+ [Carga de documentos de búsqueda](search-what-is-data-import.md)
+ [Habilitación del registro](search-monitor-logs.md)


<!-- To move your Azure Cognitive Service account from one region to another, you will create an export template to move your subscription(s). After moving your subscription, you will need to move your data and recreate your service.

In this article, you'll learn how to:

> [!div class="checklist"]
> * Export a template.
> * Modify the template: adding the target region, search and storage account names.
> * Deploy the template to create the new search and storage accounts.
> * Verify your service status in the new region
> * Clean up resources in the source region.

## Prerequisites

- Ensure that the services and features that your account uses are supported in the target region.

- For preview features, ensure that your subscription is whitelisted for the target region. For more information about preview features, see [knowledge stores](./knowledge-store-concept-intro.md), [incremental enrichment](./cognitive-search-incremental-indexing-conceptual.md), and [private endpoint](./service-create-private-endpoint.md).

## Assessment and planning

When you move your search service to the new region, you will need to [move your data to the new storage service](../storage/common/storage-account-move.md?tabs=azure-portal#configure-the-new-storage-account) and then rebuild your indexes, skillsets and knowledge stores. You should record current settings and copy json files to make the rebuilding of your service easier and faster.

## Moving your search service's resources

To start you will export and then modify a Resource Manager template.

### Export a template

1. Sign in to the [Azure portal](https://portal.azure.com).

2. Go to your Resource Group page.

> [!div class="mx-imgBorder"]
> ![Resource Group page example](./media/search-move-resource/export-template-sample.png)

3. Select **All resources**.

3. In the left hand navigation menu select **Export template**.

4. Choose **Download** in the **Export template** page.

5. Locate the .zip file that you downloaded from the portal, and unzip that file to a folder of your choice.

The zip file contains the .json files that comprise the template and scripts to deploy the template.

### Modify the template

You will modify the template by changing the search and storage account names and regions. The names must follow the rules for each service and region naming conventions. 

To obtain region location codes, see [Azure Locations](https://azure.microsoft.com/global-infrastructure/locations/).  The code for a region is the region name with no spaces, **Central US** = **centralus**.

1. In the Azure portal, select **Create a resource**.

2. In **Search the Marketplace**, type **template deployment**, and then press **ENTER**.

3. Select **Template deployment**.

4. Select **Create**.

5. Select **Build your own template in the editor**.

6. Select **Load file**, and then follow the instructions to load the **template.json** file that you downloaded and unzipped in the previous section.

7. In the **template.json** file, name the target search and storage accounts by setting the default value of the search and storage account names. 

8. Edit the **location** property in the **template.json** file to the target region for both your search and storage services. This example sets the target region to `centralus`.

```json
},
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Search/searchServices",
            "apiVersion": "2020-03-13",
            "name": "[parameters('searchServices_target_region_search_name')]",
            "location": "centralus",
            "sku": {
                "name": "standard"
            },
            "properties": {
                "replicaCount": 1,
                "partitionCount": 1,
                "hostingMode": "Default"
            }
        },
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-06-01",
            "name": "[parameters('storageAccounts_tagetstorageregion_name')]",
            "location": "centralus",
            "sku": {
                "name": "Standard_RAGRS",
                "tier": "Standard"
            },
```

### Deploy the template

1. Save the **template.json** file.

2. Enter or select the property values:

- **Subscription**: Select an Azure subscription.

- **Resource group**: Select **Create new** and give the resource group a name.

- **Location**: Select an Azure location.

3. Click the **I agree to the terms and conditions stated above** checkbox, and then click the **Select Purchase** button.

## Verifying your services' status in new region

To verify the move, open the new resource group and your services will be listed with the new region.

To move your data from your source region to the target region, please see this article's guidelines for [moving your data to the new storage account](../storage/common/storage-account-move.md?tabs=azure-portal#move-data-to-the-new-storage-account).

## Clean up resources in your original region

To commit the changes and complete the move of your service account, delete the source service account.

## Next steps

[Create an index](./search-get-started-portal.md)

[Create a skillset](./cognitive-search-quickstart-blob.md)

[Create a knowledge store](./knowledge-store-create-portal.md) -->
---
title: Protección de un área de trabajo de Azure Machine Learning con redes virtuales
titleSuffix: Azure Machine Learning
description: Use una instancia aislada de Azure Virtual Network para proteger el área de trabajo de Azure Machine Learning y los recursos asociados.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: larryfr
ms.author: aashishb
author: aashishb
ms.date: 07/07/2020
ms.topic: conceptual
ms.custom: how-to, contperfq4, tracking-python
ms.openlocfilehash: 4dc1f86ce7dbb060c747c4433f0c2b871ce5582d
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90907643"
---
# <a name="secure-an-azure-machine-learning-workspace-with-virtual-networks"></a>Protección de un área de trabajo de Azure Machine Learning con redes virtuales

En este artículo, aprenderá a proteger un área de trabajo de Azure Machine Learning y sus recursos asociados en una red virtual.


Este artículo es la segunda parte de una serie de cinco capítulos que le guía a través de la protección de un flujo de trabajo de Azure Machine Learning. Le recomendamos encarecidamente que lea [Parte uno: Introducción a las redes virtuales](how-to-network-security-overview.md) para comprender la arquitectura general en primer lugar. 

Consulte los demás artículos de esta serie:

[1. Introducción a las redes virtuales](how-to-network-security-overview.md) > **2. Protección del área de trabajo**  > [3. Protección del entorno de entrenamiento](how-to-secure-training-vnet.md) > [4. Protección del entorno de inferencia](how-to-secure-inferencing-vnet.md) > [5. Habilitación de la funcionalidad de Studio](how-to-enable-studio-virtual-network.md)

En este artículo aprenderá a habilitar los siguientes recursos de áreas de trabajo en una red virtual:
> [!div class="checklist"]
> - Área de trabajo de Azure Machine Learning
> - Cuentas de Azure Storage
> - Conjuntos de datos y almacenes de datos de Azure Machine Learning
> - Azure Key Vault
> - Azure Container Registry

## <a name="prerequisites"></a>Requisitos previos

+ Lea el artículo [Introducción a la seguridad de red](how-to-network-security-overview.md) para comprender los escenarios comunes de redes virtuales y la arquitectura de red virtual general.

+ Una red virtual y una subred existentes que se usarán con los recursos de proceso.

+ Para implementar recursos en una red virtual o subred, la cuenta de usuario debe tener permisos para realizar las siguientes acciones en los controles de acceso basados en rol (RBAC) de Azure:

    - "Microsoft.Network/virtualNetworks/join/action" en el recurso de red virtual.
    - "Microsoft.Network/virtualNetworks/subnet/join/action" en el recurso de subred.

    Para obtener más información sobre RBAC con redes, vea los [roles integrados de redes](/azure/role-based-access-control/built-in-roles#networking).


## <a name="secure-the-workspace-with-private-endpoint"></a>Protección del área de trabajo con punto de conexión privado

Azure Private Link le permite conectarse a su área de trabajo mediante un punto de conexión privado. El punto de conexión privado es un conjunto de direcciones IP privadas dentro de la red virtual. Después, puede limitar el acceso al área de trabajo para que solo se produzca en las direcciones IP privadas. Private Link ayuda a reducir el riesgo de una filtración de datos.

Para obtener más información sobre la configuración de un área de trabajo de Private Link, consulte [Configuración de Private Link](how-to-configure-private-link.md).


## <a name="secure-azure-storage-accounts"></a>Protección de cuentas de Azure Storage

En esta sección, aprenderá a proteger una cuenta de Azure Storage mediante puntos de conexión de servicio. Sin embargo, también puede usar puntos de conexión privados para proteger Azure Storage. Para obtener más información, consulte [Uso de puntos de conexión privados para Azure Storage](../storage/common/storage-private-endpoints.md).

> [!IMPORTANT]
> Se puede colocar tanto la _cuenta de almacenamiento predeterminada_ para Azure Machine Learning o las _cuentas de almacenamiento no predeterminadas_ en una red virtual.
>
> La cuenta de almacenamiento predeterminada se aprovisiona automáticamente al crear un área de trabajo.
>
> En las cuentas de almacenamiento no predeterminadas, el parámetro `storage_account` de la [función `Workspace.create()`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace(class)?view=azure-ml-py#create-name--auth-none--subscription-id-none--resource-group-none--location-none--create-resource-group-true--sku--basic---friendly-name-none--storage-account-none--key-vault-none--app-insights-none--container-registry-none--cmk-keyvault-none--resource-cmk-uri-none--hbi-workspace-false--default-cpu-compute-target-none--default-gpu-compute-target-none--exist-ok-false--show-output-true-) permite especificar una cuenta de almacenamiento personalizada mediante el identificador de recursos de Azure.

Para usar una cuenta de Azure Storage para el área de trabajo en una red virtual, siga estos pasos:

1. En Azure Portal, vaya al servicio de almacenamiento que quiera usar en el área de trabajo.

   [![Almacenamiento asociado al área de trabajo de Azure Machine Learning](./media/how-to-enable-virtual-network/workspace-storage.png)](./media/how-to-enable-virtual-network/workspace-storage.png#lightbox)

1. En la página de la cuenta de servicio de almacenamiento, seleccione __Firewalls y redes virtuales__.

   ![El área "Firewalls y redes virtuales" de la página de Azure Storage de Azure Portal](./media/how-to-enable-virtual-network/storage-firewalls-and-virtual-networks.png)

1. En la página __Firewalls y redes virtuales__, realice las acciones siguientes:
    1. Seleccione __Redes seleccionadas__.
    1. En __Redes virtuales__, seleccione el vínculo __Agregar red virtual existente__. Esta acción agrega la red virtual en la que reside el proceso (vea el paso 1).

        > [!IMPORTANT]
        > La cuenta de almacenamiento debe estar en la misma red virtual y subred que las instancias de proceso o clústeres usados para entrenamiento o inferencia.

    1. Seleccione la casilla __Permitir que los servicios de Microsoft de confianza accedan a esta cuenta de almacenamiento__.

    > [!IMPORTANT]
    > Al trabajar con el SDK de Azure Machine Learning, el entorno de desarrollo debe poder conectarse a la cuenta de Azure Storage. Si la cuenta de almacenamiento se encuentra dentro de una red virtual, el firewall debe permitir el acceso desde la dirección IP del entorno de desarrollo.
    >
    > Para habilitar el acceso a la cuenta de almacenamiento, visite __Firewalls y redes virtuales__ de la cuenta de almacenamiento *desde un explorador web en el cliente de desarrollo*. A continuación, use la casilla __Agregar la dirección IP del cliente__ para agregar la dirección IP del cliente al campo __INTERVALO DE DIRECCIONES__. También puede usar el campo __INTERVALO DE DIRECCIONES__ para especificar manualmente la dirección IP del entorno de desarrollo. Una vez agregada la dirección IP del cliente, este puede acceder a la cuenta de almacenamiento mediante el SDK.

   [![Panel "Firewalls y redes virtuales" de Azure Portal](./media/how-to-enable-virtual-network/storage-firewalls-and-virtual-networks-page.png)](./media/how-to-enable-virtual-network/storage-firewalls-and-virtual-networks-page.png#lightbox)

## <a name="secure-datastores-and-datasets"></a>Protección de almacenes de datos y conjuntos de datos

En esta sección, aprenderá a usar los almacenes de datos y los conjuntos de datos para la experiencia de SDK en una red virtual. Para obtener más información sobre la experiencia de Studio, consulte [Uso de Azure Machine Learning Studio en una red virtual](how-to-enable-studio-virtual-network.md).

Para obtener acceso a los datos mediante SDK, debe usar el método de autenticación requerido por el servicio individual en el que se almacenan los datos. Por ejemplo, si registra un almacén de datos para tener acceso a Azure Data Lake Store Gen2, debe usar una entidad de servicio como se documenta en [Conexión con los servicios de Azure Storage](how-to-access-data.md#azure-data-lake-storage-generation-2).

### <a name="disable-data-validation"></a>Deshabilitación de la validación de datos

De forma predeterminada, Azure Machine Learning realiza comprobaciones de las credenciales y la validez de los datos al intentar acceder a ellos mediante el SDK. Si los datos están detrás de una red virtual, Azure Machine Learning no puede completar estas comprobaciones. Para evitarlo, debe crear almacenes de datos y conjuntos de datos que omitan la validación.

### <a name="use-datastores"></a>Uso de almacenes de datos

 En Azure Data Lake Store Gen1 y Azure Data Lake Store Gen2 la validación se omite de forma predeterminada, por lo que no es necesario realizar ninguna otra acción. Pero para los servicios siguientes puede usar una sintaxis similar a fin de omitir la validación del almacén de datos:

- Azure Blob Storage
- Recurso compartido de archivos de Azure
- PostgreSQL
- Azure SQL Database

En el ejemplo de código siguiente se crea un almacén de datos de blobs de Azure y se establece `skip_validation=True`.

```python
blob_datastore = Datastore.register_azure_blob_container(workspace=ws,  

                                                         datastore_name=blob_datastore_name,  

                                                         container_name=container_name,  

                                                         account_name=account_name, 

                                                         account_key=account_key, 

                                                         skip_validation=True ) // Set skip_validation to true
```

### <a name="use-datasets"></a>Uso de conjuntos de datos

La sintaxis para omitir la validación de conjuntos de datos es similar para los siguientes tipos de datos:
- Archivo delimitado
- JSON 
- Parquet
- SQL
- Archivo

En el código siguiente se crea un conjunto de datos JSON y se establece `validate=False`.

```python
json_ds = Dataset.Tabular.from_json_lines_files(path=datastore_paths, 

validate=False) 

```

## <a name="secure-azure-key-vault"></a>Protección de Azure Key Vault

Azure Machine Learning usa una instancia de Key Vault asociada para almacenar las siguientes credenciales:
* La cadena de conexión de cuenta de almacenamiento asociada
* Contraseñas para las instancias de Azure Container Repository
* Cadenas de conexión a almacenes de datos

Para usar las funcionalidades de experimentación de Azure Machine Learning con Azure Key Vault detrás de una red virtual, siga los pasos siguientes:

1. Vaya a la instancia de Key Vault asociada al área de trabajo.

1. En la página de __Key Vault__, en el panel izquierdo, seleccione __Redes__.

1. En la pestaña __Firewalls y redes virtuales__, realice las acciones siguientes:
    1. En __Permitir el acceso desde__, seleccione __Punto de conexión privado y redes seleccionadas__.
    1. En __Redes virtuales__, seleccione __Agregar redes virtuales existentes__ para agregar la red virtual donde reside el proceso de experimentación.
    1. En __¿Quiere permitir que los servicios de confianza de Microsoft puedan omitir este firewall?__ , seleccione __Sí__.

   [![Sección "Firewalls y redes virtuales" del panel de Key Vault](./media/how-to-enable-virtual-network/key-vault-firewalls-and-virtual-networks-page.png)](./media/how-to-enable-virtual-network/key-vault-firewalls-and-virtual-networks-page.png#lightbox)

## <a name="enable-azure-container-registry-acr"></a>Habilitación de un registro de Azure Container Registry (ACR)

Para usar Azure Container Registry en una red virtual, debe cumplir los siguientes requisitos:

* La instancia de Azure Container Registry debe tener la versión Prémium. Para más información sobre la actualización, vea [Cambio de SKU](/azure/container-registry/container-registry-skus#changing-skus).

* La instancia de Azure Container Registry debe estar en la misma red virtual y subred que la cuenta de almacenamiento y los destinos de proceso utilizados para entrenamiento o inferencia.

* El área de trabajo de Azure Machine Learning debe contener un [clúster de proceso de Azure Machine Learning](how-to-create-attach-compute-sdk.md#amlcompute).

    Cuando la instancia de ACR está detrás de una red virtual, Azure Machine Learning no puede usarla para compilar imágenes de Docker directamente. En su lugar, se usa el clúster de proceso para compilar las imágenes.

Una vez cumplidos estos requisitos, siga los pasos que se indican a continuación para habilitar Azure Container Registry.

1. Busque el nombre de la instancia de Azure Container Registry para el área de trabajo mediante alguno de los métodos siguientes:

    __Azure Portal__

    En la sección de información general del área de trabajo, el valor __Registro__ se vincula a la instancia de Azure Container Registry.

    :::image type="content" source="./media/how-to-enable-virtual-network/azure-machine-learning-container-registry.png" alt-text="Azure Container Registry para el área de trabajo" border="true":::

    __CLI de Azure__

    Si ha [instalado la extensión de Machine Learning para la CLI de Azure](reference-azure-machine-learning-cli.md), puede usar el comando `az ml workspace show` para mostrar la información del área de trabajo.

    ```azurecli-interactive
    az ml workspace show -w yourworkspacename -g resourcegroupname --query 'containerRegistry'
    ```

    Este comando devuelve un valor similar a `"/subscriptions/{GUID}/resourceGroups/{resourcegroupname}/providers/Microsoft.ContainerRegistry/registries/{ACRname}"`. La última parte de la cadena es el nombre de Azure Container Registry para el área de trabajo.

1. Limite el acceso a la red virtual mediante los pasos descritos en [Configuración del acceso de red al registro](../container-registry/container-registry-vnet.md#configure-network-access-for-registry). Al agregar la red virtual, seleccione la red virtual y la subred para los recursos de Azure Machine Learning.

1. Use el SDK de Python para Azure Machine Learning para configurar un clúster de proceso para compilar imágenes de Docker. El siguiente fragmento de código muestra cómo hacerlo:

    ```python
    from azureml.core import Workspace
    # Load workspace from an existing config file
    ws = Workspace.from_config()
    # Update the workspace to use an existing compute cluster
    ws.update(image_build_compute = 'mycomputecluster')
    ```

    > [!IMPORTANT]
    > La cuenta de almacenamiento, el clúster de proceso y Azure Container Registry deben estar en la misma subred de la red virtual.
    
    Para más información, vea la referencia del método [update()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace.workspace?view=azure-ml-py#update-friendly-name-none--description-none--tags-none--image-build-compute-none--enable-data-actions-none-).

1. Aplique la siguiente plantilla de Azure Resource Manager. Esta plantilla permite que el área de trabajo se comunique con ACR.

    ```json
    {
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "keyVaultArmId": {
        "type": "string"
        },
        "workspaceName": {
        "type": "string"
        },
        "containerRegistryArmId": {
        "type": "string"
        },
        "applicationInsightsArmId": {
        "type": "string"
        },
        "storageAccountArmId": {
        "type": "string"
        },
        "location": {
        "type": "string"
        }
    },
    "resources": [
        {
        "type": "Microsoft.MachineLearningServices/workspaces",
        "apiVersion": "2019-11-01",
        "name": "[parameters('workspaceName')]",
        "location": "[parameters('location')]",
        "identity": {
            "type": "SystemAssigned"
        },
        "sku": {
            "tier": "basic",
            "name": "basic"
        },
        "properties": {
            "sharedPrivateLinkResources":
    [{"Name":"Acr","Properties":{"PrivateLinkResourceId":"[concat(parameters('containerRegistryArmId'), '/privateLinkResources/registry')]","GroupId":"registry","RequestMessage":"Approve","Status":"Pending"}}],
            "keyVault": "[parameters('keyVaultArmId')]",
            "containerRegistry": "[parameters('containerRegistryArmId')]",
            "applicationInsights": "[parameters('applicationInsightsArmId')]",
            "storageAccount": "[parameters('storageAccountArmId')]"
        }
        }
    ]
    }
    ```

## <a name="next-steps"></a>Pasos siguientes

Este artículo es la primera parte de una serie de cuatro capítulos sobre redes virtuales. Vea el resto de los artículos para obtener información sobre cómo proteger una red virtual:

* [Parte 1: Introducción a las redes virtuales](how-to-network-security-overview.md)
* [Parte 3: Protección del entorno de entrenamiento](how-to-secure-training-vnet.md)
* [Parte 4: Protección del entorno de inferencia](how-to-secure-inferencing-vnet.md)
* [Parte 5: Habilitación de la funcionalidad de Studio](how-to-enable-studio-virtual-network.md)
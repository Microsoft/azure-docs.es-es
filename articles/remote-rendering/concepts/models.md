---
title: Modelos
description: Describe qué es un modelo en Azure Remote Rendering.
author: jakrams
ms.author: jakras
ms.date: 02/05/2020
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.openlocfilehash: 9fd18195c0276999f445e0990838e293a5081131
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89021891"
---
# <a name="models"></a>Modelos

Un *modelo*, en Azure Remote Rendering, se refiere a una representación de objeto completa, formada por [entidades](entities.md) y [componentes](components.md). Los modelos son la manera principal de obtener datos personalizados en el servicio Remote Rendering.

## <a name="model-structure"></a>Estructura del modelo

Un modelo tiene exactamente una [entidad](entities.md) como nodo raíz. Por debajo, puede tener una jerarquía arbitraria de entidades secundarias. Al cargar un modelo, se devuelve una referencia a esta entidad raíz.

Cada entidad puede tener [componentes](components.md) adjuntos. En el caso más común, las entidades tienen *MeshComponents*, que hacen referencia a los [recursos de malla](meshes.md).

## <a name="creating-models"></a>Creación de modelos

La creación de modelos para el entorno en tiempo de ejecución se logra mediante la [conversión de modelos de entrada](../how-tos/conversion/model-conversion.md) a partir de formatos de archivo como FBX y GLTF. El proceso de conversión extrae todos los recursos, como texturas, materiales y mallas, y los convierte en formatos para entornos en tiempo de ejecución optimizados. También extrae la información estructural y la convierte en la estructura del grafo de entidad/componente de ARR.

> [!IMPORTANT]
>
> [La conversión de modelos](../how-tos/conversion/model-conversion.md) es la única manera de crear [mallas](meshes.md). Aunque las mallas se pueden compartir entre entidades en tiempo de ejecución, no hay otra manera de obtener una malla en el entorno en tiempo de ejecución, aparte de cargar un modelo.

## <a name="loading-models"></a>Carga de modelos

Una vez que se convierte un modelo, se puede cargar desde Azure Blob Storage en el entorno en tiempo de ejecución.

Hay dos funciones de carga distintas que difieren en la forma en que se trata el recurso en Blob Storage:

* El modelo se puede direccionar desde su URI de SAS. La función de carga pertinente es `LoadModelFromSASAsync` con el parámetro `LoadModelFromSASParams`. Use también esta variante al cargar [modelos integrados](../samples/sample-model.md).
* El modelo se puede direccionar directamente mediante parámetros de Blob Storage, en caso de que [Blob Storage esté vinculado a la cuenta](../how-tos/create-an-account.md#link-storage-accounts). En este caso, la función de carga pertinente es `LoadModelAsync` con el parámetro `LoadModelParams`.

Los fragmentos de código siguientes muestran cómo cargar modelos con ambas funciones. Para cargar un modelo mediante el URI de SAS, use un código como el siguiente:

```csharp
async void LoadModel(AzureSession session, Entity modelParent, string modelUri)
{
    // load a model that will be parented to modelParent
    var modelParams = new LoadModelFromSASParams(modelUri, modelParent);

    var loadOp = session.Actions.LoadModelFromSASAsync(modelParams);

    loadOp.ProgressUpdated += (float progress) =>
    {
        Debug.Log($"Loading: {progress * 100.0f}%");
    };

    await loadOp.AsTask();
}
```

```cpp
ApiHandle<LoadModelAsync> LoadModel(ApiHandle<AzureSession> session, ApiHandle<Entity> modelParent, std::string modelUri)
{
    LoadModelFromSASParams modelParams;
    modelParams.ModelUrl = modelUri;
    modelParams.Parent = modelParent;

    ApiHandle<LoadModelAsync> loadOp = *session->Actions()->LoadModelFromSASAsync(modelParams);

    loadOp->Completed([](const ApiHandle<LoadModelAsync>& async)
    {
        printf("Loading: finished.");
    });
    loadOp->ProgressUpdated([](float progress)
    {
        printf("Loading: %.1f%%", progress*100.f);
    });

    return loadOp;
}
```

Si quiere cargar un modelo directamente mediante sus parámetros de Blob Storage, use un código similar al siguiente fragmento:

```csharp
async void LoadModel(AzureSession session, Entity modelParent, string storageAccount, string containerName, string assetFilePath)
{
    // load a model that will be parented to modelParent
    var modelParams = new LoadModelParams(
        storageAccount, // storage account name + '.blob.core.windows.net', e.g., 'mystorageaccount.blob.core.windows.net'
        containerName,  // name of the container in your storage account, e.g., 'mytestcontainer'
        assetFilePath,  // the file path to the asset within the container, e.g., 'path/to/file/myAsset.arrAsset'
        modelParent
    );

    var loadOp = session.Actions.LoadModelAsync(modelParams);

    // ... (identical to the SAS URI snippet above)
}
```

```cpp
ApiHandle<LoadModelAsync> LoadModel(ApiHandle<AzureSession> session, ApiHandle<Entity> modelParent, std::string storageAccount, std::string containerName, std::string assetFilePath)
{
    LoadModelParams modelParams;
    modelParams.Parent = modelParent;
    modelParams.Blob.StorageAccountName = std::move(storageAccount);
    modelParams.Blob.BlobContainerName = std::move(containerName);
    modelParams.Blob.AssetPath = std::move(assetFilePath);

    ApiHandle<LoadModelAsync> loadOp = *session->Actions()->LoadModelAsync(modelParams);
    // ... (identical to the SAS URI snippet above)
}
```

Después, puede atravesar la jerarquía de entidades y modificar las entidades y los componentes. Al cargar el mismo modelo varias veces, se crean varias instancias, cada una con su propia copia de la estructura de entidad/componente. A pesar de que las mallas, los materiales y las texturas son [recursos compartidos](../concepts/lifetime.md), sus datos no se volverán a cargar. Por lo tanto, la creación de instancias de un modelo más de una vez incurre en una sobrecarga de memoria relativamente escasa.

> [!CAUTION]
> Todas las funciones *asincrónicas* en ARR devuelven objetos de operación asincrónica. Debe almacenar una referencia a esos objetos hasta que finalice la operación. De lo contrario, el recolector de elementos no utilizados de C# podría eliminar la operación con anterioridad y que nunca finalice. En el código de ejemplo anterior, el uso de *await* garantiza que la variable local "loadOp" contenga una referencia hasta que finalice la carga del modelo. Sin embargo, si usara el evento *Completed* en su lugar, debería almacenar la operación asincrónica en una variable miembro.

## <a name="next-steps"></a>Pasos siguientes

* [Entidades](entities.md)
* [Mallas](meshes.md)

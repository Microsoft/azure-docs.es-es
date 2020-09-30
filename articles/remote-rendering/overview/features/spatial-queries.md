---
title: Consultas espaciales
description: Cómo realizar consultas espaciales en una escena
author: jakrams
ms.author: jakras
ms.date: 02/07/2020
ms.topic: article
ms.custom: devx-track-csharp
ms.openlocfilehash: 5a80681a641757d71288dfaee4c779121030a9db
ms.sourcegitcommit: f845ca2f4b626ef9db73b88ca71279ac80538559
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89613603"
---
# <a name="spatial-queries"></a>Consultas espaciales

Las consultas espaciales son operaciones con las que puede preguntar al servicio de representación remoto qué objetos se encuentran en un área. Las consultas espaciales se usan con frecuencia para implementar interacciones, como averiguar a qué objeto apunta un usuario.

Todas las consultas espaciales se evalúan en el servidor. Por lo tanto, son operaciones asincrónicas y los resultados llegarán con un retraso que dependerá de la latencia de la red. Dado que cada consulta espacial genera tráfico de red, procure no realizar demasiadas a la vez.

## <a name="collision-meshes"></a>Mallas de colisión

Las consultas espaciales se basan en el motor de [Havok Physics](https://www.havok.com/products/havok-physics) y requieren a presencia de una malla de colisión dedicada. De forma predeterminada, la [conversión de modelos](../../how-tos/conversion/model-conversion.md) genera mallas de colisión. Si no necesita las consultas espaciales en un modelo complejo, considere la posibilidad de deshabilitar la generación de mallas de colisión en las [opciones de conversión ](../../how-tos/conversion/configure-model-conversion.md), ya que influye de varias maneras:

* La [conversión de modelos](../../how-tos/conversion/model-conversion.md) tardará bastante más.
* Los tamaños de archivo de modelo convertidos son notablemente mayores, lo que afecta a la velocidad de descarga.
* Los tiempos de carga en tiempo de ejecución son más largos.
* El consumo de memoria de la CPU es más alto.
* Se produce una pequeña sobrecarga de rendimiento en tiempo de ejecución para cada instancia del modelo.

## <a name="ray-casts"></a>Proyecciones de rayo

Una *proyección de rayo* es una consulta espacial en que el tiempo de ejecución comprueba qué objetos cruza un rayo, desde una posición indicada y hacia una dirección determinada. Como optimización, también se proporciona una distancia de rayo máxima, para no buscar objetos que estén demasiado lejos.

```cs
async void CastRay(AzureSession session)
{
    // trace a line from the origin into the +z direction, over 10 units of distance.
    RayCast rayCast = new RayCast(new Double3(0, 0, 0), new Double3(0, 0, 1), 10);

    // only return the closest hit
    rayCast.HitCollection = HitCollectionPolicy.ClosestHit;

    RayCastHit[] hits = await session.Actions.RayCastQueryAsync(rayCast).AsTask();

    if (hits.Length > 0)
    {
        var hitObject = hits[0].HitObject;
        var hitPosition = hits[0].HitPosition;
        var hitNormal = hits[0].HitNormal;

        // do something with the hit information
    }
}
```

```cpp
void CastRay(ApiHandle<AzureSession> session)
{
    // trace a line from the origin into the +z direction, over 10 units of distance.
    RayCast rayCast;
    rayCast.StartPos = { 0, 0, 0 };
    rayCast.EndPos = { 0, 0, 1 };
    rayCast.MaxHits = 10;

    // only return the closest hit
    rayCast.HitCollection = HitCollectionPolicy::ClosestHit;

    ApiHandle<RaycastQueryAsync> castQuery = *session->Actions()->RayCastQueryAsync(rayCast);

    castQuery->Completed([](const ApiHandle<RaycastQueryAsync>& async)
        {
            std::vector<RayCastHit> hits;
            async->GetResult(hits);

            if (hits.size() > 0)
            {
                auto hitObject = hits[0].HitObject;
                auto hitPosition = hits[0].HitPosition;
                auto hitNormal = hits[0].HitNormal;

                // do something with the hit information
            }
        });
}
```


Hay tres modos de recopilación de incidencias:

* **`Closest`:** en este modo, solo se indicará la incidencia más cercana.
* **`Any`:** opte por este modo si todo lo que quiere saber es *si* un rayo incidirá en algo, pero no le importa en qué. Esta consulta puede ser considerablemente más económica de evaluar, pero también tiene pocas aplicaciones.
* **`All`:** en este modo, se registran todas las incidencias del rayo, ordenadas por distancia. No utilice este modo a menos que realmente necesite más que la primera incidencia. Limite el número de incidencias mostradas con la opción `MaxHits`.

Para excluir objetos de forma selectiva de la consideración de proyecciones de rayo, se puede usar el componente [HierarchicalStateOverrideComponent](override-hierarchical-state.md).

<!--
The CollisionMask allows the query to consider or ignore some objects based on their collision layer. If an object has layer L, it will be hit only if the mask has bit L set.
It is useful in case you want to ignore objects, for instance when setting an object transparent, and trying to select another object behind it.
TODO : Add an API to make that possible.
-->

### <a name="hit-result"></a>Resultado de incidencias

El resultado de una consulta de proyección de rayo es una matriz de incidencias. La matriz está vacía, si no incide en ningún objeto.

Una incidencia tiene las siguientes propiedades:

* **`HitEntity`:** [entidad](../../concepts/entities.md) en la que ha incidido.
* **`SubPartId`:** *submalla* en la que ha incidido en [MeshComponent](../../concepts/meshes.md). Puede usarse para la indexación en `MeshComponent.UsedMaterials` y para la búsqueda del [material](../../concepts/materials.md) en ese punto.
* **`HitPosition`:** posición de espacio mundial donde el rayo se intersecó con el objeto.
* **`HitNormal`:** normal de la superficie de espacio mundial de la malla en la posición de la intersección.
* **`DistanceToHit`:** distancia desde la posición inicial del rayo hasta la incidencia.

## <a name="api-documentation"></a>Documentación de la API

* [C# RemoteManager.RayCastQueryAsync()](https://docs.microsoft.com/dotnet/api/microsoft.azure.remoterendering.remotemanager.raycastqueryasync)
* [C++ RemoteManager::RayCastQueryAsync()](https://docs.microsoft.com/cpp/api/remote-rendering/remotemanager#raycastqueryasync)

## <a name="next-steps"></a>Pasos siguientes

* [Límites de objetos](../../concepts/object-bounds.md)
* [Invalidación de estados jerárquicos](override-hierarchical-state.md)

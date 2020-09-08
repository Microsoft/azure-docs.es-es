---
title: Corte de planos
description: Explica qué son los planos de corte y cómo usarlos.
author: jakrams
ms.author: jakras
ms.date: 02/06/2020
ms.topic: article
ms.custom: devx-track-csharp
ms.openlocfilehash: b92bfad99c854f75c945121d352a7122d8c6db89
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89011606"
---
# <a name="cut-planes"></a>Corte de planos

Un *plano de corte* es una característica visual que recorta los píxeles de un lado de un plano virtual y revela el interior de las [mallas](../../concepts/meshes.md).
En la imagen siguiente se muestra el efecto. La izquierda muestra la malla original, mientras que la derecha puede buscar dentro de la malla:

![Plano de corte](./media/cutplane-1.png)

## <a name="limitations"></a>Limitaciones

* De momento, Azure Remote Rendering admite un **máximo de ocho planos de corte activos**. Puede crear más componentes de plano de corte, pero, si intenta habilitar más simultáneamente, se omitirá la activación. Deshabilite primero otros planos si quiere cambiar qué componente debe afectar a la escena.
* Cada plano de corte afecta a todos los objetos representados de forma remota. Actualmente, no hay ninguna manera de excluir objetos específicos ni piezas de malla.
* Los planos de corte son meramente una característica visual, no afectan al resultado de las [consultas espaciales](spatial-queries.md). Si quiere realizar un trazado de rayos en una malla de corte abierto, puede ajustar el punto inicial del rayo para que esté en el plano de corte. De este modo, el rayo solo puede llegar a los elementos visibles.

## <a name="performance-considerations"></a>Consideraciones de rendimiento

Cada plano de corte activo incurre en un pequeño costo durante la representación. Deshabilite o elimine los planos de corte cuando no sean necesarios.

## <a name="cutplanecomponent"></a>CutPlaneComponent

Para agregar un plano de corte a la escena, cree un *CutPlaneComponent*. La ubicación y la orientación del plano viene determinada por la [entidad](../../concepts/entities.md) del propietario del componente.

```cs
void CreateCutPlane(AzureSession session, Entity ownerEntity)
{
    CutPlaneComponent cutPlane = (CutPlaneComponent)session.Actions.CreateComponent(ObjectType.CutPlaneComponent, ownerEntity);
    cutPlane.Normal = Axis.X; // normal points along the positive x-axis of the owner object's orientation
    cutPlane.FadeColor = new Color4Ub(255, 0, 0, 128); // fade to 50% red
    cutPlane.FadeLength = 0.05f; // gradient width: 5cm
}
```

```cpp
void CreateCutPlane(ApiHandle<AzureSession> session, ApiHandle<Entity> ownerEntity)
{
    ApiHandle<CutPlaneComponent> cutPlane = session->Actions()->CreateComponent(ObjectType::CutPlaneComponent, ownerEntity)->as<CutPlaneComponent>();;
    cutPlane->SetNormal(Axis::X); // normal points along the positive x-axis of the owner object's orientation
    Color4Ub fadeColor;
    fadeColor.channels = { 255, 0, 0, 128 }; // fade to 50% red
    cutPlane->SetFadeColor(fadeColor);
    cutPlane->SetFadeLength(0.05f); // gradient width: 5cm
}
```


### <a name="cutplanecomponent-properties"></a>Propiedades de CutPlaneComponent

Las siguientes propiedades se exponen en un componente de plano de corte:

* `Enabled`: Puede desactivar temporalmente los planos de corte deshabilitando el componente. Los planos de corte deshabilitados no incurren en sobrecarga de representación y tampoco se cuentan para el límite de plano de corte global.

* `Normal`: especifica qué dirección (+X,-X,+Y,-Y,+Z,-Z) se usa como normal del plano. Esta dirección es relativa a la orientación de la entidad propietaria. Mueva y gire la entidad propietaria para obtener la ubicación exacta.

* `FadeColor` y `FadeLength`:

  Si el valor alfa de *FadeColor* es distinto de cero, los píxeles cercanos al plano de corte se atenuarán hacia la parte RGB de FadeColor. La fuerza del canal alfa determina si se desvanece completamente hacia el color de fundido o solo parcialmente. *FadeLength* define con qué distancia se llevará a cabo esta transición.

## <a name="next-steps"></a>Pasos siguientes

* [Representación de un solo lado](single-sided-rendering.md)
* [Consultas espaciales](spatial-queries.md)

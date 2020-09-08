---
title: Invalidación de estado jerárquico
description: Explica el concepto de componentes de invalidación de estado jerárquico.
author: florianborn71
ms.author: flborn
ms.date: 02/10/2020
ms.topic: article
ms.custom: devx-track-csharp
ms.openlocfilehash: 99f57c212dfc44d84640224b1526ab770fe97230
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89009464"
---
# <a name="hierarchical-state-override"></a>Invalidación de estado jerárquico

En muchos casos, es necesario cambiar dinámicamente el aspecto de las partes de un [modelo](../../concepts/models.md); por ejemplo, al ocultar subgráficos o cambiar partes a una representación transparente. Cambiar los materiales de cada parte implicada no es práctico, ya que requiere recorrer en iteración todo el gráfico de la escena y administrar la clonación y asignación de materiales en cada nodo.

Para completar este caso de uso con la menor sobrecarga posible, use `HierarchicalStateOverrideComponent`. Este componente implementa las actualizaciones de estado jerárquico en ramas arbitrarias del gráfico de escena. Esto significa que un estado se puede definir en cualquier nivel del gráfico de escena y se filtra en la jerarquía hasta que se reemplaza por un estado nuevo o se aplica a un objeto hoja.

Como ejemplo, considere el modelo de un automóvil y que quiere cambiar el vehículo completo para que sea transparente, excepto la parte del motor interno. Este caso de uso solo implica dos instancias del componente:

* El primer componente se asigna al nodo raíz del modelo y activa la representación transparente para todo el automóvil.
* El segundo componente se asigna al nodo raíz del motor y reemplaza el estado de nuevo al desactivar explícitamente el modo transparente.

## <a name="features"></a>Características

El conjunto fijo de estados que se pueden invalidar es:

* **`Hidden`** : Las mallas respectivas del gráfico de escena se ocultan o se muestran.
* **`Tint color`** : un objeto representado se puede teñir con su intensidad y color de tono individual. En la imagen siguiente se muestra el tinte de color del borde de una rueda.
  
  ![Tono de color](./media/color-tint.png)

* **`See-through`** : la geometría se representa de modo semitransparente; por ejemplo, para mostrar las partes internas de un objeto. En la imagen siguiente se muestra el coche completo que se representa en el modo transparente, excepto la mordaza de freno roja:

  ![Transparente](./media/see-through.png)

  > [!IMPORTANT]
  > El efecto transparente solo funciona cuando se usa el [modo de representación](../../concepts/rendering-modes.md) *TileBasedComposition*.

* **`Selected`** : la geometría se representa con un [contorno de selección](outlines.md).

  ![Contorno de selección](./media/selection-outline.png)

* **`DisableCollision`** : la geometría está exenta de [consultas espaciales](spatial-queries.md). La marca **`Hidden`** no afecta a la marca del estado de las colisiones, por lo que estas dos marcas se suelen establecer juntas.

## <a name="hierarchical-overrides"></a>Invalidaciones jerárquicas

`HierarchicalStateOverrideComponent` se puede adjuntar en varios niveles de una jerarquía de objetos. Puesto que solo puede haber un componente de cada tipo en una entidad, cada objeto `HierarchicalStateOverrideComponent` administra los estados oculto, transparente, selección, tono de color y colisión.

Por lo tanto, cada estado se puede establecer como:

* `ForceOn`: el estado está habilitado para todas las mallas de este nodo y niveles inferiores.
* `ForceOff`: el estado está deshabilitado para todas las mallas de este nodo y niveles inferiores.
* `InheritFromParent`: el estado no se ve afectado por este componente de invalidación.

Puede cambiar los estados directamente o mediante la función `SetState`:

```cs
HierarchicalStateOverrideComponent component = ...;

// set one state directly
component.HiddenState = HierarchicalEnableState.ForceOn;

// set a state with the SetState function
component.SetState(HierarchicalStates.SeeThrough, HierarchicalEnableState.InheritFromParent);

// set multiple states at once with the SetState function
component.SetState(HierarchicalStates.Hidden | HierarchicalStates.DisableCollision, HierarchicalEnableState.ForceOff);
```

```cpp
ApiHandle<HierarchicalStateOverrideComponent> component = ...;

// set one state directly
component->SetHiddenState(HierarchicalEnableState::ForceOn);

// or: set a state with the SetState function
component->SetState(HierarchicalStates::SeeThrough, HierarchicalEnableState::InheritFromParent);

// set multiple states at once with the SetState function
component->SetState(
    (HierarchicalStates)((int32_t)HierarchicalStates::Hidden | (int32_t)HierarchicalStates::DisableCollision), HierarchicalEnableState::ForceOff);

```

### <a name="tint-color"></a>Color de tono

La invalidación de `tint color` es ligeramente especial, ya que hay tanto un estado activado/desactivado/heredado como una propiedad de color de tono. La parte alfa del color de tono define la intensidad del efecto de tono: Si se establece en 0,0, no hay ningún color de tono visible y, si se establece en 1,0, el objeto se representará con un color de tono puro. En el caso de los valores intermedios, el color final se combinará con el color del tono. El color del tono se puede cambiar en cada fotograma para lograr una animación de color.

## <a name="performance-considerations"></a>Consideraciones de rendimiento

Una instancia de `HierarchicalStateOverrideComponent` no agrega mucha sobrecarga en tiempo de ejecución. Sin embargo, siempre es recomendable mantener un número bajo de componentes activos. Por ejemplo, al implementar un sistema de selección que resalta el objeto seleccionado, se recomienda eliminar el componente al quitar el resaltado. El mantenimiento de los componentes en torno a características neutras puede sumar rápidamente.

La representación transparente coloca más carga de trabajo en las GPU del servidor que la representación estándar. Si las partes grandes del gráfico de escena cambian a *transparente*, con muchas capas de geometría visibles, puede convertirse en un cuello de botella de rendimiento. Lo mismo es válido para los objetos con [contornos de selección](../../overview/features/outlines.md#performance).

## <a name="next-steps"></a>Pasos siguientes

* [Contornos](../../overview/features/outlines.md)
* [Modos de representación](../../concepts/rendering-modes.md)
* [Consultas espaciales](../../overview/features/spatial-queries.md)

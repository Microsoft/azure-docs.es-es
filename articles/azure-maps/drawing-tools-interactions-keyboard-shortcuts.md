---
title: Tipos de interacción y métodos abreviados de teclado de las herramientas de dibujo en el mapa | Microsoft Azure Maps
description: Cómo dibujar y editar formas con un mouse, una pantalla táctil o un teclado en el SDK web de Microsoft Azure Maps
author: rbrundritt
ms.author: richbrun
ms.date: 12/05/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.openlocfilehash: 429baa4d5dc85476192758e4500f4c1f14610e6d
ms.sourcegitcommit: 07166a1ff8bd23f5e1c49d4fd12badbca5ebd19c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90089316"
---
# <a name="interaction-types-and-keyboard-shortcuts-in-the-drawing-tools-module"></a>Tipos de interacción y métodos abreviados de teclado en el módulo de herramientas de dibujo

En este artículo se describen las distintas formas de dibujar y editar formas mediante el mouse, la pantalla táctil o métodos abreviados de teclado.

El administrador de dibujo admite tres formas diferentes de interactuar con el mapa para dibujar formas.

* `click`: se agregan coordenadas cuando se hace clic en el mouse o en la entrada táctil.
* `freehand `: se agregan coordenadas cuando se arrastra el mouse o la entrada táctil en el mapa.
* `hybrid`: se agregan coordenadas cuando se hace clic o se arrastra el mouse o la entrada táctil.

## <a name="how-to-draw-shapes"></a>Dibujo de formas

 Antes de que se pueda dibujar cualquier forma, establezca la opción `drawingMode` del administrador de dibujo en un valor de dibujo admitido. Este valor se puede programar o invocar al presionar uno de los botones de dibujo de la barra de herramientas. El modo de dibujo permanece habilitado, incluso después de dibujar una forma, lo que facilita el dibujo de otras formas del mismo tipo. Establezca mediante programación el modo de dibujo en estado de inactividad. O bien, cambie a estado de inactividad al hacer clic en el botón de modos de dibujo actuales de la barra de herramientas.

En las secciones siguientes se describen las distintas maneras de las que se pueden dibujar formas en el mapa.

### <a name="how-to-draw-a-point"></a>Dibujo de un punto

Cuando el administrador de dibujo está en el modo de dibujo `draw-point`, se pueden realizar las siguientes acciones para dibujar puntos en el mapa. Estos métodos funcionan con todos los modos de interacción.

**Inicio del dibujo**
 - Haga clic en el botón izquierdo del mouse o toque el mapa para agregar un punto a él. 
 - Si el mouse está sobre el mapa, presione la tecla `F` y se agregará un punto mediante la coordenada del puntero del mouse. Este método proporciona mayor precisión para agregar un punto al mapa. Gracias al movimiento de presionar el botón izquierdo del mouse, este se moverá menos.
 - Para agregar más puntos al mapa, pulse, toque o presione `F`.
 
**Finalización del dibujo**
 - Haga clic en cualquier botón de la barra de herramientas de dibujo. 
 - Establezca mediante programación el modo de dibujo. 
 - Presione la tecla `C`.

**Cancelación del dibujo**
 - Presione la tecla `Escape`.

### <a name="how-to-draw-a-line"></a>Dibujo de una línea

Cuando el administrador de dibujo está en modo `draw-line`, se pueden realizar las siguientes acciones para dibujar puntos en el mapa, dependiendo del modo de interacción.

**Inicio del dibujo**
 - Modo de clic
   * Haga clic en el botón izquierdo del mouse o toque el mapa para agregar cada punto de una línea en el mapa. Se agrega una coordenada a la línea por cada clic o toque. 
   * Si el mouse está sobre el mapa, presione la tecla `F` y se agregará un punto mediante la coordenada del puntero del mouse. Este método proporciona mayor precisión para agregar un punto al mapa. Gracias al movimiento de presionar el botón izquierdo del mouse, este se moverá menos.
   * Siga haciendo clic hasta que se hayan agregado todos los puntos deseados a la línea.
 - Modo de mano alzada
   * Presione el botón izquierdo del mouse o pulse en el mapa y arrastre el mouse o el punto de toque. Las coordenadas se agregan a la línea a medida que el mouse o el punto táctil se mueve por el mapa. En cuanto se desencadene el evento del mouse o de toque, se completa el dibujo. La frecuencia con la que se agregan las coordenadas se define mediante la opción `freehandInterval` de los administradores de dibujos.
 - Modo híbrido
   * Alterne entre los métodos de clic y mano alzada, según prefiera, mientras dibuja una sola línea. Por ejemplo, haga clic en unos cuantos puntos, mantenga presionado el botón del mouse y arrástrelo para agregar varios puntos y, a continuación, haga clic y dibuje unos cuantos más. 

**Finalización del dibujo**
 - Modo de clic o híbrido
   * Haga doble clic en el mapa en el último punto. 
   * Haga clic en cualquier botón de la barra de herramientas de dibujo. 
   * Establezca mediante programación el modo de dibujo. 
 - Modo de mano alzada
   * Suelte el botón del mouse o punto táctil.
 - Presione la tecla `C`.

**Cancelación del dibujo**
 - Presione la tecla `Escape`.

### <a name="how-to-draw-a-polygon"></a>Dibujo de un polígono

Cuando el administrador de dibujo está en modo `draw-polygon`, se pueden realizar las siguientes acciones para dibujar puntos en el mapa, dependiendo del modo de interacción.

**Inicio del dibujo**
 - Modo de clic
   * Haga clic en el botón izquierdo del mouse o toque el mapa para agregar cada punto de un polígono al mapa. Se agrega una coordenada al polígono por cada clic o toque. 
   * Si el mouse está sobre el mapa, presione la tecla `F` y se agregará un punto mediante la coordenada del puntero del mouse. Este método proporciona mayor precisión para agregar un punto al mapa. Gracias al movimiento de presionar el botón izquierdo del mouse, este se moverá menos.
   * Siga haciendo clic hasta que se hayan agregado todos los puntos deseados al polígono.
 - Modo de mano alzada
   * Presione el botón izquierdo del mouse o pulse en el mapa y arrastre el mouse o el punto de toque. Las coordenadas se agregan al polígono a medida que el mouse o el punto táctil se mueve por el mapa. En cuanto se desencadene el evento del mouse o de toque, se completa el dibujo. La frecuencia con la que se agregan las coordenadas se define mediante la opción `freehandInterval` de los administradores de dibujos.
 - Modo híbrido
   * Alterne entre los métodos de clic y mano alzada, según su preferencia, mientras dibuja un único polígono. Por ejemplo, haga clic en unos cuantos puntos, mantenga presionado el botón del mouse y arrástrelo para agregar varios puntos y, a continuación, haga clic y dibuje unos cuantos más. 

**Finalización del dibujo**
 - Modo de clic o híbrido
   * Haga doble clic en el mapa en el último punto. 
   * Haga clic en el primer punto del polígono.
   * Haga clic en cualquier botón de la barra de herramientas de dibujo. 
   * Establezca mediante programación el modo de dibujo. 
 - Modo de mano alzada
   * Suelte el botón del mouse o punto táctil.
 - Presione la tecla `C`.

**Cancelación del dibujo**
 - Presione la tecla `Escape`.

### <a name="how-to-draw-a-rectangle"></a>Dibujo de un rectángulo

Cuando el administrador de dibujo está en modo `draw-rectangle`, se pueden realizar las siguientes acciones para dibujar puntos en el mapa, dependiendo del modo de interacción. La forma generada seguirá la [especificación de GeoJSON extendida para los rectángulos](extend-geojson.md#rectangle).

**Inicio del dibujo**
 - Presione el botón izquierdo del mouse o pulse en el mapa para agregar el primer vértice del rectángulo y arrástrelo para crear el rectángulo. 

**Finalización del dibujo**
 - Suelte el botón del mouse o punto táctil.
 - Establezca mediante programación el modo de dibujo. 
 - Presione la tecla `C`.

**Cancelación del dibujo**
 - Presione la tecla `Escape`.

### <a name="how-to-draw-a-circle"></a>Dibujo de un círculo

Cuando el administrador de dibujo está en modo `draw-circle`, se pueden realizar las siguientes acciones para dibujar puntos en el mapa, dependiendo del modo de interacción. La forma generada seguirá la [especificación de GeoJSON extendida para los círculos](extend-geojson.md#circle).

**Inicio del dibujo**
 - Presione el botón izquierdo del mouse o pulse en el mapa para agregar el centro del círculo y arrástrelo para crear un radio al círculo. 

**Finalización del dibujo**
 - Suelte el botón del mouse o punto táctil.
 - Establezca mediante programación el modo de dibujo. 
 - Presione la tecla `C`.

**Cancelación del dibujo**
 - Presione la tecla `Escape`.

## <a name="keyboard-shortcuts"></a>Accesos directos del teclado

Las herramientas de dibujo admiten los métodos abreviados de teclado. Estos métodos abreviados de teclado son funcionales cuando el mapa tiene el foco.

| Clave      | Acción                            |
|----------|-----------------------------------|
| `C` | Completa cualquier dibujo que esté en curso y establece el modo de dibujo en inactivo. El foco se desplazará al elemento de mapa de nivel superior.  |
| `Escape` | Cancela cualquier dibujo que esté en curso y establece el modo de dibujo en inactivo. El foco se desplazará al elemento de mapa de nivel superior.  |
| `F` | Agrega una coordenada a un punto, línea o polígono si el mouse está sobre el mapa. Acción equivalente de hacer clic en el mapa cuando se está en modo de clic o híbrido. Este método abreviado permite dibujos más precisos y más rápidos. Puede usar una mano para colocar el mouse y la otra para presionar el botón sin mover el mouse con el gesto de presionar. |

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre las clases en el módulo de herramientas de dibujo:

> [!div class="nextstepaction"]
> [Administrador de dibujo](https://docs.microsoft.com/javascript/api/azure-maps-drawing-tools/atlas.drawing.drawingmanager)

> [!div class="nextstepaction"]
> [Barra de herramientas de dibujo](https://docs.microsoft.com/javascript/api/azure-maps-drawing-tools/atlas.control.drawingtoolbar)

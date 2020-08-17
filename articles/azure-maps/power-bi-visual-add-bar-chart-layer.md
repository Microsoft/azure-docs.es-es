---
title: Incorporación de una capa de gráfico de barras al objeto visual de Azure Maps en Power BI | Microsoft Azure Maps
description: En este artículo, aprenderá a usar la capa de gráfico de barras en el objeto visual de Microsoft Azure Maps para Power BI.
author: rbrundritt
ms.author: richbrun
ms.date: 06/26/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: ''
ms.openlocfilehash: 6a686f6915e332893f619e66944591999063a07c
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87288159"
---
# <a name="add-a-bar-chart-layer"></a>Adición de una capa de gráfico de barras

La **capa de gráfico de barras** resulta útil para llevar los datos a una dimensión superior al permitir la visualización de los datos de ubicación como barras 3D o cilindros en el mapa. De forma similar a la capa de burbujas, la capa de gráfico de barras permite visualizar fácilmente dos métricas a la vez utilizando el color y el alto relativo. Para que las barras tengan altura, es necesario agregar una medida al cubo **Tamaño** del panel **Campos**. Si no se proporciona una medida, las barras sin altura se muestran como cuadrados o círculos planos según la opción **Forma de la barra**.

> [!div class="mx-imgBorder"]
> ![Mapa que muestra datos de punto mediante la capa de gráfico de barras](media/power-bi-visual/bar-chart-layer-styled.png)

Los usuarios pueden inclinar y girar el mapa para ver los datos desde distintas perspectivas. El mapa se puede inclinar mediante uno de los métodos siguientes.

-   Active la opción **Controles de navegación** en la **Configuración del mapa** del panel **Formato**. Con esto se agregará un botón para inclinar el mapa.
-   Presione el botón derecho del mouse y arrastre el mouse hacia arriba o hacia abajo.
-   Con una pantalla táctil: toque el mapa con dos dedos y arrastre hacia arriba o hacia abajo.
-   Con el mapa centrado, mantenga presionada la tecla **Mayús** y presione las teclas de dirección **arriba** o **abajo**.

El mapa se puede girar mediante uno de los métodos siguientes.

-   Active la opción **Controles de navegación** en la **Configuración del mapa** del panel **Formato**. Con esto se agregará un botón para girar el mapa.
-   Presione el botón derecho del mouse y arrastre el mouse hacia la izquierda o hacia la derecha.
-   Con una pantalla táctil: toque el mapa con dos dedos y gira.
-   Con el mapa centrado, mantenga presionada la tecla **Mayús** y presione las teclas de dirección **izquierda** o **derecha**.

A continuación se muestran todas las opciones de configuración del panel **Formato** que están disponibles en la sección **Capa de gráfico de barras**.

| Configuración              | Descripción      |
|----------------------|------------------|
| Forma de la barra            | Forma de la barra 3D.<br/><br/>&nbsp;&nbsp;&nbsp;&nbsp;• Cuadro: Barras representadas como cuadros rectangulares.<br/>&nbsp;&nbsp;&nbsp;&nbsp;• Cilindro: Barras representadas como cilindros. |
| Alto               | Alto de cada barra. Si un campo se pasa al cubo **Tamaño** del panel **Campos**, las barras se escalarán en relación con este valor de alto. |
| Escalar altura al hacer zoom | Especifica si el alto de las barras se debe escalar en función del nivel de zoom. |
| Ancho                | Ancho de cada barra.  |
| Escalar grosor al aplicar zoom  | Especifica si el ancho de las barras se debe escalar en función del nivel de zoom.  |
| Color de relleno           | Color de cada barra. Esta opción está oculta cuando se pasa un campo al cubo **Leyenda** del panel **Campos**, y se muestra una sección independiente de **Colores de datos** en el panel **Formato**. |
| Transparencia         | Transparencia de cada barra. |
| Zoom mínimo             | Nivel de zoom mínimo en el que están disponibles los mosaicos. |
| Zoom máximo             | Nivel de zoom máximo en el que están disponibles los mosaicos. |
| Layer position (Posición de capa)       | Permite especificar la posición de la capa en relación con otras capas de mapa. |

> [!NOTE]
> Si las barras tienen un valor de ancho pequeño y la opción **Escalar grosor al aplicar zoom** está deshabilitada, es posible que desaparezcan cuando se alejen mucho, ya que su ancho representado tendría un tamaño menor a un píxel. Sin embargo, cuando la opción **Escalar grosor al aplicar zoom** está habilitada, se realizan cálculos adicionales cuando cambia el nivel de zoom, lo que puede afectar al rendimiento de grandes conjuntos de datos.

## <a name="next-steps"></a>Pasos siguientes

Agregue más contexto al mapa:

> [!div class="nextstepaction"]
> [Adición de una capa de referencia](power-bi-visual-add-reference-layer.md)

> [!div class="nextstepaction"]
> [Adición de una capa de icono](power-bi-visual-add-tile-layer.md)

> [!div class="nextstepaction"]
> [Mostrar el tráfico en tiempo real](power-bi-visual-show-real-time-traffic.md)

Personalización del objeto visual:

> [!div class="nextstepaction"]
> [Trucos y sugerencias para el formato de color en Power BI](https://docs.microsoft.com/power-bi/visuals/service-tips-and-tricks-for-color-formatting)

> [!div class="nextstepaction"]
> [Personalización de los títulos, los fondos y las leyendas de las visualizaciones](https://docs.microsoft.com/power-bi/visuals/power-bi-visualization-customize-title-background-and-legend)

---
title: Selección de ubicación de zonas y líneas de análisis espacial
titleSuffix: Azure Cognitive Services
description: Aprenda a configurar zonas y líneas con el análisis espacial
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 09/01/2020
ms.author: aahi
ms.openlocfilehash: 7e2a64c14d7d7a1d20b64b746969aca1e60ab218
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932287"
---
# <a name="zone-and-line-placement-guide"></a>Guía de selección de ubicación de zonas y líneas

En este artículo se ofrecen instrucciones sobre cómo definir zonas y líneas para operaciones de análisis espacial para lograr un análisis preciso de los movimientos de las personas en un espacio. Esto se aplica a todas las operaciones. 

Las zonas y las líneas se definen mediante el parámetro SPACEANALYSIS_CONFIG de JSON. Consulte el artículo sobre [operaciones de análisis espacial](spatial-analysis-operations.md) para más información.

## <a name="guidelines-for-drawing-zones"></a>Directrices para dibujar zonas

Recuerde que cada espacio es diferente; deberá actualizar la posición o el tamaño en función de sus necesidades.

Si quiere ver una sección específica de la vista de la cámara, cree la zona más grande que pueda, que cubra el área de suelo concreta que le interesa, pero sin incluir otras áreas que no le interesen. Así aumenta la precisión de los datos recopilados y evita falsos positivos en las áreas cuyo seguimiento no quiere realizar. Tenga cuidado al colocar las esquinas del polígono y asegúrese de que no quedan fuera del área cuyo seguimiento quiere realizar.  

### <a name="example-of-a-well-shaped-zone"></a>Ejemplo de una zona con la forma correcta

La zona debe ser lo suficientemente grande como para que puedan estar tres personas de pie a lo largo de cada borde y se centre en el área de interés. El análisis espacial identifica a las personas cuyos pies queden dentro de la zona, así que cuando dibuje zonas en la imagen 2D, imagine la zona como una alfombra tendida en el suelo.

![Zona con la forma correcta](./media/spatial-analysis/zone-good-example.png)

### <a name="examples-of-zones-that-arent-well-shaped"></a>Ejemplos de zonas que no tienen la forma correcta

En los siguientes ejemplos se muestran zonas que tienen una forma incorrecta. En estos ejemplos, el área de interés es el espacio frente al exhibidor de *It's Game Time*.

**La zona no está en el suelo.**

![Zona con forma incorrecta](./media/spatial-analysis/zone-not-on-floor.png) 

**La zona es demasiado pequeña.**

![la zona es demasiado pequeña](./media/spatial-analysis/zone-too-small.png)

**La zona no captura completamente el área alrededor del exhibidor.**

![la zona no captura completamente el área alrededor del extremo](./media/spatial-analysis/zone-bad-capture.png)

**La zona está demasiado cerca del borde de la imagen de la cámara y no captura el exhibidor correcto.**

![la zona está demasiado cerca del borde de la imagen de la cámara y no captura el exhibidor correcto](./media/spatial-analysis/zone-edge.png)

**La zona queda parcialmente bloqueada por la estantería, por lo que las personas y el suelo no están totalmente visibles.**

![la zona está parcialmente bloqueada, por lo que las personas no están totalmente visibles](./media/spatial-analysis/zone-partially-blocked.png)

### <a name="example-of-a-well-shaped-line"></a>Ejemplo de una línea con la forma correcta

La línea debe ser lo suficientemente larga como para abarcar toda la entrada. El análisis espacial identifica a las personas cuyos pies cruzan la línea, así que cuando dibuje líneas en la imagen 2D, imagine que las dibuja como si estuvieran en el suelo. 

Si es posible, prolongue la línea más allá de la entrada real. Si esto no se traduce en cruces adicionales (como en la imagen siguiente en que la línea queda en una pared), amplíela.

![Línea con la forma correcta](./media/spatial-analysis/zone-line-good-example.png)

### <a name="examples-of-lines-that-arent-well-shaped"></a>Ejemplos de líneas que no tienen la forma correcta

En los siguientes ejemplos se muestran líneas definidas incorrectamente.

**La línea no cubre toda la entrada en el suelo.**

![La línea no cubre toda la entrada en el suelo](./media/spatial-analysis/zone-line-bad-coverage.png)

**La línea es demasiado alta y no cubre la totalidad de la puerta.**

![La línea es demasiado alta y no cubre la totalidad de la puerta](./media/spatial-analysis/zone-line-too-high.png)

## <a name="next-steps"></a>Pasos siguientes

* [Implementación de una aplicación web de recuento de personas](spatial-analysis-web-app.md)
* [Configuración de operaciones de análisis espaciales](./spatial-analysis-operations.md)
* [Registro y solución de problemas](spatial-analysis-logging.md)
* [Guía de selección de ubicación de la cámara](spatial-analysis-camera-placement.md)

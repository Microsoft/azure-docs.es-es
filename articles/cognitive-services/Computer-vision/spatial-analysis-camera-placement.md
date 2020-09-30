---
title: Selección de ubicación de la cámara de análisis espacial
titleSuffix: Azure Cognitive Services
description: Aprenda a configurar una cámara para usarla con análisis espacial.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 09/11/2020
ms.author: aahi
ms.openlocfilehash: 00bca2d0c7e267bef8cd4a917463c5e7db159792
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931919"
---
# <a name="camera-placement-guide"></a>Guía de selección de ubicación de la cámara

En este artículo se brindan recomendaciones para la selección de ubicación de la cámara para el análisis espacial (versión preliminar pública). Se incluyen directrices generales y recomendaciones específicas con respecto a la altura, el ángulo y la distancia entre el punto focal y la cámara para todas las operaciones incluidas. 

> [!NOTE]
> Esta guía está diseñada para la cámara Axis M3045-V. Esta cámara usará la resolución 1920 x 1080, campo visual horizontal de 106 grados, campo visual vertical de 59 grados y una longitud focal fija de 2,8 mm. Los principios que se indican a continuación se aplicarán a todas las cámaras, pero las directrices específicas con respecto a la altura de la cámara y a la distancia entre el punto focal y la cámara se tendrán que ajustar para usarlos con otras cámaras. 

## <a name="general-guidelines"></a>Directrices generales

Tenga en cuenta las directrices generales siguientes al posicionar cámaras para el análisis espacial:

* **Altura de la iluminación.** Coloque las cámaras por debajo de los accesorios de iluminación para que estos no bloqueen las cámaras.
* **Obstáculos.** Para evitar obstruir las vistas de la cámara, tenga en cuenta los obstáculos como postes, letreros, estantes, paredes y cámaras LP existentes.
* **Retroiluminación ambiental.** La retroiluminación exterior afecta la calidad de las imágenes de la cámara. Para evitar condiciones de contraluz fuerte, evite apuntar las cámaras a puertas de vidrio o ventanas externas.
* **Reglas y reglamentaciones de privacidad locales.** Las reglamentaciones locales pueden restringir las imágenes que pueden capturar las cámaras. Asegúrese de comprender y respetar las reglas y reglamentaciones locales antes de posicionar las cámaras.
* **Estructura de construcción.** Los sistemas de climatización, aspersores y cableado existente pueden limitar la instalación y el montaje de las cámaras.
* **Manejo de cables.** Asegúrese de poder enrutar un cable Ethernet desde las ubicaciones planeadas de montaje de las cámaras hasta el conmutador PoE (Power over Internet).

## <a name="height-focal-point-distance-and-angle"></a>Altura, distancia de punto focal y ángulo

Al decidir cómo instalar una cámara para el análisis espacial, debe tener en cuenta tres aspectos:
- Altura de la cámara
- Distancia entre el punto focal y la cámara
- Ángulo de la cámara con respecto al plano de suelo

También es importante saber, si es posible, en qué dirección camina la mayoría de las personas (dirección de la caminata de una persona) con respecto al campo visual de la cámara. Esta dirección es importante para el rendimiento del sistema.

![Imagen de una persona que camina en una dirección](./media/spatial-analysis/person-walking-direction.png)

En la ilustración siguiente se muestra la vista de elevación correspondiente a la dirección de la caminata de una persona.

![Vista del plan y elevación](./media/spatial-analysis/person-walking-direction-diagram.png)

## <a name="camera-height"></a>Altura de la cámara

Por lo general, las cámaras se deben montar a 12 o 14 pies del suelo. Al planear el montaje de la cámara en este rango, tome en cuenta los obstáculos (por ejemplo, estantes, luminarias, letreros colgantes y pantallas) que podrían afectar la vista de la cámara y ajuste la altura según sea necesario.

## <a name="camera-to-focal-point-distance"></a>Distancia entre el punto focal y la cámara

La _distancia entre el punto focal y la cámara_ es la distancia lineal desde el punto focal (o el punto focal central del punto de interés) hasta la cámara.

![Distancia entre el punto focal y la cámara](./media/spatial-analysis/camera-focal-point.png)

Esta distancia se mide en el plano de suelo.

![Medición de la distancia entre el punto focal y la cámara desde el suelo](./media/spatial-analysis/camera-focal-point-floor-plane.png)

Desde arriba, tiene el aspecto siguiente:

![Medición de la distancia entre el punto focal y la cámara desde arriba](./media/spatial-analysis/camera-focal-point-above.png)

Use la tabla siguiente para determinar la distancia de la cámara desde el punto focal en función de alturas de montaje específicas. Estas distancias son para lograr una ubicación óptima. Tenga en cuenta que la tabla proporciona directrices por debajo de la recomendación de 12'-14', puesto que muchos cielos rasos pueden limitar la altura.

| Altura de la cámara | Distancia entre el punto focal y la cámara (mín./máx.) |  
| ------------- | ---------------------------------------- |  
| 8'            | 10' - 13'                                  |  
| 10'           | 7' - 13'                                   |  
| 12'           | 10' - 17'                                  |  
| 14'           | 11' - 18'                                  |  
| 16'           | 12' - 22'                                  |  
| 20'           | 15' - 30'                                  |  

En la ilustración siguiente se simulan vistas de cámara de las distancias más cercanas y más lejanas entre el punto focal y la cámara.

| Más cercana                                      | Más lejana                                      |  
| -------------------------------------------- | --------------------------------------------- |  
| ![Distancia más cercana entre el punto focal y la cámara](./media/spatial-analysis/focal-point-closest.png) | ![Distancia más lejana entre el punto focal y la cámara](./media/spatial-analysis/focal-point-farthest.png) |  

## <a name="camera-angle-mounting-ranges"></a>Rangos de montaje del ángulo de cámara

En esta sección se describen los rangos de montaje del ángulo de cámara aceptables. Estos rangos de montaje muestran el rango aceptable para la selección de ubicación óptima.

### <a name="line-configuration"></a>Configuración de línea

En la tabla siguiente se muestran las recomendaciones para las cámaras configuradas para la operación **cognitiveservices.vision.spatialanalysis-personcrossingline**. 

| Altura de la cámara | Distancia entre el punto focal y la cámara | Ángulo de montaje de cámara óptimo (mín./máx.) |  
| ------------- | ------------------------------ | ------------------------------------------ |  
| 8'            | 9'                             | +/-40°                                     |  
| 10'           | 10'                            | +/-30°                                     |  
| 12'           | 13'                            | +/-20°                                     |  
| 16'           | 18'                            | +/-10°                                     |  
| 20'           | 22'                            | +/-10°                                     |  

En la ilustración siguiente se simulan vistas de cámara que usan las recomendaciones de ángulo de montaje más a la izquierda (-) y más a la derecha (+) para usar **cognitiveservices.vision.spatialanalysis-personcrossingline** para contar las personas que entran por una puerta.

| Vista más a la izquierda                | Vista más a la derecha                |  
| ---------------------------- | ----------------------------- |  
| ![Ángulo de cámara más a la izquierda](./media/spatial-analysis/camera-angle-left.png) | ![Ángulo de cámara más a la derecha](./media/spatial-analysis/camera-angle-right.png) |  

En la ilustración siguiente se muestran los ángulos de montaje y selección de ubicación de la cámara desde una vista general.

![Vista general](./media/spatial-analysis/camera-angle-top.png)

### <a name="zone-configuration"></a>Configuración de zona

Se recomienda colocar cámaras a 10 pies o más sobre el suelo para garantizar que el área cubierta sea lo suficientemente grande. 

Cuando la zona está próxima a un obstáculo como una pared o un estante, monte las cámaras a la distancia especificada del destino dentro de un rango de ángulo aceptable de 120 grados, tal como se muestra en la ilustración siguiente.

![Ángulo de cámara aceptable](./media/spatial-analysis/camera-angle-acceptable.png)

En la ilustración siguiente se proporcionan simulaciones de las vistas de cámara izquierda y derecha de un área junto a un estante.

| Vista izquierda        | Vista derecha        |  
| ---------------- | ----------------- |  
| ![Vista izquierda](./media/spatial-analysis/end-cap-left.png) | ![Vista derecha](./media/spatial-analysis/end-cap-right.png) |  

#### <a name="queues"></a>Colas

Las aptitudes **cognitiveservices.vision.spatialanalysis-personcount**, **cognitiveservices.vision.spatialanalysis-persondistance** y **cognitiveservices.vision.spatialanalysis-personcrossingpolygon** se pueden usar para supervisar las colas. Para obtener una calidad óptima de los datos de la cola, se prefieren barreras con correas retráctiles para minimizar la oclusión de las personas en la cola y garantizar que la ubicación de las colas será coherente en el tiempo.

![Cola de correas retráctiles](./media/spatial-analysis/retractable-belt-queue.png)

Este tipo de barreras es preferible a las barreras opacas para la formación de colas a fin de maximizar la precisión de las conclusiones provenientes del sistema.

Hay dos tipos de colas: lineal y zigzag.

En la ilustración siguiente se muestran recomendaciones para las colas lineales:

![Recomendación para cola lineal](./media/spatial-analysis/camera-angle-linear-queue.png)

En la ilustración siguiente se proporciona simulaciones para las vistas de cámara izquierda y derecha de las colas lineales. Tenga en cuenta que puede montar la cámara en el lado opuesto de la cola.

| Vista izquierda                          | Vista derecha                          |  
| ---------------------------------- | ----------------------------------- |  
| ![Ángulo izquierdo de la cola lineal](./media/spatial-analysis/camera-angle-linear-left.png) | ![Ángulo derecho de la cola lineal](./media/spatial-analysis/camera-angle-linear-right.png) |  

En el caso de las colas en zigzag, se recomienda evitar colocar la cámara directamente frente a la dirección de la línea de la cola, como se muestra en la ilustración siguiente. Tenga en cuenta que cada una de las cuatro posiciones de la cámara de ejemplo que aparecen en la ilustración proporciona la vista ideal con una desviación aceptable de +/-15 grados en cada dirección.

En las ilustraciones siguientes se simula la vista de una cámara colocada en las ubicaciones ideales para una cola en zigzag.

| Vista 1        | Vista 2        |  
| ------------- | ------------- |  
| ![Vista 1](./media/spatial-analysis/camera-angle-ideal-location-right.png) | ![Vista 2](./media/spatial-analysis/camera-angle-ideal-location-left.png) |  

| Vista 3 |  Vista 4 |  
| ---- | ----  |
| ![Vista 3](./media/spatial-analysis/camera-angle-ideal-location-back.png) |  ![Vista 4](./media/spatial-analysis/camera-angle-ideal-location-back-left.png) | 

##### <a name="organic-queues"></a>Colas orgánicas

Las líneas de colas orgánicas se forman de manera orgánica. Este estilo de cola es aceptable si las colas no tienen más de 2 a 3 personas y la línea se forma dentro de la definición de la zona. Si la longitud de la cola tiene habitualmente más de 2 a 3 personas, se recomienda usar una barrera con correas retráctiles para ayudar a guiar la dirección de la cola y garantizar que la línea se forme dentro de la definición de la zona.

## <a name="next-steps"></a>Pasos siguientes

* [Implementación de una aplicación web de recuento de personas](spatial-analysis-web-app.md)
* [Configuración de operaciones de análisis espaciales](./spatial-analysis-operations.md)
* [Registro y solución de problemas](spatial-analysis-logging.md)
* [Guía de selección de ubicación de zonas y líneas](spatial-analysis-zone-line-placement.md)

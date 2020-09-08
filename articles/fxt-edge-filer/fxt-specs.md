---
title: Especificaciones de Microsoft Azure FXT Edge Filer | Microsoft Docs
description: Conozca las especificaciones físicas y del entorno del hardware de Microsoft Azure FXT Edge Filer.
author: ekpgh
ms.service: fxt-edge-filer
ms.topic: conceptual
ms.date: 06/20/2019
ms.author: rohogue
ms.openlocfilehash: 30af35075c06585b9c490495e9897c145e9974d9
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88184693"
---
# <a name="azure-fxt-edge-filer-specifications"></a>Especificaciones de Azure FXT Edge Filer

En este artículo se explican las especificaciones de hardware de los nodos de hardware de Azure FXT Edge Filer. En la práctica, se configuran juntos tres o más nodos para proporcionar el sistema de caché agrupado.

## <a name="hardware-specifications"></a>Especificaciones del hardware

| Componente | FXT 6600 | FXT 6400 |
|----------|-----------|-----------|
| Núcleos de CPU |  16 | 16 |
| DRAM  | 1536 GB | 768 GB |
| Puertos de red | 6 x 25/10 Gb + 2 x 1 Gb | 6 x 25/10 Gb + 2 x 1 Gb |
| Capacidad de SSD NVMe | 25,6 TB | 12,8 TB |

## <a name="drive-specifications"></a>Especificaciones de la unidad

El sistema tiene diez bahías de unidad, accesibles desde la parte frontal. Cada unidad ocupada está etiquetada a la derecha con información de capacidad. 

Los números de unidad están impresos en el espacio entre las unidades. En Azure FXT Edge Filer, la unidad 0 es la unidad superior izquierda y la unidad 1 está directamente debajo de ella.

![foto de una bahía de disco duro en el chasis de FXT, que muestra los números de unidad y las etiquetas de capacidad](media/fxt-drives-photo.png)

| Números de unidad    |  Uso   |  Especificaciones |
|------------------|--------|-----------------|
| 0, 1             | SO     | SATA SSD de 480 GB |
| 2, 3, 4, 5, 6, 7, 8, 9 | data   | FXT 6600: SSD NVMe de 3,2 TB <br> FXT 6400: SSD NVMe de 1,6 TB |


## <a name="dimensions-and-weight"></a>Dimensiones y peso

Azure FXT Edge Filer está diseñado para caber en un bastidor de equipamiento estándar de 19" y es una unidad de bastidor alta (1U). 

<!-- 10x2.5 inches version -->

| Dimensiones de Filer            | Value                    |
|-----------------------------|--------------------------|
| Alto                      | 42,8 mm (1,68 pulgadas)    |
| Ancho (incluidos los bordes del bastidor) | 482,0 mm (18,97 pulgadas)  |
| Ancho (caja principal)      | 434,0 mm (17,08 pulgadas) |
| Profundidad: desde los bordes del bastidor hasta la parte posterior de la caja principal                   | 733,82 mm (29,61 pulgadas) |
| Profundidad: desde los bordes del bastidor hasta el saliente posterior más alejado                 | 772,67 mm (30,42 pulgadas) |
| Profundidad: desde los bordes del bastidor hasta el saliente frontal más alejado, sin bisel | 22,0 mm (0,87 pulgadas)  |
| Profundidad: desde los bordes del bastidor hasta el saliente frontal más alejado, con bisel    | 35,84 mm (1,41 pulgadas) |

| Peso | Value |
|-----------------|----------------------|
| Peso del nodo (sin embalaje, sin accesorios) | 18,1 kg (40 libras) |
| Peso neto (sin embalaje, accesorios incluidos) | 23,1 kg (51 libras)|
| Peso bruto (tal y como se envía, incluido todo el embalaje) |  29,0 kg (64 libras) |

### <a name="shipping-dimensions"></a>Dimensiones del envío

| Dimensión del paquete | Milímetros | Pulgadas |
|-------------------|-------------|--------|
| Alto            | 311,2       | 12,25" |
| Ancho             | 642,8       | 25,31" |
| Length            | 1051,1     | 41,38" |

## <a name="power-and-thermal-specifications"></a>Energía y especificaciones térmicas

En esta sección se proporcionan las clasificaciones de energía y las medidas de Azure FXT Edge Filer.

### <a name="nameplate-ratings"></a>Clasificaciones nominales

| Clasificaciones nominales de los modelos de la serie FXT 6000 |
|----------------|
| 100 - 240 V~    |
| 10A - 5 A (X2)  |
| 50/60 Hz         |

<!-- matches the Dell regulatory label exactly -->

### <a name="power-and-thermal-measurements"></a>Mediciones de potencia y térmicas 

Los nodos de Azure FXT Edge Filer usan ventiladores de velocidad variable, por lo que la potencia depende de la temperatura y de la carga. Los ventiladores pueden alcanzar velocidades máximas con determinadas combinaciones de carga alta y temperaturas ambiente elevadas. 

En estos gráficos se proporcionan las medidas de consumo de energía y salida térmica de las combinaciones de frecuencia de voltaje usadas más comúnmente. 

| Potencia de FXT 6600 a temperatura ambiente <br />(22° C, 71,6 ° F) | 100 V, 60 Hz | 120 V, 60 Hz | 208 V, 60 Hz | 230 V, 50 Hz | 240 V, 50 Hz | 
|---------|---|---|---|---|---|
| Voltaje (V) | 100 | 120 | 208 | 230 | 240 | 
| Frecuencia (Hz) | 60 | 60 | 60 | 50 | 50 |
| Corriente (A) | 5,02 | 4,16 |2,40 | 2,20 | 2.16 |
| Potencia aparente (VA) | 502 | 499 | 499 | 506 | 518|
| Factor de potencia | 0,99 | 0,99 |0.98 | 0.98 | 0.98 |
| Potencia real (W) | 497 |494 | 489 | 496 | 508 |
| Disipación térmica (BTU/h) |1696 | 1686 | 1669 | 1692 | 1733 |

| Potencia de FXT 6600 con los ventiladores a velocidad máxima | 100 V, 60 Hz | 120 V, 60 Hz | 208 V, 60 Hz | 230 V, 50 Hz | 240 V, 50 Hz | 
|---------|---|---|---|---|---|
| Voltaje (V) | 100 |120 | 208 | 230 | 240| 
| Frecuencia (Hz) | 60 | 60 | 60 | 50 | 50 |
| Corriente (A) | 5,98 | 5,01 | 2.81 | 2.55 | 2,48 |
| Potencia aparente (VA) | 598 | 601 | 584 | 587 | 595 |
| Factor de potencia | 0,99 | 0,99 | 0.98 | 0.98 | 0.98 |
| Potencia real (W) | 592 | 595 | 573 | 575 | 583 |
| Disipación térmica (BTU/h) | 2020 |2031 | 1954 | 1961 | 1990 |

| Potencia de FXT 6400 a temperatura ambiente <br />(22° C, 71,6 ° F) | 100 V, 60 Hz | 120 V, 60 Hz | 208 V, 60 Hz | 230 V, 50 Hz | 240 V, 50 Hz | 
|---------|---|---|---|---|---|
| Voltaje (V) | 100 | 120 | 208 | 230 | 240 |
| Frecuencia (Hz) |60 | 60 | 60 | 50 | 50 |
| Corriente (A) | 4,63 | 3,86 | 2,24 | 2,04 | 1,94 |
| Potencia aparente (VA) | 463 | 463 | 466 | 469 | 466 |
| Factor de potencia | 0,99 | 0,99 | 0.98 | 0.98 | 0.98 | 
| Potencia real (W) | 458 | 459 | 457 | 460 | 456 |
| Disipación térmica (BTU/h) | 1564 | 1565 | 1558 | 1569 | 1557 |

| Potencia de FXT 6400 con los ventiladores a velocidad máxima | 100 V, 60 Hz | 120 V, 60 Hz | 208 V, 60 Hz | 230 V, 50 Hz | 240 V, 50 Hz |
|---------|---|---|---|---|---|
| Voltaje (V) | 100 | 120 | 208 | 230 | 240 |
| Frecuencia (Hz) | 60 | 60 | 60 | 50 | 50 |
| Corriente (A) | 5.15 | 4,28 | 2,48 | 2,28 | 2,13 |
| Potencia aparente (VA) | 515 | 514 | 516 | 524 | 511 |
| Factor de potencia | 0,99 | 0,99 | 0.98 | 0.98 | 0.98 |
| Potencia real (W) | 510 | 508 | 506 | 514 | 501 |
| Disipación térmica (BTU/h) | 1740 | 1735 | 1725 | 1753 | 1709 |

## <a name="environmental-requirements"></a>Requisitos de entorno

En esta sección se proporcionan especificaciones del entorno ambiente del hardware.

### <a name="temperature-and-humidity"></a>Temperatura y humedad

| Atributo del entorno   | Rango de funcionamiento                   | Sin rango de funcionamiento         |
|---------------------------|-----------------------------------|-----------------------------|
| Intervalo de temperatura ambiente | 10° C a 35° C (50 - 86° F)          | -40 °C a 65 °C (-40 °F - 149 °F) |
| Humedad relativa del ambiente | 10 % - 80 % sin condensación          | 5 % - 95 % sin condensación     |
| Punto de rocío máximo         | 29 °C (84 °F)                       | 33 °C (91 °F)                 |
| Altitud                  | hasta 3048 metros (10 000 pies), sujeto a la degradación por temperatura indicada a continuación | hasta 12 000 metros (39,370 pies) |

> [!NOTE] 
> **Degradación por temperatura según altitud:** La temperatura máxima se reduce en 1 °C/300 m (1 °F/547 pies) por encima de los 950 m (3117 pies).

### <a name="airflow-shock-and-vibration"></a>Flujo de aire, golpes y vibración 

| Atributo         | Especificación |
|-------------------|---------------|
| Flujo de aire                    | El flujo de aire del sistema va de delante atrás. El sistema debe funcionar con una instalación de baja presión y escape trasero. |
| Golpes (operativo)         | 6 G para 11 milisegundos (probado en 6 orientaciones) |
| Golpes (no operativo)     | 71 G para 2 milisegundos (probado en 6 orientaciones) |
| Vibraciones (operativo)     | 0,26 G<sub>RMS</sub> 5 Hz a 350 Hz aleatorio         |
| Vibraciones (no operativo) | 1,88 G<sub>RMS</sub> 10 Hz a 500 Hz durante 15 minutos (los seis lados probados).  |

## <a name="safety-regulation-compliance"></a>Cumplimiento del reglamento de seguridad 

Azure FXT Edge Filer cumple con los reglamentos indicados. 

| Category       | Especificación normativa | 
|----------------|--------------------------|
| Seguridad general | EN 60950-1:2006 +A1:2010 +A2:2013 +A11:2009 +A12:2011/IEC 60950-1:2005 ed2 +A1:2009 +A2:2013 <br>EN 62311:2008 | 
| CEM            | FCC A, ICES-003  <br>EN 55032:2012/CISPR 32:2012  <br>EN 55032:2015/CISPR 32:2015  <br>EN 55024:2010 +A1:2015/CISPR 24:2010 +A1:2015  <br>EN 61000-3-2:2014/IEC 61000-3-2:2014 (Clase D)   <br>EN 61000-3-3:2013/IEC 61000-3-3:2013 |
| Sector energético         | N.º de Reglamento de la Comisión (UE) 617/2013  |
| RoHS           |    EN 50581:2012   |
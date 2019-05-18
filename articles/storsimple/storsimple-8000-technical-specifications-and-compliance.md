---
title: Especificaciones técnicas de StorSimple | Microsoft Docs
description: Describe las especificaciones técnicas y la información sobre el cumplimiento de estándares reglamentarios para los componentes de hardware de StorSimple.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 06/02/2017
ms.author: alkohli
ms.openlocfilehash: 33becd46747f957cce67fb7b2ace170a9aa402ff
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65606922"
---
# <a name="technical-specifications-and-compliance-for-the-storsimple-device"></a>Especificaciones técnicas y cumplimiento normativo para el dispositivo StorSimple

## <a name="overview"></a>Información general

Los componentes de hardware del dispositivo StorSimple de Microsoft Azure se adhieren a las especificaciones técnicas y los estándares reglamentarios descritos en este artículo. Las especificaciones técnicas describen los módulos de alimentación y refrigeración (PCM), las unidades de disco, la capacidad de almacenamiento y los revestimientos. La información de cumplimiento normativo trata aspectos tales como las normas internacionales, la seguridad, las emisiones y el cableado.

## <a name="power-and-cooling-module-specifications"></a>Especificaciones del módulo de alimentación y refrigeración

El dispositivo StorSimple tiene dos módulos de alimentación y refrigeración (PCM) compatibles con SBB con ventilador dual de 100-240 V. Esto proporciona una configuración de alimentación redundante. Si se produce un error en un PCM, el dispositivo sigue funcionando con normalidad en el otro PCM hasta que se reemplaza el módulo con error.

El revestimiento de EBOD usa un PCM de 580 W, y el revestimiento principal usa un PCM de 764 W. En las tablas siguientes se enumeran las especificaciones técnicas asociadas a los PCM.

| Especificación | PCM de 580 W (EBOD) | PCM de 764 W (principal) |
| --- | --- | --- |
| Potencia de salida máxima |580 W |764 |
| Frecuencia |50/60 Hz |50/60 Hz |
| Selección del intervalo de voltaje |Intervalo automático: 90 – 264 V CA, 47/63 Hz |Intervalo automático: 90 - 264 V CA, 47/63 Hz |
| Corriente de entrada máxima |20 A |20 A |
| Corrección del factor de potencia |>95% del voltaje de entrada nominal  |>95% del voltaje de entrada nominal  |
| Armónicos |Cumple la norma EN 61000-3-2 |Cumple la norma EN 61000-3-2 |
| Salida |Voltaje en espera de 5 V \@ 2,0 A |Voltaje en espera de 5 V \@ 2,7 A |
| +5 V \@ 42 A |+5 V \@ 40 A | |
| +12 V \@ 38 A |+12 V \@ 38 A | |
| Conectable en funcionamiento |Sí |Sí |
| Conmutadores y LED |Conmutador de encendido y apagado de CA y cuatro LED indicadores de estado |Conmutador de encendido y apagado de CA y seis LED indicadores de estado |
| Refrigeración del revestimiento |Ventiladores axiales de refrigeración con control de velocidad del ventilador variable |Ventiladores axiales de refrigeración con control de velocidad del ventilador variable |

## <a name="power-consumption-statistics"></a>Estadísticas de consumo de energía

En la siguiente tabla se enumeran los datos de consumo de energía típicos (puede que estos valores no coincidan con los publicados) de los distintos modelos del dispositivo StorSimple.

| Condiciones | 240 V CA | 240 V CA | 240 V CA | 110 V CA | 110 V CA | 110 V CA |
| --- | --- | --- | --- | --- | --- | --- |
|  Ventiladores lentos, unidades inactivas |1,45 A |0,31 kW |1057,76 BTU/h |3,19 A |0,34 kW |1160,13 BTU/h |
|  Ventiladores lentos, unidades accediendo |1,54 A |0,33 kW |1126,01 BTU/h |3,27 A |0,36 kW |1228,37 BTU/h |
|  Ventiladores rápidos, unidades inactivas, dos fuentes de alimentación activas |2,14 A |0,49 kW |1671,95 BTU/h |4,99 A |0,54 kW |1842,56 BTU/h |
|  Ventiladores rápidos, unidades inactivas, una fuente de alimentación activa y una inactiva |2,05 A |0,48 kW |1637,83 BTU/h |4,58 A |0,50 kW |1706,07 BTU/h |
|  Ventiladores rápidos, unidades accediendo, dos fuentes de alimentación activas |2,26 A |0,51 kW |1740,19 BTU/h |4,95 A |0,54 kW |1842,56 BTU/h |
|  Ventiladores rápidos, unidades accediendo, una fuente de alimentación activa y una inactiva |2,14 A |0,49 kW |1671,95 BTU/h |4,81 A |0,53 kW |1808,44 BTU/h |

## <a name="disk-drive-specifications"></a>Especificaciones de la unidad de disco

El dispositivo StorSimple admite hasta 12 unidades de disco SCSI conectadas en serie (SAS) con factor de forma de 3,5 pulgadas. Las unidades reales pueden ser una combinación de unidades de estado sólido (SSD) o unidades de disco duro (HDD), en función de la configuración del producto. Las 12 ranuras de la unidad de disco se encuentran en una configuración de 3 por 4 delante del revestimiento. El revestimiento de EBOD permite el almacenamiento adicional de otras 12 unidades de disco. Estas son siempre unidades de disco duro.

## <a name="storage-specifications"></a>Especificaciones de almacenamiento

Los dispositivos StorSimple tienen una combinación de unidades de disco duro y unidades de estado sólido en los modelos 8100 y 8600. La capacidad total utilizable de los modelos 8100 y 8600 es de unos 15 TB y 38 TB, respectivamente. La tabla siguiente proporciona información detallada acerca de las SSD, las HDD y la capacidad de nube en el contexto de la capacidad de una solución StorSimple.

| Modelo de dispositivo/capacidad | 8100 | 8600 |
| --- | --- | --- |
| Número de unidades de disco duro (HDD) |8 |19 |
| Número de unidades de estado sólido (SSD) |4 |5 |
| Capacidad de un solo HDD |4 TB |4 TB |
| Capacidad de un solo SSD |400 GB |800 GB |
| Capacidad de reserva |4 TB |4 TB |
| Capacidad de HDD utilizable |14 TB |36 TB |
| Capacidad de SSD utilizable |800 GB |2 TB |
| Capacidad total utilizable* |~ 15 TB |~ 38 TB |
| Capacidad máxima de la solución (incluida la nube) |200 TB |500 TB |

<sup>* </sup>- *La capacidad total utilizable incluye la capacidad disponible para datos, metadatos y búferes. Puede aprovisionar volúmenes anclados localmente de hasta 8,5 TB en dispositivos del modelo 8100 o de hasta 22,5 TB en dispositivos del modelo 8600 de mayor tamaño. Para obtener más información, consulte la sección [Volúmenes de StorSimple anclados localmente](storsimple-8000-local-volume-faq.md).*

## <a name="enclosure-dimensions-and-weight-specifications"></a>Especificaciones de dimensiones y peso del revestimiento

En las tablas siguientes se enumeran las diferentes especificaciones de dimensiones y peso del revestimiento.

### <a name="enclosure-dimensions"></a>Dimensiones del revestimiento

En la tabla siguiente se enumeran las dimensiones del revestimiento en milímetros y pulgadas.

| Revestimiento | Milímetros | Pulgadas |
| --- | --- | --- |
| Altura |87,9 |3,46 |
| Ancho en brida de montaje |483 |19,02 |
| Ancho en el cuerpo del revestimiento |443 |17,44 |
| Profundidad desde la brida de montaje frontal hasta el extremo del cuerpo del revestimiento |577 |22,72 |
| Profundidad desde el panel de operaciones hasta el extremo más alejado del revestimiento |630,5 |24,82 |
| Profundidad desde la brida de montaje hasta el extremo más alejado del revestimiento |603 |23,74 |

### <a name="enclosure-weight"></a>Peso del revestimiento

Según la configuración, un revestimiento principal completamente cargado puede pesar de 21 a 33 kg y requiere dos personas para manipularlo.

| Revestimiento | Peso |
| --- | --- |
| Peso máximo (depende de la configuración) |30 kg - 33 kg |
| Vacío (sin unidades instaladas) |21 kg - 23 kg |

## <a name="enclosure-environment-specifications"></a>Especificaciones medioambientales del revestimiento

En esta sección se enumeran las especificaciones relacionadas con el entorno del revestimiento. En esta categoría se incluyen la temperatura, la humedad, la altitud, los golpes, las vibraciones, la orientación, la seguridad y la compatibilidad electromagnética (CEM).

### <a name="temperature-and-humidity"></a>Temperatura y humedad

| Revestimiento | Intervalo de temperatura ambiente | Humedad relativa del ambiente | Temperatura máxima de termómetro húmedo |
| --- | --- | --- | --- |
| Operativo |5 °C - 35 °C (41 °F - 95 °F) |20% - 80% sin condensación |28 °C (82 °F) |
| No operativo |-40 °C - 70 °C (40 °F - 158 °F) |5% - 100% sin condensación |29 °C (84 °F) |

### <a name="airflow-altitude-shock-vibration-orientation-safety-and-emc"></a>Flujo de aire, altitud, golpes, vibraciones, orientación, seguridad y CEM

| Revestimiento | Especificaciones operativas |
| --- | --- |
| Flujo de aire |El flujo de aire del sistema va de delante atrás. El sistema debe funcionar con una instalación de baja presión y escape trasero. La contrapresión creada por puertas del revestimiento y los obstáculos no debe superar los 5 pascales (medidor de agua de 0,5 mm). |
| Altitud (operativo) |De -30 metros a 3.045 metros (de -100 pies a 10.000 pies) con la temperatura máxima de funcionamiento reducida en 5 °C por encima de los 7.000 pies. |
| Altitud (no operativo) |De -305 metros a 12.192 metros (de -1.000 pies a 40.000 pies) |
| Golpes (operativo) |5g 10 ms ½ seno |
| Golpes (no operativo) |30g 10 ms ½ seno |
| Vibraciones (operativo) |0,21 g RMS 5-500 Hz aleatorio |
| Vibraciones (no operativo) |1,04 g RMS 2-200 Hz aleatorio |
| Vibraciones (reubicación) |3 g 2-200 Hz seno |
| Orientación y montaje |Montaje en bastidor de 19 pulgadas (2 unidades EIA) |
| Guías del bastidor |Para ajustarse a bastidores con una profundidad mínima de 700 mm (31,50 pulgadas) compatibles con CEI 297 |
| Seguridad y homologaciones |CE y UL EN 61000-3, CEI 61000-3, UL 61000-3 |
| CEM |EN 55022 (CISPR - A), FCC A |

## <a name="international-standards-compliance"></a>Cumplimiento de normas internacionales

El dispositivo de Microsoft Azure StorSimple cumple los estándares internacionales siguientes:  

* CE - EN 60950-1
* Informe de CB para CEI 60950-1
* UL y cUL para UL 60950-1

## <a name="safety-compliance"></a>Cumplimiento de seguridad

El dispositivo de Microsoft Azure StorSimple cumple las clasificaciones de seguridad siguientes:

* Aprobación de tipo de producto del sistema: UL, cUL, CE
* Cumplimiento de seguridad: UL 60950, IEC 60950, EN 60950

## <a name="emc-compliance"></a>Cumplimiento de normas de CEM

El dispositivo de Microsoft Azure StorSimple cumple las siguientes clasificaciones de CEM.

### <a name="emissions"></a>Emisiones

El dispositivo es compatible con las exigencias de CEM para los niveles de emisiones radiadas y conducidas.

* Lleva a cabo niveles límite de emisiones: CFR 47 parte 15B clase A EN55022 Clase A CISPR clase A
* Niveles límite de emisiones de radiadas: CFR 47 parte 15B clase A EN55022 Clase A CISPR clase A

### <a name="harmonics-and-flicker"></a>Armónicos y parpadeo

El dispositivo cumple con la norma EN 61000-3-2/3.

### <a name="immunity-limit-levels"></a>Niveles límite de inmunidad

El dispositivo cumple con la norma EN 55024.

## <a name="ac-power-cord-compliance"></a>Cumplimiento de cable de alimentación de CA

El enchufe y el cable toda la eficacia deben cumplir los estándares apropiados para el país o región en la que se usa el dispositivo, y deben tener las aprobaciones de seguridad que son aceptables en ese país o región. Las tablas siguientes enumeran los estándares de los Estados Unidos y Europa.

### <a name="ac-power-cords---usa-must-be-nrtl-listed"></a>Cables de alimentación de CA - Estados Unidos (debe estar contemplado por NRTL)

| Componente | Especificación |
| --- | --- |
| Tipo de cable |SV o SVT, AWG 18 como mínimo, 3 conductores, longitud máxima de 2,0 metros |
| Enchufe |Enchufe de toma de tierra NEMA 5-15P para 120 V, 10 A; o CEI 320 C14, 250 V, 10 A |
| Toma de corriente |CEI 320 C-13, 250 V, 10 A |

### <a name="ac-power-cords---europe"></a>Cables de alimentación de CA - Europa

| Componente | Especificación |
| --- | --- |
| Tipo de cable |Armonizado, H05-VVF-3G1.0 |
| Socket |CEI 320 C-13, 250 V, 10 A |

## <a name="supported-network-cables"></a>Cables de red admitidos

Para las interfaces de red de 10 GbE, DATA 2 y DATA 3, consulte la [lista de cables de red y módulos compatibles](storsimple-supported-hardware-for-10-gbe-network-interfaces.md).

## <a name="next-steps"></a>Pasos siguientes

Ya está listo para implementar un dispositivo StorSimple en su centro de datos. Para más información, vea [Implementar un dispositivo local](storsimple-8000-deployment-walkthrough-u2.md).


---
title: Especificaciones técnicas y cumplimiento de Microsoft Azure Stack Edge Pro con GPU | Microsoft Docs
description: Conozca las especificaciones técnicas y el cumplimiento de un dispositivo Azure Stack Edge Pro con GPU
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 08/26/2020
ms.author: alkohli
ms.openlocfilehash: b0eaa9778480a6a767a4b37bd92a395d2b1ee6cb
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90899025"
---
# <a name="technical-specifications-and-compliance-for-azure-stack-edge-pro-with-gpu"></a>Especificaciones técnicas y cumplimiento de Azure Stack Edge Pro con GPU 

Los componentes de hardware de Azure Stack Edge Pro con una unidad de procesamiento gráfico (GPU) integrada cumplen con las especificaciones técnicas y los estándares reglamentarios descritos en este artículo. Las especificaciones técnicas describen el hardware, las fuentes de alimentación, la capacidad de almacenamiento, los revestimientos y las normas ambientales.

## <a name="compute-and-memory-specifications"></a>Especificaciones de memoria y proceso

El dispositivo Azure Stack Edge Pro tiene las siguientes especificaciones de proceso y memoria:

| Especificación           | Value                  |
|-------------------------|----------------------------|
| CPU                     | 2 CPU Intel Xeon Silver 4214 (Cascade Lake)            |
| Memoria                  | 128 (8x16 GB) GB de RAM                     |


## <a name="compute-acceleration-specifications"></a>Especificaciones de aceleración de proceso

En todos los dispositivos Azure Stack Edge Pro se incluye una unidad de procesamiento gráfico (GPU) que habilita escenarios de Kubernetes, aprendizaje profundo y aprendizaje automático.

| Especificación           | Value                  |
|-------------------------|----------------------------|
| GPU   | Una o dos GPU nVidia T4 <br> Para obtener más información, consulte [NVIDIA T4](https://www.nvidia.com/en-us/data-center/tesla-t4/).| 


## <a name="power-supply-unit-specifications"></a>Especificaciones de la fuente de alimentación

El dispositivo Azure Stack Edge Pro tiene dos fuentes de alimentación (PSU) de 100-240 V con ventiladores de alto rendimiento. Las dos fuentes de alimentación proporcionan una configuración de alimentación redundante. Si se produce un error en una de ellas, el dispositivo sigue funcionando con normalidad en la otra hasta que se reemplaza el módulo con error. En la tabla siguiente se enumeran las especificaciones técnicas de las fuentes de alimentación.

| Especificación           | Fuente de alimentación de 750 W                  |
|-------------------------|----------------------------|
| Potencia de salida máxima    | 750 W                     |
| Frecuencia               | 50/60 Hz                   |
| Selección del intervalo de voltaje | Intervalo automático: 100-240 V CA |
| Conectable en funcionamiento           | Sí                        |


## <a name="network-interface-specifications"></a>Especificaciones de la interfaz de red

Su dispositivo Azure Stack Edge Pro tiene seis interfaces de red, PORT1- PORT6.

| Especificación           | Descripción                 |
|-------------------------|----------------------------|
|  Interfaces de red    | **2 interfaces de 1 GbE**: Una interfaz de administración en el puerto 1 se usa para la configuración inicial y es estática de forma predeterminada. Una vez completada la configuración inicial, puede usar la interfaz para los datos con cualquier dirección IP. Sin embargo, tras un restablecimiento, la interfaz vuelve a la dirección IP estática. <br>La otra interfaz del puerto 2 se puede configurar por el usuario, se puede usar para la transferencia de datos y es DHCP de forma predeterminada. <br>**4 interfaces de 25 GbE**: El usuario puede configurar estas interfaces de datos, del puerto 3 al puerto 6, como DHCP (predeterminado) o estáticas. Estas pueden funcionar también como interfaces de 10 GbE.  | 

El dispositivo Azure Stack Edge Pro tiene el siguiente hardware de red:

* **Adaptador personalizado Microsoft QLogic Cavium 25 G NDC**: puertos 1 a 4.
* **Adaptador de red de 4 canales, doble puerto 25G ConnectX de Mellanox**: puertos 5 y 6.

Estos son los detalles de la tarjeta de Mellanox:

| Parámetro           | Descripción                 |
|-------------------------|----------------------------|
| Modelo    | Tarjeta de interfaz de red ConnectX®-4 Lx EN                      |
| Descripción del modelo               | SFP28, 25 GbE, doble puerto; PCIe3.0 x8; ROHS R6                    |
| Número de pieza del dispositivo (R640) | MCX4121A-ACAT  |
| PSID (R640)           | MT_2420110034                         |

Para una lista completa de cables, enchufes y transceptores para estas tarjetas de red, vaya a:

- [Matriz de interoperabilidad del adaptador Qlogic Cavium 25G NDC](https://www.marvell.com/documents/xalflardzafh32cfvi0z/).
- [Productos compatibles con el adaptador de red de 4 canales, doble puerto 25G ConnectX de Mellanox](https://docs.mellanox.com/display/ConnectX4LxFirmwarev14271016/Firmware+Compatible+Products).  

## <a name="storage-specifications"></a>Especificaciones de almacenamiento

Los dispositivos Azure Stack Edge Pro tienen cinco unidades de estado sólido DC P4610 de NVMe de 2,5 pulgadas, cada una con una capacidad de 1,6 TB. La unidad de arranque es una unidad de estado sólido SATA de 240 GB. La capacidad total utilizable para el dispositivo es de aproximadamente 8,28 TB. La tabla siguiente contiene la capacidad de almacenamiento del dispositivo.

|     Especificación                          |     Value             |
|--------------------------------------------|-----------------------|
|    Unidades de estado sólido (SSD) SATA de arranque      |    1                  |
|    Número de SSD de NVMe                     |    5                  |
|    Capacidad de SSD de arranque                       |    240 GB             |
|    Capacidad de SSD de NVMe única                |    1,6 TB             |
|    Capacidad total                          |    8,28 TB            |
|    Capacidad total utilizable*                  |    ~ 7,95 TB          |
|    Controladora SAS                          |    HBA330 12 Gbps     |


**Parte del espacio está reservado para uso interno.*

<!--Remove based on feedback from Ravi
## Other hardware specifications

Your Azure Stack Edge Pro device also contains the following hardware:

* iDRAC baseboard management
* Performance fans
* Custom ID module-->

## <a name="enclosure-dimensions-and-weight-specifications"></a>Especificaciones de dimensiones y peso del revestimiento

En las tablas siguientes se enumeran las diferentes especificaciones de dimensiones y peso del revestimiento.

### <a name="enclosure-dimensions"></a>Dimensiones del revestimiento

En la tabla siguiente se enumeran las dimensiones del revestimiento del dispositivo de 1U en milímetros y pulgadas.

|     Revestimiento     |     Milímetros     |     Pulgadas     |
|-------------------|---------------------|----------------|
|    Alto         |    44,45            |    1,75"       |
|    Ancho          |    434,1            |    17,09"      |
|    Length         |    740,4            |    29,15"      |

En la tabla siguiente se enumeran las dimensiones del paquete de envío en milímetros y pulgadas.

|     Paquete     |     Milímetros     |     Pulgadas     |
|-------------------|---------------------|----------------|
|    Alto         |    311,2            |    12,25"          |
|    Ancho          |    642,8            |    25,31"          |
|    Length         |    1051,1          |    41,38"          |

### <a name="enclosure-weight"></a>Peso del revestimiento

El paquete del dispositivo pesa 66 libras y requiere dos personas para manejarlo. El peso del dispositivo depende de la configuración del revestimiento.

|     Revestimiento                                 |     Peso          |
|-----------------------------------------------|---------------------|
|    Peso total, incluido el empaquetado       |    61 libras          |
|    Peso del dispositivo                       |    35 libras          |

## <a name="enclosure-environment-specifications"></a>Especificaciones medioambientales del revestimiento

En esta sección se enumeran las especificaciones relacionadas con el entorno del revestimiento, como la temperatura, la humedad y la altitud.

### <a name="temperature-and-humidity"></a>Temperatura y humedad

|     Revestimiento         |     Intervalo de temperatura ambiente     |     Humedad relativa del ambiente     |     Punto de rocío máximo     |
|-----------------------|--------------------------------------|--------------------------------------|---------------------------|
|    Operativos        |    10 °C a 35 °C (50 °F a 86 °F)         |    10 % - 80 % sin condensación         |    29 °C (84 °F)            |
|    No operativo    |    -40 °C a 65 °C (-40 °F a 149 °F)     |    5% - 95% sin condensación          |    33 °C (91 °F)            |

### <a name="airflow-altitude-shock-vibration-orientation-safety-and-emc"></a>Flujo de aire, altitud, golpes, vibraciones, orientación, seguridad y CEM

|     Revestimiento                           |     Especificaciones operativas                                                                                                                                                                                         |
|-----------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|    Flujo de aire                              |    El flujo de aire del sistema va de delante atrás. El sistema debe funcionar con una instalación de baja presión y escape trasero. <!--Back pressure created by rack doors and obstacles should not exceed 5 pascals (0.5 mm water gauge).-->    |
|    Altitud máxima (operativo)        |    3048 metros (10 000 pies) con la temperatura máxima de funcionamiento reducida determinada por las [especificaciones de reducción de la temperatura de funcionamiento](#operating-temperature-de-rating-specifications).                                                                                |
|    Altitud máxima (no operativo)    |    12 000 metros (39 370 pies)                                                                                                                                                                                         |
|    Golpes (operativo)                   |    6 G para 11 milisegundos en 6 orientaciones                                                                                                                                                                         |
|    Golpes (no operativo)               |    71 G para 2 milisegundos en 6 orientaciones                                                                                                                                                                           |
|    Vibraciones (operativo)               |    0,26 G<sub>RMS</sub> 5 Hz a 350 Hz aleatorio                                                                                                                                                                                     |
|    Vibraciones (no operativo)           |    1,88 G<sub>RMS</sub> 10 Hz a 500 Hz durante 15 minutos (los seis lados probados).                                                                                                                                                  |
|    Orientación y montaje             |    Montaje en bastidor estándar de 19" (1U)                                                                                                                                                                                       |
|    Seguridad y homologaciones                 |    EN 60950-1:2006 +A1:2010 +A2:2013 +A11:2009 +A12:2011/IEC 60950-1:2005 ed2 +A1:2009 +A2:2013 EN 62311:2008                                                                                                                                                                       |
|    CEM                                  |    FCC A, ICES-003 <br>EN 55032:2012/CISPR 32:2012  <br>EN 55032:2015/CISPR 32:2015  <br>EN 55024:2010 +A1:2015/CISPR 24:2010 +A1:2015  <br>EN 61000-3-2:2014/IEC 61000-3-2:2014 (Clase D)   <br>EN 61000-3-3:2013/IEC 61000-3-3:2013                                                                                                                                                                                         |
|    Sector energético             |    N.º de Reglamento de la Comisión (UE) 617/2013                                                                                                                                                                                        |
|    RoHS           |    EN 50581:2012                                                                                                                                                                                        |

### <a name="operating-temperature-de-rating-specifications"></a>Especificaciones de reducción de temperatura de funcionamiento

|     Reducción de temperatura de funcionamiento     |     Intervalo de temperatura ambiente                                                         |
|--------------------------------------------|------------------------------------------------------------------------------------------|
|    Hasta 35 °C (95 °F)                       |    La temperatura máxima se reduce en 1 °C/300 m (1 °F/547 pies) por encima de los 950 m (3117 pies).    |
|    35  C a 40  C (95  F a 104  F)            |    La temperatura máxima se reduce en 1 °C/175 m (1 °F/319 pies) por encima de los 950 m (3117 pies).    |
|    40  C a 45  C (104  F a 113  F)           |    La temperatura máxima se reduce en 1 °C/125 m (1 °F/228 pies) por encima de los 950 m (3117 pies).    |

## <a name="next-steps"></a>Pasos siguientes

[Implementación de Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-prep.md)

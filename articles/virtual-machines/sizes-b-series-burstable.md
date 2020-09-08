---
title: 'Serie B ampliable: Azure Virtual Machines'
description: Describe la serie B de tamaños de las máquinas virtuales de Azure ampliables.
services: virtual-machines
ms.subservice: sizes
author: ayshakeen
ms.service: virtual-machines
ms.topic: conceptual
ms.date: 02/03/2020
ms.author: ayshak
ms.openlocfilehash: a5a3887efe9b8f19608fac63216ca53576d9ec3f
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88654742"
---
# <a name="b-series-burstable-virtual-machine-sizes"></a>Tamaños de las máquinas virtuales ampliables serie B

Las máquinas virtuales de la serie B son idóneas para cargas de trabajo que no necesitan un rendimiento completo de la CPU de forma continua, como los servidores web, las pruebas de concepto, las pequeñas bases de datos y los entornos de compilación de desarrollo. Estas cargas de trabajo suelen necesitar unos requisitos de rendimiento ampliables. La serie B le brinda la posibilidad de adquirir un tamaño de máquina virtual con un rendimiento base al tiempo que la instancia de máquina virtual acumula créditos si utiliza un rendimiento por debajo de este nivel de base. Cuando la máquina virtual ha acumulado crédito se puede ampliar por encima de la base de referencia de esta con un uso de hasta un 100 % de la vCPU si la aplicación necesita el mayor rendimiento de CPU posible.

La serie B incluye los siguientes tamaños de máquina virtual:

Premium Storage:  Compatible

Almacenamiento en caché de Premium Storage:  No compatible

Migración en vivo: Compatible

Actualizaciones con conservación de memoria: Compatible

| Size | vCPU | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Rendimiento base de CPU de la máquina virtual | Rendimiento máximo de CPU de la máquina virtual | Créditos iniciales | Créditos ingresados/hora | Créditos máximos ingresados | Discos de datos máx. | Rendimiento máximo de almacenamiento temporal y en caché: IOPS/Mbps | Rendimiento máximo del disco sin almacenamiento en la caché: IOPS/Mbps | Nº máx. NIC |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Standard_B1ls<sup>1</sup> | 1  | 0.5 | 4   | 5 %   | 100%  | 30  | 3   | 72   | 2  | 200/10    | 160/10    | 2  |
| Standard_B1s              | 1  | 1   | 4   | 10 %  | 100%  | 30  | 6   | 144  | 2  | 400/10    | 320/10    | 2  |
| Standard_B1ms             | 1  | 2   | 4   | 20%  | 100%  | 30  | 12  | 288  | 2  | 800/10    | 640/10    | 2  |
| Standard_B2s              | 2  | 4   | 8   | 40%  | 200%  | 60  | 24  | 576  | 4  | 1600/15   | 1280/15   | 3  |
| Standard_B2ms             | 2  | 8   | 16  | 60%  | 200%  | 60  | 36  | 864  | 4  | 2400/22.5 | 1920/22.5 | 3  |
| Standard_B4ms             | 4  | 16  | 32  | 90%  | 400%  | 120 | 54  | 1296 | 8  | 3600/35   | 2880/35   | 4  |
| Standard_B8ms             | 8  | 32  | 64  | 135% | 800%  | 240 | 81  | 1944 | 16 | 4320/50   | 4320/50   | 4  |
| Standard_B12ms            | 12 | 48  | 96  | 202 % | 1200 % | 360 | 121 | 2909 | 16 | 6480/75   | 4320/50   | 6  |
| Standard_B16ms            | 16 | 64  | 128 | 270 % | 1600 % | 480 | 162 | 3888 | 32 | 8640/100  | 4320/50   | 8  |
| Standard_B20ms            | 20 | 80  | 160 | 337 % | 2000 % | 600 | 203 | 4860 | 32 | 10800/125 | 4320/50   | 8  |

<sup>1</sup> B1ls solo se admite en Linux

## <a name="workload-example"></a>Ejemplo de carga de trabajo

Piense en una aplicación de registro de entrada y salida de una oficina. La aplicación necesita ráfagas de CPU durante las horas laborales, pero no mucha capacidad de proceso durante horas de inactividad. En este ejemplo, la carga de trabajo requiere una máquina virtual de 16vCPU con 64 GiB de RAM para trabajar de forma eficaz.

La tabla muestra los datos de tráfico por hora y el gráfico es una representación visual de ese tráfico.

Características de B16:

Rendimiento máximo de CPU: 16vCPU * 100 % = 1600 %

Línea de base: 270 %

![Gráfico de los datos de tráfico por hora](./media/b-series-burstable/office-workload.png)

| Escenario | Time | Uso de CPU (%) | Créditos acumulados<sup>1</sup> | Créditos disponibles |
| --- | --- | --- | --- | --- |
| Implementación de B16ms | Implementación | Implementación  | 480 (créditos iniciales) | 480 |
| Sin tráfico | 0:00 | 0 | 162 | 642 |
| Sin tráfico | 1:00 | 0 | 162 | 804 |
| Sin tráfico | 2:00 | 0 | 162 | 966 |
| Sin tráfico | 3:00 | 0 | 162 | 1128 |
| Sin tráfico | 4:00 | 0 | 162 | 1290 |
| Sin tráfico | 5:00 | 0 | 162 | 1452 |
| Poco tráfico | 6:00 | 270 | 0 | 1452 |
| Los empleados llegan a la oficina (aplicación necesita el 80 % de vCPU) | 7:00 | 1280 | -606 | 846 |
| Los empleados siguen llegando a la oficina (aplicación necesita el 80 % de vCPU) | 8:00 | 1280 | -606 | 240 |
| Poco tráfico | 9:00 | 270 | 0 | 240 |
| Poco tráfico | 10:00 | 100 | 102 | 342 |
| Poco tráfico | 11:00 | 50 | 132 | 474 |
| Poco tráfico | 12:00 | 100 | 102 | 576 |
| Poco tráfico | 13:00 | 100 | 102 | 678 |
| Poco tráfico | 14:00 | 50 | 132 | 810 |
| Poco tráfico | 15:00 | 100 | 102 | 912 |
| Poco tráfico | 16:00 | 100 | 102 | 1014 |
| Los empleados registran la salida (la aplicación necesita el 100 % de vCPU) | 17:00 | 1600 | -798 | 216 |
| Poco tráfico | 18:00 | 270 | 0 | 216 |
| Poco tráfico | 19:00 | 270 | 0 | 216 |
| Poco tráfico | 20:00 | 50 | 132 | 348 |
| Poco tráfico | 21:00 | 50 | 132 | 480 |
| Sin tráfico | 22:00 | 0 | 162 | 642 |
| Sin tráfico | 23:00 | 0 | 162 | 804 |

<sup>1</sup> Créditos acumulados y créditos utilizados en una hora son equivalente a: `((Base CPU perf of VM - CPU Usage) / 100) * 60 minutes`.  

Para un D16s_v3 que tiene 16 vCPU y 64 GiB de memoria, la tarifa por hora es de 0,936 USD por hora (673,92 USD al mes) y para B16ms con 16 vCPU y 64 GiB de memoria, la tarifa es de 0,794 USD por hora (547,86 USD al mes). <b> ¡Esto da como resultado un ahorro del 15 %!</b>

## <a name="q--a"></a>Preguntas y respuestas

### <a name="q-what-happens-if-the-credits-run-out"></a>P: ¿Qué pasa si se acaban los créditos?
**R.** : Cuando se acaban los créditos, la VM vuelve al rendimiento de base de referencia.

### <a name="q-how-do-you-get-135-baseline-performance-from-a-vm"></a>P: ¿Cómo obtener un 135% del rendimiento base de una máquina virtual?

**R**: ese 135% se debe compartir entre las 8 vCPU que componen el tamaño de la máquina virtual. Por ejemplo, si la aplicación usa 4 de los 8 núcleos que están trabajando en el procesamiento por lotes y cada una de esas 4 vCPU se ejecuta al 30 % de uso, la cantidad total de rendimiento de la CPU de la máquina virtual equivaldría al 120 %.  Lo que significa que la máquina virtual podría acumular créditos basándose en este 15% de diferencia con el rendimiento base.  Pero también significa que cuando tenga créditos disponibles, esa misma máquina virtual puede utilizar el 100% de las 8 vCPU lo cual le proporcionaría a esa máquina virtual un rendimiento de CPU máximo del 800%.


### <a name="q-how-can-i-monitor-my-credit-balance-and-consumption"></a>P: ¿Cómo puedo supervisar mi saldo de crédito y consumo?

**R**: Presentaremos 2 nuevas métricas en las próximas semanas, la métrica **Credit** le permitirá ver cuántos créditos ha acumulado su máquina virtual y la métrica **ConsumedCredit** le mostrará cuántos créditos de la CPU ha consumido la máquina virtual desde el banco.    Podrá ver estas métricas desde el panel de métricas del portal o mediante programación a través de las API de Azure Monitor.

Para más información acerca de cómo acceder a los datos de las métricas de Azure, consulte [Información general sobre las métricas en Microsoft Azure](../azure-monitor/platform/data-platform.md).

### <a name="q-how-are-credits-accumulated-and-consumed"></a>P: ¿Cómo se acumulan y consumen los créditos?

**R**: Las tasas de acumulación y consumo de la máquina virtual se establecen de tal forma que una máquina virtual que se esté ejecutando exactamente a su nivel de rendimiento base, no tendrá una acumulación ni un consumo neto de créditos de ampliación.  Una máquina virtual tendrá un aumento neto en el número de créditos siempre que se esté ejecutando por debajo de su nivel de rendimiento base y tendrá una disminución neta cada vez que utilice la CPU por encima de este nivel.

**Ejemplo**: Quiero implementar una máquina virtual con el tamaño B1ms para mi aplicación de base de datos a la que se dedica poco tiempo y atención. Este tamaño permite que mi aplicación use hasta el 20 % de una vCPU como base de referencia, lo cual significa 0,2 créditos por minuto que puedo usar o acumular.

Mi aplicación está ocupada al principio y al final de la jornada laboral de mis empleados, es decir, de 7:00 a 9:00 A.M. y de 4:00 a 6:00 PM. Durante las otras 20 horas del día, la aplicación suele estar inactiva y solo usa el 10% de la vCPU. Durante las horas de poca actividad, gano 0,2 créditos por minuto y solo consumo 0,1 créditos por minuto, por lo que la máquina virtual acumulará 0,1 x 60 = 6 créditos por hora.  Es decir, durante las 20 horas de poca actividad acumularé 120 créditos.  

Durante las horas punta, la aplicación realiza un uso promedio del 60 % de la vCPU, sigo ganando 0,2 créditos por minuto pero consumo 0,6 créditos por minuto, lo cual supone un costo neto de 0,4 créditos por minuto o 0,4 x 60 = 24 créditos por hora. Si tengo 4 horas al día de uso máximo, eso significa 4 x 24 = 96 créditos de consumo al día.

Si tomo los 120 créditos que acumulé durante las horas de poca actividad y le resto los 96 créditos que utilicé durante las horas punta, obtendré 24 créditos adicionales que puedo acumular al día. Estos créditos los podré utilizar en otras ampliaciones de actividades.

### <a name="q-how-can-i-calculate-credits-accumulated-and-used"></a>P: ¿Cómo puedo calcular los créditos acumulados y usados?

**R.** : Puede usar la siguiente fórmula:

(Rendimiento de CPU base de una máquina virtual - Uso de CPU) / 100 = Créditos bancarios o uso por minuto

Por ejemplo, en la instancia anterior, su línea de base es del 20 % y si usa el 10 % de la CPU está acumulando (20 % -10 %)/100 = 0,1 créditos por minuto.

### <a name="q-does-the-b-series-support-premium-storage-data-disks"></a>P: ¿Admite la serie B discos de datos de Premium Storage?

**R**: Sí, toda la serie B admite discos de datos de Premium Storage.

### <a name="q-why-is-my-remaining-credit-set-to-0-after-a-redeploy-or-a-stopstart"></a>P: ¿Por qué tengo el crédito restante se establece en 0 después de volver a hacer una implementación o después de una detención o inicio?

**R**: Si una máquina virtual se "REIMPLEMENTA" y se mueve la máquina virtual a otro nodo, el crédito acumulado se pierde. En cambio, si la máquina virtual se detiene y se inicia, pero sigue en el mismo nodo, conservará el crédito acumulado. Cada vez que se inicia por primera vez una máquina virtual en un nodo, obtiene un crédito inicial. En el caso de Standard_B8ms es de 240 minutos.

### <a name="q-what-happens-if-i-deploy-an-unsupported-os-image-on-b1ls"></a>P: ¿Qué sucede si implemento una imagen de sistema operativo no admitida en B1ls?

**R** : B1ls solo admite imágenes de Linux y si implementa cualquier otra imagen de sistema operativo, podría no obtener la mejor experiencia de cliente.

## <a name="other-sizes-and-information"></a>Otros tamaños e información

- [Uso general](sizes-general.md)
- [Proceso optimizado](sizes-compute.md)
- [Memoria optimizada](sizes-memory.md)
- [Almacenamiento optimizado](sizes-storage.md)
- [GPU optimizada](sizes-gpu.md)
- [Proceso de alto rendimiento](sizes-hpc.md)

Calculadora de precios: [Calculadora de precios](https://azure.microsoft.com/pricing/calculator/)

Más información sobre los tipos de disco: [Tipos de disco](./disks-types.md#ultra-disk)

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cómo las [unidades de proceso de Azure (ACU)](acu.md) pueden ayudarlo a comparar el rendimiento en los distintos SKU de Azure.
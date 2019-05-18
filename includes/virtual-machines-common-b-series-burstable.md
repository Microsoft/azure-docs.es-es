---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: ayshakeen
ms.service: virtual-machines
ms.topic: include
ms.date: 03/09/2018
ms.author: azcspmt;ayshak;cynthn
ms.custom: include file
ms.openlocfilehash: ecf70bbbeae8fd68309f3343615f021038fb10b6
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65815503"
---
La familia de máquinas virtuales de la serie B le permite elegir qué tamaño de máquina virtual proporciona el rendimiento base necesario para su carga de trabajo, con la posibilidad de ampliar el rendimiento de la CPU hasta el 100 % de una vCPU con procesador Intel® Broadwell E5-2673 v4 a 2.3 GHz o Intel® Haswell 2.4 GHz E5-2673 v3.

Las máquinas virtuales de la serie B son idóneas para cargas de trabajo que no necesitan un rendimiento completo de la CPU de forma continua, como los servidores web, pruebas de concepto, pequeñas bases de datos y entornos de desarrollo y de prueba. Estas cargas de trabajo suelen necesitar unos requisitos de rendimiento ampliables. La serie B le brinda la posibilidad de adquirir un tamaño de máquina virtual con un rendimiento base al tiempo que la instancia de máquina virtual acumula créditos si utiliza un rendimiento por debajo de este nivel de base. Cuando la máquina virtual ha acumulado crédito se puede ampliar por encima de la base de referencia de esta con un uso de hasta un 100 % de la vCPU si la aplicación necesita el mayor rendimiento de CPU posible.

La serie B incluye los siguientes seis tamaños de máquina virtual:

| Tamaño             | vCPU  | Memoria: GiB | GiB de almacenamiento temporal (SSD) | Rendimiento base de CPU de la máquina virtual | Rendimiento máximo de CPU de la máquina virtual | Crédito inicial | Créditos ingresados / hora | Créditos máximos ingresados | Discos de datos máx. | Rendimiento máximo de almacenamiento temporal y en caché: IOPS / MBps | Rendimiento máximo del disco sin almacenamiento en la caché: IOPS / MBps | Nº máx. NIC |          
|---------------|-------------|----------------|----------------------------|-----------------------|--------------------|--------------------|--------------------|----------------|----------------------------------------|-------------------------------------------|-------------------------------------------|----------|
| Standard_B1ls<sup>1</sup>  | 1           | 0,5              | 4                          | 5 %                   | 100%                   | 30                   | 3                  | 72            | 2                                      | 200 / 10                                  | 160 / 10                                  | 2  |
| Standard_B1s  | 1           | 1              | 4                          | 10%                   | 100%                   | 30                   | 6                  | 144            | 2                        | 400 / 10                                  | 320 / 10                                  | 2  |
| Standard_B1ms | 1           | 2              | 4                          | 20%                   | 100%                   | 30                   | 12                 | 288           | 2                         | 800 / 10                                  | 640 / 10                                  | 2  |
| Standard_B2s  | 2           | 4              | 8                          | 40%                   | 200%                   | 60                   | 24                 | 576            | 4                                      | 1600 / 15                                 | 1280 / 15                                 | 3  |
| Standard_B2ms | 2           | 8              | 16                         | 60%                   | 200%                   | 60                   | 36                 | 864            | 4                                      | 2400 / 22.5                               | 1920 / 22.5                               | 3  |
| Standard_B4ms | 4           | 16             | 32                         | 90%                   | 400%                   | 120                   | 54                 | 1296           | 8                                      | 3600 / 35                                 | 2880 / 35                                 | 4  |
| Standard_B8ms | 8           | 32             | 64                         | 135%                  | 800%                   | 240                   | 81                 | 1944           | 16                                     | 4320 / 50                                 | 4320 / 50                                 | 4  |

<sup>1</sup> B1ls solo se admite en Linux

## <a name="q--a"></a>Preguntas y respuestas 

### <a name="q-how-do-you-get-135-baseline-performance-from-a-vm"></a>P: ¿Cómo se consigue 135% del rendimiento base desde una máquina virtual?
**R.**: Ese 135% se debe compartir entre las 8 vCPU que componen el tamaño de máquina virtual. Por ejemplo, si la aplicación usa 4 de los 8 núcleos que están trabajando en el procesamiento por lotes y cada una de esas 4 vCPU se ejecuta al 30 % de uso, la cantidad total de rendimiento de la CPU de la máquina virtual equivaldría al 120 %.  Lo que significa que la máquina virtual podría acumular créditos basándose en este 15% de diferencia con el rendimiento base.  Pero también significa que cuando tenga créditos disponibles, esa misma máquina virtual puede utilizar el 100% de las 8 vCPU lo cual le proporcionaría a esa máquina virtual un rendimiento de CPU máximo del 800%.


### <a name="q-how-can-i-monitor-my-credit-balance-and-consumption"></a>P: ¿Cómo puedo supervisar mi saldo de crédito y el consumo
**R.**: Presentaremos 2 nuevas métricas en las próximas semanas, el **crédito** métrica le permitirá ver cuántos créditos ha acumulado la máquina virtual y el **ConsumedCredit** métrica mostrará cuántos créditos de CPU, la máquina virtual tiene consumir desde el banco.    Podrá ver estas métricas desde el panel de métricas del portal o mediante programación a través de las API de Azure Monitor.

Para más información acerca de cómo acceder a los datos de las métricas de Azure, consulte [Información general sobre las métricas en Microsoft Azure](../articles/monitoring-and-diagnostics/monitoring-overview-metrics.md).

### <a name="q-how-are-credits-accumulated"></a>P: ¿Cómo se acumulan los créditos?
**R.**: Las tasas de acumulación y consumo de la máquina virtual se establecen de forma que tendrá una máquina virtual que se ejecuta en exactamente su nivel de rendimiento base ni una acumulación neto o consumo de créditos de ampliación.  Una máquina virtual tendrá un aumento neto en el número de créditos siempre que se esté ejecutando por debajo de su nivel de rendimiento base y tendrá una disminución neta cada vez que utilice la CPU por encima de este nivel.

**Ejemplo**:  Implementar una máquina virtual con el tamaño B1ms para mi poco tiempo y la aplicación de base de datos de asistencia. Este tamaño permite que mi aplicación use hasta el 20 % de una vCPU como base de referencia, lo cual significa 0,2 créditos por minuto que puedo usar o acumular. 

Mi aplicación está ocupada al principio y al final de la jornada laboral de mis empleados, es decir, de 7:00 a 9:00 A.M. y de 4:00 a 6:00 PM. Durante las otras 20 horas del día, la aplicación suele estar inactiva y solo usa el 10% de la vCPU. Durante las horas de poca actividad, gano 0,2 créditos por minuto y solo consumo 0,1 créditos por minuto, por lo que la máquina virtual acumulará 0,1 x 60 = 6 créditos por hora.  Es decir, durante las 20 horas de poca actividad acumularé 120 créditos.  

Durante las horas punta, la aplicación realiza un uso promedio del 60 % de la vCPU, sigo ganando 0,2 créditos por minuto pero consumo 0,6 créditos por minuto, lo cual supone un costo neto de 0,4 créditos por minuto o 0,4 x 60 = 24 créditos por hora. Si tengo 4 horas al día de uso máximo, eso significa 4 x 24 = 96 créditos de consumo al día.

Si tomo los 120 créditos que acumulé durante las horas de poca actividad y le resto los 96 créditos que utilicé durante las horas punta, obtendré 24 créditos adicionales que puedo acumular al día. Estos créditos los podré utilizar en otras ampliaciones de actividades.


### <a name="q-does-the-b-series-support-premium-storage-data-disks"></a>P: ¿La serie B admite discos de datos de Premium Storage?
**R.**: Sí, todos los tamaños de la serie B admiten discos de datos de Premium Storage.   
    
### <a name="q-why-is-my-remaining-credit-set-to-0-after-a-redeploy-or-a-stopstart"></a>P: ¿Por qué se establece mi crédito restante en 0 después de una nueva implementación o un detención e inicio?
**UN** : Cuando una máquina virtual es "REDPLOYED" y la máquina virtual se mueve a otro nodo, se pierde el crédito acumulado. En cambio, si la máquina virtual se detiene y se inicia, pero sigue en el mismo nodo, conservará el crédito acumulado. Cada vez que se inicia por primera vez una máquina virtual en un nodo, obtiene un crédito inicial. En el caso de Standard_B8ms es de 240 minutos.
    
### <a name="q-what-happens-if-i-deploy-an-unsupported-os-image-on-b1ls"></a>P: ¿Qué sucede si implemento una imagen de sistema operativo no compatible en B1ls?
**UN** : B1ls sólo admite imágenes de Linux y si implementa cualquier otra imagen de sistema operativo no puede obtener la mejor experiencia del cliente.
    
### <a name="q-why-is-there-no-pricing-information-for-b1ls-windows"></a>P: ¿Por qué no hay ninguna información de precios para windows B1ls?
**UN** : B1ls sólo admite imágenes de Linux y si implementa cualquier otra imagen de sistema operativo no puede obtener la mejor experiencia del cliente, pero se le cobrará.


    

    

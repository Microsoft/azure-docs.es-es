---
title: Planeamiento de la capacidad para la recuperación ante desastres de Hyper-V con Azure Site Recovery
description: Use este artículo para calcular la capacidad al configurar la recuperación ante desastres con el servicio Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
services: site-recovery
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/12/2019
ms.author: raynew
ms.openlocfilehash: a5764e44db31755110ac99a3e8e8e0984cdf9604
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87490581"
---
# <a name="plan-capacity-for-hyper-v-vm-disaster-recovery"></a>Planeamiento de la capacidad para la recuperación ante desastres de máquinas virtuales de Hyper-V 

[Azure Site Recovery Deployment Planner] (site-recovery-hyper-v-deployment-planner.md) para la implementación de Hyper-V en Azure proporciona lo siguiente:

* Evaluación de la idoneidad de las máquinas virtuales en función del número de discos, el tamaño de estos, las IOPS, la actividad de datos y algunas otras características de las máquinas virtuales.
* Necesidad de ancho de banda de red frente a evaluación de RPO.
* Requisitos de infraestructura de Azure.
* Requisitos de la infraestructura local.
* Guía para el procesamiento por lotes de la replicación inicial.
* Costo total estimado de recuperación ante desastres en Azure.


Azure Site Recovery Capacity Planner le ayuda a determinar los requisitos de capacidad al replicar máquinas virtuales de Hyper-V con Azure Site Recovery.

Use Site Recovery Capacity Planner para analizar su entorno de origen y las cargas de trabajo. Esta herramienta le ayuda a calcular las necesidades de ancho de banda, los recursos de servidor que necesita para la ubicación de origen y los recursos (por ejemplo, las máquinas virtuales y el almacenamiento) necesarios en la ubicación de destino.

Puede ejecutar la herramienta de dos modos distintos:

* **Planeamiento rápido**: proporciona las proyecciones de red y servidor según el número medio de máquinas virtuales, discos, almacenamiento y tasa de cambio.
* **Planeamiento detallado**: proporciona detalles de cada carga de trabajo en el nivel de máquina virtual. Analice la compatibilidad de la máquina virtual y obtenga proyecciones de red y del servidor.

## <a name="before-you-start"></a>Antes de comenzar

* Recopile información sobre su entorno, incluidas las máquinas virtuales, discos por máquina virtual y almacenamiento por disco.
* Identifique la tasa de cambio (renovación) diaria para los datos replicados. Descargue la [herramienta de planeamiento de la capacidad de Hyper-V](https://www.microsoft.com/download/details.aspx?id=39057) para obtener la tasa de cambio. [Más información](./hyper-v-deployment-planner-overview.md) sobre esta herramienta. Se recomienda ejecutar esta herramienta durante una semana para capturar los promedios.


## <a name="run-the-quick-planner"></a>Ejecución de Quick Planner
1. Descargue y abra [Site Recovery Capacity Planner](https://aka.ms/asr-capacity-planner-excel). Tendrá que ejecutar macros. Cuando se le pida, realice las selecciones para habilitar la edición y el contenido.

2. En **Select a planner type** (Seleccione un tipo de planeador), seleccione **Quick Planner** (Planificador rápido) en el cuadro de lista.

   ![Captura de pantalla de la opción para seleccionar un tipo de planificador, con el planificador rápido seleccionado.](./media/site-recovery-capacity-planner/getting-started.png)

3. En la hoja de cálculo **Capacity Planner**, escriba la información que necesite. Rellene todos los campos rodeados por un círculo rojo de la captura de pantalla siguiente:

   a. En **Select your scenario** (Seleccionar escenario) elija **Hyper-V to Azure** (Hyper-V a Azure) o **VMware/Physical to Azure** (VMWare/Físico a Azure).

   b. En **Average daily data change rate (%)** [Tasa media diaria de cambio de datos (%)], escriba la información que ha recopilado con la [herramienta de planeamiento de capacidad de Hyper-V](./hyper-v-deployment-planner-overview.md) o con [Site Recovery Deployment Planner](./site-recovery-deployment-planner.md).

   c. El valor de **compresión** no se utiliza al replicar máquinas virtuales de Hyper-V en Azure. Para la compresión, use un dispositivo de terceros como Riverbed.

   d. En **Retention in days** (Retención en días), especifique en días durante cuánto tiempo desea conservar réplicas.

   e. En **Number of hours in which initial replication for the batch of virtual machines should complete** (Número de horas en las que debe completarse la replicación inicial para el lote de máquinas virtuales) y **Number of virtual machines per initial replication batch** (Número de máquinas virtuales por lote de replicación inicial) especifique la configuración que se ha usado para procesar los requisitos de replicación iniciales. Cuando se implementa Site Recovery, se carga el conjunto de datos inicial completo.

   ![Captura de pantalla de la hoja de cálculo de Capacity Planner, que muestra la información de entrada necesaria.](./media/site-recovery-capacity-planner/inputs.png)

4. Después de haber especificado los valores para el entorno de origen, la salida mostrada incluye:

   * **Bandwidth required for delta replication (in Megabits/sec)** (Ancho de banda necesario para la replicación diferencial [en Megabits por segundo]): El ancho de banda de red necesario para la replicación diferencial se calcula según la tasa media diaria de cambio de datos.
   * **Bandwidth required for initial replication (in Megabits/sec)** (Ancho de banda necesario para la replicación inicial [en Megabits por segundo]): el ancho de banda de red para la replicación inicial se calcula según los valores de replicación iniciales establecidos.
   * **Storage required (in GBs)** (Almacenamiento necesario [en GB]): almacenamiento total de Azure necesario.
   * **Total IOPS on Standard Storage** (IOPS total en cuentas de almacenamiento estándar): el número se calcula en función de un tamaño de unidad IOPS de 8 K en las cuentas de almacenamiento estándar total. Para Quick Planner, el número se calcula en función de todos los discos de máquinas virtuales de origen y la tasa de cambio de los datos diarios. Para Detailed Planner el número se calcula en función del número total de máquinas virtuales que se asignan a las máquinas virtuales estándar de Azure y a la tasa de cambio de los datos en dichas máquinas virtuales.
   * **Number of Standard storage accounts required** (Número de cuentas de almacenamiento estándar necesarias): Número total de cuentas de almacenamiento estándar necesarias para proteger las VM. Una cuenta de almacenamiento estándar puede contener hasta 20 000 IOPS en todas las máquinas virtuales de un almacenamiento estándar. Se admite un máximo de 500 IOPS por disco.
   * **Number of Blob disks required** (Número de discos de blob necesarios): Número de discos que se crearán en Azure Storage.
   * **Number of premium accounts required** (Número de cuentas premium necesarias): Número total de cuentas de almacenamiento premium necesarias para proteger las VM. Una máquina virtual de origen con una IOPS elevada (más de 20 000) necesita una cuenta de Premium Storage. Una cuenta de Premium Storage puede contener hasta 80 000 IOPS.
   * **Total IOPS on Premium Storage** (IOPS total en Premium Storage): el número se calcula en función del tamaño de unidad IOPS de 256 K en las cuentas de Premium Storage totales. Para Quick Planner, el número se calcula en función de todos los discos de máquinas virtuales de origen y la tasa de cambio de los datos diarios. Para Detailed Planner el número se calcula en función del número total de máquinas virtuales que se asignan a las máquinas virtuales premium de Azure (serie DS y GS) y a la tasa de cambio de los datos en dichas máquinas virtuales.
   * **Number of Configuration Servers required** (Número de servidores de configuración necesarios): muestra cuántos servidores de configuración son necesarios para la implementación.
   * **Number of additional Process Servers required** (Número de servidores de proceso adicionales necesarios): Muestra si se requieren servidores de procesos adicionales, además del servidor de proceso que se ejecuta en el servidor de configuración de forma predeterminada.
   * **100% additional storage on the Source** (100 % de almacenamiento adicional en el origen): Muestra si se necesita almacenamiento adicional en la ubicación de origen.

      ![Captura de pantalla de la salida mostrada en función de la entrada proporcionada.](./media/site-recovery-capacity-planner/output.png)

## <a name="run-the-detailed-planner"></a>Ejecución de Detailed Planner

1. Descargue y abra [Site Recovery Capacity Planner](https://aka.ms/asr-capacity-planner-excel). Tendrá que ejecutar macros. Cuando se le pida, realice las selecciones para habilitar la edición y el contenido.

2. En **Seleccionar un tipo de planificador**, elija **Planificador detallado** en el cuadro de lista.

   ![Captura de pantalla de la opción para seleccionar un tipo de planificador, con el planificador detallado seleccionado.](./media/site-recovery-capacity-planner/getting-started-2.png)

3. En la hoja de cálculo **Workload Qualification**, escriba la información necesaria. Debe rellenar todos los campos marcados.

   a. En **Processor Cores** (Núcleos de procesador), especifique el número total de núcleos de un servidor de origen.

   b. En **Memory allocation (in MBs)** [Asignación de memoria (en MB)], especifique el tamaño de la RAM de un servidor de origen.

   c. En el valor **Number of NICs** (Número de NIC), especifique el número de adaptadores de red de un servidor de origen.

   d. En **Total Storage (in GB)** [Almacenamiento total (en GB)], especifique el tamaño total del almacenamiento de la máquina virtual. Por ejemplo, si el servidor de origen tiene tres discos con 500 GB cada uno, el tamaño de almacenamiento total es de 1500 GB.

   e. En **Número de discos conectados**, especifique el número total de discos de un servidor de origen.

   f. En **Disk capacity utilization (%)** [Utilización de la capacidad de disco (%)], especifique el promedio de uso.

   g. En **Daily data change rate (%)** [Tasa de cambios diaria (%)], especifique la tasa de cambios de datos diaria de un servidor de origen.

   h. En **Mapping Azure VM size** (Tamaño de asignación de Azure), escriba el tamaño de la máquina virtual de Azure que desea asignar. Si no desea hacerlo manualmente, seleccione **Compute IaaS VMs** (Procesar máquinas virtuales de IaaS). Si escribe un valor manual y, a continuación, selecciona **Compute IaaS VMs** (Procesar máquinas virtuales de IaaS), es posible que se sobrescriba el valor manual. El proceso identifica automáticamente la mejor coincidencia de tamaño de máquina virtual de Azure.

   ![Captura de pantalla de la hoja de cálculo de la calificación de carga de trabajo, que muestra la información de entrada necesaria.](./media/site-recovery-capacity-planner/workload-qualification.png)

4. Al seleccionar **Compute IaaS VMs** (Procesar máquinas virtuales de IaaS):

   * Se validan las entradas obligatorias.
   * Se calculan las IOPS y se sugiere la mejor coincidencia de tamaño de máquina virtual de Azure para cada máquina virtual apta para la replicación en Azure. Se muestra un error si no se detecta un tamaño adecuado para la máquina virtual de Azure. Por ejemplo, si el número de discos conectados es 65, se muestra un error ya que el tamaño máximo de una máquina virtual de Azure es 64.
   * Sugiere una cuenta de almacenamiento que puede usar para una máquina virtual de Azure.
   * Calcula el número total de cuentas de almacenamiento estándar y de cuentas de almacenamiento premium que son necesarias para la carga de trabajo. Desplácese hacia abajo para ver el tipo de almacenamiento de Azure y la cuenta de almacenamiento que puede usar para un servidor de origen.
   * Completa y ordena el resto de la tabla basándose en el tipo de almacenamiento necesario (estándar o premium) asignado a una máquina virtual y el número de discos conectados. En todas las máquinas virtuales que cumplen los requisitos de Azure, la columna **Is VM qualified?** (¿Está calificada la máquina virtual?) muestra **Yes** (Sí). Si no se puede realizar una copia de seguridad de la máquina virtual en Azure, se muestra un error.

Las columnas AA hasta AE muestran los resultados y proporcionan información de cada máquina virtual.

![Captura de pantalla que muestra las columnas de salida de AA a AE.](./media/site-recovery-capacity-planner/workload-qualification-2.png)

### <a name="example"></a>Ejemplo
Como ejemplo, para seis máquinas virtuales con los valores que se muestran en la tabla, la herramienta calcula y asigna la mejor coincidencia de máquina virtual de Azure y los requisitos de almacenamiento de Azure.

![Captura de pantalla que muestra las asignaciones de la calificación de carga de trabajo.](./media/site-recovery-capacity-planner/workload-qualification-3.png)

* En el resultado del ejemplo, tenga en cuenta lo siguiente:

  * La primera columna es una columna de validación para las máquinas virtuales, discos y renovación.
  * Se requieren dos cuentas de almacenamiento estándar y una cuenta de almacenamiento premium para las cinco máquinas virtuales.
  * VM3 no cumple los requisitos para la protección porque uno o más discos tienen más de 1 TB.
  * VM1 y VM2 pueden usar la primera cuenta de almacenamiento estándar
  * VM4 puede usar la segunda cuenta de almacenamiento estándar
  * VM5 y VM6 necesitan una cuenta de Premium Storage y ambas pueden usar una única cuenta.

    > [!NOTE]
    > Las IOPS de almacenamiento estándar y premium se calculan en el nivel de máquina virtual y no en el nivel de disco. Una máquina virtual estándar puede controlar hasta 500 IOPS por disco. Si las IOPS de un disco son más de 500, necesitará Premium Storage. Si las IOPS de un disco son más de 500, pero las IOPS de todos los discos de máquina virtual están dentro de los límites admitidos para máquinas virtuales de Azure estándar, el planificador elige una máquina virtual estándar en lugar de la serie DS o GS (los límites de las máquinas virtual de Azure son el tamaño de la máquina virtual, el número de discos, el número de adaptadores, la CPU y la memoria). El usuario debe actualizar manualmente la asignación de la celda de tamaño de Azure con la máquina virtual de la serie DS o GS correspondiente.


Después de especificar toda la información, seleccione **Submit data to the planner tool** (Enviar datos a la herramienta de planeación) para abrir Capacity Planner. Las cargas de trabajo se resaltan para mostrar si son aptas para protegerse.

### <a name="submit-data-in-capacity-planner"></a>Envío de datos en Capacity Planner
1. Cuando se abre la hoja de cálculo **Capacity Planner**, esta se rellena en función de la configuración que haya especificado. La palabra «Workload» aparece en la celda **Infra inputs source** (Origen de entradas de infraestructura) para indicar que la entrada es la hoja de cálculo **Workload Qualification**.

2. Si desea realizar cambios, deberá modificar la hoja de cálculo **Workload Qualification**. A continuación, vuelva a seleccionar **Submit data to the planner tool** (Enviar datos a la herramienta de planeación).

   ![Captura de pantalla que muestra las entradas modificadas y las salidas resultantes en la hoja de cálculo de Capacity Planner.](./media/site-recovery-capacity-planner/capacity-planner.png)

## <a name="next-steps"></a>Pasos siguientes
[Aprenda a ejecutar](./hyper-v-deployment-planner-overview.md) la herramienta de planeamiento de capacidad.

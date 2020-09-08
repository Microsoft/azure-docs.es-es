---
title: Análisis del informe de Deployment Planner para la recuperación ante desastres de VMware con Azure Site Recovery
description: En este artículo se describe cómo analizar un informe que haya generado Deployment Planner para la recuperación ante desastres de VMware en Azure, mediante Azure Site Recovery.
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/4/2019
ms.author: mayg
ms.openlocfilehash: 901f4a9d4fd53f665c3d078f5e463dcde2af1882
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88654878"
---
# <a name="analyze-the-deployment-planner-report-for-vmware-disaster-recovery-to-azure"></a>Análisis del informe de Deployment Planner para la recuperación ante desastres de VMware en Azure

El informe de Microsoft Excel generado contiene las siguientes hojas:
## <a name="on-premises-summary"></a>Resumen local
La hoja de cálculo de resumen local proporciona una introducción al entorno de VMware en el que se han generado perfiles.

![Resumen local del entorno de VMware](media/site-recovery-vmware-deployment-planner-analyze-report/on-premises-summary-v2a.png)

**Start Date** (Fecha de inicio) y **End Date** (Fecha de finalización): las fechas de inicio y finalización de los datos de generación de perfiles que se tienen en cuenta para la generación del informe. De forma predeterminada, la fecha de inicio es la fecha en que comienza la generación de perfiles, mientras que la de finalización es la fecha en la que se detiene. Esta se puede especificar mediante los valores "StartDate" y "EndDate" si el informe se genera con estos parámetros.

**Total number of profiling days** (Número total de días de generación de perfiles): el número total de días de generación de perfiles comprendido entre las fechas de inicio y de finalización para el que se genera el informe.

**Number of compatible virtual machines** (Número de máquinas virtuales compatibles): el número total de máquinas virtuales compatibles para el que se calculan el ancho de banda de red necesario, el número de cuentas de almacenamiento requeridas, los núcleos de Microsoft Azure, los servidores de configuración y los servidores de proceso adicionales.

**Total number of disks across all compatible virtual machines** (Número total de discos en todas las máquinas virtuales compatibles): este número se usa como una de las entradas para decidir el número de servidores de configuración y servidores de procesos adicionales que se usarán en la implementación.

**Average number of disks per compatible virtual machine** (Número medio de discos en cada máquina virtual compatible): el número medio de discos calculado para todas las máquinas virtuales compatibles.

**Average disk size (GB)** (Tamaño medio de disco [GB]): el tamaño medio de disco calculado en todas las máquinas virtuales compatibles.

**Desired RPO (minutes)** (RPO deseado [en minutos]): el objetivo de punto de recuperación predeterminado o el valor pasado para el parámetro "DesiredRPO" en el momento de la generación de informes para calcular el ancho de banda necesario.

**Desired bandwidth (Mbps)** (Ancho de banda deseado [Mbps]): el valor que ha pasado para el parámetro "Bandwidth" en el momento de la generación de informes para calcular el RPO factible.

**Observed typical data churn per day (GB)** (Renovación de datos normal observada por día [GB]): el valor medio de renovación de datos observada en todos los días de generación de perfiles. Este número se utiliza como una de las entradas para decidir el número de servidores de configuración y servidores de procesos adicionales que se usarán en la implementación.

## <a name="recommendations"></a>Recomendaciones

La hoja de recomendaciones del informe de VMware a Azure tiene los siguientes detalles según el RPO deseado seleccionado:

![Recomendaciones para el informe de VMware a Azure](media/site-recovery-vmware-deployment-planner-analyze-report/Recommendations-v2a.png)

### <a name="profiled-data"></a>Datos de generación de perfiles
![La vista de datos de generación de perfiles en Deployment Planner](media/site-recovery-vmware-deployment-planner-analyze-report/profiled-data-v2a.png)

**Profiled data period** (Periodo de datos de generación de perfiles): el período durante el que se ejecutó la generación de perfiles. De manera predeterminada, la herramienta incluye todos los datos de generación de perfiles en el cálculo, a menos que genere el informe para un período específico mediante las opciones StartDate y EndDate durante la generación de informes.

**Server Name** (Nombre del servidor): el nombre o la dirección IP del servidor VMware vCenter o host de ESXi cuyo informe de máquinas virtuales se genera.

**Desired RPO** (RPO deseado): el objetivo de punto de recuperación de la implementación. De manera predeterminada, se calcula el ancho de banda de red necesario para los valores de RPO de 15, 30 y 60 minutos. En función de la selección, los valores afectados se actualizan en la hoja. Si ha usado el parámetro *DesiredRPOinMin* al generar el informe, ese valor se muestra en el resultado de RPO deseado.

### <a name="profiling-overview"></a>Información general sobre la generación de perfiles

![Resultados de generación de perfiles en Deployment Planner](media/site-recovery-vmware-deployment-planner-analyze-report/profiling-overview-v2a.png)

**Total Profiled Virtual Machines** (Total de máquinas virtuales de las que se ha generado el perfil): el número total de las máquinas virtuales cuyos datos de generación de perfiles están disponibles. Si VMListFile contiene los nombres de las máquinas virtuales de las que no se ha generado el perfil, dichas máquinas virtuales no se tienen en cuenta en la generación de informes y se excluyen del número total de máquinas virtuales de las que se ha generado el perfil.

**Compatible Virtual Machines** (Máquinas virtuales compatibles): número de máquinas virtuales que se pueden proteger en Azure mediante Site Recovery. Es el número total de máquinas virtuales compatibles para el que se calculan el ancho de banda de red, el número de cuentas de almacenamiento, el número de núcleos de Azure y el número de servidores de configuración y de servidores de proceso adicionales que se requieren. Los detalles de todas las máquinas virtuales compatibles están disponibles en la sección "Máquinas virtuales compatibles".

**Incompatible Virtual Machines** (Máquinas virtuales no compatibles): el número de máquinas virtuales de las que se ha generado el perfil que no son compatibles para la protección con Site Recovery. Los motivos de dicha incompatibilidad se indican en la sección "Máquinas virtuales no compatibles". Si VMListFile contiene los nombres de las máquinas virtuales cuyo perfil no se ha generado, dichas máquinas se excluyen del número de máquinas virtuales no compatibles. Estas máquinas aparecen como "Datos no encontrados" al final de la sección "Máquinas virtuales no compatibles".

**Desired RPO** (RPO deseado): el objetivo de punto de recuperación deseado, en minutos. El informe se genera para tres valores de RPO: 15 (valor predeterminado), 30 y 60 minutos. La recomendación que se hace sobre el ancho de banda en el informe se cambia en función de la selección que se realice en la lista desplegable de Desired RPO (RPO deseado) de la parte superior derecha de la hoja. Si el informe se ha generado mediante el parámetro *-DesiredRPO* con un valor personalizado, dicho valor personalizado se mostrará como valor predeterminado en la lista desplegable Desired RPO (RPO deseado).

### <a name="required-network-bandwidth-mbps"></a>Ancho de banda de red requerido (Mbps)

![Ancho de banda de red requerido en Deployment Planner](media/site-recovery-vmware-deployment-planner-analyze-report/required-network-bandwidth-v2a.png)

**To meet RPO 100 percent of the time** (Para cubrir el RPO el 100 % del tiempo): el ancho de banda recomendado, en Mbps, que se debe asignar para satisfacer el RPO en el 100 % de las ocasiones. Esta cantidad de ancho de banda debe estar dedicada a la replicación diferencial de estado estable de todas las máquinas virtuales compatibles para evitar infracciones de RPO.

**To meet RPO 90 percent of the time** (Para cubrir el RPO el 90 por ciento del tiempo): a causa de los precios del ancho de banda o por cualquier otra razón, si no puede establecer el ancho de banda necesario para cubrir el RPO deseado en todo momento, puede elegir usar un valor de ancho de banda inferior que le permita satisfacer el RPO deseado en el 90 % de las ocasiones. Para conocer las implicaciones de establecer un ancho de banda inferior, el informe proporciona el análisis de hipótesis del número previsible de infracciones de RPO y su duración.

**Achieved Throughput** (Rendimiento conseguido): el rendimiento del servidor en el que ha ejecutado el comando GetThroughput para la región de Microsoft Azure en la que se encuentra la cuenta de almacenamiento. Este número de rendimiento indica el nivel estimado que se puede lograr al proteger las máquinas virtuales compatibles mediante Site Recovery, siempre que las características de red y de almacenamiento tanto del servidor de configuración como del servidor de procesos sean las mismas que las del servidor en el que se ha ejecutado la herramienta.

En la replicación, debe establecer el ancho de banda recomendado de forma que cubra el RPO el 100 % del tiempo. Después de establecer el ancho de banda, si no percibe un aumento en el rendimiento obtenido que notifica la herramienta, siga estos pasos:

1. Realice las comprobaciones necesarias para ver si hay algún requisito de calidad de servicio (QoS) de la red que limite el rendimiento de Site Recovery.

2. Realice las comprobaciones necesarias para ver si el almacén de Site Recovery está en la región física de Microsoft Azure más cercana admitida para minimizar la latencia de red.

3. Compruebe las características del almacenamiento local para determinar si puede mejorar el hardware (por ejemplo, pasar de HDD a SSD).

4. Cambie la configuración de Site Recovery en el servidor de procesos para [aumentar la cantidad de ancho de banda de red que se usa para la replicación](./site-recovery-plan-capacity-vmware.md#control-network-bandwidth).

Si ejecuta la herramienta en un servidor de configuración o de procesos que ya tiene máquinas virtuales protegidas, ejecute la herramienta varias veces. El número de rendimiento logrado cambia en función de la cantidad de activación que se procesa en ese momento.

Para todas las implementaciones de Site Recovery que se realicen en la empresa, se recomienda usar [ExpressRoute](https://aka.ms/expressroute).

### <a name="required-storage-accounts"></a>Cuentas de almacenamiento requeridas
El siguiente gráfico muestra el número total de cuentas de almacenamiento (Estándar y Premium) que se requieren para proteger todas las máquinas virtuales compatibles. Para saber qué cuenta de almacenamiento se debe usar para cada máquina virtual, consulte la sección "selección de ubicación de almacenamiento de máquina virtual". Si usa la versión 2.5 de Deployment Planner, esta recomendación solo muestra el número de cuentas de almacenamiento en caché estándar que se necesitan para la replicación, ya que los datos se escriben directamente en Managed Disks.

![Cuentas de almacenamiento requeridas en Deployment Planner](media/site-recovery-vmware-deployment-planner-analyze-report/required-storage-accounts-v2a.png)

### <a name="required-number-of-azure-cores"></a>Número de núcleos de Azure requeridos
Este resultado es el número total de núcleos que se deben configurar antes de realizar una conmutación por error real o de prueba de todas las máquinas virtuales compatibles. Si hay muy pocos núcleos en la suscripción, Site Recovery no puede crear las máquinas virtuales en el momento de realizar la conmutación por error real o de prueba.

![Número de núcleos de Azure requeridos en Deployment Planner](media/site-recovery-vmware-deployment-planner-analyze-report/required-cores-v2a.png)

### <a name="required-on-premises-infrastructure"></a>Infraestructura local requerida
Esta cifra es el número total de servidores de configuración y servidores de proceso adicionales configurables que serían suficientes para proteger todas las máquinas virtuales compatibles. En función de las [recomendaciones de tamaño para el servidor de configuración](https://aka.ms/asr-v2a-on-prem-components) admitidas, la herramienta puede recomendar más servidores. Las recomendaciones se basan en el valor mayor de los siguientes elementos, la actividad diaria o el número máximo de máquinas virtuales (se supone que cada máquina virtual tiene tres discos), lo que se alcance primero en el servidor de configuración o en el servidor de procesos adicional. Encontrará los detalles de la actividad total por día y el número total de discos protegidos en la sección "Resumen local".

![Infraestructura local requerida en Deployment Planner](media/site-recovery-vmware-deployment-planner-analyze-report/required-on-premises-components-v2a.png)

### <a name="what-if-analysis"></a>Análisis de hipótesis
Este análisis indica el número de infracciones que pueden producirse durante el período de generación de perfiles si se establece un ancho de banda menor para cumplir el RPO deseado solo en el 90 % de las ocasiones. Pueden producirse una o varias infracciones de RPO en cualquier día determinado. El gráfico muestra el RPO máximo del día.
En función de este análisis, puede decidir si el número de infracciones de RPO a lo largo de todos los días y el límite máximo de RPO por día es aceptable para el ancho de banda menor especificado. Si es aceptable, puede asignar el ancho de banda menor para la replicación. En caso contrario, asigne el ancho de banda mayor como se sugirió anteriormente para satisfacer el RPO deseado en el 100 % de las ocasiones.

![Análisis de hipótesis en Deployment Planner](media/site-recovery-vmware-deployment-planner-analyze-report/what-if-analysis-v2a.png)

### <a name="recommended-vm-batch-size-for-initial-replication"></a>Tamaño de lote de máquinas virtuales recomendado para la replicación inicial
En esta sección, se recomienda el número de máquinas virtuales que se pueden proteger en paralelo para completar la replicación inicial en 72 horas con el ancho de banda sugerido para satisfacer el RPO deseado el 100 % del tiempo que se establece. Este valor se puede configurar. Para cambiarlo en el momento de generación de informes, use el parámetro *GoalToCompleteIR*.

Este gráfico muestra un intervalo de valores de ancho de banda y un recuento del tamaño del lote de máquinas virtuales que se calcula que son necesarias para completar la replicación inicial en 72 horas, en función del valor medio de tamaño de las máquinas virtuales detectadas entre todas las máquinas virtuales compatibles.

En la versión preliminar pública, el informe no especifica las máquinas virtuales que deben incluirse en un lote. Puede usar el tamaño de disco que se muestra en la sección "Máquinas virtuales compatibles" para hallar el tamaño de cada máquina virtual y seleccionarlas para un lote, o bien puede seleccionarlas en función de características de carga de trabajo conocidas. La hora de finalización de la replicación inicial cambia proporcionalmente en función del tamaño real de los discos de las máquinas virtuales, del espacio de disco usado y del rendimiento de red disponible.

![Tamaño de lote de máquinas virtuales recomendado](media/site-recovery-vmware-deployment-planner-analyze-report/ir-batching-v2a.png)

### <a name="cost-estimation"></a>Estimación de costos
En el gráfico siguiente se muestra la vista de resumen del costo total estimado de la recuperación ante desastres en Azure de la región de destino elegida y en la moneda que ha especificado para generar el informe.

![Resumen de la estimación de costos](media/site-recovery-vmware-deployment-planner-analyze-report/cost-estimation-summary-v2a.png)

El resumen le ayuda a comprender lo que debe pagar por los servicios de almacenamiento, proceso, red y licencia cuando protege todas las máquinas virtuales compatibles en Azure mediante Azure Site Recovery. El costo se calcula para las máquinas virtuales compatibles y no para todas las máquinas virtuales para las que se han generado perfiles.  

Puede ver el costo mensual o anual. Obtenga más información sobre las [regiones de destino admitidas](./site-recovery-vmware-deployment-planner-cost-estimation.md#supported-target-regions) y las [monedas admitidas](./site-recovery-vmware-deployment-planner-cost-estimation.md#supported-currencies).

**Cost by components** (Costo por componentes): el costo total de recuperación ante desastres se divide en cuatro componentes: costo de proceso, almacenamiento, red y licencia de Azure Site Recovery. El costo se calcula en función del consumo en el que se incurrirá durante la replicación y en el transcurso del simulacro de recuperación ante desastres por los servicios de proceso, almacenamiento (premium y estándar), circuito ExpressRoute o VPN configurados entre el sitio local y Azure, y la licencia de Azure Site Recovery.

**Cost by states**: el costo total de recuperación ante desastres se clasifica según dos estados diferentes: replicación y simulacro de recuperación ante desastres.

**Replication cost** (Costo de replicación):  costo en el que se incurrirá durante la replicación. Incluye el costo de almacenamiento, red y licencia de Azure Site Recovery.

**DR-Drill cost** (Costo del simulacro de recuperación ante desastres): costo en el que se incurrirá durante las conmutaciones por error de prueba. Azure Site Recovery pone en marcha las máquinas virtuales durante la conmutación por error de prueba. El costo del simulacro de recuperación ante desastres incluye el costo de los servicios de proceso y almacenamiento de las máquinas virtuales en ejecución.

**Azure storage cost per Month/Year**: muestra el costo total en el que se incurrirá por el almacenamiento premium y estándar por la replicación y el simulacro de recuperación ante desastres.
Puede ver el análisis detallado de los costos por máquina virtual en la hoja [Cost Estimation](site-recovery-vmware-deployment-planner-cost-estimation.md) (Estimación de costos).

### <a name="growth-factor-and-percentile-values-used"></a>Factor de crecimiento y valores de percentil utilizados
Esta sección de la parte inferior de la hoja muestra el valor del percentil que se utiliza en todos los contadores de rendimiento de las máquinas virtuales de las que se ha generado el perfil (el valor predeterminado es el percentil 95) y el factor de crecimiento (el valor predeterminado es el 30 %) que se usa en todos los cálculos.

![Factor de crecimiento y valores de percentil utilizados](media/site-recovery-vmware-deployment-planner-analyze-report/growth-factor-v2a.png)

## <a name="recommendations-with-available-bandwidth-as-input"></a>Recomendaciones relacionadas con el ancho de banda disponible como entrada

![Recomendaciones relacionadas con el ancho de banda disponible como entrada](media/site-recovery-vmware-deployment-planner-analyze-report/profiling-overview-bandwidth-input-v2a.png)

Puede darse el caso de que sepa que no puede establecer un ancho de banda de más de x Mbps para la replicación de Site Recovery. La herramienta le permite especificar el ancho de banda disponible (mediante el parámetro -Bandwidth durante la generación de informes) y obtener el RPO factible en minutos. Con este valor de RPO factible, puede decidir si necesita aprovisionar más ancho de banda o si está conforme con tener una solución de recuperación ante desastres con este RPO.

![RPO factible para un ancho de banda de 500 Mbps](media/site-recovery-vmware-deployment-planner-analyze-report/achievable-rpo-v2a.png)

## <a name="vm-storage-placement"></a>Selección de ubicación de almacenamiento de máquina virtual

>[!Note]
>Deployment Planner v2.5 en adelante recomienda la colocación del almacenamiento para las máquinas que se replicarán directamente en Managed Disks.

![Selección de ubicación de almacenamiento de máquina virtual](media/site-recovery-vmware-deployment-planner-analyze-report/vm-storage-placement-v2a.png)

**Tipo de almacenamiento de replicación**: disco administrado estándar o premium, que se usa para replicar todas las máquinas virtuales correspondientes que se mencionan en la columna **VMs to Place** (Máquinas virtuales para colocar).

**Tipo de cuenta de almacenamiento de registros**: todos los registros de la replicación se almacenan en una cuenta de almacenamiento estándar.

**Prefijo sugerido para la cuenta de almacenamiento**: el prefijo de tres caracteres sugerido que se puede usar para asignar un nombre a la cuenta de almacenamiento en caché. Puede usar su propio prefijo, pero el que sugiere la herramienta sigue la [convención de nomenclatura de particiones de las cuentas de almacenamiento](https://aka.ms/storage-performance-checklist).

**Suggested Log Account Name** (Nombre de cuenta de registros sugerido): el nombre de la cuenta de almacenamiento después de incluir el prefijo sugerido. Reemplace el nombre entre corchetes angulares (< y >) por una entrada personalizada.

**Placement Summary** (Resumen de la selección de ubicación): un resumen de los discos necesarios para las máquinas virtuales protegidas por tipo de almacenamiento. Incluye el número total de máquinas virtuales, el tamaño total aprovisionado en todos los discos y el número total de discos.

**Virtual Machines to Place** (Máquinas virtuales para colocar): una lista de todas las máquinas virtuales que se deben colocar en la cuenta de almacenamiento especificada para que tanto el rendimiento como el uso sean óptimos.

## <a name="compatible-vms"></a>Máquinas virtuales compatibles
![Hoja de cálculo de Excel de las máquinas virtuales compatibles](media/site-recovery-vmware-deployment-planner-analyze-report/compatible-vms-v2a.png)

**VM Name** (Nombre de la máquina virtual): el nombre o la dirección IP de la máquina virtual que se utilizan en VMListFile cuando se genera un informe. Esta columna también muestra los discos (VMDK) que están conectados a las máquinas virtuales. Para distinguir las máquinas virtuales de vCenter con nombres o direcciones IP, los nombres incluyen el nombre del host de ESXi. El host de ESXi enumerado es en el que se colocó la máquina virtual cuando la herramienta la detecto en el período de generación de perfiles.

**VM Compatibility** (Compatibilidad de la máquina virtual): los valores son **Yes** (Sí) y **Yes\*** (Sí). **Sí**\* es para las instancias en las que la máquina virtual es una opción para [discos SSD Premium](../virtual-machines/disks-types.md). En este caso, la alta actividad de la generación de perfiles o el disco de IOPS se encuadra en las categorías P20 o P30, pero el tamaño del disco hace que se asigne a una categoría P10 o P20. La cuenta de almacenamiento decide a qué tipo de disco de almacenamiento Premium se asigna un disco, en función de su tamaño. Por ejemplo:
* Menos de 128 GB es P10.
* De 128 GB a 256 GB es P15
* De 256 GB a 512 GB es P20.
* De 512 GB a 1024 GB es P30.
* De 1025 GB a 2048 GB es P40.
* De 2049 GB a 4095 GB es P50.

Por ejemplo, si las características de carga de trabajo de un disco lo colocan en la categoría P20 o P30, pero el tamaño lo asigna a un tipo de disco de almacenamiento Premium inferior, la herramienta marca la máquina virtual como **Sí**\*. La herramienta también recomienda que cambie el tamaño del disco de origen para poder encuadrarlo en el tipo de disco de almacenamiento Premium recomendado o que cambie el tipo de disco de destino después de la conmutación por error.

**Storage Type** (Tipo de almacenamiento): estándar o premium.

**Asrseeddisk (disco administrado) creado para la replicación**: el nombre del disco que se crea al habilitar la replicación. Almacena los datos y sus instantáneas en Azure.

**Peak R/W IOPS (with Growth Factor)** (Valor máximo de IOPS de lectura y escritura [con factor de crecimiento]): el valor máximo de IOPS de lectura/escritura de la carga de trabajo en el disco (el percentil 95 es el predeterminado), incluido el factor de crecimiento futuro (el valor predeterminado es el 30 %). Tenga en cuenta que el total de IOPS de lectura y escritura de una máquina virtual no es siempre la suma de las IOPS de lectura y escritura de los discos individuales de la máquina virtual, ya que las IOPS de lectura y escritura máximas de la máquina virtual es el máximo de la suma de las IOPS de lectura y escritura sus discos individuales durante cada minuto del período de generación de perfiles.

**Peak Data Churn in Mbps (with Growth Factor)** (Renovación máxima de datos en MB/s [con factor de crecimiento]): la tasa de renovación máxima en el disco (el percentil 95 es el predeterminado), incluido el factor de crecimiento futuro (el valor predeterminado es el 30 %). Tenga en cuenta que el total de actividad de datos de la máquina virtual no siempre es la suma de la actividad de datos de los discos individuales de la máquina virtual, ya que la actividad de datos máxima de la máquina virtual es el máximo de la suma de las actividades de los discos individuales durante cada minuto del período de generación de perfiles.

**Azure VM Size** (Tamaño de la máquina virtual de Azure): el tamaño ideal de la máquina virtual de Azure Cloud Services asignada para esta máquina virtual local. La asignación basada en la memoria, el número de núcleos de discos/núcleos/NIC y las IOPS de lectura y escritura de la máquina virtual local. La recomendación es usar siempre el menor tamaño de máquina virtual de Azure que cumpla todas las características de la máquina virtual local.

**Number of Disks** (Número de discos): el número total de discos de máquina virtual (VMDK) en la máquina virtual.

**Disk size (GB)** (Tamaño del disco [GB]): el tamaño total configurado de todos los discos de la máquina virtual. La herramienta también muestra el tamaño de los discos individuales de la máquina virtual.

**Cores** (Núcleos): el número de núcleos de CPU en la máquina virtual.

**Memory (MB)** (Memoria [MB]): la memoria RAM de la máquina virtual.

**NICs** (Tarjetas NIC): el número de tarjetas NIC de la máquina virtual.

**Boot Type** (Tipo de arranque): el tipo de arranque de la máquina virtual. Puede ser BIOS o EFI.  Actualmente, Azure Site Recovery admite máquinas virtuales de EFI de Windows Server (Windows Server 2012, 2012 R2 y 2016) siempre que el número de particiones en el disco de arranque sea menor que 4 y el tamaño del sector de arranque sea de 512 bytes. Para proteger las máquinas virtuales de EFI, la versión de Mobility Service de Azure Site Recovery debe ser la 9.13, o cualquier versión superior. Solo se admite la conmutación por error de las máquinas virtuales de EFI. No se admite la conmutación por recuperación.  

**OS Type** (Tipo de sistema operativo): el tipo de sistema operativo de la máquina virtual. Puede ser Windows o Linux u otra, según la plantilla de VMware vSphere elegida al crear la máquina virtual.  

## <a name="incompatible-vms"></a>Máquinas virtuales no compatibles

![Hoja de cálculo de Excel de máquinas virtuales no compatibles
](media/site-recovery-vmware-deployment-planner-analyze-report/incompatible-vms-v2a.png)

**VM Name** (Nombre de la máquina virtual): el nombre o la dirección IP de la máquina virtual que se utilizan en VMListFile cuando se genera un informe. Esta columna también muestra los VMDK que están conectados a las máquinas virtuales. Para distinguir las máquinas virtuales de vCenter con nombres o direcciones IP, los nombres incluyen el nombre del host de ESXi. El host de ESXi enumerado es en el que se colocó la máquina virtual cuando la herramienta la detecto en el período de generación de perfiles.

**VM Compatibility** (Compatibilidad de la máquina virtual): indica el motivo por el que una máquina virtual dada no es compatible con Site Recovery. Se describen las razones de cada disco incompatible de la máquina virtual, que, en función de los [límites de almacenamiento](https://aka.ms/azure-storage-scalbility-performance) publicados, pueden ser cualesquiera de las siguientes:

* El tamaño del disco de datos es incorrecto o el tamaño de disco de sistema operativo es incorrecto. [Revise](vmware-physical-azure-support-matrix.md#azure-vm-requirements) los límites compatibilidad. 
* El tamaño total de la máquina virtual (replicación + TFO) supera el límite de tamaño de la cuenta de almacenamiento que se admite (35 TB). Esta incompatibilidad se produce normalmente cuando uno de los discos de la máquina virtual tiene una característica de rendimiento que supera los límites de almacenamiento estándar de Azure o de Site Recovery. Una instancia de este tipo coloca la máquina virtual en la zona de almacenamiento premium. Sin embargo, el tamaño máximo que se admite de una cuenta de almacenamiento Premium es de 35 TB y una sola máquina virtual protegida no se puede proteger en varias cuentas de almacenamiento. Tenga también en cuenta que si se realiza una conmutación por error de prueba en una máquina virtual protegida, esta se ejecuta en la misma cuenta de almacenamiento en la que se lleva a cabo la replicación. En este caso, configure el doble del tamaño del disco para que la replicación y la conmutación por error de prueba puedan realizarse en paralelo.

* El valor de IOPS de origen supera el límite que admite el almacenamiento, 7500 por disco.

* El valor de IOPS de origen supera el límite que admite el almacenamiento, 80 000 por máquina virtual.

* La actividad de datos media supera el límite que admite Site Recovery de 20 MB/s para el tamaño medio de E/S del disco.

* La actividad de datos máxima en todos los discos de la máquina virtual supera el límite máximo que admite Site Recovery, 54 MB/s por máquina virtual.

* El valor medio de las IOPS de escritura efectiva supera el límite que admite Site Recovery, 840 por disco.

* El almacenamiento de instantáneas estimado supera el límite admitido de 10 TB.

* La actividad de datos total por día supera el límite de 2 TB que admite un servidor de procesos por día.


**Peak R/W IOPS (with Growth Factor)** (Valor máximo de IOPS de lectura y escritura [con factor de crecimiento]): el valor máximo de IOPS de la carga de trabajo en el disco (el percentil 95 es el predeterminado), incluido el factor de crecimiento futuro (el valor predeterminado es el 30 %). Tenga en cuenta que el total de IOPS de lectura y escritura de la máquina virtual no es siempre la suma de las IOPS de lectura y escritura de los discos individuales de la máquina virtual, ya que las IOPS de lectura y escritura máximas de la máquina virtual es el máximo de la suma de las IOPS de lectura y escritura sus discos individuales durante cada minuto del período de generación de perfiles.

**Peak Data Churn in Mbps (with Growth Factor)** (Renovación máxima de datos en MB/s [con factor de crecimiento]): la tasa de renovación máxima en el disco (el percentil 95 es el predeterminado), incluido el factor de crecimiento futuro (el valor predeterminado es el 30 %). Tenga en cuenta que el total de actividad de datos de la máquina virtual no siempre es la suma de la actividad de datos de los discos individuales de la máquina virtual, ya que la actividad de datos máxima de la máquina virtual es el máximo de la suma de las actividades de los discos individuales durante cada minuto del período de generación de perfiles.

**Number of Disks** (Número de discos): el número total de discos duros virtuales en la máquina virtual.

**Disk size (GB)** (Tamaño del disco [GB]): el tamaño total configurado de todos los discos de la máquina virtual. La herramienta también muestra el tamaño de los discos individuales de la máquina virtual.

**Cores** (Núcleos): el número de núcleos de CPU en la máquina virtual.

**Memory (MB)** (Memoria [MB]): la cantidad de memoria RAM en la máquina virtual.

**NICs** (Tarjetas NIC): el número de tarjetas NIC de la máquina virtual.

**Boot Type** (Tipo de arranque): el tipo de arranque de la máquina virtual. Puede ser BIOS o EFI.  Actualmente, Azure Site Recovery admite máquinas virtuales de EFI de Windows Server (Windows Server 2012, 2012 R2 y 2016) siempre que el número de particiones en el disco de arranque sea menor que 4 y el tamaño del sector de arranque sea de 512 bytes. Para proteger las máquinas virtuales de EFI, la versión de Mobility Service de Azure Site Recovery debe ser la 9.13, o cualquier versión superior. Solo se admite la conmutación por error de las máquinas virtuales de EFI. No se admite la conmutación por recuperación.

**OS Type** (Tipo de sistema operativo):  el tipo de sistema operativo de la máquina virtual. Puede ser Windows o Linux u otra, según la plantilla de VMware vSphere elegida al crear la máquina virtual.

## <a name="azure-site-recovery-limits"></a>Límites de Azure Site Recovery
En la tabla siguiente se proporcionan los límites de Azure Site Recovery. Estos límites se basan en nuestras pruebas, pero no pueden cubrir todas las combinaciones de E/S posibles de la aplicación. Los resultados reales pueden variar en función de la combinación de E/S de la aplicación. Para unos mejores resultados, incluso después del planeamiento de la implementación, es aconsejable probar siempre la aplicación de forma exhaustiva mediante una conmutación por error de prueba para obtener una imagen real del rendimiento de la aplicación.

**Destino de almacenamiento de la replicación** | **Tamaño medio de E/S de disco de origen** |**Actividad de datos media de disco de origen** | **Actividad de datos de disco de origen total por día**
---|---|---|---
Standard Storage | 8 KB | 2 MB/s | 168 GB por disco
Disco Premium P10 o P15 | 8 KB  | 2 MB/s | 168 GB por disco
Disco Premium P10 o P15 | 16 KB | 4 MB/s |  336 GB por disco
Disco Premium P10 o P15 | 32 KB, o más | 8 MB/s | 672 GB por disco
Disco Premium P20, P30, P40 o P50 | 8 KB    | 5 MB/s | 421 GB por disco
Disco Premium P20, P30, P40 o P50 | 16 KB, o más | 20 MB/s | 1684 GB por disco

**Actividad de datos de origen** | **Límite máximo**
---|---
Actividad de datos máxima entre todos los discos de una máquina virtual | 54 MB/s
Actividad de datos máxima por día admitida por un servidor de procesos | 2 TB

Estos son los números promedio si la superposición de E/S es del 30 %. Site Recovery es capaz de controlar un mayor rendimiento en función de la relación de superposición, tamaños de escritura mayores y el comportamiento real de E/S de la carga de trabajo. Los números anteriores asumen un trabajo pendiente típico de aproximadamente cinco minutos. Es decir, una vez que se cargan los datos, se procesan y se crea un punto de recuperación en menos de cinco minutos.


## <a name="cost-estimation"></a>Estimación de costos
Obtenga más información sobre la [estimación de costos](site-recovery-vmware-deployment-planner-cost-estimation.md).


## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre la [estimación de costos](site-recovery-vmware-deployment-planner-cost-estimation.md).

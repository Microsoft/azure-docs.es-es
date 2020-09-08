---
title: Análisis del informe de Deployment Planner de Hyper-V en Azure Site Recovery
description: En este artículo se describe cómo analizar un informe generado por Azure Site Recovery Deployment Planner para la recuperación ante desastres de máquinas virtuales de Hyper-V en Azure.
services: site-recovery
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 10/21/2019
ms.author: mayg
ms.openlocfilehash: b2bb351de292ff2015cdcbd388155063a86a7359
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88653586"
---
# <a name="analyze-the-azure-site-recovery-deployment-planner-report"></a>Análisis del informe de Azure Site Recovery Deployment Planner
En este artículo se describen las hojas que contiene el informe de Excel generado por Azure Site Recovery Deployment Planner en un escenario de Hyper-V en Azure.

## <a name="on-premises-summary"></a>Resumen local
La hoja de cálculo de resumen local proporciona una introducción al entorno de Hyper-V en el que se han generado perfiles.

![Resumen local](media/hyper-v-deployment-planner-analyze-report/on-premises-summary-h2a.png)

**Fecha de inicio** y **Fecha de finalización**: las fechas de inicio y finalización de los datos de generación de perfiles que se tienen en cuenta para la generación del informe. De forma predeterminada, la fecha de inicio es la fecha en que comienza la generación de perfiles, mientras que la de finalización es la fecha en la que se detiene. Esta información se puede especificar mediante los valores "StartDate" y "EndDate" si el informe se genera con estos parámetros.

**Total number of profiling days** (Número total de días de generación de perfiles): el número total de días de generación de perfiles comprendido entre las fechas de inicio y de finalización para el que se genera el informe.

**Number of compatible virtual machines** (Número de máquinas virtuales compatibles): el número total de máquinas virtuales compatibles para las que se calculan el ancho de banda de red, el número de cuentas de almacenamiento y los núcleos de Azure necesarios.

**Total number of disks across all compatible virtual machines** (Número total de discos en todas las máquinas virtuales compatibles): el número total de discos en todas las máquinas virtuales compatibles.

**Average number of disks per compatible virtual machine** (Número medio de discos en cada máquina virtual compatible): el número medio de discos calculado para todas las máquinas virtuales compatibles.

**Average disk size (GB)** (Tamaño medio de disco [GB]): el tamaño medio de disco calculado en todas las máquinas virtuales compatibles.

**Desired RPO (minutes)** (RPO deseado [en minutos]): el objetivo de punto de recuperación predeterminado o el valor pasado para el parámetro "DesiredRPO" en el momento de la generación de informes para calcular el ancho de banda necesario.

**Desired bandwidth (Mbps)** (Ancho de banda deseado [Mbps]): el valor pasado para el parámetro "Bandwidth" en el momento de la generación de informes para calcular el objetivo de punto de recuperación (RPO) factible.

**Observed typical data churn per day (GB)** (Renovación de datos normal observada por día [GB]): el valor medio de renovación de datos observada en todos los días de generación de perfiles.

## <a name="recommendations"></a>Recomendaciones 
La hoja de recomendaciones del informe de Hyper-V a Azure tiene los siguientes detalles según el RPO deseado seleccionado:

![Recomendaciones para el informe de Hyper-V a Azure](media/hyper-v-deployment-planner-analyze-report/Recommendations-h2a.png)

### <a name="profile-data"></a>Generación de perfiles de datos
![Generación de perfiles de datos](media/hyper-v-deployment-planner-analyze-report/profile-data-h2a.png)

**Profiled data period** (Periodo de datos de generación de perfiles): el período durante el que se ejecutó la generación de perfiles. De forma predeterminada, la herramienta incluye todos los datos para los que se han generado perfiles en el cálculo. Si ha usado la opción StartDate (Fecha inicial) y EndDate (Fecha final) en la generación de informes, se genera el informe para el período específico. 

**Number of Hyper-V servers profiled** (Número de servidores de Hyper-V para los que se han generado perfiles): número de servidores de Hyper-V para los que se genera el informe de sus máquinas virtuales. Seleccione el número para ver el nombre de los servidores de Hyper-V. Se abre la hoja On-premises Storage Requirement (Requisitos de almacenamiento local) para mostrar todos los servidores junto con sus requisitos de almacenamiento. 

**Desired RPO** (RPO deseado): el objetivo de punto de recuperación de la implementación. De manera predeterminada, se calcula el ancho de banda de red necesario para los valores de RPO de 15, 30 y 60 minutos. En función de la selección, los valores afectados se actualizan en la hoja. Si ha usado el parámetro DesiredRPOinMin al generar el informe, ese valor se muestra en el resultado de RPO deseado.

### <a name="profiling-overview"></a>Información general sobre la generación de perfiles
![Información general sobre la generación de perfiles](media/hyper-v-deployment-planner-analyze-report/profiling-overview-h2a.png)

**Total Profiled Virtual Machines** (Total de máquinas virtuales de las que se ha generado el perfil): el número total de las máquinas virtuales cuyos datos de generación de perfiles están disponibles. Si VMListFile contiene los nombres de las máquinas virtuales para las que no se han generado perfiles, dichas máquinas virtuales no se tienen en cuenta en la generación de informes y se excluyen del número total de máquinas virtuales de las que se han generado perfiles.

**Compatible Virtual Machines** (Máquinas virtuales compatibles): número de máquinas virtuales que se pueden proteger en Azure mediante Azure Site Recovery. Es el número total de máquinas virtuales compatibles para las que se calculan el ancho de banda de red, el número de cuentas de almacenamiento y el número de núcleos de Azure necesarios. Los detalles de todas las máquinas virtuales compatibles están disponibles en la sección "Máquinas virtuales compatibles".

**Incompatible Virtual Machines** (Máquinas virtuales no compatibles): el número de máquinas virtuales de las que se ha generado el perfil que no son compatibles para la protección con Site Recovery. Los motivos de dicha incompatibilidad se indican en la sección "Máquinas virtuales no compatibles". Si VMListFile contiene los nombres de las máquinas virtuales para las que no se han generado perfiles, dichas máquinas se excluyen del número de máquinas virtuales no compatibles. Estas máquinas aparecen como "Datos no encontrados" al final de la sección "Máquinas virtuales no compatibles".

**Desired RPO** (RPO deseado): el objetivo de punto de recuperación deseado, en minutos. El informe se genera para tres valores de RPO: 15 (valor predeterminado), 30 y 60 minutos. La recomendación que se hace sobre el ancho de banda en el informe se cambia en función de la selección que se realice en la lista desplegable **Desired RPO** (RPO deseado) de la parte superior derecha de la hoja. Si el informe se ha generado mediante el parámetro -DesiredRPO con un valor personalizado, dicho valor personalizado se muestra como valor predeterminado en la lista desplegable **Desired RPO** (RPO deseado).

### <a name="required-network-bandwidth-mbps"></a>Ancho de banda de red requerido (Mbps)
![Ancho de banda de red requerido](media/hyper-v-deployment-planner-analyze-report/required-network-bandwidth-h2a.png)

**To meet RPO 100 percent of the time** (Satisfacer el RPO en el 100 % de las ocasiones): el ancho de banda recomendado, en Mbps, que se debe asignar para satisfacer el RPO en el 100 % de las ocasiones. Esta cantidad de ancho de banda debe estar dedicada a la replicación diferencial de estado estable de todas las máquinas virtuales compatibles para evitar infracciones de RPO.

**To meet RPO 90% of the time** (Satisfacer el RPO en el 90 % de las ocasiones): es posible que debido al precio del ancho de banda, o por cualquier otro motivo, no pueda establecer el ancho de banda necesario para satisfacer el RPO en el 100 % de las ocasiones. Si este es el caso, puede usar un valor de ancho de banda inferior que pueda satisfacer el RPO deseado el 90 % del tiempo. Para conocer las implicaciones de establecer un ancho de banda inferior, el informe proporciona el análisis de hipótesis del número previsible de infracciones de RPO y su duración.

**Achieved Throughput** (Rendimiento conseguido): rendimiento del servidor en el que ha ejecutado el comando GetThroughput para la región de Azure en la que se encuentra la cuenta de almacenamiento. Este número de rendimiento indica el nivel estimado que se puede lograr al proteger las máquinas virtuales compatibles mediante Site Recovery. Las características de almacenamiento y red del servidor Hyper-V deben permanecer igual que las del servidor desde el que se ejecuta la herramienta.

Para todas las implementaciones de Site Recovery que se realicen en la empresa, se recomienda usar [ExpressRoute](https://aka.ms/expressroute).

### <a name="required-storage-accounts"></a>Cuentas de almacenamiento requeridas
El siguiente gráfico muestra el número total de cuentas de almacenamiento (Estándar y Premium) que se requieren para proteger todas las máquinas virtuales compatibles. Para saber qué cuenta de almacenamiento se debe usar para cada máquina virtual, consulte la sección "selección de ubicación de almacenamiento de máquina virtual".

![Cuentas de Azure Storage necesarias](media/hyper-v-deployment-planner-analyze-report/required-storage-accounts-h2a.png)

### <a name="required-number-of-azure-cores"></a>Número de núcleos de Azure requeridos
Este resultado es el número total de núcleos que se deben configurar antes de realizar una conmutación por error real o de prueba de todas las máquinas virtuales compatibles. Si hay muy pocos núcleos en la suscripción, Site Recovery no puede crear las máquinas virtuales en el momento de realizar la conmutación por error real o de prueba.

![Número de núcleos de Azure requeridos](media/hyper-v-deployment-planner-analyze-report/required-number-of-azure-cores-h2a.png)


### <a name="additional-on-premises-storage-requirement"></a>Requisitos de almacenamiento local adicionales

El almacenamiento disponible total necesario en los servidores de Hyper-V para que la replicación inicial y la replicación diferencial se realicen correctamente a fin de garantizar que la replicación de las máquinas virtuales no va a provocar tiempo de inactividad no deseado en las aplicaciones de producción. Se puede encontrar más información sobre los detalles de cada requisito de volumen en los [requisitos de almacenamiento local](#on-premises-storage-requirement). 

Para comprender por qué es necesario espacio libre para la replicación, consulte la sección [Requisitos de almacenamiento local](#why-do-i-need-free-space-on-the-hyper-v-server-for-the-replication).

![Requisitos de almacenamiento local y frecuencia de copia](media/hyper-v-deployment-planner-analyze-report/on-premises-storage-and-copy-frequency-h2a.png)

### <a name="maximum-copy-frequency"></a>Frecuencia de copia máxima
Debe establecerse la frecuencia de copia máxima recomendada para la replicación para lograr el RPO deseado. El valor predeterminado es cinco minutos. Puede establecer la frecuencia de copia en 30 segundos para lograr un mejor RPO.

### <a name="what-if-analysis"></a>Análisis de hipótesis
![What-if-analysis](media/hyper-v-deployment-planner-analyze-report/what-if-analysis-h2a.png) Este análisis indica el número de infracciones que pueden producirse durante el período de generación de perfiles si se establece un ancho de banda menor para cumplir el RPO deseado solo en el 90 % de las ocasiones. Pueden producirse una o varias infracciones de RPO en cualquier día determinado. El gráfico muestra el RPO máximo del día. En función de este análisis, puede decidir si el número de infracciones de RPO a lo largo de todos los días y el límite máximo de RPO por día es aceptable para el ancho de banda menor especificado. Si es aceptable, puede asignar el ancho de banda inferior para la replicación. Si no es aceptable, asigne más ancho de banda como se sugiere para satisfacer el RPO deseado en el 100 % de las ocasiones. 

### <a name="recommendation-for-successful-initial-replication"></a>Recomendación para una replicación inicial correcta
En esta sección se describe el número de lotes en el que se van a proteger las máquinas virtuales y el ancho de banda mínimo necesario para finalizar con éxito la replicación inicial (IR). 

![Procesamiento por lotes de IR](media/hyper-v-deployment-planner-analyze-report/ir-batching-h2a.png)

La máquina virtual se debe proteger en el orden de lote especificado. Cada lote tiene una lista específica de máquinas virtuales. Las máquinas virtuales del Lote 1 se deben proteger antes que las del Lote 2. Las máquinas virtuales del Lote 2 se deben proteger antes que las del Lote 3 y así sucesivamente. Una vez completada la replicación inicial de las máquinas virtuales del Lote 1, puede habilitar la replicación de las máquinas virtuales del Lote 2. De igual forma, una vez completada la replicación inicial de las máquinas virtuales del Lote 2, puede habilitar la replicación de las máquinas virtuales del Lote 3 y así sucesivamente. 

Si no se sigue el orden de los lotes, podría no haber ancho de banda suficiente para la replicación inicial de las máquinas virtuales que se protegen más adelante. El resultado es que las máquinas virtuales nunca finalizan la replicación inicial o algunas máquinas virtuales protegidas pueden entrar en modo de resincronización. La hoja Procesamiento por lotes de IR para el RPO seleccionado contiene información detallada sobre las máquinas virtuales que se incluyen en cada lote.

Aquí, el gráfico muestra la distribución del ancho de banda para la replicación inicial y la replicación diferencial entre los lotes en el orden de lote especificado. Cuando se protegen las máquinas virtuales del primer lote, todo el ancho de banda está disponible para la replicación inicial. Una vez que la replicación inicial del primer lote ha finalizado, se necesita parte del ancho de banda para la replicación diferencial. El ancho de banda restante está disponible para la replicación inicial de las máquinas virtuales del segundo lote. 

La barra del Lote 2 muestra el ancho de banda necesario para la replicación diferencial de las máquinas virtuales del Lote 1 y el ancho de banda disponible para la replicación inicial de las máquinas virtuales del Lote 2. Igualmente, la barra del Lote 3 muestra el ancho de banda necesario para la replicación diferencial de los lotes anteriores (las máquinas virtuales del Lote 1 y el Lote 2) y el ancho de banda disponible para la replicación inicial del Lote 3 y así sucesivamente. Después de que la replicación inicial de todos los lotes ha finalizado, la última barra muestra el ancho de banda necesario para la replicación diferencial de todas las máquinas virtuales protegidas.

**¿Por qué se necesita el procesamiento por lotes de la replicación inicial?**
El tiempo de finalización de la replicación inicial está en función del tamaño de los discos de las máquinas virtuales, del espacio de disco usado y del rendimiento de red disponible. Los detalles están disponibles en la hoja Procesamiento por lotes de IR para el RPO seleccionado.

### <a name="cost-estimation"></a>Estimación de costos
En el gráfico siguiente se muestra la vista de resumen del costo total estimado de la recuperación ante desastres en Azure de la región de destino elegida y en la moneda que ha especificado para generar el informe.

![Resumen de la estimación de costos](media/hyper-v-deployment-planner-analyze-report/cost-estimation-summary-h2a.png)

El resumen le ayuda a comprender lo que debe pagar por almacenamiento, proceso, red y licencia cuando protege todas las máquinas virtuales compatibles en Azure mediante Azure Site Recovery. El costo se calcula para las máquinas virtuales compatibles y no para todas las máquinas virtuales para las que se han generado perfiles. 
 
Puede ver el costo mensual o anual. Obtenga más información sobre las [regiones de destino admitidas](./hyper-v-deployment-planner-cost-estimation.md#supported-target-regions) y las [monedas admitidas](./hyper-v-deployment-planner-cost-estimation.md#supported-currencies).

**Cost by components** (Costo por componentes): el costo total de la recuperación ante desastres se divide entre cuatro componentes: proceso, almacenamiento, red y costo de la licencia de Site Recovery. El costo se calcula en función del consumo que se ha realizado durante la replicación y durante el simulacro de recuperación ante desastres. Para los cálculos se usa Compute, Storage (Premium y estándar), el circuito ExpressRoute o la red privada virtual que están configurados entre el sitio local y Azure, y la licencia de Site Recovery.

**Cost by states** (Costo por estados): el costo total de recuperación ante desastres se clasifica según dos estados diferentes: replicación y simulacro de recuperación ante desastres. 

**Replication cost** (Costo de replicación): el costo en el que se incurre durante la replicación. Incluye el costo de almacenamiento, red y de la licencia de Site Recovery. 

**DR-Drill cost** (Costo del simulacro de recuperación ante desastres): costo en el que se incurre durante las conmutaciones por error de prueba. Site Recovery pone en marcha las máquinas virtuales durante la conmutación por error de prueba. El costo del simulacro de recuperación ante desastres incluye el costo de proceso y almacenamiento de las máquinas virtuales en ejecución. 

**Azure storage cost per Month/Year** (Costo de almacenamiento de Azure por mes/año): el gráfico de barras muestra el costo total de almacenamiento en el que se incurre por el almacenamiento Premium y estándar en la replicación y el simulacro de recuperación ante desastres. Puede ver el análisis detallado de los costos por máquina virtual en la hoja [Cost Estimation](hyper-v-deployment-planner-cost-estimation.md) (Estimación de costos).

### <a name="growth-factor-and-percentile-values-used"></a>Factor de crecimiento y valores de percentil utilizados
En esta sección de la parte inferior de la hoja se muestra el valor de percentil usado en todos los contadores de rendimiento de las máquinas virtuales cuyo perfil se ha generado (el valor predeterminado es el percentil 95). También se muestra el factor de crecimiento (el valor predeterminado es 30 por ciento) que se usa en todos los cálculos.

![Factor de crecimiento y valores de percentil utilizados](media/hyper-v-deployment-planner-run/growth-factor-max-percentile-value.png)

## <a name="recommendations-with-available-bandwidth-as-input"></a>Recomendaciones relacionadas con el ancho de banda disponible como entrada
![Introducción a la generación de perfiles con el ancho de banda de entrada](media/hyper-v-deployment-planner-analyze-report/profiling-overview-bandwidth-input-h2a.png)

Puede darse el caso de que sepa que no puede establecer un ancho de banda de más de x Mbps para la replicación de Site Recovery. Puede usar la herramienta para especificar el ancho de banda disponible (mediante el parámetro -Bandwidth durante la generación de informes) y obtener el RPO factible en minutos. Con este valor de RPO factible, puede decidir si necesita aprovisionar ancho de banda adicional o si está satisfecho con una solución de recuperación ante desastres con este RPO.

![RPO factible](media/hyper-v-deployment-planner-analyze-report/achivable-rpo-h2a.PNG)

## <a name="vm-storage-placement-recommendation"></a>Recomendación de selección de ubicación de almacenamiento de máquina virtual 
![Selección de ubicación de almacenamiento de máquina virtual](media/hyper-v-deployment-planner-analyze-report/vm-storage-placement-h2a.png)

**Disk Storage Type** (Tipo de almacenamiento en disco): cuenta de almacenamiento estándar o premium, que se usa para replicar todas las máquinas virtuales correspondientes que se mencionan en la columna **VMs to Place** (Máquinas virtuales para colocar).

**Suggested Prefix** (Prefijo sugerido): el prefijo de tres caracteres sugerido que se puede usar para asignar un nombre a la cuenta de almacenamiento. Puede usar su propio prefijo, pero el que sugiere la herramienta sigue la [convención de nomenclatura de particiones de las cuentas de almacenamiento](https://aka.ms/storage-performance-checklist).

**Suggested Account Name** (Nombre de cuenta sugerido): el nombre de la cuenta de almacenamiento después de incluir el prefijo sugerido. Reemplace el nombre entre corchetes angulares (< y >) por una entrada personalizada.

**Log Storage Account** (Cuenta de almacenamiento del registro): todos los registros de la replicación se almacenan en una cuenta de almacenamiento estándar. En el caso de las máquinas virtuales que se replican en una cuenta de almacenamiento Premium, configure una cuenta de almacenamiento Estándar adicional para el almacenamiento de registros. Varias cuentas de almacenamiento de replicación Premium puede usar una única cuenta de almacenamiento de registros Estándar. Las máquinas virtuales que se replican en las cuentas de almacenamiento Estándar usan la misma cuenta de almacenamiento para los registros.

**Suggested Log Account Name** (Nombre de cuenta de registros sugerido): el nombre de la cuenta de registro de almacenamiento después de incluir el prefijo sugerido. Reemplace el nombre entre corchetes angulares (< y >) por una entrada personalizada.

**Placement Summary** (Resumen de la selección de ubicación): un resumen de la carga total de máquinas virtuales en la cuenta de almacenamiento en el momento de la replicación y la conmutación por error real o de prueba. El resumen incluye:

* El número total de máquinas virtuales asignadas a la cuenta de almacenamiento. 
* IOPS totales de lectura y escritura en todas las máquinas virtuales colocadas en esta cuenta de almacenamiento.
* IOPS totales de escritura (replicación).
* Tamaño total de instalación en todos los discos.
* Número total de discos.

**VMs to Place** (Máquinas virtuales que se colocan): una lista de todas las máquinas virtuales que se deben colocar en la cuenta de almacenamiento especificada para que tanto el rendimiento como el uso sean óptimos.

## <a name="compatible-vms"></a>Máquinas virtuales compatibles
El informe de Excel generado por Azure Site Recovery Deployment Planner proporciona los detalles de todas las máquinas virtuales compatibles en la hoja "Compatible VMs" (Máquinas virtuales compatibles).

![Máquinas virtuales compatibles](media/hyper-v-deployment-planner-analyze-report/compatible-vms-h2a.png)

**VM Name** (Nombre de la máquina virtual): el nombre de la máquina virtual que se utiliza en VMListFile cuando se genera un informe. Esta columna también muestra los discos (discos duros virtuales) que están conectados a las máquinas virtuales. Los nombres incluyen los nombres de host de Hyper-V donde se colocaron las máquinas virtuales cuando la herramienta los detectó durante el período de generación de perfiles.

**VM Compatibility** (Compatibilidad de la máquina virtual): los valores son **Yes** (Sí) y **Yes**\* (Sí). **Sí**\* es para las instancias en las que la máquina virtual es una opción para [discos SSD Premium](../virtual-machines/disks-types.md). En este caso, la renovación elevada de generación de perfiles o el disco IOPS encaja en un tamaño de disco Premium mayor que el tamaño asignado al disco. La cuenta de almacenamiento decide a qué tipo de disco de almacenamiento Premium se asigna un disco, en función de su tamaño: 
* Menos de 128 GB es P10.
* De 128 GB a 256 GB es P15
* De 256 GB a 512 GB es P20.
* De 512 GB a 1024 GB es P30.
* De 1025 GB a 2048 GB es P40.
* De 2049 GB a 4095 GB es P50.

Por ejemplo, si las características de carga de trabajo de un disco lo colocan en la categoría P20 o P30, pero el tamaño lo asigna a un tipo de disco de almacenamiento Premium inferior, la herramienta marca la máquina virtual como **Sí**\*. La herramienta también recomienda que cambie el tamaño del disco de origen para poder encuadrarlo en el tipo de disco de almacenamiento Premium recomendado o que cambie el tipo de disco de destino después de la conmutación por error.

**Storage Type** (Tipo de almacenamiento): estándar o premium.

**Suggested Prefix** (Prefijo sugerido): el prefijo de tres caracteres de la cuenta de almacenamiento.

**Storage Account** (Cuenta de almacenamiento): el nombre que utiliza el prefijo sugerido para la cuenta de almacenamiento.

**Peak R/W IOPS (with Growth Factor)** (Valor máximo de IOPS de lectura y escritura [con factor de crecimiento]): el valor máximo de IOPS de lectura/escritura en carga de trabajo en el disco (el percentil 95 es el predeterminado), incluido el factor de crecimiento futuro (el valor predeterminado es el 30 %). Las IOPS totales de lectura y escritura de una máquina virtual no siempre es la suma de las IOPS de lectura y escritura de los discos individuales de la máquina virtual. Las IOPS máximas de lectura y escritura de las máquinas virtuales es el valor máximo de la suma de las IOPS de lectura y escritura de sus discos individuales durante cada minuto del período de generación de perfiles.

**Peak Data Churn in MB/s (with Growth Factor)** (Actividad de datos máxima en MB/s [con factor de crecimiento]): el máximo índice de actividad en el disco (el percentil 95 es el predeterminado), incluido el factor de crecimiento futuro (el valor predeterminado es el 30 %). La actividad de datos total de la máquina virtual no es siempre la suma de la actividad de datos de los discos individuales de la máquina virtual. La actividad de datos máxima de la máquina virtual es el valor máximo de la suma de la actividad de sus discos individuales durante cada minuto del período de generación de perfiles.

**Azure VM Size** (Tamaño de la máquina virtual de Azure): el tamaño ideal de la máquina virtual de Azure Cloud Services asignada para esta máquina virtual local. La asignación se basa en la memoria, el número de discos/núcleos/NIC y las IOPS de lectura y escritura de la máquina virtual local. La recomendación es usar siempre el menor tamaño de máquina virtual de Azure que cumpla todas las características de la máquina virtual local.

**Number of Disks** (Número de discos): el número total de discos (discos duros virtuales) de la máquina virtual.

**Tamaño del disco (GB)** : el tamaño total de todos los discos de la máquina virtual. La herramienta también muestra el tamaño de los discos individuales de la máquina virtual.

**Cores** (Núcleos): el número de núcleos de CPU en la máquina virtual.

**Memory (MB)** (Memoria [MB]): la memoria RAM de la máquina virtual.

**NICs** (Tarjetas NIC): el número de tarjetas NIC de la máquina virtual.

**Boot Type** (Tipo de arranque): el tipo de arranque de la máquina virtual. Puede ser BIOS o EFI.

## <a name="incompatible-vms"></a>Máquinas virtuales no compatibles
El informe de Excel generado por Azure Site Recovery Deployment Planner proporciona todos los detalles de las máquinas virtuales no compatibles en la hoja "Incompatible VMs" (Máquinas virtuales no compatibles).

![Máquinas virtuales no compatibles](media/hyper-v-deployment-planner-analyze-report/incompatible-vms-h2a.png)

**VM Name** (Nombre de la máquina virtual): el nombre de la máquina virtual que se utiliza en VMListFile cuando se genera un informe. Esta columna también muestra los discos (discos duros virtuales) que están conectados a las máquinas virtuales. Los nombres incluyen los nombres de host de Hyper-V donde se colocaron las máquinas virtuales cuando la herramienta los detectó durante el período de generación de perfiles.

**VM Compatibility** (Compatibilidad de la máquina virtual): indica el motivo por el que una máquina virtual dada no es compatible con Site Recovery. Se describen las razones de cada disco incompatible de la máquina virtual, que, en función de los [límites de almacenamiento](https://aka.ms/azure-storage-scalbility-performance) publicados, pueden ser cualesquiera de las siguientes:

* El tamaño del disco es mayor que 4095 GB. Azure Storage no admite actualmente discos de datos de más de 4095 GB.

* El disco del sistema operativo es superior a 2047 GB para las máquinas virtuales de primera generación (tipo de arranque BIOS). Site Recovery no admite un tamaño de disco del sistema operativo mayor de 2047 GB para máquinas virtuales de primera generación.

* El disco del sistema operativo es mayor que 300 GB para máquinas virtuales de segunda generación (tipo de arranque EFI). Site Recovery no admite un tamaño de disco del sistema operativo mayor de 300 GB para máquinas virtuales de segunda generación.

* Los nombres de máquina virtual no admiten ninguno de los siguientes caracteres: "" [] '. La herramienta no puede obtener datos de generación de perfiles de las máquinas virtuales que tengan alguno de estos caracteres en sus nombres. 

* Un disco duro virtual se comparte entre dos o más máquinas virtuales. Azure no admite máquinas virtuales con un disco duro virtual compartido.

* No se admite una máquina virtual con canal de fibra virtual. Site Recovery no admite máquinas virtuales con canal de fibra virtual.

* Un clúster de Hyper-V no contiene un agente de replicación. Site Recovery no admite una máquina virtual en un clúster de Hyper-V si el agente de réplica de Hyper-V no está configurado para el clúster.

* Una máquina virtual no ofrece alta disponibilidad. Site Recovery no admite una máquina virtual de un nodo de clúster de Hyper-V cuyos discos duros virtuales estén almacenados en el disco local en lugar de en el disco de clúster. 

* El tamaño total de la máquina virtual (replicación + conmutación por error de prueba) supera el límite de tamaño de la cuenta de almacenamiento Premium que se admite (35 TB). Esta incompatibilidad se produce normalmente cuando uno de los discos de la máquina virtual tiene una característica de rendimiento que supera los límites de almacenamiento estándar de Azure o de Site Recovery. Una instancia de este tipo coloca la máquina virtual en la zona de almacenamiento premium. Sin embargo, el tamaño máximo admitido de una cuenta de almacenamiento Premium es 35 TB. No se puede proteger una sola máquina virtual a través de varias cuentas de almacenamiento. 

    Cuando se ejecuta una conmutación por error de prueba en una máquina virtual protegida, si un disco no administrado está configurado para la conmutación por error de prueba, esta se ejecuta en la misma cuenta de almacenamiento en la que se lleva a cabo la replicación. En este caso, se requiere la misma cantidad de espacio adicional de almacenamiento que para la replicación. Se asegura que el progreso de la replicación y la conmutación por error de prueba sean correctos en paralelo. Cuando un disco administrado está configurado para la conmutación por error de prueba, no debe tenerse en cuenta ningún espacio adicional para la máquina virtual de conmutación por error de prueba.

* El valor de IOPS de origen supera el límite de IOPS de almacenamiento admitidas, 7500 por disco.

* El valor de IOPS de origen supera el límite de IOPS de almacenamiento admitidas, 80 000 por máquina virtual.

* La actividad de datos media de la máquina virtual de origen supera el límite que admite Site Recovery, 20 MB/s para el tamaño medio de E/S de disco.

* El valor medio de IOPS de escritura efectiva de la máquina virtual de origen supera el límite admitido de Site Recovery de 840.

* El almacenamiento de instantáneas estimado supera el límite admitido de 10 TB.

**Peak R/W IOPS (with Growth Factor)** (Valor máximo de IOPS de lectura y escritura [con factor de crecimiento]): el valor máximo de IOPS en carga de trabajo en el disco (el percentil 95 es el predeterminado) incluido el factor de crecimiento futuro (el valor predeterminado es el 30 %). Las IOPS totales de lectura y escritura de una máquina virtual no siempre es la suma de las IOPS de lectura y escritura de los discos individuales de la máquina virtual. Las IOPS máximas de lectura y escritura de las máquinas virtuales es el valor máximo de la suma de las IOPS de lectura y escritura de sus discos individuales durante cada minuto del período de generación de perfiles.

**Peak Data Churn MB/s (with Growth Factor)** (Actividad de datos máxima en MB/s [con factor de crecimiento]): el máximo índice de actividad en el disco (el percentil 95 es el predeterminado), incluido el factor de crecimiento futuro (el valor predeterminado es el 30 %). Tenga en cuenta que la actividad de datos total de la máquina virtual no es siempre la suma de la actividad de datos de los discos individuales de la máquina virtual. La actividad de datos máxima de la máquina virtual es el valor máximo de la suma de la actividad de sus discos individuales durante cada minuto del período de generación de perfiles.

**Number of Disks** (Número de discos): el número total de discos duros virtuales de la máquina virtual.

**Tamaño del disco (GB)** : el tamaño total configurado de todos los discos de la máquina virtual. La herramienta también muestra el tamaño de los discos individuales de la máquina virtual.

**Cores** (Núcleos): el número de núcleos de CPU en la máquina virtual.

**Memory (MB)** (Memoria [MB]): la cantidad de memoria RAM en la máquina virtual.

**NICs** (Tarjetas NIC): el número de tarjetas NIC de la máquina virtual.

**Boot Type** (Tipo de arranque): el tipo de arranque de la máquina virtual. Puede ser BIOS o EFI.

## <a name="azure-site-recovery-limits"></a>Límites de Azure Site Recovery
En la tabla siguiente se proporcionan los límites de Site Recovery. Estos límites se basan en pruebas, pero no pueden cubrir todas las combinaciones posibles de E/S de la aplicación. Los resultados reales pueden variar en función de la combinación de E/S de la aplicación. Para unos mejores resultados, incluso después del planeamiento de la implementación, realice pruebas exhaustivas de la aplicación mediante la emisión de una conmutación por error de prueba para obtener una imagen real del rendimiento de la aplicación.
 
**Destino de almacenamiento de la replicación** | **Tamaño medio de E/S de la máquina virtual de origen** |**Actividad de datos media de la máquina virtual de origen** | **Actividad de datos total de la máquina virtual de origen por día**
---|---|---|---
Standard Storage | 8 KB | 2 MB/s por máquina virtual | 168 GB por máquina virtual
Premium Storage | 8 KB  | 5 MB/s por máquina virtual | 421 GB por máquina virtual
Premium Storage | 16 KB o superior| 20 MB/s por máquina virtual | 1684 GB por máquina virtual

Estos límites son los números promedio si la superposición de E/S es del 30 %. Site Recovery es capaz de controlar un mayor rendimiento en función de la relación de superposición, tamaños de escritura mayores y el comportamiento real de E/S de la carga de trabajo. Los números anteriores asumen un trabajo pendiente típico de aproximadamente cinco minutos. Es decir, una vez que se cargan los datos, se procesan y se crea un punto de recuperación en menos de cinco minutos.

## <a name="on-premises-storage-requirement"></a>Requisitos de almacenamiento local

La hoja de cálculo proporciona los requisitos de espacio de almacenamiento libre total para cada volumen de los servidores de Hyper-V (donde residen los discos duros virtuales) para la replicación inicial correcta y la replicación diferencial. Antes de habilitar la replicación, agregue el espacio de almacenamiento necesario en los volúmenes para asegurarse de que no se va a ocasionar ningún tiempo de inactividad no deseado en las aplicaciones de producción. 

  Site Recovery Deployment Planner identifica los requisitos óptimos de espacio de almacenamiento según el tamaño de los discos duros virtuales y el ancho de banda de red utilizado para la replicación.

![Requisitos de almacenamiento local](media/hyper-v-deployment-planner-analyze-report/on-premises-storage-requirement-h2a.png)

### <a name="why-do-i-need-free-space-on-the-hyper-v-server-for-the-replication"></a>¿Por qué necesito espacio libre en el servidor de Hyper-V para la replicación?
* Cuando se habilita la replicación de una máquina virtual, Site Recovery toma una instantánea de cada disco duro virtual de la máquina virtual para la replicación inicial. Mientras que la replicación inicial está en progreso, la aplicación escribe nuevos cambios en los discos. Site Recovery realiza un seguimiento de estos cambios diferenciales en los archivos de registro, que requieren espacio de almacenamiento adicional. Hasta que se completa la replicación inicial, los archivos de registro se almacenan localmente. 

    Si no hay suficiente espacio para los archivos de registro y la instantánea (AVHDX), la replicación pasa al modo de resincronización y nunca se completa. En el peor de los casos, se necesita un 100 % de espacio libre adicional del tamaño del disco duro virtual para la replicación inicial.
* Una vez que la replicación inicial ha finalizado, se inicia la replicación diferencial. Site Recovery realiza un seguimiento de estos cambios diferenciales en los archivos de registro, que se almacenan en el volumen donde residen los discos duros virtuales de la máquina virtual. Estos archivos de registro se replican en Azure con una frecuencia de copia configurada. Según el ancho de banda de red disponible, la replicación de los archivos de registro en Azure tarda algún tiempo en realizarse. 

    Si no hay espacio libre suficiente para almacenar los archivos de registro, se pausa la replicación. A continuación, el estado de replicación de la máquina virtual entra en resincronización necesaria.
* Si el ancho de banda de red no es suficiente para insertar los archivos de registro en Azure, estos se apilan en el volumen. En el peor de los casos, cuando aumenta el tamaño de los archivos de registro al 50 % del tamaño del disco duro virtual, la replicación de la máquina virtual pasa a resincronización necesaria. En el peor de los casos, se necesita un 50 % de espacio libre adicional del tamaño del disco duro virtual para la replicación diferencial.

**Host de Hyper-V**: lista de servidores de Hyper-V para los que se ha generado perfil. Si un servidor forma parte de un clúster de Hyper-V, se agrupan todos los nodos del clúster.

**Volume (VHD path)** (Volumen [ruta de acceso al disco duro virtual]): cada volumen de un host de Hyper-V en el que están presentes los archivos VHD o VHDX. 

**Espacio libre disponible (GB)** : el espacio libre disponible en el volumen.

**Espacio total de almacenamiento necesario en el volumen (GB)** : el espacio de almacenamiento libre total requerido en el volumen para realizar correctamente la replicación inicial y la replicación diferencial. 

**Almacenamiento adicional total que se aprovisiona en el volumen para una replicación correcta (GB)** : recomienda el espacio total adicional que debe aprovisionarse en el volumen para realizar correctamente la replicación inicial y la replicación diferencial.

## <a name="initial-replication-batching"></a>Procesamiento por lotes de la replicación inicial 

### <a name="why-do-i-need-initial-replication-batching"></a>¿Por qué se necesita el procesamiento por lotes de la replicación inicial?
Si todas las máquinas virtuales se protegen al mismo tiempo, el requisito de espacio libre de almacenamiento es mucho mayor. Si no hay suficiente espacio de almacenamiento, la replicación de las máquinas virtuales entra en modo de resincronización. Además, el requisito de ancho de banda de red es mucho más alto para finalizar la replicación inicial de todas las máquinas virtuales juntas correctamente. 

### <a name="initial-replication-batching-for-a-selected-rpo"></a>Procesamiento por lotes de la replicación inicial para un RPO seleccionado
Esta hoja de cálculo proporciona la vista detallada de cada lote de replicación inicial (IR). Para cada RPO, se crean una hoja de procesamiento por lotes de replicación inicial independiente. 

Después de haber seguido la recomendación de los requisitos de almacenamiento local para cada volumen, la información principal que necesita para realizar la replicación es la lista de máquinas virtuales que se pueden proteger en paralelo. Estas máquinas virtuales se agrupan en un lote, y puede haber varios lotes. Proteja las máquinas virtuales en el orden de lote especificado. Primero proteja las máquinas virtuales del Lote 1. Tras la replicación inicial, proteja las máquinas virtuales del Lote 2 y así sucesivamente. Puede obtener la lista de lotes y máquinas virtuales correspondientes en esta hoja. 

![Detalles del procesamiento por lotes de la replicación inicial](media/hyper-v-deployment-planner-analyze-report/ir-batching-for-rpo-h2a.png)

![Detalles de procesamiento por lotes adicional de la replicación inicial](media/hyper-v-deployment-planner-analyze-report/ir-batching-for-rpo2-h2a.png)

### <a name="each-batch-provides-the-following-information"></a>Cada lote proporciona la siguiente información 
**Host de Hyper-V**: el host de Hyper-V de la máquina virtual que se protegerá.

**Máquina virtual**: la máquina virtual que se protegerá. 

**Comentarios**: si es necesaria cualquier acción para un volumen específico de una máquina virtual, aquí se proporciona el comentario. Por ejemplo, si no hay suficiente espacio libre en un volumen, el comentario dice que se agregue almacenamiento adicional para proteger esta máquina virtual.

**Volume (VHD path)** (Volumen [ruta de acceso al disco duro virtual]): el nombre del volumen donde residen los discos duros virtuales de la máquina virtual. 

**Espacio libre disponible en el volumen (GB)** : el espacio en disco libre disponible en el volumen para la máquina virtual. Al calcular el espacio libre disponible en los volúmenes, se considera el espacio en disco usado para la replicación diferencial por las máquinas virtuales de los lotes anteriores cuyos discos duros virtuales se encuentran en el mismo volumen. 

Por ejemplo, VM1, VM2 y VM3 residen en el volumen E:\VHDpath. Antes de la réplica, el espacio libre en el volumen es de 500 GB. VM1 forma parte del Lote 1, VM2 forma parte del Lote 2 y VM3 forma parte del Lote 3. Para VM1, el espacio libre disponible es de 500 GB. Para VM2, el espacio libre disponible sería 500 menos el espacio en disco necesario para la replicación diferencial de VM1. Si VM1 requiere 300 GB de espacio para la replicación diferencial, el espacio libre disponible para VM2 es 500 GB – 300 GB = 200 GB. De forma similar, VM2 requiere 300 GB para la replicación diferencial. El espacio libre disponible para VM3 es 200 GB - 300 GB = -100 GB.

**Almacenamiento necesario en el volumen para la replicación inicial (GB)** : el espacio de almacenamiento libre requerido en el volumen para la máquina virtual para la replicación inicial.

**Storage required on the volume for delta replication (GB)** (Almacenamiento necesario en el volumen para la replicación diferencial [GB]): el espacio de almacenamiento libre requerido en el volumen para la máquina virtual para la replicación diferencial.

**Additional storage required based on deficit to avoid replication failure (GB)** (Almacenamiento adicional requerido en función del déficit para evitar un error de replicación [GB]): el espacio adicional de almacenamiento necesario en el volumen para la máquina virtual. Es el valor máximo de los requisitos de espacio de almacenamiento de la replicación inicial y la replicación diferencial menos el espacio libre disponible en el volumen.

**Minimum bandwidth required for initial replication (Mbps)** (Ancho de banda mínimo necesario para la replicación inicial [Mbps]): el ancho de banda mínimo necesario para la replicación inicial para la máquina virtual.

**Minimum bandwidth required for delta replication (Mbps)** (Ancho de banda mínimo necesario para la replicación diferencial [Mbps]): el ancho de banda mínimo necesario para la replicación diferencial para la máquina virtual.

### <a name="network-utilization-details-for-each-batch"></a>Detalles de uso de red para cada lote 
Cada tabla de lotes proporciona un resumen del uso de red del lote.

**Bandwidth available for batch** (Ancho de banda disponible para lote): el ancho de banda disponible para el lote después de considerar el ancho de banda de la replicación diferencial del lote anterior.

**Approximate bandwidth available for initial replication of batch** (Ancho de banda aproximado disponible para la replicación inicial del lote): el ancho de banda disponible para la replicación inicial de las máquinas virtuales del lote. 

**Approximate bandwidth consumed for delta replication of batch** (Ancho de banda aproximado consumido para la replicación diferencial del lote): el ancho de banda necesario para la replicación diferencial de las máquinas virtuales del lote. 

**Estimated Initial Replication time for Batch (HH:MM)** (Tiempo de replicación inicial estimado para el lote [HH:MM]): el tiempo de replicación inicial estimado en horas: minutos.



## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre la [estimación de costos](hyper-v-deployment-planner-cost-estimation.md).

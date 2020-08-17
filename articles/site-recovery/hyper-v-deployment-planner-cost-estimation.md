---
title: Revisión del informe de estimación de costos de Azure Site Recovery Deployment Planner para la recuperación ante desastres de máquinas virtuales de Hyper-V en Azure | Microsoft Docs
description: En este artículo se describe cómo revisar el informe de estimación de costos generado por Azure Site Recovery Deployment Planner para la recuperación ante desastres de Hyper-V en Azure.
services: site-recovery
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 4/9/2019
ms.author: mayg
ms.openlocfilehash: 29457f2f5021fed9d8785f5764c4119de4be1fa9
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87089556"
---
# <a name="cost-estimation-report-by-azure-site-recovery-deployment-planner"></a>Informe de estimación de costos mediante Azure Site Recovery Deployment Planner 

El informe de Azure Site Recovery Deployment Planner ofrece el resumen de estimación de costos en las hojas de [recomendaciones](hyper-v-deployment-planner-analyze-report.md#recommendations) y el análisis detallado de los costos en la hoja de estimación de costos. Incluye el análisis detallado del costo por máquina virtual. 

### <a name="cost-estimation-summary"></a>Resumen de la estimación de costos 
En el gráfico siguiente se muestra la vista de resumen del costo total estimado de la recuperación ante desastres en Azure de la región de destino elegida y en la moneda que ha especificado para generar el informe.

![Resumen de la estimación de costos](media/hyper-v-azure-deployment-planner-cost-estimation/cost-estimation-summary-h2a.png)

El resumen le ayuda a comprender lo que debe pagar por los servicios de almacenamiento, proceso, red y licencia cuando protege las máquinas virtuales compatibles mediante Azure Site Recovery. El costo se calcula para las máquinas virtuales compatibles y no para todas las máquinas virtuales para las que se han generado perfiles. 
 
Puede ver el costo mensual o anual. Obtenga más información sobre las [regiones de destino admitidas](./hyper-v-deployment-planner-cost-estimation.md#supported-target-regions) y las [monedas admitidas](./hyper-v-deployment-planner-cost-estimation.md#supported-currencies).

**Cost by components** (Costo por componentes): el costo total de la recuperación ante desastres se divide entre cuatro componentes: proceso, almacenamiento, red y costo de la licencia de Site Recovery. El costo se calcula en función del consumo que se ha realizado durante la replicación y durante el simulacro de recuperación ante desastres. Para los cálculos se usa Compute, Storage (Premium y estándar), el circuito ExpressRoute o la red privada virtual que están configurados entre el sitio local y Azure, y la licencia de Site Recovery.

**Cost by states** (Costo por estados): la categoría del costo total de recuperación ante desastres se basa en dos estados diferentes: replicación y simulacro de recuperación ante desastres. 

**Replication cost** (Costo de replicación): el costo en el que se incurre durante la replicación. Incluye el costo de almacenamiento, red y de la licencia de Site Recovery. 

**DR-Drill cost** (Costo del simulacro de recuperación ante desastres): costo en el que se incurre durante las conmutaciones por error de prueba. Site Recovery pone en marcha las máquinas virtuales durante la conmutación por error de prueba. El costo del simulacro de recuperación ante desastres incluye el costo de los servicios de proceso y almacenamiento de las máquinas virtuales en ejecución. 

**Azure storage cost per Month/Year** (Costo de Azure Storage por mes/año): el costo total de almacenamiento en el que se incurre por el almacenamiento premium y estándar en la replicación y el simulacro de recuperación ante desastres.

## <a name="detailed-cost-analysis"></a>Análisis detallado del costo
Los precios de Azure para los recursos de proceso, almacenamiento y red varían en las distintas regiones de Azure. Puede generar un informe de estimación de costos con los precios de Azure más recientes en función de su suscripción, la oferta que está asociada a su suscripción y la región de destino de Azure en la moneda especificada. De forma predeterminada, la herramienta usa la región Oeste de EE. UU. 2 de Azure y la moneda dólares estadounidenses (USD). Si ha usado una región y moneda diferentes, la próxima vez que genere un informe sin identificador de suscripción, identificador de oferta, región de destino y moneda, la herramienta usa los precios de la última región de destino y la última moneda usadas para la estimación de costos.

En esta sección se muestra el identificador de suscripción y el identificador de oferta que usó para generar el informe. Si no se usan, están en blanco.

En todo el informe, las celdas marcadas en color gris son de solo lectura. Las celdas en blanco se pueden modificar según sus necesidades.

![Detalles de la estimación de costos](media/hyper-v-azure-deployment-planner-cost-estimation/cost-estimation1-h2a.png)

### <a name="overall-dr-costs-by-components"></a>Costo total de la recuperación ante desastres por componente
La primera sección muestra el costo total de la recuperación ante desastres por componente y por estado. 

**Proceso**: costo de las máquinas virtuales IaaS que se ejecutan en Azure para satisfacer las necesidades de recuperación ante desastres. Incluye las máquinas virtuales que se crean mediante Site Recovery durante los simulacros de recuperación ante desastres. También incluye las máquinas virtuales que se ejecutan en Azure, como SQL Server con grupos de disponibilidad AlwaysOn y controladores de dominio o servidores de nombres de dominio.

**Almacenamiento**: costo del consumo de almacenamiento de Azure para satisfacer las necesidades de recuperación ante desastres. Incluye el consumo de almacenamiento para la replicación y durante los simulacros de recuperación ante desastres.

**Network** (Red): costo de ExpressRoute y VPN de sitio a sitio para satisfacer las necesidades de recuperación ante desastres. 

**Licencia de Azure Site Recovery**: costo de la licencia de Azure Site Recovery para todas las máquinas virtuales compatibles. Si ha insertado manualmente una máquina virtual en la tabla de análisis detallado del costo, el costo de la licencia de Site Recovery también está incluido para esa máquina virtual.

### <a name="overall-dr-costs-by-states"></a>Overall DR cost by states (Costo total de la recuperación ante desastres por estado)
El costo total de la recuperación ante desastres se clasifica según dos estados diferentes: replicación y simulacro de recuperación ante desastres.

**Replication** (Replicación): el costo en el que se incurre en el momento de la replicación. Incluye el costo de almacenamiento, red y de la licencia de Site Recovery. 

**DR-Drill** (Costo del simulacro de recuperación ante desastres): el costo en el que se incurre en el momento del simulacro de recuperación ante desastres. Site Recovery pone en marcha las máquinas virtuales durante los simulacros de recuperación ante desastres. El costo del simulacro de recuperación ante desastres incluye el costo de los servicios de proceso y almacenamiento de las máquinas virtuales en ejecución.

* Duración total de los simulacros de recuperación ante desastres en un año = número de simulacros de recuperación ante desastres x duración de cada simulacro de recuperación ante desastres (días)
* Costo medio de simulacros de recuperación ante desastres (por mes) = costo total de simulacros de recuperación ante desastres/12

### <a name="storage-cost-table"></a>Tabla Storage cost (Costo de Table Storage)
En esta tabla se muestran los costos de almacenamiento Premium y estándar en los que se incurre por la replicación y los simulacros de recuperación ante desastres, con y sin descuento.

### <a name="site-to-azure-network"></a>Site to Azure network (Red de sitio a Azure)
Seleccione la opción adecuada según sus requisitos. 

**ExpressRoute**: de forma predeterminada, la herramienta selecciona el plan de ExpressRoute que más se aproxime al ancho de banda de red necesario para la replicación diferencial. Puede cambiar el plan de acuerdo con sus requisitos.

**VPN Gateway type** (Tipo de VPN Gateway): seleccione la instancia de Azure VPN Gateway si tiene alguna en su entorno. De forma predeterminada, es NA.

**Target region** (Región de destino): región de Azure especificada para la recuperación ante desastres. El precio usado en el informe para proceso, almacenamiento, red y licencia se basa en los precios de Azure para esa región. 

### <a name="vm-running-on-azure"></a>VM running on Azure (Máquina virtual que se ejecuta en Azure)
Quizás tenga un controlador de dominio, una máquina virtual DNS o una máquina virtual SQL Server con grupos de disponibilidad AlwaysOn que se ejecuta en Azure para la recuperación ante desastres. Puede proporcionar el número de máquinas virtuales y su tamaño para estimar el costo de computación del costo total de recuperación ante desastres. 

### <a name="apply-overall-discount-if-applicable"></a>Apply overall discount if applicable (Aplicar el descuento total si es aplicable)
Si es un asociado o un cliente de Azure y tiene derecho a un descuento sobre el precio general de Azure, puede usar este campo. La herramienta aplica el descuento (en %) en todos los componentes.

### <a name="number-of-virtual-machines-type-and-compute-cost-per-year"></a>Number of virtual machines type and compute cost (per year) [Número de tipo de máquinas virtuales y el cálculo de costo (anual)]
En esta tabla se muestra el número de máquinas virtuales Windows y no Windows, y los costos de proceso del simulacro de recuperación ante desastres en ellas.

### <a name="settings"></a>Configuración 
**Using Managed disk** (Uso de disco administrado): esta opción especifica si se usa un disco administrado en el momento del simulacro de recuperación ante desastres. El valor predeterminado es **Sí**. Si ha establecido **-UseManagedDisks** en **No**, se usa el precio de los discos no administrados para el cálculo de los costos.

**Moneda**: la moneda en la que se genera el informe.

**Cost duration** (Duración del costo): puede ver todos los costos del mes o de todo el año. 

## <a name="detailed-cost-analysis-table"></a>Tabla Detailed cost analysis (Análisis detallado del costo)
![Análisis detallado del costo](media/hyper-v-azure-deployment-planner-cost-estimation/detailed-cost-analysis-h2a.png)

En la tabla se muestra el desglose de los costos para cada máquina virtual compatible. También puede usar esta tabla para obtener el costo estimado de la recuperación ante desastres de Azure de las máquinas virtuales sin un perfil generado mediante la adición manual de máquinas virtuales. Esta información resulta útil cuando es necesario calcular los costos en Azure de una nueva implementación de recuperación ante desastres sin realizar una generación de perfiles detallada.

Para agregar las máquinas virtuales manualmente:

1. Seleccione **Insert row** (Insertar fila) para insertar una nueva fila entre las filas **Start** (Inicio) y **End** (Fin).

1. Rellene las columnas siguientes en función del tamaño aproximado de máquina virtual y del número de máquinas virtuales que coincidan con esta configuración: 

    a. **Number of VMs** (Número de máquinas virtuales)

    b. **IaaS size (Your selection)** (Tamaño de IaaS [su selección])

    c. **Storage type Standard/Premium** (Tipo de almacenamiento estándar/Premium)

    d. **VM total storage size (GB)** (Tamaño de almacenamiento total de máquinas virtuales [GB])

    e. **Number of DR drills in a year** (Número de simulacros de recuperación ante desastres en un año)

    f. **Each DR drill duration (Days)** (Duración de cada simulacro de recuperación ante desastres [días])

    g. **OS Type** (Tipo de SO)

    h. **Data redundancy** (Redundancia de datos)

    i. **Ventaja de uso híbrido de Azure**

1. Para aplicar el mismo valor a todas las máquinas virtuales de la tabla, seleccione **Apply to all** (Aplicar a todo) en **Number of DR-Drills in a year** (Número de simulacros de recuperación ante desastres en un año), **Each DR-Drill duration (Days)** (Duración de cada simulacro de recuperación ante desastres [días]), **Data redundancy** (Redundancia de datos) y **Azure Hybrid Use Benefit** (Ventaja para uso híbrido de Azure).

1. Seleccione **Re-calculate cost** (Volver a calcular los costos) para actualizar el costo.

**VM Name** (Nombre de la máquina virtual): el nombre de la máquina virtual.

**Number of VMs** (Número de máquinas virtuales): número de máquinas virtuales que coinciden con la configuración. Puede actualizar el número de máquinas virtuales existentes si no se han generado perfiles para una configuración similar de máquinas virtuales, pero están protegidas.

**IaaS size (Recommendation)** (Tamaño de IaaS [recomendado]): el tamaño de rol de máquina virtual de la máquina virtual compatible que recomienda la herramienta. 

**IaaS size (Your selection)** (Tamaño de IaaS [su selección]): de forma predeterminada, el tamaño es el mismo que el de rol de máquina virtual recomendado. Puede cambiar el rol para ajustarlo a sus requisitos. El costo de los servicios de proceso se basa en el tamaño de rol de máquina virtual seleccionado.

**Storage type** (Tipo de almacenamiento): el tipo de almacenamiento que usa la máquina virtual. Puede ser almacenamiento estándar o Premium.

**VM total storage size (GB)** (Tamaño de almacenamiento total de máquina virtual [GB]): el almacenamiento total de la máquina virtual.

**Number of DR-Drills in a year** (Número de simulacros de recuperación ante desastres en un año): el número de veces que se realizan los simulacros de recuperación en un año. De forma predeterminada, es cuatro veces en un año. Puede modificar el período para máquinas virtuales específicas o aplicar el nuevo valor a todas las máquinas virtuales. Escriba el nuevo valor en la fila superior y seleccione **Apply to all** (Aplicar a todo). El costo total de los simulacros de recuperación ante desastres se calcula en función del número de simulacros de recuperación ante desastres realizados en un año y de la duración de cada uno de ellos. 

**Each DR-Drill duration (Days)** (Duración de cada simulacro de recuperación ante desastres [días]): la duración de cada simulacro de recuperación ante desastres. De forma predeterminada, es de 7 días cada 90 días según la [ventaja de Software Assurance para la recuperación ante desastres](https://azure.microsoft.com/pricing/details/site-recovery). Puede modificar el período para máquinas virtuales específicas, o puede aplicar un nuevo valor a todas las máquinas virtuales. Escriba un nuevo valor en la fila superior y seleccione **Apply to all** (Aplicar a todo). El costo total de los simulacros de recuperación ante desastres se calcula en función del número de simulacros de recuperación ante desastres en un año y la duración de cada uno de ellos.
 
**OS Type** (Tipo de sistema operativo): el tipo de sistema operativo (SO) de la máquina virtual. Windows o Linux. Si el sistema operativo es Windows, se puede aplicar la Ventaja para uso híbrido de Azure a esa máquina virtual. 

**Data redundancy** (Redundancia de datos): puede ser almacenamiento con redundancia local, almacenamiento con redundancia geográfica o almacenamiento con redundancia geográfica con acceso de lectura. El valor predeterminado es almacenamiento con redundancia local. Puede cambiar el tipo en función de su cuenta de almacenamiento para máquinas virtuales específicas o puede aplicar el nuevo tipo a todas las máquinas virtuales. Cambie el tipo de la fila superior y seleccione **Apply to all** (Aplicar a todo). El costo de almacenamiento de replicación se calcula en función del precio de la redundancia de datos que ha seleccionado. 

**Azure Hybrid Use Benefit** (Ventaja para uso híbrido de Azure): puede aplicar la Ventaja para uso híbrido de Azure a las máquinas virtuales Windows, si procede. El valor predeterminado es **Sí**. Puede cambiar la configuración para máquinas virtuales específicas o puede actualizar todas las máquinas virtuales. Seleccione **Apply to all** (Aplicar a todo).

**Total Azure consumption** (Consumo total de Azure): incluye los costos de proceso, almacenamiento y licencia de Site Recovery de la recuperación ante desastres. En función de los valores seleccionados, se muestra el costo mensual o anual.

**Steady state replication cost** (Costo de replicación de estado estable): el costo de almacenamiento de la replicación.

**Total DR-Drill cost (average)** (Costo total de simulacro de recuperación ante desastres [medio]): el costo de almacenamiento y proceso de simulacros de recuperación ante desastres.

**Costo de la licencia de Azure Site Recovery**: el costo de licencia de Site Recovery.

## <a name="supported-target-regions"></a>Regiones de destino admitidas
Azure Site Recovery Deployment Planner proporciona una estimación de los costos para las siguientes regiones de Azure. Si su región no aparece en la lista, puede usar cualquiera de las siguientes cuyo precio sea más parecido al de la suya:

eastus, eastus2, westus, centralus, northcentralus, southcentralus, northeurope, westeurope, eastasia, southeastasia, japaneast, japanwest, australiaeast, australiasoutheast, brazilsouth, southindia, centralindia, westindia, canadacentral, canadaeast, westus2, westcentralus, uksouth, ukwest, koreacentral, koreasouth 

## <a name="supported-currencies"></a>Monedas admitidas
Azure Site Recovery Deployment Planner puede generar el informe de costos en cualquiera de las siguientes monedas.

|Moneda|Nombre|Moneda|Nombre|Moneda|Nombre|
|---|---|---|---|---|---|---|---|
|ARS|Peso argentino ($)|AUD|Dólar australiano ($)|BRL|Real brasileño (R$)|
|CAD|Dólar canadiense ($)|CHF|Franco suizo (CHF)|DKK|Corona danesa (kr)|
|EUR|Euro (€)|GBP|Libra británica (£)|HKD|Dólar de Hong Kong (HK$)|
|IDR|Rupia indonesia (Rp)|INR|Rupia india (₹)|JPY|Yen japonés (¥)|
|KRW|Won coreano (₩)|MXN|Peso mexicano (MX$)|MYR|Ringgit malayo (RM$)|
|NOK|Corona noruega (kr)||NZD|Dólar de Nueva Zelanda ($)||RUB|Rublo ruso (руб)|
|SAR|Saudi Riyal (SR)|SEK|Corona sueca (kr)|TWD|Dólar taiwanés (NT$)|
|TRY|Lira turca (TL)|USD| Dólar estadounidense (USD)|ZAR|Rand sudafricano (R)|

## <a name="next-steps"></a>Pasos siguientes
Más información sobre cómo proteger [máquinas virtuales de Hyper-V en Azure mediante Site Recovery](hyper-v-azure-tutorial.md).

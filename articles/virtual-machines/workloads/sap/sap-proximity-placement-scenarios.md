---
title: Grupos de selección de ubicación de proximidad de Azure para aplicaciones SAP | Microsoft Docs
description: Se describen los escenarios de implementación de SAP con grupos de selección de ubicación de proximidad de Azure.
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 01/17/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 7aa71062c86d57cabe8579e13011956137804f74
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87079798"
---
# <a name="azure-proximity-placement-groups-for-optimal-network-latency-with-sap-applications"></a>Grupos de selección de ubicación de proximidad de Azure para una latencia de red óptima con aplicaciones SAP
Las aplicaciones SAP basadas en la arquitectura SAP NetWeaver o SAP S/4HANA son sensibles a la latencia de red entre la capa de aplicación de SAP y el nivel de base de datos de SAP. Esta sensibilidad es el resultado de la mayor parte de la lógica de negocios que se ejecuta en el nivel de aplicación. Dado que el nivel de aplicación de SAP ejecuta la lógica de negocios, emite consultas al nivel de base de datos con una elevada frecuencia a una velocidad de miles o decenas de miles de consultas por segundo. En la mayoría de los casos, la naturaleza de estas consultas es sencilla. A menudo, se pueden ejecutar en el nivel de base de datos en 500 microsegundos o menos.

El tiempo invertido en la red para enviar una consulta de este tipo desde la capa de aplicación al nivel de base de datos y recibir el conjunto de resultados influye considerablemente en el tiempo que se tarda en ejecutar procesos empresariales. Esta sensibilidad a la latencia de red es la razón por la que se debe lograr una latencia de red óptima en proyectos de implementación de SAP. Consulte la [Nota de SAP 1100926 - Preguntas más frecuentes: rendimiento de red](https://launchpad.support.sap.com/#/notes/1100926/E) para obtener directrices para clasificar la latencia de red.

En muchas regiones de Azure, el número de centros de datos ha crecido. Este crecimiento también se ha desencadenado por la introducción de Availability Zones. Al mismo tiempo, los clientes, especialmente en el caso de los sistemas SAP de gama alta, usan SKU de VM más especiales de la familia de la serie M o de HANA (instancias grandes). Estos son tipos de máquinas virtuales de Azure que no están disponibles en todos los centros de datos de una región de Azure específica. Debido a estas dos tendencias, los clientes han experimentado una latencia de red que no está en el intervalo óptimo. En algunos casos, esta latencia produce un rendimiento poco óptimo de los sistemas SAP.

Para evitar estos problemas, Azure ofrece [grupos de selección de ubicación de proximidad](../../linux/co-location.md). Esta nueva funcionalidad ya se ha usado para implementar varios sistemas SAP. Para conocer las restricciones de los grupos de selección de ubicación de proximidad, consulte el artículo al que se hace referencia al principio de este párrafo. En este artículo se tratan los escenarios de SAP en los que se pueden o deben usar los grupos de selección de ubicación de proximidad de Azure.

## <a name="what-are-proximity-placement-groups"></a>¿Qué son los grupos de selección de ubicación de proximidad? 
Un grupo de selección de ubicación de proximidad de Azure es una construcción lógica. Cuando se define uno, se enlaza a una región de Azure y a un grupo de recursos de Azure. Cuando se implementan VM, las siguientes hacen referencia a un grupo de selección de ubicación de proximidad:

- La primera VM de Azure implementada en el centro de datos. La primera máquina virtual se puede ver como una "VM de ámbito" que se implementa en un centro de datos basado en algoritmos de asignación de Azure que en última instancia se combinan con las definiciones del usuario para una zona de disponibilidad específica.
- Todas las VM posteriores implementadas que hacen referencia al grupo de selección de ubicación de proximidad para colocar todas las VM de Azure implementadas posteriormente en el mismo centro de datos que la primera máquina virtual.

> [!NOTE]
> Si no hay ningún hardware de host implementado que pueda ejecutar un tipo de VM específica en el mismo centro de datos en el que se colocó la primera VM, la implementación del tipo de VM solicitada no se realizará correctamente. Recibirá un mensaje de error.

Un único [grupo de recursos de Azure](../../../azure-resource-manager/management/manage-resources-portal.md) puede tener varios grupos de selección de ubicación de proximidad asignados. Sin embargo, un grupo de selección de ubicación de proximidad solo se puede asignar a un grupo de recursos de Azure.

Cuando use grupos de selección de ubicación de proximidad, tenga en cuenta estas consideraciones:

- Al intentar conseguir un rendimiento óptimo para su sistema SAP y limitarse a un solo centro de datos de Azure para este sistema mediante el uso de grupos de selección de ubicación de proximidad, es posible que no pueda combinar todos los tipos de familias de VM dentro del grupo de selección de ubicación. Estas limitaciones se producen porque es posible que el hardware del host necesario para ejecutar un determinado tipo de VM no esté presente en el centro de datos en el que se implementó la "VM de ámbito" del grupo de selección de ubicación.
- Durante el ciclo de vida de un sistema SAP de este tipo, es posible que se vea obligado a trasladar el sistema a otro centro de datos. Este traslado podría ser necesario si decide que la capa de DBMS de HANA de escalado horizontal debe, por ejemplo, pasar de cuatro nodos a 16 y no hay suficiente capacidad para obtener 12 VM adicionales del tipo que ha usado en el centro de datos.
- Debido a la retirada de hardware, Microsoft podría crear capacidades para un tipo de VM que se usó en un centro de centros de datos diferente, en lugar de la que se usó inicialmente. En ese escenario, es posible que deba trasladar todas las VM del grupo de selección de ubicación de proximidad a otro centro de datos.

## <a name="proximity-placement-groups-with-sap-systems-that-use-only-azure-vms"></a>Grupos de selección de ubicación de proximidad con sistemas SAP que solo usan VM de Azure
En la mayoría de las implementaciones de sistemas SAP NetWeaver y S/4HANA en Azure, no se usa [HANA (instancias grandes)](./hana-overview-architecture.md). En el caso de las implementaciones que no usan HANA (instancias grandes), es importante proporcionar el rendimiento óptimo entre el nivel de aplicación de SAP y el nivel de DBMS. Para ello, defina un grupo de selección de ubicación de proximidad de Azure solo para el sistema.

En la mayoría de las implementaciones de cliente, los clientes crean un único [grupo de recursos de Azure](../../../azure-resource-manager/management/manage-resources-portal.md) para los sistemas SAP. En ese caso, existe una relación uno a uno entre, por ejemplo, los grupos de recursos del sistema ERP de producción y su grupo de selección de ubicación de proximidad. En otros casos, los clientes organizan los grupos de recursos horizontalmente y recopilan todos los sistemas de producción en un único grupo de recursos. En tal caso, tendría una relación uno a varios entre el grupo de recursos para sistemas de producción de SAP y varios grupos de selección de ubicación de proximidad de SAP ERP de producción, SAP BW, etc.

Evite la agrupación de varios sistemas de producción o de no producción de SAP en un solo grupo de selección de ubicación de proximidad. Cuando un pequeño número de sistemas SAP o un sistema SAP y algunas aplicaciones circundantes deben tener una comunicación de red de baja latencia, puede considerar la posibilidad de mover estos sistemas a un solo grupo de selección de ubicación de proximidad. Se deben evitar las agrupaciones de sistemas porque cuantos más sistemas se agrupen en un grupo de selección de ubicación de proximidad, mayor será la probabilidad de lo siguiente:

- Necesitará un tipo de VM que no se puede ejecutar en el centro de datos específico en el que se ha establecido el ámbito del grupo de selección de ubicación de proximidad.
- Los recursos de las VM que no son estándar, como las VM de la serie M, podrían no cumplirse cuando necesita más porque se está agregando software a un grupo de selección de ubicación de proximidad a lo largo del tiempo.

A continuación, se muestra el aspecto de la configuración ideal, según se describe:

![Grupos de selección de ubicación de proximidad solo con VM de Azure](./media/sap-proximity-placement-scenarios/ppg-for-all-azure-vms.png)

En este caso, se agrupan sistemas SAP únicos en un solo grupo de recursos, cada uno con un grupo de selección de ubicación de proximidad. No existe ninguna dependencia respecto a si se usan configuraciones de escalabilidad horizontal de HANA o de escalabilidad vertical de DBMS.

## <a name="proximity-placement-groups-and-hana-large-instances"></a>Grupos de selección de ubicación de proximidad y HANA (instancias grandes)
Si algunos de los sistemas SAP se basan en [HANA (instancias grandes)](./hana-overview-architecture.md) para el nivel de aplicación, se pueden experimentar mejoras considerables en la latencia de red entre la unidad de HANA (instancias grandes) y las VM de Azure al usar unidades de HANA (instancias grandes) que se han implementado en [sellos o filas de la revisión 4](./hana-network-architecture.md#networking-architecture-for-hana-large-instance). Una de las mejoras es que cuando se implementan unidades de HANA (instancias grandes), lo hacen con un grupo de selección de ubicación de proximidad. Puede usar ese grupo de selección de ubicación de proximidad para implementar las máquinas virtuales del nivel de aplicación. Como resultado, esas VM se implementarán en el mismo centro de datos que hospeda la unidad de HANA (instancias grandes).

Para determinar si la unidad de HANA (instancias grandes) está implementada en un sello o fila de la revisión 4, consulte el artículo [Control de instancias grandes de HANA en Azure mediante Azure Portal](./hana-li-portal.md#look-at-attributes-of-single-hli-unit). En la información general de los atributos de la unidad de HANA (instancias grandes), también puede determinar el nombre del grupo de selección de ubicación de proximidad porque se creó cuando se implementó la unidad de HANA (instancias grandes). El nombre que aparece en la información general de los atributos es el nombre del grupo de selección de ubicación de proximidad en el que debe implementar las VM del nivel de aplicación.

En comparación con los sistemas SAP que usan solo máquinas virtuales de Azure, cuando se usan instancias grandes de HANA, tiene menos flexibilidad a la hora de decidir el número de [grupos de recursos de Azure](../../../azure-resource-manager/management/manage-resources-portal.md) que se van a usar. Todas las unidades de HANA (instancias grandes) de un [inquilino de HANA (instancias grandes)](./hana-know-terms.md) se agrupan en un único grupo de recursos, tal como se describe en [este artículo](./hana-li-portal.md#display-of-hana-large-instance-units-in-the-azure-portal). A menos que realice una implementación en distintos inquilinos para separar, por ejemplo, sistemas de producción y de no producción u otros sistemas, todas las unidades de HANA (instancias grandes) se implementarán en un mismo inquilino de HANA (instancias grandes). Este inquilino tiene una relación uno a uno con un grupo de recursos. Sin embargo, se definirá un grupo de selección de ubicación de proximidad independiente para cada una de las unidades.

Como resultado, las relaciones entre grupos de recursos de Azure y grupos de selección de ubicación de proximidad para un solo inquilino tendrán el siguiente aspecto:

![Grupos de selección de ubicación de proximidad y HANA (instancias grandes)](./media/sap-proximity-placement-scenarios/ppg-for-hana-large-instance-units.png)

## <a name="example-of-deployment-with-proximity-placement-groups"></a>Ejemplo de implementación con grupos de selección de ubicación de proximidad
A continuación, se muestran algunos comandos de PowerShell que puede usar para implementar las VM con grupos de selección de ubicación de proximidad de Azure.

El primer paso, después de iniciar sesión en [Azure Cloud Shell](https://azure.microsoft.com/features/cloud-shell/), consiste en comprobar si se encuentra en la suscripción de Azure que quiere usar para la implementación:

<pre><code>
Get-AzureRmContext
</code></pre>

Si necesita cambiar a una suscripción diferente, puede ejecutar este comando para hacerlo:

<pre><code>
Set-AzureRmContext -Subscription "my PPG test subscription"
</code></pre>

Para crear un nuevo grupo de recursos de Azure, ejecute este comando:

<pre><code>
New-AzResourceGroup -Name "myfirstppgexercise" -Location "westus2"
</code></pre>

Para crear el nuevo grupo de selección de ubicación de proximidad, ejecute este comando:

<pre><code>
New-AzProximityPlacementGroup -ResourceGroupName "myfirstppgexercise" -Name "letsgetclose" -Location "westus2"
</code></pre>

Implemente su primera VM en el grupo de selección de ubicación de proximidad con un comando como este:

<pre><code>
New-AzVm -ResourceGroupName "myfirstppgexercise" -Name "myppganchorvm" -Location "westus2" -OpenPorts 80,3389 -ProximityPlacementGroup "letsgetclose" -Size "Standard_DS11_v2"
</code></pre>

El comando anterior implementa una VM basada en Windows. Después de que la implementación de la VM se realice correctamente, el ámbito del centro de datos del grupo de selección de ubicación de proximidad se define en la región de Azure. Todas las implementaciones de VM posteriores que hagan referencia al grupo de selección de ubicación de proximidad, como se muestra en el anterior comando, se implementarán en el mismo centro de datos de Azure, siempre que el tipo de VM se pueda hospedar en el hardware colocado en ese centro de datos y haya disponible capacidad para ese tipo de VM.

## <a name="combine-availability-sets-and-availability-zones-with-proximity-placement-groups"></a>Combinación de conjuntos de disponibilidad y Availability Zones con grupos de selección de ubicación de proximidad
Una de las desventajas de usar Availability Zones para implementaciones de sistemas SAP es que no se puede implementar el nivel de aplicación de SAP mediante conjuntos de disponibilidad dentro de la zona específica. Quiere que el nivel de aplicación de SAP se implemente en las mismas zonas que la capa de DBMS. La referencia a una zona de disponibilidad y un conjunto de disponibilidad cuando se implementa una sola VM no se admite. Por ello, anteriormente, se vio obligado a implementar el nivel de aplicación haciendo referencia a una zona. Perdió la capacidad de asegurarse de que las VM de nivel de aplicación se propagasen entre diferentes dominios de actualización y error.

Mediante los grupos de selección de ubicación de proximidad, puede evitar esta restricción. A continuación, se muestra la secuencia de implementación:

- Cree un grupo de selección de ubicación de proximidad.
- Implemente su VM de anclaje, normalmente el servidor DBMS, haciendo referencia a una determinada zona de disponibilidad.
- Cree un conjunto de disponibilidad que haga referencia al grupo de proximidad de Azure. (Vea el comando más adelante en este artículo).
- Implemente las VM del nivel de aplicación haciendo referencia al conjunto de disponibilidad y al grupo de selección de ubicación de proximidad.

En lugar de implementar la primera VM como se mostró en la sección anterior, haga referencia a una zona de disponibilidad y al grupo de selección de ubicación de proximidad al implementar la VM:

<pre><code>
New-AzVm -ResourceGroupName "myfirstppgexercise" -Name "myppganchorvm" -Location "westus2" -OpenPorts 80,3389 -Zone "1" -ProximityPlacementGroup "letsgetclose" -Size "Standard_E16_v3"
</code></pre>

Una implementación correcta de esta máquina virtual hospedaría la instancia de base de datos del sistema SAP en una zona de disponibilidad. El ámbito del grupo de selección de ubicación de proximidad se fija en uno de los centros de datos que representan la zona de disponibilidad que ha definido.

Suponemos que implementa las VM de servicios centrales de la misma manera que las VM de DBMS, haciendo referencia a las mismas zonas y a los mismos grupos de selección de ubicación de proximidad. En el paso siguiente, debe crear los conjuntos de disponibilidad que quiere usar para el nivel de aplicación de su sistema SAP.

Debe definir y crear el grupo de selección de ubicación de proximidad. El comando para crear el conjunto de disponibilidad requiere una referencia adicional al identificador del grupo de selección de ubicación de proximidad (no al nombre). Puede obtener el identificador del grupo de selección de ubicación de proximidad mediante este comando:

<pre><code>
Get-AzProximityPlacementGroup -ResourceGroupName "myfirstppgexercise" -Name "letsgetclose"
</code></pre>

Al crear el conjunto de disponibilidad, debe tener en cuenta parámetros adicionales al usar discos administrados (valores predeterminados, a menos que se especifique lo contrario) y grupos de selección de ubicación de proximidad:

<pre><code>
New-AzAvailabilitySet -ResourceGroupName "myfirstppgexercise" -Name "myppgavset" -Location "westus2" -ProximityPlacementGroupId "/subscriptions/my very long ppg id string" -sku "aligned" -PlatformUpdateDomainCount 3 -PlatformFaultDomainCount 2 
</code></pre>

Idealmente, debe utilizar tres dominios de error. Sin embargo, el número de dominios de error admitidos puede variar de una región a otra. En este caso, el número máximo de dominios de error posibles para las regiones específicas es dos. Para implementar las VM del nivel de aplicación, debe agregar una referencia al nombre del conjunto de disponibilidad y el nombre del grupo de selección de ubicación de proximidad, como se muestra aquí:

<pre><code>
New-AzVm -ResourceGroupName "myfirstppgexercise" -Name "myppgavsetappvm" -Location "westus2" -OpenPorts 80,3389 -AvailabilitySetName "myppgavset" -ProximityPlacementGroup "letsgetclose" -Size "Standard_DS11_v2"
</code></pre>

El resultado de esta implementación es:
- Una capa de DBMS y servicios centrales para el sistema SAP que se encuentra en una zona de disponibilidad específica o en Availability Zones.
- Un nivel de aplicación de SAP que se encuentra a través de conjuntos de disponibilidad en los mismos centros de datos de Azure que las VM de DBMS.

> [!NOTE]
> Dado que implementa una VM de DBMS en una zona y una segunda VM de DBMS en otra zona para crear una configuración de alta disponibilidad, será necesario disponer de grupos de selección de ubicación de proximidad diferentes para cada una de las zonas. Lo mismo se aplica a cualquier conjunto de disponibilidad que use.

## <a name="move-an-existing-system-into-proximity-placement-groups"></a>Traslado de un sistema existente en grupos de selección de ubicación de proximidad
Si ya tiene sistemas SAP implementados, es posible que quiera optimizar la latencia de red de algunos de los sistemas críticos y colocar el nivel de aplicación y la capa de DBMS en el mismo centro de datos. Para mover las máquinas virtuales de un conjunto de disponibilidad completo de Azure a un grupo de selección de ubicación de proximidad existente que ya está restringido, tiene que cerrar todas las máquinas virtuales del conjunto de disponibilidad y asignarlo al grupo de selección de ubicación de proximidad existente mediante Azure Portal, PowerShell o la CLI. Si desea trasladar una máquina virtual que no forma parte de un conjunto de disponibilidad a un grupo de selección de ubicación de proximidad existente, solo tiene que cerrarla y asignarla a ese grupo. 


## <a name="next-steps"></a>Pasos siguientes
Consulte la documentación:

- [Lista de comprobación de planeamiento e implementación de cargas de trabajo de SAP en Azure](./sap-deployment-checklist.md)
- [Vista previa: Implementación de máquinas virtuales en grupos de selección de ubicación de proximidad con la CLI de Azure](../../linux/proximity-placement-groups.md)
- [Vista previa: Implementación de máquinas virtuales en grupos de selección de ubicación de proximidad con PowerShell](../../windows/proximity-placement-groups.md)
- [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](./dbms_guide_general.md)

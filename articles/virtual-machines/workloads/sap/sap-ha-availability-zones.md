---
title: Configuraciones de cargas de trabajo de SAP con Azure Availability Zones | Microsoft Docs
description: Escenarios y arquitectura de alta disponibilidad para SAP NetWeaver con Azure Availability Zones
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 887caaec-02ba-4711-bd4d-204a7d16b32b
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 03/05/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 78a4a22771f7880c48722f410f3a2fae0c66e9c8
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87035798"
---
# <a name="sap-workload-configurations-with-azure-availability-zones"></a>Configuraciones de cargas de trabajo de SAP con Azure Availability Zones
Una de las características de alta disponibilidad que Azure ofrece es [Azure Availability Zones](../../../availability-zones/az-overview.md). El uso de Availability Zones mejora la disponibilidad general de las cargas de trabajo de SAP en Azure. Esta característica ya está disponible en algunas [regiones de Azure](https://azure.microsoft.com/global-infrastructure/regions/). En el futuro, estará disponible en más regiones.

Este gráfico muestra la arquitectura básica de alta disponibilidad de SAP:

![Configuración de alta disponibilidad estándar](./media/sap-ha-availability-zones/standard-ha-config.png)

El nivel de aplicación de SAP se implementa en un [conjunto de disponibilidad](../../windows/manage-availability.md) de Azure. Para lograr alta disponibilidad de Servicios centrales de SAP, implementa dos máquinas virtuales en un conjunto de disponibilidad independiente. Use clústeres de conmutación por error de Windows Server o Pacemaker (Linux) como marco de alta disponibilidad con conmutación automática por error en el caso de problemas de infraestructura o software. Para más información acerca de estas implementaciones, consulte:

- [Agrupación de una instancia de ASCS/SCS de SAP en un clúster de conmutación por error de Windows con un disco compartido de clúster](./sap-high-availability-guide-wsfc-shared-disk.md)
- [Agrupación de una instancia de ASCS/SCS de SAP en un clúster de conmutación por error de Windows con un recurso compartido de archivos](./sap-high-availability-guide-wsfc-file-share.md)
- [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SUSE Linux Enterprise Server para aplicaciones de SAP](./high-availability-guide-suse.md)
- [Alta disponibilidad de Azure Virtual Machines para SAP NetWeaver en Red Hat Enterprise Linux](./high-availability-guide-rhel.md)

Se aplica una arquitectura similar a la capa de DBMS de SAP NetWeaver, S/4HANA o sistemas Hybris. Implementa la capa de DBMS en un modo activo/pasivo con una solución de clúster de conmutación por error para protegerse frente a errores de software o infraestructura. La solución de clúster de conmutación por error podría ser cualquier marco de conmutación por error específico de DBMS, clústeres de conmutación por error de Windows Server o Pacemaker.

Para implementar la misma arquitectura con Azure Availability Zones, deben realizarse algunos cambios en la arquitectura descrita anteriormente. En este artículo se describen estos cambios.

## <a name="considerations-for-deploying-across-availability-zones"></a>Consideraciones de implementación entre Availability Zones


Al usar Availability Zones, tenga en cuenta lo siguiente:

- No hay ninguna garantía relativa a las distancias entre distintas zonas de Availability Zones dentro de una región de Azure.
- Availability Zones no es una solución ideal de recuperación ante desastres. Los desastres naturales pueden provocar daños en zonas muy extensas de algunas regiones del mundo, incluidas las infraestructuras energéticas, que pueden resultar muy perjudicadas. Las distancias entre distintas zonas podrían no ser lo suficientemente grandes como para constituir una solución de recuperación ante desastres adecuada.
- La latencia de red entre las zonas de Availability Zones no es la misma en todas las regiones de Azure. Hay casos en los que puede implementar y ejecutar el nivel de aplicación de SAP en diferentes zonas porque la latencia de red de una zona a la máquina virtual de DBMS activa es aceptable. Sin embargo, en algunas regiones de Azure la latencia entre la máquina virtual de DBMS activa y la instancia de la aplicación de SAP, implementadas en zonas diferentes, puede no ser aceptable para los procesos empresariales de SAP. En estos casos, la arquitectura de implementación debe ser diferente con una arquitectura activa/activa para la aplicación o activa/pasiva si la latencia de red entre zonas es demasiado alta.
- Al decidir dónde usar Availability Zones, base la decisión en la latencia de red entre las zonas. La latencia de red desempeña un papel importante en dos áreas:
    - Latencia entre las dos instancias de DBMS que deben tener una replicación sincrónica. Cuanto mayor sea la latencia de red, más afectará a la escalabilidad de la carga de trabajo.
    - La diferencia de latencia de red entre una máquina virtual que ejecuta una instancia de diálogo de SAP en zona con la instancia de DBMS activa y una máquina virtual similar en otra zona. A medida que aumente esta diferencia, mayor será la influencia en el tiempo de ejecución de procesos empresariales y trabajos por lotes, en función de si se ejecutan en zona con DBMS o en una zona distinta.

Hay algunas restricciones al implementar máquinas virtuales de Azure entre zonas de Availability Zones y el establecimiento de las soluciones de conmutación por error en la misma región de Azure.

- Debe usar [Azure Managed Disks](https://azure.microsoft.com/services/managed-disks/) al implementar en Azure Availability Zones. 
- Se ha corregido la asignación de las enumeraciones de zona para las zonas físicas en cada suscripción de Azure. Si utiliza suscripciones diferentes para implementar los sistemas SAP, debe definir las zonas ideales para cada suscripción.
- No se pueden implementar conjuntos de disponibilidad de Azure en una zona de disponibilidad de Azure Availability Zones a menos que use un [grupo de selección de ubicación de proximidad de Azure](../../linux/co-location.md). La forma de implementar la capa de DBMS de SAP y los servicios centrales en todas las zonas y, al mismo tiempo, implementar el nivel de aplicación de SAP con conjuntos de disponibilidad y, además, lograr una proximidad en las máquinas virtuales está documentada en el artículo [Grupos de selección de ubicación de proximidad de Azure para una latencia de red óptima con aplicaciones SAP](sap-proximity-placement-scenarios.md). Si no aprovecha los grupos de selección de ubicación de proximidad de Azure, tiene que elegir uno u otro como marco de implementación para las máquinas virtuales.
- No puede usar [Azure Load Balancer Básico](../../../load-balancer/load-balancer-overview.md) para crear soluciones de clúster de conmutación por error basadas en los clústeres de conmutación por error de Windows Server o en Linux Pacemaker. En su lugar, deberá usar la [SKU de Azure Load Balancer estándar](../../../load-balancer/load-balancer-standard-availability-zones.md).



## <a name="the-ideal-availability-zones-combination"></a>Combinación ideal de Availability Zones
Antes de decidir cómo usar las zonas de Availability Zones, debe determinar:

- La latencia de red entre las tres zonas de una región de Azure. Esto le permitirá elegir las zonas con la latencia de red mínima en el tráfico de red entre zonas.
- La diferencia entre la latencia de máquina virtual a máquina virtual dentro de una de las zonas elegidas y la latencia de red entre las dos zonas seleccionadas.
- Una determinación que indique si los tipos de máquinas virtuales que tiene que implementar están disponibles en las dos zonas seleccionadas. Con algunas máquinas virtuales, especialmente las de la serie M, pueden producirse situaciones en las que algunas de las SKU están disponibles en solo dos de las tres zonas.

## <a name="network-latency-between-and-within-zones"></a>Latencia de red entre zonas y dentro de una zona
Para determinar la latencia entre las distintas zonas, debe hacer lo siguiente:

- Implemente la SKU de la máquina virtual que desea utilizar para la instancia de DBMS en las tres zonas. Asegúrese de que [Azure Accelerated Networking](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/) está habilitado al realizar esta medición.
- Tan pronto encuentre las dos zonas con la mínima latencia de red, implemente otras tres máquinas virtuales de la SKU de máquina virtual que desea usar como máquina virtual de nivel de aplicación en las tres zonas de disponibilidad. Mida la latencia de red de las dos máquinas virtuales de DBMS en las dos zonas de DBMS distintas que elija. 
- Use **niping** como herramienta de medición. Esta herramienta de SAP se describe en las notas de soporte técnico de SAP [#500235](https://launchpad.support.sap.com/#/notes/500235) y [#1100926](https://launchpad.support.sap.com/#/notes/1100926/E). Céntrese en los comandos documentados para medir la latencia. Dado que **ping** no funciona a través de las rutas de código Azure Accelerated Networking, no se recomienda que lo use.

No es necesario realizar estas pruebas manualmente. Puede encontrar un procedimiento de PowerShell llamado [prueba de latencia de zonas de disponibilidad](https://github.com/Azure/SAP-on-Azure-Scripts-and-Utilities/tree/master/AvZone-Latency-Test) que automatiza las pruebas de latencia descritas. 

En función de las medidas y la disponibilidad de las SKU de máquina virtual de Availability Zones, debe tomar las siguientes decisiones:

- Defina las zonas ideales para la capa de DBMS.
- Determine si desea distribuir el nivel de aplicación de SAP activo a una, dos o las tres zonas, en función de las diferencias de latencia de red dentro de una zona o entre zonas.
- Determine si desea implementar una configuración en modo activo/pasivo o activo/activo desde el punto de vista de una aplicación. (Estas configuraciones se explican posteriormente en este artículo).

Para tomar estas decisiones, tenga también en cuenta las recomendaciones sobre latencia de red de SAP documentadas en la nota de SAP [#1100926](https://launchpad.support.sap.com/#/notes/1100926/E).

> [!IMPORTANT]
> Las medidas y decisiones tomadas son válidas para la suscripción de Azure utilizada para realizar estas medidas. Si usa otra suscripción de Azure, deberá repetir las mediciones. La asignación de zonas enumeradas puede ser diferente para otra suscripción de Azure.


> [!IMPORTANT]
> Se espera que las medidas realizadas anteriormente muestren resultados distintos en cada región de Azure que admita [Availability Zones](../../../availability-zones/az-overview.md). Aunque los requisitos de latencia de red sean los mismos, puede que deba adaptar diferentes estrategias de implementación en distintas regiones de Azure, ya que la latencia de red entre zonas puede variar. En algunas regiones de Azure, la latencia de red entre las tres zonas diferentes puede variar significativamente. En otras regiones, la latencia de red entre las tres zonas diferentes puede ser más uniforme. La afirmación de que siempre hay una latencia de red entre 1 y 2 milisegundos no es correcta. La latencia de red entre las zonas de Availability Zones de las regiones de Azure no se puede generalizar.

## <a name="activeactive-deployment"></a>Implementación activa/activa
Esta arquitectura de implementación se denomina activa/activa porque se implementan los servidores de aplicaciones de SAP activos en dos o tres zonas. Los servicios centrales de SAP que usan la instancia que usa la replicación de puesta en cola se van a implementar entre dos zonas. Lo mismo sucede con la capa de DBMS, que se va a implementar entre las mismas zonas que el servicio central de SAP.

Para contemplar dicha configuración, debe encontrar las dos zonas de Availability Zones de su región que ofrezcan una latencia de red entre zonas que sea aceptable para la carga de trabajo y la replicación de DBMS sincrónica. Además, desea asegurarse de que la diferencia entre la latencia de red dentro de las zonas seleccionadas y la latencia de red entre zonas no sea demasiado grande. Esto se debe a que no desea que haya variaciones demasiado grandes, dependiendo del hecho de que un trabajo se ejecute dentro de una zona con el servidor de DBMS o entre distintas zonas, en los tiempos de ejecución de los procesos empresariales o los trabajos por lotes. Algunas variaciones son aceptables, pero no los factores de diferencia.

Un esquema simplificado de una implementación activa/activa entre dos zonas podría ser similar a este:

![Implementación activa/activa](./media/sap-ha-availability-zones/active_active_zones_deployment.png)

Las siguientes consideraciones se aplican a esta configuración:

- Si no usa ningún [grupo de selección de ubicación de proximidad de Azure](../../linux/co-location.md), trate las zonas de Azure Availability Zones como dominios de error y actualización para todas las máquinas virtuales, ya que los conjuntos de disponibilidad no se pueden implementar en Azure Availability Zones.
- Si quiere combinar implementaciones de zonas para la capa de DBMS y los servicios centrales, pero quiere usar conjuntos de disponibilidad de Azure para el nivel de aplicación, tiene que usar los grupos de proximidad de Azure, tal como se describe en el artículo [Grupos de selección de ubicación de proximidad de Azure para una latencia de red óptima con aplicaciones SAP](sap-proximity-placement-scenarios.md).
- Para los equilibradores de carga de los clústeres de conmutación por error de los servicios centrales de SAP y la capa de DBMS, necesita instancias de [Azure Load Balancer de SKU estándar](../../../load-balancer/load-balancer-standard-availability-zones.md). La instancia básica de Load Balancer no funcionará entre zonas.
- La red virtual de Azure implementada para hospedar el sistema SAP, junto con sus subredes, se extienden entre zonas. No necesita separar las redes virtuales para cada zona.
- Para todas las máquinas virtuales implementadas, debe usar [Azure Managed Disks](https://azure.microsoft.com/services/managed-disks/). Los discos no administrados no se admiten para implementaciones zonales.
- Azure Premium Storage o el [almacenamiento SSD Ultra](../../windows/disks-types.md#ultra-disk) no admiten ningún tipo de replicación de almacenamiento entre zonas. La aplicación (DBMS o Servicios centrales de SAP) debe replicar los datos importantes.
- Ocurre lo mismo con el directorio compartido sapmnt, que es un disco compartido (Windows), un recurso compartido CIFS (Windows) o un recurso compartido NFS (Linux). Debe utilizar una tecnología que replique dichos discos compartidos o recursos compartidos entre las zonas. Estas tecnologías son compatibles con:
  - Para Windows, una solución de clúster que usa SIOS Datakeeper como se documenta en [Agrupación de una instancia de ASCS/SCS de SAP en un clúster de conmutación por error de Windows con un disco compartido de clúster en Azure](./sap-high-availability-guide-wsfc-shared-disk.md).
  - Para SUSE Linux, un recurso compartido NFS como se documenta en [Alta disponibilidad para NFS en máquinas virtuales de Azure en SUSE Linux Enterprise Server](./high-availability-guide-suse-nfs.md).
    
    Actualmente, la solución que usa el servidor de archivos de escalabilidad horizontal de Microsoft, documentada en [Preparación de la infraestructura de Azure para la alta disponibilidad de SAP con un clúster de conmutación por error de Windows y el recurso compartido de archivos para instancias de SAP ASCS/SCS](./sap-high-availability-infrastructure-wsfc-file-share.md), no se admite entre zonas.
- La tercera zona se usa para hospedar el dispositivo SBD en caso de que compile un [clúster de Pacemaker en SUSE Linux](./high-availability-guide-suse-pacemaker.md#create-azure-fence-agent-stonith-device) o instancias de aplicación adicionales.
- Para lograr la coherencia del tiempo de ejecución para algunos trabajos o transacciones empresariales críticos, puede intentar dirigir determinados trabajos por lotes y usuarios directamente a instancias de aplicación específicas que se encuentran en la zona con la instancia de DBMS activa con el uso de grupos de servidor por lotes de SAP, grupos de inicio de sesión SAP o grupos de RFC. Sin embargo, en el caso de una conmutación por error zonal, debe mover manualmente estos grupos a instancias que se ejecutan en VM en zona con la VM de la base de datos activa.  
- Es posible que desee implementar instancias de diálogo inactivas en cada una de las zonas. Esto sirve para habilitar una devolución inmediata a la capacidad de recursos anterior si una zona que se usa por parte de las instancias de la aplicación se queda fuera de servicio.

> [!IMPORTANT]
> En este escenario activo/activo, Microsoft anuncia que a partir del 1 de abril de 2020 habrá cargos adicionales por el ancho de banda. Consulte el documento [Detalles de precios de ancho de banda](https://azure.microsoft.com/pricing/details/bandwidth/). La transferencia de datos entre la capa de aplicación de SAP y la capa de DBMS de SAP es bastante intensiva. Por consiguiente, el escenario activo/activo puede contribuir al aumento de costos. Para conocer los costos exactos, siga leyendo este artículo 


## <a name="activepassive-deployment"></a>Implementación activa/pasiva
Si no encuentra una diferencia aceptable entre la latencia de red dentro de una zona y el tráfico de red entre zonas, puede implementar una arquitectura que tenga un carácter activo/pasivo desde el punto de vista del nivel de aplicación de SAP. Defina una zona *activa*, que es la zona en la que implementa el nivel de aplicación completo y donde intenta ejecutar la instancia de DBMS activa y la instancia de Servicios centrales de SAP. Con dicha configuración,tiene que asegurarse de que no haya una diferencia muy elevada del tiempo de ejecución en las transacciones empresariales y los trabajos por lotes, en función de si un trabajo se ejecuta en zona con la instancia de DBMS activa o no.

El diseño básico de este tipo de arquitectura tiene el siguiente aspecto:

![Implementación de zona activa/pasiva](./media/sap-ha-availability-zones/active_passive_zones_deployment.png)

Las siguientes consideraciones se aplican a esta configuración:

- Los conjuntos de disponibilidad no se pueden implementar en Azure Availability Zones. Como compensación, puede usar grupos de ubicación de proximidad de Azure, tal como se documenta en el artículo [Grupos de selección de ubicación de proximidad de Azure para una latencia de red óptima con aplicaciones SAP](sap-proximity-placement-scenarios.md).
- Al utilizar esta arquitectura, debe realizar una estrecha supervisión del estado y tratar de mantener las instancias de servicios centrales de SAP y DBMS activas en la misma zona en la que implementó el nivel de aplicación. En el caso de una conmutación por error de la instancia de DBMS o del servicio central de SAP, desea asegurarse de que puede realizar una conmutación por recuperación manual en la zona con el nivel de aplicación de SAP implementado lo antes posible.
- Para los equilibradores de carga de los clústeres de conmutación por error de los servicios centrales de SAP y la capa de DBMS, necesita instancias de [Azure Load Balancer de SKU estándar](../../../load-balancer/load-balancer-standard-availability-zones.md). La instancia básica de Load Balancer no funcionará entre zonas.
- La red virtual de Azure implementada para hospedar el sistema SAP, junto con sus subredes, se extienden entre zonas. No necesita separar las redes virtuales para cada zona.
- Para todas las máquinas virtuales implementadas, debe usar [Azure Managed Disks](https://azure.microsoft.com/services/managed-disks/). Los discos no administrados no se admiten para implementaciones zonales.
- Azure Premium Storage o el [almacenamiento SSD Ultra](../../windows/disks-types.md#ultra-disk) no admiten ningún tipo de replicación de almacenamiento entre zonas. La aplicación (DBMS o Servicios centrales de SAP) debe replicar los datos importantes.
- Ocurre lo mismo con el directorio compartido sapmnt, que es un disco compartido (Windows), un recurso compartido CIFS (Windows) o un recurso compartido NFS (Linux). Debe utilizar una tecnología que replique dichos discos compartidos o recursos compartidos entre las zonas. Estas tecnologías son compatibles con:
    - Para Windows, una solución de clúster que usa SIOS Datakeeper como se documenta en [Agrupación de una instancia de ASCS/SCS de SAP en un clúster de conmutación por error de Windows con un disco compartido de clúster en Azure](./sap-high-availability-guide-wsfc-shared-disk.md).
    - Para SUSE Linux, un recurso compartido NFS como se documenta en [Alta disponibilidad para NFS en máquinas virtuales de Azure en SUSE Linux Enterprise Server](./high-availability-guide-suse-nfs.md).
    
  Actualmente, la solución que usa el servidor de archivos de escalabilidad horizontal de Microsoft, documentada en [Preparación de la infraestructura de Azure para la alta disponibilidad de SAP con un clúster de conmutación por error de Windows y el recurso compartido de archivos para instancias de SAP ASCS/SCS](./sap-high-availability-infrastructure-wsfc-file-share.md), no se admite entre zonas.
- La tercera zona se usa para hospedar el dispositivo SBD en caso de que compile un [clúster de Pacemaker en SUSE Linux](./high-availability-guide-suse-pacemaker.md#create-azure-fence-agent-stonith-device) o instancias de aplicación adicionales.
- Debe implementar máquinas virtuales inactivas en la zona pasiva (desde un punto de vista de DBMS) para poder iniciar los recursos de aplicación en caso de error de una zona.
    - [Azure Site Recovery](https://azure.microsoft.com/services/site-recovery/) no puede replicar actualmente máquinas virtuales activas en máquinas virtuales inactivas entre zonas. 
- Debe invertir en una automatización que le permita, en caso de error de una zona, iniciar automáticamente el nivel de aplicación de SAP en la segunda zona.

## <a name="combined-high-availability-and-disaster-recovery-configuration"></a>Configuración combinada de alta disponibilidad y recuperación ante desastres
Microsoft no comparte ninguna información sobre las distancias geográficas entre las instalaciones que hospedan zonas de Azure Availability Zones diferentes en la región de Azure. Aun así, algunos clientes están utilizando las zonas para una configuración de alta disponibilidad y recuperación ante desastres combinada que promete un objetivo de punto de recuperación (RPO) de cero. Esto significa que no debe perder ninguna transacción de base de datos confirmada incluso en el caso de una recuperación ante desastres. 

> [!NOTE]
> Se recomienda que utilice una configuración similar a la siguiente solo en determinadas circunstancias. Por ejemplo, se podría utilizar cuando los datos no puedan salir de la región de Azure por motivos de seguridad o cumplimiento. 

Este es un ejemplo del aspecto que podría tener este tipo de configuración:

![Recuperación ante desastres de alta disponibilidad combinada en zonas](./media/sap-ha-availability-zones/combined_ha_dr_in_zones.png)

Las siguientes consideraciones se aplican a esta configuración:

- Se supone que hay una distancia significativa entre las instalaciones que hospedan una zona de Availability Zones o se le obliga a permanecer dentro de una región determinada de Azure. Los conjuntos de disponibilidad no se pueden implementar en Azure Availability Zones. Como compensación, puede usar grupos de ubicación de proximidad de Azure, tal como se documenta en el artículo [Grupos de selección de ubicación de proximidad de Azure para una latencia de red óptima con aplicaciones SAP](sap-proximity-placement-scenarios.md).
- Al utilizar esta arquitectura, debe realizar una estrecha supervisión del estado y tratar de mantener las instancias de servicios centrales de SAP y DBMS activas en la misma zona en la que implementó el nivel de aplicación. En el caso de una conmutación por error de la instancia de DBMS o del servicio central de SAP, desea asegurarse de que puede realizar una conmutación por recuperación manual en la zona con el nivel de aplicación de SAP implementado lo antes posible.
- Debería tener instancias de aplicación de producción preinstaladas en las máquinas virtuales que ejecutan las instancias de aplicación activas de control de calidad.
- En caso de error zonal, apague las instancias de aplicación de control de calidad e inicia las instancias de producción en su lugar. Tenga en cuenta que necesita utilizar los nombres virtuales de las instancias de aplicación para hacer esta tarea.
- Para los equilibradores de carga de los clústeres de conmutación por error de los servicios centrales de SAP y la capa de DBMS, necesita instancias de [Azure Load Balancer de SKU estándar](../../../load-balancer/load-balancer-standard-availability-zones.md). La instancia básica de Load Balancer no funcionará entre zonas.
- La red virtual de Azure implementada para hospedar el sistema SAP, junto con sus subredes, se extienden entre zonas. No necesita separar las redes virtuales para cada zona.
- Para todas las máquinas virtuales implementadas, debe usar [Azure Managed Disks](https://azure.microsoft.com/services/managed-disks/). Los discos no administrados no se admiten para implementaciones zonales.
- Azure Premium Storage o el [almacenamiento SSD Ultra](../../windows/disks-types.md#ultra-disk) no admiten ningún tipo de replicación de almacenamiento entre zonas. La aplicación (DBMS o Servicios centrales de SAP) debe replicar los datos importantes.
- Ocurre lo mismo con el directorio compartido sapmnt, que es un disco compartido (Windows), un recurso compartido CIFS (Windows) o un recurso compartido NFS (Linux). Debe utilizar una tecnología que replique dichos discos compartidos o recursos compartidos entre las zonas. Estas tecnologías son compatibles con:
    - Para Windows, una solución de clúster que usa SIOS Datakeeper como se documenta en [Agrupación de una instancia de ASCS/SCS de SAP en un clúster de conmutación por error de Windows con un disco compartido de clúster en Azure](./sap-high-availability-guide-wsfc-shared-disk.md).
    - Para SUSE Linux, un recurso compartido NFS como se documenta en [Alta disponibilidad para NFS en máquinas virtuales de Azure en SUSE Linux Enterprise Server](./high-availability-guide-suse-nfs.md).

  Actualmente, la solución que usa el servidor de archivos de escalabilidad horizontal de Microsoft, documentada en [Preparación de la infraestructura de Azure para la alta disponibilidad de SAP con un clúster de conmutación por error de Windows y el recurso compartido de archivos para instancias de SAP ASCS/SCS](./sap-high-availability-infrastructure-wsfc-file-share.md), no se admite entre zonas.
- La tercera zona se usa para hospedar el dispositivo SBD en caso de que compile un [clúster de Pacemaker en SUSE Linux](./high-availability-guide-suse-pacemaker.md#create-azure-fence-agent-stonith-device) o instancias de aplicación adicionales.





## <a name="next-steps"></a>Pasos siguientes
Estos son algunos pasos para implementar entre zonas de Azure Availability Zones:

- [Agrupación de una instancia de ASCS/SCS de SAP en un clúster de conmutación por error de Windows con un disco compartido de clúster en Azure](./sap-high-availability-guide-wsfc-shared-disk.md)
- [Preparación de la infraestructura de Azure para la alta disponibilidad de SAP con un clúster de conmutación por error de Windows y el recurso compartido de archivos para instancias de SAP ASCS/SCS](./sap-high-availability-infrastructure-wsfc-file-share.md)

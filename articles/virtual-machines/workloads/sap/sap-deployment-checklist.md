---
title: Lista de comprobación de planeamiento e implementación de cargas de trabajo de SAP | Microsoft Docs
description: Lista de comprobación para planear las implementaciones de cargas de trabajo de SAP en Azure y realizar la implementación
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: patfilot
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 08/10/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: f195d4096baaa1d6a03b4b6c7c589ccef8fbd036
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88651733"
---
# <a name="sap-workloads-on-azure-planning-and-deployment-checklist"></a>Lista de comprobación de planeamiento e implementación de cargas de trabajo de SAP en Azure

Esta lista de comprobación está diseñada para los clientes que trasladan sus aplicaciones SAP NetWeaver, S/4HANA e Hybris a la infraestructura como servicio de Azure. La lista de comprobación debe ser revisada por un cliente o partner SAP a lo largo del proyecto. Es importante señalar que muchas de las comprobaciones se realizan al principio del proyecto y en la fase de planeamiento. Una vez realizada la implementación, pueden resultar complicado efectuar cambios elementales en la infraestructura de Azure o en las versiones de software de SAP implementadas.

Revise la lista de comprobación en los hitos clave del proyecto. De este modo, podrá detectar los pequeños problemas antes de que se conviertan en problemas serios y tendrá tiempo suficiente para volver a diseñar y probar los cambios necesarios. No considere que esta lista de comprobación está completa. En función de su situación, es posible que tenga que realizar muchas más comprobaciones.

La lista de comprobación no incluye tareas que son independientes de Azure. Por ejemplo, las interfaces de aplicaciones SAP cambian durante un traslado a la plataforma Azure o a un proveedor de hospedaje.

Esta lista de comprobación puede usarse también con los sistemas ya implementados. Es posible que se hayan agregado nuevas características, como el Acelerador de escritura y Availability Zones, y nuevos tipos de máquinas virtuales desde la implementación. Por lo tanto, resulta útil revisar la lista de comprobación periódicamente para asegurarse de que está al tanto de las nuevas características de la plataforma Azure.

## <a name="project-preparation-and-planning-phase"></a>Fase de preparación y planificación del proyecto
Durante esta fase, se planea la migración de la carga de trabajo de SAP a la plataforma Azure. Como mínimo, durante esta fase debe crear los siguientes documentos, además de definir y analizar los siguientes elementos de la migración:

1. Documento de diseño general. Este documento debe contener:
    - El inventario actual de componentes y aplicaciones de SAP, y el inventario de aplicaciones de destino para Azure.
    - Una matriz de asignación de responsabilidades (RACI) que defina las responsabilidades y las asignaciones de las partes implicadas. Empiece en un nivel general y continúe hacia niveles más pormenorizados durante el planeamiento y las primeras implementaciones.
    - La arquitectura general de la solución.
    - Una decisión sobre las regiones de Azure en las que se implementará. Consulte la [lista de regiones de Azure](https://azure.microsoft.com/global-infrastructure/regions/). Para conocer qué servicios están disponibles en cada región, consulte [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/).
    - La arquitectura de redes para conectarse desde el entorno local con Azure. Empiece a familiarizarse con el [plano técnico del centro de datos virtual de Azure](/azure/architecture/vdc/).
    - Los principios de seguridad para la ejecución de datos empresariales de gran impacto en Azure. Para obtener información sobre la seguridad de los datos, comience consultando la [documentación de seguridad de Azure](../../../security/index.yml).
2.  Documento de diseño técnico. Este documento debe contener:
    - Un diagrama de bloques de la solución.
    - El cálculo de tamaño de los componentes de proceso, almacenamiento y redes en Azure. Para conocer el tamaño de las máquinas virtuales de Azure en SAP, consulte la [nota 
    -  de SAP 1928533](https://launchpad.support.sap.com/#/notes/1928533).
    - La arquitectura de continuidad empresarial y recuperación ante desastres.
    - Información detallada sobre las versiones del sistema operativo, la base de datos, el kernel y los paquetes de soporte de SAP. El hecho de que SAP NetWeaver o S/4HANA admitan una versión de sistema operativo determinada no implica necesariamente que también la admitan las máquinas virtuales de Azure. Lo mismo puede decirse de las versiones de DBMS. Compruebe los siguientes orígenes para alinear y, si es necesario, actualizar las versiones de SAP, DBMS y sistema operativo para garantizar el soporte técnico de SAP y Azure. Para obtener soporte técnico completo de SAP y Microsoft debe tener combinaciones de versiones compatibles con SAP y Azure. Si es necesario, deberá planear la actualización de algunos componentes de software. Aquí encontrará más detalles sobre el software compatible de SAP, sistema operativo y DBMS:
        - [Nota de soporte técnico de SAP 1928533](https://launchpad.support.sap.com/#/notes/1928533). En esta nota se definen las versiones mínimas del sistema operativo compatibles con las máquinas virtuales de Azure. También se definen las versiones mínimas de base de datos necesarias para la mayoría de las bases de datos que no son HANA. Por último, se proporciona el cálculo de tamaño de SAP para los tipos de máquina virtual de Azure compatibles con SAP.
        - [Nota de soporte técnico de SAP 2015553](https://launchpad.support.sap.com/#/notes/2015553). En esta nota se definen las directivas de soporte técnico relativas a Azure Storage y la relación de soporte técnico necesaria con Microsoft.
        - [Nota de soporte técnico de SAP 2039619](https://launchpad.support.sap.com/#/notes/2039619). En esta nota se define la matriz de compatibilidad de Oracle para Azure. Oracle solo admite Windows y Oracle Linux como sistemas operativos invitados en Azure para cargas de trabajo de SAP. Esta declaración de soporte se aplica también en el nivel de aplicación de SAP que ejecuta instancias de SAP. No obstante, Oracle no permite utilizar una alta disponibilidad para SAP Central Services en Oracle Linux mediante Pacemaker. Si necesita tener una alta disponibilidad para ASCS en Oracle Linux, debe utilizar SIOS Protection Suite para Linux. Para obtener información detallada sobre la certificación de SAP, consulte la nota de soporte técnico de SAP [1662610 - Support details for SIOS Protection Suite for Linux](https://launchpad.support.sap.com/#/notes/1662610) (1662610: información de soporte técnico sobre SIOS Protection Suite para Linux). Para Windows, se admite la solución compatible con SAP de clústeres de conmutación por error de Windows Server para SAP Central Services en combinación con Oracle como nivel de DBMS.
        - [Nota de soporte técnico de SAP 2235581](https://launchpad.support.sap.com/#/notes/2235581). Esta nota proporciona la matriz de compatibilidad para SAP HANA en diferentes versiones del sistema operativo.
        - En el [sitio web de SAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) se indican las máquinas virtuales de Azure compatibles con SAP HANA y las [instancias grandes de HANA](./hana-overview-architecture.md).
        - [Matriz de disponibilidad de productos SAP](https://support.sap.com/en/).
        - [Nota de soporte técnico de SAP 2555629 - SAP HANA 2.0 Dynamic Tiering – Hypervisor and Cloud Support](https://launchpad.support.sap.com/#/notes/2555629) (SAP HANA 2.0 Dynamic Tiering: hipervisor y soporte en la nube)
        - [Nota de soporte técnico de SAP 1662610 - Support details for SIOS Protection Suite for Linux](https://launchpad.support.sap.com/#/notes/1662610) (Detalles de soporte técnico de SIOS Protection Suite para Linux)
        - Notas de SAP para otros productos específicos de SAP.     
    - Se admite el uso de configuraciones de clúster de varios SID para SAP Central Services en sistemas operativos invitados de Windows, SLES y RHEL en Azure. Tenga en cuenta que el radio de acción puede aumentar el número de ASCS/SCS que se coloca en un clúster de varios SID. Puede encontrar la documentación sobre el escenario de SO invitado correspondiente en estos artículos:
        - [Alta disponibilidad con varios identificadores de seguridad de instancia de ASCS/SCS de SAP para los clústeres de conmutación por error de Windows Server y el disco compartido en Azure](./sap-ascs-ha-multi-sid-wsfc-shared-disk.md)
        - [Alta disponibilidad con varios identificadores de seguridad de instancia SAP ASCS/SCS para los clústeres de conmutación por error de Windows Server y los recursos compartidos de archivos en Azure](./sap-ascs-ha-multi-sid-wsfc-file-share.md)
        - [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SUSE Linux Enterprise Server para SAP Applications: guía de varios SID](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-guide-suse-multi-sid)
        - [Alta disponibilidad para SAP NetWeaver en VM de Azure en Red Hat Enterprise Linux para SAP Applications: guía de varios SID](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-guide-rhel-multi-sid)
    - Arquitectura de alta disponibilidad y recuperación ante desastres.
        - Basándose en el RTO y el RPO, defina cómo debe ser la arquitectura de alta disponibilidad y recuperación ante desastres.
        - Para lograr alta disponibilidad en una zona, compruebe qué ofrece el DBMS deseado en Azure. La mayoría de los paquetes de DBMS ofrecen métodos sincrónicos de una espera activa sincrónica, nuestra recomendación para los sistemas de producción. Compruebe también la documentación relacionada con SAP para las diferentes bases de datos, empezando por [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](./dbms_guide_general.md) y documentos relacionados.
           No se admite el uso de clústeres de conmutación por error de Windows Server con configuración de disco compartido para el nivel de DBMS como, por ejemplo, se ha [descrito para SQL Server](/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server?view=sql-server-2017). En su lugar, use soluciones como:
           - [SQL Server Always On](/previous-versions/azure/virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-ps-sql-alwayson-availability-groups)
           - [Protección de datos de Oracle](../oracle/configure-oracle-dataguard.md)
           - [Replicación del sistema de HANA](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/b74e16a9e09541749a745f41246a065e.html)
        - Para la recuperación ante desastres en regiones de Azure, revise las soluciones que ofrecen los distintos proveedores de DBMS. La mayoría de ellos admiten replicación asincrónica o trasvase de registros.
        - Para el nivel de aplicación de SAP, determine si va a ejecutar sistemas de prueba de regresión empresarial, que en condiciones ideales son réplicas de las implementaciones de producción, en la misma región de Azure o en la región de recuperación ante desastres. En el segundo caso, puede utilizar ese sistema de regresión empresarial como destino de recuperación ante desastres para las implementaciones de producción.
        - Si decide que no desea incluir sistemas de no producción en el sitio de recuperación ante desastres, considere la posibilidad de usar Azure Site Recovery como método para replicar el nivel de aplicación de SAP en la región de recuperación ante desastres de Azure. Para más información, consulte [Configuración de la recuperación ante desastres para la implementación de una aplicación de SAP NetWeaver de niveles múltiples](../../../site-recovery/site-recovery-sap.md).
        - Si decide usar una configuración de HADR combinada mediante [Azure Availability Zones](../../../availability-zones/az-overview.md), familiarícese con las regiones de Azure donde hay zonas de disponibilidad. Tenga en cuenta también las restricciones que pueden surgir debido al aumento de las latencias de red entre dos zonas de disponibilidad.  
3.  Un inventario de todas las interfaces de SAP (SAP y no SAP).
4.  Diseño de servicios básicos. Este diseño debe incluir los siguientes elementos:
    - Diseño de Active Directory y DNS.
    - La topología de red dentro de Azure y la asignación de distintos sistemas SAP.
    - La estructura del [acceso basado en rol de Azure (Azure RBAC)](../../../role-based-access-control/overview.md) para los equipos que administran la infraestructura y las aplicaciones de SAP en Azure.
    - La topología del grupo de recursos.
    - La [estrategia de etiquetado](../../../azure-resource-manager/management/tag-resources.md#tags-and-billing).
    - La convención de nomenclatura para las máquinas virtuales y otros componentes de la infraestructura o los nombres lógicos.
5.  Contrato de soporte técnico Premier y Profesional de Microsoft. Identifique a su responsable técnico de cuenta (TAM) de Microsoft si tiene un contrato de soporte técnico Premier con Microsoft. Para conocer los requisitos de soporte técnico de SAP, consulte la [nota de soporte técnico de SAP 2015553](https://launchpad.support.sap.com/#/notes/2015553).
6.  El número de suscripciones de Azure y la cuota de núcleos para las suscripciones. [Abra solicitudes de soporte técnico para aumentar las cuotas de suscripciones de Azure](../../../azure-portal/supportability/resource-manager-core-quotas-request.md) según sea necesario.
7.  Plan de reducción y migración de datos para migrar datos de SAP a Azure. SAP dispone de instrucciones para los sistemas SAP NetWeaver sobre cómo limitar el volumen de grandes cantidades de datos. Consulte [esta guía de SAP](https://wiki.scn.sap.com/wiki/download/attachments/247399467/DVM_%20Guide_7.2.pdf?version=1&modificationDate=1549365516000&api=v2) sobre la administración de datos en sistemas ERP de SAP. Parte del contenido también se aplica a los sistemas NetWeaver y S/4HANA en general.
8.  Una estrategia de implementación automatizada. La automatización en las implementaciones de infraestructura en Azure tiene como objetivo implementar de manera determinista y obtener resultados deterministas. Muchos clientes usan scripts basados en PowerShell o en la CLI. Sin embargo, hay diversas tecnologías de código abierto que pueden usarse para implementar la infraestructura de Azure para SAP e incluso para instalar software de SAP. Puede encontrar ejemplos en GitHub:
    - [Automated SAP Deployments in Azure Cloud](https://github.com/Azure/sap-hana) (Implementaciones automatizadas de SAP en la nube de Azure)
    - [Instalación de SAP HANA](https://github.com/AzureCAT-GSI/SAP-HANA-ARM)
9.  Defina una cadencia periódica de revisión del diseño y la implementación entre usted como cliente, el integrador de sistemas, Microsoft y otras partes implicadas.


## <a name="pilot-phase-strongly-recommended"></a>Fase piloto (expresamente recomendada)
 
Puede ejecutar una prueba piloto antes o durante el planeamiento y la preparación del proyecto. También puede utilizar la fase piloto para probar las estrategias y los diseños realizados durante la fase de planeamiento y preparación. Además, puede ampliar la fase piloto para que sea una prueba de concepto real.

Se recomienda configurar y validar una solución completa de alta disponibilidad y recuperación ante desastres, así como el diseño de la seguridad durante una implementación piloto. Algunos clientes realizan pruebas de escalabilidad durante esta fase. Otros clientes usan implementaciones de sistemas de espacio aislado de SAP como fase piloto. Se supone que ya ha identificado un sistema que desea migrar a Azure para la prueba piloto.

1. Optimice la transferencia de datos a Azure. La opción óptima depende en gran medida del escenario específico. La transferencia desde el entorno local a través de [Azure ExpressRoute](https://azure.microsoft.com/services/expressroute/) es más rápida si el circuito ExpressRoute tiene suficiente ancho de banda. En otros escenarios, es más rápida la transferencia a través de Internet.
2. En el caso de una migración de plataforma heterogénea de SAP, que implica la exportación e importación de datos, pruebe y optimice las fases de exportación e importación. Para migraciones de gran tamaño en las que SQL Server es la plataforma de destino, puede encontrar [recomendaciones](https://techcommunity.microsoft.com/t5/Running-SAP-Applications-on-the/SAP-OS-DB-Migration-to-SQL-Server-8211-FAQ-v6-2-April-2017/ba-p/368070). Puede usar Migration Monitor/SWPM si no necesita una actualización de versión combinada. Puede usar el proceso [SAP DMO](https://blogs.sap.com/2013/11/29/database-migration-option-dmo-of-sum-introduction/) si combina la migración con una actualización de la versión de SAP. Para ello, debe cumplir ciertos requisitos respecto a la combinación de las plataforma DBMS de origen y destino. Este proceso se documenta en [Database Migration Option (DMO) of SUM 2.0 SP03](https://launchpad.support.sap.com/#/notes/2631152) (Opción de migración de base de datos [DMO] de SUM 2.0 SP03).
   1.  Rendimiento de la exportación al origen, la carga de archivos de exportación en Azure y la importación. Maximice la superposición entre la exportación y la importación.
   2.  Evalúe el volumen de la base de datos en las plataformas de origen y destino con el fin de ajustar el tamaño de la infraestructura.
   3.  Valide y optimice los tiempos.
1. Validación técnica.
   1. Tipos de VM.
        - Revise los recursos en las notas de soporte técnico de SAP, en el directorio de hardware de SAP HANA y, de nuevo, en SAP PAM. Asegúrese de que no hay cambios en las máquinas virtuales compatibles con Azure, las versiones admitidas del sistema operativo para esos tipos de máquinas virtuales, y las versiones admitidas de SAP y DBMS.
        - Valide de nuevo el tamaño de la aplicación y la infraestructura que implementa en Azure. Si traslada aplicaciones existentes, a menudo puede derivar los valores de SAPS necesarios de la infraestructura que utiliza y de la [página web del banco de pruebas de SAP](https://www.sap.com/dmc/exp/2018-benchmark-directory/#/sd), y compararlos con los números de SAPS que aparecen en la [nota de soporte técnico de SAP 1928533](https://launchpad.support.sap.com/#/notes/1928533). Tenga en cuenta también [este artículo sobre las puntuaciones SAPS](https://techcommunity.microsoft.com/t5/Running-SAP-Applications-on-the/SAPS-ratings-on-Azure-VMs-8211-where-to-look-and-where-you-can/ba-p/368208).
        - Evalúe y pruebe el tamaño de las máquinas virtuales de Azure en relación con el rendimiento máximo de almacenamiento y el rendimiento de la red en los tipos de máquina virtual que eligió en la fase de planeamiento. Puede encontrar los datos aquí:
           -  [Tamaños de las máquinas virtuales Windows en Azure](../../sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Es importante tener en cuenta el *rendimiento máximo de disco sin almacenamiento en caché* para ajustar el tamaño.
           -  [Tamaños de las máquinas virtuales Linux en Azure](../../sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Es importante tener en cuenta el *rendimiento máximo de disco sin almacenamiento en caché* para ajustar el tamaño.
   2. Almacenamiento.
        - Consulte el documento [Tipos de Azure Storage para una carga de trabajo de SAP](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/planning-guide-storage)
        - Como mínimo, use el [almacenamiento SSD estándar de Azure](../../disks-types.md#standard-ssd) para las máquinas virtuales que representan niveles de aplicación de SAP y para la implementación de los DBMS que no son sensibles al rendimiento.
        - En general, no se recomienda el uso de [discos HDD estándar de Azure](../../disks-types.md#standard-hdd).
        - Use [Azure Premium Storage](../../disks-types.md#premium-ssd) para todas las máquinas virtuales de DBMS que sean remotamente sensibles al rendimiento.
        - Utilice [discos administrados de Azure](https://azure.microsoft.com/services/managed-disks/).
        - Utilice el Acelerador de escritura de Azure para las unidades de registro DBMS con la serie M. Tenga en cuenta el uso y los límites del Acelerador de escritura, tal y como se documenta en [Acelerador de escritura](../../how-to-enable-write-accelerator.md).
        - Para ver los distintos tipos de DBMS, consulte la [documentación genérica de DBMS relacionada con SAP](./dbms_guide_general.md) y las documentaciones específicas de DBMS a las que apunta el documento genérico.
        - Para más información sobre SAP HANA, consulte [Configuraciones y operaciones de infraestructura de SAP HANA en Azure](./hana-vm-operations.md).
        - No monte discos de datos de Azure en una máquina virtual Linux de Azure mediante el identificador de dispositivo. En su lugar, use el identificador único universal (UUID). Tenga cuidado al utilizar herramientas gráficas para montar discos de datos de Azure, por ejemplo. Compruebe las entradas de /etc/fstab para asegurarse de que se usa el UUID para montar los discos. Puede encontrar más información en [este artículo](../../linux/attach-disk-portal.md#connect-to-the-linux-vm-to-mount-the-new-disk).
   3. Redes.
        - Pruebe y evalúe la infraestructura de red virtual y la distribución de las aplicaciones de SAP a través o dentro de las distintas redes virtuales de Azure.
        -  Evalúe la aplicación de una estrategia de arquitectura de red virtual de concentrador y radio o bien de una estrategia de microsegmentación dentro de una única red virtual Azure. Base esta evaluación en:
               1. Los costos del intercambio de datos entre [redes virtuales de Azure emparejadas](../../../virtual-network/virtual-network-peering-overview.md). Para información sobre los costos, consulte [Precios de Virtual Network](https://azure.microsoft.com/pricing/details/virtual-network/).
               2. Las ventajas de una desconexión rápida del emparejamiento entre las redes virtuales de Azure respecto al cambio del grupo de seguridad de red para aislar una subred dentro de una red virtual. Esta evaluación se aplica en los casos en los que las aplicaciones o las máquinas virtuales hospedadas en una subred de la red virtual se convierten en un riesgo para la seguridad.
                3. El registro y la auditoría centralizados del tráfico de red entre el entorno local, el mundo exterior y el centro de datos virtual que creó en Azure.
        - Evalúe y pruebe la ruta de acceso de datos entre el nivel de aplicación de SAP y el nivel de DBMS de SAP.
            -  No se admite la inclusión de [aplicaciones virtuales de red de Azure](https://azure.microsoft.com/solutions/network-appliances/) en la ruta de comunicación entre la aplicación de SAP y el nivel de DBMS de los sistemas SAP basados en SAP NetWeaver, Hybris o S/4HANA.
            -  No se admite la colocación del nivel de aplicación de SAP y del DBMS de SAP en diferentes redes virtuales de Azure que no están emparejadas.
            -  Puede usar [reglas de grupo de seguridad de aplicación y de grupo de seguridad de red](../../../virtual-network/security-overview.md) para definir rutas entre el nivel de aplicación de SAP y el nivel de DBMS de SAP.
        - Asegúrese de que se habilitan [redes aceleradas de Azure](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/) en las máquinas virtuales que se usan en el nivel de aplicación de SAP y en el nivel de DBMS de SAP. Tenga en cuenta que se necesitan distintos niveles de sistema operativo para admitir redes aceleradas en Azure:
            - Windows Server 2012 R2 o posterior.
            - SUSE Linux 12 SP3 o posterior.
            - RHEL 7.4 o posterior.
            - Oracle Linux 7.5. Si usa el kernel de RHCKL, se requiere la versión 3.10.0-862.13.1.el7. Si usa el kernel de Oracle UEK, se requiere la versión 5.
        - Pruebe y evalúe la latencia de red entre las máquinas virtuales del nivel de aplicación de SAP y las máquinas virtuales de DBMS según la notas de soporte técnico de SAP [500235](https://launchpad.support.sap.com/#/notes/500235) y [1100926](https://launchpad.support.sap.com/#/notes/1100926/E). Evalúe los resultados respecto a la guía de latencia de red de la [nota de soporte técnico de SAP 1100926](https://launchpad.support.sap.com/#/notes/1100926/E). La latencia de red debe estar en el intervalo de moderada a buena. Se aplican excepciones al tráfico entre las máquinas virtuales y las unidades de instancias grandes de HANA, tal y como se describe en [este artículo](./hana-network-architecture.md#networking-architecture-for-hana-large-instance).
        - Asegúrese de que las implementaciones de ILB se configuran para usar Direct Server Return. Este valor reducirá la latencia en los casos en que se usen los ILB de Azure para las configuraciones de alta disponibilidad en el nivel de DBMS.
        - Si usa Azure Load Balancer junto con sistemas operativos invitados de Linux, compruebe que el parámetro de red de Linux **net.ipv4.tcp_timestamps** esté establecido en **0**. Esta recomendación entra en conflicto con las recomendaciones de las versiones anteriores de [la nota de SAP 2382421](https://launchpad.support.sap.com/#/notes/2382421). Esta nota de SAP se ha actualizado para indicar que el parámetro debe establecerse en **0** para trabajar con equilibradores de carga de Azure.
        - Considere la posibilidad de usar los [grupos de selección de ubicación de proximidad de Azure](../../linux/co-location.md) para conseguir una latencia de red óptima. Para más información, consulte [Grupos de selección de ubicación de proximidad de Azure para una latencia de red óptima con aplicaciones SAP](sap-proximity-placement-scenarios.md).
   4. Implementaciones de alta disponibilidad y recuperación ante desastres.
        - Si implementa el nivel de aplicación de SAP sin definir una zona de disponibilidad de Azure concreta, asegúrese de que todas las máquinas virtuales que ejecutan instancias de diálogo SAP o instancias middleware de un único sistema SAP se implementan en un [conjunto de disponibilidad](../../windows/manage-availability.md).
        - Si no necesita alta disponibilidad para SAP Central Services y DBMS, puede implementar estas máquinas virtuales en el mismo conjunto de disponibilidad que el nivel de aplicación de SAP.
        - Si protege la alta disponibilidad de SAP Central Services y el nivel de DBMS con réplicas pasivas, sitúe los dos nodos de SAP Central Services en un único conjunto de disponibilidad independiente y los dos nodos de DBMS en otro conjunto de disponibilidad.
        - Si implementa en Azure Availability Zones, no puede usar los conjuntos de disponibilidad. Sin embargo, tiene que asegurarse de que implementa los nodos activo y pasivo de Central Services en dos zonas de disponibilidad diferentes. Use zonas de disponibilidad que tengan la menor latencia entre ellas.
          Tenga en cuenta que deberá usar [Azure Standard Load Balancer](../../../load-balancer/load-balancer-standard-availability-zones.md) en caso de que establezca clústeres de conmutación por error de Windows o Pacemaker para el nivel de SAP Central Services y DBMS en las zonas de disponibilidad. No se puede usar [Load Balancer básico](../../../load-balancer/load-balancer-overview.md) con implementaciones zonales.
   5. Configuración del tiempo de espera.
        - Compruebe los seguimientos de desarrollador de SAP NetWeaver de las instancias de SAP para asegurarse de que no se producen interrupciones de conexión entre el servidor de puesta en cola y los procesos de trabajo de SAP. Estas interrupciones de conexión se pueden evitar estableciendo estos dos parámetros del registro:
            - HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\KeepAliveTime = 120000. Para más información, consulte [KeepAliveTime](/previous-versions/windows/it-pro/windows-2000-server/cc957549(v=technet.10)).
            - HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\KeepAliveInterval = 120000. Para más información, consulte [KeepAliveInterval](/previous-versions/windows/it-pro/windows-2000-server/cc957548(v=technet.10)).
        - Para evitar tiempos de espera de la interfaz gráfica de usuario entre las interfaces GUI de SAP implementadas en un entorno local y los niveles de aplicación de SAP implementados en Azure, compruebe si están establecidos los parámetros siguientes en default.pfl o en el perfil de la instancia:
            - rdisp/keepalive_timeout = 3600
            - rdisp/keepalive = 20
        - Para evitar la interrupción de las conexiones establecidas entre el proceso de puesta en cola de SAP y los procesos de trabajo de SAP, debe establecer el parámetro **enque/encni/set_so_keepalive** en **true**. Consulte también la [nota de SAP 2743751](https://launchpad.support.sap.com/#/notes/2743751).  
        - Si usa una configuración de clúster de conmutación por error de Windows, asegúrese de que el tiempo para reaccionar en nodos que no responden se configura correctamente para Azure. En el artículo sobre el [ajuste de los umbrales de red en clúster de conmutación por error](https://techcommunity.microsoft.com/t5/Failover-Clustering/Tuning-Failover-Cluster-Network-Thresholds/ba-p/371834) se muestran los parámetros y cómo afectan a las sensibilidades de la conmutación por error. Suponiendo que los nodos del clúster estén en la misma subred, debe cambiar estos parámetros:
            - SameSubNetDelay = 2000
            - SameSubNetThreshold = 15
            - RoutingHistorylength = 30
    6. Correcciones o configuraciones del sistema operativo
        - Para ejecutar HANA en SAP, lea estas notas y documentos:
            -   [Nota de soporte técnico de SAP 2814271 - SAP HANA Backup fails on Azure with Checksum Error](https://launchpad.support.sap.com/#/notes/2814271) (No se puede realizar la copia de seguridad de SAP HANA en Azure debido a un error de suma de comprobación)
            -   [Nota de soporte técnico de SAP 2753418 - Potential Performance Degradation Due to Timer Fallback ](https://launchpad.support.sap.com/#/notes/2753418) (Posible degradación del rendimiento debido a la reserva del temporizador)
            -   [Nota de soporte técnico de SAP 2791572 - Performance Degradation Because of Missing VDSO Support For Hyper-V in Azure](https://launchpad.support.sap.com/#/notes/2791572) (Degradación del rendimiento debido a la falta de compatibilidad de VDSO con Hyper-V en Azure)
            -   [Nota de soporte técnico de SAP 2382421 - Optimizing the Network Configuration on HANA- and OS-Level](https://launchpad.support.sap.com/#/notes/2382421) (Optimización de la configuración de red en los niveles de HANA y de sistema operativo)
            -   [Nota de soporte técnico de SAP 2694118 - Red Hat Enterprise Linux HA Add-On on Azure](https://launchpad.support.sap.com/#/notes/2694118) (Complemento de alta disponibilidad de Red Hat Enterprise Linux en Azure)
            -   [Nota de soporte técnico de SAP 1984787 - SUSE LINUX Enterprise Server 12: Installation notes](https://launchpad.support.sap.com/#/notes/1984787) (SUSE LINUX Enterprise Server 12: notas de instalación)
            -   [SAP support note #2002167 - Red Hat Enterprise Linux 7.x: Installation and Upgrade](https://launchpad.support.sap.com/#/notes/0002002167) (Red Hat Enterprise Linux 7.x: instalación y actualización)
            -   [SAP support note #2292690 - SAP HANA DB: Recommended OS settings for RHEL 7](https://launchpad.support.sap.com/#/notes/0002292690) (2292690 - SAP HANA DB: configuraciones de sistema operativo recomendadas para RHEL 7)
            -   [Nota de soporte técnico de SAP 2772999 - Red Hat Enterprise Linux 8.x: Installation and Configuration](https://launchpad.support.sap.com/#/notes/2772999) (Red Hat Enterprise Linux 8.x: instalación y configuración)
            -   [Nota de soporte técnico de SAP 2777782 - SAP HANA DB: Recommended OS settings for RHEL 8](https://launchpad.support.sap.com/#/notes/2777782) (SAP HANA DB: configuración recomendada del sistema operativo para RHEL 8).
            -   [Nota de soporte técnico de SAP 2578899 – SUSE Linux Enterprise Server 15: Installation Note](https://launchpad.support.sap.com/#/notes/2578899) (SUSE Linux Enterprise Server 15: nota de instalación)
            -   [Nota de soporte técnico de SAP 2455582 - Linux: Ejecución de aplicaciones SAP compiladas con GCC 6.x](https://launchpad.support.sap.com/#/notes/0002455582)
            -    [Nota de soporte técnico de SAP 2729475 - HWCCT Failed with Error "Hypervisor is not supported" on Azure VMs certified for SAP HANA ](https://launchpad.support.sap.com/#/notes/2729475) (HWCCT: error "no se admite el hipervisor" en las máquinas virtuales de Azure certificadas para SAP HANA)
1. Pruebe los procedimientos de alta disponibilidad y recuperación ante desastres.
   1. Simule situaciones de conmutación por error apagando las máquinas virtuales (sistemas operativos invitados de Windows) o colocando los sistemas operativos en modo de pánico (sistemas operativos invitados de Linux). Este paso le ayudará a averiguar si las configuraciones de conmutación por error funcionan según lo previsto.
   1. Mida cuánto tiempo se tarda en ejecutar una conmutación por error. Si tarda demasiado, considere las siguientes opciones:
        - Para SUSE Linux, use dispositivos SBD en lugar del agente de barrera de Azure para acelerar la conmutación por error.
        - Para SAP HANA, si la recarga de datos tarda demasiado, considere la posibilidad de aprovisionar más ancho de banda de almacenamiento.
   3. Pruebe la secuencia de copia de seguridad y restauración, y el tiempo que lleva; realice correcciones si es necesario. Asegúrese de que las horas de copia de seguridad son suficientes. También debe probar las actividades de restauración y los tiempos. Asegúrese de que los tiempos de restauración están dentro del objetivo de tiempo de recuperación del contrato de nivel de servicio en los casos en que este objetivo se basa en un proceso de restauración de base de datos o de máquina virtual.
   4. Pruebe la funcionalidad y la arquitectura de la recuperación ante desastres entre regiones.
1. Comprobaciones de seguridad.
   1. Pruebe la validez de la arquitectura de control de acceso basado en rol de Azure (Azure RBAC). El objetivo es separar y limitar el acceso y los permisos de los diferentes equipos. Por ejemplo, los miembros del equipo de SAP Basis deberían poder implementar máquinas virtuales y asignar discos de Azure Storage a una red virtual Azure determinada. Este equipo, sin embargo, no debería poder crear sus propias redes virtuales ni modificar la configuración de las redes virtuales existentes. Por otra parte, los miembros del equipo de red no deberían poder implementar máquinas virtuales en redes virtuales donde se ejecuten máquinas virtuales de DBMS y aplicaciones de SAP. Tampoco deberían poder cambiar los atributos de las máquinas virtuales ni eliminar máquinas virtuales o discos.  
   1.  Compruebe que las reglas del [grupo de seguridad de red y de ASC](../../../virtual-network/security-overview.md) funcionan según lo previsto y blindan los recursos protegidos.
   1.  Asegúrese de que se cifren todos los recursos que deben cifrarse. Defina e implemente procesos para realizar copias de seguridad de los certificados, almacenarlos y acceder a ellos, y para restaurar las entidades cifradas.
   1.  Utilice [Azure Disk Encryption](../../../security/fundamentals/azure-disk-encryption-vms-vmss.md) para los discos del sistema operativo siempre que sea posible desde el punto de vista de la compatibilidad con el sistema operativo.
   1.  Asegúrese de que no está usando demasiados niveles de cifrado. En algunos casos, tiene sentido usar Azure Disk Encryption con uno de los métodos de Cifrado de datos transparente del DBMS para proteger distintos discos o componentes en el mismo servidor.  Por ejemplo, en un servidor de DBMS de SAP, Azure Disk Encryption (ADE) puede estar habilitado en el disco de arranque del sistema operativo (si el sistema operativo admite ADE) y en los discos de datos no utilizados por los archivos de persistencia de datos de DBMS.  Un ejemplo consiste en usar ADE en el disco que contiene las claves de cifrado TDE de DBMS.
1. Pruebas de rendimiento. Basándose en las medidas y el seguimiento de SAP, realice estas comparaciones en SAP:
   - Si procede, compare los diez principales informes en línea con la implementación actual.
   - Si procede, compare los diez principales trabajos por lotes con la implementación actual.
   - Compare las transferencias de datos a través de las interfaces en el sistema SAP. Céntrese en las interfaces en las que sepa que la transferencia se está llevando a cabo entre distintas ubicaciones; por ejemplo, desde el entorno local a Azure.


## <a name="non-production-phase"></a>Fase de no producción 
En esta fase se supone que, después de completar una prueba piloto o una prueba de concepto correctamente, se empiezan a implementar sistemas SAP que no son de producción en Azure. Incorpore en esta implementación todo lo que aprendió y experimentó durante la POC. Todos los criterios y los pasos indicados para las pruebas de concepto se aplican también a esta implementación.

En esta fase, normalmente se implementan sistemas de desarrollo, sistemas de pruebas unitarias y sistemas de pruebas de regresión empresarial en Azure. Se recomienda que al menos un sistema de no producción en una línea de aplicación de SAP tenga la configuración de alta disponibilidad completa que va a tener el futuro sistema de producción. Estos son algunos pasos adicionales que debe completar durante esta fase:  

1.  Antes de pasar los sistemas de la plataforma anterior a Azure, recopile datos de consumo de recursos, como el uso de CPU, el rendimiento de almacenamiento y los datos de E/S por segundo. Recopile estos datos especialmente en las unidades del nivel de DBMS, pero también en las unidades del nivel de aplicación. Mida también la latencia de la red y del almacenamiento.
2.  Registre los patrones de tiempo de uso de disponibilidad de los sistemas. El objetivo es averiguar si los sistemas de no producción deben estar disponibles permanentemente o si hay sistemas de no producción que se pueden apagar durante ciertas fases de una semana o un mes.
3.  Pruebe y determine si quiere crear sus propias imágenes de sistema operativo para las máquinas virtuales en Azure o si desea utilizar una imagen de Azure Shared Image Gallery. Si utiliza una imagen de Shared Image Gallery, asegúrese de usar una imagen que refleje el contrato de soporte técnico con el proveedor del sistema operativo. En el caso de algunos proveedores de sistemas operativos, Shared Image Gallery le permite traer sus propias imágenes de licencia. Para otras imágenes de sistema operativo, el soporte técnico se incluye en el precio presupuestado por Azure. Si decide crear sus propias imágenes de sistema operativo, encontrará documentación en estos artículos:
    -   [Creación de una imagen generalizada de una máquina virtual Windows implementada en Azure](../../windows/capture-image-resource.md)
    -   [Creación de una imagen generalizada de una máquina virtual Linux implementada en Azure](../../linux/capture-image.md)
3.  Si usa imágenes de SUSE o de Red Hat Linux procedentes de Shared Image Gallery, tendrá que usar las imágenes para SAP que proporcionan los proveedores de Linux en ella.
4.  Asegúrese de cumplir los requisitos de soporte técnico de SAP para los contratos de soporte técnico de Microsoft. Consulte la [nota de soporte técnico de SAP 2015553](https://launchpad.support.sap.com/#/notes/2015553). Para instancias grandes de HANA, consulte [Requisitos de incorporación](./hana-onboarding-requirements.md).
4.  Asegúrese de que las personas adecuadas obtienen las [notificaciones de mantenimiento planeado](https://azure.microsoft.com/blog/a-new-planned-maintenance-experience-for-your-virtual-machines/), para que pueda elegir los mejores tiempos de inactividad.
5.  Busque con cierta frecuencia presentaciones de Azure en canales como [Channel 9](https://channel9.msdn.com/), para conocer las nuevas funciones que pueden aplicarse a sus implementaciones.
6.  Compruebe las notas de SAP relacionadas con Azure, como la [nota de soporte técnico 1928533](https://launchpad.support.sap.com/#/notes/1928533), para informarse sobre nuevas SKU de máquina virtual y las nuevas versiones compatibles del sistema operativo y DBMS. Compare los precios de los nuevos tipos de máquinas virtuales con los tipos de máquinas virtuales anteriores, de modo que pueda implementar máquinas virtuales con la mejor relación precio/rendimiento.
7.  Vuelva a comprobar las notas de soporte técnico de SAP, el directorio de hardware de SAP HANA y SAP PAM. Asegúrese de que no hay cambios en las máquinas virtuales compatibles con Azure, las versiones admitidas del sistema operativo para esas máquinas virtuales, y las versiones admitidas de SAP y DBMS.
8.  Busque en [el sitio web de SAP](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) nuevas SKU con certificación HANA en Azure. Compare los precios de las nuevas SKU con las que planeó usar. Realice los cambios necesarios para usar las que ofrezcan la mejor relación precio/rendimiento.
9.  Adapte los scripts de implementación para usar los nuevos tipos de máquina virtual e incorporar las nuevas características de Azure que quiere usar.
10. Tras la implementación de la infraestructura, pruebe y evalúe la latencia de red entre las máquinas virtuales del nivel de aplicación de SAP y las máquinas virtuales de DBMS, según la notas de soporte técnico de SAP [500235](https://launchpad.support.sap.com/#/notes/500235) y [1100926](https://launchpad.support.sap.com/#/notes/1100926/E). Evalúe los resultados respecto a la guía de latencia de red de la [nota de soporte técnico de SAP 1100926](https://launchpad.support.sap.com/#/notes/1100926/E). La latencia de red debe estar en el intervalo de moderada a buena. Se aplican excepciones al tráfico entre las máquinas virtuales y las unidades de instancias grandes de HANA, tal y como se describe en [este artículo](./hana-network-architecture.md#networking-architecture-for-hana-large-instance). Asegúrese de que ninguna de las restricciones mencionadas en [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](./dbms_guide_general.md#azure-network-considerations) y [Configuraciones y operaciones de infraestructura de SAP HANA en Azure](./hana-vm-operations.md) se aplican a la implementación.
11. Asegúrese de que las máquinas virtuales se implementan en el [grupo de selección de ubicación de proximidad de Azure](../../linux/co-location.md) correcto, tal como se describe en [Grupos de selección de ubicación de proximidad de Azure para una latencia de red óptima con aplicaciones SAP](sap-proximity-placement-scenarios.md).
11. Antes de aplicar la carga de trabajo, realice las demás comprobaciones indicadas para la fase de prueba de concepto.
12. A medida que se aplique la carga de trabajo, registre el consumo de recursos de los sistemas en Azure. Compare este consumo con los registros de la plataforma anterior. Si observa grandes diferencias, ajuste el tamaño de las máquinas virtuales de las futuras implementaciones. Tenga en cuenta que si reduce el tamaño, también se reducirán los anchos de banda de almacenamiento y red de las máquinas virtuales.
    - [Tamaños de las máquinas virtuales Windows en Azure](../../sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
    - [Tamaños de las máquinas virtuales Linux en Azure](../../sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 
13. Realice pruebas con la funcionalidad y los procesos de copia del sistema. El objetivo es facilitar la copia de un sistema de desarrollo o un sistema de pruebas, para que los equipos de proyecto puedan obtener rápidamente los nuevos sistemas. Considere la posibilidad de usar [SAP LaMa](https://wiki.scn.sap.com/wiki/display/ATopics/SAP+Landscape+Management+%28SAP+LaMa%29+at+a+Glance) para estas tareas.
14. Optimice y perfeccione los accesos basados en rol, los permisos y los procesos de Azure de su equipo para asegurarse de conseguir la separación de funciones. Al mismo tiempo, asegúrese de que todos los equipos pueden realizar sus tareas en la infraestructura de Azure.
15. Practique, pruebe y documente los procedimientos de alta disponibilidad y recuperación ante desastres para permitir que su personal ejecute estas tareas. Identifique las deficiencias y adapte la nueva funcionalidad de Azure que va a integrar en las implementaciones.


## <a name="production-preparation-phase"></a>Fase de preparación de producción 
En esta fase recopilará lo que experimentó y aprendió durante las implementaciones que no son de producción y lo aplicará a las futuras implementaciones de producción. También deberá preparar el trabajo de la transferencia de datos entre la ubicación actual de hospedaje y Azure.

1.  Lleve a cabo las actualizaciones de versiones de SAP necesarias en los sistemas de producción antes de pasar a Azure.
1.  Póngase de acuerdo con los propietarios de empresas sobre las pruebas funcionales y empresariales que deben realizarse después de la migración del sistema de producción.
1.  Asegúrese de que todas estas pruebas se llevan a cabo con los sistemas de origen en la ubicación de hospedaje actual. Evite realizar pruebas por primera vez después de que el sistema se mueva a Azure.
1.  Pruebe el proceso de migración de los sistemas de producción a Azure. Si no traslada todos los sistemas de producción a Azure en el mismo plazo de tiempo, cree grupos de sistemas de producción que deben estar en la misma ubicación de hospedaje. Pruebe la migración de datos. Estos son algunos métodos habituales:
    - Use métodos de DBMS, como la copia de seguridad y restauración en combinación con SQL Server Always On, la replicación del sistema de HANA o el trasvase de registros para inicializar y sincronizar el contenido de base de datos en Azure.
    - Use la copia de seguridad y restauración para las bases de datos más pequeñas.
    - Use SAP Migration Monitor, integrado en SAP SWPM, para realizar migraciones heterogéneas.
    - Use el proceso [SAP DMO](https://blogs.sap.com/2013/11/29/database-migration-option-dmo-of-sum-introduction/) si tiene que combinar la migración con una actualización de la versión de SAP. Tenga en cuenta que no todas las combinaciones de DBMS de origen y de destino son compatibles. Encontrará más información en las notas de soporte técnico de SAP específicas para las diferentes versiones de DMO. Por ejemplo, [Database Migration Option (DMO) of SUM 2.0 SP04](https://launchpad.support.sap.com/#/notes/2644872) (Opción de migración de base de datos [DMO] de SUM 2.0 SP04).
    - Compruebe si el rendimiento de la transferencia de datos es mejor a través de Internet o de ExpressRoute, en caso de que tenga que mover copias de seguridad o archivos de exportación de SAP. Si va a mover datos a través de Internet, es posible que deba cambiar algunas de las reglas del grupo de seguridad de red o del grupo de seguridad de aplicación que necesitará aplicar en los futuros sistemas de producción.
1.  Antes de mover los sistemas de la plataforma anterior a Azure, recopile los datos de consumo de recursos. Entre los datos útiles se incluyen el uso de CPU, el rendimiento de almacenamiento y los datos de IOPS. Recopile estos datos especialmente en las unidades del nivel de DBMS, pero también en las unidades del nivel de aplicación. Mida también la latencia de la red y del almacenamiento.
1.  Vuelva a consultar las notas de soporte técnico de SAP y la configuración necesaria del sistema operativo, el directorio de hardware de SAP HANA, y SAP PAM. Asegúrese de que no hay cambios en las máquinas virtuales compatibles con Azure, las versiones admitidas del sistema operativo para esas máquinas virtuales, y las versiones admitidas de SAP y DBMS.
1.  Actualice los scripts de implementación para tener en cuenta las decisiones más recientes que ha adoptado en lo que respecta a los tipos de máquina virtual y la funcionalidad de Azure.
1.  Después de implementar la infraestructura y las aplicaciones, compruebe lo siguiente:
    - Se implementaron los tipos correctos de máquina virtual, con los atributos y los tamaños de almacenamiento correctos.
    - Las máquinas virtuales tienen las versiones de sistema operativo y las revisiones correctas que desea, y son uniformes.
    - Las máquinas virtuales están protegidas según es necesario y de forma uniforme.
    - Se han instalado e implementado las versiones y revisiones correctas de las aplicaciones.
    - Las máquinas virtuales se implementaron en conjuntos de disponibilidad de Azure según lo previsto.
    - Se usó Azure Premium Storage para los discos sensibles a la latencia o cuando se requiere el [contrato de nivel de servicio del 99,9 % para una única máquina virtual](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_8/).
    - El Acelerador de escritura de Azure se implementó correctamente.
        - Asegúrese de que, en las máquinas virtuales, se han creado correctamente los espacios de almacenamiento o conjuntos de franjas en los discos que necesitan Acelerador de escritura.
        - Compruebe la [configuración del software RAID en Linux](../../linux/configure-raid.md).
        - Compruebe la [configuración del LVM en las máquinas virtuales de Linux en Azure](../../linux/configure-lvm.md).
    - Se usaron exclusivamente [discos administrados de Azure](https://azure.microsoft.com/services/managed-disks/).
    - Las máquinas virtuales se implementaron en los conjuntos y las zonas de disponibilidad correctos.
    - Las [redes aceleradas de Azure](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/) están habilitadas en las máquinas virtuales que se usan en el nivel de aplicación de SAP y el nivel de DBMS de SAP.
    - No existen [aplicaciones virtuales de red de Azure](https://azure.microsoft.com/solutions/network-appliances/) en la ruta de comunicación entre el nivel de aplicación de SAP y de DBMS de un sistema SAP basado en SAP NetWeaver, Hybris o S/4HANA.
    - Las reglas de grupo de seguridad de aplicación y de grupo de seguridad de red permiten la comunicación deseada y planeada, y bloquean la comunicación donde es necesario.
    - La configuración del tiempo de espera se ha establecido de forma correcta, tal y como se describió anteriormente.
    - Las máquinas virtuales se implementan en el [grupo de selección de ubicación de proximidad de Azure](../../linux/co-location.md) correcto, tal como se describe en [Grupos de selección de ubicación de proximidad de Azure para una latencia de red óptima con aplicaciones SAP](sap-proximity-placement-scenarios.md).
    - La latencia de red entre las máquinas virtuales del nivel de aplicación de SAP y las máquinas virtuales del DBMS se prueba y se valida según se describe en las notas de soporte técnico de SAP [500235](https://launchpad.support.sap.com/#/notes/500235) y [1100926](https://launchpad.support.sap.com/#/notes/1100926/E). Evalúe los resultados respecto a la guía de latencia de red de la [nota de soporte técnico de SAP 1100926](https://launchpad.support.sap.com/#/notes/1100926/E). La latencia de red debe estar en el intervalo de moderada a buena. Se aplican excepciones al tráfico entre las máquinas virtuales y las unidades de instancias grandes de HANA, tal y como se describe en [este artículo](./hana-network-architecture.md#networking-architecture-for-hana-large-instance).
    - El cifrado se implementó donde era necesario y con el método de cifrado adecuado.
    - Las interfaces y otras aplicaciones pueden conectarse la infraestructura recién implementada.
1.  Cree un cuaderno de estrategias para reaccionar ante el mantenimiento planeado de Azure. Determine el orden en el que los sistemas y las máquinas virtuales deben reiniciarse para el mantenimiento planeado.
    

## <a name="go-live-phase"></a>Fase de puesta en marcha
Durante la fase de puesta en marcha, asegúrese de seguir los cuadernos de estrategias que desarrolló en las fases anteriores. Ejecute los pasos que probó y entrenó; no acepte cambios de última hora en las configuraciones y los procesos. Complete también los siguientes pasos:

1. Compruebe que la supervisión de Azure Portal y otras herramientas de supervisión funcionan. Se recomiendan el Monitor de rendimiento (perfmon) para Windows y SAR para Linux.
    - Contadores de CPU.
        - Tiempo medio de CPU, total (todas las CPU)
        - Tiempo medio de CPU, cada procesador individual (128 procesadores en máquinas virtuales M128)
        - Tiempo de kernel de CPU, cada procesador individual
        - Tiempo de usuario de CPU, cada procesador individual
    - Memoria.
        - Memoria libre
        - Entrada de páginas en memoria/segundo
        - Salida de páginas de memoria/segundo
    - disco.
        - Lectura de disco en Kbps, por disco individual
        - Lecturas de disco/segundo, por disco individual
        - Lectura de disco en microsegundos/lectura, por disco individual
        - Escritura en disco en Kbps, por disco individual
        - Escritura en disco/segundo, por disco individual
        - Escritura en disco en microsegundos/lectura, por disco individual
    - Red.
        - Entrada de paquetes en red/segundo
        - Salida de paquetes de red/segundo
        - kB de entrada en red/segundo
        - kB de salida de red/segundo
1.  Después de la migración de datos, realice todas las pruebas de validación acordadas con los propietarios de empresas. Acepte únicamente los resultados de pruebas de validación si dispone de los resultados de los sistemas de origen originales.
1.  Compruebe si las interfaces funcionan y si otras aplicaciones pueden comunicarse con los sistemas de producción recién implementados.
1.  Compruebe el sistema de transporte y corrección mediante STMS de transacciones de SAP.
1.  Realice copias de seguridad de las bases de datos una vez que el sistema esté listo para producción.
1.  Realice copias de seguridad de las máquinas virtuales en el nivel de aplicación de SAP una vez que el sistema está listo para producción.
1.  Para los sistemas SAP que no formaban parte de la fase de puesta en marcha actual, pero que se comunican con los sistemas SAP que trasladó a Azure en esta fase de puesta en marcha, debe restablecer el búfer de nombres de host en SM51. Este paso eliminará las direcciones IP anteriores almacenadas en caché, asociadas a los nombres de las instancias de aplicación que trasladó a Azure.  


## <a name="post-production"></a>Postproducción
En esta fase se trata de la supervisión, el funcionamiento y la administración del sistema. Desde el punto de vista de SAP, se aplican las tareas habituales que era necesario realizar en la anterior ubicación de hospedaje. Complete también estas tareas específicas de Azure:

1. Revise las facturas de Azure para detectar sistemas con cargos elevados.
2. Optimice la eficiencia de precio/rendimiento en las máquinas virtuales y en el almacenamiento.
3. Optimice las horas en las que puede apagar los sistemas.  


## <a name="next-steps"></a>Pasos siguientes
Consulte estos artículos:

- [Implementación y planeamiento de Azure Virtual Machines para SAP NetWeaver](./planning-guide.md)
- [Implementación de Azure Virtual Machines para SAP NetWeaver](./deployment-guide.md)
- [Consideraciones para la implementación de DBMS de Azure Virtual Machines para la carga de trabajo de SAP](./dbms_guide_general.md)
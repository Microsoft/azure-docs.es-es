---
title: 'SAP en Azure: escenarios admitidos con VM de Azure'
description: Escenarios admitidos en máquinas virtuales de Azure con carga de trabajo de SAP
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: SAP
ms.assetid: d7c59cc1-b2d0-4d90-9126-628f9c7a5538
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 03/11/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 1945dc3b9fa03354ef447f813d95b6040a4b7b91
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87833341"
---
# <a name="sap-workload-on-azure-virtual-machine-supported-scenarios"></a>Carga de trabajo de SAP en escenarios admitidos en máquinas virtuales de Azure
El diseño de la arquitectura de los sistemas SAP NetWeaver, Business One, `Hybris` o S/4HANA en Azure abre muchas oportunidades diferentes para las distintas arquitecturas y herramientas que se pueden usar para obtener una implementación escalable, eficiente y de alta disponibilidad. Aunque depende del sistema operativo o DBMS utilizado, existen restricciones. Además, no todos los escenarios que se admiten de forma local se admiten de la misma manera en Azure. Este documento le guiará a través de las configuraciones admitidas que no son de alta disponibilidad y las configuraciones y arquitecturas de alta disponibilidad que usan máquinas virtuales de Azure de forma exclusiva. En el caso de los escenarios compatibles con [instancias grandes de HANA](./hana-overview-architecture.md), consulte el artículo [Escenarios admitidos para instancias grandes de HANA](./hana-supported-scenario.md). 


## <a name="2-tier-configuration"></a>Configuración de 2 niveles
Se considera que una configuración de 2 niveles de SAP se ha creado a partir de una capa combinada del DBMS de SAP y el nivel de aplicación que se ejecuta en el mismo servidor o unidad de máquina virtual. El segundo nivel se considera la capa de la interfaz de usuario. En el caso de una configuración de 2 niveles, el DBMS y el nivel de aplicación de SAP comparten los recursos de la máquina virtual de Azure. Como resultado, debe configurar los distintos componentes de forma que no compitan por los recursos. También debe tener cuidado de no sobrescribir los recursos de la máquina virtual. Esta configuración no proporciona ninguna alta disponibilidad, más allá de los [acuerdos de nivel de servicio de Azure](https://azure.microsoft.com/support/legal/sla/) de los distintos componentes de Azure implicados.

Una representación gráfica de este tipo de configuración puede ser similar a la siguiente:

![Configuración sencilla de 2 niveles](./media/sap-planning-supported-configurations/two-tier-simple-configuration.png)

Estas configuraciones se admiten con Windows, Red Hat, SUSE y Oracle Linux para los sistemas DBMS de SQL Server, Oracle, Db2, maxDB y SAP ASE para los casos de producción y de no producción. Para SAP HANA como DBMS, este tipo de configuraciones solo se admite para los casos que no son de producción. Esto incluye también el caso de implementación de [instancias grandes de Azure HANA](./hana-overview-architecture.md).
Para todas las combinaciones de SO/DBMS compatibles en Azure, se admite este tipo de configuración. Sin embargo, es obligatorio establecer la configuración del DBMS y los componentes de SAP de forma que los componentes de DBMS y de SAP no compitan por la memoria y los recursos de CPU y, de este modo, superen los recursos disponibles físicamente. Esto se debe hacer restringiendo la memoria que el DBMS puede asignar. También debe limitar la memoria extendida de SAP en las instancias de la aplicación. También debe supervisar el consumo de CPU de la máquina virtual para asegurarse de que los componentes no estén maximizando los recursos de la CPU. 

> [!NOTE]
> En el caso de los sistemas SAP de producción, se recomienda una alta disponibilidad adicional y configuraciones de recuperación ante desastres eventuales, como se describe más adelante en este documento.


## <a name="3-tier-configuration"></a>Configuración de 3 niveles
En estas configuraciones, se separa el nivel de aplicación de SAP y el nivel de DBMS en diferentes máquinas virtuales. Por lo general, esto se hace en sistemas más grandes y sin buscar mayor flexibilidad en los recursos del nivel de aplicación de SAP. En la configuración más sencilla, no hay alta disponibilidad más allá de los [acuerdos de nivel de servicio de Azure](https://azure.microsoft.com/support/legal/sla/) de los distintos componentes de Azure implicados. 

La representación gráfica tiene el siguiente aspecto:

![Configuración sencilla de 2 niveles](./media/sap-planning-supported-configurations/three-tier-simple-configuration.png)

Este tipo de configuración se admite en Windows, Red Hat, SUSE y Oracle Linux para los sistemas DBMS de SQL Server, Oracle, Db2, SAP HANA, maxDB y SAP ASE para los casos de producción y de no producción. Esta es la configuración de implementación predeterminada para [instancias grandes de Azure HANA](./hana-overview-architecture.md). Para simplificar, no hemos distinguido entre las instancias de servicios centrales de SAP y las instancias de diálogo de SAP en el nivel de aplicación de SAP. En esta sencilla configuración de 3 niveles, no habría ninguna protección de alta disponibilidad para los servicios centrales de SAP.

> [!NOTE]
> En el caso de los sistemas SAP de producción, se recomienda una alta disponibilidad adicional y configuraciones de recuperación ante desastres eventuales, como se describe más adelante en este documento.


## <a name="multiple-dbms-instances-per-vm-or-hana-large-instance-unit"></a>Varias instancias de DBMS por máquina virtual o unidad de instancia grande de HANA
En este tipo de configuración, hospeda varias instancias de DBMS por cada máquina virtual de Azure o unidad de instancia grande de HANA. El motivo puede ser tener menos sistemas operativos que mantener y, con eso, menos costes. Otros motivos pueden ser tener más flexibilidad y más eficiencia compartiendo recursos de una máquina virtual mayor o una unidad de instancia grande de HANA entre varias instancias de DBMS. Hasta ahora, estas configuraciones se mostraban principalmente para sistemas que no son de producción.

Una configuración como esta podría ser similar a la siguiente:

![Varias instancias de DBMS en una unidad](./media/sap-planning-supported-configurations/multiple-database-instances.png)

Este tipo de implementación de DBMS es compatible con:

- SQL Server en Windows
- IBM Db2. Encuentre detalles en el artículo [Varias instancias (Linux, UNIX)](https://www.ibm.com/support/knowledgecenter/en/SSEPGG_10.5.0/com.ibm.db2.luw.admin.dbobj.doc/doc/c0004904.html)
- Para Oracle. Para obtener más información, consulte la [nota de soporte de SAP 1778431](https://launchpad.support.sap.com/#/notes/1778431) y las notas de SAP relacionadas.
- Para SAP HANA, se admiten varias instancias en una máquina virtual, en un método de implementación que SAP llama MCOS. Para más información, consulte el artículo de SAP [Varios sistemas SAP HANA en un host (MCOS)] (https://help.sap.com/viewer/eb3777d5495d46c5b2fa773206bbfb46/2.0.02/.
- /b2751fd43bec41a9a14e01913f1edf18.html).

Al ejecutar varias instancias de base de datos en un host, debe asegurarse de que las distintas instancias no compitan por los recursos y, por tanto, superen los límites de recursos físicos de la máquina virtual. Esto es especialmente pertinente en el caso de la memoria en la que es necesario limitar la memoria que puede asignar cualquiera de las instancias que comparten la máquina virtual. También podría ser pertinente para los recursos de CPU que pueden aprovechar las distintas instancias de base de datos. Todo el DBMS mencionado tiene configuraciones que permiten limitar la asignación de memoria y los recursos de CPU en un nivel de instancia.
Para que este tipo de configuración sea compatible con las máquinas virtuales de Azure, se espera que los discos o volúmenes que se usan para los archivos de registro de datos o para registrar/rehacer de las bases de datos administradas por las distintas instancias sean independientes. En otras palabras, los archivos de registro de datos o para registrar/rehacer de las bases de datos administrados por una instancia de DBMS diferente no deben compartir los mismos discos o volúmenes. 

La configuración de disco para instancias grandes de HANA se entrega configurada y se detalla en [Escenarios admitidos para instancias grandes de HANA](./hana-supported-scenario.md#single-node-mcos). 

> [!NOTE]
> En el caso de los sistemas SAP de producción, se recomienda una alta disponibilidad adicional y configuraciones de recuperación ante desastres eventuales, como se describe más adelante en este documento. Las máquinas virtuales con varias instancias de DBMS no se admiten con las configuraciones de alta disponibilidad descritas más adelante en este documento.


## <a name="multiple-sap-dialog-instances-in-one-vm"></a>Varias instancias de diálogo de SAP en una máquina virtual
En muchos casos, se han implementado varias instancias de diálogo en servidores sin sistema operativo o incluso en máquinas virtuales que se ejecutan en nubes privadas. Estas configuraciones buscaban adaptar determinadas instancias de diálogo de SAP a ciertas cargas de trabajo, funcionalidad empresarial o tipos de carga de trabajo. La razón para no aislar esas instancias en máquinas virtuales independientes era el esfuerzo del mantenimiento y las operaciones del sistema operativo. O, en muchos casos, los costos en caso de que el anfitrión o el operador de la máquina virtual solicitasen una cuota mensual por máquina virtual operada y administrada. En Azure, un escenario de hospedaje de varias instancias de diálogo de SAP dentro de una sola máquina virtual es compatible con fines de producción y no producción en los sistemas operativos Windows, Red Hat, SUSE y Oracle Linux. El parámetro kernel SAP PHYS_MEMSIZE, disponible en los kernels de Linux moderno y Windows, debe establecerse si varias instancias del servidor de aplicaciones de SAP se ejecutan en una sola máquina virtual. También se recomienda limitar la expansión de la memoria extendida de SAP en los sistemas operativos, como Windows, donde se implementa el crecimiento automático de la memoria extendida de SAP. Esto puede hacerse con el parámetro de perfil predeterminado `em/max_size_MB`.

Una configuración de 3 niveles en la que varias instancias de diálogo de SAP se ejecutan en máquinas virtuales de Azure puede tener el aspecto siguiente:

![Varias instancias de DBMS en una unidad](./media/sap-planning-supported-configurations/multiple-dialog-instances.png)

Para simplificar, no hemos distinguido entre las instancias de servicios centrales de SAP y las instancias de diálogo de SAP en el nivel de aplicación de SAP. En esta sencilla configuración de 3 niveles, no habría ninguna protección de alta disponibilidad para los servicios centrales de SAP. En el caso de los sistemas de producción, no se recomienda dejar los servicios centrales de SAP desprotegidos. Para obtener información específica sobre las llamadas configuraciones de varios SID en torno a las instancias centrales de SAP y la alta disponibilidad de dichas configuraciones de varios SID, consulte secciones posteriores de este documento.

## <a name="high-availability-protection-for-the-sap-dbms-layer"></a>Protección de alta disponibilidad para el nivel de DBMS de SAP
A medida que implementa sistemas de producción de SAP, debe considerar el tipo en espera activa de configuraciones de alta disponibilidad. Especialmente con SAP HANA, donde los datos deben cargarse en memoria antes de poder obtener el máximo rendimiento y escalabilidad, la recuperación del servicio de Azure no es una medida ideal para lograr una alta disponibilidad. 

En general, Microsoft solo admite configuraciones de alta disponibilidad y paquetes de software que se describen en la sección de la carga de trabajo de SAP en docs.microsoft.com. Puede leer la misma declaración en la nota de SAP [1928533](https://launchpad.support.sap.com/#/notes/1928533). Microsoft no proporcionará soporte técnico para otros marcos de software de terceros de alta disponibilidad que no estén documentados por Microsoft junto con la carga de trabajo de SAP. En tales casos, el proveedor de terceros del marco de alta disponibilidad es la parte auxiliar para la configuración de alta disponibilidad, que debe ser contratada por usted como cliente en el proceso de soporte técnico. En este artículo se van a mencionar excepciones. 

En general, Microsoft admite un conjunto limitado de configuraciones de alta disponibilidad en máquinas virtuales de Azure o unidades de instancias grandes de HANA. Para ver los escenarios compatibles de instancias grandes de HANA, lea el documento [Escenarios admitidos para instancias grandes de HANA](./hana-supported-scenario.md).

En el caso de las máquinas virtuales de Azure, se admiten las siguientes configuraciones de alta disponibilidad en el nivel de DBMS:

- Replicación del sistema SAP HANA basada en Linux Pacemaker en SUSE y Red Hat. Vea los artículos detallados:
    - [Alta disponibilidad de SAP HANA en máquinas virtuales de Azure en SUSE Linux Enterprise Server](./sap-hana-high-availability.md)
    - [Alta disponibilidad de SAP HANA en máquinas virtuales de Azure en Red Hat Enterprise Linux](./sap-hana-high-availability-rhel.md)
- Configuraciones de n+m de escalado horizontal de SAP HANA mediante [Azure NetApp Files](https://azure.microsoft.com/services/netapp/) en SUSE y Red Hat. Los detalles aparecen en estos artículos:
    - [Implementación de un sistema de escalabilidad horizontal de SAP HANA con nodo en espera en VM de Azure mediante Azure NetApp Files en SUSE Linux Enterprise Server](./sap-hana-scale-out-standby-netapp-files-suse.md)
    - [Implementación de un sistema de escalabilidad horizontal de SAP HANA con nodo en espera en VM de Azure mediante Azure NetApp Files en Red Hat Enterprise Linux](./sap-hana-scale-out-standby-netapp-files-rhel.md)
- Clúster de conmutación por error de SQL Server basado en servicios de archivos de escalabilidad horizontal de Windows. No obstante, la recomendación de los sistemas de producción es usar SQL Server Always On en lugar de la agrupación en clústeres. SQL Server Always On proporciona una mejor disponibilidad mediante almacenamiento independiente. Los detalles se describen en este artículo: 
    - [Configuración de una instancia de clúster de conmutación por error de SQL Server en Azure Virtual Machines](../../../azure-sql/virtual-machines/windows/failover-cluster-instance-storage-spaces-direct-manually-configure.md)
- SQL Server Always On es compatible con el sistema operativo Windows para SQL Server en Azure. Esta es la recomendación predeterminada para las instancias de SQL Server de producción en Azure. Los detalles aparecen en estos artículos:
    - [Introducción a los grupos de disponibilidad Always On de SQL Server en Azure Virtual Machines](../../../azure-sql/virtual-machines/windows/availability-group-overview.md).
    - [Configuración de un grupo de disponibilidad AlwaysOn en máquinas virtuales de Azure en distintas regiones](../../../azure-sql/virtual-machines/windows/availability-group-manually-configure-multiple-regions.md).
    - [Configuración de un equilibrador de carga para un grupo de disponibilidad AlwaysOn en Azure](../../../azure-sql/virtual-machines/windows/availability-group-load-balancer-portal-configure.md).
- Oracle Data Guard para Windows y Oracle Linux. Puede encontrar más información sobre Oracle Linux en este artículo:
    - [Implementación de Oracle Data Guard en una máquina virtual Linux en Azure](../oracle/configure-oracle-dataguard.md)
- Aquí se proporciona la documentación detallada de IBM Db2 HADR en SUSE y RHEL para SUSE y RHEL mediante Pacemaker:
    - [Alta disponibilidad de IBM Db2 LUW en máquinas virtuales de Azure en SUSE Linux Enterprise Server con Pacemaker](./dbms-guide-ha-ibm.md)
    - [Alta disponibilidad de IBM Db2 LUW en máquinas virtuales de Azure en Red Hat Enterprise Linux Server](./high-availability-guide-rhel-ibm-db2-luw.md)
- La configuración de SAP ASE y SAP maxDB tal y como se detalla en estos documentos:
    - [Implementación de DBMS de Azure Virtual Machines de SAP ASE para una carga de trabajo de SAP](./dbms_guide_sapase.md)
    - [Implementación de SAP MaxDB, liveCache y del servidor de contenido en máquinas virtuales de Azure](./dbms_guide_maxdb.md)
- Los escenarios de alta disponibilidad de instancias grandes de HANA se detallan en:
    - [Escenarios admitidos para instancias grandes de HANA: HSR con STONITH para alta disponibilidad](./hana-supported-scenario.md#hsr-with-stonith-for-high-availability)
    - [Escenarios admitidos para instancias grandes de HANA: conmutación por error automática del host (1+1)](./hana-supported-scenario.md#host-auto-failover-11)

> [!IMPORTANT]
> Para ninguno de los escenarios descritos anteriormente se admiten configuraciones de varias instancias de DBMS en una máquina virtual. Esto significa que, en cada uno de los casos, solo se puede implementar una instancia de base de datos por máquina virtual y protegerse con los métodos de alta disponibilidad descritos. La protección de varias instancias de DBMS en el mismo clúster de conmutación por error de Windows o Pacemaker **NO** se admite en este momento. También se admite Oracle Data Guard para casos de implementación de instancia única por máquina virtual. 

Varios sistemas de base de datos permiten hospedar varias bases de datos en una instancia de DBMS. Como en el caso de SAP HANA, se pueden hospedar varias bases de datos en varios contenedores de base de datos (MDC). En los casos en los que estas configuraciones de varias bases de datos funcionan en un recurso de clúster de conmutación por error, se admiten estas configuraciones. Las configuraciones que no se admiten son casos en los que se necesitarían varios recursos de clúster. En el caso de las configuraciones en las que se definirían varios grupos de disponibilidad de SQL Server, en una instancia de SQL Server.


![Configuración de alta disponibilidad de DBMS](./media/sap-planning-supported-configurations/database-high-availability-configuration.png)

En función de los sistemas operativos o de DBMS, es posible que componentes como Azure Load Balancer no sean necesarios como parte de la arquitectura de la solución. 

Específicamente para maxDB, la configuración de almacenamiento debe ser diferente. En maxDB, los archivos de datos y de registro se deben encontrar en el almacenamiento compartido para las configuraciones de alta disponibilidad. Solo en el caso de maxDB, se admite el almacenamiento compartido para lograr alta disponibilidad. Para todos los demás DBMS, las pilas de almacenamiento independientes por nodo son las únicas configuraciones de disco compatibles.

Se sabe que existen otros marcos de alta disponibilidad y se sabe que se ejecutan también en Microsoft Azure. Sin embargo, Microsoft no ha probado estos marcos. Si desea compilar la configuración de alta disponibilidad con esos marcos, deberá trabajar con el proveedor de ese software en materia de:

- Desarrollo de una arquitectura de implementación
- Implementación de la arquitectura
- Compatibilidad de la arquitectura

> [!IMPORTANT]
> Microsoft Azure Marketplace ofrece una variedad de dispositivos de software que proporcionan soluciones de almacenamiento sobre el almacenamiento nativo de Azure. Estos dispositivos flexibles se pueden usar para crear recursos compartidos de NFS y, en teoría, podrían usarse en las implementaciones escaladas de SAP HANA en las que se requiere un nodo en espera. Debido a varios motivos, ninguno de estos dispositivos de software de almacenamiento es compatible con ninguna de las implementaciones de DBMS de Microsoft y SAP en Azure. En este momento no se admiten implementaciones de DBMS en recursos compartidos de SMB. Las implementaciones de DBMS en los recursos compartidos de NFS se limitan a recursos compartidos de NFS 4.1 en [Azure NetApp Files](https://azure.microsoft.com/services/netapp/).


## <a name="high-availability-for-sap-central-service"></a>Alta disponibilidad para los servicios centrales de SAP
Los servicios centrales de SAP representan un segundo único punto de error de la configuración de SAP. Como resultado, también necesitará proteger estos procesos de servicios centrales. La oferta compatible y documentada para cargas de trabajo de SAP es esta:

- Servidor de clústeres de conmutación por error de Windows con servicios de archivos de escalabilidad horizontal de Windows para sapmnt y directorio de transporte global. Los detalles se describen en este artículo:
    - [Agrupación de una instancia de ASCS/SCS de SAP en un clúster de conmutación por error de Windows con un recurso compartido de archivos en Azure](./sap-high-availability-guide-wsfc-file-share.md)
    - [Preparación de la infraestructura de Azure para la alta disponibilidad de SAP con un clúster de conmutación por error de Windows y el recurso compartido de archivos para instancias de SAP ASCS/SCS](./sap-high-availability-infrastructure-wsfc-file-share.md)
- Servidor de clústeres de conmutación por error de Windows con recurso compartido de archivos de SMB basado en [Azure NetApp Files](https://azure.microsoft.com/services/netapp/) para sapmnt y directorio de transporte global. Los detalles aparecen en este artículo:
    - [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en Windows con Azure NetApp Files (SMB) para aplicaciones SAP](./high-availability-guide-windows-netapp-files-smb.md)
- Servidor de clústeres de conmutación por error de Windows basado en SIOS `Datakeeper`. Aunque está documentado por Microsoft, necesita una relación de soporte técnico con SIOS, por lo que puede interactuar con el soporte técnico de SIOS al usar esta solución. Los detalles se describen en este artículo:
    - [Agrupación de una instancia de ASCS/SCS de SAP en un clúster de conmutación por error de Windows con un disco compartido de clúster en Azure](./sap-high-availability-guide-wsfc-shared-disk.md)
    - [Preparación de la infraestructura de Azure para alta disponibilidad de SAP con un clúster de conmutación por error de Windows y un disco compartido para ASCS/SCS de SAP](./sap-high-availability-infrastructure-wsfc-shared-disk.md)
- Pacemaker en el sistema operativo SUSE con la creación de un recurso compartido de archivos de NFS de alta disponibilidad con dos máquinas virtuales de SUSE y `drdb` para la replicación de archivos. Los detalles están documentados en el artículo.
    - [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SUSE Linux Enterprise Server para aplicaciones de SAP](./high-availability-guide-suse.md)
    - [Alta disponibilidad para NFS en máquinas virtuales de Azure en SUSE Linux Enterprise Server](./high-availability-guide-suse-nfs.md)
- Sistema operativo SUSE en Pacemaker con el aprovechamiento de recursos compartidos de archivos de NFS proporcionados por [Azure NetApp Files](https://azure.microsoft.com/services/netapp/). Los detalles están documentados en
    - [Alta disponibilidad de SAP NetWeaver en VM de Azure en SUSE Linux Enterprise Server con Azure NetApp Files para las aplicaciones de SAP](./high-availability-guide-suse-netapp-files.md)
- Pacemaker en el sistema operativo Red Hat con recurso compartido de archivos de NFS hospedado en un clúster de `glusterfs`. Se puede encontrar más detalles en los artículos siguientes:
    - [Alta disponibilidad de Azure Virtual Machines para SAP NetWeaver en Red Hat Enterprise Linux](./high-availability-guide-rhel.md)
    - [`GlusterFS` en máquinas virtuales de Azure de Red Hat Enterprise Linux para SAP NetWeaver](./high-availability-guide-rhel-glusterfs.md)
- Pacemaker en el sistema operativo Red Hat con recurso compartido de archivos NFS hospedado en [Azure NetApp Files](https://azure.microsoft.com/services/netapp/). Los detalles se describen en este artículo:
    - [Alta disponibilidad de Azure Virtual Machines para SAP NetWeaver en Red Hat Enterprise Linux con Azure NetApp Files para aplicaciones SAP](./high-availability-guide-rhel-netapp-files.md)

De las soluciones enumeradas, necesita una relación de soporte técnico con SIOS para admitir el producto `Datakeeper` y para interactuar con SIOS directamente en caso de que surjan problemas. En función de la licencia de Windows, Red Hat o sistema operativo SUSE, también podría ser necesario tener un contrato de soporte técnico con el proveedor del sistema operativo para plena compatibilidad con las configuraciones de alta disponibilidad enumeradas.

La configuración también se puede mostrar como:

![Configuración de alta disponibilidad de DBMS y ASCS](./media/sap-planning-supported-configurations/high-available-3-tier-configuration.png)

En el lado derecho de los gráficos, se muestran los servicios centrales de SAP de alta disponibilidad. Además de tener los servicios centrales de SAP protegidos con un marco de clúster de conmutación por error que puede conmutar por error en caso de que se produzca un problema, es necesario un recurso compartido de archivos de NFS o SMB de alta disponibilidad, o un disco compartido de Windows para asegurarse de que el directorio de transporte global y sapmnt estén disponibles independientemente de la existencia de una sola máquina virtual. Además, algunas de las soluciones, como el servidor de clústeres de conmutación por error de Windows y Pacemaker, van a requerir que una instancia de Azure Load Balancer dirija o redirija el tráfico a un nodo en buen estado.

En la lista que se muestra, no hay ninguna mención al sistema operativo Oracle Linux. Oracle Linux no admite Pacemaker como marco de clústeres. Si desea implementar el sistema SAP en Oracle Linux y necesita un marco de alta disponibilidad para Oracle Linux, debe trabajar con proveedores de terceros. Uno de los proveedores es SIOS con su conjunto de protección para Linux compatible con SAP en Azure. Para más información, lea la nota de SAP [1662610 - Detalles de compatibilidad del conjunto de protección de SIOS para Linux](https://launchpad.support.sap.com/#/notes/1662610) para obtener más detalles.



### <a name="supported-storage-with-the-sap-central-services-scenarios-listed-above"></a>Almacenamiento compatible con los escenarios de servicios centrales de SAP enumerados anteriormente
Dado que solo un subconjunto de tipos de almacenamiento de Azure proporciona recursos compartidos de archivos de NFS o SMB de alta disponibilidad para el uso en nuestro clúster de servicios centrales de SAP, se muestra una lista de tipos de almacenamiento admitidos.

- El servidor de clústeres de conmutación por error de Windows con el servidor de archivos de escalabilidad horizontal de Windows se puede implementar en todos los tipos de almacenamiento nativos de Azure, excepto Azure NetApp Files. Sin embargo, se recomienda aprovechar Premium Storage debido a acuerdos de nivel de servicio superiores en cuanto a rendimiento e IOPS.
- El servidor de clústeres de conmutación por error de Windows con SMB en Azure NetApp Files se admite en Azure NetApp Files. Los recursos compartidos de archivos de SMB en los servicios de archivos de Azure **NO** se admiten en este momento.
- El servidor de clústeres de conmutación por error de Windows con un disco compartido de Windows basado en SIOS `Datakeeper` se puede implementar en todos los tipos de almacenamiento nativos de Azure, excepto Azure NetApp Files. Sin embargo, se recomienda aprovechar Premium Storage debido a acuerdos de nivel de servicio superiores en cuanto a rendimiento e IOPS.
- SUSE o Red Hat Pacemaker con recursos compartidos de archivos de NFS en Azure NetApp Files se admiten en Azure NetApp Files. 
- SUSE Pacemaker con una configuración de `drdb` entre dos máquinas virtuales es compatible con los tipos de almacenamiento nativos de Azure, excepto Azure NetApp Files. Sin embargo, se recomienda aprovechar Premium Storage debido a acuerdos de nivel de servicio superiores en cuanto a rendimiento e IOPS.
- Red Hat Pacemaker con `glusterfs` para proporcionar un recurso compartido de archivos de NFS es compatible con los tipos de almacenamiento nativos de Azure, excepto Azure NetApp Files. Sin embargo, se recomienda aprovechar Premium Storage debido a acuerdos de nivel de servicio superiores en cuanto a rendimiento e IOPS.

> [!IMPORTANT]
> Microsoft Azure Marketplace ofrece una variedad de dispositivos de software que proporcionan soluciones de almacenamiento sobre el almacenamiento nativo de Azure. Estos dispositivos de software se pueden usar para crear recursos compartidos de archivos de NFS o SMB también que teóricamente podrían usarse en los servicios centrales de SAP en clúster de conmutación por error. Microsoft no admite directamente estas soluciones para la carga de trabajo de SAP. Si decide usar una solución de este tipo para crear el recurso compartido de archivos de NFS o SMB, la compatibilidad con la configuración del servicio central de SAP debe proporcionarla el software de terceros que posee el software en el dispositivo de software de almacenamiento.


## <a name="multi-sid-sap-central-services-failover-clusters"></a>Clústeres de conmutación por error de servicios centrales de SAP con varios SID
Para reducir el número de máquinas virtuales que se necesitan en entornos SAP de gran tamaño, SAP permite ejecutar instancias de servicios centrales de SAP de varios sistemas SAP diferentes en la configuración del clúster de conmutación por error. Imagine casos en los que tiene 30 o más sistemas de producción de NetWeaver o S/4HANA. Sin la agrupación en clústeres de varios SID, estas configuraciones requerirían 60 o más máquinas virtuales en 30 o más configuraciones de clúster de conmutación por error de Windows o Pacemaker. Además de los clústeres de conmutación por error de DBMS necesarios. La implementación de varios servicios centrales de SAP en dos nodos en una configuración de clúster de conmutación por error puede reducir considerablemente el número de máquinas virtuales. Sin embargo, la implementación de varias instancias de servicios centrales de SAP en una sola configuración de clúster de dos nodos también tiene algunas desventajas. Los problemas relacionados con una sola máquina virtual en la configuración del clúster ocurren en varios sistemas SAP. El mantenimiento en el sistema operativo invitado que se ejecuta en la configuración del clúster requiere más coordinación, ya que se ven afectados varios sistemas SAP de producción. Herramientas como SAP LaMa no admiten la agrupación en clústeres de varios SID en el proceso de clonación del sistema.

En Azure, se admite una configuración de clúster de varios SID para el sistema operativo Windows con ENSA1 y ENSA2. La recomendación es no combinar la antigua arquitectura del servicio de replicación en cola (ENSA1) con la nueva arquitectura (ENSA2) en un clúster de varios SID. Los detalles sobre esta arquitectura se documentan en los artículos siguientes:

- [Alta disponibilidad con varios identificadores de seguridad de instancia de ASCS/SCS de SAP para los clústeres de conmutación por error de Windows Server y el disco compartido en Azure](./sap-ascs-ha-multi-sid-wsfc-shared-disk.md) 
- [Alta disponibilidad con varios identificadores de seguridad de instancia SAP ASCS/SCS para los clústeres de conmutación por error de Windows Server y los recursos compartidos de archivos en Azure](./sap-ascs-ha-multi-sid-wsfc-file-share.md) 

Para SUSE, también se admite un clúster de varios SID basado en Pacemaker. Hasta ahora, la configuración es compatible con:

- Un máximo de cinco instancias de ASCS/SCS de SAP
- La arquitectura de ICE del antiguo servidor de replicación en cola (ENSA1)
- Configuraciones de clúster de Pacemaker de dos nodos

La configuración se documenta en [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SUSE Linux Enterprise Server para SAP Applications: guía de varios SID](./high-availability-guide-suse-multi-sid.md).

Un clúster de varios SID con el servidor de replicación en cola tiene esquemáticamente un aspecto como el siguiente:

![Configuración de alta disponibilidad de DBMS y ASCS](./media/sap-planning-supported-configurations/high-available-multi-system-configuration.png)


## <a name="sap-hana-scale-out-scenarios"></a>Escenarios de escalabilidad horizontal de SAP HANA
Los escenarios de escalado horizontal de SAP HANA se admiten para un subconjunto de máquinas virtuales de Azure con certificación de HANA, tal como se muestra en el directorio de [hardware de SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure). Todas las máquinas virtuales marcadas con "Sí" en la columna "Agrupación en clústeres" se pueden usar para el escalado horizontal de OLAP o S/4HANA. Las configuraciones sin modo de espera se admiten con los tipos Azure Storage siguientes: 

- Azure Premium Storage, incluido el acelerador de escritura de Azure para el volumen /hana/log.
- [Disco Ultra](../../linux/disks-enable-ultra-ssd.md)
- [Azure NetApp Files](https://azure.microsoft.com/services/netapp/) 

Las configuraciones de escalado horizontal de SAP HANA para OLAP o S/4HANA con nodos en espera se admiten exclusivamente con NFS compartido hospedado en Azure NetApp Files.

Para obtener más información sobre las configuraciones de almacenamiento exactas con o sin nodo en espera, consulte los artículos:

- [Configuraciones de almacenamiento de máquinas virtuales de Azure en SAP HANA](./hana-vm-operations-storage.md) 
- [Implementación de un sistema de escalabilidad horizontal de SAP HANA con nodo en espera en VM de Azure mediante Azure NetApp Files en SUSE Linux Enterprise Server](./sap-hana-scale-out-standby-netapp-files-suse.md)
- [Implementación de un sistema de escalabilidad horizontal de SAP HANA con nodo en espera en VM de Azure mediante Azure NetApp Files en Red Hat Enterprise Linux](./sap-hana-scale-out-standby-netapp-files-rhel.md)
- [Nota de soporte técnico de SAP 2080991](https://launchpad.support.sap.com/#/notes/2080991)

Para obtener información detallada de las configuraciones de escalado horizontal de HANA que admiten las instancias grandes de HANA, consulte la siguiente documentación:

- [Escenarios admitidos para escalado horizontal de instancias grandes de HANA con espera](./hana-supported-scenario.md#scale-out-with-standby)
- [Escenarios admitidos para escalado horizontal de instancias grandes de HANA sin espera](./hana-supported-scenario.md#scale-out-without-standby)


## <a name="disaster-recovery-scenario"></a>Escenario de recuperación ante desastres
Se admiten diferentes escenarios de recuperación ante desastres. Definimos las arquitecturas de desastres como aquellas arquitecturas que deberían compensar las regiones de Azure completas que se quedan desconectadas. Esto significa que necesitamos que el destino de recuperación ante desastres sea una región de Azure diferente para ejecutar el entorno de SAP. Separamos métodos y configuraciones en nivel de DBMS y nivel que no es de DBMS. 

### <a name="dbms-layer"></a>Nivel de DBMS
En el nivel de DBMS, se admiten las configuraciones que usan mecanismos de replicación nativa de DBMS, como Always On, Oracle Data Guard, DB2 HADR, SAP ASE Always-On o la replicación del sistema de HANA. Es obligatorio que la secuencia de replicación en estos casos sea asincrónica, en lugar de sincrónica como en los típicos escenarios de alta disponibilidad que se implementan dentro de una única región de Azure. Se describe un ejemplo habitual de una configuración de recuperación ante desastres de DBMS compatible en el artículo [Disponibilidad de SAP HANA entre regiones de Azure](./sap-hana-availability-across-regions.md#combine-availability-within-one-region-and-across-regions). En el segundo gráfico de esa sección se describe un escenario con HANA como ejemplo. Las bases de datos principales compatibles con las aplicaciones de SAP se pueden implementar en este tipo de escenario.

Se admite el uso de una máquina virtual más pequeña como instancia de destino en la región de recuperación ante desastres, ya que la máquina virtual no experimenta todo el tráfico de la carga de trabajo. Al hacerlo, tenga en cuenta las siguientes consideraciones:

- Los tipos de máquina virtual más pequeños no permiten muchos discos conectados a máquinas virtuales más pequeñas.
- Las máquinas virtuales más pequeñas tienen menos rendimiento de red y almacenamiento.
- Cambiar el tamaño de las familias de máquinas virtuales puede ser un problema cuando se recopilan las diferentes máquinas virtuales en un conjunto de disponibilidad de Azure o cuando se produce un cambio de tamaño entre la familia de la serie M y la familia de máquinas virtuales MV2.
- Consumo de CPU y memoria para que la instancia de base de datos pueda recibir el flujo de cambios con un retraso mínimo y suficientes recursos de CPU y memoria para aplicar estos cambios con un retraso mínimo a los datos.  

Puede encontrar más detalles sobre las limitaciones de diferentes tamaños de máquina virtual [aquí](../../sizes.md) 

Otro método admitido para implementar un destino de recuperación ante desastres es tener una segunda instancia de DBMS instalada en una máquina virtual que ejecute una instancia de DBMS que no sea de producción de una instancia de SAP que no sea de producción. Esto puede ser un poco más complicado, ya que es necesario averiguar qué memoria, recursos de CPU, ancho de banda de red y ancho de banda de almacenamiento son necesarios para las instancias de destino concretas que deberían funcionar como instancia principal en el escenario de recuperación ante desastres. Especialmente en HANA, se recomienda encarecidamente que configure la instancia que funciona como destino de recuperación ante desastres en un host compartido, de modo que los datos no se carguen previamente en la instancia de destino de recuperación ante desastres.

En escenarios de recuperación ante desastres de instancias grandes de HANA, compruebe estos documentos:

- [Nodo único con DR mediante replicación de almacenamiento](./hana-supported-scenario.md#single-node-with-dr-using-storage-replication)
- [Nodo único con DR (multipropósito) mediante replicación de almacenamiento](./hana-supported-scenario.md#single-node-with-dr-multipurpose-using-storage-replication)
- [Nodo único con DR (multipropósito) mediante replicación de almacenamiento](./hana-supported-scenario.md#single-node-with-dr-multipurpose-using-storage-replication)
- [Alta disponibilidad con HSR y recuperación ante desastres con replicación de almacenamiento](./hana-supported-scenario.md#high-availability-with-hsr-and-dr-with-storage-replication)
- [Escalabilidad horizontal con DR mediante replicación de almacenamiento](./hana-supported-scenario.md#scale-out-with-dr-using-storage-replication)
- [Nodo único con DR mediante HSR](./hana-supported-scenario.md#single-node-with-dr-using-hsr)
- [HSR de nodo único a DR (optimizado para costo)](./hana-supported-scenario.md#single-node-hsr-to-dr-cost-optimized)
- [Alta disponibilidad y recuperación ante desastres con HSR](./hana-supported-scenario.md#high-availability-and-disaster-recovery-with-hsr)
- [Alta disponibilidad y recuperación ante desastres con HSR (optimizado para costes)](./hana-supported-scenario.md#high-availability-and-disaster-recovery-with-hsr-cost-optimized)
- [Escalabilidad horizontal con DR mediante HSR](./hana-supported-scenario.md#scale-out-with-dr-using-hsr)

> [!NOTE]
> No se ha probado el uso de [Azure Site Recovery](https://azure.microsoft.com/services/site-recovery/) para las implementaciones de DBMS en la carga de trabajo de SAP. Como resultado, no se admite para el nivel de DBMS de sistemas SAP en este momento. No se admiten otros métodos de replicación de Microsoft y SAP que no aparecen en la lista. El uso de software de terceros para replicar el nivel de DBMS de sistemas SAP entre diferentes regiones de Azure debe ser compatible con el proveedor del software y no se admitirá a través de los canales de soporte técnico de Microsoft y SAP. 
 
## <a name="non-dbms-layer"></a>Nivel que no es de DBMS
Para el nivel de aplicación de SAP y los eventuales recursos compartidos de archivos o ubicaciones de almacenamiento que se necesiten, los clientes aprovechan los dos escenarios principales:

- Los destinos de recuperación ante desastres en la segunda región de Azure no se usan para fines de producción o de no producción. En este escenario, las máquinas virtuales que funcionan como destino de recuperación ante desastres no se implementan idealmente, y la imagen y los cambios en las imágenes del nivel de aplicación de SAP de producción se replican en la región de recuperación ante desastres. Una funcionalidad que puede realizar esta tarea es [Azure Site Recovery](../../../site-recovery/azure-to-azure-move-overview.md). Azure Site Recovery admite escenarios de replicación de Azure a Azure como este. 
- Los destinos de recuperación ante desastres son máquinas virtuales que en realidad están en uso por sistemas que no son de producción. Todo el panorama de SAP se distribuye entre dos regiones diferentes de Azure con sistemas de producción normalmente en una región y sistemas que no son de producción en otra región. En muchas implementaciones de clientes, el cliente tiene un sistema que no es de producción y que es equivalente a un sistema de producción. El cliente tiene instaladas las instancias de la aplicación de producción previamente en los sistemas que no son de producción. En el caso de una conmutación por error, se apagarían las instancias que no son de producción, los nombres virtuales de las máquinas virtuales de producción se moverían a las máquinas virtuales que no son de producción (después de asignar nuevas direcciones IP en DNS) y se iniciarían las instancias de producción preinstaladas.

### <a name="sap-central-services-clusters"></a>Clústeres de servicios centrales de SAP
Los clústeres de servicios centrales de SAP que usan discos compartidos (Windows), recursos compartidos de archivos de SMB (Windows) o recursos compartidos de archivos de NFS son un poco más difíciles de replicar. En el lado de Windows, la replicación de almacenamiento de Windows es una posible solución. En Linux, rsync es una solución viable.



## <a name="non-supported-scenario"></a>Escenario no admitido
Hay una lista de escenarios que no se admiten para la carga de trabajo de SAP en arquitecturas de Azure. **No admitido** significa que SAP y Microsoft no podrán admitir estas configuraciones y necesitan diferirlas a un eventual tercero participante que proporcione software para establecer dichas arquitecturas. Dos de las categorías son:

- Dispositivos de software de almacenamiento: Hay una serie de dispositivos de software de almacenamiento que se ofrecen en Azure Marketplace. Algunos de los proveedores ofrecen documentación sobre cómo usar estos dispositivos de software de almacenamiento en Azure en relación con el software de SAP. El proveedor de esos dispositivos de software de almacenamiento debe proporcionar compatibilidad para las configuraciones o las implementaciones que impliquen dichos dispositivos de software de almacenamiento. Este hecho también se manifiesta en la [nota de soporte de SAP 2015553](https://launchpad.support.sap.com/#/notes/2015553)
- Marcos de alta disponibilidad: Solo los clústeres de conmutación por error de Windows Server y Pacemaker son marcos de alta disponibilidad admitidos para la carga de trabajo de SAP en Azure. Como se mencionó anteriormente, Microsoft describe y documenta la solución de SIOS `Datakeeper`. No obstante, los componentes de SIOS `Datakeeper` deben ser compatibles con SIOS como proveedor que proporciona esos componentes. SAP también enumera otros marcos de alta disponibilidad certificados en varias notas de SAP. Algunos de ellos también están certificados por el proveedor de terceros de Azure. No obstante, el proveedor del producto debe proporcionar soporte técnico para las configuraciones que usan estos productos. Los distintos proveedores tienen una integración diferente en los procesos de compatibilidad de SAP. Debe aclarar qué proceso de soporte técnico funciona mejor para el proveedor en particular antes de decidirse a usar el producto en las configuraciones de SAP implementadas en Azure.
- Los clústeres de discos compartidos en los que los archivos de base de datos residen en los discos compartidos no son compatibles con la excepción de maxDB. Para todas las demás bases de datos, la solución admitida es tener ubicaciones de almacenamiento independientes en lugar de un recurso compartido de archivos de SMB o NFS o un disco compartido para configurar escenarios de alta disponibilidad.

Otros escenarios no compatibles son, por ejemplo, los siguientes:

- Escenarios de implementación que introducen una mayor latencia de red entre la capa de aplicación de SAP y el nivel de DBMS de SAP en la arquitectura común de SAP, tal como se muestra en NetWeaver, S/4HANA y, por ejemplo, `Hybris`. Esto incluye:
    - Implementación de uno de los niveles locales, mientras que el otro nivel se implementa en Azure.
    - Implementación de la capa de aplicación de SAP de un sistema en una región de Azure diferente a la del nivel de DBMS.
    - Implementación de un nivel en centros de recursos que se encuentran en la misma ubicación que Azure y el otro nivel en Azure, excepto en los casos en los que un servicio nativo de Azure proporciona estos patrones de arquitectura.
    - Implementación de aplicaciones virtuales de red entre el nivel de aplicación de SAP y el nivel de DBMS.
    - Aprovechamiento del almacenamiento que se hospeda en centros de información que se encuentran en la misma ubicación que el centro de datos de Azure para el nivel de DBMS de SAP o el directorio de transporte global de SAP.
    - Implementación de dos niveles con dos proveedores de nube diferentes. Por ejemplo, la implementación del nivel de DBMS en la infraestructura de la nube de Oracle y el nivel de aplicación en Azure.
- Configuraciones de clúster de Pacemaker de HANA de varias instancias
- Configuraciones de clúster de Windows con discos compartidos a través de SOFS o SMB en ANF para las bases de datos de SAP compatibles con Windows. En su lugar, se recomienda el uso de la replicación nativa de alta disponibilidad de las bases de datos concretas y el uso de pilas de almacenamiento independientes.
- Implementación de bases de datos de SAP que se admiten en Linux con archivos de base de datos ubicados en recursos compartidos de archivos de NFS sobre ANF con la excepción de SAP HANA.
- Implementación de DBMS de Oracle en cualquier otro sistema operativo invitado además de Windows y Oracle Linux. Consulte también la [nota de soporte técnico de SAP 2039619](https://launchpad.support.sap.com/#/notes/2039619).

Escenarios que no hemos probado y, por lo tanto, con los que no tenemos experiencia; por ejemplo:

- Azure Site Recovery replicando máquinas virtuales de nivel de DBMS. Como resultado, se recomienda aprovechar la funcionalidad de replicación asincrónica nativa de la base de datos para una posible configuración de recuperación ante desastres.
 

## <a name="next-steps"></a>Pasos siguientes
Consulte los pasos siguientes en [Implementación y planeamiento de Azure Virtual Machines para SAP NetWeaver](./planning-guide.md).




  
---
title: Configuración de la recuperación ante desastres para la implementación de aplicaciones de SAP NetWeaver de niveles múltiples con Azure Site Recovery | Microsoft Docs
description: En este artículo se describe cómo configurar la recuperación ante desastres para la implementación de aplicaciones de SAP NetWeaver mediante Azure Site Recovery.
author: asgang
manager: rochakm
ms.service: site-recovery
ms.workload: backup-recovery
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 11/27/2018
ms.author: asgang
ms.openlocfilehash: 0848738b71a605d8baf049847daa3ae2428a7abe
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65793672"
---
# <a name="set-up-disaster-recovery-for-a-multi-tier-sap-netweaver-app-deployment"></a>Configuración de la recuperación ante desastres para la implementación de una aplicación de SAP NetWeaver de niveles múltiples

La mayor parte de las implementaciones SAP grandes y medianas usan algún tipo de solución de recuperación ante desastres. La importancia de las soluciones de recuperación ante desastres sólidas y verificables ha aumentado a medida que cada vez más procesos empresariales básicos se trasladan a aplicaciones como SAP. Azure Site Recovery se probó e integró con aplicaciones SAP. Site Recovery supera las funcionalidades de la mayoría de las soluciones de recuperación ante desastres locales, con un costo total de propiedad (TCO) menor que las soluciones de la competencia.

Con Site Recovery, puede:
* **Habilitar la protección de aplicaciones de protección de SAP NetWeaver y no NetWeaver que se ejecutan de manera local**, mediante la replicación de componentes en Azure.
* **Habilitar la protección de aplicaciones de producción de SAP NetWeaver y no NetWeaver que se ejecutan en Azure**, mediante la replicación de componentes en otro centro de datos de Azure.
* **Simplificar la migración a la nube** mediante Site Recovery para migrar la implementación de SAP a Azure.
* **Simplificar las actualizaciones, las pruebas y la creación de prototipos de los proyectos de SAP**, mediante la creación de una clon en producción a petición para probar las aplicaciones SAP.

En este artículo se explica cómo proteger las implementaciones de aplicaciones SAP NetWeaver mediante [Azure Site Recovery](site-recovery-overview.md). En este artículo se describen los procedimientos recomendados para proteger una implementación de SAP NetWeaver de tres niveles en Azure mediante la replicación en otro centro de datos de Azure a través de Site Recovery. Describe los escenarios y las configuraciones compatibles y cómo realizar conmutaciones por error de prueba (maniobras de recuperación ante desastres) y conmutaciones ante desastres reales.

## <a name="prerequisites"></a>Requisitos previos
Antes de comenzar, asegúrese de que sabe cómo realizar las tareas siguientes:

* [Replicar una máquina virtual en Azure](azure-to-azure-walkthrough-enable-replication.md)
* [Diseñar una red de recuperación](site-recovery-azure-to-azure-networking-guidance.md)
* [Realizar una conmutación por error de prueba en Azure](azure-to-azure-walkthrough-test-failover.md)
* [Realizar una conmutación por error en Azure](site-recovery-failover.md)
* [Replicar un controlador de dominio](site-recovery-active-directory.md)
* [Replicación de SQL Server](site-recovery-sql.md)

## <a name="supported-scenarios"></a>Escenarios admitidos
Puede usar Site Recovery para implementar una solución de recuperación ante desastres en los escenarios siguientes:
* Sistemas SAP que se ejecutan en un centro de datos de Azure que se replican en otro centro de datos de Azure (recuperación ante desastres de Azure en Azure). Para más información, consulte [Arquitectura de replicación de Azure en Azure](https://aka.ms/asr-a2a-architecture).
* Sistemas SAP que se ejecutan en servidores VMware (o físicos) locales que se replican en un sitio de recuperación ante desastres de un centro de datos de Azure (recuperación ante desastres de VMware en Azure). Se requieren componentes adicionales para este escenario. Para más información, consulte el artículo sobre la [arquitectura de replicación de VMware en Azure](https://aka.ms/asr-v2a-architecture).
* Sistemas SAP que se ejecutan en una instancia local de Hyper-V que se replican en un sitio de recuperación ante desastres de un centro de datos de Azure (recuperación ante desastres de Hyper-V en Azure). Se requieren componentes adicionales para este escenario. Para más información, consulte [Arquitectura de replicación de Hyper-V en Azure](https://aka.ms/asr-h2a-architecture).

En este artículo se usa un escenario de recuperación ante desastres de **Azure en Azure** para demostrar las funcionalidades de recuperación ante desastres de SAP de Site Recovery. Debido a que la replicación de Site Recovery no es específica para la aplicación, el proceso descrito también se puede aplicar a otros escenarios.

### <a name="required-foundation-services"></a>Servicios básicos requeridos
En el escenario que se analiza en este artículo, se implementan los servicios básicos siguientes:
* Azure ExpressRoute o Azure VPN Gateway
* Al menos un servidor DNS y un controlador de dominio de Active Directory que se ejecutan en Azure

Se recomienda establecer esta infraestructura antes de implementar Site Recovery.

## <a name="reference-sap-application-deployment"></a>Implementación de referencia de aplicación SAP

Esta arquitectura de referencia muestra la ejecución de SAP NetWeaver en un entorno Windows en Azure con alta disponibilidad.  Esta arquitectura se implementa con tamaños de máquina virtual (VM) específicos que se pueden cambiar para acomodarse a las necesidades de la organización.

![Diagrama de un modelo de implementación de SAP típico](./media/site-recovery-sap/sap-netweaver_latest.png)

## <a name="disaster-recovery-considerations"></a>Consideraciones acerca de la recuperación ante desastres

Para la recuperación ante desastres (DR), debe ser capaz de realizar la conmutación por error en una región secundaria. Cada nivel utiliza una estrategia diferente para proporcionar protección mediante la recuperación ante desastres (DR).

#### <a name="vms-running-sap-web-dispatcher-pool"></a>Máquinas virtuales que ejecutan el grupo de SAP Web Dispatcher 
El componente Web Dispatcher se usa como equilibrador de carga para el tráfico SAP que circula entre los servidores de aplicaciones de SAP. Para lograr una alta disponibilidad para el componente de Web Dispatcher, se usa Azure Load Balancer para implementar la configuración paralela de Web Dispatcher en una configuración round-robin para la distribución del tráfico HTTP(S) entre las instancias disponibles de Web Dispatcher del grupo de equilibradores de carga. Esto se replicará mediante Azure Site Recovery (ASR) y los scripts de automatización se usarán para configurar el equilibrador de carga en la región de recuperación ante desastres. 

#### <a name="vms-running-application-servers-pool"></a>Máquinas virtuales que ejecutan el grupo de servidores de aplicaciones
Para administrar grupos de inicio de sesión para servidores de aplicaciones de ABAP, se utiliza la transacción SMLG. Usa la función de equilibrio de carga en el servidor de mensajes de Central Services para distribuir la carga de trabajo entre el grupo de servidores de aplicaciones de SAP para SAPGUI y el tráfico de RFC. Esto se replicará mediante Azure Site Recovery 

#### <a name="vms-running-sap-central-services-cluster"></a>Máquinas virtuales que ejecutan el clúster de SAP Central Services
Esta arquitectura de referencia ejecuta Central Services en máquinas virtuales en la capa de aplicación. Central Services es un posible único punto de error (SPOF) cuando se implementa en una sola máquina virtual (que es la implementación más habitual cuando la alta disponibilidad no es un requisito).<br>

Para implementar una solución de alta disponibilidad, se puede utilizar un clúster de disco compartido o un clúster de recurso compartido de archivos. Para configurar las máquinas virtuales en un clúster de disco compartido, use el clúster de conmutación por error de Windows Server. Cloud Witness se recomienda como testigo de cuórum. 
 > [!NOTE]
 > Azure Site Recovery no replica el testigo en la nube, por tanto, se recomienda implementarlo en la región de recuperación ante desastres.

Para admitir el entorno del clúster de conmutación por error, [SIOS DataKeeper Cluster Edition](https://azuremarketplace.microsoft.com/marketplace/apps/sios_datakeeper.sios-datakeeper-8) realiza la función de volumen compartido de clústeres mediante la replicación de discos independientes que pertenecen a los nodos del clúster. Azure no admite de forma nativa discos compartidos y, por consiguiente, requiere las soluciones que proporciona SIOS. 

Otra forma de controlar la agrupación en clústeres es implementar un clúster de recurso compartido de archivos. [SAP](https://blogs.sap.com/2018/03/19/migration-from-a-shared-disk-cluster-to-a-file-share-cluster) ha modificado recientemente el modelo de implementación de Central Services para acceder a los directorios globales de /sapmnt a través de una ruta de acceso UNC. Pero sigue siendo recomendable asegurarse de que el recurso compartido UNC /sapmnt tiene alta disponibilidad. Esto se puede realizar en la instancia de Central Services mediante Windows Server Failover Cluster con las características Servidor de archivos de escalabilidad horizontal (SOFS) y Espacios de almacenamiento directo (S2D) en Windows Server 2016. 
 > [!NOTE]
 > Actualmente la compatibilidad con Azure Site Recovery solo crash punto coherente con la replicación de máquinas virtuales con el nodo de directa y pasivos de espacios de almacenamiento de SIOS Datakeeper


## <a name="disaster-recovery-considerations"></a>Consideraciones acerca de la recuperación ante desastres

Puede utilizar Azure Site Recovery para orquestar la conmutación por error de la implementación completa de SAP en regiones de Azure.
A continuación, se muestran los pasos para configurar la recuperación ante desastres. 

1. Replicación de máquinas virtuales 
2. Diseño de una red de recuperación
3.  Replicación de un controlador de dominio
4.  Replicación del nivel de base de datos 
5.  Ejecución de una conmutación por error de prueba 
6.  Ejecución de una conmutación por error 

A continuación, se muestra la recomendación para la recuperación ante desastres de cada nivel utilizado en este ejemplo. 

 **Niveles SAP** | **Recomendación**
 --- | ---
**Grupo de SAP Web Dispatcher** |  Replicación con Site Recovery 
**Grupo de SAP Application Server** |  Replicación con Site Recovery 
**Clúster de SAP Central Services** |  Replicación con Site Recovery 
**Máquinas virtuales de Active Directory** |  Replicación de Active Directory 
**Servidores de SQL Database** |  SQL siempre en replicación

## <a name="replicate-virtual-machines"></a>Replicación de máquinas virtuales

Para comenzar a replicar todas las máquinas virtuales de la aplicación SAP en el centro de datos de recuperación ante desastres de Azure, siga las instrucciones que aparecen en el artículo sobre la [replicación de una máquina virtual en Azure](azure-to-azure-walkthrough-enable-replication.md).


* Para obtener instrucciones sobre la protección de Active Directory y DNS, consulte el documento [Protección de Active Directory y DNS con Azure Site Recovery](site-recovery-active-directory.md).

* Para obtener instrucciones sobre la protección de niveles de bases de datos ejecutadas en SQL server, consulte el documento [Protección de Active Directory y DNS con Azure Site Recovery](site-recovery-active-directory.md).

## <a name="networking-configuration"></a>Configuración de redes

Si usa una dirección IP estática, puede especificar la dirección IP que desea que use la máquina virtual. Para establecer la dirección IP, vaya a **Configuración de proceso y red** > **Adaptador de red**.

![Captura de pantalla que muestra cómo establecer una dirección IP privada en el panel del adaptador de red de Site Recovery](./media/site-recovery-sap/sap-static-ip.png)


## <a name="creating-a-recovery-plan"></a>Creación de un plan de recuperación
Un plan de recuperación admite la secuenciación de distintas capas en una aplicación de varios niveles durante una conmutación por error. La secuenciación ayuda a mantener la coherencia de la aplicación. Cuando cree un plan de recuperación para una aplicación web de varios niveles, complete los pasos descritos en [Creación de un plan de recuperación mediante Site Recovery](site-recovery-create-recovery-plans.md).

### <a name="adding-virtual-machines-to-failover-groups"></a>Incorporación de máquinas virtuales a grupos de conmutación por error

1.  Cree un plan de recuperación agregando el servidor de aplicaciones, Web Dispatcher y las máquinas virtuales de SAP Central Services.
2.  Haga clic en Personalizar para agrupar las máquinas virtuales. De forma predeterminada, todas las máquinas virtuales forman parte del grupo 1.



### <a name="add-scripts-to-the-recovery-plan"></a>Incorporación de scripts al plan de recuperación
Para que las aplicaciones funciones correctamente, es posible que tenga que realizar algunas operaciones en las máquinas virtuales de Azure después de la conmutación por error o durante una conmutación por error de prueba. Algunas de las operaciones posteriores a la conmutación por error se pueden automatizar. Por ejemplo, puede actualizar la entrada DNS y modificar los enlaces y las conexiones mediante la incorporación de los scripts correspondientes en el plan de recuperación.


Puede implementar los scripts de Azure Site Recovery utilizados en su cuenta de Automation haciendo clic en el botón Implementar en Azure más abajo. Si utiliza un script publicado, asegúrese de seguir las instrucciones en el script.

[![Implementación en Azure](https://azurecomcdn.azureedge.net/mediahandler/acomblog/media/Default/blog/c4803408-340e-49e3-9a1f-0ed3f689813d.png)](https://aka.ms/asr-automationrunbooks-deploy)

1. Agregue un script anterior a la acción del grupo 1 al grupo de disponibilidad de SQL de conmutación por error. Use el script ASR-SQL-FailoverAG publicado en los scripts de ejemplo. Asegúrese de seguir las instrucciones en el script y realice los cambios necesarios de forma adecuada.
2. Agregue un script posterior a la acción para adjuntar un equilibrador de carga en las máquinas virtuales conmutadas por error del nivel web (grupo 1). Use el script ASR-AddSingleLoadBalancer publicado en los scripts de ejemplo. Asegúrese de seguir las instrucciones en el script y realice los cambios necesarios de forma adecuada.

![Plan de recuperación de SAP](./media/site-recovery-sap/sap_recovery_plan.png)


## <a name="run-a-test-failover"></a>Ejecución de una conmutación por error de prueba

1.  En Azure Portal, seleccione el almacén de Recovery Services.
2.  Seleccione el plan de recuperación que creó para las aplicaciones SAP.
3.  Seleccione **Conmutación por error de prueba**.
4.  Para iniciar el proceso de conmutación por error de prueba, seleccione el punto de recuperación y la red virtual de Azure.
5.  Cuando el entorno secundario esté activo, realice las validaciones.
6.  Cuando se completen las validaciones, seleccione **Limpiar conmutación por error de prueba** para limpiar el entorno de conmutación por error.

Para más información, consulte [Conmutación por error de prueba a Azure en Site Recovery](site-recovery-test-failover-to-azure.md).

## <a name="run-a-failover"></a>Ejecución de la conmutación por error

1.  En Azure Portal, seleccione el almacén de Recovery Services.
2.  Seleccione el plan de recuperación que creó para las aplicaciones SAP.
3.  Seleccione **Failover** (Conmutación por error).
4.  Para iniciar el proceso de conmutación por error, seleccione el punto de recuperación.

Para más información, consulte [Conmutación por error en Site Recovery](site-recovery-failover.md).

## <a name="next-steps"></a>Pasos siguientes
* Para más información acerca de cómo crear una solución de recuperación ante desastres con las implementaciones de SAP NetWeaver mediante Site Recovery, consulte las notas del producto sobre la [creación de una solución de recuperación ante desastres de SAP NetWeaver con Azure Site Recovery](https://aka.ms/asr_sap), que puede descargar. En las notas del producto se analizan las recomendaciones para distintas arquitecturas SAP, se muestran las aplicaciones y los tipos de máquinas virtuales compatibles con SAP en Azure y se describen las opciones del plan de pruebas para la solución de recuperación ante desastres.
* Obtenga más información sobre cómo [replicar otras cargas de trabajo](site-recovery-workload.md) con Site Recovery.

---
title: Movimiento del proceso del sistema central a Azure Virtual Machines
description: Los recursos de proceso de Azure son comparables a la capacidad del sistema central, lo que le permite migrar y modernizar las aplicaciones de IBM z14.
author: njray
ms.author: larryme
ms.date: 04/02/2019
ms.topic: article
ms.service: multiple
ms.openlocfilehash: 9c5941ec88cd793961ad66245d0dc0b5e0d7772f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "86998942"
---
# <a name="move-mainframe-compute-to-azure"></a>Movimiento del proceso del sistema central a Azure

Los sistemas centrales tienen buena reputación en cuanto a fiabilidad y disponibilidad, por lo que siguen siendo el sistema vertebrador de confianza en muchas empresas. Se suele considerar que tienen una escalabilidad y una capacidad de proceso casi ilimitadas. Sin embargo, en algunas empresas se ha superado la capacidad de los sistemas centrales más grandes que hay disponibles. Si esto le suena, Azure le ofrece agilidad, alcance y ahorros en infraestructura.

Para ejecutar las cargas de trabajo de los sistemas centrales en Microsoft Azure, debe saber cuáles son las similitudes entre las capacidades de proceso de su sistema central y las de Azure. En este artículo se explica cómo obtener resultados similares en Azure basándose en el sistema central de IBM z14 (el modelo más reciente al redactar este documento).

Para empezar, compararemos ambos entornos en paralelo. En la siguiente ilustración se compara un entorno de sistema central para aplicaciones en ejecución con un entorno de hospedaje de Azure.

![Los entornos de emulación y los servicios de Azure ofrecen un soporte técnico y una optimización de migración comparables](media/mainframe-compute-azure.png)

A menudo se usa la eficacia de los sistemas centrales para sistemas de procesamiento de transacciones en línea (OLTP) que controlan millones de actualizaciones para miles de usuarios. Estas aplicaciones suelen usar software para el procesamiento de transacciones, el control de la pantalla y la entrada del formulario. Además, pueden usar un sistema de control de información del cliente (CICS), un sistema de administración de la información (IMS) o un paquete de la interfaz de transacciones (TIP).

Como se muestra en la ilustración, un emulador de TPM en Azure puede controlar las cargas de trabajo del CICS y del IMS. Un emulador del sistema por lotes en Azure realiza la función de lenguaje de control de tareas (JCL). Los datos del sistema central se migran a bases de datos de Azure, como Azure SQL Database. Se pueden usar los servicios de Azure u otro software hospedado en Azure Virtual Machines para administrar el sistema.

## <a name="mainframe-compute-at-a-glance"></a>Información general sobre el proceso del sistema central

En el sistema central z14, los procesadores se organizan en hasta cuatro *alimentadores*. Un *alimentador* es simplemente un clúster de procesadores y de conjuntos de chips. Cada alimentador puede tener seis chips de procesador central (CP) activos y cada CP tiene diez chips de controlador del sistema (SC). En la terminología de Intel x86, hay seis sockets por alimentador, diez núcleos por socket y cuatro alimentadores. Esta arquitectura proporciona un equivalente aproximado de 24 sockets y 240 núcleos, como máximo, para un z14.

El CP z14 rápido tiene una velocidad de reloj de 5,2 GHz. Normalmente, el z14 incluye todos los CP, que se activan según sea necesario. Lo más habitual es que al cliente se le cobre por un mínimo de cuatro horas de tiempo de proceso al mes, independientemente de cuál sea su uso real.

Un procesador de sistema central se puede configurar como uno de los siguientes tipos:

- Procesador de propósito general (GP)
- Procesador de información integrada en el sistema z (zIIP)
- Procesador con instalaciones integradas para Linux (IFL)
- Procesador de asistencia al sistema (SAP)
- Procesador de instalaciones de acoplamiento integrado (ICF)

## <a name="scaling-mainframe-compute-up-and-out"></a>Escalado vertical y horizontal del proceso de sistema central

Los sistemas centrales de IBM ofrecen la posibilidad de escalar verticalmente hasta 240 núcleos (el tamaño de z14 actual para un sistema único). Además, los sistemas centrales de IBM pueden escalar horizontalmente con una característica denominada “recurso de acoplamiento” (CF). El CF permite que diferentes sistemas centrales accedan simultáneamente a los mismos datos. Con este recurso, la tecnología Parallel Sysplex del sistema central agrupa los procesadores del sistema central en clústeres. En el momento de escribir esta guía, la característica de Parallel Sysplex admite 32 agrupaciones de 64 procesadores cada una. Se pueden agrupar de esta forma hasta 2048 procesadores para escalar horizontalmente la capacidad del proceso.

Los recursos de acoplamiento permiten a los clústeres de proceso compartir datos con acceso directo. Se utilizan para bloquear información, información de la caché y la lista de recursos de datos compartido. Un Sysplex Parallel que use un CP o más se puede considerar como un clúster de proceso de escalabilidad horizontal “con todo compartido”. Para más información sobre estas características, vea [Parallel Sysplex on IBM Z](https://www.ibm.com/it-infrastructure/z/technologies/parallel-sysplex-resources) (Parallel Sysplex en IBM Z) en el sitio web de IBM.

Las aplicaciones pueden usar estas características para ofrecer un rendimiento de escalabilidad horizontal y alta disponibilidad. Para obtener información sobre cómo el CICS puede utilizar Parallel Sysplex con CF, descargue el redbook [IBM CICS and the Coupling Facility: Beyond the Basics](https://www.redbooks.ibm.com/redbooks/pdfs/sg248420.pdf) (IBM CICS y el recurso de acoplamiento: más allá de los aspectos básicos).

## <a name="azure-compute-at-a-glance"></a>Información general sobre el proceso de Azure

Algunas personas piensan equivocadamente que los servidores basados en Intel no son tan potentes como los sistemas centrales. Sin embargo, los nuevos sistemas basados en Intel con un gran número de núcleos tienen tanta capacidad de proceso como los sistemas centrales. En esta sección se describen las opciones de infraestructura como servicio (IaaS) de Azure para el proceso y almacenamiento. Azure también ofrece opciones de plataforma como servicio (PaaS), pero este artículo se centra en las opciones de IaaS que proporcionan la capacidad de sistema central comparable.

Azure Virtual Machines proporciona capacidad de proceso en varios tamaños y tipos. En Azure, una CPU virtual (vCPU) equivale aproximadamente a un núcleo de un sistema central.

Actualmente, los tamaños de Azure Virtual Machines disponibles van de 1 a 128 vCPU. Los tipos de máquinas virtuales (VM) están optimizados para cargas de trabajo concretas. Por ejemplo, la siguiente lista recoge los tipos de máquina virtual (existentes al redactar este documento) y sus usos recomendados:

| Size     | Tipo y descripción                                                                 |
|----------|--------------------------------------------------------------------------------------|
| Serie D | Uso general con 64 vCPU y velocidad de reloj de hasta 3,5 GHz                           |
| Serie E | Optimizada para memoria con hasta 64 vCPU                                                 |
| Serie F | Optimizada para proceso con hasta 64 vCPU y velocidad de reloj de 3,7 GHz                       |
| Serie H | Optimizada para aplicaciones de informática de alto rendimiento (HPC)                          |
| Serie L | Optimizada para almacenamiento para aplicaciones de alto rendimiento respaldadas por bases de datos como NoSQL |
| Serie M | Máquinas virtuales optimizadas para memoria y con el proceso más largo, con hasta 128 vCPU                        |

Para obtener más información sobre las máquinas virtuales disponibles, vea [Serie de máquinas virtuales](https://azure.microsoft.com/pricing/details/virtual-machines/series/).

Un sistema central z14 puede tener hasta 240 núcleos. Sin embargo, los sistemas centrales z14 casi nunca usan todos los núcleos para una sola aplicación o carga de trabajo, sino que aíslan las cargas de trabajo en particiones lógicas (LPAR). A su vez, las LPAR tienen clasificaciones: MIPS (millones de instrucciones por segundo) o MSU (millones de unidades de servicio). Al determinar el tamaño comparable de máquina virtual necesario para ejecutar una carga de trabajo del sistema central en Azure, hay que factorizar la clasificación de MIPS (o MSU).

Las siguientes son estimaciones generales:

-   150 MIPS por vCPU

-   1000 MIPS por procesador

Para determinar el tamaño de máquina virtual correcto para una carga de trabajo determinada en una LPAR, primero optimice la máquina virtual para la carga de trabajo. A continuación, determine el número de vCPU necesarias. Una estimación conservadora es 150 MIPS por vCPU. Según esta estimación, por ejemplo, una máquina virtual de la serie F con 16 vCPU podría admitir fácilmente una carga de trabajo de IBM Db2 procedente de una LPAR con 2400 MIPS.

## <a name="azure-compute-scale-up"></a>Escalabilidad vertical del proceso de Azure

Las máquinas virtuales de la serie M pueden escalar verticalmente hasta 128 vCPU (en el momento de escribir este artículo). Al usar la estimación conservadora de 150 MIPS por vCPU, la máquina virtual de la serie M equivale a unos 19 000 MIPS. La regla general para estimar MIPS para un sistema central es de 1000 MIPS por procesador. Un sistema central z14 puede tener hasta 24 procesadores y proporcionar aproximadamente 24 000 MIPS para un sistema central único.

El sistema central z14 más grande tiene aproximadamente 5000 MIPS más que la máquina virtual más grande disponible en Azure. Aun así, es importante comparar cómo se implementan las cargas de trabajo. Si un sistema central tiene una aplicación y una base de datos relacional, normalmente se implementa en el mismo sistema central físico, cada uno en su propia LPAR. A menudo se implementa la misma solución en Azure mediante una máquina virtual para la aplicación y una máquina virtual independiente con el tamaño apropiado para la base de datos.

Por ejemplo, si un sistema de vCPU M64 admite la aplicación y se usa una vCPU M96 para la base de datos, se necesitan aproximadamente 150 vCPU o 24 000 MIPS, como se muestra en la figura siguiente.

![Comparación de las implementaciones de carga de trabajo de 24 000 MIPS](media/mainframe-compute-mips.png)

El enfoque es migrar las LPAR a máquinas virtuales individuales. A continuación, Azure escala verticalmente y de forma sencilla hasta el tamaño necesario para la mayoría de las aplicaciones que se implementan en un sistema central único.

## <a name="azure-compute-scale-out"></a>Escalabilidad horizontal del proceso de Azure

Una de las ventajas de una solución basada en Azure es la capacidad para escalar horizontalmente. La escalabilidad pone a disposición de una aplicación una capacidad de proceso casi ilimitada. Azure admite varios métodos para escalar horizontalmente la capacidad de proceso:

- **Equilibrio de carga en un clúster.** En este escenario, una aplicación puede utilizar un [equilibrador de carga](../../../../load-balancer/load-balancer-overview.md) o un administrador de recursos para repartir la carga de trabajo entre varias máquinas virtuales de un clúster. Si se necesita más capacidad de procesamiento, se agregan más máquinas virtuales al clúster.

- **Conjuntos de escalado de máquinas virtuales.** En este escenario de ráfaga, una aplicación puede escalar a [recursos de proceso](../../../../virtual-machine-scale-sets/overview.md) adicionales según el uso de la máquina virtual. Cuando la demanda cae, el número de máquinas virtuales de un conjunto de escalado también puede disminuir, lo que garantiza un uso eficaz de la capacidad de proceso.

- **Escalado de PaaS.** Las ofertas de PaaS de Azure escalan recursos de proceso. Por ejemplo, [Azure Service Fabric](../../../../service-fabric/service-fabric-overview.md) asigna recursos de proceso para satisfacer aumentos en el volumen de solicitudes.

- **Clústeres de Kubernetes.** Las aplicaciones en Azure pueden usar [clústeres de Kubernetes](../../../../aks/concepts-clusters-workloads.md) para servicios de proceso en los recursos especificados. Azure Kubernetes Service (AKS) es un servicio administrado que organiza los nodos, grupos y clústeres de Kubernetes en Azure.

Para elegir el método adecuado para el escalado horizontal de los recursos de proceso, es importante entender las diferencias entre Azure y los sistemas centrales. La clave está en cómo los recursos de proceso comparten los datos o si lo hacen. En Azure, las diferentes máquinas virtuales no acostumbran a compartir los datos (de forma predeterminada). Si varias máquinas virtuales requieren el uso compartido de datos en un clúster de proceso de escalabilidad horizontal, los datos compartidos deben residir en un recurso que admita esta funcionalidad. En Azure, el uso compartido de datos implica almacenamiento tal como se describe en la sección siguiente.

## <a name="azure-compute-optimization"></a>Optimización del proceso de Azure

Puede optimizar cada nivel de procesamiento en una arquitectura de Azure. Utilice el tipo de máquina virtual y las características más adecuadas para cada entorno. En la siguiente ilustración se muestra un patrón posible para implementar máquinas virtuales en Azure de forma que se admita una aplicación CICS que use Db2. En el sitio principal, las máquinas virtuales de producción, preproducción y pruebas se implementan con alta disponibilidad. El sitio secundario es para recuperación ante desastres y copia de seguridad.

Además, cada nivel también puede proporcionar servicios adecuados de recuperación ante desastres. Por ejemplo, las máquinas virtuales de producción y base de datos pueden requerir una recuperación activa o semiactiva, mientras que las máquinas virtuales de desarrollo y pruebas permiten una recuperación pasiva.

![Implementación de alta disponibilidad que admite la recuperación ante desastres](media/mainframe-compute-dr.png)

## <a name="next-steps"></a>Pasos siguientes

- [Migración del sistema central](/azure/architecture/cloud-adoption/infrastructure/mainframe-migration/overview)
- [Rehospedaje del sistema central en Azure Virtual Machines](../overview.md)
- [Mover el almacenamiento del sistema central a Azure](mainframe-storage-Azure.md)

### <a name="ibm-resources"></a>Recursos de IBM

- [Parallel Sysplex on IBM Z](https://www.ibm.com/it-infrastructure/z/technologies/parallel-sysplex-resources) (Parallel Sysplex en IBM Z)
- [IBM CICS and the Coupling Facility: Beyond the Basics](https://www.redbooks.ibm.com/redbooks/pdfs/sg248420.pdf) (IBM CICS y el recurso de acoplamiento: más allá de los aspectos básicos)
- [Creación de los usuarios necesarios para una instalación de Db2 pureScale Feature](https://www.ibm.com/support/knowledgecenter/en/SSEPGG_11.1.0/com.ibm.db2.luw.qb.server.doc/doc/t0055374.html?pos=2)
- [Mandato db2icrt - Crear instancia](https://www.ibm.com/support/knowledgecenter/en/SSEPGG_11.1.0/com.ibm.db2.luw.admin.cmd.doc/doc/r0002057.html)
- [Db2 pureScale Clustered Database Solution](https://www.ibmbigdatahub.com/blog/db2-purescale-clustered-database-solution-part-1) (Solución de base de datos en clúster de Db2 pureScale)
- [IBM Data Studio](https://www.ibm.com/developerworks/downloads/im/data/index.html/)

### <a name="azure-government"></a>Azure Government

- [Microsoft Azure Government cloud for mainframe applications](https://azure.microsoft.com/resources/microsoft-azure-government-cloud-for-mainframe-applications/) (Nube de Microsoft Azure Government para aplicaciones de sistema central)
- [Microsoft y FedRAMP](https://www.microsoft.com/TrustCenter/Compliance/FedRAMP)

### <a name="more-migration-resources"></a>Más recursos sobre migración

- [Azure Virtual Data Center Lift and Shift Guide](https://azure.microsoft.com/resources/azure-virtual-datacenter-lift-and-shift-guide/) (Centro de datos virtual de Azure: guía para una migración lift-and-shift)
- [iSCSI en GlusterFS](https://docs.gluster.org/en/latest/Administrator%20Guide/GlusterFS%20iSCSI/)

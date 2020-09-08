---
title: Clústeres de MATLAB en máquinas virtuales
description: Utilice Máquinas virtuales de Microsoft Azure para crear clústeres de MATLAB Distributed Computing Server con el fin de ejecutar sus cargas de trabajo MATLAB paralelas de proceso intensivo.
author: mscurrell
ms.service: virtual-machines-windows
ms.subservice: workloads
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 05/09/2016
ms.author: markscu
ms.openlocfilehash: c2561a6dc3ad8c0af1c266b3822a80c76f45c174
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88639688"
---
# <a name="create-matlab-distributed-computing-server-clusters-on-azure-vms"></a>Creación de clústeres de MATLAB Distributed Computing Server en Máquinas virtuales de Azure
Utilice Máquinas virtuales de Microsoft Azure para crear uno o más clústeres de MATLAB Distributed Computing Server con el fin de ejecutar sus cargas de trabajo MATLAB paralelas de proceso intensivo. Instale el software MATLAB Distributed Computing Server en una máquina virtual para usarlo como una imagen base y usar una plantilla de inicio rápido de Azure o un script de Azure PowerShell (disponible en [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/matlab-cluster)) para implementar y administrar el clúster. Después de la implementación, conéctese al clúster para ejecutar sus cargas de trabajo.

> [!IMPORTANT]
> Desde que este artículo se ha escrito, ahora hay compatibilidad oficial con el uso de aplicaciones de MATLAB en Azure. Se recomienda usar estas funcionalidades más recientes en lugar de la plantilla y los scripts a los que se hace referencia en este artículo. Busque "MATLAB" en [Azure Marketplace](https://azuremarketplace.microsoft.com/); puede encontrar más información sobre la ejecución de aplicaciones de MATLAB en Azure en [MathWorks](https://www.mathworks.com/solutions/cloud.html#public-cloud).

## <a name="about-matlab-and-matlab-distributed-computing-server"></a>Información acerca de MATLAB y MATLAB Distributed Computing Server 
La plataforma [MATLAB](https://www.mathworks.com/products/matlab/) está optimizada para solucionar problemas científicos y de ingeniería. Los usuarios de MATLAB con tareas de simulación y procesamiento de datos a gran escala pueden utilizar productos de computación en paralelo de MathWorks para acelerar sus cargas de trabajo de proceso intensivo, ya que aprovechan las ventajas de los clústeres de proceso y los servicios en malla. [Parallel Computing Toolbox](https://www.mathworks.com/products/parallel-computing/) permite a los usuarios de MATLAB usar las aplicaciones en paralelo y aprovechar las ventajas de los procesadores de núcleo múltiple, GPU y clústeres de proceso. [MATLAB Distributed Computing Server](https://www.mathworks.com/products/distriben/) permite a los usuarios de MATLAB usar muchos equipos en un clúster de proceso.

Mediante el uso de máquinas virtuales de Azure, puede crear clústeres de MATLAB Distributed Computing Server, que tienen los mismos mecanismos disponibles para enviar el trabajo paralelo como clústeres locales; por ejemplo, trabajos interactivos, trabajos por lotes, tareas independientes y tareas de comunicación. El uso de Azure junto con la plataforma MATLAB tiene muchas ventajas en comparación con el aprovisionamiento y uso tradicional de hardware local: diversos tamaños de máquina virtual, creación de clústeres a petición para pagar solo por los recursos de proceso que utilice y la capacidad de probar modelos a escala.  

## <a name="prerequisites"></a>Requisitos previos
* **Equipo cliente** : necesitará un equipo cliente basado en Windows para comunicarse con Azure y el clúster de MATLAB Distributed Computing Server después de la implementación.
* **Azure PowerShell** : consulte [Cómo instalar y configurar Azure PowerShell](/powershell/azure/) para instalarlo en el equipo cliente.
* **Suscripción de Azure** : si no tiene ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/) en un par de minutos. En los clústeres más grandes, considere la posibilidad de una suscripción de pago por uso u otras opciones de compra.
* **Cuota de vCPU**: es posible que tenga que aumentar la cuota de vCPU para implementar un clúster grande o más de un clúster de MATLAB Distributed Computing Server. Para aumentar una cuota, [abra una solicitud de soporte técnico al cliente en línea](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/) sin cargo alguno.
* **Licencias de MATLAB, Parallel Computing Toolbox y MATLAB Distributed Computing Server** ; los scripts asumen que se usa [MathWorks Hosted License Manager](https://www.mathworks.com/help/install/license-management.html) para todas las licencias.  
* **Software MATLAB Distributed Computing Server** : se instalará en una máquina virtual que se utilizará como la imagen de máquina virtual base para las máquinas virtuales del clúster.

## <a name="high-level-steps"></a>Pasos de alto nivel
Para usar máquinas virtuales de Azure para los clústeres de MATLAB Distributed Computing Server, son necesarios los siguientes pasos de alto nivel. Hay instrucciones detalladas en la documentación que acompaña a la plantilla de inicio rápido y scripts en [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/matlab-cluster).

1. **Creación de una imagen de máquina virtual base**  

   * Descargue e instale el software MATLAB Distributed Computing Server en esta máquina virtual.

     > [!NOTE]
     > Este proceso puede tardar un par de horas, pero solo tiene que hacerlo una vez para cada versión de MATLAB que use.   
     >
     >
2. **Creación de uno o varios clústeres**  

   * Use el script de PowerShell proporcionado o utilice la plantilla de inicio rápido para crear un clúster a partir de la imagen de máquina virtual base.   
   * Administre los clústeres mediante el script de PowerShell proporcionado, que permite enumerar, pausar, reanudar y eliminar los clústeres.

## <a name="cluster-configurations"></a>Configuraciones de clústeres
Actualmente, el script y la plantilla de creación de clústeres permiten crear una sola topología de MATLAB Distributed Computing Server. Si lo desea, cree uno o más clústeres adicionales en los que cada clúster tendrá un número diferente de máquinas virtuales de trabajo, usará diferentes tamaños de máquina virtual, etc.

### <a name="matlab-client-and-cluster-in-azure"></a>Clúster y cliente de MATLAB en Azure
El nodo cliente de MATLAB, el nodo de MATLAB Job Scheduler y los nodos de "trabajo" de MATLAB Distributed Computing Server se configuran como máquinas virtuales de Azure en una red virtual, como se muestra en la ilustración siguiente.


* Para utilizar el clúster, conéctese mediante Escritorio remoto al nodo cliente. El nodo cliente ejecuta el cliente de MATLAB.
* El nodo cliente tiene un recurso compartido de archivos al que tienen acceso todos los trabajadores.
* MathWorks Hosted License Manager se usa para las comprobaciones de licencia de todo el software de MATLAB.
* De forma predeterminada, se crea un trabajo de MATLAB Distributed Computing Server por vCPU en las máquinas virtuales de trabajo, pero se puede especificar cualquier número.

## <a name="use-an-azure-based-cluster"></a>Uso de un clúster de Azure
Como con otros tipos de clústeres de MATLAB Distributed Computing Server, debe usar Cluster Profile Manager en el cliente de MATLAB (en la máquina virtual del cliente) para crear un perfil de clúster en MATLAB Job Scheduler.

![Cluster Profile Manager](./media/matlab-mdcs-cluster/cluster_profile_manager.png)

## <a name="next-steps"></a>Pasos siguientes
* Para instrucciones detalladas para implementar y administrar clústeres de MATLAB Distributed Computing Server en Azure, consulte el repositorio [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/matlab-cluster) , que contiene las plantillas y los scripts.
* Vaya al [sitio de MathWorks](https://www.mathworks.com/) para ver documentación detallada de MATLAB y MATLAB Distributed Computing Server.

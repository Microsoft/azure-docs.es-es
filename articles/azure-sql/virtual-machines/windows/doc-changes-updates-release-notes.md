---
title: Cambios en la documentación de SQL Server en Azure Virtual Machines | Microsoft Docs
description: Conozca las nuevas características y mejoras de las diferentes versiones de SQL Server en Azure Virtual Machines.
services: virtual-machines-windows
author: MashaMSFT
ms.author: mathoma
tags: azure-service-management
ms.assetid: 2fa5ee6b-51a6-4237-805f-518e6c57d11b
ms.service: virtual-machines-sql
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 08/20/2020
ms.openlocfilehash: 5f0ba27a0964f08b32e1af19d6f578b3f41ee300
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89006421"
---
# <a name="documentation-changes-for-sql-server-on-azure-virtual-machines"></a>Cambios en la documentación de SQL Server en Azure Virtual Machines
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Azure permite implementar una máquina virtual (VM) con una imagen de SQL Server integrada. En este artículo se resumen los cambios en la documentación asociados a las nuevas características y mejoras de las versiones recientes de [SQL Server en Azure Virtual Machines](https://azure.microsoft.com/services/virtual-machines/sql-server/). 

## <a name="august-2020"></a>Agosto de 2020

| Cambios | Detalles |
| --- | --- |
| **Configuración AG en el portal** | Ahora es posible [configurar el grupo de disponibilidad a través de Azure Portal](availability-group-azure-portal-configure.md). Esta característica se encuentra actualmente en versión preliminar y en implementación, de modo que si la región deseada no está disponible, vuelva a comprobarlo pronto. | 


## <a name="july-2020"></a>Julio de 2020


| Cambios | Detalles |
| --- | --- |
| **Migración del registro a un disco Ultra** | Obtenga información acerca de cómo puede [migrar el archivo de registro a un disco Ultra](storage-migrate-to-ultradisk.md) para aprovechar el alto rendimiento y la baja latencia. | 
| **Creación de AG mediante PowerShell** | Ahora es posible simplificar la creación de un grupo de disponibilidad mediante [PowerShell](availability-group-az-commandline-configure.md), así como la CLI de Azure. | 


## <a name="june-2020"></a>Junio de 2020

| Cambios | Detalles |
| --- | --- |
| **Nombre de red distribuida (DNN)** | Ahora, SQL Server 2019 en Windows Server 2016+ ofrece en versión preliminar compatibilidad con el enrutamiento del tráfico a la instancia de clúster de conmutación por error (FCI) mediante el uso de un [nombre de red distribuida](hadr-distributed-network-name-dnn-configure.md) en lugar de con Azure Load Balancer. Esta compatibilidad simplifica y agiliza la conexión a la solución de alta disponibilidad en Azure. | 
| **FCI con discos compartidos de Azure** | Ahora es posible implementar la [instancia de clúster de conmutación por error (FCI)](failover-cluster-instance-overview.md) mediante [discos compartidos de Azure](failover-cluster-instance-azure-shared-disks-manually-configure.md). |
| **Documentos de FCI reorganizados** | La documentación sobre las [instancias de clúster de conmutación por error con SQL Server en Azure Virtual Machines](failover-cluster-instance-overview.md) se ha vuelto a redactar y reorganizar para mayor claridad. Hemos separado parte del contenido de configuración, como los [procedimientos recomendados de configuración de clústeres](hadr-cluster-best-practices.md), cómo preparar una [máquina virtual para una FCI de SQL Server](failover-cluster-instance-prepare-vm.md) y cómo configurar [Azure Load Balancer](hadr-vnn-azure-load-balancer-configure.md). | 
| &nbsp; | &nbsp; |


## <a name="may-2020"></a>Mayo de 2020 

| Cambios | Detalles |
| --- | --- |
| **Familia Azure SQL** | SQL Server en Azure Virtual Machines ahora forma parte de la [familia de productos de Azure SQL](../../azure-sql-iaas-vs-paas-what-is-overview.md). Eche un vistazo a nuestro [nuevo aspecto](../index.yml). No ha cambiado nada en el producto, pero la documentación pretende facilitar la decisión sobre los productos de Azure SQL. | 


## <a name="january-2020"></a>Enero de 2020

| Cambios | Detalles |
| --- | --- |
| **Soporte de Azure Government** | Ahora es posible registrar máquinas virtuales con SQL Server mediante el proveedor de recursos de VM con SQL para máquinas virtuales hospedadas en la nube de [Azure Government](https://azure.microsoft.com/global-infrastructure/government/). | 
| &nbsp; | &nbsp; |

## <a name="2019"></a>2019

|Cambios | Detalles |
 --- | --- |
| **Réplica de DR gratuita en Azure** | Puede hospedar una [instancia pasiva gratuita](business-continuity-high-availability-disaster-recovery-hadr-overview.md#free-dr-replica-in-azure) para la recuperación ante desastres en Azure para su instancia de SQL Server local si tiene [Software Assurance](https://www.microsoft.com/licensing/licensing-programs/software-assurance-default?rtc=1&activetab=software-assurance-default-pivot:primaryr3). | 
| **Registro masivo del proveedor de recursos** | Ahora puede [registrar masivamente](sql-vm-resource-provider-bulk-register.md) las máquinas virtuales con SQL Server mediante el proveedor de recursos. | 
|**Configuración del almacenamiento optimizado para el rendimiento** | Ya se puede [personalizar totalmente la configuración del almacenamiento](storage-configuration.md#new-vms) al crear una nueva máquina virtual con SQL Server. |
|**Recursos compartidos de archivos premium para FCI** | Ahora puede crear una instancia de clúster de conmutación por error mediante un [recurso compartido de archivos Premium](failover-cluster-instance-premium-file-share-manually-configure.md) en lugar del método original de [Espacios de almacenamiento directo](failover-cluster-instance-storage-spaces-direct-manually-configure.md). 
| **Azure Dedicated Host** | Puede ejecutar VM con SQL Server en [Azure Dedicated Host](dedicated-host.md). | 
| **Migración de VM con SQL Server a una región diferente** | Use Azure Site Recovery para [migrar las máquinas virtuales con SQL Server de una región de Azure a otra](move-sql-vm-different-region.md). |
|  **Nuevos modos de instalación de IaaS de SQL** | Ahora es posible instalar la extensión IaaS de SQL Server en [modo ligero](sql-server-iaas-agent-extension-automate-management.md) para evitar tener que reiniciar el servicio SQL Server.  |
| **Modificación de la edición de SQL Server** | Ahora puede cambiar la [propiedad edition](change-sql-server-edition.md) de la VM con SQL Server. |
| **Cambios con el proveedor de recursos de VM con SQL** | Puede [registrar la máquina virtual con SQL Server con el proveedor de recursos de máquina virtual con SQL](sql-vm-resource-provider-register.md) mediante los nuevos modos de IaaS de SQL. Esta funcionalidad incluye [imágenes de Windows Server 2008](sql-vm-resource-provider-register.md#management-modes).|
| **Imágenes del tipo "traiga su propia licencia" mediante la Ventaja híbrida de Azure** | Ahora es posible cambiar al [tipo de licencia de pago por uso](licensing-model-azure-hybrid-benefit-ahb-change.md#remarks) las imágenes del tipo "traiga su propia licencia" implementadas desde Azure Marketplace.| 
| **Nueva administración de VM con SQL Server en Azure Portal** | Ahora existe una nueva forma de administrar una máquina virtual con SQL Server en Azure Portal. Para más información, consulte [Administración de máquinas virtuales con SQL Server en Azure Portal](manage-sql-vm-portal.md).  | 
| **Compatibilidad ampliada con SQL Server 2008 y SQL Server 2008 R2** | [Amplíe la compatibilidad](sql-server-2008-extend-end-of-support.md) con SQL Server 2008 y SQL Server 2008 R2 mediante la migración *tal cual* a una máquina virtual de Azure. | 
| **Compatibilidad de imágenes personalizada** | Ahora puede instalar la [extensión IaaS de SQL Server](sql-server-iaas-agent-extension-automate-management.md#installation) en las imágenes de SQL Server y de sistemas operativos personalizadas, lo que ofrece una funcionalidad limitada de [licencias flexibles](licensing-model-azure-hybrid-benefit-ahb-change.md). Al registrar una imagen personalizada con el proveedor de recursos de VM con SQL, especifique el tipo de licencia como "AHUB". De lo contrario, el registro no podrá realizarse. | 
| **Compatibilidad de instancias con nombre** | Ahora puede usar la [extensión IaaS de SQL Server](sql-server-iaas-agent-extension-automate-management.md#installation) con una instancia con nombre, si la instancia predeterminada se ha desinstalado correctamente. | 
| **Renovación del portal** | La experiencia de Azure Portal para implementar una VM con SQL Server se ha renovado para mejorar la facilidad de uso. Para más información, vea el breve [inicio rápido](sql-vm-create-portal-quickstart.md) y la guía [paso a paso](create-sql-vm-portal.md) detallada para implementar una máquina virtual con SQL Server.|
| **Mejora del portal** | Ahora es posible cambiar el modelo de licencias para una máquina virtual con SQL Server del tipo "pago por uso" a "traiga su propia licencia" mediante [Azure Portal](licensing-model-azure-hybrid-benefit-ahb-change.md#vms-already-registered-with-the-resource-provider).|
| **Simplificación de la implementación de un grupo de disponibilidad en una instancia de VM con SQL Server mediante la CLI de Azure** | Ahora es más fácil que nunca implementar un grupo de disponibilidad en una máquina virtual con SQL Server en Azure. Puede usar la [CLI de Azure](/cli/azure/sql/vm?view=azure-cli-2018-03-01-hybrid) para crear el clúster de conmutación por error de Windows, el equilibrador de carga interno y los clientes de escucha de grupo de disponibilidad, todo ello desde la línea de comandos. Para más información, consulte [Uso de la CLI de Azure para configurar un grupo de disponibilidad Always On para SQL Server en una máquina virtual de Azure](availability-group-az-cli-configure.md). | 
| &nbsp; | &nbsp; |

## <a name="2018"></a>2018 

 Cambios | Detalles |
| --- | --- |
|  **Nuevo proveedor de recursos para un clúster de SQL Server** | Un nuevo proveedor de recursos (Microsoft.SqlVirtualMachine/SqlVirtualMachineGroups) que define los metadatos del clúster de conmutación por error de Windows. Al unir una máquina virtual con SQL Server a *SqlVirtualMachineGroups*, se arranca el servicio Clúster de conmutación por error de Windows Server (WSFC) y la máquina virtual se une al clúster.  |
| **Configuración automatizada de la implementación de un grupo de disponibilidad con plantillas de inicio rápido de Azure** |Ahora es posible crear el clúster de conmutación por error de Windows, unirlo a las máquinas virtuales VM con SQL Server, crear el cliente de escucha y configurar el equilibrador de carga interno con dos plantillas de inicio rápido de Azure. Para más información, consulte [Uso de plantillas de inicio rápido de Azure para configurar un grupo de disponibilidad Always On para SQL Server en una máquina virtual de Azure](availability-group-quickstart-template-configure.md). | 
| **Registro automático del proveedor de recursos de máquina virtual con SQL Server** | Las máquinas virtuales VM con SQL Server implementadas después de este mes se registran automáticamente con el nuevo proveedor de recursos de SQL Server. Las máquinas virtuales con SQL Server implementadas antes de este mes todavía deben registrarse manualmente. Para más información, consulte [Registro de una máquina virtual con SQL Server en Azure con el proveedor de recursos de máquina virtual con SQL](sql-vm-resource-provider-register.md).|
|**Nuevo proveedor de recursos de VM con SQL** |  Un nuevo proveedor de recursos (Microsoft.SqlVirtualMachine) proporciona una mejor administración de las máquinas virtuales con SQL Server. Para más información sobre cómo registrar las máquinas virtuales, consulte [Registro de una máquina virtual con SQL Server en Azure con el proveedor de recursos de máquina virtual con SQL](sql-vm-resource-provider-register.md). |
|**Cambiar el modelo de licencia** | Ahora puede cambiar entre los modelos de "pago por uso" y "traiga su propia licencia" para la máquina virtual con SQL Server mediante PowerShell o la CLI de Azure. Para más información, consulte [Modificación del modelo de licencia para una máquina virtual con SQL Server en Azure](licensing-model-azure-hybrid-benefit-ahb-change.md). | 
| &nbsp; | &nbsp; |

## <a name="additional-resources"></a>Recursos adicionales

**Máquinas virtuales Windows**:

* [Introducción a SQL Server en máquinas virtuales Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
* [Aprovisionamiento de SQL Server en una máquina virtual Windows](create-sql-vm-portal.md)
* [Migración de una base de datos a SQL Server en una máquina virtual de Azure](migrate-to-vm-from-sql-server.md)
* [Alta disponibilidad y recuperación ante desastres para SQL Server en Azure Virtual Machines](business-continuity-high-availability-disaster-recovery-hadr-overview.md)
* [Directrices de rendimiento para SQL Server en Azure Virtual Machines](performance-guidelines-best-practices.md)
* [Estrategias de desarrollo y patrones de aplicación de SQL Server en Azure Virtual Machines](application-patterns-development-strategies.md)

**Máquinas virtuales Linux**:

* [Introducción a SQL Server en máquinas virtuales Linux](../linux/sql-server-on-linux-vm-what-is-iaas-overview.md)
* [Aprovisionamiento de una máquina virtual Linux con SQL Server en Azure Portal](../linux/sql-vm-create-portal-quickstart.md)
* [Preguntas más frecuentes (Linux)](../linux/frequently-asked-questions-faq.md)
* [Documentación de SQL Server en Linux](https://docs.microsoft.com/sql/linux/sql-server-linux-overview)

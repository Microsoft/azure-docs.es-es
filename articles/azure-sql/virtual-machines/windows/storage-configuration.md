---
title: Configuración del almacenamiento para VM con SQL Server | Microsoft Docs
description: En este tema se describe cómo Azure configura el almacenamiento para las máquinas virtuales de SQL Server durante el aprovisionamiento (modelo de implementación de Azure Resource Manager). También se explica cómo configurar el almacenamiento para sus máquinas virtuales de SQL Server existentes.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.assetid: 169fc765-3269-48fa-83f1-9fe3e4e40947
ms.service: virtual-machines-sql
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 12/26/2019
ms.author: mathoma
ms.openlocfilehash: de0402febe94e50877367dc37d448a4a13893f93
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88653365"
---
# <a name="storage-configuration-for-sql-server-vms"></a>Configuración del almacenamiento para máquinas virtuales de SQL Server
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Al configurar una imagen de máquina virtual de SQL Server en Azure, Azure Portal le ayuda a automatizar la configuración del almacenamiento. Esto incluye asociar el almacenamiento a la máquina virtual, hacer que el almacenamiento esté accesible para SQL Server y configurarlo para optimizarlo para sus requisitos de rendimiento específicos.

Este tema explica cómo Azure configura el almacenamiento para sus máquinas virtuales de SQL Server durante el aprovisionamiento y para las máquinas virtuales existentes. Esta configuración se basa en los [procedimientos recomendados de rendimiento](performance-guidelines-best-practices.md) para máquinas virtuales de Azure en las que se ejecuta SQL Server.

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-rm-include.md)]

## <a name="prerequisites"></a>Requisitos previos

Para usar la configuración del almacenamiento automática, la máquina virtual requiere las siguientes características:

* Aprovisionada con una [imagen de la galería de SQL Server](sql-server-on-azure-vm-iaas-what-is-overview.md#payasyougo).
* Usa el [modelo de implementación de Resource Manager](../../../azure-resource-manager/management/deployment-models.md).
* Usa [discos SSD Premium](../../../virtual-machines/disks-types.md).

## <a name="new-vms"></a>Nuevas máquinas virtuales

En las secciones siguientes se describe cómo configurar el almacenamiento para nuevas máquinas virtuales de SQL Server.

### <a name="azure-portal"></a>Azure portal

Si aprovisiona una máquina virtual de Azure mediante una imagen de la galería de SQL Server, seleccione **Cambiar configuración** en la pestaña **Configuración de SQL Server** para abrir la página Performance Optimized Storage Configuration (Configuración de almacenamiento optimizada para rendimiento). Puede dejar los valores predeterminados o modificar el tipo de configuración de disco, con el fin de disfrutar la que mejor se adapte a sus necesidades en función de la carga de trabajo. 

![Configuración del almacenamiento de máquinas virtuales de SQL Server durante el aprovisionamiento](./media/storage-configuration/sql-vm-storage-configuration-provisioning.png)

Seleccione el tipo de carga de trabajo para el que va a implementar SQL Server en **Optimización de almacenamiento**. Con la opción de optimización **General**, de forma predeterminada tendrá un disco de datos con 5000 IOPS como máximo, y usará esta misma unidad para los datos, el registro de transacciones y el almacenamiento de TempDB. Si se seleccionan **Procesamiento de transacciones** (OLTP) o **Almacenamiento de datos** , se creará un disco independiente para los datos, un disco independiente para el registro de transacciones y se usará un disco SSD local para TempDB. No hay ninguna diferencia a nivel de almacenamiento entre **Procesamiento de transacciones** y **Almacenamiento de datos**, pero cambia la [configuración de las bandas y las marcas de seguimiento](#workload-optimization-settings). Si se elige el almacenamiento prémium, el almacenamiento en caché se establece en *ReadOnly* (Solo lectura) para la unidad de datos y en *None* (Ninguno) para la unidad de registro según los [procedimientos recomendados de rendimiento de la máquina virtual de SQL Server](performance-guidelines-best-practices.md). 

![Configuración del almacenamiento de máquinas virtuales de SQL Server durante el aprovisionamiento](./media/storage-configuration/sql-vm-storage-configuration.png)

La configuración del disco es totalmente personalizable, es decir, se pueden configurar la topología de almacenamiento, el tipo de disco y el IOPS necesarios para la carga de trabajo de SQL Server. También tiene la capacidad de usar UltraSSD (versión preliminar) como opción en **Tipo de disco** si la máquina virtual con SQL Server se encuentra en una de las regiones admitidas (Este de EE. UU. 2, Sudeste de Asia y Norte de Europa) y ha habilitado [discos Ultra para su suscripción](/azure/virtual-machines/windows/disks-enable-ultra-ssd).  

Además, tiene la capacidad de establecer el almacenamiento en caché de los discos. Las máquinas virtuales de Azure tienen una tecnología de almacenamiento en caché multinivel llamada [Blob Cache](/azure/virtual-machines/windows/premium-storage-performance#disk-caching) (Caché de blob) cuando se usa con [discos Prémium](/azure/virtual-machines/windows/disks-types#premium-ssd). Blob Cache usa una combinación de la RAM de la máquina virtual y el disco SSD local para almacenar en caché. 

El almacenamiento en caché de disco para SSD Premium puede ser *ReadOnly* (Solo lectura), *ReadWrite* (Lectura y escritura) o *None* (Ninguno). 

- El almacenamiento en caché de solo lectura *ReadOnly* es muy beneficioso para los archivos de datos de SQL Server almacenados en Premium Storage. El almacenamiento en caché *ReadOnly* ofrece una latencia de lectura baja y una IOPS de lectura y un rendimiento altos, ya que las lecturas se realizan desde la caché, que está dentro de la memoria de la VM y del disco SSD local. Estas lecturas son mucho más rápidas que las que se realizan del disco de datos, que proceden de Azure Blob Storage. Premium Storage no cuenta las lecturas que se atienden desde la caché para la IOPS y el rendimiento del disco. Por lo tanto, la aplicación es capaz de lograr una IOPS y un rendimiento totales mayores. 
- La configuración de la caché *None* (Ninguno) se debe usar para los discos que hospedan el archivo de registro de SQL Server, ya que el archivo de registro se escribe secuencialmente y no se beneficia del almacenamiento en caché de solo lectura *ReadOnly*. 
- El almacenamiento en caché de lectura y escritura *ReadWrite* no debe usarse para hospedar archivos de SQL Server, ya que SQL Server no admite la coherencia de los datos con la memoria caché de lectura y escritura *ReadWrite*. Escribe la capacidad para residuos de la memoria caché de blobs de solo lectura *ReadOnly* y las latencias aumentan ligeramente si las escrituras atraviesan las capas de la caché de blobs de solo lectura *ReadOnly*. 


   > [!TIP]
   > Asegúrese de que la configuración de almacenamiento coincide con las limitaciones impuestas por el tamaño de máquina virtual seleccionado. Si se eligen parámetros de almacenamiento que superen el límite de rendimiento del tamaño de la máquina virtual, se producirá un error: `The desired performance might not be reached due to the maximum virtual machine disk performance cap.`. Reduzca el límite de IOPS, para lo que debe cambiar el tipo de disco, o aumente el límite de rendimiento, para lo que debe aumentar el tamaño de la máquina virtual. 


En función de lo que elija, Azure realiza las siguientes tareas de configuración del almacenamiento después de crear la máquina virtual:

* Crea y asocia discos SSD Premium a la máquina virtual.
* Configura los discos de datos para que sean accesibles para SQL Server.
* Configura los discos de datos en un grupo de almacenamiento en función de los requisitos de tamaño y rendimiento (IOPS y rendimiento) especificados.
* Asocia el grupo de almacenamiento a una unidad nueva en la máquina virtual.
* Optimiza esta nueva unidad en función de su tipo de carga de trabajo (almacenamiento de datos, procesamiento de transaccional o general).

Para más información sobre cómo Azure define la configuración del almacenamiento, consulte la sección [Configuración del almacenamiento](#storage-configuration). Para ver información más detallada acerca de cómo crear una máquina virtual con SQL Server en Azure Portal, consulte el [tutorial de aprovisionamiento](../../../azure-sql/virtual-machines/windows/create-sql-vm-portal.md).

### <a name="resource-manager-templates"></a>Plantillas de Resource Manager

Si utiliza las siguientes plantillas de Resource Manager, se asocian dos discos de datos premium de forma predeterminada, sin configuración del grupo de almacenamiento. Sin embargo, puede personalizar estas plantillas para cambiar el número de discos de datos premium que se asocian a la máquina virtual.

* [Creación de máquinas virtuales con Automated Backup](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-sql-full-autobackup)
* [Creación de máquinas virtuales con la aplicación de revisión automatizada](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-sql-full-autopatching)
* [Creación de máquinas virtuales con la integración de AKV](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-sql-full-keyvault)

### <a name="quickstart-template"></a>Plantilla de inicio rápido

Puede usar la siguiente plantilla de inicio rápido para implementar una máquina virtual con SQL Server mediante la optimización de almacenamiento. 

* [Creación de una máquina virtual con optimización de almacenamiento](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sql-vm-new-storage/)
* [Creación de una máquina virtual mediante UltraSSD](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sql-vm-new-storage-ultrassd)

## <a name="existing-vms"></a>Máquinas virtuales existentes

[!INCLUDE [windows-virtual-machines-sql-use-new-management-blade](../../../../includes/windows-virtual-machines-sql-new-resource.md)]

Para las máquinas virtuales de SQL Server existentes, puede modificar algunas opciones de configuración del almacenamiento en el Portal de Azure. Abra el [recurso máquinas virtuales SQL](manage-sql-vm-portal.md#access-the-sql-virtual-machines-resource) y seleccione **Información general**. La página SQL Server Overview (Información general de SQL Server) muestra el uso del almacenamiento actual de la máquina virtual. En este gráfico se muestran todas las unidades que existen en la máquina virtual. Para cada unidad, el espacio de almacenamiento se muestra en cuatro secciones:

* SQL data
* Registro de SQL
* Otros (almacenamiento que no es de SQL)
* Disponible

Para modificar la configuración de almacenamiento, seleccione **Configurar** en **Configuración**. 

![Configuración del almacenamiento para la máquina virtual de SQL Server existente](./media/storage-configuration/sql-vm-storage-configuration-existing.png)

Puede modificar la configuración de disco de las unidades que se configuraron durante el proceso de creación de la máquina virtual con SQL Server. Al seleccionar **Extender unidad**, se abre la página de modificación de la unidad, lo que permite cambiar el tipo de disco, así como agregar discos adicionales. 

![Configuración del almacenamiento para la máquina virtual de SQL Server existente](./media/storage-configuration/sql-vm-storage-extend-drive.png)


## <a name="storage-configuration"></a>Configuración de almacenamiento

Esta sección proporciona una referencia para los cambios en la configuración del almacenamiento que Azure realiza automáticamente durante el aprovisionamiento de la máquina virtual de SQL Server o la configuración en Azure Portal.

* Azure configura un bloque de almacenamiento desde el almacenamiento seleccionado de la máquina virtual. En la sección siguiente de este tema se proporcionan los detalles de la configuración del bloque de almacenamiento.
* La configuración automática del almacenamiento siempre utiliza discos de datos P30 de [discos SSD Premium](../../../virtual-machines/disks-types.md). En consecuencia, hay una asignación 1:1 entre el número de terabytes seleccionado y el número de discos de datos asociados a la máquina virtual.

Para más información, consulte la página [Storage pricing](https://azure.microsoft.com/pricing/details/storage) (Precios de almacenamiento) en la pestaña **Almacenamiento en disco** .

### <a name="creation-of-the-storage-pool"></a>Creación del grupo de almacenamiento

Azure usa la siguiente configuración para crear el grupo de almacenamiento en máquinas virtuales de SQL Server.

| Configuración | Value |
| --- | --- |
| Stripe size (Tamaño de las franjas) |256 KB (almacenamiento de datos); 64 KB (transaccional) |
| Tamaños de disco |1 TB cada uno |
| Cache |Lectura |
| Tamaño de la asignación |Tamaño de la unidad de asignación NTFS = 64 KB |
| Recuperación | Recuperación simple (sin resistencia) |
| Número de columnas |Número de discos de datos hasta 8<sup>1</sup> |


<sup>1</sup> después de crear el grupo de almacenamiento, no puede modificar el número de columnas en el grupo de almacenamiento.


## <a name="workload-optimization-settings"></a>Configuración de optimización de la carga de trabajo

En la tabla siguiente se describen las opciones de tres tipos de carga de trabajo disponibles y sus optimizaciones correspondientes:

| Tipo de carga de trabajo | Descripción | Optimizaciones |
| --- | --- | --- |
| **General** |Configuración predeterminada que admite la mayoría de las cargas de trabajo |None |
| **Procesamiento transaccional** |Optimiza el almacenamiento para las cargas de trabajo OLTP de bases de datos tradicionales. |Marca de seguimiento 1117<br/>Marca de seguimiento 1118 |
| **Almacenamiento de datos** |Optimiza el almacenamiento para las cargas de trabajo de informes y análisis. |Marca de seguimiento 610<br/>Marca de seguimiento 1117 |

> [!NOTE]
> Solo puede especificar el tipo de carga de trabajo cuando se aprovisiona una máquina virtual de SQL Server; para ello, selecciónelo en el paso de configuración del almacenamiento.

## <a name="next-steps"></a>Pasos siguientes

Para ver otros temas sobre la ejecución de SQL Server en Azure Virtual Machines, consulte [SQL Server en Azure Virtual Machines](sql-server-on-azure-vm-iaas-what-is-overview.md).

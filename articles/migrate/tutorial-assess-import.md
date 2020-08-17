---
title: Evaluación de servidores con datos de servidor importados con Server Assessment de Azure Migrate
description: Describe cómo evaluar los servidores en el entorno local para la migración a Azure con Azure Migrate Server Assessment y datos importados.
author: rayne-wiselman
manager: carmonm
ms.service: azure-migrate
ms.topic: tutorial
ms.date: 10/23/2019
ms.author: raynew
ms.openlocfilehash: 40dd81b9eb7ff2b20abb1bf3ea4337b63fb2a831
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87447075"
---
# <a name="assess-servers-by-using-imported-data"></a>Evaluación de servidores con datos importados

En este artículo se explica cómo evaluar servidores locales con [Azure Migrate: Herramienta Server Assessment](migrate-services-overview.md#azure-migrate-server-assessment-tool), mediante la importación de metadatos de servidor en formato de valores separados por comas (CSV). Este método de evaluación no requiere la configuración del dispositivo de Azure Migrate para crear una evaluación. Es útil si:

- Desea crear una evaluación inicial rápida antes de implementar el dispositivo.
- No se puede implementar el dispositivo de Azure Migrate en la organización.
- No se pueden compartir credenciales que permitan el acceso a servidores locales.
- Las restricciones de seguridad impiden la recopilación y el envío de datos recopilados por el dispositivo a Azure. Puede controlar los datos que se comparten en un archivo importado. Además, gran parte de los datos (por ejemplo, proporcionar direcciones IP) es opcional.

## <a name="before-you-start"></a>Antes de comenzar

Tenga en cuenta estos puntos:

- Puede agregar hasta un máximo de 20 000 servidores en un único archivo CSV.
- Puede agregar hasta 20 000 servidores en un proyecto de Azure Migrate mediante CSV.
- Mediante CSV, puede cargar información de servidor en Server Assessment varias veces.
- La recopilación de información de la aplicación es útil al evaluar el entorno local para la migración. Sin embargo, Server Assessment no realiza actualmente ninguna evaluación en el nivel de aplicación ni tiene en cuenta las aplicaciones al crear una evaluación.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Crear un proyecto de Azure Migrate.
> * Rellene un archivo CSV con información del servidor.
> * Importe el archivo para agregar información del servidor en Server Assessment.
> * Cree y revise una evaluación.

> [!NOTE]
> En los tutoriales se muestra la ruta de implementación más sencilla para un escenario, de modo que pueda configurar rápidamente una prueba de concepto. En ellos se usan las opciones predeterminadas siempre que es posible y no muestran todos los valores y rutas de acceso posibles. Para instrucciones detalladas, revise las guías paso a paso.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) antes de empezar.

## <a name="set-azure-permissions-for-azure-migrate"></a>Configuración de los permisos de Azure para Azure Migrate

la cuenta de Azure necesita permisos para crear un proyecto de Azure Migrate.

1. En Azure Portal, abra la suscripción y seleccione **Control de acceso (IAM)** .
2. En **Comprobar acceso**, busque la cuenta correspondiente y selecciónela para ver los permisos.
3. Asegúrese de tener los permisos **Colaborador** o **Propietario**.
    - Si acaba de crear una cuenta de Azure gratuita, es el propietario de la suscripción.
    - Si no es el propietario, trabaje con él para asignar el rol.

## <a name="set-up-an-azure-migrate-project"></a>Configuración de un proyecto de Azure Migrate

Para configurar un proyecto nuevo de Azure Migrate, haga lo siguiente:

1. En Azure Portal, en **Todos los servicios**, busque **Azure Migrate**.
2. En **Servicios**, seleccione **Azure Migrate**.
3. En **Información general**, en **Detectar, evaluar y migrar servidores**, seleccione **Evaluar y migrar servidores**.

    ![Detección y evaluación de servidores](./media/tutorial-assess-import/assess-migrate.png)

4. En **Inicio**, seleccione **Agregar herramientas**.
5. En **Migrar proyecto**, seleccione la suscripción a Azure y cree un grupo de recursos, en caso de que no lo tenga.
6. En **Detalles del proyecto**, especifique el nombre del proyecto y la región geográfica en la que quiere crearlo. Para obtener más información:

    - Revise las zonas geográficas admitidas para nubes [públicas](migrate-support-matrix.md#supported-geographies-public-cloud) y [nubes gubernamentales](migrate-support-matrix.md#supported-geographies-azure-government).
    - Al realizar una migración se puede seleccionar cualquier región de destino.

    ![Crear un proyecto de Azure Migrate](./media/tutorial-assess-import/migrate-project.png)

7. Seleccione **Next** (Siguiente).
8. En **Seleccione una herramienta de evaluación**, seleccione **Azure Migrate: Server Assessment** > **Siguiente**.

    ![Creación de una evaluación de Azure Migrate](./media/tutorial-assess-import/assessment-tool.png)

9. En **Seleccione una herramienta de migración**, seleccione **Omitir por ahora la adición de una herramienta de migración** > **Siguiente**.
10. En **Review + add tools** (Revisar y agregar herramientas), revise la configuración y seleccione **Add tools** (Agregar herramientas).
11. Espere unos minutos para que se implemente el proyecto de Azure Migrate. Se le dirigirá a la página del proyecto. Si no ve el proyecto, puede acceder a él desde **Servidores** en el panel de Azure Migrate.

## <a name="prepare-the-csv"></a>Preparación del archivo CSV

Descargue la plantilla de CSV y agréguele información de servidor.

### <a name="download-the-template"></a>Descarga de la plantilla

1. En **Objetivos de migración** > **Servidores** > **Azure Migrate: Server Assessment**, seleccione **Detectar**.
2. En **Detectar máquinas**, seleccione **Importar mediante CSV**.
3. Seleccione **Descargar** para descargar la plantilla CSV. O bien, puede [descargarla directamente](https://go.microsoft.com/fwlink/?linkid=2109031).

    ![Descarga de la plantilla CSV](./media/tutorial-assess-import/download-template.png)

### <a name="add-server-information"></a>Incorporación de información del servidor

Recopile los datos del servidor y agréguelos al archivo CSV.

- Para recopilar los datos, puede exportarlos desde herramientas que se usan para la administración de servidores locales, como VMware vSphere o la base de datos de administración de configuración (CMDB).
- Para revisar los datos de ejemplo, descargue el [archivo de ejemplo](https://go.microsoft.com/fwlink/?linkid=2108405).

En la tabla siguiente se resumen los campos de archivo que se van a rellenar:

**Nombre del campo** | **Obligatorio** | **Detalles**
--- | --- | ---
**Nombre del servidor** | Sí | Es recomendable especificar el nombre de dominio completo (FQDN).
**Dirección IP** | No | Dirección del servidor.
**Núcleos** | Sí | Número de núcleos de procesador asignados al servidor.
**Memoria** | Sí | RAM total (en MB) asignada al servidor.
**OS name** (Nombre de SO) | Sí | Sistema operativo del servidor. <br/> La valoración reconoce los nombres del sistema operativo que coinciden con los nombres de [esta](#supported-operating-system-names) lista, o que los contienen.
**Versión del sistema operativo** | No | Versión del sistema operativo del servidor.
**Arquitectura del sistema operativo** | No | Arquitectura del sistema operativo del servidor <br/> Los valores válidos son: x64, x86, amd64, 32 bits o 64 bits
**Número de discos** | No | No es necesario si se proporcionan detalles del disco individual.
**Disk 1 size** (Tamaño de disco 1)  | No | Tamaño máximo del disco (en GB).<br/>Puede agregar detalles de más discos mediante la [adición de columnas](#add-multiple-disks) en la plantilla. Puede agregar hasta ocho discos.
**Disk 1 read ops** (Operaciones de lectura de disco 1) | No | Operaciones de lectura de disco por segundo.
**Disk 1 write ops** (Operaciones de escritura de disco 1) | No | Operaciones de escritura de discos por segundo.
**Disk 1 read throughput** (Rendimiento de lectura de disco 1) | No | Datos leídos del disco por segundo (en MB por segundo).
**Disk 1 write throughput** (Rendimiento de escritura de disco 1) | No | Datos escritos en el disco por segundo (en MB por segundo).
**CPU utilization percentage** (Porcentaje de uso de la CPU) | No | Porcentaje de CPU en uso.
**Memory utilization percentage** (Porcentaje de uso de la memoria) | No | Porcentaje de RAM en uso.
**Total disks read ops** (Total de operaciones de lectura de discos) | No | Operaciones de lectura de disco por segundo.
**Total disks write ops** (Total de operaciones de escritura de discos) | No | Operaciones de escritura de discos por segundo.
**Total disks read throughput** (Total de rendimiento de lectura de discos) | No | Datos leídos del disco (en MB por segundo).
**Total disks write throughput** (Total de rendimiento de escritura de discos) | No | Datos escritos en el disco (en MB por segundo).
**Network in throughput** (Rendimiento de entrada de red) | No | Datos recibidos por el servidor (en MB por segundo).
**Network out throughput** (Rendimiento de salida de red) | No | Datos transmitidos por el servidor (en MB por segundo).
**Firmware type** (Tipo de firmware) | No | Firmware del servidor. Los valores pueden ser "BIOS" o "UEFI".
**MAC address** (Dirección MAC)| No | Dirección MAC de servidor.


### <a name="add-operating-systems"></a>Incorporación de sistemas operativos

La evaluación reconoce nombres específicos del sistema operativo. Cualquier nombre que especifique debe coincidir con una de las cadenas de la [lista de nombres admitidos](#supported-operating-system-names).

### <a name="add-multiple-disks"></a>Agregar varios discos

La plantilla proporciona los campos predeterminados para el primer disco. Puede agregar columnas similares para un máximo de ocho discos.

Por ejemplo, para especificar todos los campos de un segundo disco, agregue estas columnas:

- Tamaño de disco 2
- Operaciones de lectura de disco 2
- Operaciones de escritura de disco 2
- Rendimiento de lectura de disco 2
- Rendimiento de escritura de disco 2


## <a name="import-the-server-information"></a>Importación de la información de servidor

Después de agregar información a la plantilla de CSV, importe los servidores en Server Assessment.

1. En Azure Migrate, en **Detectar máquinas**, vaya a la plantilla rellena.
2. Seleccione **Import** (Importar).
3. Se muestra el estado de la importación.
    - Si aparecen advertencias en el estado, puede corregirlas o continuar.
    - Para mejorar la precisión de la evaluación, mejore la información del servidor como se sugiere en las advertencias.
    - Para ver y corregir advertencias, seleccione **Download warning details .CSV** (Descargar .CSV de detalles de advertencias). Esta operación descarga el archivo CSV con las advertencias. Revíselas y corrija los problemas según sea necesario.
    - Si aparecen errores de estado (que el estado de la importación sea **Failed**), debe corregirlos para poder continuar con la importación:
        1. Descargue el archivo CSV, que ahora incluye los detalles del error.
        1. Revise y solucione los errores según sea necesario. 
        1. Vuelva a cargar el archivo modificado.
4. Si el estado de la importación es **Completed**, esto significa que la información del servidor se ha importado.

## <a name="update-server-information"></a>Actualización de la información del servidor

Para actualizar la información de un servidor, vuelva a importar los datos para el servidor con el mismo valor para **Nombre de servidor**. El campo **Nombre de servidor** no se puede modificar. La eliminación de servidores no se admite actualmente.

## <a name="verify-servers-in-the-portal"></a>Comprobación de los servidores en el portal

Para comprobar que los servidores aparecen en Azure Portal tras la detección:

1. Abra el panel de Azure Migrate.
2. En la página **Azure Migrate - Servidores** > **Azure Migrate: Server Assessment**, seleccione el icono que muestra el número de **Servidores detectados**.
3. Seleccione la pestaña **Import based** (Basado en la importación).

## <a name="set-up-and-run-an-assessment"></a>Configuración y ejecución de una evaluación

Se pueden crear dos tipos de evaluaciones mediante Server Assessment.


**Tipo de evaluación** | **Detalles**
--- | --- 
**MV de Azure** | Evaluaciones para la migración de los servidores locales a máquinas virtuales de Azure. <br/><br/> Puede evaluar las [máquinas virtuales de VMware](how-to-set-up-appliance-vmware.md), las [máquinas virtuales de Hyper-V](how-to-set-up-appliance-hyper-v.md) y los [servidores físicos](how-to-set-up-appliance-physical.md) locales para la migración a Azure con este tipo de evaluación.
**Azure VMware Solution (AVS)** | Evaluaciones para la migración de los servidores locales a [Azure VMware Solution (AVS)](../azure-vmware/introduction.md). <br/><br/> Puede evaluar las [máquinas virtuales VMware](how-to-set-up-appliance-vmware.md) locales para la migración a Azure VMware Solution (AVS) con este tipo de evaluación. [Más información](concepts-azure-vmware-solution-assessment-calculation.md)

### <a name="sizing-criteria"></a>Criterios de ajuste de tamaño

Server Assessment proporciona dos opciones de criterios de ajuste de tamaño:

**Criterio de tamaño** | **Detalles** | **Data**
--- | --- | ---
**Basada en el rendimiento** | Evaluaciones que realizan recomendaciones basadas en datos de rendimiento recopilados | **Evaluación de máquinas virtuales de Azure**: La recomendación del tamaño de VM se basa en los datos de uso de la CPU y la memoria.<br/><br/> La recomendación de tipo de disco (SSD/disco duro estándar o discos administrados prémium) se basa en el IOPS y el rendimiento de los discos locales.<br/><br/> **Evaluación de Azure VMware Solution (AVS)** : La recomendación de los nodos de AVS se basa en los datos de uso de la CPU y la memoria.
**Tal cual en el entorno local** | Evaluaciones que no usan datos de rendimiento para hacer recomendaciones. | **Evaluación de máquinas virtuales de Azure**: La recomendación de tamaño de la máquina virtual se basa en el tamaño de la máquina virtual local.<br/><br> El tipo de disco recomendado se basa en lo que selecciona en la configuración de tipo de almacenamiento para la evaluación.<br/><br/> **Evaluación de Azure VMware Solution (AVS)** : La recomendación de los nodos de AVS se basa en el tamaño de la máquina virtual local.


Para ejecutar una evaluación:

1. Consulte los [procedimientos recomendados](best-practices-assessment.md) para crear evaluaciones.
2. En la pestaña **Servidores**, en el icono **Azure Migrate: Server Assessment**, seleccione **Evaluar**.

    ![Evaluar](./media/tutorial-assess-physical/assess.png)

3. En **Evaluar los servidores**, especifique el nombre de la evaluación y seleccione el tipo de **evaluación** como *Máquina virtual de Azure* si tiene previsto realizar evaluaciones de máquinas virtuales de Azure o bien *Azure VMware Solution (AVS)* si tiene previsto realizar evaluaciones de AVS.

    ![Aspectos básicos de la evaluación](./media/how-to-create-assessment/assess-servers-azurevm.png)

4. En **Discovery source** (Origen de detección), seleccione **Machines added via import to Azure Migrate** (Máquinas agregadas a través de la importación a Azure Migrate).

5. Seleccione **View all** (Ver todo) para revisar la configuración de la evaluación.

    ![Propiedades de la evaluación](./media/tutorial-assess-physical/view-all.png)

6. Haga clic en **Siguiente** para **seleccionar las máquinas que se van a evaluar**. En **Seleccionar o crear un grupo**, seleccione **Crear nuevo** y especifique un nombre de grupo. Un grupo recopila una o varias máquinas virtuales para su evaluación.
7. En **Agregar máquinas al grupo**, seleccione los servidores que se van a agregar al grupo.
8. Haga clic en **Siguiente** para **Revisar y crear evaluación** para revisar los detalles de la evaluación.
9. Seleccione **Crear evaluación** para crear el grupo y, a continuación, ejecutar la evaluación.

    ![Crear una evaluación](./media/tutorial-assess-physical/assessment-create.png)

9. Una vez creada la evaluación, se puede ver en **Servidores** > **Azure Migrate: Server Assessment** > **Evaluaciones**.
10. Seleccione **Exportar la evaluación** para descargarla como archivo de Microsoft Excel.

Para más información sobre la evaluación de **Azure VMware Solution (AVS)** , consulte [aquí](how-to-create-azure-vmware-solution-assessment.md). 

## <a name="review-an-azure-vm-assessment"></a>Revisión de una evaluación de máquinas virtuales de Azure

Una evaluación de máquinas virtuales de Azure describe:

- **Preparación para Azure**: si los servidores son adecuados para la migración a Azure.
- **Estimación del costo mensual**: costos mensuales estimados de proceso y almacenamiento por ejecutar los servidores en Azure.
- **Estimación del costo mensual de almacenamiento**: costos estimados del almacenamiento en disco después de la migración.

### <a name="view-an-assessment"></a>Visualización de una evaluación

1. En **Objetivos de migración** > **Servidores**, seleccione **Evaluaciones** en **Azure Migrate: Server Assessment**.
2. En **Evaluaciones**, seleccione una evaluación para abrirla.

    ![Resumen de evaluaciones](./media/tutorial-assess-physical/assessment-summary.png)

### <a name="review-azure-readiness"></a>Revisión de la preparación para Azure

1. En **Preparación para Azure**, compruebe si los servidores están listos para la migración a Azure.
2. Revise el estado:
    - **Preparada para Azure**: Azure Migrate recomienda unas estimaciones de tamaño y costo de las máquinas virtuales en la evaluación.
    - **Preparado con condiciones**: muestra los problemas y las soluciones que se sugieren.
    - **No está preparado para Azure**: muestra los problemas y las soluciones que se sugieren.
    - **Preparación desconocida**: Azure Migrate no puede evaluar la preparación por problemas de disponibilidad de los datos.

3. Seleccione uno de los estados de **Preparación para Azure**. Puede ver los detalles de la preparación del servidor y explorar en profundidad sus detalles, entre los que se incluye la configuración del proceso, el almacenamiento y la red.

### <a name="review-cost-details"></a>Revisión de los datos de costo

Esta vista muestra el costo estimado de almacenamiento y proceso que conlleva ejecutar máquinas virtuales en Azure. Puede:

- Revise los costos mensuales de proceso y almacenamiento. Los costos se agregan para todos los servidores del grupo evaluado.

    - Las estimaciones de costo se basan en las recomendaciones de tamaño de una máquina, así como en sus discos y propiedades.
    - Se muestran los costos mensuales estimados de proceso y almacenamiento.
    - La estimación de los costos es por ejecutar los servidores locales como máquinas virtuales de infraestructura como servicio (IaaS). Server Assessment no tiene en cuenta los costos de las plataformas como servicio (PaaS) o el software como servicio (SaaS).

- Revise las estimaciones del costo de almacenamiento mensuales. Esta vista muestra los costos de almacenamiento agregados del grupo evaluado, divididos por los diferentes tipos de discos de almacenamiento.
- Explore en profundidad para ver los detalles de cada una de las máquinas virtuales.

> [!NOTE]
> No se asignan clasificaciones de confianza a las evaluaciones de servidor importadas en Server Assessment mediante archivos CSV.

## <a name="supported-operating-system-names"></a>Nombres de los sistemas operativos compatibles

Los nombres de sistema operativo proporcionados en el CSV deben contener los nombres de esta lista o coincidir con ellos. Esto es necesario para que los nombres especificados se reconozcan como válidos en la valoración.

<!-- BEGIN A - H -->

:::row:::
   :::column span="2":::
      **A - H**
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      Apple Mac OS X 10
   :::column-end:::
   :::column span="":::
      Asianux 3<br/>
      Asianux 4<br/>
      Asianux 5
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      CentOS<br/>
      CentOS 4/5
   :::column-end:::
   :::column span="":::
      CoreOS Linux
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      Debian GNU/Linux 4<br/>
      Debian GNU/Linux 5<br/>
      Debian GNU/Linux 6<br/>
      Debian GNU/Linux 7<br/>
      Debian GNU/Linux 8
   :::column-end:::
   :::column span="":::
      FreeBSD
   :::column-end:::
:::row-end:::

<!-- BEGIN I - R -->

:::row:::
   :::column span="2":::
      **I - R**
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      IBM OS/2
   :::column-end:::
   :::column span="":::
      MS-DOS
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      Novell NetWare 5<br/>
      Novell NetWare 6
   :::column-end:::
   :::column span="":::
      Oracle Linux<br/>
       Oracle Linux 4/5<br/>
      Oracle Solaris 10<br/>
       Oracle Solaris 11
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      Red Hat Enterprise Linux 2<br/>
      Red Hat Enterprise Linux 3<br/>
      Red Hat Enterprise Linux 4<br/>
      Red Hat Enterprise Linux 5<br/>
      Red Hat Enterprise Linux 6<br/>
      Red Hat Enterprise Linux 7<br/>
      Red Hat Fedora
   :::column-end:::
:::row-end:::

<!-- BEGIN S - T -->

:::row:::
   :::column span="2":::
      **S - T**
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      SCO OpenServer 5<br/>
      SCO OpenServer 6<br/>
      SCO UnixWare 7
   :::column-end:::
   :::column span="":::
      Serenity Systems eComStation 1<br/>
      Serenity Systems eComStation 2
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      Sun Microsystems Solaris 8<br/>
      Sun Microsystems Solaris 9
   :::column-end:::
   :::column span="":::
      SUSE Linux Enterprise 10<br/>
      SUSE Linux Enterprise 11<br/>
      SUSE Linux Enterprise 12<br/>
      SUSE Linux Enterprise 8/9<br/>
      SUSE Linux Enterprise 11<br/>
      SUSE openSUSE
   :::column-end:::
:::row-end:::

<!-- BEGIN U - Z -->
:::row:::
   :::column span="2":::
      **U - Z**
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      Ubuntu Linux
   :::column-end:::
   :::column span="":::
      VMware ESXi 4<br/>
      VMware ESXi 5<br/>
      VMware ESXi 6
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="":::
      Windows 10<br/>
      Windows 2000<br/>
      Windows 3<br/>
      Windows 7<br/>
      Windows 8<br/>
      Windows 95<br/>
      Windows 98<br/>
      Windows NT<br/>
      Windows Server (R) 2008<br/>
      Windows Server 2003
   :::column-end:::
   :::column span="":::
      Windows Server 2008<br/>
      Windows Server 2008 R2<br/>
      Windows Server 2012<br/>
      Windows Server 2012 R2<br/>
      Windows Server 2016<br/>
      Windows Server 2019<br/>
      Windows Server Threshold<br/>
      Windows Vista<br/>
      Windows Web Server 2008 R2<br/>
      Windows XP Professional
   :::column-end:::
:::row-end:::

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Importó servidores en Azure Migrate: Server Assessment con CSV.
> * Creó y revisó una evaluación.

Ahora, [implemente un dispositivo](./migrate-appliance.md) para realizar evaluaciones más precisas y recopile los servidores en grupos para una evaluación más minuciosa con el [análisis de dependencias](./concepts-dependency-visualization.md).

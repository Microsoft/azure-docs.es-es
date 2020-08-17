---
title: Administración de almacenes y servidores de Azure Recovery Services
description: En este artículo se explica cómo usar el panel de información general del almacén de Recovery Services para supervisar y administrar los almacenes de Recovery Services.
ms.topic: conceptual
ms.date: 07/08/2019
ms.openlocfilehash: 1b3cc317b8a26b16bbeda9793f87d089f2fbc59f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87054904"
---
# <a name="monitor-and-manage-recovery-services-vaults"></a>Supervisión y administración de almacenes de Recovery Services

En este artículo se explica cómo usar el panel de **información general** del almacén de Recovery Services para supervisar y administrar los almacenes de Recovery Services. Al abrir un almacén de Recovery Services de la lista, se abre el panel de **información general** del mismo. El panel proporciona diversos detalles acerca del almacén. Hay *iconos* que muestran el estado de las alertas críticas y de advertencia, los trabajos de copia de seguridad en curso y con errores, y la cantidad usada de almacenamiento con redundancia local (LRS) y almacenamiento con redundancia geográfica (GRS). Si realiza copias de seguridad de máquinas virtuales de Azure en el almacén, el icono [**Estado de la comprobación previa a la copia de seguridad** muestra todos los elementos críticos o de advertencia](#backup-pre-check-status). La imagen siguiente es el panel de **información general** de **Contoso-vault**. El icono **Elementos de copia de seguridad** muestra que hay nueve elementos registrados en el almacén.

![panel del almacén de Servicios de recuperación](./media/backup-azure-manage-windows-server/rs-vault-blade.png)

Los requisitos previos de este artículo son: una suscripción de Azure, un almacén de Recovery Services y que haya al menos un elemento de copia de seguridad configurado para el almacén.

[!INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-rm-include.md)]

## <a name="open-a-recovery-services-vault"></a>Apertura de un almacén de Recovery Services

Para supervisar las alertas o ver los datos de administración de un almacén de Recovery Services, abra el almacén.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) mediante la suscripción de Azure.

2. En el portal, haga clic en **Todos los servicios**.

   ![Apertura de la lista de almacenes de Recovery Services paso 1.](./media/backup-azure-manage-windows-server/open-rs-vault-list.png)

3. En el cuadro de diálogo **Todos los servicios**, escriba **Recovery Services**. Cuando comience a escribir, la lista se filtrará en función de la entrada. Cuando aparezca la opción **Almacenes de Recovery Services**, haga clic en ella para abrir la lista de almacenes de Recovery Services de su suscripción.

    ![Creación del almacén de Recovery Services, paso 1](./media/backup-azure-manage-windows-server/list-of-rs-vaults.png) <br/>

4. En la lista de almacenes, haga clic en cualquiera de ellos para abrir su panel de **información general**.

    ![panel del almacén de Servicios de recuperación](./media/backup-azure-manage-windows-server/rs-vault-blade.png) <br/>

    El panel de información general utiliza iconos para proporcionar alertas y datos de los trabajos de copia de seguridad.

## <a name="monitor-backup-jobs-and-alerts"></a>Supervisión de trabajos y alertas de copia de seguridad

El panel de **información general** del almacén de Recovery Services proporciona iconos para la información acerca de la supervisión y el uso. Los iconos de la sección de supervisión muestran alertas críticas y de advertencia, así como trabajos en curso y con errores. Haga clic en una alerta o un trabajo concretos para abrir los menús Alertas de copia de seguridad o Trabajos de copia de seguridad, filtrado por el trabajo o la alerta concretos.

![Copia de seguridad de las tareas del panel](./media/backup-azure-manage-windows-server/monitor-dashboard-tiles-warning.png)

La sección de supervisión muestra los resultados de las consultas de **Alertas de copia de seguridad** y **Trabajos de copia de seguridad** predefinidas. Los iconos de supervisión proporcionan información actualizada acerca de:

* Alertas críticas y de advertencia para los trabajos de copia de seguridad (en las últimas 24 horas)
* Compruebe el estado de las máquinas virtuales de Azure. Para una información más completa sobre el estado de la comprobación previa, consulte [Estado de comprobación previa de copia de seguridad](#backup-pre-check-status).
* Los trabajos de copia de seguridad en curso y los trabajos con errores (en las últimas 24 horas).

Los iconos de uso proporcionan:

* El número de elementos de copia de seguridad configurados para el almacén.
* El almacenamiento de Azure (separado por LRS y GRS) que ha consumido el almacén.

Haga clic en los iconos (excepto Almacenamiento de copia de seguridad) para abrir el menú asociado. En la imagen anterior, el icono de alertas de copia de seguridad muestra tres alertas críticas. Al hacer clic en la fila de alertas críticas del icono de alertas de copia de seguridad, se abren las alertas de copia de seguridad filtradas para alertas críticas.

![Menú Alertas de copia de seguridad filtrado para alertas críticas](./media/backup-azure-manage-windows-server/critical-backup-alerts.png)

El menú Alertas de copia de seguridad, en la imagen anterior, se filtra por: estado activo, gravedad crítica y la hora es las 24 horas posteriores.

### <a name="backup-pre-check-status"></a>Estado de comprobación previa a la copia de seguridad

Las comprobaciones previas de copia de seguridad, comprueba la configuración de las máquinas virtuales en busca de problemas que puedan afectar negativamente a las copias de seguridad. Agregan esta información para que pueda verla directamente desde el panel del almacén de Recovery Services y proporcionar recomendaciones para medidas correctivas para garantizar copias de seguridad correctas coherentes con el archivo o coherentes con la aplicación. No requieren ninguna infraestructura y no tienen ningún coste adicional.  

Las comprobaciones previas de copia de seguridad se ejecutan como parte de las operaciones de copia de seguridad programadas para las máquinas virtuales de Azure. Concluyen con uno de los siguientes estados:

* **Superado**: Este estado indica que la configuración de la máquina virtual debe llevar a copias de seguridad correctas y no es necesario realizar ninguna acción correctiva.
* **Advertencia**: Este estado indica uno o más problemas en la configuración de la máquina virtual que *podrían* provocar fallas en la copia de seguridad. Proporciona pasos *recomendados* para garantizar copias de seguridad correctas. Por ejemplo, si no tiene instalado el Agente de máquina virtual más reciente, pueden producirse errores en las copias de seguridad de forma intermitente. Esta situación proporciona un estado de advertencia.
* **Crítico**: Este estado indica uno o más problemas críticos en la configuración de la máquina virtual que *provocarán* errores de copia de seguridad y proporciona los pasos *necesarios* para garantizar copias de seguridad correctas. Por ejemplo, un problema de red provocado por una actualización de las reglas de NSG de una máquina virtual hará que se produzca un error en las copias de seguridad, ya que impide que la máquina virtual se comunique con el servicio de Azure Backup. Esta situación proporcionaría un estado crítico.

Siga los pasos que se indican a continuación para comenzar a resolver los problemas de comprobación previa de copias de seguridad de máquinas virtuales en el almacén de Recovery Services.

* Seleccione el icono **Estado de la comprobación previa de la copia de seguridad (máquinas virtuales de Azure)** en el panel del almacén de Recovery Services.
* Seleccione cualquier máquina virtual con un Estado de comprobación previa de copia de seguridad de **Crítico** o **Advertencia**. Esta acción abrirá el panel de **detalles de la máquina virtual**.
* Seleccione la notificación del panel en la parte superior del panel para ver la descripción del problema de configuración y los pasos correctos.

## <a name="manage-backup-alerts"></a>Administración de alertas de copia de seguridad

Para acceder al menú Alertas de copia de seguridad, en el menú del almacén de Recovery Services, haga clic en **Alertas de copia de seguridad**.

![Alertas de copias de seguridad](./media/backup-azure-manage-windows-server/backup-alerts-menu.png)

El informe Alertas de copia de seguridad enumera las alertas del almacén.

![Alertas de copias de seguridad](./media/backup-azure-manage-windows-server/backup-alerts.png)

### <a name="alerts"></a>Alertas

La lista de alertas de copia de seguridad muestra la información seleccionada de las alertas filtradas. En el menú Alertas de copia de seguridad, puede filtrar por alertas críticas o de advertencia.

| Nivel de alerta | Eventos que generan alertas |
| ----------- | ----------- |
| Crítico | Se reciben alertas críticas cuando: se producen errores en trabajos de copia de seguridad o en trabajos de recuperación y cuando se detiene la protección de un servidor, pero se conservan los datos.|
| Advertencia | Las alertas de advertencia se reciben cuando: los trabajos de copia de seguridad se completan con advertencias, por ejemplo cuando no se realiza la copia de seguridad de menos de 100 archivos por problemas de daños, o cuando se realiza una copia de seguridad de más de 1 000 000 archivos correctamente). |
| Informativo | Actualmente, no hay alertas informativas en uso. |

### <a name="viewing-alert-details"></a>Visualización de los detalles de una alerta

El informe Alertas de copia de seguridad realiza un seguimiento de ocho detalles de cada alerta. Use el botón **Elegir columnas** para editar los detalles en el informe.

![Alertas de copias de seguridad](./media/backup-azure-manage-windows-server/backup-alerts.png)

De forma predeterminada, todos los detalles, excepto **Hora de la última repetición**, aparecen en el informe.

* Alerta
* Elemento de copia de seguridad
* Servidor protegido
* severity
* Duration
* Hora de creación
* Estado
* Hora de la última repetición

### <a name="change-the-details-in-alerts-report"></a>Cambio de los detalles en un informe de alertas

1. Para cambiar la información del informe, en el menú **Alertas de copia de seguridad**, haga clic en **Elegir columnas**.

   ![Alertas de copias de seguridad](./media/backup-azure-manage-windows-server/alerts-menu-choose-columns.png)

   Se abre el menú **Elegir columnas**.

2. En el menú **Elegir columnas**, elija los detalles que desea que aparezcan en el informe.

    ![Menú Elegir columnas](./media/backup-azure-manage-windows-server/choose-columns-menu.png)

3. Haga clic en **Listo** para guardar los cambios y cerrar el menú Elegir columnas.

   Si realiza cambios, pero no desea conservarlos, haga clic en **Restablecer** para devolver el texto seleccionado a la última configuración guardada.

### <a name="change-the-filter-in-alerts-report"></a>Cambio del filtro en un informe de alertas

Use el menú **Filtro** para cambiar la gravedad, el estado, la hora de inicio y la hora de finalización de las alertas.

> [!NOTE]
> La edición del filtro de alertas de copia de seguridad no cambia las alertas críticas o de advertencia en el panel de información general del almacén.
>  

1. Para cambiar el filtro de alertas de copia de seguridad, en el menú Alertas de copia de seguridad, haga clic en **Filtro**.

   ![Elegir menú Filtro](./media/backup-azure-manage-windows-server/alerts-menu-choose-filter.png)

   Aparece el menú Filtro.

   ![Elegir menú Filtro](./media/backup-azure-manage-windows-server/filter-alert-menu.png)

2. Edite la gravedad, el estado, la hora de inicio o la hora de finalización y haga clic en **Listo** para guardar los cambios.

## <a name="configuring-notifications-for-alerts"></a>Configuración de notificaciones de alertas

Configure notificaciones para que se generen mensajes de correo electrónico cuando se produzca una alerta crítica o de advertencia. Las alertas de correo electrónico se pueden enviar cada hora, o cuando se produzca una alerta determinada.

   ![Filtrar alertas](./media/backup-azure-manage-windows-server/configure-notification.png)

De manera predeterminada, la opción de notificaciones por correo electrónico es **Activado**. Haga clic en **Desactivado** para detener las notificaciones por correo electrónico.

En el control **Notificar**, elija **Por alerta** si no desea agrupar las notificaciones o no tiene muchos elementos que puedan generar alertas. Todas las alertas generan una notificación (configuración predeterminada) y se envía inmediatamente un correo electrónico de resolución.

Si selecciona **Resumen cada hora**, se envía un correo electrónico a los destinatarios en el que se explican las alertas sin resolver generadas en la última hora. Se envía un correo electrónico de resolución al final del período de una hora.

Elija la gravedad de alerta (Crítica o Advertencia) usada para generar el correo electrónico. Actualmente no hay alertas de información.

## <a name="manage-backup-items"></a>Administración de elementos de copia de seguridad

Un almacén de Recovery Services contiene muchos tipos de datos de copia de seguridad. [Más información](backup-overview.md#what-can-i-back-up) sobre de qué se puede hacer copia de seguridad. Para administrar los distintos servidores, equipos, bases de datos y cargas de trabajo, haga clic en el icono **Elementos de copia de seguridad** para ver el contenido del almacén.

![Icono Elementos de copia de seguridad](./media/backup-azure-manage-windows-server/backup-items.png)

Se abre la lista de elementos de copia de seguridad, organizados por tipo de administración de copia de seguridad.

![Lista de elementos de copia de seguridad](./media/backup-azure-manage-windows-server/list-backup-items.png)

Para explorar un tipo concreto de instancia protegida, haga clic en el elemento en la columna Tipo de administración de copias de seguridad. Por ejemplo, en la imagen anterior, hay dos máquinas virtuales de Azure protegidas en este almacén. Al hacer clic en **Máquina virtual de Azure**, se abre la lista de máquinas virtuales protegidas de este almacén.

![Lista de tipo de copia de seguridad](./media/backup-azure-manage-windows-server/list-of-protected-virtual-machines.png)

La lista de máquinas virtuales tiene datos útiles: el grupo de recursos asociados, [Comprobación previa a la copia de seguridad](#backup-pre-check-status) anterior, Estado de la última copia de seguridad y la fecha del punto de restauración más reciente. El botón de puntos suspensivos, en la última columna, abre el menú para desencadenar tareas comunes. Los datos útiles que se proporcionan en las columnas son diferentes para cada tipo de copia de seguridad.

![Lista de tipo de copia de seguridad](./media/backup-azure-manage-windows-server/ellipsis-menu.png)

## <a name="manage-backup-jobs"></a>Administración de trabajos de copia de seguridad

El icono **Trabajos de copia de seguridad** del panel del almacén muestra el número de trabajos que están en curso o con errores en las últimas 24 horas. El icono permite acceder al menú Trabajos de copia de seguridad.

![Elementos de copia de seguridad en la configuración](./media/backup-azure-manage-windows-server/backup-jobs-tile.png)

Para ver más detalles acerca de los trabajos, haga clic en **En curso** o en **Con errores** para abrir el menú Trabajos de copia de seguridad filtrado de ese estado.

### <a name="backup-jobs-menu"></a>Menú Trabajos de copia de seguridad

El menú **Trabajos de copia de seguridad** muestra información acerca del tipo de elemento, la operación, el estado, la hora de inicio y la duración.  

Para abrir el menú Trabajos de copia de seguridad, en el menú principal del almacén, haga clic en **Trabajos de copia de seguridad**.

![Elementos de copia de seguridad en la configuración](./media/backup-azure-manage-windows-server/backup-jobs-menu-item.png)

Se abre la lista de trabajos de copia de seguridad.

![Elementos de copia de seguridad en la configuración](./media/backup-azure-manage-windows-server/backup-jobs-list.png)

El menú Trabajos de copia de seguridad muestra el estado de todas las operaciones, en todos los tipos de copia de seguridad, de las últimas 24 horas. Use **Filtro** para cambiar los filtros. Los filtros se explican en las siguientes secciones.

Para cambiar los filtros:

1. En el menú Trabajos de copia de seguridad del almacén, haga clic en **Filtro**.

   ![Elementos de copia de seguridad en la configuración](./media/backup-azure-manage-windows-server/vault-backup-job-menu-filter.png)

    Se abre el menú Filtro.

   ![Elementos de copia de seguridad en la configuración](./media/backup-azure-manage-windows-server/filter-menu-backup-jobs.png)

2. Elija la configuración de filtro y haga clic en **Listo**. La lista filtrada se actualiza en función de la nueva configuración.

#### <a name="item-type"></a>Tipo de elemento

El tipo de elemento es el tipo de administración de copia de seguridad de la instancia protegida. Hay cuatro tipos; consulte la lista siguiente. Puede ver todos los tipos de elemento o un tipo de elemento. No puede seleccionar dos o tres tipos de elemento. Los tipos de elemento disponibles son:

* Todos los tipos de elemento
* Máquina virtual de Azure
* Archivos y carpetas
* Azure Storage
* Carga de trabajo de Azure

#### <a name="operation"></a>Operación

Puede ver una sola operación o todas las operaciones. No puede seleccionar dos o tres operaciones. Las operaciones disponibles son:

* Todas las operaciones
* Register
* Configuración de la copia de seguridad
* Copia de seguridad
* Restauración
* Deshabilitar copias de seguridad
* Eliminación de datos de copia de seguridad

#### <a name="status"></a>Estado

Puede ver todos los estados o solo uno. No puede seleccionar dos o tres estados. Los estados disponibles son:

* Todos los estados
* Completed
* En curso
* Con error
* Canceled
* Completado con advertencias

#### <a name="start-time"></a>Hora de inicio

El día y hora en que comienza la consulta. El valor predeterminado es un período de 24 horas.

#### <a name="end-time"></a>Hora de finalización

El día y hora en que finaliza la consulta.

### <a name="export-jobs"></a>Exportar trabajos

Use **Exportar trabajos** para crear una hoja de cálculo que contiene información del menú de todos los trabajos. Dicha hoja de cálculo tiene una hoja que contiene un resumen de todos los trabajos y las hojas individuales de cada trabajo.

Para exportar la información de los trabajos a una hoja de cálculo, haga clic en **Exportar trabajos**. El servicio crea una hoja de cálculo con el nombre del almacén y la fecha, pero puede cambiar el nombre.

## <a name="monitor-backup-usage"></a>Supervisión del uso de Copia de seguridad

El icono Almacenamiento de copia de seguridad del panel muestra el almacenamiento consumido en Azure. El uso de almacenamiento se proporciona para:

* Uso de almacenamiento LRS en la nube asociado con el almacén
* Uso de almacenamiento GRS en la nube asociado con el almacén

## <a name="troubleshooting-monitoring-issues"></a>Solución de problemas de supervisión

**Problema:** los trabajos y las alertas del agente de Azure Backup no aparecen en el portal.

**Pasos para solucionar problemas**: el proceso ```OBRecoveryServicesManagementAgent``` envía los datos de alerta y del trabajo al servicio de Azure Backup. En ocasiones este proceso puede quedar atascado o apagado.

1. Para comprobar que el proceso no se está ejecutando, abra el **Administrador de tareas** y compruebe si ```OBRecoveryServicesManagementAgent``` se está ejecutando.

2. Si el proceso no se está ejecutando, abra el **Panel de Control** y examine la lista de servicios. Inicie o reinicie el **agente de administración de Microsoft Azure Recovery Services**.

    Para más información, examine los registros en:<br/>
   Por ejemplo: `<AzureBackup_agent_install_folder>\Microsoft Azure Recovery Services Agent\Temp\GatewayProvider*`<br/>
   `C:\Program Files\Microsoft Azure Recovery Services Agent\Temp\GatewayProvider0.errlog`

## <a name="next-steps"></a>Pasos siguientes

* [Restauración de Windows Server o el cliente de Windows desde Azure](backup-azure-restore-windows-server.md)
* Para obtener más información sobre Azure Backup, consulte [Información general de Azure Backup](./backup-overview.md)

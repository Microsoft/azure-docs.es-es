---
title: 'Tutorial: Copia de seguridad de bases de datos de SQL Server en Azure'
description: En este tutorial, aprenderá a hacer una copia de seguridad de una base de datos de SQL Server que se ejecuta en una VM de Azure en un almacén de Azure Backup Recovery Services.
ms.topic: tutorial
ms.date: 06/18/2019
ms.openlocfilehash: 17a8472da2595c08cb198baaf853faf110a619fa
ms.sourcegitcommit: cd0a1ae644b95dbd3aac4be295eb4ef811be9aaa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88612459"
---
# <a name="back-up-a-sql-server-database-in-an-azure-vm"></a>Copia de seguridad de una base de datos de SQL Server en una máquina virtual de Azure

En este tutorial se muestra cómo hacer una copia de seguridad de una base de datos de SQL Server que se ejecuta en una máquina virtual de Azure en un almacén de Azure Backup Recovery Services. En este artículo aprenderá a:

> [!div class="checklist"]
>
> * Crear y configurar un almacén.
> * Detectar bases de datos y configurar copias de seguridad.
> * Configurar la protección automática de las bases de datos.
> * Ejecutar una copia de seguridad a petición.

## <a name="prerequisites"></a>Prerrequisitos

Para poder realizar copias de seguridad de la base de datos de SQL Server, primero debe comprobar si reúne las siguientes condiciones:

1. Identifique o [cree](backup-sql-server-database-azure-vms.md#create-a-recovery-services-vault) un almacén de Recovery Services en la misma región o configuración regional que la máquina virtual que hospeda la instancia de SQL Server.
2. [Compruebe los permisos de máquina virtual](backup-azure-sql-database.md#set-vm-permissions) necesarios para realizar una copia de seguridad de las bases de datos SQL.
3. Compruebe que la máquina virtual tenga [conectividad de red](backup-sql-server-database-azure-vms.md#establish-network-connectivity).
4. Compruebe que las bases de datos de SQL Server se denominan de acuerdo con las [directrices de nomenclatura](#verify-database-naming-guidelines-for-azure-backup) de Azure Backup.
5. Compruebe que no dispone de otras soluciones de copia de seguridad habilitadas para la base de datos. Deshabilite todas las demás copias de seguridad de SQL Server antes de configurar este escenario. Puede habilitar Azure Backup para una máquina virtual de Azure junto con Azure Backup para una base de datos de SQL Server que se ejecute en la máquina virtual sin que se produzcan conflictos.

### <a name="establish-network-connectivity"></a>Establecimiento de conectividad de red

Todas las operaciones realizadas en la máquina virtual con SQL Server necesitan conectividad a las direcciones IP públicas de Azure. Si falta dicha conectividad, operaciones como detectar bases de datos, configurar y programar copias de seguridad o restaurar puntos de recuperación producirán errores. Establezca la conectividad con una de estas opciones:

* **Allow the Azure datacenter IP ranges** (Permitir los intervalos de direcciones IP del centro de datos de Azure): permite los [intervalos IP](https://www.microsoft.com/download/details.aspx?id=41653) en la descarga. Para acceder a un grupo de seguridad de red (NSG), use el cmdlet **Set-AzureNetworkSecurityRule**.
* **Deploy an HTTP proxy server to route traffic** (Implementar un servidor proxy HTTP para enrutar el tráfico): cuando hace copia de seguridad de una base de datos de SQL Server en una máquina virtual de Azure, la extensión de copia de seguridad en la máquina virtual usa las API HTTPS para enviar comandos de administración a Azure Backup y datos a Azure Storage. La extensión de copia de seguridad también usa Azure Active Directory (Azure AD) para la autenticación. Enrute el tráfico de extensión de copia de seguridad de estos tres servicios a través del proxy HTTP. Las extensiones son el único componente configurado para el acceso a la red pública de Internet.

Cada opción tiene sus ventajas y desventajas.

**Opción** | **Ventajas** | **Desventajas**
--- | --- | ---
Permitir intervalos IP | Sin costos adicionales. | Este escenario es complejo de administrar, ya que los intervalos de direcciones IP cambian con el tiempo. <br/><br/> Proporciona acceso a la totalidad de Azure, no solo a Azure Storage.
Usar un servidor proxy HTTP   | Se permite un control detallado en el proxy sobre las direcciones URL de almacenamiento. <br/><br/> Un único punto de acceso a Internet para las máquinas virtuales. <br/><br/> No están sujetas a cambios de direcciones IP de Azure. | Costos adicionales de ejecutar una máquina virtual con el software de proxy.

### <a name="set-vm-permissions"></a>Establecer permisos de máquina virtual

Azure Backup lleva a cabo una serie de acciones al configurar la copia de seguridad para una base de datos de SQL Server:

* Agrega la extensión **AzureBackupWindowsWorkload**.
* Para detectar las bases de datos en la máquina virtual, Azure Backup crea la cuenta **NT SERVICE\AzureWLBackupPluginSvc**. Esta cuenta se usa para la copia de seguridad y restauración, y es necesario tener permisos de administrador del sistema.
* Azure Backup aprovecha la cuenta **NT AUTHORITY\SYSTEM** para la detección o consulta de bases de datos, por lo que esta cuenta debe tener un inicio de sesión público en SQL.

Si no se creó la máquina virtual con SQL Server en Azure Marketplace, podría recibir el error **UserErrorSQLNoSysadminMembership**. Si esto sucede, [siga estas instrucciones](backup-azure-sql-database.md#set-vm-permissions).

### <a name="verify-database-naming-guidelines-for-azure-backup"></a>Compruebe las directrices de nomenclatura de las bases de datos de Azure Backup.

Evite lo siguiente en los nombres de bases de datos:

* Espacios iniciales o finales
* Signos '!' finales
* Corchete de cierre ‘]’
* Nombres de bases de datos que empiezan por "F:\"

Aunque se dispone de alias para los caracteres no admitidos de la tabla Azure, se recomienda evitarlos. [Más información](/rest/api/storageservices/understanding-the-table-service-data-model).

[!INCLUDE [How to create a Recovery Services vault](../../includes/backup-create-rs-vault.md)]

## <a name="discover-sql-server-databases"></a>Detección de bases de datos SQL Server

Detecte las bases de datos que se ejecutan en la máquina virtual.

1. En [Azure Portal](https://portal.azure.com), abra el almacén de Recovery Services que usa para la copia de seguridad de la base de datos.

2. En el panel del **almacén de Recovery Services**, seleccione **Copia de seguridad**.

   ![Seleccionar Copia de seguridad para abrir el menú Objetivo de Backup](./media/backup-azure-sql-database/open-backup-menu.png)

3. En el menú **Backup Goal** (Objetivo de Backup), establezca **Where is your workload running** (¿Dónde se ejecuta su carga de trabajo?) en **Azure** (el valor predeterminado).

4. En **What do you want to backup** (¿De qué quiere realizar una copia de seguridad?), seleccione **SQL Server in Azure VM** (SQL Server en la máquina virtual de Azure).

    ![Seleccionar SQL Server en una máquina virtual de Azure para la copia de seguridad](./media/backup-azure-sql-database/choose-sql-database-backup-goal.png)

5. En **Backup Goal** > **Discover DBs in VMs** (Objetivo de Backup > Detectar bases de datos en máquinas virtuales), seleccione **Start Discovery** (Iniciar detección) para buscar las máquinas virtuales sin protección en la suscripción. Puede que la operación tarde un rato, según el número de máquinas virtuales sin protección en la suscripción.

   * Tras la detección, las máquinas virtuales no protegidas deben aparecer en la lista por nombre y grupo de recursos.
   * Si una máquina virtual no aparece en la lista como cabría esperar, compruebe que no se haya copiado ya en un almacén.
   * Varias máquinas virtuales pueden tener el mismo nombre pero pertenecer a distintos grupos de recursos.

     ![La copia de seguridad queda pendiente durante la búsqueda de bases de datos en máquinas virtuales](./media/backup-azure-sql-database/discovering-sql-databases.png)

6. En la lista de máquinas virtuales, seleccione la que ejecuta la base de datos de SQL Server > **Discover DBs** (Detectar bases de datos).

7. Realice el seguimiento de la detección de las bases de datos en el área **Notificaciones**. El trabajo puede tardar un rato en completarse, según el número de bases de datos que haya en la máquina virtual. Cuando se detectan las bases de datos seleccionadas, aparece un mensaje de operación correcta.

    ![Mensaje de implementación correcta](./media/backup-azure-sql-database/notifications-db-discovered.png)

8. Azure Backup detecta todas las bases de datos de SQL Server en la máquina virtual. Durante la detección, tiene lugar lo siguiente en segundo plano:

    * Azure Backup registra la máquina virtual en el almacén para la copia de seguridad de la carga de trabajo. Todas las bases de datos de la máquina virtual registrada solo se pueden copiar en este almacén.
    * Azure Backup instala la extensión **AzureBackupWindowsWorkload** en la máquina virtual. No se instala ningún agente en la base de datos SQL.
    * Azure Backup crea la cuenta de servicio **NT Service\AzureWLBackupPluginSvc** en la máquina virtual.
      * Todas las operaciones de copia de seguridad y restauración utilizan la cuenta de servicio.
      * **NT Service\AzureWLBackupPluginSvc** necesita permisos de administrador del sistema de SQL. Todas las máquinas virtuales de SQL Server creadas en Azure Marketplace llevan instalado **SqlIaaSExtension**. La extensión **AzureBackupWindowsWorkload** usa **SQLIaaSExtension** para obtener automáticamente los permisos necesarios.
    * Si no ha creado la máquina virtual desde Marketplace, no tendrá instalado **SqlIaaSExtension** y la operación de detección no se podrá realizar; se muestra el mensaje de error **UserErrorSQLNoSysAdminMembership**. Siga las [instrucciones](backup-azure-sql-database.md#set-vm-permissions) para solucionar el problema.

        ![Seleccionar la máquina virtual y la base de datos](./media/backup-azure-sql-database/registration-errors.png)

## <a name="configure-backup"></a>Configuración de la copia de seguridad  

Configure la copia de seguridad de la manera siguiente:

1. En **Objetivo de Backup**, seleccione **Configurar copia de seguridad**.

   ![Seleccionar Configurar copia de seguridad](./media/backup-azure-sql-database/backup-goal-configure-backup.png)

2. Seleccione **Configurar copia de seguridad**, en el panel **Seleccionar elementos de los que realizar copia de seguridad** que aparece. Esto hace que se enumeren todos los grupos de disponibilidad registrados y los servidores de SQL Server independientes. Expanda el botón de contenido adicional situado a la izquierda de la fila para ver todas las bases de datos desprotegidas de esa instancia o los grupos de disponibilidad de AlwaysOn.  

    ![Mostrar todas las instancias de SQL Server con bases de datos independientes](./media/backup-azure-sql-database/list-of-sql-databases.png)

3. Seleccione todas las bases de datos que quiera proteger > **OK** (Aceptar).

   ![Protección de la base de datos](./media/backup-azure-sql-database/select-database-to-protect.png)

   Para optimizar las cargas de copia de seguridad, el número máximo de bases de datos en un trabajo de copia de seguridad en Azure Backup se establece en 50.

     * Como alternativa, puede habilitar la protección automática en la instancia completa o el grupo de disponibilidad AlwaysOn si selecciona la opción **ON** (Activado) en la lista desplegable correspondiente de la columna **AUTOPROTECT** (Protección automática). La característica de protección automática no solo habilita la protección en todas las bases de datos existentes en una sola operación, sino que también protege automáticamente las nuevas bases de datos que se agregarán a esa instancia o al grupo de disponibilidad en el futuro.  

4. Seleccione **Aceptar** para abrir el panel **Directiva de copia de seguridad**.

    ![Habilitar la protección automática en el grupo de disponibilidad Always On](./media/backup-azure-sql-database/enable-auto-protection.png)

5. En **Elegir directiva de copia de seguridad**, seleccione una directiva y haga clic en **Aceptar**.

   * Selección de la directiva predeterminada: HourlyLogBackup.
   * Elegir una directiva de copia de seguridad existente creada previamente para SQL.
   * Defina una nueva directiva basada en el objetivo de punto de recuperación (RPO) y en la duración de retención.

     ![Seleccionar directiva de copia de seguridad](./media/backup-azure-sql-database/select-backup-policy.png)

6. En el menú **Copia de seguridad**, seleccione **Habilitar copia de seguridad**.

    ![Habilitar la directiva de copia de seguridad elegida](./media/backup-azure-sql-database/enable-backup-button.png)

7. Realice el seguimiento del progreso de la configuración en el área de **notificaciones** del portal.

    ![Área de notificaciones](./media/backup-azure-sql-database/notifications-area.png)

### <a name="create-a-backup-policy"></a>Crear una directiva de copia de seguridad

Una directiva de copia de seguridad define cuándo se realizan las copias de seguridad y cuánto tiempo se conservan.

* Una directiva se crea en el nivel de almacén.
* Varios almacenes pueden usar la misma directiva de copia de seguridad, pero debe aplicar la directiva de copia de seguridad a cada almacén.
* Al crear una directiva de copia de seguridad, la copia de seguridad completa diaria es la predeterminada.
* Puede agregar una copia de seguridad diferencial, pero solo si configura las copias de seguridad completas para que se realicen semanalmente.
* [Más información](backup-architecture.md#sql-server-backup-types) sobre los diferentes tipos de directivas de copia de seguridad.

Para crear una directiva de copia de seguridad:

1. En el almacén, seleccione **Directivas de copia de seguridad** > **Agregar**.
2. En el menú **Agregar**, seleccione **SQL Server en una máquina virtual de Azure** para definir el tipo de directiva.

   ![Elección de un tipo de directiva para la nueva directiva de copia de seguridad](./media/backup-azure-sql-database/policy-type-details.png)

3. En **Nombre de la directiva**, escriba un nombre para la nueva directiva.
4. En el menú **Full Backup policy** (Directiva de copia de seguridad completa), seleccione un valor para **Backup Frequency** (Frecuencia de copia de seguridad) entre **Daily** (Diaria) o **Weekly** (Semanal).

   * En **Diariamente**, seleccione la hora y zona horaria en la que comienza el trabajo de copia de seguridad.
   * Es necesario ejecutar una copia de seguridad completa; no se puede desactivar la opción **Copia de seguridad completa**.
   * Seleccione **Copia de seguridad completa** para ver la directiva.
   * No puede crear copias de seguridad diferenciales para copias de seguridad completas diarias.
   * Para **Semanalmente**, seleccione el día de la semana, la hora y la zona horaria, en la que se inicia el trabajo de copia de seguridad.

     ![Nuevos campos de directiva de copia de seguridad](./media/backup-azure-sql-database/full-backup-policy.png)  

5. De forma predeterminada, todas las opciones de **Retention Range** (Duración de retención) están seleccionadas. Desactive los límites de duración de retención que no quiera usar y establezca los intervalos deseados.

    * El período de retención mínimo para cualquier tipo de copia de seguridad (completa, diferencial o de registro) es de siete días.
    * Los puntos de recuperación se etiquetan para la retención en función de su duración de retención. Por ejemplo, si selecciona la frecuencia diaria, solo se desencadena una copia de seguridad completa cada día.
    * La copia de seguridad de un día específico se etiqueta y se retiene según la duración de la retención semanal y la configuración de esta.
    * La duración de retención mensual y anual se comporta de forma similar.

   ![Configuración del intervalo de la duración de retención](./media/backup-azure-sql-database/retention-range-interval.png)

6. En el menú de la **directiva de copia de seguridad completa**, seleccione **Aceptar** para aceptar la configuración.
7. Para agregar una directiva de copia de seguridad diferencial, seleccione **Copia de seguridad diferencial**.

   ![Configuración del intervalo de la duración de retención](./media/backup-azure-sql-database/retention-range-interval.png)
   ![Apertura del menú de directiva de copia de seguridad diferencial](./media/backup-azure-sql-database/backup-policy-menu-choices.png)

8. En **Differential Backup policy** (Directiva de copia de seguridad diferencial), seleccione **Enable** (Habilitar) para abrir los controles de retención y frecuencia.

    * A lo sumo, puede desencadenar una copia de seguridad diferencial al día.
    * Como máximo, las copias de seguridad diferenciales se pueden retener durante 180 días. Si necesita más tiempo de retención, debe usar copias de seguridad completas.

9. Seleccione **Aceptar** para guardar la directiva y volver al menú principal de la **directiva de copia de seguridad**.

10. Para agregar una directiva de copia de seguridad del registro de transacciones, seleccione **Copia de seguridad de registros**.
11. En el menú **Log Backup** (Copia de seguridad de registros), seleccione **Enable** (Habilitar) y, luego, establezca los controles de retención y frecuencia. Las copia de seguridad de registros pueden realizarse cada 15 minutos y se pueden retener durante un período máximo de 35 días.
12. Seleccione **Aceptar** para guardar la directiva y volver al menú principal de la **directiva de copia de seguridad**.

    ![Editar la directiva de copia de seguridad de registros](./media/backup-azure-sql-database/log-backup-policy-editor.png)

13. En el menú **Directiva de copia de seguridad**, elija si desea habilitar la **compresión de copia de seguridad de SQL**.
    * La compresión está deshabilitada de manera predeterminada.
    * En el back-end, Azure Backup usa la compresión de copia de seguridad nativa de SQL.

14. Después de completar las modificaciones en la directiva de copia de seguridad, seleccione **Aceptar**.

## <a name="run-an-on-demand-backup"></a>Ejecución de una copia de seguridad a petición

1. En el almacén de Recovery Services, elija los elementos de copia de seguridad.
2. Seleccione "SQL en Azure VM".
3. Haga clic con el botón derecho en una base de datos y elija "Realizar copia de seguridad ahora".
4. Elija el tipo de copia de seguridad (completa, diferencial, de registro, copiar solo copia de seguridad completa) y Compresión (Habilitar o Deshabilitar).
5. Seleccione Aceptar para iniciar la copia de seguridad.
6. Supervise el trabajo de copia de seguridad. Para ello, vaya al almacén de Recovery Services y elija "Trabajos de copia de seguridad".

## <a name="next-steps"></a>Pasos siguientes

En este tutorial se usa Azure Portal para las siguientes acciones:

> [!div class="checklist"]
>
> * Crear y configurar un almacén.
> * Detectar bases de datos y configurar copias de seguridad.
> * Configurar la protección automática de las bases de datos.
> * Ejecutar una copia de seguridad a petición.

Continúe con el tutorial siguiente para restaurar una máquina virtual de Azure desde el disco.

> [!div class="nextstepaction"]
> [Restauración de bases de datos SQL Server en máquinas virtuales de Azure](./restore-sql-database-azure-vm.md)

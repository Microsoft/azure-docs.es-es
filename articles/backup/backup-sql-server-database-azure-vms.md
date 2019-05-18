---
title: Copia de seguridad de bases de datos de SQL Server en máquinas virtuales de Azure | Microsoft Docs
description: Obtenga información sobre cómo realizar copias de seguridad de bases de datos de SQL Server en máquinas virtuales de Azure
services: backup
author: sachdevaswati
manager: vijayts
ms.service: backup
ms.topic: conceptual
ms.date: 03/23/2019
ms.author: sachdevaswati
ms.openlocfilehash: ae1f5f9148fa516c98d78afdd57887d4279f92dc
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65827679"
---
# <a name="back-up-sql-server-databases-in-azure-vms"></a>Copia de seguridad de bases de datos de SQL Server en máquinas virtuales de Azure

Las bases de datos de SQL Server son cargas de trabajo críticas que requieren un objetivo de baja de punto de recuperación (RPO) y la retención a largo plazo. Puede realizar copias de seguridad de bases de datos de SQL Server que se ejecutan en máquinas virtuales (VM) mediante el uso de [Azure Backup](backup-overview.md).

En este artículo se muestra cómo realizar copias de seguridad de una base de datos de SQL Server que se ejecuta en una máquina virtual de Azure en un almacén de copia de seguridad de Azure Recovery Services.

En este artículo, aprenderá cómo:

> [!div class="checklist"]
> * Crear y configurar un almacén.
> * Detectar bases de datos y configurar copias de seguridad.
> * Configurar la protección automática de las bases de datos.


## <a name="prerequisites"></a>Requisitos previos

Antes de realizar la copia de seguridad de una base de datos de SQL Server, compruebe los siguientes criterios:

1. Identifique o cree una [almacén de Recovery Services](backup-sql-server-database-azure-vms.md#create-a-recovery-services-vault) en la misma región o configuración regional que la máquina virtual que hospeda la instancia de SQL Server.
2. Compruebe el [permisos de máquina virtual necesitados](backup-azure-sql-database.md#fix-sql-sysadmin-permissions) para realizar copias de seguridad de bases de datos SQL.
3. Compruebe que la máquina virtual tiene [conectividad de red](backup-sql-server-database-azure-vms.md#establish-network-connectivity).
4. Asegúrese de que las bases de datos de SQL Server siguen el [directrices de nomenclatura de base de datos de copia de seguridad de Azure](#database-naming-guidelines-for-azure-backup).
5. Compruebe que no tiene otras soluciones de copia de seguridad habilitados para la base de datos. Deshabilitar todas las otras copias de seguridad de SQL Server antes de realizar la copia de seguridad de la base de datos.

> [!NOTE]
> Puede habilitar la copia de seguridad de Azure para una máquina virtual de Azure y también para una base de datos de SQL Server que se ejecutan en la máquina virtual sin conflictos.


### <a name="establish-network-connectivity"></a>Establecimiento de conectividad de red

Todas las operaciones, una máquina virtual de SQL Server requiere conectividad a direcciones IP públicas de Azure. Se producirá un error en las operaciones de máquina virtual (detección de base de datos, configurar copias de seguridad, programar copias de seguridad, restaurar puntos de recuperación y así sucesivamente) sin conectividad a direcciones IP públicas de Azure.

Establecer la conectividad con una de las siguientes opciones:

- **Permitir que los intervalos IP de centro de datos Azure**. Esta opción permite [intervalos IP](https://www.microsoft.com/download/details.aspx?id=41653) en la descarga. Para obtener acceso a un grupo de seguridad de red (NSG), use el cmdlet Set-AzureNetworkSecurityRule. Si le específicas de región única lista blanca de direcciones IP, se deberá también necesita a la lista blanca de Azure Active Directory (Azure AD) etiqueta de servicio para habilitar la autenticación.

- **Permitir el acceso mediante etiquetas NSG**. Si usa los NSG para restringir la conectividad, esta opción agrega una regla para el NSG que permita el acceso saliente a Azure Backup mediante el uso de la etiqueta de AzureBackup. Además de esta etiqueta, también necesitará correspondiente [reglas](https://docs.microsoft.com/en-us/azure/virtual-network/security-overview#service-tags) para Azure AD y el almacenamiento de Azure para permitir la conectividad de transferencia de datos y autenticación. La etiqueta de AzureBackup sólo está actualmente disponible en PowerShell. Para crear una regla mediante el uso de la etiqueta de AzureBackup:

    - Agregue las credenciales de cuenta de Azure y actualice las nubes nacionales<br/>
    `Add-AzureRmAccount`

    - Seleccione la suscripción de NSG<br/>
    `Select-AzureRmSubscription "<Subscription Id>"`

     - Seleccione el NSG<br/>
    `$nsg = Get-AzureRmNetworkSecurityGroup -Name "<NSG name>" -ResourceGroupName "<NSG resource group name>"`

    - Agregar Permitir regla de salida de la etiqueta de servicio de copia de seguridad de Azure<br/>
    `Add-AzureRmNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg -Name "AzureBackupAllowOutbound" -Access Allow -Protocol * -Direction Outbound -Priority <priority> -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix "AzureBackup" -DestinationPortRange 443 -Description "Allow outbound traffic to Azure Backup service"`

  - Guardar el NSG<br/>
    `Set-AzureRmNetworkSecurityGroup -NetworkSecurityGroup $nsg`
- **Permitir el acceso mediante el uso de etiquetas de Firewall de Azure**. Si usa Firewall de Azure, crear una regla de aplicación mediante el AzureBackup [etiqueta FQDN](https://docs.microsoft.com/en-us/azure/firewall/fqdn-tags). Esto permite el acceso saliente a Azure Backup.
- **Implementar un servidor proxy HTTP para enrutar el tráfico**. Cuando hace una copia de seguridad de una base de datos de SQL Server en una máquina virtual de Azure, la extensión de copia de seguridad en la máquina virtual usa las API de HTTPS para enviar comandos de administración para Azure Backup y datos a Azure Storage. La extensión de copia de seguridad también usa Azure AD para la autenticación. Enrute el tráfico de extensión de copia de seguridad de estos tres servicios a través del proxy HTTP. Las extensiones son el único componente que está configurado para tener acceso a la red internet pública.

Las opciones de conectividad incluyen las siguientes ventajas y desventajas:

**Opción** | **Ventajas** | **Desventajas**
--- | --- | ---
Permitir intervalos IP | Sin costo adicional | Difícil de administrar, ya que los intervalos de direcciones IP cambian con el tiempo <br/><br/> Proporciona acceso a la totalidad de Azure, no solo el almacenamiento de Azure
Usar etiquetas de servicio NSG | Más fácil de administrar como se mezclan automáticamente los cambios de intervalo <br/><br/> Sin costo adicional <br/><br/> | Solo se puede usar con NSG <br/><br/> Proporciona acceso a todo el servicio
Usar etiquetas de FQDN del Firewall de Azure | Más fácil de administrar como se administran automáticamente los FQDN necesarios | Solo se puede usar con Firewall de Azure
Usar un servidor proxy HTTP | Se permite un control detallado en el proxy sobre el almacenamiento de direcciones URL <br/><br/> Único punto de acceso a internet a las máquinas virtuales <br/><br/> No están sujetas a cambios en la dirección IP de Azure | Costos adicionales para ejecutar una máquina virtual con el software de proxy

### <a name="set-vm-permissions"></a>Establecer permisos de máquina virtual

Al configurar una copia de seguridad para una base de datos de SQL Server, Azure Backup hace lo siguiente:

- Agrega la extensión AzureBackupWindowsWorkload.
- Crea una cuenta de NT SERVICE\AzureWLBackupPluginSvc para detectar las bases de datos en la máquina virtual. Esta cuenta se usa para una copia de seguridad y restauración y requiere permisos de sysadmin SQL.
- Detecta bases de datos que se ejecutan en una máquina virtual, Azure Backup usa la cuenta NT AUTHORITY\SYSTEM. Esta cuenta debe ser un inicio de sesión pública en SQL.

Si no se creó la máquina virtual de SQL Server en Azure Marketplace, puede recibir un error UserErrorSQLNoSysadminMembership. Para obtener más información, consulte la sección de consideraciones y limitaciones de la característica se encuentra en [sobre SQL Server Backup en Azure Virtual Machines](backup-azure-sql-database.md#fix-sql-sysadmin-permissions).

### <a name="database-naming-guidelines-for-azure-backup"></a>Directrices de nomenclatura de Azure Backup de la base de datos

Evite el uso de los elementos siguientes en los nombres de base de datos:

  * Al final y los espacios iniciales
  * Los signos de exclamación (!)
  * Cierre los corchetes ([])
  * A partir de F:\

Creación de alias está disponible para los caracteres no admitidos, pero se recomienda evitarlos. Para obtener más información, consulte [Descripción del modelo de datos del servicio Tabla](https://docs.microsoft.com/rest/api/storageservices/Understanding-the-Table-Service-Data-Model?redirectedfrom=MSDN).


[!INCLUDE [How to create a Recovery Services vault](../../includes/backup-create-rs-vault.md)]

## <a name="discover-sql-server-databases"></a>Detección de bases de datos SQL Server

Cómo detectar bases de datos que se ejecuta en una máquina virtual:

1. En [Azure Portal](https://portal.azure.com), abra el almacén de Recovery Services que usa para la copia de seguridad de la base de datos.

2. En el **almacén de Recovery Services** panel, seleccione **copia de seguridad**.

   ![Seleccionar Copia de seguridad para abrir el menú Objetivo de Backup](./media/backup-azure-sql-database/open-backup-menu.png)

3. En **objetivo de Backup**, establezca **donde se ejecuta la carga de trabajo?** a **Azure**.

4. En **What do you want to backup** (¿De qué quiere realizar una copia de seguridad?), seleccione **SQL Server in Azure VM** (SQL Server en la máquina virtual de Azure).

    ![Seleccionar SQL Server en una máquina virtual de Azure para la copia de seguridad](./media/backup-azure-sql-database/choose-sql-database-backup-goal.png)

5. En **Backup Goal** > **Discover DBs in VMs** (Objetivo de Backup > Detectar bases de datos en máquinas virtuales), seleccione **Start Discovery** (Iniciar detección) para buscar las máquinas virtuales sin protección en la suscripción. Esta búsqueda puede tardar unos minutos, dependiendo del número de máquinas virtuales no protegidas en la suscripción.

   - Tras la detección, las máquinas virtuales no protegidas deben aparecer en la lista por nombre y grupo de recursos.
   - Si una máquina virtual no se muestra según lo esperado, consulte si lo es ya una copia de seguridad en un almacén.
   - Varias máquinas virtuales pueden tener el mismo nombre, pero que le pertenecen a distintos grupos de recursos.

     ![La copia de seguridad queda pendiente durante la búsqueda de bases de datos en máquinas virtuales](./media/backup-azure-sql-database/discovering-sql-databases.png)

6. En la lista de máquinas virtuales, seleccione la que ejecuta la base de datos de SQL Server > **Discover DBs** (Detectar bases de datos).

7. Detección de base de datos de seguimiento en **notificaciones**. El tiempo necesario para esta acción depende del número de bases de datos de máquina virtual. Cuando se detectan las bases de datos seleccionadas, aparece un mensaje de operación correcta.

    ![Mensaje de implementación correcta](./media/backup-azure-sql-database/notifications-db-discovered.png)

8. Azure Backup detecta todas las bases de datos de SQL Server en la máquina virtual. Durante la detección, los elementos siguientes se producen en segundo plano:

    - Copia de seguridad de Azure registra la máquina virtual con el almacén de copia de seguridad de cargas de trabajo. Pueden una copia de seguridad de todas las bases de datos en la máquina virtual registrada a sólo este almacén.
    - Azure Backup instala la extensión de AzureBackupWindowsWorkload en la máquina virtual. No hay ningún agente está instalado en una base de datos SQL.
    - Azure Backup crea la cuenta de servicio NT Service\AzureWLBackupPluginSvc en la máquina virtual.
      - Todas las operaciones de copia de seguridad y restauración utilizan la cuenta de servicio.
      - NT Service\AzureWLBackupPluginSvc requiere permisos de sysadmin SQL. Todas las VM de SQL Server creado en el Marketplace conlleva la SqlIaaSExtension instalado. La extensión de AzureBackupWindowsWorkload usa el SQLIaaSExtension para obtener automáticamente los permisos necesarios.
    - Si no se creó la máquina virtual de Marketplace, la máquina virtual no tiene el SqlIaaSExtension instalado y se produce un error en la operación de detección con el mensaje de error UserErrorSQLNoSysAdminMembership. Para corregir este problema, siga el [instrucciones](backup-azure-sql-database.md#fix-sql-sysadmin-permissions).

        ![Seleccionar la máquina virtual y la base de datos](./media/backup-azure-sql-database/registration-errors.png)

## <a name="configure-backup"></a>Configuración de la copia de seguridad  

1. En **objetivo de copia de seguridad** > **paso 2: Configurar copia de seguridad**, seleccione **Configurar copia de seguridad**.

   ![Seleccionar Configurar copia de seguridad](./media/backup-azure-sql-database/backup-goal-configure-backup.png)

2. En **seleccionar elementos para copia de seguridad**, verá todos los grupos de disponibilidad registrada y las instancias de SQL Server independiente. Seleccione la flecha situada a la izquierda de una fila para expandir la lista de todas las bases de datos no protegidos en esa instancia o un grupo de disponibilidad Always On.  

    ![Mostrar todas las instancias de SQL Server con bases de datos independientes](./media/backup-azure-sql-database/list-of-sql-databases.png)

3. Elija todas las bases de datos que desea proteger y, a continuación, seleccione **Aceptar**.

   ![Protección de la base de datos](./media/backup-azure-sql-database/select-database-to-protect.png)

   Para optimizar las cargas de copia de seguridad, el número máximo de bases de datos en un trabajo de copia de seguridad en Azure Backup se establece en 50.

     * Para proteger más de 50 bases de datos, configure varias copias de seguridad.
     * Para habilitar [ ](#enable-auto-protection) toda la instancia o el grupo de disponibilidad Always On. En el **AUTOPROTECT** lista desplegable, seleccione **ON**y, a continuación, seleccione **Aceptar**.
     
    > [!NOTE]
    > El [protección automática](#enable-auto-protection) característica permite no solo la protección en todas las bases de datos existentes a la vez, pero protege también automáticamente las nuevas bases de datos agregados a esa instancia o el grupo de disponibilidad.  

4. Seleccione **Aceptar** para abrir **directiva de copia de seguridad**.

    ![Habilitar la protección automática para el grupo de disponibilidad Always On](./media/backup-azure-sql-database/enable-auto-protection.png)

5. En **directiva de copia de seguridad**, elija una directiva y, a continuación, seleccione **Aceptar**.

   - Seleccione la directiva predeterminada como HourlyLogBackup.
   - Elegir una directiva de copia de seguridad existente creada previamente para SQL.
   - Definir una nueva directiva basada en el intervalo de retención y RPO.

     ![Seleccionar directiva de copia de seguridad](./media/backup-azure-sql-database/select-backup-policy.png)

6. En **copia de seguridad**, seleccione **habilitar copia de seguridad**.

    ![Habilitar la directiva de copia de seguridad elegida](./media/backup-azure-sql-database/enable-backup-button.png)

7. Realizar un seguimiento del progreso de la configuración en el **notificaciones** área del portal.

    ![Área de notificaciones](./media/backup-azure-sql-database/notifications-area.png)

### <a name="create-a-backup-policy"></a>Creación de una directiva de copia de seguridad

Una directiva de copia de seguridad define cuándo se realizan las copias de seguridad y cuánto tiempo se conservan.

- Una directiva se crea en el nivel de almacén.
- Varios almacenes pueden usar la misma directiva de copia de seguridad, pero debe aplicar la directiva de copia de seguridad a cada almacén.
- Al crear una directiva de copia de seguridad, la copia de seguridad completa diaria es la predeterminada.
- Puede agregar una copia de seguridad diferencial, pero solo si configura las copias de seguridad completas para que se realicen semanalmente.
- Obtenga información sobre [diferentes tipos de directivas de copia de seguridad](backup-architecture.md#sql-server-backup-types).

Para crear una directiva de copia de seguridad:

1. En el almacén, seleccione **las directivas de copia de seguridad** > **agregar**.
2. En **agregar**, seleccione **SQL Server en la máquina virtual de Azure** para definir el tipo de directiva.

   ![Elección de un tipo de directiva para la nueva directiva de copia de seguridad](./media/backup-azure-sql-database/policy-type-details.png)

3. En **Nombre de la directiva**, escriba un nombre para la nueva directiva.
4. En **directiva de copia de seguridad completa**, seleccione un **frecuencia de copia de seguridad**. Elija **diario** o **semanal**.

   - En **Diariamente**, seleccione la hora y zona horaria en la que comienza el trabajo de copia de seguridad.
   - Para **Semanalmente**, seleccione el día de la semana, la hora y la zona horaria, en la que se inicia el trabajo de copia de seguridad.
   - Ejecutar una copia de seguridad completa, porque no se puede desactivar la **copia de seguridad completa** opción.
   - Seleccione **copia de seguridad completa** para ver la directiva.
   - No puede crear copias de seguridad diferenciales para copias de seguridad completas diarias.

     ![Nuevos campos de directiva de copia de seguridad](./media/backup-azure-sql-database/full-backup-policy.png)  

5. En **duración de retención**, todas las opciones están seleccionadas de forma predeterminada. Borre cualquier duración de retención limita que no desee y, a continuación, establezca los intervalos para usar.

    - Período de retención mínimo para cualquier tipo de copia de seguridad (completas, diferenciales y del registro) es siete días.
    - Los puntos de recuperación se etiquetan para la retención en función de su duración de retención. Por ejemplo, si selecciona la frecuencia diaria, solo se desencadena una copia de seguridad completa cada día.
    - La copia de seguridad de un día concreto es etiquetado y se mantienen se basa en la duración de retención semanal y la configuración de retención semanal.
    - Mensual y anual en intervalos de retención se comportan de forma similar.

       ![Configuración del intervalo de la duración de retención](./media/backup-azure-sql-database/retention-range-interval.png)

6. En el menú de la **directiva de copia de seguridad completa**, seleccione **Aceptar** para aceptar la configuración.
7. Para agregar una directiva de copia de seguridad diferencial, seleccione **Copia de seguridad diferencial**.

   ![Configuración del intervalo de la duración de retención](./media/backup-azure-sql-database/retention-range-interval.png)
   ![Apertura del menú de directiva de copia de seguridad diferencial](./media/backup-azure-sql-database/backup-policy-menu-choices.png)

8. En **Differential Backup policy** (Directiva de copia de seguridad diferencial), seleccione **Enable** (Habilitar) para abrir los controles de retención y frecuencia.

    - Puede desencadenar la copia de seguridad diferencial solo uno al día.
    - Como máximo, las copias de seguridad diferenciales se pueden retener durante 180 días. Tiempo de retención, use copias de seguridad completas.

9. Seleccione **Aceptar** para guardar la directiva y volver al menú principal de la **directiva de copia de seguridad**.

10. Para agregar una directiva de copia de seguridad del registro de transacciones, seleccione **Copia de seguridad de registros**.
11. En el menú **Log Backup** (Copia de seguridad de registros), seleccione **Enable** (Habilitar) y, luego, establezca los controles de retención y frecuencia. Las copias de seguridad del registro pueden producirse con la frecuencia que cada 15 minutos y se pueden conservar hasta 35 días.
12. Seleccione **Aceptar** para guardar la directiva y volver al menú principal de la **directiva de copia de seguridad**.

    ![Editar la directiva de copia de seguridad de registros](./media/backup-azure-sql-database/log-backup-policy-editor.png)

13. En el menú **Directiva de copia de seguridad**, elija si desea habilitar la **compresión de copia de seguridad de SQL**.
    - La compresión está deshabilitada de manera predeterminada.
    - En el back-end, Azure Backup usa la compresión de copia de seguridad nativa de SQL.

14. Después de completar las modificaciones en la directiva de copia de seguridad, seleccione **Aceptar**.


### <a name="modify-policy"></a>Modificar directiva
Modificar la directiva para cambiar el intervalo de frecuencia o la retención de copia de seguridad.

> [!NOTE]
> Cualquier cambio en el período de retención se aplicará a posteriori a todos los puntos de recuperación anteriores además de las nuevas.

En el panel del almacén, vaya a **administrar** > **las directivas de copia de seguridad** y elija la directiva que desea editar.

  ![Administrar la directiva de copia de seguridad](./media/backup-azure-sql-database/modify-backup-policy.png)


## <a name="enable-auto-protection"></a>Habilitación de la protección automática  

Puede habilitar la protección automática para copias de seguridad automáticas todas las bases de datos existentes y futuros en una instancia de SQL Server independiente o a un grupo de disponibilidad Always On.

- No hay ningún límite en el número de bases de datos que se pueden seleccionar para la protección automática al mismo tiempo.
- No se puede proteger o excluir las bases de datos de la protección en una instancia en el momento de que habilitar la protección automática de forma selectiva.
- Si la instancia ya incluye algunas bases de datos protegidos, deberá permanecen protegidos en sus respectivas directivas incluso después de activar la protección automática. Todas las bases de datos desprotegidas agregados posteriormente tendrá solo una única directiva que se define en el momento de habilitar la protección automática, anuncie **Configurar copia de seguridad**. Sin embargo, puede cambiar la directiva asociada con una base de datos protegida automáticamente más tarde.  

Para habilitar la protección automática:

  1. En **Items to backup** (Elementos para copia de seguridad), seleccione la instancia para la que quiere habilitar la protección automática.
  2. Seleccione la lista desplegable bajo **AUTOPROTECT**, elija **ON**y, a continuación, seleccione **Aceptar**.

      ![Habilitar la protección automática en el grupo de disponibilidad](./media/backup-azure-sql-database/enable-auto-protection.png)

  3. La copia de seguridad se configura para todas las bases de datos juntas y se puede realizar un seguimiento de ella en **Backup Jobs** (Trabajos de copia de seguridad).

Si tiene que deshabilitar la protección automática, seleccione el nombre de instancia **Configurar copia de seguridad**y, a continuación, seleccione **deshabilitar Autoprotect** para la instancia. Todas las bases de datos seguirán una copia de seguridad, pero futuras bases de datos no estará protegidas automáticamente.

![Deshabilite la protección automática en dicha instancia](./media/backup-azure-sql-database/disable-auto-protection.png)

 
## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo:

- [Restaurar bases de datos de SQL Server de copia de seguridad](restore-sql-database-azure-vm.md)
- [Administrar bases de datos de SQL Server de copia de seguridad](manage-monitor-sql-database-backup.md)

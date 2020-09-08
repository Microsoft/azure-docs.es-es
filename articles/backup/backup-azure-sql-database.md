---
title: Copia de seguridad de bases de datos de SQL Server en Azure
description: En este artículo se explica cómo realizar una copia de seguridad de SQL Server en Azure. En este tutorial también se explica cómo se realiza la recuperación de SQL Server.
ms.topic: conceptual
ms.date: 06/18/2019
ms.openlocfilehash: 3627162ef2f4330a4b6a78625b5e07bdcf56419b
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89376543"
---
# <a name="about-sql-server-backup-in-azure-vms"></a>Acerca de la copia de seguridad de SQL Server en máquinas virtuales de Azure

[Azure Backup](backup-overview.md) ofrece una solución especializada basada en secuencias para realizar copias de seguridad de SQL Server que se ejecutan en máquinas virtuales de Azure. Esta solución se ajusta a los beneficios de copia de seguridad sin necesidad de infraestructura, retención a largo plazo y administración central que ofrece Azure Backup. Además, proporciona las siguientes ventajas específicamente para SQL Server:

1. Copias de seguridad con reconocimiento de la carga de trabajo que admiten todos los tipos de copia de seguridad: completa, diferencial y de registros
2. Objetivo de punto de recuperación (RPO) de 15 minutos con copias de seguridad frecuentes de registros
3. Recuperación a un momento dado en un segundo preciso
4. Copia de seguridad y restauración individuales a nivel de base de datos

Para ver los escenarios de copia de seguridad y restauración que se admiten en la actualidad, consulte la [matriz de compatibilidad](sql-support-matrix.md#scenario-support).

## <a name="backup-process"></a>Proceso de copia de seguridad

Esta solución aprovecha las API nativas de SQL para realizar copias de seguridad de las bases de datos SQL.

* Una vez que especifique la máquina virtual con SQL Server que desea proteger y que consulte las bases de datos que hay en ella, el servicio Azure Backup instalará una extensión de copia de seguridad de cargas de trabajo en la máquina virtual con la extensión `AzureBackupWindowsWorkload` del nombre.
* Esta extensión consta de un coordinador y un complemento SQL. Mientras que el coordinador es el responsable de desencadenar los flujos de trabajo de varias operaciones como la configuración de las copias de seguridad o la copia de seguridad y restauración, el complemento es responsable de flujo de datos real.
* Para poder detectar bases de datos en esta máquina virtual, Azure Backup crea la cuenta `NT SERVICE\AzureWLBackupPluginSvc`. Dicha cuenta se usa para realizar operaciones de copia de seguridad y restauración, y requiere permisos de administrador del sistema de SQL. La cuenta `NT SERVICE\AzureWLBackupPluginSvc` es una [cuenta de servicio virtual](/windows/security/identity-protection/access-control/service-accounts#virtual-accounts) y, por lo tanto, no requiere administración de contraseñas. Azure Backup utiliza la cuenta `NT AUTHORITY\SYSTEM` para la detección y la consulta de bases de datos, por lo que debe tener un inicio de sesión público en SQL. Si no se creó la máquina virtual con SQL Server en Azure Marketplace, podría recibir el error **UserErrorSQLNoSysadminMembership**. Si esto sucede, [siga estas instrucciones](#set-vm-permissions).
* Una vez que desencadene la protección de la configuración en las bases de datos seleccionadas, el servicio de copia de seguridad configura el coordinador con las programaciones de las copias de seguridad y otros detalles de la directiva, que la extensión almacena en caché localmente en la máquina virtual.
* A la hora programada, el coordinador se comunica con el complemento y empieza a transmitir los datos de la copia de seguridad en secuencias desde el servidor con SQL Server mediante VDI.  
* El complemento envía los datos directamente al almacén de Recovery Services, por lo que se elimina la necesidad de una ubicación de almacenamiento provisional. El servicio Azure Backup cifra los datos y los almacena en cuentas de almacenamiento.
* Cuando se completa la transferencia de datos, el coordinador realiza la confirmación con el servicio de copia de seguridad.

  ![Arquitectura de copia de seguridad de SQL](./media/backup-azure-sql-database/backup-sql-overview.png)

## <a name="before-you-start"></a>Antes de comenzar

Antes de empezar, haga las siguientes comprobaciones:

1. Asegúrese de tener una instancia de SQL Server que se ejecuta en Azure. Puede [crear rápidamente una instancia de SQL Server](../azure-sql/virtual-machines/windows/sql-vm-create-portal-quickstart.md) en Marketplace.
2. Revise los apartados en que se realizan [consideraciones acerca de las características](sql-support-matrix.md#feature-considerations-and-limitations) y se indica la [compatibilidad con los escenarios](sql-support-matrix.md#scenario-support).
3. [Revise las preguntas más frecuentes](faq-backup-sql-server.md) sobre este escenario.

## <a name="set-vm-permissions"></a>Establecer permisos de máquina virtual

  Al ejecutar la detección en un servidor SQL Server, Azure Backup hace lo siguiente:

* Agrega la extensión AzureBackupWindowsWorkload.
* Crea una cuenta NT SERVICE\AzureWLBackupPluginSvc para detectar las bases de datos en la máquina virtual. Dicha cuenta se usa para realizar una operación de copia de seguridad y restauración, y requiere permisos de administrador del sistema de SQL.
* Detecta las bases de datos que se ejecutan en una CM, Azure Backup usa la cuenta NT AUTHORITY\SYSTEM. Esta cuenta ofrecer inicio de sesión público en SQL.

Si no creó la máquina virtual con SQL Server en Azure Marketplace o si usa SQL 2008 o 2008 R2, es posible que reciba un error **UserErrorSQLNoSysadminMembership**.

Para conceder permisos en el caso de que **SQL 2008** y **2008 R2** se ejecuten en Windows 2008 R2, consulte [aquí](#give-sql-sysadmin-permissions-for-sql-2008-and-sql-2008-r2).

Para las demás versiones, realice los pasos siguientes para corregir los permisos:

  1. Use una cuenta con permisos sysadmin de SQL Server para iniciar sesión en SQL Server Management Studio (SSMS). A menos que necesite permisos especiales, debería funcionar la autenticación de Windows.
  2. En el servidor SQL Server, abra la carpeta **Seguridad/Inicios de sesión**.

      ![Abrir carpeta Security/Logins para ver las cuentas](./media/backup-azure-sql-database/security-login-list.png)

  3. Haga clic con el botón derecho en la carpeta **Logins** y seleccione **Nuevo inicio de sesión**. En **Inicio de sesión: Nuevo**, seleccione **Buscar**.

      ![En el cuadro de diálogo Inicio de sesión - Nuevo, seleccionar Buscar](./media/backup-azure-sql-database/new-login-search.png)

  4. La cuenta de servicio virtual de Windows **NT SERVICE\AzureWLBackupPluginSvc** se creó durante las fases de registro de la máquina virtual y de detección de SQL. Escriba el nombre de la cuenta como se muestra en **Escriba el nombre del objeto que desea seleccionar**. Seleccione **Comprobar nombres** para resolver el nombre. Seleccione **Aceptar**.

      ![Seleccionar Comprobar nombres para resolver el nombre de servicio desconocido](./media/backup-azure-sql-database/check-name.png)

  5. En **Roles de servidor**, asegúrese de que está seleccionado el rol **sysadmin**. Seleccione **Aceptar**. Ahora deben existir los permisos necesarios.

      ![Asegúrese de que el rol del servidor administrador del sistema está seleccionado](./media/backup-azure-sql-database/sysadmin-server-role.png)

  6. Ahora, asocie la base de datos con el almacén de Recovery Services. En Azure Portal, en la lista **Servidores protegidos**, haga clic con el botón derecho en el servidor que tiene un estado de error > **Rediscover DBs** (Volver a detectar bases de datos).

      ![Comprobar que el servidor tiene los permisos adecuados](./media/backup-azure-sql-database/check-erroneous-server.png)

  7. Compruebe el progreso en el área **Notificaciones**. Cuando se detectan las bases de datos seleccionadas, aparece un mensaje de operación correcta.

      ![Mensaje de implementación correcta](./media/backup-azure-sql-database/notifications-db-discovered.png)

> [!NOTE]
> Si su servidor SQL Server tiene varias instancias de SQL Server instaladas, debe agregar permisos de administrador del sistema a la cuenta **NT Service\AzureWLBackupPluginSvc** en todas las instancias de SQL.

### <a name="give-sql-sysadmin-permissions-for-sql-2008-and-sql-2008-r2"></a>Conceder permisos de administrador del sistema para SQL 2008 y SQL 2008 R2

Agregue los inicios de sesión **NT AUTHORITY\SYSTEM** y **Service\AzureWLBackupPluginSvc NT** en la instancia de SQL Server:

1. Vaya a la instancia de SQL Server en el Explorador de objetos.
2. Vaya a Seguridad -> Inicios de sesión.
3. Haga clic con el botón derecho en Inicios de sesión y seleccione *Nuevo inicio de sesión*.

    ![Nuevo inicio de sesión mediante SSMS](media/backup-azure-sql-database/sql-2k8-new-login-ssms.png)

4. Vaya a la pestaña General y escriba **NT AUTHORITY\SYSTEM** como nombre de inicio de sesión.

    ![Nombre de inicio de sesión para SSMS](media/backup-azure-sql-database/sql-2k8-nt-authority-ssms.png)

5. Vaya a *Roles del servidor* y elija los roles *public* y *sysadmin*.

    ![Elegir de roles en SSMS](media/backup-azure-sql-database/sql-2k8-server-roles-ssms.png)

6. Vaya a *Estado*. En Permiso de conexión al motor de base de datos, seleccione *Conceder* y en Inicio de sesión, seleccione *Habilitado*.

    ![Conceder permisos en SSMS](media/backup-azure-sql-database/sql-2k8-grant-permission-ssms.png)

7. Seleccione Aceptar.
8. Repita la misma secuencia de pasos (de 1 a 7 arriba) para agregar el inicio de sesión NT Service\AzureWLBackupPluginSvc a la instancia de SQL Server. Si el inicio de sesión ya existe, asegúrese de que tenga el rol de servidor sysadmin y, en Estado, en Permiso de conexión al motor de base de dato, se haya seleccionado Conceder y, en Inicio de sesión, se haya seleccionado Habilitado.
9. Después de conceder el permiso, seleccione **Re-discover DBs** (Volver a detectar bases de datos) en el portal: Almacén **->** Infraestructura de Backup **->** Workload in Azure VM (Carga de trabajo en la VM de Azure):

    ![Volver a detectar bases de datos en Azure Portal](media/backup-azure-sql-database/sql-rediscover-dbs.png)

Como alternativa, puede automatizar la concesión de los permisos al ejecutar los siguientes comandos de PowerShell en modo de administrador. El nombre de instancia se establece en MSSQLSERVER de manera predeterminada. Si fuera necesario, cambie el argumento de nombre de instancia en el script:

```powershell
param(
    [Parameter(Mandatory=$false)]
    [string] $InstanceName = "MSSQLSERVER"
)
if ($InstanceName -eq "MSSQLSERVER")
{
    $fullInstance = $env:COMPUTERNAME   # In case it is the default SQL Server Instance
}
else
{
    $fullInstance = $env:COMPUTERNAME + "\" + $InstanceName   # In case of named instance
}
try
{
    sqlcmd.exe -S $fullInstance -Q "sp_addsrvrolemember 'NT Service\AzureWLBackupPluginSvc', 'sysadmin'" # Adds login with sysadmin permission if already not available
}
catch
{
    Write-Host "An error occurred:"
    Write-Host $_.Exception|format-list -force
}
try
{
    sqlcmd.exe -S $fullInstance -Q "sp_addsrvrolemember 'NT AUTHORITY\SYSTEM', 'sysadmin'" # Adds login with sysadmin permission if already not available
}
catch
{
    Write-Host "An error occurred:"
    Write-Host $_.Exception|format-list -force
}
```

## <a name="next-steps"></a>Pasos siguientes

* [Más información](backup-sql-server-database-azure-vms.md) acerca de la copia de seguridad de bases de datos de SQL Server.
* [Más información](restore-sql-database-azure-vm.md) sobre la restauración de copias de seguridad de bases de datos de SQL Server.
* [Más información](manage-monitor-sql-database-backup.md) sobre la administración de copias de seguridad de bases de datos de SQL Server.

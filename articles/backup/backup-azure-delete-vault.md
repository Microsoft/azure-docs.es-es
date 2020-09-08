---
title: Eliminación de un almacén de Microsoft Azure Recovery Services
description: En este artículo, aprenderá a quitar dependencias y, luego, a eliminar un almacén de Azure Backup Recovery Services (MARS).
ms.topic: conceptual
ms.date: 06/04/2020
ms.openlocfilehash: c0b75d147abba45a745f811de5e4b8ac45088bd8
ms.sourcegitcommit: ac7ae29773faaa6b1f7836868565517cd48561b2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88826742"
---
# <a name="delete-an-azure-backup-recovery-services-vault"></a>Eliminación de un almacén de Recovery Services de Azure Backup

En este artículo se describe cómo eliminar un almacén de Recovery Services de [Azure Backup](backup-overview.md). Contiene instrucciones para quitar las dependencias y, a continuación, eliminar un almacén.

## <a name="before-you-start"></a>Antes de comenzar

No se puede eliminar un almacén de Recovery Services con cualquiera de las siguientes dependencias:

- No es posible eliminar un almacén que contenga orígenes de datos protegidos (por ejemplo, máquinas virtuales de IaaS, bases de datos de SQL o recursos compartidos de archivos de Azure).
- No puede eliminar un almacén que contiene datos de copia de seguridad. Una vez eliminados los datos de copia de seguridad, entrará en el estado de eliminación temporal.
- No puede eliminar un almacén que contiene datos de copia de seguridad en el estado de eliminación temporal.
- No puede eliminar un almacén que tiene cuentas de almacenamiento registradas.

Si intenta eliminar el almacén sin quitar las dependencias, se producirá uno de los siguientes mensajes de error:

- No se puede eliminar el almacén porque contiene recursos. Asegúrese de que no haya elementos de copia de seguridad, servidores protegidos o servidores de administración de copia de seguridad asociados a este almacén. Anule el registro de los siguientes contenedores asociados a este almacén antes de continuar con la eliminación.

- No es posible eliminar el almacén de Recovery Services si contiene elementos de copia de seguridad en estado de eliminación temporal. Los elementos eliminados temporalmente se eliminan de forma permanente 14 días después de la operación de eliminación. Pruebe a eliminar el almacén después de que se eliminen de forma permanente los elementos de copia de seguridad y no quede ningún elemento en estado de eliminación temporal en el almacén. Para obtener más información, consulte [Eliminación temporal para Azure Backup](./backup-azure-security-feature-cloud.md).

## <a name="proper-way-to-delete-a-vault"></a>Manera adecuada de eliminar un almacén

>[!WARNING]
>La operación siguiente es destructiva y no se puede deshacer. Todos los datos de copia de seguridad y los elementos de copia de seguridad asociados con el servidor protegido se eliminarán de forma permanente. Proceda con precaución.

Para eliminar correctamente un almacén, debe seguir los pasos en este orden:

- **Paso 1**: Deshabilite la característica de eliminación temporal. [Consulte aquí](./backup-azure-security-feature-cloud.md#enabling-and-disabling-soft-delete) los pasos para deshabilitar la eliminación temporal.

- **Paso 2**: Después de deshabilitar la eliminación temporal, compruebe si hay alguno de los elementos que anteriormente estaban en el estado de eliminación temporal. Si hay elementos en el estado de eliminación temporal, debe *recuperarlos* y *eliminarlos* de nuevo. [Siga estos pasos](./backup-azure-security-feature-cloud.md#permanently-deleting-soft-deleted-backup-items) para buscar elementos eliminados temporalmente y eliminarlos de forma permanente.

- **Paso 3**: Debe comprobar los tres lugares siguientes para verificar si hay elementos protegidos:

  - **Elementos protegidos por la nube**: Vaya al menú del panel del almacén > **Elementos de copia de seguridad**. Todos los elementos que se enumeran aquí deben eliminarse con **Detener copia de seguridad** o **Eliminar datos de copia de seguridad** junto con sus datos de copia de seguridad.  [Siga estos pasos](#delete-protected-items-in-the-cloud) para quitar esos elementos.
  - **Instancia de SQL Server**: Vaya al menú del panel del almacén > **Infraestructura de copia de seguridad** > **Servidores protegidos**. En Servidores protegidos, seleccione el servidor del que desea anular el registro. Para eliminar el almacén, debe anular el registro de todos los servidores. Haga clic con el botón derecho en el servidor protegido y seleccione **Anular registro**.
  - **Servidores protegidos MARS**: Vaya al menú del panel del almacén > **Infraestructura de copia de seguridad** > **Servidores protegidos**. Si tiene servidores protegidos MARS, todos los elementos que se enumeran aquí deben eliminarse junto con sus datos de copia de seguridad. [Siga estos pasos](#delete-protected-items-on-premises) para eliminar los servidores protegidos MARS.
  - **Servidores de administración MABS o DPM**: Vaya al menú del panel del almacén > **Infraestructura de copia de seguridad** > **Servidores de administración de copia de seguridad**. Si tiene DPM o Azure Backup Server (MABS), todos los elementos que se enumeran aquí se deben eliminar junto con sus datos de copia de seguridad (o debe anularse su registro). [Siga estos pasos](#delete-protected-items-on-premises) para eliminar los servidores de administración.

- **Paso 4**: Debe asegurarse de que se han eliminado todas las cuentas de almacenamiento registradas. Vaya al menú del panel del almacén > **Infraestructura de copia de seguridad** > **Cuentas de almacenamiento**. Si aparecen cuentas de almacenamiento, debe anular el registro de todas ellas. Para aprender a anular el registro de la cuenta, consulte [Anulación del registro de una cuenta de almacenamiento](manage-afs-backup.md#unregister-a-storage-account).

Después de completar estos pasos, puede continuar y [eliminar el almacén](#delete-the-recovery-services-vault).

Si no tiene elementos protegidos locales o en la nube, pero sigue obteniendo el error de eliminación del almacén, siga los pasos de [Eliminación del almacén de Recovery Services mediante Azure Resource Manager](#delete-the-recovery-services-vault-by-using-azure-resource-manager).

## <a name="delete-protected-items-in-the-cloud"></a>Eliminación de elementos protegidos en la nube

En primer lugar, lea la sección **[Antes de comenzar](#before-you-start)** para comprender el proceso de eliminación de dependencias y almacenes.

Para detener la protección y eliminar los datos de copia de seguridad, siga estos pasos:

1. En el portal, vaya a **Almacén de Recovery Services** y, a continuación, vaya a **Elementos de copia de seguridad**. A continuación, elija los elementos protegidos en la nube (por ejemplo,Azure Virtual Machines, Azure Storage [el servicio Azure Files] o SQL Server en Azure Virtual Machines).

    ![Seleccione el tipo de copia de seguridad.](./media/backup-azure-delete-vault/azure-storage-selected.png)

2. Haga clic con el botón derecho para seleccionar el elemento de copia de seguridad. En función de si el elemento de copia de seguridad está protegido o no, el menú muestra el panel **Detener copia de seguridad** o **Eliminar datos de copia de seguridad**.

    - Si aparece el panel **Detener copia de seguridad**, seleccione **Eliminar datos de copia de seguridad** en el menú desplegable. Escriba el nombre del elemento de copia de seguridad (este campo distingue entre mayúsculas y minúsculas) y, a continuación, seleccione un motivo en el menú desplegable. Escriba sus comentarios, si tiene alguno. Luego, seleccione **Detener copia de seguridad**.

        ![El panel Detener copia de seguridad.](./media/backup-azure-delete-vault/stop-backup-item.png)

    - Si aparece el panel **Detener copia de seguridad**, escriba el nombre del elemento de copia de seguridad (este campo distingue entre mayúsculas y minúsculas) y, a continuación, seleccione un motivo en el menú desplegable. Escriba sus comentarios, si tiene alguno. A continuación, seleccione **Eliminar**.

         ![El panel Eliminar datos de copia de seguridad.](./media/backup-azure-delete-vault/stop-backup-blade-delete-backup-data.png)

3. Active el icono **Notificación**: ![El icono de notificación.](./media/backup-azure-delete-vault/messages.png) Una vez finalizado el proceso, el servicio muestra el siguiente mensaje: *Deteniendo copia de seguridad y eliminando datos de copia de seguridad para "* elemento de copia de seguridad *"* . *La operación se completó correctamente*.
4. Seleccione **Actualizar** en el menú **Elementos de copia de seguridad** para comprobar si se ha eliminado el elemento de copia de seguridad.

      ![La página Eliminar elementos de copia de seguridad.](./media/backup-azure-delete-vault/empty-items-list.png)

## <a name="delete-protected-items-on-premises"></a>Eliminación de elementos protegidos locales

En primer lugar, lea la sección **[Antes de comenzar](#before-you-start)** para comprender el proceso de eliminación de dependencias y almacenes.

1. En el menú del panel del almacén, seleccione **Infraestructura de copia de seguridad**.
2. Según el escenario local, elija una de las opciones siguientes:

      - En MARS, seleccione **Servidores y protegidos** y luego **Agente de Azure Backup**. A continuación, seleccione el servidor que quiere eliminar.

        ![Para MARS, seleccione el almacén para abrir su panel.](./media/backup-azure-delete-vault/identify-protected-servers.png)

      - Para MABS o DPM, seleccione **Servidores de administración de copias de seguridad**. A continuación, seleccione el servidor que quiere eliminar.

          ![Para MABS o DPM, seleccione el almacén para abrir su panel.](./media/backup-azure-delete-vault/delete-backup-management-servers.png)

3. Aparece el panel **Eliminar** con un mensaje de advertencia.

     ![El panel Eliminar.](./media/backup-azure-delete-vault/delete-protected-server.png)

     Revise el mensaje de advertencia y las instrucciones en la casilla de consentimiento.
    > [!NOTE]
    >
    >- Si el servidor protegido está sincronizado con los servicios de Azure y existen elementos de copia de seguridad, la casilla de consentimiento mostrará el número de elementos de copia de seguridad dependientes y el vínculo para ver esos elementos.
    >- Si el servidor protegido no está sincronizado con los servicios de Azure y existen elementos de copia de seguridad, la casilla de consentimiento mostrará solo el número de elementos de copia de seguridad.
    >- Si no hay elementos de copia de seguridad, la casilla de consentimiento pedirá la eliminación.

4. Seleccione la casilla de consentimiento y seleccione **Eliminar**.

5. Active el icono de **notificación**![eliminación de datos de copia de seguridad](./media/backup-azure-delete-vault/messages.png). Una vez finalizada la operación, el servicio muestra el mensaje: *Deteniendo copia de seguridad y eliminando datos de copia de seguridad para "elemento de copia de seguridad".* *La operación se completó correctamente*.
6. Seleccione **Actualizar** en el menú **Elementos de copia de seguridad** para comprobar si se ha eliminado el elemento de copia de seguridad.

>[!NOTE]
>Si elimina un elemento protegido local de un portal que contiene dependencias, recibirá una advertencia con el mensaje "La eliminación del registro del servidor es una operación destructiva y no se puede deshacer. Todos los datos de copia de seguridad (puntos de recuperación necesarios para restaurar los datos) y los elementos de copia de seguridad asociados con el servidor protegido se eliminarán permanentemente."

Una vez finalizado el proceso, puede continuar para eliminar los elementos de copia de seguridad de la consola de administración:

- [Eliminación de elementos de copia de seguridad desde la consola de administración de MARS](#delete-backup-items-from-the-mars-management-console)
- [Eliminación de elementos de copia de seguridad desde la consola de administración de MABS o DPM](#delete-backup-items-from-the-mabs-or-dpm-management-console)

### <a name="delete-backup-items-from-the-mars-management-console"></a>Eliminación de elementos de copia de seguridad desde la consola de administración de MARS

>[!NOTE]
>Si eliminó o perdió la máquina de origen sin detener la copia de seguridad, se producirá un error en la siguiente copia de seguridad programada. El punto de recuperación anterior expira según la directiva, pero el último punto de recuperación siempre se conserva hasta que se detiene la copia de seguridad y se eliminan los datos. Para ello, siga los pasos descritos en [esta sección](#delete-protected-items-on-premises).

1. Abra la consola de administración de MARS, vaya al panel **Acciones** y seleccione **Programar copia de seguridad**.
2. En la página **Modificar o detener una copia de seguridad programada**, seleccione **Dejar de utilizar esta programación de copias de seguridad y eliminar todas las copias de seguridad almacenadas**. Después, seleccione **Siguiente**.

    ![Modificación o detención de una copia de seguridad programada.](./media/backup-azure-delete-vault/modify-schedule-backup.png)

3. En la página **Detención de una copia de seguridad programada**, seleccione **Finalizar**.

    ![Detención de una copia de seguridad programada.](./media/backup-azure-delete-vault/stop-schedule-backup.png)
4. Se le pedirá que escriba un PIN (número de identificación personal) de seguridad, que debe generar manualmente. Para ello, primero inicie sesión en Azure Portal.
5. Vaya a **Almacén de Recovery Services** > **Configuración** > **Propiedades**.
6. En **PIN de seguridad**, seleccione **Generar**. Copie este código PIN. Este PIN solo es válido durante cinco minutos.
7. En la consola de administración, pegue el código PIN y seleccione **Aceptar**.

    ![Generación de un PIN de seguridad.](./media/backup-azure-delete-vault/security-pin.png)

8. En la página **Modificar progreso de la copia de seguridad**, aparece el mensaje siguiente: *Los datos de copia de seguridad eliminados se conservarán durante 14 días. Tras ese período, se eliminarán permanentemente los datos de copia de seguridad.*  

    ![Eliminación de la infraestructura de copia de seguridad.](./media/backup-azure-delete-vault/deleted-backup-data.png)

Después de eliminar los elementos de copia de seguridad locales, siga los pasos siguientes desde el portal.

### <a name="delete-backup-items-from-the-mabs-or-dpm-management-console"></a>Eliminación de elementos de copia de seguridad desde la consola de administración de MABS o DPM

>[!NOTE]
>Si eliminó o perdió la máquina de origen sin detener la copia de seguridad, se producirá un error en la siguiente copia de seguridad programada. El punto de recuperación anterior expira según la directiva, pero el último punto de recuperación siempre se conserva hasta que se detiene la copia de seguridad y se eliminan los datos. Para ello, siga los pasos descritos en [esta sección](#delete-protected-items-on-premises).

Existen dos métodos que puede usar para eliminar elementos de copia de seguridad desde la consola de administración de MABS o DPM.

#### <a name="method-1"></a>Método 1

Para detener la protección y eliminar los datos de copia de seguridad, siga estos pasos:

1. Abra la Consola de administrador DPM y luego seleccione **Protección** en la barra de navegación.
2. En el panel de información, seleccione el miembro del grupo de protección que quiere quitar. Haga clic con el botón derecho para elegir la opción **Stop Protection of Group Members** (Detener la protección de miembros del grupo).
3. Desde el cuadro de diálogo **Detener protección**, seleccione **Eliminar datos protegidos** y luego active la casilla **Eliminar almacenamiento en línea**. A continuación, seleccione **Detener protección**.

    ![Selección de Eliminar datos protegidos desde el panel Detener protección.](./media/backup-azure-delete-vault/delete-storage-online.png)

    Se cambia el estado de miembro protegido a *Réplica inactiva disponible*.

4. Haga clic en el grupo de protección inactiva y seleccione **Quitar protección inactiva**.

    ![Retirada de protección inactiva.](./media/backup-azure-delete-vault/remove-inactive-protection.png)

5. Desde la ventana **Eliminar protección inactiva**, active **Eliminar almacenamiento en línea** y seleccione **Aceptar**.

    ![Eliminación de almacenamiento en línea.](./media/backup-azure-delete-vault/remove-replica-on-disk-and-online.png)

#### <a name="method-2"></a>Método 2

Abra la consola de **administración de MABS** o **DPM**. En **Seleccionar método de protección de datos**, desactive la casilla **Deseo protección en línea**.

  ![Seleccione el método de protección de datos.](./media/backup-azure-delete-vault/data-protection-method.png)

Después de eliminar los elementos de copia de seguridad locales, siga los pasos siguientes desde el portal.

## <a name="delete-the-recovery-services-vault"></a>Eliminación del almacén de Recovery Services

1. Cuando se hayan quitado todas las dependencias, desplácese al panel **Essentials** en el menú del almacén.
2. Compruebe que no haya elementos de copia de seguridad, servidores de administración de copias de seguridad ni elementos replicados en la lista. Si los elementos siguen apareciendo en el almacén, consulte la sección [Antes de comenzar](#before-you-start).

3. Cuando no haya ningún otro elemento en el almacén, seleccione **Eliminar** en el panel del almacén.

    ![Seleccione Eliminar en el panel del almacén.](./media/backup-azure-delete-vault/vault-ready-to-delete.png)

4. Para verificar que desea eliminar el almacén, seleccione **Sí**. El almacén se elimina. El portal vuelve al menú **Nuevo** servicio.

## <a name="delete-the-recovery-services-vault-by-using-powershell"></a>Eliminación del almacén de Recovery Services mediante PowerShell

En primer lugar, lea la sección **[Antes de comenzar](#before-you-start)** para comprender el proceso de eliminación de dependencias y almacenes.

Para detener la protección y eliminar los datos de copia de seguridad:

- Si usa SQL la copia de seguridad de las máquinas virtuales de Azure y ha habilitado la protección automática para las instancias de SQL, deshabilite primero la protección automática.

    ```PowerShell
        Disable-AzRecoveryServicesBackupAutoProtection
           [-InputItem] <ProtectableItemBase>
           [-BackupManagementType] <BackupManagementType>
           [-WorkloadType] <WorkloadType>
           [-PassThru]
           [-VaultId <String>]
           [-DefaultProfile <IAzureContextContainer>]
           [-WhatIf]
           [-Confirm]
           [<CommonParameters>]
    ```

  [Más información](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackupautoprotection) sobre cómo deshabilitar la protección de los elementos protegidos de Azure Backup

- Detenga la protección y elimine los datos de todos los elementos protegidos de copia de seguridad en la nube (por ejemplo, máquina virtual IaaS, recurso compartido de archivos de Azure, etc.):

    ```PowerShell
       Disable-AzRecoveryServicesBackupProtection
       [-Item] <ItemBase>
       [-RemoveRecoveryPoints]
       [-Force]
       [-VaultId <String>]
       [-DefaultProfile <IAzureContextContainer>]
       [-WhatIf]
       [-Confirm]
       [<CommonParameters>]
    ```

    [Más información](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackupprotection) acerca de cómo deshabilitar la protección de un elemento protegido de Backup.

- En el caso de los archivos y carpetas locales protegidos mediante la copia de seguridad de Azure Backup Agent (MARS) en Azure, use el siguiente comando de PowerShell para eliminar los datos de la copia de seguridad de todos los módulos de PowerShell de MARS:

    ```powershell
    Get-OBPolicy | Remove-OBPolicy -DeleteBackup -SecurityPIN <Security Pin>
    ```

    Después, aparecerá el siguiente mensaje:

    *Microsoft Azure Backup Are you sure you want to remove this backup policy? Deleted backup data will be retained for 14 days. Después de ese tiempo se eliminarán de forma permanente. <br/> [Y] Sí  [A] Sí a todo  [N] No  [L] No a todo  [S] Suspender  [?] Ayuda (el valor predeterminado es "Y"):*

- En el caso de los equipos locales protegidos mediante MABS (Microsoft Azure Backup Server) o DPM (System Center Data Protection Manager) en Azure, use el siguiente comando para eliminar los datos de la copia de seguridad en Azure.

    ```powershell
    Get-OBPolicy | Remove-OBPolicy -DeleteBackup -SecurityPIN <Security Pin>
    ```

    Después, aparecerá el siguiente mensaje:

   *Microsoft Azure Backup* ¿Está seguro de que desea quitar esta directiva de copia de seguridad? Deleted backup data will be retained for 14 days. Tras ese período, se eliminarán permanentemente los datos de copia de seguridad. <br/>
   [Y] Sí [A] Sí a todo [N] No [L] No a todo [S] Suspender [?] Ayuda (el valor predeterminado es "Y"):*

Después de eliminar los datos de la copia de seguridad, anule el registro de los contenedores locales y de los servidores de administración.

- En el caso de los archivos y carpetas locales protegidos mediante la copia de seguridad del agente de Azure Backup (MARS) en Azure:

    ```PowerShell
    Unregister-AzRecoveryServicesBackupContainer
              [-Container] <ContainerBase>
              [-PassThru]
              [-VaultId <String>]
              [-DefaultProfile <IAzureContextContainer>]
              [-WhatIf]
              [-Confirm]
              [<CommonParameters>]
    ```

    [Más información](/powershell/module/az.recoveryservices/unregister-azrecoveryservicesbackupcontainer) sobre cómo anular el registro de un servidor de Windows Server o de otro contenedor del almacén.

- En el caso de equipos locales protegidos mediante MABS (Microsoft Azure Backup Server) o DPM en Azure (System Center Data Protection Manager):

    ```PowerShell
        Unregister-AzRecoveryServicesBackupManagementServer
          [-AzureRmBackupManagementServer] <BackupEngineBase>
          [-PassThru]
          [-VaultId <String>]
          [-DefaultProfile <IAzureContextContainer>]
          [-WhatIf]
          [-Confirm]
          [<CommonParameters>]
    ```

    [Más información](/powershell/module/az.recoveryservices/unregister-azrecoveryservicesbackupcontainer) sobre la anulación del registro de un contenedor de administración de Backup del almacén.

Después de eliminar permanentemente los datos de copia de seguridad y anular el registro de todos los contenedores, proceda a eliminar el almacén.

Para eliminar un almacén de Recovery Services:

   ```PowerShell
       Remove-AzRecoveryServicesVault
      -Vault <ARSVault>
      [-DefaultProfile <IAzureContextContainer>]
      [-WhatIf]
      [-Confirm]
      [<CommonParameters>]
   ```

[Más información](/powershell/module/az.recoveryservices/remove-azrecoveryservicesvault) sobre cómo eliminar un almacén de Recovery Services.

## <a name="delete-the-recovery-services-vault-by-using-cli"></a>Eliminación del almacén de Recovery Services mediante CLI

En primer lugar, lea la sección **[Antes de comenzar](#before-you-start)** para comprender el proceso de eliminación de dependencias y almacenes.

> [!NOTE]
> Actualmente, la CLI de Azure Backup solo admite la administración de copias de seguridad de máquinas virtuales de Azure, por lo que el siguiente comando para eliminar el almacén solo funciona si contiene copias de seguridad de máquinas virtuales de Azure. No es posible eliminar un almacén mediante la CLI de Azure Backup si el almacén contiene algún elemento de copia de seguridad que sea distinto a las máquinas virtuales de Azure.

Para eliminar el almacén de Recovery Services existente, sigas estos pasos:

- Para detener la protección y eliminar los datos de copia de seguridad

    ```azurecli
    az backup protection disable --container-name
                             --item-name
                             [--delete-backup-data {false, true}]
                             [--ids]
                             [--resource-group]
                             [--subscription]
                             [--vault-name]
                             [--yes]
    ```

    Para obtener más información, consulte este  [artículo](/cli/azure/backup/protection#az-backup-protection-disable).

- Elimine un almacén de Recovery Services existente:

    ```azurecli
    az backup vault delete [--force]
                       [--ids]
                       [--name]
                       [--resource-group]
                       [--subscription]
                       [--yes]
    ```

    Para obtener más información, consulte este  [artículo](/cli/azure/backup/vault?view=azure-cli-latest).

## <a name="delete-the-recovery-services-vault-by-using-azure-resource-manager"></a>Eliminación del almacén de Recovery Services mediante Azure Resource Manager

Esta opción para eliminar el almacén de Recovery Services solo se recomienda cuando se quitan todas las dependencias y sigue apareciendo el *error de eliminación del almacén*. Pruebe alguna de las recomendaciones siguientes, o todas ellas:

- En el panel **Información esencial** del menú del almacén, compruebe que no aparecen elementos de copia de seguridad, servidores de administración de copias de seguridad ni elementos replicados. Si hay elementos de copia de seguridad, consulte la sección [Antes de comenzar](#before-you-start).
- Vuelva a intentar [eliminar el almacén desde el portal](#delete-the-recovery-services-vault).
- Si se quitan todas las dependencias y sigue apareciendo el *error de eliminación del almacén*, use la herramienta ARMClient para realizar los pasos indicados a continuación (después de la nota).

1. Vaya a [chocolatey.org](https://chocolatey.org/) para descargar e instalar Chocolatey. Luego, instale ARMClient ejecutando el siguiente comando:

   `choco install armclient --source=https://chocolatey.org/api/v2/`
2. Inicie sesión en su cuenta de Azure y luego ejecute el comando siguiente:

    `ARMClient.exe login [environment name]`

3. En Azure Portal, recopile el id. de suscripción y el nombre del grupo de recursos del almacén que quiere eliminar.

Para obtener más información sobre el comando de ARMClient, consulte el documento [Léame de ARMClient](https://github.com/projectkudu/ARMClient/blob/master/README.md).

### <a name="use-the-azure-resource-manager-client-to-delete-a-recovery-services-vault"></a>Uso del cliente de Azure Resource Manager para eliminar el almacén de Recovery Services

1. Ejecute el comando siguiente con el identificador de suscripción, el nombre del grupo de recursos y el nombre de almacén. Si no tiene dependencias, el almacén se elimina al ejecutar el comando siguiente:

   ```azurepowershell
   ARMClient.exe delete /subscriptions/<subscriptionID>/resourceGroups/<resourcegroupname>/providers/Microsoft.RecoveryServices/vaults/<Recovery Services vault name>?api-version=2015-03-15
   ```

2. Si el almacén no está vacío, recibirá el mensaje de error siguiente: *No se puede eliminar el almacén porque contiene recursos*. Para quitar un contenedor o elementos protegidos de un almacén, ejecute el comando siguiente:

   ```azurepowershell
   ARMClient.exe delete /subscriptions/<subscriptionID>/resourceGroups/<resourcegroupname>/providers/Microsoft.RecoveryServices/vaults/<Recovery Services vault name>/registeredIdentities/<container name>?api-version=2016-06-01
   ```

3. En Azure Portal, compruebe que se ha eliminado el almacén.

## <a name="next-steps"></a>Pasos siguientes

[Más información sobre los almacenes de Recovery Services](backup-azure-recovery-services-vault-overview.md)
[Más información sobre la supervisión y administración de almacenes de Recovery Services](backup-azure-manage-windows-server.md)

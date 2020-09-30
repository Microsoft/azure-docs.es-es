---
title: Copia de seguridad de máquinas virtuales de VMware con Azure Backup Server
description: En este artículo, aprenderá a usar Azure Backup Server para realizar una copia de seguridad de las máquinas virtuales de VMware que se ejecutan en un servidor de VMWare vCenter y ESXi.
ms.topic: conceptual
ms.date: 05/24/2020
ms.openlocfilehash: e18b5c51446446103a91ef7d6a00277c2b41db77
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89017573"
---
# <a name="back-up-vmware-vms-with-azure-backup-server"></a>Copia de seguridad de máquinas virtuales de VMware con Azure Backup Server

En este artículo se explica cómo realizar copias de seguridad de máquinas virtuales de VMware que se ejecutan en hosts de VMware ESXi o vCenter Server en Azure con Azure Backup Server.

En este artículo se explica lo siguiente:

- Configurar un canal seguro para que Azure Backup Server pueda comunicarse con los servidores de VMware a través de HTTPS.
- Configurar una cuenta de VMware que Azure Backup Server usa para acceder al servidor de VMware.
- Agregar las credenciales de la cuenta Azure Backup.
- Agregar el servidor de vCenter o ESXi a Azure Backup Server.
- Configurar un grupo de protección que contenga las máquinas virtuales de VMware de las que desea realizar copias de seguridad, y especificar la configuración de copia de seguridad y programarla.

## <a name="before-you-start"></a>Antes de comenzar

- Compruebe que está ejecutando una versión de vCenter/ESXi compatible con la copia de seguridad. Consulte la matriz de compatibilidad [aquí](./backup-mabs-protection-matrix.md).
- Asegúrese de que ha configurado Azure Backup Server. En caso contrario, [hágalo](backup-azure-microsoft-azure-backup.md) antes de empezar. Se debe ejecutar Azure Backup Server con las actualizaciones más recientes.
- Asegúrese de que los siguientes puertos de red están abiertos:
  - TCP 443 entre MABS y vCenter
  - TCP 443 y TCP 902 entre MABS y el host ESXi

## <a name="create-a-secure-connection-to-the-vcenter-server"></a>Creación de una conexión segura con vCenter Server

De forma predeterminada, Azure Backup Server se comunica con los servidores de VMware a través de HTTPS. Para configurar la conexión HTTPS, descargue el certificado de la entidad de certificación (CA) de VMware e impórtelo en Azure Backup Server.

### <a name="before-you-begin"></a>Antes de empezar

- Si no quiere usar HTTPS, puede [deshabilitar la validación de certificados HTTPS para todos los servidores VMware](backup-azure-backup-server-vmware.md#disable-https-certificate-validation).
- Normalmente se usa un explorador de la máquina de Azure Backup Server para conectarse al servidor de vCenter o ESXi mediante el cliente web de vSphere. La primera vez que lo haga, la conexión no es segura y mostrará lo siguiente.
- Es importante entender cómo Azure Backup Server administra las copias de seguridad.
  - Como primer paso, Azure Backup Server realiza una copia de seguridad de los datos en el espacio de almacenamiento del disco local. Azure Backup Server usa un grupo de almacenamiento, un conjunto de discos y volúmenes en los que Azure Backup Server almacenará los datos protegidos en los puntos de recuperación del disco. El grupo de almacenamiento puede ser almacenamiento conectado directamente (DAS), una red de área de almacenamiento de canal de fibra o un dispositivo de almacenamiento o una red de área de almacenamiento iSCI. Es importante asegurarse de que tiene suficiente espacio de almacenamiento para la copia de seguridad local de los datos de la máquina virtual de VMware.
  - En ese momento, Azure Backup Server realiza la copia de seguridad del espacio de almacenamiento del disco local en Azure.
  - [Consulte](/system-center/dpm/create-dpm-protection-groups#figure-out-how-much-storage-space-you-need) cuánto espacio de almacenamiento necesita. La información es para DPM, pero también se puede usar para Azure Backup Server.

### <a name="set-up-the-certificate"></a>Configuración del certificado

Configure un canal seguro como sigue:

1. En el explorador de Azure Backup Server, escriba la dirección URL del cliente web de vSphere. Si no aparece la página de inicio de sesión, compruebe la configuración del proxy del explorador y la conexión.

    ![Cliente web de vSphere](./media/backup-azure-backup-server-vmware/vsphere-web-client.png)

2. En la página de inicio de sesión de cliente web de vSphere, seleccione **Descarga del certificado de la entidad de certificación raíz de confianza**.

    ![Descarga del certificado de la entidad de certificación raíz de confianza](./media/backup-azure-backup-server-vmware/vmware-download-ca-cert-prompt.png)

3. Se descarga un archivo denominado **download**. Según el navegador, recibirá un mensaje que le pregunta si desea abrir o guardar el archivo.

    ![Descarga del certificado de entidad de certificación](./media/backup-azure-backup-server-vmware/download-certs.png)

4. Guarde el archivo en la máquina de Azure Backup Server con extensión .zip.

5. Haga clic con el botón derecho en **download.zip** > **Extract All** (Extraer todo). El archivo .zip extrae su contenido en una carpeta **certs**, que contiene lo siguiente:
   - El archivo del certificado raíz tiene una extensión que comienza con una secuencia numerada como .0 y .1.
   - El archivo CRL tiene una extensión que comienza con una secuencia numerada como .r0 y .r1. El archivo CRL está asociado a un certificado.

    ![Certificados descargados](./media/backup-azure-backup-server-vmware/extracted-files-in-certs-folder.png)

6. En la carpeta **certs**, haga clic con el botón derecho en el archivo del certificado raíz > **Rename** (Cambiar nombre).

    ![Cambiar nombre de certificado raíz](./media/backup-azure-backup-server-vmware/rename-cert.png)

7. Cambie la extensión del certificado raíz a .crt y confirme el cambio. El icono del archivo cambia a uno que representa un certificado raíz.

8. Haga clic con el botón derecho en el certificado raíz y, en el menú emergente, seleccione **Instalar certificado**.

9. En el **Asistente para importar certificados**, seleccione **Máquina local** como destino del certificado y, después, **Siguiente**. Si se le pregunta si desea permitir cambios en el equipo, confírmelo.

    ![Bienvenida del asistente](./media/backup-azure-backup-server-vmware/certificate-import-wizard1.png)

10. En la página **Almacén de certificados**, seleccione **Colocar todos los certificados en el siguiente almacén** y seleccione **Examinar** para elegir el almacén de certificados.

    ![Almacenamiento de certificados](./media/backup-azure-backup-server-vmware/cert-import-wizard-local-store.png)

11. En **Seleccionar almacén de certificados**, seleccione **Entidades de certificación raíz de confianza** como carpeta de destino para los certificados y, después, **Aceptar**.

    ![Carpeta de destino de certificado](./media/backup-azure-backup-server-vmware/certificate-store-selected.png)

12. En **Finalización del Asistente para importar certificados**, compruebe la carpeta y seleccione **Finalizar**.

    ![Comprobar que el certificado se encuentra en la carpeta correcta](./media/backup-azure-backup-server-vmware/cert-wizard-final-screen.png)

13. Una vez confirmada la importación del certificado, inicie sesión en vCenter Server para comprobar que la conexión es segura.

### <a name="disable-https-certificate-validation"></a>Deshabilitación de la validación de certificados HTTPS

Si su organización tiene límites de seguridad y no desea usar el protocolo HTTPS entre los servidores de VMware y la máquina de Azure Backup Server, deshabilite HTTPS como sigue:

1. Copie y pegue el texto siguiente en el archivo .txt.

    ```text
    Windows Registry Editor Version 5.00
    [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft Data Protection Manager\VMWare]
    "IgnoreCertificateValidation"=dword:00000001
    ```

2. Guarde el archivo en la máquina de Azure Backup Server con el nombre **DisableSecureAuthentication.reg**.

3. Haga doble clic en el archivo para activar la entrada del Registro.

## <a name="create-a-vmware-role"></a>Creación de un rol de VMware

Azure Backup Server necesita una cuenta de usuario con permiso de acceso al host de vCenter Server y ESXi. Cree un rol de VMware con privilegios específicos y asóciele una cuenta de usuario.

1. Inicie sesión en vCenter Server (o el host de ESXi si no usa vCenter Server).
2. En el panel **Navegador**, seleccione **Administración**.

    ![Administración](./media/backup-azure-backup-server-vmware/vmware-navigator-panel.png)

3. En **Administración** > **Roles**, seleccione el icono para agregar roles (el símbolo +).

    ![Agregar rol](./media/backup-azure-backup-server-vmware/vmware-define-new-role.png)

4. En **Create Role** > **Role name** (Crear rol > Nombre de rol), escriba *BackupAdminRole*. El nombre del rol puede ser el que desee, pero debe ser reconocible para el propósito del rol.

5. Seleccione los privilegios como se resume en la tabla siguiente y, después, seleccione **Aceptar**.  El nuevo rol aparecerá en la lista del panel **Roles** (Roles).
   - Seleccione el icono junto a la etiqueta principal para expandir los privilegios primarios y ver los privilegios secundarios.
   - Para seleccionar los privilegios de la máquina virtual, debe tener varios niveles en la jerarquía primaria-secundaria.
   - No es necesario seleccionar todos los privilegios secundarios de un privilegio principal.

    ![Jerarquía de privilegios primaria-secundaria](./media/backup-azure-backup-server-vmware/cert-add-privilege-expand.png)

### <a name="role-permissions"></a>Permisos de los roles

En la tabla siguiente se capturan los privilegios que debe asignar a la cuenta de usuario que cree:

| Privilegios de las cuentas de usuario de vCenter 6.5                          | Privilegios de las cuentas de usuario de vCenter 6.7                            |
|----------------------------------------------------------------------------|----------------------------------------------------------------------------|
| Datastore cluster.Configure a datastore cluster                           | Datastore cluster.Configure a datastore cluster                           |
| Datastore.AllocateSpace                                                    | Datastore.AllocateSpace                                                    |
| Datastore.Browse datastore                                                 | Datastore.Browse datastore                                                 |
| Datastore.Low-level file operations                                        | Datastore.Low-level file operations                                        |
| Global.Disable methods                                                     | Global.Disable methods                                                     |
| Global.Enable methods                                                      | Global.Enable methods                                                      |
| Global.Licenses                                                            | Global.Licenses                                                            |
| Global.Log event                                                           | Global.Log event                                                           |
| Global.Manage custom attributes                                            | Global.Manage custom attributes                                            |
| Global.Set custom attribute                                                | Global.Set custom attribute                                                |
| Host.Local operations.Create virtual machine                               | Host.Local operations.Create virtual machine                               |
| Network.Assign network                                                     | Network.Assign network                                                     |
| Resource. Asignar máquina virtual a grupo de recursos                          | Resource. Asignar máquina virtual a grupo de recursos                          |
| vApp.Add virtual machine                                                   | vApp.Add virtual machine                                                   |
| vApp.Assign resource pool                                                  | vApp.Assign resource pool                                                  |
| vApp.Unregister                                                            | vApp.Unregister                                                            |
| VirtualMachine.Configuration. Agregar o eliminar dispositivos                         | VirtualMachine.Configuration. Agregar o eliminar dispositivos                         |
| Virtual machine.Configuration.Disk lease                                   | Virtual machine.Configuration.Acquire disk lease                           |
| Virtual machine.Configuration.Add new disk                                 | Virtual machine.Configuration.Add new disk                                 |
| Virtual machine.Configuration.Advanced                                     | Virtual machine.Configuration.Advanced configuration                       |
| Virtual machine.Configuration.Disk change tracking                         | Virtual machine.Configuration.Toggle disk change tracking                  |
| Virtual machine.Configuration.Host USB device                              | Virtual machine.Configuration.Configure Host USB device                    |
| Virtual machine.Configuration.Extend virtual disk                          | Virtual machine.Configuration.Extend virtual disk                          |
| Virtual machine.Configuration.Query unowned files                          | Virtual machine.Configuration.Query unowned files                          |
| Virtual machine.Configuration.Swapfile placement                           | Virtual machine.Configuration.Change Swapfile placement                    |
| Virtual machine.Guest Operations.Guest Operation Program Execution         | Virtual machine.Guest Operations.Guest Operation Program Execution         |
| Virtual machine.Guest Operations.Guest Operation Modifications             | Virtual machine.Guest Operations.Guest Operation Modifications             |
| Virtual machine.Guest Operations.Guest Operation Queries                   | Virtual machine.Guest Operations.Guest Operation Queries                   |
| Virtual machine .Interaction .Device connection                            | Virtual machine .Interaction .Device connection                            |
| Virtual machine .Interaction .Guest operating system management by VIX API | Virtual machine .Interaction .Guest operating system management by VIX API |
| Virtual machine .Interaction .Power Off                                    | Virtual machine .Interaction .Power Off                                    |
| Virtual machine .Inventory.Create new                                      | Virtual machine .Inventory.Create new                                      |
| Virtual machine .Inventory.Remove                                          | Virtual machine .Inventory.Remove                                          |
| Virtual machine .Inventory.Register                                        | Virtual machine .Inventory.Register                                        |
| Virtual machine .Provisioning.Allow disk access                            | Virtual machine .Provisioning.Allow disk access                            |
| Virtual machine .Provisioning.Allow file access                            | Virtual machine .Provisioning.Allow file access                            |
| Virtual machine .Provisioning.Allow read-only disk access                  | Virtual machine .Provisioning.Allow read-only disk access                  |
| Virtual machine .Provisioning.Allow virtual machine download               | Virtual machine .Provisioning.Allow virtual machine download               |
| Virtual machine .Snapshot management. Creación de instantáneas                      | Virtual machine .Snapshot management. Creación de instantáneas                      |
| Virtual machine .Snapshot management.Remove Snapshot                       | Virtual machine .Snapshot management.Remove Snapshot                       |
| Virtual machine .Snapshot management.Revert to snapshot                    | Virtual machine .Snapshot management.Revert to snapshot                    |

> [!NOTE]
> En la tabla siguiente se identifican los privilegios necesarios para vCenter 6.0 y vCenter 5.5.

| Privilegios de las cuentas de usuario de vCenter 6.0 | Privilegios de las cuentas de usuario de vCenter 5.5 |
| --- | --- |
| Datastore.AllocateSpace | Network.Assign |
| Global.Manage custom attributes | Datastore.AllocateSpace |
| Global.Set custom attribute | VirtualMachine.Config.ChangeTracking |
| Host.Local operations.Create virtual machine | VirtualMachine.State.RemoveSnapshot |
| Red. Asignar red | VirtualMachine.State.CreateSnapshot |
| Resource. Asignar máquina virtual a grupo de recursos | VirtualMachine.Provisioning.DiskRandomRead |
| Virtual machine.Configuration.Add new disk | VirtualMachine.Interact.PowerOff |
| Virtual machine.Configuration.Advanced | VirtualMachine.Inventory.Create |
| Virtual machine.Configuration.Disk change tracking | VirtualMachine.Config.AddNewDisk |
| Virtual machine.Configuration.Host USB device | VirtualMachine.Config.HostUSBDevice |
| Virtual machine.Configuration.Query unowned files | VirtualMachine.Config.AdvancedConfig |
| Virtual machine.Configuration.Swapfile placement | VirtualMachine.Config.SwapPlacement |
| Virtual machine.Interaction.Power Off | Global.ManageCustomFields |
| Virtual machine.Inventory. Crear nuevo |   |
| Virtual machine.Provisioning.Allow disk access |   |
| Virtual machine.Provisioning. Permitir acceso a disco de solo lectura |   |
| Virtual machine.Snapshot management.Create snapshot |   |
| Virtual machine.Snapshot management.Remove Snapshot |   |

## <a name="create-a-vmware-account"></a>Creación de una cuenta de VMware

1. En el panel **Navegador** de VCenter Server, seleccione **Usuarios y grupos**. Si no usa vCenter Server, cree la cuenta en el host de ESXi que corresponda.

    ![Opción Usuarios y grupos](./media/backup-azure-backup-server-vmware/vmware-userandgroup-panel.png)

    Aparece el panel **Usuarios y grupos de vCenter**.

2. En el panel **Usuarios y grupos de vCenter**, seleccione la pestaña **Usuarios** y seleccione el icono Agregar usuarios (el símbolo +).

    ![Panel Usuarios y grupos de vCenter](./media/backup-azure-backup-server-vmware/usersandgroups.png)

3. En el cuadro de diálogo **New User** (Nuevo usuario), agregue la información del usuario > **OK** (Aceptar). En este procedimiento, el nombre de usuario es BackupAdmin.

    ![Cuadro de diálogo Nuevo usuario](./media/backup-azure-backup-server-vmware/vmware-new-user-account.png)

4. Para asociar la cuenta de usuario con el rol, en el panel **Navegador**, seleccione **Permisos globales**. En el panel **Permisos globales**, seleccione la pestaña **Administrar** y, después, seleccione el icono Agregar (el símbolo +).

    ![Panel Permisos globales](./media/backup-azure-backup-server-vmware/vmware-add-new-perms.png)

5. En **Global Permission Root - Add Permission** (Raíz de permisos globales: agregar permiso), seleccione **Agregar** para elegir el usuario o grupo.

    ![Elegir un usuario o grupo](./media/backup-azure-backup-server-vmware/vmware-add-new-global-perm.png)

6. En **Select Users/Groups** (Seleccionar usuarios y grupos), elija **BackupAdmin** > **Add** (Agregar). En **Usuarios**, se usa el formato *dominio\nombre de usuario* para la cuenta de usuario. Si desea usar otro dominio, elíjalo en la lista **Dominio**. Seleccione **Aceptar** para agregar los usuarios seleccionados al cuadro de diálogo **Agregar permiso**.

    ![Agregar usuario BackupAdmin](./media/backup-azure-backup-server-vmware/vmware-assign-account-to-role.png)

7. En el área **Assigned Role** (Rol asignado), en el menú desplegable, seleccione **BackupAdminRole** > **OK** (Aceptar).

    ![Asignar el usuario al rol](./media/backup-azure-backup-server-vmware/vmware-choose-role.png)

En la pestaña **Administrar** del panel **Permisos globales**, la nueva cuenta de usuario y el rol asociado aparecen en la lista.

## <a name="add-the-account-on-azure-backup-server"></a>Incorporación de la cuenta a Azure Backup Server

1. Abra Azure Backup Server. Si no encuentra el icono en el escritorio, abra Microsoft Azure Backup a partir de la lista de aplicaciones.

    ![Icono de Azure Backup Server](./media/backup-azure-backup-server-vmware/mabs-icon.png)

2. En la consola de Azure Backup Server, seleccione **Administración** >  **Servidores de producción** > **Administrar VMware**.

    ![Consola de Azure Backup Server](./media/backup-azure-backup-server-vmware/add-vmware-credentials.png)

3. En el cuadro de diálogo **Administrar credenciales**, seleccione **Agregar**.

    ![Administrar credenciales (cuadro de diálogo)](./media/backup-azure-backup-server-vmware/mabs-manage-credentials-dialog.png)

4. En **Agregar credencial**, escriba un nombre y una descripción y especifique el nombre de usuario y la contraseña que se definieron en el servidor de VMware. El nombre, *Contoso Vcenter credential*, sirve para identificar las credenciales en este procedimiento. Si el servidor de VMware y Azure Backup Server no están en el mismo dominio, especifique el dominio en el nombre de usuario.

    ![Cuadro de diálogo Agregar credenciales de Azure Backup Server](./media/backup-azure-backup-server-vmware/mabs-add-credential-dialog2.png)

5. Haga clic en **Agregar** para agregar la nueva credencial.

    ![Agregar nuevas credenciales](./media/backup-azure-backup-server-vmware/new-list-of-mabs-creds.png)

## <a name="add-the-vcenter-server"></a>Incorporación de vCenter Server

Agregue vCenter Server a Azure Backup Server.

1. En la consola de Azure Backup Server, seleccione **Administración** > **Servidores de producción** > **Agregar**.

    ![Abrir el Asistente para adición del servidor de producción](./media/backup-azure-backup-server-vmware/add-vcenter-to-mabs.png)

2. En la página **Asistente de adición del servidor de producción** > **Seleccionar tipo de servidor de producción**, seleccione **Servidores de VMware** y, después, **Siguiente**.

    ![Asistente para adición del servidor de producción](./media/backup-azure-backup-server-vmware/production-server-add-wizard.png)

3. En **Seleccionar equipos**  **Nombre o dirección IP del servidor**, especifique el nombre de dominio completo o la dirección IP del servidor de VMware. Si todos los servidores de ESXi están administrados por la misma instancia de vCenter, especifique su nombre. En caso contrario, agregue el host de ESXi.

    ![Especificación del servidor de VMware](./media/backup-azure-backup-server-vmware/add-vmware-server-provide-server-name.png)

4. En **SSL Port** (Puerto SSL), especifique el puerto usado para la comunicación con el servidor de VMware. 443 es el puerto predeterminado, pero puede cambiarlo si el servidor de VMware escucha en un puerto diferente.

5. En **Especificar credenciales**, seleccione la credencial que ha creado antes.

    ![Especificar credenciales](./media/backup-azure-backup-server-vmware/identify-creds.png)

6. Seleccione **Agregar** para agregar el servidor de VMware a la lista de servidores. Luego, seleccione **Siguiente**.

    ![Agregar credencial y el servidor de VMWare](./media/backup-azure-backup-server-vmware/add-vmware-server-credentials.png)

7. En la página **Resumen**, seleccione **Agregar** para agregar el servidor de VMware a Azure Backup Server. El nuevo servidor se agrega inmediatamente, no se necesita ningún agente en el servidor de VMware.

    ![Agregar el servidor de VMware a Azure Backup Server](./media/backup-azure-backup-server-vmware/tasks-screen.png)

8. Compruebe la configuración en la página **Finish** (Finalizar).

   ![Página Finalizar](./media/backup-azure-backup-server-vmware/summary-screen.png)

Si tiene varios hosts de ESXi que no están administrados por vCenter Server o si tiene varias instancias de vCenter Server, deberá volver a ejecutar el asistente para agregar los servidores.

## <a name="configure-a-protection-group"></a>Configuración de un grupo de protección

Agregue máquinas virtuales de VMware para la copia de seguridad. Los grupos de protección recopilan varias máquinas virtuales y aplican la misma configuración de copia de seguridad y de retención de datos para todas las máquinas virtuales del grupo.

1. En la consola de Azure Backup Server, seleccione **Protección** > **Nuevo**.

    ![Abrir el asistente Crear nuevo grupo de protección.](./media/backup-azure-backup-server-vmware/open-protection-wizard.png)

1. En la página principal del asistente **Crear grupo de protección**, seleccione **Siguiente**.

    ![Cuadro de diálogo del asistente Crear nuevo grupo de protección](./media/backup-azure-backup-server-vmware/protection-wizard.png)

1. En la página **Seleccionar tipo de grupo de protección**, seleccione **Servidores** y, después, **Siguiente**. Se muestra la página **Seleccionar miembros del grupo**.

1. En **Seleccionar miembros del grupo** > seleccione las máquinas virtuales (o carpetas de la máquina virtual) de las que desea realizar copias de seguridad. Luego, seleccione **Siguiente**.

    - Al seleccionar una carpeta, las máquinas virtuales o carpetas dentro de esa carpeta también se seleccionan para la copia de seguridad. Puede desactivar las carpetas o máquinas virtuales de las que no desee copia de seguridad.
1. Si ya se está realizando la copia de seguridad de una máquina virtual o carpeta, estas no se pueden seleccionar. De este moro se garantiza que no se crean puntos de recuperación duplicados para una máquina virtual.

    ![Seleccionar a miembros del grupo](./media/backup-azure-backup-server-vmware/server-add-selected-members.png)

1. En la página **Select Data Protection Method** (Seleccionar método de protección de datos), introduzca un nombre para el grupo de protección y la configuración. Para volver a Azure, configure la protección a corto plazo en **Disk** (Disco) y habilite la protección en línea. Luego, seleccione **Siguiente**.

    ![Seleccionar método de protección de datos](./media/backup-azure-backup-server-vmware/name-protection-group.png)

1. En **Specify Short-Term Goals** (Especificar objetivos a corto plazo), especifique cuánto tiempo desea mantener la copia de seguridad de los datos en el disco.
   - En **Retention Range** (Duración de retención), especifique cuántos días deben mantenerse los puntos de recuperación de disco.
   - En **Synchronization frequency** (Frecuencia de sincronización), especifique con qué frecuencia se crearán los puntos de recuperación de disco.
       - Si no desea establecer un intervalo de copia de seguridad, puede marcar **Solo antes de un punto de recuperación** para que se ejecute una copia de seguridad solo antes de la creación de cada punto de recuperación.
       - Las copias de seguridad a corto plazo son copias de seguridad completas y no incrementales.
       - Seleccione **Modificar** para cambiar las fechas/horas con las copias de seguridad a corto plazo.

         ![Especificar objetivos a corto plazo](./media/backup-azure-backup-server-vmware/short-term-goals.png)

1. En **Review Disk Allocation** (Revisar asignación de disco), revise el espacio en disco para las copias de seguridad de las máquinas virtuales.

   - Las asignaciones de disco recomendadas se basan en la duración de retención especificada, el tipo de carga de trabajo y el tamaño de los datos protegidos. Realice los cambios necesarios y seleccione **Siguiente**.
   - **Tamaño de los datos:** tamaño de los datos del grupo de protección.
   - **Espacio en disco:** espacio en disco recomendado para el grupo de protección. Si quiere modificar esta configuración, debe asignar un espacio total que sea ligeramente mayor que la cantidad que calcula que va a crecer cada origen de datos.
   - **Ubicación compartida de datos:** si activa la ubicación compartida, varios orígenes de datos de la protección pueden asignarse a una sola réplica y a un volumen de puntos de recuperación. La ubicación compartida no es compatible con todas las cargas de trabajo.
   - **Expandir automáticamente:** al habilitar esta configuración, si los datos del grupo protegido sobrepasan la asignación inicial, Azure Backup Server intenta aumentar el tamaño del disco en un 25 %.
   - **Detalles del grupo de almacenamiento:** muestra el estado actual del grupo de almacenamiento, incluido el tamaño total y restante del disco.

    ![Revisar la asignación de disco](./media/backup-azure-backup-server-vmware/review-disk-allocation.png)

1. En la página **Seleccionar método de creación de réplicas**, especifique cómo desea realizar la copia de seguridad inicial y, después, seleccione **Siguiente**.
   - El valor predeterminado es **Automáticamente a través de la red** y **Ahora**.
   - Si usa el valor predeterminado, recomendamos que especifique una hora de poco tráfico. Elija **Más tarde** y especifique un día y una hora.
   - Para grandes cantidades de datos o condiciones de red no del todo óptimas, considere la posibilidad de replicar los datos sin conexión con medios extraíbles.

    ![Elegir método de creación de réplica](./media/backup-azure-backup-server-vmware/replica-creation.png)

1. En **Consistency Check Options** (Opciones de comprobación de coherencia), seleccione cómo y cuándo automatizar las comprobaciones de coherencia. Luego, seleccione **Siguiente**.
      - Puede ejecutar comprobaciones de coherencia si los datos de réplica no son coherentes o en una programación establecida.
      - Si no desea configurar la comprobación de coherencia automática, puede ejecutar una comprobación manual. Para ello, haga clic con el botón derecho en el grupo de protección > **Perform Consistency Check** (Realizar comprobación de coherencia).

1. En la página **Specify Online Protection Data** (Especificar datos de protección en línea), seleccione las máquinas virtuales o las carpetas de máquina virtual de las que desee realizar copias de seguridad. Puede seleccionar los miembros individualmente o hacer clic en **Seleccionar todo** para elegir todos los miembros. Luego, seleccione **Siguiente**.

    ![Especificar datos de protección en línea](./media/backup-azure-backup-server-vmware/select-data-to-protect.png)

1. En la página **Specify Online Backup Schedule** (Especificar la programación de copia de seguridad en línea), especifique con qué frecuencia desea realizar la copia de seguridad de los datos del espacio de almacenamiento local en Azure.

    - Se generarán puntos de recuperación de los datos en la nube según la programación. Luego, seleccione **Siguiente**.
    - Después de generar el punto de recuperación, se transfiere al almacén de Recovery Services de Azure.

    ![Especificar programación de copia de seguridad en línea](./media/backup-azure-backup-server-vmware/online-backup-schedule.png)

1. En la página **Specify Online Retention Policy** (Especificar la directiva de retención en línea), indique el tiempo de conservación de los puntos de recuperación que se crean de las copias de seguridad diarias, semanales, mensuales y anuales en Azure Después, seleccione **Siguiente**.

    - Los datos pueden guardarse en Azure sin límite de tiempo.
    - El único límite es que no se pueden tener más de 9999 puntos de recuperación por instancia protegida. En este ejemplo, la instancia protegida es el servidor de VMware.

    ![Especificar directiva de retención en línea](./media/backup-azure-backup-server-vmware/retention-policy.png)

1. En la página **Resumen**, revise la configuración y seleccione **Crear grupo**.

    ![Resumen de configuración y miembros del grupo de protección](./media/backup-azure-backup-server-vmware/protection-group-summary.png)

## <a name="vmware-parallel-backups"></a>Copias de seguridad paralelas de VMware

>[!NOTE]
> Esta característica es aplicable a MABS V3 UR1.

Con las versiones anteriores de MABS, las copias de seguridad paralelas se realizaban solo entre grupos de protección. Con MABS V3 UR1, todas las copias de seguridad de máquinas virtuales VMWare de un solo grupo de protección serán paralelas, lo que agiliza las copias de seguridad de máquinas virtuales. Todos los trabajos de replicación diferencial de VMWare se ejecutan en paralelo. De forma predeterminada, el número de trabajos que se ejecuta en paralelo está establecido en ocho.

Para modificar el número de trabajos, utilice la clave del Registro como se muestra a continuación (no está presente de forma predeterminada, por lo que es preciso agregarla):

**Ruta de acceso de la clave:** : `Software\Microsoft\Microsoft Data Protection Manager\Configuration\ MaxParallelIncrementalJobs\VMWare`<BR>
**Tipo de clave**: Valor de DWORD (32 bits).

> [!NOTE]
> Puede modificar el número de trabajos a un valor mayor. Si establece el número de trabajos en 1, los trabajos de replicación se ejecutan en serie. Para aumentar el número, debe tener en cuenta el rendimiento de VMWare. Teniendo en cuenta el número de recursos que se utilizan y el uso adicional que se requiere en el servidor de VMWare vSphere, debe determinar el número de trabajos de replicación diferencial que se ejecutan en paralelo. Además, este cambio solo afectará a los grupos de protección recién creados. Para los grupos de protección existentes, debe agregar temporalmente otra máquina virtual al grupo de protección. De este modo, se debería actualizar la configuración del grupo de protección en consecuencia. Puede quitar esta máquina virtual del grupo de protección después de completar el procedimiento.

## <a name="vmware-vsphere-67"></a>VMWare vSphere 6.7

Para realizar una copia de seguridad de vSphere 6.7, siga estos pasos:

- Habilite TLS 1.2 en el servidor DPM.

>[!NOTE]
>En VMWare 6.7 y versiones posteriores se ha habilitado TLS como protocolo de comunicación.

- Establezca las claves del Registro de esta forma:

```text
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v2.0.50727]
"SystemDefaultTlsVersions"=dword:00000001
"SchUseStrongCrypto"=dword:00000001

[HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319]
"SystemDefaultTlsVersions"=dword:00000001
"SchUseStrongCrypto"=dword:00000001

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v2.0.50727]
"SystemDefaultTlsVersions"=dword:00000001
"SchUseStrongCrypto"=dword:00000001

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319]
"SystemDefaultTlsVersions"=dword:00000001
"SchUseStrongCrypto"=dword:00000001
```

## <a name="exclude-disk-from-vmware-vm-backup"></a>Exclusión del disco de la copia de seguridad de máquina virtual de VMware

> [!NOTE]
> Esta característica es aplicable a MABS V3 UR1.

Con MABS V3 UR1, puede excluir discos específicos de una copia de seguridad de máquina virtual de VMware. El script de configuración **ExcludeDisk.ps1** se encuentra en la `C:\Program Files\Microsoft Azure Backup Server\DPM\DPM\bin folder`.

Para configurar la exclusión de disco, siga estos pasos:

### <a name="identify-the-vmware-vm-and-disk-details-to-be-excluded"></a>Identificación de la máquina virtual de VMware y los detalles del disco que se van a excluir

  1. En la consola de VMware, vaya a la configuración de la máquina virtual para la que desee excluir el disco.
  2. Seleccione el disco que desee excluir y anote su ruta de acceso.

        Por ejemplo, para excluir el disco duro 2 de TestVM4, la ruta de acceso del disco duro 2 es **[datastore1] TestVM4/TestVM4\_1.vmdk**.

        ![Disco duro que se va a excluir](./media/backup-azure-backup-server-vmware/test-vm.png)

### <a name="configure-mabs-server"></a>Configuración del servidor de MABS

Vaya al servidor de MABS donde la máquina virtual de VMware esté configurada para la protección y configure la exclusión de disco.

  1. Obtenga los detalles del host de VMware que está protegido en el servidor de MABS.

        ```powershell
        $psInfo = get-DPMProductionServer
        $psInfo
        ```

        ```output
        ServerName   ClusterName     Domain            ServerProtectionState
        ----------   -----------     ------            ---------------------
        Vcentervm1                   Contoso.COM       NoDatasourcesProtected
        ```

  2. Seleccione el host de VMware y enumere la protección de las máquinas virtuales para él.

        ```powershell
        $vmDsInfo = get-DPMDatasource -ProductionServer $psInfo[0] -Inquire
        $vmDsInfo
        ```

        ```output
        Computer     Name     ObjectType
        --------     ----     ----------
        Vcentervm1  TestVM2      VMware
        Vcentervm1  TestVM1      VMware
        Vcentervm1  TestVM4      VMware
        ```

  3. Seleccione la máquina virtual para la que desee excluir un disco.

        ```powershell
        $vmDsInfo[2]
        ```

        ```output
        Computer     Name      ObjectType
        --------     ----      ----------
        Vcentervm1   TestVM4   VMware
        ```

  4. Para excluir el disco, vaya a la carpeta `Bin` y ejecute el script *ExcludeDisk.ps1* con los parámetros siguientes:

        > [!NOTE]
        > Antes de ejecutar este comando, detenga el servicio DPMRA en el servidor de MABS. De lo contrario, el script devuelve un mensaje que indica que se ejecutó correctamente, pero no actualiza la lista de exclusión. Asegúrese de que no haya trabajos en curso antes de detener el servicio.

     **Para agregar o quitar el disco de la exclusión, ejecute el siguiente comando:**

      ```powershell
      ./ExcludeDisk.ps1 -Datasource $vmDsInfo[0] [-Add|Remove] "[Datastore] vmdk/vmdk.vmdk"
      ```

     **Ejemplo**:

     para agregar la exclusión de disco para TestVM4, ejecute el siguiente comando:

       ```powershell
      C:\Program Files\Microsoft Azure Backup Server\DPM\DPM\bin> ./ExcludeDisk.ps1 -Datasource $vmDsInfo[2] -Add "[datastore1] TestVM4/TestVM4\_1.vmdk"
       ```

      ```output
       Creating C:\Program Files\Microsoft Azure Backup Server\DPM\DPM\bin\excludedisk.xml
       Disk : [datastore1] TestVM4/TestVM4\_1.vmdk, has been added to disk exclusion list.
      ```

  5. Compruebe que el disco se ha agregado para excluirlo.

     **Para ver la exclusión existente de máquinas virtuales específicas, ejecute el siguiente comando:**

        ```powershell
        ./ExcludeDisk.ps1 -Datasource $vmDsInfo[0] [-view]
        ```

     **Ejemplo**

        ```powershell
        C:\Program Files\Microsoft Azure Backup Server\DPM\DPM\bin> ./ExcludeDisk.ps1 -Datasource $vmDsInfo[2] -view
        ```

        ```output
        <VirtualMachine>
        <UUID>52b2b1b6-5a74-1359-a0a5-1c3627c7b96a</UUID>
        <ExcludeDisk>[datastore1] TestVM4/TestVM4\_1.vmdk</ExcludeDisk>
        </VirtualMachine>
        ```

     Una vez que configure la protección para esta máquina virtual, el disco excluido no aparecerá en la lista durante la protección.

        > [!NOTE]
        > Si va a realizar estos pasos para una máquina virtual ya protegida, debe ejecutar la comprobación de coherencia manualmente después de agregar el disco para excluirlo.

### <a name="remove-the-disk-from-exclusion"></a>Eliminación del disco de la exclusión

Para quitar el disco de la exclusión, ejecute el siguiente comando:

```powershell
C:\Program Files\Microsoft Azure Backup Server\DPM\DPM\bin> ./ExcludeDisk.ps1 -Datasource $vmDsInfo[2] -Remove "[datastore1] TestVM4/TestVM4\_1.vmdk"
```

## <a name="next-steps"></a>Pasos siguientes

Para solucionar problemas de configuración de las copias de seguridad, revise la [guía de solución de problemas de Azure Backup Server](./backup-azure-mabs-troubleshoot.md).

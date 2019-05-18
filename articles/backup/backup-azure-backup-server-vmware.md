---
title: Copia de seguridad de máquinas virtuales de VMware con Azure Backup Server
description: Use Azure Backup Server para realizar una copia de seguridad de las máquinas virtuales de VMware que se ejecutan en un servidor de VMWare vCenter y ESXi.
services: backup
author: rayne-wiselman
manager: carmonm
ms.service: backup
ms.topic: conceptual
ms.date: 12/11/2018
ms.author: raynew
ms.openlocfilehash: f034f31f2c8c49bbdfb88e2ba0a009ff5b795fa2
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65789604"
---
# <a name="back-up-vmware-vms-with-azure-backup-server"></a>Copia de seguridad de máquinas virtuales de VMware con Azure Backup Server

En este artículo se explica cómo realizar copias de seguridad de máquinas virtuales de VMware que se ejecutan en hosts de VMware ESXi o vCenter Server en Azure con Azure Backup Server.

En este artículo se explica cómo:

- Configurar un canal seguro para que Azure Backup Server pueda comunicarse con los servidores de VMware a través de HTTPS.
- Configurar una cuenta de VMware que Azure Backup Server usa para acceder al servidor de VMware.
- Agregar las credenciales de la cuenta Azure Backup.
- Agregar el servidor de vCenter o ESXi a Azure Backup Server.
- Configurar un grupo de protección que contenga las máquinas virtuales de VMware de las que desea realizar copias de seguridad, y especificar la configuración de copia de seguridad y programarla.

## <a name="before-you-start"></a>Antes de comenzar
- Compruebe que está ejecutando una versión de vCenter/ESXi compatible con la copia de seguridad, estas son la 6.5, la 6.0 y la 5.5.
- Asegúrese de que ha configurado Azure Backup Server. En caso contrario, [hágalo](backup-azure-microsoft-azure-backup.md) antes de empezar. Se debe ejecutar Azure Backup Server con las actualizaciones más recientes.


## <a name="create-a-secure-connection-to-the-vcenter-server"></a>Creación de una conexión segura con vCenter Server

De forma predeterminada, Azure Backup Server se comunica con los servidores de VMware a través de HTTPS. Para configurar la conexión HTTPS, descargue el certificado de la entidad de certificación (CA) de VMware e impórtelo en Azure Backup Server.


### <a name="before-you-start"></a>Antes de comenzar

- Si no desea usar HTTPS puede [deshabilitar la validación de certificados HTTPS para todos los servidores VMware](backup-azure-backup-server-vmware.md#disable-https-certificate-validation).
- Normalmente se usa un explorador de la máquina de Azure Backup Server para conectarse al servidor de vCenter o ESXi mediante el cliente web de vSphere. La primera vez que lo haga, la conexión no es segura y se mostrará lo siguiente.
- Es importante entender cómo Azure Backup Server administra las copias de seguridad.
    - Como primer paso, Azure Backup Server realiza una copia de seguridad de los datos en el espacio de almacenamiento del disco local. Azure Backup Server usa un grupo de almacenamiento, un conjunto de discos y volúmenes en los que Azure Backup Server almacenará los datos protegidos en los puntos de recuperación del disco. El grupo de almacenamiento puede ser almacenamiento conectado directamente (DAS), una red de área de almacenamiento de canal de fibra o un dispositivo de almacenamiento o una red de área de almacenamiento iSCI. Es importante asegurarse de que tiene suficiente espacio de almacenamiento para la copia de seguridad local de los datos de la máquina virtual de VMware.
    - En ese momento, Azure Backup Server realiza la copia de seguridad del espacio de almacenamiento del disco local en Azure.
    - [Consulte](https://docs.microsoft.com/system-center/dpm/create-dpm-protection-groups?view=sc-dpm-1807#figure-out-how-much-storage-space-you-need) cuánto espacio de almacenamiento necesita. La información es para DPM, pero también se puede usar para Azure Backup Server.

### <a name="set-up-the-certificate"></a>Configuración del certificado

Configure un canal seguro como sigue:

1. En el explorador de Azure Backup Server, escriba la dirección URL del cliente web de vSphere. Si no aparece la página de inicio de sesión, compruebe la configuración del proxy del explorador y la conexión.

    ![Cliente web de vSphere](./media/backup-azure-backup-server-vmware/vsphere-web-client.png)

2. En la página de inicio de sesión de cliente web de vSphere, haga clic en **Download trusted root CA certificates** (Descargar certificados de la entidad de certificación raíz de confianza).

    ![Descarga del certificado de la entidad de certificación raíz de confianza](./media/backup-azure-backup-server-vmware/vmware-download-ca-cert-prompt.png)

3. Se descarga un archivo denominado **download**. Según el navegador, recibirá un mensaje que le pregunta si desea abrir o guardar el archivo.

    ![Descarga del certificado de entidad de certificación](./media/backup-azure-backup-server-vmware/download-certs.png)

4. Guarde el archivo en la máquina de Azure Backup Server con extensión .zip.

5. Haga clic con el botón derecho en **download.zip** > **Extract All** (Extraer todo). El archivo .zip extrae su contenido en una carpeta **certs**, que contiene lo siguiente:
   - El archivo del certificado raíz tiene una extensión que comienza con una secuencia numerada como .0 y .1.
   - El archivo CRL tiene una extensión que comienza con una secuencia numerada como .r0 y .r1. El archivo CRL está asociado a un certificado.

     ![Certificados descargados](./media/backup-azure-backup-server-vmware/extracted-files-in-certs-folder.png)

5. En la carpeta **certs**, haga clic con el botón derecho en el archivo del certificado raíz > **Rename** (Cambiar nombre).

    ![Cambiar nombre de certificado raíz](./media/backup-azure-backup-server-vmware/rename-cert.png)

6. Cambie la extensión del certificado raíz a .crt y confirme el cambio. El icono del archivo cambia a uno que representa un certificado raíz.

7. Haga clic con el botón derecho en el certificado raíz y, en el menú emergente, seleccione **Instalar certificado**.

8. En el **Asistente para importar certificados**, seleccione **Máquina local** como destino del certificado y haga clic en **Siguiente**. Si se le pregunta si desea permitir cambios en el equipo, confírmelo.

    ![Bienvenida del asistente](./media/backup-azure-backup-server-vmware/certificate-import-wizard1.png)


9. En la página **Almacén de certificados**, seleccione **Colocar todos los certificados en el siguiente almacén** y haga clic en **Examinar** para elegir el almacén de certificados.

     ![Almacenamiento de certificados](./media/backup-azure-backup-server-vmware/cert-import-wizard-local-store.png)

10. En **Seleccionar almacén de certificados**, seleccione **Entidades de certificación raíz de confianza** como carpeta de destino para los certificados y haga clic en **Aceptar**.

    ![Carpeta de destino de certificado](./media/backup-azure-backup-server-vmware/certificate-store-selected.png)

11. En **Finalización del Asistente para importar certificados**, compruebe la carpeta y haga clic en **Finalizar**.

    ![Comprobar que el certificado se encuentra en la carpeta correcta](./media/backup-azure-backup-server-vmware/cert-wizard-final-screen.png)


12. Una vez confirmada la importación del certificado, inicie sesión en vCenter Server para comprobar que la conexión es segura.




### <a name="disable-https-certificate-validation"></a>Deshabilitar la validación de certificados HTTPS

Si su organización tiene límites de seguridad y no desea usar el protocolo HTTPS entre los servidores de VMware y la máquina de Azure Backup Server, deshabilite HTTPS como sigue:
1. Copie y pegue el texto siguiente en el archivo .txt.

      ```
      Windows Registry Editor Version 5.00
      [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft Data Protection Manager\VMWare]
      "IgnoreCertificateValidation"=dword:00000001
      ```

2. Guarde el archivo en la máquina de Azure Backup Server con el nombre **DisableSecureAuthentication.reg**.

3. Haga doble clic en el archivo para activar la entrada del Registro.


## <a name="create-a-vmware-role"></a>Creación de un rol de VMware

Azure Backup Server necesita una cuenta de usuario con permiso de acceso al host de vCenter Server y ESXi. Cree un rol de VMware con privilegios específicos y asóciele una cuenta de usuario.

1. Inicie sesión en vCenter Server (o el host de ESXi si no usa vCenter Server).
2. En el panel **Navigator** (Navegador), haga clic en **Administration** (Administración).

    ![Administración](./media/backup-azure-backup-server-vmware/vmware-navigator-panel.png)

3. En **Administration** > **Roles** (Administración > Roles), haga clic en el icono para agregar rol (el símbolo +).

    ![Agregar rol](./media/backup-azure-backup-server-vmware/vmware-define-new-role.png)


4. En **Create Role** > **Role name** (Crear rol > Nombre de rol), escriba *BackupAdminRole*. El nombre del rol puede ser el que desee, pero debe ser reconocible para el propósito del rol.

5. Seleccione los privilegios como se resume en la tabla siguiente y haga clic en **OK** (Aceptar).  El nuevo rol aparecerá en la lista del panel **Roles** (Roles).
   - Haga clic en el icono junto a la etiqueta principal para expandir los privilegios primarios y ver los privilegios secundarios.
   - Para seleccionar los privilegios de la máquina virtual, debe tener varios niveles en la jerarquía primaria-secundaria.
   - No es necesario seleccionar todos los privilegios secundarios de un privilegio principal.

     ![Jerarquía de privilegios primaria-secundaria](./media/backup-azure-backup-server-vmware/cert-add-privilege-expand.png)

### <a name="role-permissions"></a>Permisos de roles
**6.5/6.0** | **5.5**
--- | ---
Datastore.AllocateSpace | Datastore.AllocateSpace
Global.ManageCustomFields | Global.ManageCustomFields
Global.SetCustomField |
Host.Local.CreateVM | Network.Assign
Network.Assign |
Resource.AssignVMToPool |
VirtualMachine.Config.AddNewDisk  | VirtualMachine.Config.AddNewDisk   
VirtualMachine.Config.AdvancedConfig| VirtualMachine.Config.AdvancedConfig
VirtualMachine.Config.ChangeTracking| VirtualMachine.Config.ChangeTracking
VirtualMachine.Config.HostUSBDevice |
VirtualMachine.Config.QueryUnownedFiles |
VirtualMachine.Config.SwapPlacement| VirtualMachine.Config.SwapPlacement
VirtualMachine.Interact.PowerOff| VirtualMachine.Interact.PowerOff
VirtualMachine.Inventory.Create| VirtualMachine.Inventory.Create
VirtualMachine.Provisioning.DiskRandomAccess |
VirtualMachine.Provisioning.DiskRandomRead | VirtualMachine.Provisioning.DiskRandomRead
VirtualMachine.State.CreateSnapshot | VirtualMachine.State.CreateSnapshot
VirtualMachine.State.RemoveSnapshot | VirtualMachine.State.RemoveSnapshot




## <a name="create-a-vmware-account"></a>Creación de una cuenta de VMware

1. En el panel **Navigator** (Navegador) de vCenter Server, haga clic en **Users and Groups** (Usuarios y grupos). Si no usa vCenter Server, cree la cuenta en el host de ESXi que corresponda.

    ![Opción Usuarios y grupos](./media/backup-azure-backup-server-vmware/vmware-userandgroup-panel.png)

    El **vCenter usuarios y grupos** aparecen el panel.


2. En el panel **Usuarios y grupos** de vCenter, seleccione la pestaña **Usuarios** y haga clic en el icono Agregar usuarios (el símbolo +).

     ![Panel Usuarios y grupos de vCenter](./media/backup-azure-backup-server-vmware/usersandgroups.png)


3. En el cuadro de diálogo **New User** (Nuevo usuario), agregue la información del usuario > **OK** (Aceptar). En este procedimiento, el nombre de usuario es BackupAdmin.

    ![Cuadro de diálogo Nuevo usuario](./media/backup-azure-backup-server-vmware/vmware-new-user-account.png)


4. Para asociar la cuenta de usuario con el rol, en el panel **Navegador**, haga clic en **Permisos globales**. En el panel **Permisos globales**, seleccione la pestaña **Administrar** y, después, haga clic en el icono Agregar (el símbolo +).

    ![Panel Permisos globales](./media/backup-azure-backup-server-vmware/vmware-add-new-perms.png)


5. En **Global Permission Root - Add Permission** (Raíz de permisos globales: agregar permiso), haga clic en **Add** (Agregar) para elegir el usuario o el grupo.

    ![Elegir un usuario o grupo](./media/backup-azure-backup-server-vmware/vmware-add-new-global-perm.png)

6. En **Select Users/Groups** (Seleccionar usuarios y grupos), elija **BackupAdmin** > **Add** (Agregar). En **Usuarios**, se usa el formato *dominio\nombre de usuario* para la cuenta de usuario. Si desea usar otro dominio, elíjalo en la lista **Dominio**. Haga clic en **Aceptar** para agregar los usuarios seleccionados al cuadro de diálogo **Agregar permiso**.

    ![Agregar usuario BackupAdmin](./media/backup-azure-backup-server-vmware/vmware-assign-account-to-role.png)


7.  En el área **Assigned Role** (Rol asignado), en el menú desplegable, seleccione **BackupAdminRole** > **OK** (Aceptar).

    ![Asignar el usuario al rol](./media/backup-azure-backup-server-vmware/vmware-choose-role.png)


En la pestaña **Administrar** del panel **Permisos globales**, la nueva cuenta de usuario y el rol asociado aparecen en la lista.


## <a name="add-the-account-on-azure-backup-server"></a>Incorporación de la cuenta a Azure Backup Server


1. Abra Azure Backup Server. Si no encuentra el icono en el escritorio, abra Microsoft Azure Backup a partir de la lista de aplicaciones.

    ![Icono de Azure Backup Server](./media/backup-azure-backup-server-vmware/mabs-icon.png)

2. En la consola de Azure Backup Server, haga clic en **Management** >  **Production Servers** > **Manage VMware** (Administración > Servidores de producción > Administrar VMware).

    ![Consola de Azure Backup Server](./media/backup-azure-backup-server-vmware/add-vmware-credentials.png)


3. En el cuadro de diálogo **Manage Credentials** (Administrar credenciales), haga clic en **Add** (Agregar).

    ![Cuadro de diálogo Administrar credenciales de Azure Backup Server](./media/backup-azure-backup-server-vmware/mabs-manage-credentials-dialog.png)

4. En **Add Credential** (Agregar credenciales), escriba un nombre y una descripción y especifique el nombre de usuario y la contraseña que se definieron en el servidor de VMware. El nombre, *Contoso Vcenter credential*, sirve para identificar las credenciales en este procedimiento. Si el servidor de VMware y Azure Backup Server no están en el mismo dominio, especifique el dominio en el nombre de usuario.

    ![Cuadro de diálogo Agregar credenciales de Azure Backup Server](./media/backup-azure-backup-server-vmware/mabs-add-credential-dialog2.png)

5. Haga clic en **Add** (Agregar) para agregar las nuevas credenciales.

    ![Cuadro de diálogo Administrar credenciales de Azure Backup Server](./media/backup-azure-backup-server-vmware/new-list-of-mabs-creds.png)


## <a name="add-the-vcenter-server"></a>Incorporación de vCenter Server

Agregue vCenter Server a Azure Backup Server.


1. En la consola de Azure Backup Server, haga clic en **Management** > **Production Servers** > **Add** (Administración > Servidores de producción > Agregar).

    ![Abrir el Asistente para adición del servidor de producción](./media/backup-azure-backup-server-vmware/add-vcenter-to-mabs.png)


2. En la página **Production Server Addition Wizard** (Asistente para incorporación del servidor de producción) > **Select Production Server type** (Seleccionar tipo de servidor de producción), seleccione **VMware Servers** (Servidores de VMware) y haga clic en **Next** (Siguiente).

     ![Asistente para adición del servidor de producción](./media/backup-azure-backup-server-vmware/production-server-add-wizard.png)

3. En **Server Name/IP Address** (Nombre o dirección IP del servidor) de **Select Computers** (Seleccionar equipos), especifique el nombre de dominio completo o la dirección IP del servidor de VMware. Si todos los servidores de ESXi están administrados por la misma instancia de vCenter, especifique su nombre. En caso contrario, agregue el host de ESXi.

    ![Especificación del servidor de VMware](./media/backup-azure-backup-server-vmware/add-vmware-server-provide-server-name.png)

4. En **SSL Port** (Puerto SSL), especifique el puerto usado para la comunicación con el servidor de VMware. 443 es el puerto predeterminado, pero puede cambiarlo si el servidor de VMware escucha en un puerto diferente.

5. En **Especificar credenciales**, seleccione la credencial que ha creado antes.

    ![Especificar credenciales](./media/backup-azure-backup-server-vmware/identify-creds.png)

6. Haga clic en **Add** (Agregar) para agregar el servidor de VMware a la lista de servidores. A continuación, haga clic en **Siguiente**.

    ![Agregar credencial y el servidor de VMWare](./media/backup-azure-backup-server-vmware/add-vmware-server-credentials.png)

7. En la pantalla **Summary** (Resumen), haga clic en **Add** (Agregar) para agregar el servidor de VMware a Azure Backup Server. El nuevo servidor se agrega inmediatamente, no se necesita ningún agente en el servidor de VMware.

    ![Agregar el servidor de VMware a Azure Backup Server](./media/backup-azure-backup-server-vmware/tasks-screen.png)

8. Compruebe la configuración en la página **Finish** (Finalizar).

   ![Página Finalizar](./media/backup-azure-backup-server-vmware/summary-screen.png)

Si tiene varios hosts de ESXi que no están administrados por vCenter Server o si tiene varias instancias de vCenter Server, deberá volver a ejecutar el asistente para agregar los servidores.




## <a name="configure-a-protection-group"></a>Configuración de un grupo de protección

Agregue máquinas virtuales de VMware para la copia de seguridad. Los grupos de protección recopilan varias máquinas virtuales y aplican la misma configuración de copia de seguridad y de retención de datos para todas las máquinas virtuales del grupo.


1. En la consola de Azure Backup Server, haga clic en **Protection** > **New** (Protección > Nuevo).

    ![Abrir el asistente Crear nuevo grupo de protección.](./media/backup-azure-backup-server-vmware/open-protection-wizard.png)

1. En la página principal del asistente **Create New Protection Group** (Crear grupo de protección), haga clic en **Next** (Siguiente).

    ![Cuadro de diálogo del asistente Crear nuevo grupo de protección](./media/backup-azure-backup-server-vmware/protection-wizard.png)

1. En la página **Seleccionar tipo de grupo de protección**, seleccione **Servidores** y, después, haga clic en **Siguiente**. Se muestra la página **Seleccionar miembros del grupo**.

1. En **Select group members** (Seleccionar miembros del grupo) > seleccione las máquinas virtuales (o carpetas de la máquina virtual) de las que desea realizar copias de seguridad. A continuación, haga clic en **Siguiente**.

    - Al seleccionar una carpeta, las máquinas virtuales o carpetas dentro de esa carpeta también se seleccionan para la copia de seguridad. Puede desactivar las carpetas o máquinas virtuales de las que no desee copia de seguridad.
1. Si ya se está realizando la copia de seguridad de una máquina virtual o carpeta, estas no se pueden seleccionar. De este moro se garantiza que no se crean puntos de recuperación duplicados para una máquina virtual. .

     ![Seleccionar a miembros del grupo](./media/backup-azure-backup-server-vmware/server-add-selected-members.png)


1. En la página **Select Data Protection Method** (Seleccionar método de protección de datos), introduzca un nombre para el grupo de protección y la configuración. Para volver a Azure, configure la protección a corto plazo en **Disk** (Disco) y habilite la protección en línea. A continuación, haga clic en **Siguiente**.

    ![Seleccionar método de protección de datos](./media/backup-azure-backup-server-vmware/name-protection-group.png)

1. En **Specify Short-Term Goals** (Especificar objetivos a corto plazo), especifique cuánto tiempo desea mantener la copia de seguridad de los datos en el disco.
   - En **Retention Range** (Duración de retención), especifique cuántos días deben mantenerse los puntos de recuperación de disco.
   - En **Synchronization frequency** (Frecuencia de sincronización), especifique con qué frecuencia se crearán los puntos de recuperación de disco.
       - Si no desea establecer un intervalo de copia de seguridad puede marcar **Just before a recovery point** (Solo antes de un punto de recuperación) para que se ejecute una copia de seguridad solo antes de la creación de cada punto de recuperación.
       - Las copias de seguridad a corto plazo son copias de seguridad completas y no incrementales.
       - Haga clic en **Modify** (Modificar) para cambiar las fechas/horas con las copias de seguridad a corto plazo.

     ![Especificar objetivos a corto plazo](./media/backup-azure-backup-server-vmware/short-term-goals.png)

1. En **Review Disk Allocation** (Revisar asignación de disco), revise el espacio en disco para las copias de seguridad de las máquinas virtuales.

   - Las asignaciones de disco recomendadas se basan en la duración de retención especificada, el tipo de carga de trabajo y el tamaño de los datos protegidos. Realice los cambios necesarios y haga clic en **Next** (Siguiente).
   - **Tamaño de datos**: tamaño de los datos del grupo de protección.
   - **Espacio en disco:** espacio en disco recomendado para el grupo de protección. Si desea modificar esta configuración, debe asignar un espacio total que sea ligeramente mayor que la cantidad que calcula que va a crecer cada origen de datos.
   - **Ubicación compartida de datos:** si activa la ubicación compartida, varios orígenes de datos de la protección pueden asignarse a una sola réplica y a un volumen de puntos de recuperación. La ubicación compartida no es compatible con todas las cargas de trabajo.
   - **Expandir automáticamente:** al habilitar esta configuración, si los datos del grupo protegido sobrepasan la asignación inicial, Azure Backup Server intenta aumentar el tamaño del disco en un 25 %.
   - **Detalles del grupo de almacenamiento:** muestra el estado actual del grupo de almacenamiento, incluido el tamaño total y restante del disco.

     ![Revisar la asignación de disco](./media/backup-azure-backup-server-vmware/review-disk-allocation.png)

1. En la página **Choose Replica Creation method** (Seleccionar método de creación de réplicas), especifique cómo desea realizar la copia de seguridad inicial y haga clic en **Next** (Siguiente).
   - El valor predeterminado es **Automáticamente a través de la red** y **Ahora**.
   - Si usa el valor predeterminado, recomendamos que especifique una hora de poco tráfico. Elija **Más tarde** y especifique un día y una hora.
   - Para grandes cantidades de datos o condiciones de red no del todo óptimas, considere la posibilidad de replicar los datos sin conexión con medios extraíbles.

     ![Elegir método de creación de réplica](./media/backup-azure-backup-server-vmware/replica-creation.png)

1. En **Consistency Check Options** (Opciones de comprobación de coherencia), seleccione cómo y cuándo automatizar las comprobaciones de coherencia. A continuación, haga clic en **Siguiente**.
      - Puede ejecutar comprobaciones de coherencia si los datos de réplica no son coherentes o en una programación establecida.
      - Si no desea configurar la comprobación de coherencia automática, puede ejecutar una comprobación manual. Para ello, haga clic con el botón derecho en el grupo de protección > **Perform Consistency Check** (Realizar comprobación de coherencia).

1. En la página **Specify Online Protection Data** (Especificar datos de protección en línea), seleccione las máquinas virtuales o las carpetas de máquina virtual de las que desee realizar copias de seguridad. Puede seleccionar los miembros individualmente o hacer clic en **Seleccionar todo** para elegir todos los miembros. A continuación, haga clic en **Siguiente**.

      ![Especificar datos de protección en línea](./media/backup-azure-backup-server-vmware/select-data-to-protect.png)

1. En la página **Specify Online Backup Schedule** (Especificar la programación de copia de seguridad en línea), especifique con qué frecuencia desea realizar la copia de seguridad de los datos del espacio de almacenamiento local en Azure.

    - Se generarán puntos de recuperación de los datos en la nube según la programación. A continuación, haga clic en **Siguiente**.
    - Después de generar el punto de recuperación, se transfiere al almacén de Recovery Services de Azure.

      ![Especificar programación de copia de seguridad en línea](./media/backup-azure-backup-server-vmware/online-backup-schedule.png)

1. En la página **Specify Online Retention Policy** (Especificar la directiva de retención en línea), indique el tiempo de conservación de los puntos de recuperación que se crean de las copias de seguridad diarias, semanales, mensuales y anuales en Azure y haga clic en **Next** (Siguiente).

    - Los datos pueden guardarse en Azure sin límite de tiempo.
    - El único límite es que no se pueden tener más de 9999 puntos de recuperación por instancia protegida. En este ejemplo, la instancia protegida es el servidor de VMware.

      ![Especificar directiva de retención en línea](./media/backup-azure-backup-server-vmware/retention-policy.png)


1. En la página **Summary** (Resumen), revise la configuración y haga clic en **Create Group** (Crear grupo).

     ![Resumen de configuración y miembros del grupo de protección](./media/backup-azure-backup-server-vmware/protection-group-summary.png)

## <a name="vmware-vsphere-67"></a>6.7 de VMWare vSphere

Para hacerlo de copia de seguridad vSphere 6.7 lo siguiente:

- Habilitar TLS 1.2 en el servidor DPM
  >[!Note]
  >6.7 y versiones posteriores de VMWare habilitó TLS como protocolo de comunicación.

- Establezca las claves del registro de la manera siguiente:  

  Editor del registro de Windows versión 5.00

  [HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\\.NETFramework\v2.0.50727] "SystemDefaultTlsVersions"=dword:00000001 "SchUseStrongCrypto"=dword:00000001

  [HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\\.NETFramework\v4.0.30319] "SystemDefaultTlsVersions"=dword:00000001 "SchUseStrongCrypto"=dword:00000001

  [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\\.NETFramework\v2.0.50727] "SystemDefaultTlsVersions"=dword:00000001 "SchUseStrongCrypto"=dword:00000001

  [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\\.NETFramework\v4.0.30319] "SystemDefaultTlsVersions"=dword:00000001 s"SchUseStrongCrypto"=dword:00000001


## <a name="next-steps"></a>Pasos siguientes

Para solucionar problemas de configuración de las copias de seguridad, revise la [guía de solución de problemas de Azure Backup Server](./backup-azure-mabs-troubleshoot.md).

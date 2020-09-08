---
title: Unión de una máquina virtual SLE a Azure AD Domain Services | Microsoft Docs
description: Aprenda a configurar y unir una máquina virtual SUSE Linux Enterprise a un dominio administrado de Azure AD Domain Services.
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 08/12/2020
ms.author: iainfou
ms.openlocfilehash: 9f50be95e456802c6ad403acd6a2f539780e53a2
ms.sourcegitcommit: c293217e2d829b752771dab52b96529a5442a190
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/15/2020
ms.locfileid: "88251134"
---
# <a name="join-a-suse-linux-enterprise-virtual-machine-to-an-azure-active-directory-domain-services-managed-domain"></a>Unión de una máquina virtual SUSE Linux Enterprise a un dominio administrado de Azure Active Directory Domain Services

Para que los usuarios puedan iniciar sesión en máquinas virtuales (VM) en Azure con un único conjunto de credenciales, puede unir VM a un dominio administrado de Azure Active Directory Domain Services (Azure AD DS). Al unir una máquina virtual a un dominio administrado de Azure AD DS, se pueden usar las cuentas de usuario y las credenciales del dominio para iniciar sesión y administrar servidores. También se aplican las pertenencias a grupos del dominio administrado para permitirle controlar el acceso a los archivos o servicios de la VM.

En este artículo se muestra cómo unir una máquina virtual SUSE Linux Enterprise (SLE) a un dominio administrado.

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesitará los siguientes recursos y privilegios:

* Una suscripción de Azure activa.
    * Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un inquilino de Azure Active Directory asociado a su suscripción, ya sea sincronizado con un directorio en el entorno local o con un directorio solo en la nube.
    * Si es necesario, [cree un inquilino de Azure Active Directory][create-azure-ad-tenant] o [asocie una suscripción a Azure con su cuenta][associate-azure-ad-tenant].
* Un dominio administrado de Azure Active Directory Domain Services habilitado y configurado en su inquilino de Azure AD.
    * Si es necesario, el primer tutorial [crea y configura un dominio administrado de Azure Active Directory Domain Services][create-azure-ad-ds-instance].
* Una cuenta de usuario que forme parte del dominio administrado.

## <a name="create-and-connect-to-a-sle-linux-vm"></a>Creación y conexión a una máquina virtual SLE Linux

Si ya tiene una máquina virtual SLE Linux en Azure, conéctese a ella mediante SSH y luego continúe con el paso siguiente para [empezar a configurar la máquina virtual](#configure-the-hosts-file).

Si necesita crear una máquina virtual SLE Linux o desea crear una máquina virtual de prueba para usarla con este artículo, puede utilizar uno de los métodos siguientes:

* [Azure Portal](../virtual-machines/linux/quick-create-portal.md)
* [CLI de Azure](../virtual-machines/linux/quick-create-cli.md)
* [Azure PowerShell](../virtual-machines/linux/quick-create-powershell.md)

Al crear la VM, preste atención a la configuración de la red virtual para asegurarse de que la VM puede comunicarse con el dominio administrado:

* Implemente la máquina virtual en la misma red virtual, o en otra emparejada, en la que haya habilitado Azure AD Domain Services.
* Implemente la VM en una subred diferente a la del dominio administrado de Azure AD Domain Services.

Una vez implementada la máquina virtual, siga los pasos para conectarse a la máquina virtual mediante SSH.

## <a name="configure-the-hosts-file"></a>Configuración del archivo hosts

Para asegurarse de que el nombre de host de la máquina virtual está configurado correctamente para el dominio administrado, edite el archivo */etc/hosts* y establezca el nombre de host:

```console
sudo vi /etc/hosts
```

En el archivo *hosts*, actualice la dirección *localhost*. En el ejemplo siguiente:

* *aaddscontoso.com* es el nombre de dominio DNS del dominio administrado.
* *linux-q2gr* es el nombre de host de la máquina virtual SLE que se va a unir al dominio administrado.

Actualice estos nombres con sus propios valores:

```console
127.0.0.1 linux-q2gr linux-q2gr.aaddscontoso.com
```

Cuando haya terminado, guarde y salga del archivo *hosts* mediante el comando `:wq` del editor.

## <a name="join-vm-to-the-managed-domain-using-sssd"></a>Unión de una máquina virtual al dominio administrado mediante SSSD

Para unirse al dominio administrado mediante **SSSD** y el módulo *User Logon Management* (Administración de inicio de sesión de usuario) de YaST, complete los pasos siguientes:

1. Instale el módulo *User Logon Management* (Administración de inicio de sesión de usuario) de YaST:

    ```bash
    sudo zypper install yast2-auth-client
    ```

1. Abra YaST.

1. Para usar correctamente la detección automática de DNS más adelante, configure las direcciones IP del dominio administrado (el *servidor de Active Directory*) como servidor de nombres para el cliente.

    En YaST, seleccione **System > Network Settings** (Sistema > Configuración de red).

1. Seleccione la pestaña *Hostname/DNS* (Nombre de host/DNS) y, a continuación, escriba las direcciones IP del dominio administrado en el cuadro de texto *Name Server 1* (Servidor de nombres 1). Estas direcciones IP se muestran en la ventana *Propiedades* de Azure Portal del dominio administrado, como *10.0.2.4* y *10.0.2.5*.

    Agregue sus propias direcciones IP de dominio administrado y, después, seleccione **Aceptar**.

1. En la ventana principal de YaST, elija *Network Services* > *User Logon Management* (Servicios de red > Administración de inicio de sesión del usuario).

    Se abre el módulo con información general que muestra las distintas propiedades de red del equipo y el método de autenticación actualmente en uso, tal como se muestra en la siguiente captura de pantalla de ejemplo:

    ![Captura de pantalla de ejemplo de la ventana de administración de inicio de sesión de usuario en YaST](./media/join-suse-linux-vm/overview-window.png)

    Para empezar la edición, seleccione **Cambiar configuración**.

Para unir la máquina virtual a un dominio administrado, realice los siguientes pasos:

1. En el cuadro de diálogo, seleccione **Agregar dominio**.

1. Especifique el *Nombre de dominio* correcto, como *aaddscontoso.com* y, a continuación, especifique los servicios que se usarán para la autenticación y los datos de identidad. Seleccione *Microsoft Active Directory* para ambos.

    Asegúrese de que la opción *Enable the domain* (Habilitar el dominio) está seleccionada.

1. Cuando esté preparado, seleccione **Aceptar**.

1. Acepte la configuración predeterminada en el siguiente cuadro de diálogo y, a continuación, seleccione **Aceptar**.

1. La máquina virtual instala el software adicional necesario y, a continuación, comprueba si el dominio administrado está disponible.

    Si todo es correcto, se muestra el siguiente cuadro de diálogo de ejemplo para indicar que la máquina virtual ha detectado el dominio administrado, pero que *Not yet enrolled* (Aún no está inscrito).

    ![Captura de pantalla de ejemplo de la ventana de inscripción de Active Directory en YaST](./media/join-suse-linux-vm/enroll-window.png)

1. En el cuadro de diálogo, especifique el *Nombre de usuario* y *Contraseña* de un usuario que forme parte del dominio administrado. Si es necesario, [agregue una cuenta de usuario a un grupo en Azure AD](../active-directory/fundamentals/active-directory-groups-members-azure-portal.md).

    Para asegurarse de que el dominio actual está habilitado para Samba, active *Overwrite Samba configuration to work with this AD* (Sobrescribir la configuración de Samba para que funcione con este AD).

1. Para inscribirse, seleccione **Aceptar**.

1. Se muestra un mensaje para confirmar que se ha inscrito correctamente. Para finalizar, seleccione **Aceptar**.

Después de inscribir la máquina virtual en el dominio administrado, configure el cliente mediante *Manage Domain User Logon* (Administrar inicio de sesión de usuario de dominio), como se muestra en la siguiente captura de pantalla de ejemplo:

![Captura de pantalla de ejemplo de la ventana de administración de inicio de sesión de usuario de dominio en YaST](./media/join-suse-linux-vm/manage-domain-user-logon-window.png)

1. Para permitir los inicios de sesión con los datos proporcionados por el dominio administrado, active la casilla *Allow Domain User Logon* (Permitir inicio de sesión de usuario de dominio).

1. Opcionalmente, en *Enable domain data source* (Habilitar origen de datos de dominio), compruebe los orígenes de datos adicionales según sea necesario para su entorno. Entre estas opciones se incluyen los usuarios que pueden usar **sudo** o las unidades de red que están disponibles.

1. Para permitir que los usuarios del dominio administrado tengan directorios particulares en la máquina virtual, active la casilla *Create Home Directories* (Crear directorios particulares).

1. En la barra lateral, seleccione **Service Options › Name switch** (Opciones de servicio › Cambio de nombre) y, a continuación, *Extended Options* (Opciones extendidas). En esa ventana, seleccione *fallback_homedir* u *override_homedir* y, a continuación, seleccione **Agregar**.

1. Especifique un valor para la ubicación del directorio particular. Para que los directorios particulares sigan el formato */home/USER_NAME*, use */home/%u*. Para obtener más información sobre las posibles variables, vea la página man sssd.conf (`man 5 sssd.conf`), sección *override_homedir*.

1. Seleccione **Aceptar**.

1. Seleccione **Aceptar** para guardar los cambios. Después, asegúrese de que los valores mostrados ahora son correctos. Para salir del cuadro de diálogo, seleccione **Cancelar**.

1. Si piensa ejecutar SSSD y Winbind simultáneamente (por ejemplo, cuando se une a través de SSSD, pero después ejecuta un servidor de archivos Samba), la opción de *método de Kerberos* de Samba debe establecerse en *secrets and keytab* en smb.conf. La opción SSSD *ad_update_samba_machine_account_password* debe establecerse también en *true* en sssd.conf. Estas opciones impiden que el archivo keytab del sistema deje de estar sincronizado.

## <a name="join-vm-to-the-managed-domain-using-winbind"></a>Unión de una máquina virtual al dominio administrado mediante Winbind

Para unirse al dominio administrado mediante **windbind** y el módulo *Windows Domain Membership* (Pertenencia a dominio de Windows) de YaST, complete los pasos siguientes:

1. En YaST, seleccione **Network Services > Windows Domain Membership** (Servicios de red > Pertenencia a dominio de Windows).

1. Escriba el dominio al que desea unirse en *Domain or Workgroup* (Dominio o grupo de trabajo) de la pantalla *Windows Domain Membership* (Pertenencia a dominio de Windows). Escriba el nombre de dominio administrado como, por ejemplo, *aaddscontoso.com*.

    ![Captura de pantalla de ejemplo de la ventana de pertenencia a dominio de Windows en YaST](./media/join-suse-linux-vm/samba-client-window.png)

1. Para usar el origen SMB para la autenticación de Linux, marque la opción *Use SMB Information for Linux Authentication* (Usar información de SMB para la autenticación de Linux).

1. Para crear automáticamente un directorio particular local para los usuarios del dominio administrado en la máquina virtual, marque la opción *Create Home Directory on Login* (Crear directorio particular en el inicio de sesión).

1. Marque la opción *Offline Authentication* (Autenticación sin conexión) para permitir que los usuarios del dominio inicien sesión, incluso si el dominio administrado no está disponible temporalmente.

1. Si desea cambiar los rangos de UID y GID para los usuarios y grupos de Samba, seleccione *Expert Settings* (Configuración de experto).

1. Configure la sincronización de la hora NTP para el dominio administrado seleccionando *NTP Configuration* (Configuración de NTP). Escriba las direcciones IP del dominio administrado. Estas direcciones IP se muestran en la ventana *Propiedades* de Azure Portal del dominio administrado, como *10.0.2.4* y *10.0.2.5*.

1. Seleccione **Aceptar** y confirme la unión al dominio cuando se le solicite.

1. Proporcione la contraseña de un administrador en el dominio administrado y seleccione **Aceptar**.

    ![Captura de pantalla de ejemplo del mensaje de cuadro de diálogo de autenticación al unir una máquina virtual SLE al dominio administrado](./media/join-suse-linux-vm/domain-join-authentication-prompt.png)

Una vez que se ha unido al dominio administrado, puede iniciar sesión en él desde la estación de trabajo con el administrador de pantalla del escritorio o la consola.

## <a name="allow-password-authentication-for-ssh"></a>Permitir autenticación de contraseña para SSH

De forma predeterminada, los usuarios solo pueden iniciar sesión en una máquina virtual mediante la autenticación basada en clave pública de SSH. La autenticación basada en contraseñas produce un error. Cuando une la máquina virtual a un dominio administrado, esas cuentas de dominio deben usar la autenticación basada en contraseñas. Actualice la configuración de SSH para permitir la autenticación basada en contraseñas como se indica a continuación.

1. Abra el archivo *sshd_conf* con un editor:

    ```console
    sudo vi /etc/ssh/sshd_config
    ```

1. Actualice la línea de *PasswordAuthentication* a *yes* (sí):

    ```console
    PasswordAuthentication yes
    ```

    Cuando haya terminado, guarde y salga del archivo *sshd_conf* mediante el comando `:wq` del editor.

1. Para aplicar los cambios y permitir que los usuarios inicien sesión con una contraseña, reinicie el servicio SSH:

    ```console
    sudo systemctl restart sshd
    ```

## <a name="grant-the-aad-dc-administrators-group-sudo-privileges"></a>Conceda privilegios de sudo al grupo "Administradores de controlador de dominio de AAD"

Para conceder privilegios administrativos a los miembros del grupo *Administradores de controlador de dominio de AAD* en la máquina virtual SLE, puede agregar una entrada a */etc/sudoers*. Una vez agregada, los miembros del grupo *Administradores de controlador de dominio de AAD* pueden usar el comando `sudo` en la máquina virtual SLE.

1. Abra el archivo *sudoers* para editarlo:

    ```console
    sudo visudo
    ```

1. Agregue la siguiente entrada al final del archivo */etc/sudoers*. El grupo *Administradores del controlador de dominio de AAD* contiene un espacio en blanco en el nombre, por lo que debe incluir el carácter de escape de barra diagonal inversa en el nombre del grupo. Agregue su propio nombre de dominio, por ejemplo, *aaddscontoso.com*:

    ```console
    # Add 'AAD DC Administrators' group members as admins.
    %AAD\ DC\ Administrators@aaddscontoso.com ALL=(ALL) NOPASSWD:ALL
    ```

    Cuando haya terminado, guarde y salga del editor mediante el comando `:wq`.

## <a name="sign-in-to-the-vm-using-a-domain-account"></a>Inicio de sesión en la máquina virtual mediante una cuenta de dominio

Para comprobar que la VM se ha unido correctamente al dominio administrado, inicie una nueva conexión SSH con una cuenta de usuario de dominio. Confirme que se ha creado un directorio particular y que se ha aplicado la pertenencia a grupos del dominio.

1. Cree una nueva conexión SSH desde la consola. Use una cuenta de dominio que pertenezca al dominio administrado mediante el comando `ssh -l` como, por ejemplo, `contosoadmin@aaddscontoso.com` y, a continuación, escriba la dirección de la máquina virtual, por ejemplo: *linux-q2gr.aaddscontoso.com*. Si usa Azure Cloud Shell, use la dirección IP pública de la máquina virtual en lugar del nombre DNS interno.

    ```console
    ssh -l contosoadmin@AADDSCONTOSO.com linux-q2gr.aaddscontoso.com
    ```

2. Cuando se haya conectado correctamente a la máquina virtual, compruebe que el directorio particular se ha inicializado correctamente:

    ```console
    pwd
    ```

    Debe estar en el directorio */home* con su propio directorio que coincide con la cuenta de usuario.

3. Ahora, compruebe que las pertenencias a grupos se están resolviendo correctamente:

    ```console
    id
    ```

    Debería ver las pertenencias a grupos del dominio administrado.

4. Si ha iniciado sesión en la máquina virtual como miembro del grupo *Administradores del controlador de dominio de AAD*, compruebe que puede usar correctamente el comando `sudo`:

    ```console
    sudo zypper update
    ```

## <a name="next-steps"></a>Pasos siguientes

Si tiene problemas para conectar la VM al dominio administrado o al iniciar sesión con una cuenta de dominio, consulte [Solución de problemas de unión al dominio](join-windows-vm.md#troubleshoot-domain-join-issues).

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md

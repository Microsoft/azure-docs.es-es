---
title: 'Creación de un recurso compartido de archivos de Azure Files con un controlador de dominio: Azure'
description: Configure un contenedor de perfil de FSLogix en un recurso compartido de archivos de Azure, en un grupo host de escritorio virtual de Windows existente, con su dominio de Active Directory.
author: Heidilohr
ms.topic: how-to
ms.date: 06/05/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: e03d496881b0d563387ee5a5943b60f456530453
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88009229"
---
# <a name="create-a-profile-container-with-azure-files-and-ad-ds"></a>Creación de un contenedor de perfiles con Azure Files y AD DS

En este artículo, aprenderá a crear un recurso compartido de archivos de Azure autenticado por un controlador de dominio, en un grupo host de escritorio virtual de Windows existente. Puede usar este recurso compartido de archivos para almacenar los perfiles de almacenamiento.

Este proceso usa Active Directory Domain Services (AD DS), que es un servicio de directorio local. Si busca información sobre cómo crear un contenedor de perfil de FSLogix con Azure AD DS, consulte [Creación de un contenedor de perfiles de FSLogix con Azure Files](create-profile-container-adds.md).

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar, asegúrese de que el controlador de dominio esté sincronizado con Azure y que se pueda resolver desde la red virtual de Azure a la que se conectan los hosts de la sesión.

## <a name="set-up-a-storage-account"></a>Configuración de una cuenta de almacenamiento

En primer lugar, deberá configurar una cuenta de almacenamiento de Azure Files.

Para configurar una cuenta de almacenamiento:

1. Inicie sesión en Azure Portal.

2. Busque la **cuenta de almacenamiento** en la barra de búsqueda.

3. Seleccione **+Agregar**.

4. En la página **Crear cuenta de almacenamiento**, escriba la siguiente información:

    - Cree un nuevo grupo de recursos.
    - Escriba un nombre único para la cuenta de almacenamiento.
    - En **Ubicación**, se recomienda elegir la misma ubicación que el grupo host de Windows Virtual Desktop.
    - En **Rendimiento**, seleccione **Estándar**. (En función de los requisitos de IOPS. Para más información, consulte [Opciones de almacenamiento para los contenedores de perfiles de FSLogix de Windows Virtual Desktop](store-fslogix-profile.md)).
    - En **Tipo de cuenta**, seleccione **StorageV2** o **FileStorage** (solo disponible si el nivel de rendimiento es Premium).
    - En **Replicación**, seleccione **Almacenamiento con redundancia local (LRS)** .

5. Cuando haya terminado, seleccione **Revisar y crear** y, después, **Crear**.

Si necesita instrucciones de configuración más detalladas, consulte [Disponibilidad regional](../storage/files/storage-files-identity-auth-active-directory-enable.md#regional-availability).

## <a name="create-an-azure-file-share"></a>Creación de un recurso compartido de archivos de Azure

A continuación, deberá crear un recurso compartido de archivos de Azure.

Para crear un recurso compartido de archivos:

1. Haga clic en **Go to resource** (Ir al recurso).

2. En la página Información general, seleccione **Recursos compartidos de archivos**.

3. Seleccione **+ Recursos compartidos de archivos**, cree un nuevo recurso compartido de archivos denominado **perfiles**, especifique una cuota adecuada o deje el campo en blanco para no incluir una cuota.

4. Seleccione **Crear**.

## <a name="enable-active-directory-authentication"></a>Habilitar la autenticación de Active Directory

Después, deberá habilitar la autenticación de Active Directory (AD). Para habilitar esta directiva, deberá seguir las instrucciones de esta sección en una máquina que ya esté unida a un dominio. Para habilitar la autenticación, siga estas instrucciones en la máquina virtual que ejecuta el controlador de dominio:

1. Protocolo de Escritorio remoto en la máquina virtual unida a un dominio.

2. Siga las instrucciones del artículo [Parte 1: Habilitación de la autenticación de AD DS para los recursos compartidos de archivos de Azure](../storage/files/storage-files-identity-ad-ds-enable.md) para instalar el módulo AzFilesHybrid y habilitar la autenticación.

3.  Abra Azure Portal, abra la cuenta de almacenamiento, seleccione **Configuración** y, después, confirme que **Active Directory (AD)** está establecido en **Habilitado**.

     > [!div class="mx-imgBorder"]
     > ![Captura de pantalla de la página Configuración con Azure Active Directory (AD) habilitado](media/active-directory-enabled.png)

## <a name="assign-azure-rbac-permissions-to-windows-virtual-desktop-users"></a>Asignación de permisos de RBAC de Azure a los usuarios de Windows Virtual Desktop

A todos los usuarios que necesiten tener perfiles de FSLogix almacenados en la cuenta de almacenamiento se les debe asignar el rol de colaborador de recursos compartidos de SMB de datos de archivos de almacenamiento.

Los usuarios que inician sesión en los hosts de sesión de Windows Virtual Desktop necesitan permisos de acceso para acceder al recurso compartido de archivos. Conceder acceso a un recurso compartido de archivos de Azure implica configurar permisos tanto en el nivel de recurso compartido como en el nivel de NTFS, de forma similar a un recurso compartido de Windows tradicional.

Para configurar los permisos de nivel de recurso compartido, asigne a cada usuario un rol con los permisos de acceso adecuados. Los permisos se pueden asignar a usuarios individuales o a un grupo de Azure AD. Para más información, consulte [Parte dos: Asignación de permisos de nivel de recurso compartido a una identidad](../storage/files/storage-files-identity-ad-ds-assign-permissions.md).

>[!NOTE]
>Las cuentas o grupos a los que asigna permisos deben haberse creado en el dominio y sincronizarse con Azure AD. Las cuentas creadas en Azure AD no funcionan.

Para asignar permisos de control de acceso basado en roles (RBAC):

1. Abra Azure Portal.

2. Abra la cuenta de almacenamiento que creó en [Configuración de una cuenta de almacenamiento](#set-up-a-storage-account).

3. Seleccione **Recursos compartidos de archivos** y, después, seleccione el nombre del recurso compartido de archivos que tenga previsto usar.

4. Seleccione **Access Control (IAM)** .

5. Seleccione **Agregar una asignación de roles**.

6. En la pestaña **Agregar asignación de roles**, seleccione **Colaborador con privilegios elevados de recursos compartidos de SMB de datos de archivos de almacenamiento** para la cuenta de administrador.

     Para asignar permisos de usuario para los perfiles de FSLogix, siga estas mismas instrucciones. Sin embargo, cuando llegue al paso 5, seleccione **Colaborador de recursos compartidos de SMB de datos de archivos de almacenamiento** en su lugar.

7. Seleccione **Guardar**.

## <a name="assign-users-permissions-on-the-azure-file-share"></a>Asignación de permisos de usuarios en el recurso compartido de archivos de Azure

Una vez que haya asignado permisos de RBAC a los usuarios, deberá configurar los permisos NTFS.

Necesitará conocer lo siguiente de Azure Portal para comenzar:

- Ruta de acceso UNC.
- La clave de la cuenta de almacenamiento.

### <a name="get-the-unc-path"></a>Obtención de la ruta de acceso UNC

Aquí se muestra cómo obtener la ruta de acceso UNC:

1. Abra Azure Portal.

2. Abra la cuenta de almacenamiento que creó en [Configuración de una cuenta de almacenamiento](#set-up-a-storage-account).

3. Seleccione **Configuración** y luego **Propiedades**.

4. Copie el URI del **punto de conexión del servicio de archivo principal**  en el editor de texto de su elección.

5. Después de copiar el URI, realice las siguientes acciones para cambiarlo a la ruta UNC:

    - Quite `https://` y reemplácelo por `\\`.
    - Reemplace la barra diagonal `/` por una barra diagonal inversa `\`.
    - Agregue el nombre del recurso compartido de archivos que creó en [Creación de un recurso compartido de archivos de Azure](#create-an-azure-file-share) al final de la UNC.

        Por ejemplo: `\\customdomain.file.core.windows.net\<fileshare-name>`

### <a name="get-the-storage-account-key"></a>Obtención de la clave de la cuenta de almacenamiento

Para obtener la clave de la cuenta de almacenamiento:

1. Abra Azure Portal.

2. Abra la cuenta de almacenamiento que creó en [Configuración de una cuenta de almacenamiento](#set-up-a-storage-account).

3. En la pestaña **Cuenta de almacenamiento**, seleccione **Claves de acceso**.

4. Copie **key1** o **key2** en un archivo del equipo local.

### <a name="configure-ntfs-permissions"></a>Configuración de los permisos NTFS

Para configurar los permisos NTFS:

1. Abra un símbolo del sistema en una máquina virtual unida a un dominio.

2. Ejecute el siguiente cmdlet para montar el recurso compartido de archivos de Azure y asígnele una letra de unidad:

     ```powershell
     net use <desired-drive-letter>: <UNC-pat> <SA-key> /user:Azure\<SA-name>
     ```

3. Ejecute el siguiente cmdlet para revisar los permisos de acceso al recurso compartido de archivos de Azure:

    ```powershell
    icacls <mounted-drive-letter>:
    ```

    Reemplace `<mounted-drive-letter>` por la letra de la unidad a la que realizó la asignación.

    Tanto *NT AUTHORITY\Usuarios autenticados* como *BUILTIN\Usuarios* tienen determinados permisos de forma predeterminada. Estos permisos predeterminados permiten a los usuarios leer los contenedores de perfil de otros usuarios. Sin embargo, los permisos descritos en [Configuración de permisos de almacenamiento para su uso con contenedores de perfiles y contenedores de Office](/fslogix/fslogix-storage-config-ht) no permiten a los usuarios leer los contenedores de perfiles de los demás.

4. Ejecute los siguientes cmdlets para permitir que los usuarios de Windows Virtual Desktop creen sus propios contenedores de perfiles mientras bloquean el acceso a su contenedor de perfiles de otros usuarios.

     ```powershell
     icacls <mounted-drive-letter>: /grant <user-email>:(M)
     icacls <mounted-drive-letter>: /grant "Creator Owner":(OI)(CI)(IO)(M)
     icacls <mounted-drive-letter>: /remove "Authenticated Users"
     icacls <mounted-drive-letter>: /remove "Builtin\Users"
     ```

     - Reemplace <mounted-drive-letter> por la letra de la unidad que usó para asignar la unidad.
     - Reemplace <user-email> por el UPN del usuario o grupo de Active Directory que contiene los usuarios que necesitarán acceso al recurso compartido.

     Por ejemplo:

     ```powershell
     icacls <mounted-drive-letter>: /grant john.doe@contoso.com:(M)
     icacls <mounted-drive-letter>: /grant "Creator Owner":(OI)(CI)(IO)(M)
     icacls <mounted-drive-letter>: /remove "Authenticated Users"
     icacls <mounted-drive-letter>: /remove "Builtin\Users"
     ```

## <a name="configure-fslogix-on-session-host-vms"></a>Configuración de FSLogix en máquinas virtuales de host de sesión

En esta sección se muestra cómo configurar una máquina virtual con FSLogix. Deberá seguir estas instrucciones cada vez que configure un host de sesión. Antes de empezar a configurar, siga las instrucciones de [Descarga e instalación de FSLogix](/fslogix/install-ht). Hay varias opciones disponibles que garantizan que las claves del Registro se establezcan en todos los hosts de la sesión. Puede establecer estas opciones en una imagen o configurar una directiva de grupo.

Para configurar FSLogix en la máquina virtual del host de sesión:

1. Ejecución de RDP en la máquina virtual del host de sesión del grupo host de Windows Virtual Desktop.

2. [Descarga e instalación de FSLogix](/fslogix/install-ht)

5. Siga las instrucciones de [Configuración del registro de contenedor de perfiles](/fslogix/configure-profile-container-tutorial#configure-profile-container-registry-settings):

    - Vaya a **Equipo** > **HKEY_LOCAL_MACHINE** > **SOFTWARE** > **FSLogix**.

    - Cree una clave **Perfiles**.

    - Cree **Habilitado, DWORD** con un valor de 1.

    - Cree **VHDLocations, MULTI_SZ**.

    - Establezca el valor de **VHDLocations** en la ruta de acceso UNC que generó en [Obtención de la ruta de acceso UNC](#get-the-unc-path).

6. Reinicie la VM.

## <a name="testing"></a>Prueba

Cuando haya instalado y configurado FSLogix, puede probar la implementación iniciando sesión con una cuenta de usuario a la que se haya asignado un grupo de aplicaciones o un escritorio en el grupo host. Asegúrese de que la cuenta de usuario con la que inicia sesión tenga permiso en el recurso compartido de archivos.

Si el usuario ha iniciado sesión antes, tendrá un perfil local existente que se usará durante esta sesión. Para evitar la creación de un perfil local, cree una nueva cuenta de usuario para usarla en las pruebas o use los métodos de configuración descritos en [Tutorial: Configuración del contenedor de perfiles para redirigir perfiles de usuario](/fslogix/configure-profile-container-tutorial/).

Para comprobar los permisos de la sesión:

1. Inicie una sesión en Windows Virtual Desktop.

2. Abra Azure Portal.

3. Abra la cuenta de almacenamiento que creó en [Configuración de una cuenta de almacenamiento](#set-up-a-storage-account).

4. Seleccione **Crear un recurso compartido** en la página de creación de un recurso compartido de archivos de Azure.

5. Asegúrese de que ya existe en los archivos una carpeta que contiene el perfil de usuario.

Para realizar pruebas adicionales, siga las instrucciones de la sección [Comprobación de que el perfil funciona](create-profile-container-adds.md#make-sure-your-profile-works).

## <a name="next-steps"></a>Pasos siguientes

Para solucionar problemas de FSLogix, consulte [esta guía de solución de problemas](/fslogix/fslogix-trouble-shooting-ht).

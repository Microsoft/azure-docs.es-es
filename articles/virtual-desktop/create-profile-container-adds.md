---
title: Creación de un contenedor de perfiles de FSLogix con Azure Files y Active Directory Domain Services en Azure
description: En este artículo se describe cómo crear un contenedor de perfiles de FSLogix con Azure Files y Azure Active Directory Domain Services.
author: Heidilohr
ms.topic: how-to
ms.date: 04/10/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: ea834ed874f3011d95f8b924df860576f72bc4ee
ms.sourcegitcommit: ac7ae29773faaa6b1f7836868565517cd48561b2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88825620"
---
# <a name="create-a-profile-container-with-azure-files-and-azure-ad-ds"></a>Creación de un contenedor de perfiles con Azure Files y Azure AD DS

En este artículo se muestra cómo crear un contenedor de perfiles de FSLogix con Azure Files y Azure Active Directory Domain Services (AD DS).

## <a name="prerequisites"></a>Requisitos previos

En este artículo se supone que ya ha configurado una instancia de Azure AD DS. Si aún no tiene una, siga primero las instrucciones del artículo sobre la [creación de un dominio administrado básico](../active-directory-domain-services/tutorial-create-instance.md) y, después, continúe aquí.

## <a name="add-azure-ad-ds-admins"></a>Adición de administradores de Azure AD DS

Para agregar administradores adicionales, debe crear un nuevo usuario y concederle permisos.

Para agregar un administrador:

1. Seleccione **Azure Active Directory** en la barra lateral, después, **Usuarios** y, finalmente, **Nuevo usuario**.

2.  Escriba los detalles del usuario en los campos correspondientes.

3. En el panel de Azure Active Directory, a la izquierda de la pantalla, seleccione **Grupos**.

4. Seleccione el grupo **Administradores de controladores de dominio de AAD**.

5. En el panel izquierdo, seleccione **Miembros** y, después, seleccione **Agregar miembros** en el panel principal. Se mostrará una lista de todos los usuarios disponibles en Azure AD. Seleccione el nombre del perfil de usuario que acaba de crear.

## <a name="set-up-an-azure-storage-account"></a>Configuración de una cuenta de Azure Storage

Ahora es el momento de habilitar la autenticación de Azure AD DS en el bloque de mensajes del servidor (SMB).

Para habilitar la autenticación:

1. Si aún no lo ha hecho, configure e implemente una cuenta de Azure Storage v2 de uso general siguiendo las instrucciones de [Creación de una cuenta de Azure Storage](../storage/common/storage-account-create.md).

2. Cuando haya terminado de configurar la cuenta, seleccione **Ir al recurso**.

3. Seleccione **Configuración** en el panel de la izquierda de la pantalla y, a continuación, habilite **Autenticación de Azure Active Directory para Azure Files** en el panel principal. Cuando finalice, seleccione **Guardar**.

4. Seleccione **Información general** en el panel de la izquierda de la pantalla y, a continuación, **Archivos** en el panel principal.

5. Seleccione **Recurso compartido de archivos** y escriba el **nombre** y la **cuota** en los campos que aparecen en el lado derecho de la pantalla.

## <a name="assign-access-permissions-to-an-identity"></a>Asignar permisos de acceso a una identidad

Otros usuarios necesitarán permisos de acceso para acceder al recurso compartido de archivos. Para ello, deberá asignar a cada usuario un rol con los permisos de acceso adecuados.

Para asignar permisos de acceso a los usuarios:

1. En Azure Portal, abra el recurso compartido de archivos que creó en [Configuración de una cuenta de Azure Storage](#set-up-an-azure-storage-account).

2. Seleccione **Access Control (IAM)** .

3. Seleccione **Agregar una asignación de roles**.

4. En la pestaña **Agregar asignación de roles**, seleccione el rol integrado apropiado en la lista de roles. Tendrá que seleccionar al menos **Colaborador de recursos compartidos de SMB de datos de archivos de almacenamiento** para que la cuenta obtenga los permisos adecuados.

5. En **Asignar acceso a**, seleccione **Usuario, grupo o entidad de servicio de Azure Active Directory**.

6. Seleccione un nombre o una dirección de correo electrónico para la identidad de Azure Active Directory de destino.

7. Seleccione **Guardar**.

## <a name="get-the-storage-account-access-key"></a>Obtención de la clave de acceso de la cuenta de almacenamiento

A continuación, deberá obtener la clave de acceso de la cuenta de almacenamiento.

Para obtener la clave de acceso a la cuenta de almacenamiento:

1. En Azure Portal, seleccione **Cuentas de almacenamiento**.

2. En la lista de cuentas de almacenamiento, seleccione la cuenta para la que habilitó Azure AD DS y creó los roles personalizados en los pasos anteriores.

3. En **Configuración**, seleccione **Claves de acceso** y copie la clave de **key1**.

4. Vaya a la pestaña **Máquinas virtuales** y busque cualquier máquina virtual que se convertirá en parte del grupo de hosts.

5. Seleccione el nombre de la máquina virtual en **Máquinas virtuales (adVM)** y seleccione **Conectar**.

    Se descargará un archivo RDP que le permitirá iniciar sesión en la máquina virtual con sus propias credenciales.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la pestaña RDP de la ventana Conectar a máquina virtual](media/rdp-tab.png)

6. Cuando haya iniciado sesión en la máquina virtual, ejecute un símbolo del sistema como administrador.

7. Ejecute el siguiente comando:

     ```cmd
     net use <desired-drive-letter>: \\<storage-account-name>.file.core.windows.net\<share-name> <storage-account-key> /user:Azure\<storage-account-name>
     ```

    - Reemplace `<desired-drive-letter>` por una letra de unidad de su elección (por ejemplo, `y:`).
    - Reemplace todas las instancias de `<storage-account-name>` por el nombre de la cuenta de almacenamiento que especificó anteriormente.
    - Reemplace `<share-name>` por el nombre del recurso compartido que creó anteriormente.
    - Reemplace `<storage-account-key>` por la clave de la cuenta de almacenamiento de Azure.

    Por ejemplo:

     ```cmd
     net use y: \\fsprofile.file.core.windows.net\share HDZQRoFP2BBmoYQ=(truncated)= /user:Azure\fsprofile)
     ```

8. Ejecute los siguientes comandos para permitir que los usuarios de Windows Virtual Desktop creen sus propios contenedores de perfiles mientras bloquean el acceso a los contenedores de perfiles de otros usuarios.

     ```cmd
     icacls <mounted-drive-letter>: /grant <user-email>:(M)
     icacls <mounted-drive-letter>: /grant "Creator Owner":(OI)(CI)(IO)(M)
     icacls <mounted-drive-letter>: /remove "Authenticated Users"
     icacls <mounted-drive-letter>: /remove "Builtin\Users"
     ```

    - Reemplace `<mounted-drive-letter>` por la letra de la unidad que usó para asignar la unidad.
    - Reemplace `<user-email>` por el UPN del usuario o grupo de Active Directory que contiene los usuarios que necesitarán acceso al recurso compartido.

    Por ejemplo:

     ```cmd
     icacls <mounted-drive-letter>: /grant john.doe@contoso.com:(M)
     icacls <mounted-drive-letter>: /grant "Creator Owner":(OI)(CI)(IO)(M)
     icacls <mounted-drive-letter>: /remove "Authenticated Users"
     icacls <mounted-drive-letter>: /remove "Builtin\Users"
     ```

## <a name="create-a-profile-container"></a>Creación de un contenedor de perfiles

Ahora que los perfiles están listos para usar, vamos a crear un contenedor de perfiles de FSLogix.

Para configurar el contenedor de perfiles de FSLogix:

1. Inicie sesión en la máquina virtual del host de sesión que configuró al principio de este artículo y, después, [descargue e instale el agente de FSLogix](/fslogix/install-ht/).

2. Descomprima el archivo del agente FSLogix que ha descargado y vaya a **x64** > **Releases** (Versiones) y, a continuación, abra **FSLogixAppsSetup.exe**.

3. Cuando se inicie el instalador, seleccione **Acepto los términos y las condiciones de la licencia.** Si procede, proporcione una nueva clave.

4. Seleccione **Instalar**.

5. Abra la **unidad C**, vaya a **Archivos de programa** > **FSLogix** > **Apps** para asegurarse de que el agente de FSLogix está correctamente instalado.

     >[!NOTE]
     > Si hay varias máquinas virtuales en el grupo de hosts, deberá repetir los pasos 1 a 5 para cada máquina virtual.

6. Ejecute **Editor del Registro** (RegEdit) como administrador.

7. Vaya a **Equipo** > **HKEY_LOCAL_MACHINE** > **Software** > **FSLogix**, haga clic con el botón derecho en **FSLogix**, seleccione **Nuevo** y, después, **Clave**.

8. Cree una clave denominada **Profiles** (Perfiles).

9.  Haga clic con el botón derecho en **Profiles** (Perfiles), seleccione **Nuevo** y, después, **Valor de DWORD (32 bits)** . Asigne un nombre al valor **Habilitado** y establezca el valor de **Datos** en **1**.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la clave Profiles El archivo REG_DWORD está resaltado y su valor de datos se establece en 1.](media/dword-value.png)

10. Haga clic con el botón derecho en **Profiles**, seleccione **Nuevo** y, a continuación, seleccione **Valor de cadena múltiple**. Asigne un nombre al valor **VHDLocations** y escriba el URI para el recurso compartido de Azure Files `\\fsprofile.file.core.windows.net\share` como valor de Datos.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la clave Profiles que muestra el archivo VHDLocations. Su valor de datos muestra el URI del recurso compartido de Azure Files.](media/multi-string-value.png)

## <a name="assign-users-to-a-session-host"></a>Asignación de usuarios a un host de sesión

Ahora deberá asignar usuarios al host de sesión.

Para asignar usuarios:

1. Ejecute Windows PowerShell como administrador y después ejecute el siguiente cmdlet para iniciar sesión en Windows Virtual Desktop con PowerShell:

   ```powershell
   Import-Module Microsoft.RdInfra.RdPowershell

   #Optional
   Install-Module Microsoft.RdInfra.RdPowershell

   $brokerurl = "https://rdbroker.wvd.microsoft.com"

   Add-RdsAccount -DeploymentUrl $brokerurl
   ```

   Cuando se le pidan las credenciales, especifique el mismo usuario al que se le concedió el rol Creador de inquilinos, Propietario de RDS o Colaborador de RDS en el inquilino de Windows Virtual Desktop.

2. Ejecute los siguientes cmdlets para asignar el usuario a un grupo de Escritorio remoto:

     ```powershell
     $tenant = "<your-wvd-tenant>"

     $pool1 = "<wvd-pool>"

     $appgroup = "Desktop Application Group"

     $user1 = "<user-principal>"

     Add-RdsAppGroupUser $tenant $pool1 $appgroup $user1
     ```

    Al igual que en los cmdlets anteriores, asegúrese de reemplazar `<your-wvd-tenant>`, `<wvd-pool>` y `<user-principal>` por los valores correspondientes.

    Por ejemplo:

     ```powershell
     $pool1 = "contoso"

     $tenant = "contoso"

     $appgroup = "Desktop Application Group"

     $user1 = "jane.doe@contoso.com"

     Add-RdsAppGroupUser $tenant $pool1 $appgroup $user1
     ```

## <a name="make-sure-your-profile-works"></a>Comprobación de que el perfil funciona

Ahora todo lo que tiene que hacer es asegurarse de que el perfil que creó existe y de que funciona según lo previsto.

Para comprobar el perfil:

1. Abra un explorador y vaya al [cliente de Windows Virtual Desktop](https://rdweb.wvd.microsoft.com/arm/webclient).

2. Inicie sesión con la cuenta de usuario asignada al grupo de Escritorio remoto.

3. Cuando se haya establecido la sesión de usuario, abra Azure Portal e inicie sesión con una cuenta administrativa.

4. En la barra lateral, seleccione **Cuentas de almacenamiento**.

5. Seleccione la cuenta de almacenamiento que ha configurado como recurso compartido de archivos para el grupo de hosts de sesión y que está habilitada con Azure AD DS.

6. Seleccione el icono **Archivos** y, a continuación, expanda el recurso compartido.

    Si todo está configurado correctamente, debería ver un **directorio** con un nombre con el siguiente formato: `<user SID>-<username>`.

## <a name="next-steps"></a>Pasos siguientes

Si está buscando formas alternativas de crear contenedores de perfiles de FSLogix, consulte los artículos siguientes:

- [Creación de un contenedor de perfiles para un grupo host mediante un recurso compartido de archivos](create-host-pools-user-profile.md)
- [Creación de un contenedor de perfiles de FSLogix para un grupo host mediante Azure NetApp Files](create-fslogix-profile-container.md)

Puede encontrar información más detallada sobre los conceptos relacionados con los contenedores de FSlogix para archivos de Azure en [Contenedores de perfiles de FSLogix y archivos de Azure](fslogix-containers-azure-files.md).

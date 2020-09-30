---
title: Inicio de sesión en una máquina virtual Windows en Azure mediante Azure Active Directory (versión preliminar)
description: Inicio de sesión de Azure AD en una VM de Azure que ejecuta Windows
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: how-to
ms.date: 07/20/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sandeo
ms.custom: references_regions
ms.collection: M365-identity-device-management
ms.openlocfilehash: 42b980f809074c275a5e62098d4162e4cb4340a0
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89259377"
---
# <a name="sign-in-to-windows-virtual-machine-in-azure-using-azure-active-directory-authentication-preview"></a>Inicio de sesión en una máquina virtual Windows en Azure mediante la autenticación de Azure Active Directory (versión preliminar)

Las organizaciones ahora pueden utilizar la autenticación de Azure Active Directory (AD) en sus máquinas virtuales (VM) de Azure que ejecutan **Windows Server 2019 Datacenter Edition** o **Windows 10 1809** y versiones posteriores. El uso de Azure AD para autenticarse en VM le proporciona una manera de controlar y aplicar directivas de forma centralizada. Herramientas como, por ejemplo, el control de acceso basado en rol de Azure (Azure RBAC) y el acceso condicional de Azure AD, permiten controlar quién puede acceder a una VM. En este artículo se muestra cómo crear y configurar una VM Windows Server 2019 para usar la autenticación de Azure AD.

> [!NOTE]
> El inicio de sesión de Azure AD para VM Windows de Azure es una Característica en vista previa (GB) pública de Azure Active Directory. Para más información sobre las versiones preliminares, consulte [Términos de uso complementarios de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Usar la autenticación de Azure AD para iniciar sesión en VM Windows en Azure implica varias ventajas, entre las que se incluyen:

- Usar las mismas credenciales de Azure AD administradas o federadas que utiliza normalmente.
- Ya no hace falta administrar cuentas de administrador locales.
- RBAC de Azure permite conceder el acceso adecuado a las VM en función de las necesidades y quitarlas cuando ya no se necesiten.
- Antes de permitir el acceso a una VM, el acceso condicional Azure AD puede aplicar requisitos adicionales, como: 
   - Multi-Factor Authentication
   - Comprobación de riesgo de inicio de sesión
- Automatice y escale la unión a Azure AD de las máquinas virtuales Windows de Azure que forman parte de las implementaciones de VDI.

> [!NOTE]
> Una vez habilitada esta funcionalidad, las máquinas virtuales de Windows en Azure se unirán a Azure AD. No se puede unir a otro dominio, como AD local o Azure AD DS. Si tiene que hacerlo, tendrá que desconectar la máquina virtual del inquilino de Azure AD desinstalando la extensión.

## <a name="requirements"></a>Requisitos

### <a name="supported-azure-regions-and-windows-distributions"></a>Regiones de Azure compatibles y distribuciones de Windows

La versión preliminar de esta característica actualmente admite estas distribuciones de Windows:

- Windows Server 2019 Datacenter
- Windows 10 1809 y versiones posteriores

> [!IMPORTANT]
> La conexión remota a las VM unidas a Azure AD solo se permite desde equipos con Windows 10 que están unidos a Azure AD o a Azure AD en entornos híbridos al **mismo** directorio que la VM. 

La versión preliminar de esta característica actualmente admite estas regiones de Azure:

- Todas las regiones globales de Azure

> [!IMPORTANT]
> Para usar esta Característica en vista previa (GB), solo debe implementar una distribución de Windows compatible en una región de Azure compatible. Actualmente, la característica no es compatible con Azure Government ni con las nubes soberanas.

### <a name="network-requirements"></a>Requisitos de red

Para habilitar la autenticación de Azure AD para las VM Windows en Azure, debe asegurarse de que la configuración de red de las VM permita el acceso de salida a los siguientes puntos de conexión a través del puerto TCP 443:

- https:\//enterpriseregistration.windows.net
- https:\//login.microsoftonline.com
- https:\//device.login.microsoftonline.com
- https:\//pas.windows.net

## <a name="enabling-azure-ad-login-in-for-windows-vm-in-azure"></a>Habilitar el inicio de sesión de Azure AD en VM Windows en Azure

Para usar el inicio de sesión de Azure AD en VM de Windows en Azure, primero debe habilitar la opción de inicio de sesión de Azure AD para la VM de Windows y luego configurar las asignaciones de roles de Azure para los usuarios que tengan autorización para iniciar sesión en la VM.
Hay varias maneras de habilitar el inicio de sesión de Azure AD para la VM Windows:

- Uso de la experiencia de Azure Portal al crear una VM Windows
- Uso de la experiencia de Azure Cloud Shell al crear una máquina virtual Windows **o en una V de este tipo que ya exista**

### <a name="using-azure-portal-create-vm-experience-to-enable-azure-ad-login"></a>Uso de la experiencia de creación de VM en Azure Portal para habilitar el inicio de sesión de Azure AD

Puede habilitar el inicio de sesión de Azure AD para las imágenes de VM de Windows Server 2019 Datacenter o Windows 10 1809 y versiones posteriores. 

Para crear una VM de Windows Server 2019 Datacenter en Azure con inicio de sesión de Azure AD: 

1. Inicie sesión en el [Azure Portal](https://portal.azure.com) con una cuenta que tenga acceso para crear VM y seleccione **+ Crear un recurso**.
1. Escriba **Windows Server** en el campo de búsqueda de la barra de búsqueda de Marketplace.
   1. Haga clic en **Windows Server** y elija **Windows Server 2019 Datacenter** de la lista desplegable Seleccionar un plan de software.
   1. Haga clic en **Crear**.
1. En la pestaña "Administración", cambie la opción **Iniciar sesión con las credenciales de AAD (versión preliminar)** de la sección Azure Active Directory de Desactivado a **Activado**.
1. Asegúrese de que la opción **Identidad administrada asignada por el sistema** en la sección Identidad se haya establecido en **Activado**. Esta acción debe realizarse automáticamente una vez habilitado el inicio de sesión con credenciales de Azure AD.
1. Pase por el resto de la experiencia de creación de una máquina virtual. Durante esta versión preliminar, tendrá que crear un nombre de usuario y contraseña de administrador para la VM.

![Inicio de sesión con credenciales de Azure AD al crear una VM](./media/howto-vm-sign-in-azure-ad-windows/azure-portal-login-with-azure-ad.png)

> [!NOTE]
> Para iniciar sesión en la VM con sus credenciales de Azure AD, primero debe configurar las asignaciones de rol para la VM tal y como se describe en una de las secciones siguientes.

### <a name="using-the-azure-cloud-shell-experience-to-enable-azure-ad-login"></a>Uso de la experiencia de Azure Cloud Shell para habilitar el inicio de sesión de Azure AD

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Cloud Shell incluye herramientas comunes de Azure preinstaladas y configuradas para que las use con su cuenta. Solo hay que seleccionar el botón Copiar para copiar el código, pegarlo en Cloud Shell y presionar Entrar para que se ejecute. Existen varias maneras de abrir Cloud Shell:

Seleccione Pruébelo en la esquina superior derecha de un bloque de código.
Abra Cloud Shell en el explorador.
Seleccione el botón Cloud Shell en el menú de la esquina superior derecha de [Azure Portal](https://portal.azure.com).

Si decide instalar y usar la CLI localmente, para este artículo es preciso que ejecute la versión 2.0.31 o posterior de la CLI de Azure. Para saber qué versión tiene, ejecute el comando az --version. Si necesita instalar o actualizar, consulte el artículo [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

1. Cree un grupo de recursos con [az group create](/cli/azure/group#az-group-create). 
1. Cree una VM con [az vm create](/cli/azure/vm#az-vm-create) con una distribución compatible en una región compatible. 
1. Instale la extensión de VM para el inicio de sesión de Azure AD. 

En el ejemplo siguiente se implementa una VM denominada myVM que usa Win2019Datacenter en un grupo de recursos denominado myResourceGroup en la región southcentralus. En los ejemplos siguientes, puede proporcionar sus propios nombres de máquinas virtuales y grupos de recursos según sea necesario.

```AzureCLI
az group create --name myResourceGroup --location southcentralus

az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image Win2019Datacenter \
    --assign-identity \
    --admin-username azureuser \
    --admin-password yourpassword
```

> [!NOTE]
> Antes de instalar la extensión de máquina virtual de inicio de sesión de Azure AD es necesario habilitar la identidad administrada asignada por el sistema en la máquina virtual.

La creación de la máquina virtual y los recursos auxiliares tarda unos minutos en realizarse.

Por último, instale la extensión de VM de inicio de sesión de Azure AD para habilitar el inicio de sesión de Azure AD para la VM Windows. Las extensiones de máquina virtual son aplicaciones pequeñas que realizan tareas de automatización y configuración posterior a la implementación en máquinas virtuales de Azure. Use [az vm extension set](/cli/azure/vm/extension#az-vm-extension-set) para instalar la extensión AADLoginForWindows en la Vm denominada myVM en el grupo de recursos myResourceGroup:

> [!NOTE]
> Puede instalar la extensión AADLoginForWindows en una VM existente de Windows Server 2019 o Windows 10 1809 y versiones posteriores para habilitarla para la autenticación de Azure AD. A continuación se muestra un ejemplo de la CLI de AZ.

```AzureCLI
az vm extension set \
    --publisher Microsoft.Azure.ActiveDirectory \
    --name AADLoginForWindows \
    --resource-group myResourceGroup \
    --vm-name myVM
```

Se muestra `provisioningState` de `Succeeded`, una vez que la extensión se instala en la VM.

## <a name="configure-role-assignments-for-the-vm"></a>Configuración de asignaciones de roles para la máquina virtual

Ahora que ha creado la VM, debe configurar la directiva RBAC de Azure para determinar quién puede iniciar sesión en la ella. Para autorizar el inicio de sesión de una VM se usan dos roles de Azure:

- **Inicio de sesión de administrador de Virtual Machine**: Los usuarios que tienen asignado este rol pueden iniciar sesión en una máquina virtual de Azure con privilegios de administrador.
- **Inicio de sesión de usuario de Virtual Machine**: los usuarios que tienen asignado este rol pueden iniciar sesión en una máquina virtual de Azure con privilegios de usuario habitual.

> [!NOTE]
> Para permitir que un usuario inicie sesión en la VM a través de RDP, debe asignar el rol Inicio de sesión de administrador de máquina virtual o Inicio de sesión de usuario de máquina virtual. Un usuario de Azure con los roles Propietario o Colaborador asignados para una VM no tienen automáticamente privilegios para iniciar sesión en la VM a través de RDP. Esto es para proporcionar una separación auditada entre el conjunto de personas que controlan máquinas virtuales frente al conjunto de personas que pueden tener acceso a las máquinas virtuales.

Hay varias maneras de configurar las asignaciones de roles para la VM:

- Mediante la experiencia de Azure AD Portal
- Mediante la experiencia de Azure Cloud Shell

### <a name="using-azure-ad-portal-experience"></a>Mediante la experiencia de Azure AD Portal

Para configurar las asignaciones de roles para las VM de Windows Server 2019 Datacenter compatibles con Azure AD:

1. Vaya a la página de información general de la máquina virtual específica
1. Seleccione **Control de acceso (IAM)** en las opciones de menú
1. Seleccione **Agregar**, **Agregar asignación de rol** para abrir el panel Agregar asignación de rol.
1. En la lista desplegable **Rol**, seleccione un rol, como **Inicio de sesión de administrador de máquina virtual** o **Inicio de sesión de usuario de máquina virtual**.
1. En la lista **Seleccionar**, seleccione un usuario, grupo, entidad de servicio o identidad administrada. Si no ve la entidad de seguridad en la lista, puede escribir en el cuadro **Seleccionar** para buscar nombres para mostrar, direcciones de correo electrónico e identificadores de objeto en el directorio.
1. Seleccione **Guardar** para asignar el rol.

Transcurridos unos instantes, se asigna el rol a la entidad de seguridad en el ámbito seleccionado.

![Asignar roles a los usuarios que accederán a la VM](./media/howto-vm-sign-in-azure-ad-windows/azure-portal-access-control-assign-role.png)

### <a name="using-the-azure-cloud-shell-experience"></a>Mediante la experiencia de Azure Cloud Shell

En el ejemplo siguiente se usa [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create) para asignar el rol Inicio de sesión de administrador de Virtual Machine a la VM para el usuario de Azure actual. El nombre de usuario de la cuenta de Azure activa se obtiene con [az account show](/cli/azure/account#az-account-show) y el ámbito se establece en la VM que se creó en un paso anterior con [az vm show](/cli/azure/vm#az-vm-show). El ámbito también se podría asignar en el nivel de un grupo de recursos o de suscripción, y se aplican los permisos de herencia de RBAC de Azure normales. Para obtener información, consulte [Inicio de sesión en una máquina virtual Linux en Azure mediante la autenticación de Azure Active Directory (versión preliminar)](../../virtual-machines/linux/login-using-aad.md).

```   AzureCLI
username=$(az account show --query user.name --output tsv)
vm=$(az vm show --resource-group myResourceGroup --name myVM --query id -o tsv)

az role assignment create \
    --role "Virtual Machine Administrator Login" \
    --assignee $username \
    --scope $vm
```

> [!NOTE]
> Si su dominio de AAD y el dominio del nombre de usuario de inicio de sesión no coinciden, debe especificar el identificador de objeto de su cuenta de usuario mediante `--assignee-object-id`, y no solo el nombre de usuario para `--assignee`. Puede obtener el identificador de objeto para su cuenta de usuario mediante [az ad user list](/cli/azure/ad/user#az-ad-user-list).

Para obtener más información sobre cómo usar RBAC de Azure para administrar el acceso a los recursos de la suscripción de Azure, consulte los siguientes artículos:

- [Incorporación o eliminación de asignaciones de roles de Azure mediante la CLI de Azure](../../role-based-access-control/role-assignments-cli.md)
- [Incorporación o eliminación de asignaciones de roles de Azure con Azure Portal](../../role-based-access-control/role-assignments-portal.md)
- [Incorporación o eliminación de asignaciones de roles de Azure con Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md)

## <a name="using-conditional-access"></a>Uso del acceso condicional

Puede aplicar directivas de acceso condicional, como la autenticación multifactor o la comprobación de riesgo de inicio de sesión de usuario antes de autorizar el acceso a máquinas virtuales Windows en Azure que están habilitadas con el inicio de sesión de Azure AD. Para aplicar la directiva de acceso condicional, debe seleccionar la aplicación de inicio de sesión de máquinas virtuales Windows de Azure desde la opción de asignación de acciones o aplicaciones en la nube y usar Riesgo de inicio de sesión como condición, o bien requerir la autenticación multifactor como un control de acceso de concesión. 

> [!NOTE]
> Si usa "Requerir autenticación multifactor" como control de acceso de concesión para solicitar acceso a la aplicación de inicio de sesión de máquinas virtuales Windows de Azure, debe proporcionar una notificación de autenticación multifactor como parte del cliente que inicia la sesión RDP en la máquina virtual Windows de destino en Azure. La única manera de hacer esto en un cliente de Windows 10 es usar el PIN de Windows Hello para empresas o la autenticación biométrica con el cliente RDP. En la versión 1809 de Windows 10 se agregó compatibilidad con la autenticación biométrica al cliente RDP. El escritorio remoto que usa la autenticación de Windows Hello para empresas solo está disponible para implementaciones que emplean el modelo de confianza de certificados. Actualmente no está disponible para el modelo de confianza de claves.

> [!WARNING]
> Azure Multi-Factor Authentication habilitado o forzado por el usuario no es compatible con el inicio de sesión de la máquina virtual.

## <a name="log-in-using-azure-ad-credentials-to-a-windows-vm"></a>Inicio de sesión mediante las credenciales de Azure AD en una VM Windows

> [!IMPORTANT]
> La conexión remota a las VM unidas a Azure AD solo se permite desde equipos con Windows 10 que están registrados en Azure AD (la compilación mínima requerida es 20H1), unidos a Azure AD o unidos a Azure AD híbrido en el **mismo** directorio que la máquina virtual. Además, para que RDP pueda usar las credenciales de Azure AD, el usuario debe pertenecer a uno de estos dos roles de Azure: Inicio de sesión de administrador de la máquina virtual o Inicio de sesión de usuario de la máquina virtual. Si usa un equipo con Windows 10 registrado en Azure AD, debe escribir las credenciales con el formato AzureAD\UPN (por ejemplo, AzureAD\john@contoso.com). En este momento, no se puede usar Azure Bastion para iniciar sesión mediante la autenticación de Azure Active Directory con la extensión AADLoginForWindows; solo se admite RDP directo.

Para iniciar sesión en la máquina virtual Windows Server 2019 mediante Azure AD: 

1. Vaya a la página de información general de la máquina virtual que se ha habilitado con inicio de sesión de Azure AD.
1. Seleccione **Conectar** para abrir la hoja Conectarse a una máquina virtual.
1. Seleccione **Descargar archivo RDP**.
1. Seleccione **Abrir** para iniciar el cliente Conexión a Escritorio remoto.
1. Seleccione **Conectar** para iniciar el cuadro de diálogo de inicio de sesión de Windows.
1. Inicie sesión con sus credenciales de Azure AD.

Habrá iniciado sesión en la máquina virtual Windows Server 2019 de Azure con los permisos de rol asignados, como Usuario de VM o Administrador de VM. 

> [!NOTE]
> Puede guardar el archivo .RDP localmente en su equipo e iniciar conexiones de escritorio remoto futuras a la máquina virtual en lugar de tener que navegar a la página de información general de la máquina virtual en Azure Portal y usar la opción Conectar.

## <a name="troubleshoot"></a>Solución de problemas

### <a name="troubleshoot-deployment-issues"></a>Solución de problemas de implementación

Para que la VM complete el proceso de unión a Azure AD, la extensión AADLoginForWindows se debe instalar correctamente. Si la extensión de la VM no se instala correctamente, realice los pasos siguientes.

1. Establezca una conexión RDP a la máquina virtual con la cuenta de administrador local y examine el registro CommandExecution.log en  
   
   C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.ActiveDirectory.AADLoginForWindows\0.3.1.0. 

   > [!NOTE]
   > Si la extensión se reinicia después del error inicial, el registro con el error de implementación se guardará como CommandExecution_AAAAMMDDHHMMSSSSS.log. "
1. Abra un símbolo del sistema en la VM y compruebe estas consultas con el punto de conexión de Instance Metadata Service (IMDS) que se ejecuta en el host de Azure:

   | Comando para ejecutar | Salida prevista |
   | --- | --- |
   | `curl -H @{"Metadata"="true"} "http://169.254.169.254/metadata/instance?api-version=2017-08-01"` | Información correcta sobre la VM de Azure |
   | `curl -H @{"Metadata"="true"} "http://169.254.169.254/metadata/identity/info?api-version=2018-02-01"` | Id. de inquilino válido asociado con la suscripción a Azure |
   | `curl -H @{"Metadata"="true"} "http://169.254.169.254/metadata/identity/oauth2/token?resource=urn:ms-drs:enterpriseregistration.windows.net&api-version=2018-02-01"` | Token de acceso válido que emite Azure Active Directory para la identidad administrada asignada a esta VM |

   > [!NOTE]
   > El token de acceso se puede descodificar mediante una herramienta, como [http://calebb.net/](http://calebb.net/). Compruebe que el objeto "appid" del token de acceso coincide con la identidad administrada asignada a la VM.

1. Asegúrese de que los punto de conexión necesarios sean accesibles desde la VM. Para ello, use la siguiente línea de comandos:
   
   - curl https:\//login.microsoftonline.com/ -D –
   - curl https:\//login.microsoftonline.com/`<TenantID>`/ -D –

   > [!NOTE]
   > Reemplace `<TenantID>` por el identificador de inquilino Azure AD que está asociado a la suscripción de Azure.

   - curl https:\//enterpriseregistration.windows.net/ -D -
   - curl https:\//device.login.microsoftonline.com/ -D -
   - curl https:\//pas.windows.net/ -D -

1. El estado del dispositivo se puede ver al ejecutar `dsregcmd /status`. El objetivo es que el estado del dispositivo se muestre como `AzureAdJoined : YES`.

   > [!NOTE]
   > La actividad de unión a Azure AD se captura en el visor de eventos, en el registro de dispositivos Registration\Admin.

Si se produce un error en la extensión AADLoginForWindows con determinado código de error, puede realizar los pasos siguientes:

#### <a name="issue-1-aadloginforwindows-extension-fails-to-install-with-terminal-error-code-1007-and-exit-code--2145648574"></a>Problema 1: La extensión AADLoginForWindows no se puede instalar con el código de error de terminal "1007" y el código de salida:-2145648574.

Este código de salida se convierte en DSREG_E_MSI_TENANTID_UNAVAILABLE porque la extensión no puede consultar la información del inquilino de Azure AD.

1. Compruebe que la VM de Azure puede recuperar el id. de inquilino de Instance Metadata Service.

   - Establezca una conexión RDP a la VM como administrador local y compruebe que el punto de conexión devuelve un id. de inquilino válido. Para ello, ejecute este comando desde una línea de comandos con privilegios elevados en la VM:
      
      - curl -H Metadata:true http://169.254.169.254/metadata/identity/info?api-version=2018-02-01

1. El administrador de la VM intenta instalar la extensión AADLoginForWindows, pero una identidad administrada asignada por el sistema no ha habilitado la VM en primer lugar. Vaya a la hoja identidad de la VM. En la pestaña Asignado por el sistema, compruebe que el estado es Activado.

#### <a name="issue-2-aadloginforwindows-extension-fails-to-install-with-exit-code--2145648607"></a>Problema 2: La extensión AADLoginForWindows no se puede instalar con el código de salida:-2145648607

Este código de salida se convierte en DSREG_AUTOJOIN_DISC_FAILED porque la extensión no puede alcanzar el punto de conexión de `https://enterpriseregistration.windows.net`.

1. Compruebe que los punto de conexión necesarios son accesibles desde la VM. Para ello, use la siguiente línea de comandos:

   - curl https:\//login.microsoftonline.com/ -D –
   - curl https:\//login.microsoftonline.com/`<TenantID>`/ -D –
   
   > [!NOTE]
   > Reemplace `<TenantID>` por el identificador de inquilino Azure AD que está asociado a la suscripción de Azure. Si necesita buscar el id. de inquilino, puede mantener el mouse sobre el nombre de la cuenta para obtener el id. de directorio o inquilino, o bien seleccionar Azure Active Directory > Propiedades > Id. de directorio en Azure Portal.

   - curl https:\//enterpriseregistration.windows.net/ -D -
   - curl https:\//device.login.microsoftonline.com/ -D -
   - curl https:\//pas.windows.net/ -D -

1. Si en cualquiera de los comandos se produce el error "No se pudo resolver el host `<URL>`", intente ejecutar este comando para determinar el servidor DNS que la VM usa.
   
   `nslookup <URL>`

   > [!NOTE] 
   > Reemplace `<URL>` por los nombres de dominio completos que los puntos de conexión usan, como "login.microsoftonline.com".

1. A continuación, compruebe que la especificación de un servidor DNS público permite que el comando se ejecute correctamente:

   `nslookup <URL> 208.67.222.222`

1. Si fuera necesario, cambie el servidor DNS asignado al grupo de seguridad de red al que pertenece la VM de Azure.

#### <a name="issue-3-aadloginforwindows-extension-fails-to-install-with-exit-code-51"></a>Problema 3: La extensión AADLoginForWindows no se puede instalar con el código de salida: 51

El código de salida 51 se traduce en "esta extensión no se admite en el sistema operativo de la VM".

En la versión preliminar pública, la extensión AADLoginForWindows solo está diseñada para instalarse en Windows Server 2019 o Windows 10 (compilación 1809 o posterior). Asegúrese de que se admita la versión de Windows. Si no se admite la compilación de Windows, desinstale la extensión de VM.

### <a name="troubleshoot-sign-in-issues"></a>Solución de problemas con el inicio de sesión

Entre algunos de errores comunes al intentar usar RDP con las credenciales de Azure AD se incluyen la falta de asignación de roles de Azure, que haya clientes no autorizados o que se necesiten métodos de inicio de sesión de autenticación en dos fases. Use la siguiente información para corregir estos problemas.

Para ver el estado del dispositivo y SSO, ejecute `dsregcmd /status`. El objetivo es que el estado del dispositivo se muestre como `AzureAdJoined : YES` y `SSO State` para mostrar `AzureAdPrt : YES`.

Además, el inicio de sesión de RDP mediante cuentas de Azure AD se captura en el visor de eventos, en los registros de eventos AAD\Operational.

#### <a name="azure-role-not-assigned"></a>No hay ningún rol de Azure asignado

Si ve el siguiente mensaje de error al iniciar una conexión de Escritorio remoto a la VM: 

- Su cuenta está configurada para impedir el uso de este dispositivo. Para más información, póngase en contacto con el administrador del sistema.

![Su cuenta está configurada para impedir el uso de este dispositivo.](./media/howto-vm-sign-in-azure-ad-windows/rbac-role-not-assigned.png)

Compruebe que ha [configurado directivas de RBAC de Azure](../../virtual-machines/linux/login-using-aad.md) para la VM que concedan al usuario los roles de Inicio de sesión de administrador de máquina virtual o Inicio de sesión de usuario de máquina virtual:
 
#### <a name="unauthorized-client"></a>Cliente no autorizado

Si ve el siguiente mensaje de error al iniciar una conexión de Escritorio remoto a la VM: 

- Las credenciales no funcionaron.

![Las credenciales no funcionaron.](./media/howto-vm-sign-in-azure-ad-windows/your-credentials-did-not-work.png)

Compruebe que el equipo con Windows 10 que usa para iniciar la conexión de Escritorio remoto es uno de los unidos a Azure AD o unidos al entorno híbrido de Azure AD al mismo directorio de Azure AD al que se una la VM. Para más información sobre la identidad del dispositivo, consulte el artículo [¿Qué es una identidad de dispositivo?](./overview.md)

> [!NOTE]
> La compilación 20H1 de Windows 10 agregó compatibilidad con un equipo registrado en Azure AD para iniciar la conexión RDP a la máquina virtual. Al usar un equipo registrado en Azure AD (no unido a Azure AD ni unido a Azure AD híbrido) como el cliente RDP para iniciar conexiones a la máquina virtual, debe escribir las credenciales con el formato AzureAD\UPn (por ejemplo, AzureAD\john@contoso.com).

Además, compruebe que la extensión AADLoginForWindows no se ha desinstalado una vez que completada la unión a Azure AD.
 
#### <a name="mfa-sign-in-method-required"></a>Se requiere el método de inicio de sesión MFA

Si ve el siguiente mensaje de error al iniciar una conexión de Escritorio remoto a la VM: 

- No se permite el método de inicio de sesión que intenta usar. Pruebe con otro método de inicio de sesión o póngase en contacto con el administrador del sistema.

![No se permite el método de inicio de sesión que intenta usar.](./media/howto-vm-sign-in-azure-ad-windows/mfa-sign-in-method-required.png)

Si configuró una directiva de acceso condicional que requiere que la autenticación multifactor (MFA) se realice antes de poder acceder al recurso, debe asegurarse de que el equipo Windows 10 que inicia la conexión de Escritorio remoto a la VM inicie sesión con un método de autenticación seguro, como Windows Hello. Si no usa un método de autenticación seguro para la conexión de Escritorio remoto, verá el anterior error.

Si no ha implementado Windows Hello para empresas ni se plantea hacerlo por ahora, puede excluir el requisito de MFA configurando la directiva de acceso condicional que excluye la aplicación de inicio de sesión de máquinas virtuales Windows de Azure en la lista de aplicaciones en la nube que requieren MFA. Para obtener más información sobre Windows Hello para empresas, consulte [Información general de Windows Hello para empresas](/windows/security/identity-protection/hello-for-business/hello-identity-verification).

> [!NOTE]
> La autenticación con el PIN de Windows Hello para empresas con RDP es compatible con varias versiones de Windows 10. También se ha agregado compatibilidad con la autenticación biométrica con RDP en la versión 1809 de Windows 10. El uso de la autenticación de Windows Hello para empresas durante RDP solo está disponible para las implementaciones que usan el modelo de confianza de certificados y actualmente no están disponibles para el modelo de confianza de claves.
 
## <a name="preview-feedback"></a>Comentarios sobre la versión preliminar

Comparta sus comentarios sobre esta Característica en vista previa (GB) o notifique cualquier problema mediante el [foro de comentarios de Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=166032).

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure Active Directory, consulte [¿Qué es Azure Active Directory?](../fundamentals/active-directory-whatis.md)
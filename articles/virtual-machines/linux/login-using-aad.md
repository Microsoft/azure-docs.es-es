---
title: Inicio de sesión en una máquina virtual Linux con credenciales de Azure Active Directory
description: Aprenda a crear y configurar una máquina virtual Linux para iniciar sesión con la autenticación de Azure Active Directory.
author: SanDeo-MSFT
ms.service: virtual-machines-linux
ms.topic: how-to
ms.workload: infrastructure
ms.date: 08/29/2019
ms.author: sandeo
ms.openlocfilehash: 96fb914b5dafe5eb818f2b491bbe2d856763bd02
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87534743"
---
# <a name="preview-log-in-to-a-linux-virtual-machine-in-azure-using-azure-active-directory-authentication"></a>Vista previa: Inicio de sesión en una máquina virtual Linux en Azure mediante la autenticación de Azure Active Directory

Para mejorar la seguridad de las máquinas virtuales Linux en Azure, puede integrarla con la autenticación de Azure Active Directory (AD). Cuando usa la autenticación de Azure AD para las máquinas virtuales Linux, administra y aplica de manera centralizada las directivas que permiten o deniegan el acceso a las máquinas virtuales. En este artículo se muestra cómo crear y configurar una máquina virtual Linux para usar la autenticación de Azure AD.


> [!IMPORTANT]
> La autenticación de Azure Active Directory se encuentra actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
> Use esta característica en una máquina virtual de prueba que espere descartar después de realizar pruebas.
>


Usar la autenticación de Azure AD para iniciar sesión en máquinas virtuales Linux en Azure implica varias ventajas, entre las que se incluyen:

- **Seguridad mejorada:**
  - Puede usar las credenciales corporativas de AD para iniciar sesión en máquinas virtuales Linux de Azure. No necesita crear cuentas de administrador local ni administrar la vigencia de las credenciales.
  - Como se disminuye la dependencia de las cuentas de administrador local, no tiene que preocuparse de la pérdida o el robo de las credenciales ni tampoco si los usuarios configuraron credenciales poco seguras, etc.
  - Las directivas de complejidad y vigencia de las contraseñas configuradas para el directorio de Azure AD también ayudan a proteger las máquinas virtuales Linux.
  - Para proteger aún más el inicio de sesión en máquinas virtuales de Azure, puede configurar la autenticación multifactor.
  - La capacidad de iniciar sesión en máquinas virtuales Linux con Azure Active Directory también funciona con clientes que usan [Servicios de federación](../../active-directory/hybrid/how-to-connect-fed-whatis.md).

- **Colaboración sin complicaciones**: con el control de acceso basado en rol (RBAC), puede especificar quién puede iniciar sesión en una máquina virtual determinada como usuario habitual o con privilegios de administrador. Cuando los usuarios se unen o dejan el equipo, puede actualizar la directiva de RBAC de la máquina virtual para conceder acceso según corresponda. Esta experiencia es mucho más simple que limpiar las máquinas virtuales para quitar las claves SSH públicas innecesarias. Cuando los empleados dejan la organización y su cuenta de usuario se deshabilita o quita de Azure AD, dejan de tener acceso a los recursos.

## <a name="supported-azure-regions-and-linux-distributions"></a>Regiones de Azure y distribuciones de Linux compatibles

La versión preliminar de esta característica actualmente admite estas distribuciones de Linux:

| Distribución | Versión |
| --- | --- |
| CentOS | CentOS 6, CentOS 7 |
| Debian | Debian 9 |
| openSUSE | openSUSE Leap 42.3 |
| RedHat Enterprise Linux | RHEL 6, RHEL 7 | 
| SUSE Linux Enterprise Server | SLES 12 |
| Ubuntu Server | Ubuntu 14.04 LTS, Ubuntu Server 16.04 y Ubuntu Server 18.04 |


La versión preliminar de esta característica actualmente admite estas regiones de Azure:

- Todas las regiones de Azure global

>[!IMPORTANT]
> Para usar esta característica en versión preliminar, solo debe implementar una distribución de Linux compatible en una región de Azure compatible. La característica no es compatible con Azure Government ni con las nubes soberanas.
>
> No se admite el uso de esta extensión en clústeres de Azure Kubernetes Service (AKS). Para obtener más información, consulte [Directivas de soporte técnico para AKS](../../aks/support-policies.md).


Si elige instalar y usar la CLI localmente, para este tutorial es preciso que ejecute la CLI de Azure de la versión 2.0.31 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="network-requirements"></a>Requisitos de red

Para habilitar la autenticación de Azure AD para las máquinas virtuales Windows en Azure es preciso asegurarse de que la configuración de red de dichas máquinas permita el acceso de salida a los siguientes puntos de conexión a través del puerto TCP 443:

* https:\//login.microsoftonline.com
* https:\//login.windows.net
* https:\//device.login.microsoftonline.com
* https:\//pas.windows.net
* https:\//management.azure.com
* https:\//packages.microsoft.com

> [!NOTE]
> Actualmente, los grupos de seguridad de red de Azure no se pueden configurar en máquinas virtuales que tengan habilitada la autenticación de Azure AD.

## <a name="create-a-linux-virtual-machine"></a>Creación de una máquina virtual con Linux

Cree un grupo de recursos con [az group create](/cli/azure/group#az-group-create) y luego cree una máquina virtual con [az vm create](/cli/azure/vm#az-vm-create), mediante una distribución compatible en una región compatible. En el ejemplo siguiente se implementa una máquina virtual denominada *myVM* que usa *Ubuntu 16.04 LTS* en un grupo de recursos denominado *myResourceGroup* en la región *southcentralus*. En los ejemplos siguientes, puede proporcionar sus propios nombres de máquinas virtuales y grupos de recursos según sea necesario.

```azurecli-interactive
az group create --name myResourceGroup --location southcentralus

az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys
```

La creación de la máquina virtual y los recursos auxiliares tarda unos minutos en realizarse.

## <a name="install-the-azure-ad-login-vm-extension"></a>Instalación de la extensión de máquina virtual para el inicio de sesión de Azure AD

> [!NOTE]
> Si implementa esta extensión en una máquina virtual creada anteriormente, asegúrese de que la máquina tenga al menos 1 GB de memoria asignada o la extensión no se instalará

Para iniciar sesión en una máquina virtual Linux con credenciales de Azure AD, instale la extensión de máquina virtual de inicio de sesión de Azure Active Directory. Las extensiones de máquina virtual son aplicaciones pequeñas que realizan tareas de automatización y configuración posterior a la implementación en máquinas virtuales de Azure. Use [az vm extension set](/cli/azure/vm/extension#az-vm-extension-set) para instalar la extensión *AADLoginForLinux* en la máquina virtual denominada *myVM* en el grupo de recursos *myResourceGroup*:

```azurecli-interactive
az vm extension set \
    --publisher Microsoft.Azure.ActiveDirectory.LinuxSSH \
    --name AADLoginForLinux \
    --resource-group myResourceGroup \
    --vm-name myVM
```

El estado *provisioningState* con valor *Succeeded* aparece una vez que la extensión se instala correctamente en la máquina virtual. La máquina virtual necesita un agente de máquina virtual en ejecución para instalar la extensión. Para más información, consulte [Introducción al agente de máquina virtual](../extensions/agent-windows.md).

## <a name="configure-role-assignments-for-the-vm"></a>Configuración de asignaciones de roles para la máquina virtual

La directiva de control de acceso basado en rol (Azure RBAC) de Azure determina quién puede iniciar sesión en la VM. Para autorizar el inicio de sesión de una máquina virtual se usan dos roles de Azure:

- **Inicio de sesión de administrador de Virtual Machine**: los usuarios que tienen asignado este rol pueden iniciar sesión en una máquina virtual de Azure con privilegios de usuario raíz de Linux o de administrador de Windows.
- **Inicio de sesión de usuario de Virtual Machine**: los usuarios que tienen asignado este rol pueden iniciar sesión en una máquina virtual de Azure con privilegios de usuario habitual.

> [!NOTE]
> Para permitir que un usuario inicie sesión en la máquina virtual a través de SSH, debe asignar el rol *Inicio de sesión de administrador de Virtual Machine* o *Inicio de sesión de usuario de Virtual Machine*. Un usuario de Azure con los roles de *Propietario* o *Colaborador* asignados para una máquina virtual no tienen automáticamente privilegios para iniciar sesión en la máquina virtual a través de SSH.

En el ejemplo siguiente se usa [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create) para asignar el rol *Inicio de sesión de administrador de Virtual Machine* a la máquina virtual para el usuario de Azure actual. El nombre de usuario de la cuenta de Azure activa se obtiene con [az account show](/cli/azure/account#az-account-show) y el *ámbito* se establece en la máquina virtual que se creó en un paso anterior con [az vm show](/cli/azure/vm#az-vm-show). El ámbito también se podría asignar en el nivel de un grupo de recursos o de suscripción y se aplican los permisos de herencia de RBAC normales. Para más información, consulte la información sobre los [controles de acceso basado en rol](../../role-based-access-control/overview.md).

```azurecli-interactive
username=$(az account show --query user.name --output tsv)
vm=$(az vm show --resource-group myResourceGroup --name myVM --query id -o tsv)

az role assignment create \
    --role "Virtual Machine Administrator Login" \
    --assignee $username \
    --scope $vm
```

> [!NOTE]
> Si su dominio de AAD y el dominio del nombre de usuario de inicio de sesión no coinciden, debe especificar el identificador de objeto de su cuenta de usuario mediante *--assignee-object-id* y no solo el nombre de usuario para *--assignee*. Puede obtener el identificador de objeto para su cuenta de usuario mediante [az ad user list](/cli/azure/ad/user#az-ad-user-list).

Para más información sobre cómo usar RBAC para administrar el acceso a los recursos de suscripción de Azure, consulte cómo usar la [CLI de Azure](../../role-based-access-control/role-assignments-cli.md), [Azure Portal](../../role-based-access-control/role-assignments-portal.md) o [Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md).

También puede configurar Azure AD para que requiera la autenticación multifactor para que un usuario específico inicie sesión en la máquina virtual Linux. Para más información, consulte [Introducción a Azure Multi-Factor Authentication en la nube](../../active-directory/authentication/howto-mfa-getstarted.md).

## <a name="log-in-to-the-linux-virtual-machine"></a>Inicio de sesión en la máquina virtual Linux

En primer lugar, use [az vm show](/cli/azure/vm#az-vm-show) para ver la dirección IP pública de la máquina virtual:

```azurecli-interactive
az vm show --resource-group myResourceGroup --name myVM -d --query publicIps -o tsv
```

Inicie sesión en la máquina virtual Linux de Azure con sus credenciales de Azure AD. El parámetro `-l` le permite especificar la dirección de su propia cuenta de Azure AD. Reemplace la cuenta de ejemplo por la suya propia. Las direcciones de la cuenta deben especificarse completamente en minúscula. Reemplace la dirección IP de ejemplo por la dirección IP pública de la máquina virtual del comando anterior.

```console
ssh -l azureuser@contoso.onmicrosoft.com 10.11.123.456
```

Se le pedirá que inicie sesión en Azure AD con un código de un solo uso en [https://microsoft.com/devicelogin](https://microsoft.com/devicelogin). Copie y pegue el código de uso único en la página de inicio de sesión del dispositivo.

Cuando se le solicite, escriba las credenciales de inicio de sesión de Azure AD en la página de inicio de sesión. 

Si se autentica de manera correcta, aparecerá el mensaje siguiente en el explorador web: `You have signed in to the Microsoft Azure Linux Virtual Machine Sign-In application on your device.`.

Cierre la ventana del explorador, vuelva al símbolo del sistema SSH y presione la tecla **Entrar**. 

Ya inició sesión en la máquina virtual Linux de Azure con los permisos de rol asignados, como *Usuario de VM* o *Administrador de VM*. Si la cuenta de usuario tiene asignado el rol *Inicio de sesión de administrador de máquina virtual*, puede usar `sudo` para ejecutar los comandos que requieren privilegios raíz.

## <a name="sudo-and-aad-login"></a>Inicio de sesión de sudo y AAD

La primera vez que ejecute sudo, se le pedirá autenticarse una segunda vez. Si no quiere tener que volver a autenticarse para ejecutar sudo, puede editar el archivo de sudoers `/etc/sudoers.d/aad_admins` y reemplazar esta línea:

```bash
%aad_admins ALL=(ALL) ALL
```

por esta otra:

```bash
%aad_admins ALL=(ALL) NOPASSWD:ALL
```


## <a name="troubleshoot-sign-in-issues"></a>Solución de problemas con el inicio de sesión

Algunos errores comunes que se producen al intentar establecer SSH con las credenciales de Azure AD incluyen no tener asignado ningún rol de Azure y solicitudes repetidas para iniciar sesión. Use las secciones siguientes para corregir estos problemas.

### <a name="access-denied-azure-role-not-assigned"></a>Acceso denegado: no hay ningún rol de Azure asignado

Si ve el error siguiente en el aviso de SSH, compruebe que ha configurado directivas de RBAC configuradas para la máquina virtual que concede al usuario los roles de *Inicio de sesión de administrador de máquina virtual* o *Inicio de sesión de usuario de máquina virtual*:

```output
login as: azureuser@contoso.onmicrosoft.com
Using keyboard-interactive authentication.
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code FJX327AXD to authenticate. Press ENTER when ready.
Using keyboard-interactive authentication.
Access denied:  to sign-in you be assigned a role with action 'Microsoft.Compute/virtualMachines/login/action', for example 'Virtual Machine User Login'
Access denied
```

### <a name="continued-ssh-sign-in-prompts"></a>Solicitudes de inicio de sesión SSH continuas

Si completa correctamente el paso de autenticación en un explorador web, es posible que se le pida de inmediato que inicie sesión nuevamente con un código nuevo. Por lo general, este error se debe a una falta de coincidencia entre el nombre de inicio de sesión que especificó en el símbolo del sistema SSH y la cuenta con que inició sesión en Azure AD. Para corregir este error:

- Compruebe que el nombre de inicio de sesión que especificó en el símbolo del sistema SSH sea correcto. Un error tipográfico en el nombre de inicio de sesión puede provocar una falta de coincidencia entre el nombre de inicio de sesión que especificó en el símbolo del sistema SSH y la cuenta con que inició sesión en Azure AD. Por ejemplo, ha escrito *azuresuer\@contoso.onmicrosoft.com* en lugar de *azureuser\@contoso.onmicrosoft.com*.
- Si tiene varias cuentas de usuario, asegúrese de no escribir una distinta en la ventana del explorador cuando inicie sesión en Azure AD.
- Linux es un sistema operativo que distingue mayúsculas de minúsculas. Hay una diferencia entre "Azureuser@contoso.onmicrosoft.com" y "azureuser@contoso.onmicrosoft.com", lo que puede provocar un error de coincidencia. Asegúrese de especificar el UPN con el uso correcto de mayúsculas y minúsculas en el símbolo del sistema SSH.

### <a name="other-limitations"></a>Otras limitaciones

Actualmente no se admiten los usuarios que hereden los derechos de acceso a través de grupos anidados o asignaciones de roles. Al usuario o grupo se les debe asignar directamente los [roles necesarios](#configure-role-assignments-for-the-vm). Por ejemplo, el uso de grupos de administración o asignaciones de roles de grupos anidados no concederá los permisos correctos que permitan que el usuario inicie sesión.

## <a name="preview-feedback"></a>Comentarios sobre la versión preliminar

Comparta sus comentarios sobre esta característica en versión preliminar o notifique cualquier problema mediante el [foro de comentarios de Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=166032).

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure Active Directory, consulte [¿Qué es Azure Active Directory?](../../active-directory/fundamentals/active-directory-whatis.md)

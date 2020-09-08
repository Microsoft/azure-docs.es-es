---
title: 'Creación de un grupo de hosts de Windows Virtual Desktop con PowerShell: Azure'
description: Cómo crear un grupo de hosts en Windows Virtual Desktop con cmdlets de PowerShell.
author: Heidilohr
ms.topic: how-to
ms.date: 08/11/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 1275eab36e21ea6befdda13e14759a30ef5398a3
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88121160"
---
# <a name="create-a-windows-virtual-desktop-host-pool-with-powershell"></a>Creación de un grupo de hosts de Windows Virtual Desktop con PowerShell

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop con objetos de Windows Virtual Desktop de Azure Resource Manager. Si usa Windows Virtual Desktop (clásico) sin objetos de Azure Resource Manager, consulte [este artículo](./virtual-desktop-fall-2019/create-host-pools-powershell-2019.md).

Los grupos hosts son una colección de una o más máquinas virtuales idénticas en entornos de inquilino de Windows Virtual Desktop. Cada grupo de hosts puede estar asociado a varios grupos de RemoteApp, un grupo de aplicaciones de escritorio y varios hosts de sesión.

## <a name="prerequisites"></a>Prerrequisitos

En este artículo se supone que ya ha seguido las instrucciones del artículo [Configuración del módulo de PowerShell](powershell-module.md).

## <a name="use-your-powershell-client-to-create-a-host-pool"></a>Uso del cliente de PowerShell para crear un grupo hosts

Ejecute el siguiente cmdlet para iniciar sesión en el entorno de Windows Virtual Desktop:

```powershell
New-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -WorkspaceName <workspacename> -HostPoolType <Pooled|Personal> -LoadBalancerType <BreadthFirst|DepthFirst|Persistent> -Location <region> -DesktopAppGroupName <appgroupname>
```

Este cmdlet creará el grupo de hosts, el área de trabajo y el grupo de aplicaciones de escritorio. Además, registrará el grupo de aplicaciones de escritorio en el área de trabajo. Solo puede crear un área de trabajo con este cmdlet o utilizar un área de trabajo existente.

Ejecute el siguiente cmdlet para crear un token de registro para autorizar que un host de sesión se una el grupo de hosts y guárdelo en un archivo nuevo en el equipo local. Puede especificar cuánto tiempo es válido el token de registro mediante el parámetro -ExpirationHours.

>[!NOTE]
>La fecha de expiración del token no puede ser inferior a una hora ni superior a un mes. Si establece *-ExpirationTime* fuera de ese límite, el cmdlet no creará el token.

```powershell
New-AzWvdRegistrationInfo -ResourceGroupName <resourcegroupname> -HostPoolName <hostpoolname> -ExpirationTime $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
```

Por ejemplo, si quiere crear un token que expire al cabo de dos horas, ejecute este cmdlet:

```powershell
New-AzWvdRegistrationInfo -ResourceGroupName <resourcegroupname> -HostPoolName <hostpoolname> -ExpirationTime $((get-date).ToUniversalTime().AddHours(2).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
```

Después, ejecute este cmdlet para agregar usuarios de Azure Active Directory al grupo de aplicaciones de escritorio predeterminado para el grupo de hosts.

```powershell
New-AzRoleAssignment -SignInName <userupn> -RoleDefinitionName "Desktop Virtualization User" -ResourceName <hostpoolname+"-DAG"> -ResourceGroupName <resourcegroupname> -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
```

Luego, ejecute este cmdlet para agregar grupos de usuarios de Azure Active Directory al grupo de aplicaciones de escritorio predeterminado del grupo de hosts.

```powershell
New-AzRoleAssignment -ObjectId <usergroupobjectid> -RoleDefinitionName "Desktop Virtualization User" -ResourceName <hostpoolname+"-DAG"> -ResourceGroupName <resourcegroupname> -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
```

Ejecute el siguiente cmdlet para exportar el token de registro a una variable, que usará más adelante en el [Registro de máquinas virtuales en el grupo de hosts de Windows Virtual Desktop](#register-the-virtual-machines-to-the-windows-virtual-desktop-host-pool).

```powershell
$token = Get-AzWvdRegistrationInfo -ResourceGroupName <resourcegroupname> -HostPoolName <hostpoolname>
```

## <a name="create-virtual-machines-for-the-host-pool"></a>Creación de máquinas virtuales para el grupo de hosts

Ahora puede crear una máquina virtual de Azure que puede unirse al grupo de hosts de Windows Virtual Desktop.

Puede crear una máquina virtual de varias maneras:

- [Crear una máquina virtual desde una imagen de la galería de Azure](../virtual-machines/windows/quick-create-portal.md#create-virtual-machine)
- [Crear una máquina virtual desde una imagen administrada](../virtual-machines/windows/create-vm-generalized-managed.md)
- [Crear una máquina virtual desde una imagen no administrada](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-user-image-data-disks)

>[!NOTE]
>Si va a implementar una máquina virtual en la que Windows 7 es el sistema operativo del host, el proceso de creación e implementación será un poco diferente. Para más información, consulte [Implementación de una máquina virtual Windows 7 en Windows Virtual Desktop](./virtual-desktop-fall-2019/deploy-windows-7-virtual-machine.md).

Una vez que haya creado las máquinas virtuales de host de sesión, [aplique una licencia de Windows a una máquina virtual de host de sesión](./apply-windows-license.md#apply-a-windows-license-to-a-session-host-vm) para ejecutar las máquinas virtuales Windows o Windows Server sin pagar por otra licencia.

## <a name="prepare-the-virtual-machines-for-windows-virtual-desktop-agent-installations"></a>Preparación de las máquinas virtuales para las instalaciones de agentes de Windows Virtual Desktop

Deberá hacer lo siguiente para preparar las máquinas virtuales antes de poder instalar los agentes de Windows Virtual Desktop y registrar las máquinas virtuales en el grupo de hosts de Windows Virtual Desktop:

- Debe unir la máquina al dominio. Esto permite que los usuarios entrantes de Windows Virtual Desktop se asignen desde sus cuentas de Azure Active Directory a su cuenta de Active Directory, así como tener acceso correctamente a la máquina virtual.
- Debe instalar el rol de host de sesión de Escritorio remoto (RDSH) si la máquina virtual está ejecutando un sistema operativo de Windows Server. El rol de RDSH permite que los agentes de Windows Virtual Desktop se instalen correctamente.

Para realizar correctamente una unión a un dominio, realice los siguientes pasos en cada máquina virtual:

1. [Conéctese a la máquina virtual](../virtual-machines/windows/quick-create-portal.md#connect-to-virtual-machine) con las credenciales que proporcionó al crear la máquina virtual.
2. En la máquina virtual, inicie el **Panel de control** y seleccione **Sistema**.
3. Seleccione **Nombre del equipo**, seleccione **Cambiar configuración** y, luego, seleccione **Cambiar…** .
4. Seleccione **Dominio** y, luego, escriba el dominio de Active Directory en la red virtual.
5. Autentíquese con una cuenta de dominio que tenga privilegios en máquinas unidas a dominio.

    >[!NOTE]
    > Si va a unir sus máquinas virtuales a un entorno de Azure Active Directory Domain Services (Azure AD DS), asegúrese de que su usuario de unión a un dominio también es miembro del [grupo de administradores del controlador de dominio de AAD](../active-directory-domain-services/tutorial-create-instance-advanced.md#configure-an-administrative-group).

## <a name="register-the-virtual-machines-to-the-windows-virtual-desktop-host-pool"></a>Registro de las máquinas virtuales en el grupo de hosts de Windows Virtual Desktop

El registro de las máquinas virtuales en un grupo de hosts de Windows Virtual Desktop es tan sencillo como instalar los agentes de Windows Virtual Desktop.

Para registrar los agentes de Windows Virtual Desktop, realice los siguientes pasos en cada máquina virtual:

1. [Conéctese a la máquina virtual](../virtual-machines/windows/quick-create-portal.md#connect-to-virtual-machine) con las credenciales que proporcionó al crear la máquina virtual.
2. Descargue e instale el agente de Windows Virtual Desktop.
   - Descargue el [agente de Windows Virtual Desktop](https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrmXv).
   - Ejecute al programa de instalación. Cuando el instalador le solicite el token de registro, escriba el valor obtenido del cmdlet **Get-AzWvdRegistrationInfo**.
3. Descargue e instale el cargador de arranque del agente de Windows Virtual Desktop.
   - Descargue el [cargador de arranque del agente de Windows Virtual Desktop](https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrxrH).
   - Ejecute al programa de instalación.

>[!IMPORTANT]
>Para ayudar a proteger su entorno de Windows Virtual Desktop en Azure, se recomienda no abrir el puerto de entrada 3389 en las máquinas virtuales. Windows Virtual Desktop no requiere un puerto de entrada abierto 3389 para que los usuarios accedan a máquinas virtuales del grupo host. Si debe abrir el puerto 3389 para solucionar problemas, se recomienda usar [acceso de máquina virtual Just-in-Time](../security-center/security-center-just-in-time.md). También se recomienda no asignar las máquinas virtuales a una dirección IP pública.

## <a name="update-the-agent"></a>Actualización del agente

Es necesario actualizar el agente en cualquiera de estas situaciones:

- Quiere migrar una sesión registrada previamente a un nuevo grupo de hosts
- El host de sesión no aparece en el grupo de hosts tras una actualización

Para actualizar el agente:

1. Inicie sesión en la máquina virtual como administrador.
2. Vaya a **Servicios** y detenga los procesos **Rdagent** y **Remote Desktop Agent Loader**.
3. Luego busque los MSI del agente y el cargador de arranque. Se encuentran en la carpeta **C:\DeployAgent** o en aquella ubicación en la que los haya guardado al instalar.
4. Busque los siguientes archivos y desinstálelos:
     
     - Microsoft.RDInfra.RDAgent.Installer-x64-verx.x.x
     - Microsoft.RDInfra.RDAgentBootLoader.Installer-x64

   Para desinstalar estos archivos, haga clic con el botón derecho en cada nombre de archivo y seleccione **Desinstalar**.
5. También puede quitar los siguientes valores del registro:
     
     - Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\RDInfraAgent
     - Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\RDAgentBootLoader

6. Una vez desinstalados estos elementos, se deberían quitar todas las asociaciones con el grupo de hosts anterior. Si quiere volver a registrar este host en el servicio, siga las instrucciones de [Registro de las máquinas virtuales en el grupo de hosts de Windows Virtual Desktop](create-host-pools-powershell.md#register-the-virtual-machines-to-the-windows-virtual-desktop-host-pool).


## <a name="next-steps"></a>Pasos siguientes

Ahora que ha creado un grupo de hosts, puede rellenarlo con RemoteApp. Para obtener más información sobre cómo administrar las aplicaciones de Windows Virtual Desktop, consulte el tutorial Administración de grupos de aplicaciones.

> [!div class="nextstepaction"]
> [Tutorial: Administración de grupos de aplicaciones](./manage-app-groups.md)

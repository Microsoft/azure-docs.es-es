---
title: Crear un grupo de host de vista previa de Escritorio Virtual Windows con PowerShell, Azure
description: Cómo crear un grupo host en Windows Vista previa del escritorio Virtual con cmdlets de PowerShell.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: how-to
ms.date: 05/06/2019
ms.author: helohr
ms.openlocfilehash: 8e6783bb56f55949824e4ce94c85145f53dd9f4b
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65523956"
---
# <a name="create-a-host-pool-with-powershell"></a>Creación de un grupo host con PowerShell

Los grupos de hosts son una colección de una o más máquinas virtuales idénticas en entornos de inquilinos de Windows Virtual Desktop (versión preliminar). Cada grupo de hosts puede contener un grupo de aplicaciones con las que los usuarios pueden interactuar igual que harían en un equipo de escritorio físico.

## <a name="use-your-powershell-client-to-create-a-host-pool"></a>Use el cliente de PowerShell para crear un grupo host

En primer lugar y, si aún no lo ha hecho, [descargue e importe el módulo de PowerShell para Windows Virtual Desktop](https://docs.microsoft.com/powershell/windows-virtual-desktop/overview) que se usará en la sesión de PowerShell.

Ejecute el siguiente cmdlet para iniciar sesión en el entorno de Escritorio Virtual de Windows

```powershell
Add-RdsAccount -DeploymentUrl https://rdbroker.wvd.microsoft.com
```

A continuación, ejecute este cmdlet para crear un nuevo grupo host en el inquilino de Escritorio Virtual de Windows:

```powershell
New-RdsHostPool -TenantName <tenantname> -Name <hostpoolname>
```

Ejecute el siguiente cmdlet para crear un token de registro para autorizar a un host de sesión para unir el grupo host y guardarla en un archivo nuevo en el equipo local. Puede especificar cuánto tiempo es válido el token de registro mediante el parámetro - ExpirationHours.

```powershell
New-RdsRegistrationInfo -TenantName <tenantname> -HostPoolName <hostpoolname> -ExpirationHours <number of hours> | Select-Object -ExpandProperty Token > <PathToRegFile>
```

Después, ejecute este cmdlet para agregar usuarios de Azure Active Directory al grupo de aplicación de escritorio predeterminado para el grupo host.

```powershell
Add-RdsAppGroupUser -TenantName <tenantname> -HostPoolName <hostpoolname> -AppGroupName "Desktop Application Group" -UserPrincipalName <userupn>
```

El **agregar RdsAppGroupUser** cmdlet no admite la adición de grupos de seguridad y solo agrega un usuario a la vez para el grupo de aplicaciones. Si desea agregar varios usuarios al grupo de aplicación, vuelva a ejecutar el cmdlet con los nombres principales de usuario adecuado.

Ejecute el siguiente cmdlet para exportar el token de registro a una variable, que usará más adelante en [registrar las máquinas virtuales al grupo host de Escritorio Virtual de Windows](#register-the-virtual-machines-to-the-windows-virtual-desktop-preview-host-pool).

```powershell
$token = (Export-RdsRegistrationInfo -TenantName <tenantname> -HostPoolName <hostpoolname>).Token
```

## <a name="create-virtual-machines-for-the-host-pool"></a>Crear máquinas virtuales para el grupo host

Ahora puede crear una máquina virtual de Azure que puede unirse al grupo de host de Escritorio Virtual de Windows.

Puede crear una máquina virtual de varias maneras:

- [Crear una máquina virtual desde una imagen de galería de Azure](https://docs.microsoft.com/azure/virtual-machines/windows/quick-create-portal#create-virtual-machine)
- [Crear una máquina virtual desde una imagen administrada](https://docs.microsoft.com/azure/virtual-machines/windows/create-vm-generalized-managed)
- [Crear una máquina virtual desde una imagen no administrada](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-user-image)

## <a name="prepare-the-virtual-machines-for-windows-virtual-desktop-preview-agent-installations"></a>Preparar las máquinas virtuales para las instalaciones de agente de Windows Vista previa del escritorio Virtual

Deberá hacer lo siguiente para preparar las máquinas virtuales antes de poder instalar a los agentes de Escritorio Virtual de Windows y registrar las máquinas virtuales al grupo de host de Escritorio Virtual de Windows:

- Debe unión al dominio la máquina. Esto permite a los usuarios de Escritorio Virtual de Windows entrantes a asignarse desde su cuenta de Azure Active Directory a su cuenta de Active Directory y correctamente se les permitirá el acceso a la máquina virtual.
- Debe instalar el rol de Host de sesión de escritorio remoto (RDSH) si la máquina virtual se está ejecutando un sistema operativo Windows Server. El rol RDSH permite que los agentes de Escritorio Virtual de Windows instalar correctamente.

Para correctamente unión a dominio, realice lo siguiente en cada máquina virtual:

1. [Conectarse a la máquina virtual](https://docs.microsoft.com/azure/virtual-machines/windows/quick-create-portal#connect-to-virtual-machine) con las credenciales que proporcionó al crear la máquina virtual.
2. En la máquina virtual, inicie **Panel de Control** y seleccione **sistema**.
3. Seleccione **nombre_equipo**, seleccione **cambiar la configuración de**y, a continuación, seleccione **cambios...**
4. Seleccione **dominio** y, a continuación, escriba el dominio de Active Directory en la red virtual.
5. Autenticar con una cuenta de dominio que tenga privilegios para unir máquinas mediante dominio.

## <a name="register-the-virtual-machines-to-the-windows-virtual-desktop-preview-host-pool"></a>Registrar las máquinas virtuales al grupo host de vista previa de Escritorio Virtual de Windows

Registrar las máquinas virtuales a un grupo de host de Escritorio Virtual de Windows es tan sencillo como instalar los agentes de Escritorio Virtual de Windows.

Para registrar a los agentes de Escritorio Virtual de Windows, haga lo siguiente en cada máquina virtual:

1. [Conectarse a la máquina virtual](https://docs.microsoft.com/azure/virtual-machines/windows/quick-create-portal#connect-to-virtual-machine) con las credenciales que proporcionó al crear la máquina virtual.
2. Descargue e instale al agente de Escritorio Virtual de Windows.
   - Descargue el [agente de Escritorio Virtual de Windows](https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrmXv).
   - Haga clic en el instalador descargado, seleccione **propiedades**, seleccione **desbloquear**, a continuación, seleccione **Aceptar**. Esto permitirá que el sistema debe confiar en el programa de instalación.
   - Ejecute al programa de instalación. Cuando el programa de instalación le solicita el token de registro, escriba el valor obtenido en el **exportación RdsRegistrationInfo** cmdlet.
3. Descargue e instale el cargador de arranque de agente de Escritorio Virtual de Windows.
   - Descargue el [cargador de arranque de Windows del agente de Escritorio Virtual](https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrxrH).
   - Haga clic en el instalador descargado, seleccione **propiedades**, seleccione **desbloquear**, a continuación, seleccione **Aceptar**. Esto permitirá que el sistema debe confiar en el programa de instalación.
   - Ejecute al programa de instalación.

>[!IMPORTANT]
>Para ayudar a proteger su entorno de Windows Virtual Desktop en Azure, se recomienda no abrir el puerto de entrada 3389 en las máquinas virtuales. Windows Virtual Desktop no requiere un puerto de entrada abierto 3389 para que los usuarios accedan a máquinas virtuales del grupo host. Si debe abrir el puerto 3389 para solucionar problemas, se recomienda usar [acceso de máquina virtual Just-in-Time](https://docs.microsoft.com/azure/security-center/security-center-just-in-time).

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha realizado un grupo host, puede rellenarlo con RemoteApp. Para obtener más información sobre cómo administrar las aplicaciones de Windows Virtual Desktop, consulte el tutorial Administración de grupos de aplicaciones.

> [!div class="nextstepaction"]
> [Tutorial: Administración de grupos de aplicaciones](./manage-app-groups.md)

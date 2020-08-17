---
title: 'Tipo de asignación de escritorio personal de Windows Virtual Desktop: Azure'
description: Procedimiento para configurar la asignación automática o directa para un grupo de host de escritorio personal de Windows Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 07/09/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 446dae3c064f5f23d35cb12b2b24bdfea9e27012
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88007818"
---
# <a name="configure-the-personal-desktop-host-pool-assignment-type"></a>Configuración del tipo de asignación de grupo de host de escritorio personal

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop con objetos de Windows Virtual Desktop para Azure Resource Manager. Si usa Windows Virtual Desktop (clásico) sin objetos para Azure Resource Manager, consulte [este artículo](./virtual-desktop-fall-2019/configure-host-pool-personal-desktop-assignment-type-2019.md).

Puede configurar el tipo de asignación del grupo de host de escritorio personal para ajustar el entorno de Windows Virtual Desktop para que se adapte mejor a sus necesidades. En este tema, se mostrará cómo configurar la asignación automática o directa para los usuarios.

>[!NOTE]
> Las instrucciones de este artículo solo se aplican a los grupos de host de escritorio personal, no a los grupos de host agrupados, ya que los usuarios de grupos de host agrupados no se asignan a hosts de sesión específicos.

## <a name="prerequisites"></a>Requisitos previos

En este artículo se da por supuesto que ya ha descargado e instalado el módulo de PowerShell para Windows Virtual Desktop. Si no lo ha hecho, siga las instrucciones que se indican en [Configuración del módulo de PowerShell](powershell-module.md).

## <a name="configure-automatic-assignment"></a>Configuración de la asignación automática

La asignación automática es el tipo de asignación predeterminado para los nuevos grupos de host de escritorio personal creados en el entorno de Windows Virtual Desktop. La asignación automática de usuarios no requiere un host de sesión específico.

Para asignar usuarios de forma automática, asígnelos primero al grupo de host de escritorio personal para que puedan ver el escritorio en su fuente. Cuando un usuario asignado inicia el escritorio en su fuente, notificará un host de sesión disponible si todavía no se ha conectado al grupo de host, que completa el proceso de asignación.

Para configurar un grupo de host a fin de asignar automáticamente usuarios a las máquinas virtuales, ejecute el siguiente cmdlet de PowerShell:

```powershell
Update-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -PersonalDesktopAssignmentType Automatic
```

Para asignar un usuario al grupo de host de escritorio personal, ejecute el siguiente cmdlet de PowerShell:

```powershell
New-AzRoleAssignment -SignInName <userupn> -RoleDefinitionName "Desktop Virtualization User" -ResourceName <appgroupname> -ResourceGroupName <resourcegroupname> -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
```

## <a name="configure-direct-assignment"></a>Configuración de la asignación directa

A diferencia de la asignación automática, cuando se usa la asignación directa, debe asignar el usuario al grupo de host de escritorio personal y a un host de sesión específico antes de que se pueda conectar a su escritorio personal. Si el usuario solo se asigna a un grupo de host sin una asignación de host de sesión, no podrá acceder a los recursos.

Para configurar un grupo de host para que requiera la asignación directa de usuarios a los hosts de sesión, ejecute el siguiente cmdlet de PowerShell:

```powershell
Update-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -PersonalDesktopAssignmentType Direct
```

Para asignar un usuario al grupo de host de escritorio personal, ejecute el siguiente cmdlet de PowerShell:

```powershell
New-AzRoleAssignment -SignInName <userupn> -RoleDefinitionName "Desktop Virtualization User" -ResourceName <appgroupname> -ResourceGroupName <resourcegroupname> -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
```

Para asignar un usuario a un host de sesión específico, ejecute el siguiente cmdlet de PowerShell:

```powershell
Update-AzWvdSessionHost -HostPoolName <hostpoolname> -Name <sessionhostname> -ResourceGroupName <resourcegroupname> -AssignedUser <userupn>
```

Para asignar directamente un usuario a un host de sesión en Azure Portal:

1. Inicie sesión en Azure Portal en <https://portal.azure.com>.
2. Escriba **Windows Virtual Desktop** en la barra de búsqueda.
3. En **Servicios**, seleccione **Windows Virtual Desktop**.
4. En la página Windows Virtual Desktop, vaya al menú a la izquierda de la ventana y seleccione **Grupos de hosts**.
5. Seleccione el nombre del grupo de hosts que quiera actualizar.
6. Luego, vaya al menú del lado izquierdo de la ventana y seleccione **Grupos de aplicaciones**.
7. Seleccione el nombre del grupo de aplicaciones de escritorio que desee editar y, a continuación, seleccione **Asignaciones** en el menú del lado izquierdo de la ventana.
8. Seleccione **+ Agregar** y, a continuación, seleccione los usuarios o grupos de usuarios para los que desea publicar este grupo de aplicaciones de escritorio.
9. Seleccione **Assign VM** (Asignar VM) en la barra de información para asignar un host de sesión a un usuario.
10. Seleccione el host de sesión que desea asignar al usuario y, a continuación, seleccione **Asignar**.
11. Seleccione el usuario al que desea asignar el host de sesión de la lista de usuarios disponibles.
12. Cuando finalice, haga clic en **Seleccionar**.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha configurado el tipo de asignación de escritorio personal, puede iniciar sesión en un cliente de Windows Virtual Desktop para probarlo como parte de una sesión de usuario. En las dos guías paso a paso siguientes se indica cómo conectarse a una sesión mediante el cliente que elija:

- [Conexión con el cliente de Escritorio de Windows](connect-windows-7-10.md)
- [Conexión con el cliente web](connect-web.md)
- [Conexión con el cliente de Android](connect-android.md)
- [Conexión con el cliente iOS](connect-ios.md)
- [Conexión con el cliente macOS](connect-macos.md)
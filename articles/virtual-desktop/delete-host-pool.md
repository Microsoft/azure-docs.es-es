---
title: 'Eliminación de grupos de hosts de Windows Virtual Desktop: Azure'
description: Eliminación de un grupo de hosts en Windows Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 07/11/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: dfc9858bea468389d8ce90677f048e5d1fd3bb82
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88007597"
---
# <a name="delete-a-host-pool"></a>Eliminación de un grupo de hosts

Todos los grupos de hosts creados en Windows Virtual Desktop se adjuntan a los hosts de sesión y a los grupos de aplicaciones. Para eliminar un grupo de hosts, debe eliminar los grupos de aplicaciones y los hosts de sesión asociados. Eliminar un grupo de aplicaciones es bastante sencillo, pero eliminar un host de sesión es más complicado. Cuando se elimina un host de sesión, es necesario asegurarse de que no tiene ninguna sesión de usuario activa. Se deben cerrar todas las sesiones de usuario en el host de sesión para evitar que los usuarios pierdan datos.

## <a name="delete-a-host-pool-with-powershell"></a>Eliminación de un grupo de hosts con PowerShell

Para eliminar un grupo de hosts con PowerShell, primero debe eliminar todos los grupos de aplicaciones del grupo de hosts. Para eliminar todos los grupos de aplicaciones, ejecute el cmdlet de PowerShell siguiente:

```powershell
Remove-AzWvdApplicationGroup -Name <appgroupname> -ResourceGroupName <resourcegroupname>
```

A continuación, ejecute este cmdlet para eliminar el grupo de hosts:

```powershell
Remove-AzWvdHostPool -Name <hostpoolname> -ResourceGroupName <resourcegroupname> -Force:$true
```

Este cmdlet quita todas las sesiones de usuario existentes en el host de sesión del grupo de hosts. También anula el registro del host de sesión del grupo de hosts. Las máquinas virtuales (VM) relacionadas seguirán existiendo dentro de su suscripción.

## <a name="delete-a-host-pool-with-the-azure-portal"></a>Eliminación de un grupo de hosts con Azure Portal

Para eliminar un grupo de hosts en Azure Portal:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

2. Busque y seleccione **Windows Virtual Desktop**.

3. Seleccione **Grupos de hosts** en el menú de la izquierda de la página y seleccione el nombre del grupo de hosts que quiere eliminar.

4. En el menú de la izquierda de la página, seleccione **Grupos de aplicaciones**.

5. Seleccione todos los grupos de aplicaciones del grupo de hosts que va a eliminar y seleccione **Quitar**.

6. Una vez que quite los grupos de aplicaciones, vaya al menú del lado izquierdo de la página y seleccione **Información general**.

7. Seleccione **Quitar**.

8. Si hay hosts de sesión en el grupo de hosts que está eliminando, verá un mensaje que le pedirá permiso para continuar. Seleccione **Sí**.

9. En Azure Portal se quitarán todos los hosts de sesión y se eliminará el grupo de hosts. Las máquinas virtuales relacionadas con el host de sesión no se eliminarán y permanecerán en la suscripción.

## <a name="next-steps"></a>Pasos siguientes

Para información sobre cómo crear un grupo de hosts, consulte estos artículos:

- [Creación de un grupo de hosts con Azure Portal](create-host-pools-azure-marketplace.md)
- [Creación de un grupo host con PowerShell](create-host-pools-powershell.md)

Para información sobre cómo configurar grupos de hosts, consulte estos artículos:

- [Personalización de las propiedades de Protocolo de escritorio remoto para un grupo de hosts](customize-rdp-properties.md)
- [Configuración del método de equilibrio de carga de Windows Virtual Desktop](configure-host-pool-load-balancing.md)
- [Configuración del tipo de asignación de grupo de host de escritorio personal](configure-host-pool-personal-desktop-assignment-type.md)
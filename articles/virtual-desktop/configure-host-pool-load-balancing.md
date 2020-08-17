---
title: 'Configuración del equilibrio de carga de Windows Virtual Desktop: Azure'
description: Cómo configurar el método de equilibrio de carga de un entorno de Windows Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 08/29/2019
ms.author: helohr
manager: lizross
ms.openlocfilehash: 07eae73a36bf4051925547fa375f46963a162881
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88010113"
---
# <a name="configure-the-windows-virtual-desktop-load-balancing-method"></a>Configuración del método de equilibrio de carga de Windows Virtual Desktop

La configuración del método de equilibrio de carga de un grupo de hosts le permite ajustar el entorno de Windows Virtual Desktop para satisfacer mejor sus necesidades.

>[!NOTE]
> Esto no se aplica a un grupo de hosts de escritorio persistente, ya que los usuarios siempre tienen una asignación de 1:1 a un host de sesión dentro del grupo de hosts.

## <a name="prerequisites"></a>Prerrequisitos

En este artículo se supone que ha seguido las instrucciones que se indican en [Configuración del módulo de PowerShell para Windows Virtual Desktop](powershell-module.md) para descargar e instalar el módulo de PowerShell e iniciar sesión en su cuenta de Azure.

## <a name="configure-breadth-first-load-balancing"></a>Configuración del equilibrio de carga en amplitud

El equilibrio de carga en amplitud es la configuración predeterminada de los nuevos grupos de hosts no persistentes. El equilibrio de carga en amplitud distribuye nuevas sesiones de usuario entre todos los hosts de sesión disponibles del grupo de hosts. Al configurar el equilibrio de carga en amplitud, puede establecer un límite de sesiones máximo por host de sesión en el grupo de hosts.

Para configurar un grupo de hosts para realizar el equilibrio de carga en amplitud sin ajustar el límite de sesiones máximo, ejecute el siguiente cmdlet de PowerShell:

```powershell
Update-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -LoadBalancerType 'BreadthFirst'
```

Después, para asegurarse de que ha establecido el método de equilibrio de carga en amplitud, ejecute el siguiente cmdlet:

```powershell
Get-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> | format-list Name, LoadBalancerType

Name             : hostpoolname
LoadBalancerType : BreadthFirst
```

Para configurar un grupo de hosts para realizar el equilibrio de carga en amplitud y usar un nuevo límite de sesiones máximo, ejecute el siguiente cmdlet de PowerShell:

```powershell
Update-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -LoadBalancerType 'BreadthFirst' -MaxSessionLimit ###
```

## <a name="configure-depth-first-load-balancing"></a>Configuración del equilibrio de carga en profundidad

El equilibrio de carga en profundidad distribuye nuevas sesiones de usuario a un host de sesión disponible con el máximo número de conexiones, pero que no ha alcanzado su umbral del límite de sesiones máximo. Al configurar el equilibrio de carga en profundidad, debe establecer un límite de sesiones máximo por host de sesión en el grupo de hosts.

Para configurar un grupo de hosts para realizar el equilibrio de carga en profundidad, ejecute el siguiente cmdlet de PowerShell:

```powershell
Update-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -LoadBalancerType 'DepthFirst' -MaxSessionLimit ###
```

Para asegurarse de que la configuración se ha actualizado, ejecute este cmdlet:

```powershell
Get-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> | format-list Name, LoadBalancerType, MaxSessionLimit

Name             : hostpoolname
LoadBalancerType : DepthFirst
MaxSessionLimit  : 6
```

## <a name="configure-load-balancing-with-the-azure-portal"></a>Configuración del equilibrio de carga con Azure Portal

También puede configurar el equilibrio de carga con Azure Portal.

Para configurar el equilibrio de carga:

1. Inicie sesión en Azure Portal en https://portal.azure.com.
2. Busque y seleccione **Windows Virtual Desktop** en Servicios.
3. En la página de Windows Virtual Desktop, seleccione **Grupos de hosts**.
4. Seleccione el nombre del grupo de hosts que quiera editar.
5. Seleccione **Propiedades**.
6. En el menú desplegable, escriba el **límite máximo de sesiones** en el campo y seleccione el **algoritmo de equilibrio de carga** que desee para este grupo de hosts.
7. Seleccione **Guardar**. Esto aplicará la nueva configuración de equilibrio de carga.
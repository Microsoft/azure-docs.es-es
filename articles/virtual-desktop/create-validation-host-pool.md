---
title: 'Actualizaciones de servicio para los grupos de host de Windows Virtual Desktop: Azure'
description: Cómo crear un grupo host de validación para supervisar actualizaciones de servicio antes de implementar las actualizaciones en producción.
author: Heidilohr
ms.topic: tutorial
ms.date: 03/13/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 8eac40ad958a10b8c853304ee2be8b2dc27af1a2
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88008719"
---
# <a name="tutorial-create-a-host-pool-to-validate-service-updates"></a>Tutorial: Creación de un grupo host para validar las actualizaciones del servicio

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop con objetos de Windows Virtual Desktop de Azure Resource Manager. Si usa Windows Virtual Desktop (clásico) sin objetos de Azure Resource Manager, consulte [este artículo](./virtual-desktop-fall-2019/create-validation-host-pool-2019.md).

Los grupos hosts son una colección de una o más máquinas virtuales idénticas en entornos de inquilino de Windows Virtual Desktop. Antes de implementar grupos host en el entorno de producción, se recomienda encarecidamente crear un grupo host de validación. Primero, las actualizaciones se aplican a los grupos host de validación, lo que permite supervisar las actualizaciones de servicio antes de implementarlas en el entorno de producción. Sin un grupo host de validación, es posible que no se detecten cambios que introducen errores, lo que podría dar lugar a tiempos de inactividad para los usuarios en el entorno de producción.

Para garantizar que las aplicaciones funcionan con las actualizaciones más recientes, el grupo host de validación debe ser lo más parecido posible a los grupos host del entorno de producción. Los usuarios deben conectarse al grupo host de validación con la misma frecuencia que lo hacen al grupo host de producción. Si ha automatizado las pruebas en su grupo host, debe incluir las pruebas automatizadas en el grupo host de validación.

Puede depurar incidencias en el grupo host de validación con [la característica de diagnóstico](diagnostics-role-service.md) o con [artículos de solución de problemas de Windows Virtual Desktop](troubleshoot-set-up-overview.md).

>[!NOTE]
> Se recomienda que mantenga el grupo host de validación para probar todas las actualizaciones futuras.

>[!IMPORTANT]
>Windows Virtual Desktop con la integración de Administración de recursos de Azure actualmente tiene problemas para habilitar y deshabilitar los entornos de validación. Este artículo se actualizará cuando se haya resuelto el problema.

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar, siga las instrucciones que se indican en [Configuración del módulo de PowerShell para Windows Virtual Desktop](powershell-module.md) para configurar el módulo de PowerShell e iniciar sesión en Azure.

## <a name="create-your-host-pool"></a>Creación del grupo host

Puede crear un grupo host siguiendo las instrucciones de cualquiera de estos artículos:
- [Tutorial: Creación de un grupo host con Azure Marketplace](create-host-pools-azure-marketplace.md)
- [Creación de un grupo host con PowerShell](create-host-pools-powershell.md)

## <a name="define-your-host-pool-as-a-validation-host-pool"></a>Definición del grupo host como grupo host de validación

Ejecute los siguientes cmdlets de PowerShell para definir el nuevo grupo host como grupo host de validación. Reemplace los valores entre corchetes por los valores pertinentes para su sesión:

```powershell
Update-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -ValidationEnvironment:$true
```

Ejecute el siguiente cmdlet de PowerShell para confirmar que la propiedad de validación se ha definido. Reemplace los valores entre corchetes por los valores pertinentes para su sesión.

```powershell
Get-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> | Format-List
```

Los resultados del cmdlet deben ser similares a estos:

```powershell
    HostPoolName        : hostpoolname
    FriendlyName        :
    Description         :
    Persistent          : False
    CustomRdpProperty   : use multimon:i:0;
    MaxSessionLimit     : 10
    LoadBalancerType    : BreadthFirst
    ValidationEnvironment : True
```

## <a name="update-schedule"></a>Programación de actualizaciones

Las actualizaciones del servicio se realizan mensualmente. Si hay incidencias importantes, se proporcionarán actualizaciones críticas con mayor frecuencia.

Si hay alguna actualización del servicio, asegúrese de que tenga al menos un pequeño grupo de usuarios que inician sesión cada día para validar el entorno. Le recomendamos que visite periódicamente nuestro [sitio de TechCommunity](https://techcommunity.microsoft.com/t5/forums/searchpage/tab/message?filter=location&q=wvdupdate&location=forum-board:WindowsVirtualDesktop&sort_by=-topicPostDate&collapse_discussion=true) y siga cualquier publicación con WVDUPdate para mantenerse informado sobre las actualizaciones del servicio.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha creado un grupo de hosts de validación, puede obtener información sobre cómo usar Azure Service Health para supervisar la implementación de Windows Virtual Desktop.

> [!div class="nextstepaction"]
> [Configuración de alertas de servicio](./set-up-service-alerts.md)

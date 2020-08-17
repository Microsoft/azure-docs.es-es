---
title: 'Configuración de las alertas de servicio para Windows Virtual Desktop (clásico): Azure'
description: Configuración de Azure Service Health para recibir notificaciones del servicio de Windows Virtual Desktop (clásico).
author: Heidilohr
ms.topic: tutorial
ms.date: 05/27/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 3f2171d3271a4ffc4770dd0c9faea16c23e44d02
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88005509"
---
# <a name="tutorial-set-up-service-alerts-for-windows-virtual-desktop-classic"></a>Tutorial: Configuración de las alertas de servicio para Windows Virtual Desktop (clásico)

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop (clásico), que no admite objetos de Windows Virtual Desktop para Azure Resource Manager. Si está tratando de administrar objetos de Windows Virtual Desktop para Azure Resource Manager, consulte [este artículo](../set-up-service-alerts.md).

Puede usar Azure Service Health para supervisar los problemas del servicio y los avisos de estado de Windows Virtual Desktop. Azure Service Health puede notificarlo con distintos tipos de alertas (por ejemplo, por correo electrónico o SMS), ayudarlo a entender el impacto de un problema y mantenerlo actualizado con respecto a la resolución del problema. Azure Service Health también puede ayudarlo a mitigar el tiempo de inactividad y preparar el mantenimiento planeado y los cambios que podrían afectar la disponibilidad de los recursos.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear y configurar las alertas de servicio.

Para más información sobre Azure Service Health, consulte la [documentación sobre el estado de Azure](https://docs.microsoft.com/azure/service-health/).

## <a name="prerequisites"></a>Prerrequisitos

- [Tutorial: Creación de un inquilino en Windows Virtual Desktop](tenant-setup-azure-active-directory.md)
- [Tutorial: Creación de entidades de servicio y asignaciones de roles con PowerShell](create-service-principal-role-powershell.md)
- [Tutorial: Creación de un grupo host con Azure Marketplace](create-host-pools-azure-marketplace-2019.md)

## <a name="create-service-alerts"></a>Creación de alertas de servicio

En esta sección se muestra cómo configurar Azure Service Health y cómo establecer las notificaciones a las que puede acceder en Azure Portal. Puede configurar distintos tipos de alertas y programarlas para recibir notificaciones de manera oportuna.

### <a name="recommended-service-alerts"></a>Alertas de servicio recomendadas

Se recomienda crear alertas de servicio para los tipos de eventos de estado siguientes:

- **Problema de servicio:** reciba notificaciones sobre los problemas importantes que afectan la conectividad de los usuarios con el servicio o la capacidad de administrar el inquilino de Windows Virtual Desktop.
- **Aviso de estado:** reciba las notificaciones que requiere su atención. Estos son algunos ejemplos de este tipo de notificación:
    - Máquinas virtuales (VM) no configuradas de manera segura con el puerto 3389 abierto.
    - Funcionalidad en desuso.

### <a name="configure-service-alerts"></a>Configuración de las alertas de servicio

Para configurar las alertas de servicio:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Seleccione **Service Health**.
3. Use las instrucciones que aparecen en [Creación de alertas del registro de actividad en notificaciones del servicio](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log-service-notifications?toc=%2Fazure%2Fservice-health%2Ftoc.json#alert-and-new-action-group-using-azure-portal) para configurar las alertas y las notificaciones.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió a configurar y usar Azure Service Health para supervisar los problemas del servicio y los avisos de estado de Windows Virtual Desktop. Para información sobre el inicio de sesión en Windows Virtual Desktop, continúe con los procedimientos de conexión a Windows Virtual Desktop.

> [!div class="nextstepaction"]
> [Conexión al cliente de Escritorio remoto en Windows 7 y Windows 10](connect-windows-7-10-2019.md)

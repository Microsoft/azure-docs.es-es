---
title: Administrar perfiles del Azure Traffic Manager | Microsoft Docs
description: La información contenida en este artículo le ayuda a crear, deshabilitar, habilitar y eliminar un perfil de Azure Traffic Manager.
services: traffic-manager
documentationcenter: ''
author: duongau
ms.service: traffic-manager
manager: twooley
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/10/2017
ms.author: duau
ms.openlocfilehash: 7bb5559f21289a40b6c75a3481680ab5b2c1c269
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89392891"
---
# <a name="manage-an-azure-traffic-manager-profile"></a>Administrar un perfil de Azure Traffic Manager

Los perfiles de Traffic Manager usan métodos de enrutamiento de tráfico para controlar la distribución del tráfico a los puntos de conexión de servicios en la nube o de sitios web. En este artículo se explica cómo crear y administrar estos perfiles.

## <a name="create-a-traffic-manager-profile"></a>Crear un perfil de Traffic Manager

Puede crear un perfil de Traffic Manager mediante el portal de Azure. Después de crear el perfil, puede configurar puntos de conexión, supervisión y otras opciones en el portal de Azure. El Administrador de tráfico admite un máximo de 200 extremos por perfil. Sin embargo, la mayoría de los escenarios de uso tan solo requieren unos pocos puntos de conexión.

### <a name="to-create-a-traffic-manager-profile"></a>Para crear un perfil de Traffic Manager

1. En un explorador, inicie sesión en [Azure Portal](https://portal.azure.com). Si aún no tiene cuenta, puede registrarse para obtener [una evaluación gratuita durante un mes](https://azure.microsoft.com/free/). 
2. Haga clic en **Crear un recurso** > **Redes** > **Perfil del Traffic Manager** > **Crear**.
4. En **Crear perfil de Traffic Manager**, complete los campos de la manera siguiente:
    1. En **Nombre**, proporcione un nombre para el perfil. Este nombre debe ser único en la zona trafficmanager.net y generará el nombre DNS `<name>`,trafficmanager.net que se usa para acceder al perfil de Traffic Manager.
    2. En **Método de enrutamiento**, seleccione el método de enrutamiento por **prioridad**.
    3. En **Suscripción**, seleccione la suscripción en la que desea crear este perfil.
    4. En **Grupo de recursos**, cree un grupo de recursos nuevo en el cual colocar este perfil.
    5. En **Ubicación del grupo de recursos**, seleccione la ubicación del grupo de recursos. Esta configuración se refiere a la ubicación del grupo de recursos y no tiene efecto alguno sobre el perfil de Traffic Manager que se implementará globalmente.
    6. Haga clic en **Crear**.
    7. Una vez que se complete la implementación global del perfil de Traffic Manager, aparece en el grupo de recursos respectivo como uno de los recursos.

## <a name="disable-enable-or-delete-a-profile"></a>Deshabilitar, habilitar o eliminar un perfil

Puede deshabilitar un perfil existente para que Traffic Manager no refiera solicitudes de usuario a los puntos de conexión configurados. Cuando deshabilite un perfil de Traffic Manager, dicho perfil y la información que contiene se mantienen intactos y se pueden editar en la interfaz de Traffic Manager.  Las referencias se reanudarán cuando vuelva a habilitar el perfil. Cuando se crea un perfil de Traffic Manager en Azure Portal, este se habilita automáticamente. Si decide que un perfil ha dejado de ser necesario, puede eliminarlo.

### <a name="to-disable-a-profile"></a>Para deshabilitar un perfil

1. Si usa un nombre de dominio personalizado, cambie el registro CNAME en el servidor DNS de Internet para que deje de apuntar a su perfil de Traffic Manager.
2. El tráfico dejará de dirigirse a los puntos de conexión por medio de la configuración del perfil de Traffic Manager.
3. En un explorador, inicie sesión en [Azure Portal](https://portal.azure.com).
2. En la barra de búsqueda, busque el nombre del **perfil de Traffic Manager** que desea modificar y haga clic en él en los resultados que se muestran.
3. Haga clic en **Información general** > **Deshabilitar**.
4. Confirme para deshabilitar el perfil de Traffic Manager.

### <a name="to-enable-a-profile"></a>Para habilitar un perfil

1. En un explorador, inicie sesión en [Azure Portal](https://portal.azure.com).
2. En la barra de búsqueda, busque el nombre del **perfil de Traffic Manager** que desea modificar y haga clic en él en los resultados que se muestran.
3. Haga clic en **Información general** > **Habilitar**.
1. Si usa un nombre de dominio personalizado, cree un registro de recursos CNAME en el servidor DNS de Internet para que apunte al nombre de dominio de su perfil de Traffic Manager.
2. El tráfico se dirige a los puntos de conexión de nuevo.

### <a name="to-delete-a-profile"></a>Para eliminar un perfil

1. Asegúrese de que el registro de recursos DNS del servidor DNS de Internet ya no usa un registro de recursos CNAME que señale al nombre de dominio del perfil del Administrador de tráfico.
2. En la barra de búsqueda, busque el nombre del **perfil de Traffic Manager** que desea modificar y haga clic en él en los resultados que se muestran.
3. Haga clic en **Información general** > **Eliminar**.
4. Confirme para eliminar el perfil de Traffic Manager.

## <a name="next-steps"></a>Pasos siguientes

* [Agregación de un extremo](traffic-manager-endpoints.md)
* [Configuración del método de enrutamiento por prioridad](traffic-manager-configure-priority-routing-method.md)
* [Configuración del método de enrutamiento geográfico](traffic-manager-configure-geographic-routing-method.md) 
* [Configuración del método de enrutamiento ponderado](traffic-manager-configure-weighted-routing-method.md)
* [Configuración del método de enrutamiento de rendimiento](traffic-manager-configure-performance-routing-method.md)

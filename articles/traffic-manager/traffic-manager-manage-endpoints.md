---
title: Administración de extremos en el Azure Traffic Manager | Microsoft Docs
description: Este artículo le ayudará a agregar, quitar, habilitar y deshabilitar extremos del Administrador de tráfico de Azure.
services: traffic-manager
documentationcenter: ''
author: duongau
ms.service: traffic-manager
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/08/2017
ms.author: duau
ms.openlocfilehash: fc6e8bcda17f9d788a430ed763bd9f8702eaf250
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89392925"
---
# <a name="add-disable-enable-or-delete-endpoints"></a>Incorporación, deshabilitación, habilitación o eliminación de puntos de conexión

La función Web Apps de Azure App Service ya proporciona la funcionalidad de enrutamiento del tráfico de conmutación por error y round robin para sitios web en un centro de datos, independientemente del modo del sitio web. El Administrador de tráfico de Azure permite especificar el enrutamiento del tráfico de conmutación por error y round robin para sitios web y servicios en la nube en distintos centros de datos. El primer paso necesario para proporcionar esa funcionalidad es agregar el extremo del sitio web o del servicio en la nube al Administrador de tráfico.

También puede deshabilitar los extremos individuales que forman parte de un perfil del Administrador de tráfico. Al deshabilitar un extremo, se deja como parte del perfil, pero el perfil actúa como si el extremo no estuviera incluido en él. Esta acción es útil para quitar temporalmente un punto de conexión que se encuentre en modo de mantenimiento o que se vaya a implementar. Cuando el extremo vuelva a estar en funcionamiento, se puede habilitar

> [!NOTE]
> Deshabilitar un extremo no tiene nada que ver con su estado de implementación en Azure. Sigue activo un punto de conexión en buen estado que puede recibir tráfico incluso cuando se deshabilite en Traffic Manager. Además, al deshabilitar un extremo de un perfil, no se afecta al estado en otro perfil.

## <a name="to-add-a-cloud-service-or-an-app-service-endpoint-to-a-traffic-manager-profile"></a>Para agregar un servicio en la nube o un punto de conexión de App Service a un perfil de Traffic Manager

1. En un explorador, inicie sesión en [Azure Portal](https://portal.azure.com).
2. En la barra de búsqueda, busque el nombre del **perfil de Traffic Manager** que desea modificar y haga clic en él en los resultados que se muestran.
3. En la hoja **Perfil de Traffic Manager**, en la sección **Configuración**, haga clic en **Puntos de conexión**.
4. En la hoja **Puntos de conexión** que aparece, haga clic en **Agregar**.
5. En la hoja **Agregar punto de conexión**, complete los campos de la siguiente manera:
    1. En **Tipo**, haga clic en **Punto de conexión de Azure**.
    2. Proporcione un **nombre** con el que desea reconocer este punto de conexión.
    3. En **Tipo de recurso de destino**, en la lista desplegable, elija el tipo de recurso apropiado.
    4. Para el **Recurso de destino**, haga clic en el selector **Elegir...**  para enumerar los recursos de la lista bajo la misma suscripción en la **hoja de recursos**. En la hoja **Recursos** que aparece, elija el servicio de aplicaciones que desea agregar como primer punto de conexión.
    5. En **Prioridad**, seleccione **1**. Esto hace que todo el tráfico vaya a este punto de conexión si funciona correctamente.
    6. No active la opción **Agregar como deshabilitado**.
    7. Haga clic en **Aceptar**
6.  Repita los pasos 4 y 5 para agregar el siguiente punto de conexión de Azure. Asegúrese de agregarlo con el valor de **prioridad** establecido en **2**.
7.  Cuando termine de agregar ambos puntos de conexión, aparecerán en la hoja **Perfil de Traffic Manager** junto con el estado de supervisión como **En línea**.

> [!NOTE]
> Después de agregar o quitar un punto de conexión de un perfil mediante el método de enrutamiento de tráfico *Conmutación por error*, no podrá ordenar como desee la lista de prioridad de conmutación por error. Puede ajustar el orden de la lista de prioridad de conmutación por error en la página Configuración. Para obtener más información, consulte [Método de enrutamiento del tráfico de conmutación por error](traffic-manager-configure-failover-routing-method.md).

## <a name="to-disable-an-endpoint"></a>Para deshabilitar un extremo

1. En un explorador, inicie sesión en [Azure Portal](https://portal.azure.com).
2. En la barra de búsqueda, busque el nombre del **perfil de Traffic Manager** que desea modificar y haga clic en él en los resultados que se muestran.
3. En la hoja **Perfil de Traffic Manager**, en la sección **Configuración**, haga clic en **Puntos de conexión**. 
4. Haga clic en el punto de conexión que desea deshabilitar.
5. En la hoja **Punto de conexión**, cambie el estado del punto de conexión a **Deshabilitado**y haga clic en **Guardar**.
6. Los clientes siguen enviando tráfico al punto de conexión durante el período de vida (TTL). Puede cambiar el valor de TTL en la página Configuración del perfil de Traffic Manager.

## <a name="to-enable-an-endpoint"></a>Para habilitar un extremo

1. En un explorador, inicie sesión en [Azure Portal](https://portal.azure.com).
2. En la barra de búsqueda, busque el nombre del **perfil de Traffic Manager** que desea modificar y haga clic en él en los resultados que se muestran.
3. En la hoja **Perfil de Traffic Manager**, en la sección **Configuración**, haga clic en **Puntos de conexión**. 
4. Haga clic en el punto de conexión que desea habilitar.
5. En la hoja **Punto de conexión**, cambie el estado del punto de conexión a **Habilitado**y haga clic en **Guardar**.
6. Los clientes siguen enviando tráfico al punto de conexión durante el período de vida (TTL). Puede cambiar el valor de TTL en la página Configuración del perfil de Traffic Manager.

## <a name="to-delete-an-endpoint"></a>Para eliminar un punto de conexión

1. En un explorador, inicie sesión en [Azure Portal](https://portal.azure.com).
2. En la barra de búsqueda, busque el nombre del **perfil de Traffic Manager** que desea modificar y haga clic en él en los resultados que se muestran.
3. En la hoja **Perfil de Traffic Manager**, en la sección **Configuración**, haga clic en **Puntos de conexión**. 
4. Haga clic en el punto de conexión que desea eliminar.
5. En la hoja **Punto de conexión**, haga clic en **Eliminar**.


## <a name="next-steps"></a>Pasos siguientes

* [Administración de perfiles de Traffic Manager](traffic-manager-manage-profiles.md)
* [Configurar métodos de enrutamiento](traffic-manager-configure-routing-method.md)
* [Solución de problemas de estado degradado del Administrador de tráfico](traffic-manager-troubleshooting-degraded.md)
* [Consideraciones de rendimiento sobre el Administrador de tráfico](traffic-manager-performance-considerations.md)
* [Operaciones del Administrador de tráfico (referencia de la API de REST)](https://go.microsoft.com/fwlink/p/?LinkID=313584)


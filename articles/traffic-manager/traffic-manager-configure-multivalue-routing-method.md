---
title: 'Configuración del enrutamiento de tráfico de varios valores: Azure Traffic Manager'
description: En este artículo se explica cómo configurar Traffic Manager para enrutar el tráfico a puntos de conexión A/AAAA.
services: traffic-manager
documentationcenter: ''
author: duongau
ms.service: traffic-manager
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/10/2018
ms.author: duau
ms.openlocfilehash: 85e088dda767a6f6c80ac0a9f6eed84e8802e5ee
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89401051"
---
# <a name="configure-multivalue-routing-method-in-traffic-manager"></a>Configuración del método de enrutamiento de varios valores en Traffic Manager

En este artículo se describe cómo configurar el método de enrutamiento de tráfico de varios valores. El método de enrutamiento de tráfico **de varios valores** permite devolver varios puntos de conexión en buen estado y ayuda a aumentar la fiabilidad de la aplicación, puesto que los clientes tienen más opciones de reintento sin tener que realizar otra búsqueda de DNS. El enrutamiento de varios valores está habilitado solo para perfiles que tienen todos sus puntos de conexión especificados mediante direcciones IPv4 o IPv6. Al recibirse una consulta relacionada con este perfil, se devuelven todos los puntos de conexión correctos en función del número máximo de devoluciones configurado. 

>[!NOTE]
> En este momento, agregar puntos de conexión mediante direcciones IPv4 o IPv6 solo se admite para puntos de conexión de tipo **externo** y, por tanto, el enrutamiento de varios valores también se admite en esos puntos de conexión.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure 

Inicie sesión en Azure Portal en https://portal.azure.com.
## <a name="create-a-resource-group"></a>Crear un grupo de recursos
Cree un grupo de recursos para el perfil de Traffic Manager.
1. En el panel izquierdo de Azure Portal, seleccione **Grupos de recursos**.
2. En **Grupo de recursos**, en la parte superior de la página, seleccione **Agregar**.
3. En **Nombre del grupo de recursos**, escriba un nombre *myResourceGroupTM1*. Para **Ubicación del grupo de recursos**, seleccione **Este de EE. UU.** y, después, seleccione **Aceptar**.

## <a name="create-a-traffic-manager-profile"></a>Crear un perfil de Traffic Manager
Cree un perfil de Traffic Manager que dirija el tráfico de usuario mediante el envío al punto de conexión con la latencia más baja.

1. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso** > **Redes** > **Perfil de Traffic Manager** > **Crear**.
2. En **Crear perfil de Traffic Manager**, escriba o seleccione la siguiente información, acepte los valores predeterminados para el resto de la configuración y, a continuación, seleccione **Crear**:
    
    | Configuración                 | Value                                              |
    | ---                     | ---                                                |
    | Nombre                   | Este nombre debe ser único en la zona trafficmanager.net y generará el nombre DNS, trafficmanager.net, que se usa para acceder al perfil de Traffic Manager.                                   |
    | Método de enrutamiento          | Seleccione el método de enrutamiento de **varios valores**.                                       |
    | Subscription            | Seleccione su suscripción.                          |
    | Resource group          | Seleccione *myResourceGroupTM1*. |
    | Location                | Esta configuración se refiere a la ubicación del grupo de recursos y no tiene efecto alguno sobre el perfil de Traffic Manager que se implementará globalmente.                              |
   |        |           | 
  
   ![Crear un perfil de Traffic Manager](./media/traffic-manager-multivalue-routing-method/create-traffic-manager-profile.png)

## <a name="add-traffic-manager-endpoints"></a>Incorporación de puntos de conexión de Traffic Manager

Agregue dos direcciones IP como puntos de conexión externos para el perfil de Traffic Manager de varios valores que ha creado en el paso anterior.

1. En la barra de búsqueda del portal, busque el nombre del perfil de Traffic Manager que creó en la sección anterior y seleccione el perfil en los resultados que aparecen.
2. En **perfil de Traffic Manager**, en la sección **Configuración**, haga clic en **Puntos de conexión** y, a continuación, haga clic en **Agregar**.
3. Escriba o seleccione la siguiente información, acepte los valores predeterminados para el resto de la configuración y luego seleccione **Aceptar**:

    | Configuración                 | Value                                              |
    | ---                     | ---                                                |
    | Tipo                    | Punto de conexión externo                                   |
    | Nombre           | myEndpoint1                                        |
    | Nombre de dominio completo (FQDN) o IP           | Escriba la dirección IP pública del punto de conexión que desea agregar a este perfil de Traffic Manager                         |
    |        |           |

4. Repita los pasos 2 y 3 para agregar otro punto de conexión denominado *myEndpoint2*, para **Nombre de dominio completo (FQDN) o IP**, escriba la dirección IP pública del segundo punto de conexión.
5. Cuando termine de agregar ambos puntos de conexión, aparecerán en **Perfil de Traffic Manager** junto con el estado de supervisión como **En línea**.

   ![Incorporación de un punto de conexión de Traffic Manager](./media/traffic-manager-multivalue-routing-method/add-endpoint.png)
 
## <a name="next-steps"></a>Pasos siguientes

- Información sobre el [método de enrutamiento de tráfico ponderado](traffic-manager-configure-weighted-routing-method.md).
- Sepa cómo funciona el [método de enrutamiento por prioridad](traffic-manager-configure-priority-routing-method.md).
- Información sobre el [método de enrutamiento de rendimiento](traffic-manager-configure-performance-routing-method.md)
- Información sobre el [método de enrutamiento geográfico](traffic-manager-configure-geographic-routing-method.md).



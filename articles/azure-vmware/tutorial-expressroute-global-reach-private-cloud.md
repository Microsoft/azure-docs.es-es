---
title: Emparejamiento de entornos locales con una nube privada
description: En este tutorial de Azure VMware Solution se crea el emparejamiento de Global Reach de ExpressRoute con una nube privada de Azure VMware Solution.
ms.topic: tutorial
ms.date: 07/16/2020
ms.openlocfilehash: db3f5988cb8c07d9b6e80f500ac6aff8f96dfded
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88750450"
---
# <a name="tutorial-peer-on-premises-environments-to-a-private-cloud"></a>Tutorial: Emparejamiento de entornos locales con una nube privada

Global Reach de ExpressRoute conecta el entorno local a las nubes privadas. La conexión de Global Reach de ExpressRoute se establece entre un circuito ExpressRoute de la nube privada y una conexión ExpressRoute existente a los entornos locales.  Hay instrucciones para configurar Global Reach de ExpressRoute con la CLI de Azure y PowerShell y hemos ampliado los [comandos de la CLI](../expressroute/expressroute-howto-set-global-reach-cli.md) con detalles y ejemplos específicos que le ayudarán a configurar el emparejamiento de Global Reach de ExpressRoute entre los entornos locales y una nube privada de Azure VMware Solution.   

Antes de habilitar la conectividad entre dos circuitos ExpressRoute mediante Global Reach, consulte la documentación sobre cómo [habilitar la conectividad en distintas suscripciones de Azure](../expressroute/expressroute-howto-set-global-reach-cli.md#enable-connectivity-between-expressroute-circuits-in-different-azure-subscriptions).  El circuito ExpressRoute que se usa al [configurar redes de Azure a la nube privada](tutorial-configure-networking.md) requiere la creación y el uso de claves de autorización para emparejar puertas de enlace de ExpressRoute u otros circuitos ExpressRoute mediante Global Reach. Ya habrá usado una clave de autorización del circuito ExpressRoute y va a crear una segunda para el emparejamiento con el circuito ExpressRoute local.

> [!TIP]
> En el contexto de estas instrucciones, el circuito ExpressRoute local es _circuit 1_ y el circuito ExpressRoute de la nube privada está en otra suscripción con la etiqueta _circuit 2_. 

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Aprovechar las instrucciones existentes para administrar los circuitos ExpressRoute y los emparejamientos de Global Reach de ExpressRoute
> * Crear una clave de autorización para _circuit 2_, el circuito ExpressRoute de la nube privada
> * Usar la CLI de Azure en Cloud Shell en la suscripción de _circuit 1_ para habilitar el emparejamiento de Global Reach de ExpressRoute entre el entorno local y la nube privada

## <a name="prerequisites"></a>Requisitos previos

Los requisitos previos de este tutorial son los siguientes:
- Una nube privada con el circuito ExpressRoute emparejado con una puerta de enlace de ExpressRoute en una red virtual de Azure: _circuit 2_ desde la perspectiva de los procedimientos de emparejamiento.
- Un circuito ExpressRoute independiente y en funcionamiento que se usa para conectar entornos locales a Azure: _circuit 1_ desde la perspectiva de los procedimientos de emparejamiento.
- Un [bloque de direcciones de la red](../expressroute/expressroute-routing.md#ip-addresses-used-for-peerings) /29 sin superposición para el emparejamiento de Global Reach de ExpressRoute.

## <a name="create-an-expressroute-authorization-key-in-the-azure-vmware-solution-private-cloud"></a>Creación de una clave de autorización de ExpressRoute en la nube privada de Azure VMware Solution

1. En la nube privada **Información general**, en Administrar, seleccione **Conectividad > ExpressRoute > Solicitar una clave de autorización**.

   :::image type="content" source="media/expressroute-global-reach/start-request-auth-key.png" alt-text="Selección de Conectividad > ExpressRoute > Solicitar una clave de autorización para iniciar una nueva solicitud.":::

2. Escriba el nombre de la clave de autorización y seleccione **Crear**. 

   :::image type="content" source="media/expressroute-global-reach/create-global-reach-auth-key.png" alt-text="Haga clic en Crear para crear una clave de autorización. ":::

   Una vez creada, la nueva clave aparece en la lista de claves de autorización para la nube privada. 

   :::image type="content" source="media/expressroute-global-reach/show-global-reach-auth-key.png" alt-text="Confirme que la nueva clave de autorización aparece en la lista de claves de la nube privada. ":::

3. Tome nota de la clave de autorización y del identificador de ExpressRoute, junto con el bloque de direcciones /29. Los usará en el paso siguiente para completar el emparejamiento. 

## <a name="peer-private-cloud-to-on-premises-using-authorization-key"></a>Emparejamiento de la nube privada a un entorno local con clave de autorización

Ahora que ha creado una clave de autorización para el circuito ExpressRoute de la nube privada, puede emparejarlo con el circuito ExpressRoute local.  El emparejamiento se realiza desde la perspectiva del circuito ExpressRoute local mediante el CLI de Azure en Cloud Shell, y un identificador de recurso y una clave de autorización del circuito ExpressRoute de la nube privada (que se crearon en los pasos anteriores).

> [!TIP]  
> Para una salida más breve del comando de la CLI de Azure, estas instrucciones pueden usar un argumento [`–query` para ejecutar una consulta JMESPath y que se muestren solo los resultados necesarios](https://docs.microsoft.com/cli/azure/query-azure-cli?view=azure-cli-latest).


1. Inicie sesión en Azure Portal con la misma suscripción que el circuito ExpressRoute local y abra Cloud Shell. Deje el shell como Bash.
 
   :::image type="content" source="media/expressroute-global-reach/open-cloud-shell.png" alt-text="Inicie sesión en Azure Portal y abra Cloud Shell.":::
 
2. En la línea de comandos, escriba el comando de la CLI de Azure para crear el emparejamiento con su información e identificador de recurso específicos, la clave de autorización y el bloque de red /29 CIDR. 

   A continuación se muestra un ejemplo del comando que usará y la salida que indica un emparejamiento correcto. El comando de ejemplo se basa en el comando que se usa en el [paso 3 de "Habilitación de la conectividad entre circuitos ExpressRoute en distintas suscripciones de Azure"](../expressroute/expressroute-howto-set-global-reach-cli.md#enable-connectivity-between-expressroute-circuits-in-different-azure-subscriptions).

   :::image type="content" source="media/expressroute-global-reach/azure-command-with-results.png" alt-text="Creación del emparejamiento de Global Reach de ExpressRoute con el comando de la CLI de Azure en Cloud Shell.":::
 
   Ahora debería poder conectarse desde los entornos locales a la nube privada mediante el emparejamiento de Global Reach de ExpressRoute.

> [!TIP]
> Para eliminar el emparejamiento que acaba de crear siga las instrucciones de [Deshabilitación de la conectividad entre las redes locales](../expressroute/expressroute-howto-set-global-reach-cli.md#disable-connectivity-between-your-on-premises-networks).


## <a name="next-steps"></a>Pasos siguientes

Si tiene previsto usar la extensión de nube híbrida (HCX) para migrar las cargas de trabajo de máquinas virtuales a la nube privada, use el procedimiento [Instalación de HCX para Azure VMware Solution](hybrid-cloud-extension-installation.md).


<!-- LINKS - external-->

<!-- LINKS - internal -->

---
title: 'Tutorial: Creación y modificación de un circuito con ExpressRoute'
description: En este tutorial aprenderá a crear, aprovisionar, comprobar, actualizar, eliminar y desaprovisionar un circuito ExpressRoute.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: tutorial
ms.date: 09/01/2020
ms.author: duau
ms.openlocfilehash: 58c35b094d21dc562e61b4819c0d8e063908392d
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89322148"
---
# <a name="tutorial-create-and-modify-an-expressroute-circuit"></a>Tutorial: Creación y modificación de un circuito ExpressRoute

> [!div class="op_single_selector"]
> * [Azure Portal](expressroute-howto-circuit-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-circuit-arm.md)
> * [CLI de Azure](howto-circuit-cli.md)
> * [Plantilla de Azure Resource Manager](expressroute-howto-circuit-resource-manager-template.md)
> * [Vídeo: Azure Portal](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-an-expressroute-circuit)
> * [PowerShell (clásico)](expressroute-howto-circuit-classic.md)
>

En este tutorial se muestra cómo crear un circuito ExpressRoute mediante Azure Portal y el modelo de implementación de Azure Resource Manager. También puede comprobar el estado, la actualización, la eliminación o el desaprovisionamiento de un circuito.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Creación de un circuito ExpressRoute
> * Obtención del estado actual de un circuito
> * Modificación de un circuito
> * Desaprovisionamiento y eliminación de un circuito

## <a name="before-you-begin"></a>Antes de empezar

* Revise los [Requisitos previos y lista de comprobación de ExpressRoute](expressroute-prerequisites.md) y los [Flujos de trabajo de ExpressRoute para aprovisionamiento de circuitos y estados de circuitos de ExpressRoute](expressroute-workflows.md) antes de comenzar la configuración.
* Compruebe que tiene acceso al [Portal de Azure](https://portal.azure.com).
* Asegúrese de que tiene permisos para crear recursos de red. Si no tiene los permisos adecuados, póngase en contacto con el administrador de cuenta.
* Puede [ver un vídeo](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-an-expressroute-circuit) antes de comenzar para entender mejor los pasos.

## <a name="create-and-provision-an-expressroute-circuit"></a><a name="create"></a>Crear y aprovisionar un circuito ExpressRoute

### <a name="1-sign-in-to-the-azure-portal"></a>1. Inicio de sesión en Azure Portal

Desde un explorador, navegue al [Portal de Azure](https://portal.azure.com) e inicie sesión con su cuenta de Azure.

### <a name="2-create-a-new-expressroute-circuit"></a>2. Creación de un circuito ExpressRoute.

> [!IMPORTANT]
> El circuito ExpressRoute se factura a partir del momento en que se emite una clave de servicio. Asegúrese de realizar esta operación cuando el proveedor de conectividad esté listo para aprovisionar el circuito.

Puede crear un circuito ExpressRoute seleccionando la opción de creación de un recurso. 

1. En el menú de Azure Portal, seleccione **Crear un recurso**. Seleccione **Redes** > **ExpressRoute**, tal y como se muestra en la imagen siguiente:

    :::image type="content" source="./media/expressroute-howto-circuit-portal-resource-manager/create-expressroute-circuit-menu.png" alt-text="Creación de un circuito ExpressRoute":::

2. Tras hacer clic en **ExpressRoute**, verá la página **Crear circuito ExpressRoute**. Proporcione el **grupo de recursos**, la **región** y el **nombre** del circuito. A continuación, haga clic en **Siguiente: Configuración >** .

    :::image type="content" source="./media/expressroute-howto-circuit-portal-resource-manager/expressroute-create-basic.png" alt-text="Configuración del grupo de recursos y la región":::

3. Después de rellenar los valores de esta página, asegúrese de especificar el nivel de SKU adecuado (Local, Estándar o Premium) y el modelo de facturación de medición de datos (Ilimitado o Limitado).

    :::image type="content" source="./media/expressroute-howto-circuit-portal-resource-manager/expressroute-create-configuration.png" alt-text="Configuración del circuito":::
    
    * **Tipo de puerto** determina si se va a conectar a un proveedor de servicios o directamente a la red global de Microsoft en una ubicación de emparejamiento.
    * **Crear o importar a partir de clásico** determina si se está creando un circuito nuevo o si está migrando un circuito clásico a ARM.
    * **Proveedor** es el servicio de Internet que le proporciona a quién va a solicitar el servicio.
    * La **ubicación de emparejamiento** es la ubicación física de emparejamiento con Microsoft.

    > [!IMPORTANT]
    > La ubicación de emparejamiento indica la [ubicación física](expressroute-locations.md) de emparejamiento con Microsoft. **No** está vinculada a la propiedad Location, que hace referencia a la ubicación geográfica donde se encuentra el proveedor de recursos de red de Azure. Aunque no están relacionadas, se recomienda elegir un proveedor de recursos de red geográficamente cerca de la ubicación de emparejamiento del circuito.

    * **SKU** determina si está habilitado un complemento local, estándar o premium de ExpressRoute. Puede especificar **Local** para obtener la SKU local, **Estándar** para obtener la SKU estándar o **Premium** si quiere el complemento Premium.
    * **Modelo de facturación** determina el tipo de facturación. Puede especificar **Metered** (Limitado) para un plan de datos limitado y **Unlimited** (Ilimitado) para un plan de datos ilimitado. Tenga en cuenta que puede cambiar el tipo de facturación de **Uso medido** a **Sin límite**.

    > [!IMPORTANT]
    > No se puede cambiar el tipo de **Ilimitado** a **Limitado**.

    * **Permitir la operación clásica** permitirá que las redes virtuales clásicas se vinculen al circuito.

### <a name="3-view-the-circuits-and-properties"></a>3. Visualización de circuitos y propiedades

**Visualización de todos los circuitos**

Puede ver todos los circuitos creados seleccionando **Todos los servicios > Redes > Circuitos ExpressRoute** en el menú de la izquierda.

:::image type="content" source="./media/expressroute-howto-circuit-portal-resource-manager/expressroute-circuit-menu.png" alt-text="Menú Circuito de ExpressRoute":::

Todos los circuitos Expressroute creados en la suscripción se mostrarán aquí.

:::image type="content" source="./media/expressroute-howto-circuit-portal-resource-manager/expressroute-circuit-list.png" alt-text="Lista del Circuito de ExpressRoute":::

**Visualización de las propiedades**

Puede ver las propiedades del circuito seleccionándolo. En la página **Introducción** del circuito, la clave de servicio se muestra en el campo Clave de servicio. Consulte la clave de servicio del circuito y proporciónela al proveedor de servicios para completar el proceso de aprovisionamiento. La clave de servicio es específica para su circuito.

:::image type="content" source="./media/expressroute-howto-circuit-portal-resource-manager/expressroute-circuit-overview.png" alt-text="Vista de propiedades":::

### <a name="4-send-the-service-key-to-your-connectivity-provider-for-provisioning"></a>4. Envío de la clave de servicio al proveedor de conectividad para el aprovisionamiento

En esta página, en **Estado de proveedor** se ofrece información sobre el estado actual del aprovisionamiento en el lado del proveedor de servicios. **Estado de circuito** proporciona el estado relativo al lado de Microsoft. Para más información sobre los estados de aprovisionamiento del circuito, consulte el artículo [Flujos de trabajo de ExpressRoute para aprovisionamiento de circuitos y estados de circuitos de ExpressRoute](expressroute-workflows.md#expressroute-circuit-provisioning-states) .

Cuando se crea un nuevo circuito ExpressRoute, dicho circuito tiene el siguiente estado:

Estado de proveedor: **No aprovisionado**<BR>
Estado de circuito: **Enabled**

:::image type="content" source="./media/expressroute-howto-circuit-portal-resource-manager/expressroute-circuit-overview-provisioning-state.png" alt-text="Inicio del proceso de aprovisionamiento":::

El circuito pasa al estado siguiente cuando el proveedor de conectividad se encuentra en el proceso de habilitarlo:

Estado de proveedor: **Aprovisionamiento**<BR>
Estado de circuito: **Enabled**

Para poder usar un circuito ExpressRoute, dicho circuito tiene que estar en el siguiente estado.

Estado de proveedor: **Aprovisionado**<BR>
Estado de circuito: **Enabled**

### <a name="5-periodically-check-the-status-and-the-state-of-the-circuit-key"></a>5. Comprobación periódica del estado y la condición de la clave del circuito

Puede ver las propiedades del circuito que le interese seleccionándolo. Compruebe el **Estado de proveedor** y asegúrese de que se ha pasado a **Aprovisionado** antes de continuar.

:::image type="content" source="./media/expressroute-howto-circuit-portal-resource-manager/provisioned.png" alt-text="Estado del circuito y proveedor":::

### <a name="6-create-your-routing-configuration"></a>6. Creación de la configuración de enrutamiento

Consulte el artículo [Creación y modificación del enrutamiento de un circuito ExpressRoute mediante PowerShell](expressroute-howto-routing-portal-resource-manager.md) para ver las instrucciones paso a paso.

> [!IMPORTANT]
> Estas instrucciones se aplican solo a los circuitos creados con proveedores de servicios que ofrecen servicios de conectividad de nivel 2. Si usa un proveedor de servicios que ofrece servicios administrados de nivel 3 (normalmente VPN IP, como MPLS), el mismo proveedor de conectividad configura y administra el enrutamiento.

### <a name="7-link-a-virtual-network-to-an-expressroute-circuit"></a>7. Vinculación de una red virtual a un circuito ExpressRoute

A continuación, vincule una red virtual a su circuito ExpressRoute. Consulte el artículo [Vinculación de redes virtuales a circuitos ExpressRoute](expressroute-howto-linkvnet-arm.md) al trabajar con el modelo de implementación de Resource Manager.

## <a name="getting-the-status-of-an-expressroute-circuit"></a><a name="status"></a>Obtención del estado de un circuito ExpressRoute

Puede ver el estado de un circuito seleccionándolo y consultando la página Información.

## <a name="modifying-an-expressroute-circuit"></a><a name="modify"></a>Modificación de un circuito ExpressRoute

Puede modificar determinadas propiedades de un circuito ExpressRoute sin afectar a la conectividad. Puede modificar el ancho de banda, la SKU, el modelo de facturación y permitir operaciones clásicas en la página **Configuración**. Para obtener información sobre los límites y las limitaciones, consulte las [preguntas más frecuentes de ExpressRoute](expressroute-faqs.md).

Puede realizar las siguientes tareas sin experimentar tiempo de inactividad:

* Habilitar o deshabilitar un complemento ExpressRoute Premium en su circuito ExpressRoute.
* Aumente el ancho de banda del circuito ExpressRoute, siempre que haya capacidad disponible en el puerto.

  > [!IMPORTANT]
  > No se admite la degradación del ancho de banda de un circuito.

* Cambie el plan de medición de *Datos limitados* a *Datos ilimitados*.

  > [!IMPORTANT]
  > No se admite cambiar el plan de medición de **datos ilimitados** a **datos limitados**.

* Puede habilitar y deshabilitar *Allow Classic Operations*(Permitir operaciones clásicas).
  > [!IMPORTANT]
  > Si el puerto existente no tiene la capacidad adecuada, tendrá que volver a crear el circuito ExpressRoute. El circuito no se puede actualizar si no hay más capacidad disponible en la ubicación.
  >
  > Aunque puede actualizar el ancho de banda sin problemas, no puede reducir el ancho de banda de un circuito de ExpressRoute sin interrupciones. Para degradar un ancho de banda, es necesario desaprovisionar el circuito ExpressRoute y luego volver a aprovisionar un nuevo circuito ExpressRoute.
  >
  > La operación de deshabilitación del complemento premium puede producir un error si usa recursos que son más grandes de lo que está permitido para el circuito estándar.

Para modificar un circuito ExpressRoute, haga clic en **Configuración**.

:::image type="content" source="./media/expressroute-howto-circuit-portal-resource-manager/expressroute-circuit-configuration.png" alt-text="Modificación del circuito":::

## <a name="deprovisioning-and-deleting-an-expressroute-circuit"></a><a name="delete"></a>Desaprovisionar y eliminar un circuito ExpressRoute

Si el estado de aprovisionamiento del proveedor de servicios del circuito ExpressRoute es **Aprovisionando** o **Aprovisionado**, debe colaborar con su proveedor de servicios para que desaprovisionen el circuito. Se le siguen reservando recursos y facturándole por ello hasta que el proveedor de servicios complete el desaprovisionamiento del circuito y nos lo notifique.

> [!NOTE]
>* Debe desvincular *todas las redes virtuales* del circuito ExpressRoute antes del desaprovisionamiento. Si se produce un error en esta operación, compruebe si hay alguna red virtual vinculada al circuito.
>* Si el proveedor de servicios ha desaprovisionado el circuito (el estado de aprovisionamiento del proveedor de servicios está establecido en **No aprovisionado**), puede eliminar el circuito. Esto detiene la facturación del circuito.

Para eliminar el circuito ExpressRoute, puede seleccionar el icono **Eliminar**. 

:::image type="content" source="./media/expressroute-howto-circuit-portal-resource-manager/expressroute-circuit-delete.png" alt-text="Eliminación del circuito":::

## <a name="next-steps"></a>Pasos siguientes

Después de crear el circuito, lleve a cabo los pasos siguientes:

* [Crear y modificar el enrutamiento para el circuito ExpressRoute](expressroute-howto-routing-portal-resource-manager.md)
* [Vincular la red virtual a su circuito ExpressRoute](expressroute-howto-linkvnet-arm.md)
---
title: 'Azure ExpressRoute: Configuración de ExpressRoute Direct: portal'
description: Esta página le ayuda a configurar ExpressRoute Direct mediante el portal.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 08/06/2020
ms.author: duau
ms.openlocfilehash: 09d3723224f9c678e41a6cbc86b5f8f9655173c8
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89401476"
---
# <a name="create-expressroute-direct-using-the-azure-portal"></a>Creación de un recurso de ExpressRoute Direct mediante Azure Portal

En este artículo se muestra cómo crear un recurso de ExpressRoute Direct mediante Azure Portal.
ExpressRoute Direct le permite conectarse directamente a la red global de Microsoft en ubicaciones de emparejamiento distribuidas estratégicamente por todo el mundo. Para obtener más información, consulte [Acerca de ExpressRoute Direct](expressroute-erdirect-about.md).

## <a name="before-you-begin"></a><a name="before"></a>Antes de empezar

Compruebe que el proveedor de recursos **Microsoft.Network** está registrado en su suscripción. Al registrar un proveedor de recursos se configura la suscripción para que funcione con este.

1. Acceda a la configuración de la suscripción como se describe en [Proveedores y tipos de recursos de Azure](../azure-resource-manager/management/resource-providers-and-types.md).
1. En la suscripción, en **Proveedores de recursos**, compruebe que el proveedor **Microsoft.Network** muestra un estado **Registrado**. Si el proveedor de recursos Microsoft.Network no está en la lista de proveedores registrados, agréguelo.

## <a name="1-create-expressroute-direct"></a><a name="create-erdir"></a>1. Creación de un recurso de ExpressRoute Direct

1. En el menú [Azure Portal](https://portal.azure.com) o en la página **Inicio**, seleccione **Crear un recurso**.

1. En la página **Nuevo**, en el campo ***Buscar en Marketplace***, escriba **ExpressRoute Direct** y haga seleccione **Entrar** para ir a los resultados de la búsqueda.

1. En los resultados, seleccione **ExpressRoute Direct**.

1. En la página **ExpressRoute Direct**, seleccione **Crear** para abrir la página **Create ExpressRoute Direct** (Crear recurso de ExpressRoute Direct).

1. Empiece por completar los campos de la página **Datos básicos**.

    :::image type="content" source="./media/how-to-expressroute-direct-portal/basics.png" alt-text="Datos básicos":::

    * **Suscripción**: suscripción de Azure que quiere usar para crear un nuevo recurso de ExpressRoute Direct. El recurso de ExpressRoute Direct y los circuitos de ExpressRoute deben estar en la misma suscripción.
    * **Grupo de recursos**: grupo de recursos de Azure en el que se creará el nuevo recurso de ExpressRoute Direct. Si no tiene un grupo de recursos existente, puede crear uno.
    * **Región**: región pública de Azure en la que se creará el recurso.
    * **Name**: nombre del nuevo recurso directo de ExpressRoute Direct.

1. A continuación, complete los campos de la página **Configuración**.

    :::image type="content" source="./media/how-to-expressroute-direct-portal/configuration.png" alt-text="Página Configuración":::

    * **Ubicación del emparejamiento**: ubicación del emparejamiento en la que se conectará al recurso de ExpressRoute Direct. Para más información sobre las ubicaciones de emparejamiento, consulte [Ubicaciones de ExpressRoute](expressroute-locations-providers.md).
   * **Ancho de banda**: ancho de banda del par de puertos que desea reservar. ExpressRoute Direct admite opciones de ancho de banda de 10 GB y 100 GB. Si el ancho de banda deseado no está disponible en la ubicación de emparejamiento especificada, [abra una solicitud de soporte técnico en Azure Portal](https://aka.ms/azsupt).
   * **Encapsulación:** ExpressRoute Direct admite la encapsulación de tipo QinQ y Dot1Q.
     * Si se selecciona QinQ, a cada circuito de ExpressRoute se le asignará dinámicamente una S-Tag y será único en todo el recurso de ExpressRoute Direct.
     *  Cada C-Tag del circuito debe ser única, pero no en ExpressRoute Direct.
     * Si se selecciona la encapsulación Dot1Q, debe administrar la exclusividad de C-Tag (VLAN) en todo el recurso ExpressRoute Direct.
     >[!IMPORTANT]
     >ExpressRoute Direct solo puede ser un tipo de encapsulación. La encapsulación no se puede cambiar después de crear ExpressRoute Direct.
     >

1. Especifique las etiquetas de recursos y seleccione **Revisar y crear** para validar la configuración de recursos de ExpressRoute Direct.

    :::image type="content" source="./media/how-to-expressroute-direct-portal/validate.png" alt-text="Revisar y crear":::

1. Seleccione **Crear**. Verá un mensaje en el que se le indica que la implementación está en curso. El estado se mostrará en esta página a medida que se creen los recursos. 

## <a name="2-change-admin-state-of-links"></a><a name="state"></a>2. Cambiar el estado de administración de vínculos

Este proceso debe usarse para llevar a cabo una prueba de nivel 1, para garantizar que cada conexión cruzada está correctamente revisada en cada enrutador principal y secundario.

1. En la página **Introducción** del recurso de ExpressRoute Direct, en la sección **Vínculos**, seleccione **vínculo1**.

    :::image type="content" source="./media/how-to-expressroute-direct-portal/link.png" alt-text="Vínculo 1" lightbox="./media/how-to-expressroute-direct-portal/link-expand.png":::

1. Cambie la configuración de **Admin State** (Estado de administración) a **Habilitado** y luego seleccione **Guardar**.

    :::image type="content" source="./media/how-to-expressroute-direct-portal/state.png" alt-text="Admin State"::: (Estado de administración)

    >[!IMPORTANT]
    >La facturación comenzará cuando el estado de administración esté habilitado en cualquiera de los vínculos.
    >

1. Repita el mismo proceso para **vínculo2**.

## <a name="3-create-a-circuit"></a><a name="circuit"></a>3. Crear un circuito

De forma predeterminada, puede crear 10 circuitos en la suscripción donde se encuentra el recurso ExpressRoute Direct. Si desea aumentar este número, puede ponerse en contacto con el soporte técnico. Recuerde que debe realizar usted mismo el seguimiento tanto del ancho de banda aprovisionado como el del utilizado. El ancho de banda aprovisionado es la suma del ancho de banda de todos los circuitos en el recurso de ExpressRoute Direct. El ancho de banda utilizado es el uso físico de las interfaces físicas subyacentes.

* Asimismo, existen anchos de banda de circuito adicionales que se pueden utilizar en ExpressRoute Direct solo para admitir los escenarios descritos anteriormente. Dichos componentes son: 40 Gbps y 100 Gbps.

* SkuTier puede ser Local, Estándar o Premium.

* SkuFamily solo debe ser MeteredData. Ilimitado no se admite en ExpressRoute Direct.

Los siguientes pasos le ayudarán a crear un circuito ExpressRoute desde el flujo de trabajo ExpressRoute Direct. En lugar de ello, también puede crear un circuito mediante el flujo de trabajo de circuito normal, aunque no hay ninguna ventaja en el uso de los pasos del flujo de trabajo de circuito convencional para esta configuración. Consulte [Creación y modificación de un circuito ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md).

1. En la sección **Configuración** de ExpressRoute Direct, seleccione **Circuitos** y luego **+Agregar**. 

    :::image type="content" source="./media/how-to-expressroute-direct-portal/add.png" alt-text="Add (Agregar)" lightbox="./media/how-to-expressroute-direct-portal/add-expand.png":::

1. Defina la configuración en la página **Configuración**.

   :::image type="content" source="./media/how-to-expressroute-direct-portal/configuration2.png" alt-text="Página Configuración":::

1. Especifique cualquier etiqueta de recurso, seleccione **Revisar y crear** para validar los valores antes de crear el recurso.

   :::image type="content" source="./media/how-to-expressroute-direct-portal/review.png" alt-text="Revisar y crear":::

1. Seleccione **Crear**. Verá un mensaje en el que se le indica que la implementación está en curso. El estado se mostrará en esta página a medida que se creen los recursos. 

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre ExpressRoute Direct, consulte la [información general](expressroute-erdirect-about.md).

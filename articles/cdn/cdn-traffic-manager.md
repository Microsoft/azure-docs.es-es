---
title: Configuración de la conmutación por error entre varios puntos de conexión de Azure CDN con Traffic Manager
description: Obtenga información sobre cómo configurar la conmutación por error en varios puntos de conexión de Azure Content Delivery Network mediante Azure Traffic Manager.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 03/18/2019
ms.author: allensu
ms.custom: ''
ms.openlocfilehash: b55e418393d6d446ae0d3557f2d1f4cf98d89293
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88192510"
---
# <a name="set-up-failover-across-multiple-azure-cdn-endpoints-with-azure-traffic-manager"></a>Configurar la conmutación por error entre varios puntos de conexión de Azure CDN con Azure Traffic Manager

Al configurar Azure Content Delivery Network (CDN), puede seleccionar el proveedor óptimo y el plan de tarifa que mejor se ajuste a sus necesidades. Azure CDN, con su infraestructura distribuida globalmente, crea de forma predeterminada redundancia local y geográfica, así como equilibrio de carga global para mejorar el rendimiento y disponibilidad del servicio. Si una ubicación no está disponible para servir el contenido, las solicitudes se enrutan automáticamente a otra ubicación y se usa el POP óptimo (según factores como la ubicación y la carga del servidor) para atender cada solicitud de cliente. 
 
Si tiene varios perfiles de CDN, puede mejorar aún más la disponibilidad y el rendimiento con Azure Traffic Manager. Puede usar Azure Traffic Manager con Azure CDN para equilibrar la carga entre varios puntos de conexión de CDN para la conmutación por error, el geo-equilibrio de carga y otros escenarios. En un escenario típico de conmutación por error, todas las solicitudes de cliente se dirigen en primer lugar al perfil de CDN principal; si el perfil no está disponible, las solicitudes se pasan al perfil de CDN secundario hasta que el perfil de CDN principal se vuelve a conectar. Al usar Azure Traffic Manager de este modo se garantiza que la aplicación web esté disponible siempre. 

En este artículo se proporcionan instrucciones y un ejemplo sobre cómo configurar la conmutación por error con perfiles de **Azure CDN estándar de Verizon** y **Azure CDN estándar de Akamai**.

## <a name="set-up-azure-cdn"></a>Configuración de Azure CDN 
Cree dos o más perfiles de Azure CDN y puntos de conexión con diferentes proveedores.

1. Cree perfiles de **Azure CDN estándar de Verizon** y **Azure CDN estándar de Akamai** siguiendo los pasos descritos en [Crear un nuevo perfil de CDN](cdn-create-new-endpoint.md#create-a-new-cdn-profile).
 
   ![Varios perfiles de CDN](./media/cdn-traffic-manager/cdn-multiple-profiles.png)

2. En cada uno de los perfiles nuevos, cree al menos un punto de conexión siguiendo los pasos descritos en [Crear un nuevo extremo de CDN](cdn-create-new-endpoint.md#create-a-new-cdn-endpoint).

## <a name="set-up-azure-traffic-manager"></a>Configurar Azure Traffic Manager
Cree un perfil de Traffic Manager de Azure y configure el equilibrio de carga entre los puntos de conexión de CDN. 

1. Cree un perfil de Azure Traffic Manager siguiendo los pasos descritos en [Crear un perfil de Traffic Manager](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-create-profile). 

    Para **Método de enrutamiento**, seleccione **Prioridad**.

2. Agregue los puntos de conexión de CDN a su perfil de Traffic Manager siguiendo los pasos descritos en [Incorporación de puntos de conexión de Traffic Manager](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-create-profile#add-traffic-manager-endpoints).

    Para **Tipo**, seleccione **Puntos de conexión externos**. Para **Prioridad**, escriba un número.

    Por ejemplo, cree *cdndemo101akamai.azureedge.net* con una prioridad de *1* y *cdndemo101verizon.azureedge.net* con una prioridad de *2*.

   ![Puntos de conexión de CDN y Traffic Manager](./media/cdn-traffic-manager/cdn-traffic-manager-endpoints.png)


## <a name="set-up-custom-domain-on-azure-cdn-and-azure-traffic-manager"></a>Configurar un dominio personalizado en Azure CDN y Azure Traffic Manager
Después de configurar los perfiles de CDN y Traffic Manager, siga estos pasos para agregar la asignación de DNS y registrar el dominio personalizado en los puntos de conexión de CDN. En este ejemplo, el nombre de dominio personalizado es *cdndemo101.dustydogpetcare.online*.

1. Vaya al sitio web para el proveedor del dominio personalizado, como GoDaddy y cree dos entradas CNAME de DNS. 

    a. Para la primera entrada CNAME, asigne su dominio personalizado, con el subdominio cdnverify, al punto de conexión de CDN. Esta entrada es un paso necesario para registrar el dominio personalizado en el punto de conexión de CDN que agregó a Traffic Manager en el paso 2.

      Por ejemplo: 

      `cdnverify.cdndemo101.dustydogpetcare.online  CNAME  cdnverify.cdndemo101akamai.azureedge.net`  

    b. Para la segunda entrada de CNAME, asigne su dominio personalizado, sin el subdominio cdnverify, al punto de conexión de CDN. Esta entrada asigna el dominio personalizado a Traffic Manager. 

      Por ejemplo: 
      
      `cdndemo101.dustydogpetcare.online  CNAME  cdndemo101.trafficmanager.net`   

    > [!NOTE]
    > Si el dominio está actualmente activo y no se puede interrumpir, realice este paso al final. Compruebe que los puntos de conexión de CDN y los dominios de Traffic Manager estén activos antes de actualizar el DNS de dominio personalizado a Traffic Manager.
    >


2.  En el perfil de Azure CDN, seleccione el primer punto de conexión de CDN (Akamai). Seleccione **Agregar dominio personalizado** y especifique *cdndemo101.dustydogpetcare.online*. Compruebe que la marca de verificación para validar el dominio personalizado sea de color verde. 

    Azure CDN usa el subdominio *cdnverify* para validar la asignación de DNS para completar este proceso de registro. Para más información, consulte [Creación de un registro DNS de CNAME](cdn-map-content-to-custom-domain.md#create-a-cname-dns-record). En este paso se habilita a Azure CDN para que reconozca el dominio personalizado de modo que pueda responder a sus solicitudes.
    
    > [!NOTE]
    > Para habilitar TLS en una instancia de **Azure CDN desde perfiles de Akamai**, debe aplicar directamente cname de dominio personalizado al punto de conexión. cdnverify para habilitar TLS aún no se admite. 
    >

3.  Vuelva al sitio web para el proveedor del dominio personalizado y actualice la primera asignación de DNS que creó para que el dominio personalizado se asigne al segundo punto de conexión de CDN.
                             
    Por ejemplo: 

    `cdnverify.cdndemo101.dustydogpetcare.online  CNAME  cdnverify.cdndemo101verizon.azureedge.net`  

4. En el perfil de Azure CDN, seleccione el segundo punto de conexión de CDN (Verizon) y repita el paso 2. Seleccione **Agregar dominio personalizado** y especifique *cdndemo101.dustydogpetcare.online*.
 
Después de completar estos pasos, el servicio de varias CDN con capacidades de conmutación por error está configurado con Azure Traffic Manager. Podrá obtener acceso a las direcciones URL de prueba desde el dominio personalizado. Para probar la funcionalidad, deshabilite el punto de conexión de CDN principal y compruebe que la solicitud se mueve correctamente al punto de conexión de CDN secundario. 

## <a name="next-steps"></a>Pasos siguientes
También puede configurar otros métodos de enrutamiento, como el geográfico, para equilibrar la carga entre distintos puntos de conexión de CDN. Para obtener más información, consulte [Configuración del método de enrutamiento de tráfico geográfico con Traffic Manager](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-configure-geographic-routing-method).




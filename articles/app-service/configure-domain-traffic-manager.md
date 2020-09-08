---
title: Configuración de nombres DNS con Traffic Manager
description: Aprenda a configurar un dominio personalizado para una aplicación de Azure App Service que se integra con Traffic Manager para el equilibrio de carga.
ms.assetid: 0f96c0e7-0901-489b-a95a-e3b66ca0a1c2
ms.topic: article
ms.date: 03/05/2020
ms.custom: seodec18
ms.openlocfilehash: 0e8d5fa14678a2a26234dfcd73f4a50af62ca7aa
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962883"
---
# <a name="configure-a-custom-domain-name-in-azure-app-service-with-traffic-manager-integration"></a>Configuración de un nombre de dominio personalizado en Azure App Service con la integración de Traffic Manager

[!INCLUDE [web-selector](../../includes/websites-custom-domain-selector.md)]

> [!NOTE]
> Para Cloud Services, vea [Configuración de un nombre de dominio personalizado para un servicio en la nube de Azure](../cloud-services/cloud-services-custom-domain-name-portal.md).

Cuando se usa [Azure Traffic Manager](../traffic-manager/index.yml) para equilibrar la carga de tráfico a [Azure App Service](overview.md), se puede acceder a la aplicación de App Service mediante **\<traffic-manager-endpoint>.trafficmanager.net**. También puede asignar un nombre de dominio personalizado, como www\.contoso.com, a la aplicación App Service con el fin de proporcionar un nombre de dominio más reconocible para los usuarios.

En este artículo se muestra cómo configurar un nombre de dominio personalizado con una aplicación App Service que se integra con [Traffic Manager](../traffic-manager/traffic-manager-overview.md).

> [!NOTE]
> Solo se admiten registros [CNAME](https://en.wikipedia.org/wiki/CNAME_record) al configurar un nombre de dominio mediante el punto de conexión de Traffic Manager. Dado que no se admiten registros D, tampoco se admite una asignación de dominio raíz, como contoso.com.
> 

## <a name="prepare-the-app"></a>Preparación de la aplicación

Para asignar un nombre DNS personalizado a una aplicación que se integra con Azure Traffic Manager, el [plan de App Service](https://azure.microsoft.com/pricing/details/app-service/) de la aplicación web debe estar en el nivel **Estándar** o superior. En este paso, asegúrese de que la aplicación de App Service se encuentra en el plan de tarifa compatible.

### <a name="check-the-pricing-tier"></a>Comprobar el plan de tarifa

En [Azure Portal](https://portal.azure.com), busque y seleccione **App Services**.

En la página **App Services**, seleccione el nombre de la aplicación de Azure.

![Navegación en el portal a la aplicación de Azure](./media/app-service-web-tutorial-custom-domain/select-app.png)

En el panel izquierdo de navegación de la página de la aplicación, seleccione **Escalar verticalmente (plan de App Service)** .

![Menú Escalar verticalmente](./media/app-service-web-tutorial-custom-domain/scale-up-menu.png)

El nivel actual de la aplicación aparece resaltado con un cuadro azul. Asegúrese de que la aplicación se encuentra en el nivel **Estándar** o superior (cualquier nivel de la categoría **Producción** o **Aislado**). En caso afirmativo, cierre la página **Escalar verticalmente** y vaya a [Create the CNAME mapping](#create-the-cname-mapping) (Crear la asignación de CNAME).

![Comprobar plan de tarifa](./media/app-service-web-tutorial-custom-domain/check-pricing-tier.png)

### <a name="scale-up-the-app-service-plan"></a>Escalado vertical del plan de App Service

Si necesita escalar verticalmente la aplicación, seleccione cualquiera de los planes de tarifa de la categoría **Production** (Producción). Para ver opciones adicionales, haga clic en **Ver opciones adicionales**.

Haga clic en **Aplicar**.

## <a name="create-traffic-manager-endpoint"></a>Creación de un punto de conexión de Traffic Manager

Siga los pasos descritos en [Incorporación, deshabilitación, habilitación o eliminación de puntos de conexión](../traffic-manager/traffic-manager-manage-endpoints.md) y agregue la aplicación App Service como un punto de conexión en su perfil de Traffic Manager.

Una vez que la aplicación App Service se encuentre en un plan de tarifa admitido, se mostrará en la lista de destinos de App Service disponibles al agregar el punto de conexión. Si la aplicación no aparece en la lista, [compruebe su plan de tarifa](#prepare-the-app).

## <a name="create-the-cname-mapping"></a>Creación de la asignación de CNAME
> [!NOTE]
> Para configurar un [dominio de App Service que haya adquirido](manage-custom-dns-buy-domain.md), omita esta sección y vaya a [Habilitación del dominio personalizado](#enable-custom-domain).
> 

[!INCLUDE [Access DNS records with domain provider](../../includes/app-service-web-access-dns-records-no-h.md)]

Aunque los detalles de cada proveedor de dominio varían, en general se asigna *desde* un [nombre de dominio personalizado no raíz](#what-about-root-domains) (como **www.contoso.com**) *al* nombre de dominio de Traffic Manager (**contoso.trafficmanager.net**) que se integra con la aplicación. 

> [!NOTE]
> Si un registro ya se está usando y necesita enlazar de forma preventiva sus aplicaciones a él, puede crear otro registro CNAME. Por ejemplo, para enlazar de forma preferente **www\.contoso.com** a su aplicación, cree un registro CNAME de **awverify.www** a **contoso.trafficmanager.net**. Después, puede agregar "www\.contoso.com" a la aplicación web sin tener que cambiar el registro CNAME de "www". Para más información, consulte [Migración de un nombre de DNS activo a Azure App Service](manage-custom-dns-migrate-domain.md).

Una vez que haya terminado de agregar o modificar registros DNS en su proveedor de dominio, guarde los cambios.

### <a name="what-about-root-domains"></a>¿Qué ocurre con los dominios raíz?

Como Traffic Manager solo admite la asignación de dominios personalizados con registros CNAME, y dado que los estándares DNS no admiten registros CNAME para asignar dominios raíz (por ejemplo, **contoso.com**), Traffic Manager no admite la asignación a dominios raíz. Para solucionar este problema, use una dirección URL redirigida desde el nivel de la aplicación. En ASP.NET Core, por ejemplo, puede usar [Reescritura de direcciones URL](/aspnet/core/fundamentals/url-rewriting). A continuación, use Traffic Manager para equilibrar la carga del subdominio (**www.contoso.com**).

En escenarios de alta disponibilidad, puede implementar una configuración de DNS tolerante a errores sin Traffic Manager creando varios *registros A* que apunten del dominio raíz a cada dirección IP de la copia de la aplicación. A continuación, [asigne el mismo dominio raíz a todas las copias de la aplicación](app-service-web-tutorial-custom-domain.md#map-an-a-record). Dado que el mismo nombre de dominio no se puede asignar a dos aplicaciones diferentes en la misma región, este programa de instalación solo funciona cuando las copias de la aplicación se encuentran en regiones diferentes.

## <a name="enable-custom-domain"></a>Habilitación del dominio personalizado
Una vez que se hayan propagado los registros de su nombre de dominio, use el explorador para comprobar que el nombre de dominio personalizado se resuelve en la aplicación App Service.

> [!NOTE]
> El CNAME puede tardar un tiempo en propagarse por el sistema DNS. Puede usar un servicio como <a href="https://www.digwebinterface.com/">https://www.digwebinterface.com/</a> para comprobar que el CNAME está disponible.
> 
> 

1. Una vez que la resolución de dominio se realice correctamente, vuelva a la página de la aplicación en [Azure Portal](https://portal.azure.com).
2. En el panel de navegación izquierdo, seleccione **Dominios personalizados** > **Agregar nombre de host**.
4. Escriba el nombre de dominio personalizado que asignó anteriormente y seleccione **Validar**.
5. Asegúrese de que en **Tipo de registro de nombre de host** está seleccionado **CNAME (www\.example.com o cualquier subdominio)** .

6. Dado que la aplicación App Service ahora está integrada con un punto de conexión de Traffic Manager, debería ver el nombre de dominio de Traffic Manager en **Configuración de CNAME**. Selecciónelo y haga clic en **Agregar dominio personalizado**.

    ![Agregar nombre DNS a la aplicación](./media/configure-domain-traffic-manager/enable-traffic-manager-domain.png)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Protección de un nombre DNS personalizado con un enlace SSL en Azure App Service](configure-ssl-bindings.md)
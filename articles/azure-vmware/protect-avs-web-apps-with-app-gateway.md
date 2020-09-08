---
title: Uso de Azure Application Gateway para proteger aplicaciones web en Azure VMware Solution
description: Configure Azure Application Gateway para exponer de forma segura las aplicaciones web que se ejecutan en Azure VMware Solution.
ms.topic: how-to
ms.date: 07/31/2020
ms.openlocfilehash: d4e193c58c5eccb29f603c3b4d56a09d26686975
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88750602"
---
# <a name="use-azure-application-gateway-to-protect-your-web-apps-on-azure-vmware-solution"></a>Uso de Azure Application Gateway para proteger aplicaciones web en Azure VMware Solution

[Azure Application Gateway](https://azure.microsoft.com/services/application-gateway/) es un equilibrador de carga de tráfico web de capa 7 que permite administrar el tráfico a las aplicaciones web. Ofrece muchas funcionalidades: afinidad de sesión basada en cookies, enrutamiento basado en URL y firewall de aplicaciones web (WAF), entre otras. Para ver una lista completa de las características, consulte [Características de Azure Application Gateway](../application-gateway/features.md). Se ofrece en dos versiones: v1 y v2. Ambas se han probado con aplicaciones web en ejecución en Azure VMware Solution.

En este artículo, se describe paso a paso un escenario común en el que se usa Application Gateway delante de una granja de servidores web con un conjunto de configuraciones y recomendaciones para proteger una aplicación web que se ejecuta en Azure VMware Solution. 

## <a name="topology"></a>Topología
Como se muestra en la ilustración siguiente, Application Gateway se puede usar para proteger máquinas virtuales de IaaS de Azure, conjuntos de escalado de máquinas virtuales de Azure o servidores locales. Application Gateway tratará las máquinas virtuales de Azure VMware Solution como servidores locales.

![Application Gateway protege las máquinas virtuales de Azure VMware Solution.](media/protect-avs-web-apps-with-app-gw/app-gateway-protects.png)

> [!IMPORTANT]
> Azure Application Gateway es actualmente el único método admitido para exponer las aplicaciones web que se ejecutan en máquinas virtuales de Azure VMware Solution.

En el siguiente diagrama se muestra el escenario de prueba que se usa para validar Application Gateway con aplicaciones web de Azure VMware Solution.

![Integración de Application Gateway con aplicaciones web en ejecución en Azure VMware Solution.](media/protect-avs-web-apps-with-app-gw/app-gateway-avs-scenario.png)

La instancia de Application Gateway se implementa en el centro de una subred dedicada. Tiene una dirección IP pública de Azure. Se recomienda activar la protección contra DDoS estándar para la red virtual. El servidor web se hospeda en una nube privada de Azure VMware Solution detrás de los enrutadores NSX T0 y T1. Azure VMware Solution usa [Global Reach de ExpressRoute](../expressroute/expressroute-global-reach.md) para habilitar la comunicación con el centro de conectividad y los sistemas locales.

## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure con una suscripción activa.
- Una nube privada de Azure VMware Solution implementada y en ejecución.

## <a name="deployment-and-configuration"></a>Implementación y configuración

1. En Azure Portal, busque **Application Gateway** y seleccione **Crear puerta de enlace de aplicaciones**.

2. Proporcione los detalles básicos como se indica en la ilustración siguiente. A continuación, seleccione **Siguiente:Front-end>** . 

    :::image type="content" source="media/protect-avs-web-apps-with-app-gw/create-app-gateway.png" alt-text="Creación de la instancia de Application Gateway":::

3. Seleccione el tipo de dirección IP de front-end. Para que sea pública, seleccione una dirección IP pública existente o cree una nueva. Seleccione **Siguiente:Back-end>** .

    > [!NOTE]
    > Solo se admiten SKU estándar y del firewall de aplicaciones web para los servidores front-end privados.

4. A continuación, agregue un grupo de back-end, que describe un conjunto de instancias que forman parte de la aplicación o servicio (en este caso, las máquinas virtuales que se ejecutan en la infraestructura de Azure VMware Solution). Proporcione los detalles de los servidores web que se ejecutan en la nube privada de Azure VMware Solution, seleccione **Agregar** y, a continuación, seleccione **Siguiente:Configuración>** .

1. En la pestaña **Configuración**, seleccione **Agregar una regla de enrutamiento**.

6. En la pestaña **Cliente de escucha**, proporcione los detalles del cliente de escucha. Si se selecciona HTTPS, se debe proporcionar un certificado, ya sea de un archivo PFX o un certificado existente de Azure Key Vault. 

7. Seleccione la pestaña **Destinos de back-end** y seleccione el grupo de back-end que creó anteriormente. En el campo **Configuración HTTP**, seleccione **Agregar nueva**.

8. Configure los parámetros para la configuración HTTP. Seleccione **Agregar**.

9. Si desea configurar reglas basadas en rutas de acceso, seleccione **Agregar varios destinos para crear una regla basada en ruta de acceso**. 

10. Agregue una regla basada en ruta de acceso y seleccione **Agregar**. Repita este paso para agregar más reglas basadas en rutas de acceso. 

11. Cuando haya terminado de agregar reglas basadas en rutas de acceso, seleccione **Agregar** de nuevo. A continuación, seleccione **Siguiente:Etiquetas>** . 

12. Agregue todas las etiquetas que desee. Seleccione **Siguiente:Revisar y crear**.

13. Se ejecutará una validación en la instancia de Application Gateway. Si es correcta, seleccione **Crear** para implementar.

## <a name="configuration-examples"></a>Ejemplos de configuración

En esta sección, aprenderá a configurar Application Gateway con máquinas virtuales de Azure VMware Solution como grupos de back-end para los siguientes casos de uso: 

- [Hospedaje de varios sitios](#hosting-multiple-sites)
- [Enrutamiento por dirección URL](#routing-by-url)

### <a name="hosting-multiple-sites"></a>Hospedaje de varios sitios

Puede usar Azure Portal para configurar el hospedaje de varios sitios web al crear una puerta de enlace de aplicación. En este tutorial, definirá grupos de direcciones de back-end mediante máquinas virtuales que se ejecutan en una nube privada de Azure VMware Solution en una puerta de enlace de aplicación existente. La puerta de enlace de aplicación forma parte de una red virtual del centro de conectividad, tal como se describe en [Integración de Azure VMware Solution en una arquitectura en estrella tipo hub-and-spoke](concepts-avs-hub-and-spoke-integration.md). En este tutorial se da por supuesto que posee varios dominios y se van a utilizar ejemplos de www.contoso.com y www.fabrikam.com.

1. Cree las máquinas virtuales. En la nube privada de Azure VMware Solution, cree dos grupos diferentes de máquinas virtuales. Uno representará a Contoso y el otro a Fabrikam. 

    :::image type="content" source="media/protect-avs-web-apps-with-app-gw/app-gateway-multi-backend-pool-avs.png" alt-text="Cree las máquinas virtuales.":::

    Para ilustrar este tutorial, hemos usado Windows Server 2016 con el rol Internet Information Services (IIS) instalado. Una vez instaladas las máquinas virtuales, ejecute los siguientes comandos de PowerShell para configurar IIS en cada una de ellas. 

    ```powershell
    Install-WindowsFeature -Name Web-Server
    Add-Content -Path C:\inetpub\wwwroot\Default.htm -Value $($env:computername)
    ```

2. Agregue los grupos de back-end. En una instancia de Application Gateway existente, seleccione **Grupos de back-end** en el menú de la izquierda, seleccione **Agregar** y especifique los detalles de los grupos nuevos. Seleccione **Agregar** en el panel derecho.

    :::image type="content" source="media/protect-avs-web-apps-with-app-gw/app-gateway-multi-backend-pool-avs-02.png" alt-text="Agregue los grupos de back-end." lightbox="media/protect-avs-web-apps-with-app-gw/app-gateway-multi-backend-pool-avs-02.png":::

3. En la sección **Clientes de escucha**, cree un nuevo cliente de escucha para cada sitio web. Especifique los detalles de cada cliente de escucha y seleccione **Agregar**.

4. En el panel de navegación izquierdo, seleccione **Configuración HTTP** y seleccione **Agregar** en el panel izquierdo. Complete los detalles para crear una nueva configuración HTTP y seleccione **Guardar**.

    :::image type="content" source="media/protect-avs-web-apps-with-app-gw/app-gateway-multi-backend-pool-avs-03.png" alt-text="Complete los detalles para crear una configuración HTTP y seleccione Guardar" lightbox="media/protect-avs-web-apps-with-app-gw/app-gateway-multi-backend-pool-avs-03.png":::.

5. Cree las reglas en la sección **Reglas** del menú de la izquierda. Asocie cada regla con el cliente de escucha correspondiente. Seleccione **Agregar**.

6. Configure el grupo de back-end y la configuración HTTP correspondientes. Seleccione **Agregar**.

7. Pruebe la conexión. Abra el explorador que prefiera y vaya a los distintos sitios web hospedados en su entorno de Azure VMware Solution, como por ejemplo http://www.fabrikam.com.

    :::image type="content" source="media/protect-avs-web-apps-with-app-gw/app-gateway-multi-backend-pool-avs-07.png" alt-text="Pruebe la conexión":::.

### <a name="routing-by-url"></a>Enrutamiento por dirección URL

Puede usar Azure Application Gateway para configurar reglas de enrutamiento basadas en rutas de dirección URL. En este tutorial, definirá grupos de direcciones de back-end mediante máquinas virtuales que se ejecutan en una nube privada de Azure VMware Solution en una puerta de enlace de aplicación existente. La puerta de enlace de aplicación forma parte de una red virtual del centro, tal como se describe en la [documentación de integración nativa de Azure de Azure VMware Solution](concepts-avs-hub-and-spoke-integration.md). A continuación, creará reglas de enrutamiento que garanticen que el tráfico web llega a los servidores adecuados de los grupos.

1. Cree las máquinas virtuales. En la nube privada de Azure VMware Solution, cree un grupo de máquinas virtuales para representar la granja de servidores web. 

    :::image type="content" source="media/protect-avs-web-apps-with-app-gw/app-gateway-url-route-backend-pool-avs.png" alt-text="Cree un grupo de máquinas virtuales en Azure VMware Solution":::.

    Se ha usado Windows Server 2016 con el rol de IIS instalado para ilustrar este tutorial. Una vez instaladas las máquinas virtuales, ejecute los siguientes comandos de PowerShell para configurar IIS para el tutorial en cada una de ellas. 

    La primera máquina virtual, contoso-web-01, hospedará el sitio web principal.

    ```powershell
    Install-WindowsFeature -Name Web-Server
    Add-Content -Path C:\inetpub\wwwroot\Default.htm -Value $($env:computername)
    ```

    La segunda máquina virtual, contoso-web-02, hospedará el sitio de imágenes.
 
    ```powershell
    Install-WindowsFeature -Name Web-Server
    New-Item -Path "C:\inetpub\wwwroot\" -Name "images" -ItemType "directory"
    Add-Content -Path C:\inetpub\wwwroot\images\test.htm -Value $($env:computername)
    ```

    La tercera máquina virtual, contoso-web-03, hospedará el sitio de vídeos.

    ```powershell
    Install-WindowsFeature -Name Web-Server
    New-Item -Path "C:\inetpub\wwwroot\" -Name "video" -ItemType "directory"
    Add-Content -Path C:\inetpub\wwwroot\video\test.htm -Value $($env:computername)
    ```

2. Agregue los grupos de back-end. Tendrá que agregar tres nuevos grupos de back-end en una instancia de Application Gateway existente. Seleccione los **grupos back-end** en el menú de la izquierda. Seleccione **Agregar** y escriba los detalles del primer grupo, **contoso-web**. Agregue una máquina virtual como destino. Seleccione **Agregar**. Repita este proceso para **contoso-images** y **contoso-video**, agregando una máquina virtual única a cada uno como destino. 

    :::image type="content" source="media/protect-avs-web-apps-with-app-gw/app-gateway-url-route-backend-pool-avs-02.png" alt-text="Agregue tres grupos de back-end nuevos" lightbox="media/protect-avs-web-apps-with-app-gw/app-gateway-url-route-backend-pool-avs-02.png":::.

3. En la sección **Clientes de escucha**, cree un nuevo cliente de escucha de tipo Básico mediante el puerto 8080.

4. En el panel de navegación izquierdo, seleccione **Configuración HTTP** y seleccione **Agregar** en el panel izquierdo. Complete los detalles para crear una nueva configuración HTTP y seleccione **Guardar**.

    :::image type="content" source="media/protect-avs-web-apps-with-app-gw/app-gateway-url-route-backend-pool-avs-04.png" alt-text="Configuración HTTP":::.

5. Cree las reglas en la sección **Reglas** del menú de la izquierda. Asocie cada regla con el cliente de escucha creado anteriormente. A continuación, configure el grupo de back-end principal y la configuración HTTP. Seleccione **Agregar**.

    :::image type="content" source="media/protect-avs-web-apps-with-app-gw/app-gateway-url-route-backend-pool-avs-07.png" alt-text="Cree las reglas en la sección Reglas del menú de la izquierda":::.

6. Pruebe la configuración. Acceda a la puerta de enlace de aplicación en Azure Portal y, en la sección **Información general**, copie la dirección IP pública. A continuación, abra una nueva ventana del explorador y escriba la dirección URL `http://<app-gw-ip-address>:8080`. 

    :::image type="content" source="media/protect-avs-web-apps-with-app-gw/app-gateway-url-route-backend-pool-avs-08.png" alt-text="Pruebe la configuración en Azure Portal":::.

    Cambie la dirección URL a `http://<app-gw-ip-address>:8080/images/test.htm`.

    :::image type="content" source="media/protect-avs-web-apps-with-app-gw/app-gateway-url-route-backend-pool-avs-09.png" alt-text="Cambie la dirección URL":::.

    Vuelva a cambiar la dirección URL a `http://<app-gw-ip-address>:8080/video/test.htm`.

    :::image type="content" source="media/protect-avs-web-apps-with-app-gw/app-gateway-url-route-backend-pool-avs-10.png" alt-text="Vuelva a cambiar la dirección URL":::.

## <a name="next-steps"></a>Pasos siguientes

Revise la [documentación de Azure Application Gateway](https://docs.microsoft.com/azure/application-gateway/) para ver más ejemplos de configuración.

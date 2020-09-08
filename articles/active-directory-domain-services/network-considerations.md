---
title: Planeamiento y conexiones de red para Azure AD Domain Services | Microsoft Docs
description: Obtenga información sobre algunas consideraciones de diseño y recursos de redes virtuales que se usan para la conectividad al ejecutar Azure Active Directory Domain Services.
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: conceptual
ms.date: 07/06/2020
ms.author: iainfou
ms.openlocfilehash: ec38f16c5a658848eab505794ed1a2d072f22aea
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88749609"
---
# <a name="virtual-network-design-considerations-and-configuration-options-for-azure-active-directory-domain-services"></a>Consideraciones de diseño y opciones de configuración de red virtual para Azure Active Directory Domain Services

Azure Active Directory Domain Services (Azure AD DS) proporciona servicios de autenticación y administración a otras aplicaciones y cargas de trabajo. La conectividad de red es un componente clave. Sin una configuración correcta de los recursos de red virtual, las aplicaciones y las cargas de trabajo no se pueden comunicar con Azure AD DS ni usar las características que proporciona. Planee los requisitos de la red virtual para asegurarse de que Azure AD DS pueda atender las aplicaciones y las cargas de trabajo según sea necesario.

En este artículo se describen las consideraciones de diseño y los requisitos de las redes virtuales de Azure compatibles con Azure AD DS.

## <a name="azure-virtual-network-design"></a>Diseño de red virtual de Azure

Para proporcionar conectividad de red y permitir que las aplicaciones y los servicios se autentiquen en un dominio administrado de Azure AD DS, se usa una red virtual y una subred de Azure. Lo ideal es que el dominio administrado se implemente en su propia red virtual.

Puede incluir una subred de aplicación independiente en la misma red virtual para hospedar la máquina virtual de administración o las cargas de trabajo de aplicaciones ligeras. Para cargas de trabajo de aplicaciones más grandes o complejas, el diseño más adecuado suele ser una red virtual independiente emparejada con la red virtual de Azure AD DS.

También se admiten otras opciones de diseño, siempre y cuando se cumplan los requisitos descritos en las siguientes secciones para la red virtual y la subred.

Al diseñar la red virtual para Azure AD DS, debe tener en cuenta las siguientes consideraciones:

* Azure AD DS debe implementarse en la misma región de Azure que la red virtual.
    * En este momento, solo puede implementar un dominio administrado por inquilino de Azure AD. El dominio administrado se implementa en una sola región. Asegúrese de crear o seleccionar una red virtual en una [región que admita Azure AD DS](https://azure.microsoft.com/global-infrastructure/services/?products=active-directory-ds&regions=all).
* Tenga en cuenta la proximidad de otras regiones de Azure y de las redes virtuales que hospedan las cargas de trabajo de aplicaciones.
    * Para minimizar la latencia, mantenga las aplicaciones principales cerca o en la misma región que la subred de la red virtual para el dominio administrado. Puede usar el emparejamiento de redes virtuales o conexiones de red privada virtual (VPN) entre las redes virtuales de Azure. Estas opciones de conexión se describen en una sección subsiguiente.
* La red virtual no puede basarse en servicios DNS distintos de los proporcionados por el dominio administrado.
    * Azure AD DS proporciona su propio servicio DNS. La red virtual debe estar configurada para usar las direcciones de este servicio DNS. La resolución de nombres para los espacios de nombres adicionales puede realizarse mediante reenviadores condicionales.
    * No se puede usar una configuración personalizada del servidor DNS para dirigir consultas de otros servidores DNS, incluidos los que se encuentran en máquinas virtuales. Los recursos de la red virtual deben usar el servicio DNS proporcionado por el dominio administrado.

> [!IMPORTANT]
> No se puede mover Azure AD DS a otra red virtual después de haber habilitado el servicio.

Un dominio administrado se conecta a una subred de una red virtual de Azure. Para diseñar esta subred para Azure AD DS, tenga en cuenta las siguientes consideraciones:

* Un dominio administrado debe implementarse en su propia subred. No use una subred existente ni una subred de puerta de enlace.
* Durante la implementación de un dominio administrado se crea un grupo de seguridad de red. Dicho grupo de seguridad de red contiene las reglas necesarias para la correcta comunicación del servicio.
    * No cree ni use un grupo de seguridad de red existente con sus propias reglas personalizadas.
* Un dominio administrado requiere de 3 a 5 direcciones IP. Asegúrese de que el intervalo de direcciones IP de la subred puede proporcionar este número de direcciones.
    * Restringir las direcciones IP disponibles puede impedir que el dominio administrado mantenga dos controladores de dominio.

En el siguiente diagrama de ejemplo se describe un diseño válido en el que el dominio administrado tiene su propia subred, hay una subred de puerta de enlace para la conectividad externa y las cargas de trabajo de la aplicación se encuentran en una subred conectada dentro de la red virtual:

![Diseño de subred recomendado](./media/active-directory-domain-services-design-guide/vnet-subnet-design.png)

## <a name="connections-to-the-azure-ad-ds-virtual-network"></a>Conexiones a la red virtual de Azure AD DS

Como se indicó en la sección anterior, solo puede crear un dominio administrado en una única red virtual en Azure y solo se puede crear un dominio administrado por inquilino de Azure AD. En función de esta arquitectura, puede que necesite conectar una o varias redes virtuales que hospeden las cargas de trabajo de la aplicación a la red virtual del dominio administrado.

Puede conectar cargas de trabajo de aplicaciones hospedadas en otras redes virtuales de Azure mediante uno de los métodos siguientes:

* Emparejamiento de redes virtuales de Azure
* Red privada virtual (VPN)

### <a name="virtual-network-peering"></a>Emparejamiento de redes virtuales de Azure

el emparejamiento de red virtual es un mecanismo que conecta dos redes virtuales de la misma región mediante la red troncal de Azure. Con el emparejamiento global de redes virtuales, puede conectar redes virtuales entre regiones de Azure. Una vez emparejadas, las dos redes virtuales permiten que los recursos, como las máquinas virtuales, se comuniquen entre sí directamente mediante direcciones IP privadas. El uso del emparejamiento de redes virtuales permite implementar un dominio administrado con las cargas de trabajo de la aplicación implementadas en otras redes virtuales.

![Conectividad de Virtual Network mediante emparejamiento](./media/active-directory-domain-services-design-guide/vnet-peering.png)

Para obtener más información, consulte [Emparejamiento de redes virtuales de Azure](../virtual-network/virtual-network-peering-overview.md).

### <a name="virtual-private-networking-vpn"></a>Red privada virtual (VPN)

Puede conectar una red virtual a otra (de red virtual a red virtual) de la misma manera en que se configura una red virtual en la ubicación de un sitio local. Ambas conexiones usan una puerta de enlace de VPN para crear un túnel seguro con IPsec/IKE. Este modelo de conexión le permite implementar el dominio administrado en una red virtual de Azure y después, conectar ubicaciones locales u otras nubes.

![Conectividad de red virtual mediante VPN Gateway](./media/active-directory-domain-services-design-guide/vnet-connection-vpn-gateway.jpg)

Para obtener más información sobre el uso de redes privadas virtuales, consulte [Configuración de una conexión de VPN Gateway de red virtual a red virtual mediante Azure Portal](../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md).

## <a name="name-resolution-when-connecting-virtual-networks"></a>Resolución de nombres al conectar redes virtuales

Las redes virtuales conectadas a la red virtual del dominio administrado normalmente tienen su propia configuración de DNS. Al conectar las redes virtuales, no se configura automáticamente la resolución de nombres para la red virtual que se conecta para resolver los servicios que proporciona el dominio administrado. La resolución de nombres en las redes virtuales de conexión debe estar configurada para permitir que las cargas de trabajo de la aplicación ubiquen el dominio administrado.

Puede habilitar la resolución de nombres con reenviadores de DNS condicionales en el servidor DNS que admite las redes virtuales de conexión o con las mismas direcciones IP de DNS de la red virtual del dominio administrado.

## <a name="network-resources-used-by-azure-ad-ds"></a>Recursos de red usados por Azure AD DS

Un dominio administrado crea algunos recursos de red durante la implementación. Estos recursos son necesarios para una correcta operación y administración del dominio administrado y no deben configurarse manualmente.

| Recurso de Azure                          | Descripción |
|:----------------------------------------|:---|
| Tarjeta de interfaz de red                  | Azure AD DS hospeda el dominio administrado en dos controladores de dominio que se ejecutan en Windows Server como máquinas virtuales de Azure. Cada máquina virtual tiene una interfaz de red virtual que se conecta a la subred de la red virtual. |
| Dirección IP pública estándar dinámica      | Azure AD DS se comunica con el servicio de sincronización y administración mediante una dirección IP pública de SKU básica. Para obtener más información sobre las direcciones IP públicas, consulte [Tipos de direcciones IP y métodos de asignación en Azure](../virtual-network/public-ip-addresses.md). |
| Azure Standard Load Balancer            | Azure AD DS usa un equilibrador de carga de SKU básica para la traducción de direcciones de red (NAT) y el equilibrio de carga (cuando se usa con LDAP seguro). Para obtener más información sobre los equilibradores de carga de Azure, consulte [¿Qué es Azure Load Balancer?](../load-balancer/load-balancer-overview.md) |
| Reglas de traducción de direcciones de red (NAT) | Azure AD DS crea y usa tres reglas NAT en el equilibrador de carga: una para el tráfico HTTP seguro y dos para proteger la comunicación remota de PowerShell. |
| Reglas de equilibrador de carga                     | Cuando un dominio administrado está configurado para LDAP seguro en el puerto TCP 636, se crean tres reglas y se usan en un equilibrador de carga para distribuir el tráfico. |

> [!WARNING]
> No elimine ni modifique ninguno de los recursos de red creados por Azure AD DS, como la configuración manual del equilibrador de carga o las reglas. Si elimina o modifica alguno de los recursos de red, es posible que se produzca una interrupción del servicio de Azure AD DS.

## <a name="network-security-groups-and-required-ports"></a>Grupos de seguridad de red y puertos necesarios

Un [grupo de seguridad de red (NSG)](../virtual-network/security-overview.md) contiene una lista de reglas que permiten o deniegan el tráfico de red al tráfico de una red virtual de Azure. Cuando se implementa un dominio administrado que contiene un conjunto de reglas que permiten que el servicio proporcione funciones de autenticación y administración, se crea un grupo de seguridad de red. Este grupo de seguridad de red predeterminado está asociado a la subred de la red virtual en la que se implementa el dominio administrado.

Las siguientes reglas de grupo de seguridad de red son necesarias para que el dominio administrado proporcione servicios de autenticación y administración. No edite ni elimine estas reglas del grupo de seguridad de red para la subred de la red virtual en la que está implementado el dominio administrado.

| Número de puerto | Protocolo | Source                             | Destination | Acción | Obligatorio | Propósito |
|:-----------:|:--------:|:----------------------------------:|:-----------:|:------:|:--------:|:--------|
| 443         | TCP      | AzureActiveDirectoryDomainServices | Any         | Allow  | Sí      | Sincronización con el inquilino de Azure AD. |
| 3389        | TCP      | CorpNetSaw                         | Any         | Allow  | Sí      | Administración del dominio. |
| 5986        | TCP      | AzureActiveDirectoryDomainServices | Any         | Allow  | Sí      | Administración del dominio. |

Se crea un equilibrador de carga estándar de Azure que exige la presencia de estas reglas. Este grupo de seguridad de red protege Azure AD DS y es necesario para que el dominio administrado funcione correctamente. No elimine este grupo de seguridad de red. El equilibrador de carga no funcionará correctamente sin él.

> [!WARNING]
> No edite manualmente estos recursos y configuraciones de red. Cuando asocia un grupo de seguridad de red mal configurado o una tabla de rutas definida por el usuario con la subred en la que está implementado el dominio administrado, puede interrumpir la capacidad de Microsoft para atender y administrar el dominio. También se interrumpirá la sincronización entre el inquilino de Azure AD y el dominio administrado.
>
> Si usa LDAP seguro, puede agregar la regla de puerto TCP 636 necesaria para permitir el tráfico externo, si es necesario. Al agregar esta regla no se colocan las reglas del grupo de seguridad de red en un estado no admitido. Para obtener más información, consulte [Bloqueo del acceso LDAP seguro a través de Internet](tutorial-configure-ldaps.md#lock-down-secure-ldap-access-over-the-internet).
>
> También existen reglas predeterminadas para *AllowVnetInBound*, *AllowAzureLoadBalancerInBound*, *DenyAllInBound*, *AllowVnetOutBound*, *AllowInternetOutBound* y *DenyAllOutBound* para el grupo de seguridad de red. No modifique ni elimine estas reglas predeterminadas.
>
> El Acuerdo de Nivel de Servicio de Azure no se aplica a las implementaciones en las que se han aplicado grupos de seguridad de red mal configurados o tablas de rutas definidas por el usuario que impiden que Azure AD DS actualice y administre el dominio.

### <a name="port-443---synchronization-with-azure-ad"></a>Puerto 443: sincronización con Azure AD

* Se usa para sincronizar el inquilino de Azure AD con el dominio administrado.
* Sin acceso a este puerto, el dominio administrado no se puede sincronizar con el inquilino de Azure AD. Es posible que los usuarios no puedan iniciar sesión, ya que los cambios en sus contraseñas no se sincronizarán con el dominio administrado.
* El acceso de entrada a este puerto para direcciones IP está restringido de forma predeterminada mediante la etiqueta de servicio **AzureActiveDirectoryDomainServices**.
* No restrinja el acceso de salida desde este puerto.

### <a name="port-3389---management-using-remote-desktop"></a>Puerto 3389: administración mediante Escritorio remoto

* Se usa para las conexiones de escritorio remoto a los controladores de dominio del dominio administrado.
* La regla del grupo de seguridad de red predeterminado usa la etiqueta de servicio *CorpNetSaw* para restringir aún más el tráfico.
    * Esta etiqueta de servicio solo permite que las estaciones de trabajo de acceso seguro de la red corporativa de Microsoft usen el escritorio remoto para el dominio administrado.
    * Solo se permite el acceso con justificación comercial, por ejemplo, para escenarios de administración o solución de problemas.
* Esta regla se puede establecer en *Denegar*, y en *Permitir* solo cuando sea necesario. La mayoría de las tareas de administración y supervisión se realizan mediante la comunicación remota de PowerShell. El Protocolo de escritorio remoto (RDP) se usa únicamente en el caso excepcional de que Microsoft necesite conectarse de forma remota al dominio administrado para llevar a cabo estrategias avanzadas para solucionar problemas.

> [!NOTE]
> No puede seleccionar manualmente la etiqueta de servicio *CorpNetSaw* desde el portal si intenta editar esta regla del grupo de seguridad de red. Debe usar Azure PowerShell o la CLI de Azure para configurar manualmente una regla que use la etiqueta de servicio *CorpNetSaw*.
>
> Por ejemplo, puede usar el script siguiente para crear una regla que permita RDP: 
>
> `Get-AzureRmNetworkSecurityGroup -Name "nsg-name" -ResourceGroupName "resource-group-name" | Add-AzureRmNetworkSecurityRuleConfig -Name "new-rule-name" -Access "Allow" -Protocol "TCP" -Direction "Inbound" -Priority "priority-number" -SourceAddressPrefix "CorpNetSaw" -SourcePortRange "" -DestinationPortRange "3389" -DestinationAddressPrefix "" | Set-AzureRmNetworkSecurityGroup`

### <a name="port-5986---management-using-powershell-remoting"></a>Puerto 5986: administración mediante la comunicación remota de PowerShell

* Se usa para realizar tareas de administración mediante la comunicación remota de PowerShell en el dominio administrado.
* Sin acceso a este puerto, el dominio administrado no se puede actualizar, configurar, hacer copia de seguridad ni supervisar.
* En el caso de los dominios administrados que usan una red virtual basada en Resource Manager, puede restringir el acceso de entrada a este puerto a la etiqueta de servicio *AzureActiveDirectoryDomainServices*.
    * En el caso de los dominios administrados heredados con una red virtual basada en el modelo clásico, puede restringir el acceso de entrada a este puerto a las direcciones IP de origen siguientes: *52.180.183.8*, *23.101.0.70*, *52.225.184.198*, *52.179.126.223*, *13.74.249.156*, *52.187.117.83*, *52.161.13.95*, *104.40.156.18* y *104.40.87.209*.

    > [!NOTE]
    > En 2017, Azure AD Domain Services se comenzó a poder hospedar en una red de Azure Resource Manager. Desde entonces, hemos podido crear un servicio más seguro gracias a las modernas funcionalidades de Azure Resource Manager. Como las implementaciones de Azure Resource Manager están reemplazando totalmente a las implementaciones clásicas, las implementaciones de las redes virtuales clásicas de Azure AD DS se retirarán el 1 de marzo de 2023.
    >
    > Para obtener más información, vea el [aviso de desuso oficial](https://azure.microsoft.com/updates/we-are-retiring-azure-ad-domain-services-classic-vnet-support-on-march-1-2023/).

## <a name="user-defined-routes"></a>Rutas definidas por el usuario

Las rutas definidas por el usuario no se crean de forma predeterminada y no son necesarias para que Azure AD DS funcione correctamente. Si necesita usar tablas de rutas, evite realizar cambios en la ruta *0.0.0.0*. Los cambios en esta ruta interrumpen Azure AD DS y colocan el dominio administrado en un estado no admitido.

También debe enrutar el tráfico entrante desde las direcciones IP incluidas en las etiquetas de servicio de Azure respectivas a la subred del dominio administrado. Para obtener más información sobre las etiquetas de servicio y la dirección IP asociada, consulte [Intervalos de direcciones IP de Azure y etiquetas de servicio: nube pública](https://www.microsoft.com/en-us/download/details.aspx?id=56519).

> [!CAUTION]
> Estos intervalos de direcciones IP del centro de datos de Azure pueden cambiar sin previo aviso. Asegúrese de que tiene procesos para validar que dispone de las direcciones IP más recientes.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre algunos de los recursos de red y las opciones de conexión que usa Azure AD DS, consulte los siguientes artículos:

* [Emparejamiento de Azure Virtual Network](../virtual-network/virtual-network-peering-overview.md)
* [Puertas de enlace de VPN de Azure](../vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md)
* [Grupos de seguridad de red de Azure](../virtual-network/security-overview.md)
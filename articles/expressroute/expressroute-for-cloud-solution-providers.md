---
title: 'ExpressRoute para proveedores de soluciones en la nube: Azure | Microsoft Docs'
description: Este artículo proporciona información para los proveedores de soluciones en la nube que deseen incorporar servicios de Azure y ExpressRoute en su oferta de servicios.
services: expressroute
author: richcar
ms.service: expressroute
ms.topic: article
ms.date: 10/10/2016
ms.author: ricarlse
ms.custom: seodec18
ms.openlocfilehash: 99b51610e41aaf8358e7e3069d38dfd8c68ae422
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87446780"
---
# <a name="expressroute-for-cloud-solution-providers-csp"></a>ExpressRoute para proveedores de soluciones en la nube (CSP)
Microsoft proporciona servicios de hiperescala para que los revendedores y distribuidores tradicionales (CSP) puedan aprovisionar rápidamente nuevos servicios y soluciones para sus clientes, sin que sea preciso invertir en el desarrollo de dichos servicios nuevos. Para que el proveedor de soluciones en la nube (CSP) disponga de la capacidad para administrar directamente estos nuevos servicios, Microsoft proporciona programas y API que permiten que el CSP administre recursos de Microsoft Azure en nombre de sus clientes. Uno de esos recursos es ExpressRoute. ExpressRoute permite que el CSP conecte recursos existentes de clientes a servicios de Azure. ExpressRoute es un vínculo privado de comunicaciones a gran velocidad con los servicios de Azure. 

ExpressRoute está formado por un par de circuitos de alta disponibilidad que se conectan a la suscripción o las suscripciones de un único cliente y no se pueden compartir entre varios. Cada circuito debe finalizar en un enrutador diferente para mantener la alta disponibilidad.

> [!NOTE]
> Existen límites de ancho de banda y de conexión en ExpressRoute, lo que significa que las implementaciones grandes o complejas requerirán varios circuitos ExpressRoute para un único cliente.
> 
> 

Microsoft Azure proporciona un número creciente de servicios que puede ofrecer a sus clientes. Para poder aprovechar mejor estos servicios, ExpressRoute proporciona acceso de alta velocidad y baja latencia al entorno de Microsoft Azure.

## <a name="microsoft-azure-management"></a>Administración de Microsoft Azure
Microsoft proporciona a los CSP las API para administrar las suscripciones de clientes de Azure y permite la integración mediante programación con los sistemas de administración de servicios propios del proveedor. Las funcionalidades de administración compatibles se explican [aquí](https://msdn.microsoft.com/library/partnercenter/dn974944.aspx).

## <a name="microsoft-azure-resource-management"></a>Administración de recursos de Microsoft Azure
La forma en que se administre la suscripción dependerá del contrato que haya firmado con su cliente. El CSP puede administrar directamente la creación y el mantenimiento de los recursos o el cliente puede mantener el control sobre la suscripción de Microsoft Azure y crear los recursos de Azure cuando los necesite. Si el cliente administra la creación de recursos en su suscripción de Microsoft Azure, usará uno de estos modelos: "*Conexión a través*" o "*Conexión directa*". Estos modelos se describen con más detalle en las secciones siguientes.  

### <a name="connect-through-model"></a>Modelo Conexión a través
![texto alternativo](./media/expressroute-for-cloud-solution-providers/connect-through.png)  

En el modelo Conexión a través, el CSP crea una conexión directa entre el centro de datos y suscripción de Azure del cliente. La conexión directa se realiza mediante ExpressRoute, entre Azure y su red, a la que después se conecta el cliente. En este escenario, es necesario que el cliente pase a través de la red del CSP para acceder a los servicios de Azure. 

Si su cliente tiene otras suscripciones de Azure que administra de forma independiente, usará la red pública de Internet o su propia conexión privada para conectarse a esos servicios aprovisionados en la suscripción que no es del CSP. 

En el caso de los CSP que administran servicios de Azure, se da por supuesto que el CSP ya tenía establecido un almacén de identidades para clientes que después se replica en Azure Active Directory para administrar su suscripción de CSP a través de la modalidad de administración en nombre de (AOBO). Entre los impulsores clave para este escenario, se incluyen las situaciones en que un asociado o proveedor de servicios disfruta de una relación establecida con el cliente, el cliente ya está consumiendo servicios de un proveedor o el asociado desea proporcionar una combinación de soluciones hospedadas por el proveedor y por Azure para ofrecer flexibilidad y solucionar retos del cliente que el CSP no puede afrontar en solitario. Este modelo se muestra en la **figura**siguiente.

![texto alternativo](./media/expressroute-for-cloud-solution-providers/connect-through-model.png)

### <a name="connect-to-model"></a>Modelo Conexión directa
![texto alternativo](./media/expressroute-for-cloud-solution-providers/connect-to.png)

En el modelo Conexión directa, el proveedor de servicios crea una conexión directa entre el centro de datos de su cliente y la suscripción de Azure aprovisionada por CSP mediante ExpressRoute a través de la red del cliente (cliente).

> [!NOTE]
> Para ExpressRoute, el cliente necesita crear y mantener el circuito ExpressRoute.  
> 
> 

Este escenario de conectividad requiere que el cliente se conecte directamente a través de una red de cliente para acceder a la suscripción a Azure administrada por CSP, mediante una conexión de red directa que el cliente crea, posee y administra total o parcialmente. Para estos clientes se asume que el proveedor no tiene establecido actualmente un almacén de identidades del cliente y ayudaría al cliente a replicar su almacén de identidades actual en Azure Active Directory para la administración de su suscripción a través de AOBO. Entre los impulsores clave para este escenario, se incluyen las situaciones en que un asociado o proveedor de servicios dado disfruta de una relación establecida con el cliente, el cliente ya está consumiendo servicios de un proveedor o el asociado desea proporcionar servicios basados únicamente en soluciones hospedadas por Azure sin necesidad de que el proveedor contribuya una infraestructura o un centro de datos.

![texto alternativo](./media/expressroute-for-cloud-solution-providers/connect-to-model.png)

La elección entre ambas opciones se basa en los requisitos del cliente y la necesidad del proveedor de prestar servicios de Azure. Los detalles de estos modelos y los modelos de diseño asociados para el control de acceso basado en rol, las redes y las identidades se explican en los siguientes vínculos:

* **Control de acceso basado en rol de Azure (Azure RBAC)** : RBAC se basa en Azure Active Directory.  Para más información sobre RBAC en Azure, consulte [este artículo](../role-based-access-control/role-assignments-portal.md).
* **Redes** : se tratan los distintos temas de las redes en Microsoft Azure.
* **Azure Active Directory (Azure AD)** : Azure AD proporciona la administración de identidades para Microsoft Azure y aplicaciones SaaS de terceros. Para más información sobre Azure AD, consulte [este artículo](https://azure.microsoft.com/documentation/services/active-directory/).  

## <a name="network-speeds"></a>Velocidades de red
ExpressRoute admite velocidades de red entre 50 Mb/s y 10 Gb/s. Esto permite que los clientes compren la cantidad de ancho de banda de red que necesiten para su entorno particular.

> [!NOTE]
> Se puede aumentar el ancho de banda de red según sea necesario sin interrumpir las comunicaciones, pero para reducir la velocidad de red se debe eliminar el circuito y volver a crearlo con la velocidad de red menor.  
> 
> 

ExpressRoute admite la conexión de varias redes virtuales a un solo circuito ExpressRoute para hacer mejor uso de las conexiones de alta velocidad. Un único circuito ExpressRoute se puede compartir entre varias suscripciones de Azure que sean propiedad del mismo cliente.

## <a name="configuring-expressroute"></a>Configuración de ExpressRoute
ExpressRoute se puede configurar para admitir tres tipos de tráfico ([dominios de enrutamiento](#expressroute-routing-domains)) en un único circuito ExpressRoute. Este tráfico se divide entre emparejamiento de Microsoft, pares públicos de Azure y pares privados. Puede elegir que uno de los tipos de tráfico o todos ellos se envíen a través de un solo circuito ExpressRoute, o bien usar varios circuitos ExpressRoute según el tamaño del circuito ExpressRoute y el aislamiento que requiera el cliente. Es posible que la posición del cliente en cuanto a la seguridad no permita que el tráfico público y el privado recorran el mismo circuito.

### <a name="connect-through-model"></a>Modelo Conexión a través
En una configuración de conexión a través, como proveedor, será responsable de todos los elementos fundamentales de la red necesarios para conectar los recursos del centro de datos del cliente a las suscripciones hospedadas en Azure. Cada uno de los clientes que vaya a usar las funcionalidades de Azure contará con su propia conexión ExpressRoute, de la que usted será el administrador. Usará los mismos métodos que utilizaría el cliente para conseguir el circuito ExpressRoute. Seguirá los mismos pasos que se describen en el artículo [Flujos de trabajo de ExpressRoute](expressroute-workflows.md) para aprovisionar el circuito y los estados de circuitos. Después, configurará las rutas de Border Gateway Protocol (BGP) para controlar el tráfico que fluye entre la red local y la red virtual de Azure.

### <a name="connect-to-model"></a>Modelo Conexión directa
En una configuración de conexión directa, el cliente ya posee una conexión con Azure o iniciará una conexión al proveedor de acceso a Internet que vincula ExpressRoute desde el centro de datos propiedad del cliente directamente con Azure, y no desde el centro de datos del proveedor. Para comenzar el proceso de aprovisionamiento, el cliente seguirá los pasos descritos antes para el modelo Conexión a través. Una vez establecido el circuito, su cliente tendrá que configurar los enrutadores locales para poder acceder tanto a la red del proveedor como a las redes virtuales de Azure.

Puede ayudar a establecer la conexión y configurar las rutas para permitir que los recursos en su centro o centros de datos propios se comuniquen con los recursos del cliente también en su centro de datos, o con los recursos hospedados en Azure.

## <a name="expressroute-routing-domains"></a>Dominios de enrutamiento de ExpressRoute
ExpressRoute ofrece tres dominios de enrutamiento: público, privado y emparejamiento de Microsoft. Todos los dominios de enrutamiento utilizan los mismos enrutadores con una configuración en activo-activo de alta disponibilidad. Para ver más detalles sobre los dominios de enrutamiento de ExpressRoute, consulte [este artículo](expressroute-circuit-peerings.md).

Puede definir filtros de rutas personalizados para permitir solamente las rutas que desee o necesite. Para más información o para saber cómo hacer estos cambios, consulte el artículo sobre la [creación y modificación del enrutamiento de un circuito ExpressRoute mediante PowerShell](expressroute-howto-routing-classic.md) para más detalles sobre los filtros de enrutamiento.

> [!NOTE]
> Para la conectividad con emparejamiento de Microsoft y pares públicos, se debe utilizar una dirección IP pública que sea propiedad del cliente o el CSP, y se deben cumplir todas las reglas definidas. Para más información, consulte la página [Requisitos previos de ExpressRoute](expressroute-prerequisites.md) .  
> 
> 

## <a name="routing"></a>Enrutamiento
ExpressRoute se conecta a las redes de Azure a través de la puerta de enlace de Azure Virtual Network. Las puertas de enlace de red proporcionan enrutamiento para las redes virtuales de Azure.

Cuando se crean instancias de Azure Virtual Network, también se crea una tabla de enrutamiento predeterminado para la red virtual, que se usará para dirigir el tráfico desde las subredes de la red virtual y hacia ellas. Si la tabla de enrutamiento predeterminada no es suficiente para la solución, se pueden crear rutas personalizadas para enrutar el tráfico saliente hacia dispositivos personalizados o bloquear rutas hacia subredes específicas o redes externas.

### <a name="default-routing"></a>Enrutamiento predeterminado
La tabla de enrutamiento predeterminada incluye las siguientes rutas:

* Enrutamiento dentro de una subred
* Entre subredes dentro de la red virtual
* Hacia Internet
* Entre redes virtuales mediante un puerta de enlace de VPN
* Entre una red virtual y una red local mediante una puerta de enlace de VPN o ExpressRoute

![texto alternativo](./media/expressroute-for-cloud-solution-providers/default-routing.png)  

### <a name="user-defined-routing-udr"></a>Enrutamiento definido por el usuario (UDR)
Las rutas definidas por el usuario permiten controlar el tráfico saliente de la subred asignada a otras subredes de la red virtual o a través de una de las otras puertas de enlace predefinidas (ExpressRoute; Internet o VPN). La tabla de enrutamiento predeterminado del sistema se puede reemplazar por una definida por el usuario que contiene rutas personalizadas. Con el enrutamiento definido por el usuario, los clientes pueden crear rutas específicas hacia dispositivos como firewalls o dispositivos de detección de intrusiones, o bien bloquear el acceso a subredes específicas desde la subred que hospeda la ruta definida por el usuario. Para ver información general sobre las rutas definidas por el usuario, lea [este artículo](../virtual-network/virtual-networks-udr-overview.md). 

## <a name="security"></a>Seguridad
En función del modelo que se use (Conexión directa o Conexión a través), su cliente define las directivas de seguridad en su red virtual o proporciona los requisitos de directiva de seguridad al CSP para que los defina en sus redes virtuales. Se pueden definir los siguientes criterios de seguridad:

1. **Aislamiento de clientes**: la plataforma Azure proporciona el aislamiento de los clientes mediante el almacenamiento del Id. de cliente y la información de la red virtual en una base de datos segura, que se utiliza para encapsular el tráfico de cada cliente en un túnel GRE.
2. **grupo de seguridad de red (NSG)** sirven para definir el tráfico que se permite que entre y salga de las subredes dentro de las redes virtuales en Azure. De forma predeterminada, el NSG contiene reglas de bloqueo para bloquear el tráfico procedente de Internet hacia la red virtual y reglas de permiso para el tráfico dentro de una red virtual. Para más información sobre los grupos de seguridad de red, consulte [este artículo](https://azure.microsoft.com/blog/network-security-groups/).
3. **Túnel forzado** : se trata de una opción para redirigir el tráfico que se origina en Azure y va destinado a Internet a través de la conexión ExpressRoute con el centro de datos local. Para más información acerca de Túnel forzado, consulte [este artículo](expressroute-routing.md#advertising-default-routes).  
4. **Cifrado** : aunque los circuitos ExpressRoute están dedicados a un cliente específico, es posible que el proveedor de red sufra una infracción de seguridad, lo que permitiría que un intruso examinara el tráfico de paquetes. Para evitar esta posible situación, un cliente o un CSP puede cifrar el tráfico que atraviesa la conexión si define directivas de modo de túnel IPSec para todo el tráfico que fluye entre los recursos locales y los recursos de Azure (consulte el modo de túnel IPSec opcional para el cliente 1 en la ilustración 5: sobre la seguridad de ExpressRoute, más arriba). La segunda opción es usar un dispositivo de firewall en cada punto de conexión del circuito ExpressRoute. Para esto, será necesario instalar más dispositivos o máquinas virtuales de firewall de terceros en ambos puntos de conexión para cifrar el tráfico que atraviesa el circuito ExpressRoute.

![texto alternativo](./media/expressroute-for-cloud-solution-providers/expressroute-security.png)  

## <a name="next-steps"></a>Pasos siguientes
El servicio Proveedor de soluciones en la nube proporciona una manera de aumentar el valor que ofrece a sus clientes sin necesidad de costosas inversiones en infraestructura y capacidad, al mismo tiempo que mantiene su puesto como proveedor principal de subcontratación. La integración sin problemas con Microsoft Azure puede realizarse mediante la API de CSP, lo que le permite integrar la administración de Microsoft Azure en los marcos de administración de los que ya dispone.  

Puede encontrar más información en los vínculos siguientes:

[Azure en el programa Proveedor de soluciones en la nube](https://docs.microsoft.com/azure/cloud-solution-provider).  
[Preparación para realizar transacciones como Proveedor de soluciones en la nube](https://partner.microsoft.com/solutions/cloud-reseller-pre-launch).  
[Recursos de Proveedor de soluciones en la nube de Microsoft](https://partner.microsoft.com/solutions/cloud-reseller-resources).

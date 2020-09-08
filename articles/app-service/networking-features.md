---
title: Características de red
description: Obtenga información sobre las características de red de Azure App Service y las características que necesita para sus necesidades de red para la seguridad o la funcionalidad.
author: ccompy
ms.assetid: 5c61eed1-1ad1-4191-9f71-906d610ee5b7
ms.topic: article
ms.date: 03/16/2020
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: d2b74af723e3ba8b1d71e9f481bf96d009540a52
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962101"
---
# <a name="app-service-networking-features"></a>Características de redes de App Service

Las aplicaciones de Azure App Service se pueden implementar de varias maneras. De forma predeterminada, las aplicaciones hospedadas en App Service son directamente accesibles desde Internet y solo pueden alcanzar puntos de conexión hospedados en Internet. Sin embargo, muchas aplicaciones de cliente necesitan controlar el tráfico de red entrante y saliente. En App Service hay varias características disponibles para satisfacer esas necesidades. El desafío consiste en saber qué característica se debe usar para resolver un problema determinado. Este documento está pensado para ayudar a los clientes a determinar qué característica se debe utilizar según algunos casos de uso de ejemplo.

Hay dos tipos de implementación principales para Azure App Service. Está el servicio público multiinquilino, que hospeda planes de App Service en las SKU de precios Gratis, Compartido, Básico, Estándar, Premium y Premium v2. Y también está App Service Environment (ASE) de un único inquilino, que hospeda planes de App Service de SKU aislada directamente en su red virtual (VNet) de Azure. Las características que puede usar variarán en función de si está en el servicio multiinquilino o en un entorno de ASE. 

## <a name="multi-tenant-app-service-networking-features"></a>Características de redes de App Service multiinquilino 

Azure App Service es un sistema distribuido. Los roles que controlan las solicitudes HTTP/HTTPS entrantes se denominan front-ends. Los roles que hospedan la carga de trabajo del cliente se llaman roles de trabajo. Todos los roles de una implementación de App Service existen en una red de varios inquilinos. Puesto que hay muchos clientes diferentes en la misma unidad de escalado de App Service, no se puede conectar la red de App Service directamente a su red. En lugar de conectarse a redes, se necesitan características para administrar los diferentes aspectos de la comunicación de aplicaciones. Las características que controlan las solicitudes A la aplicación no se pueden usar para solucionar problemas al realizar llamadas DESDE la aplicación. Del mismo modo, las características que resuelven problemas para las llamadas DESDE la aplicación no se pueden usar para solucionar problemas de llamadas A la aplicación.  

| Características de entrada | Características de salida |
|---------------------|-------------------|
| Direcciones asignadas a las aplicaciones | conexiones híbridas |
| Restricciones de acceso | Integración con red virtual con requisito de puerta de enlace |
| Puntos de conexión de servicio | Integración con red virtual |

A menos que se indique lo contrario, todas las características se pueden usar conjuntamente. Puede combinar las características para solucionar diversos problemas.

## <a name="use-case-and-features"></a>Caso de uso y características

Para un caso de uso determinado, puede haber varias maneras de solucionar el problema.  La característica adecuada a utilizar a veces viene dada por razones más allá del propio caso de uso. Los siguientes casos de uso de entrada sugieren cómo usar las características de redes de App Service para solucionar problemas acerca de cómo controlar el tráfico que va a la aplicación. 
 
| Casos de uso de entrada | Característica |
|---------------------|-------------------|
| Compatibilidad con las necesidades de SSL basado en IP de la aplicación | direcciones asignadas a las aplicaciones |
| Dirección entrante dedicada y no compartida para la aplicación | direcciones asignadas a las aplicaciones |
| Restricción del acceso a la aplicación desde un conjunto de direcciones bien definidas | Restricciones de acceso |
| Restricción del acceso a la aplicación desde los recursos de una red virtual | Puntos de conexión de servicio </br> ASE de ILB </br> Punto de conexión privado (versión preliminar) |
| Exponer la aplicación en una dirección IP privada de mi red virtual | ASE de ILB </br> dirección IP privada para la entrada en una instancia de Application Gateway con puntos de conexión de servicio </br> Punto de conexión de servicio (versión preliminar) |
| Proteger la aplicación con un WAF | Application Gateway + ASE de ILB </br> Application Gateway con puntos de conexión de servicio </br> Azure Front Door con restricciones de acceso |
| Equilibrar la carga del tráfico a las aplicaciones en diferentes regiones | Azure Front Door con restricciones de acceso | 
| Equilibrar la carga del tráfico en la misma región | [Application Gateway con puntos de conexión de servicio][appgwserviceendpoints] | 

Los siguientes casos de uso de salida sugieren cómo usar las características de redes de App Service para resolver las necesidades de acceso de salida de la aplicación. 

| Casos de uso de salida | Característica |
|---------------------|-------------------|
| Acceso a los recursos de una red virtual de Azure en la misma región | Integración con red virtual </br> ASE |
| Acceso a los recursos de una red virtual de Azure en una región diferente | Integración con red virtual con requisito de puerta de enlace </br> ASE y emparejamiento de redes virtuales |
| Acceso a los recursos protegidos por puntos de conexión de servicio | Integración con red virtual </br> ASE |
| Acceso a los recursos de una red privada no conectada a Azure | conexiones híbridas |
| Acceso a los recursos mediante circuitos de ExpressRoute | Integración con red virtual </br> ASE | 
| Protección del tráfico saliente de su aplicación web | Integración de VNet y grupos de seguridad de red </br> ASE | 
| Enrutamiento del tráfico saliente de su aplicación web | Integración de VNet y tablas de ruta </br> ASE | 


### <a name="default-networking-behavior"></a>Comportamiento predeterminado de la redes

Las unidades de escalado de Azure App Service admiten muchos clientes en cada implementación. Los planes de SKU Gratis y Compartido hospedan cargas de trabajo de clientes en roles de trabajo multiinquilino. El plan Básico y los planes superiores hospedan cargas de trabajo que se dedican a un único plan de App Service (ASP). Si tuviera un plan de App Service Estándar, todas las aplicaciones de ese plan se ejecutarán en el mismo rol de trabajo. Si escala horizontalmente el rol de trabajo, todas las aplicaciones de ese ASP se replicarán en un rol de trabajo nuevo para cada instancia del ASP. Los roles de trabajo que se usan para planes Premium v2 son diferentes de los roles de trabajo que se usan para los otros planes. Cada implementación de App Service tiene una dirección IP que se usa para todo el tráfico entrante a las aplicaciones de esa implementación de App Service. Sin embargo, hay de 4 a 11 direcciones utilizadas para realizar llamadas salientes. Estas direcciones se comparten para todas las aplicaciones de esa implementación de App Service. Las direcciones de salida son diferentes en función de los diferentes tipos de roles de trabajo. Esto significa que las direcciones usadas por los ASP Gratis, Compartido, Básico, Estándar y Premium son diferentes de las direcciones usadas para las llamadas salientes desde los ASP Premium v2. Si observa en las propiedades de la aplicación, podrá ver las direcciones de entrada y de salida utilizadas por la aplicación. Si necesita bloquear una dependencia con una ACL de IP, utilice las direcciones de salida posibles. 

![Propiedades de la aplicación](media/networking-features/app-properties.png)

App Service tiene varios puntos de conexión que se usan para administrar el servicio.  Dichas direcciones se publican en un documento independiente y también se encuentran en la etiqueta de servicio de direcciones IP AppServiceManagement. La etiqueta AppServiceManagement solo se usa con un entorno de App Service Environment (ASE) en el que se debe permitir este tráfico. En la etiqueta de servicio IP de App Service se realiza un seguimiento de las direcciones de entrada de App Service. No hay ninguna etiqueta de servicio IP que contiene las direcciones de salida usadas por App Service. 

![Diagrama de entrada y salida de App Service](media/networking-features/default-behavior.png)

### <a name="app-assigned-address"></a>Direcciones asignadas a las aplicaciones 

La característica de direcciones asignadas a las aplicaciones es una rama de la funcionalidad SSL basada en IP y se accede mediante la configuración de SSL en la aplicación. Esta característica se puede usar para las llamadas SSL basadas en IP, pero también se puede utilizar para proporcionar una dirección única para su aplicación. 

![Diagrama de direcciones asignadas a las aplicaciones](media/networking-features/app-assigned-address.png)

Cuando se utiliza una dirección asignada a la aplicación, el tráfico sigue pasando por los mismos roles de front-end que controlan todo el tráfico entrante en la unidad de escalado de App Service. La dirección que se asigna a la aplicación, sin embargo, solo es utilizada por la aplicación. Los casos de uso para esta característica son:

* Compatibilidad con las necesidades de SSL basado en IP de la aplicación
* Establecer una dirección dedicada para la aplicación que no se comparta con nada más

Puede obtener información sobre cómo establecer una dirección en la aplicación con el tutorial [Adición de un certificado TLS/SSL en Azure App Service][appassignedaddress]. 

### <a name="access-restrictions"></a>Restricciones de acceso 

La funcionalidad Restricciones de acceso le permite filtrar las solicitudes **entrantes** en función de la dirección IP de origen. La acción de filtrado tiene lugar en los roles de front-end, que están en un nivel superior al de los roles de trabajo en los que se ejecutan las aplicaciones. Puesto que los roles de front-end están en un nivel superior al de los roles de trabajo, la funcionalidad Restricciones de acceso se puede considerar como una protección de nivel de red para las aplicaciones. La característica le permite crear una lista de bloques de direcciones permitidas y denegadas que se evalúan en orden de prioridad. Es similar a la característica Grupo de seguridad de red (NSG) que existe en las redes de Azure.  Puede usar esta característica en un entorno de ASE o en el servicio multiinquilino. Cuando se usa con un ASE de ILB, puede restringir el acceso desde bloques de direcciones privadas.

![Restricciones de acceso](media/networking-features/access-restrictions.png)

La característica Restricciones de acceso es de utilidad en escenarios donde se desea restringir las direcciones IP que se pueden usar para llegar a la aplicación. Algunos casos de uso para esta característica son:

* Restricción del acceso a la aplicación desde un conjunto de direcciones bien definidas 
* Restricción del acceso a las solicitudes que proceden de un servicio de equilibrio de carga, como Azure Front Door. Si desea bloquear el tráfico entrante a Azure Front Door, cree reglas para permitir el tráfico del intervalo entre 147.243.0.0/16 y 2a01:111:2050::/44. 

![Restricciones de acceso con Front Door](media/networking-features/access-restrictions-afd.png)

Si desea bloquear el acceso a la aplicación para que solo sea accesible desde los recursos de su red virtual de Azure (VNet), necesita una dirección pública estática en aquel que sea el elemento de origen desde dicha red virtual. Si los recursos no tienen una dirección pública, debe usar la característica Puntos de conexión de servicio en su lugar. Obtenga información sobre cómo habilitar esta característica con el tutorial [Configuración de Restricciones de acceso][iprestrictions].

### <a name="service-endpoints"></a>Puntos de conexión del servicio

Los puntos de conexión de servicio le permite bloquear el acceso **entrante** a la aplicación de modo que la dirección de origen deba proceder de un conjunto de subredes que ha seleccionado. Esta característica funciona junto con la característica Restricciones de acceso IP. Los puntos de conexión de servicio se establecen en la misma experiencia de usuario que las restricciones de acceso IP. Puede crear una lista de reglas de acceso tipo permitir/denegar que incluya direcciones públicas, así como las subredes de sus redes virtuales. Esta característica admite escenarios como:

![puntos de conexión del servicio](media/networking-features/service-endpoints.png)

* Configuración de Application Gateway con la aplicación para bloquear el tráfico entrante a la aplicación
* Restricción del acceso a la aplicación a los recursos de su red virtual. Estos pueden ser máquinas virtuales, entornos de ASE o incluso otras aplicaciones que usen la integración con red virtual. 

![puntos de conexión de servicio con Application Gateway](media/networking-features/service-endpoints-appgw.png)

Más información sobre cómo configurar los puntos de conexión de servicio con la aplicación en el tutorial [Configuración de restricciones de acceso del punto de conexión de servicio][serviceendpoints].

### <a name="private-endpoint-preview"></a>Punto de conexión privado (versión preliminar)

Un punto de conexión privado es una interfaz de red que le conecta de forma privada y segura a la aplicación web con Azure Private Link. El punto de conexión privado usa una dirección IP privada de la red virtual y así coloca el la aplicación web de manera eficaz en la red virtual. Esta característica es solo para flujos **entrantes** en la aplicación web.
[Uso de puntos de conexión privados para una aplicación web de Azure (versión preliminar)][privateendpoints]
 
### <a name="hybrid-connections"></a>conexiones híbridas

Conexiones híbridas de App Service permite que las aplicaciones realicen llamadas **salientes** a puntos de conexión TCP especificados. El punto de conexión puede ser local, de una red virtual o estar en cualquier lugar que permita el tráfico saliente a Azure en el puerto 443. Esta característica requiere la instalación de un agente de retransmisión llamado Administrador de conexiones híbridas (HCM) en un host Windows Server 2012 o posterior. El Administrador de conexiones híbridas debe ser capaz de conectarse a Azure Relay en el puerto 443. El Administrador de conexiones híbridas se puede descargar desde la interfaz de usuario de Conexiones híbridas de App Service en el portal. 

![Flujo de red de Conexiones híbridas](media/networking-features/hybrid-connections.png)

La característica Conexiones híbridas de App Service se basa en la funcionalidad Conexiones híbridas de Azure Relay. App Service usa una forma especializada de la característica que solo admite la realización de llamadas salientes desde la aplicación en un host y puerto TCP. La resolución de este host y puerto solo es necesaria en el host en el que está instalado HCM. Cuando la aplicación, en App Service, realiza una búsqueda DNS sobre el host y puerto definidos en la Conexión híbrida, el tráfico se redirige automáticamente a través de la Conexión híbrida y fuera del Administrador de conexiones híbridas. Para más información sobre Conexiones híbridas, consulte la documentación en [Conexiones híbridas de App Service][hybridconn].

Esta característica se utiliza normalmente para:

* Acceder a recursos en redes privadas que no están conectadas a Azure con una VPN o ExpressRoute
* Admitir la migración tipo levantar y mover de aplicaciones locales a App Service sin tener que mover también las bases de datos necesarias  
* Proporcionar acceso seguro a un único host y puerto por cada Conexión híbrida. La mayoría de las características de redes abren el acceso a una red y con Conexiones híbridas solo tiene un único host y puerto al que contactar.
* Tratar escenarios no cubiertos por otros métodos de conectividad de salida
* Realizar desarrollos en App Service en los que las aplicaciones pueden aprovechar fácilmente los recursos locales 

Dado que la característica permite el acceso a los recursos locales sin un agujero en el firewall de entrada, es popular entre los desarrolladores. Las otras características de redes de App Service de salida están muy relacionadas con las redes virtuales de Azure. Conexiones híbridas no tiene la dependencia de pasar por una red virtual y se puede usar para una amplia variedad de necesidades de redes. Es importante tener en cuenta que la característica Conexiones híbridas de App Service no tiene conocimiento de lo que se hace sobre ella. Es decir, que se puede usar para acceder a una base de datos, un servicio web o un socket TCP arbitrario en un sistema central. La característica esencialmente tuneliza paquetes TCP. 

Aunque Conexiones híbridas es popular para el desarrollo, también se utiliza en numerosas aplicaciones de producción. Es muy útil para acceder a un servicio web o una base de datos, pero no es adecuada para situaciones que implican crear muchas conexiones. 

### <a name="gateway-required-vnet-integration"></a>Integración con red virtual con requisito de puerta de enlace 

La característica Integración con redes virtuales de App Service requerida por la puerta de enlace permite a la aplicación realizar solicitudes **salientes** a una red virtual de Azure. La característica funciona mediante la conexión del host en el que se ejecuta la aplicación a una puerta de enlace de red virtual de la red virtual con una conexión VPN de punto a sitio. Al configurar la característica, la aplicación obtiene una de las direcciones de punto a sitio asignadas a cada instancia. Esta característica le permite acceder a recursos de redes virtuales clásicas o de Resource Manager en cualquier región. 

![Integración con red virtual con requisito de puerta de enlace](media/networking-features/gw-vnet-integration.png)

Esta característica soluciona el problema de acceso a recursos existentes en otras redes virtuales e incluso se puede usar para conectarse a través de una red virtual a otras redes virtuales o incluso al entorno local. No funciona con redes virtuales conectadas mediante ExpressRoute, pero funciona con redes conectadas mediante VPN. Normalmente es inadecuado usar esta característica desde una aplicación en un entorno de App Service Environment (ASE), porque la instancia de ASE ya está en su red virtual. Los casos de uso que soluciona esta característica son:

* Acceso a los recursos en direcciones IP privadas de sus redes virtuales de Azure 
* Acceso a los recursos locales si hay una conexión VPN de sitio a sitio 
* Acceso a los recursos en redes virtuales emparejadas 

Cuando esta característica está habilitada, la aplicación usará el servidor DNS configurado en la red virtual de destino. Puede leer más sobre esta característica en la documentación [Integración con red virtual de App Service][vnetintegrationp2s]. 

### <a name="vnet-integration"></a>Integración con red virtual

La característica Integración con redes virtuales requerida por la puerta de enlace es muy útil, pero no resuelve el acceso a los recursos a través de ExpressRoute. Más allá de la necesidad de conexiones a través de ExpressRoute, las aplicaciones necesitan poder realizar llamadas a servicios protegidos por puntos de conexión de servicio. Para solucionar ambas necesidades adicionales, se ha agregado otra funcionalidad de Integración con red virtual. La nueva característica Integración con red virtual le permite colocar el back-end de la aplicación en una subred de una red virtual de Resource Manager de la misma región. Esta característica no está disponible desde un entorno de App Service Environment, que ya está en una red virtual. Esta característica permite:

* Acceso a recursos en redes virtuales de Resource Manager de la misma región
* Acceso a los recursos que están protegidos por puntos de conexión de servicio 
* Acceso a recursos que son accesibles a través de conexiones ExpressRoute o VPN
* Protección de todo el tráfico saliente 
* Fuerce la tunelización de todo el tráfico saliente. 

![Integración con red virtual](media/networking-features/vnet-integration.png)

Para más información sobre esta característica, lea los documentos sobre [Integración con red virtual de App Service][vnetintegration].

## <a name="app-service-environment"></a>Entorno de App Service 

Un entorno de App Service Environment (ASE) es una implementación de inquilino único de Azure App Service que se ejecuta en su red virtual. ASE permite casos de uso como:

* Acceder a los recursos de la red virtual
* Acceder a los recursos a través de ExpressRoute
* Exponer sus aplicaciones con una dirección privada en la red virtual 
* Acceder a los recursos a través de puntos de conexión de servicio 

Con un entorno de ASE, no es necesario utilizar características como la integración con redes virtuales o los puntos de conexión de servicio porque la instancia de ASE ya está en la red virtual. Si desea tener acceso a recursos como SQL o Azure Storage sobre puntos de conexión de servicio, habilite puntos de conexión de servicio en la subred de ASE. Si desea acceder a recursos de la red virtual, no es necesaria ninguna configuración adicional.  Si desea acceder a recursos a través de ExpressRoute, ya está en la red virtual y no es necesario configurar nada en el entorno de ASE o en las aplicaciones dentro de él. 

Dado que las aplicaciones de un ASE de ILB pueden ser expuestas en una dirección IP privada, puede agregar fácilmente dispositivos WAF para exponer solo las aplicaciones que desee a Internet y proteger el resto. Se presta para el desarrollo sencillo de aplicaciones de varios niveles. 

Hay algunas cosas que no son posibles desde el servicio multiinquilino y lo son desde un entorno de ASE. Cosas como por ejemplo:

* Exponer las aplicaciones en una dirección IP privada
* Proteger todo el tráfico saliente con controles de red que no forman parte de la aplicación 
* Hospedar sus aplicaciones en un servicio de inquilino único 
* Escalar verticalmente a muchas más instancias de las posibles en el servicio multiinquilino 
* Cargar certificados de cliente de entidad de certificación privada para su uso por las aplicaciones con puntos de conexión protegidos por entidad de certificación privada 
* Forzar TLS 1.1 en todas las aplicaciones hospedadas en el sistema sin ninguna posibilidad de deshabilitarlo en el nivel de aplicación 
* Proporcionar una dirección de salida dedicada para todas las aplicaciones del entorno de ASE que no se comparte con otros clientes 

![ASE en una red virtual](media/networking-features/app-service-environment.png)

El entorno de ASE proporciona la mejor opción para el hospedaje de una aplicación aislada y dedicada, pero conlleva algunos desafíos de administración. Algunos aspectos a tener en cuenta antes de usar una instancia de ASE operativa son:
 
 * Una instancia de ASE se ejecuta dentro de su red virtual, pero tiene dependencias fuera de la red virtual. Se deben permitir esas dependencias. Más información en [Consideraciones de red para un entorno de App Service Environment][networkinfo].
 * Una instancia de ASE no se escala inmediatamente al igual que en el servicio multiinquilino. Es necesario anticipar las necesidades de escalado en lugar de escalar de forma reactiva. 
 * Una instancia de ASE tiene un mayor costo por adelantado asociado. Para obtener el máximo partido de su entorno de ASE, debe planear poner muchas cargas de trabajo en un ASE en lugar de utilizarlo para pequeños esfuerzos.
 * Las aplicaciones de un entorno de ASE no pueden restringir el acceso a algunas aplicaciones del ASE y a otras no.
 * El entorno de ASE está en una subred y las reglas de red se aplican a todo el tráfico hacia y desde dicho ASE. Si desea asignar reglas de tráfico entrante para una sola aplicación, utilice Restricciones de acceso. 

## <a name="combining-features"></a>Combinación de características 

Las características indicadas para el servicio multiinquilino se pueden usar conjuntamente para resolver casos de uso más elaborados. Aquí se describen dos de los casos de uso más comunes, pero son solo ejemplos. Al comprender lo que hacen las distintas características, puede resolver casi todas las necesidades de arquitectura del sistema.

### <a name="inject-app-into-a-vnet"></a>Inserción de la aplicación en una red virtual

Cómo poner la aplicación en una red virtual es una solicitud común. Poner la aplicación en una red virtual significa que los puntos de conexión de entrada y de salida de una aplicación están en una red virtual. El entorno de ASE proporciona la mejor solución para este problema, pero puede satisfacer sus necesidades con el servicio multiinquilino mediante la combinación de características. Por ejemplo, puede hospedar aplicaciones solo de intranet con direcciones privadas de entrada y de salida mediante:

* Crear una puerta de enlace de aplicaciones con direcciones de entrada y de salida privadas
* Proteger el tráfico entrante a la aplicación con puntos de conexión de servicio 
* Usar la nueva Integración con red virtual para que el back-end de la aplicación esté en la red virtual 

Este estilo de implementación no le proporcionará una dirección dedicada para el tráfico saliente a Internet ni le dará la posibilidad de bloquear todo el tráfico saliente de la aplicación.  Este estilo de implementación le aportara mucho de lo que de otro modo solo obtendría con un entorno de ASE. 

### <a name="create-multi-tier-applications"></a>Creación de aplicaciones de varios niveles

Una aplicación de varios niveles es una aplicación en la que solo se puede acceder a las aplicaciones de back-end de la API desde el nivel de front-end. Para crear una aplicación de varios niveles, puede:

* Usar la Integración con red virtual para conectar el back-end de la aplicación web de front-end con una subred de una red virtual
* Usar puntos de conexión de servicio para proteger el tráfico entrante a la aplicación de API para que solo proceda de la subred usada por la aplicación web de front-end

![aplicación de varios niveles](media/networking-features/multi-tier-app.png)

Puede hacer que varias aplicaciones de front-end usen la misma aplicación de API mediante la Integración con red virtual de las otras aplicaciones de front-end y puntos de conexión de servicio de la aplicación de API con sus subredes.  

<!--Links-->
[appassignedaddress]: ./configure-ssl-certificate.md
[iprestrictions]: ./app-service-ip-restrictions.md
[serviceendpoints]: ./app-service-ip-restrictions.md
[hybridconn]: ./app-service-hybrid-connections.md
[vnetintegrationp2s]: ./web-sites-integrate-with-vnet.md
[vnetintegration]: ./web-sites-integrate-with-vnet.md
[networkinfo]: ./environment/network-info.md
[appgwserviceendpoints]: ./networking/app-gateway-with-service-endpoints.md
[privateendpoints]: ./networking/private-endpoint.md
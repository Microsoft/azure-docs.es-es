---
title: Funcionamiento de Azure Traffic Manager | Microsoft Docs
description: Este artículo lo ayudará a comprender cómo Traffic Manager enruta el tráfico para mejorar el rendimiento y la disponibilidad de las aplicaciones web.
services: traffic-manager
documentationcenter: ''
author: duongau
manager: twooley
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/05/2019
ms.author: duau
ms.openlocfilehash: 6566d70413ec287a58f5d02764d6ec34f9cdce15
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89400677"
---
# <a name="how-traffic-manager-works"></a>Funcionamiento de Traffic Manager

Azure Traffic Manager permite controlar la distribución del tráfico a través de los puntos de conexión de la aplicación. Un punto de conexión es cualquier servicio accesible desde Internet hospedado dentro o fuera de Azure.

Traffic Manager ofrece dos ventajas principales:

- La distribución del tráfico de acuerdo con uno o varios [métodos de enrutamiento de tráfico](traffic-manager-routing-methods.md)
- [supervisión continua del estado de los puntos de conexión](traffic-manager-monitoring.md) y la conmutación por error automática en caso de error en estos

Cuando un cliente intenta conectarse a un servicio, debe resolver primero el nombre DNS del servicio en una dirección IP. Luego, el cliente se conecta a esa dirección IP para tener acceso al servicio.

**Es fundamental entender que Traffic Manager funciona a nivel de DNS.**  Traffic Manager usa DNS para dirigir a los clientes a puntos de conexión específicos del servicio basados en las reglas del método de enrutamiento de tráfico. Los clientes se conectan **directamente**al punto de conexión seleccionado. Traffic Manager no es un proxy ni una puerta de enlace. Traffic Manager no ve el tráfico que circula entre el cliente y el servicio.

## <a name="traffic-manager-example"></a>Ejemplo de Traffic Manager

Contoso Corp ha desarrollado un nuevo portal para asociados. La URL de este portal es `https://partners.contoso.com/login.aspx`. La aplicación está hospedada en tres regiones de Azure. Para mejorar la disponibilidad y maximizar el rendimiento global, usan Traffic Manager para distribuir el tráfico de cliente al punto de conexión disponible más próximo.

Para lograr esta configuración, se han completado los pasos siguientes:

1. Se implementan tres instancias de su servicio. Los nombres DNS de estas implementaciones son "contoso-us.cloudapp.net", "contoso-eu.cloudapp.net" y "contoso-asia.cloudapp.net".
1. Se crea un perfil de Traffic Manager denominado "contoso.trafficmanager.net" y este se configura para utilizar el método de enrutamiento de tráfico "Rendimiento" entre los tres puntos de conexión.
1. Se configura un nombre de dominio personal, "partners.contoso.com", para que apunte a "contoso.trafficmanager.net" con un registro CNAME de DNS.

![Configuración de DNS de Traffic Manager][1]

> [!NOTE]
> Cuando se utiliza un dominio personal con Azure Traffic Manager, debe usar un registro CNAME para que el nombre de dominio personalizado apunte a su nombre de dominio de Traffic Manager. Los estándares DNS no permiten crear un registro CNAME en el "vértice" (o raíz) de un dominio. Por lo tanto no se puede crear un registro CNAME para "contoso.com" (lo que también se conoce como un dominio "desnudo"). Solo se puede crear un registro CNAME para un dominio bajo "contoso.com", como "www.contoso.com". Para solucionar esta limitación, se recomienda hospedar el dominio DNS en [Azure DNS](../dns/dns-overview.md) y el uso de [registros de Alias](../dns/tutorial-alias-tm.md) para apuntar a su perfil del administrador de tráfico. También puede usar una sencilla redirección HTTP para dirigir las solicitudes de "contoso.com" a un nombre alternativo como "www.contoso.com".

### <a name="how-clients-connect-using-traffic-manager"></a>Conexión de clientes mediante Traffic Manager

Continuando a partir del ejemplo anterior, cuando un cliente solicita la página `https://partners.contoso.com/login.aspx`, este realiza los pasos siguientes para resolver el nombre DNS y establecer una conexión:

![Establecimiento de la conexión mediante Traffic Manager][2]

1. El cliente envía una consulta de DNS a su servicio DNS recursivo configurado para resolver el nombre "partners.contoso.com". Un servicio DNS recursivo, a veces denominado servicio "DNS local", no hospeda los dominios DNS directamente. En su lugar, el cliente descarga el trabajo de ponerse en contacto con los diversos servicios DNS autoritativos a través de Internet, lo cual es un paso necesario para resolver un nombre DNS.
2. Para resolver el nombre DNS, el servicio DNS recursivo busca entre los servidores de nombres el dominio "contoso.com". A continuación, se pone en contacto con esos servidores DNS a fin de solicitar el registro "partners.contoso.com". Los servidores DNS de contoso.com devuelven el registro CNAME que apunta a contoso.trafficmanager.net.
3. A continuación, el servicio DNS recursivo busca los servidores DNS para el dominio trafficmanager.net, que se proporcionan con el servicio Azure Traffic Manager. A continuación, envía una solicitud para el registro DNS de "contoso.trafficmanager.net" a esos servidores DNS.
4. Los servidores DNS de Traffic Manager reciben la solicitud. Eligen un punto de conexión en función de:

    - El estado configurado de cada punto de conexión (no se devuelven los puntos de conexión deshabilitados).
    - El estado actual de cada punto de conexión, según lo determinado por las comprobaciones de estado de Traffic Manager. Para más información, consulte [Acerca de la supervisión de Traffic Manager](traffic-manager-monitoring.md).
    - El método de enrutamiento de tráfico elegido. Para más información, consulte [Métodos de enrutamiento de Traffic Manager](traffic-manager-routing-methods.md).

5. El punto de conexión elegido se devuelve como otro registro CNAME de DNS. En este caso, supongamos que contoso-us.cloudapp.net se devuelve.
6. A continuación, el servicio DNS recursivo busca los servidores DNS para el dominio "cloudapp.net". Se pone en contacto con estos servidores DNS para solicitar el registro DNS "contoso-us.cloudapp.net". Se devuelve un registro "A" de DNS que contiene la dirección IP del punto de conexión de servicio basado en Estados Unidos.
7. El servicio DNS recursivo consolida los resultados y devuelve una única respuesta DNS al cliente.
8. El cliente recibe los resultados DNS y se conecta a la dirección IP especificada. El cliente se conecta al punto de conexión de servicio de la aplicación directamente, no mediante Traffic Manager. Puesto que es un punto de conexión HTTPS, el cliente lleva a cabo el protocolo de enlace SSL/TLS necesario y, a continuación, realiza una solicitud GET de HTTP para la página "/login.aspx".

El servicio DNS recursivo almacena en la memoria caché las respuestas DNS que recibe. La resolución DNS del dispositivo cliente también almacena en caché el resultado. El almacenamiento en caché permite que las consultas DNS posteriores se respondan más rápidamente al utilizar datos de la memoria caché en lugar de consultar otros servidores DNS. La duración de la memoria caché viene determinada por la propiedad de período de vida (TTL) de cada registro DNS. Unos valores más cortos producen una expiración de caché más rápida y, por tanto, más recorridos de ida y vuelta a los servidores de nombres de Traffic Manager. Unos valores mayores significan que se puede tardar más tiempo en alejar el tráfico de un punto de conexión con error. Traffic Manager permite configurar el valor de TTL utilizado en las respuestas DNS de Traffic Manager entre 0 segundos y 2.147.483.647 segundos (el intervalo máximo compatible con [RFC-1035](https://www.ietf.org/rfc/rfc1035.txt)), lo que le permite elegir el valor que responda mejor a las necesidades de su aplicación.

## <a name="faqs"></a>Preguntas más frecuentes

* [¿Qué dirección IP usa Traffic Manager?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#what-ip-address-does-traffic-manager-use)

* [¿Qué tipos de tráfico se pueden enrutar mediante Traffic Manager?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#what-types-of-traffic-can-be-routed-using-traffic-manager)

* [¿Admite Traffic Manager sesiones temporales?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#does-traffic-manager-support-sticky-sessions)

* [¿Por qué obtengo un error HTTP al utilizar Traffic Manager?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#why-am-i-seeing-an-http-error-when-using-traffic-manager)

* [¿Cómo afecta al rendimiento el uso de Traffic Manager?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#what-is-the-performance-impact-of-using-traffic-manager)

* [¿Qué protocolos de aplicación puedo usar con Traffic Manager?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#what-application-protocols-can-i-use-with-traffic-manager)

* [¿Puedo usar Traffic Manager con un nombre de dominio desnudo?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#can-i-use-traffic-manager-with-a-naked-domain-name)

* [¿Traffic Manager considera la dirección de subred de cliente cuando controla las consultas de DNS?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#does-traffic-manager-consider-the-client-subnet-address-when-handling-dns-queries)

* [¿Qué es el TTL de DNS y cómo afecta a mis usuarios?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#what-is-dns-ttl-and-how-does-it-impact-my-users)

* [¿Cuál es el límite superior e inferior en que puedo establecer el TTL para las respuestas de Traffic Manager?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-high-or-low-can-i-set-the-ttl-for-traffic-manager-responses)

* [¿Cómo puedo comprender el volumen de las consultas que llegan a mi perfil?](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs#how-can-i-understand-the-volume-of-queries-coming-to-my-profile)

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre la [supervisión del punto de conexión y la conmutación por error automática](traffic-manager-monitoring.md) de Traffic Manager.

Obtenga más información sobre los [métodos de enrutamiento del tráfico](traffic-manager-routing-methods.md) de Traffic Manager.

<!--Image references-->
[1]: ./media/traffic-manager-how-traffic-manager-works/dns-configuration.png
[2]: ./media/traffic-manager-how-traffic-manager-works/flow.png


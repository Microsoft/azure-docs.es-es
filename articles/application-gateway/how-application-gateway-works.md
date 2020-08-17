---
title: Funcionamiento de una puerta de enlace de aplicaciones
description: En este artículo se proporciona información sobre cómo una puerta de enlace de aplicaciones para acepta las solicitudes entrantes y las enruta al back-end.
services: application-gateway
author: abshamsft
ms.service: application-gateway
ms.topic: conceptual
ms.date: 11/16/2019
ms.author: absha
ms.openlocfilehash: d33ec829e490ae45d38d33f5784126a71ae2d0aa
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86506560"
---
# <a name="how-an-application-gateway-works"></a>Funcionamiento de una puerta de enlace de aplicaciones

En este artículo se explica el proceso que usa una puerta de enlace de aplicaciones para aceptar las solicitudes entrantes y enrutarlas al back-end.

![Cómo una puerta de enlace de aplicaciones acepta una solicitud](./media/how-application-gateway-works/how-application-gateway-works.png)

## <a name="how-an-application-gateway-accepts-a-request"></a>Cómo una puerta de enlace de aplicaciones acepta una solicitud

1. Antes de que un cliente envíe una solicitud a una puerta de enlace de aplicaciones, resuelve el nombre de dominio de la puerta de enlace de aplicaciones mediante un servidor del sistema de nombres de dominio (DNS). Azure controla la entrada de DNS, ya que todas las puertas de enlace de aplicaciones están en el dominio de azure.com.

2. Azure DNS devuelve la dirección IP al cliente, que es la dirección IP de front-end de la puerta de enlace de aplicaciones.

3. La puerta de enlace de aplicaciones acepta el tráfico entrante en uno o varios clientes de escucha. Un cliente de escucha es una entidad lógica que comprueba si hay solicitudes de conexión. Se configura con una dirección IP de front-end, un protocolo y un número de puerto para las conexiones de los clientes a la puerta de enlace de aplicaciones.

4. Si el firewall de aplicaciones web (WAF) está en uso, la puerta de enlace de aplicaciones comprueba los encabezados y el cuerpo de la solicitud, si están presentes, según las reglas de WAF. Esta acción determina si la solicitud es una solicitud válida o una amenaza de seguridad. Si la solicitud es válida, se enruta al back-end. Si la solicitud no es válida y WAF se encuentra en modo de prevención, se bloquea como una amenaza de seguridad. Si está en modo de detección, la solicitud se evalúa y se registra, pero se reenvía al servidor back-end.

Azure Application Gateway puede usarse como un equilibrador de carga interno de la aplicación o como un equilibrador de carga de aplicación orientado a Internet. Una puerta de enlace de aplicaciones orientada a Internet usa direcciones IP públicas. El nombre DNS de una puerta de enlace de aplicaciones orientada Internet puede resolverse públicamente en su dirección IP pública. Como resultado, las puertas de enlace de aplicaciones orientadas a Internet pueden enrutar las solicitudes de cliente a Internet.

Las puertas de enlace de aplicaciones internas usan solo las direcciones IP privadas. Si usa una [zona DNS privada](https://docs.microsoft.com/azure/dns/private-dns-overview) o personalizada, el nombre de dominio debe poder resolverse internamente en la dirección IP privada de la instancia de Application Gateway. Por lo tanto, los equilibradores de carga internos solo pueden enrutar las solicitudes de clientes con acceso a una red virtual para la puerta de enlace de aplicaciones.

## <a name="how-an-application-gateway-routes-a-request"></a>Cómo una puerta de enlace de aplicaciones enruta una solicitud

Si una solicitud es válida o no está bloqueada por WAF, la puerta de enlace de aplicaciones evalúa la regla de enrutamiento de solicitud que está asociada con el cliente de escucha. Esta acción determina a qué grupo de back-end se va a enrutar la solicitud.

Según la regla de enrutamiento de solicitud, la puerta de enlace de aplicaciones determina si debe enrutar todas las solicitudes en el cliente de escucha a un grupo de back-end específico, enrutar las solicitudes a grupos de back-end diferentes en función de la ruta de acceso URL, o redirigir las solicitudes al otro puerto o sitio externos.
>[!NOTE]
>Las reglas se procesan en el orden en que aparecen en el portal para SKU v1. 

Cuando la puerta de enlace de aplicaciones selecciona el grupo de back-end, envía la solicitud a uno de los servidores back-end correctos en el grupo (y.y.y.y). Un sondeo de estado determina el estado del servidor. Si el grupo de back-end contiene varios servidores, la puerta de enlace de aplicaciones utiliza un algoritmo round robin para enrutar las solicitudes entre los servidores en buen estado. Esta carga equilibra las solicitudes en los servidores.

Después de que la puerta de enlace de aplicaciones determina el servidor back-end, abre una nueva sesión TCP con el servidor back-end según la configuración HTTP. La configuración HTTP especifica el protocolo, el puerto y otras configuraciones relacionadas con el enrutamiento que son necesarias para establecer una nueva sesión con el servidor back-end.

El puerto y el protocolo usados en la configuración HTTP determinan si el tráfico entre los servidores back-end y la puerta de enlace de aplicaciones está cifrado (para lograr TLS de un extremo a otro) o sin cifrar.

Cuando una puerta de enlace de la aplicación envía la solicitud original al servidor back-end, respeta las opciones personalizadas en la configuración HTTP relacionadas con reemplazar el nombre de host, la ruta de acceso y el protocolo. Esta acción mantiene la afinidad de sesión basada en cookies, la purga de la conexión y la selección de nombre de host desde el back-end, entre otros.

 >[!NOTE]
>Si el grupo de back-end:
> - **Es un punto de conexión público**, la puerta de enlace de aplicaciones usa su dirección IP pública de front-end para tener acceso al servidor. Si no hay una dirección IP pública de front-end, se asigna una a la conectividad saliente externa.
> - **Contiene un FQDN que se puede resolver internamente o una dirección IP privada**, la puerta de enlace de aplicaciones enruta la solicitud al servidor back-end mediante las direcciones IP privadas de su instancia.
> - **Contiene un punto de conexión externo o un FQDN que puede resolverse externamente**, la puerta de enlace de aplicaciones enruta la solicitud al servidor back-end mediante la dirección IP pública de su front-end. La resolución de DNS se basa en una zona DNS privada o un servidor DNS personalizado (si está configurado), o bien utiliza el valor predeterminado proporcionado por Azure DNS. Si no hay una dirección IP pública de front-end, se asigna una a la conectividad saliente externa.

### <a name="modifications-to-the-request"></a>Modificaciones a una solicitud

Una puerta de enlace de aplicaciones inserta cuatro encabezados adicionales en todas las solicitudes antes de reenviarlas al back-end. Dichos encabezados son x-forwarded-for, x-forwarded-proto, x-forwarded-port y x-original-host. El formato del encabezado x-forwarded-for es una lista separada por comas de valores IP:puerto.

Los valores válidos para x-forwarded-proto son HTTP o HTTPS. X-forwarded-port especifica el puerto al que llegó la solicitud en la puerta de enlace de aplicaciones. El encabezado x-original-host contiene el encabezado de host original con el que llegó la solicitud. Este encabezado es útil en la integración de Azure Website, donde el encabezado de host entrante se modifica antes de que el tráfico se enrute al back-end. Si la opción de afinidad de sesión está habilitada, se agrega una cookie de afinidad administrada por la puerta de enlace.

Puede configurar la puerta de enlace de aplicaciones para que modifique los encabezados y URL de solicitud y respuesta mediante la [rescritura de encabezados HTTP y URL](rewrite-http-headers-url.md) o para que modifique la ruta de acceso URI mediante una configuración de invalidación de la ruta de acceso. Sin embargo, a menos que realice la configuración, todas las solicitudes entrantes se procesan con proxy hacia el back-end.

## <a name="next-steps"></a>Pasos siguientes

[Información sobre los componentes de Application Gateway](application-gateway-components.md)

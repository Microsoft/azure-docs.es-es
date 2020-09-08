---
title: 'Configuración de una red virtual: nivel premium de Azure Cache for Redis'
description: Obtenga información sobre cómo crear y administrar la compatibilidad de red virtual con instancias de Azure Cache for Redis de nivel Premium
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.custom: devx-track-csharp
ms.topic: conceptual
ms.date: 05/15/2017
ms.openlocfilehash: 82003ef84571c8e07982826124b33763c0e53194
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88205553"
---
# <a name="how-to-configure-virtual-network-support-for-a-premium-azure-cache-for-redis"></a>Configuración de la compatibilidad de red virtual con el nivel Premium de Azure Cache for Redis
Azure Cache for Redis cuenta con diferentes opciones de caché, lo que proporciona flexibilidad en la elección del tamaño y las características de la memoria caché, incluidas algunas características del nivel Prémium, como la agrupación en clústeres, la persistencia y la compatibilidad con las redes virtuales. Una red virtual es una red privada en la nube. Cuando una instancia de Azure Cache for Redis se configure con una red virtual, no será posible acceder a ella públicamente, solo se podrá acceder a ella desde máquinas virtuales y aplicaciones de dentro de la red virtual. En este artículo se describe cómo configurar la compatibilidad con redes virtuales de una instancia de Azure Cache for Redis de nivel Premium.

> [!NOTE]
> Azure Cache for Redis admite redes virtuales clásicas y de Resource Manager.
> 
> 

## <a name="why-vnet"></a>¿Por qué una red virtual?
La implementación de [Azure Virtual Network](https://azure.microsoft.com/services/virtual-network/) aporta más seguridad y aislamiento de su instancia de Azure Cache for Redis, así como subredes, directivas de control de acceso y otras características para restringir aún más el acceso.

## <a name="virtual-network-support"></a>Compatibilidad con redes virtuales
La compatibilidad con Virtual Network se configura en la hoja **New Azure Cache for Redis** (Nueva instancia de Azure Cache for Redis) durante la creación de la memoria caché. 

[!INCLUDE [redis-cache-create](../../includes/redis-cache-premium-create.md)]

Una vez que haya seleccionado un plan de tarifa premium, la integración con una red virtual de Redis se puede configurar seleccionando una red virtual que esté en la misma suscripción y ubicación que la memoria caché. Para usar una nueva red virtual, primero es preciso crearla, para lo que es preciso seguir los pasos de [Creación de una red virtual mediante Azure Portal](../virtual-network/manage-virtual-network.md#create-a-virtual-network) o [Creación de Virtual Network (clásica) mediante Azure Portal](../virtual-network/virtual-networks-create-vnet-classic-pportal.md) y, luego, volver a la hoja **New Azure Cache for Redis** (Nueva instancia de Azure Cache for Redis) para crear y configurar una memoria caché Premium.

Para configurar la red virtual de la memoria caché nueva, haga clic en **Virtual Network** en la hoja **New Azure Cache for Redis** (Nueva instancia de Azure Cache for Redis) y seleccione la red virtual deseada en la lista desplegable.

![Virtual network][redis-cache-vnet]

En la lista desplegable **Subred**, seleccione la subred deseada.  Si lo desea, especifique una **dirección IP estática**. El campo **Dirección IP estática** es opcional y, si no se especifica ninguna, se elige una de la subred seleccionada.

> [!IMPORTANT]
> Si se implementa una instancia de Azure Cache for Redis en una red virtual de Resource Manager, la caché debe estar en una subred dedicada que no contenga otros recursos, salvo instancias de Azure Cache for Redis. Si se intenta implementar una instancia de Azure Cache for Redis en una red virtual de Resource Manager en una subred que contiene otros recursos, se produce un error en la implementación.
> 
> 

![Virtual network][redis-cache-vnet-ip]

> [!IMPORTANT]
> Azure reserva algunas direcciones IP dentro de cada subred y estas direcciones no se pueden usar. La primera y la última dirección IP de las subredes están reservadas para la conformidad con el protocolo, junto con otras tres direcciones usadas para los servicios de Azure. Para más información, consulte [¿Hay alguna restricción en el uso de direcciones IP dentro de estas subredes?](../virtual-network/virtual-networks-faq.md#are-there-any-restrictions-on-using-ip-addresses-within-these-subnets)
> 
> Además de las direcciones IP que usa la infraestructura de Azure Virtual Network, cada instancia de Redis de la subred usa dos direcciones IP por partición y una dirección IP adicional para el equilibrador de carga. Se considera que una caché no en clúster tiene una partición.
> 
> 

Una vez que se crea la memoria caché, para ver la configuración de la red virtual es preciso hacer clic en **Virtual Network** en el **menú Recursos**.

![Virtual network][redis-cache-vnet-info]

Para conectarse a la instancia de Azure Cache for Redis cuando usa una red virtual, especifique el nombre de host de la memoria caché en la cadena de conexión, como se muestra en el ejemplo siguiente:

```csharp
private static Lazy<ConnectionMultiplexer>
    lazyConnection = new Lazy<ConnectionMultiplexer> (() =>
    {
        return ConnectionMultiplexer.Connect("contoso5premium.redis.cache.windows.net,abortConnect=false,ssl=true,password=password");
    });

public static ConnectionMultiplexer Connection
{
    get
    {
        return lazyConnection.Value;
    }
}
```

## <a name="azure-cache-for-redis-vnet-faq"></a>Preguntas frecuentes sobre redes virtuales de Azure Cache for Redis
La lista siguiente contiene las respuestas a las preguntas más frecuentes sobre el escalado de Azure Cache for Redis.

* ¿Cuáles son algunos de los problemas comunes de configuración incorrecta con Azure Cache for Redis y las redes virtuales?
* [¿Cómo se puede comprobar que la memoria caché funciona una red virtual?](#how-can-i-verify-that-my-cache-is-working-in-a-vnet)
* Al intentar conectarme a la instancia de Azure Cache for Redis en una red virtual, ¿por qué aparece un error que indica que el certificado remoto no es válido?
* [¿Se pueden usar redes virtuales con una memoria caché Basic o Estándar?](#can-i-use-vnets-with-a-standard-or-basic-cache)
* ¿Por qué se produce un error al crear una instancia de Azure Cache for Redis en algunas subredes, pero no en otras?
* [¿Cuáles son los requisitos de espacio de direcciones de subred?](#what-are-the-subnet-address-space-requirements)
* [¿Funcionarán todas las características al alojar una caché en una red virtual?](#do-all-cache-features-work-when-hosting-a-cache-in-a-vnet)

### <a name="what-are-some-common-misconfiguration-issues-with-azure-cache-for-redis-and-vnets"></a>¿Cuáles son algunos de los problemas comunes de configuración incorrecta con Azure Cache for Redis y las redes virtuales?
Cuando Azure Cache for Redis se hospeda en una red virtual, se usan los puertos de la tabla siguiente. 

>[!IMPORTANT]
>Si los puertos de las siguientes tablas están bloqueados, puede que la memoria caché no funcione correctamente. Tener bloqueados uno o varios de estos puertos es el problema más común de una configuración incorrecta cuando se utiliza Azure Cache for Redis en una red virtual.
> 
> 

- [Requisitos de puerto de salida](#outbound-port-requirements)
- [Requisitos de puerto de entrada](#inbound-port-requirements)

#### <a name="outbound-port-requirements"></a>Requisitos de puerto de salida

Existen nueve requisitos de puerto de salida. Las solicitudes que salen de estos intervalos son de salida a otros servicios necesarios para que la memoria caché funcione o internas a la subred de Redis para la comunicación entre nodos. Para la replicación geográfica, existen requisitos de salida adicionales para la comunicación entre subredes de la caché principal y la caché de réplica.

| Puertos | Dirección | Protocolo de transporte | Propósito | IP local | Dirección IP remota |
| --- | --- | --- | --- | --- | --- |
| 80, 443 |Salida |TCP |Dependencias de Redis en Azure Storage/PKI (Internet) | (Subred de Redis) |* |
| 443 | Salida | TCP | Dependencia de Redis en Azure Key Vault | (Subred de Redis) | AzureKeyVault <sup>1</sup> |
| 53 |Salida |TCP/UDP |Dependencias de Redis en DNS (Internet y red virtual) | (Subred de Redis) | 168.63.129.16 y 169.254.169.254 <sup>2</sup> y cualquier servidor DNS personalizado para la subred <sup>3</sup> |
| 8443 |Salida |TCP |Comunicaciones internas en Redis | (Subred de Redis) | (Subred de Redis) |
| 10221-10231 |Salida |TCP |Comunicaciones internas en Redis | (Subred de Redis) | (Subred de Redis) |
| 20226 |Salida |TCP |Comunicaciones internas en Redis | (Subred de Redis) |(Subred de Redis) |
| 13000-13999 |Salida |TCP |Comunicaciones internas en Redis | (Subred de Redis) |(Subred de Redis) |
| 15000-15999 |Salida |TCP |Comunicaciones internas de Redis y replicación geográfica | (Subred de Redis) |(Subred de Redis) (Subred del mismo nivel de réplica geográfica) |
| 6379-6380 |Salida |TCP |Comunicaciones internas en Redis | (Subred de Redis) |(Subred de Redis) |

<sup>1</sup> Puede usar la etiqueta de servicio "AzureKeyVault" con los grupos de seguridad de red de Resource Manager.

<sup>2</sup> Estas direcciones IP que pertenecen a Microsoft se usan para dirigir la máquina virtual del host que trabaja con Azure DNS.

<sup>3</sup> No es necesario para aquellas subredes sin ningún servidor DNS personalizado o cachés en Redis que ignoran los DNS personalizados.

#### <a name="geo-replication-peer-port-requirements"></a>Requisitos de los puertos del mismo nivel en la replicación geográfica

Si usa la replicación geográfica entre cachés en instancias de Azure Virtual Network, tenga en cuenta que la configuración recomendada es desbloquear los puertos 15000-15999 en toda la subred tanto para las direcciones de entrada como para las direcciones de salida dirigidas a ambas cachés. De este modo, todos los componentes de réplica que conforman la subred pueden comunicarse directamente entre sí aunque se produzca una conmutación por error geográfica.

#### <a name="inbound-port-requirements"></a>Requisitos de puerto de entrada

Existen ocho requisitos de intervalo de puertos de entrada. Las solicitudes entrantes de estos intervalos provienen de otros servicios hospedados en la misma red virtual o son comunicaciones internas de la subred de Redis.

| Puertos | Dirección | Protocolo de transporte | Propósito | IP local | Dirección IP remota |
| --- | --- | --- | --- | --- | --- |
| 6379, 6380 |Entrada |TCP |Comunicación del cliente con Redis, equilibrio de carga de Azure | (Subred de Redis) | (Subred de Redis), Virtual Network, Azure Load Balancer <sup>1</sup> |
| 8443 |Entrada |TCP |Comunicaciones internas en Redis | (Subred de Redis) |(Subred de Redis) |
| 8500 |Entrada |TCP/UDP |Equilibrio de carga de Azure | (Subred de Redis) |Azure Load Balancer |
| 10221-10231 |Entrada |TCP |Comunicaciones internas en Redis | (Subred de Redis) |(Subred de Redis), Azure Load Balancer |
| 13000-13999 |Entrada |TCP |Comunicación del cliente con clústeres de Redis, Equilibrio de carga de Azure | (Subred de Redis) |Virtual Network, Azure Load Balancer |
| 15000-15999 |Entrada |TCP |Comunicación del cliente con los clústeres de Redis, el equilibrio de carga de Azure y la replicación geográfica | (Subred de Redis) |Virtual Network, Azure Load Balancer, (subred del mismo nivel de réplica geográfica) |
| 16001 |Entrada |TCP/UDP |Equilibrio de carga de Azure | (Subred de Redis) |Azure Load Balancer |
| 20226 |Entrada |TCP |Comunicaciones internas en Redis | (Subred de Redis) |(Subred de Redis) |

<sup>1</sup> Puede usar la etiqueta de servicio "AzureLoadBalancer" (Resource Manager) (o "AZURE_LOADBALANCER" para el modelo clásico) para crear las reglas del grupo de seguridad de red.

#### <a name="additional-vnet-network-connectivity-requirements"></a>Requisitos adicionales de conectividad de red virtual

Existen requisitos de conectividad de red para entornos para una instancia de Azure Cache for Redis que no pueden cumplirse inicialmente en una red virtual. Azure Cache for Redis requiere todos los elementos siguientes para funcionar correctamente cuando se usa en una red virtual.

* Conectividad de red saliente a los puntos de conexión de Azure Storage en todo el mundo. Aquí se incluyen los puntos de conexión ubicados en la misma región que la instancia de Azure Cache for Redis, así como los puntos de conexión de almacenamiento ubicados en **otras** regiones de Azure. Los puntos de conexión de Azure Storage se resuelven en los siguientes dominios de DNS: *table.core.windows.net*, *blob.core.windows.net*, *queue.core.windows.net* y *file.core.windows.net*. 
* Conectividad de red saliente con *ocsp.msocsp.com*, *mscrl.microsoft.com* y *crl.microsoft.com*. Esta conectividad es necesaria para admitir la funcionalidad TLS/SSL.
* La configuración de DNS para la red virtual debe ser capaz de resolver todos los puntos de conexión y dominios mencionados en los puntos anteriores. Se pueden cumplir los requisitos de DNS al asegurar que se configura y se mantiene una infraestructura DNS válida para la red virtual.
* Conectividad de red saliente con los siguientes puntos de conexión de Azure Monitor que se resuelven en los siguientes dominios DNS: shoebox2-black.shoebox2.metrics.nsatc.net, north-prod2.prod2.metrics.nsatc.net, azglobal-black.azglobal.metrics.nsatc.net, shoebox2-red.shoebox2.metrics.nsatc.net, east-prod2.prod2.metrics.nsatc.net, azglobal-red.azglobal.metrics.nsatc.net.

### <a name="how-can-i-verify-that-my-cache-is-working-in-a-vnet"></a>¿Cómo se puede comprobar que la memoria caché funciona una red virtual?

>[!IMPORTANT]
>Al conectarse a una instancia de Azure Cache for Redis que se hospeda en una red virtual, los clientes de caché deben estar en la misma red virtual o en una red virtual con el emparejamiento de VNET habilitado dentro de la misma región de Azure. El emparejamiento de VNET global no se admite actualmente. Esto incluye cualquier aplicación de prueba o herramienta de diagnóstico de hacer ping. Independientemente de dónde se hospede la aplicación cliente, los grupos de seguridad de red deben configurarse de modo que el tráfico de red del cliente pueda llegar a la instancia de Redis.
>
>

Cuando los requisitos de puerto se configuran tal como se describe en la sección anterior, puede comprobar que la memoria caché funciona mediante la realización de los pasos siguientes.

- [Reinicie](cache-administration.md#reboot) todos los nodos de la caché. Si no se pueden alcanzar todas las dependencias de caché necesarias (como se documenta en [Requisitos del puerto de entrada](cache-how-to-premium-vnet.md#inbound-port-requirements) y [Requisitos de puerto de salida](cache-how-to-premium-vnet.md#outbound-port-requirements)), la caché no podrá reiniciarse correctamente.
- Cuando se han reiniciado los nodos de la memoria caché (como indica el estado de la caché en Azure Portal), puede realizar las siguientes pruebas:
  - hacer ping en el punto de conexión de caché (con el puerto 6380) desde una máquina que esté dentro de la misma red virtual como la memoria caché, con [tcping](https://www.elifulkerson.com/projects/tcping.php). Por ejemplo:
    
    `tcping.exe contosocache.redis.cache.windows.net 6380`
    
    Si la herramienta `tcping` informa de que el puerto está abierto, la memoria caché está disponible para su conexión en los clientes de la red virtual.

  - Otra manera de realizar la prueba consiste en crear un cliente de caché de prueba (que podría ser una sencilla aplicación de consola mediante StackExchange.Redis) que se conecta a la memoria caché y agrega y recupera algunos elementos de la caché. Instale la aplicación de cliente de ejemplo en una máquina virtual que está en la misma red virtual que la caché y ejecútela para comprobar la conectividad en la memoria caché.


### <a name="when-trying-to-connect-to-my-azure-cache-for-redis-in-a-vnet-why-am-i-getting-an-error-stating-the-remote-certificate-is-invalid"></a>Al intentar conectarme a la instancia de Azure Cache for Redis en una red virtual, ¿por qué aparece un error que indica que el certificado remoto no es válido?

Cuando intente conectarse a una instancia de Azure Cache for Redis en una red virtual, verá un error de validación de certificado como este:

`{"No connection is available to service this operation: SET mykey; The remote certificate is invalid according to the validation procedure.; …"}`

La causa podría ser que se está conectando al host mediante la dirección IP. Se recomienda utilizar el nombre de host. En otras palabras, use lo siguiente:     

`[mycachename].redis.windows.net:6380,password=xxxxxxxxxxxxxxxxxxxx,ssl=True,abortConnect=False`

Evite el uso de la dirección IP similar a la cadena de conexión siguiente:

`10.128.2.84:6380,password=xxxxxxxxxxxxxxxxxxxx,ssl=True,abortConnect=False`

Si no puede resolver el nombre DNS, algunas bibliotecas de cliente incluyen opciones de configuración como `sslHost`, que proporciona el cliente StackExchange.Redis. Esto le permite invalidar el nombre de host utilizado para la validación del certificado. Por ejemplo:

`10.128.2.84:6380,password=xxxxxxxxxxxxxxxxxxxx,ssl=True,abortConnect=False;sslHost=[mycachename].redis.windows.net`

### <a name="can-i-use-vnets-with-a-standard-or-basic-cache"></a>¿Se pueden usar redes virtuales con una memoria caché Basic o Estándar?
Las redes virtuales solo se pueden usar con memorias caché Premium.

### <a name="why-does-creating-an-azure-cache-for-redis-fail-in-some-subnets-but-not-others"></a>¿Por qué se produce un error al crear una instancia de Azure Cache for Redis en algunas subredes, pero no en otras?
Si se implementa una instancia de Azure Cache for Redis en una red virtual de Resource Manager, la memoria caché debe estar en una subred dedicada que no contenga ningún otro tipo de recursos. Si se intenta implementar una caché de Azure Cache for Redis en una subred de red virtual de Resource Manager que contiene otros recursos, se produce un error en la implementación. Para poder crear una nueva instancia de Azure Cache for Redis, es preciso eliminar los recursos existentes en la subred.

Puede implementar varios tipos de recursos en una red virtual con el enfoque clásico, siempre que tenga suficientes direcciones IP disponibles.

### <a name="what-are-the-subnet-address-space-requirements"></a>¿Cuáles son los requisitos de espacio de direcciones de subred?
Azure reserva algunas direcciones IP dentro de cada subred y estas direcciones no se pueden usar. La primera y la última dirección IP de las subredes están reservadas para la conformidad con el protocolo, junto con otras tres direcciones usadas para los servicios de Azure. Para más información, consulte [¿Hay alguna restricción en el uso de direcciones IP dentro de estas subredes?](../virtual-network/virtual-networks-faq.md#are-there-any-restrictions-on-using-ip-addresses-within-these-subnets)

Además de las direcciones IP que usa la infraestructura de Azure Virtual Network, cada instancia de Redis de la subred usa dos direcciones IP por partición y una dirección IP adicional para el equilibrador de carga. Se considera que una caché no en clúster tiene una partición.

### <a name="do-all-cache-features-work-when-hosting-a-cache-in-a-vnet"></a>¿Funcionarán todas las características al alojar una caché en una red virtual?
Cuando la memoria caché forma parte de una red virtual, solo los clientes de la red virtual pueden tener acceso a la memoria caché. Como resultado, las siguientes características de administración de la memoria caché no funcionan en este momento.

* Consola de Redis. Como la consola de Redis se ejecuta en el explorador local, que está fuera de la red virtual, no se puede conectar a la caché.


## <a name="use-expressroute-with-azure-cache-for-redis"></a>Uso de ExpressRoute con Azure Cache for Redis

Los clientes pueden conectar un circuito de [Azure ExpressRoute](https://azure.microsoft.com/services/expressroute/) a su infraestructura de red virtual, lo que permite ampliar la red local a Azure. 

De forma predeterminada, un circuito ExpressRoute recién creado no realiza la tunelización forzada (anuncio de una ruta predeterminada, 0.0.0.0/0) en una red virtual. Como resultado, se permite la conectividad de salida de Internet directamente desde la red virtual y las aplicaciones cliente pueden conectarse a otros puntos de conexión de Azure, incluido Azure Cache for Redis.

Pero una configuración de cliente común es usar la tunelización forzada (anunciar una ruta predeterminada), que fuerza a que el tráfico de salida de Internet fluya a nivel local. Este flujo de tráfico interrumpe la conectividad con Azure Cache for Redis si el tráfico de salida se bloquea en el entorno local debido a que la instancia de Azure Cache for Redis no se puede comunicar con sus dependencias.

La solución es definir una, o varias, rutas definidas por el usuario (UDR) en la subred que contiene el servicio Azure Cache for Redis. Una ruta definida por el usuario define las rutas de subred específica que se respetarán en lugar de la ruta predeterminada.

Si es posible, se recomienda usar la configuración siguiente:

* La configuración de ExpressRoute anuncia 0.0.0.0/0 y, de manera predeterminada, fuerza la tunelización de todo el tráfico saliente a local.
* El UDR aplicado a la subred que contiene Azure Cache for Redis define 0.0.0.0/0 con una ruta de trabajo para tráfico TCP/IP a Internet; por ejemplo, estableciendo el tipo de próximo salto en "Internet".

El efecto combinado de estos pasos es que la UDR a nivel de subred tiene prioridad sobre la tunelización forzada de ExpressRoute, lo que garantiza el acceso saliente a Internet desde la instancia de Azure Cache for Redis.

La conexión a una instancia de Azure Cache for Redis desde una aplicación local mediante ExpressRoute no es un escenario de uso típico debido a motivos de rendimiento (para que el rendimiento sea óptimo, los clientes de Azure Cache for Redis deben estar en la misma región que Azure Cache for Redis).

>[!IMPORTANT] 
>Las rutas definidas en una UDR **deben** ser lo suficientemente específicas para que tengan prioridad sobre cualquier otra ruta anunciada por la configuración de ExpressRoute. En el ejemplo siguiente se usa el intervalo de direcciones 0.0.0.0/0 amplio y como tal puede reemplazarse accidentalmente por los anuncios de ruta con intervalos de direcciones más específicos.

>[!WARNING]  
>Azure Cache for Redis no es compatible con las configuraciones de ExpressRoute que **anuncian incorrectamente rutas entre la ruta de acceso de emparejamiento público y la ruta de acceso de emparejamiento privado**. Las configuraciones de ExpressRoute con el emparejamiento público configurado reciben anuncios de ruta de Microsoft para un amplio conjunto de intervalos de direcciones IP de Microsoft Azure. Si estos intervalos de direcciones se anuncian incorrectamente en la ruta de acceso de emparejamiento privado, el resultado es que se forzará incorrectamente la tunelización de todos los paquetes de red salientes desde la subred de la instancia de Azure Cache for Redis a la infraestructura de red local del cliente. Este flujo de red interrumpe Azure Cache for Redis. La solución a este problema consiste en detener rutas anunciadas entre la ruta de acceso de interconexión pública y la ruta de acceso de interconexión privada.


Se puede encontrar información de contexto sobre las rutas definidas por el usuario en esta [información general](../virtual-network/virtual-networks-udr-overview.md).

Para más información sobre ExpressRoute, consulte [Información técnica sobre ExpressRoute](../expressroute/expressroute-introduction.md).

## <a name="next-steps"></a>Pasos siguientes
Más información sobre las características de Azure Cache for Redis.

* [Niveles de servicio de Azure Cache for Redis Premium](cache-overview.md#service-tiers)

<!-- IMAGES -->

[redis-cache-vnet]: ./media/cache-how-to-premium-vnet/redis-cache-vnet.png

[redis-cache-vnet-ip]: ./media/cache-how-to-premium-vnet/redis-cache-vnet-ip.png

[redis-cache-vnet-info]: ./media/cache-how-to-premium-vnet/redis-cache-vnet-info.png

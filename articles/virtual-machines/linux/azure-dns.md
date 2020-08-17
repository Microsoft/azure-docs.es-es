---
title: Opciones de resolución de nombres DNS para máquinas virtuales Linux
description: Escenarios de resolución de nombres DNS para máquinas virtuales Linux en IaaS de Azure, incluidos los servicios DNS proporcionados y el servidor DNS externo híbrido y Bring Your Own DNS.
author: RicksterCDN
ms.service: virtual-machines-linux
ms.topic: conceptual
ms.date: 10/19/2016
ms.author: rclaus
ms.openlocfilehash: aa007888c68df41242f937e1062a90ec1b7fc3ce
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87372831"
---
# <a name="dns-name-resolution-options-for-linux-virtual-machines-in-azure"></a>Opciones de resolución de nombres DNS para máquinas virtuales Linux en Azure
Azure proporciona resolución de nombres DNS de forma predeterminada para todas las máquinas virtuales de una única red virtual. Puede implementar su propia solución de resolución de nombre DNS si configura sus propios servicios DNS en las máquinas virtuales hospedadas de Azure. Los siguientes escenarios deberían ayudarle a elegir cuál funciona mejor para su situación.

* [Resolución de nombres que proporciona Azure](#name-resolution-that-azure-provides)
* [Resolución de nombres mediante su propio servidor DNS](#name-resolution-using-your-own-dns-server)

El tipo de resolución de nombres que use dependerá de cómo se comuniquen las máquinas virtuales y las instancias de rol entre sí.

En la siguiente tabla se muestran los escenarios y las soluciones de resolución de nombre correspondientes:

| **Escenario** | **Solución** | **Sufijo** |
| --- | --- | --- |
| Resolución de nombres entre instancias de rol o máquinas virtuales de la misma red virtual |Resolución de nombres que proporciona Azure |El nombre de host o el nombre de dominio completo (FQDN) |
| Resolución de nombres entre instancias de rol o máquinas virtuales de redes virtuales diferentes |Servidores DNS administrados por el cliente que reenvían consultas entre redes virtuales para la resolución mediante Azure (proxy DNS). Consulte [Resolución de nombres mediante su propio servidor DNS](#name-resolution-using-your-own-dns-server). |Solo FQDN |
| Resolución de nombres de servicios y de equipos locales desde instancias de rol o máquinas virtuales en Azure |Servidores DNS administrados por el cliente (por ejemplo, controlador de dominio local, controlador de dominio de solo lectura local o un DNS secundario sincronizado mediante transferencias de zona). Consulte [Resolución de nombres mediante su propio servidor DNS](#name-resolution-using-your-own-dns-server). |Solo FQDN |
| Resolución de nombres de host de Azure desde equipos locales |Reenvío de consultas a un servidor proxy DNS administrado por el cliente en la red virtual correspondiente. El servidor proxy reenvía consultas a Azure para resolverlas. Consulte [Resolución de nombres mediante su propio servidor DNS](#name-resolution-using-your-own-dns-server). |Solo FQDN |
| DNS inverso para direcciones IP internas |[Resolución de nombres mediante su propio servidor DNS](#name-resolution-using-your-own-dns-server) |N/D |

## <a name="name-resolution-that-azure-provides"></a>Resolución de nombres que proporciona Azure
Junto con la resolución de nombres DNS públicos, Azure proporciona resolución de nombres interna para máquinas virtuales e instancias de rol que se encuentren en la misma red virtual. En las redes virtuales basadas en Azure Resource Manager, el sufijo DNS es coherente en toda la red virtual. El FQDN no es necesario. Los nombres DNS pueden asignarse tanto a tarjetas de interfaz de red (NIC) como a máquinas virtuales. Aunque la resolución de nombres que proporciona Azure no necesita ningún tipo de configuración, no es la opción adecuada para todos los escenarios de implementación, tal y como se mostró en la tabla anterior.

### <a name="features-and-considerations"></a>Características y consideraciones
**Características**

* No es necesario realizar configuración alguna para usar la resolución de nombres proporcionada por Azure.
* El servicio de resolución de nombres que proporciona Azure tiene una alta disponibilidad. No es necesario crear clústeres de sus propios servidores DNS ni administrarlos.
* El servicio de resolución de nombres proporcionado por Azure puede usarse junto con sus propios servidores DNS para resolver nombres de host tanto de Azure como locales.
* Se proporciona resolución de nombres entre las máquinas virtuales en redes virtuales sin necesidad de FQDN.
* Puede usar los nombres de host que describan mejor las implementaciones, en lugar de trabajar con nombres generados automáticamente.

**Consideraciones:**

* No se puede modificar el sufijo DNS creado por Azure.
* No se pueden registrar manualmente los registros propios.
* No se admiten ni WINS ni NetBIOS.
* Los nombres de host deben ser compatibles con DNS.
    Los nombres solamente pueden contener caracteres 0-9, a-z y "-", y no pueden comenzar ni terminar por "-". Consulte la sección 2 de RFC 3696.
* El tráfico de consultas de DNS está limitado en cada máquina virtual. La limitación no debería afectar a la mayoría de las aplicaciones.  Si se observa una limitación de solicitudes, asegúrese de que está habilitado el almacenamiento en caché del lado cliente.  Para obtener más información, consulte [Obtención del máximo partido de la resolución de nombres de Azure](#getting-the-most-from-name-resolution-that-azure-provides).

### <a name="getting-the-most-from-name-resolution-that-azure-provides"></a>Obtención del máximo partido de la resolución de nombres de Azure
**Almacenamiento en caché del lado cliente:**

Algunas consultas de DNS no se envían a través de la red. El almacenamiento en caché del lado cliente ayuda a reducir la latencia y mejorar la resistencia a las incoherencias de la red mediante la resolución de consultas de DNS periódicas desde una caché local. Los registros DNS contienen un período de vida (TTL) que permite a la caché almacenar el registro durante tanto tiempo como sea posible sin afectar a la actualización de los registros. Por ello, la caché del cliente es adecuada para la mayoría de las situaciones.

Algunas distribuciones de Linux no incluyen el almacenamiento en caché de forma predeterminada. Se recomienda agregar una caché a cada máquina virtual Linux una vez comprobado que no exista ya una caché local.

Hay disponibles varios paquetes de almacenamiento en caché DNS, como dnsmasq. Estos son los pasos que hay que seguir para instalar dnsmasq en las distribuciones más habituales:

**Ubuntu (usa resolvconf)**
  * Instale el paquete dnsmasq ("sudo apt-get install dnsmasq").

**SUSE (usa netconf)** :
1. Instale el paquete dnsmasq ("sudo zypper install dnsmasq").
2. Habilite el servicio de dnsmasq ("systemctl enable dnsmasq.service").
3. Inicie el servicio de dnsmasq ("systemctl start dnsmasq.service").
4. Edite "/etc/sysconfig/network/config" y cambie NETCONFIG_DNS_FORWARDER="" por "dnsmasq".
5. Actualice resolv.conf ("netconfig update") para establecer la memoria caché como la resolución DNS local.

**CentOS de Rogue Wave Software (anteriormente OpenLogic; usa NetworkManager)**
1. Instale el paquete dnsmasq ("sudo yum install dnsmasq").
2. Habilite el servicio de dnsmasq ("systemctl enable dnsmasq.service").
3. Inicie el servicio de dnsmasq ("systemctl start dnsmasq.service").
4. Agregue "prepend domain-name-servers 127.0.0.1;" a "/etc/dhclient-eth0.conf".
5. Reinicie el servicio de red ("service network restart") para establecer la caché como la resolución DNS local.

> [!NOTE]
> : El paquete "dnsmasq" es solo una de las muchas cachés DNS disponibles para Linux. Antes de usarlo, compruebe su idoneidad para sus necesidades y asegúrese de que no haya instalada ninguna otra memoria caché.
>
>

**Reintentos del cliente:**

DNS es principalmente un protocolo UDP. Dado que el protocolo UDP no garantiza la entrega de mensajes, es el propio protocolo DNS el que controla la lógica de reintento. Cada cliente DNS (sistema operativo) puede presentar una lógica de reintento diferente en función de la preferencia del creador:

* Los sistemas operativos Windows realizan un reintento tras un segundo y después tras otros dos, cuatro y otros cuatro segundos.
* El programa de instalación predeterminado de Linux lo intenta después de cinco segundos.  Se recomienda cambiar esta opción para reintentarlo cinco veces a intervalos de un segundo.  

Para comprobar la configuración actual en una máquina virtual Linux, vaya a "cat /etc/resolv.conf" y busque la línea "options", por ejemplo:

```config-conf
options timeout:1 attempts:5
```

El archivo resolv.conf es autogenerado y no se debe editar. Los pasos específicos para agregar la línea "options" varían según la distribución:

**Ubuntu** (usa resolvconf)
1. Agregue la línea options a '/etc/resolvconf/resolv.conf.d/head'.
2. Ejecute "resolvconf -u" para efectuar la actualización.

**SUSE** (usa netconf)
1. Agregue "timeout:1 attempts:5" al parámetro NETCONFIG_DNS_RESOLVER_OPTIONS="" en "/etc/sysconfig/network/config".
2. Ejecute "netconfig update" para efectuar la actualización.

**CentOS de Rogue Wave Software (anteriormente OpenLogic)** (usa NetworkManager)
1. Agregue 'RES_OPTIONS="timeout:1 attempts:5"' a '/etc/sysconfig/network'.
2. Ejecute "service network restart" para efectuar la actualización.

## <a name="name-resolution-using-your-own-dns-server"></a>Resolución de nombres mediante su propio servidor DNS
Puede que sus necesidades de resolución de nombres se escapen a las posibilidades de las características de Azure. Por ejemplo, puede necesitar resolución DNS entre redes virtuales. Para resolver este tipo de escenarios, puede usar sus propios servidores DNS.  

Los servidores DNS de una red virtual pueden reenviar consultas de DNS a resoluciones recursivas de Azure para resolver los nombres de host que se encuentren en la misma red virtual. Por ejemplo, un servidor DNS que se ejecute en Azure puede responder a consultas de DNS de sus propios archivos de zonas DNS y reenviar todas las demás consultas a Azure. Esta funcionalidad permite a la máquina virtual detectar tanto las entradas de los archivos de zona como los nombres de host proporcionados por Azure (a través del reenviador). El acceso a las resoluciones recursivas de Azure se proporciona a través de la IP virtual 168.63.129.16.

El reenvío de DNS también habilita la resolución DNS entre redes virtuales y permite que las máquinas locales resuelvan los nombres de host que proporciona Azure. Para resolver el nombre de host de una máquina virtual, la máquina virtual del servidor DNS debe residir en la misma red virtual y debe configurarse para desviar consultas de nombre de host a Azure. Dado que el sufijo DNS es diferente en cada red virtual, puede usar las reglas de desvío condicional para enviar consultas de DNS a la red virtual correcta para su resolución. En la imagen siguiente se muestran dos redes virtuales y una red local que está realizando resolución DNS entre redes virtuales con este método:

![Resolución DNS entre redes virtuales](./media/azure-dns/inter-vnet-dns.png)

Cuando se usa la resolución de nombres proporcionada por Azure, el sufijo DNS interno se proporciona a cada máquina virtual mediante DHCP. Cuando usa su propia solución de resolución de nombres, no se proporciona este sufijo a las máquinas virtuales puesto que interfiere en otras arquitecturas DNS. Para hacer referencia a máquinas mediante FQDN, o para configurar el sufijo en las máquinas virtuales, este puede determinarse mediante PowerShell o la API:

* Para aquellas redes virtuales administradas por Azure Resource Manager, el sufijo está disponible a través del recurso de [tarjeta de interfaz de red](/rest/api/virtualnetwork/networkinterfaces). También puede ejecutar el comando `azure network public-ip show <resource group> <pip name>` para mostrar los datos de la IP pública, entre los que se incluye el FQDN de la NIC.

Si el reenvío de consultas a Azure no satisface sus necesidades, debe proporcionar su propia solución de DNS.  La solución DNS debe:

* Proporcionar la resolución adecuada de nombres de host, por ejemplo, mediante [DDNS](../../virtual-network/virtual-networks-name-resolution-ddns.md). Si usa DDNS, puede necesitar deshabilitar la limpieza de registros DNS. Las concesiones DHCP de Azure son muy largas y la limpieza puede eliminar registros DNS de forma prematura.
* Proporcionar la resolución recursiva adecuada para permitir la resolución de nombres de dominio externos.
* Estar accesible (TCP y UDP en el puerto 53) desde los clientes a los que sirve y poder acceder a Internet.
* Tener protección contra el acceso desde Internet para mitigar las amenazas que suponen agentes externos.

> [!NOTE]
> Para conseguir un rendimiento óptimo, al usar máquinas virtuales en servidores DNS de Azure, deshabilite IPv6 y asigne una [Dirección IP pública a nivel de instancia](/previous-versions/azure/virtual-network/virtual-networks-instance-level-public-ip) a cada máquina virtual de servidor DNS.  
>
>

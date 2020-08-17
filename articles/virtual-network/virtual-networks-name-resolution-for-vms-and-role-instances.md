---
title: Resolución de nombres de recursos en redes virtuales de Azure
titlesuffix: Azure Virtual Network
description: Escenarios de resolución de nombres para IaaS de Azure, soluciones híbridas, entre servicios en la nube diferentes, Active Directory y con su propio servidor DNS.
services: virtual-network
documentationcenter: na
author: rohinkoul
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 3/2/2020
ms.author: rohink
ms.custom: fasttrack-edit
ms.openlocfilehash: cf630f6028248d799a3953d25db27a2150602586
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87087018"
---
# <a name="name-resolution-for-resources-in-azure-virtual-networks"></a>Resolución de nombres de recursos en redes virtuales de Azure

Según cómo utilice Azure para hospedar soluciones híbridas, IaaS y PaaS, puede que tenga que permitir que las máquinas virtuales (VM) y otros recursos implementados en una red virtual se comuniquen entre sí. Aunque puede habilitar la comunicación mediante las direcciones IP, es mucho más sencillo usar nombres, ya que podrá recordarlos con más facilidad y no cambian. 

Si los recursos implementados en redes virtuales necesitan resolver nombres de dominio en direcciones IP internas, pueden usar uno de tres métodos:

* [Zonas privadas de Azure DNS](../dns/private-dns-overview.md)
* [Resolución de nombres de Azure](#azure-provided-name-resolution)
* [Resolución de nombres con su propio servidor DNS](#name-resolution-that-uses-your-own-dns-server) (que puede reenviar consultas a los servidores DNS proporcionados por Azure)

El tipo de resolución de nombres que tenga que usar dependerá de cómo se comuniquen los recursos entre sí. En la siguiente tabla se muestran los escenarios y las soluciones de resolución de nombre correspondientes:

> [!NOTE]
> Las zonas privadas de Azure DNS son la solución preferida y proporcionan flexibilidad para la administración de los registros y las zonas DNS. Para obtener más información, vea [Uso de Azure DNS para dominios privados](../dns/private-dns-overview.md).

> [!NOTE]
> Si usa un DNS proporcionado por Azure, el sufijo DNS adecuado se aplicará automáticamente a las máquinas virtuales. Para todas las demás opciones, debe usar nombres de dominio completos (FQDN) o aplicar manualmente el sufijo DNS adecuado a las máquinas virtuales.

| **Escenario** | **Solución** | **Sufijo DNS** |
| --- | --- | --- |
| Resolución de nombres entre máquinas virtuales ubicadas en la misma red virtual, o instancias de rol de Azure Cloud Services en el mismo servicio en la nube. | [Zonas privadas de Azure DNS](../dns/private-dns-overview.md) o [resolución de nombres proporcionada por Azure](#azure-provided-name-resolution) |Nombre de host o FQDN |
| Resolución de nombres entre máquinas virtuales en redes virtuales diferentes o instancias de rol en diferentes servicios en la nube. |[Zonas privadas de Azure DNS](../dns/private-dns-overview.md) o servidores DNS administrados por el cliente que reenvían consultas entre redes virtuales para la resolución mediante Azure (proxy DNS). Consulte [Resolución de nombres mediante su propio servidor DNS](#name-resolution-that-uses-your-own-dns-server). |Solo FQDN |
| Resolución de nombres de Azure App Service (aplicación web, función o bot) mediante la integración de red virtual con instancias de rol o máquinas virtuales en la misma red virtual. |Servidores DNS administrados por el cliente que reenvían consultas entre redes virtuales para la resolución mediante Azure (proxy DNS). Consulte [Resolución de nombres mediante su propio servidor DNS](#name-resolution-that-uses-your-own-dns-server). |Solo FQDN |
| Resolución de nombres entre instancias de App Service Web Apps en máquinas virtuales en la misma red virtual. |Servidores DNS administrados por el cliente que reenvían consultas entre redes virtuales para la resolución mediante Azure (proxy DNS). Consulte [Resolución de nombres mediante su propio servidor DNS](#name-resolution-that-uses-your-own-dns-server). |Solo FQDN |
| Resolución de nombres de App Service Web Apps de una máquina virtual a las máquinas virtuales de otra red virtual. |Servidores DNS administrados por el cliente que reenvían consultas entre redes virtuales para la resolución mediante Azure (proxy DNS). Consulte [Resolución de nombres mediante su propio servidor DNS](#name-resolution-that-uses-your-own-dns-server). |Solo FQDN |
| Resolución de nombres de servicios y de equipos locales de máquinas virtuales o instancias de rol en Azure. |Servidores DNS administrados por el cliente (controlador de dominio local, controlador de dominio de solo lectura local o un DNS secundario sincronizado mediante transferencias de zona, por ejemplo). Consulte [Resolución de nombres mediante su propio servidor DNS](#name-resolution-that-uses-your-own-dns-server). |Solo FQDN |
| Resolución de nombres de host de Azure desde equipos locales. |Reenvío de consultas a un servidor proxy DNS administrado por el cliente en la red virtual correspondiente: el servidor proxy reenvía consultas a Azure para su resolución. Consulte [Resolución de nombres mediante su propio servidor DNS](#name-resolution-that-uses-your-own-dns-server). |Solo FQDN |
| DNS inverso para direcciones IP internas. |[Zonas privadas de Azure DNS](../dns/private-dns-overview.md), [resolución de nombres proporcionada por Azure](#azure-provided-name-resolution) o [resolución de nombres mediante su propio servidor DNS](#name-resolution-that-uses-your-own-dns-server). |No aplicable |
| Resolución de nombres entre máquinas virtuales o instancias de rol ubicadas en diferentes servicios en la nube y no en una red virtual. |No aplicable. La conectividad entre máquinas virtuales e instancias de rol de servicios en la nube diferentes no es compatible fuera de una red virtual. |No aplicable|

## <a name="azure-provided-name-resolution"></a>Resolución de nombres de Azure

La resolución de nombres proporcionada por Azure solo ofrece las funcionalidades autoritativas básicas de DNS. Si usa esta opción, Azure administrará automáticamente los registros y nombres de las zonas DNS, y no podrá controlar los nombres de zona DNS ni el ciclo de vida de los registros DNS. Si necesita una solución DNS completa para sus redes virtuales, debe usar [zonas privadas de Azure DNS](../dns/private-dns-overview.md) o [servidores DNS administrados por el cliente](#name-resolution-that-uses-your-own-dns-server).

Junto con la resolución de nombres DNS públicos, Azure proporciona una resolución de nombres interna para máquinas virtuales e instancias de rol que residen dentro de la misma red virtual o servicio en la nube. Las máquinas virtuales y las instancias en un servicio en la nube comparten el mismo sufijo DNS (por lo que el nombre de host por sí solo es suficiente). No obstante, en las redes virtuales implementadas con el modelo de implementación clásica, diferentes servicios en la nube tienen distintos sufijos DNS. En esta situación, es necesario el FQDN para resolver los nombres entre diferentes servicios en la nube. En las redes virtuales implementadas con el modelo de implementación de Azure Resource Manager, el sufijo DNS es coherente en todas las máquinas virtuales de una red virtual, de modo que el FQDN no es necesario. Los nombres DNS se pueden asignar a máquinas virtuales y a interfaces de red. Aunque la resolución de nombres que proporciona Azure no necesita ningún tipo de configuración, no es la opción más adecuada para todos los escenarios de implementación, tal y como se explicó en la tabla anterior.

> [!NOTE]
> En el caso de los roles web y de trabajo de servicios en la nube, puede acceder a las direcciones IP internas de las instancias de rol mediante la API de REST de administración de servicios de Azure. Para obtener más información, consulte la [Referencia de la API REST de administración de servicios](https://msdn.microsoft.com/library/azure/ee460799.aspx). La dirección se basa en el nombre de rol y el número de instancia. 
>

### <a name="features"></a>Características

La resolución de nombres proporcionada por Azure incluye las siguientes características:
* Facilidad de uso. No se requiere ninguna configuración.
* Alta disponibilidad. No es necesario crear clústeres de sus propios servidores DNS ni administrarlos.
* Puede utilizar el servicio junto con sus propios servidores DNS para resolver nombres de host de Azure y locales.
* Puede usar la resolución de nombres entre las máquinas virtuales y las instancias de rol del mismo servicio en la nube, sin necesidad de un nombre de dominio completo.
* Puede usar la resolución de nombres entre las máquinas virtuales en redes virtuales que usan el modelo de implementación de Azure Resource Manager, sin necesidad de un nombre de dominio completo. En el modelo de implementación clásica, las redes virtuales requieren un FQDN cuando se resuelven nombres en diferentes servicios en la nube. 
* Puede usar los nombres de host que describan mejor las implementaciones, en lugar de trabajar con nombres generados automáticamente.

### <a name="considerations"></a>Consideraciones

Puntos que deben considerarse cuando se utiliza la resolución de nombres de Azure:
* No se puede modificar el sufijo DNS creado por Azure.
* La búsqueda de DNS está en el ámbito de una red virtual. Los nombres DNS creados para una red virtual no se pueden resolver desde otras redes virtuales.
* No se pueden registrar manualmente los registros propios.
* No se admiten ni WINS ni NetBIOS. Las máquinas virtuales no se pueden ver en el Explorador de Windows.
* Los nombres de host deben ser compatibles con DNS. Los nombres solamente pueden contener los caracteres 0-9, a-z y "-", y no pueden comenzar ni terminar por "-".
* El tráfico de consultas de DNS está limitado por cada máquina virtual. La limitación no debería afectar a la mayoría de las aplicaciones. Si se observa una limitación de solicitudes, asegúrese de que está habilitado el almacenamiento en caché del lado cliente. Para más información, consulte [Configuración de cliente DNS](#dns-client-configuration).
* En un modelo de implementación clásico, solo se registran las máquinas virtuales de los 180 primeros servicios en la nube para cada red virtual clásica. Este límite no se aplica a las redes virtuales en Azure Resource Manager.
* La dirección IP de Azure DNS es 168.63.129.16. Es una dirección IP estática y no cambiará.

### <a name="reverse-dns-considerations"></a>Consideraciones sobre el DNS inverso
El DNS inverso es compatible con todas las redes virtuales basadas en ARM. Puede emitir consultas de DNS inverso (consultas PTR) para asignar las direcciones IP de las máquinas virtuales a los FQDN de las máquinas virtuales.
* Todas las consultas PTR para las direcciones IP de las máquinas virtuales devolverán FQDN con el formato \[vmname\].internal.cloudapp.net.
* La búsqueda directa en FQDN con el formato \[vmname\].internal.cloudapp.net se resolverá en la dirección IP asignada a la máquina virtual.
* Si la red virtual está vinculada a una [zona privada de Azure DNS](../dns/private-dns-overview.md) como una red virtual de registro, las consultas de DNS inverso devolverán dos registros. Un registro tiene el formato \[vmname\].[privatednszonename] y el otro \[vmname\].internal.cloudapp.net.
* La búsqueda de DNS inverso está en el ámbito de una red virtual determinada aunque esté emparejada con otras redes virtuales. Las consultas de DNS inverso (consultas PTR) para las direcciones IP de las máquinas virtuales que se encuentran en redes virtuales emparejadas devolverán NXDOMAIN.
* Si desea desactivar la función de DNS inverso en una red virtual, puede crear una zona de búsqueda inversa mediante las [zonas privadas de Azure DNS](../dns/private-dns-overview.md) y vincular esta zona a la red virtual. Por ejemplo, si el espacio de direcciones IP de la red virtual es 10.20.0.0/16, puede crear una zona DNS privada vacía 20.10.in-addr.arpa y vincularla a la red virtual. Al vincular la zona a la red virtual, debe deshabilitar el registro automático en el vínculo. Esta zona invalidará las zonas de búsqueda inversa predeterminadas para la red virtual y, puesto que esta zona está vacía, obtendrá NXDOMAIN para las consultas de DNS inversas. Consulte nuestra [guía de inicio rápido](https://docs.microsoft.com/azure/dns/private-dns-getstarted-portal) para más información sobre cómo crear una zona DNS privada y vincularla a una red virtual.

> [!NOTE]
> Si desea que la búsqueda de DNS inverso se extienda a través de la red virtual, puede crear una zona de búsqueda inversa (in-addr.arpa) denominada [zonas privadas de Azure DNS](../dns/private-dns-overview.md) y vincularla a varias redes virtuales. Sin embargo, tendrá que administrar manualmente los registros de DNS inverso para las máquinas virtuales.
>


## <a name="dns-client-configuration"></a>Configuración del cliente DNS

Esta sección trata los intentos del lado cliente y el almacenamiento en caché del cliente.

### <a name="client-side-caching"></a>Almacenamiento en caché del lado cliente

No todas las consultas DNS deben enviarse a través de la red. El almacenamiento en caché del cliente ayuda a reducir la latencia y a mejorar la resistencia a la señalización visual de la red mediante la resolución de las consultas DNS periódicas desde una memoria caché local. Los registros DNS contienen un mecanismo para contabilizar el período de vida (TTL), lo que permite a la memoria caché almacenar el registro el máximo tiempo posible sin afectar a la actualización de registros. Por ello, el almacenamiento en caché del cliente es adecuado para la mayoría de las situaciones.

El cliente DNS de Windows predeterminado tiene una memoria caché DNS integrada. Algunas distribuciones de Linux no incluyen el almacenamiento en caché de forma predeterminada. Si comprueba que aún no hay una memoria caché local, agregue una memoria caché DNS a cada máquina virtual Linux.

Hay una serie de diferentes paquetes de caché DNS disponibles, como dnsmasq. Aquí se indica cómo instalar dnsmasq en las distribuciones más habituales:

* **Ubuntu (usa resolvconf)** :
  * Instale el paquete dnsmasq con `sudo apt-get install dnsmasq`.
* **SUSE (usa netconf)** :
  * Instale el paquete dnsmasq con `sudo zypper install dnsmasq`.
  * Habilite el servicio dnsmasq con `systemctl enable dnsmasq.service`. 
  * Inicie el servicio dnsmasq con `systemctl start dnsmasq.service`. 
  * Edite **/etc/sysconfig/network/config** y cambie *NETCONFIG_DNS_FORWARDER=""* por *dnsmasq*.
  * Actualice resolv.conf con `netconfig update` para establecer la memoria caché como la resolución DNS local.
* **CentOS (usa NetworkManager)** :
  * Instale el paquete dnsmasq con `sudo yum install dnsmasq`.
  * Habilite el servicio dnsmasq con `systemctl enable dnsmasq.service`.
  * Inicie el servicio dnsmasq con `systemctl start dnsmasq.service`.
  * Agregue *prepend domain-name-servers 127.0.0.1;* a **/etc/dhclient-eth0.conf**.
  * Reinicie el servicio de red con `service network restart` para establecer la memoria caché como la resolución DNS local.

> [!NOTE]
> El paquete dnsmasq constituye solo una de las muchas memorias cachés DNS disponibles para Linux. Antes de usarlo, compruebe su idoneidad para sus necesidades concretas y que no esté instalada ninguna otra memoria caché.

    
### <a name="client-side-retries"></a>Reintentos del cliente

DNS es principalmente un protocolo UDP. Como el protocolo UDP no garantiza la entrega de mensajes, la lógica de reintento se controla en el mismo protocolo DNS. Cada cliente DNS (sistema operativo) puede presentar una lógica de reintento diferente en función de la preferencia del creador:

* Los sistemas operativos Windows realizan un reintento tras un segundo y después tras otros dos, cuatro y otros cuatro segundos. 
* El programa de instalación predeterminado de Linux lo intenta después de cinco segundos. Se recomienda cambiar las especificaciones de reintento a cinco veces, en intervalos de un segundo.

Compruebe la configuración actual en una VM de Linux con `cat /etc/resolv.conf`. Mire la línea *options*, por ejemplo:

```bash
options timeout:1 attempts:5
```

El archivo resolv.conf suele generarse de forma automática y no se debe editar. Los pasos específicos para agregar la línea *options* varían según la distribución:

* **Ubuntu** (usa resolvconf):
  1. Agregue la línea *options* a **/etc/resolvconf/resolv.conf.d/tail**.
  2. Ejecute `resolvconf -u` para actualizar.
* **SUSE** (usa netconf):
  1. Agregue *timeout:1 attempts:5* al parámetro **NETCONFIG_DNS_RESOLVER_OPTIONS=""** en **/etc/sysconfig/network/config**.
  2. Ejecute `netconfig update` para actualizar.
* **CentOS** (usa NetworkManager):
  1. Agregue *echo "options timeout:1 attempts:5"* a **/etc/NetworkManager/dispatcher.d/11-dhclient**.
  2. Actualice con `service network restart`.

## <a name="name-resolution-that-uses-your-own-dns-server"></a>Resolución de nombres con su propio servidor DNS

En esta sección se tratan las máquinas virtuales, las instancias de rol y las aplicaciones web.

### <a name="vms-and-role-instances"></a>Máquinas virtuales e instancias de rol

Puede que sus necesidades de resolución de nombres superen las posibilidades de las características de Azure. Por ejemplo, es posible que necesite utilizar dominios de Microsoft Windows Server Active Directory y resolver nombres DNS entre redes virtuales. Para abarcar estos escenarios, Azure le ofrece la posibilidad de que use sus propios servidores DNS.

Los servidores DNS dentro de una red virtual pueden reenviar consultas DNS a las resoluciones recursivas en Azure. Esto le permite resolver los nombres de host dentro de esa red virtual. Por ejemplo, un controlador de dominio (DC) que se ejecute en Azure puede responder a las consultas DNS referidas a sus dominios y reenviar todas las demás consultas a Azure. El reenvío de consultas permite que las máquinas virtuales vean sus recursos locales (mediante el controlador de dominio) y los nombres de host proporcionados por Azure (mediante el reenviador). El acceso a las resoluciones recursivas de Azure se proporciona mediante la IP virtual 168.63.129.16.

El reenvío de DNS también habilita la resolución de DNS entre redes virtuales y permite que los equipos locales resuelvan nombres de host proporcionados por Azure. Para resolver el nombre de host de una máquina virtual, la máquina virtual del servidor DNS debe residir en la misma red virtual y debe configurarse para reenviar consultas de nombre de host a Azure. Dado que el sufijo DNS es diferente en cada red virtual, puede usar las reglas de desvío condicional para enviar consultas de DNS a la red virtual correcta para su resolución. En la imagen siguiente se muestran dos redes virtuales y una red local que está realizando resolución DNS entre redes virtuales con este método. Puede encontrar un reenviador DNS de ejemplo disponible en la [galería de plantillas de inicio rápido de Azure](https://azure.microsoft.com/documentation/templates/301-dns-forwarder/) y en [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/301-dns-forwarder).

> [!NOTE]
> Una instancia de rol puede realizar la resolución de nombres de máquinas virtuales dentro de la misma red virtual. Para ello, use el FQDN, que consta del nombre de host de la máquina virtual y el sufijo DNS **internal.cloudapp.net**. Sin embargo, en este caso, la resolución de nombres solo es correcta si la instancia de rol tiene el nombre de máquina virtual definido en el [esquema Role (archivo .cscfg)](https://msdn.microsoft.com/library/azure/jj156212.aspx).
> `<Role name="<role-name>" vmName="<vm-name>">`
>
> Las instancias de rol que tienen que realizar la resolución de nombres de máquinas virtuales en otra red virtual (FQDN utilizando el sufijo **internal.cloudapp.net**) tienen que hacerlo con el método descrito en esta sección (reenvío de servidores DNS personalizados entre las dos redes virtuales).
>

![Diagrama de DNS entre redes virtuales](./media/virtual-networks-name-resolution-for-vms-and-role-instances/inter-vnet-dns.png)

Cuando se utiliza la resolución de nombres proporcionada con Azure, el Protocolo dinámico de configuración de host (DHCP) de Azure proporciona un sufijo DNS interno ( **. internal.cloudapp.net**) a cada máquina virtual. Este sufijo permite la resolución de nombres de host, dado que los registros de nombre de host están en la zona **internal.cloudapp.net**. Si usa su propia solución de resolución de nombres, no se proporciona este sufijo a las máquinas virtuales puesto que interfiere con otras arquitecturas DNS (como es el caso de los escenarios con unión a un dominio). En su lugar, Azure proporciona un marcador de posición no funcional (*reddog.microsoft.com*).

Si es necesario, el sufijo DNS interno se puede determinar con PowerShell o la API:

* En el caso de redes virtuales en modelos de implementación de Azure Resource Manager, el sufijo está disponible con el recurso de la [API REST de interfaz de red](https://docs.microsoft.com/rest/api/virtualnetwork/networkinterfaces) o del cmdlet [Get-AzNetworkInterface](/powershell/module/az.network/get-aznetworkinterface) de PowerShell y del comando de la CLI de Azure [az network nic show](/cli/azure/network/nic#az-network-nic-show).
* En los modelos de implementación clásica, el sufijo está disponible mediante la llamada a la [API de Get Deployment](https://msdn.microsoft.com/library/azure/ee460804.aspx) o por medio del cmdlet [Get-AzureVM -Debug](/powershell/module/servicemanagement/azure.service/get-azurevm).

Si el reenvío de consultas a Azure no satisface sus necesidades, debería proporcionar su propia solución DNS. La solución DNS debe:

* Proporcionar la resolución adecuada de nombres de host, por ejemplo, mediante [DDNS](virtual-networks-name-resolution-ddns.md). Si usa DDNS, puede que tenga que deshabilitar la limpieza de registros DNS. Las concesiones DHCP de Azure son muy largas y la limpieza puede eliminar registros DNS de forma prematura. 
* Proporcionar la resolución recursiva adecuada para permitir la resolución de nombres de dominio externos.
* Estar accesible (TCP y UDP en el puerto 53) desde los clientes a los que sirve y poder acceder a Internet.
* Tener protección contra el acceso desde Internet para mitigar las amenazas que suponen los agentes externos.

> [!NOTE]
> Para obtener el mejor rendimiento, cuando se usan máquinas virtuales de Azure como servidores DNS, debe deshabilitarse IPv6.

### <a name="web-apps"></a>Aplicaciones web
Supongamos que necesita realizar la resolución de nombres de una aplicación web integrada con App Service y vinculada a una red virtual, a las máquinas virtuales en la misma red virtual. Además de configurar un servidor DNS personalizado que tenga un reenviador DNS que reenvíe las consultas a Azure (dirección IP virtual 168.63.129.16), realice los pasos siguientes:
1. Habilite la integración de red virtual de la aplicación web, si aún no lo ha hecho, tal y como se describe en [Integración de su aplicación con una instancia de Azure Virtual Network](../app-service/web-sites-integrate-with-vnet.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
2. En Azure Portal, para el plan de App Service que hospeda la aplicación web, seleccione **Sincronizar red** en **Redes**, **Integración de Virtual Network**.

    ![Captura de pantalla de la resolución de nombres de red virtual](./media/virtual-networks-name-resolution-for-vms-and-role-instances/webapps-dns.png)

Si necesita ejecutar la resolución de nombres de la aplicación web integrada mediante App Service vinculada a una red virtual, a las máquinas virtuales que se encuentran en una red virtual diferente, debe usar servidores DNS personalizados en las dos redes virtuales, del modo siguiente:

* Configure un servidor DNS en la red virtual de destino en una máquina virtual que también pueda reenviar consultas a la resolución recursiva de Azure (dirección IP virtual 168.63.129.16). Puede encontrar un reenviador DNS de ejemplo disponible en la [galería de plantillas de inicio rápido de Azure](https://azure.microsoft.com/documentation/templates/301-dns-forwarder) y en [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/301-dns-forwarder). 
* Configure un reenviador DNS en la red virtual de origen en una máquina virtual. Configure este reenviador DNS para que reenvíe consultas al servidor DNS de la red virtual de destino.
* Configure el servidor DNS de origen en la configuración de su red virtual de origen.
* Habilite la integración de red virtual para la aplicación web para vincular a la red virtual de origen, siguiendo las instrucciones de [Integración de su aplicación con una instancia de Azure Virtual Network](../app-service/web-sites-integrate-with-vnet.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
* En Azure Portal, para el plan de App Service que hospeda la aplicación web, seleccione **Sincronizar red** en **Redes**, **Integración de Virtual Network**.

## <a name="specify-dns-servers"></a>Especificación de los servidores DNS
Si usa sus propios servidores DNS, Azure permite especificar varios servidores DNS por cada red virtual. También puede especificar varios servidores DNS por interfaz de red (para Azure Resource Manager) o por servicio en la nube (para el modelo de implementación clásica). Los servidores DNS especificados para una interfaz de red o un servicio en la nube tienen prioridad sobre los servidores DNS especificados para la red virtual.

> [!NOTE]
> Las propiedades de conexión de red, como las direcciones IP del servidor DNS, no deben editarse directamente dentro de las máquinas virtuales. Esto se debe a que podrían borrarse durante el servicio de reparación cuando se reemplaza el adaptador de la red virtual. Esto se aplica a las máquinas virtuales Windows y Linux.

Si se usa el modelo de implementación de Azure Resource Manager, puede especificar servidores DNS para una red virtual y una interfaz de red. Para obtener más información, consulte [Administración de una red virtual](manage-virtual-network.md) y [Administración de una interfaz de red](virtual-network-network-interface.md).

> [!NOTE]
> Si opta por el servidor DNS personalizado para la red virtual, debe especificar al menos una dirección IP de servidor DNS; en caso contrario, la red virtual omitirá la configuración y usará en su lugar el DNS proporcionado por Azure.

Cuando se usa el modelo de implementación clásica, puede especificar servidores DNS para la red virtual en Azure Portal o en el [archivo de configuración de red](https://msdn.microsoft.com/library/azure/jj157100). En el caso de los servicios en la nube, puede especificar los servidores DNS mediante el [archivo de configuración del servicio](https://msdn.microsoft.com/library/azure/ee758710) o mediante PowerShell, con [New-AzureVM](/powershell/module/servicemanagement/azure.service/new-azurevm).

> [!NOTE]
> Si cambia la configuración de DNS de una máquina o red virtual que ya está implementada, para que la nueva configuración de DNS surta efecto, debe realizar una renovación de la concesión de DHCP en todas las máquinas virtuales afectadas de la red virtual. En el caso de las máquinas virtuales que ejecutan el sistema operativo Windows, puede hacerlo escribiendo `ipconfig /renew` directamente en la máquina virtual. Los pasos varían en función del sistema operativo. Consulte la documentación correspondiente para su tipo de sistema operativo.

## <a name="next-steps"></a>Pasos siguientes

Modelo de implementación de Azure Resource Manager:

* [Administración de una red virtual](manage-virtual-network.md)
* [Administración de una interfaz de red](virtual-network-network-interface.md)

Modelo de implementación clásico:

* [Esquema de configuración del servicio de Azure](https://msdn.microsoft.com/library/azure/ee758710)
* [Esquema de configuración de Virtual Network](https://msdn.microsoft.com/library/azure/jj157100)
* [Configuración de Virtual Network con un archivo de configuración de red](virtual-networks-using-network-configuration-file.md)

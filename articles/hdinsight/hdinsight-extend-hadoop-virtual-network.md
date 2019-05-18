---
title: Extensión de HDInsight con Virtual Network en Azure
description: Aprenda a usar Azure Virtual Network para conectar HDInsight con otros recursos en la nube o recursos en su centro de datos.
author: hrasheed-msft
ms.author: hrasheed
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 03/29/2019
ms.openlocfilehash: e586ab1bdcca9d6109cf42b6341c333fabb02993
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65601668"
---
# <a name="extend-azure-hdinsight-using-an-azure-virtual-network"></a>Extender Azure HDInsight mediante una instancia de Azure Virtual Network

Obtenga información sobre cómo usar HDInsight con una instancia de [Azure Virtual Network](../virtual-network/virtual-networks-overview.md). El uso de una instancia de Azure Virtual Network permite los siguientes escenarios:

* Conectarse a HDInsight directamente desde una red local.

* Conectar HDInsight a almacenes de datos en una instancia de Azure Virtual Network.

* Obtener acceso directo a los servicios de [Apache Hadoop](https://hadoop.apache.org/) que no están disponibles públicamente en Internet. Por ejemplo, las API [Apache Kafka](https://kafka.apache.org/) o la API [Apache HBase](https://hbase.apache.org/) de Java.

> [!IMPORTANT]  
> Después del 28 de febrero de 2019, los recursos de red (por ejemplo, NIC, LBs, etcetera) para los nuevos clústeres creados en una red virtual se aprovisionará en el mismo grupo de recursos de clúster de HDInsight. Anteriormente, estos recursos se aprovisionaron en el grupo de recursos de red virtual. No hay ningún cambio en los clústeres de ejecución actuales y los clústeres creados sin una red virtual.

## <a name="prerequisites-for-code-samples-and-examples"></a>Requisitos previos para los ejemplos de código y ejemplos

* Comprensión de las redes TCP/IP. Si no está familiarizado con las redes TCP/IP, debe asociarse con alguien que lo esté antes de realizar modificaciones a las redes de producción.
* Si usa PowerShell, necesitará el [módulo AZ](https://docs.microsoft.com/powershell/azure/overview).
* Si desean usar la CLI de Azure y aún no lo ha instalado, consulte [instalar la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli).

> [!IMPORTANT]  
> Si busca guías paso a paso sobre cómo conectar HDInsight con la red local mediante una instancia de Azure Virtual Network, consulte el documento [Conexión de HDInsight a la red local](connect-on-premises-network.md).

## <a name="planning"></a>Planificación

Debe responder a las preguntas siguientes cuando planee instalar HDInsight en una red virtual:

* ¿Necesita instalar HDInsight en una red virtual existente? ¿O bien está creando una red nueva?

    Si está usando una red virtual existente, puede que tenga que modificar la configuración de red antes de poder instalar HDInsight. Para más información, vea la sección [Agregar HDInsight a una red virtual existente](#existingvnet).

* ¿Quiere conectar la red virtual que contiene HDInsight a otra red virtual o a la red local?

    Para trabajar fácilmente con recursos entre redes, puede que tenga que crear un DNS personalizado y configurar el reenvío de DNS. Para más información, vea la sección [Conexión de varias redes](#multinet).

* ¿Quiere restringir o redirigir el tráfico de entrada o salida a HDInsight?

    HDInsight debe tener comunicación sin restricciones con direcciones IP específicas en el centro de datos de Azure. También hay varios puertos que deben permitirse a través de los firewalls para la comunicación del cliente. Para más información, vea la sección [Control del tráfico de red](#networktraffic).

## <a id="existingvnet"></a>Agregar HDInsight una red virtual existente

Use los pasos de esta sección para saber cómo agregar un nuevo HDInsight a una instancia de Azure Virtual Network existente.

> [!NOTE]  
> No se puede agregar un clúster de HDInsight existente a una red virtual.

1. ¿Está usando un modelo de implementación clásico o de Resource Manager para la red virtual?

    HDInsight 3.4 y versiones posteriores requieren una red virtual de Resource Manager. Las versiones anteriores de HDInsight requieren una red virtual clásica.

    Si la red existente es una red virtual clásica, debe crear una red virtual de Resource Manager y después conectar las dos. [Conexión de redes virtuales clásicas a redes virtuales nuevas](../vpn-gateway/vpn-gateway-connect-different-deployment-models-portal.md).

    Una vez combinadas, HDInsight instalado en la red de Resource Manager puede interactuar con los recursos de la red clásica.

2. ¿Usa tunelización forzada? La tunelización forzada es una configuración de subred que fuerza el tráfico de salida de Internet a un dispositivo para la inspección y el registro. HDInsight no es compatible con la tunelización forzada. Quite la tunelización forzada antes de implementar HDInsight en una subred existente, o bien cree una subred para HDInsight sin ninguna tunelización forzada.

3. ¿Usa grupos de seguridad de red, rutas definidas por el usuario o dispositivos de red virtual para restringir el tráfico de entrada y salida de la red virtual?

    Como servicio administrado, HDInsight requiere acceso sin restricciones a varias direcciones IP en el centro de datos de Azure. Para permitir la comunicación con estas direcciones IP, actualice las rutas definidas por el usuario o los grupos de seguridad de red existentes.
    
    HDInsight hospeda varios servicios, que usan varios puertos. No bloquee el tráfico a estos puertos. Para obtener una lista de puertos que se pueden permitir mediante los firewalls de los dispositivos virtuales, consulte la sección Seguridad.
    
    Para buscar la configuración de seguridad existente, use los siguientes comandos de Azure PowerShell o de la CLI de Azure:

    * Grupos de seguridad de red

        Reemplace `RESOURCEGROUP` con el nombre del grupo de recursos que contiene la red virtual y, a continuación, escriba el comando:
    
        ```powershell
        Get-AzNetworkSecurityGroup -ResourceGroupName  "RESOURCEGROUP"
        ```
    
        ```azurecli
        az network nsg list --resource-group RESOURCEGROUP
        ```

        Para más información, vea el documento sobre [Solución de problemas de los grupos de seguridad de red](../virtual-network/diagnose-network-traffic-filter-problem.md).

        > [!IMPORTANT]  
        > Se aplican reglas de grupo de seguridad de red en orden según la prioridad de la regla. Se aplica la primera regla que coincide con el patrón de tráfico y no se aplica ninguna otra para el tráfico. El orden de las reglas es de más a menos permisiva. Para más información, vea el documento [Filtrar el tráfico de red con grupos de seguridad de red](../virtual-network/security-overview.md).

    * Rutas definidas por el usuario

        Reemplace `RESOURCEGROUP` con el nombre del grupo de recursos que contiene la red virtual y, a continuación, escriba el comando:

        ```powershell
        Get-AzRouteTable -ResourceGroupName "RESOURCEGROUP"
        ```

        ```azurecli
        az network route-table list --resource-group RESOURCEGROUP
        ```

        Para más información, vea el documento sobre [Solución de problemas de rutas](../virtual-network/diagnose-network-routing-problem.md).

4. Cree un clúster de HDInsight y seleccione la instancia de Azure Virtual Network durante la configuración. Para entender el proceso de creación de clúster, use los pasos descritos en los siguientes documentos:

    * [Creación de HDInsight con Azure Portal](hdinsight-hadoop-create-linux-clusters-portal.md)
    * [Creación de HDInsight con Azure PowerShell](hdinsight-hadoop-create-linux-clusters-azure-powershell.md)
    * [Creación de clústeres de HDInsight mediante la CLI de Azure clásica](hdinsight-hadoop-create-linux-clusters-azure-cli.md)
    * [Creación de una instancia de HDInsight mediante el uso de una plantilla de Azure Resource Manager](hdinsight-hadoop-create-linux-clusters-arm-templates.md)

   > [!IMPORTANT]  
   > Agregar HDInsight a una red virtual es un paso de configuración opcional. Asegúrese de seleccionar la red virtual al configurar el clúster.

## <a id="multinet"></a>Conectar varias redes

El mayor desafío con una configuración de varias redes es la resolución de nombres entre las redes.

Azure proporciona resolución de nombres para los servicios de Azure instalados en una red virtual. Esta resolución de nombres integrada permite conectar HDInsight a los siguientes recursos mediante un nombre de dominio completo (FQDN):

* Cualquier recurso que está disponible en Internet. Por ejemplo, microsoft.com, windowsupdate.com.

* Cualquier recurso que se encuentre en la misma instancia de Azure Virtual Network, mediante el uso del __nombre DNS interno__ del recurso. Por ejemplo, cuando se usa la resolución de nombres predeterminada, los siguientes son nombres DNS internos de ejemplo que se asignan a nodos de trabajo de HDInsight:

  * wn0-hdinsi.0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net
  * wn2-hdinsi.0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net

    Estos dos nodos pueden comunicarse directamente entre sí y con otros nodos de HDInsight, mediante el uso de nombres DNS internos.

La resolución de nombres predeterminada __no__ permite que HDInsight resuelva los nombres de recursos en redes que se unen a la red virtual. Por ejemplo, es habitual unir la red local a la red virtual. Con solo la resolución de nombres predeterminada, HDInsight no puede tener acceso a los recursos de la red local por su nombre. Lo contrario también es cierto, los recursos de la red local no pueden tener acceso a recursos de la red virtual por su nombre.

> [!WARNING]  
> Debe crear el servidor DNS personalizado y configurar la red virtual para usarla antes de crear el clúster de HDInsight.

Para habilitar la resolución de nombres entre la red virtual y los recursos en redes combinadas, debe realizar las siguientes acciones:

1. Cree un servidor DNS personalizado en la instancia de Azure Virtual Network en la que planea instalar HDInsight.

2. Configure la red virtual para usar el servidor DNS personalizado.

3. Busque el sufijo DNS que Azure asigna a la red virtual. Este valor es similar a `0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net`. Para información sobre cómo buscar el sufijo DNS, consulte la sección [Ejemplo: DNS personalizado](#example-dns).

4. Configure el reenvío entre los servidores DNS. La configuración depende del tipo de red remota.

   * Si la red remota es una red local, configure DNS como sigue:
        
     * __DNS personalizado__ (en la red virtual):

         * Reenvíe las solicitudes para el sufijo DNS de la red virtual a la resolución recursiva de Azure (168.63.129.16). Azure administra las solicitudes de recursos en la red virtual.

         * Reenvíe las demás solicitudes al servidor DNS local. El servidor DNS local administra todas las solicitudes de resolución de nombres, incluso las de recursos de Internet como Microsoft.com.

     * __DNS local__: reenvíe las solicitudes para el sufijo DNS de red virtual al servidor DNS personalizado. El servidor DNS personalizado reenvía a la resolución recursiva de Azure.

       Esta configuración dirige las solicitudes de nombres de dominio completos que contienen el sufijo DNS de la red virtual al servidor DNS personalizado. Todas las demás solicitudes (incluso para las direcciones públicas de Internet) se controlan mediante el servidor DNS local.

   * Si la red remota es otra instancia de Azure Virtual Network, configure DNS como sigue:

     * __DNS personalizado__ (en cada red virtual):

         * Las solicitudes para el sufijo DNS de las redes virtuales se reenvían a los servidores DNS personalizados. El DNS de cada red virtual es el responsable de resolver los recursos dentro de su red.

         * Todas las demás solicitudes se reenvían a la resolución recursiva de Azure. La resolución recursiva es responsable de resolver los recursos locales y de Internet.

       El servidor DNS de cada red reenvía las solicitudes al otro, según el sufijo DNS. Las demás solicitudes se resuelven mediante la resolución recursiva de Azure.

     Para obtener un ejemplo de cada configuración, consulte la sección [Ejemplo: DNS personalizado](#example-dns).

Para más información, vea el documento [Resolución de nombres para máquinas virtuales e instancias de rol](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md).

## <a name="directly-connect-to-apache-hadoop-services"></a>Conexión directa a los servicios de Apache Hadoop

Puede conectarse al clúster en `https://CLUSTERNAME.azurehdinsight.net`. Esta dirección usa una dirección IP pública, que podría no ser accesible en caso de haber usado grupos de seguridad de red para restringir el tráfico entrante de Internet. Además, al implementar el clúster en una red virtual puede acceder a ella mediante el punto de conexión privado `https://CLUSTERNAME-int.azurehdinsight.net`. Este punto de conexión se resuelve en una dirección IP privada dentro de la red virtual para el acceso al clúster.

Para conectarse a Apache Ambari y otras páginas web a través de la red virtual, siga estos pasos:

1. Para detectar los nombres de dominio completos (FQDN) internos de los nodos de clúster de HDInsight, use uno de los métodos siguientes:

    Reemplace `RESOURCEGROUP` con el nombre del grupo de recursos que contiene la red virtual y, a continuación, escriba el comando:

    ```powershell
    $clusterNICs = Get-AzNetworkInterface -ResourceGroupName "RESOURCEGROUP" | where-object {$_.Name -like "*node*"}

    $nodes = @()
    foreach($nic in $clusterNICs) {
        $node = new-object System.Object
        $node | add-member -MemberType NoteProperty -name "Type" -value $nic.Name.Split('-')[1]
        $node | add-member -MemberType NoteProperty -name "InternalIP" -value $nic.IpConfigurations.PrivateIpAddress
        $node | add-member -MemberType NoteProperty -name "InternalFQDN" -value $nic.DnsSettings.InternalFqdn
        $nodes += $node
    }
    $nodes | sort-object Type
    ```

    ```azurecli
    az network nic list --resource-group RESOURCEGROUP --output table --query "[?contains(name,'node')].{NICname:name,InternalIP:ipConfigurations[0].privateIpAddress,InternalFQDN:dnsSettings.internalFqdn}"
    ```

    En la lista de nodos que se devuelve, busque el FQDN de los nodos principales y use los FQDN para conectarse a Ambari y otros servicios web. Por ejemplo, use `http://<headnode-fqdn>:8080` para tener acceso a Ambari.

    > [!IMPORTANT]  
    > Algunos servicios hospedados en los nodos principales solo están activos en un nodo a la vez. Si intenta obtener acceso a un servicio en un nodo principal y se devuelve un error 404, cambie al otro nodo principal.

2. Para determinar el nodo y el puerto en que está disponible un servicio, vea el documento [Puertos utilizados por los servicios Hadoop en HDInsight](./hdinsight-hadoop-port-settings-for-services.md).

## <a id="networktraffic"></a> Control del tráfico de red

El tráfico de red en instancias de Azure Virtual Network se puede controlar mediante los métodos siguientes:

* Los **Grupos de seguridad de red** (NSG) permiten filtrar el tráfico de entrada y salida de la red. Para más información, vea el documento [Filtrar el tráfico de red con grupos de seguridad de red](../virtual-network/security-overview.md).

    > [!WARNING]  
    > HDInsight no admite la restricción de tráfico saliente. Se debe permitir todo el tráfico saliente.

* Las **Rutas definidas por el usuario** (UDR) definen cómo fluye el tráfico entre los recursos de la red. Para más información, vea el documento [Rutas definidas por el usuario y reenvío IP](../virtual-network/virtual-networks-udr-overview.md).

* Los **Dispositivos de red virtual** replican las funciones de dispositivos como enrutadores y firewalls. Para más información, vea el documento [Dispositivos de red](https://azure.microsoft.com/solutions/network-appliances).

Como un servicio administrado, HDInsight requiere acceso sin restricciones para el estado de HDInsight y administración de servicios para el tráfico entrante y saliente de la red virtual. Al usar grupos de seguridad de red y rutas definidas por el usuario, debe asegurarse de que estos servicios pueden seguir comunicándose con el clúster de HDInsight.

### <a id="hdinsight-ip"></a> HDInsight con grupos de seguridad de red y rutas definidas por el usuario

Si planea usar **grupos de seguridad de red** o **rutas definidas por el usuario** para controlar el tráfico de red, realice las siguientes acciones antes de instalar HDInsight:

1. Identificar la región de Azure que va a usar para HDInsight.

2. Identificar las direcciones IP requeridas para HDInsight. Para más información, vea la sección [Direcciones IP requeridas por HDInsight](#hdinsight-ip).

3. Cree o modifique los grupos de seguridad de red o las rutas definidas por el usuario para la subred en la que tiene previsto instalar HDInsight.

    * __Grupos de seguridad de red__: permita tráfico de __entrada__ en el puerto __443__ desde las direcciones IP. Esto garantizará que los servicios de administración de HDI puedan comunicarse con el clúster desde fuera de la red virtual.
    * __Rutas definidas por el usuario__: si pretende usar rutas definidas por el usuario, cree una ruta para cada dirección IP y establezca el __Tipo del próximo salto__ en __Internet__. También debe permitir el tráfico saliente de la red virtual sin restricciones. Por ejemplo, puede enrutar todo el tráfico a la red o firewall de aplicación virtual de Azure (hospedado en Azure) con fines de supervisión, pero no se debe bloquear el tráfico saliente.

Para más información sobre grupos de seguridad de red o rutas definidas por el usuario, vea la documentación siguiente:

* [Grupo de seguridad de red](../virtual-network/security-overview.md)

* [Rutas definidas por el usuario](../virtual-network/virtual-networks-udr-overview.md)

#### <a name="forced-tunneling-to-on-premise"></a>Tunelización forzada a una ubicación local

La tunelización forzada es una configuración de enrutamiento definida por el usuario en la que todo el tráfico de subred se fuerza a una red o ubicación específica, por ejemplo, la red local. HDInsight __no__ soporte técnico de la tunelización forzada a las redes locales. Si usa Firewall de Azure o un dispositivo de red virtual hospedada en Azure, puede usar Udr para enrutar el tráfico a él con fines de supervisión y permitir todo el tráfico saliente.

## <a id="hdinsight-ip"></a> Direcciones IP necesarias

> [!IMPORTANT]  
> Los servicios de mantenimiento y administración de Azure deben poder comunicarse con HDInsight. Si utiliza grupos de seguridad de red o rutas definidas por el usuario, permita el tráfico de las direcciones IP a estos servicios para llegar a HDInsight.
>
> Si no usa grupos de seguridad de red ni rutas definidas por el usuario para controlar el tráfico, puede omitir esta sección.

Si usa grupos de seguridad de red, debe permitir que el tráfico de los servicios de mantenimiento y administración de Azure llegue a los clústeres de HDInsight en el puerto 443. También debe permitir el tráfico entre máquinas virtuales dentro de la subred. Siga los pasos siguientes para buscar las direcciones IP que se deben permitir:

1. Siempre debe permitir el tráfico de las siguientes direcciones IP:

    | Dirección IP de origen | Destino  | Direction |
    | ---- | ----- | ----- |
    | 168.61.49.99 | \*:443 | Entrada |
    | 23.99.5.239 | \*:443 | Entrada |
    | 168.61.48.131 | \*:443 | Entrada |
    | 138.91.141.162 | \*:443 | Entrada |

2. Si el clúster de HDInsight está en una de las siguientes regiones, debe permitir el tráfico de las direcciones IP mostradas para la región:

    > [!IMPORTANT]  
    > Si la región de Azure que está usando no aparece, use únicamente las cuatro direcciones IP del paso 1.

    | País | Área | Direcciones IP de origen permitidas | Destino permitido | Direction |
    | ---- | ---- | ---- | ---- | ----- |
    | Asia | Asia oriental | 23.102.235.122</br>52.175.38.134 | \*:443 | Entrada |
    | &nbsp; | Sudeste asiático | 13.76.245.160</br>13.76.136.249 | \*:443 | Entrada |
    | Australia | Este de Australia | 104.210.84.115</br>13.75.152.195 | \*:443 | Entrada |
    | &nbsp; | Sudeste de Australia | 13.77.2.56</br>13.77.2.94 | \*:443 | Entrada |
    | Brasil | Sur de Brasil | 191.235.84.104</br>191.235.87.113 | \*:443 | Entrada |
    | Canadá | Este de Canadá | 52.229.127.96</br>52.229.123.172 | \*:443 | Entrada |
    | &nbsp; | Centro de Canadá | 52.228.37.66</br>52.228.45.222 |\*: 443 | Entrada |
    | China | Norte de China | 42.159.96.170</br>139.217.2.219</br></br>42.159.198.178</br>42.159.234.157 | \*:443 | Entrada |
    | &nbsp; | Este de China | 42.159.198.178</br>42.159.234.157</br></br>42.159.96.170</br>139.217.2.219 | \*:443 | Entrada |
    | &nbsp; | Norte de China 2 | 40.73.37.141</br>40.73.38.172 | \*:443 | Entrada |
    | &nbsp; | Este de China 2 | 139.217.227.106</br>139.217.228.187 | \*:443 | Entrada |
    | Europa | Europa del Norte | 52.164.210.96</br>13.74.153.132 | \*:443 | Entrada |
    | &nbsp; | Europa occidental| 52.166.243.90</br>52.174.36.244 | \*:443 | Entrada |
    | Francia | Centro de Francia| 20.188.39.64</br>40.89.157.135 | \*:443 | Entrada |
    | Alemania | Centro de Alemania | 51.4.146.68</br>51.4.146.80 | \*:443 | Entrada |
    | &nbsp; | Noreste de Alemania | 51.5.150.132</br>51.5.144.101 | \*:443 | Entrada |
    | India | India Central | 52.172.153.209</br>52.172.152.49 | \*:443 | Entrada |
    | &nbsp; | Sur de la India | 104.211.223.67<br/>104.211.216.210 | \*:443 | Entrada |
    | Japón | Este de Japón | 13.78.125.90</br>13.78.89.60 | \*:443 | Entrada |
    | &nbsp; | Oeste de Japón | 40.74.125.69</br>138.91.29.150 | \*:443 | Entrada |
    | Corea | Corea Central | 52.231.39.142</br>52.231.36.209 | \*:433 | Entrada |
    | &nbsp; | Corea del Sur | 52.231.203.16</br>52.231.205.214 | \*:443 | Entrada
    | Reino Unido | Oeste de Reino Unido | 51.141.13.110</br>51.141.7.20 | \*:443 | Entrada |
    | &nbsp; | Sur de Reino Unido 2 | 51.140.47.39</br>51.140.52.16 | \*:443 | Entrada |
    | Estados Unidos | Centro de EE. UU. | 13.67.223.215</br>40.86.83.253 | \*:443 | Entrada |
    | &nbsp; | Este de EE. UU | 13.82.225.233</br>40.71.175.99 | \*:443 | Entrada |
    | &nbsp; | Centro-Norte de EE. UU | 157.56.8.38</br>157.55.213.99 | \*:443 | Entrada |
    | &nbsp; | Centro occidental de EE.UU. | 52.161.23.15</br>52.161.10.167 | \*:443 | Entrada |
    | &nbsp; | Oeste de EE. UU. | 13.64.254.98</br>23.101.196.19 | \*:443 | Entrada |
    | &nbsp; | Oeste de EE. UU. 2 | 52.175.211.210</br>52.175.222.222 | \*:443 | Entrada |

    Para más información sobre las direcciones IP que se van a usar para Azure Government, vea el documento [Azure Government Intelligence + Analytics](https://docs.microsoft.com/azure/azure-government/documentation-government-services-intelligenceandanalytics) (Inteligencia y análisis de Azure Government).

3. También debe permitir el acceso desde __168.63.129.16__. Esta es la dirección de la resolución recursiva de Azure. Para más información, vea el documento [Resolución de nombres para las máquinas virtuales e instancias de rol](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md).

Para más información, vea la sección [Control del tráfico de red](#networktraffic).

Si está usando rutas definidas por el usuario (UDR), debe especificar una ruta y permitir el tráfico saliente desde la red virtual a las IP anteriores, con el siguiente salto configurado en "Internet".
    
## <a id="hdinsight-ports"></a> Puertos necesarios

Si planea usar un **firewall** para obtener acceso al clúster fuera de determinados puertos, debe permitir el tráfico en esos puertos, ya que son necesarios para su escenario. De manera predeterminada, no se necesita una lista blanca especial de puertos, siempre que el tráfico de administración de Azure que se explica en la sección anterior tenga permitido llegar al clúster en el puerto 443.

Para una lista de puertos para servicios específicos, consulte el documento [Puertos utilizados por los servicios Apache Hadoop en HDInsight](hdinsight-hadoop-port-settings-for-services.md).

Para más información sobre las reglas de firewall para aplicaciones virtuales, vea el documento [Escenario de aplicación virtual](../virtual-network/virtual-network-scenario-udr-gw-nva.md).

## <a id="hdinsight-nsg"></a>Ejemplo: grupos de seguridad de red con HDInsight

En los ejemplos de esta sección se muestra cómo crear reglas de grupo de seguridad de red que permiten a HDInsight comunicarse con los servicios de administración de Azure. Antes de usar los ejemplos, ajuste las direcciones IP para que coincidan con las de la región de Azure que esté usando. Puede encontrar esta información en la sección [HDInsight con grupos de seguridad de red y rutas definidas por el usuario](#hdinsight-ip).

### <a name="azure-resource-management-template"></a>Plantilla de administración de recursos de Azure

La siguiente plantilla de administración de recursos crea una red virtual que restringe el tráfico de entrada pero permite el tráfico desde las direcciones IP requeridas por HDInsight. Esta plantilla crea también un clúster de HDInsight en la red virtual.

* [Implementar una instancia segura de Azure Virtual Network y un clúster de Hadoop de HDInsight](https://azure.microsoft.com/resources/templates/101-hdinsight-secure-vnet/)

> [!IMPORTANT]  
> Cambie las direcciones IP que se usan en este ejemplo para que coincidan con la región de Azure que está usando. Puede encontrar esta información en la sección [HDInsight con grupos de seguridad de red y rutas definidas por el usuario](#hdinsight-ip).

### <a name="azure-powershell"></a>Azure PowerShell

Use el siguiente script de PowerShell para crear una red virtual que restringe el tráfico de entrada y permite el tráfico de las direcciones IP para la región de Europa del Norte.

> [!IMPORTANT]  
> Cambie las direcciones IP que se usan en este ejemplo para que coincidan con la región de Azure que está usando. Puede encontrar esta información en la sección [HDInsight con grupos de seguridad de red y rutas definidas por el usuario](#hdinsight-ip).

```powershell
$vnetName = "Replace with your virtual network name"
$resourceGroupName = "Replace with the resource group the virtual network is in"
$subnetName = "Replace with the name of the subnet that you plan to use for HDInsight"
# Get the Virtual Network object
$vnet = Get-AzVirtualNetwork `
    -Name $vnetName `
    -ResourceGroupName $resourceGroupName
# Get the region the Virtual network is in.
$location = $vnet.Location
# Get the subnet object
$subnet = $vnet.Subnets | Where-Object Name -eq $subnetName
# Create a Network Security Group.
# And add exemptions for the HDInsight health and management services.
$nsg = New-AzNetworkSecurityGroup `
    -Name "hdisecure" `
    -ResourceGroupName $resourceGroupName `
    -Location $location `
    | Add-AzNetworkSecurityRuleConfig `
        -name "hdirule1" `
        -Description "HDI health and management address 52.164.210.96" `
        -Protocol "*" `
        -SourcePortRange "*" `
        -DestinationPortRange "443" `
        -SourceAddressPrefix "52.164.210.96" `
        -DestinationAddressPrefix "VirtualNetwork" `
        -Access Allow `
        -Priority 300 `
        -Direction Inbound `
    | Add-AzNetworkSecurityRuleConfig `
        -Name "hdirule2" `
        -Description "HDI health and management 13.74.153.132" `
        -Protocol "*" `
        -SourcePortRange "*" `
        -DestinationPortRange "443" `
        -SourceAddressPrefix "13.74.153.132" `
        -DestinationAddressPrefix "VirtualNetwork" `
        -Access Allow `
        -Priority 301 `
        -Direction Inbound `
    | Add-AzNetworkSecurityRuleConfig `
        -Name "hdirule3" `
        -Description "HDI health and management 168.61.49.99" `
        -Protocol "*" `
        -SourcePortRange "*" `
        -DestinationPortRange "443" `
        -SourceAddressPrefix "168.61.49.99" `
        -DestinationAddressPrefix "VirtualNetwork" `
        -Access Allow `
        -Priority 302 `
        -Direction Inbound `
    | Add-AzNetworkSecurityRuleConfig `
        -Name "hdirule4" `
        -Description "HDI health and management 23.99.5.239" `
        -Protocol "*" `
        -SourcePortRange "*" `
        -DestinationPortRange "443" `
        -SourceAddressPrefix "23.99.5.239" `
        -DestinationAddressPrefix "VirtualNetwork" `
        -Access Allow `
        -Priority 303 `
        -Direction Inbound `
    | Add-AzNetworkSecurityRuleConfig `
        -Name "hdirule5" `
        -Description "HDI health and management 168.61.48.131" `
        -Protocol "*" `
        -SourcePortRange "*" `
        -DestinationPortRange "443" `
        -SourceAddressPrefix "168.61.48.131" `
        -DestinationAddressPrefix "VirtualNetwork" `
        -Access Allow `
        -Priority 304 `
        -Direction Inbound `
    | Add-AzNetworkSecurityRuleConfig `
        -Name "hdirule6" `
        -Description "HDI health and management 138.91.141.162" `
        -Protocol "*" `
        -SourcePortRange "*" `
        -DestinationPortRange "443" `
        -SourceAddressPrefix "138.91.141.162" `
        -DestinationAddressPrefix "VirtualNetwork" `
        -Access Allow `
        -Priority 305 `
        -Direction Inbound `
# Set the changes to the security group
Set-AzNetworkSecurityGroup -NetworkSecurityGroup $nsg
# Apply the NSG to the subnet
Set-AzVirtualNetworkSubnetConfig `
    -VirtualNetwork $vnet `
    -Name $subnetName `
    -AddressPrefix $subnet.AddressPrefix `
    -NetworkSecurityGroup $nsg
$vnet | Set-AzVirtualNetwork
```

> [!IMPORTANT]  
> En este ejemplo se muestra cómo agregar reglas para permitir el tráfico de entrada en las direcciones IP necesarias. No contiene una regla para restringir el acceso de entrada desde otros orígenes.
>
> En el siguiente ejemplo se muestra cómo habilitar el acceso de SSH desde Internet:
>
> ```powershell
> Add-AzNetworkSecurityRuleConfig -Name "SSH" -Description "SSH" -Protocol "*" -SourcePortRange "*" -DestinationPortRange "22" -SourceAddressPrefix "*" -DestinationAddressPrefix "VirtualNetwork" -Access Allow -Priority 306 -Direction Inbound
> ```

### <a name="azure-cli"></a>Azure CLI

Use los pasos siguientes para crear una red virtual que restringe el tráfico de entrada pero permite el tráfico desde las direcciones IP requeridas por HDInsight.

1. Use el siguiente comando para crear un nuevo grupo de seguridad de red denominado " `hdisecure`". Reemplace `RESOURCEGROUP` con el grupo de recursos que contiene la red Virtual de Azure. Reemplace `LOCATION` con la ubicación (región) que se creó el grupo en.

    ```azurecli
    az network nsg create -g RESOURCEGROUP -n hdisecure -l LOCATION
    ```

    Cuando se haya creado el grupo, recibirá información sobre el nuevo grupo.

2. Utilice lo siguiente para agregar reglas al nuevo grupo de seguridad de red que permitan la comunicación entrante en el puerto 443 desde el servicio de mantenimiento y administración de HDInsight de Azure. Reemplace `RESOURCEGROUP` con el nombre del grupo de recursos que contiene la red Virtual de Azure.

    > [!IMPORTANT]  
    > Cambie las direcciones IP que se usan en este ejemplo para que coincidan con la región de Azure que está usando. Puede encontrar esta información en la sección [HDInsight con grupos de seguridad de red y rutas definidas por el usuario](#hdinsight-ip).

    ```azurecli
    az network nsg rule create -g RESOURCEGROUP --nsg-name hdisecure -n hdirule1 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "52.164.210.96" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 300 --direction "Inbound"
    az network nsg rule create -g RESOURCEGROUP --nsg-name hdisecure -n hdirule2 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "13.74.153.132" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 301 --direction "Inbound"
    az network nsg rule create -g RESOURCEGROUP --nsg-name hdisecure -n hdirule3 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "168.61.49.99" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 302 --direction "Inbound"
    az network nsg rule create -g RESOURCEGROUP --nsg-name hdisecure -n hdirule4 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "23.99.5.239" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 303 --direction "Inbound"
    az network nsg rule create -g RESOURCEGROUP --nsg-name hdisecure -n hdirule5 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "168.61.48.131" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 304 --direction "Inbound"
    az network nsg rule create -g RESOURCEGROUP --nsg-name hdisecure -n hdirule6 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "138.91.141.162" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 305 --direction "Inbound"
    ```

3. Para recuperar el identificador único para este grupo de seguridad de red, use el comando siguiente:

    ```azurecli
    az network nsg show -g RESOURCEGROUP -n hdisecure --query 'id'
    ```

    Este comando devuelve un valor similar al siguiente texto:

        "/subscriptions/SUBSCRIPTIONID/resourceGroups/RESOURCEGROUP/providers/Microsoft.Network/networkSecurityGroups/hdisecure"

    Utilice comillas dobles alrededor de `id` en el comando si no obtiene los resultados esperados.

4. Use el siguiente comando para aplicar el grupo de seguridad de red a una subred. Reemplace el `GUID` y `RESOURCEGROUP` valores con los que se devuelven desde el paso anterior. Reemplace `VNETNAME` y `SUBNETNAME` con el nombre de red virtual y subred que desea crear.

    ```azurecli
    az network vnet subnet update -g RESOURCEGROUP --vnet-name VNETNAME --name SUBNETNAME --set networkSecurityGroup.id="/subscriptions/GUID/resourceGroups/RESOURCEGROUP/providers/Microsoft.Network/networkSecurityGroups/hdisecure"
    ```

    Una vez completado este comando, puede instalar HDInsight en la red virtual.

> [!IMPORTANT]  
> Con estos pasos solo se abre el acceso al servicio de mantenimiento y administración de HDInsight en Azure Cloud. Cualquier otro acceso al clúster de HDInsight desde fuera de Virtual Network está bloqueado. Para permitir el acceso desde fuera de la red virtual, debe agregar reglas adicionales del grupo de seguridad de red.
>
> En el siguiente ejemplo se muestra cómo habilitar el acceso de SSH desde Internet:
>
> ```azurecli
> az network nsg rule create -g RESOURCEGROUP --nsg-name hdisecure -n hdirule5 --protocol "*" --source-port-range "*" --destination-port-range "22" --source-address-prefix "*" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 306 --direction "Inbound"
> ```

## <a id="example-dns"></a> Ejemplo: Configuración de DNS

### <a name="name-resolution-between-a-virtual-network-and-a-connected-on-premises-network"></a>Resolución de nombres entre una red virtual y una red local conectada

En este ejemplo se da por supuesto lo siguiente:

* Dispone de una instancia de Azure Virtual Network que está conectada a una red local mediante una puerta de enlace de VPN.

* El servidor DNS personalizado en la red virtual ejecuta Linux o Unix como sistema operativo.

* [Bind](https://www.isc.org/downloads/bind/) está instalado en el servidor DNS personalizado.

En el servidor DNS en la red virtual:

1. Use Azure PowerShell o la CLI de Azure para encontrar el sufijo DNS de la red virtual:

    Reemplace `RESOURCEGROUP` con el nombre del grupo de recursos que contiene la red virtual y, a continuación, escriba el comando:

    ```powershell
    $NICs = Get-AzNetworkInterface -ResourceGroupName "RESOURCEGROUP"
    $NICs[0].DnsSettings.InternalDomainNameSuffix
    ```

    ```azurecli
    az network nic list --resource-group RESOURCEGROUP --query "[0].dnsSettings.internalDomainNameSuffix"
    ```

2. En el servidor DNS personalizado para la red virtual, use el siguiente texto como el contenido del archivo `/etc/bind/named.conf.local`:

    ```
    // Forward requests for the virtual network suffix to Azure recursive resolver
    zone "0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net" {
        type forward;
        forwarders {168.63.129.16;}; # Azure recursive resolver
    };
    ```

    Reemplace el valor `0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net` con el sufijo DNS de la red virtual.

    Esta configuración dirige todas las solicitudes DNS para el sufijo DNS de la red virtual a la resolución recursiva de Azure.

2. En el servidor DNS personalizado para la red virtual, use el siguiente texto como el contenido del archivo `/etc/bind/named.conf.options`:

    ```
    // Clients to accept requests from
    // TODO: Add the IP range of the joined network to this list
    acl goodclients {
        10.0.0.0/16; # IP address range of the virtual network
        localhost;
        localnets;
    };

    options {
            directory "/var/cache/bind";

            recursion yes;

            allow-query { goodclients; };

            # All other requests are sent to the following
            forwarders {
                192.168.0.1; # Replace with the IP address of your on-premises DNS server
            };

            dnssec-validation auto;

            auth-nxdomain no;    # conform to RFC1035
            listen-on { any; };
    };
    ```
    
    * Reemplace el valor `10.0.0.0/16` con el intervalo de direcciones IP de la red virtual. Esta entrada permite direcciones de solicitudes de resolución de nombres dentro de este intervalo.

    * Agregue el intervalo de direcciones IP de la red local a la sección `acl goodclients { ... }`.  Esta entrada permite las solicitudes de resolución de nombres de recursos de la red local.
    
    * Reemplace el valor `192.168.0.1` por la dirección IP del servidor DNS local. Esta entrada enruta todas las solicitudes DNS al servidor DNS local.

3. Para usar la configuración, reinicie Bind. Por ejemplo, `sudo service bind9 restart`.

4. Agregue un reenviador condicional al servidor DNS local. Configure el reenviador condicional para poder enviar solicitudes para el sufijo DNS del paso 1 al servidor DNS personalizado.

    > [!NOTE]  
    > Consulte la documentación del software de DNS para obtener información específica sobre cómo agregar un reenviador condicional.

Después de completar estos pasos, puede conectarse a los recursos de cualquiera de las redes mediante nombres de dominio completo (FQDN). Ahora puede instalar HDInsight en la red virtual.

### <a name="name-resolution-between-two-connected-virtual-networks"></a>Resolución de nombres entre dos redes virtuales conectadas

En este ejemplo se da por supuesto lo siguiente:

* Tiene dos instancias de Azure Virtual Network que se conectan mediante una puerta de enlace de VPN o un emparejamiento.

* El servidor DNS personalizado en ambas redes ejecuta Linux o Unix como sistema operativo.

* [Bind](https://www.isc.org/downloads/bind/) está instalado en los servidores DNS personalizados.

1. Use Azure PowerShell o la CLI de Azure para buscar el sufijo DNS de las dos redes virtuales:

    Reemplace `RESOURCEGROUP` con el nombre del grupo de recursos que contiene la red virtual y, a continuación, escriba el comando:

    ```powershell
    $NICs = Get-AzNetworkInterface -ResourceGroupName "RESOURCEGROUP"
    $NICs[0].DnsSettings.InternalDomainNameSuffix
    ```

    ```azurecli
    az network nic list --resource-group RESOURCEGROUP --query "[0].dnsSettings.internalDomainNameSuffix"
    ```

2. Use el siguiente texto como contenido del archivo `/etc/bind/named.config.local` en el servidor DNS personalizado. Realice el cambio en el servidor DNS personalizado de las dos redes virtuales.

    ```
    // Forward requests for the virtual network suffix to Azure recursive resolver
    zone "0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net" {
        type forward;
        forwarders {10.0.0.4;}; # The IP address of the DNS server in the other virtual network
    };
    ```

    Reemplace el valor `0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net` con el sufijo DNS de la __otra__ red virtual. Esta entrada enruta las solicitudes para el sufijo DNS de la red remota al DNS personalizado en esa red.

3. En los servidores DNS personalizados de las dos redes virtuales, use el siguiente texto como contenido del archivo `/etc/bind/named.conf.options`:

    ```
    // Clients to accept requests from
    acl goodclients {
        10.1.0.0/16; # The IP address range of one virtual network
        10.0.0.0/16; # The IP address range of the other virtual network
        localhost;
        localnets;
    };

    options {
            directory "/var/cache/bind";

            recursion yes;

            allow-query { goodclients; };

            forwarders {
            168.63.129.16;   # Azure recursive resolver         
            };

            dnssec-validation auto;

            auth-nxdomain no;    # conform to RFC1035
            listen-on { any; };
    };
    ```
    
   Reemplace los valores `10.0.0.0/16` y `10.1.0.0/16` con el intervalo de direcciones IP de las redes virtuales. Esta entrada permite que los recursos de cada red realicen solicitudes de los servidores DNS.

    Cualquier solicitud que no sea para los sufijos DNS de las redes virtuales (por ejemplo, microsoft.com) se administra mediante la resolución recursiva de Azure.

4. Para usar la configuración, reinicie Bind. Por ejemplo, `sudo service bind9 restart` en ambos servidores DNS.

Después de completar estos pasos, puede conectarse a recursos en la red virtual mediante nombres de dominio completo (FQDN). Ahora puede instalar HDInsight en la red virtual.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener un ejemplo integral de la configuración de HDInsight para conectarse a una red local, vea [Conexión de HDInsight a la red local](./connect-on-premises-network.md).
* Para configurar clústeres de Apache Hbase en las redes virtuales de Azure, consulte [Creación de clústeres de Apache HBase en HDInsight en Azure Virtual Network](hbase/apache-hbase-provision-vnet.md).
* Para configurar la replicación geográfica de Apache HBase, consulte [Configuración de la replicación de clúster de Apache HBase en redes virtuales de Azure](hbase/apache-hbase-replication.md).
* Para más información sobre las redes virtuales de Azure, vea la [información general sobre Azure Virtual Network](../virtual-network/virtual-networks-overview.md).

* Para más información sobre los grupos de seguridad de red, vea [Grupos de seguridad de red](../virtual-network/security-overview.md).

* Para obtener más información sobre las rutas definidas por el usuario, consulte [Rutas definidas por el usuario y reenvío de IP](../virtual-network/virtual-networks-udr-overview.md).

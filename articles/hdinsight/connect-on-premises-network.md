---
title: Conexión de Azure HDInsight a la red local
description: Obtenga información sobre cómo crear un clúster de HDInsight en una instancia de Azure Virtual Network y conectarlo a la red local. Obtenga información sobre cómo usar un servidor DNS personalizado para configurar la resolución de nombres entre HDInsight y la red local.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 03/04/2020
ms.openlocfilehash: 3ab706b9cdf3c071fd5d3ceca732cff6b660db6b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87086564"
---
# <a name="connect-hdinsight-to-your-on-premises-network"></a>Conexión de HDInsight a la red local

Obtenga información sobre cómo conectar HDInsight a la red local mediante instancias de Azure Virtual Network y una instancia de VPN Gateway. En este documento se brinda información de planeación sobre:

* Uso de HDInsight en una instancia de Azure Virtual Network que se conecta con la red local.
* Configuración de resolución de nombres DNS entre la red virtual y la red local.
* Configuración de grupos de seguridad de red para restringir acceso a HDInsight a través de Internet.
* Puertos que HDInsight proporciona en la red virtual.

## <a name="overview"></a>Información general

Para permitir que HDInsight y los recursos de la red combinada se comuniquen por nombre, debe realizar las acciones siguientes:

1. Cree una instancia de Azure Virtual Network.
1. Cree un servidor DNS personalizado en la instancia de Azure Virtual Network.
1. Configure la red virtual para que use el servidor DNS personalizado en lugar de la resolución recursiva de Azure predeterminada.
1. Configure el reenvío entre el servidor DNS personalizado y el servidor DNS local.

Estas configuraciones permiten que se produzca el siguiente comportamiento:

* Las solicitudes de nombres de dominio completos con el sufijo DNS __para la red virtual__ se reenvían al servidor DNS personalizado. Entonces, el servidor DNS personalizado reenvía estas solicitudes a la resolución recursiva de Azure, que devuelve la dirección IP.
* Todas las otras solicitudes se reenvían al servidor DNS local. Incluso las solicitudes de recursos de Internet, como microsoft.com, se reenvían al servidor DNS local para la resolución de nombres.

En el diagrama siguiente, las líneas verdes son solicitudes de recursos que finalizan en el sufijo DNS de la red virtual. Las líneas azules son solicitudes de recursos en la red local o en Internet.

![Diagrama de cómo se resuelven las solicitudes DNS en la configuración](./media/connect-on-premises-network/on-premises-to-cloud-dns.png)

## <a name="prerequisites"></a>Prerrequisitos

* Un cliente SSH. Para más información, consulte [Conexión a través de SSH con HDInsight (Apache Hadoop)](./hdinsight-hadoop-linux-use-ssh-unix.md).
* Si usa PowerShell, necesitará el [Módulo AZ](https://docs.microsoft.com/powershell/azure/).
* Si quiere usar la CLI de Azure y todavía no la ha instalado, consulte [Instalación de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli).

## <a name="create-virtual-network-configuration"></a>Creación de la configuración de la red virtual

Use los documentos siguientes para obtener información sobre cómo crear una instancia de Azure Virtual Network conectada a la red local:

* [Uso de Azure Portal](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md)
* [Uso de Azure PowerShell](../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md)
* [Uso de la CLI de Azure](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-cli.md)

## <a name="create-custom-dns-server"></a>Creación de un servidor DNS personalizado

> [!IMPORTANT]  
> Debe crear y configurar el servidor DNS antes de instalar HDInsight en la red virtual.

Los pasos siguientes usan [Azure Portal](https://portal.azure.com) para crear una instancia de Azure Virtual Machine. Para conocer otras formas de crear una máquina virtual, consulte los documentos [Creación de una máquina virtual: CLI de Azure](../virtual-machines/linux/quick-create-cli.md) y [Creación de una máquina virtual: Azure PowerShell](../virtual-machines/linux/quick-create-powershell.md).  Para crear una máquina virtual Linux que usa el software DNS [Bind](https://www.isc.org/downloads/bind/), use los pasos siguientes:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
  
1. En el menú superior, seleccione **+ Crear un recurso**.

    ![Creación de una máquina virtual Ubuntu](./media/connect-on-premises-network/azure-portal-create-resource.png)

1. Seleccione **Proceso** > **Virtual machine** (Máquina virtual) para ir a la página **Crear una máquina virtual**.

1. Introduzca la siguiente información de la pestaña __Aspectos básicos__:  
  
    | Campo | Value |
    | --- | --- |
    |Subscription |Seleccione una suscripción adecuada.|
    |Resource group |Seleccione el grupo de recursos que contiene la red virtual que ha creado antes.|
    |Nombre de la máquina virtual | Escriba un nombre descriptivo que identifique esta máquina virtual. En este ejemplo se utiliza **DNSProxy**.|
    |Region | Seleccione la misma región que la de la red virtual que creó anteriormente.  No todos los tamaños de máquina virtual están disponibles en todas las regiones.  |
    |Opciones de disponibilidad |  Seleccione el nivel de disponibilidad que desee.  Azure ofrece varias opciones para administrar la disponibilidad y resistencia de las aplicaciones.  Diseñe su solución para que use las máquinas virtuales replicadas en Availability Zones o en conjuntos de disponibilidad para proteger sus aplicaciones y datos de las interrupciones del centro de datos y de los eventos de mantenimiento. En este ejemplo se usa **No se requiere redundancia de la infraestructura**. |
    |Imagen | Déjelo en **Ubuntu Server 18.04 LTS**. |
    |Tipo de autenticación | __Contraseña__ o __Clave pública SSH__: El método de autenticación de la cuenta SSH. Se recomienda usar claves públicas, porque son más seguras. Este ejemplo utiliza **Contraseña**.  Para más información, consulte el documento [Creación y uso de claves SSH para máquinas virtuales Linux](../virtual-machines/linux/mac-create-ssh-keys.md).|
    |Nombre de usuario |Escriba el nombre de usuario del administrador de la máquina virtual.  En el ejemplo se utiliza **sshuser**.|
    |Contraseña o clave pública SSH | El campo disponible lo determina la opción elegida en **Tipo de autenticación**.  Escriba el valor adecuado.|
    |Puertos de entrada públicos|Seleccione **Permitir los puertos seleccionados**. A continuación, seleccione **SSH (22)** en la lista desplegable **Seleccionar puertos de entrada**.|

    ![Configuración básica de la máquina virtual](./media/connect-on-premises-network/virtual-machine-basics.png)

    Deje otras entradas con los valores predeterminados y seleccione la pestaña **Redes**.

4. Introduzca la siguiente información de la pestaña **Redes**:

    | Campo | Value |
    | --- | --- |
    |Virtual network | Seleccione la red virtual que ha creado antes.|
    |Subnet | Seleccione la subred predeterminada de la red virtual que ha creado antes. __No__ seleccione la subred que la puerta de enlace de VPN usa.|
    |Dirección IP pública | Use el valor que se rellena automáticamente.  |

    ![Configuración de red virtual de HDInsight](./media/connect-on-premises-network/virtual-network-settings.png)

    Deje otras entradas con los valores predeterminados y seleccione **Revisar y crear** .

5. En la pestaña **Revisar y crear**, seleccione **Crear** para crear la máquina virtual.

### <a name="review-ip-addresses"></a>Examen de direcciones IP

Una vez que se haya creado la máquina virtual, recibirá una notificación de **Implementación correcta** con un botón **Ir al recurso**.  Seleccione **Ir al recurso** para ir a la nueva máquina virtual.  En la vista predeterminada de la nueva máquina virtual, siga estos pasos para identificar las direcciones IP asociadas:

1. En **Configuración**, seleccione **Propiedades**.

2. Anote los valores de **ETIQUETA CDE NOMBRE DNS O DIRECCIÓN IP PÚBLICA** y **DIRECCIÓN IP PRIVADA** para su uso posterior.

   ![Direcciones IP pública y privada](./media/connect-on-premises-network/virtual-machine-ip-addresses.png)

### <a name="install-and-configure-bind-dns-software"></a>Instalación y configuración de Bind (software DNS)

1. Use SSH para conectarse a la __dirección IP pública__ de la máquina virtual. Reemplace `sshuser` por la cuenta de usuario SSH que especificó cuando creó la máquina virtual. En el ejemplo siguiente, se realiza una conexión a una máquina virtual en 40.68.254.142:

    ```bash
    ssh sshuser@40.68.254.142
    ```

2. Para instalar Bind, use los comandos siguientes en la sesión de SSH:

    ```bash
    sudo apt-get update -y
    sudo apt-get install bind9 -y
    ```

3. Para configurar Bind a fin de reenviar las solicitudes de resolución de nombres al servidor DNS local, use el texto siguiente como contenido del archivo `/etc/bind/named.conf.options`:

    ```DNS Zone file
    acl goodclients {
        10.0.0.0/16; # Replace with the IP address range of the virtual network
        10.1.0.0/16; # Replace with the IP address range of the on-premises network
        localhost;
        localnets;
    };

    options {
            directory "/var/cache/bind";

            recursion yes;

            allow-query { goodclients; };

            forwarders {
            192.168.0.1; # Replace with the IP address of the on-premises DNS server
            };

            dnssec-validation auto;

            auth-nxdomain no;    # conform to RFC1035
            listen-on { any; };
    };
    ```

    > [!IMPORTANT]  
    > Reemplace los valores de la sección `goodclients` por el intervalo de direcciones IP de la red virtual y la red local. En esta sección se definen las direcciones desde las cuales el servidor DNS acepta solicitudes.
    >
    > Reemplace la entrada `192.168.0.1` en la sección `forwarders` por la dirección IP del servidor DNS local. Esta entrada enruta las solicitudes DNS al servidor DNS local para la resolución.

    Para editar el archivo, use el comando siguiente:

    ```bash
    sudo nano /etc/bind/named.conf.options
    ```

    Para guardar el archivo, use __Ctrl+X__, __Y__ y, luego, __Entrar__.

4. En la sesión de SSH, use el comando siguiente:

    ```bash
    hostname -f
    ```

    Este comando devuelve un valor similar al siguiente texto:

    ```output
    dnsproxy.icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net
    ```

    El texto `icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net` está en el __sufijo DNS__ de esta red virtual. Guarde este valor, porque se utiliza más adelante.

5. Para configurar Bind a fin de resolver nombres DNS para recursos dentro de la red virtual, use el texto siguiente como contenido del archivo `/etc/bind/named.conf.local`:

    ```DNS Zone file
    // Replace the following with the DNS suffix for your virtual network
    zone "icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net" {
        type forward;
        forwarders {168.63.129.16;}; # The Azure recursive resolver
    };
    ```

    > [!IMPORTANT]  
    > Debe reemplazar `icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net` por el sufijo DNS que recuperó anteriormente.

    Para editar el archivo, use el comando siguiente:

    ```bash
    sudo nano /etc/bind/named.conf.local
    ```

    Para guardar el archivo, use __Ctrl+X__, __Y__ y, luego, __Entrar__.

6. Para iniciar Bind, use el comando siguiente:

    ```bash
    sudo service bind9 restart
    ```

7. Para comprobar que Bind puede resolver los nombres de recursos en la red local, use los comandos siguientes:

    ```bash
    sudo apt install dnsutils
    nslookup dns.mynetwork.net 10.0.0.4
    ```

    > [!IMPORTANT]  
    > Reemplace `dns.mynetwork.net` por el nombre de dominio completo (FQDN) de un recurso en la red local.
    >
    > Reemplace `10.0.0.4` por la __dirección IP interna__ del servidor DNS personalizado en la red virtual.

    La respuesta es similar al texto siguiente:

    ```output
    Server:         10.0.0.4
    Address:        10.0.0.4#53

    Non-authoritative answer:
    Name:   dns.mynetwork.net
    Address: 192.168.0.4
    ```

## <a name="configure-virtual-network-to-use-the-custom-dns-server"></a>Configuración de la red virtual para usar el servidor DNS personalizado

Para configurar la red virtual para que use el servidor DNS personalizado, en lugar de la resolución recursiva de Azure, use los pasos siguientes en [Azure Portal](https://portal.azure.com):

1. En el menú izquierdo, vaya a **Todos los servicios** > **Redes** > **Redes virtuales**.

2. Seleccione la red virtual en la lista y se abrirá la vista predeterminada de la red virtual.  

3. En la vista predeterminada, en **Configuración**, seleccione **Servidores DNS**.  

4. Seleccione __Personalizar__ y escriba la **DIRECCIÓN IP PRIVADA** del servidor DNS personalizado.

5. Seleccione __Guardar__.  <br />  

    ![Establecimiento del servidor DNS personalizado para la red](./media/connect-on-premises-network/configure-custom-dns.png)

## <a name="configure-on-premises-dns-server"></a>Configuración del servidor DNS local

En la sección anterior, configuró el servidor DNS personalizado para reenviar las solicitudes al servidor DNS local. A continuación, debe configurar el servidor DNS local para reenviar las solicitudes al servidor DNS personalizado.

Para conocer los pasos específicos sobre cómo configurar el servidor DNS, consulte la documentación correspondiente al software de servidor DNS. Busque los pasos sobre cómo configurar un __reenvío condicional__.

Un reenvío condicional solo reenvía solicitudes para un sufijo DNS específico. En este caso, debe configurar un reenvío para el sufijo DNS de la red virtual. Las solicitudes para este sufijo se deben reenviar a la dirección IP del servidor DNS personalizado. 

El texto siguiente es un ejemplo de la configuración de un reenvío condicional para el software DNS **Bind**:

```DNS Zone file
zone "icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net" {
    type forward;
    forwarders {10.0.0.4;}; # The custom DNS server's internal IP address
};
```

Para información sobre cómo usar DNS en **Windows Server 2016**, consulte la documentación de [Add-DnsServerConditionalForwarderZone](https://technet.microsoft.com/itpro/powershell/windows/dnsserver/add-dnsserverconditionalforwarderzone).

Una vez que configure el servidor DNS local, puede usar `nslookup` desde la red local para comprobar que es posible resolver nombres en la red virtual. El ejemplo siguiente 

```bash
nslookup dnsproxy.icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net 196.168.0.4
```

En este ejemplo se usa el servidor DNS local en 196.168.0.4 para resolver el nombre del servidor DNS personalizado. Reemplace la dirección IP por la dirección IP del servidor DNS local. Reemplace la dirección `dnsproxy` por el nombre de dominio completo del servidor DNS personalizado.

## <a name="optional-control-network-traffic"></a>Opcional: control del tráfico de red

Puede usar grupos de seguridad de red (NSG) o rutas definidas por el usuario (UDR) para controlar el tráfico de red. Los NSG permiten filtrar el tráfico de entrada y de salida, además de permitir o denegar el tráfico. Las UDR permiten controlar el flujo del tráfico entre los recursos de la red virtual, Internet y la red local.

> [!WARNING]  
> HDInsight requiere acceso de entrada desde direcciones IP específicas en la nube de Azure y acceso de salida sin restricciones. Cuando use NSG o UDR para controlar el tráfico, debe realizar los pasos siguientes:

1. Encuentre las direcciones IP de la ubicación que contiene la red virtual. Para obtener una lista de las direcciones IP requeridas por ubicación, consulte [Direcciones IP requeridas](./hdinsight-management-ip-addresses.md).

2. Para las direcciones IP identificadas en el paso 1, permita el tráfico entrante desde ellas.

   * Si usa __NSG__: permita el tráfico __entrante__ en el puerto __443__ para las direcciones IP.
   * Si usa __UDR__: Establezca el tipo __Próximo salto__ de la ruta a __Internet__ de las direcciones IP.

Para un ejemplo de cómo usar Azure PowerShell o la CLI de Azure para crear NSG, consulte el documento [Extensión de HDInsight con redes virtuales de Azure](hdinsight-create-virtual-network.md#hdinsight-nsg).

## <a name="create-the-hdinsight-cluster"></a>Creación del clúster de HDInsight

> [!WARNING]  
> Debe configurar el servidor DNS antes de instalar HDInsight en la red virtual.

Use los pasos que aparecen en el documento sobre cómo [crear un clúster de HDInsight mediante Azure Portal de un clúster](./hdinsight-hadoop-create-linux-clusters-portal.md) para crear un clúster de HDInsight.

> [!WARNING]  
> * Durante la creación del clúster, debe elegir la ubicación que contiene la red virtual.
> * En la parte __Configuración avanzada__ de la configuración, debe seleccionar la red virtual y la subred que creó anteriormente.

## <a name="connecting-to-hdinsight"></a>Conexión a HDInsight

En la mayor parte de la documentación sobre HDInsight se supone que tiene acceso al clúster a través de Internet. Por ejemplo, que se puede conectar al clúster en `https://CLUSTERNAME.azurehdinsight.net`. Esta dirección usa la puerta de enlace pública, que no está disponible si ha usado NSG o UDR para restringir el acceso desde Internet.

Alguna documentación también hace referencia a `headnodehost` al conectarse al clúster desde una sesión de SSH. Esta dirección solo está disponible en los nodos de un clúster y no se puede usar en los clientes conectados a través de la red virtual.

Para conectarse directamente a HDInsight a través de la red virtual, use los pasos siguientes:

1. Para detectar los nombres de dominio completos internos de los nodos de clúster de HDInsight, use uno de los métodos siguientes:

    ```powershell
    $resourceGroupName = "The resource group that contains the virtual network used with HDInsight"

    $clusterNICs = Get-AzNetworkInterface -ResourceGroupName $resourceGroupName | where-object {$_.Name -like "*node*"}

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
    az network nic list --resource-group <resourcegroupname> --output table --query "[?contains(name,'node')].{NICname:name,InternalIP:ipConfigurations[0].privateIpAddress,InternalFQDN:dnsSettings.internalFqdn}"
    ```

2. Para determinar el puerto en el que está disponible un servicio, consulte el documento [Puertos utilizados por los servicios de Apache Hadoop en HDInsight](./hdinsight-hadoop-port-settings-for-services.md).

    > [!IMPORTANT]  
    > Algunos servicios hospedados en los nodos principales solo están activos en un nodo a la vez. Si intenta acceder a un servicio en un nodo principal y se produce un error, cambie al otro nodo principal.
    >
    > Por ejemplo, Apache Ambari solo está activo en un nodo principal a la vez. Si intenta acceder a Ambari en un nodo principal y se devuelve un error 404, es porque se ejecuta en el otro nodo principal.

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre cómo usar HDInsight en una red virtual, consulte el documento sobre el [planeamiento de una implementación de red virtual para clústeres de Azure HDInsight](./hdinsight-plan-virtual-network-deployment.md).

* Para más información sobre las redes virtuales de Azure, vea la [información general sobre Azure Virtual Network](../virtual-network/virtual-networks-overview.md).

* Para más información sobre los grupos de seguridad de red, vea [Grupos de seguridad de red](../virtual-network/security-overview.md).

* Para obtener más información sobre las rutas definidas por el usuario, consulte [Rutas definidas por el usuario y reenvío de IP](../virtual-network/virtual-networks-udr-overview.md).

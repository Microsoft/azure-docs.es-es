---
title: Creación de redes virtuales para clústeres de Azure HDInsight
description: Aprenda a crear una red virtual de Azure para conectar HDInsight con otros recursos en la nube o recursos en su centro de datos.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 04/16/2020
ms.openlocfilehash: 8e68bd2d164e3a8de60a9061363b839c4dfd4777
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87074762"
---
# <a name="create-virtual-networks-for-azure-hdinsight-clusters"></a>Creación de redes virtuales para clústeres de Azure HDInsight

En este artículo se proporcionan ejemplos y ejemplos de código para crear y configurar [instancias de Azure Virtual Network](../virtual-network/virtual-networks-overview.md). Para usar con clústeres de Azure HDInsight. Se presentan ejemplos detallados de creación de grupos de seguridad de red (NSG) y configuración de DNS.

Para obtener información general acerca del uso de redes virtuales con HDInsight, consulte el documento acerca del [planeamiento de una red virtual para Azure HDInsight](hdinsight-plan-virtual-network-deployment.md).

## <a name="prerequisites-for-code-samples-and-examples"></a>Requisitos previos para los ejemplos de código

Antes de ejecutar cualquiera de los ejemplos de código de este artículo, debe conocer las redes TCP/IP. Si no está familiarizado con las redes TCP/IP, póngase en contacto con alguien que lo esté antes de realizar modificaciones a las redes de producción.

Otros requisitos previos de los ejemplos de este artículo son los elementos siguientes:

* Si va a usar PowerShell, necesitará instalar el [módulo AZ](https://docs.microsoft.com/powershell/azure/).
* Si desea usar la CLI de Azure y aún no la ha instalado, consulte [Instalación de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli).

> [!IMPORTANT]  
> Si busca guías paso a paso sobre cómo conectar HDInsight con la red local mediante una instancia de Azure Virtual Network, consulte el documento [Conexión de HDInsight a la red local](connect-on-premises-network.md).

## <a name="example-network-security-groups-with-hdinsight"></a><a id="hdinsight-nsg"></a>Ejemplo: grupos de seguridad de red con HDInsight

En los ejemplos de esta sección se muestra cómo crear reglas de grupo de seguridad de red. Las reglas permiten que HDInsight se comunique con los servicios de administración de Azure. Antes de usar los ejemplos, ajuste las direcciones IP para que coincidan con las de la región de Azure que esté usando. Puede encontrar esta información en [Direcciones IP de administración de HDInsight](hdinsight-management-ip-addresses.md).

### <a name="azure-resource-management-template"></a>Plantilla de administración de recursos de Azure

La siguiente plantilla de administración de recursos crea una red virtual que restringe el tráfico de entrada pero permite el tráfico desde las direcciones IP requeridas por HDInsight. Esta plantilla crea también un clúster de HDInsight en la red virtual.

* [Implementar una instancia segura de Azure Virtual Network y un clúster de Hadoop de HDInsight](https://azure.microsoft.com/resources/templates/101-hdinsight-secure-vnet/)

### <a name="azure-powershell"></a>Azure PowerShell

Use el siguiente script de PowerShell para crear una red virtual que restringe el tráfico de entrada y permite el tráfico de las direcciones IP para la región de Norte de Europa.

> [!IMPORTANT]  
> Cambie las direcciones IP de `hdirule1` y `hdirule2` de este ejemplo para que coincidan con la región de Azure que utiliza. Puede encontrar esta información en [Direcciones IP de administración de HDInsight](hdinsight-management-ip-addresses.md).

```azurepowershell
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

En este ejemplo se muestra cómo agregar reglas para permitir el tráfico de entrada en las direcciones IP necesarias. No contiene una regla para restringir el acceso de entrada desde otros orígenes. En el siguiente código se muestra cómo habilitar el acceso mediante SSH desde Internet:

```azurepowershell
Get-AzNetworkSecurityGroup -Name hdisecure -ResourceGroupName RESOURCEGROUP |
Add-AzNetworkSecurityRuleConfig -Name "SSH" -Description "SSH" -Protocol "*" -SourcePortRange "*" -DestinationPortRange "22" -SourceAddressPrefix "*" -DestinationAddressPrefix "VirtualNetwork" -Access Allow -Priority 306 -Direction Inbound
```

### <a name="azure-cli"></a>Azure CLI

Use los pasos siguientes para crear una red virtual que restringe el tráfico de entrada pero permite el tráfico desde las direcciones IP requeridas por HDInsight.

1. Use el siguiente comando para crear un nuevo grupo de seguridad de red denominado " `hdisecure`". Reemplace `RESOURCEGROUP` por el grupo de recursos que contiene la red virtual de Azure. Reemplace `LOCATION` por la ubicación (región) en la que se haya creado el grupo.

    ```azurecli
    az network nsg create -g RESOURCEGROUP -n hdisecure -l LOCATION
    ```

    Cuando se haya creado el grupo, recibirá información sobre el nuevo grupo.

2. Utilice lo siguiente para agregar reglas al nuevo grupo de seguridad de red que permitan la comunicación entrante en el puerto 443 desde el servicio de mantenimiento y administración de HDInsight de Azure. Reemplace `RESOURCEGROUP` por el nombre del grupo de recursos que contiene la red virtual de Azure.

    > [!IMPORTANT]  
    > Cambie las direcciones IP de `hdirule1` y `hdirule2` de este ejemplo para que coincidan con la región de Azure que utiliza. Puede encontrar esta información en [Direcciones IP de administración de HDInsight](hdinsight-management-ip-addresses.md).

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
    az network nsg show -g RESOURCEGROUP -n hdisecure --query "id"
    ```

    Este comando devuelve un valor similar al siguiente texto:

    ```output
    "/subscriptions/SUBSCRIPTIONID/resourceGroups/RESOURCEGROUP/providers/Microsoft.Network/networkSecurityGroups/hdisecure"
    ```

4. Use el siguiente comando para aplicar el grupo de seguridad de red a una subred. Reemplace los valores de `GUID` y `RESOURCEGROUP` por los que se devolvieron en el paso anterior. Reemplace `VNETNAME` y `SUBNETNAME` por el nombre de la red virtual y el nombre de la subred que quiere crear.

    ```azurecli
    az network vnet subnet update -g RESOURCEGROUP --vnet-name VNETNAME --name SUBNETNAME --set networkSecurityGroup.id="/subscriptions/GUID/resourceGroups/RESOURCEGROUP/providers/Microsoft.Network/networkSecurityGroups/hdisecure"
    ```

    Una vez completado este comando, puede instalar HDInsight en la red virtual.

Con estos pasos solo se abre el acceso al servicio de mantenimiento y administración de HDInsight en Azure Cloud. Cualquier otro acceso al clúster de HDInsight desde fuera de Virtual Network está bloqueado. Para permitir el acceso desde fuera de la red virtual, debe agregar reglas adicionales del grupo de seguridad de red.

En el siguiente código se muestra cómo habilitar el acceso mediante SSH desde Internet:

```azurecli
az network nsg rule create -g RESOURCEGROUP --nsg-name hdisecure -n ssh --protocol "*" --source-port-range "*" --destination-port-range "22" --source-address-prefix "*" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 306 --direction "Inbound"
```

## <a name="example-dns-configuration"></a><a id="example-dns"></a> Ejemplo: Configuración de DNS

### <a name="name-resolution-between-a-virtual-network-and-a-connected-on-premises-network"></a>Resolución de nombres entre una red virtual y una red local conectada

En este ejemplo se da por supuesto lo siguiente:

* Dispone de una instancia de Azure Virtual Network que está conectada a una red local mediante una puerta de enlace de VPN.

* El servidor DNS personalizado en la red virtual ejecuta Linux o Unix como sistema operativo.

* [Bind](https://www.isc.org/downloads/bind/) está instalado en el servidor DNS personalizado.

En el servidor DNS en la red virtual:

1. Use Azure PowerShell o la CLI de Azure para encontrar el sufijo DNS de la red virtual:

    Reemplace `RESOURCEGROUP` por el nombre del grupo de recursos que contiene la red virtual y, a continuación, escriba el comando:

    ```azurepowershell
    $NICs = Get-AzNetworkInterface -ResourceGroupName "RESOURCEGROUP"
    $NICs[0].DnsSettings.InternalDomainNameSuffix
    ```

    ```azurecli
    az network nic list --resource-group RESOURCEGROUP --query "[0].dnsSettings.internalDomainNameSuffix"
    ```

1. En el servidor DNS personalizado para la red virtual, use el siguiente texto como el contenido del archivo `/etc/bind/named.conf.local`:

    ```
    // Forward requests for the virtual network suffix to Azure recursive resolver
    zone "0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net" {
        type forward;
        forwarders {168.63.129.16;}; # Azure recursive resolver
    };
    ```

    Reemplace el valor `0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net` con el sufijo DNS de la red virtual.

    Esta configuración dirige todas las solicitudes DNS para el sufijo DNS de la red virtual a la resolución recursiva de Azure.

1. En el servidor DNS personalizado para la red virtual, use el siguiente texto como el contenido del archivo `/etc/bind/named.conf.options`:

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

1. Para usar la configuración, reinicie Bind. Por ejemplo, `sudo service bind9 restart`.

1. Agregue un reenviador condicional al servidor DNS local. Configure el reenviador condicional para poder enviar solicitudes para el sufijo DNS del paso 1 al servidor DNS personalizado.

    > [!NOTE]  
    > Consulte la documentación del software de DNS para obtener información específica sobre cómo agregar un reenviador condicional.

Después de completar estos pasos, puede conectarse a los recursos de cualquiera de las redes mediante nombres de dominio completo (FQDN). Ahora puede instalar HDInsight en la red virtual.

### <a name="name-resolution-between-two-connected-virtual-networks"></a>Resolución de nombres entre dos redes virtuales conectadas

En este ejemplo se da por supuesto lo siguiente:

* Tiene dos instancias de Azure Virtual Network que se conectan mediante una puerta de enlace de VPN o un emparejamiento.

* El servidor DNS personalizado en ambas redes ejecuta Linux o Unix como sistema operativo.

* [Bind](https://www.isc.org/downloads/bind/) está instalado en los servidores DNS personalizados.

1. Use Azure PowerShell o la CLI de Azure para buscar el sufijo DNS de las dos redes virtuales:

    Reemplace `RESOURCEGROUP` por el nombre del grupo de recursos que contiene la red virtual y, a continuación, escriba el comando:

    ```azurepowershell
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

* Para obtener un ejemplo completo de la configuración de HDInsight para conectarse a una red local, consulte [Conexión de HDInsight a la red local](./connect-on-premises-network.md).
* Para configurar clústeres de Apache HBase en redes virtuales de Azure, consulte [Creación de clústeres de Apache HBase en una red virtual de Azure](hbase/apache-hbase-provision-vnet.md).
* Para configurar la replicación geográfica de Apache HBase, consulte [Configuración de la replicación de clúster de Apache HBase en redes virtuales de Azure](hbase/apache-hbase-replication.md).
* Para más información sobre las redes virtuales de Azure, vea la [información general sobre Azure Virtual Network](../virtual-network/virtual-networks-overview.md).

* Para más información sobre los grupos de seguridad de red, vea [Grupos de seguridad de red](../virtual-network/security-overview.md).

* Para obtener más información sobre las rutas definidas por el usuario, consulte [Rutas definidas por el usuario y reenvío de IP](../virtual-network/virtual-networks-udr-overview.md).

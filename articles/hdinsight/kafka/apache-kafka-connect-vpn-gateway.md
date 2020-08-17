---
title: Conexión a Kafka mediante redes virtuales - Azure HDInsight
description: Aprenda a conectarse directamente a Kafka en HDInsight mediante una instancia de Azure Virtual Network. Aprenda cómo conectarse a Kafka desde clientes de desarrollo mediante una puerta de enlace de red privada virtual o desde clientes de la red local mediante un dispositivo de puerta de enlace de red privada virtual.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-python
ms.date: 03/04/2020
ms.openlocfilehash: ca5bff7f8a14efc0916210296048e3731aec5a59
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87873473"
---
# <a name="connect-to-apache-kafka-on-hdinsight-through-an-azure-virtual-network"></a>Conexión con Apache Kafka en HDInsight mediante una instancia de Azure Virtual Network

Aprenda a conectarse directamente a Apache Kafka en HDInsight mediante una instancia de Azure Virtual Network. En este documento se proporciona información sobre la conexión a Kafka mediante las siguientes configuraciones:

* Desde recursos de una red local. Esta conexión se establece mediante un dispositivo de red privada virtual (software o hardware) en la red local.
* Desde un entorno de desarrollo con un cliente de software de red privada virtual.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="architecture-and-planning"></a>Arquitectura y planeación

HDInsight no permite la conexión directa a Kafka a través de la red pública de Internet. Los clientes de Kafka (productores y consumidores) deben usar uno de los siguientes métodos de conexión:

* Ejecutar el cliente en la misma red virtual que Kafka en HDInsight. Esta configuración se usa en el documento [Introducción a Apache Kafka en HDInsight](apache-kafka-get-started.md). El cliente se ejecuta directamente en los nodos del clúster de HDInsight o en otra máquina virtual de la misma red.

* Conectar una red privada, como la red local, a la red virtual. Esta configuración permite a los clientes de la red local trabajar directamente con Kafka. Para habilitar esta configuración, realice las siguientes tareas:

  1. Cree una red virtual.
  2. Cree una puerta de enlace de red privada virtual que use una configuración de sitio a sitio. La configuración usada en este documento conecta a un dispositivo de puerta de enlace de red privada virtual en la red local.
  3. Cree un servidor DNS en la red virtual.
  4. Configure el reenvío entre el servidor DNS de cada red.
  5. Cree una instancia de Kafka en un clúster de HDInsight en la red virtual.

     Para más información, vea la sección [Conexión a Apache Kafka desde una red](#on-premises).

* Conectar equipos individuales a la red virtual mediante una puerta de enlace de red privada virtual y un cliente de red privada virtual. Para habilitar esta configuración, realice las siguientes tareas:

  1. Cree una red virtual.
  2. Cree una puerta de enlace de red privada virtual que use una configuración de punto a sitio. Esta configuración se puede utilizar tanto con clientes de Windows como de macOS.
  3. Cree una instancia de Kafka en un clúster de HDInsight en la red virtual.
  4. Configurar Kafka para anunciar direcciones IP. Esta configuración permite al cliente conectarse mediante direcciones IP de agente en lugar de nombres de dominio.
  5. Descargue y use el cliente de VPN en el sistema de desarrollo.

     Para más información, vea la sección [Conexión a Apache Kafka con un cliente VPN](#vpnclient).

     > [!WARNING]  
     > Esta configuración solo se recomienda para fines de desarrollo debido a las limitaciones siguientes:
     >
     > * Cada cliente debe conectarse con un cliente de software de VPN.
     > * El cliente VPN no pasa las solicitudes de resolución de nombres a la red virtual, por lo que debe usar direcciones IP para comunicarse con Kafka. La comunicación IP exige configuración adicional en el clúster de Kafka.

Para más información sobre cómo usar HDInsight en una red virtual, consulte el documento sobre el [planeamiento de una red virtual para clústeres de Azure HDInsight](../hdinsight-plan-virtual-network-deployment.md).

## <a name="connect-to-apache-kafka-from-an-on-premises-network"></a><a id="on-premises"></a> Conexión a Apache Kafka desde una red local

Para crear un clúster de Kafka que se comunique con la red local, siga los pasos del documento [Connect HDInsight to your on-premises network (Conexión de HDInsight a la red local)](./../connect-on-premises-network.md).

> [!IMPORTANT]  
> Al crear el clúster de HDInsight, seleccione el tipo de clúster __Kafka__.

Estos pasos crean la siguiente configuración:

* Azure Virtual Network
* Puerta de enlace de VPN de sitio a sitio
* Cuenta de Azure Storage (usada por HDInsight)
* Kafka en HDInsight

Para comprobar que un cliente de Kafka puede conectarse al clúster desde el entorno local, siga los pasos de la sección [Ejemplo: cliente de Python](#python-client).

## <a name="connect-to-apache-kafka-with-a-vpn-client"></a><a id="vpnclient"></a> Conexión a Apache Kafka con un cliente VPN

Use los pasos de esta sección para crear la configuración siguiente:

* Azure Virtual Network
* Puerta de enlace de VPN de punto a sitio
* Cuenta de Azure Storage (usada por HDInsight)
* Kafka en HDInsight

1. Siga los pasos del documento [Working with self-signed certificates for Point-to-site connections (Funcionamiento de los certificados autofirmados para conexiones de punto a sitio)](../../vpn-gateway/vpn-gateway-certificates-point-to-site.md). Este documento crea los certificados necesarios para la puerta de enlace.

2. Abra un símbolo del sistema de PowerShell y use el código siguiente para iniciar sesión en la suscripción de Azure:

    ```powershell
    Connect-AzAccount
    # If you have multiple subscriptions, uncomment to set the subscription
    #Select-AzSubscription -SubscriptionName "name of your subscription"
    ```

3. Use el código siguiente para crear variables que contengan la información de configuración:

    ```powershell
    # Prompt for generic information
    $resourceGroupName = Read-Host "What is the resource group name?"
    $baseName = Read-Host "What is the base name? It is used to create names for resources, such as 'net-basename' and 'kafka-basename':"
    $location = Read-Host "What Azure Region do you want to create the resources in?"
    $rootCert = Read-Host "What is the file path to the root certificate? It is used to secure the VPN gateway."

    # Prompt for HDInsight credentials
    $adminCreds = Get-Credential -Message "Enter the HTTPS user name and password for the HDInsight cluster" -UserName "admin"
    $sshCreds = Get-Credential -Message "Enter the SSH user name and password for the HDInsight cluster" -UserName "sshuser"

    # Names for Azure resources
    $networkName = "net-$baseName"
    $clusterName = "kafka-$baseName"
    $storageName = "store$baseName" # Can't use dashes in storage names
    $defaultContainerName = $clusterName
    $defaultSubnetName = "default"
    $gatewaySubnetName = "GatewaySubnet"
    $gatewayPublicIpName = "GatewayIp"
    $gatewayIpConfigName = "GatewayConfig"
    $vpnRootCertName = "rootcert"
    $vpnName = "VPNGateway"

    # Network settings
    $networkAddressPrefix = "10.0.0.0/16"
    $defaultSubnetPrefix = "10.0.0.0/24"
    $gatewaySubnetPrefix = "10.0.1.0/24"
    $vpnClientAddressPool = "172.16.201.0/24"

    # HDInsight settings
    $HdiWorkerNodes = 4
    $hdiVersion = "3.6"
    $hdiType = "Kafka"
    ```

4. Use el código siguiente para crear la red virtual y el grupo de recursos de Azure:

    ```powershell
    # Create the resource group that contains everything
    New-AzResourceGroup -Name $resourceGroupName -Location $location

    # Create the subnet configuration
    $defaultSubnetConfig = New-AzVirtualNetworkSubnetConfig -Name $defaultSubnetName `
        -AddressPrefix $defaultSubnetPrefix
    $gatewaySubnetConfig = New-AzVirtualNetworkSubnetConfig -Name $gatewaySubnetName `
        -AddressPrefix $gatewaySubnetPrefix

    # Create the subnet
    New-AzVirtualNetwork -Name $networkName `
        -ResourceGroupName $resourceGroupName `
        -Location $location `
        -AddressPrefix $networkAddressPrefix `
        -Subnet $defaultSubnetConfig, $gatewaySubnetConfig

    # Get the network & subnet that were created
    $network = Get-AzVirtualNetwork -Name $networkName `
        -ResourceGroupName $resourceGroupName
    $gatewaySubnet = Get-AzVirtualNetworkSubnetConfig -Name $gatewaySubnetName `
        -VirtualNetwork $network
    $defaultSubnet = Get-AzVirtualNetworkSubnetConfig -Name $defaultSubnetName `
        -VirtualNetwork $network

    # Set a dynamic public IP address for the gateway subnet
    $gatewayPublicIp = New-AzPublicIpAddress -Name $gatewayPublicIpName `
        -ResourceGroupName $resourceGroupName `
        -Location $location `
        -AllocationMethod Dynamic
    $gatewayIpConfig = New-AzVirtualNetworkGatewayIpConfig -Name $gatewayIpConfigName `
        -Subnet $gatewaySubnet `
        -PublicIpAddress $gatewayPublicIp

    # Get the certificate info
    # Get the full path in case a relative path was passed
    $rootCertFile = Get-ChildItem $rootCert
    $cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($rootCertFile)
    $certBase64 = [System.Convert]::ToBase64String($cert.RawData)
    $p2sRootCert = New-AzVpnClientRootCertificate -Name $vpnRootCertName `
        -PublicCertData $certBase64

    # Create the VPN gateway
    New-AzVirtualNetworkGateway -Name $vpnName `
        -ResourceGroupName $resourceGroupName `
        -Location $location `
        -IpConfigurations $gatewayIpConfig `
        -GatewayType Vpn `
        -VpnType RouteBased `
        -EnableBgp $false `
        -GatewaySku Standard `
        -VpnClientAddressPool $vpnClientAddressPool `
        -VpnClientRootCertificates $p2sRootCert
    ```

    > [!WARNING]  
    > Este proceso puede tardar varios minutos en completarse.

5. Use el código siguiente para crear el contenedor de blobs y la cuenta de Azure Storage:

    ```powershell
    # Create the storage account
    New-AzStorageAccount `
        -ResourceGroupName $resourceGroupName `
        -Name $storageName `
        -SkuName Standard_GRS `
        -Location $location `
        -Kind StorageV2 `
        -EnableHttpsTrafficOnly 1

    # Get the storage account keys and create a context
    $defaultStorageKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName `
        -Name $storageName)[0].Value
    $storageContext = New-AzStorageContext -StorageAccountName $storageName `
        -StorageAccountKey $defaultStorageKey

    # Create the default storage container
    New-AzStorageContainer -Name $defaultContainerName `
        -Context $storageContext
    ```

6. Use el código siguiente para crear el clúster de HDInsight:

    ```powershell
    # Create the HDInsight cluster
    New-AzHDInsightCluster `
        -ResourceGroupName $resourceGroupName `
        -ClusterName $clusterName `
        -Location $location `
        -ClusterSizeInNodes $hdiWorkerNodes `
        -ClusterType $hdiType `
        -OSType Linux `
        -Version $hdiVersion `
        -HttpCredential $adminCreds `
        -SshCredential $sshCreds `
        -DefaultStorageAccountName "$storageName.blob.core.windows.net" `
        -DefaultStorageAccountKey $defaultStorageKey `
        -DefaultStorageContainer $defaultContainerName `
        -DisksPerWorkerNode 2 `
        -VirtualNetworkId $network.Id `
        -SubnetName $defaultSubnet.Id
    ```

   > [!WARNING]  
   > Este proceso tarda unos 15 minutos en completarse.

### <a name="configure-kafka-for-ip-advertising"></a>Configuración de Kafka para anunciar direcciones IP

De manera predeterminada, Apache Zookeeper devuelve el nombre de dominio de los agentes de Kafka a los clientes. Esta configuración no funciona con el cliente de software de VPN, ya que no puede usar la resolución de nombres para entidades de la red virtual. Para esta configuración, use los pasos siguientes para configurar Kafka con el fin de anunciar direcciones IP en lugar de nombres de dominio:

1. Vaya a `https://CLUSTERNAME.azurehdinsight.net` desde un explorador web. Reemplace `CLUSTERNAME` por el nombre del clúster de Kafka en HDInsight.

    Cuando se le solicite, use el nombre de usuario y la contraseña HTTPS para el clúster. Aparece la interfaz de usuario web de Ambari para el clúster.

2. Para ver información sobre Kafka, seleccione __Kafka__ en la lista de la izquierda.

    ![Lista de servicios donde Kafka aparece resaltado](./media/apache-kafka-connect-vpn-gateway/select-kafka-service.png)

3. Para ver la configuración de Kafka, seleccione __Configs__ (Configuraciones) en la parte superior central.

    ![Configuración de servicios de Apache Ambari](./media/apache-kafka-connect-vpn-gateway/select-kafka-config1.png)

4. Para encontrar la configuración __kafka-env__, escriba `kafka-env` en el campo __Filtrar__ que se encuentra en la esquina superior derecha.

    ![Configuración de Kafka para kafka-env](./media/apache-kafka-connect-vpn-gateway/search-for-kafka-env.png)

5. Para configurar Kafka y anunciar direcciones IP, agregue el texto siguiente en la parte inferior del campo __kafka-env-template__:

    ```
    # Configure Kafka to advertise IP addresses instead of FQDN
    IP_ADDRESS=$(hostname -i)
    echo advertised.listeners=$IP_ADDRESS
    sed -i.bak -e '/advertised/{/advertised@/!d;}' /usr/hdp/current/kafka-broker/conf/server.properties
    echo "advertised.listeners=PLAINTEXT://$IP_ADDRESS:9092" >> /usr/hdp/current/kafka-broker/conf/server.properties
    ```

6. Para configurar la interfaz en que escucha Kafka, escriba `listeners` en el campo __Filtrar__ que se encuentra en la esquina superior derecha.

7. Para configurar Kafka para que escuche en todas las interfaces de red, cambie el valor del campo __listeners__ (agentes de escucha) a `PLAINTEXT://0.0.0.0:9092`.

8. Use el botón __Guardar__ para guardar los cambios en la configuración. Escriba un mensaje de texto para describir los cambios. Seleccione __Aceptar__ una vez que se guarden los cambios.

    ![Apache Ambari: guardar configuración](./media/apache-kafka-connect-vpn-gateway/save-configuration-button.png)

9. Para evitar errores al reiniciar Kafka, use el botón __Acciones de servicio__ y seleccione __Activar el modo de mantenimiento__. Seleccione Aceptar para completar esta operación.

    ![Acciones de servicio, en que Activar mantenimiento aparece resaltado](./media/apache-kafka-connect-vpn-gateway/turn-on-maintenance-mode.png)

10. Para reiniciar Kafka, use el botón __Reiniciar__ y seleccione __Restart All Affected__ (Reiniciar todos los elementos afectados). Confirme el reinicio y use el botón __Aceptar__ una vez que se complete la operación.

    ![Botón Reiniciar con la opción Restart All Affected resaltada](./media/apache-kafka-connect-vpn-gateway/restart-required-button.png)

11. Para deshabilitar el modo de mantenimiento, use el botón __Acciones de servicio__ y seleccione __Desactivar el modo de mantenimiento__. Seleccione **Aceptar** para completar esta operación.

### <a name="connect-to-the-vpn-gateway"></a>Conexión a la puerta de enlace de VPN

Para conectarse a la puerta de enlace de VPN, use la sección __Conexión a Azure__ del documento [Configuración de una conexión de punto a sitio](../../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md#connect).

## <a name="example-python-client"></a><a id="python-client"></a> Ejemplo: cliente de Python

Para validar la conectividad con Kafka, use los pasos siguientes para crear y ejecutar un productor y un consumidor:

1. Use uno de los métodos siguientes para recuperar el nombre de dominio completo (FQDN) y las direcciones IP de los nodos del clúster de Kafka:

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

    En este script se supone que `$resourceGroupName` es el nombre del grupo de recursos de Azure que contiene la red virtual.

    Guarde la información devuelta para su uso en los pasos siguientes.

2. Use el código siguiente para instalar el cliente [kafka-python](https://kafka-python.readthedocs.io/):

    ```bash
    pip install kafka-python
    ```

3. Para enviar datos a Kafka, use el código Python siguiente:

   ```python
   from kafka import KafkaProducer
   # Replace the `ip_address` entries with the IP address of your worker nodes
   # NOTE: you don't need the full list of worker nodes, just one or two.
   producer = KafkaProducer(bootstrap_servers=['kafka_broker_1','kafka_broker_2'])
   for _ in range(50):
      producer.send('testtopic', b'test message')
   ```

    Sustituya las entradas `'kafka_broker'` por las direcciones devueltas en el paso 1 de esta sección:

   * Si usa un __cliente de software de VPN__, sustituya las entradas `kafka_broker` por la dirección IP de los nodos de trabajo.

   * Si tiene __habilitada la resolución de nombres a través de un servidor DNS personalizado__, sustituya las entradas `kafka_broker` por el FQDN de los nodos de trabajo.

     > [!NOTE]
     > Este código envía la cadena `test message` al tema `testtopic`. La configuración predeterminada de Kafka en HDInsight es crear el tema si no existe.

4. Para recuperar los mensajes de Kafka, use el código Python siguiente:

   ```python
   from kafka import KafkaConsumer
   # Replace the `ip_address` entries with the IP address of your worker nodes
   # Again, you only need one or two, not the full list.
   # Note: auto_offset_reset='earliest' resets the starting offset to the beginning
   #       of the topic
   consumer = KafkaConsumer(bootstrap_servers=['kafka_broker_1','kafka_broker_2'],auto_offset_reset='earliest')
   consumer.subscribe(['testtopic'])
   for msg in consumer:
     print (msg)
   ```

    Sustituya las entradas `'kafka_broker'` por las direcciones devueltas en el paso 1 de esta sección:

    * Si usa un __cliente de software de VPN__, sustituya las entradas `kafka_broker` por la dirección IP de los nodos de trabajo.

    * Si tiene __habilitada la resolución de nombres a través de un servidor DNS personalizado__, sustituya las entradas `kafka_broker` por el FQDN de los nodos de trabajo.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo usar HDInsight con una red virtual, consulte el documento sobre el [planeamiento de la implementación de una red virtual para clústeres de Azure HDInsight](../hdinsight-plan-virtual-network-deployment.md).

Para más información sobre cómo crear una instancia de Azure Virtual Network con puerta de enlace de VPN de punto a sitio, consulte los documentos siguientes:

* [Configuración de una conexión de punto a sitio mediante Azure Portal](../../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md)

* [Configuración de una conexión de punto a sitio mediante Azure PowerShell](../../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md)

Para más información sobre cómo trabajar con Apache Kafka en HDInsight, consulte los documentos siguientes:

* [Introducción a Apache Kafka en HDInsight](apache-kafka-get-started.md)
* [Uso de creación de reflejos con Apache Kafka en HDInsight](apache-kafka-mirroring.md)

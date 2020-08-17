---
title: 'Inicio rápido: Creación de Apache Kafka con Azure PowerShell: HDInsight'
description: En esta guía de inicio rápido, aprenderá a crear un clúster de Apache Kafka en Azure HDInsight con Azure PowerShell. También aprenderá sobre los temas, los suscriptores y los consumidores de Kafka.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: mvc
ms.topic: quickstart
ms.date: 06/12/2019
ms.openlocfilehash: 57f9338841b599e10c8a1d7eec8fd4f371ceecb9
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87081022"
---
# <a name="quickstart-create-apache-kafka-cluster-in-azure-hdinsight-using-powershell"></a>Inicio rápido: Creación de un clúster de Apache Kafka en Azure HDInsight mediante PowerShell

[Apache Kafka](https://kafka.apache.org/) es una plataforma de streaming distribuida y de código abierto. A menudo se usa como agente de mensajes, ya que proporciona una funcionalidad similar a una cola de mensajes de publicación o suscripción. 

En esta guía de inicio rápido, aprenderá a crear un clúster de [Apache Kafka](https://kafka.apache.org) con Azure PowerShell. También aprenderá a usar las utilidades incluidas para enviar y recibir mensajes con Kafka.

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

Solo los recursos dentro de la misma red virtual pueden tener acceso a la API de Kafka. En esta guía de inicio rápido, tendrá acceso al clúster directamente mediante SSH. Para conectar otros servicios, redes o máquinas virtuales con Kafka, primero debe crear una red virtual y, a continuación, crear los recursos dentro de la red. Para más información, consulte el documento [Conexión a Kafka en HDInsight mediante una instancia de Azure Virtual Network](apache-kafka-connect-vpn-gateway.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

* Instalación del [módulo Az](https://docs.microsoft.com/powershell/azure/) de PowerShell.

* Un cliente SSH. Para más información, consulte [Conexión a través de SSH con HDInsight (Apache Hadoop)](../hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en la suscripción de Azure con el cmdlet `Connect-AzAccount` y siga las instrucciones que aparecen en pantalla.

```azurepowershell-interactive
# Login to your Azure subscription
$sub = Get-AzSubscription -ErrorAction SilentlyContinue
if(-not($sub))
{
    Connect-AzAccount
}

# If you have multiple subscriptions, set the one to use
# Select-AzSubscription -SubscriptionId "<SUBSCRIPTIONID>"
```

## <a name="create-resource-group"></a>Creación de un grupo de recursos

Cree un grupo de recursos de Azure con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. En el ejemplo siguiente, se le pide el nombre y la ubicación y, a continuación, crea un nuevo grupo de recursos:

```azurepowershell-interactive
$resourceGroup = Read-Host -Prompt "Enter the resource group name"
$location = Read-Host -Prompt "Enter the Azure region to use"

New-AzResourceGroup -Name $resourceGroup -Location $location
```

## <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

Mientras que Kafka en HDInsight usa Azure Managed Disks para almacenar datos de Kafka, el clúster también usa Azure Storage para almacenar información, como registros. Use [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount) para crear una cuenta de almacenamiento nueva.

> [!IMPORTANT]  
> El tipo de cuenta de almacenamiento `BlobStorage` solo puede usarse como almacenamiento secundario para los clústeres de HDInsight.

```azurepowershell-interactive
$storageName = Read-Host -Prompt "Enter the storage account name"

New-AzStorageAccount `
    -ResourceGroupName $resourceGroup `
    -Name $storageName `
    -Location $location `
    -SkuName Standard_LRS `
    -Kind StorageV2 `
    -EnableHttpsTrafficOnly 1
```

HDInsight almacena los datos en la cuenta de almacenamiento en un contenedor de blobs. Use [New-AzStorageContainer](/powershell/module/Az.Storage/New-AzStorageContainer) para crear un contenedor nuevo.

```azurepowershell-interactive
$containerName = Read-Host -Prompt "Enter the container name"

$storageKey = (Get-AzStorageAccountKey `
                -ResourceGroupName $resourceGroup `
                -Name $storageName)[0].Value
$storageContext = New-AzStorageContext `
                    -StorageAccountName $storageName `
                    -StorageAccountKey $storageKey

New-AzStorageContainer -Name $containerName -Context $storageContext
```

## <a name="create-an-apache-kafka-cluster"></a>Creación de un clúster de Apache Kafka

Cree un clúster de Apache Kafka en HDInsight con [New-AzHDInsightCluster](/powershell/module/az.HDInsight/New-azHDInsightCluster).

```azurepowershell-interactive
# Create a Kafka 1.1 cluster
$clusterName = Read-Host -Prompt "Enter the name of the Kafka cluster"
$httpCredential = Get-Credential -Message "Enter the cluster login credentials" -UserName "admin"
$sshCredentials = Get-Credential -Message "Enter the SSH user credentials" -UserName "sshuser"

$numberOfWorkerNodes = "4"
$clusterVersion = "3.6"
$clusterType="Kafka"
$disksPerNode=2

$kafkaConfig = New-Object "System.Collections.Generic.Dictionary``2[System.String,System.String]"
$kafkaConfig.Add("kafka", "1.1")

New-AzHDInsightCluster `
        -ResourceGroupName $resourceGroup `
        -ClusterName $clusterName `
        -Location $location `
        -ClusterSizeInNodes $numberOfWorkerNodes `
        -ClusterType $clusterType `
        -OSType "Linux" `
        -Version $clusterVersion `
        -ComponentVersion $kafkaConfig `
        -HttpCredential $httpCredential `
        -DefaultStorageAccountName "$storageName.blob.core.windows.net" `
        -DefaultStorageAccountKey $storageKey `
        -DefaultStorageContainer $clusterName `
        -SshCredential $sshCredentials `
        -DisksPerWorkerNode $disksPerNode
```

Un clúster de HDInsight puede tardar hasta 20 minutos en crearse.

El parámetro `-DisksPerWorkerNode` configura la escalabilidad de Kafka en HDInsight. Kafka en HDInsight usa el disco local de las máquinas virtuales del clúster para almacenar datos. Como Kafka tiene muchas E/S, [Azure Managed Disks](../../virtual-machines/windows/managed-disks-overview.md) se usa para proporcionar alto rendimiento y mayor espacio de almacenamiento por nodo.

El tipo de disco administrado puede ser __Estándar__ (HDD) o __Premium__ (SSD). El tipo de disco depende del tamaño de máquina virtual que usan los nodos de trabajo (agentes de Kafka). Los discos Premium se usan automáticamente con máquinas virtuales de las series DS y GS. Todos los otros tipos de máquina virtual usan discos estándar. Puede establecer el tipo de máquina virtual mediante el parámetro `-WorkerNodeSize`. Para más información sobre parámetros, consulte la documentación de [New-AzHDInsightCluster](/powershell/module/az.HDInsight/New-azHDInsightCluster).

Si tiene pensado usar más de 32 nodos de trabajo (al crear el clúster o escalando el clúster después de su creación), también debe usar el parámetro `-HeadNodeSize` para especificar un tamaño de máquina virtual con al menos 8 núcleos y 14 GB de RAM. Para obtener más información acerca de los tamaños de nodo y los costos asociados, consulte [Precios de HDInsight](https://azure.microsoft.com/pricing/details/hdinsight/).

## <a name="connect-to-the-cluster"></a>Conectarse al clúster

1. Para conectarse con el nodo principal del clúster de Kafka, use el siguiente comando. Reemplace `sshuser` por el nombre de usuario de SSH. Reemplace `mykafka` por el nombre del clúster de Kafka.

    ```bash
    ssh sshuser@mykafka-ssh.azurehdinsight.net
    ```

2. Cuando se conecte por primera vez al clúster, es posible que el cliente SSH muestre una advertencia de que no se puede establecer la autenticidad del host. Cuando se le solicite, escriba __yes__ (sí) y presione __Entrar__ para agregar el host a la lista de servidores de confianza de su cliente SSH.

3. Cuando se le solicite, escriba la contraseña del usuario de SSH.

Una vez que se haya conectado, verá información similar al texto siguiente:

```output
Authorized uses only. All activity may be monitored and reported.
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.13.0-1011-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    https://www.ubuntu.com/business/services/cloud

83 packages can be updated.
37 updates are security updates.



Welcome to Kafka on HDInsight.

Last login: Thu Mar 29 13:25:27 2018 from 108.252.109.241
```

## <a name="get-the-apache-zookeeper-and-broker-host-information"></a><a id="getkafkainfo"></a>Obtención de la información del host de Apache Zookeeper y del agente

Cuando se trabaja con Kafka, debe conocer los hosts de *Apache Zookeeper* y del *agente*. Estos hosts se usan con la API de Kafka y muchas de las utilidades que se incluyen con Kafka.

En esta sección, obtendrá la información de host de la API de REST de Apache Ambari en el clúster.

1. Desde la conexión SSH al clúster, use el siguiente comando para instalar la utilidad `jq`. Esta utilidad se usa para analizar documentos JSON y es útil para recuperar la información de host:
   
    ```bash
    sudo apt -y install jq
    ```

2. Para establecer una variable de entorno en el nombre del clúster, use el comando siguiente:

    ```bash
    read -p "Enter the Kafka on HDInsight cluster name: " CLUSTERNAME
    ```

    Cuando se le solicite, escriba el nombre del clúster de Kafka.

3. Para establecer una variable de entorno con la información de host de Zookeeper, use el comando siguiente. El comando recupera todos los hosts de Zookeeper y, a continuación, devuelve solo las dos primeras entradas. Esto se debe a que quiere cierta redundancia en caso de que un host sea inaccesible.

    ```bash
    export KAFKAZKHOSTS=`curl -sS -u admin -G https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/ZOOKEEPER/components/ZOOKEEPER_SERVER | jq -r '["\(.host_components[].HostRoles.host_name):2181"] | join(",")' | cut -d',' -f1,2`
    ```

    Cuando se le solicite, escriba la contraseña de la cuenta de inicio de sesión del clúster (no la cuenta de SSH).

4. Para comprobar que la variable de entorno se ha establecido correctamente, use el comando siguiente:

    ```bash
     echo '$KAFKAZKHOSTS='$KAFKAZKHOSTS
    ```

    Este comando devuelve información similar al texto siguiente:

    `zk0-kafka.eahjefxxp1netdbyklgqj5y1ud.ex.internal.cloudapp.net:2181,zk2-kafka.eahjefxxp1netdbyklgqj5y1ud.ex.internal.cloudapp.net:2181`

5. Para establecer una variable de entorno con la información de host del agente de Kafka, use el comando siguiente:

    ```bash
    export KAFKABROKERS=`curl -sS -u admin -G https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2`
    ```

    Cuando se le solicite, escriba la contraseña de la cuenta de inicio de sesión del clúster (no la cuenta de SSH).

6. Para comprobar que la variable de entorno se ha establecido correctamente, use el comando siguiente:

    ```bash   
    echo '$KAFKABROKERS='$KAFKABROKERS
    ```

    Este comando devuelve información similar al texto siguiente:
   
    `wn1-kafka.eahjefxxp1netdbyklgqj5y1ud.cx.internal.cloudapp.net:9092,wn0-kafka.eahjefxxp1netdbyklgqj5y1ud.cx.internal.cloudapp.net:9092`

## <a name="manage-apache-kafka-topics"></a>Administración de temas de Apache Kafka

Kafka almacena flujos de datos en *temas*. Puede usar la utilidad `kafka-topics.sh` para administrar temas.

* **Para crear un tema**, use el comando siguiente en la conexión SSH:

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic test --zookeeper $KAFKAZKHOSTS
    ```

    Este comando se conecta a Zookeeper mediante la información de host almacenada en `$KAFKAZKHOSTS`. Y, luego, crea un tema de Kafka llamado **test**. 

    * Los datos almacenados en este tema se dividen en ocho particiones.

    * Cada partición se replica en tres nodos de trabajo del clúster.

        Si ha creado el clúster en una región de Azure que proporciona tres dominios de error, use un factor de replicación de 3. De lo contrario, use un factor de replicación de 4.
        
        En regiones con tres dominios de error, un factor de replicación de 3 permite que las réplicas se distribuyan entre los dominios de error. En regiones con dos dominios de error, un factor de replicación de cuatro permite que las réplicas se distribuyan equitativamente entre los dominios.
        
        Para información sobre el número de dominios de error de una región, consulte el documento sobre la [disponibilidad de las máquinas virtuales Linux](../../virtual-machines/windows/manage-availability.md#use-managed-disks-for-vms-in-an-availability-set).

        Kafka no es compatible con dominios de error de Azure. Al crear réplicas de la partición de temas, puede que estas no se distribuyan correctamente con alta disponibilidad.

        Para garantizar la alta disponibilidad, use la [herramienta de reequilibrado de particiones de Apache Kafka](https://github.com/hdinsight/hdinsight-kafka-tools). Esta herramienta se debe ejecutar desde una conexión SSH en el nodo principal del clúster de Kafka.

        Para obtener la máxima disponibilidad de los datos de Kafka, debe reequilibrar las réplicas de las particiones del tema cuando:

        * Cree un nuevo tema o una partición

        * Escale verticalmente un clúster

* **Para mostrar temas**, use el comando siguiente:

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --list --zookeeper $KAFKAZKHOSTS
    ```

    Este comando muestra los temas disponibles en el clúster de Kafka.

* **Para eliminar un tema**, use el comando siguiente:

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --delete --topic topicname --zookeeper $KAFKAZKHOSTS
    ```

    Este comando elimina el tema `topicname`.

    > [!WARNING]  
    > Si elimina el tema `test` que ha creado anteriormente, debe volver a crearlo. Se usa más adelante en este documento.

Para obtener más información acerca de los comandos disponibles con la utilidad `kafka-topics.sh`, use el siguiente comando:

```bash
/usr/hdp/current/kafka-broker/bin/kafka-topics.sh
```

## <a name="produce-and-consume-records"></a>Generación y consumo de registros

Kafka almacena *registros* en temas. Los registros se generan mediante *productores* y se consumen mediante *consumidores*. Los productores y consumidores se comunican con el servicio de *agente de Kafka*. Cada nodo de trabajo del clúster de HDInsight es un host de agente de Kafka.

Use los pasos siguientes para almacenar registros en el tema de prueba que creó anteriormente y luego leerlos mediante un consumidor:

1. Para escribir registros en el tema, use la utilidad `kafka-console-producer.sh` desde la conexión SSH:
   
    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --broker-list $KAFKABROKERS --topic test
    ```
   
    Después de este comando, llega a una línea vacía.

2. Escriba un mensaje de texto en la línea vacía y presione ENTRAR. Escriba algunos mensajes de esta forma y, a continuación, use **Ctrl + C** para volver al símbolo del sistema normal. Cada línea se envía como un registro independiente al tema de Kafka.

3. Para leer registros del tema, use la utilidad `kafka-console-consumer.sh` desde la conexión SSH:
   
    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --bootstrap-server $KAFKABROKERS --topic test --from-beginning
    ```
   
    Este comando recupera los registros del tema y los muestra. Con `--from-beginning` se indica al consumidor que comience desde el principio del flujo, de modo que se recuperan todos los registros.

    Si está utilizando una versión anterior de Kafka, reemplace `--bootstrap-server $KAFKABROKERS` por `--zookeeper $KAFKAZKHOSTS`.

4. Use __Ctrl + C__ para detener el consumidor.

También puede crear mediante programación los productores y consumidores. Para obtener un ejemplo del uso de esta API, consulte el documento [Producer y Consumer API de Apache Kafka](apache-kafka-producer-consumer-api.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, puede usar el comando [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos, HDInsight y todos los recursos relacionados.

```azurepowershell-interactive
Remove-AzResourceGroup -Name $resourceGroup
```

> [!WARNING]  
> La facturación del clúster de HDInsight se inicia una vez creado el clúster y solo se detiene cuando se elimina. Se facturan por minuto realizando una prorrata, por lo que siempre debe eliminar aquellos que ya no se estén utilizando.
> 
> Al eliminar un clúster de Kafka en HDInsight se eliminan todos los datos almacenados en Kafka.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Uso de Apache Spark con Apache Kafka](../hdinsight-apache-kafka-spark-structured-streaming.md)

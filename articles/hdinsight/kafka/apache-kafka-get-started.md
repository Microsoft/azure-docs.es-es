---
title: 'Inicio rápido: configuración de Apache Kafka en HDInsight mediante Azure Portal'
description: En esta guía de inicio rápido, aprenderá a crear un clúster de Apache Kafka en Azure HDInsight con Azure Portal. También aprenderá sobre los temas, los suscriptores y los consumidores de Kafka.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: quickstart
ms.custom: mvc
ms.date: 04/29/2020
ms.openlocfilehash: 6198475025ff5222edeeb14cf25634ad2d916a1f
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88651444"
---
# <a name="quickstart-create-apache-kafka-cluster-in-azure-hdinsight-using-azure-portal"></a>Inicio rápido: Creación de un clúster de Apache Kafka en Azure HDInsight mediante Azure Portal

[Apache Kafka](./apache-kafka-introduction.md) es una plataforma de streaming distribuida y de código abierto. A menudo se usa como agente de mensajes, ya que proporciona una funcionalidad similar a una cola de mensajes de publicación o suscripción.

En este inicio rápido aprenderá a crear un clúster de Apache Kafka mediante Azure Portal. También aprenderá a usar las utilidades incluidas para enviar y recibir mensajes con Apache Kafka. Para una explicación detallada de las configuraciones disponibles, consulte el artículo sobre la [configuración de clústeres en HDInsight](../hdinsight-hadoop-provision-linux-clusters.md). Para más información sobre el uso del portal para crear clústeres, consulte el artículo sobre la [creación de clústeres en el portal](../hdinsight-hadoop-create-linux-clusters-portal.md).

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

Solo los recursos dentro de la misma red virtual pueden tener acceso a la API de Apache Kafka. En este inicio rápido, accederá al clúster directamente mediante SSH. Para conectar otros servicios, redes o máquinas virtuales con Apache Kafka, primero debe crear una red virtual y, a continuación, crear los recursos dentro de la red. Para más información, consulte el documento [Conexión a Kafka en HDInsight mediante una instancia de Azure Virtual Network](apache-kafka-connect-vpn-gateway.md). Para más información general sobre la planificación de redes virtuales para HDInsight, consulte [Planificación de una red virtual para Azure HDInsight](../hdinsight-plan-virtual-network-deployment.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Un cliente SSH. Para más información, consulte [Conexión a través de SSH con HDInsight (Apache Hadoop)](../hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="create-an-apache-kafka-cluster"></a>Creación de un clúster de Apache Kafka

Para crear un clúster de Apache Kafka en HDInsight, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. En el menú superior, seleccione **+ Crear un recurso**.

    ![Creación de recursos de HDInsight en Azure Portal](./media/apache-kafka-get-started/azure-portal-create-resource.png)

1. Seleccione **Analytics** > **Azure HDInsight** para ir a la página **Crear clúster de HDInsight**.

1. En la pestaña **Básico**, especifique la siguiente información:

    |Propiedad  |Descripción  |
    |---------|---------|
    |Subscription    |  En la lista desplegable, seleccione la suscripción de Azure que se usa para el clúster. |
    |Resource group     | Cree un grupo de recursos o seleccione uno existente.  Un grupo de recursos es un contenedor de componentes de Azure.  En este caso, el grupo de recursos contiene el clúster de HDInsight y la cuenta de Azure Storage dependiente. |
    |Nombre del clúster   | Escriba un nombre único global. El nombre puede tener un máximo de 59 caracteres, letras, números y guiones incluidos. El primer y el último carácter del nombre no pueden ser guiones. |
    |Region    | En la lista desplegable, seleccione una región donde crear el clúster.  Elija la región más cercana para mejorar el rendimiento. |
    |Tipo de clúster| Seleccione **Seleccionar tipo de clúster** para abrir una lista. En ella, seleccione **Kafka** como tipo de clúster.|
    |Versión|Se especificará la versión predeterminada para el tipo de clúster. Seleccione en la lista desplegable si desea especificar una versión diferente.|
    |Nombre de usuario y contraseña de inicio de sesión del clúster    | El nombre de inicio de sesión predeterminado es **admin**. La contraseña debe tener un mínimo de 10 caracteres y contener al menos un dígito, una letra mayúscula y una letra minúscula, y un carácter no alfanumérico (excepto los caracteres ' " y `\). Asegúrese de **no proporcionar** contraseñas comunes, como "Pass@word1".|
    |Nombre de usuario de Secure Shell (SSH) | El nombre de usuario predeterminado es **sshuser**.  Puede proporcionar otro nombre para el nombre de usuario de SSH. |
    |Uso de la contraseña de inicio de sesión del clúster para SSH| Seleccione esta casilla para que el usuario de SSH tenga la misma contraseña que la proporcionada para el usuario de inicio de sesión del clúster.|

   ![Conceptos básicos de creación de clústeres de Azure Portal](./media/apache-kafka-get-started/azure-portal-cluster-basics.png)

    Cada región de Azure (ubicación) proporciona _dominios de error_. Un dominio de error es una agrupación lógica del hardware subyacente en un centro de datos de Azure. Todos los dominios de error comparten la fuente de energía y el conmutador de red. Las máquinas virtuales y los discos administrados que implementan los nodos en un clúster de HDInsight se distribuyen por estos dominios de error. Esta arquitectura limita el impacto potencial de errores del hardware físico.

    Para lograr la alta disponibilidad de los datos, seleccione una ubicación (región) que contenga __tres dominios de error__. Para información sobre el número de dominios de error de una región, consulte el documento sobre la [disponibilidad de las máquinas virtuales Linux](../../virtual-machines/windows/manage-availability.md#use-managed-disks-for-vms-in-an-availability-set).

    Seleccione el botón **Siguiente: Almacenamiento>>** para avanzar a la configuración de almacenamiento.

1. En la pestaña **Almacenamiento**, proporcione los valores siguientes:

    |Propiedad  |Descripción  |
    |---------|---------|
    |Tipo de almacenamiento principal|Use el valor predeterminado **Azure Storage**.|
    |Método de selección|Use el valor predeterminado **Seleccionar de la lista**.|
    |Cuenta de almacenamiento principal|Utilice la lista desplegable para seleccionar una cuenta de almacenamiento existente o bien elija **Crear nuevo**. Si crea una cuenta nueva, el nombre debe tener una longitud de entre 3 y 24 caracteres y solo puede contener números y letras minúsculas.|
    |Contenedor|Use el valor que se rellena automáticamente.|

    ![HDInsight Linux: introducción a la especificación de valores de almacenamiento en clúster](./media/apache-kafka-get-started/azure-portal-cluster-storage.png "Especificación de valores de almacenamiento para crear un clúster de HDInsight")

    Seleccione la pestaña **Seguridad y redes**.

1. En este inicio rápido, deje la configuración de seguridad predeterminada. Para más información acerca de Enterprise Security Package, visite [Configurar un clúster de HDInsight con Enterprise Security Package mediante Azure Active Directory Domain Services](../domain-joined/apache-domain-joined-configure-using-azure-adds.md). Para aprender a usar su propia clave para el Cifrado de disco de Apache Kafka, visite [Cifrado de disco mediante claves administradas por el cliente](../disk-encryption.md).

   Si desea conectar su clúster a una red virtual, seleccione una red virtual desde la lista desplegable **Red virtual**.

   ![Incorporación de un clúster a una red virtual](./media/apache-kafka-get-started/azure-portal-cluster-security-networking-kafka-vnet.png)

    Seleccione la pestaña **Configuration + pricing** (Configuración y precios).

1. Para garantizar la disponibilidad de Apache Kafka en HDInsight, la entrada del __número de nodos__ para **Nodo de trabajo** debe establecerse en tres o más. El valor predeterminado es 4.

    La entrada **Standard disks per worker node** (Discos estándar por nodo de trabajo) configura la escalabilidad de Apache Kafka en HDInsight. Apache Kafka en HDInsight usa el disco local de las máquinas virtuales del clúster para almacenar datos. Como Apache Kafka tiene muchas E/S, [Azure Managed Disks](../../virtual-machines/managed-disks-overview.md) se usa para proporcionar alto rendimiento y mayor espacio de almacenamiento por nodo. El tipo de disco administrado puede ser __Estándar__ (HDD) o __Premium__ (SSD). El tipo de disco depende del tamaño de máquina virtual que usan los nodos de trabajo (agentes de Apache Kafka). Los discos Premium se usan automáticamente con máquinas virtuales de las series DS y GS. Todos los otros tipos de máquina virtual usan discos estándar.

   ![Establecimiento del tamaño de clúster de Apache Kafka](./media/apache-kafka-get-started/azure-portal-cluster-configuration-pricing-kafka.png)

    Seleccione la pestaña **Revisar y crear**.

1. Revise la configuración del clúster. Cambie la configuración que sea incorrecta. Por último, seleccione **Crear** para crear el clúster.

    ![Resumen de configuración del clúster de Kafka](./media/apache-kafka-get-started/azure-portal-cluster-review-create-kafka.png)

    Un clúster puede tardar hasta 20 minutos en crearse.

## <a name="connect-to-the-cluster"></a>Conectarse al clúster

1. Use el [comando SSH](../hdinsight-hadoop-linux-use-ssh-unix.md) para conectarse al clúster. Modifique el comando siguiente: reemplace CLUSTERNAME por el nombre del clúster y, luego, escriba el comando:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Cuando se le solicite, escriba la contraseña del usuario de SSH.

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


    Welcome to Apache Kafka on HDInsight.

    Last login: Thu Mar 29 13:25:27 2018 from 108.252.109.241
    ```

## <a name="get-the-apache-zookeeper-and-broker-host-information"></a><a id="getkafkainfo"></a>Obtención de la información del host de Apache Zookeeper y del agente

Cuando se trabaja con Kafka, debe conocer los hosts de *Apache Zookeeper* y del *agente*. Estos hosts se usan con la API de Apache Kafka y muchas de las utilidades que se incluyen con Kafka.

En esta sección, obtendrá la información de host de la API de REST de Apache Ambari en el clúster.

1. Instale [jq](https://stedolan.github.io/jq/), un procesador JSON de línea de comandos. Esta utilidad se usa para analizar documentos JSON y es útil para analizar la información de host. En la conexión SSH abierta, escriba el siguiente comando para instalar `jq`:

    ```bash
    sudo apt -y install jq
    ```

1. Configure una variable de contraseña. Reemplace `PASSWORD` por la contraseña de inicio de sesión del clúster y, después, escriba el comando:

    ```bash
    export password='PASSWORD'
    ```

1. Extraiga el nombre del clúster con las mayúsculas y minúsculas correctas. Las mayúsculas y minúsculas reales del nombre del clúster pueden no ser como cabría esperar, dependen de la forma en que se haya creado el clúster. Este comando obtendrá las mayúsculas y minúsculas reales y después las almacenará en una variable. Escriba el comando siguiente:

    ```bash
    export clusterName=$(curl -u admin:$password -sS -G "http://headnodehost:8080/api/v1/clusters" | jq -r '.items[].Clusters.cluster_name')
    ```

    > [!Note]  
    > Si está realizando este proceso desde fuera del clúster, hay un procedimiento diferente para almacenar el nombre del clúster. Obtenga el nombre del clúster en minúsculas desde Azure Portal. A continuación, sustituya el nombre del clúster por `<clustername>` en el siguiente comando y ejecútelo: `export clusterName='<clustername>'`.


1. Para establecer una variable de entorno con la información de host de Zookeeper, use el comando siguiente. El comando recupera todos los hosts de Zookeeper y, a continuación, devuelve solo las dos primeras entradas. Esto se debe a que quiere cierta redundancia en caso de que un host sea inaccesible.

    ```bash
    export KAFKAZKHOSTS=$(curl -sS -u admin:$password -G https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/ZOOKEEPER/components/ZOOKEEPER_SERVER | jq -r '["\(.host_components[].HostRoles.host_name):2181"] | join(",")' | cut -d',' -f1,2);
    ```

    > [!Note]  
    > Este comando requiere acceso a Ambari. Si el clúster se encuentra detrás de un grupo de seguridad de red, ejecute este comando desde una máquina que pueda acceder a Ambari.

1. Para comprobar que la variable de entorno se ha establecido correctamente, use el comando siguiente:

    ```bash
    echo $KAFKAZKHOSTS
    ```

    Este comando devuelve información similar al texto siguiente:

    `zk0-kafka.eahjefxxp1netdbyklgqj5y1ud.ex.internal.cloudapp.net:2181,zk2-kafka.eahjefxxp1netdbyklgqj5y1ud.ex.internal.cloudapp.net:2181`

1. Para establecer una variable de entorno con la información de host del agente de Apache Kafka, use el comando siguiente:

    ```bash
    export KAFKABROKERS=$(curl -sS -u admin:$password -G https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2);
    ```

    > [!Note]  
    > Este comando requiere acceso a Ambari. Si el clúster se encuentra detrás de un grupo de seguridad de red, ejecute este comando desde una máquina que pueda acceder a Ambari.

1. Para comprobar que la variable de entorno se ha establecido correctamente, use el comando siguiente:

    ```bash
    echo $KAFKABROKERS
    ```

    Este comando devuelve información similar al texto siguiente:

    `wn1-kafka.eahjefxxp1netdbyklgqj5y1ud.cx.internal.cloudapp.net:9092,wn0-kafka.eahjefxxp1netdbyklgqj5y1ud.cx.internal.cloudapp.net:9092`

## <a name="manage-apache-kafka-topics"></a>Administración de temas de Apache Kafka

Kafka almacena flujos de datos en *temas*. Puede usar la utilidad `kafka-topics.sh` para administrar temas.

* **Para crear un tema**, use el comando siguiente en la conexión SSH:

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic test --zookeeper $KAFKAZKHOSTS
    ```

    Este comando se conecta a Zookeeper mediante la información de host almacenada en `$KAFKAZKHOSTS`. Y, luego, crea un tema de Apache Kafka llamado **test**.

    * Los datos almacenados en este tema se dividen en ocho particiones.

    * Cada partición se replica en tres nodos de trabajo del clúster.

        * Si ha creado el clúster en una región de Azure que proporciona tres dominios de error, use un factor de replicación de 3. De lo contrario, use un factor de replicación de 4.
        
        * En regiones con tres dominios de error, un factor de replicación de 3 permite que las réplicas se distribuyan entre los dominios de error. En regiones con dos dominios de error, un factor de replicación de cuatro permite que las réplicas se distribuyan equitativamente entre los dominios.
        
        * Para información sobre el número de dominios de error de una región, consulte el documento sobre la [disponibilidad de las máquinas virtuales Linux](../../virtual-machines/windows/manage-availability.md#use-managed-disks-for-vms-in-an-availability-set).

        * Apache Kafka no es compatible con dominios de error de Azure. Al crear réplicas de la partición de temas, puede que estas no se distribuyan correctamente con alta disponibilidad.

        * Para garantizar la alta disponibilidad, use la [herramienta de reequilibrado de particiones de Apache Kafka](https://github.com/hdinsight/hdinsight-kafka-tools). Esta herramienta se debe ejecutar desde una conexión SSH en el nodo principal del clúster de Apache Kafka.

        * Para obtener la máxima disponibilidad de los datos de Apache Kafka, debe reequilibrar las réplicas de las particiones del tema cuando:

            * Cree un nuevo tema o una partición

            * Escale verticalmente un clúster

* **Para mostrar temas**, use el comando siguiente:

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --list --zookeeper $KAFKAZKHOSTS
    ```

    Este comando muestra los temas disponibles en el clúster de Apache Kafka.

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

Kafka almacena *registros* en temas. Los registros se generan mediante *productores* y se consumen mediante *consumidores*. Los productores y consumidores se comunican con el servicio de *agente de Kafka*. Cada nodo de trabajo del clúster de HDInsight es un host de agente de Apache Kafka.

Use los pasos siguientes para almacenar registros en el tema de prueba que creó anteriormente y luego leerlos mediante un consumidor:

1. Para escribir registros en el tema, use la utilidad `kafka-console-producer.sh` desde la conexión SSH:

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --broker-list $KAFKABROKERS --topic test
    ```

    Después de este comando, llega a una línea vacía.

2. Escriba un mensaje de texto en la línea vacía y presione ENTRAR. Escriba algunos mensajes de esta forma y, a continuación, use **Ctrl + C** para volver al símbolo del sistema normal. Cada línea se envía como un registro independiente al tema de Apache Kafka.

3. Para leer registros del tema, use la utilidad `kafka-console-consumer.sh` desde la conexión SSH:

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --bootstrap-server $KAFKABROKERS --topic test --from-beginning
    ```

    Este comando recupera los registros del tema y los muestra. Con `--from-beginning` se indica al consumidor que comience desde el principio del flujo, de modo que se recuperan todos los registros.

    Si está utilizando una versión anterior de Kafka, reemplace `--bootstrap-server $KAFKABROKERS` por `--zookeeper $KAFKAZKHOSTS`.

4. Use __Ctrl + C__ para detener el consumidor.

También puede crear mediante programación los productores y consumidores. Para obtener un ejemplo del uso de esta API, consulte el documento [Producer y Consumer API de Apache Kafka](apache-kafka-producer-consumer-api.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Para limpiar los recursos creados por esta guía de inicio rápido, puede eliminar el grupo de recursos. Al eliminar el grupo de recursos, también se elimina el clúster de HDInsight asociado y otros recursos asociados al grupo.

Para quitar el grupo de recursos mediante Azure Portal:

1. En Azure Portal, expanda el menú en el lado izquierdo para abrir el menú de servicios y elija __Grupos de recursos__ para mostrar la lista de sus grupos de recursos.
2. Busque el grupo de recursos que desea eliminar y haga clic con el botón derecho en __Más__ (...) en el lado derecho de la lista.
3. Seleccione __Eliminar grupo de recursos__ y confirme la elección.

> [!WARNING]  
> Al eliminar un clúster de Apache Kafka en HDInsight, se eliminan todos los datos almacenados en Kafka.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Uso de Apache Spark con Apache Kafka](../hdinsight-apache-kafka-spark-structured-streaming.md)

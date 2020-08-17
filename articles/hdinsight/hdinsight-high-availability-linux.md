---
title: Alta disponibilidad de Hadoop en Azure HDInsight
description: Obtenga información sobre cómo los clústeres de HDInsight mejoran la confiabilidad y la disponibilidad gracias al uso de un nodo principal extra. Obtenga información sobre cómo esto afecta a los servicios de Hadoop como Ambari y Hive, y cómo conectarse individualmente a cada nodo principal mediante SSH.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
keywords: hadoop alta disponibilidad
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 10/28/2019
ms.openlocfilehash: f97b4877d87acec923812fd0289d5161529dbe7a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87081039"
---
# <a name="availability-and-reliability-of-apache-hadoop-clusters-in-hdinsight"></a>Disponibilidad y fiabilidad de clústeres de Apache Hadoop en HDInsight

Los clústeres de HDInsight proporcionan dos nodos principales para aumentar la disponibilidad y la confiabilidad de los servicios y trabajos de Apache Hadoop en ejecución.

Hadoop logra una alta disponibilidad y confiabilidad al replicar datos y servicios en varios nodos de un clúster. Sin embargo, las distribuciones estándar de Hadoop suelen tener un único nodo principal. Cualquier interrupción de ese nodo principal puede causar que el clúster deje de funcionar. HDInsight proporciona dos nodos principales para mejorar la disponibilidad y la confiabilidad de Hadoop.

## <a name="availability-and-reliability-of-nodes"></a>Disponibilidad y confiabilidad de los nodos

Los nodos de un clúster de HDInsight se implementan mediante Azure Virtual Machines. En las secciones siguientes se describen los tipos de nodo individuales usados con HDInsight.

> [!NOTE]  
> No todos los tipos de nodo se utilizan para un tipo de clúster. Por ejemplo, un tipo de clúster de Hadoop no tiene ningún nodo Nimbus. Para más información sobre los nodos usados por los tipos de clúster de HDInsight, vea la sección Tipos de clúster en el documento [Creación de clústeres de Hadoop basados en Linux en HDInsight](hdinsight-hadoop-provision-linux-clusters.md#cluster-type).

### <a name="head-nodes"></a>Nodos principales

HDInsight proporciona dos nodos principales para garantizar una alta disponibilidad de los servicios de Hadoop. Ambos nodos principales están activos y en ejecución dentro del clúster de HDInsight al mismo tiempo. Algunos servicios, como Apache HDFS o Apache Hadoop YARN, solo están “activos” en un nodo principal en un determinado momento. Otros servicios como HiveServer2 o MetaStore de Hive están activos en ambos nodos principales al mismo tiempo.

Para obtener los nombres de host de los distintos tipos de nodo del clúster, use la [API de REST de Ambari](hdinsight-hadoop-manage-ambari-rest-api.md#get-the-fqdn-of-cluster-nodes).

> [!IMPORTANT]  
> No asocie el valor numérico a si un nodo es principal o secundario. El valor numérico solo está presente para proporcionar un nombre único para cada nodo.

### <a name="nimbus-nodes"></a>Nodos Nimbus

Los nodos Nimbus están disponibles con los clústeres de Apache Storm. Los nodos Nimbus proporcionan una funcionalidad similar a la de JobTracker de Hadoop al distribuir y supervisar el procesamiento a través de nodos de trabajo. HDInsight proporciona dos nodos Nimbus de clústeres de Storm.

### <a name="apache-zookeeper-nodes"></a>Nodos Apache ZooKeeper

Los nodos [ZooKeeper](https://zookeeper.apache.org/) sirven para seleccionar el líder de los servicios principales en los nodos principales. También sirven para garantizar que los servicios, los nodos de datos (trabajo) y las puertas de enlace saben en qué nodo principal está activo un servicio principal. De forma predeterminada, HDInsight proporciona tres nodos ZooKeeper.

### <a name="worker-nodes"></a>Nodos de trabajo

Los nodos de trabajo realizan el análisis de los datos reales cuando se envía un trabajo al clúster. Si se produce un error en un nodo de trabajo, la tarea que estaba realizando se envía a otro nodo de trabajo. De forma predeterminada, HDInsight crea cuatro nodos de trabajo. Puede cambiar este número para satisfacer sus necesidades durante y después de la creación del clúster.

### <a name="edge-node"></a>Nodo perimetral

Un nodo perimetral no participa activamente en el análisis de datos dentro del clúster, sino que lo usan desarrolladores o científicos de datos al trabajar con Hadoop. El nodo perimetral se encuentra en la misma instancia de Azure Virtual Network como los demás nodos del clúster y puede acceder directamente a todos los demás nodos. El nodo perimetral se puede usar sin tener que quitar recursos a los trabajos de análisis o servicios críticos de Hadoop.

Actualmente, Machine Learning Services en HDInsight es el único tipo de clúster que proporciona un nodo perimetral de forma predeterminada. Para Machine Learning Services en HDInsight, se usa el nodo perimetral para probar el código de R localmente en el nodo antes de enviarlo al clúster para su procesamiento distribuido.

Para más información sobre el uso de un nodo perimetral con otros tipos de clúster, consulte el documento [Uso de nodos perimetrales en HDInsight](hdinsight-apps-use-edge-node.md).

## <a name="accessing-the-nodes"></a>Acceso a los nodos

Se proporciona acceso al clúster a través de Internet mediante una puerta de enlace pública. El acceso está limitado a la conexión a los nodos principales y, si existe, al nodo perimetral. El hecho de contar con varios nodos principales no afecta al acceso a servicios que se ejecutan en los nodos principales. La puerta de enlace pública enruta las solicitudes al nodo principal que hospeda el servicio solicitado. Por ejemplo, si Apache Ambari está hospedado en el nodo principal secundario, la puerta de enlace enruta las solicitudes entrantes de Ambari a ese nodo.

El acceso a través de la puerta de enlace pública se limita a los puertos 443 (HTTPS), 22 y 23.

|Port |Descripción |
|---|---|
|443|Se usa para acceder a Ambari y a otra interfaz de usuario web o API de REST hospedadas en los nodos principales.|
|22|Se usa para acceder al nodo principal primario o al nodo perimetral mediante SSH.|
|23|Se usa para acceder al nodo principal secundario mediante SSH. Por ejemplo, `ssh username@mycluster-ssh.azurehdinsight.net` se conecta al nodo principal primario del clúster llamado **mycluster**.|

Para más información sobre cómo usar SSH, vea el documento [Uso de SSH con HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

### <a name="internal-fully-qualified-domain-names-fqdn"></a>Nombres completos de dominio (FQDN) internos

Los nodos de un clúster de HDInsight tienen una dirección IP interna y el FQDN al que solo se puede acceder desde el clúster. Al obtener acceso a servicios en el clúster mediante la dirección IP o FQDN interna, debe usar Ambari para comprobar la dirección IP o FQDN que se usará al obtener acceso al servicio.

Por ejemplo, el servicio de Apache Oozie solo puede ejecutarse en un nodo principal y el uso del comando `oozie` desde una sesión de SSH requiere la dirección URL del servicio. La dirección URL puede conseguirse en Ambari mediante el comando siguiente:

```bash
export password='PASSWORD'
export clusterName="CLUSTERNAME"

curl -u admin:$password "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/configurations?type=oozie-site&tag=TOPOLOGY_RESOLVED" | grep oozie.base.url
```

Este comando devuelve un valor similar al siguiente, que contiene la dirección URL interna para usarla con el comando `oozie`:

```output
"oozie.base.url": "http://<ACTIVE-HEADNODE-NAME>cx.internal.cloudapp.net:11000/oozie"
```

Para más información sobre cómo trabajar con la API de REST de Ambari, vea [Supervisión y administración de HDInsight con la API de REST de Apache Ambari](hdinsight-hadoop-manage-ambari-rest-api.md).

### <a name="accessing-other-node-types"></a>Acceso a otros tipos de nodos

Puede conectarse a los nodos que no son accesibles directamente a través de Internet mediante los métodos siguientes:

|Método |Descripción |
|---|---|
|SSH|una vez conectado a un nodo principal mediante SSH, puede usar SSH desde el nodo principal para conectarse a otros nodos del clúster. Para más información, vea el documento [Uso de SSH con HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).|
|Tunelización SSH|Si tiene que acceder a un servicio web hospedado en uno de los nodos que no está expuesto a Internet, debe usar un túnel SSH. Para más información, vea el documento [Uso de un túnel SSH con HDInsight](hdinsight-linux-ambari-ssh-tunnel.md).|
|Azure Virtual Network|si el clúster de HDInsight forma parte de una instancia de Azure Virtual Network, cualquier recurso en la misma red virtual puede acceder directamente a todos los nodos del clúster. Para más información, consulte el documento [Planeamiento de una red virtual para HDInsight](hdinsight-plan-virtual-network-deployment.md).|

## <a name="how-to-check-on-a-service-status"></a>Cómo comprobar el estado del servicio

Para comprobar el estado de los servicios que se ejecutan en el nodo principal, use la interfaz de usuario web de Ambari o la API de REST de Ambari.

### <a name="ambari-web-ui"></a>Interfaz de usuario web de Ambari

La UI web de Ambari se puede ver en `https://CLUSTERNAME.azurehdinsight.net`. Reemplace **CLUSTERNAME** por el nombre del clúster. Si se le solicita, introduzca las credenciales de usuario HTTP de su clúster. El nombre de usuario HTTP predeterminado es **admin** y la contraseña es la contraseña que especificó al crear el clúster.

Cuando llegue a la página de Ambari, se enumeran los servicios instalados a la izquierda de la página.

![Servicios instalados de Apache Ambari](./media/hdinsight-high-availability-linux/hdinsight-installed-services.png)

Hay una serie de iconos que pueden aparecer junto a un servicio para indicar el estado. Las alertas relacionadas con un servicio se pueden ver mediante el vínculo **Alertas** que se encuentra en la parte superior de la página.  Ambari ofrece varias alertas predefinidas.

Las siguientes alertas ayudan a supervisar la disponibilidad de un clúster:

| Nombre de la alerta                               | Descripción                                                                                                                                                                                  |
|------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Estado del monitor de métricas                    | Esta alerta indica el estado del proceso de supervisión de las métricas según lo determinado por el script de estado del monitor.                                                                                   |
| Latido del agente de Ambari                   | Esta alerta se desencadena si el servidor ha perdido el contacto con un agente.                                                                                                                        |
| Proceso de servidor de ZooKeeper                 | Esta alerta de nivel de host se desencadena si no se puede confirmar que el proceso de servidor de ZooKeeper está activo y en escucha en la red.                                                               |
| Estado del servidor de metadatos de IOCache           | Esta alerta de nivel de host se desencadena si no se puede confirmar que el servidor de metadatos de IOCache está activo y respondiendo a las solicitudes de cliente.                                                            |
| Interfaz de usuario web de JournalNode                       | Esta alerta de nivel de host se desencadena si no se puede acceder a la interfaz de usuario web de JournalNode.                                                                                                                 |
| Servidor Thrift de Spark2                     | Esta alerta de nivel de host se desencadena si no se puede confirmar que el servidor Thrift de Spark2 está activo.                                                                                                |
| Proceso del servidor de historial                   | Esta alerta de nivel de host se desencadena si no se puede confirmar que el proceso de servidor de historial está activo y en escucha en la red.                                                                |
| Interfaz de usuario web del servidor de historial                    | Esta alerta de nivel de host se desencadena si no se puede acceder a la interfaz de usuario web del servidor de historial.                                                                                                              |
| Interfaz de usuario web de `ResourceManager`                   | Esta alerta de nivel de host se desencadena si no se puede acceder a la interfaz de usuario web de `ResourceManager`.                                                                                                             |
| Resumen de estado de NodeManager               | Esta alerta de nivel de servicio se desencadena si hay elementos NodeManager en mal estado.                                                                                                                    |
| Interfaz de usuario web de la escala de tiempo de la aplicación                      | Esta alerta de nivel de host se desencadena si no se puede acceder a la interfaz de usuario web del servidor de la escala de tiempo de la aplicación.                                                                                                         |
| Resumen de estado de DataNode                  | Esta alerta de nivel de servicio se desencadena si hay elementos DataNodes en mal estado.                                                                                                                       |
| Interfaz de usuario web de NameNode                          | Esta alerta de nivel de host se desencadena si no se puede acceder a la interfaz de usuario web de NameNode.                                                                                                                    |
| Proceso del controlador de conmutación por error de ZooKeeper    | Esta alerta de nivel de host se desencadena si no se puede confirmar que el proceso del controlador de conmutación por error de ZooKeeper está activo y en escucha en la red.                                                   |
| Interfaz de usuario web del servidor de Oozie                      | Esta alerta de nivel de host se desencadena si no se puede acceder a la interfaz de usuario web del servidor de Oozie.                                                                                                                |
| Estado del servidor de Oozie                      | Esta alerta de nivel de host se desencadena si no se puede confirmar que el servidor de Oozie está activo y respondiendo a las solicitudes de cliente.                                                                      |
| Proceso de Hive Metastore                   | Esta alerta de nivel de host se desencadena si no se puede confirmar que el proceso de Hive Metastore está activo y en escucha en la red.                                                                 |
| Proceso de HiveServer2                      | Esta alerta de nivel de host se desencadena si no se puede confirmar que HiveServer está activo y respondiendo a las solicitudes de cliente.                                                                        |
| Estado del servidor de WebHCat                    | Esta alerta de nivel de host se desencadena si el estado del servidor de `templeton` no es correcto.                                                                                                            |
| Porcentaje de servidores de ZooKeeper disponibles      | Esta alerta se desencadena si el número de servidores inactivos de ZooKeeper en el clúster es mayor que el umbral crítico configurado. Agrega los resultados de las comprobaciones del proceso de ZooKeeper.     |
| Servidor Livy de Spark2                       | Esta alerta de nivel de host se desencadena si no se puede confirmar que el servidor Livy2 está activo.                                                                                                        |
| Servidor de historial de Spark2                    | Esta alerta de nivel de host se desencadena si no se puede confirmar que el servidor de historial de Spark2 está activo.                                                                                               |
| Proceso de recopilador de métricas                | Esta alerta se desencadena si no se puede confirmar que el recopilador de métricas está activo y en escucha en el puerto configurado durante un número de segundos igual que el umbral.                                 |
| Recopilador de métricas: proceso de HBase Master | Esta alerta se desencadena si no se puede confirmar que los procesos maestros de HBase del recopilador de métricas están instalados y en escucha en la red durante el umbral crítico configurado, en segundos. |
| Porcentaje de monitores de métricas disponibles       | Esta alerta se desencadena si un porcentaje de los procesos de monitores de métricas no están activos y en escucha en la red durante los umbrales críticos y de advertencia configurados.                             |
| Porcentaje de NodeManagers disponibles           | Esta alerta se desencadena si el número de elementos NodeManager inactivos en el clúster es mayor que el umbral crítico configurado. Agrega los resultados de las comprobaciones del proceso de NodeManager.        |
| Estado de NodeManager                       | Esta alerta de nivel de host comprueba la propiedad de estado del nodo disponible en el componente de NodeManager.                                                                                              |
| Interfaz de usuario web de NodeManager                       | Esta alerta de nivel de host se desencadena si no se puede acceder a la interfaz de usuario web de NodeManager.                                                                                                                 |
| Estado de disponibilidad alta de NameNode        | Esta alerta de nivel de servicio se desencadena si el elemento NameNode activo o NameNode en espera no está en ejecución.                                                                                     |
| Proceso DataNode                         | Esta alerta de nivel de host se desencadena si no se puede confirmar que los procesos individuales de DataNode están activos y en escucha en la red.                                                         |
| Interfaz de usuario web de DataNode                          | Esta alerta de nivel de host se desencadena si no se puede acceder a la interfaz de usuario web de DataNode.                                                                                                                    |
| Porcentaje de JournalNodes disponible           | Esta alerta se desencadena si el número de elementos JournalNodes inactivos en el clúster es mayor que el umbral crítico configurado. Agrega los resultados de las comprobaciones del proceso JournalNode.        |
| Porcentaje de DataNodes disponible              | Esta alerta se desencadena si el número de elementos DataNodes inactivos en el clúster es mayor que el umbral crítico configurado. Agrega los resultados de las comprobaciones del proceso DataNode.              |
| Estado del servidor Zeppelin                   | Esta alerta de nivel de host se desencadena si no se puede confirmar que el servidor de Zeppelin está activo y respondiendo a las solicitudes de cliente.                                                                   |
| Proceso interactivo de HiveServer2          | Esta alerta de nivel de host se desencadena si no se puede confirmar que HiveServerInteractive está activo y respondiendo a las solicitudes de cliente.                                                             |
| Aplicación LLAP                         | Esta alerta se desencadena si no se puede confirmar que la aplicación de LLAP está activo y respondiendo a las solicitudes.                                                                                    |

Puede seleccionar cada servicio para ver más información sobre él.

Mientras que la página del servicio proporciona información sobre el estado y la configuración de cada servicio, no proporciona información sobre en qué nodo principal se está ejecutando el servicio. Para ver esta información, use el vínculo **Hosts** de la parte superior de la página. En esta página se muestran los hosts del clúster, incluidos los nodos principales.

![Lista de hosts de nodo principal de Apache Ambari](./media/hdinsight-high-availability-linux/hdinsight-hosts-list.png)

Al seleccionar el vínculo de uno de los nodos principales se muestran los servicios y componentes que se ejecutan en ese nodo.

![Estado del componente de Apache Ambari](./media/hdinsight-high-availability-linux/hdinsight-node-services.png)

Para más información sobre el uso de Ambari, vea [Supervisión y administración de HDInsight con la interfaz de usuario web de Apache Ambari](hdinsight-hadoop-manage-ambari.md).

### <a name="ambari-rest-api"></a>API de REST de Ambari

La API de REST de Ambari está disponible en Internet. La puerta de enlace pública de HDInsight controla las solicitudes de enrutamiento dirigidas al nodo principal que hospeda actualmente la API de REST.

Puede usar el siguiente comando para comprobar el estado de un servicio a través de la API de REST de Ambari:

```bash
curl -u admin:PASSWORD https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/SERVICENAME?fields=ServiceInfo/state
```

* Reemplace **PASSWORD** por la contraseña de la cuenta del usuario (admin) HTTP.
* Reemplace **CLUSTERNAME** por el nombre del clúster.
* Reemplace **SERVICENAME** por el nombre del servicio cuyo estado quiere conocer.

Por ejemplo, para comprobar el estado del servicio **HDFS** en un clúster denominado **mycluster**, con la contraseña **password**, debería usar el comando siguiente:

```bash
curl -u admin:password https://mycluster.azurehdinsight.net/api/v1/clusters/mycluster/services/HDFS?fields=ServiceInfo/state
```

La respuesta es similar al siguiente formato JSON:

```json
{
    "href" : "http://mycluster.wutj3h4ic1zejluqhxzvckxq0g.cx.internal.cloudapp.net:8080/api/v1/clusters/mycluster/services/HDFS?fields=ServiceInfo/state",
    "ServiceInfo" : {
    "cluster_name" : "mycluster",
    "service_name" : "HDFS",
    "state" : "STARTED"
    }
}
```

La dirección URL nos indica que el servicio se está ejecutando en el nodo principal denominado **mycluster.wutj3h4ic1zejluqhxzvckxq0g**.

El estado nos indica que el servicio se está ejecutando o se ha **INICIADO**.

Si no sabe qué servicios están instalados en el clúster, puede usar el comando siguiente para recuperar una lista:

```bash
curl -u admin:PASSWORD https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services
```

Para más información sobre cómo trabajar con la API de REST de Ambari, vea [Supervisión y administración de HDInsight con la API de REST de Apache Ambari](hdinsight-hadoop-manage-ambari-rest-api.md).

#### <a name="service-components"></a>Componentes de servicio

Los servicios pueden contener componentes cuyo estado desea comprobar de forma individual. Por ejemplo, HDFS contiene el componente NameNode. Para ver información sobre un componente, el comando sería:

```bash
curl -u admin:PASSWORD https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/SERVICE/components/component
```

Si no sabe qué componentes proporciona un servicio, puede usar el comando siguiente para recuperar una lista:

```bash
curl -u admin:PASSWORD https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/SERVICE/components/component
```

## <a name="how-to-access-log-files-on-the-head-nodes"></a>Acceso a los archivos de registro en los nodos principales

### <a name="ssh"></a>SSH

Mientras está conectado a un nodo principal a través de SSH, los archivos de registro pueden encontrarse en **/var/log**. Por ejemplo, **/var/log/hadoop-yarn/yarn** contiene registros de YARN.

Cada nodo principal puede tener entradas de registro único, por lo que debe comprobar los registros en ambos.

### <a name="sftp"></a>SFTP

También se puede conectar con el nodo principal mediante el protocolo SSH File Transfer Protocol o el protocolo seguro de transferencia de archivos (SFTP) y descargar los archivos de registro directamente.

De igual forma a utilizar un cliente SSH, al conectarse al clúster debe proporcionar el nombre de cuenta de usuario SSH y la dirección SSH del clúster. Por ejemplo, `sftp username@mycluster-ssh.azurehdinsight.net`. Especifique la contraseña de la cuenta cuando se le solicite o proporcione una clave pública con el parámetro `-i`.

Una vez conectado, se le presentará un símbolo del sistema `sftp>`. Desde este símbolo del sistema, puede cambiar los directorios, cargar y descargar archivos. Por ejemplo, los siguientes comandos cambian los directorios al directorio **/var/log/hadoop/hdfs** y después descargan todos los archivos en el directorio.

```bash
cd /var/log/hadoop/hdfs
get *
```

Para ver una lista de comandos disponibles, escriba `help` en el símbolo del sistema `sftp>`.

> [!NOTE]  
> También hay interfaces gráficas que le permiten visualizar el sistema de archivos cuando se conecta mediante SFTP. Por ejemplo, [MobaXTerm](https://mobaxterm.mobatek.net/) le permite examinar el sistema de archivos mediante una interfaz similar al Explorador de Windows.

### <a name="ambari"></a>Ambari

> [!NOTE]  
> Para acceder a archivos de registro mediante Ambari, debe usar un túnel SSH. Las interfaces web de los servicios individuales no se exponen públicamente en Internet. Para más información sobre cómo usar el túnel SSH, vea el documento [Uso de la tunelación SSH](hdinsight-linux-ambari-ssh-tunnel.md).

En la interfaz de usuario web de Ambari, seleccione el servicio para el que desea consultar los registros (por ejemplo, YARN). Después, utilice **Vínculos rápidos** para seleccionar de qué nodo principal desea consultar los registros.

![Uso de vínculos rápidos para ver los registros](./media/hdinsight-high-availability-linux/quick-links-view-logs.png)

## <a name="how-to-configure-the-node-size"></a>Configuración del tamaño del nodo

El tamaño de un nodo solo se puede seleccionar durante la creación del clúster. Puede encontrar una lista de los diferentes tamaños de máquina virtual disponibles para HDInsight en la [página de precios de HDInsight](https://azure.microsoft.com/pricing/details/hdinsight/).

Al crear un clúster, puede especificar el tamaño de los nodos. A continuación se ofrece información sobre cómo especificar el tamaño mediante [Azure Portal](https://portal.azure.com/), el [módulo Az de Azure PowerShell](/powershell/azure/) y la [CLI de Azure](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest):

* **Portal de Azure**: al crear un clúster, puede establecer el tamaño de los nodos que usa el clúster:

    ![Imagen del asistente para creación de clústeres con selección del tamaño del nodo](./media/hdinsight-high-availability-linux/azure-portal-cluster-configuration-pricing-hadoop.png)

* **CLI de Azure**: cuando se usa el comando [`az hdinsight create`](https://docs.microsoft.com/cli/azure/hdinsight?view=azure-cli-latest#az-hdinsight-create), puede establecer el tamaño de los nodos principal, de trabajo y ZooKeeper mediante los parámetros `--headnode-size`, `--workernode-size` y `--zookeepernode-size`.

* **Azure PowerShell**: cuando se usa el cmdlet [New-AzHDInsightCluster](https://docs.microsoft.com/powershell/module/az.hdinsight/new-azhdinsightcluster), puede establecer el tamaño de los nodos principal, de trabajo y ZooKeeper mediante los parámetros `-HeadNodeSize`, `-WorkerNodeSize` y `-ZookeeperNodeSize`.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre los elementos que se describen en este artículo, consulte:

* [Referencia de REST de Apache Ambari](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md)
* [Instalación y configuración de la interfaz de la línea de comandos de Azure](https://docs.microsoft.com//cli/azure/install-azure-cli?view=azure-cli-latest)
* [Instale y configure el módulo Az de Azure PowerShell](/powershell/azure/)
* [Administración de HDInsight mediante Apache Ambari](hdinsight-hadoop-manage-ambari.md)
* [Aprovisionamiento de clústeres de HDInsight basado en Linux](hdinsight-hadoop-provision-linux-clusters.md)
